# RENTAL — TS + Phaser 3 架构设计文档

> 日期: 2026-07-27
> 状态: Draft（待用户审阅后确认）
> 目标: 评估从 Godot 4.6 + GDScript 迁移到 TypeScript + Phaser 3 的技术架构可行性

---

## 1. 架构总览

### 1.1 分层架构

```
┌─────────────────────────────────────────────────────┐
│  Presentation Layer (Phaser 3)                       │
│  Scenes / Sprites / Camera / Particles / Tilemap     │
│  只读 State → 渲染到 Canvas                          │
└───────────────────────┬─────────────────────────────┘
                        │ subscribe(render)
┌───────────────────────▼─────────────────────────────┐
│  Simulation Core (纯 TypeScript，零 Phaser 依赖)      │
│                                                      │
│  ┌───────────┐ ┌───────────┐ ┌───────────────────┐  │
│  │ Combat    │ │ AI        │ │ Inventory         │  │
│  │ System    │ │ System    │ │ System            │  │
│  │ (damage/  │ │ (FSM/     │ │ (grid/add/remove/ │  │
│  │  bullet/  │ │  patrol/  │ │  stack/sort/equip)│  │
│  │  ammo)    │ │  combat)  │ │                   │  │
│  └───────────┘ └───────────┘ └───────────────────┘  │
│  ┌───────────┐ ┌───────────┐ ┌───────────────────┐  │
│  │ Economy   │ │ Mission   │ │ Extraction        │  │
│  │ System    │ │ System    │ │ System            │  │
│  │ (trader/  │ │ (progress/│ │ (snapshot/        │  │
│  │  faction/ │ │  reward)  │ │  insurance)       │  │
│  │  currency)│ │           │ │                   │  │
│  └───────────┘ └───────────┘ └───────────────────┘  │
│  ┌───────────┐                                     │
│  │ Effect    │ ← 义体/套装/饰品 效果管线             │
│  │ Pipeline  │                                     │
│  └───────────┘                                     │
│                                                      │
│  [System 之间零直接引用！]                             │
│  [每个 System 只 query Entity + Component]           │
│  [System 输出 Event → 其他 System 消费]               │
└───────────────────────┬─────────────────────────────┘
                        │ load/parse
┌───────────────────────▼─────────────────────────────┐
│  Data Layer (纯 JSON + CSV parser)                   │
│  weapons.json / enemies.json / items.json            │
│  missions.json / traders.json / loot_tables.json     │
│  descriptions/*.md（运行时按需加载）                   │
└─────────────────────────────────────────────────────┘
```

### 1.2 与 Godot 架构的对照

| Godot 4.6 原架构 | TS + Phaser 3 新架构 | 改善点 |
|---|---|---|
| `GameManager` autoload（全局可变状态，1161 行） | `GameState` 纯数据对象 + `StateStore`（Immutable 更新） | 状态变更可追踪、可撤销、可序列化 |
| `player.gd` 直接读写 `GameManager.equipment` | `InventorySystem` 通过 `equipItem()` / `unequipItem()` 接口操作 | 接口边界防止全局变量污染 |
| `bullet.gd` 根据 `behavior_tags` 字符串 `split("|")` 分支 | `BehaviorTag` 接口，每个 tag 一个独立小模块 | 加新 tag 不改已有代码 |
| `effect_manager.gd` 直接改 `GameManager.runtime_stats` | `EffectPipeline` 纯函数：`(base: Stats, effects: Effect[]) => Stats` | 可单元测试，无副作用 |
| autoload 循环依赖风险 | DI 容器（`typedi` 或手动注册）明确依赖方向 | 编译期报错，不运行时炸 |
| 信号无类型约束 | TypeScript `EventBus<T>` 泛型 | `emit("enemy_killed", {id: "not_a_weapon"})` → 编译不过 |

---

## 2. 核心模块设计

### 2.1 类型系统（`src/types.ts`）

```typescript
// === Entity Component System ===
type EntityId = string;

interface Entity {
  id: EntityId;
  components: Map<ComponentType, Component>;
}

// === Components（纯数据，零逻辑）===
interface PositionComponent { x: number; y: number; rotation: number; }
interface HealthComponent { current: number; max: number; }
interface WeaponComponent { weaponId: string; ammo: AmmoState; cooldownTimer: number; }
interface EnemyAIComponent { type: EnemyType; state: AIState; targetId: EntityId | null; }
interface InventoryComponent { ownerId: string; grid: GridState; equipment: EquipmentState; }

// === Events ===
type GameEvent =
  | { type: "damage_dealt"; source: EntityId; target: EntityId; amount: number }
  | { type: "entity_died"; entityId: EntityId; killerId: EntityId }
  | { type: "item_picked_up"; entityId: EntityId; item: ItemInstance }
  | { type: "mission_progress"; missionId: string; objective: string; delta: number }
  | { type: "extraction_started"; entityId: EntityId };

// === State ===
interface GameState {
  world: WorldState;
  player: PlayerState;
  meta: MetaState; // 阵营声望 / 货币 / 保险 / 存档
}
```

### 2.2 System 接口（`src/systems/system.ts`）

```typescript
interface System<T extends GameEvent = GameEvent> {
  /** 此 System 需要读取的 Component 类型 */
  queries: ComponentQuery[];

  /** 每帧执行，纯逻辑，不碰 Phaser */
  update(state: GameState, events: T[], dt: number): T[];

  /** 可选：初始化时的副作用（如加载资源） */
  init?(): void;
}

// 示例：战斗 System
class CombatSystem implements System {
  queries = [
    { has: ["Position", "Health"], as: "damageables" },
    { has: ["Position", "Weapon"], as: "shooters" }
  ];

  update(state: GameState, events: GameEvent[], dt: number): GameEvent[] {
    const output: GameEvent[] = [];

    // 处理武器冷却
    for (const [id, entity] of state.world.entities) {
      const weapon = entity.components.get("Weapon");
      if (weapon?.cooldownTimer > 0) {
        weapon.cooldownTimer -= dt;
      }
    }

    // 处理伤害事件
    for (const event of events) {
      if (event.type === "damage_dealt") {
        const target = state.world.entities.get(event.target);
        const health = target?.components.get("Health");
        if (health) {
          health.current -= event.amount;
          if (health.current <= 0) {
            output.push({
              type: "entity_died",
              entityId: event.target,
              killerId: event.source
            });
          }
        }
      }
    }
    return output;
  }
}
```

### 2.3 Behavior Tag 模块化（`src/systems/combat/behaviors/`）

```typescript
// src/systems/combat/behaviors/behavior.ts
interface BehaviorTag {
  name: string;
  /** 修改子弹的初始参数 */
  modifyBullet?(bullet: BulletConfig): BulletConfig;
  /** 子弹命中时的副作用 */
  onHit?(context: HitContext): GameEvent[];
  /** 子弹飞行时的每帧行为 */
  onTick?(bullet: BulletState, dt: number): GameEvent[];
}

// src/systems/combat/behaviors/pierce.ts
const PierceBehavior: BehaviorTag = {
  name: "pierce",
  modifyBullet(bullet) {
    return { ...bullet, pierceCount: (bullet.pierceCount ?? 0) + 1 };
  },
  onHit(context) {
    // 穿透不减弹，继续飞行
    return [];
  }
};

// src/systems/combat/behaviors/bio_dot.ts
const BioDOTBehavior: BehaviorTag = {
  name: "bio_projectile",
  onHit(context) {
    const dotDuration = 3.0;
    const dotDamage = context.damage * 0.3;
    return [{
      type: "dot_applied",
      targetId: context.targetId,
      effect: { damage: dotDamage, duration: dotDuration, interval: 1.0 }
    }];
  }
};

// 注册表（加新 behavior 只需加一行，不改任何已有代码）
const BEHAVIOR_REGISTRY: Record<string, BehaviorTag> = {
  pierce: PierceBehavior,
  bio_projectile: BioDOTBehavior,
  splash: SplashBehavior,
  laser_beam: LaserBeamBehavior,
  charge_release: ChargeReleaseBehavior,
  scrap_railgun: ScrapRailgunBehavior,
  // ... 加新 tag 就在这里加一条
};
```

### 2.4 Effect Pipeline（`src/systems/effects/pipeline.ts`）

```typescript
// 纯函数管线：输入 base stats + effects → 输出 final stats
interface Effect {
  type: "implant" | "set_bonus" | "accessory" | "temporary";
  modifier: Partial<Record<StatKey, number>>; // +10 max_hp, -5% move_speed
  condition?: (state: GameState) => boolean;   // "仅当装备 TSAR 阵营武器时"
}

function computeEffects(
  baseStats: Stats,
  effects: Effect[],
  context: GameState
): Stats {
  let result = { ...baseStats };

  // Phase 1: source modifiers (义体加成)
  for (const e of effects) {
    if (e.type === "implant" && (!e.condition || e.condition(context))) {
      result = applyModifier(result, e.modifier);
    }
  }

  // Phase 2: equipment bonuses (套装/饰品)
  const setBonuses = collectSetBonuses(context.player.equipment);
  for (const bonus of setBonuses) {
    result = applyModifier(result, bonus);
  }

  // Phase 3: temporary effects (buff/debuff)
  for (const e of effects) {
    if (e.type === "temporary" && (!e.condition || e.condition(context))) {
      result = applyModifier(result, e.modifier);
    }
  }

  return result;
}

// 可单元测试：
// test("L-12 日光 + 能量义体应增加 20% 伤害", () => {
//   const base = { damage: 80, maxHp: 100 };
//   const effects = [{ type: "implant", modifier: { damage: 1.2 } }];
//   const result = computeEffects(base, effects, mockState);
//   expect(result.damage).toBe(96);
// });
```

### 2.5 UI 架构（HTML Overlay）

```
src/ui/
├── components/
│   ├── InventoryGrid.ts      ← 网格背包（HTML <div> + CSS Grid + Drag API）
│   ├── EquipmentPanel.ts     ← 装备槽面板
│   ├── ItemSlot.tsx          ← 单个物品格（可选 React 或 Vanilla TS）
│   ├── Terminal.ts           ← 终端基类（全屏 overlay）
│   ├── HUD.ts                ← 血条/弹药/武器栏
│   └── PauseMenu.ts
├── terminals/
│   ├── MissionTerminal.ts
│   ├── ShopTerminal.ts
│   ├── WarehouseTerminal.ts
│   ├── InsuranceTerminal.ts
│   ├── BioTerminal.ts
│   └── PrinterTerminal.ts
└── theme.ts                   ← CSS 变量（替代 despair_theme.tres）
```

**UI 通信机制**：
```
Phaser Scene (Canvas)                  HTML UI Layer (DOM)
┌──────────────────────┐              ┌─────────────────────┐
│  Input → Event       │  ──emit──►  │ Event → rerender    │
│  (wasd/mouse/click)  │              │ (React state/Vanilla)│
│                      │              │                     │
│  render(state)  ◄────│──subscribe──│  UI 不写 state       │
│  (sprites/camera)    │              │  只读 + 展示        │
└──────────────────────┘              └─────────────────────┘
```

---

## 3. 文件结构

```
rental-phaser/
├── index.html                    ← <canvas> + <div id="ui-layer">
├── package.json
├── tsconfig.json
├── vite.config.ts
│
├── src/
│   ├── main.ts                   ← 入口：初始化 Game + UI
│   │
│   ├── types/                    ← 所有 TypeScript 类型/接口定义
│   │   ├── index.ts              ← 统一导出
│   │   ├── entity.ts             ← Entity/Component/EntityId
│   │   ├── weapon.ts             ← WeaponConfig/AmmoType/FireMode
│   │   ├── item.ts               ← ItemInstance/Category/Rarity
│   │   ├── mission.ts            ← Mission/MissionStep/Reward
│   │   ├── enemy.ts              ← EnemyConfig/AIState
│   │   └── events.ts             ← GameEvent 联合类型
│   │
│   ├── data/                     ← 数据加载与解析
│   │   ├── DataLoader.ts         ← 加载 JSON + CSV + MD
│   │   ├── WeaponDB.ts           ← weapons.csv → WeaponConfig[]
│   │   ├── ItemDB.ts             ← items.csv → ItemConfig[]
│   │   ├── EnemyDB.ts            ← enemies.csv → EnemyConfig[]
│   │   └── DescriptionLoader.ts  ← 按需加载 description_id → Markdown
│   │
│   ├── state/                    ← 全局状态管理
│   │   ├── GameState.ts          ← GameState 类型 + 初始化
│   │   ├── StateStore.ts         ← Immutable 更新 + subscribe
│   │   └── SaveManager.ts        ← IndexedDB 存档/读档
│   │
│   ├── systems/                  ← 纯逻辑 System（零 Phaser 依赖）
│   │   ├── index.ts              ← System 注册 + update 循环
│   │   ├── MovementSystem.ts     ← 移动/碰撞
│   │   ├── CombatSystem.ts       ← 伤害/死亡
│   │   ├── BulletSystem.ts       ← 子弹生命周期 + behavior tag 路由
│   │   ├── AISystem.ts           ← FSM（IDLE/ALERT/COMBAT）
│   │   ├── InventorySystem.ts    ← 背包/装备/堆叠/排序
│   │   ├── EconomySystem.ts      ← 货币/声望/商店
│   │   ├── MissionSystem.ts      ← 任务进度/奖励
│   │   ├── ExtractionSystem.ts   ← 撤离/保险/快照
│   │   ├── effects/
│   │   │   ├── EffectPipeline.ts ← computeEffects 纯函数
│   │   │   ├── SetBonusDB.ts     ← 套装效果查询
│   │   │   └── AccessoryDB.ts    ← 饰品效果查询
│   │   └── combat/
│   │       ├── behaviors/        ← 每个 BehaviorTag 一个文件
│   │       │   ├── index.ts
│   │       │   ├── pierce.ts
│   │       │   ├── bio_dot.ts
│   │       │   ├── splash.ts
│   │       │   ├── laser_beam.ts
│   │       │   ├── charge_release.ts
│   │       │   └── scrap_railgun.ts
│   │       ├── FireModeResolver.ts
│   │       └── SpreadCalculator.ts
│   │
│   ├── scenes/                   ← Phaser 场景（渲染层）
│   │   ├── BootScene.ts          ← 加载数据 + 初始化 state
│   │   ├── MenuScene.ts          ← 主菜单
│   │   ├── CharacterSelectScene.ts
│   │   ├── SafeHouseScene.ts     ← 安全屋
│   │   ├── BattleScene.ts        ← 战斗
│   │   └── TrainingScene.ts      ← 训练场
│   │
│   ├── entities/                 ← Phaser 渲染实体（只读 state → 显示）
│   │   ├── PlayerSprite.ts
│   │   ├── EnemySprite.ts
│   │   ├── BulletSprite.ts
│   │   └── ExtractionPointSprite.ts
│   │
│   ├── ui/                       ← HTML overlay UI（DOM 操作）
│   │   ├── UIManager.ts          ← 统一管理所有 UI 面板
│   │   ├── InventoryUI.ts
│   │   ├── EquipmentPanelUI.ts
│   │   ├── HUD.ts
│   │   ├── PauseMenu.ts
│   │   ├── DebugMenu.ts
│   │   ├── terminals/
│   │   │   ├── TerminalBase.ts
│   │   │   ├── MissionTerminal.ts
│   │   │   ├── ShopTerminal.ts
│   │   │   ├── WarehouseTerminal.ts
│   │   │   ├── InsuranceTerminal.ts
│   │   │   ├── BioTerminal.ts
│   │   │   └── PrinterTerminal.ts
│   │   └── theme.css             ← despair 主题色
│   │
│   └── utils/                    ← 工具函数
│       ├── EventBus.ts           ← 类型安全的事件总线
│       ├── ObjectPool.ts         ← 子弹/粒子对象池
│       └── MathUtils.ts
│
├── public/
│   ├── data/                     ← 所有 JSON + CSV（构建时直接拷贝）
│   │   ├── weapons.json
│   │   ├── items.json
│   │   ├── enemies.json
│   │   ├── player.json
│   │   └── missions/
│   │       ├── chains.json
│   │       └── repeatable.json
│   ├── descriptions/             ← 51 个 .md 描述文件
│   └── sprites/                  ← 像素画（待绘制）
│
└── tests/                        ← 单元测试
    ├── CombatSystem.test.ts
    ├── InventorySystem.test.ts
    ├── EffectPipeline.test.ts
    ├── BehaviorTags.test.ts
    └── SaveManager.test.ts
```

---

## 4. 数据流与事件链

### 4.1 一帧的生命周期

```
1. InputSystem          ← Phaser Keyboard/Pointer → {W, A, S, D, mousePos, click}
2. [所有 System].update(state, events[], dt)
   ├── MovementSystem   ← InputEvents → Position 更新
   ├── AISystem         ← EnemyAI + Position → 巡逻/追击/攻击决策
   ├── CombatSystem     ← Weapon + Input → BulletSpawnEvent
   ├── BulletSystem     ← BulletSpawnEvent → BulletEntity + 命中检测
   ├── InventorySystem  ← ItemPickupEvent → 背包/装备变更
   ├── MissionSystem    ← KillEvent → 任务进度更新
   ├── ExtractionSystem ← 撤离区域检测 → 撤离/保险逻辑
   └── EconomySystem    ← 交易事件 → 货币/声望变更
3. StateStore.commit()  ← 合并所有 System 输出 → 新 state
4. Renderer             ← subscribe(state) → Phaser 渲染 + UI 更新
```

### 4.2 关键事件链示例：玩家开火击杀敌人

```
Input: mouse_click
  → CombatSystem: 检查 Weapon.ammo > 0 → 消耗 1 发 → 生成 BulletSpawnEvent
  → BulletSystem: 创建 BulletEntity（应用 SpreadCalculator + BehaviorTag.modifyBullet）
  → Phaser: 渲染子弹 sprite（位置插值）
  → 碰撞检测: bullet hits enemy
  → BulletSystem: BehaviorTag.onHit → 生成 DamageDealtEvent + 可能的 DOTAppliedEvent
  → CombatSystem: 处理 DamageDealtEvent → Enemy.Health -= damage
  → CombatSystem: Enemy.Health <= 0 → 生成 EntityDiedEvent
  → MissionSystem: 收到 EntityDiedEvent → mission progress +1 → 检查目标完成
  → EconomySystem: 收到 EntityDiedEvent → 掉落战利品（LootTable roll）
  → Phaser: 移除敌人的 sprite，播放死亡动画
  → HUD UI: 弹药数更新（subscribe state.player.equipment.weapon1.ammo）
```

---

## 5. 关键模块的复杂度对比

### 5.1 背包拖拽系统

| 维度 | Godot 版（inventory_ui.gd, 998 行） | TS + Phaser 版 |
|---|---|---|
| 网格布局 | 手写 GridContainer + 4×5 循环创建 Control 节点 | CSS Grid `grid-template-columns: repeat(4, 80px)` |
| 拖拽 | 手写 `_get_drag_data` / `_can_drop_data` / `_drop_data` | HTML Drag and Drop API / pointer events |
| 右键菜单 | 手写 PopupMenu + `id_pressed` 信号路由 | CSS `position: absolute` + 点击事件，10 行 |
| 排序 | 手写 `inventory_slots.sort_custom()` | `Array.sort()` 原生 |
| 测试 | 需要启动 Godot 运行场景 | Jest 模拟 grid state，纯函数测试 |

### 5.2 终端 UI

| 维度 | Godot 版（terminal_base.gd + 6 子类） | TS + Phaser 版 |
|---|---|---|
| 全屏覆盖 | CanvasLayer + z_index | `position: fixed; z-index: 100` |
| ESC 关闭 | `_unhandled_input` 中捕获 | `keydown` listener |
| 列表/滚动 | ScrollContainer + VBoxContainer 手动填充 | HTML `<div>` + `overflow-y: scroll` |
| 主题 | `.tres` 按节点逐一覆盖 | CSS 变量，全局改一行 |
| 按钮 | Godot Button + `add_theme_color_override` | `<button>` + CSS `:hover` / `:active` |

### 5.3 技术栈对照表

| Godot 原概念 | TS + Phaser 等价物 | 复杂度变化 |
|---|---|---|
| `CharacterBody2D` + 碰撞 | `Arcade.Physics.Body` + `collider` | 简 |
| `Area2D` + `body_entered` | `overlap()` 每帧检测 | 持平 |
| `Timer` / `await` | `setTimeout` / `async/await` | 简 |
| `.tscn` 场景 | Phaser `Scene` + 代码创建 Sprite | 换（代码量增但全控） |
| `autoload` 单例 | `StateStore` 单例 + ES module import | 好（类型安全） |
| `signal` | `EventBus<T>` 泛型 | 好（类型约束） |
| `@export var` | 构造函数参数 / JSON 配置 | 好 |
| GUT 测试 | Jest / Vitest | 好（生态成熟） |
| CSV 解析 | PapaParse 库 / 手写 parser | 持平 |

---

## 6. 技术决策

| 决策 | 方案 | 理由 |
|---|---|---|
| UI 技术 | **Vanilla HTML + CSS**（不用 React/Vue） | 减少构建依赖，AI 写原生 HTML/CSS 最熟，网格背包 CSS Grid 天然支持 |
| 状态管理 | **自定义 StateStore**（不用 Redux/Zustand） | 游戏状态更新频率高，Redux immutability overhead 太大；简单 `commit()` + `subscribe()` 够用 |
| ECS or not | **类 ECS**（Entity 有 Component Map，System 纯函数） | 不完全照搬 Bevy 的严格 ECS，但坚持 System 零引用原则 |
| 渲染层 | **Phaser 3**（不用 Pixi.js / 纯 Canvas） | 内置 Tilemap/物理/动画/摄像机/粒子，省大量手写 |
| 构建 | **Vite** | TypeScript 原生支持，HMR 热更新 |
| 存档 | **IndexedDB** | 浏览器原生，支持大文件，比 localStorage 可靠 |
| CSV→JSON | 构建时转换脚本 | CSV 编辑用 Excel，部署前转 JSON 自动去注释行 |

---

## 7. 迁移路径

### Phase 0 — 概念验证（预计 2-4 小时）

只做这个闭环：**一张地图 + 一把枪 + 一个敌人 + 一个撤离点**

实现文件：
- `BootScene.ts` — 加载 weapons.json / enemies.json
- `BattleScene.ts` — 2400×1600 tile map + Player + Enemy + Extraction
- `PlayerSprite.ts` — WASD 移动 + 鼠标瞄准
- `BulletSprite.ts` — 弹道 + 碰撞
- `EnemySprite.ts` — AI FSM（idle/alert/combat）
- `AISystem.ts` — 巡逻/追击逻辑
- `CombatSystem.ts` — 伤害/死亡
- `ExtractionSystem.ts` — 撤离检测
- `HUD.ts` — 血条/弹药
- `types/index.ts` — 基础类型

### Phase 1 — 核心系统（2-4 周）

- InventorySystem + HTML 背包 UI
- EconomySystem + 商店终端
- MissionSystem + 任务终端
- EffectPipeline + 生物终端
- InsuranceSystem + 保险终端
- SaveManager

### Phase 2 — 内容与打磨（4-8 周）

- 武器改装 + 配件系统
- 敌人变体 + Boss
- 芯片天赋树
- 多地图 + 程序化
- 美术资产绘制

---

## 8. 风险与应对

| 风险 | 概率 | 应对 |
|---|---|---|
| Phaser 3 版本升级破坏兼容 | 低 | Phaser 3 已稳定，锁定版本号 |
| HTML UI 与 Canvas 渲染同步问题 | 中 | 单向数据流（State → render），UI 只读 state |
| 大量子弹/敌人时 Canvas 性能不足 | 低 | 2D 俯视角 + 对象池足以应对 |
| 像素画美术进度拖延 | 中 | Phase 0-1 用纯色矩形占位，Phase 2 并行美术 |

---

## 9. 结论

**在技术架构层面，TS + Phaser 3 的模块化能力显著优于 Godot 4.6 + GDScript：**

1. **类型系统**：TypeScript 严格模式 + 接口 + 泛型 → 编译期抓 90% Bug
2. **模块边界**：ES modules + System 零引用原则 → 加功能不改已有代码
3. **UI 开发**：HTML/CSS overlay → AI 写 UI 效率 5x+
4. **测试能力**：Jest/Vitest 纯函数测试 → 不启动游戏验证逻辑
5. **迭代速度**：Vite HMR → 改代码瞬间刷新

**Godot 中"随便一点脚本之间的 Bug 都很难修复"的根本原因 — 全局可变状态 + 信号无类型 + 模块边界模糊 — 在新架构中不复存在。**

**迁移成本**：105 个数据文件直接复用。~10,000 行 GDScript 逻辑需重写，但同样的功能在 TS 中预计只需 ~6,000 行（类型系统减少防御性代码，模块复用减少重复，HTML UI 减少布局代码）。
