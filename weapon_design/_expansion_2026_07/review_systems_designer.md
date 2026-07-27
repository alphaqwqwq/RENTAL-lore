# systems-designer 数值与管线评审报告

> 评审对象：`design/weapon_expansion_proposal.md`（game-designer 草案 v2026-07-17）
> 评审视角：数值平衡 / 槽位定义 / 配件管线 / CSV schema 扩展 / 套装效果接入 / 货币系统
> 评审人：systems-designer
> 评审日期：2026-07-17
> 铁律：本评审不修改提案文件，不写实现代码，仅给出数值复核、接入方案与改动点清单。

---

## 一、数值平衡审查（DPS 对比表 + 离群值标注）

### 1.1 DPS 计算公式

```
direct_dps = damage × fire_rate / 60
```

| 符号 | 类型 | 范围 | 说明 |
|--------|------|-------|-------------|
| damage | int | 10~120 | weapons.csv `damage` 字段（单发基础伤害） |
| fire_rate | int | 25~1000 | weapons.csv `fire_rate` 字段（RPM，每分钟发数） |
| direct_dps | float | 30~450 | 输出范围（按现有 10 把武器观察） |

**输出范围说明**：现有 10 把武器直伤 DPS 落在 30~450 区间（提案 §1.4 锚定 30~300 为新武器目标）。DPS 不钳制，但超出 300 须有显著代价机制（蓄能/反伤/低弹匣/重载）。

**Worked example**：W1 PM-T damage=55, fire_rate=120 → 55×120/60 = **110 DPS**。

### 1.2 现有 10 把武器 DPS 基线（复核提案 §1.4）

| 武器 | damage | fire_rate | 直伤 DPS | 提案标注 | 偏差 |
|---|---|---|---|---|---|
| AK2150 | 45 | 600 | 450 | 450 | ✓ |
| RPK88 | 38 | 700 | 443 | 443 | ✓ |
| qbz99 | 32 | 650 | 347 | 347 | ✓ |
| L12 | 80 | 60 | 80 | 80 | ✓ |
| ying | 20 | 180 | 60 | 60 | ✓ |
| Milano | 10 | 200 | 33 | 33 | ✓ |
| N4 | 15 | 120 | 30 | 30 | ✓ |
| TSAR30 | 12×6 弹丸 | 45 | 324（6 弹丸）| 135（15 弹丸×9）| ⚠️ 提案标 15 弹丸×9=135，但 weapons.csv `damage=12` 且 player.gd `SHOTGUN_PELLET_COUNT=6`，实际 12×6×45/60=54 DPS（单弹丸 12 × 6 弹丸 × 45/60）。提案 135 数值依据不明，**应澄清** |
| luanhua | 25 | 30 | 12.5（单发）| 200（16 弹齐射）| ⚠️ 提案按"16 弹齐射"算 200，但 weapons.csv damage=25 fire_rate=30 → 直伤 12.5 DPS；16 发齐射是单次爆发伤害 25×16=400，按 5s 循环 ≈ 80 DPS。提案 200 数值依据不明 |
| G31 | 28 | 450 | 210 | 210 | ✓ |

**离群值标注**：提案 §1.4 现有武器 DPS 表中 TSAR30 与 luanhua 的 DPS 计算口径与 weapons.csv 数值不一致。这两把是"弹丸齐射型"武器，DPS 公式应明确：
- 单弹丸 DPS = damage × fire_rate / 60
- 齐射 DPS = damage × pellet_count × fire_rate / 60

建议 proposal 修订时统一口径，否则后续新武器 DPS 对比会失真。

### 1.3 新增 6 把武器 DPS 复核

| 武器 | damage | fire_rate | 直伤 DPS | 提案标 | 含机制后 DPS | 离群？ |
|---|---|---|---|---|---|---|
| W1 PM-T | 55 | 120 | 110 | 110 | 首发 25% 暴击 ×1.5 ≈ 165（前 3 发） | ✓ 在区间 |
| W2 C-9 | 12 | 800 | 160 | 160 | 160 + toxin_cloud 67 = 227 | ✓ 在区间（DOT 折算合理） |
| W3 L-77 | 120 | 40 | 80 | 80 | 满蓄 180/1.5s = 120 + 穿透 | ✓ 在区间 |
| **W4 K-08** | 18 | 1000 | **300** | 300 | **过载后 ×1.5 = 450** | ⚠️ **离群**：450 超出锚定上限 300 的 50%，与 RPK88(443) 持平但 RPK88 无自伤。反伤 2/s 在 80 弹匣持续 4.8s 内仅扣 9.6 HP，相对于 DPS 提升 150 来说代价过低 |
| W5 Mukhlis | 95 | 50 | 79 | 79 | ijtihad_aim 触发后 ≈ 100 | ✓ 在区间 |
| W6 El Ahorcado | 90 | 25 | 37.5 | 38 | + 3m AoE 双目标 ≈ 75 | ✓ 在区间下限，靠 AoE 补偿合理 |

**必修改点 #1（数值）**：W4 K-08 过载后 450 DPS 离群。建议二选一：
- (A) 过载倍率从 ×1.5 降至 ×1.3 → 过载后 390 DPS（仍超 300 但更接近 RPK88）
- (B) 反伤从 2/s 提升至 5/s（80 弹匣 4.8s 扣 24 HP，相当于玩家 base_hp 100 的 24%），并加"过载持续超过 3s 后自伤翻倍"机制
- 推荐 (B)，落实 "代价转嫁给碳基决策节点" 的设计意图

### 1.4 防具护甲值梯度审查

现有 4 件护甲值（items.csv effect_value）：

| 槽位 | 现有 | 现有护甲 | 重量 |
|---|---|---|---|
| helm | armor_helm_steel (common) | 10 | 2.0 |
| chest | armor_chest_kevlar (uncommon) | 25 | 4.5 |
| legs | armor_legs_tactical (common) | 12 | 2.5 |
| boots | armor_boots_reinforced (common) | 8 | 1.5 |
| **总和** | — | **55** | **10.5** |

新防具护甲值梯度：

| 防具 | rarity | 护甲 | 重量 | 护甲/重量 | 评估 |
|---|---|---|---|---|---|
| A1 aura_helm_soiree | rare | 14 | 1.2 | 11.67 | ✓ 高于 steel(5.0) 但 rare 级合理 |
| A2 aura_chest_soiree | epic | 35 | 3.5 | 10.00 | ✓ 高于 kevlar(5.56)，epic 级合理 |
| A3 aura_legs_soiree | epic | 20 | 2.0 | 10.00 | ✓ 高于 tactical(4.8)，epic 级合理 |
| A4 aura_boots_soiree | rare | 12 | 1.0 | 12.00 | ✓ 高于 reinforced(5.33)，rare 级合理 |
| A5 choice_legs_bio | uncommon | 15 | 3.0 | 5.00 | ✓ 略高于 tactical(4.8) 但重量更重，uncommon 合理 |
| A6 ilm_chest_itqan | epic | 32 | 4.0 | 8.00 | ✓ 介于 kevlar(25) 与 aura_chest(35) 之间，合理 |

**套装 B 宴会 4 件护甲总和** = 14+35+20+12 = **81**，比现有 TSAR 4 件 (55) 高 47%。
**离群值标注**：套装 B 4 件护甲 81 + 4 件套 damage_reduction ×0.85（等效护甲再 +15%），实际有效护甲 ≈ 93。考虑 epic/rare 级别 + 高价（1500+1100+700+600=3900 aura_credit）+ 建议的 durability 易损机制，**整体可接受**，但需 game-designer 确认"高阶套装允许护甲上限突破现有 common 基线 1.7 倍"。

**护甲抗性差异化（建议字段）**：提案 §5.2 提出 `damage_type_resist` 字段。强烈支持——若无此字段，所有防具仅靠 effect_value 数值差异，AURA 宴会套与 ILM itqan 套的"阵营风格"无法落到机制层。建议本字段与本提案一并落地。

### 1.5 饰品 effect_value 审查

| 饰品 | effect_type | effect_value | 实际效果 | 评估 |
|---|---|---|---|---|
| AC1 tsar_badge | stat_bonus | 5 | +5% TSAR 武器伤害 | ⚠️ 见下方问题 |
| AC2 tsar_ring | stat_bonus | 15 | max_hp +15 | ✓ 与 heart_combat(+20) 同级，饰品槽平衡 |
| AC3 aura_ring | stat_bonus | 8 | crit_chance +8% | ✓ 绝对值加成，合理 |
| AC4 aura_necklace | special_effect | 20 | 主动迷彩冷却 -20% | ✓ 乘性缩减，走 item_id 硬编码合理 |
| AC5 choice_implant | stat_bonus | 30 | max_hp +30 | ⚠️ 比 heart_combat(+20) 高 50%，但占饰品槽不占义体槽，可接受 |
| AC6 dens_overclock | skill_bonus | 2 | compute_regen +2/s | ✓ 与 brain_nc02 同数值，合理 |
| AC7 ilm_tasbih | special_effect | 10 | spread ×0.90 | ✓ 乘性，走硬编码合理 |
| AC8 harambee_kimya | special_effect | 3 | heal_on_kill +3 | ✓ 小幅回血，合理 |

**无 +100% 破坏性数值**。最高单件是 AC5 +30 max_hp（占 base_hp 100 的 30%），属合理区间。

**必修改点 #2（语义）**：AC1 tsar_badge 的 effect_type=stat_bonus 与实际语义冲突。
- `stat_bonus` 在 IMPLANT_DB 范式里是直接 stat_modifier（add/multiply），由 `recompute_player_stats()` 在结算时无条件注入 modifiers。
- 但 AC1 实际效果是"装备 TSAR 制造商武器时 +5% 伤害"——这是 **conditional effect**，取决于当前手持武器 manufacturer。
- 若按 stat_bonus 实现，玩家装备非 TSAR 武器时也会享受 +5% 伤害，违反设计意图。
- **建议**：AC1 改为 `effect_type=special_effect`，由 EffectManager 按 item_id 路由到 `_conditional_modifiers`（与套装 6 件套 conditional 同路径）。effect_value=5 仍保留为百分比含义。

---

## 二、槽位定义审查（饰品槽位方案建议）

### 2.1 现有槽位架构（关键发现）

**GameManager.VALID_EQUIPMENT_SLOTS**（`src/autoload/game_manager.gd:20`）：
```gdscript
const VALID_EQUIPMENT_SLOTS := ["helmet", "chest", "legs", "boots", "weapon1", "weapon2", "melee", "backpack", "accessory", "artifact"]
```

**关键冲突**：现有装备系统仅有 **1 个 `accessory` 槽 + 1 个 `artifact` 槽 = 2 个非护甲非武器槽**。
提案 §四要求 **6 个饰品槽**（ring×2 / necklace / implant_module / badge / trinket），与现有 schema 严重不匹配。

**这是本提案最大的接入冲突点**，必须先解决再讨论饰品落地。

### 2.2 义体终端 12 槽位与饰品槽的共存问题

**BioTerminal.SLOT_LAYOUT**（`src/ui/terminals/bio/bio_terminal.gd:7-12`）：
```gdscript
const SLOT_LAYOUT := {
    "head": ["eye", "brain", "mouth", "ear"],
    "upper": ["heart", "lung", "rib", "spine", "immune"],
    "lower": ["leg", "foot"],
    "body": ["skin"],
}
```

12 个义体槽位是解剖学位（eye/brain/heart 等），**没有"implant_module"槽位**。提案 §四说 `implant_module` "与义体系统并存但独立槽"，但未指明接入点。若把 implant_module 饰品塞进义体终端，会破坏义体槽位的解剖学语义；若塞进 accessory 槽，又只有 1 个槽不够用。

### 2.3 饰品槽位方案建议（3 个备选）

#### 方案 A：扩展 VALID_EQUIPMENT_SLOTS（推荐）

将现有 `accessory` 单槽扩展为 6 个独立槽：

```gdscript
const VALID_EQUIPMENT_SLOTS := [
    "helmet", "chest", "legs", "boots",
    "weapon1", "weapon2", "melee", "backpack",
    "ring1", "ring2", "necklace", "implant_module", "badge", "trinket"
]
# EQUIPMENT_SLOTS 常量从 10 改为 14
```

- 优点：数据结构最直接，equipment Dictionary 仍单层；存档兼容（旧存档的新槽位为 null）
- 缺点：`EQUIPMENT_SLOTS := 10` 常量在 game_manager.gd:11 多处引用，需同步改为 14；HUD 武器栏 / InventoryUI 装备槽 UI 需扩展
- 改动面：game_manager.gd + HUD.tscn/hud.gd + InventoryUI.tscn/inventory_ui.gd + bio_terminal.gd（implant_module 接入）

#### 方案 B：独立 accessory_slots Dictionary

保留现有 equipment 不变，新增 `accessory_slots: Dictionary`：

```gdscript
var accessory_slots: Dictionary = {}  # 6 槽：ring1/ring2/necklace/implant_module/badge/trinket
const VALID_ACCESSORY_SLOTS := ["ring1", "ring2", "necklace", "implant_module", "badge", "trinket"]
```

- 优点：现有 equipment 系统零改动，向后兼容性最佳
- 缺点：装备接口分裂（modify_equipment vs modify_accessory），UI 需新建独立饰品终端
- 改动面：game_manager.gd（新增接口）+ 新建 accessory_terminal.gd + 新建 AccessoryTerminal.tscn

#### 方案 C：复用 artifact 槽 + accessory 槽扩展为列表

将 accessory 槽值改为 Array，artifact 槽废弃或保留为"神器"特殊槽。技术上可行但语义混乱，**不推荐**。

**systems-designer 推荐**：方案 A。理由：
1. 数据结构最简单（单层 Dictionary），与现有 equipment 接口一致
2. 存档向后兼容：旧存档 load 时新槽位为 null（`_init_default_state` 会填充）
3. UI 改动可控：HUD 武器栏只显示 weapon1/2/melee，不影响；InventoryUI 装备槽面板需扩 4 格（ring1/ring2/necklace/implant_module/badge/trinket 选 4 显示或滚动）

### 2.4 武器 slots_available / slots_blocked / slots_required 审查

提案 6 把武器的槽位字段语法与现有 weapons.csv 完全一致（`|` 分隔，`-` 表示空）。逐项核对：

| 武器 | slots_available | slots_blocked | slots_required | 语法合规 |
|---|---|---|---|---|
| W1 PM-T | `magazine\|optic\|muzzle` | `stock\|grip` | `magazine` | ✓ |
| W2 C-9 | `stock\|magazine\|muzzle\|optic\|grip` | `-` | `magazine` | ✓ |
| W3 L-77 | `stock\|magazine\|optic` | `muzzle\|grip` | `magazine` | ✓ |
| W4 K-08 | `stock\|magazine\|optic` | `muzzle\|grip` | `magazine` | ✓ |
| W5 Mukhlis | `stock\|magazine\|muzzle\|optic` | `grip` | `magazine` | ✓ |
| W6 El Ahorcado | `muzzle\|optic` | `stock\|magazine\|grip` | `-` | ✓ |

**新武器类别 marksman_rifle / heavy_weapon 槽位配置合理性**：
- marksman_rifle（W3 L-77, W5 Mukhlis）：block muzzle 或 grip，符合"精密步枪不挂近战配件"语义 ✓
- heavy_weapon（W4 K-08）：block muzzle + grip，符合"重武器集成度高，无改装位"语义 ✓
- W6 El Ahorcado（special）：slots_required=`-`（无必装件），与 TSAR30 一致（霰弹枪也无 magazine 槽），合理 ✓

**结论**：武器槽位定义全部合规，无修改点。

### 2.5 default_attachments JSON 结构审查

| 武器 | default_attachments | 与现有 weapons.csv 一致？ |
|---|---|---|
| W1-W5 | `{"magazine":"xxx"}` | ✓ 与 AK2150/G31/L12/Milano/N4/RPK88/qbz99/ying 一致 |
| W6 | `{}` | ✓ 与 TSAR30 一致（无 magazine 槽的武器用空 JSON） |

**结论**：default_attachments 结构合规，无修改点。

---

## 三、配件管线审查（新增 behavior_tag 清单 + 代码接入点）

### 3.1 新 behavior_tag 清单与处理点

提案 §1.3.1 引入 6 个新 tag。现有 bullet.gd 仅处理 `splash` 与 `bio`（`src/entities/bullet/bullet.gd:118-121`），其余 tag 在 player.gd 中无显式处理（现有武器 tag 如 `random_jump` 通过 `recoil_type` 字段而非 tag 路由）。

| 新 tag | 含义 | 处理点 | 接入方式 |
|---|---|---|---|
| `precision_first_shot` | 静止≥2s 首发 spread=0 + 暴击+25% | `player.gd._try_fire()` + 新增静止时间跟踪 | 在 `_try_fire` 入口检查 tag，命中则临时覆盖 `current_spread=0` 并注入临时 crit_chance modifier |
| `toxin_cloud` | 命中点 1.5m 毒雾 2s 4dmg/s 可叠 3 层 | `bullet.gd._on_body_entered()` 新增 `_apply_toxin_cloud()` | 类似 `_apply_bio_dot` 但生成 Area2D 毒雾节点（持续 2s，对进入敌人每秒 4 伤害） |
| `charge_shot` | 按住蓄能 0.3~1.5s 伤害线性至 1.5× | `player.gd` 新增蓄能状态机 + `_update_shoot_input` 新分支 | 需新增 `_charge_time` 字段 + HUD 蓄能弧线（复用 reload bar） |
| `overcharge` | 累计热量满后 ×1.5 + 反伤 2/s | `player.gd` 新增热量跟踪 + `_try_fire` 注入 | 需新增 `_weapon_heat` 字段 + 散热计时器 + 自伤 take_damage 调用 |
| `ijtihad_aim` | 静止≥1.5s 暴击+20% spread×0.6 | `player.gd` 复用静止时间跟踪（与 precision_first_shot 共享） | 在 `_try_fire` 入口检查 tag，命中则临时注入 crit_chance + spread_multiplier modifier |
| `arc_projectile` | 抛物线弹道 + 3m 溅射 + 蓄力改射程 | `bullet.gd._physics_process()` 重写 + `player.gd` 蓄力 | bullet.gd 需新增 `_arc_trajectory` 字段 + 重写运动方程（gravity + 初速度） |

### 3.2 新 fire_type `charge_release` 接入

提案 §1.3.2 新增 `charge_release` 枚举值。`player.gd._update_shoot_input()`（line 251-279）现有 match 分支：`semi_auto / full_auto / shotgun_spread / melee`。需新增 `charge_release` 分支：

- 按下扳机：开始蓄能（不消耗弹药）
- 蓄能 < 0.3s 松开：取消（不消耗弹药）
- 蓄能 ≥ 0.3s 松开：激发（消耗 1 发弹药，伤害按蓄能比例）
- 蓄能 ≥ 1.5s：自动激发（最大蓄能）

**接入点**：
- `src/entities/player/player.gd._update_shoot_input()` — 新增 match 分支
- `src/entities/player/player.gd._try_fire()` — 新增 charge_release 路径（不走现有 semi_auto/full_auto 分支）
- `src/entities/player/player.gd` — 新增 `_charge_time: float` 字段 + 蓄能 HUD 弧线（复用 `_reload_progress_bar`）

### 3.3 套装效果是否需要新 behavior_tag

**不需要**。套装效果通过 `set_id` + `SET_BONUS_DB` 路由，不依赖 behavior_tag。套装的 conditional effect（如 6 件套 TSAR 武器加成）通过 `condition` 字段表达，由 `EffectManager.compute_damage()` 在伤害计算时检查武器 manufacturer。

### 3.4 配件管线审查结论

**必修改点 #3（实现工作量）**：6 个新 tag 中 4 个（charge_shot / overcharge / precision_first_shot / ijtihad_aim）需要在 player.gd 新增状态机或时间跟踪字段，2 个（toxin_cloud / arc_projectile）需要在 bullet.gd 新增节点生成或运动方程重写。这是本提案实现工作量最大的部分。

**给 godot-gdscript-specialist 的接入点清单**：
1. `src/entities/player/player.gd` — 6 处改动（_update_shoot_input 新分支 / _try_fire 入口 tag 检查 / 新增 _charge_time/_weapon_heat/_still_time 字段 / 蓄能 HUD / 自伤 take_damage / 静止时间跟踪）
2. `src/entities/bullet/bullet.gd` — 2 处改动（_apply_toxin_cloud 新方法 + Area2D 毒雾节点 / _physics_process 抛物线运动分支）
3. 新增场景：`res://src/entities/bullet/ToxinCloud.tscn`（毒雾 Area2D 节点）

---

## 四、CSV schema 扩展审查（字段清单 + DataManager 改动点）

### 4.1 提案 §5 字段清单合规性

| 新字段 | 类型 | 提案标类型 | DataManager 兼容？ | 备注 |
|---|---|---|---|---|
| `accessory_type` | string | FIELD_TYPE_STRING | ✓ | 饰品子类型路由 |
| `set_id` | string | FIELD_TYPE_STRING | ✓ | 套装归属 |
| `damage_type_resist` | string (LIST) | FIELD_TYPE_LIST | ✓ | `kinetic\|energy\|bio` 语法 |
| `durability` | int | FIELD_TYPE_INT | ✓ | -1=无限 |
| `effect_value_float` | float | FIELD_TYPE_FLOAT | ✓ | 乘性修饰用 |
| `equip_slot` | string | FIELD_TYPE_STRING | ✓ | 显式槽位 |

### 4.2 DataManager 改动点（具体文件 + 函数）

**文件**：`src/autoload/data_manager.gd`

**改动 1**：`ITEMS_FIELD_TYPES` 常量（line 72-85）追加 6 个字段：

```gdscript
const ITEMS_FIELD_TYPES := {
    # ... 现有 12 字段 ...
    "accessory_type": FIELD_TYPE_STRING,
    "set_id": FIELD_TYPE_STRING,
    "damage_type_resist": FIELD_TYPE_LIST,
    "durability": FIELD_TYPE_INT,
    "effect_value_float": FIELD_TYPE_FLOAT,
    "equip_slot": FIELD_TYPE_STRING,
}
```

**改动 2**：`_load_csv()`（line 123）逻辑无需修改——已按 header 动态读取列名（line 173-176），新增列会被自动读取。但未在 ITEMS_FIELD_TYPES 注册的列会被 `_convert_value` 走 default 分支返回 raw 字符串（line 273-274），导致类型转换不生效。所以改动 1 是落地前置条件。

**改动 3**：`validate_pipeline()`（line 334）建议追加饰品槽位校验：
- category=accessory 的 item 必须有 accessory_type 字段
- set_id 非空的 item 必须在 SET_BONUS_DB（EffectManager）中存在
- 但 SET_BONUS_DB 在 EffectManager 中，跨单例校验需谨慎，可作为 Phase 2 优化

### 4.3 向后兼容性分析

**现有 items.csv 15 行**（4 防具 + 10 弹药/医疗/投掷物/武器记录 + 1 弹药）：

| 现有行 | 新字段默认值 | 兼容性 |
|---|---|---|
| 4 防具（armor_helm_steel 等） | set_id 需回填 `tsar_suppressor`（套装 A 成员）| ⚠️ 数据迁移 |
| 10 弹药/医疗/投掷物/武器 | accessory_type="" / set_id="" / durability=-1 / effect_value_float=0.0 / equip_slot="" / damage_type_resist=[] | ✓ 全部留空即可 |

**必修改点 #4（数据迁移）**：套装 A（tsar_suppressor）6 件成员中 4 件是现有 items.csv 已有记录（armor_helm_steel / armor_chest_kevlar / armor_legs_tactical / armor_boots_reinforced）。需明确迁移策略：

- **方案 1（回填）**：给现有 4 行追加 `set_id=tsar_suppressor`。改动小（4 行 CSV），但修改了"原始数据"。
- **方案 2（新建）**：新建 4 件 `armor_helm_tsar_suppressor` 等系列防具。无数据迁移，但增加 4 件冗余防具，且玩家现有 TSAR 防具无法触发套装 A。

**systems-designer 推荐方案 1（回填）**。理由：
1. 现有 4 件 TSAR 防具本就是 common/uncommon 入门级，符合套装 A"入门级"定位
2. 数据迁移影响面小（仅 CSV 4 行加 1 列值）
3. 玩家体验连续（不需重新收集 4 件才能组套）
4. 现有存档加载时 set_id 字段缺失会读为空字符串，需在 `load_from_slot` 后由 EffectManager.recompute_player_stats() 重新结算套装（已有机制）

**必修改点 #5（CSV header 更新）**：items.csv 第 1 行 header 需追加 6 个新列名。现有 15 行数据需补齐新列值（4 防具回填 set_id，其余留空）。

### 4.4 weapons.csv schema 扩展

提案 §5.3 明确 weapons.csv **无需加列**。systems-designer 复核确认：
- 新 behavior_tags 是 `behavior_tags` 列的字符串值，FIELD_TYPE_LIST 解析已支持
- 新 fire_type `charge_release` 是 `fire_type` 列的字符串值，FIELD_TYPE_STRING 解析已支持
- 新类别 marksman_rifle / heavy_weapon 是 `category` 列的字符串值，无需枚举校验

**结论**：weapons.csv schema 零改动，仅追加 6 行数据。

---

## 五、套装效果接入方案（数据结构 + EffectManager 改动 + 是否新增单例）

### 5.1 套装触发规则接入 EffectManager._stat_registry

**现有机制**（`src/autoload/effect_manager.gd`）：
- `IMPLANT_DB` 常量（line 9-56）定义义体效果
- `_stat_registry`（line 60）注册 17 个 stat（line 98-137）
- `recompute_player_stats()`（line 186-247）遍历 `GameManager.implants`，对每个非 "original" 义体查 IMPLANT_DB 取 effects，调 `apply_effect()` 注入 modifiers
- `apply_effect()`（line 146-176）按 effect.type 路由（stat_modifier / damage / heal / cooldown）
- 最终计算顺序：先 add → 后 multiply → 最后 override（line 210-229）

**套装接入方案（不新增单例，扩展 EffectManager）**：

systems-designer 同意提案 §6.5.2 的"不新增 SetManager 单例"判断。理由：
1. 套装效果本质是 stat_modifier，与义体 effects 同路径，复用 `apply_effect()` 即可
2. 新增单例会破坏现有 EffectManager 单一职责（所有 stat 修饰集中在一处）
3. SET_BONUS_DB 与 IMPLANT_DB 模式镜像，维护成本低

### 5.2 数据结构（SET_BONUS_DB）

提案 §6.5.2 步骤 1 给出的 SET_BONUS_DB 结构合理，systems-designer 补充字段约束：

```gdscript
const SET_BONUS_DB := {
    "tsar_suppressor": {
        "members": ["armor_helm_steel", "armor_chest_kevlar", "armor_legs_tactical",
                    "armor_boots_reinforced", "tsar_badge_suppressor", "tsar_ring_steelhand"],
        "2pc": [
            {"type": "stat_modifier", "stat": "damage_multiplier", "value": 1.05,
             "operation": "multiply", "source": "set:tsar_suppressor:2pc"},
        ],
        "4pc": [
            {"type": "stat_modifier", "stat": "max_hp", "value": 20,
             "operation": "add", "source": "set:tsar_suppressor:4pc"},
            {"type": "stat_modifier", "stat": "recoil_multiplier", "value": 0.9,
             "operation": "multiply", "source": "set:tsar_suppressor:4pc"},
        ],
        "6pc": [
            {"type": "stat_modifier", "stat": "damage_multiplier", "value": 1.15,
             "operation": "multiply", "source": "set:tsar_suppressor:6pc",
             "condition": "weapon_manufacturer=TSAR"},
            {"type": "stat_modifier", "stat": "armor_penetration", "value": 0.10,
             "operation": "add", "source": "set:tsar_suppressor:6pc",
             "condition": "weapon_manufacturer=TSAR"},
        ],
    },
    "aura_soiree": { /* 同结构，省略 */ },
}
```

**补充字段**：
- `members` 数组：显式列出 6 件成员 item_id，供 `recompute_player_stats()` 校验装备件数（不依赖 set_id 字段计数，避免数据不一致）
- `condition` 字段：本提案新增的 conditional effect 机制，标注 effect 仅在条件满足时生效

**向后兼容**：SET_BONUS_DB 是 EffectManager 内部常量，不影响现有 IMPLANT_DB 与 _stat_registry。

### 5.3 新增 _stat_registry 属性

提案 §6.5.2 步骤 2 要求新增 3 个 stat。systems-designer 确认接入点：

**文件**：`src/autoload/effect_manager.gd`
**函数**：`_init_stat_registry()`（line 98-137）
**改动**：在 line 137（crit_multiplier 注册后）追加：

```gdscript
# Set bonus stats（提案扩展）
_stat_registry["armor_penetration"] = _make_stat(0.0)  # 0.0~1.0
_stat_registry["heal_on_kill"] = _make_stat(0.0)        # int
_stat_registry["damage_reduction"] = _make_stat(0.0)    # 0.0~1.0
```

**输出范围**：
- `armor_penetration`: [0.0, 1.0]，钳制（超过 1.0 无意义）
- `heal_on_kill`: [0, ∞)，不钳制（但实际单件饰品 effect_value=3，套装不叠加则上限 3）
- `damage_reduction`: [0.0, 0.95]，钳制（不允许 100% 免疫，留 5% 最低受伤）

### 5.4 recompute_player_stats() 扩展

**文件**：`src/autoload/effect_manager.gd`
**函数**：`recompute_player_stats()`（line 186-247）
**改动**：在 line 208（义体 effects 应用后）插入套装结算块：

```gdscript
# 2.5. 套装效果结算（提案扩展）
_apply_set_bonuses()
```

新增私有方法 `_apply_set_bonuses()`：
1. 遍历 `GameManager.equipment` 中 helm/chest/legs/boots/ring1/ring2/necklace/implant_module/badge/trinket 槽位
2. 收集每件装备的 `set_id`（从 items.csv 查 item_def.get("set_id", "")）
3. 按 set_id 分组计数
4. 对每个 set_id，查 SET_BONUS_DB，按命中件数（≥2/≥4/≥6）取对应阈值 effects 数组
5. 对每个 effect：
   - 无 condition 字段：调 `apply_effect(null, effect)` 注入 modifiers
   - 有 condition 字段：暂存到 `_conditional_modifiers` 数组（不注入 modifiers）

### 5.5 conditional effect 处理

**新增字段**：`var _conditional_modifiers: Array = []`（EffectManager 内部状态）

**接入点**：`compute_damage()`（line 253-264）入口处检查条件：

```gdscript
func compute_damage(base_damage: int, source: Node, target: Node) -> int:
    var damage_mult := _stat_current("damage_multiplier")
    # 套装 conditional 检查（提案扩展）
    var weapon_manufacturer := ""
    if source != null and source.has_method("get_current_weapon_id"):
        var wid := source.get_current_weapon_id()
        var wdef := DataManager.get_weapon(wid)
        weapon_manufacturer = String(wdef.get("manufacturer", ""))
    for mod in _conditional_modifiers:
        if String(mod.get("condition", "")) == "weapon_manufacturer=" + weapon_manufacturer:
            if mod.get("operation") == "multiply":
                damage_mult *= float(mod.get("value", 1.0))
            elif mod.get("operation") == "add":
                # armor_penetration 走伤害管线折减
                pass
    # ... 现有暴击判定与最终计算 ...
```

**武器切换触发重算**：player.gd `_switch_weapon()`（line 692）已有 `EventBus.weapon_switched.emit(slot)` 信号。EffectManager 需订阅此信号，触发 `recompute_player_stats()` 重算（清空 _conditional_modifiers 重新结算）。

### 5.6 compute_damage 完整管线实现（被前置的 Step 6 工作）

**关键风险**：`compute_damage()` 当前是简化实现（line 252 注释 "Step 6 再实现"）。提案 §6.5.2 步骤 5 要求：
- 读取 `armor_penetration`，按 `(1 - armor_penetration) * target.armor` 折减护甲
- 读取 `damage_reduction`，在玩家受伤时按 `incoming_damage * (1 - damage_reduction)` 折减

这意味着**本提案实际要求 Step 6 的完整伤害管线一并实现**。systems-designer 标记此项为：

**必修改点 #6（被前置的工作）**：完整伤害管线（pre → 计算 → post）原本是 STATUS.md 中 Step 6 待办。本提案的 `damage_reduction` / `armor_penetration` 依赖完整管线才能生效。需明确：
- (A) 本提案一并实现完整管线（工作量 +2~3 天）
- (B) 本提案先落地 stat 注册与 modifiers 注入，伤害管线仍走简化路径，待 Step 6 完成后再接入（damage_reduction/armor_penetration 暂不生效）
- 推荐 (A)，因为套装 4 件套 damage_reduction ×0.85 是核心卖点，不接入则套装体验断裂

### 5.7 heal_on_kill 接入

**接入点**：`src/entities/player/player.gd` 或 `src/autoload/event_bus.gd`

现有 `EventBus.enemy_killed(weapon_name, enemy_name)` 信号（STATUS.md Step 6 提到）。player.gd 订阅此信号，命中时读取 `_stat_current("heal_on_kill")`，调用 `self.heal(value)`。

**改动文件**：
- `src/entities/player/player.gd._ready()` — 连接 `EventBus.enemy_killed` 信号
- 新增 `_on_enemy_killed(weapon_name, enemy_name)` 方法

### 5.8 set_id 在 GameManager.equipment 中识别与计数

**现有 equipment Dictionary**：`equipment[slot] = item_dict或null`，item_dict 含 `item_id` 字段。

**识别流程**：
1. `EffectManager._apply_set_bonuses()` 遍历 equipment 各槽位
2. 对非 null 槽位，取 `item_id`，调 `DataManager.get_item(item_id)` 查 item_def
3. 从 item_def 取 `set_id` 字段（需 items.csv 已加 set_id 列且 DataManager 已注册字段类型）
4. 按 set_id 分组计数

**武器槽特殊处理**：weapon1/weapon2/melee 槽位的 item 含 `weapon_id` 而非 `item_id`（见 game_manager.gd:726-729 的兼容处理）。但本提案套装成员不含武器，仅防具+饰品，所以武器槽不参与套装计数。仍建议 `_apply_set_bonuses()` 跳过 weapon1/weapon2/melee/backpack 槽位。

---

## 六、货币系统审查（faction_wallet 扩展方案）

### 6.1 现有 faction_wallet 架构

**GameManager 关键定义**（`src/autoload/game_manager.gd`）：
- `const FACTIONS := ["AURA", "TSAR", "DENS", "CHOICE"]`（line 14）— 4 阵营硬编码
- `var faction_wallet: Dictionary = {}`（line 35）— Dictionary 形式，key=faction name
- `var faction_reputation: Dictionary = {}`（line 34）
- `_init_default_state()`（line 68-91）：遍历 FACTIONS 初始化 wallet/reputation 为 0
- `get_faction_wallet(faction)` / `modify_faction_wallet(faction, delta)`（line 197-209）：校验 `FACTIONS.has(faction)`，未知阵营返回 0 并 push_warning
- `set_faction_reputation(faction, value)`（line 189-193）：同样校验 FACTIONS

**关键约束**：FACTIONS 常量被 `get/set_faction_wallet/reputation` 接口用于校验，扩展第 5 货币必须同步扩展 FACTIONS。

### 6.2 ILM 迪拉姆扩展方案

**方案 A：扩展 FACTIONS 常量（推荐）**

```gdscript
const FACTIONS := ["AURA", "TSAR", "DENS", "CHOICE", "ILM"]
```

- 优点：所有现有接口（get_faction_wallet / modify_faction_wallet / set_faction_reputation）零改动，自动支持 ILM
- 缺点：FACTIONS 常量被多处引用（_init_default_state / load_from_slot / save_raid_snapshot 等），扩展后旧存档加载时 ILM 钱包为 0（合理）
- 影响面：
  - `game_manager.gd:14` — FACTIONS 常量加 "ILM"
  - `game_manager.gd:326-333`（load_from_slot）— 已按 FACTIONS 遍历，自动兼容
  - `game_manager.gd:75`（_init_default_state）— 已按 FACTIONS 遍历，自动兼容
  - `src/ui/terminals/shop/shop_terminal.gd` — 5 店铺 Tab 需新增 ILM Tab
  - `GAME_MANUAL.md §13` — 文档需更新阵营列表
- 存档兼容：旧存档无 ILM key，load_from_slot 时按 FACTIONS 遍历会自动补 ILM=0 ✓

**方案 B：独立 ilm_wallet 变量**

```gdscript
var ilm_wallet: int = 0  # 第 5 货币，独立于 faction_wallet
```

- 优点：现有 faction_wallet 系统零改动
- 缺点：接口分裂（get_ilm_wallet / modify_ilm_wallet），UI 需独立处理，存档需新增字段
- 不推荐

**systems-designer 推荐方案 A**。

### 6.3 哈拉姆贝 price=0 / currency=- 处理

**DataManager 解析行为**（`src/autoload/data_manager.gd._convert_value` line 248-274）：
- `price` 字段类型 FIELD_TYPE_INT（line 80）：raw="-" 时 `raw.is_valid_int()` 返回 false，返回 0
- `currency` 字段类型 FIELD_TYPE_STRING（line 81）：raw="-" 时返回字符串 "-"

**结论**：DataManager 可解析 price=0 / currency="-"，不会报错。但语义上 currency="-" 不优雅。

**建议**：
- currency 字段值改为 `"none"`（语义更清晰）或 `"harambee_credit"`（提案 §7.3 提到的"信用/人情"）
- 若采用 `"harambee_credit"`，需在 FACTIONS 中加 "HARAMBEE"（方案 A 扩展），faction_wallet 自动支持
- 若采用 `"none"`，shop_terminal.gd 需特殊处理（不显示购买按钮，仅显示"通过任务获得"）

**systems-designer 推荐**：currency=`"harambee_credit"`，FACTIONS 加 "HARAMBEE"。理由：
1. 与 ILM 迪拉姆扩展方案 A 一致
2. 哈拉姆贝饰品虽 price=0（不能购买），但仍可显示"哈拉姆贝信用"作为货币标识
3. 未来若哈拉姆贝扩展可购买物品，钱包已就绪

**必修改点 #7（货币扩展）**：FACTIONS 常量扩展为 `["AURA", "TSAR", "DENS", "CHOICE", "ILM", "HARAMBEE"]`（6 阵营）。影响：
- `game_manager.gd:14` — 1 行改动
- `src/ui/terminals/shop/shop_terminal.gd` — 新增 ILM + HARAMBEE 两个 Tab
- `GAME_MANUAL.md §13` — 文档更新阵营列表
- `player.csv` — `faction_wallet_init` 注释更新（值仍为 0，无改动）

### 6.4 items.csv currency 字段值审查

提案新增 14 件物品的 currency 字段值：

| 物品 | currency | 现有 FACTIONS 含？ |
|---|---|---|
| A1-A4 aura_soiree 防具 | `aura_credit` | ✓（AURA） |
| A5 choice_legs_bio | `choice_ticket` | ✓（CHOICE） |
| A6 ilm_chest_itqan | `dirham` | ⚠️ 新货币，需扩展 |
| AC1 tsar_badge | `tsar_point` | ✓（TSAR） |
| AC2 tsar_ring | `tsar_point` | ✓（TSAR） |
| AC3 aura_ring | `aura_credit` | ✓（AURA） |
| AC4 aura_necklace | `aura_credit` | ✓（AURA） |
| AC5 choice_implant | `choice_ticket` | ✓（CHOICE） |
| AC6 dens_overclock | `dens_voucher` | ✓（DENS） |
| AC7 ilm_tasbih | `dirham` | ⚠️ 新货币 |
| AC8 harambee_kimya | `-` | ⚠️ 建议改 `harambee_credit` |
| ammo_338_ilm | `dirham` | ⚠️ 新货币 |
| ammo_microfusion_dens | `dens_voucher` | ✓（DENS） |

**注意**：现有 items.csv 用 currency 值如 `tsar_point` / `choice_ticket` / `aura_credit` / `dens_voucher`，与 FACTIONS 常量值（`TSAR` / `CHOICE` / `AURA` / `DENS`）**不一致**。这是现有系统的命名分裂（FACTIONS 用全大写短名，currency 用小写长名）。shop_terminal.gd 应有映射表。新增 `dirham` 与 `harambee_credit` 时需同步更新该映射表。

---

## 七、合成结论

### 7.1 可保留的提案内容（无需修改）

1. **6 把武器数值**（除 W4 K-08 过载 DPS 离群外）：DPS 均在 30~300 锚定区间内，damage/fire_rate/magazine/reload/spread/recoil 与现有 10 把武器在同一量级
2. **6 件防具护甲值梯度**：14/35/20/12/15/32 形成合理梯度，weight/effect_value 比率符合 rarity 递增
3. **8 件饰品 effect_value**：无 +100% 破坏性数值，最高 AC5 +30 max_hp 属合理区间
4. **武器 slots_available/blocked/required 语法**：与现有 weapons.csv 完全一致
5. **default_attachments JSON 结构**：与现有 weapons.csv 一致
6. **CSV schema 扩展字段清单**（6 字段）：类型映射正确，DataManager 兼容
7. **weapons.csv schema 零改动**：新 behavior_tags / fire_type / category 均为字符串值，无需加列
8. **套装效果数据结构思路**（SET_BONUS_DB 镜像 IMPLANT_DB）：合理，不新增单例
9. **套装 2/4/6 件套触发规则**：阈值设计合理，6 件套为上限明确
10. **6 个新 behavior_tag 的 `|` 分隔语法**：与现有 weapons.csv 一致，FIELD_TYPE_LIST 解析已支持

### 7.2 必须修改后才能进实现阶段的点（7 项）

| # | 修改点 | 涉及提案章节 | 优先级 |
|---|---|---|---|
| 1 | W4 K-08 过载后 450 DPS 离群，需降倍率或提升反伤 | §二 W4 | 高 |
| 2 | AC1 tsar_badge effect_type=stat_bonus 与 conditional 语义冲突，改为 special_effect | §四 AC1 | 高 |
| 3 | 6 个新 behavior_tag 的代码接入工作量被低估，4 个需 player.gd 新增状态机 | §1.3.1 + §三 | 高 |
| 4 | 套装 A 回填 set_id 到现有 4 件 TSAR 防具的数据迁移策略需明确（推荐回填方案） | §7.5 + §六 | 高 |
| 5 | items.csv header 需追加 6 个新列名，现有 15 行需补齐新列值 | §5 | 高 |
| 6 | 完整伤害管线（compute_damage）被前置要求实现，原是 Step 6 待办 | §6.5.2 步骤 5 | 高 |
| 7 | FACTIONS 常量扩展为 6 阵营（加 ILM + HARAMBEE），影响 shop_terminal 等多处 | §7.4 + §7.3 | 高 |

**额外建议（非阻塞）**：
- 提案 §1.4 现有武器 DPS 表中 TSAR30 与 luanhua 的 DPS 计算口径需统一（单弹丸 DPS vs 齐射 DPS）
- 饰品槽位方案推荐方案 A（扩展 VALID_EQUIPMENT_SLOTS 至 14 槽）
- 套装 B 4 件护甲总和 81 比现有 TSAR 4 件 (55) 高 47%，需 game-designer 确认高阶套装允许护甲上限突破
- AC5 choice_implant effect_value=30 比 heart_combat(+20) 高 50%，可接受但建议 game-designer 复核

### 7.3 实现 phase 的工作量估算（按文件改动数）

| 文件 | 改动类型 | 工作量 |
|---|---|---|
| `assets/data/items.csv` | +14 行新数据 + 4 行回填 set_id + header 加 6 列 | 0.5 天 |
| `assets/data/weapons.csv` | +6 行新数据 | 0.2 天 |
| `assets/data/descriptions/*.md` | +20 个新描述文件 | 0.5 天 |
| `src/autoload/data_manager.gd` | ITEMS_FIELD_TYPES 追加 6 字段 | 0.1 天 |
| `src/autoload/effect_manager.gd` | SET_BONUS_DB + 3 stat + recompute 扩展 + compute_damage 完整管线 + conditional modifier | **2~3 天**（最复杂） |
| `src/autoload/game_manager.gd` | VALID_EQUIPMENT_SLOTS 扩展至 14 + FACTIONS 扩展至 6 + 饰品装备接口 | 1 天 |
| `src/entities/player/player.gd` | charge_release fire_type + 6 个新 behavior_tag 处理 + heal_on_kill 信号订阅 | **2~3 天**（最复杂） |
| `src/entities/bullet/bullet.gd` | toxin_cloud 新方法 + arc_projectile 抛物线运动 | 1 天 |
| `src/ui/terminals/bio/bio_terminal.gd` | implant_module 饰品槽接入（或新建 accessory_terminal） | 1 天 |
| `src/ui/terminals/shop/shop_terminal.gd` | 新增 ILM + HARAMBEE 两个 Tab + currency 映射表更新 | 0.5 天 |
| `src/ui/inventory/inventory_ui.gd` | 装备槽面板扩展至 14 槽 | 0.5 天 |
| `src/ui/hud/hud.gd` | 饰品栏显示（可选） | 0.3 天 |
| 新增场景 `ToxinCloud.tscn` | 毒雾 Area2D 节点 | 0.2 天 |
| `GAME_MANUAL.md` | §9 装备槽位 + §13 阵营列表更新 | 0.2 天 |
| **合计** | — | **~10~12 天**（含 2 个高复杂度文件） |

**最复杂的 1 个接入点**：`src/autoload/effect_manager.gd` 的 `compute_damage()` 完整管线实现 + 套装 conditional modifier 暂存机制。原因：
1. 该函数当前是简化实现（line 252 注释 "Step 6 再实现"），本提案要求一并实现完整管线
2. 完整管线需读取 `armor_penetration` / `damage_reduction` 两个新 stat，并接入 target.armor 折减
3. conditional modifier 暂存机制（`_conditional_modifiers` 数组）是新引入的状态，需在 recompute_player_stats 中清空、在 compute_damage 中检查、在武器切换时重算
4. 涉及 EffectManager.gd 5 处改动（_init_stat_registry / recompute_player_stats / _apply_set_bonuses / compute_damage / 信号订阅）

### 7.4 给 godot-gdscript-specialist 的接力 brief 要点

1. **CSV 改动**：items.csv header 加 6 列（accessory_type / set_id / damage_type_resist / durability / effect_value_float / equip_slot），现有 15 行补齐（4 防具 set_id=tsar_suppressor，其余留空/-1/0.0/""）。weapons.csv 加 6 行。descriptions/ 加 20 个 md。
2. **DataManager**：ITEMS_FIELD_TYPES 追加 6 字段类型映射（line 72-85）。`_load_csv` 与 `_convert_value` 无需改动。
3. **GameManager**：VALID_EQUIPMENT_SLOTS 从 10 扩至 14（加 ring1/ring2/necklace/implant_module/badge/trinket）；EQUIPMENT_SLOTS 常量从 10 改 14；FACTIONS 加 "ILM" + "HARAMBEE"；modify_equipment / get_equipment 自动兼容（已按 VALID_EQUIPMENT_SLOTS 校验）。
4. **EffectManager**：新增 SET_BONUS_DB 常量（镜像 IMPLANT_DB）；_init_stat_registry 追加 armor_penetration / heal_on_kill / damage_reduction 3 个 stat；recompute_player_stats 在义体 effects 后调 _apply_set_bonuses()；新增 _conditional_modifiers 数组；compute_damage 实现完整管线（pre → 计算 → post，含 armor_penetration 折减 + damage_reduction 免伤 + conditional 检查）；订阅 EventBus.weapon_switched 信号触发重算。
5. **player.gd**：_update_shoot_input 新增 charge_release 分支（蓄能状态机）；_try_fire 入口检查 precision_first_shot / ijtihad_aim tag（临时注入 crit/spread modifier）；新增 _charge_time / _weapon_heat / _still_time 字段；overcharge tag 处理（热量累计 + 自伤 take_damage）；订阅 EventBus.enemy_killed 信号触发 heal_on_kill。
6. **bullet.gd**：新增 _apply_toxin_cloud 方法（生成 ToxinCloud.tscn Area2D 节点）；_physics_process 新增 arc_projectile 抛物线运动分支（gravity + 初速度 + 蓄力改射程）。
7. **bio_terminal.gd**：implant_module 饰品槽接入（或新建 accessory_terminal.gd 独立终端，方案 A 下推荐扩展 bio_terminal 或新建）。
8. **shop_terminal.gd**：新增 ILM + HARAMBEE 两个 Tab；currency 映射表加 dirham / harambee_credit。
9. **测试**：tests/ 新增 test_set_bonus.gd（套装 2/4/6 件套触发）+ test_charge_release.gd（蓄能机制）+ test_overcharge.gd（过载自伤）。
10. **风险点**：compute_damage 完整管线是 Step 6 待办被前置，需与 producer 确认排期；6 个新 behavior_tag 中 4 个需 player.gd 状态机，工作量集中。

---

> 评审结束。本报告未修改提案文件，未写实现代码。必修改点 #1~#7 需 game-designer 与 systems-designer 协同修订提案后，再交 godot-gdscript-specialist 进入实现 phase。
