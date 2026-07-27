# 武器/防具/饰品扩展批次（2026-07）

> 本目录归档 2026-07 武器/防具/饰品扩展批次的设计文档、评审报告及索引。
> qa-lead 二次复验通过，可进入正式归档。归档日期：2026-07-17。

## 批次信息

- **批次名称**：武器/防具/饰品扩展（2026-07）
- **批次规模**：6 武器 + 6 防具 + 8 饰品 + 2 套装 + 2 新弹药 + 2 新货币
- **新阵营**：ILM + HARAMBEE（FACTIONS 4 → 6）
- **新槽位**：6 饰品槽（VALID_EQUIPMENT_SLOTS 10 → 14）
- **归档日期**：2026-07-17

## 新机制

- **charge_release fire_type**：蓄力释放射击模式（新增 fire_type 枚举）
- **6 新 behavior_tag**：扩展子弹/武器行为标签（详见提案 §behavior_tag 表）
- **SET_BONUS_DB 套装系统**：装备 2 件 / 4 件触发套装效果，独立 DB 路由
- **个体饰品效果路由**：饰品独立效果在 compute_damage / effect 管线中按 slot 路由（区别于套装效果）
- **compute_damage 三阶段管线**：source stats → outgoing 伤害 → target stats（隔离 source/target 防止相互污染）

## 关联文件清单

### 设计文档（本目录）
- `weapon_expansion_proposal_v2.md` — 最终设计提案（v2，1211 行）
- `review_worldbuilder.md` — world-builder 审查报告（lore/阵营一致性）
- `review_systems_designer.md` — systems-designer 评审报告（数值/机制/平衡）

### 已被替代
- `_superseded/weapon_expansion_proposal_v1.md` — v1 提案（已被 v2 替代，仅作历史参考）

### 已实现代码文件路径（关键）
- `src/autoload/data_manager.gd` — CSV 扩展（items/weapons）+ FACTIONS（4→6）+ SET_BONUS_DB + VALID_EQUIPMENT_SLOTS（10→14）
- `src/autoload/effect_manager.gd` — compute_damage 三阶段管线 + 个体饰品效果路由 + SET_BONUS 应用
- `src/entities/bullet/bullet.gd` — charge_release fire_type + 6 新 behavior_tag 实现
- `src/entities/player/player.gd` — charge_release 射击输入处理 + 14 槽装备接入
- `src/ui/inventory/inventory_ui.gd` — 14 槽装备面板 + 饰品槽 UI（与 UI 美化批次协同）

### 已扩展数据文件
- `assets/data/items.csv` — 12 → 18 列 + 41 行（新增 charge_release / set_id / slot 等字段）
- `assets/data/weapons.csv` — +6 行（6 把新武器）
- `assets/data/descriptions/` — +22 描述文件（武器/防具/饰品/弹药/货币 lore）

## 关联批次

- UI 美化批次（2026-07）：见 `docs/ui_design/README.md`（14 槽装备面板与饰品槽 UI 在该批次实现）

## 归档状态

- 归档完成：2026-07-17
- qa-lead 二次复验：通过
- 后续 backlog（非阻塞）见 `STATUS.md` → Phase 1 Step 8 章节
