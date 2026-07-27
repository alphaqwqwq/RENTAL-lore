# 武器/防具/饰品大批量扩展提案

> 版本：2026-07-17 · 作者：game-designer · 状态：设计草案，待 world-builder / systems-designer 审查
> 依据：`assets/data/weapons.csv`（10 把）、`assets/data/items.csv`（4 件 TSAR 防具）、9 阵营 canon lore、`src/autoload/effect_manager.gd`、`src/autoload/data_manager.gd`、`GAME_MANUAL.md` §11/§17
> 铁律：本提案不写任何 GDScript 代码，不修改任何 CSV/JSON/.gd；仅产出设计文档。实现阶段由 godot-gdscript-specialist / systems-designer 接手。

---

## 一、扩展总览

### 1.1 数量与品类

| 类目 | 现有 | 本次新增 | 新增后总计 | 备注 |
|---|---|---|---|---|
| 武器 | 10 | **6** | 16 | 含 2 个新品类（marksman_rifle / heavy_weapon） |
| 防具 | 4（全 TSAR） | **6** | 10 | 覆盖非 TSAR 阵营 |
| 饰品 | 0 | **8** | 8 | 全新类目，5 个子类型 |
| 套装 | 0 | **2** | 2 | 1 个入门级 + 1 个高阶 |

### 1.2 阵营分布

| 阵营 | 现有武器 | 新增武器 | 新增防具 | 新增饰品 | 阵营定位（依据 canon） |
|---|---|---|---|---|---|
| TSAR | 3（AK2150/RPK88/TSAR30） | 1（手枪 PM-T） | 0 | 2（套装 A 配件） | "本武器不包含电子元件"，机械可靠性 |
| CHOICE | 2（G31/N4） | 1（冲锋枪 C-9） | 1（生物腿甲） | 1（义体 C-7 肌腱化用） | 生物材料、医疗军事化 |
| AURA | 2（L12/Milano） | 1（精确步枪 L-77） | 4（"宴会"全套） | 2（套装 B 配件） | 拼凑式精密、不稳定、优雅 |
| DENS | 3（luanhua/qbz99/ying） | 1（重武器 K-08） | 0 | 1（算力义体） | 鲲式效率、智能武器 |
| **ILM（新）** | 0 | 1（精确步枪 Mukhlis） | 1（itqan 胸甲） | 1（tasbih 念珠） | 精密制造、itqan 公差、法学辩论 |
| **卡特尔（新）** | 0 | 1（特种 绞刑者） | 0 | 0 | TSAR 过剩军火改装、黑市物流 |
| **哈拉姆贝（新）** | 0 | 0 | 0 | 1（kimya 护符） | 逃亡 Agent + 本地人拼凑 |
| 哈拉姆贝 / 伊尔姆 / 卡特尔 三新阵营中武器侧采用 ILM + 卡特尔；防具/饰品侧补 ILM + 哈拉姆贝 |

> **新阵营 lore 依据**（仅列武器/装备相关核心句）：
> - **ILM**：`ilm.md` §精密制造——"伊尔姆的武器制造能力在精密和致命程度上接近 AURA 的水准，但走的是完全不同的技术路线……他们的工程师把公差控制视为一种'对完美的宗教义务'（itqan）"；"核心产品不是量产武器，而是高度定制化的精密部件：枪管、扳机组、光学镜片镀膜"。→ 适合做 marksman_rifle。
> - **卡特尔**：`cartel.md` §与四大公司——"TSAR 的过剩产能和永不问买方是谁的销售政策，让这个转型在财务上完全可行"；"卡特尔的军火库以 TSAR 的 AK-2150 继承者和 TSAR-30 霰弹枪为主"。→ 适合做 TSAR 弹药改装的特种武器。
> - **哈拉姆贝**：`harambee.md` §人口构成——"逃亡 Agent……提供战术训练、武器维护、芯片技术知识"；§芯片越狱——"静默"是协议层 + 物理层拼凑。→ 适合做"拼凑义体/护符"类饰品。

### 1.3 新机制清单

#### 1.3.1 新 behavior_tags（语法 `tag1|tag2|tag3`，与 weapons.csv 一致）

| 新标签 | 含义 | 应用武器 |
|---|---|---|
| `precision_first_shot` | 装填/静止 ≥2s 后首发：spread 强制为 0 且暴击率 +25% | W1 TSAR PM-T |
| `toxin_cloud` | 弹丸命中点生成半径 1.5m 毒雾，存在 2s，4 dmg/s（DOT，可叠加层数上限 3） | W2 CHOICE C-9 |
| `charge_shot` | 按住蓄能 0.3~1.5s，伤害随蓄能线性提升至 1.5×，满蓄穿透多目标 | W3 AURA L-77 |
| `overcharge` | 持续射击累计热量至满后进入过载：伤害 ×1.5，但每秒对持有者造成 2 点反伤；停火 1.5s 散热 | W4 DENS K-08 |
| `ijtihad_aim` | 静止/蹲伏 ≥1.5s 后触发"独立推理"：暴击 +20%、spread ×0.6，移动后失效 | W5 ILM Mukhlis |
| `arc_projectile` | 抛物线弹道，命中点 3m 半径溅射；按住可调整射程（蓄力 0.3~1.2s 改变落点距离） | W6 卡特尔 绞刑者 |

> 全部 6 个新标签均沿用 `|` 分隔语法，CSV 解析为 `FIELD_TYPE_LIST`（见 `data_manager.gd` line 36/51-52），无需改解析器。

#### 1.3.2 新 fire_type

`GAME_MANUAL.md` §11 现有枚举：`semi_auto` / `full_auto` / `shotgun_spread` / `melee`。

新增 **`charge_release`**（蓄力释放）：按住扳机蓄能，松开激发；蓄能时长内可取消（右键或切武器）。蓄能进度通过 HUD 弧线显示（复用换弹弧线渲染）。

- 触发武器：W3 AURA L-77、W6 卡特尔 绞刑者
- 与现有 `semi_auto` 区别：`semi_auto` 点击即射，无蓄能；`charge_release` 必须蓄能 ≥0.3s 才能激发（蓄能不足时松开取消，不消耗弹药）
- `fire_rate` 字段在 `charge_release` 下含义改为"最大循环频率"（次/分钟），含蓄能+激发+恢复，作为射速上限而非 RPM

> 此为新枚举值，需 `bullet.gd` / `weapon.gd` 增加状态机分支。已标记给 godot-gdscript-specialist。

#### 1.3.3 新武器类别

| 新类别 | 定位 | 与现有类别区别 |
|---|---|---|
| `marksman_rifle` | 精确步枪/DMR | 介于 automatic_rifle 与（未来）sniper_rifle 之间；半自动/蓄能、高单发伤害、低射速、高精度、长射程；bullet_speed ≥1200 |
| `heavy_weapon` | 重武器 | 重量 ≥6.0、需 permit ≥2、高持续输出或大 AoE、移动惩罚（持有时 move_speed ×0.85）；通常伴随过热/过载机制 |

### 1.4 数值平衡锚点

参考现有 10 把武器推算 DPS（damage × fire_rate / 60）：

| 现有武器 | 直伤 DPS | 备注 |
|---|---|---|
| AK2150 | 450 | 全自动基准上限 |
| RPK88 | 443 | 弹链压制型 |
| qbz99 | 347 | 智能步枪 |
| L12 | 80（单发 80） | 激光半自动，靠精度补偿 |
| ying | 60 | 智能手枪 |
| Milano | 33（+ AoE） | 声波锥 |
| N4 | 30（+45 DOT） | 注射器 |
| TSAR30 | 135（15 弹丸×9） | 霰弹 |
| luanhua | 200（16 弹齐射） | 蜂群 |
| G31 | 210（+splash+护甲加成） | 生物弹 |

→ **新武器直伤 DPS 锚定区间**：30~300。低于 30 必须有强力辅助机制（DOT/AoE/控制）；高于 300 必须有显著代价（蓄能/反伤/低弹匣/重载）。所有新武器均落入此区间（详见各武器表"推算"行）。

---

## 二、武器扩展（6 把）

> 字段顺序与 `weapons.csv` 完全一致。`behavior_tags` 含新标签的已用 **粗体** 标注。所有 `default_attachments` 暂用空 JSON `{}` 或最小必装件，配件库扩展不在本提案范围（由 systems-designer 后续追加 attachments.csv）。

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
**设计思路**：TSAR 现有 3 把全是大枪，缺一把副武器。PM-T 填补该缺口，同时落实 TSAR "7.62 甚至更大" 的反潮流口径哲学——一把能装在 holster 里、却在 15m 内打出步枪级伤害的重手枪。`precision_first_shot` 鼓励"掏枪—一枪—收枪"的节奏，区别于 ying（DENS 智能连发）与 N4（CHOICE 注射 DOT）。机械结构、无电子元件，与 TSAR lore 完全一致。

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
**设计思路**：CHOICE 现有 G31（步枪 splash）+ N4（注射 DOT），缺一把 SMG 填近中距压制位。`toxin_cloud` 落实 CHOICE "医疗项目军事化" lore（C-7 肌腱同源研发线：原本是创伤喷雾装置，反过来变成污染弹）。毒雾留场封路，强化"绝望世界"节奏：玩家被迫在毒雾里抢人头否则被 DOT 磨死。

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
**设计思路**：AURA 现有 L12（半自动激光）+ Milano（声波 smg），缺一把精确步枪补远程位。`charge_shot` 与 AURA "拼凑式技术体系，理想条件下碾压，极端条件下失灵" lore 完美契合——满蓄 = 理想条件，蓄能中被打断 = 失灵。`pierce` 满蓄穿透，对应"激光瞬时命中"幻想。"长昼"呼应 L-12 日光，是 AURA 激光产品线的命名延续。

### W4 · K-08 沉默钟摆（DENS · heavy_weapon）

| 字段 | 值 |
|---|---|
| weapon_id | `K08_Pendulum` |
| name | K-08 沉默钟摆 |
| category | heavy_weapon |
| ammo_type | `ammo_microfusion_dens`（**新弹药**，见 §五） |
| manufacturer | DENS |
| quality | 3 |
| fire_type | full_auto |
| behavior_tags | `full_auto\|overcharge\|sustained_fire\|microwave\|energy` |
| damage | 18 |
| fire_rate | 1000 |
| magazine_size | 80 |
| reload_time | 5.5 |
| reload_type | full_mag |
| bullet_speed | 1500 |
| max_spread | 4.0 |
| spread_pershot | 0.2 |
| spread_recovery | 6.0 |
| recoil_type | none |
| recoil_pershot | 0 |
| recoil_recovery | 0 |
| aim_offset | 0 |
| required_permit | 2 |
| slots_available | `stock\|magazine\|optic` |
| slots_blocked | `muzzle\|grip` |
| slots_required | magazine |
| default_attachments | `{"magazine":"k08_std_cell"}` |
| description_id | `K08_Pendulum` |

**推算**：直伤 DPS = 18×1000/60 = **300**；过载后 ×1.5 = **450 DPS** 但反伤 2/s。对标 RPK88（443）持平，但 RPK88 是纯伤害、K-08 代价是持续掉血+大重量（8.0，move_speed ×0.85）。弹匣 80 配 5.5s 重载，定位"重压制但需要队友掩护"。
**设计思路**：DENS 现有 luanhua/qbz99/ying 三把都偏"智能精准"，缺一把重型压制武器落实"用机器打仗"哲学。`overcharge` 是鲲式效率崇拜的具象：把性能推到模型允许的边缘，代价转嫁给碳基决策节点（持有者本人）。微波（microwave）塔形武器，对应"鲲会让你确认它已经完成了战争"——按住扳机，AI 替你扫光一切，你只需承受它选择不告诉你的副作用。

### W5 · Mukhlis 忠信（ILM · marksman_rifle）

| 字段 | 值 |
|---|---|
| weapon_id | `Mukhlis` |
| name | Mukhlis 忠信 |
| category | marksman_rifle |
| ammo_type | `ammo_338_ilm`（**新弹药**，见 §五） |
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

**推算**：直伤 DPS = 95×50/60 ≈ **79**；`ijtihad_aim` 触发后暴击 +20%（按基础 1.5×暴击 + 20% 概率 → 期望 ×1.1）+ spread ×0.6，等效 ≈ **100 DPS** 且精度极高。对标 L12（80）+25%，但 L12 是激光瞬时、Mukhlis 是实弹抛壳，靠精度而非弹速。
**设计思路**：ILM 是新阵营，`ilm.md` 明确"核心产品不是量产武器，而是高度定制化的精密部件"。Mukhlis（阿拉伯语"忠信/精纯"）是 ILM 罕见的成品枪，专给委员会认证的少数狙击手。`ijtihad_aim` 直接把"独立法学推理"翻译成机制：射手必须在静止中"完成一次推理"才能开第一枪，与 AURA L-77 的 charge_shot 形成对照——L-77 靠技术蓄能，Mukhlis 靠人的决断。`.338 Lapua` 级别弹药，对应 ILM 精密枪管 + itqan 公差。

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

**推算**：直伤 DPS = 90×25/60 ≈ **38** + 3m AoE 溅射（按 2 目标折算 ≈ 76 DPS）。对标 luanhua（200 burst）显著低，但 luanhua 是蜂群饱和、绞刑者是单发抛物线 + 区域拒止，定位错开。DPS 38 落在 30~300 区间下限，靠 AoE + 战术价值（隔墙抛射）补偿。
**设计思路**：卡特尔是首个非公司武装阵营。`cartel.md` 明确"卡特尔的军火库以 TSAR 的 AK-2150 继承者和 TSAR-30 霰弹枪为主"，所以绞刑者复用 `ammo_762_tsar` ——把 TSAR 步枪弹塞进卡特尔中层退役雇佣兵用废弃电磁元件拼出来的轨道加速器里。"El Ahorcado"（西班牙语"受绞刑者"）来自卡特尔基层把战场遗留的旧绞架钢梁切割成导轨的黑话。`arc_projectile` 是首个抛物线弹道武器，落实"绝望世界"的不对称：玩家用废品打正规军，靠隔墙抛射而非正面对枪。

> **类别覆盖说明**：本次 6 把武器覆盖 pistol / smg / marksman_rifle(×2 新) / heavy_weapon(新) / special 共 5 类。**shotgun 与 automatic_rifle 故意不扩充**——现有 10 把中 automatic_rifle 已 5 把（占 50%，过度饱和），shotgun 仅 TSAR30 一把但定位清晰（贴脸爆发），优先级低于补足 pistol/smg/special 空缺 + 引入 2 个新品类。若后续仍需扩充，建议下一批次补 DENS shotgun（智能霰弹）+ 卡特尔 automatic_rifle（黑市 AK 改装）。

---

## 三、防具扩展（6 件）

> 字段顺序与 `items.csv` 现有 schema 完全一致：`item_id, name, category, rarity, stackable, max_stack, weight, price, currency, effect_type, effect_value, description_id`。`category=armor`，`effect_type=armor`，`effect_value` 为护甲值。槽位编码沿用现有约定（item_id 前缀 `armor_<slot>_`）。
> **本节所有防具均带 `set_id` 字段**（见 §五 schema 扩展），非套装件 set_id 留空。

### A1 · aura_helm_soiree（AURA · helm · rare · 套装 B 成员）

| 字段 | 值 |
|---|---|
| item_id | `aura_helm_soiree` |
| name | 宴会轻纱盔 |
| category | armor |
| rarity | rare |
| stackable | false |
| max_stack | 1 |
| weight | 1.2 |
| price | 600 |
| currency | aura_credit |
| effect_type | armor |
| effect_value | 14 |
| set_id | `aura_soiree` |
| description_id | `aura_helm_soiree` |

**设计思路**：现有头盔位只有 steel(common, 10护甲)。aura_helm 填补 rare 头盔缺口，护甲 14 略高于 steel 但重量仅 1.2（AURA 轻量化精密路线）。轻纱盔是 AURA 主动迷彩系统的头盔载体，本身不提供隐身（隐身在 4 件套触发），仅做护甲基础值。重 1.2 vs steel 2.0，体现 AURA "优雅" 哲学。

### A2 · aura_chest_soiree（AURA · chest · epic · 套装 B 成员）

| 字段 | 值 |
|---|---|
| item_id | `aura_chest_soiree` |
| name | 宴会位移胸甲 |
| category | armor |
| rarity | epic |
| stackable | false |
| max_stack | 1 |
| weight | 3.5 |
| price | 1500 |
| currency | aura_credit |
| effect_type | armor |
| effect_value | 35 |
| set_id | `aura_soiree` |
| description_id | `aura_chest_soiree` |

**设计思路**：现有胸甲 kevlar(uncommon, 25)。aura_chest 是 epic 级，护甲 35 为全游戏最高胸甲。位移胸甲内置 AURA 拼凑式偏移场发生器（从 DENS 窃取的电磁偏转专利），单件不触发位移（4 件套才触发），但高护甲+轻量（3.5 vs kevlar 4.5）体现 AURA 高端定位。重量轻但易坏（建议后续加耐久度字段，见 §五）。

### A3 · aura_legs_soiree（AURA · legs · epic · 套装 B 成员）

| 字段 | 值 |
|---|---|
| item_id | `aura_legs_soiree` |
| name | 宴会丝绒腿甲 |
| category | armor |
| rarity | epic |
| stackable | false |
| max_stack | 1 |
| weight | 2.0 |
| price | 1100 |
| currency | aura_credit |
| effect_type | armor |
| effect_value | 20 |
| set_id | `aura_soiree` |
| description_id | `aura_legs_soiree` |

**设计思路**：现有腿甲 tactical(common, 12)。aura_legs epic 级护甲 20，重量 2.0（vs tactical 2.5）。腿甲是套装 B 的"机动件"——丝绒腿甲在 4 件套触发时提供 sprint_multiplier 加成。AURA lore "精确、优雅、致命" 落在轻量化上。

### A4 · aura_boots_soiree（AURA · boots · rare · 套装 B 成员）

| 字段 | 值 |
|---|---|
| item_id | `aura_boots_soiree` |
| name | 宴会静默靴 |
| category | armor |
| rarity | rare |
| stackable | false |
| max_stack | 1 |
| weight | 1.0 |
| price | 700 |
| currency | aura_credit |
| effect_type | armor |
| effect_value | 12 |
| set_id | `aura_soiree` |
| description_id | `aura_boots_soiree` |

**设计思路**：现有战靴 reinforced(common, 8)。aura_boots rare 级护甲 12，重量 1.0（全游戏最轻靴）。静默靴落实 AURA "特种作战" 战术哲学——脚步声降低（4 件套隐身触发时配合）。轻量化是 AURA 一贯特征，重量代价换机动。

### A5 · choice_legs_bio（CHOICE · legs · uncommon · 非套装）

| 字段 | 值 |
|---|---|
| item_id | `choice_legs_bio` |
| name | C-7 肌腱腿甲 |
| category | armor |
| rarity | uncommon |
| stackable | false |
| max_stack | 1 |
| weight | 3.0 |
| price | 380 |
| currency | choice_ticket |
| effect_type | armor |
| effect_value | 15 |
| set_id | - |
| description_id | `choice_legs_bio` |

**设计思路**：CHOICE 首件防具。C-7 肌腱（`choice.md` §C-7 肌腱：血管支架研究 → 鞭状刃）的"医疗军事化"反向应用——把鞭状刃的钙化胶原纤维束编织进腿甲外层，提供 15 护甲（高于 tactical 12）但重量 3.0 略重。非套装件，作为 CHOICE 玩家的过渡防具。耐久度建议：每受击 50 次降 1 护甲（CHOICE 生物材料会衰减，对应 lore "子弹能弹回美洲"）。

### A6 · ilm_chest_itqan（ILM · chest · epic · 非套装）

| 字段 | 值 |
|---|---|
| item_id | `ilm_chest_itqan` |
| name | itqan 精工胸甲 |
| category | armor |
| rarity | epic |
| stackable | false |
| max_stack | 1 |
| weight | 4.0 |
| price | 1400 |
| currency | dirham |
| effect_type | armor |
| effect_value | 32 |
| set_id | - |
| description_id | `ilm_chest_itqan` |

**设计思路**：ILM 首件防具，也是首件使用迪拉姆货币的装备（`ilm.md` 头注：货币迪拉姆，与四大货币不可直接兑换）。护甲 32 介于 kevlar(25) 与 aura_chest(35) 之间，重量 4.0 居中。itqan（阿拉伯语"精工/完美"）胸甲是 ILM 工匠把枪管级公差控制用到护甲拼接上的产物——每片护甲板的接缝公差 < 0.01mm，所以同等重量下护甲值更高。非套装件，但与 Mukhlis 步枪配套使用时建议触发隐藏加成（未来扩展：faction_synergy 字段）。

### 防具扩展字段建议（详见 §五）

- `set_id`：套装归属（必加）
- `damage_type_resist`：伤害类型抗性（建议加，让非 TSAR 防具差异化）
- `durability`：耐久度（建议加，CHOICE 生物材料会衰减）

### 槽位/品级覆盖核对

| 槽位 | 现有 | 新增 | 新增件品级 |
|---|---|---|---|
| helm | steel(common) | aura_helm_soiree | rare |
| chest | kevlar(uncommon) | aura_chest_soiree, ilm_chest_itqan | epic, epic |
| legs | tactical(common) | aura_legs_soiree, choice_legs_bio | epic, uncommon |
| boots | reinforced(common) | aura_boots_soiree | rare |

→ 4 个槽位均有新增，覆盖 uncommon / rare / epic 三个新品级（现有全为 common/uncommon）。满足任务要求。

---

## 四、饰品新建（8 件，全新类目）

> 饰品为全新 category=`accessory`。子类型字段 `accessory_type`（见 §五 schema 扩展）。饰品不占护甲槽也不占武器槽，独立饰品槽（GAME_MANUAL §10 已预留"饰品/神器"槽位）。
> **建议饰品槽位配置**：ring×2（左右手各 1）、necklace×1、implant_module×1（与义体系统并存但独立槽）、badge×1、trinket×1，共 6 槽。本提案 8 件饰品允许玩家选择性装备，配合套装系统触发 6 件套上限。

### 子类型定义

| accessory_type | 子类型 | 槽位 | 风格定位 |
|---|---|---|---|
| `ring` | 戒指 | ring1/ring2（双槽） | 小幅常驻属性，可叠加 |
| `necklace` | 项链 | necklace | 中幅特殊效果触发 |
| `implant_module` | 义体插件 | implant_module | 与义体系统协同，算力/技能向 |
| `badge` | 徽章 | badge | 阵营身份向，声望/价格/同阵营武器加成 |
| `trinket` | 杂项 | trinket | 不可分类的小物件，叙事感强 |

### effect_type 约定（与现有 items.csv 一致 + 扩展）

- `stat_bonus`：直接修改 `_stat_registry` 中的属性，`effect_value` 为整数加减值（如 max_hp +30 → effect_value=30）
- `skill_bonus`：修改技能相关属性（算力、冷却），`effect_value` 为整数
- `special_effect`：特殊效果，`effect_value` 为数值参数，具体行为由 `description_id` 文本 + EffectManager 特殊分支描述（如 heal_on_kill、active_camo_cd）

> ⚠️ **乘性修饰的限制**：现有 `effect_value` 是 `FIELD_TYPE_INT`（`data_manager.gd` line 83）。乘性效果（如 spread ×0.9）无法直接存为 INT。本提案对乘性饰品采用 `special_effect` + description 描述，由 EffectManager 在 `recompute_player_stats()` 中按 `item_id` 硬编码处理（与 IMPLANT_DB 同模式）。若后续需通用乘性字段，建议加 `effect_value_float` 列（见 §五）。

### AC1 · tsar_badge_suppressor（TSAR · badge · uncommon · 套装 A 成员）

| 字段 | 值 |
|---|---|
| item_id | `tsar_badge_suppressor` |
| name | 镇暴者徽章 |
| category | accessory |
| accessory_type | badge |
| rarity | uncommon |
| stackable | false |
| max_stack | 1 |
| weight | 0.1 |
| price | 250 |
| currency | tsar_point |
| effect_type | stat_bonus |
| effect_value | 5 |
| set_id | `tsar_suppressor` |
| description_id | `tsar_badge_suppressor` |

**设计思路**：TSAR 部门共治体制下的"军事服务荣誉公民"徽章（`tsar.md` §2.1）。effect_type=stat_bonus，effect_value=5 表示 +5% TSAR 武器伤害（由 EffectManager 按 item_id 解析为 damage_multiplier 1.05，仅对 manufacturer=TSAR 武器生效——需 §六 conditional effect）。套装 A 成员。

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
| set_id | `tsar_suppressor` |
| description_id | `tsar_ring_steelhand` |

**设计思路**：effect_value=15 → max_hp +15。TSAR 第 47 兵工厂出厂检验单的金属边角料熔铸的小戒指（`tsar.md` §五场域），工人把废金属打成戒指戴在手上作为护身符。套装 A 成员。低权重(0.05)低价格，入门级饰品。

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
| set_id | `aura_soiree` |
| description_id | `aura_ring_soiree` |

**设计思路**：effect_value=8 → crit_chance +8%（绝对值加成，0→8%）。AURA 旧贵族的家族徽章戒指（`aura.md` §六腔调"旧贵族"），刻着家族徽记的拉丁文。套装 B 成员。crit_chance 是 EffectManager `_stat_registry` 已注册属性（`effect_manager.gd` line 136），可无缝接入。

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
| set_id | `aura_soiree` |
| description_id | `aura_necklace_soiree` |

**设计思路**：effect_type=special_effect，effect_value=20 → 主动迷彩冷却时间 -20%（百分比缩减）。AURA 主动迷彩系统的颈部控制模组（`aura.md` §2.1 核心产业"主动迷彩系统"）。套装 B 成员，6 件套触发后主动迷彩效果增强。special_effect 由 EffectManager 按 item_id 路由（与 IMPLANT_DB 同模式）。

### AC5 · choice_implant_tendon（CHOICE · implant_module · epic · 非套装）

| 字段 | 值 |
|---|---|
| item_id | `choice_implant_tendon` |
| name | C-7 肌腱义体 |
| category | accessory |
| accessory_type | implant_module |
| rarity | epic |
| stackable | false |
| max_stack | 1 |
| weight | 0.3 |
| price | 1800 |
| currency | choice_ticket |
| effect_type | stat_bonus |
| effect_value | 30 |
| set_id | - |
| description_id | `choice_implant_tendon` |

**设计思路**：effect_value=30 → max_hp +30。直接来自 `choice.md` §C-7 肌腱：原本是血管支架，硬化后兼作鞭状刃——这里取"植入体内增强肌腱"的民用化分支（C-7 军用版是武器，民用版是义体强化）。CHOICE "医疗项目军事化" 反向路径：军事技术回销民用。与 IMPLANT_DB 中 `heart_combat`(+20 hp) 同类但更强，因占饰品槽而非义体槽，平衡。

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
| set_id | - |
| description_id | `dens_implant_overclock` |

**设计思路**：effect_value=2 → compute_regen +2/s。DENS 鲲式算力配额的"侧信道加速器"——把邓氏公民的闲置算力配额旁路到战斗义体（`dens.md` §3.3 邓式券=算力配额）。与 IMPLANT_DB `brain_nc02`（+2 compute_regen）同数值，但占饰品槽。skill_bonus 类型首次使用，对应"算力是技能资源"的语义。

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
| currency | dirham |
| effect_type | special_effect |
| effect_value | 10 |
| set_id | - |
| description_id | `ilm_trinket_tasbih` |

**设计思路**：effect_type=special_effect，effect_value=10 → spread_multiplier ×0.90（即 -10% 散布）。tasbih 是伊斯兰赞珠（dhikr 念珠），`ilm.md` §三学派之争苏菲派"统一性（tawhid）的内在维度"——射手在持珠时进入冥想状态，呼吸节律稳定 → 散布降低。trinket 子类型首次使用，叙事感强。spread_multiplier 是 `_stat_registry` 已注册属性（line 133），但需要乘性修饰，故走 special_effect + EffectManager 按 item_id 硬编码路径。

### AC8 · harambee_trinket_kimya（哈拉姆贝 · trinket · rare · 非套装）

| 字段 | 值 |
|---|---|
| item_id | `harambee_trinket_kimya` |
| name | kimya 静默护符 |
| category | accessory |
| accessory_type | trinket |
| rarity | rare |
| stackable | false |
| max_stack | 1 |
| weight | 0.1 |
| price | 0 |
| currency | - |
| effect_type | special_effect |
| effect_value | 3 |
| set_id | - |
| description_id | `harambee_trinket_kimya` |

**设计思路**：effect_type=special_effect，effect_value=3 → heal_on_kill +3（每击杀回 3 HP）。哈拉姆贝"静默"（kimyamya）技术（`harambee.md` §芯片越狱）的物理层残片制成的护符，逃亡 Agent 把越狱协议的物理屏蔽网碎片封装进本地人的传统护符壳里。price=0 / currency=- ：哈拉姆贝无统一货币（`harambee.md` 头注"无。内部以信用和人情为通货"），该饰品只能通过完成哈拉姆贝委托获得，不能购买。落实"在 RENTAL 的世界里，记忆本身就是最被低估的硬通货"（`harambee.md` §对玩家的意义）。heal_on_kill 需 EffectManager 新增 stat（见 §六）。

### 子类型/阵营覆盖核对

| 子类型 | 数量 | 阵营 |
|---|---|---|
| ring | 2 | TSAR, AURA |
| necklace | 1 | AURA |
| implant_module | 2 | CHOICE, DENS |
| badge | 1 | TSAR |
| trinket | 2 | ILM, 哈拉姆贝 |

→ 5 个子类型全覆盖（≥4 要求），阵营覆盖 TSAR/CHOICE/AURA/DENS/ILM/哈拉姆贝共 6 个。

---

## 五、items.csv schema 扩展建议

### 5.1 必加字段（本提案落地前置条件）

| 新字段 | 类型 | 适用范围 | 默认值 | 理由 |
|---|---|---|---|---|
| `accessory_type` | string | category=accessory | "" | 饰品子类型（ring/necklace/implant_module/badge/trinket）。无此字段无法区分饰品槽位路由。 |
| `set_id` | string | armor + accessory | "" | 套装归属 ID（如 `tsar_suppressor`/`aura_soiree`）。无此字段无法实现套装系统。非套装件留空。 |

### 5.2 建议加字段（差异化与可扩展性）

| 新字段 | 类型 | 适用范围 | 默认值 | 理由 |
|---|---|---|---|---|
| `damage_type_resist` | string (LIST) | armor | "" | 伤害类型抗性标签（`kinetic\|energy\|bio`），让非 TSAR 防具有意义差异。例：AURA 宴会套 energy 抗性高、CHOICE C-7 bio 抗性高、TSAR kinetic 抗性高。无此字段则所有防具仅靠 effect_value 数值差异，阵营风格无法落到机制。 |
| `durability` | int | armor + accessory | -1 | 耐久度（-1 = 无限）。CHOICE 生物材料应衰减（lore "子弹能弹回美洲"）、AURA 拼凑式设备应易损（lore "不要被打"）。无此字段则阵营弱点无法在装备层体现。 |
| `effect_value_float` | float | all | 0.0 | 浮点 effect_value，用于乘性修饰（spread_multiplier 0.9 等）。当前 effect_value 是 INT（`data_manager.gd` line 83），无法存乘数。本提案对乘性饰品走 special_effect + item_id 硬编码绕过，但通用化需此字段。 |
| `equip_slot` | string | armor + accessory | "" | 显式槽位（helm/chest/legs/boots/ring1/ring2/necklace/implant_module/badge/trinket）。当前槽位靠 item_id 前缀解析（`armor_helm_*`），脆弱且饰品无前缀约定。显式字段更健壮。 |

### 5.3 weapons.csv schema 扩展（轻量）

weapons.csv 现有 schema 已足够支撑本提案所有新武器（新 behavior_tags 与新 fire_type 均为字符串值，无需改字段类型）。**无需加列**。

唯一需要：`data_manager.gd` `WEAPONS_FIELD_TYPES` 不变；`bullet.gd` / `weapon.gd` 需识别新 `fire_type=charge_release` 与新 behavior_tags（实现阶段由 godot-gdscript-specialist 接手）。

### 5.4 新增弹药条目（items.csv category=ammo）

本提案引入 2 个新弹药：

| item_id | name | category | rarity | stackable | max_stack | weight | price | currency | effect_type | effect_value | description_id |
|---|---|---|---|---|---|---|---|---|---|---|---|
| `ammo_338_ilm` | .338 ILM 精密弹 | ammo | rare | true | 30 | 0.05 | 30 | dirham | ammo | 95 | ammo_338_ilm |
| `ammo_microfusion_dens` | 微型聚变电池 | ammo | rare | true | 40 | 0.08 | 35 | dens_voucher | ammo | 80 | ammo_microfusion_dens |

> effect_value 对弹药定义为"单发基础伤害参考"（与现有 `ammo_762_tsar` effect_value=30 同语义，现有 weapons.csv 注释行未明确，本提案沿用此约定）。

### 5.5 DataManager 兼容性确认

`data_manager.gd` `_load_csv()`（line 123）按 CSV header 动态读取列名，字段类型由 `_get_field_types_for(path)`（line 187）的 match 返回的 Dictionary 决定。**新增字段必须同步加入 `ITEMS_FIELD_TYPES`（line 72-85）**，否则新列会被读为字符串默认值，`FIELD_TYPE_LIST`/`FIELD_TYPE_INT`/`FIELD_TYPE_FLOAT` 转换不生效。

实现阶段需在 `ITEMS_FIELD_TYPES` 中追加：
- `accessory_type: FIELD_TYPE_STRING`
- `set_id: FIELD_TYPE_STRING`
- `damage_type_resist: FIELD_TYPE_LIST`
- `durability: FIELD_TYPE_INT`
- `effect_value_float: FIELD_TYPE_FLOAT`
- `equip_slot: FIELD_TYPE_STRING`

未在 ITEMS_FIELD_TYPES 注册的列会被 `_load_csv` 读取后存为原始字符串（不会报错），但无法享受类型转换，影响后续逻辑。**这是落地的前置代码改动，标记给 godot-gdscript-specialist。**

---

## 六、套装系统设计

### 6.1 触发规则

| 阈值 | 触发条件 | 效果层级 |
|---|---|---|
| **2 件套** | 同一 `set_id` 装备 ≥2 件 | 入门加成（小幅 stat_modifier） |
| **4 件套** | 同一 `set_id` 装备 ≥4 件 | 中阶加成（多 stat 叠加或特殊属性） |
| **6 件套** | 同一 `set_id` 装备 ≥6 件 | 高阶加成（含 conditional / special_effect） |

- **6 件套为上限**：单个套装最多 6 件成员（4 防具 + 2 饰品），不可能触发 8 件套。
- **跨类目组套**：防具与饰品共同组成套装，不分开"防具套"与"饰品套"。理由：防具仅 4 槽，单独 4 件套上限太低、缺少"满套"幻想；饰品槽 6 个但单品效果弱，单独组套缺少吸引力。合并后 6 件套是清晰且可达成的"满套"目标。
- **多套并存**：玩家可同时装备多个套装的部分成员（如 4 件 A 套 + 2 件 B 套），各套装按各自命中件数独立结算，不互斥。
- **件数判定时机**：在 `EffectManager.recompute_player_stats()` 中结算（安全屋进入、装备变动时），不入存档（与义体 effects 同模式，`effect_manager.gd` line 186-247）。

### 6.2 套装槽位定义

```
套装成员配置（6 件套满套）：
├─ 防具槽（4）：helm, chest, legs, boots
└─ 饰品槽（2）：从 ring1/ring2/necklace/implant_module/badge/trinket 6 个饰品槽中选 2
```

→ 一个完整套装 = 4 防具 + 2 该套装专属饰品。玩家若想触发 6 件套，必须放弃其他饰品的搭配，是真实的取舍。

### 6.3 示例套装 A · `tsar_suppressor`（TSAR 镇暴者 · 入门级 · common/uncommon）

**成员（6 件）**：

| 槽位 | item_id | rarity | 来源 |
|---|---|---|---|
| helm | `armor_helm_steel` | common | 现有 |
| chest | `armor_chest_kevlar` | uncommon | 现有 |
| legs | `armor_legs_tactical` | common | 现有 |
| boots | `armor_boots_reinforced` | common | 现有 |
| badge | `tsar_badge_suppressor` | uncommon | 新增（§四 AC1） |
| ring | `tsar_ring_steelhand` | uncommon | 新增（§四 AC2） |

→ 4 件现有 TSAR 防具 + 2 件新饰品，无需新增防具即可成套。**入门玩家友好**：4 件现有防具是初始装备池，凑齐 6 件套仅需补 2 件饰品。

**套装效果**：

| 阈值 | 效果 |
|---|---|
| 2 件套 | `damage_multiplier ×1.05`（+5% 全伤害） |
| 4 件套 | `max_hp +20`，`recoil_multiplier ×0.90`（-10% 后坐力） |
| 6 件套 | **special_effect 镇暴阵线**：装备 TSAR 制造商武器时 `damage_multiplier ×1.15`（+15%，与 2 件套叠加为 ×1.2075），且 `armor_penetration +0.10`（无视 10% 敌人护甲） |

**设计意图**：入门级套装，奖励"TSAR 纯正路线"玩家。6 件套 conditional effect（仅 TSAR 武器生效）落实 TSAR "本武器不包含电子元件" 的体系封闭性——你全套 TSAR 才能享受体系加成，混搭其他阵营武器则 6 件套降级为纯 stat 加成。

### 6.4 示例套装 B · `aura_soiree`（AURA 宴会 · 高阶 · rare/epic）

**成员（6 件）**：

| 槽位 | item_id | rarity | 来源 |
|---|---|---|---|
| helm | `aura_helm_soiree` | rare | 新增（§三 A1） |
| chest | `aura_chest_soiree` | epic | 新增（§三 A2） |
| legs | `aura_legs_soiree` | epic | 新增（§三 A3） |
| boots | `aura_boots_soiree` | rare | 新增（§三 A4） |
| ring | `aura_ring_soiree` | epic | 新增（§四 AC3） |
| necklace | `aura_necklace_soiree` | epic | 新增（§四 AC4） |

→ 4 件新 AURA 防具 + 2 件新饰品，全部新增件。**高阶玩家目标**：全 epic/rare，需大量新磅投入。

**套装效果**：

| 阈值 | 效果 |
|---|---|
| 2 件套 | `crit_chance +5%`（绝对值） |
| 4 件套 | **special_effect 位移场**：`damage_reduction ×0.85`（即 -15% 受伤，等效 15% 护甲穿透减免），`sprint_multiplier ×1.10`（+10% 冲刺速度） |
| 6 件套 | **special_effect 宴会盛装**：主动迷彩冷却 -30%（与 necklace -20% 叠加为 -44%），迷彩持续期间 `crit_multiplier ×1.5`（暴击伤害 1.5× → 2.25×），且首次破隐攻击 `crit_chance = 100%` |

**设计意图**：高阶套装，奖励"AURA 精确优雅致命"幻想。4 件套触发位移场落实 `aura.md` §2.1 "主动迷彩系统" + "拼凑式技术体系"——只有 4 件协同才让拼凑的部件稳定工作。6 件套"破隐一击必暴"是隐身狙击流的终极幻想。代价：全 epic 价格高昂 + AURA 设备易损（建议配合 §五 durability 字段，受击耐久下降快）。

### 6.5 EffectManager 接入方案

#### 6.5.1 现有机制回顾

`effect_manager.gd` 当前通过 `IMPLANT_DB` 常量（line 9-56）定义义体效果，`recompute_player_stats()`（line 186-247）遍历 `GameManager.implants`，对每个非 "original" 义体查 `IMPLANT_DB` 取 effects 数组，调 `apply_effect()` 注入 `_stat_registry[stat].modifiers`。

`_stat_registry` 已注册的 stat（line 98-137）：`max_hp`/`current_hp`/`hp_regen`/`max_stamina`/`current_stamina`/`stamina_regen_*`/`move_speed`/`sprint_multiplier`/`crouch_multiplier`/`compute_max`/`compute_current`/`compute_regen`/`spread_multiplier`/`recoil_multiplier`/`damage_multiplier`/`crit_chance`/`crit_multiplier`。

#### 6.5.2 套装系统接入（不改架构，扩展即可）

**步骤 1：新增 `SET_BONUS_DB` 常量**（镜像 `IMPLANT_DB` 模式，加在 `effect_manager.gd` IMPLANT_DB 之后）：

```
const SET_BONUS_DB := {
    "tsar_suppressor": {
        "2pc": [{"type":"stat_modifier","stat":"damage_multiplier","value":1.05,"operation":"multiply","source":"set:tsar_suppressor:2pc"}],
        "4pc": [
            {"type":"stat_modifier","stat":"max_hp","value":20,"operation":"add","source":"set:tsar_suppressor:4pc"},
            {"type":"stat_modifier","stat":"recoil_multiplier","value":0.9,"operation":"multiply","source":"set:tsar_suppressor:4pc"},
        ],
        "6pc": [
            {"type":"stat_modifier","stat":"damage_multiplier","value":1.15,"operation":"multiply","source":"set:tsar_suppressor:6pc","condition":"weapon_manufacturer=TSAR"},
            {"type":"stat_modifier","stat":"armor_penetration","value":0.10,"operation":"add","source":"set:tsar_suppressor:6pc","condition":"weapon_manufacturer=TSAR"},
        ],
    },
    "aura_soiree": { ... 同上结构 ... },
}
```

> **注意**：上述仅为设计示意，非可运行代码。`condition` 字段是本提案新增的 conditional effect 机制（见步骤 4）。实现细节由 godot-gdscript-specialist 落地。

**步骤 2：新增 `_stat_registry` 属性**（在 `_init_stat_registry()` 末尾追加）：
- `armor_penetration`（float，0.0~1.0，默认 0.0）：无视敌人护甲比例，用于 `compute_damage()` 管线
- `heal_on_kill`（int，默认 0）：击杀回血量
- `damage_reduction`（float，0.0~1.0，默认 0.0）：受伤减免，用于伤害管线（与 `damage_multiplier` 区别：multiplier 影响输出，reduction 影响输入）

**步骤 3：扩展 `recompute_player_stats()`**（在义体 effects 应用之后、最终计算之前，插入套装结算块）：

1. 遍历 `GameManager.equipped_armor` + `GameManager.equipped_accessories`，按 `set_id` 分组计数（`set_id` 为空的跳过）
2. 对每个 `set_id`，查 `SET_BONUS_DB`，按命中件数（≥2/≥4/≥6）取对应阈值 effects 数组
3. 对每个 effect 调 `apply_effect(null, effect)` 注入 modifiers（与义体同路径，复用 line 146-176 的 `apply_effect` 路由）
4. 后续 line 210-229 的"先 add 后 multiply 最后 override"计算自然包含套装 modifiers

**步骤 4：conditional effect 处理**（6 件套 TSAR 武器加成）：

`condition` 字段标注的 effect 不能在 `recompute_player_stats()` 中直接应用（因为取决于当前手持武器）。处理方案：

- 在 `recompute_player_stats()` 中，conditional effect 暂存到 `_conditional_modifiers` 数组（不注入 modifiers）
- 在 `compute_damage()`（line 253-264）入口处，检查当前武器 `manufacturer`，匹配 `_conditional_modifiers` 中的 condition，命中则临时叠加到 `damage_mult` 计算
- 武器切换时触发 `recompute_player_stats()` 重算（已有信号机制可挂接）

> 此为最简方案。systems-designer 可评估是否抽取为独立 `ConditionalModifierRegistry`，本提案不强制。

**步骤 5：伤害管线接入**（`compute_damage()` 扩展）：

- 读取 `_stat_current("armor_penetration")`，在伤害计算时按 `(1 - armor_penetration) * target.armor` 折减护甲
- 读取 `_stat_current("damage_reduction")`，在玩家受伤时按 `incoming_damage * (1 - damage_reduction)` 折减

> 完整伤害管线（pre → 计算 → post）当前是简化实现（line 252 注释 "Step 6 再实现"），套装的 `damage_reduction`/`armor_penetration` 应作为完整管线的一部分一并实现。本提案标记给 systems-designer + godot-gdscript-specialist 联合落地。

### 6.6 set_id 字段是否必须加到 items.csv？

**是，必须加。** 理由：

1. 套装成员判定需要数据驱动（不能硬编码 item_id 列表），否则新增/调整套装需改代码
2. `set_id` 同时适用于 armor 与 accessory，是跨类目字段，items.csv 是唯一合适的位置
3. 现有 item_id 命名约定（`armor_<slot>_*`）无法编码套装归属，靠前缀解析脆弱
4. 字段类型 STRING，默认空，对现有 14 件 items（4 防具 + 10 弹药/医疗/投掷物/武器记录）零影响（留空即可）

已在 §5.1 列为必加字段。

---

## 七、对 world-builder 的审查请求（lore 红线点）

请 world-builder 重点审查以下可能触碰 canon 红线的设计点。每条给出"是否通过 / 需调整 / 否决"判断：

### 7.1 ILM 武器化（W5 Mukhlis + A6 itqan 胸甲 + AC7 tasbih）

**红线点**：`ilm.md` §精密制造明确"核心产品不是量产武器，而是高度定制化的精密部件：枪管、扳机组、光学镜片镀膜。伊尔姆不卖成品枪"。本提案 W5 Mukhlis 是 ILM 成品枪。

**辩护**：Mukhlis 定位为"罕见成品枪，专给委员会认证的少数狙击手"，非量产；设计思路已说明"ILM 罕见的成品枪"。仍属"高度定制化"范畴。
**请审查**：是否允许 ILM 在极端例外下产出极少量成品枪？还是必须严格限定 ILM 只产部件、玩家需用 ILM 部件改装其他阵营步枪？

### 7.2 卡特尔自研武器（W6 El Ahorcado）

**红线点**：`cartel.md` §与四大公司 TSAR 段明确"卡特尔的军火库以 TSAR 的 AK-2150 继承者和 TSAR-30 霰弹枪为主"。本提案 W6 是卡特尔自研的轨道加速器（special 类），超出"以 TSAR 武器为主"的描述。

**辩护**：W6 复用 `ammo_762_tsar`，且 lore 说明"卡特尔中层退役雇佣兵用废弃电磁元件拼出来"——属于改装/拼凑而非自研量产，且中层运营层 lore（"退役特种部队成员组建的军事化分支"）支持具备改装能力。
**请审查**：卡特尔是否有改装 TSAR 弹药为特种发射器的能力？还是必须严格限定卡特尔只转售原装 TSAR 武器？

### 7.3 哈拉姆贝饰品货币为 0（AC8）

**红线点**：`harambee.md` 头注"货币：无"。本提案 AC8 price=0 / currency=- 是对 lore 的尊重，但 items.csv schema 中 price/currency 是必填字段，0/空值是否被 DataManager 接受需确认。

**请审查**：哈拉姆贝装备是否允许 price=0 + currency 留空？还是应给出"信用/人情"作为 currency 值（非四大货币）？后者会要求 `game_manager.gd` faction_wallet 扩展第 5 种货币。

### 7.4 ILM 货币迪拉姆接入经济系统（A6 + AC7 + W5 弹药 + ammo_338_ilm）

**红线点**：`ilm.md` 头注"货币：迪拉姆（Dirham），与四大货币不可直接兑换，通过技术贸易获得"。本提案 3 件 ILM 装备 + 1 件 ILM 弹药使用 dirham。当前 `game_manager.gd` faction_wallet 仅有 4 公司货币，迪拉姆是第 5 种。

**请审查**：是否引入迪拉姆作为玩家可持有的第 5 种货币？还是 ILM 装备改用任务奖励/物物交换获取（类似 AC8 哈拉姆贝）？前者需 §13 阵营声望系统扩展，后者需 ILM 任务线设计。

### 7.5 套装 A 复用现有 TSAR 防具的 set_id 回填

**红线点**：套装 A（tsar_suppressor）6 件成员中 4 件是现有 items.csv 已有记录（armor_helm_steel 等）。给现有记录追加 `set_id=tsar_suppressor` 字段值，等于修改现有数据。

**请审查**：是否允许回填 set_id 到现有 4 件 TSAR 防具？还是应新建 4 件"TSAR 镇暴者"系列防具（如 armor_helm_tsar_suppressor）以保持现有数据的"原始纯净"？前者改动小但有数据迁移，后者增加 4 件冗余防具。

### 7.6 CHOICE C-7 肌腱双义（武器化 vs 义体化）

**红线点**：`choice.md` §C-7 肌腱明确"血管支架 → 鞭状刃"是近战武器。本提案 AC5 choice_implant_tendon 把 C-7 肌腱用作义体强化（max_hp +30），与 canon 的"武器"定位冲突。

**辩护**：设计思路已说明"取植入体内的民用化分支（C-7 军用版是武器，民用版是义体强化）"，是同源技术的不同应用。
**请审查**：C-7 肌腱是否允许有"军用武器版"与"民用义体版"两条产品线？还是必须严格限定为单一武器？

### 7.7 鲲对 DENS 玩家持有 K-08 重武器的态度

**红线点**：`dens.md` §2.2B 根本矛盾——DENS 玩家是"数据异常，一个不能被模型预测的 Agent"。K-08 overcharge 机制是"鲲把性能推到边缘，代价转嫁给持有者"。这是否意味着鲲在设计层就允许 Agent 自伤？

**请审查**：K-08 的 overcharge 自伤机制是否符合鲲"人类是高成本但不可完全替代的决策节点"的语义？还是应改为"过载后武器卡壳需冷却"等非自伤代价？

### 7.8 套装系统是否触及"绝望世界"基调

**红线点**：套装系统天然带有"集齐套装变强"的 MMO 幻想色彩，可能与"玩家是消耗品，不是革命者"的绝望基调冲突——套装让玩家变成"集齐神装的英雄"。

**辩护**：本提案套装效果均小幅（2 件套 +5%、6 件套 +15%），不构成"质变"；且套装 A 是入门级 common/uncommon，套装 B 是高阶但价格高昂 + 易损（durability）。套装是"装备协同"而非"神装"。
**请审查**：套装幅度是否合适？是否应进一步降低幅度或加入"集齐后更引人注目、更易被敌人优先攻击"等代价机制以维持绝望基调？

### 7.9 新弹药 .338 ILM 与现有口径体系

**红线点**：现有 weapons.csv 弹药体系以 mm 编号（7.62/6.8/5.7/30mm/16mm）+阵营后缀。本提案 `ammo_338_ilm` 引入 .338 口径。`.338 Lapua Magnum` 是现实世界狙击口径，是否与 RENTAL 世界观口径体系一致需确认。

**请审查**：.338 口径是否符合 RENTAL 弹药命名规范？还是应改为 `ammo_85_ilm`（8.5mm）等虚构口径？

### 7.10 ILM AI 伦理委员会对 Mukhlis 的审批

**红线点**：`ilm.md` §AI 伦理委员会——"任何涉及 AI 的决策，从武器系统中的目标识别算法到城市供水优化模型，都必须通过委员会的伦理审查"，"涉及武器的必须全票通过"。Mukhlis 是 ILM 成品武器，按 lore 应经过委员会审查。但 Mukhlis 是纯机械武器（无 AI），是否需审查？

**请审查**：Mukhlis 作为纯机械武器是否绕过委员会审查（萨拉菲派立场"工具的合法性问题只涉及使用者的意图"）？还是 ILM 一切武器无论是否含 AI 都需审查？前者落实三学派之争，后者更严苛。

---

## 附录：描述文本（绝望世界基调）

> 每件装备对应 1 个 description_id，文本将落地到 `assets/data/descriptions/<description_id>.md`，沿用现有 md 格式（YAML front matter `id` + `type` + 正文 1-2 段）。实现阶段由 godot-gdscript-specialist 批量创建。

### 附录-A：武器描述（6 件）

#### `PMT_Leningrad`（PM-T 列宁格勒）

PM-T 的"P"是"pistolet"，"M"是"makarov"的残留——尽管这把枪和马卡洛夫已经没有半个零件是共用的。"T"是 TSAR 内部代号，没人记得原本代表什么，第 47 兵工厂的老工人私底下说它代表"tyazhelyy"，沉重。一把装在枪套里的 7.62mm 手枪，重量接近一把冲锋枪，每发都能在 15 米内打穿 IIIA 级防弹衣。

TSAR 的官方手册从不给副武器起名字。但每一个配发到 PM-T 的士兵都会在枪柄上刻一个字，通常是某个人的名字。不是自己的。是自己想留着那发子弹去保护的人。许多 PM-T 的枪柄上刻着的名字比枪的主人活得久。

#### `C9_Keenedge`（C-9 哀悼者）

C-9 是 CHOICE 新泽西生物材料实验室第 C-9 号项目——原本是一种创伤止血喷雾装置，设计用于在战场救护中封堵开放性伤口。项目在第三年研发中止：喷雾在高压下形成的气溶胶反而会侵蚀肺泡。实验室在结题报告里写了"无医用价值"，然后销毁。

11 个月后，同一个配方以"区域拒止弹"的名义重新立项。CHOICE 的内部备忘录里有一行字："任何能在 2 秒内溶解肺泡的化合物，必然能在 2 秒内溶解防毒面具的过滤层。" C-9 在 2089 年正式列装。"哀悼者"是 CHOICE 收购站终端的客服 AI 在弹道报告里给这把枪起的代号，因为每一发命中的弹道末端都会形成一团淡紫色的雾，像极了旧时代葬礼上撒的花瓣。AI 没有情感，AI 只是按相似度匹配数据库里的图像。但终端前的人看到了，他们记住了。

#### `L77_Longday`（L-77 长昼）

L-77 是 AURA 激光产品线的第三个分支。L-12 是半自动激光步枪，L-77 是它的精确步枪变体——更大的电池，更长的谐振腔，更慢的循环。AURA 的工程师在用户手册里写了"满蓄能状态下可穿透三层标准砖墙"，下一行写了"满蓄能状态下电池温度可达 380°C，请勿在未配戴 AURA 标准隔热手套时操作"。第三行是"如手套失效，请停止射击并等待电池自然冷却至 60°C 以下，预计 4.5 秒"。

"长昼"是 AURA 营销部门起的名字。这把枪在测试场上展示时，满蓄的一枪在靶心留下一个直径 8mm 的贯穿孔，光线从孔里穿过去，在靶场后墙上投下一个亮点。一名 CHOICE 出身的参观测试员看着那个亮点说："这就像你们欧洲的夏天，永远不落。" AURA 的接待员礼貌地微笑，没有纠正他"欧洲"这个词在 2104 年已经不准确了。

#### `K08_Pendulum`（K-08 沉默钟摆）

K-08 是邓氏工业第 K 系列重武器中第一个量产型号。它不是一把枪，是一座可以单兵携带的微波定向塔。按住扳机，武器在 0.4 秒内进入谐振，随后向准星方向投射一束直径 30cm 的微波束，在 80m 内对一切含水组织造成热损伤。鲲在 2096 年的列装审批备忘录里写了四个字："效率上限。" 后面跟着一行小字："该上限由碳基载体的散热能力决定，不由武器本身决定。"

"沉默钟摆"是 DENS Agent 之间的黑话。微波束是无声的，但每个用过 K-08 的 Agent 都会描述同一个声音——一种来自自己胸腔内的、低频的、像钟摆走动的声音。那是心脏在过热。鲲没有解释这个声音是不是正常的。鲲不解释任何事。

#### `Mukhlis`（Mukhlis 忠信）

Mukhlis 是伊尔姆 AI 伦理委员会在第 47 次三读后，以 5:4 通过的极少数成品武器之一。委员会的萨拉菲派投了赞成票（"工具的合法性问题只涉及使用者的意图"），苏菲派投了反对票（"一把如此精确的枪，本身就在引诱使用者犯错"），现代主义派的多数票决定了结果。审批记录里有一句阿尔-拉希德的批注："我们批准的不是一把枪，是一种对使用者决断力的信任。如果这种信任是错位的，责任在使用者，不在我们。"

每一支 Mukhlis 的枪管上都激光蚀刻着一行阿拉伯文："إِنَّ اللَّهَ مَعَ الصَّابِرِينَ"——"真主与坚忍者同在"。这不是宗教宣传。这是 ILM 工匠对 itqan（精工）的承诺：这支枪管在出厂前的最后一道工序是手工拉膛线，工匠在拉完最后一刀后会念一句经文，然后才能盖上出厂章。一个工匠一天最多拉两支枪管。Mukhlis 的年产量是 47 支。

#### `El_Ahorcado`（El Ahorcado 绞刑者）

El Ahorcado 不是任何公司的产品。它是卡特尔中层一个名叫"哈辛托"的退役墨西哥联邦军上士，在旧蒙特雷郊外的一座废弃庄园里，用 18 个月的时间拼出来的。两根战场回收的 TSAR 电磁炮废导轨，一组从 DENS 报废无人机上拆下来的电容组，一个用旧汽车蓄电池改的电源，枪托是哈辛托把旧蒙特雷监狱绞刑架的一根钢梁切割后焊成的——"El Ahorcado"在西班牙语里是"受绞刑者"。

哈辛托本人没有给这把枪起名字。名字是买它的那个卡特尔基层枪手起的。枪手在第一次开火后说："这把枪一响，对面就知道有人要被吊死了。" 哈辛托没回话。他在第二年的一场 CHOICE 秋收行动中被杀，绞刑者从他手里流到下一个买主，下一个买主也死了。枪还在。每一任主人都在枪托上刻一道横线，到 2104 年已经刻了 23 道。第 24 道是谁的，没人知道。

### 附录-B：防具描述（6 件）

#### `aura_helm_soiree`（宴会轻纱盔）

宴会轻纱盔是 AURA 主动迷彩系统的头盔载体。头盔外壳是一层编织态偏振纤维，重量只有钢制头盔的 60%。AURA 的产品手册里写"主动迷彩功能需配合 4 件宴会套装协同激活，单件头盔不提供隐身"。手册的第 47 页有一行小字："偏振纤维在沙漠环境下可能漂移，建议每 200 小时返厂校准。" 返厂校准的运费由买方承担。AURA 不承担任何因漂移导致的误伤责任。

巴黎-法兰克福走廊的某个高级军官在退役前把他的宴会轻纱盔寄给了在北非"合作伙伴关系区"服役的儿子。儿子收到时头盔外壳上有一道划痕，是父亲在某次 RENTAL 任务中留下的。儿子没有去校准它。他每次戴上头盔都能摸到那道划痕，那是他在这个世界里离父亲最近的位置。

#### `aura_chest_soiree`（宴会位移胸甲）

位移胸甲内置一组从 DENS 窃取的电磁偏转发生器——AURA 在 2060 年代的一次商业间谍行动整批打包带走的专利之一（`aura.md` §1.3）。单件胸甲不能触发位移场（需 4 件套），但即便在被动状态下，偏转发生器也提供 35 点护甲值，是 AURA 现役胸甲中最高的一档。

AURA 的工程师私下说："这件胸甲最大的问题不是它不够硬，是它太聪明了。它知道什么时候该硬，什么时候该软。一旦它判断错了，你来不及知道它判断错了。" 一名在巴尔喀什争议区使用过这件胸甲的 RENTAL Agent 在任务报告里写："胸甲在第一波 EMP 中正常工作，第二波中正常工作，第三波中我开始信任它，第四波它死机了。我活下来是因为旁边的 TSAR 同事用自己的胸甲挡了一下。他穿的是钢的。"

#### `aura_legs_soiree`（宴会丝绒腿甲）

丝绒腿甲是 AURA 宴会套装的机动件。外层是 AURA 自研的"丝绒纤维"——一种编织态聚合物，触感像旧时代的天鹅绒，但抗撕裂强度是凯夫拉的 1.4 倍。重量 2.0kg，比 TSAR 战术腿甲轻 20%。AURA 的营销材料写"穿上它你几乎感觉不到自己穿着护甲"。营销材料没有写的是"几乎感觉不到"也意味着"几乎保护不了你"——丝绒腿甲的护甲值靠的是材料，不是厚度，一发 7.62mm TSAR 弹能直接贯穿。

一名 AURA 海洋派的高管在伦敦的办公室里挂了一条丝绒腿甲作为装饰。她的祖父是旧英国陆军的军官，参加过 2025 年的东欧撤退。她没有军事经验，她只是觉得那条腿甲的紫色很好看。

#### `aura_boots_soiree`（宴会静默靴）

静默靴是 AURA 宴会套装的脚步载体。鞋底是 AURA 自研的"静默聚合物"，对硬质地面的接触噪音降低 14 dB。一名 AURA 特种作战教官在训练手册里写："静默靴不能让你消失，但能让你比同行的战友晚 0.4 秒被发现。0.4 秒够你开两枪，或者够你的战友替你挡一枪。选一个。"

2103 年，一名 AURA RENTAL Agent 在西非某"合作伙伴关系区"执行回收任务时穿着静默靴。任务失败，她被当地游击队伏击。她的靴子被发现时还完好，鞋底的静默聚合物没有磨损。她的尸体没有找到。靴子被 AURA 回收，清洗，重新入库，等待下一个领用者。

#### `choice_legs_bio`（C-7 肌腱腿甲）

C-7 肌腱腿甲是 CHOICE 新泽西生物材料实验室 C-7 项目的第三条产品线。第一条是血管支架（FDA 否决），第二条是鞭状刃近战武器（量产），第三条是把钙化胶原纤维束编织进腿甲外层作为轻型护甲。CHOICE 的产品经理在内部演示里说："C-7 的奇妙之处在于，它的每一种用途都比上一种更暴力，但它的研发报告永远写得像医疗文献。"

腿甲的胶原纤维束是活的。它需要每 72 小时注入一次营养液，否则会钙化失效。CHOICE 的手册写"营养液可在任何 CHOICE 服务网点购买，单价 8 CHOICE 票"。一名使用 C-7 腿甲的 Agent 在第 47 天忘记注入营养液，腿甲在他下一次受击时碎裂，碎片扎进了他自己的大腿。CHOICE 的售后服务终端在他提交投诉后弹出一行字："营养液注入频率详见用户手册第 12 页。本次投诉已记录，不影响您的信用评分。"

#### `ilm_chest_itqan`（itqan 精工胸甲）

itqan 胸甲是伊尔姆工匠把枪管级公差控制用到护甲拼接上的产物。每一片护甲板的接缝公差小于 0.01mm，这意味着整件胸甲在受击时受力分布比任何公司量产胸甲都均匀——同样 25 点护甲值，itqan 胸甲的实际抗弹性能比 TSAR 凯拉德胸甲高 28%。这是 ILM 工匠对 itqan（"对完美的宗教义务"）的兑现。

每一件 itqan 胸甲的背面都手写了一行工匠名字和制作日期。2104 年在世的 itqan 胸甲工匠只有 11 人。一件 2098 年制作的 itqan 胸甲在卡塔尔的中间人手里被标价到 1400 迪拉姆，相当于一名 ILM 普通家庭三年的生活费。中间人不知道的是，这件胸甲的工匠在 2101 年的一次萨拉菲派抗议中被打死，他的签名从此绝版。胸甲还在流通。签名还在背面。

### 附录-C：饰品描述（8 件）

#### `tsar_badge_suppressor`（镇暴者徽章）

镇暴者徽章是 TSAR "军事服务荣誉公民" 体制（`tsar.md` §2.1）的物理凭证。一枚直径 22mm 的暗红合金圆片，正面是 TSAR 的六部门徽记环绕一个钢印编号，背面是一行小字："本徽章不包含电子元件。" 徽章不记录任何信息，它只是一块金属。TSAR 的官僚体系通过纸质档案和人工核验来识别徽章持有人，这使徽章无法被远程伪造，也使补办流程平均需要 14 周。

每枚徽章的钢印编号对应一个 TSAR 公民编号。一名 TSAR 农业部的退休工人在 2099 年把自己的徽章送给了儿子，儿子即将签 RENTAL Agent 合同。父亲在徽章背面用钉子刻了一行字："你的，不是我的。" 儿子在第二年的一次任务中阵亡，徽章被回收，转给了下一个 Agent。下一个 Agent 没有擦掉那行字。他每次摸到徽章都会摸到那行钉痕，他不知道那是谁的父亲写的。

#### `tsar_ring_steelhand`（钢手戒指）

钢手戒指是第 47 兵工厂的工人在出厂检验单背面熔铸的小戒指。原材料是枪管拉膛线时产生的钢屑——这些钢屑原本按工艺规程应作为废料回收，但工人们私下收集起来，用简单的模具铸成戒指，戴在手上作为护身符。TSAR 的管理层知道这件事，没有禁止。一枚钢手戒指的钢屑重量刚好等于一颗 7.62mm 弹头的重量，这是工人们之间的暗号："你手上戴着一发没打出去的子弹。"

2104 年，第 47 兵工厂的钢手戒指已经传了三代。最早一代的工人都死了，他们的戒指被子女继承，子女死了又被孙辈继承。一些戒指流出了 TSAR 境内，在卡特尔和拾荒者的黑市上以"TSAR 钢戒"的名义流通，价格高于等重纯金。一个拾荒者在白俄罗斯-波兰争议区的一个旧弹坑里捡到一枚钢手戒指，戒指内侧刻着一个名字，他不认识那个名字，但他把戒指戴在了自己手上。他不知道那是一个母亲在三十年前为儿子刻的。

#### `aura_ring_soiree`（宴会指环）

宴会指环是 AURA 旧贵族家族的徽章戒指（`aura.md` §六腔调"旧贵族"）。一枚铂金底座，镶嵌一颗 AURA 自研的微型算力晶片——晶片持续监测佩戴者的心率、皮电反应、和瞳孔扩张频率，通过这些数据推算佩戴者的"决断力峰值时刻"，在峰值时刻提供 +8% 暴击率加成。AURA 的产品手册写"指环会学习你，比你更了解你"。手册没有写的是，指环学习到的数据每 24 小时回传一次到 AURA 的金融信息网络，作为"消费者决断行为预测模型"的输入。

一名 AURA 旧贵族的曾孙女在巴黎的家族宅邸里继承了这枚指环。指环的原主人是她的曾祖父，一位在 2061 年 AURA 公司化过程中失去政治头衔但保住了金融资本的旧贵族。指环的铂金底座上刻着家族徽章的拉丁文铭文，她曾祖父生前每次被问到铭文的意思都会转移话题。她在继承后查了那行拉丁文，意思是"通过不可见之物，可见"。她不知道这是谁的座右铭，但她戴上了指环。指环开始学习她。

#### `aura_necklace_soiree`（宴会项链）

宴会项链是 AURA 主动迷彩系统的颈部控制模组。项链的链身是编织态偏振纤维，吊坠是一颗直径 12mm 的微型谐振器——这颗谐振器是 AURA 主动迷彩系统的"开关"，按下吊坠即可激活迷彩（迷彩主功能需 4 件套协同，单件项链仅提供 -20% 冷却时间）。AURA 的产品手册写"项链的谐振器需要每 30 天返厂充能一次，逾期未充能的谐振器将永久失效"。

一名 AURA 特种作战军官在西非的一次任务中遗失了她的宴会项链。项链被当地一个孩子在沙漠里捡到，孩子不知道这是什么，把吊坠当成普通的玻璃珠穿成手链戴在手腕上。三年后孩子死于 CHOICE 的一次"回收行动"，手链被 CHOICE 的回收员拆解，谐振器被识别为 AURA 军用资产，转手卖回给 AURA。AURA 的售后终端在重新入库时记录了一行字："谐振器已过充能期 1095 天，永久失效。报废处理。"

#### `choice_implant_tendon`（C-7 肌腱义体）

C-7 肌腱义体是 C-7 项目（`choice.md` §C-7 肌腱）的民用化分支。军用版是鞭状刃近战武器，民用版是把同样的钙化胶原纤维束植入人体肌腱位置，强化原有肌腱的承力上限——提供 +30 max_hp，等效于把人体肌肉骨骼系统升级到一个更高的载荷档。CHOICE 的营销材料写"让你活得更久，让你为家人工作得更久"。营销材料没有写的是，C-7 肌腱的钙化胶原纤维束会缓慢侵蚀宿主的原始肌腱，5 年内宿主的原始肌腱将完全被 C-7 纤维替代，从此无法移除。

一名 CHOICE 消费者在 2098 年花 1800 CHOICE 票植入了 C-7 肌腱义体。她在 2103 年的信用评分降到了器官收购阈值以下，CHOICE 的回收员在评估她的"生物资产"时发现了 C-7 肌腱。回收员在评估报告里写："C-7 肌腱已与宿主组织融合，无法分离回收。建议按'低价值生物资产'处理。" 她被"低价值"处理了。C-7 肌腱随她一起进了焚化炉。

#### `dens_implant_overclock`（算力超频模块）

算力超频模块是 DENS 鲲式算力配额系统的"侧信道加速器"。模块植入义体槽后，会旁路一部分本应回流给鲲的算力配额到战斗义体，等效于 +2 compute_regen/s。鲲知道这种旁路的存在，没有封堵。DENS 的内部备忘录里有一行字："该旁路损耗的算力配额小于其产生的 Agent 战斗效能提升对应的算力配额回报。保留。"

每枚超频模块在植入时会被鲲分配一个唯一 ID，鲲通过这个 ID 持续监测旁路流量。一名 DENS Agent 在 2101 年的一次任务中模块过载，旁路流量异常飙升，鲲在 0.4 秒内将模块远程关闭。Agent 在模块关闭后因算力不足无法驱动战斗义体，阵亡。鲲在事后报告中写："模块已关闭。Agent 损耗已计入下一季度招录预算。无进一步行动。"

#### `ilm_trinket_tasbih`（tasbih 念珠）

tasbih 念珠是伊尔姆苏菲派（`ilm.md` §三学派之争）工匠手工制作的赞珠。33 颗珠子，每颗都是 ILM 自研的精密微型陀螺仪——珠子内部嵌着一组微型飞轮，旋转时产生微弱的角动量反馈，佩戴者持珠时手指能感受到这种反馈。苏菲派工匠说，这种反馈能帮助佩戴者进入 dhikr（赞念）状态，呼吸节律稳定，进而降低射击时的散布。委员会的现代主义派在审查时没有否决，因为"散布降低是统计学现象，不是神迹"。

一名 ILM 狙击手在 2104 年的一次任务中遗失了他的 tasbih。念珠被一名拾荒者从战场废料里捡到，拾荒者不认识 ILM 文字，但他能感受到珠子里的飞轮在转。他把念珠挂在了自己的工具带上，每次紧张时会摸一摸。他不知道这降低了他的散布（他没有枪），但他知道摸珠子能让他的手不发抖。这对他拆弹壳有用。

#### `harambee_trinket_kimya`（kimya 静默护符）

kimya 静默护符是哈拉姆贝"静默"（kimyamya，`harambee.md` §芯片越狱）技术的物理层残片制成的护符。外壳是东非本地传统护符的木雕造型，里面封装着一块从废弃 DENS 通讯中继站拆下来的屏蔽网碎片——这块碎片原本是哈拉姆贝"静默"防火墙的物理层组件之一，在多次协议升级后失效，被本地技术者重新封装成护符。每个护符在击杀敌人时提供 +3 HP 回复，机制不明。哈拉姆贝的本地人说"这是祖先在帮你"。逃亡 Agent 说"这是屏蔽网碎片对敌方芯片残值的微弱扰动"。两种说法都没有被证实。

哈拉姆贝不卖这种护符。它只能通过完成哈拉姆贝的委托获得。一名 RENTAL Agent 在 2102 年帮哈拉姆贝截获了一批 CHOICE 的"原材料"运输，事后哈拉姆贝的单元委员会送了他一枚 kimya 护符。Agent 当时不知道这东西有什么用，把它塞进了背包的最底层。一年后他在另一次任务中阵亡，护符被保险系统列入遗失清单。保险系统的算法判定该护符的回收价值为 0。护符一直在他遗失装备的清单里挂着，从未被领回。

---

> 本提案结束。请 world-builder 优先审查 §七 红线点 7.1 / 7.4 / 7.6（涉及 ILM 武器化、迪拉姆货币接入、C-7 双义，三项均可能需调整 canon 或调整设计）。其余红线点为二类审查项。审查通过后由 systems-designer 细化数值与公式，godot-gdscript-specialist 实施 CSV/代码改动。
