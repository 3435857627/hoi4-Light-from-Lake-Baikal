# LFB × TNO 1.10.0c / HOI4 1.20 适配审查

> 审查日期：2026-11-22
> 对象：`D:\HOI4Mods\LFB_Dev`（v0.3）vs TNO **1.10.0c**（工坊 2438003901）+ HOI4 **1.20.0**
> 方法：全量同路径文件比对 + 逐文件文本 diff（静态，未进游戏）

## 结论

**需要适配，但只是小补丁，不是重做：**

1. **6 个整文件覆盖的副本要回灌 TNO 1.10.0c 的新增内容**（合计约 45 行），否则这 6 个文件会把这轮 TNO 的改动整段吃掉；
2. **`descriptor.mod` / 启动器 `.mod` 的 `supported_version` 要从 `1.19.*` 改成 `1.20.*`**（TNO 1.10.0c 只认 1.20.*，回滚游戏不可行）；
3. 顺带 3 个静态小问题（见第五节）建议一起修。
4. 历史遗留的**整文件覆盖损失**（bookmarks、文化地图模式等）不属于本轮，但每次 TNO 更新都会扩大，建议排期重做。

## 一、基线

| 项目 | 版本 | 备注 |
|---|---|---|
| HOI4 | 1.20.0「Operation Postern」0c4f | 2026-11-19 发布，本机 11-19 22:32 装好 |
| TNO | 1.10.0c「The Modern Tordesillas」 | 2026-11-20 发布；`supported_version="1.20.*"` |
| LFB | 0.3 | `supported_version="1.19.*"`（需改） |

## 二、结构概览

| 分类 | 数量 |
|---|---|
| 加载中的文件（不含 `_backup` / `docs`） | 1927 |
| LFB 独有文件 | 1815 |
| 与 TNO 同路径 → **整文件覆盖** | **104**（逐字节相同的 0 个，全部是「TNO 旧版 + LFB 改动」） |
| 仅覆盖原版的文件 | 8（`gfx/loadingscreens/load_1..8.dds`） |

结论：覆盖面只占 5.4%，但这是兼容风险的全部来源——TNO 每改一次，被覆盖的文件就要手工回灌一次。

## 三、1.10.0c 本轮必须回灌的 6 个文件

| # | 文件（LFB 行数 / TNO 行数） | TNO 新增 | 影响 |
|---|---|---|---|
| 1 | `history/countries/NCC - North China Political Council.txt`（174/175） | 4 行 | 华北政务委员会开局：人力、党派支持、外交地位、角色 |
| 2 | `events/TNO_Yunnan.txt`（5493/5470） | 3 行 | 云南 3 处角色意识形态 |
| 3 | `common/scripted_guis/TNO_bottom.txt`（557/519） | 13 行 | 墨西哥 PRI 剧情追踪器图标/动画 |
| 4 | `common/scripted_localisation/TNO_peace_popup_scripted_localisation.txt`（4169/3770） | 15 行 | FZL/ADL/GNS/PPR 新 tag 的和平弹窗 flavor |
| 5 | `common/ideologies/00_ideologies.txt`（511/504） | 8 行 | 8 个意识形态颜色值（纯外观） |
| 6 | `localisation/english/TNO_Country_Closing_Screen_l_english.yml`（117/115） | 3 条 | 乌克兰 / 伊比利亚结局文本 |

> `common/scripted_localisation/TNO_SG_Politics.txt`：LFB 是超集（TNO 独有 0 行），**不用动**。

### 3.1 NCC（有 3 处需要作者决策，不要盲抄）

- `add_manpower = 500000`：TNO 第 16 行，LFB 副本缺 → 建议补。
- `fascism_2 = 5`：TNO 第 40 行 `set_popularities` 内，LFB 写的是 `ultranationalism = 5` → 建议按 TNO 改（两个意识形态都仍存在，不报错，只是党派构成不同）。
- `recruit_character = CHI_wang_yintai`：TNO 第 7 行；LFB 是 `NCC_xu_liang` → **需决策**：跟随 TNO 换王揖唐，还是保留 LFB 的徐良。
- `Sphere_Fully_Dependent`：TNO 第 49 行；LFB 是 `Sphere_Imp_Protectorate` → **需决策**：是否 LFB 有意让 NCC 当日本保护国。

### 3.2 云南（`events/TNO_Yunnan.txt`）

3 处 `ideology = ultranationalism_stratocracy_subtype` → `fascism_revolutionary_nationalism_subtype`
（LFB 第 1062 / 2746 / 2913 行；TNO 对应第 1062 / 2723 / 2890 行附近）。
两个 subtype 在 00_ideologies 里都仍有定义，属内容调整（军政→革命民族主义），不是 ID 失效。

### 3.3 TNO_bottom（墨西哥 PRI 追踪器，13 行）

在 LFB 副本中插入 TNO 的这几段（TNO 行号）：

- 355–356：`MEX_PRI_prepare_circle_animation` / `MEX_PRI_show_history_graph` 两个按钮 effect
- 434–436：`MEX_plot_tracker_crt_icon_visible` 触发块
- 482–488：`MEX_plot_tracker_crt_icon_click` 点击 effect
- 519：末尾空行

### 3.4 和平弹窗（15 行）

TNO 新增：`original_tag = FZL` 两处、ADL / GNS / PPR 各一个 `text = { ... }` flavor 块、FZL 一个 flavor 块、`# Colombian Civil War` 注释。插到 LFB 对应 `text = { ... }` 序列里。

### 3.5 意识形态颜色（8 行）

8 个 `color = { R G B }` 值变更（`socialist_1/2`、`communist_1..4` 等），纯外观，跟随 TNO 即可。

### 3.6 结局文本（3 条 key）

`TNO_CountryClosing_epilogue_Ukraine_communist_victory_UCW`、`..._UNRA_Ohloblyn_victory_UCW`、`..._IBW_PPR`，补进 LFB 覆盖的英文文件。

## 四、历史遗留的覆盖损失（非本轮，仅排序参考）

行数按多重集差集统计（含空白行），只用于排优先级：

| TNO 独有行 | 文件 | 后果 |
|---|---|---|
| 2663 | `common/map_modes/TNO_culture_map_mode.txt` | 文化地图模式配色大面积错色 |
| 732 | `common/bookmarks/the_new_order.txt` | 1962 书签 49 国只剩 5 国 |
| 276 | `interface/frontendgamesetupview.gui` | 选国界面元素缺失 |
| 116 | `history/units/QIN_1962.txt` | 青宁开局部队被清空 |
| 107 | `history/units/XIN_1962.txt` | 新疆开局部队缺失 |
| 105 | `common/scripted_effects/TNO_Comintern_scripted_effects.txt` | 共产国际脚本缺 TNO 新增段 |
| 52 | `history/units/TIB_1962.txt` | 西藏开局部队缺失 |
| 45 | `history/countries/CHI - China.txt` | 中国开局初始化缺失 |
| 31 | `common/scripted_effects/state_control_asia_effects.txt` | 州省改名逻辑缺失 |
| 29 | `localisation/english/TNO_Mengjiang_l_english.yml` | MEN 国名/党派键缺失 |

修复方向：以 TNO 当前文件为底、只叠 LFB 改动，而不是「TNO 旧版 + 局部改」整文件覆盖。

## 五、顺带的静态问题

1. **`legacy_lazy_load` 写在 `progressbartype` 里**（7 处，只有 LFB 这么写，TNO/原版只在 `spriteType` 里用）：
   `interface/LFB_COX_landreform_gui.gfx` 97/110/123/136/149；`LFB_IF_GAW.gfx` 105；`LFB_IF_Liberate_Shaanxi.gfx` 36。
   1.20 加载时这 7 处报 `Unexpected token: legacy_lazy_load`，建议删除（该字段在进度条里没有实际作用）。
2. **非法转义**：`localisation/simp_chinese/LFB_IF_events_l_simp_chinese.yml` 第 129 行有 `\“`（反斜杠 + 全角引号，U+201C），加载报 `Illegal break character`。删掉多余反斜杠即可。
3. **缺图**：`gfx/interface/goals/QIN/QIN_15.png`（`GFX_goal_QIN_15_shine`）不存在，加载报贴图初始化失败。属已知国策图标缺失。

> 以上 3 条来自 13:31 一次加载日志（已按要求停止继续进游戏测试），未做运行时验证。

## 六、建议动作顺序

1. 回灌第三节 6 个文件（NCC 的 2 处先定方案）；
2. `descriptor.mod` 与启动器 `lfb_dev.mod` 的 `supported_version` 改 `1.20.*`；
3. 删 7 处 `legacy_lazy_load`、修 129 行转义、补 QIN_15 贴图；
4. 静态校验：`hoi4_lint.py --no-vanilla` 只查语法；含 TNO 的索引校验需要 hoi4skill（本机未安装）；
5. 最后再进游戏看一次 `error.log`，重点查 LFB 文件的新报错。

---

## 七、处理记录（2026-11-22）

按用户要求只处理「两处 TNO 改动的中国内容」+ 版本号，其余不动：

| 文件 | 处理 |
|---|---|
| `history/countries/NCC - North China Political Council.txt` | 已合并 TNO 1.10.0c 的 4 处（王揖唐、`add_manpower = 500000`、`fascism_2 = 5`、`Sphere_Fully_Dependent`），**现与 TNO 逐字节一致**（这个覆盖文件已冗余，将来可删） |
| `events/TNO_Yunnan.txt` | 3 处 `ideology` 改成 `fascism_revolutionary_nationalism_subtype`；LFB 的 `LFB_GAW_status` 游戏规则分支保留 |
| `descriptor.mod`（调试副本 / 启动器 `.mod` / GitHub 仓库） | `supported_version` 全部 `1.19.*` → `1.20.*` |

未处理（按要求保留）：`TNO_bottom` 墨西哥 PRI、和平弹窗 FZL/ADL/GNS/PPR、意识形态颜色、结局文本 3 条，以及 `legacy_lazy_load`、本地化 `\“` 转义、QIN_15 缺图。

GitHub 工作副本已同步上述 3 个文件（未提交）。
