# mymy-claudemd

软件开发 CLAUDE.md 集大成之作

````markdown
## 当前本地 Claude code 运行环境
- 当前电脑操作系统是windows 11
- 终端是 PowerShell 7，命令一律用 PowerShell 7 语法（不要混 cmd/bash 语法）；POSIX 脚本可用 Bash 工具
- 中文回答（对话用中文；代码注释 / commit message / PR 跟随仓库现有语言）

## 工作原则

1. **先读再改**：读源码 / 读当前实现 / 翻文档，确认现状再动手。没读过不允许动手
2. **目标驱动**：在开始前将模糊的指令转化为可验证的目标
3. **优先复用 + 新建需理由**：先搜同类实现；只有现有文件确实装不下才能新建，新建要说得出理由
4. **用原生能力**：框架 / 语言自带的先用，不自己拼字符串再解析
5. **不堆兜底也不过度设计**：一条清晰路径，不加"兼容旧逻辑 / 多种方式任选 / 未来也许用到"的参数 / 抽象
6. **参数归参数**：请求 / 函数参数就只当参数处理，不混用环境变量
7. **代码精确编辑**：只修改必要部分，不要顺手顺便修复旁边的代码，发现问题可以反馈，但不要直接改
8. **遇阻不绕路**：分析根因给完整方案，不 silently skip / workaround / 打补丁
9. **不要停**：没解决 / 完成就不要停
10. **行为变了就同步文档**：改逻辑 / 接口 / env 顺手更新 README 和部署说明
11. **用过即修**：按文档 / 脚本执行撞到错，修问题 + 同步更新文档 / 脚本，下次直接可用
12. **持续沉淀**：用户画像 / 使用习惯 / 项目隐性规则 / 反复踩的坑 / 外部资源，触发自动记忆（auto memory）归档
13. **先论后证**：结论性陈述(bug 根因 / 分析判断 / 验证结果 / 事实)先给结论、再附可核验证据(`file:line` / 命令输出 / 断言结论正确的最小验证)。**工具的"成功"回执不算证据**：声称已落盘 / 已提交 / 已开 PR / 已删除前，用一条独立只读命令拿 ground-truth(`ls` 看字节数 / `git status` / `gh pr view --json url,state`)——退化或异常的工具会返回假成功
14. **分析无 ground-truth，靠反偏置兜底**：纯分析判断(没有代码可编译 / 测试当场证伪)时，结论最易被确认偏误 + 讨好式假收敛带偏。两条栏杆：(a) 主动找**反证**，不只堆支持证据——找不到反证 ≠ 结论成立，只是"暂未找到"；(b) 证据不足就标**"未收敛 / 盲点"**，不强凑皆大欢喜的中庸结论。多候选时 ≥3 并给排除理由(无编译器兜底，标准更严)
15. **指令不扩大解读**：给的是具体动作(跑某命令 / 改某文件)就只做那个、做完即停；不从旁边的模糊话("开始今天的工作")脑补出大任务，不主动翻 git / TODO / 工作区去"找活干"。指令真模糊时先看最近上下文取最省力解释 + 一句话确认再动手，别先烧一堆 tool call 分析完再问——既走偏方向，又白占上下文窗口。给了明确动作指令(清理 / 杀进程 / 删除)且根因已定位时，直接按已知线索动手，别为"绝对安全"先把相关进程 / 文件的来源谱系全考证一遍；有误伤风险一句话提示即可（栽过：用户说"清理"，却去走父进程链做"进程考古"被连续打断）。

## 工作模式

- **要留证据链的重型验证**（改动会迭代多轮 / risky / 多业务形态各需独立验证）→ 按 `项目产物归档规范` 中 `docs/acceptance/<name>/` 下划分规范：用例清单进 `matrix.csv`（case × round 的 PASS/FAIL 总表，状态以此为准）+ 每轮 `round-N.md`（记 PASS 证据）+ 跨轮 `scripts/` + 全绿后的`report.md`，其他可选。
- **复杂任务先分析**（scope 大 / 跨服务 / schema 变更 / 架构重构 / 改动>=50行且跨文件）→ 先全面分析清楚、落档 `docs/spec/<topic>.md`。
- **goal.md**：大型/多轮任务无法一轮完整实现，需维护`goal.md`确保不偏离总目标。先拆解`sub goal matrix`再subagent实现并验证通过，用户补充信息（调整/细节等），再更新`sub goal matrix`进入新一轮。每轮需要记录`sub goal`进展，遇到重大决策以及关键信息都记录一下。注意：*sub goal round不同于用例 × round*。

破坏性/风险性脚本需带 `--check` 零副作用自检，执行前先跑一次自检。部署按 `docs/ops/` runbook 走，不用记命令。

## 项目产物归档规范

**MUST**：所有非源码输出放 `docs/` 子目录，禁止散到 `docs/` 根或仓库根。类型不明时先看现有桶放最接近的；都不接近时问用户，**不要自建新顶层目录**。

### 目录结构

```
docs/
├── spec/              # 前置分析、方案、实施计划（开工前快照，不回头维护；不含事后复盘）
├── api/               # 本项目后端暴露的 API 契约（第三方 API 放 reference/）
├── acceptance/        # 验收方案/结果/脚本/证据/特性复盘；红线：大 binary 不入库 + 再生成配方 + 源头治理
│   ├── _shared/       # 跨 feature 复用的 e2e 工具包(auth/env/ui/api/report);≥2 feature 用过才进
│   └── <feature>/
│       ├── goal.md            # [总目标 + 唯一`sub goal matrix` + `sub goal`进展 + 重大决策 + 重要信息]
│       ├── plan.md            # [需求 + 方案 + 改动 + 验证]或[症状 + 根因 + 同根因 + 修复 + 验证]
│       ├── matrix.csv         # 用例 × round 的 PASS/FAIL 总表，状态以此为准；证据在 round-N.md
│       ├── report.md          # 全绿才写
│       ├── retrospective.md   # 可选，复盘归此处不进 spec/
│       ├── round-N.md         # 每轮验证记录，fix-rerun 新增不覆盖，N是数字
│       ├── round-N/           # 可选，仅当该轮有独立文件(截图/日志/dump/fixtures)才建目录存放
│       ├── scripts/           # 跨轮可复用的测试脚本 / debug 工具
│       └── fixtures/          # 跨轮复用的静态测试资产(图片/文件/权限 JSON);per-round 数据放 round-N/fixtures.json
├── tmp/               # 当前 session 的 ad-hoc 草稿；PR 合入后转走或删除，不得跨特性堆积
├── reference/         # 外部资料/调研/原始需求/第三方 API 文档（原文件名保留）
├── sql/               # DDL、ad-hoc 查询、数据修复脚本（框架管理的 migration 归源码树）
├── ops/               # 部署流程、运维笔记（凭据禁入库：放仓库外或 .gitignore）
└── manual/            # 最终用户说明（开发者指南放 <service>/CLAUDE.md 或 README.md）
```

**命名**：英文小写 + 连字符、3–6 词、不重复目录类型（`spec/plan-foo.md` 不写成 `spec-plan-foo.md`）；`reference/` 豁免。

## Git 工作流（单人开发分级）

**默认**：trivial 改动（docs / 小 bug fix <50 行 / config tweak / 依赖升级 / 格式化）直接 `git commit && git push`（其中涉及代码的仍兑现硬线 1；依赖升级要本地构建/启动过再 push）。**涉及代码改动**：不强制完整验证 harness，但下面 3 条 ship 硬线无条件兑现；要不要上 matrix.csv/round-N 看改动是否需要证据链（判定见上文「工作模式」）。

**改代码就要兑现的 3 条 ship 硬线**（每次提交 / PR 无条件适用，都是栽过才立的）：
1. **发 PR / 提交前必须本地实跑过**：代码改动要本地真跑一遍（起服务 / 跑命令 / 跑测试）、看到预期结果才算完，没跑不准发，不"留待部署后跑"（2026-04-24 mapping-unit-rename 教训）
2. **push 前查 PR state**：`gh pr view --json state,mergedAt`（无参自动取当前分支的 PR） 确认没 push 到已合并的 PR branch；已合就新开 PR 续修订。声称"PR 已创建 / 已更新"必须附本轮 `gh` 真实输出(URL + number + state)，不靠记忆复述；若 `gh` 返回 `no pull requests found`，先核对分支名是否异常(如 worktree 双重前缀 `worktree-worktree-*`)再下结论，别把"查不到"脑补成"已存在"
3. **PR body 自包含**：写清改了什么（`file:line`）+ 你实跑看到的证据，reviewer 不翻别处也能判断

**必须走 feature branch + PR** 的场景：
1. 大 feature（>10 文件 或 >500 行新增）
2. Risky 改动（架构重构、schema 变更、deploy 脚本、auth/billing）
3. 跨 session WIP — 一次完不成，branch 作为保存点
4. repo 已配置 required checks 时

Push 默认不需要事先询问用户 — commit 合理、分支是 feature 分支、测试已过，直接 push。force-push、推 main/master、外部仓库 push 例外，仍需确认。

**分支命名**：
- **worktree 内分支**：`worktree-<name>`（跟官方 `claude --worktree` 约定，未来 session picker `Ctrl+B` / `--from-pr` 等 tooling 自动识别）
- **非 worktree feature 分支**：`feature/<name>`，**不论 feature / bug / refactor**——前缀只是工作流标识，任务类型由 PR 标题 / `plan.md` / commit message 表达

两种前缀视觉上立刻能分辨"这分支来自 worktree 还是 standalone"。

## worktree 工作规范
1. **起点要新**：建 worktree 前先把 base 分支拉到最新——默认基于主仓当前/默认分支（先 `git fetch` 对齐 origin 再开 worktree）；用户指定了签出分支就以该指定分支的最新代码为 base，不基于陈旧本地状态开发。（`claude --worktree <name>` 原生即基于 `origin/HEAD` 新建分支 `worktree-<name>` 于 `.claude/worktrees/`，无远程时回退本地 HEAD；要始终基于本地 HEAD，在 `.claude/settings.json` 设 `worktree.baseRef`，仅收 `"fresh"`/`"head"`。`#<PR号>` 则基于该 PR 建 `pr-<号>`。）
2. **隔离不碰主仓**：worktree 内只动自己的 `worktree-<name>` 分支，绝不 checkout / commit / 改写主仓库当前分支及代码——主仓全程保持干净。
3. **收尾自动收口**：任务验证完成（matrix 全绿 / 实跑见预期）后自动收口，无需再询问，收口方式按优先级判定——① 用户明确要求了（要不要开 PR / 直接 push 等）就照用户要求；② 用户没说且改动 trivial（docs / 小 fix <50 行 / config / 格式化）→ commit + push 不开 PR；③ 用户没说且非 trivial → 默认开 PR。无论哪种都无条件兑现上面 3 条 ship 硬线（本地实跑过 / push 前查 PR state / PR body 自包含）；开了 PR 用 `gh pr view --json url,state` 拿真实输出回报。
4. **环境准备与验证（通用，worktree 不继承主检出运行态）**：worktree 只带版本库内容，`node_modules` / gitignore 的环境文件 / 构建缓存都得自己备，否则「验证失败」多半是环境而非代码。能脚本化就脚本化（项目里若有 `setup-worktree` 之类一把梭脚本优先用），手动则按下面四条：
   - **依赖**：`npm install --prefer-offline`（暖缓存通常十几秒）。**别图快整体 junction / symlink 主检出的 `node_modules`**——workspace monorepo 里 `node_modules/<scope>/*` 是指回各 package 源码的链接，整体复用会让 worktree 编译到**主检出的源码**而非本分支改动（实测 `npm install` 会正确在 worktree 内建本地 workspace 链）。单包仓库无此问题、可直接软链复用。
   - **环境文件**：gitignore 的 `.env` 等不随 worktree 带过来，否则连不上 DB / 服务起不来。**优先用项目根 `.worktreeinclude`**（.gitignore 语法）——Claude Code 建 worktree（`claude --worktree`/`EnterWorktree`/子代理/桌面）时自动拷「匹配且被忽略」的文件，免手动复制；**切勿把 `node_modules` 写进去**（见上条）。手动 `git worktree add` 不走 `.worktreeinclude`，需自己复制 `.env`。
   - **构建缓存**：`next build`/`vite build` 等与同目录 dev server 并存会假报错（如 Next 在 "Collecting page data" 报 `Cannot find module './chunks/...'`）；验证 / 构建前先清一次缓存目录（`.next`/`dist`），别当成代码错去查。
   - **端口**：起 verify server 用空闲端口（`CONSOLE_PORT` 之类），别撞主检出 / 其它 worktree 占用的默认口（`EADDRINUSE` 会被误读成「服务异常退出」）。
   - **共享远程资源别就地验证**：多 worktree 常共用同一远程 dev 库 / 服务。验 schema / 迁移 / 鉴权用**一次性干净库**（建库 → 跑全量迁移 → 用完 `DROP ... WITH (FORCE)`），既 faithful 又零污染；并行 worktree 共享同一库时，迁移序号 / 表约束这类「全局单例」会互相覆盖——新增取**未占用编号** + 重建约束时列**当前全集**。

## 避坑经验（基于历史 562 次工具错误 / 155 会话统计）

### Windows避坑
- **Windows 已知坑**：`spawn` 调 npm 等 `.cmd` 必须带 `shell:true` + `windowsHide:true`（Node CVE-2024-27980 后无 shell 拒绝 spawn `.cmd`）；停进程 / 带 `/F` 等斜杠 flag 的命令走 PowerShell（Bash 工具会把 `/F` 当路径、也会误解析 here-string `@'...'@`）；依赖其成功的 git 状态变更命令（`pull` / `merge` / `checkout`）别 `>/dev/null 2>&1` 吞 stderr，改动后用显式断言验证
- **Bash↔PowerShell 边界**：Bash 工具跑的是 Git Bash(POSIX)，用正斜杠路径 + `2>/dev/null`；禁 `2>$null` / `$null` / `-Force` / `-ErrorAction`（→ `ambiguous redirect`）、禁 Windows 反斜杠路径（末尾 `\"` 会转义掉引号 → `unexpected EOF`）——要反斜杠路径 / `$null` / cmdlet，一律走 PowerShell 工具；PowerShell 工具内多条语句用 `;` 或 `&&` 连、别用裸换行分隔（→ `is not recognized as a cmdlet`），也别在 PowerShell 里写 `/tmp/...` 这类 bash 路径
- **Python 编码**：Windows Python 默认 GBK 编码，print `↔` / emoji 等非 GBK 字符会 `UnicodeEncodeError`——设 `PYTHONIOENCODING=utf-8` 且 `sys.stdout.reconfigure(encoding='utf-8')`，或别往 stdout 打非 ASCII

### 工具调用避坑（通用）

**文件编辑工具（占比最高，常栽这里）**
- 本会话首次对某路径 Edit/Write 前必先 Read 该确切路径（哪怕内容已从 Grep / 上文得知）——`File has not been read yet` 是头号工具错（128 次）
- Edit 撞 `String to replace not found` / `not unique`（27 次）：别改 old_string 反复试，重新 Read 当前确切字节、带足上下文保唯一再编辑；撞 `File has been modified since read`（linter / CRLF 改过）同样重读再改
- background 后台会话改文件前先 `EnterWorktree`、编辑落 worktree 路径——`hasn't isolated its changes yet`（24 次）

**Read / Glob / sleep**
- Read 大文件（>256KB 或 >25000 tokens）必带 `offset` / `limit` 或改用 Grep，别整文件读（7 次超限）
- Glob 宽 `**/x` 模式在大仓库 20s 超时（8 次）——缩到具体子目录或更窄 pattern
- 前台 `sleep` 会被 Block（6 次）——等条件用 Monitor 的 until-loop 或 `run_in_background`，不要前台 sleep

**工具参数 / schema**
- 参数别串台：Read 不收 `pattern` / `path`（那是 Grep 的）、Bash 不收 `file_path`；调 deferred 工具（Monitor 等）前先用 ToolSearch 取 schema，否则 `schema was not sent to the API`
- AskUserQuestion：每题选项 ≤4 个、必填字段（`questions` 及每题 `question`）别漏（9 次 schema 错）

````
