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

基础：步兵武器研发加成 ×1、赤卫队组织度 +5 / 士气 +0.1、侦察连 侦察 +1 / 防御 +5%。

| # | 节点 | 效果 |
|---|---|---|
| 1 | 三湾改编 | 赤卫队 组织度 +5、士气 +0.1；侦察连 防御 +5%、堑壕 +1 |
| 2 | 支部建在连上 | 赤卫队 组织度 +5、防御 +5%；侦察连 侦察 +1；协同 +5% |
| 3 | 官兵一致 | 赤卫队 士气 +0.1、经验损失 −10%；侦察连 突破 +5%；步兵武器研发 ×0.5 |
| 4 | 三大纪律八项注意 | 赤卫队 补给消耗 −10%、缴获 +1%；补充率 +2%；后勤研发 ×0.5 |
| 5 | 星火燎原 | 赤卫队 突破 +15%、组织度 +5；侦察连 突破 +10%、侦察 +1、穿甲 +10%；陆军经验获取 +10% |

## 二、火力支援：袭扰破坏（`COX_harassment_sabotage`，track = combat_support，50 陆军经验，COX 专属）

只强化**赤卫队破坏小组**（`red_Guards_anti_tank`）。基础：步兵武器研发加成 ×1、破坏小组 组织度 +5 / 士气 +0.1。

| # | 节点 | 效果 |
|---|---|---|
| 1 | 破袭战 | 破坏小组 硬攻 +10%、突破 +5% |
| 2 | 地雷战 | 破坏小组 防御 +15%、堑壕 +1 |
| 3 | 麻雀战 | 破坏小组 穿甲 +10%、硬攻 +5%、补给消耗 −5% |
| 4 | 地道战 | 破坏小组 组织度 +5、防御 +10%；堑壕速度 +20% |
| 5 | 交通破击 | 破坏小组 突破 +20%、硬攻 +15%、缴获 +2%；协同 +10% |

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
        equipment_capture_factor = 0.02
    }
}
```

结果：军事学说轨道里**只有一张「人民战争」**（名字、图标、其余 4 个节点与 TNO 完全一致），COX 点满「消磨意志」时额外拿到赤卫队 士气 +0.1 / 缴获 +2%；其他国家照常吃轻步兵加成。

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
- [ ] 点满「消磨意志」后，赤卫队确实多出 士气 +0.1 / 缴获 +2%。
- [ ] `error.log` 无新增 `Unknown modifier` / `duplicate` 报错。
