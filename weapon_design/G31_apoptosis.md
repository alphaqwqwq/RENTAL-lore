# G-31"凋亡" 设计文档

## 基本信息

- **ID**: `G31_apoptosis`
- **名称**: G-31"凋亡"（Apoptosis）
- **分类**: 自动步枪 / 生物腐蚀武器
- **口径/弹药**: 生物腐蚀弹（ammo_bio_round）
- **制造商**: CHOICE（THE CHOICE Inc.）
- **品质**: 3（精良）

## 设计思路

G-31 的每一发弹头都是一个微型生物反应器，弹头内封装的工程化溶酶体在撞击时破裂，释放水解酶混合物，在 0.3 秒内将目标护甲表层的大分子聚合物降解为单体。

CHOICE 国防供应部在武器手册中引用了一个细胞生物学概念为其命名："凋亡"，程序性细胞死亡。不是杀死，是触发目标自身结构的自我终结。

护甲越厚，降解产物越多，理论上可回收。CHOICE 内部有一支团队专门研究如何从战场上回收降解产物，该项目代号为"回收循环"，尚未产品化，但已经在 G-31 的实战数据上运行了三年。

这是 CHOICE 商业逻辑的武器层面映射：将一切转化为资产。护甲的降解产物是资产，被腐蚀的义体是资产，连战场上被遗忘的弹壳都是资产，如果它能被回收的话。

枪身采用 CHOICE 标准紫色（#8C33D9）+ 纯黑有机质混合材质。枪身左侧蚀刻："PROGRAMMED CELL DEATH — INITIATED"（程序性细胞死亡，已启动）。脉动生物管线沿护木延伸至枪口，在开火时发出微弱的生物荧光，是活性溶酶体在弹匣冷藏后的激活反应。

## 行为标签

| 标签 | 含义 | 代码影响 |
|---|---|---|
| `bio_projectile` | 生物腐蚀弹 | bullet.gd `_is_bio_projectile` 缓存 → `_apply_corrosive_mult()` + `_apply_splash_damage()` |
| `splash` | 命中溅射 | bullet.gd 60px 溅射范围 30% 伤害 |
| `bio` | fire_type 标记 | 仅匹配 bio 附件，拒绝 ballistic/energy 附件 |

## 配件接口

### 可用槽位

stock, magazine, muzzle, optic, grip

### 禁用槽位

无

### 必装槽位

magazine

### 默认配件

magazine:G31_std_mag

## 弹药特征

生物腐蚀弹（`ammo_bio_round`）：

- 7.8×42mm 生物弹，弹头内含工程化溶酶体 + 水解酶混合物
- 撞击时玻璃安瓿破裂，溶酶体在 0.3 秒内降解护甲大分子聚合物
- 弹匣内壁涂有抗溶酶体涂层，防止在弹匣内意外破裂
- 对穿戴护甲的目标额外 +40% 伤害（bullet.gd `_apply_corrosive_mult`）
- 命中点 60px 范围溅射 30% 伤害 + 腐蚀 debuff
- 每一发弹头是一个微型生物反应器，弹匣是它们的冷库
- CHOICE 国防供应部在弹药箱侧印："PROGRAMMED CELL DEATH — BIOWEAPONS DIVISION — NJ-12"

## 校准坐标

由校准工具导出至 `data/weapon_config/G31_apoptosis.json`。（当前为占位坐标，待精灵图制作后校准）

## 备注

- corrosive 护甲加成通过 bullet.gd 检查目标 faction_tier ≥ 2 或装备中 helm/chest/legs 来实现
- splash 溅射排除直接被命中的目标（dist > 10px），防止双倍伤害
- 在 CHOICE 武器体系中：C-7（近战·生物材料）+ N-4（手枪·化学DOT）+ G-31（步枪·腐蚀溅射）= 完整的非线性作战体系
- "回收循环"项目为 P2 技能树/打印机系统预埋，可从战场回收腐蚀产物的经济系统
