# mymy-claudemd
项目开发集大成之作的CLAUDE.md

````markdown
## 当前本地 Claude code 运行环境
- 当前电脑操作系统是windows 11
- 运行Claude code的终端是PowerShell 7
- 执行命令的终端要使用PowerShell 7
- 中文回答

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
13. **先论后证**：结论性陈述(bug 根因 / 分析判断 / 验证结果 / 事实)先给结论、再附可核验证据(`file:line` / 命令输出 / 断言结论正确的最小验证)
14. **分析无 ground-truth，靠反偏置兜底**：纯分析判断(没有代码可编译 / 测试当场证伪)时，结论最易被确认偏误 + 讨好式假收敛带偏。两条栏杆：(a) 主动找**反证**，不只堆支持证据——找不到反证 ≠ 结论成立，只是"暂未找到"；(b) 证据不足就标**"未收敛 / 盲点"**，不强凑皆大欢喜的中庸结论。多候选时 ≥3 并给排除理由(比写码的 ≥2 严一档，因为这里没编译器兜底)

## 工作模式

- **要留证据链的重型验证**（改动会迭代多轮 / risky / 多业务形态各需独立验证）→ 按 `项目产物归档规范` 中 `docs/acceptance/<name>/` 下划分规范：每轮 `round-N.md`（记 PASS 证据，粒度见 `docs/acceptance/README.md`）+ 跨轮 `scripts/` + 全绿后的`report.md`，其他可选。
- **复杂任务先分析**（scope 大 / 跨服务 / schema 变更 / 架构重构 / 改动>=50行）→ 先全面分析清楚、落档 `docs/spec/<topic>.md`。

破坏性/风险性脚本需带 `--check` 零副作用自检，执行前先跑一次自检。部署按 `docs/ops/` runbook 走，不用记命令。

## 项目产物归档规范

**MUST**：所有非源码输出放 `docs/` 子目录，禁止散到 `docs/` 根或仓库根。类型不明时先看现有桶放最接近的；都不接近时问用户，**不要自建新顶层目录**。

### 目录结构

```
docs/
├── spec/              # 前置分析、方案、实施计划（不含事后复盘）
├── api/               # 本项目后端暴露的 API 契约（第三方 API 放 reference/）
├── acceptance/        # 验收方案/结果/脚本/证据/特性复盘
│   ├── README.md      # 证据归档红线（大 binary 不入库 + 再生成配方 + 源头治理）
│   ├── _shared/       # 跨 feature 复用的 e2e 工具包(auth/env/ui/api/report);≥2 feature 用过才进
│   └── <feature>/
│       ├── plan.md            # [需求 + 方案 + 改动 + 验证]或[症状 + 根因 + 同根因 + 修复 + 验证]
│       ├── report.md          # 全绿才写
│       ├── retrospective.md   # 可选，复盘归此处不进 spec/
│       ├── round-N.md         # 每轮验证记录，fix-rerun 新增不覆盖，N是数字
│       ├── round-N/           # 可选，仅当该轮有独立证据(截图/日志/dump)才建目录存放
│       ├── scripts/           # 跨轮可复用的测试脚本 / debug 工具
│       └── fixtures/          # 跨轮复用的静态测试资产(图片/文件/权限 JSON);per-round 数据放 round-N/fixtures.json
├── tmp/               # 当前 session 的 ad-hoc 草稿；PR 合入后转走或删除，不得跨特性堆积
├── reference/         # 外部资料/调研/原始需求/第三方 API 文档（原文件名保留）
├── sql/               # DDL、migration、查询
├── ops/               # 部署流程、运维笔记（凭据禁入库：放仓库外或 .gitignore）
└── manual/            # 最终用户说明（开发者指南放 <service>/CLAUDE.md 或 README.md）
```

**命名**：英文小写 + 连字符、3–6 词、不重复目录类型（`spec/plan-foo.md` 不写成 `spec-plan-foo.md`）；`reference/` 豁免。

## Git 工作流（单人开发分级）

**默认**：trivial 改动（docs / 小 bug fix <50 行 / config tweak / 依赖升级 / 格式化）直接 `git commit && git push`。**涉及代码改动**：不强制完整验证 harness，但下面 3 条 ship 硬线无条件兑现；要不要上 matrix/round-N 看改动是否需要证据链（判定见上文「工作模式」）。

**改代码就要兑现的 3 条 ship 硬线**（每次提交 / PR 无条件适用，都是栽过才立的）：
1. **发 PR / 提交前必须本地实跑过**：代码改动要本地起服务真跑一遍、看到预期结果才算完，没跑不准发，不"留待部署后跑"（2026-04-24 mapping-unit-rename 教训）
2. **push 前查 PR state**：`gh pr view <num> --json state,mergedAt` 确认没 push 到已合并的 PR branch；已合就新开 PR 续修订
3. **PR body 自包含**：写清改了什么（`file:line`）+ 你实跑看到的证据，reviewer 不翻别处也能判断

**必须走 feature branch + PR** 的场景：
1. 大 feature（>10 文件 或 >500 行新增）
2. Risky 改动（架构重构、schema 变更、deploy 脚本、auth/billing）
3. 跨 session WIP — 一次完不成，branch 作为保存点
4. 将来 CI 加 required checks 之后

Push 默认不需要事先询问用户 — commit 合理、分支是 feature 分支、测试已过，直接 push。force-push、推 main/master、外部仓库 push 例外，仍需确认。

**分支命名**：
- **worktree 内分支**：`worktree-<name>`（跟官方 `claude --worktree` 约定，未来 session picker `Ctrl+B` / `--from-pr` 等 tooling 自动识别）
- **非 worktree feature 分支**：`feature/<name>`，**不论 feature / bug / refactor**——前缀只是工作流标识，任务类型由 PR 标题 / `plan.md` / commit message 表达

两种前缀视觉上立刻能分辨"这分支来自 worktree 还是 standalone"。

````
