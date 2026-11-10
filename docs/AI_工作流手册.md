# MOD 制作 / 备份 / 上传 git —— 标准工作流（给新会话的 AI）

> 2026-11-08 建立。**新会话动任何文件之前，先读本文件 + `docs/兼容性待修.md`（历史交接清单）**，再动手。
> 本文件只写"怎么做、别踩什么坑"，具体内容设计写在各功能文档里（见文末索引）。
> 所有命令均为 PowerShell（Windows），路径按本机实际填写。

---

## 0. 一页速查

| 动作 | 要点 |
|---|---|
| 改 | **只改调试副本** `D:\HOI4Mods\LFB_Dev`；永远不要改游戏安装目录 |
| 备份 | 改前先 `Copy-Item` 到 `D:\HOI4Mods\LFB_Dev\_backup\<文件名>.<yyyyMMdd_HHmmss>.bak` |
| 测 | **完全重启 HOI4**（脚本与本地化都不热重载），看 `...\Hearts of Iron IV\logs\error.log` |
| 传（仅用户要求时） | 先比对 dev 与仓库差异 → 只复制本次改过的文件 → `git add -A` → `git commit -F 消息文件` → `git push` → 校验 `HEAD == origin` |
| 禁 | 改游戏目录；把 `descriptor.mod`、`_backup\` 同步进仓库；盲目录镜像（会覆盖掉上游修复）；**未经要求 `fetch`/`pull`/`merge` 远端，或把仓库文件复制进调试副本** |

---

## 1. 环境（本机固定路径）

| 用途 | 路径 |
|---|---|
| 游戏本体 | `D:\SteamLibrary\steamapps\common\Hearts of Iron IV` |
| 玩家目录（日志/存档/崩溃包/mod 挂载） | `%USERPROFILE%\Documents\Paradox Interactive\Hearts of Iron IV` |
| **调试副本（游戏实际加载的那份）** | `D:\HOI4Mods\LFB_Dev`（无 `.git`） |
| 启动器挂载文件 | `...\Hearts of Iron IV\mod\lfb_dev.mod`（显示名「LFB Dev 调试版」） |
| **git 工作副本（唯一要提交的地方）** | `C:\Users\gzzqzz\Documents\GitHub\hoi4-Light-from-Lake-Baikal`，分支 `1.7.0_Compat` |
| 远程 | `https://github.com/3435857627/hoi4-Light-from-Lake-Baikal.git` |
| git 可执行文件（不在 PATH） | `%LOCALAPPDATA%\GitHubDesktop\app-3.6.10\resources\app\git\cmd\git.exe` |
| TNO 本体（工坊） | `D:\SteamLibrary\steamapps\workshop\content\394360\2438003901` |
| TNO 汉化 | `...\394360\2243912940` |
| LFB 工坊版（测试时关闭） | `...\394360\3350747146` |

约定：设一个变量省事——

```powershell
$git="$env:LOCALAPPDATA\GitHubDesktop\app-3.6.10\resources\app\git\cmd\git.exe"
$repo="C:\Users\gzzqzz\Documents\GitHub\hoi4-Light-from-Lake-Baikal"
$dev="D:\HOI4Mods\LFB_Dev"
$game="D:\SteamLibrary\steamapps\common\Hearts of Iron IV"
$log="$env:USERPROFILE\Documents\Paradox Interactive\Hearts of Iron IV\logs"
```

测试时：启动器里勾选「LFB Dev 调试版」，**关闭**工坊版「新秩序：光自贝加尔湖来」，避免两份 LFB 同时加载。

---

## 2. 修改流程

### 2.1 先备份（每个要改的文件）

```powershell
$bak="$dev\_backup"; New-Item -ItemType Directory -Force -Path $bak | Out-Null
$stamp=Get-Date -Format 'yyyyMMdd_HHmmss'
Copy-Item "$dev\common\units\red_Guards.txt" "$bak\red_Guards.txt.$stamp.bak" -Force
```

`_backup\` 只存在于调试副本，**永远不要同步进仓库**。

### 2.2 修改

- 优先用编辑工具做**定点替换**（`edit`），不要整文件重写；整文件重写只用于自己生成的新文件。
- 脚本类文件（`.txt`/`.gfx`/`.gui`）改完立刻做**括号平衡**检查：

```powershell
$t=[System.IO.File]::ReadAllText("$dev\common\units\red_Guards.txt",[System.Text.Encoding]::UTF8)
"open={0} close={1}" -f ([regex]::Matches($t,'\{')).Count,([regex]::Matches($t,'\}')).Count
```

- 本地化 `.yml`：必须 **UTF-8 with BOM**，目录名与首行根键一致（`simp_chinese` → `l_simp_chinese:`），行首用 1~2 个空格缩进；值里不要出现**直双引号**（会截断字符串，用中文引号「“”」）。

### 2.3 静态校验清单（改完、进游戏前）

1. 括号平衡（上）；
2. `.yml` BOM 存在、key 与脚本里的 id 一一对应：

```powershell
$p="$dev\localisation\simp_chinese\LFB_doctrines_l_simp_chinese.yml"
$b=[System.IO.File]::ReadAllBytes($p); "BOM={0}" -f ($b[0] -eq 0xEF)
```

3. 引用的 GFX/图标名在 TNO 或本 mod 的 `interface/*.gfx` 里存在；
4. 新增/修改的脚本 id 不与 TNO 冲突（除非就是有意覆盖，见第 6 节第 1 条）。

### 2.4 进游戏验证

- **完全退出游戏再启动**（没有热重载）。
- 日志目录 `$log`：`error.log`（报错）、`game.log`（事件/国策执行、脚本 `log=` 输出）、`system.log`（音频/资源加载，可反推点了什么 UI）、`setup.log`（加载顺序）。
- 只 grep 自己的文件名/ID，原版与 TNO 的噪声占 95%：

```powershell
Select-String -Path "$log\error.log" -Pattern 'LFB|red_Guard|COX_' -Encoding UTF8 | Select-Object -Last 40
```

- 崩溃时：崩溃包在 `...\Hearts of Iron IV\crashes\<时间戳>\`（`exception.txt`、`meta.yml`、`logs\`、`minidump.dmp`，**注意里面常常没有 `game.log`**，重启前手动复制 `logs` 目录）。

---

## 3. 备份与回退

| 场景 | 做法 |
|---|---|
| 回退单个文件 | `Copy-Item "$dev\_backup\<文件>.<时间戳>.bak" "$dev\<原路径>" -Force` |
| 回退已提交的某文件 | `& $git -C $repo checkout -- <相对路径>` |
| 回退到某次提交 | `& $git -C $repo log --oneline` 找到 hash → `& $git -C $repo revert <hash>`（不要 `reset --hard`，会丢别人改动） |

**调试副本没有 `.git`，git 救不了它**——所有试错前必须先在 `_backup\` 留底。

---

## 4. 同步到 git（最容易翻车的一步）

> **2026-11-10 用户约定：除非用户明确要求，不要随意 `git fetch` / `pull` / `merge` 远端，也不要把仓库里的文件复制进调试副本。** 本节命令只在用户要求同步 / 提交时执行；平时以调试副本为准，不要让仓库内容自己“流”进来。

### 4.1 先看仓库状态，落后就先追上（仅用户要求同步时）

```powershell
& $git -C $repo status -sb          # 形如 "## 1.7.0_Compat...origin/1.7.0_Compat [behind N]"
& $git -C $repo fetch origin
& $git -C $repo merge --ff-only origin/1.7.0_Compat
```

> 该仓库经常“fetch 到新提交但工作区没动”：**用户要求同步时**，看到 `behind N` 就执行上面的 `merge --ff-only` 落地，不要直接 commit，否则会分叉；**没被要求就不要动远端引用**。

### 4.2 比对调试副本与仓库（排除 `.git`、`_backup`、`descriptor.mod`）

```powershell
function RelMap($root){ $m=@{}; Get-ChildItem $root -Recurse -File -Force -ErrorAction SilentlyContinue | ForEach-Object {
  $rel=$_.FullName.Substring($root.Length+1)
  if($rel -notmatch '^\.git\\' -and $rel -notmatch '^_backup\\' -and $rel -ne 'descriptor.mod'){ $m[$rel]=(Get-FileHash $_.FullName -Algorithm SHA256).Hash } }; return $m }
$d=RelMap $dev; $r=RelMap $repo
"仅调试副本:"; @($d.Keys | Where-Object { -not $r.ContainsKey($_) } | Sort-Object)
"仅仓库:";     @($r.Keys | Where-Object { -not $d.ContainsKey($_) } | Sort-Object)
"内容不同:";   @($d.Keys | Where-Object { $r.ContainsKey($_) -and $r[$_] -ne $d[$_] } | Sort-Object)
```

### 4.3 三类差异分别处理

| 类别 | 处理 |
|---|---|
| **仅调试副本**（新增） | 明确是本次新增的文件 → 复制进仓库 |
| **仅仓库**（调试副本里没有） | 确认是**有意删除**（例如废弃的 gfx）→ 仓库里也删；否则说明调试副本缺文件，先补 |
| **两边不同** | **逐个判断谁更新**：上游（别人）刚修过的文件（`events/`、`gfx/leaders/`、图标 `.dds` 等）**不要**用调试副本的旧版覆盖；只复制"这次任务确实改过"的文件 |

复制示例（只列本次改的文件，不要整目录镜像）：

```powershell
$files=@(
 "localisation\simp_chinese\LFB_doctrines_l_simp_chinese.yml",
 "docs\COX次学说_设计.md"
)
foreach($rel in $files){ $s=Join-Path $dev $rel; $t=Join-Path $repo $rel
  $td=Split-Path $t -Parent; if(-not (Test-Path $td)){ New-Item -ItemType Directory -Force -Path $td | Out-Null }
  Copy-Item $s $t -Force }
& $git -C $repo status --porcelain -uall
& $git -C $repo add -A
& $git -C $repo diff --cached --stat
```

### 4.4 提交（用文件传消息，避免中文乱码）

```powershell
$msg=@"
标题：一句话概括（模块 / 目的）

## 模块一
- 改了什么、为什么

## 模块二
- ...

## 技术债
- 例如：整文件覆盖了 TNO 的 xxx，TNO 更新后需重新同步
"@
$mf="$env:TEMP\commit_msg.txt"
[System.IO.File]::WriteAllText($mf,$msg,(New-Object System.Text.UTF8Encoding($false)))
& $git -C $repo commit -F $mf
```

提交信息规则：
- 中文、分模块（`## 赤卫队`、`## 次学说`、`## 修复`、`## 其他`、`## 技术债`）；
- 写"为什么"和"影响范围"，不贴报错全文；
- **技术债必须写**：整文件覆盖 TNO 的文件、TNO 更新后要重新同步、未验证项。

### 4.5 推送并校验

```powershell
$env:GIT_TERMINAL_PROMPT="0"
& $git -C $repo push origin 1.7.0_Compat
& $git -C $repo status -sb
& $git -C $repo rev-parse HEAD origin/1.7.0_Compat   # 两行必须相同
```

### 4.6 反向同步：把仓库里的上游修复带回调试副本（需用户确认）

提交后，仓库里可能有些文件比调试副本新（别人刚提交的头像、图标、事件）。**先向用户说明是哪几个文件、为什么需要**，得到同意后再把它们复制回调试副本，保证“调试副本 = 仓库 + 自己的 `descriptor.mod` + `_backup\`”：

```powershell
$up=@("events\LFB_COX.txt","gfx\interface\counters\divisions_large\red_guards_icon.dds")
foreach($rel in $up){ Copy-Item (Join-Path $repo $rel) (Join-Path $dev $rel) -Force }
```

再跑一次 4.2 的比对，确认"仅调试副本 / 仅仓库 / 内容不同"都为空（`descriptor.mod` 与 `_backup\` 本来就要排除）。

---

## 5. 目录与文件约定

| 路径 | 说明 |
|---|---|
| `common/units/`、`common/technologies/`、`common/national_focus/` | 单位/科技/国策 |
| `common/doctrines/subdoctrines/land/` | 次学说；覆盖 TNO 时用**同路径整文件** |
| `common/unit_tags/00_categories.txt` | 子单位类别并集表（TNO 会覆盖原版，见第 6 节） |
| `common/units/names_divisions/` | 编制名称组（**不要**放 `history/units/`） |
| `localisation/<语言>/LFB_<主题>_l_<语言>.yml` | 本地化，BOM，一个主题一个文件 |
| `docs/` | 设计/交接文档；**新功能先写文档再写脚本** |
| `_backup/` | 只在调试副本，不进仓库 |

---

## 6. 已知坑（先读，能省一晚上）

1. **HOI4 学说同 id 定义是"追加"不是覆盖**：两个文件写同一个次学说 id 会同时生效、界面上出现两张同名卡。改写 TNO 的次学说必须用**同路径整文件覆盖**（DUPL 规则），并在提交信息里记技术债。
2. **编制名称组必须放 `common/units/names_divisions/`**；放 `history/units/names_divisions/` 不生效，脚本 `division_template` 全部创建失败，还会因为 `create_unit` 造出"模板不存在的部队"导致 UI 崩溃。
3. **TNO 整文件覆盖了原版 `common/unit_tags/00_categories.txt`**，缺 1.19 的 `category_anti_tank` 等类别；本 mod 提供"原版 + TNO 并集表"，新增子单位才能用新类别。
4. **`progressbartype` 不接受 `legacy_lazy_load`**（只属于 `spriteType`），写了整个进度条类型不注册。
5. **子单位属性按"装备类型"求和，与 `need` 里的数量无关**；给营加一种装备会把该装备的软攻/防御/突破/穿甲一起带进来（数值调整时要重新算）。
6. **本地化 `.yml` 必须 UTF-8 BOM**；值里不要出现直双引号。
7. **`.mod` 字段是 `picture=` 不是 `thumbnail=`**；`supported_version` 写 `"1.19.*"`。
8. **调试副本的 `descriptor.mod` 名字是「LFB Dev 调试版」**，仅本地用，不要同步进仓库。
9. **不要用 `robocopy /MIR` 从调试副本镜像到仓库**：会把上游修复覆盖回旧版。用 4.2 的哈希比对 + 定向复制。
10. **崩溃诊断**：先看 `crashes\<时间>\exception.txt`（异常码/地址）与 `meta.yml`（mod 列表、启动参数），再看 `logs\error.log` 末尾；`game.log` 往往没被复制进崩溃包，复现前先备份 `logs`。`minidump.dmp` 可解析出“异常流”（异常码、读写地址、出错 RVA）。
11. **不要随意取用 git 仓库内容**（2026-11-10 用户约定）：未经用户明确要求，不执行 `git fetch` / `pull` / `merge`，也不把仓库里的文件复制进调试副本；调试副本只接受用户确认过的改动，需要同步先问。
12. **`_backup\` 文件名可能大小写撞车**：`common/scripted_effects/LFB_XBJ_northwest_crisis.txt` 与 `common/scripted_localisation/LFB_XBJ_Northwest_crisis.txt` 只差大小写，Windows 文件系统不区分大小写，`_backup\<文件名>.<时间戳>.bak` 会互相覆盖。备份时在文件名里加模块后缀（如 `.scripted_effects.` / `.scripted_localisation.`）。

---

## 7. 提交前验收清单

- [ ] 差异清单已逐条确认，**没有误带**上游修复（4.3）
- [ ] 脚本括号平衡；`.yml` BOM 与 key 覆盖通过（2.3）
- [ ] 游戏内验证通过（未验证的要在提交信息里写明）
- [ ] `docs/` 已更新（设计变更、技术债）
- [ ] 提交信息分模块 + 写清技术债
- [ ] `push` 后 `HEAD == origin`
- [ ] 调试副本已反向同步上游修复，与仓库重新一致

---

## 8. 文档索引

| 文档 | 内容 |
|---|---|
| `docs/兼容性待修.md` | 2026-11-05 交接清单（历史 issue、已修/待修） |
| `docs/赤卫队_最终版.md` | 赤卫队/破坏小组定稿数值、脚本、实测截图 |
| `docs/COX次学说_设计.md` | 工农红军 / 袭扰破坏 / 人民战争：节点、效果、名称与 id 对应、技术债 |
| 工作区 `次要学说_重写版.md` | 两条次学说的定稿文本（介绍 + 10 个节点） |

## 9. 当前状态（2026-11-08 17:20）

- 已推送：`4560f58 赤卫队定稿 / COX 专属次学说 / 修复点国策必崩`（本地与 origin 一致）。
- 未提交：`localisation/simp_chinese/LFB_doctrines_l_simp_chinese.yml`（次学说文本更新）、`docs/COX次学说_设计.md`（第七节）、本手册。
- 待办：工农红军 / 袭扰破坏的**数值重做**（「知己知彼」「诱敌深入」「地道作战」三个节点的现行效果与文字不符）；英文本地化；兵模方案（暂定不动）。
- **2026-11-10 约定**：不主动取用仓库 / 远端内容（`fetch` / `pull` / `merge` / 复制仓库文件均需用户要求）。此前一次 `pull --ff`（`551fd97` 修 COX `remove_ideas`）已按用户要求撤回，本地仓库回到 `ae1f500`；origin 上仍有 `551fd97`，未再拉取。
