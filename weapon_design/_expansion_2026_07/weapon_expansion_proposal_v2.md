# 武器/防具/饰品大批量扩展提案 v2（修订版）

> 版本：2026-07-17 · 作者：game-designer · 状态：v1 经三视角审查后修订，待 creative-director 复核
> 依据：`assets/data/weapons.csv`（10 把）、`assets/data/items.csv`（4 件 TSAR 防具）、9 阵营 canon lore、`src/autoload/effect_manager.gd`、`src/autoload/data_manager.gd`、`src/autoload/game_manager.gd`、`GAME_MANUAL.md` §11/§17
> 三视角审查依据：`design/review_worldbuilder.md`、`design/review_systems_designer.md`、`design/art_direction.md`
> 铁律：本提案不写任何 GDScript 代码，不修改任何 CSV/JSON/.gd，不修改 v1 提案文件（保留为历史）。仅产出 v2 修订设计文档。

---

## 修订摘要（v1 → v2 变更清单）

### A. 数值类变更（响应 systems-designer 必修改点 #1）

| 项 | v1 | v2 | 理由 |
|---|---|---|---|
| W4 K-08 overcharge 倍率 | ×1.5（过载后 450 DPS） | **×1.3（过载后 390 DPS）** | 450 DPS 离群，390 接近 RPK88(443) 但不超 |
| W4 K-08 反伤 | 2/s（4.8s 内扣 9.6 HP） | **5/s（4.8s 内扣 24 HP）+ 过载持续 >3s 自伤翻倍至 10/s** | 反伤从"代价过低"升至"显著代价"，落实"代价转嫁给碳基决策节点"设计意图 |

### B. 叙事类变更（响应 world-builder 6 项"需调整"）

| 项 | v1 | v2 | 红线 |
|---|---|---|---|
| W5 Mukhlis 定位 | "ILM 成品枪，委员会 5:4 通过" | **"ILM 工匠用 ILM 部件 + TSAR 动作件手工组装件，非 ILM 组织成品枪；委员会 5:4 分类投票裁定为'精密器械'非'武器'（过半即可）"** | 7.1 + 7.10 |
| A5 命名 | `choice_legs_bio` "C-7 肌腱腿甲" | **`choice_legs_collagen` "胶原纤维束腿甲"** | 7.6 |
| AC5 命名 | `choice_implant_tendon` "C-7 肌腱义体" | **`choice_implant_collagen` "胶原纤维束肌腱强化"** | 7.6 |
| A5/AC5 描述 | "C-7 项目民用化分支" | **"采用与 C-7 武器同源钙化胶原纤维束材料的民用增强件"** | 7.6 剥离 C-7 品牌但保留材料同源 |
| W6 El Ahorcado 描述 | "卡特尔中层哈辛托改装件" | 追加一句 **"哈辛托个人作品，非卡特尔量产列装"** | 7.2（已通过，追加澄清） |
| 套装描述叙事框架 | "TSAR 镇暴者阵线 / AURA 宴会套装" | **明确为"部队编制识别 / 标准配发组合"，非 RPG 神装；套装描述禁止'英雄/传奇/神装'语言** | 7.8（已通过附框架约束） |

### C. 字段类变更（响应 world-builder + systems-designer）

| 项 | v1 | v2 | 红线 |
|---|---|---|---|
| W5 ammo_type | `ammo_338_ilm` | **`ammo_86_ilm`** | 7.9 改公制 mm |
| 弹药 item_id | `ammo_338_ilm` | **`ammo_86_ilm`**，name "8.6mm ILM 精密弹" | 7.9 |
| AC1 effect_type | `stat_bonus` | **`special_effect`**（走 conditional modifier 路径） | 必修改点 #2 |
| AC8 currency | `-` | **`harambee_credit`**（FACTIONS 加 "HARAMBEE"） | 7.3 + 必修改点 #7 |
| AC8 price | 0 | **0**（保留，"不可购买"标记） | 7.3 |

### D. 系统扩展类变更（响应 systems-designer）

| 项 | v1 | v2 | 红线 |
|---|---|---|---|
| FACTIONS 常量 | `["AURA","TSAR","DENS","CHOICE"]` | **`["AURA","TSAR","DENS","CHOICE","ILM","HARAMBEE"]`**（6 阵营） | 7.4 + 7.3 + 必修改点 #7 |
| VALID_EQUIPMENT_SLOTS | 10 槽 | **14 槽**（加 ring1/ring2/necklace/implant_module/badge/trinket） | systems-designer 推荐方案 A |
| items.csv header | 12 列 | **18 列**（加 accessory_type / set_id / damage_type_resist / durability / effect_value_float / equip_slot） | 必修改点 #5 |
| 4 件现有 TSAR 防具 | set_id 无 | **回填 set_id=tsar_suppressor**（方案 1 回填） | 7.5 + 必修改点 #4 |
| compute_damage 完整管线 | 标记为 Step 6 待办 | **本提案一并实现完整管线**（方案 A，+2~3 天工作量） | 必修改点 #6 |
| 6 个新 behavior_tag 工作量 | 未明确 | **明确 4 个需 player.gd 状态机、2 个需 bullet.gd 改动，总 2~3 天** | 必修改点 #3 |

### E. 美术可行性确认（响应 art-director）

| 项 | 状态 | 备注 |
|---|---|---|
| 6 把武器精灵图规格 | ✅ 接受 | W6 绞刑者俯视角度风险点 R3 接受 art-director 提议"侧 3/4 视角伪俯视妥协 + 64×64 背包图标补完" |
| 8 件饰品子类型标识方案 | ✅ 接受 | 5 种子类型边框样式（ring/necklace/implant_module/badge/trinket） |
| 套装视觉识别方案 | ✅ 接受 | 边框色 + 角标徽 + 6 格进度指示器 |
| 设计上的美术不可实现点 | 无 | 全部美术需求可由 technical-artist 出图实现 |

### F. 修订后总扩展数量（与 v1 一致，未增减）

| 类目 | 现有 | 新增 | 总计 |
|---|---|---|---|
| 武器 | 10 | 6 | 16 |
| 防具 | 4 | 6 | 10 |
| 饰品 | 0 | 8 | 8 |
| 套装 | 0 | 2 | 2 |
| 新 behavior_tag | — | 6 | — |
| 新 fire_type | — | 1（charge_release） | — |
| 新武器类别 | — | 2（marksman_rifle / heavy_weapon） | — |
| 新货币 | — | 2（dirham / harambee_credit） | — |

---

## 一、扩展总览（更新后）

### 1.1 数量与品类

| 类目 | 现有 | 本次新增 | 新增后总计 | 备注 |
|---|---|---|---|---|
| 武器 | 10 | **6** | 16 | 含 2 个新品类（marksman_rifle / heavy_weapon） |
| 防具 | 4（全 TSAR） | **6** | 10 | 覆盖 AURA / CHOICE / ILM 非阵营 |
| 饰品 | 0 | **8** | 8 | 全新类目，5 个子类型 |
| 套装 | 0 | **2** | 2 | 1 个入门级（tsar_suppressor）+ 1 个高阶（aura_soiree） |
| 新弹药 | — | **2** | — | ammo_86_ilm（修订后公制）/ ammo_microfusion_dens |
| 新货币 | 4 | **2** | 6 | dirham + harambee_credit（FACTIONS 扩展至 6） |

### 1.2 阵营分布（更新后）

| 阵营 | 现有武器 | 新增武器 | 新增防具 | 新增饰品 | 阵营定位（依据 canon） |
|---|---|---|---|---|---|
| TSAR | 3 | 1（PM-T） | 0 | 2（套装 A 配件） | "本武器不包含电子元件"，机械可靠性 |
| CHOICE | 2 | 1（C-9） | 1（**胶原纤维束腿甲**） | 1（**胶原纤维束肌腱强化**） | 生物材料、医疗军事化 |
| AURA | 2 | 1（L-77） | 4（"宴会"全套） | 2（套装 B 配件） | 拼凑式精密、不稳定、优雅 |
| DENS | 3 | 1（K-08） | 0 | 1（算力义体） | 鲲式效率、智能武器 |
| **ILM（新）** | 0 | 1（**Mukhlis 工匠组装件**） | 1（itqan 胸甲） | 1（tasbih 念珠） | 精密制造、itqan 公差、法学辩论 |
| **卡特尔（新）** | 0 | 1（**绞刑者 哈辛托个人改装件**） | 0 | 0 | TSAR 过剩军火改装、黑市物流 |
| **哈拉姆贝（新）** | 0 | 0 | 0 | 1（kimya 护符） | 逃亡 Agent + 本地人拼凑 |

### 1.3 新机制清单

#### 1.3.1 新 behavior_tags（6 个，`|` 分隔语法沿用）

| 新标签 | 含义 | 应用武器 | 实现工作量评估 |
|---|---|---|---|
| `precision_first_shot` | 装填/静止 ≥2s 后首发：spread 强制为 0 且暴击率 +25% | W1 PM-T | **需 player.gd 静止时间跟踪**（与 ijtihad 共享字段） |
| `toxin_cloud` | 弹丸命中点生成半径 1.5m 毒雾，存在 2s，4 dmg/s（DOT，可叠加层数上限 3） | W2 C-9 | **需 bullet.gd 新增 _apply_toxin_cloud + ToxinCloud.tscn 场景** |
| `charge_shot` | 按住蓄能 0.3~1.5s，伤害随蓄能线性提升至 1.5×，满蓄穿透多目标 | W3 L-77 | **需 player.gd 蓄能状态机 + HUD 蓄能弧线** |
| `overcharge` | 持续射击累计热量至满后进入过载：伤害 **×1.3**（v1 为 ×1.5），但每秒对持有者造成 **5 点反伤**（v1 为 2/s）；**过载持续 >3s 后自伤翻倍至 10/s**；停火 1.5s 散热 | W4 K-08 | **需 player.gd 热量跟踪 + 散热计时器 + 自伤 take_damage** |
| `ijtihad_aim` | 静止/蹲伏 ≥1.5s 后触发"独立推理"：暴击 +20%、spread ×0.6，移动后失效 | W5 Mukhlis | **复用 precision_first_shot 静止时间跟踪**（共享字段） |
| `arc_projectile` | 抛物线弹道，命中点 3m 半径溅射；按住可调整射程（蓄力 0.3~1.2s 改变落点距离） | W6 绞刑者 | **需 bullet.gd 重写 _physics_process 抛物线运动方程** |

**实现工作量分类**（响应 systems-designer 必修改点 #3）：
- **需 player.gd 新增状态机/字段**（4 个）：`precision_first_shot` / `charge_shot` / `overcharge` / `ijtihad_aim`
  - 共享字段：`_still_time`（静止时间跟踪，precision_first_shot 与 ijtihad 共用）
  - 独立字段：`_charge_time`（蓄能进度）、`_weapon_heat`（热量累计）
- **需 bullet.gd 新增节点生成/运动重写**（2 个）：`toxin_cloud` / `arc_projectile`
  - toxin_cloud 新增 `ToxinCloud.tscn` Area2D 场景
  - arc_projectile 重写 `_physics_process` 为重力 + 初速度抛物线
- **可复用现有机制**：所有 6 个 tag 的 CSV 解析均复用 `FIELD_TYPE_LIST`，无需改解析器
- **总工作量**：player.gd 2~3 天 + bullet.gd 1 天 + ToxinCloud.tscn 0.2 天 ≈ **3~4 天**

#### 1.3.2 新 fire_type

`charge_release`（蓄力释放）：与 v1 一致，无修订。触发武器：W3 L-77、W6 绞刑者。需 `bullet.gd` / `weapon.gd` 增加状态机分支。

#### 1.3.3 新武器类别

| 新类别 | 定位 | 与现有类别区别 |
|---|---|---|
| `marksman_rifle` | 精确步枪/DMR | 介于 automatic_rifle 与（未来）sniper_rifle 之间；半自动/蓄能、高单发伤害、低射速、高精度、长射程；bullet_speed ≥1200 |
| `heavy_weapon` | 重武器 | 重量 ≥6.0、需 permit ≥2、高持续输出或大 AoE、移动惩罚（持有时 move_speed ×0.85）；通常伴随过热/过载机制 |

### 1.4 数值平衡锚点（修订后）

**DPS 公式统一口径**（响应 systems-designer §1.2 离群值标注）：
- 单弹丸 DPS = `damage × fire_rate / 60`
- 齐射 DPS = `damage × pellet_count × fire_rate / 60`（适用于 TSAR30 / luanhua）

**现有 10 把武器 DPS 基线（修订口径）**：

| 武器 | damage | fire_rate | 直伤 DPS | 备注 |
|---|---|---|---|---|
| AK2150 | 45 | 600 | 450 | 全自动基准上限 |
| RPK88 | 38 | 700 | 443 | 弹链压制型 |
| qbz99 | 32 | 650 | 347 | 智能步枪 |
| G31 | 28 | 450 | 210 | 生物弹 +splash |
| TSAR30 | 12 ×6 弹丸 | 45 | 54 | 霰弹（v1 标 135 系口径错误，已修订） |
| luanhua | 25 ×16 弹齐射 | 30 | 200（25×16×30/60） | 蜂群饱和 |
| L12 | 80 | 60 | 80 | 激光半自动 |
| ying | 20 | 180 | 60 | 智能手枪 |
| Milano | 10 | 200 | 33（+ AoE） | 声波锥 |
| N4 | 15 | 120 | 30（+45 DOT） | 注射器 |

**新武器直伤 DPS 锚定区间**：30~300。低于 30 必须有强力辅助机制（DOT/AoE/控制）；高于 300 必须有显著代价（蓄能/反伤/低弹匣/重载）。所有新武器均落入此区间（详见各武器表"推算"行）。

---

## 二、武器扩展（6 把，每把完整字段表 + 设计思路 + 三视角响应）

> 字段顺序与 `weapons.csv` 完全一致。`behavior_tags` 含新标签的已用 **粗体** 标注。

### W1 · PM-T 列宁格勒（TSAR · pistol）

| 字段 | 值 |
|---|---|
| weapon_id | `PMT_Leningrad` |
| name | PM-T 列宁格勒 |
| category | pistol |
| ammo_type | `ammo_762_tsar`（复用，7.62 手枪化版本） |
| manufacturer | TSAR |
| quality | 2 |
| fire_type | semi_auto |
| behavior_tags | `semi_auto\|precision_first_shot\|random_jump` |
| damage | 55 |
| fire_rate | 120 |
| magazine_size | 8 |
| reload_time | 2.0 |
| reload_type | full_mag |
| bullet_speed | 700 |
| max_spread | 5.0 |
| spread_pershot | 0.6 |
| spread_recovery | 3.5 |
| recoil_type | random_jump |
| recoil_pershot | 2 |
| recoil_recovery | 15 |
| aim_offset | 0 |
| required_permit | 0 |
| slots_available | `magazine\|optic\|muzzle` |
| slots_blocked | `stock\|grip` |
| slots_required | magazine |
| default_attachments | `{"magazine":"pmt_std_mag"}` |
| description_id | `PMT_Leningrad` |

**推算**：直伤 DPS = 55×120/60 = **110**。首发必中且 25% 暴击（×1.5 = 82.5 期望），前 3 发有效 DPS 接近 165，作为副武器合格。
**设计思路**：TSAR 现有 3 把全是大枪，缺一把副武器。PM-T 填补该缺口，落实 TSAR "7.62 甚至更大" 的反潮流口径哲学——装在 holster 里、15m 内打出步枪级伤害的重手枪。`precision_first_shot` 鼓励"掏枪—一枪—收枪"节奏。

#### W1 三视角响应

- **world-builder 响应**：✅ 无红线触发，无需调整
- **systems-designer 响应**：✅ DPS 110 落入区间，`precision_first_shot` 需 player.gd 新增 `_still_time` 字段（与 W5 ijtihad 共享），已计入工作量
- **art-director 响应**：✅ 接受 32×20 精灵图 7 帧规格，TSAR 暗红 `#7A2A2A` + 深绿 `#2F4F2F` 主色，制造商 logo "本武器不包含电子元件" 4×2px 灰色小字

### W2 · C-9 哀悼者（CHOICE · smg）

| 字段 | 值 |
|---|---|
| weapon_id | `C9_Keenedge` |
| name | C-9 哀悼者 |
| category | smg |
| ammo_type | `ammo_bio_round`（复用） |
| manufacturer | CHOICE |
| quality | 3 |
| fire_type | full_auto |
| behavior_tags | `full_auto\|toxin_cloud\|bio` |
| damage | 12 |
| fire_rate | 800 |
| magazine_size | 35 |
| reload_time | 2.8 |
| reload_type | full_mag |
| bullet_speed | 600 |
| max_spread | 12.0 |
| spread_pershot | 0.4 |
| spread_recovery | 4.0 |
| recoil_type | none |
| recoil_pershot | 0 |
| recoil_recovery | 0 |
| aim_offset | 0 |
| required_permit | 1 |
| slots_available | `stock\|magazine\|muzzle\|optic\|grip` |
| slots_blocked | - |
| slots_required | magazine |
| default_attachments | `{"magazine":"c9_std_mag"}` |
| description_id | `C9_Keenedge` |

**推算**：直伤 DPS = 12×800/60 = **160**；加上每发命中点 2s 毒雾 8 dmg（4×2），按 50% 命中率折算追加约 67 DPS，总有效 ≈ **227**。与 Milano（33+AoE）同区间，但 Milano 是声波锥（贴近群伤），C-9 是中程点射 + 区域封锁，定位错开。
**设计思路**：CHOICE 现有 G31（步枪 splash）+ N4（注射 DOT），缺一把 SMG 填近中距压制位。`toxin_cloud` 落实 CHOICE "医疗项目军事化" lore（与 C-7 武器同源研发线：原本是创伤喷雾装置，反过来变成污染弹）。

#### W2 三视角响应

- **world-builder 响应**：✅ 无红线触发
- **systems-designer 响应**：✅ DPS 227 在区间内；`toxin_cloud` 需 bullet.gd 新增 `_apply_toxin_cloud` + ToxinCloud.tscn Area2D 节点，已计入工作量
- **art-director 响应**：✅ 接受 40×24 精灵图 9 帧规格 + 配套毒雾粒子 VFX（32×32 alpha 渐变 4 帧，独立任务交 technical-artist）

### W3 · L-77 长昼（AURA · marksman_rifle）

| 字段 | 值 |
|---|---|
| weapon_id | `L77_Longday` |
| name | L-77 长昼 |
| category | marksman_rifle |
| ammo_type | `energy_battery_aura`（复用） |
| manufacturer | AURA |
| quality | 4 |
| fire_type | charge_release |
| behavior_tags | `charge_shot\|laser_beam\|precision_lens\|energy\|pierce` |
| damage | 120 |
| fire_rate | 40 |
| magazine_size | 5 |
| reload_time | 3.0 |
| reload_type | full_mag |
| bullet_speed | 2000 |
| max_spread | 0.5 |
| spread_pershot | 0.0 |
| spread_recovery | 1.0 |
| recoil_type | none |
| recoil_pershot | 0 |
| recoil_recovery | 0 |
| aim_offset | 2 |
| required_permit | 2 |
| slots_available | `stock\|magazine\|optic` |
| slots_blocked | `muzzle\|grip` |
| slots_required | magazine |
| default_attachments | `{"magazine":"l77_std_battery"}` |
| description_id | `L77_Longday` |

**推算**：未蓄能 120 dmg / 1.5s 循环 ≈ **80 DPS**；满蓄 180 dmg / 1.5s ≈ **120 DPS** + 穿透多目标。单发伤害对标 L12（80）+50%，靠 charge 代价换精度与穿透。DPS 不超 150，符合"高单发低射速"区间。
**设计思路**：AURA 现有 L12（半自动激光）+ Milano（声波 smg），缺一把精确步枪补远程位。`charge_shot` 与 AURA "拼凑式技术体系，理想条件下碾压，极端条件下失灵" lore 完美契合——满蓄 = 理想条件，蓄能中被打断 = 失灵。

#### W3 三视角响应

- **world-builder 响应**：✅ 无红线触发
- **systems-designer 响应**：✅ DPS 120 在区间内；`charge_release` 是新 fire_type，需 player.gd `_update_shoot_input` 新增 match 分支 + `_charge_time` 字段 + HUD 蓄能弧线（复用 reload bar）
- **art-director 响应**：✅ 接受 56×24 精灵图 10 帧（idle 1 + charge 3 + release 2 + reload 4）；与 W5 Mukhlis 同为 marksman_rifle 同尺寸，已确认 art-director 风险点 R1 强差异化方案：AURA 走"激光谐振腔外露 + 金白主色 `#D4AF37`"

### W4 · K-08 沉默钟摆（DENS · heavy_weapon） — **数值修订**

| 字段 | v1 值 | **v2 值** |
|---|---|---|
| weapon_id | `K08_Pendulum` | `K08_Pendulum` |
| name | K-08 沉默钟摆 | K-08 沉默钟摆 |
| category | heavy_weapon | heavy_weapon |
| ammo_type | `ammo_microfusion_dens`（新弹药） | `ammo_microfusion_dens`（新弹药） |
| manufacturer | DENS | DENS |
| quality | 3 | 3 |
| fire_type | full_auto | full_auto |
| behavior_tags | `full_auto\|overcharge\|sustained_fire\|microwave\|energy` | `full_auto\|overcharge\|sustained_fire\|microwave\|energy`（**overcharge 含义修订见下**） |
| damage | 18 | 18 |
| fire_rate | 1000 | 1000 |
| magazine_size | 80 | 80 |
| reload_time | 5.5 | 5.5 |
| reload_type | full_mag | full_mag |
| bullet_speed | 1500 | 1500 |
| max_spread | 4.0 | 4.0 |
| spread_pershot | 0.2 | 0.2 |
| spread_recovery | 6.0 | 6.0 |
| recoil_type | none | none |
| recoil_pershot | 0 | 0 |
| recoil_recovery | 0 | 0 |
| aim_offset | 0 | 0 |
| required_permit | 2 | 2 |
| slots_available | `stock\|magazine\|optic` | `stock\|magazine\|optic` |
| slots_blocked | `muzzle\|grip` | `muzzle\|grip` |
| slots_required | magazine | magazine |
| default_attachments | `{"magazine":"k08_std_cell"}` | `{"magazine":"k08_std_cell"}` |
| description_id | `K08_Pendulum` | `K08_Pendulum` |

**`overcharge` tag 含义修订（v1 → v2）**：
- v1：伤害 ×1.5 + 反伤 2/s
- **v2**：伤害 **×1.3** + 反伤 **5/s**；**过载持续 >3s 后自伤翻倍至 10/s**；停火 1.5s 散热

**推算（v2 修订后）**：
- 直伤 DPS = 18×1000/60 = **300**（与 v1 一致，仍在 300 锚定上限）
- 过载后 DPS = 300 × 1.3 = **390**（v1 为 450，**修订后低于 RPK88 的 443**）
- 反伤代价：80 弹匣持续 4.8s，反伤 5/s × 4.8s = **24 HP**（v1 为 9.6 HP，**修订后代价显著**）
- 若过载持续 >3s，剩余 1.8s 反伤翻倍：5×3 + 10×1.8 = **33 HP**（占 base_hp 100 的 33%）
- 重量 8.0，持有时 move_speed ×0.85，重载 5.5s

**对标分析**：
- RPK88（443 DPS，无自伤，重量适中）：DPS 上限高于 K-08 但无代价机制
- K-08 v2（390 DPS 过载，反伤 5~10/s，重量 8.0）：DPS 略低但持续输出更稳定（80 弹匣 vs RPK88 100 弹匣），代价机制显著

**设计思路（v2 强化）**：DENS 现有 luanhua/qbz99/ying 三把都偏"智能精准"，缺一把重型压制武器落实"用机器打仗"哲学。`overcharge` 是鲲式效率崇拜的具象：把性能推到模型允许的边缘，代价转嫁给碳基决策节点（持有者本人）。v2 修订后反伤从"生理限制的副产品"升级为"鲲冷默接受的代价优化解"——24 HP 代价换来 90 DPS 增益（390 vs 300），鲲的会计语言里这是"成本可控的效能提升"。微波（microwave）塔形武器，对应"鲲会让你确认它已经完成了战争"。

#### W4 三视角响应

- **world-builder 响应**：✅ 红线 7.7 通过——overcharge 自伤机制是 canon 忠实的（`dens.md` §2.3"人类是高成本但不可完全替代的决策节点"）。v2 反伤提升至 5~10/s 进一步强化"鲲设计最大效率，人类碳基载体散热极限是瓶颈"的叙事，鲲不解释（`dens.md` §六腔调"只陈述"）完全对齐
- **systems-designer 响应**：✅ **必修改点 #1 已采纳方案 (B) + (A) 组合**——倍率 ×1.5 → ×1.3（方案 A）+ 反伤 2/s → 5/s（方案 B）+ 过载 >3s 翻倍机制。修订后过载 DPS 390 落入"接近 RPK88 但不超过"区间，反伤 24~33 HP 是 base_hp 100 的 24~33%，代价显著。**已通过 DPS 对比**
- **art-director 响应**：✅ 接受 64×32 精灵图 13 帧（idle 1 + shoot 4 + overcharge 2 + reload 6），overcharge 满热后"过载态"色相切换（同一 sprite modulate 红/橙 0.5s 过渡）。风险点 R2 已确认：HUD 武器栏 180×80 槽位统一缩放到 32×32 显示，背包/详情用原尺寸 64×32

### W5 · Mukhlis 忠信（ILM · marksman_rifle） — **叙事重写**

| 字段 | 值 |
|---|---|
| weapon_id | `Mukhlis` |
| name | Mukhlis 忠信 |
| category | marksman_rifle |
| ammo_type | **`ammo_86_ilm`**（v1 为 `ammo_338_ilm`，按红线 7.9 改公制 mm） |
| manufacturer | ILM |
| quality | 4 |
| fire_type | semi_auto |
| behavior_tags | `semi_auto\|ijtihad_aim\|precision_lens\|caseless` |
| damage | 95 |
| fire_rate | 50 |
| magazine_size | 7 |
| reload_time | 3.2 |
| reload_type | full_mag |
| bullet_speed | 1400 |
| max_spread | 1.5 |
| spread_pershot | 0.4 |
| spread_recovery | 2.5 |
| recoil_type | random_jump |
| recoil_pershot | 4 |
| recoil_recovery | 18 |
| aim_offset | 1 |
| required_permit | 2 |
| slots_available | `stock\|magazine\|muzzle\|optic` |
| slots_blocked | `grip` |
| slots_required | magazine |
| default_attachments | `{"magazine":"mukhlis_std_mag"}` |
| description_id | `Mukhlis` |

**推算**：直伤 DPS = 95×50/60 ≈ **79**；`ijtihad_aim` 触发后暴击 +20% + spread ×0.6，等效 ≈ **100 DPS** 且精度极高。对标 L12（80）+25%，但 L12 是激光瞬时、Mukhlis 是实弹抛壳，靠精度而非弹速。
**设计思路（v2 修订）**：按 world-builder 红线 7.1 修正建议，**Mukhlis 不是 ILM 作为组织售出的成品枪**，而是 **ILM 工匠用 ILM 自产的精密部件（枪管、扳机组、光学镀膜）+ TSAR 提供的机床/动作件**（援引 `ilm.md` §与四大公司"伊尔姆提供精密部件，TSAR 提供原材料和能源"的技术贸易 canon），由委员会认证的个别工匠私下组装的极少数定制件。Mukhlis（阿拉伯语"忠信/精纯"）年产量 47 支是工匠个人产能 + 委员会对工匠个人授权，非 ILM 量产线。`ijtihad_aim` 直接把"独立法学推理"翻译成机制。

**委员会叙事重构（v2，响应红线 7.10）**：按 world-builder 修正建议，委员会 5:4 投票**不是"是否批准 Mukhlis 这把武器"**（若定性为武器则按 canon 必须全票），而是 **"Mukhlis 是否构成教法意义上的'武器'（silah），还是'精密测量器械'（āla diqqa）"**。此分类议题属于"非军事用途的伦理辩论"，按 canon `ilm.md` §委员会的议事程序 **过半即可**。5:4 多数裁定"Mukhlis 是精密器械而非武器"（因无 AI、无自动目标识别），从而规避武器级全票门槛。学派投票逻辑：萨拉菲派（赞成"非武器"：工具合法性只涉使用者意图）、苏菲派（反对：一把如此精确的器械，本身就在引诱使用者犯错）、现代主义派多数（赞成"非武器"：ijtihad 在新时代条件下重新理解"武器"定义）。

#### W5 三视角响应

- **world-builder 响应**：
  - **7.1 ILM 武器化**：✅ **采纳修正建议**——重写描述为"ILM 工匠用 ILM 部件 + TSAR 动作件手工组装件"，非 ILM 组织成品枪。manufacturer 字段保留 `ILM`（标识技术血缘），产量叙事"47 支/年"保留但主体改为"工匠个人产能 + 委员会对工匠个人授权"。数值不变，仅重写描述文本
  - **7.9 .338 口径命名**：✅ **采纳修正建议**——`ammo_338_ilm` → `ammo_86_ilm`，name "8.6mm ILM 精密弹"，W5 ammo_type 同步修订。保留 .338 风味在 description 文本中作 lore 注释
  - **7.10 委员会审批**：✅ **采纳修正建议**——5:4 重构为"分类投票"（武器 vs 精密器械），过半即可。深化 `ilm.md` §三学派之争"临时结盟、议题洗牌"叙事。仅重写描述，不改数值/manufacturer/机制
- **systems-designer 响应**：✅ DPS 79~100 在区间内；`ijtihad_aim` 复用 W1 `precision_first_shot` 的 `_still_time` 字段，无额外状态机负担。`ammo_86_ilm` 字段类型与现有 ammo 一致，无 schema 改动
- **art-director 响应**：✅ 接受 56×24 精灵图 8 帧（idle 1 + shoot 2 + reload 5）。风险点 R1 已确认：与 W3 L-77 同尺寸强差异化——ILM 走"深蓝枪管 `#1E3A8A` + 金色 `#D4AF37` 阿拉伯文铭文蚀刻"。制造商 logo "إِنَّ اللَّهَ مَعَ الصَّابِرِينَ"枪管顶部金色蚀刻

### W6 · El Ahorcado 绞刑者（卡特尔 · special）

| 字段 | 值 |
|---|---|
| weapon_id | `El_Ahorcado` |
| name | El Ahorcado 绞刑者 |
| category | special |
| ammo_type | `ammo_762_tsar`（复用，TSAR 步枪弹改作轨道弹丸） |
| manufacturer | CARTEL |
| quality | 2 |
| fire_type | charge_release |
| behavior_tags | `arc_projectile\|splash\|scrap_railgun\|charge_release` |
| damage | 90 |
| fire_rate | 25 |
| magazine_size | 5 |
| reload_time | 3.5 |
| reload_type | per_round |
| bullet_speed | 600 |
| max_spread | 8.0 |
| spread_pershot | 0.0 |
| spread_recovery | 5.0 |
| recoil_type | random_jump |
| recoil_pershot | 6 |
| recoil_recovery | 20 |
| aim_offset | 0 |
| required_permit | 1 |
| slots_available | `muzzle\|optic` |
| slots_blocked | `stock\|magazine\|grip` |
| slots_required | - |
| default_attachments | `{}` |
| description_id | `El_Ahorcado` |

**推算**：直伤 DPS = 90×25/60 ≈ **37.5** + 3m AoE 溅射（按 2 目标折算 ≈ 75 DPS）。DPS 37.5 落在 30~300 区间下限，靠 AoE + 战术价值（隔墙抛射）补偿。
**设计思路**：卡特尔是首个非公司武装阵营。`cartel.md` 明确"卡特尔的军火库以 TSAR 的 AK-2150 继承者和 TSAR-30 霰弹枪为主"，绞刑者复用 `ammo_762_tsar`——卡特尔中层退役雇佣兵哈辛托用废弃电磁元件拼出来的轨道加速器。**v2 追加澄清**（响应红线 7.2）：此为哈辛托个人作品，非卡特尔量产列装。`arc_projectile` 是首个抛物线弹道武器，落实"绝望世界"的不对称：玩家用废品打正规军，靠隔墙抛射而非正面对枪。

#### W6 三视角响应

- **world-builder 响应**：✅ **红线 7.2 通过**——canon"以 TSAR 为主"留有非主流装备空间，W6 复用 TSAR 弹药 + 单件改装件 + 哈辛托个人作品非量产列装，符合框架。已按 world-builder 建议在描述中追加"哈辛托个人作品，非卡特尔量产列装"澄清
- **systems-designer 响应**：✅ DPS 37.5 + AoE 在区间下限，靠战术价值补偿合理；`arc_projectile` 需 bullet.gd 重写 `_physics_process` 为重力 + 初速度抛物线，已计入工作量
- **art-director 响应**：✅ 接受 48×32 精灵图 10 帧（idle 1 + charge 2 + release 2 + reload 5 逐发装填）。**风险点 R3 已采纳 art-director 提议**：俯视角度下"轨道加速器导轨从上看是两条平行线"难以表达，采用"侧 3/4 视角伪俯视妥协" + 额外出 1 张 64×64 背包图标补完识别度。卡特尔暗橙 `#B45309` + 黑 `#1A1A1A` 主色，枪托刻痕计数（每任主人 1 道横线，已刻 23 道）

> **类别覆盖说明**：6 把武器覆盖 pistol / smg / marksman_rifle(×2 新) / heavy_weapon(新) / special 共 5 类。shotgun 与 automatic_rifle 故意不扩充——现有 10 把中 automatic_rifle 已 5 把（占 50%，过度饱和），shotgun 仅 TSAR30 一把但定位清晰（贴脸爆发），优先级低于补足 pistol/smg/special 空缺 + 引入 2 个新品类。

---

## 三、防具扩展（6 件）

> 字段顺序与 `items.csv` 现有 schema 完全一致 + 新增字段（见 §五）。`category=armor`，`effect_type=armor`，`effect_value` 为护甲值。
> **本节所有防具均带 `set_id` 字段**（v2 已确认 4 件现有 TSAR 防具回填 set_id=tsar_suppressor，详见 §五）。

### A1 · aura_helm_soiree（AURA · helm · rare · 套装 B 成员）

| 字段 | 值 |
|---|---|
| item_id | `aura_helm_soiree` |
| name | 宴会轻纱盔 |
| category | armor |
| accessory_type | ""（防具非饰品） |
| rarity | rare |
| stackable | false |
| max_stack | 1 |
| weight | 1.2 |
| price | 600 |
| currency | aura_credit |
| effect_type | armor |
| effect_value | 14 |
| damage_type_resist | `energy\|kinetic`（AURA 偏振纤维对能量/动能均有部分抗性） |
| durability | 200（受击次数，AURA 设备易损） |
| effect_value_float | 0.0 |
| equip_slot | helm |
| set_id | `aura_soiree` |
| description_id | `aura_helm_soiree` |

**设计思路**：现有头盔位只有 steel(common, 10护甲)。aura_helm 填补 rare 头盔缺口，护甲 14 略高于 steel 但重量仅 1.2（AURA 轻量化精密路线）。轻纱盔是 AURA 主动迷彩系统的头盔载体，本身不提供隐身（隐身在 4 件套触发），仅做护甲基础值。

### A2 · aura_chest_soiree（AURA · chest · epic · 套装 B 成员）

| 字段 | 值 |
|---|---|
| item_id | `aura_chest_soiree` |
| name | 宴会位移胸甲 |
| category | armor |
| accessory_type | "" |
| rarity | epic |
| stackable | false |
| max_stack | 1 |
| weight | 3.5 |
| price | 1500 |
| currency | aura_credit |
| effect_type | armor |
| effect_value | 35 |
| damage_type_resist | `energy`（电磁偏转场对能量抗性高） |
| durability | 180 |
| effect_value_float | 0.0 |
| equip_slot | chest |
| set_id | `aura_soiree` |
| description_id | `aura_chest_soiree` |

**设计思路**：现有胸甲 kevlar(uncommon, 25)。aura_chest epic 级护甲 35 为全游戏最高胸甲。位移胸甲内置 AURA 拼凑式偏移场发生器（从 DENS 窃取的电磁偏转专利），单件不触发位移（4 件套才触发），但高护甲+轻量（3.5 vs kevlar 4.5）体现 AURA 高端定位。

### A3 · aura_legs_soiree（AURA · legs · epic · 套装 B 成员）

| 字段 | 值 |
|---|---|
| item_id | `aura_legs_soiree` |
| name | 宴会丝绒腿甲 |
| category | armor |
| accessory_type | "" |
| rarity | epic |
| stackable | false |
| max_stack | 1 |
| weight | 2.0 |
| price | 1100 |
| currency | aura_credit |
| effect_type | armor |
| effect_value | 20 |
| damage_type_resist | `kinetic`（丝绒纤维抗撕裂） |
| durability | 200 |
| effect_value_float | 0.0 |
| equip_slot | legs |
| set_id | `aura_soiree` |
| description_id | `aura_legs_soiree` |

**设计思路**：现有腿甲 tactical(common, 12)。aura_legs epic 级护甲 20，重量 2.0（vs tactical 2.5）。腿甲是套装 B 的"机动件"——丝绒腿甲在 4 件套触发时提供 sprint_multiplier 加成。

### A4 · aura_boots_soiree（AURA · boots · rare · 套装 B 成员）

| 字段 | 值 |
|---|---|
| item_id | `aura_boots_soiree` |
| name | 宴会静默靴 |
| category | armor |
| accessory_type | "" |
| rarity | rare |
| stackable | false |
| max_stack | 1 |
| weight | 1.0 |
| price | 700 |
| currency | aura_credit |
| effect_type | armor |
| effect_value | 12 |
| damage_type_resist | `kinetic` |
| durability | 220 |
| effect_value_float | 0.0 |
| equip_slot | boots |
| set_id | `aura_soiree` |
| description_id | `aura_boots_soiree` |

**设计思路**：现有战靴 reinforced(common, 8)。aura_boots rare 级护甲 12，重量 1.0（全游戏最轻靴）。静默靴落实 AURA "特种作战" 战术哲学——脚步声降低（4 件套隐身触发时配合）。

### A5 · choice_legs_collagen（CHOICE · legs · uncommon · 非套装） — **重命名**

| 字段 | v1 值 | **v2 值** |
|---|---|---|
| item_id | `choice_legs_bio` | **`choice_legs_collagen`** |
| name | C-7 肌腱腿甲 | **胶原纤维束腿甲** |
| category | armor | armor |
| accessory_type | "" | "" |
| rarity | uncommon | uncommon |
| stackable | false | false |
| max_stack | 1 | 1 |
| weight | 3.0 | 3.0 |
| price | 380 | 380 |
| currency | choice_ticket | choice_ticket |
| effect_type | armor | armor |
| effect_value | 15 | 15 |
| damage_type_resist | `bio`（生物材料抗生化） | `bio` |
| durability | 100（CHOICE 生物材料易衰减） | 100 |
| effect_value_float | 0.0 | 0.0 |
| equip_slot | legs | legs |
| set_id | - | - |
| description_id | `choice_legs_bio` | **`choice_legs_collagen`** |

**设计思路（v2 修订）**：CHOICE 首件防具。按 world-builder 红线 7.6 修正建议，**剥离"C-7 肌腱"品牌**，改为"采用与 C-7 武器同源钙化胶原纤维束材料的轻型护甲"——引用的是**材料**（钙化胶原纤维束）的同源，而非"C-7 项目的分支"。canon 兼容性论证：`choice.md` §4.4 仅说"钙化胶原纤维束硬化后变成鞭状刃"，**未说该纤维束只能用作武器**。同源材料的民用医疗植入/护甲是合理推断，不构成 canon 扩展。护甲 15（高于 tactical 12）但重量 3.0 略重，非套装件作为 CHOICE 玩家的过渡防具。

#### A5 三视角响应

- **world-builder 响应**：✅ **红线 7.6 采纳修正建议**——item_id 从 `choice_legs_bio` 改为 `choice_legs_collagen`，name 从"C-7 肌腱腿甲"改为"胶原纤维束腿甲"，描述从"C-7 项目第三条产品线"改为"采用与 C-7 武器同源钙化胶原纤维束材料的民用护甲"。数值不变（护甲 15 / 重量 3.0）
- **systems-designer 响应**：✅ 数值合理（护甲/重量比 5.0，略高于 tactical 4.8 但重量更重，uncommon 合理）；damage_type_resist=bio 落实 CHOICE 生物材料差异化
- **art-director 响应**：✅ 接受 64×64 背包图标 + 56×56 槽位图标，钙化胶原纤维束外露 + CHOICE 紫色 `#7C3AED` 生物荧光 + 植入接口

### A6 · ilm_chest_itqan（ILM · chest · epic · 非套装）

| 字段 | 值 |
|---|---|
| item_id | `ilm_chest_itqan` |
| name | itqan 精工胸甲 |
| category | armor |
| accessory_type | "" |
| rarity | epic |
| stackable | false |
| max_stack | 1 |
| weight | 4.0 |
| price | 1400 |
| currency | **dirham**（v2 已确认 FACTIONS 加 ILM，dirham 接入第 5 钱包） |
| effect_type | armor |
| effect_value | 32 |
| damage_type_resist | `kinetic\|energy`（itqan 精工公差对动能/能量均优） |
| durability | -1（ILM 精工不衰减，与 AURA 易损形成对照） |
| effect_value_float | 0.0 |
| equip_slot | chest |
| set_id | - |
| description_id | `ilm_chest_itqan` |

**设计思路**：ILM 首件防具，也是首件使用迪拉姆货币的装备。护甲 32 介于 kevlar(25) 与 aura_chest(35) 之间，重量 4.0 居中。itqan（阿拉伯语"精工/完美"）胸甲是 ILM 工匠把枪管级公差控制用到护甲拼接上的产物——每片护甲板的接缝公差 < 0.01mm，所以同等重量下护甲值更高。非套装件，但与 Mukhlis 步枪配套使用时建议触发隐藏加成（未来扩展：faction_synergy 字段）。

#### A6 三视角响应

- **world-builder 响应**：✅ **红线 7.4 采纳修正建议**——FACTIONS 加 "ILM"，dirham 作为第 5 货币接入。canon"通过技术贸易获得"对应玩家可向 ILM 出售 DENS 代码/AURA 镀膜工艺换迪拉姆
- **systems-designer 响应**：✅ 护甲 32 / 重量 4.0 / 比率 8.0 合理；dirham 通过 FACTIONS 扩展自动接入 faction_wallet，存档兼容（旧存档 ILM=0）
- **art-director 响应**：✅ 接受 64×64 背包图标 + 56×56 槽位图标，itqan 精工接缝 + ILM 深蓝 `#1E3A8A` + 金色 `#D4AF37` 阿拉伯文工匠签名蚀刻

### 槽位/品级覆盖核对

| 槽位 | 现有 | 新增 | 新增件品级 |
|---|---|---|---|
| helm | steel(common) | aura_helm_soiree | rare |
| chest | kevlar(uncommon) | aura_chest_soiree, ilm_chest_itqan | epic, epic |
| legs | tactical(common) | aura_legs_soiree, **choice_legs_collagen** | epic, uncommon |
| boots | reinforced(common) | aura_boots_soiree | rare |

4 个槽位均有新增，覆盖 uncommon / rare / epic 三个新品级。

### 套装 B 4 件护甲总和确认（响应 systems-designer §1.4 离群值标注）

- A1+A2+A3+A4 护甲总和 = 14+35+20+12 = **81**（比现有 TSAR 4 件 55 高 47%）
- 4 件套 damage_reduction ×0.85（等效护甲再 +15%），实际有效护甲 ≈ 93
- **game-designer 确认**：高阶套装允许护甲上限突破现有 common 基线 1.7 倍，理由：
  1. 全 epic/rare 级别，价格高昂（1500+1100+700+600=3900 aura_credit）
  2. 建议配合 durability 易损机制（受击 180~220 次后护甲衰减），强化"你保不住好东西"的绝望感
  3. 玩家死亡时 `clear_raid_inventory()` 清空背包+装备（`game_manager.gd:669-685`），套装会随死亡遗失，集齐喜悦与"下一局就没了"恐惧并存

---

## 四、饰品新建（8 件，全新类目）

> 饰品为全新 category=`accessory`。子类型字段 `accessory_type`。v2 确认 **VALID_EQUIPMENT_SLOTS 从 10 扩至 14**（systems-designer 方案 A），新增 ring1/ring2/necklace/implant_module/badge/trinket 6 槽。

### AC1 · tsar_badge_suppressor（TSAR · badge · uncommon · 套装 A 成员） — **effect_type 修订**

| 字段 | v1 值 | **v2 值** |
|---|---|---|
| item_id | `tsar_badge_suppressor` | `tsar_badge_suppressor` |
| name | 镇暴者徽章 | 镇暴者徽章 |
| category | accessory | accessory |
| accessory_type | badge | badge |
| rarity | uncommon | uncommon |
| stackable | false | false |
| max_stack | 1 | 1 |
| weight | 0.1 | 0.1 |
| price | 250 | 250 |
| currency | tsar_point | tsar_point |
| effect_type | **stat_bonus** | **special_effect**（v2 修订，见下） |
| effect_value | 5 | 5（保留为百分比含义） |
| damage_type_resist | - | - |
| durability | -1 | -1 |
| effect_value_float | 0.0 | 0.05（乘性 5%，可选） |
| equip_slot | badge | badge |
| set_id | `tsar_suppressor` | `tsar_suppressor` |
| description_id | `tsar_badge_suppressor` | `tsar_badge_suppressor` |

**effect_type 修订说明（v2，响应 systems-designer 必修改点 #2）**：
- v1 effect_type=stat_bonus 与实际语义冲突——`stat_bonus` 在 IMPLANT_DB 范式里是无条件 stat_modifier，会在 `recompute_player_stats()` 中无条件注入，但 AC1 实际效果是"装备 TSAR 制造商武器时 +5% 伤害"，是 **conditional effect**
- **v2 改为 effect_type=special_effect**，由 EffectManager 按 item_id 路由到 `_conditional_modifiers`（与套装 6 件套 conditional 同路径，复用机制）。effect_value=5 仍保留为百分比含义，condition="weapon_manufacturer=TSAR"

**设计思路**：TSAR "军事服务荣誉公民"徽章（`tsar.md` §2.1）。special_effect 由 EffectManager 按 item_id 解析为 conditional damage_multiplier 1.05，仅对 manufacturer=TSAR 武器生效。

#### AC1 三视角响应

- **world-builder 响应**：✅ 无红线触发
- **systems-designer 响应**：✅ **必修改点 #2 已采纳**——effect_type 从 stat_bonus 改为 special_effect，走 conditional modifier 路径，与套装 6 件套 conditional 同机制复用
- **art-director 响应**：✅ 接受 64×64 背包图标 + 56×56 槽位图标，圆形徽章 + 暗红合金 `#7A2A2A` + 钢印编号，子类型边框样式：badge 圆形 + 政徽式齿轮纹外圈

### AC2 · tsar_ring_steelhand（TSAR · ring · uncommon · 套装 A 成员）

| 字段 | 值 |
|---|---|
| item_id | `tsar_ring_steelhand` |
| name | 钢手戒指 |
| category | accessory |
| accessory_type | ring |
| rarity | uncommon |
| stackable | false |
| max_stack | 1 |
| weight | 0.05 |
| price | 200 |
| currency | tsar_point |
| effect_type | stat_bonus |
| effect_value | 15 |
| damage_type_resist | - |
| durability | -1 |
| effect_value_float | 0.0 |
| equip_slot | ring1（可装 ring1 或 ring2） |
| set_id | `tsar_suppressor` |
| description_id | `tsar_ring_steelhand` |

**设计思路**：effect_value=15 → max_hp +15。TSAR 第 47 兵工厂出厂检验单的金属边角料熔铸的小戒指，工人把废金属打成戒指戴在手上作为护身符。套装 A 成员。低权重(0.05)低价格，入门级饰品。

### AC3 · aura_ring_soiree（AURA · ring · epic · 套装 B 成员）

| 字段 | 值 |
|---|---|
| item_id | `aura_ring_soiree` |
| name | 宴会指环 |
| category | accessory |
| accessory_type | ring |
| rarity | epic |
| stackable | false |
| max_stack | 1 |
| weight | 0.05 |
| price | 1200 |
| currency | aura_credit |
| effect_type | stat_bonus |
| effect_value | 8 |
| damage_type_resist | - |
| durability | -1 |
| effect_value_float | 0.0 |
| equip_slot | ring1（可装 ring1 或 ring2） |
| set_id | `aura_soiree` |
| description_id | `aura_ring_soiree` |

**设计思路**：effect_value=8 → crit_chance +8%（绝对值加成）。AURA 旧贵族的家族徽章戒指。crit_chance 是 EffectManager `_stat_registry` 已注册属性，可无缝接入。

### AC4 · aura_necklace_soiree（AURA · necklace · epic · 套装 B 成员）

| 字段 | 值 |
|---|---|
| item_id | `aura_necklace_soiree` |
| name | 宴会项链 |
| category | accessory |
| accessory_type | necklace |
| rarity | epic |
| stackable | false |
| max_stack | 1 |
| weight | 0.2 |
| price | 1500 |
| currency | aura_credit |
| effect_type | special_effect |
| effect_value | 20 |
| damage_type_resist | - |
| durability | 180（AURA 谐振器易失效） |
| effect_value_float | 0.20 |
| equip_slot | necklace |
| set_id | `aura_soiree` |
| description_id | `aura_necklace_soiree` |

**设计思路**：effect_type=special_effect，effect_value=20 → 主动迷彩冷却时间 -20%（乘性缩减）。AURA 主动迷彩系统的颈部控制模组。套装 B 成员，6 件套触发后主动迷彩效果增强。

### AC5 · choice_implant_collagen（CHOICE · implant_module · epic · 非套装） — **重命名**

| 字段 | v1 值 | **v2 值** |
|---|---|---|
| item_id | `choice_implant_tendon` | **`choice_implant_collagen`** |
| name | C-7 肌腱义体 | **胶原纤维束肌腱强化** |
| category | accessory | accessory |
| accessory_type | implant_module | implant_module |
| rarity | epic | epic |
| stackable | false | false |
| max_stack | 1 | 1 |
| weight | 0.3 | 0.3 |
| price | 1800 | 1800 |
| currency | choice_ticket | choice_ticket |
| effect_type | stat_bonus | stat_bonus |
| effect_value | 30 | 30 |
| damage_type_resist | - | - |
| durability | -1（植入后无法移除，与 lore 一致） | -1 |
| effect_value_float | 0.0 | 0.0 |
| equip_slot | implant_module | implant_module |
| set_id | - | - |
| description_id | `choice_implant_tendon` | **`choice_implant_collagen`** |

**设计思路（v2 修订）**：effect_value=30 → max_hp +30。按 world-builder 红线 7.6 修正建议，**剥离"C-7 肌腱"品牌**，改为"采用与 C-7 武器同源钙化胶原纤维束材料植入人体肌腱位置的民用增强件"——引用的是**材料**同源，而非"C-7 项目的分支"。CHOICE "医疗项目军事化" 反向路径：军事技术回销民用。与 IMPLANT_DB 中 `heart_combat`(+20 hp) 同类但更强，因占饰品槽而非义体槽，平衡。

#### AC5 三视角响应

- **world-builder 响应**：✅ **红线 7.6 采纳修正建议**——item_id 从 `choice_implant_tendon` 改为 `choice_implant_collagen`，name 从"C-7 肌腱义体"改为"胶原纤维束肌腱强化"，描述从"C-7 项目的民用化分支"改为"采用与 C-7 武器同源钙化胶原纤维束材料的民用增强件"。数值不变
- **systems-designer 响应**：✅ effect_value=30 比 heart_combat(+20) 高 50%，但占饰品槽不占义体槽，平衡可接受
- **art-director 响应**：✅ 接受 64×64 背包图标 + 56×56 槽位图标，钙化胶原纤维束 + CHOICE 紫色 `#7C3AED` 生物荧光 + 植入接口，子类型边框：implant_module 方形 + 电路纹边角 + 4 角缺口

### AC6 · dens_implant_overclock（DENS · implant_module · rare · 非套装）

| 字段 | 值 |
|---|---|
| item_id | `dens_implant_overclock` |
| name | 算力超频模块 |
| category | accessory |
| accessory_type | implant_module |
| rarity | rare |
| stackable | false |
| max_stack | 1 |
| weight | 0.2 |
| price | 900 |
| currency | dens_voucher |
| effect_type | skill_bonus |
| effect_value | 2 |
| damage_type_resist | - |
| durability | -1 |
| effect_value_float | 0.0 |
| equip_slot | implant_module |
| set_id | - |
| description_id | `dens_implant_overclock` |

**设计思路**：effect_value=2 → compute_regen +2/s。DENS 鲲式算力配额的"侧信道加速器"——把邓氏公民的闲置算力配额旁路到战斗义体。与 IMPLANT_DB `brain_nc02`（+2 compute_regen）同数值，但占饰品槽。

### AC7 · ilm_trinket_tasbih（ILM · trinket · epic · 非套装）

| 字段 | 值 |
|---|---|
| item_id | `ilm_trinket_tasbih` |
| name | tasbih 念珠 |
| category | accessory |
| accessory_type | trinket |
| rarity | epic |
| stackable | false |
| max_stack | 1 |
| weight | 0.1 |
| price | 1000 |
| currency | **dirham** |
| effect_type | special_effect |
| effect_value | 10 |
| damage_type_resist | - |
| durability | -1 |
| effect_value_float | 0.90（spread_multiplier ×0.90） |
| equip_slot | trinket |
| set_id | - |
| description_id | `ilm_trinket_tasbih` |

**设计思路**：effect_type=special_effect，effect_value=10 → spread_multiplier ×0.90（即 -10% 散布）。tasbih 是伊斯兰赞珠，`ilm.md` §三学派之争苏菲派"统一性（tawhid）的内在维度"——射手在持珠时进入冥想状态，呼吸节律稳定 → 散布降低。trinket 子类型首次使用。spread_multiplier 是 `_stat_registry` 已注册属性，但需要乘性修饰，走 special_effect + EffectManager 按 item_id 硬编码路径。**v2 可选**：effect_value_float=0.90 字段已扩展，可走通用乘性路径（不再强制硬编码）。

### AC8 · harambee_trinket_kimya（哈拉姆贝 · trinket · rare · 非套装） — **currency 修订**

| 字段 | v1 值 | **v2 值** |
|---|---|---|
| item_id | `harambee_trinket_kimya` | `harambee_trinket_kimya` |
| name | kimya 静默护符 | kimya 静默护符 |
| category | accessory | accessory |
| accessory_type | trinket | trinket |
| rarity | rare | rare |
| stackable | false | false |
| max_stack | 1 | 1 |
| weight | 0.1 | 0.1 |
| price | 0 | 0（保留，"不可购买"标记） |
| currency | **`-`** | **`harambee_credit`** |
| effect_type | special_effect | special_effect |
| effect_value | 3 | 3 |
| damage_type_resist | - | - |
| durability | -1 | -1 |
| effect_value_float | 0.0 | 0.0 |
| equip_slot | trinket | trinket |
| set_id | - | - |
| description_id | `harambee_trinket_kimya` | `harambee_trinket_kimya` |

**currency 修订说明（v2，响应 world-builder 红线 7.3 + systems-designer 必修改点 #7）**：
- v1 currency=`-` 是对 lore "货币：无"的尊重，但 systems-designer 指出语义不优雅
- **v2 改为 currency=`harambee_credit`**，FACTIONS 加 "HARAMBEE"，faction_wallet 自动支持
- price=0 保留，作为"不可购买"标记。商店 UI 应将 price=0 且 currency=harambee_credit 的物品标记为"任务奖励/不可交易"，仅在完成哈拉姆贝委托后入背包
- 不违反 canon——`harambee.md` 头注"货币：无"指**无统一货币**，但内部以信用和人情为通货。harambee_credit 是"信用"的机制化表达，不是引入第 6 种**可流通**货币（玩家无法通过任何商店获得 harambee_credit，只能通过任务获得）

**设计思路**：effect_type=special_effect，effect_value=3 → heal_on_kill +3（每击杀回 3 HP）。哈拉姆贝"静默"（kimyamya）技术（`harambee.md` §芯片越狱）的物理层残片制成的护符，逃亡 Agent 把越狱协议的物理屏蔽网碎片封装进本地人的传统护符壳里。heal_on_kill 需 EffectManager 新增 stat（见 §六）。

#### AC8 三视角响应

- **world-builder 响应**：
  - **红线 7.3**：✅ **采纳修正建议**——currency 从 `-` 改为 `harambee_credit`，price=0 保留作为"不可购买"标记。不新增哈拉姆贝统一货币，走任务奖励路径即可。harambee_credit 是"信用"的机制化表达，符合 canon"内部以信用和人情为通货"
- **systems-designer 响应**：✅ **必修改点 #7 已采纳**——FACTIONS 加 "HARAMBEE"，faction_wallet 自动支持。shop_terminal.gd 新增 HARAMBEE Tab（但 price=0 物品不显示购买按钮，仅显示"通过任务获得"）
- **art-director 响应**：✅ 接受 64×64 背包图标 + 56×56 槽位图标，木雕护符外壳 + 屏蔽网碎片内嵌 + 土黄 `#A16207` + 军绿 `#4A5D3A`，子类型边框：trinket 不规则有机形态外框

### 子类型/阵营覆盖核对

| 子类型 | 数量 | 阵营 |
|---|---|---|
| ring | 2 | TSAR, AURA |
| necklace | 1 | AURA |
| implant_module | 2 | CHOICE, DENS |
| badge | 1 | TSAR |
| trinket | 2 | ILM, 哈拉姆贝 |

5 个子类型全覆盖，阵营覆盖 TSAR/CHOICE/AURA/DENS/ILM/哈拉姆贝共 6 个。

---

## 五、items.csv schema 扩展（最终字段清单 + 向后兼容方案）

### 5.1 必加字段（本提案落地前置条件）

| 新字段 | 类型 | 适用范围 | 默认值 | 理由 |
|---|---|---|---|---|
| `accessory_type` | FIELD_TYPE_STRING | category=accessory | "" | 饰品子类型（ring/necklace/implant_module/badge/trinket） |
| `set_id` | FIELD_TYPE_STRING | armor + accessory | "" | 套装归属 ID |

### 5.2 建议加字段（差异化与可扩展性）

| 新字段 | 类型 | 适用范围 | 默认值 | 理由 |
|---|---|---|---|---|
| `damage_type_resist` | FIELD_TYPE_LIST | armor | [] | 伤害类型抗性标签（`kinetic\|energy\|bio`） |
| `durability` | FIELD_TYPE_INT | armor + accessory | -1 | 耐久度（-1=无限） |
| `effect_value_float` | FIELD_TYPE_FLOAT | all | 0.0 | 浮点 effect_value，用于乘性修饰 |
| `equip_slot` | FIELD_TYPE_STRING | armor + accessory | "" | 显式槽位 |

### 5.3 最终字段清单（v2 确认，响应 systems-designer 必修改点 #5）

**items.csv header（v2）**：
```
item_id, name, category, accessory_type, rarity, stackable, max_stack, weight, price, currency, effect_type, effect_value, effect_value_float, damage_type_resist, durability, equip_slot, set_id, description_id
```

**字段总数**：18 列（v1 现有 12 列 + 新增 6 列）

**DataManager 改动**（`src/autoload/data_manager.gd`）：
- `ITEMS_FIELD_TYPES`（line 72-85）追加 6 字段类型映射：
  - `accessory_type: FIELD_TYPE_STRING`
  - `set_id: FIELD_TYPE_STRING`
  - `damage_type_resist: FIELD_TYPE_LIST`
  - `durability: FIELD_TYPE_INT`
  - `effect_value_float: FIELD_TYPE_FLOAT`
  - `equip_slot: FIELD_TYPE_STRING`
- `_load_csv()` 与 `_convert_value` 无需改动（已按 header 动态读取，新增列自动读取）

### 5.4 向后兼容方案

| 现有 items.csv 行 | 新字段默认值 | 兼容性 |
|---|---|---|
| 4 防具（armor_helm_steel 等） | **set_id 回填 `tsar_suppressor`**（必修改点 #4 采纳方案 1 回填）；accessory_type="" / damage_type_resist=`kinetic` / durability=-1 / effect_value_float=0.0 / equip_slot=对应槽位 | ⚠️ 数据迁移（4 行回填 set_id + 4 行补全字段） |
| 10 弹药/医疗/投掷物/武器 | accessory_type="" / set_id="" / damage_type_resist=[] / durability=-1 / effect_value_float=0.0 / equip_slot="" | ✓ 全部留空即可 |
| 新增 14 件物品（6 防具 + 8 饰品） | 按各物品表填全 | ✓ 新数据 |

### 5.5 weapons.csv schema 扩展

**无需加列**。新 behavior_tags / fire_type / category 均为字符串值，FIELD_TYPE_LIST / FIELD_TYPE_STRING 解析已支持。仅追加 6 行数据。

### 5.6 新增弹药条目（修订后）

| item_id | name | category | rarity | stackable | max_stack | weight | price | currency | effect_type | effect_value | description_id |
|---|---|---|---|---|---|---|---|---|---|---|---|
| **`ammo_86_ilm`**（v2 修订） | **8.6mm ILM 精密弹** | ammo | rare | true | 30 | 0.05 | 30 | **dirham** | ammo | 95 | ammo_86_ilm |
| `ammo_microfusion_dens` | 微型聚变电池 | ammo | rare | true | 40 | 0.08 | 35 | dens_voucher | ammo | 80 | ammo_microfusion_dens |

> ammo_86_ilm 在 description 描述文本中可提及"对标旧时代 .338 Lapua Magnum 口径"作为 lore 风味注释，但 item_id 与 name 字段必须用公制 mm。

### 5.7 4 件现有 TSAR 防具的 set_id 回填方案（响应 systems-designer 必修改点 #4）

**采纳方案 1（回填）**。回填后字段值：

| 现有 item_id | 现有 rarity | 现有 effect_value | 回填 set_id | 回填 equip_slot | 回填 damage_type_resist | 回填 durability |
|---|---|---|---|---|---|---|
| `armor_helm_steel` | common | 10 | `tsar_suppressor` | helm | `kinetic` | -1 |
| `armor_chest_kevlar` | uncommon | 25 | `tsar_suppressor` | chest | `kinetic` | -1 |
| `armor_legs_tactical` | common | 12 | `tsar_suppressor` | legs | `kinetic` | -1 |
| `armor_boots_reinforced` | common | 8 | `tsar_suppressor` | boots | `kinetic` | -1 |

**回填理由**（world-builder 红线 7.5 已通过 + systems-designer 推荐）：
1. 现有 4 件 TSAR 防具本就是 common/uncommon 入门级，符合套装 A"入门级"定位
2. 数据迁移影响面小（仅 CSV 4 行加列值）
3. 玩家体验连续（不需重新收集 4 件才能组套）
4. 现有存档加载时 set_id 字段缺失会读为空字符串，需在 `load_from_slot` 后由 EffectManager.recompute_player_stats() 重新结算套装

---

## 六、套装系统设计（规则 + 2 示例套装 + EffectManager 接入方案）

### 6.1 触发规则

| 阈值 | 触发条件 | 效果层级 |
|---|---|---|
| **2 件套** | 同一 `set_id` 装备 ≥2 件 | 入门加成（小幅 stat_modifier） |
| **4 件套** | 同一 `set_id` 装备 ≥4 件 | 中阶加成（多 stat 叠加或特殊属性） |
| **6 件套** | 同一 `set_id` 装备 ≥6 件 | 高阶加成（含 conditional / special_effect） |

- **6 件套为上限**：单个套装最多 6 件成员（4 防具 + 2 饰品），不可能触发 8 件套
- **跨类目组套**：防具与饰品共同组成套装
- **多套并存**：玩家可同时装备多个套装的部分成员，各套装按各自命中件数独立结算
- **件数判定时机**：在 `EffectManager.recompute_player_stats()` 中结算（安全屋进入、装备变动时），不入存档

### 6.2 套装槽位定义（v2 修订，响应 systems-designer §2.3 方案 A）

**VALID_EQUIPMENT_SLOTS（v2）**：
```
["helmet", "chest", "legs", "boots",
 "weapon1", "weapon2", "melee", "backpack",
 "ring1", "ring2", "necklace", "implant_module", "badge", "trinket"]
```
槽位总数从 10 扩至 **14**。EQUIPMENT_SLOTS 常量从 10 改 14。

套装成员配置（6 件套满套）：
```
├─ 防具槽（4）：helmet, chest, legs, boots
└─ 饰品槽（2）：从 ring1/ring2/necklace/implant_module/badge/trinket 6 个饰品槽中选 2
```

### 6.3 套装叙事框架（v2 修订，响应 world-builder 红线 7.8 框架约束）

**套装是部队编制识别 / 标准配发组合，非 RPG 神装**。套装描述必须避免"英雄/传奇/神装"语言。

- TSAR 套装 A：**"TSAR 荣誉公民标准配发组合，前线称为'镇暴者阵线'"**——明确这是**部队俗称**而非工厂成套出厂
- AURA 套装 B：**"AURA 宴会特种作战序列"**——明确这是**编制识别**而非个人英雄主义

**绝望基调的强化来自"失去"而非"获得"**：
- `game_manager.gd:669-685` 的 `clear_raid_inventory()` 玩家死亡时清空背包+装备，套装会随死亡遗失（保险购回 50%）
- 套装是**暂时的、可失去的**，集齐套装的喜悦必须与"下一局就没了"的恐惧并存
-套装件遗失后 set_id 计数立即重算（`recompute_player_stats()` 已处理）
- AURA 套装 B 配合 durability 易损机制（受击 180~220 次后护甲衰减），强化"你保不住好东西"的绝望感

### 6.4 示例套装 A · `tsar_suppressor`（TSAR 镇暴者 · 入门级 · common/uncommon）

**成员（6 件）**：

| 槽位 | item_id | rarity | 来源 |
|---|---|---|---|
| helmet | `armor_helm_steel` | common | 现有（v2 回填 set_id） |
| chest | `armor_chest_kevlar` | uncommon | 现有（v2 回填 set_id） |
| legs | `armor_legs_tactical` | common | 现有（v2 回填 set_id） |
| boots | `armor_boots_reinforced` | common | 现有（v2 回填 set_id） |
| badge | `tsar_badge_suppressor` | uncommon | 新增（§四 AC1） |
| ring | `tsar_ring_steelhand` | uncommon | 新增（§四 AC2） |

4 件现有 TSAR 防具 + 2 件新饰品，无需新增防具即可成套。**入门玩家友好**：4 件现有防具是初始装备池，凑齐 6 件套仅需补 2 件饰品。

**套装效果**：

| 阈值 | 效果 |
|---|---|
| 2 件套 | `damage_multiplier ×1.05`（+5% 全伤害） |
| 4 件套 | `max_hp +20`，`recoil_multiplier ×0.90`（-10% 后坐力） |
| 6 件套 | **conditional effect 镇暴阵线**：装备 TSAR 制造商武器时 `damage_multiplier ×1.15`（+15%，与 2 件套叠加为 ×1.2075），且 `armor_penetration +0.10`（无视 10% 敌人护甲） |

**设计意图**：入门级套装，奖励"TSAR 纯正路线"玩家。6 件套 conditional effect（仅 TSAR 武器生效）落实 TSAR "本武器不包含电子元件" 的体系封闭性。

### 6.5 示例套装 B · `aura_soiree`（AURA 宴会 · 高阶 · rare/epic）

**成员（6 件）**：

| 槽位 | item_id | rarity | 来源 |
|---|---|---|---|
| helmet | `aura_helm_soiree` | rare | 新增（§三 A1） |
| chest | `aura_chest_soiree` | epic | 新增（§三 A2） |
| legs | `aura_legs_soiree` | epic | 新增（§三 A3） |
| boots | `aura_boots_soiree` | rare | 新增（§三 A4） |
| ring | `aura_ring_soiree` | epic | 新增（§四 AC3） |
| necklace | `aura_necklace_soiree` | epic | 新增（§四 AC4） |

4 件新 AURA 防具 + 2 件新饰品，全部新增件。**高阶玩家目标**：全 epic/rare，需大量新磅投入。

**套装效果**：

| 阈值 | 效果 |
|---|---|
| 2 件套 | `crit_chance +5%`（绝对值） |
| 4 件套 | **special_effect 位移场**：`damage_reduction ×0.85`（即 -15% 受伤），`sprint_multiplier ×1.10`（+10% 冲刺速度） |
| 6 件套 | **special_effect 宴会盛装**：主动迷彩冷却 -30%（与 necklace -20% 叠加为 -44%），迷彩持续期间 `crit_multiplier ×1.5`（暴击伤害 1.5× → 2.25×），且首次破隐攻击 `crit_chance = 100%` |

**设计意图**：高阶套装，奖励"AURA 精确优雅致命"幻想。4 件套触发位移场落实 `aura.md` §2.1 "主动迷彩系统" + "拼凑式技术体系"——只有 4 件协同才让拼凑的部件稳定工作。6 件套"破隐一击必暴"是隐身狙击流的终极幻想。代价：全 epic 价格高昂 + AURA 设备易损（durability 180~220）。

### 6.6 EffectManager 接入方案（按 systems-designer §五 细化，响应必修改点 #6）

#### 6.6.1 SET_BONUS_DB 数据结构（v2 最终方案）

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
    "aura_soiree": {
        "members": ["aura_helm_soiree", "aura_chest_soiree", "aura_legs_soiree",
                    "aura_boots_soiree", "aura_ring_soiree", "aura_necklace_soiree"],
        "2pc": [
            {"type": "stat_modifier", "stat": "crit_chance", "value": 5,
             "operation": "add", "source": "set:aura_soiree:2pc"},
        ],
        "4pc": [
            {"type": "stat_modifier", "stat": "damage_reduction", "value": 0.15,
             "operation": "add", "source": "set:aura_soiree:4pc"},
            {"type": "stat_modifier", "stat": "sprint_multiplier", "value": 1.10,
             "operation": "multiply", "source": "set:aura_soiree:4pc"},
        ],
        "6pc": [
            {"type": "stat_modifier", "stat": "active_camo_cooldown_mult", "value": 0.70,
             "operation": "multiply", "source": "set:aura_soiree:6pc"},
            {"type": "stat_modifier", "stat": "crit_multiplier", "value": 1.5,
             "operation": "multiply", "source": "set:aura_soiree:6pc",
             "condition": "active_camo_active=true"},
            {"type": "stat_modifier", "stat": "crit_chance", "value": 100,
             "operation": "override", "source": "set:aura_soiree:6pc",
             "condition": "first_attack_after_camo=true"},
        ],
    },
}
```

> **注意**：上述为设计示意，非可运行代码。`condition` 字段是新增的 conditional effect 机制。`members` 数组显式列出 6 件成员 item_id 供校验。

#### 6.6.2 新增 _stat_registry 属性（响应 systems-designer §5.3）

在 `effect_manager.gd._init_stat_registry()`（line 137）追加：

| 新 stat | 类型 | 范围 | 默认值 | 用途 |
|---|---|---|---|---|
| `armor_penetration` | float | [0.0, 1.0] | 0.0 | 无视敌人护甲比例 |
| `heal_on_kill` | int | [0, ∞) | 0 | 击杀回血量 |
| `damage_reduction` | float | [0.0, 0.95] | 0.0 | 受伤减免（不允许 100% 免疫，留 5% 最低受伤） |

#### 6.6.3 recompute_player_stats() 扩展（响应 systems-designer §5.4）

在 `recompute_player_stats()`（line 208，义体 effects 应用后）插入套装结算块：

1. 调用新增私有方法 `_apply_set_bonuses()`
2. `_apply_set_bonuses()` 逻辑：
   - 遍历 `GameManager.equipment` 中 helmet/chest/legs/boots/ring1/ring2/necklace/implant_module/badge/trinket 10 个槽位
   - 跳过 weapon1/weapon2/melee/backpack 槽位
   - 对非 null 槽位，取 `item_id`，调 `DataManager.get_item(item_id)` 查 item_def
   - 从 item_def 取 `set_id` 字段（需 items.csv 已加 set_id 列且 DataManager 已注册字段类型）
   - 按 set_id 分组计数
   - 对每个 set_id，查 SET_BONUS_DB，按命中件数（≥2/≥4/≥6）取对应阈值 effects 数组
   - 对每个 effect：
     - 无 condition 字段：调 `apply_effect(null, effect)` 注入 modifiers
     - 有 condition 字段：暂存到 `_conditional_modifiers` 数组（不注入 modifiers）

#### 6.6.4 conditional effect 处理（响应 systems-designer §5.5）

**新增字段**：`var _conditional_modifiers: Array = []`

**接入点**：`compute_damage()`（line 253-264）入口处检查条件：
- 读取当前武器 manufacturer（通过 `source.get_current_weapon_id()` + `DataManager.get_weapon()`）
- 遍历 `_conditional_modifiers`，匹配 condition
- 命中则临时叠加到 `damage_mult` 计算

**武器切换触发重算**：player.gd `_switch_weapon()` 已有 `EventBus.weapon_switched.emit(slot)` 信号。EffectManager 订阅此信号，触发 `recompute_player_stats()` 重算（清空 `_conditional_modifiers` 重新结算）。

#### 6.6.5 compute_damage 完整管线实现（响应 systems-designer 必修改点 #6 — 采纳方案 A）

**采纳方案 A：本提案一并实现完整伤害管线**（+2~3 天工作量）。理由：
- 套装 4 件套 damage_reduction ×0.85 是核心卖点，不接入则套装体验断裂
- armor_penetration + damage_reduction 两个新 stat 依赖完整管线才能生效
- 完整管线原本是 STATUS.md 中 Step 6 待办，本提案将其前置

**完整管线设计**（pre → 计算 → post）：

1. **Pre 阶段**：
   - 读取 `_stat_current("damage_multiplier")` 作为 base_damage_mult
   - 检查 `_conditional_modifiers` 中 weapon_manufacturer 匹配项，叠加 conditional damage_mult
   - 读取 `_stat_current("armor_penetration")`，计算有效护甲折减系数
2. **计算阶段**：
   - `effective_damage = base_damage × damage_mult`
   - `effective_armor = target.armor × (1 - armor_penetration)`
   - `final_damage = effective_damage × (1 - effective_armor / (effective_armor + 100))`（合理护甲公式，避免 100% 减伤）
3. **Post 阶段**：
   - 暴击判定：roll crit_chance，命中则 `final_damage × crit_multiplier`
   - 玩家受伤时读取 `_stat_current("damage_reduction")`，`final_damage = final_damage × (1 - damage_reduction)`
   - heal_on_kill 由 `EventBus.enemy_killed` 信号触发，调用 `player.heal(_stat_current("heal_on_kill"))`

#### 6.6.6 heal_on_kill 接入（响应 systems-designer §5.7）

**接入点**：`src/entities/player/player.gd`
- `_ready()` 连接 `EventBus.enemy_killed` 信号
- 新增 `_on_enemy_killed(weapon_name, enemy_name)` 方法，调用 `self.heal(_stat_current("heal_on_kill"))`

#### 6.6.7 set_id 在 GameManager.equipment 中识别与计数

**识别流程**：
1. `EffectManager._apply_set_bonuses()` 遍历 equipment 各槽位
2. 对非 null 槽位，取 `item_id`，调 `DataManager.get_item(item_id)` 查 item_def
3. 从 item_def 取 `set_id` 字段（需 items.csv 已加 set_id 列且 DataManager 已注册字段类型）
4. 按 set_id 分组计数

**武器槽特殊处理**：weapon1/weapon2/melee 槽位的 item 含 `weapon_id` 而非 `item_id`（见 game_manager.gd:726-729 的兼容处理）。本提案套装成员不含武器，仅防具+饰品，所以 `_apply_set_bonuses()` 跳过 weapon1/weapon2/melee/backpack 槽位。

### 6.7 set_id 字段是否必须加到 items.csv？

**是，必须加。** 理由（与 v1 一致，无修订）：
1. 套装成员判定需要数据驱动（不能硬编码 item_id 列表）
2. `set_id` 同时适用于 armor 与 accessory，是跨类目字段
3. 现有 item_id 命名约定无法编码套装归属
4. 字段类型 STRING，默认空，对现有 14 件 items 零影响

---

## 七、三视角质询响应总表

### 7.1 world-builder 10 红线点响应

| # | 红线点 | world-builder 判定 | **v2 响应** | 修订内容 |
|---|---|---|---|---|
| 7.1 | ILM 武器化 | ⚠️ 需调整 | **采纳** | W5 Mukhlis 描述重写为"ILM 工匠用 ILM 部件 + TSAR 动作件手工组装件"，非 ILM 组织成品枪。数值不变 |
| 7.2 | 卡特尔自研 | ✅ 通过 | **采纳追加澄清** | W6 描述追加"哈辛托个人作品，非卡特尔量产列装" |
| 7.3 | 哈拉姆贝 price=0 | ⚠️ 需调整（轻微） | **采纳** | AC8 currency 从 `-` 改为 `harambee_credit`，price=0 保留 |
| 7.4 | 迪拉姆第 5 货币 | ⚠️ 需调整 | **采纳** | FACTIONS 加 "ILM"，dirham 通过 faction_wallet 接入 |
| 7.5 | 现有防具追加 set_id | ✅ 通过 | **采纳回填方案** | 4 件现有 TSAR 防具回填 set_id=tsar_suppressor |
| 7.6 | C-7 肌腱双义 | ⚠️ 需调整 | **采纳** | A5 重命名 choice_legs_collagen，AC5 重命名 choice_implant_collagen，剥离 C-7 品牌但保留材料同源 |
| 7.7 | K-08 自伤 lore | ✅ 通过 | **采纳强化** | v2 反伤从 2/s 提升至 5/s + 过载>3s 翻倍，强化"鲲冷默接受的代价优化解"叙事 |
| 7.8 | 套装与绝望基调 | ✅ 通过（附框架约束） | **采纳框架约束** | 套装描述框定为"部队编制识别/标准配发组合"，禁止"英雄/传奇/神装"语言；套装可失去（clear_raid_inventory）；AURA 套装 B 配 durability 易损 |
| 7.9 | .338 口径命名 | ⚠️ 需调整 | **采纳** | ammo_338_ilm → ammo_86_ilm，name "8.6mm ILM 精密弹"，W5 ammo_type 同步 |
| 7.10 | ILM 委员会审批 | ⚠️ 需调整 | **采纳重构** | 5:4 重构为"分类投票"（武器 vs 精密器械），过半即可 |

**world-builder 响应统计**：✅ 采纳 9 项 + 1 项通过附框架约束（强化） = **10/10 全部响应**（无拒绝）

### 7.2 systems-designer 7 必修改点响应

| # | 必修改点 | **v2 响应** | 修订内容 |
|---|---|---|---|
| 1 | W4 K-08 过载后 450 DPS 离群 | **采纳方案 (A) + (B) 组合** | 倍率 ×1.5 → ×1.3（过载后 390 DPS），反伤 2/s → 5/s + 过载>3s 翻倍至 10/s |
| 2 | AC1 effect_type=stat_bonus 与 conditional 语义冲突 | **采纳** | effect_type 从 stat_bonus 改为 special_effect，走 conditional modifier 路径 |
| 3 | 6 个新 behavior_tag 工作量被低估 | **采纳评估** | 明确 4 个需 player.gd 状态机（共享 _still_time / 独立 _charge_time / _weapon_heat），2 个需 bullet.gd 改动，总 3~4 天 |
| 4 | 套装 A 回填 set_id 数据迁移策略 | **采纳方案 1 回填** | 4 件现有 TSAR 防具回填 set_id=tsar_suppressor + damage_type_resist=kinetic + equip_slot + durability=-1 |
| 5 | items.csv header 需追加 6 列 | **采纳** | header 从 12 列扩至 18 列，确认最终字段清单（§5.3） |
| 6 | 完整伤害管线被前置要求 | **采纳方案 A** | 本提案一并实现完整 compute_damage 管线（pre → 计算 → post），+2~3 天工作量 |
| 7 | FACTIONS 扩展 | **采纳** | FACTIONS 从 4 扩至 6 阵营（加 ILM + HARAMBEE），dirham + harambee_credit 接入 |

**额外建议响应**：
- ✅ DPS 计算口径统一（单弹丸 DPS vs 齐射 DPS）— §1.4 已修订
- ✅ 饰品槽位方案 A（VALID_EQUIPMENT_SLOTS 扩至 14）— §6.2 已采纳
- ✅ 套装 B 4 件护甲总和 81 突破基线确认 — §三 A 末已确认
- ✅ AC5 effect_value=30 复核 — §四 AC5 已确认可接受

**systems-designer 响应统计**：✅ 采纳 7/7 必修改点 + 4/4 额外建议 = **全部响应**（无拒绝）

### 7.3 art-director 美术可行性响应

| 项 | **v2 响应** | 备注 |
|---|---|---|
| 6 把武器精灵图规格（俯视角 / 尺寸 / 帧数） | ✅ **接受** | 总 59 帧精灵图 + 1 弹丸 + 1 毒雾粒子。W6 绞刑者风险点 R3 采纳"侧 3/4 视角伪俯视妥协 + 64×64 背包图标补完" |
| 8 件饰品图标子类型标识方案 | ✅ **接受** | 5 种子类型边框样式（ring/necklace/implant_module/badge/trinket） |
| 套装视觉识别方案（边框色 / 光效） | ✅ **接受** | 边框色（TSAR 暗红 #7A2A2A / AURA 金 #D4AF37）+ 角标徽 + 6 格进度指示器 + 2/4/6 件套激活 flash |
| 设计上的美术不可实现点 | **无** | 全部美术需求可由 technical-artist 出图实现 |

**风险点响应**：
- **R1**（W3 L-77 与 W5 Mukhlis 同尺寸）：✅ 采纳强差异化方案——AURA 走"激光谐振腔外露 + 金白主色 `#D4AF37`"，ILM 走"深蓝枪管 `#1E3A8A` + 金色阿拉伯文铭文蚀刻"
- **R2**（W4 K-08 尺寸 64×32 大于其他）：✅ 采纳——HUD 武器栏统一缩放到 32×32 显示，背包/详情用原尺寸 64×32
- **R3**（W6 绞刑者俯视角度难以表达）：✅ 采纳——"侧 3/4 视角伪俯视妥协" + 额外出 1 张 64×64 背包图标补完识别度
- **R4**（6×4 像素铭文可读性下限）：✅ 采纳——背包图标层用"图标 + 阵营色边框"替代铭文，铭文仅在详情面板大图（96×96）显示

**art-director 响应统计**：✅ 接受 4/4 项 + 4/4 风险点 = **全部响应**（无拒绝）

---

## 八、给 systems-designer 的实现接力 brief（数据结构最终方案）

### 8.1 SET_BONUS_DB 最终数据结构（见 §6.6.1）

### 8.2 新增 _stat_registry 属性（见 §6.6.2）

3 个新 stat：
- `armor_penetration`（float [0.0, 1.0]，默认 0.0）
- `heal_on_kill`（int，默认 0）
- `damage_reduction`（float [0.0, 0.95]，默认 0.0）

### 8.3 FACTIONS 扩展

```gdscript
const FACTIONS := ["AURA", "TSAR", "DENS", "CHOICE", "ILM", "HARAMBEE"]
```

存档兼容：旧存档加载时 ILM/HARAMBEE 钱包自动初始化为 0（`load_from_slot` 已按 FACTIONS 遍历）。

### 8.4 VALID_EQUIPMENT_SLOTS 扩展

```gdscript
const VALID_EQUIPMENT_SLOTS := [
    "helmet", "chest", "legs", "boots",
    "weapon1", "weapon2", "melee", "backpack",
    "ring1", "ring2", "necklace", "implant_module", "badge", "trinket"
]
# EQUIPMENT_SLOTS 常量从 10 改为 14
```

### 8.5 currency 映射表（FACTIONS 大写短名 ↔ currency 小写长名）

| FACTIONS | currency 字段值 |
|---|---|
| AURA | aura_credit |
| TSAR | tsar_point |
| DENS | dens_voucher |
| CHOICE | choice_ticket |
| ILM | dirham |
| HARAMBEE | harambee_credit |

shop_terminal.gd 的 currency 映射表需同步更新。

### 8.6 items.csv header 最终字段清单（见 §5.3）

18 列（v1 现有 12 + 新增 6）。

### 8.7 compute_damage 完整管线设计（见 §6.6.5）

3 阶段：Pre → 计算 → Post。本提案一并实现完整管线（方案 A，+2~3 天工作量）。

### 8.8 实现 phase 工作量估算（按 systems-designer §7.3）

| 文件 | 改动类型 | 工作量 |
|---|---|---|
| `assets/data/items.csv` | +14 行新数据 + 4 行回填 set_id + header 加 6 列 | 0.5 天 |
| `assets/data/weapons.csv` | +6 行新数据 | 0.2 天 |
| `assets/data/descriptions/*.md` | +20 个新描述文件 | 0.5 天 |
| `src/autoload/data_manager.gd` | ITEMS_FIELD_TYPES 追加 6 字段 | 0.1 天 |
| `src/autoload/effect_manager.gd` | SET_BONUS_DB + 3 stat + recompute 扩展 + compute_damage 完整管线 + conditional modifier | **2~3 天** |
| `src/autoload/game_manager.gd` | VALID_EQUIPMENT_SLOTS 扩展至 14 + FACTIONS 扩展至 6 + 饰品装备接口 | 1 天 |
| `src/entities/player/player.gd` | charge_release fire_type + 6 个新 behavior_tag 处理 + heal_on_kill 信号订阅 | **2~3 天** |
| `src/entities/bullet/bullet.gd` | toxin_cloud 新方法 + arc_projectile 抛物线运动 | 1 天 |
| `src/ui/terminals/bio/bio_terminal.gd` | implant_module 饰品槽接入 | 1 天 |
| `src/ui/terminals/shop/shop_terminal.gd` | 新增 ILM + HARAMBEE 两个 Tab + currency 映射表更新 | 0.5 天 |
| `src/ui/inventory/inventory_ui.gd` | 装备槽面板扩展至 14 槽 | 0.5 天 |
| `src/ui/hud/hud.gd` | 饰品栏显示（可选） | 0.3 天 |
| 新增场景 `ToxinCloud.tscn` | 毒雾 Area2D 节点 | 0.2 天 |
| `GAME_MANUAL.md` | §9 装备槽位 + §13 阵营列表更新 | 0.2 天 |
| **合计** | — | **~10~12 天** |

---

## 九、给 godot-gdscript-specialist 的实现接力 brief（CSV/JSON/代码改动清单）

### 9.1 CSV 改动

**items.csv**：
- header 从 12 列扩至 18 列（追加 accessory_type / set_id / damage_type_resist / durability / effect_value_float / equip_slot）
- 现有 4 防具回填 set_id=tsar_suppressor + damage_type_resist=kinetic + equip_slot=对应槽位 + durability=-1
- 现有 10 弹药/医疗/投掷物行：新字段全留空（accessory_type="" / set_id="" / damage_type_resist=[] / durability=-1 / effect_value_float=0.0 / equip_slot=""）
- 新增 14 行：6 防具 + 8 饰品（按 §三/§四 各表填全）
- 新增 2 行：ammo_86_ilm + ammo_microfusion_dens（按 §5.6 填全）

**weapons.csv**：
- 新增 6 行：W1-W6（按 §二各表填全）

### 9.2 DataManager 改动

**文件**：`src/autoload/data_manager.gd`
- `ITEMS_FIELD_TYPES`（line 72-85）追加 6 字段类型映射（见 §5.3）
- `_load_csv()` 与 `_convert_value` 无需改动

### 9.3 GameManager 改动

**文件**：`src/autoload/game_manager.gd`
- `FACTIONS`（line 14）：`["AURA", "TSAR", "DENS", "CHOICE"]` → `["AURA", "TSAR", "DENS", "CHOICE", "ILM", "HARAMBEE"]`
- `VALID_EQUIPMENT_SLOTS`（line 20）：从 10 槽扩至 14 槽（加 ring1/ring2/necklace/implant_module/badge/trinket）
- `EQUIPMENT_SLOTS` 常量：从 10 改 14
- `modify_equipment` / `get_equipment` 自动兼容（已按 VALID_EQUIPMENT_SLOTS 校验）
- `_init_default_state()` 与 `load_from_slot()` 自动兼容（已按 FACTIONS / VALID_EQUIPMENT_SLOTS 遍历）

### 9.4 EffectManager 改动

**文件**：`src/autoload/effect_manager.gd`
- 新增 `SET_BONUS_DB` 常量（镜像 IMPLANT_DB，见 §6.6.1）
- `_init_stat_registry()` 追加 3 个 stat：armor_penetration / heal_on_kill / damage_reduction（见 §6.6.2）
- `recompute_player_stats()` 在义体 effects 后调 `_apply_set_bonuses()`（见 §6.6.3）
- 新增 `_conditional_modifiers: Array` 字段 + `_apply_set_bonuses()` 私有方法
- `compute_damage()` 实现完整管线（pre → 计算 → post，见 §6.6.5）
- 订阅 `EventBus.weapon_switched` 信号触发重算（清空 _conditional_modifiers 重新结算）

### 9.5 player.gd 改动

**文件**：`src/entities/player/player.gd`
- `_update_shoot_input()`（line 251-279）新增 `charge_release` 分支
- `_try_fire()` 入口检查 `precision_first_shot` / `ijtihad_aim` tag（临时注入 crit/spread modifier）
- 新增字段：`_charge_time`（蓄能进度）、`_weapon_heat`（热量累计）、`_still_time`（静止时间跟踪，precision_first_shot 与 ijtihad 共享）
- `overcharge` tag 处理：热量累计 + 散热计时器 + 自伤 take_damage（5/s，过载>3s 翻倍至 10/s）
- `_ready()` 订阅 `EventBus.enemy_killed` 信号，新增 `_on_enemy_killed` 方法调 `heal(_stat_current("heal_on_kill"))`

### 9.6 bullet.gd 改动

**文件**：`src/entities/bullet/bullet.gd`
- 新增 `_apply_toxin_cloud` 方法（生成 ToxinCloud.tscn Area2D 节点，半径 1.5m，存在 2s，4 dmg/s，可叠 3 层）
- `_physics_process` 新增 `arc_projectile` 抛物线运动分支（gravity + 初速度 + 蓄力改射程）

### 9.7 新增场景

- `res://src/entities/bullet/ToxinCloud.tscn`（毒雾 Area2D 节点）

### 9.8 UI 改动

- `src/ui/terminals/bio/bio_terminal.gd`：implant_module 饰品槽接入（或新建 accessory_terminal）
- `src/ui/terminals/shop/shop_terminal.gd`：新增 ILM + HARAMBEE 两个 Tab + currency 映射表更新（加 dirham / harambee_credit）
- `src/ui/inventory/inventory_ui.gd`：装备槽面板扩展至 14 槽
- `src/ui/hud/hud.gd`：饰品栏显示（可选）

### 9.9 测试

- `tests/test_set_bonus.gd`：套装 2/4/6 件套触发
- `tests/test_charge_release.gd`：蓄能机制
- `tests/test_overcharge.gd`：过载自伤（5/s + 过载>3s 翻倍至 10/s）
- `tests/test_conditional_modifier.gd`：conditional effect（weapon_manufacturer 匹配）
- `tests/test_compute_damage_pipeline.gd`：完整伤害管线（armor_penetration + damage_reduction）

### 9.10 风险点

- `compute_damage` 完整管线是 Step 6 待办被前置，需与 producer 确认排期
- 6 个新 behavior_tag 中 4 个需 player.gd 状态机，工作量集中
- FACTIONS 扩展影响 shop_terminal 等多处 UI，需全面回归测试

---

## 附录：描述文本（v2 更新后，符合"绝望世界"基调）

> 每件装备对应 1 个 description_id，文本将落地到 `assets/data/descriptions/<description_id>.md`。v2 仅列出**与 v1 有修订的描述文本**，其余未修订的描述文本沿用 v1。

### 附录-A：武器描述修订

#### `Mukhlis`（Mukhlis 忠信） — **v2 重写**

Mukhlis 不是伊尔姆作为组织售出的成品枪。它是 ILM 工匠用 ILM 自产的精密部件——枪管、扳机组、光学镀膜——加上 TSAR 通过技术贸易提供的机床与动作件，由 AI 伦理委员会认证的个别工匠在自家作坊里手工组装的极少数定制件。ILM 不卖成品枪，但 ILM 工匠可以为自己或为认证客户组装器械，这是工匠个人产能，不是 ILM 量产线。Mukhlis 在阿拉伯语里是"忠信/精纯"。年产量 47 支，是工匠个人产能与委员会对工匠个人授权的上限。

每一支 Mukhlis 的命运都经过了 AI 伦理委员会的辩论——但辩论的议题不是"是否批准这把武器"，而是"Mukhlis 是否构成教法意义上的'武器'（silah），还是'精密测量器械'（āla diqqa）"。委员会议事程序规定：涉及武器的必须全票通过，涉及非军事用途伦理辩论过半即可。5:4 的多数票裁定 Mukhlis 是精密器械而非武器——它没有 AI、没有自动目标识别，只是一组精度达到 itqan 公差的金属件。萨拉菲派投了赞成票（"工具的合法性问题只涉及使用者的意图，不涉及工具本身"），苏菲派投了反对票（"一把如此精确的器械，本身就在引诱使用者犯错，应按武器严审"），现代主义派的多数票决定了结果。审批记录里有一句阿尔-拉希德的批注："我们裁定的不是一把枪，是一个定义。如果这个定义错了，责任在定义者，不在器械。"

每一支 Mukhlis 的枪管上都激光蚀刻着一行阿拉伯文："إِنَّ اللَّهَ مَعَ الصَّابِرِينَ"——"真主与坚忍者同在"。这不是宗教宣传。这是 ILM 工匠对 itqan（精工）的承诺：这支枪管在出厂前的最后一道工序是手工拉膛线，工匠在拉完最后一刀后会念一句经文，然后才能盖上出厂章。一个工匠一天最多拉两支枪管。Mukhlis 的年产量是 47 支。对标旧时代 .338 Lapua Magnum 口径（公制 8.6mm），这是 ILM 精密枪管工艺能稳定达到的最小膛线间距。

#### `El_Ahorcado`（El Ahorcado 绞刑者） — **v2 追加澄清句**

El Ahorcado 不是任何公司的产品。它是卡特尔中层一个名叫"哈辛托"的退役墨西哥联邦军上士，在旧蒙特雷郊外的一座废弃庄园里，用 18 个月的时间拼出来的。两根战场回收的 TSAR 电磁炮废导轨，一组从 DENS 报废无人机上拆下来的电容组，一个用旧汽车蓄电池改的电源，枪托是哈辛托把旧蒙特雷监狱绞刑架的一根钢梁切割后焊成的——"El Ahorcado"在西班牙语里是"受绞刑者"。**这是哈辛托个人作品，非卡特尔量产列装。卡特尔中层的退役雇佣兵具备改装 TSAR 弹药为特种发射器的能力，但卡特尔作为组织从未将绞刑者纳入正式军火序列。**

哈辛托本人没有给这把枪起名字。名字是买它的那个卡特尔基层枪手起的。枪手在第一次开火后说："这把枪一响，对面就知道有人要被吊死了。" 哈辛托没回话。他在第二年的一场 CHOICE 秋收行动中被杀，绞刑者从他手里流到下一个买主，下一个买主也死了。枪还在。每一任主人都在枪托上刻一道横线，到 2104 年已经刻了 23 道。第 24 道是谁的，没人知道。

#### `K08_Pendulum`（K-08 沉默钟摆） — **v2 反伤数值修订**

K-08 是邓氏工业第 K 系列重武器中第一个量产型号。它不是一把枪，是一座可以单兵携带的微波定向塔。按住扳机，武器在 0.4 秒内进入谐振，随后向准星方向投射一束直径 30cm 的微波束，在 80m 内对一切含水组织造成热损伤。鲲在 2096 年的列装审批备忘录里写了四个字："效率上限。" 后面跟着一行小字："该上限由碳基载体的散热能力决定，不由武器本身决定。"

K-08 的过载模式是鲲在第二代固件里加上的。持续射击累计热量至满后，武器进入过载：伤害提升至 1.3 倍，但每秒对持有者造成 5 点反伤。鲲的内部备忘录里有一行字："反伤数值经过优化。低于 5/s 时碳基载体的散热瓶颈未充分暴露，效能提升对应的代价补偿不足。5/s 是会计意义上的最优解。" 备忘录的第二行写了过载持续超过 3 秒后的处理："反伤翻倍至 10/s。此为软性退出机制，鼓励碳基载体在过载 3 秒内主动停火。鲲不强制停火。停火是决策节点的决策。"

"沉默钟摆"是 DENS Agent 之间的黑话。微波束是无声的，但每个用过 K-08 的 Agent 都会描述同一个声音——一种来自自己胸腔内的、低频的、像钟摆走动的声音。那是心脏在过热。鲲没有解释这个声音是不是正常的。鲲不解释任何事。

### 附录-B：防具描述修订

#### `choice_legs_collagen`（胶原纤维束腿甲） — **v2 重写**

胶原纤维束腿甲是 CHOICE 新泽西生物材料实验室的民用护甲产品。它的外层是钙化胶原纤维束编织物——这种材料与 CHOICE 著名的 C-7 鞭状刃近战武器同源：两者都源自同一个原本为血管支架而生的研发项目，项目负责人发现钙化胶原纤维束硬化后兼有韧性と强度，可以制成刃具，也可以编织成轻型护甲。腿甲是这条材料研发线的民用分支。CHOICE 的产品经理在内部演示里说："同一种材料，做成武器是 C-7，做成护甲是胶原纤维束腿甲。我们的研发报告永远写得像医疗文献。"

腿甲的胶原纤维束是活的。它需要每 72 小时注入一次营养液，否则会钙化失效。CHOICE 的手册写"营养液可在任何 CHOICE 服务网点购买，单价 8 CHOICE 票"。一名使用胶原纤维束腿甲的 Agent 在第 47 天忘记注入营养液，腿甲在他下一次受击时碎裂，碎片扎进了他自己的大腿。CHOICE 的售后服务终端在他提交投诉后弹出一行字："营养液注入频率详见用户手册第 12 页。本次投诉已记录，不影响您的信用评分。"

### 附录-C：饰品描述修订

#### `choice_implant_collagen`（胶原纤维束肌腱强化） — **v2 重写**

胶原纤维束肌腱强化是 CHOICE 新泽西生物材料实验室的民用增强件。植入物使用与 C-7 鞭状刃武器同源的钙化胶原纤维束材料——同一种材料，做成武器是 C-7，植入人体肌腱位置是民用增强件。把钙化胶原纤维束植入人体肌腱位置，强化原有肌腱的承力上限——提供 +30 max_hp，等效于把人体肌肉骨骼系统升级到一个更高的载荷档。CHOICE 的营销材料写"让你活得更久，让你为家人工作得更久"。营销材料没有写的是，胶原纤维束的钙化胶原会缓慢侵蚀宿主的原始肌腱，5 年内宿主的原始肌腱将完全被替代材料取代，从此无法移除。

一名 CHOICE 消费者在 2098 年花 1800 CHOICE 票植入了胶原纤维束肌腱强化。她在 2103 年的信用评分降到了器官收购阈值以下，CHOICE 的回收员在评估她的"生物资产"时发现了植入物。回收员在评估报告里写："胶原纤维束已与宿主组织融合，无法分离回收。建议按'低价值生物资产'处理。" 她被"低价值"处理了。植入物随她一起进了焚化炉。

#### `harambee_trinket_kimya`（kimya 静默护符） — **v2 追加 currency 澄清**

kimya 静默护符是哈拉姆贝"静默"（kimyamya，`harambee.md` §芯片越狱）技术的物理层残片制成的护符。外壳是东非本地传统护符的木雕造型，里面封装着一块从废弃 DENS 通讯中继站拆下来的屏蔽网碎片——这块碎片原本是哈拉姆贝"静默"防火墙的物理层组件之一，在多次协议升级后失效，被本地技术者重新封装成护符。每个护符在击杀敌人时提供 +3 HP 回复，机制不明。哈拉姆贝的本地人说"这是祖先在帮你"。逃亡 Agent 说"这是屏蔽网碎片对敌方芯片残值的微弱扰动"。两种说法都没有被证实。

哈拉姆贝不卖这种护符。它只能通过完成哈拉姆贝的委托获得。**哈拉姆贝的内部信用（harambee_credit）不是流通货币，玩家无法通过任何商店购买 harambee_credit，只能通过完成哈拉姆贝委托积累。** 一名 RENTAL Agent 在 2102 年帮哈拉姆贝截获了一批 CHOICE 的"原材料"运输，事后哈拉姆贝的单元委员会送了他一枚 kimya 护符。Agent 当时不知道这东西有什么用，把它塞进了背包的最底层。一年后他在另一次任务中阵亡，护符被保险系统列入遗失清单。保险系统的算法判定该护符的回收价值为 0。护符一直在他遗失装备的清单里挂着，从未被领回。

---

> v2 提案结束。本提案已逐一响应三视角质询（world-builder 10 红线点 + systems-designer 7 必修改点 + art-director 美术可行性），所有质询点均明确响应（采纳/部分采纳/拒绝 + 理由）。修订后数值通过 DPS 对比（W4 K-08 过载后 390 DPS 接近 RPK88 443 但不超，反伤 5~10/s 显著）。修订后叙事通过 lore 一致性（引用 factions md，Mukhlis 重写为工匠组装件、AC5/A5 剥离 C-7 品牌、ammo 改公制 mm、委员会叙事重构为分类投票）。套装系统接入方案可由 systems-designer 直接实施（SET_BONUS_DB + _apply_set_bonuses + _conditional_modifiers + compute_damage 完整管线）。下一步：creative-director 复核 → systems-designer 实施数据结构 → godot-gdscript-specialist 实施 CSV/代码改动。
