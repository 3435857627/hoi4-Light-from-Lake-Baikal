# COX 决议组：赤卫队

> 2026-11-08 建立。
> 用途：承载赤卫队 / 破坏小组相关的决议；后续同类内容都加在这里。

## 一、决议组

| 项 | 值 |
|---|---|
| 分类 id | `COX_red_guards` |
| 显示名 | 赤卫队 |
| 图标 | `GFX_decision_category_generic_arms_crate`（TNO 通用"军火箱"分类图标） |
| 归属 | 仅 `tag = COX`（`allowed`） |
| 解锁 | 完成国策「工农红军新生」`COX_A_Red_Army_Reborn` 后可见 |
| 解锁判定 | `OR = { has_country_flag = COX_red_army_reborn, has_completed_focus = COX_A_Red_Army_Reborn }`（flag 由该焦点写入；老存档里已完成过焦点的用 `has_completed_focus` 兜底） |
| 文件 | `common/decisions/categories/LFB_COX_red_guards_catego.txt` |
| 决议文件 | `common/decisions/LFB_COX_red_guards_dec.txt`（顶层 key 必须等于分类 id） |

## 二、决议：给赤卫队派发防空武器

| 项 | 值 |
|---|---|
| id | `COX_issue_aa_weapons` |
| 名称 | 给赤卫队派发防空武器 |
| 花费 | 25 政治点数，只能执行一次（`fire_only_once = yes`） |
| 前置 | 已研发**任意一档**单兵防空装备：`infantry_aa1` / `aa2` / `aa4` / `aa6` / `aa8` 任一（对应 `anti_air_shoulder_equipment_0`～`_4`；TNO 科技树可以跳着研究，所以不能只判原型） |
| 效果 | `set_technology = { red_Guards_aa = 1 }` → 给赤卫队营加 `need = { anti_air_shoulder_equipment = 10 }` |
| 图标 | `GFX_XBJ_Soviet_Military_Support`（复用本 mod 已有的"苏援"图） |

## 三、机制说明（为什么用"隐藏科技"）

HOI4 里**决策/事件的 effect 无法直接修改子单位的装备需求**；能做这件事的是**科技**——科技块里可以写：

```txt
red_Guards = {
    need = {
        anti_air_shoulder_equipment = 10
    }
}
```

研究（或被 `set_technology` 授予）该科技后，已有的师与编制模板立刻多出这份需求；防空能力来自装备自身的 `air_attack`，不需要额外面板加成。

所以实现拆成两件：

1. `common/technologies/LFB_red_guard_technologies.txt` 里新增**隐藏科技** `red_Guards_aa`（`allow = { always = no }`，不出现在科技树，只能由决议授予）；
2. 决议 `COX_issue_aa_weapons` 用 `set_technology` 授予它。

TNO 原版的做法是直接把 `need` 块塞进 `infantry_aa1` 等科技里（`common/technologies/infantry.txt:3397` 起），但那是整文件覆盖（4000+ 行快照）的代价；本方案不碰 TNO 文件。

## 四、需要实测确认

- [ ] 完成「工农红军新生」后，决议面板出现「赤卫队」分类；
- [ ] 未研究任何单兵防空科技时，决议灰显；研究任一档后可点；
- [ ] 点完之后，赤卫队营的需求里出现 `10 × 单兵防空装备`，师面板的对空攻击上升（每营约 +1.5，3 营约 +4.5）；
- [ ] `error.log` 无 `Unknown trigger` / `Invalid` 类新增报错。

## 五、已知风险 / 待定

1. **COX 能否研究到 AA 科技**：TNO 的 `infantry_aa1` 有 `allow = { TNO_KD_Can_Research_Outdated_Tech = yes ... }`，若 COX 因国家条件够不到，这条决议会永久灰着——实测后若够不到，需要放宽前置（例如改为"库存里有单兵防空装备"或由某个焦点直接授予 `infantry_aa1`）。
2. **配发后立刻吃库存**：10 具/营 × 3 营 = 30 具/师，没有产能会缺装（决议描述里已警告）。
3. **图标是复用的**：决议组用了 TNO 通用军火箱图标、决议用了本 mod 的苏援图；要专属图标后期再补。
4. 后续赤卫队相关内容（扩编、训练、夜间战术等）直接往 `COX_red_guards` 分类里加即可。

## 六、2026-11-08：军事改革并入（方案 B）

旧的「军事改革」决议组（10 条决议 + 民兵受训度动态修正 + 3 个变量 + 2 份 OOB）已经**整体并入**本决议组，训练度系统删除。

### 6.1 入口与解锁

- 决议组可见条件改为 **完成国策「检阅革命军队」`COX_Inspect_The_Army`**（旧 flag `COX_red_army_reborn` 作兜底）；
- 焦点 `COX_A_Base_New_Born` 的 `COX_militia_reform_startup` 已删；
- 旧分类 `COX_militia_reform` 已从 `LFB_COX_catego.txt` 删除。

### 6.2 决议表（8 条，全部在 `LFB_COX_red_guards_dec.txt`）

| # | 决议 id | 类型 | 消耗 | 奖励 |
|---|---|---|---|---|
| 1 | `COX_Assign_commanders_and_fighters` | 兑换（可重复，30 天冷却）| 50 指挥点 | 陆军经验 +20 |
| 2 | `COX_Organize_large_scale_military_training` | 兑换（可重复，30 天冷却）| 25 陆军经验 | 学说精通度 +50 |
| 3 | `COX_Establish_equal_relations_between_commanders_and_fighters` | 改革 I（一次）| 20 政治点 | 陆军经验 +25、旧军队理念一级、事件 COX.201 |
| 4 | `COX_Organizational_democratization` | 改革 II（一次）| 20 政治点 + 2000 人力 | 陆军经验 +25、理念下一级、职业度 +2/月、事件 COX.202 |
| 5 | `COX_Popularize_guerrilla_warfare_and_carry_forward_mobile_warfare` | 改革 III（一次）| 20 政治点 | 精通度 +75、理念下一级、事件 COX.203 |
| 6 | `COX_Mobilize_militia_to_strengthen_border_defense` | 混乱清除（可重复至 3 级）| 10 政治点 + 4000 人力 + 400 步枪 | 混乱的边境降一级 |
| 7 | `COX_New_army_new_look` | 收官（一次）| 50 政治点 + 600 步枪 | 见 6.5 |
| 8 | `COX_issue_aa_weapons` | 装备（一次）| 25 政治点 | 每营 10 具单兵防空需求 |

### 6.3 兑换规则

| 兑换 | 比例 | 冷却 |
|---|---|---|
| 指挥点 → 陆军经验 | 50 CP → 20 XP | 30 天 |
| 陆军经验 → 学说精通度 | 25 XP → 50 精通度（**1:2**）| 30 天 |

- 精通度用 `add_mastery = { amount = N folder = land }`（三条轨道同时吃）；
- 扣经验用 `has_army_experience > 25` 判条件 + `army_experience = -25` 扣；
- 取 1:2 而非 1:3 的理由：分支学说单价 50 XP，1:3 会让"存经验换精通度"比直接买学说更划算，违背"优先解锁学说"的设计意图。

### 6.4 混乱边境链（4 级，全部固定值）

| 级 | 理念 | 效果 |
|---|---|---|
| 1 | `chaotic_band` | 征兵 −15%、稳定 −5% |
| 2 | `chaotic_band2` | 征兵 −10%、稳定 −3% |
| 3 | `chaotic_band3` | 征兵 −5%、稳定 −1% |
| 4 | `border_order`（新）| **征兵 +10%、稳定 +5%** |

（第 2、3 级原来的"每周稳定 −0.3%/−0.1%"改为固定 `stability_factor`。）

### 6.5 收官「新军队，新面貌」

一次性奖励：`red_Guards_tech`（赤卫队科技）、把 `chaotic_band3` 换成 `border_order`、**两个赤卫队编制**、**1 个精锐师 + 3 个守备师**、+800 步枪（兜底）、陆军经验 +50、精通度 +100、职业度 +2/月。

编制（`COX_new_army_setup` 脚本效果，用 `division_template` 直接建）：

| 编制 | 编成 |
|---|---|
| 精锐工农红军师 | 6 × 赤卫队营 + 1 × 破坏小组（团属支援槽，不吃支援装备）|
| 工农红军守备师 | 4 × 赤卫队营（无任何支援连）|

刷兵：`random_owned_controlled_state` + `create_unit`（`owner = ROOT`，count 1 + 3）。

### 6.6 删除清单

- `common/dynamic_modifiers/LFB_COX_militia_dynamic_modifiers.txt`（整文件删）
- `history/units/COX_extra_militia.txt`、`history/units/COX_exp_army.txt`
- `LFB_COX_dec.txt` 的旧 10 条决议、`LFB_COX_catego.txt` 的 `COX_militia_reform` 分类
- `LFB_COX_militia_reform.txt` 里的 startup/训练度变量效果（保留三个新脚本效果）
- 焦点 `COX_Militia_Normalized` 的 `set_technology = { red_Guards_tech = 1 }`（科技移到收官决议）

### 6.7 本次待实测

- [ ] 决议组在「检阅革命军队」完成后出现；
- [ ] 兑换 1 得 20 经验、兑换 2 扣 25 经验并给 50 精通度（看学说界面的精通度）；
- [ ] 三条改革决议依次给理念 1 → 12 → 123，并触发 COX.201/202/203；
- [ ] 混乱链 1→2→3 的稳定值显示为 −5%/−3%/−1%；
- [ ] 收官：两个编制出现在编制界面（含团属支援的破坏小组）、刷出 1+3 个师、`border_order` 生效、赤卫队科技解锁；
- [ ] `error.log` 无 `Unknown effect` / `Invalid` 报错（重点看 `division_template` 的 `regimental_support` 与 `create_unit` 的中文模板名）。
