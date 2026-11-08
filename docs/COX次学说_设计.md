# COX 次学说设计：工农红军 / 袭扰破坏 / 人民战争（赤卫队版）

> 2026-11-08 定稿
> 挂靠：TNO「非常规战争」`unconventional_warfare`
> 脚本：
> - `common/doctrines/subdoctrines/land/LFB_COX_subdoctrines.txt` —— 两条新增次学说（工农红军 / 袭扰破坏）
> - `common/doctrines/subdoctrines/land/operations_army_subdoctrines.txt` —— **整文件覆盖** TNO 同名文件，只为人民战争追加赤卫队加成
> 本地化：`localisation/simp_chinese/LFB_doctrines_l_simp_chinese.yml`、`localisation/english/LFB_doctrines_l_english.yml`

TNO 的次学说 = `track` + 基础效果 + **5 个 `rewards` 节点**（按学说掌握度解锁）。
节点里的 `<子单位 id> = { ... }` 只对该兵种生效，所以可以直接写 `red_Guards` / `red_Guards_anti_tank`。

---

## 一、陆军核心：工农红军（`COX_red_army`，track = infantry，50 陆军经验，COX 专属）

**设计原则（2026-11-08 重做）**：只补短板、不补长板——不加士气、组织度、缴获率（这三项赤卫队已经够高）；补的是**堑壕、经验损失、补给消耗、战斗宽度**，外加一张进攻战术卡和科研卡。

基础：步兵武器研发加成 ×1（`infantry_weapons`）；**赤卫队 组织度 +15**（抵消「非常规战争」大战略 `category_army` 的 −15，净组织度保持 70）。

| # | 节点 | 效果 |
|---|---|---|
| 1 | 乌鲁木齐改编 | 赤卫队 补给消耗 −10%；补充率 +2% |
| 2 | 政务军官 | 赤卫队 经验损失 −15%；协同 +5% |
| 3 | 官兵平等 | 赤卫队 堑壕 +2；堑壕速度 +20% |
| 4 | 知己知彼 | 科研卡：步兵武器 ×0.5、反坦克科技 `infantry_at2` ×0.5 |
| 5 | 凤凰涅槃 | 解锁进攻战术「人海战术」`tactic_human_wave_tactics`；己方战场宽度 +10%；陆军经验获取 +10% |

**战术卡为什么选「人海战术」**：TNO 的 `tactic_human_wave_tactics`（人民军队进攻战术）是攻击型、`active = no`、TNO 没有任何次学说启用它；效果为攻方伤害 +10%、防守方 +10%（血战）、**战场宽度 +0.5**，正好与第 5 节点的战场宽度叠加。触发条件 `frontage_full + has_reserves`——赤卫队便宜、铺满战线且有预备队时常用。备选是 `tactic_infantry_charge`（+10%/−5%，触发无门槛，但 TNO 轻步兵主导已用它）。

## 二、火力支援：袭扰破坏（`COX_harassment_sabotage`，track = combat_support，50 陆军经验，COX 专属）

**设计原则（2026-11-08 重做）**：补强破坏小组的**硬攻与穿甲**，保留**堑壕/堑壕速度**，新增**对空生存**（`cas_damage_reduction`，不是防空攻击）与**防御战术卡**，并补**火炮/支援科技卡**。组织度、士气对支援连无意义，全部移除。

基础：步兵武器研发加成 ×1（`infantry_weapons`，已涵盖反坦克科技）；**破坏小组 缴获 −2%**（抵消大战略给所有 `category_army` 单位的 +2%，支援连不参与装备回收）。

| # | 节点 | 效果 |
|---|---|---|
| 1 | 要道伏击 | 破坏小组 **硬攻 +15%**；火炮科技卡 ×0.5 |
| 2 | 诱敌深入 | 破坏小组 堑壕 +1；**对空生存 `cas_damage_reduction = 0.10`** |
| 3 | 地道作战 | 破坏小组 堑壕 +1；**堑壕速度 +20%**；解锁防御战术 **`tactic_delay`（迟滞战术）** |
| 4 | 小规模袭扰 | 破坏小组 **穿甲 +15%**；支援科技卡：侦察 ×0.5、工兵 ×0.5 |
| 5 | 密集夜袭 | 破坏小组 **硬攻 +15%**、突破 +10%；火炮科技卡 ×0.5；协同 +5% |

**累计**：硬攻 +30%、穿甲 +15%、堑壕 +2、堑壕速度 +20%、对空减伤 10%、火炮卡 1.0、侦察卡 0.5、工兵卡 0.5、防御战术 1 张、缴获净 0%（−2% 抵消大战略 +2%）。

**为什么防御战术选「迟滞战术」而不是「游击战术」**：`tactic_guerrilla_tactics` 已经由 TNO 的「人民战争」基础效果提供（`guerrilla_operational_doctrine` 里 `enable_tactic = tactic_guerrilla_tactics`），再给一次是重复；`tactic_delay` 无技能门槛（`is_attacker = no` 即可触发）、敌方移动 −25%/敌方伤害 −25%、我方 −15%，正好对应"破坏交通、迟滞敌军"的定位。

**对空生存为什么用 `cas_damage_reduction`**：这是陆军类别修饰符，直接降低被 CAS（近距空中支援）命中的伤害，与"防空攻击"（`air_attack`）无关；TNO 的「人民战争」基础效果本身就是 `cas_damage_reduction = 0.3`，两者可叠加（都点满 = 40%）。

## 三、人民战争：整文件覆盖 + 追加赤卫队（只有一张卡）

**做法**：把 TNO 的 `common/doctrines/subdoctrines/land/operations_army_subdoctrines.txt` **整文件复制**到 mod 的同路径，只改一处——`guerrilla_operational_doctrine` 的 `attrition_of_the_will` 节点里，在原来的 `light_infantry` 之后**追加** `red_Guards`：

```txt
attrition_of_the_will = {
    dig_in_speed_factor = 0.2
    org_loss_at_low_org_factor = 0.2
    light_infantry = {           # 原版保留
        default_morale = 0.1
        equipment_capture_factor = 0.02
    }
    red_Guards = {               # 新增：赤卫队
        default_morale = 0.1
        equipment_capture_factor = 0.005
    }
}
```

结果：军事学说轨道里**只有一张「人民战争」**（名字、图标、其余 4 个节点与 TNO 完全一致），COX 点满「消磨意志」时额外拿到赤卫队 士气 +0.1 / 缴获 +0.5%；其他国家照常吃轻步兵加成。

**为什么不能用"同 id 覆盖"**：实测发现 HOI4 的 `common/doctrines` 对同 id 定义是**追加**而不是覆盖——两个定义会同时生效，界面上出现**两张同名卡**（2026-11-08 实测：两张「人民战争」并存）。所以改写某条学说只能用同路径整文件覆盖（DUPL 规则）。

**技术债**：`operations_army_subdoctrines.txt` 现在是 TNO 该文件（2026-11-07 版）的快照 + 3 行表头注释 + 4 行赤卫队块。TNO 以后更新这个文件时，需要重新同步（把新版本复制过来，再把那 4 行重新加进去）。文件头注释里已写明这一点。

## 四、技术要点

1. **挂靠现成 grand doctrine**：`available = { has_doctrine = unconventional_warfare }`，不另起 grand doctrine。
2. **定向自定义兵种**：节点头里直接写子单位 id（`red_Guards = { ... }`），和 TNO 写 `light_infantry` 同一机制。
3. **COX 专属**：`available` 里加 `tag = COX`（只用在新增的两条次学说上；人民战争按用户要求保持全世界可用）。
4. **节点本地化 key** = `<次学说id>_<节点id>`，描述再加 `_desc`；次学说名用 `SUBDOCTRINE_<ID>` / `_DESC`。
5. 同一 `track` 只能选一条次学说；「工农红军」与 TNO 的「轻步兵主导」互斥，「袭扰破坏」与「反坦克/高炮火力」互斥。
6. **学说没有"按国家隐藏"**：只有 `available`，UI 永远列出同 track 的所有次学说、不满足条件的显示灰卡打叉（TNO 自身「治安战」「卓越单兵」即如此）。

## 五、待进游戏验证

- [ ] 「工农红军」「袭扰破坏」出现在对应轨道，且只有 COX 能看到。
- [ ] 军事学说轨道里「人民战争」**只有一张卡**（这次的关键修复点）。
- [ ] 点满「消磨意志」后，赤卫队确实多出 士气 +0.1 / 缴获 +0.5%。
- [ ] `error.log` 无新增 `Unknown modifier` / `duplicate` 报错。

## 七、介绍文本（2026-11-08 实装）

文本定稿在工作区 `次要学说_重写版.md`（用户在 AI 重写稿基础上改过），已落进 `localisation/simp_chinese/LFB_doctrines_l_simp_chinese.yml`。

节点名与现有 script id 的对应（**id 没改，避免牵动英文 loc；数值重做时再统一改名**，顺序即 `rewards` 顺序）：

| 顺序 | 工农红军（新名） | script id | 当前效果（2026-11-08 已按"补短板"重做） |
|---|---|---|---|
| 1 | 乌鲁木齐改编 | `COX_red_army_sanwan_reorganization` | 赤卫队 补给消耗 −10%；补充率 +2% |
| 2 | 政务军官 | `COX_red_army_party_branch_in_company` | 赤卫队 经验损失 −15%；协同 +5% |
| 3 | 官兵平等 | `COX_red_army_officers_and_men_alike` | 赤卫队 堑壕 +2；堑壕速度 +20% |
| 4 | 知己知彼 | `COX_red_army_three_rules_eight_notices` | 科研卡：步兵武器 ×0.5、`infantry_at2` ×0.5 |
| 5 | 凤凰涅槃 | `COX_red_army_spark_starts_prairie_fire` | 战术卡 `tactic_human_wave_tactics`；己方战场宽度 +10%；陆军经验获取 +10% |

| 顺序 | 袭扰破坏（新名） | script id | 当前效果（2026-11-08 已按"补短板"重做） |
|---|---|---|---|
| 1 | 要道伏击 | `COX_harassment_sabotage_railway_sabotage` | 破坏小组 硬攻 +15%；火炮科技卡 ×0.5 |
| 2 | 诱敌深入 | `COX_harassment_sabotage_mine_warfare` | 破坏小组 堑壕 +1；对空减伤 10% |
| 3 | 地道作战 | `COX_harassment_sabotage_sparrow_warfare` | 破坏小组 堑壕 +1；堑壕速度 +20%；战术卡 `tactic_delay` |
| 4 | 小规模袭扰 | `COX_harassment_sabotage_tunnel_warfare` | 破坏小组 穿甲 +15%；侦察/工兵科技卡 ×0.5 |
| 5 | 密集夜袭 | `COX_harassment_sabotage_interdict_communications` | 破坏小组 硬攻 +15%、突破 +10%；火炮科技卡 ×0.5；协同 +5% |

- 英文本地化仍是旧节点名（Sanwan Reorganization 等）；等数值定稿、id 一并改名后再统一重写英文。
- 数值重做时按 `次要学说_重写版.md` 第三节的「数值钩子」表逐节点对齐。
