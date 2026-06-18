---
name: pr-writing
description: 写或重写 GitHub Pull Request 描述。默认使用 AS IS / TO BE 结构让评审者快速判断"现在哪里坏了"和"这个 PR 改了什么"；并给出 bug 修复、安全修复、新功能、重构、依赖升级、回滚等场景的变体。Use when drafting or rewriting PR descriptions, preparing security/bugfix PR bodies, release-review notes, or reviewer-facing change summaries. Also use when the user mentions "写 PR"、"PR 描述"、"PR body"、"PR summary"、"review notes".
---

# PR Writing

## 目标

PR 描述只服务一种读者：**评审者和经理**。他们要在最短时间内回答两个问题：

1. 这个 PR 改了什么？
2. 为什么要改？

任何不直接服务这两个问题的句子都应该被砍。代码 diff 本身已经回答"逐行怎么改"，PR 描述不需要复述。

## 语言

- **标题永远英语**——Conventional Commits 规范本身基于英文 type，且英文标题在国际化协作 / CI 工具 / 搜索中通用。即使中文团队也不破例。
- **正文跟作者已有 PR 的语言**——先扫一眼作者在本仓库的历史 PR（`gh pr list --author @me --repo <owner>/<repo> --state all --limit 10` 看 title + body），用 ta 已经在用的语言。没有历史就跟仓库主流语言（看最近合入的 10 个 PR），仍无法判断再退到英文。
- 命令、路径、接口名、代码标识符无论正文什么语言都保持原样不翻译。

## 默认风格

- 惜字如金。优先用条目，不写转场句、铺垫句、评价句。
- AS IS / TO BE 每段通常 1-3 个 bullet；每个 bullet 只说一个事实。
- 默认不写验证段。只有用户明确要求、仓库模板强制要求、或验证结果本身影响评审判断时才写。
- 不写工具链元信息，不写生成过程，不提 coding agent、模型、自动生成来源，除非它是 PR 的产品/代码对象。
- Branch name、PR title、正文标题都用功能/问题命名；不要出现 `codex`、`coding-agent`、`agent-generated`、`ai-generated` 等制作来源。

## 用户偏好：简洁准确的 PR

当用户要求撰写或发布 PR 时：

- 标题始终只用英文 Conventional Commits；正文语言遵循用户明确要求，未指定时再按作者历史 PR 或仓库主流语言判断。
- 正文保持短，只写“为什么改”和“改成什么”。不要默认加入验证段。
- AS IS / TO BE 优先写 bullet；删除“同时”、“另外”、“为了”、“本 PR”等填充词。
- 创建分支或 PR 时，分支名、PR title、正文标题都不要带 coding agent 相关信息。
- 先给 diff 准确定性：替换旧方案就写 `replace`，不要写成 `add`；重命名 / 边界整理就明确写成伴随替换发生的整理。
- 如果 diff 很大，用一句话解释原因：通常是旧抽象、DB table、worker entrypoint、API、SDK 等边界都带着旧命名或旧数据形状，不是只换模型加载代码。
- 模型或架构替换时，把用户明确给出的真实动机写进 AS IS；例如用户说明从 CLIP 切到 SigLIP2 是因为日语文本查询更强，就直接写这个原因。避免把替换描述成“新增服务”。

## 标题

遵循 [Conventional Commits](https://www.conventionalcommits.org/) 规范。评审者在 PR 列表里只看到 title，且 squash merge 时它会成为最终 commit message，规范化收益最大。

格式：

```
<type>(<scope>): <subject>
```

**type**（必填）：

| type | 用途 |
|---|---|
| `feat` | 新功能 |
| `fix` | bug 修复 |
| `docs` | 文档 |
| `style` | 不影响行为的格式（空格、分号、排版） |
| `refactor` | 重构（既非 feat 也非 fix） |
| `perf` | 性能优化 |
| `test` | 测试新增 / 修改 |
| `build` | 构建系统 / 依赖 |
| `ci` | CI 配置 |
| `chore` | 杂项（版本号、配置） |
| `revert` | 回滚 |

**scope**（可选但推荐）：影响的模块 / 包 / 子系统，如 `auth`、`api`、`ui`、`db`。

**subject**：动词原形开头、小写、不加句号；说做了什么不说为什么（why 留给 body）；整行 ≤ 70 字符（含 type / scope）。

禁止把制作来源写进标题或分支名。用 `feat/billing-record-edits`，不要用 `codex/billing-record-edits`；用 `feat(agent): bill record edits`，不要用 `[codex] ...` 或 `agent-generated ...`。

**破坏性变更**用 `!`：`feat(api)!: rename /v1/users to /v2/users`。

对照：

- ❌ `bugfix` / `update auth` / `Improvements to the authentication flow`
- ✅ `fix(auth): race condition in session-token refresh under concurrent login`
- ✅ `feat(api): add pagination cursor to /search`
- ✅ `refactor(db): extract connection pool from request middleware`

**始终英语**——即使正文中文也一样（详见「语言」节）。

## 默认结构：AS IS / TO BE

绝大多数 PR（bug 修复、安全修复、行为变更）使用两段式：

```markdown
## AS IS

- <当前行为 / 缺失能力>
- <触发条件或影响，只在有用时写>

## TO BE

- <改后行为>
- <影响范围 / 兼容性，只在有用时写>
```

只有两个一级小节。不要平行新增 "Background"、"Motivation"、"Implementation"、"Validation"、"Future Work" 等小节，除非用户或模板明确要求。

## AS IS：现状

描述 PR 之前的状态。只写评审者判断范围所需的信息：

- **对象**：接口、页面、流程、任务或数据口径。
- **现状**：缺失、错误、偏差或不一致。
- **影响**：只在代码 diff 看不出业务后果时写。

AS IS 只描述现状，不写修复方案。能一条说清就不要拆成三条。

## TO BE：改动后

描述 PR 之后的状态。只写改后行为：

- **行为**：新逻辑、新口径、新入口或新约束。
- **范围**：会影响哪些用户可见或后台可见路径。
- **兼容性 / 迁移**：只在需要评审者留意时写。

不要默认写验证、测试、CI。用户要求时，单独加短条目，不写长段。

## 关联 issue / 工单

有关联的 issue / 工单时，单独列在 PR 正文末尾，**用列表条目而不是行内引用**：

```markdown
- resolves #123
- refs JIRA-4567
```

为什么必须是列表条目：GitHub 在条目里渲染 `#123` 时会展开成 issue 标题，行内的 `#123` 只显示编号。同理 cross-repo `owner/repo#123`、外部工单 ID（Linear、JIRA、Sentry）也都要走条目格式才有 hover 预览。

关键词选择（决定 merge 时是否自动关闭 issue）：

- `resolves` / `fixes` / `closes`：merge 后自动关闭对应 issue。**只用在这个 PR 真的解决该 issue 时**
- `refs` / `related to`：只关联不关闭，用于"这个 PR 部分推进了某 issue"或"修复但未关闭，因为还有后续工作"

每个引用占一个条目；不要在一条里塞两个编号（`- resolves #1, #2` 会丢预览）。

## 其他场景的变体

AS IS / TO BE 是"修复某个明确问题"的天然模板。其他场景在不改变骨架的前提下调整两段的内涵：

| 场景 | AS IS 怎么写 | TO BE 怎么写 |
|---|---|---|
| 新功能 | 目前没有 X 能力，导致用户 / 调用方只能 Y | 新能力的形状、入口、使用方式 |
| 重构 / 内部清理 | 当前结构带来的具体痛点（重复、耦合点、难以测试的边界），不是"代码不优雅" | 新结构如何解决这些痛点 |
| 依赖升级 | 旧版本的具体问题（CVE 编号、弃用 API、已知 bug） | 变化的接口、需要适配的调用方、潜在影响 |
| 紧急回滚 | 引入了什么问题、什么时候、怎么发现的 | 回到的版本、需要后续什么修复 |

## 何时简化

PR 太小时不必硬套 AS IS / TO BE：

- README / 注释 / typo / lockfile 自动更新
- 单一参数 / 配置值微调
- 纯样式 / 排版

一句话说明（"为什么改、改成什么、依据"）即可。判断标准：评审者看 diff 5 秒能懂的，跳过结构化。

## 完整示例

> **Title**: `fix(auth): race condition in session-token refresh under concurrent login`
>
> ## AS IS
>
> - `POST /api/auth/refresh` 并发刷新时会各自生成新 token。
> - 旧 token 残留 5 分钟。
>
> ## TO BE
>
> - 用 `SETNX` 保证并发刷新只保留一份有效 token。
> - 写入失败时复用已有 token；旧 token 即时 invalidate。
>
> ---
>
> - resolves #842
> - refs SEC-431

## 砍掉的内容

以下内容**默认不要写**，除非读者真的需要：

- **长 Background 段落**——评审者打开 PR 时已经有上下文，压缩到一句话或直接进入 AS IS。
- **完整 diff 复述**——他们会读代码。PR 描述提供的是"代码看不出来的意图"。
- **验证段**——除非用户明确要求、模板强制要求，或验证结果影响发布判断。
- **工具链元信息**（"用 X 生成"、"参考了 Y 讨论"）——除非该信息对评审决策有用。
- **Future work 清单**——除非必须和这个 PR 配对发布。开 issue 跟踪，别塞进描述。
- **形容词式成果**（"显著提升"、"大幅简化"、"全面优化"）——用数字或具体行为替代，否则删掉。

## 评审前检查

发出 PR 或标记 ready for review 之前过一遍：

- AS IS 是否说清了现状 / 缺失场景？
- TO BE 是否说清了改后行为？
- 每段是否控制在 1-3 个 bullet？超过就拆或砍。
- 有没有可删的填充词、转场句、过程描述？
- 有没有"提升了安全性"这种空话？后面有没有跟上"具体收紧了哪个边界"？
- 是否默认加了验证段？没有明确要求就删。
- 标题符合 Conventional Commits 格式（`type(scope): subject`）、≤ 70 字符、单独看能判断要不要点开？
- 有关联 issue 时，是否以列表条目形式列出（`- resolves #123`）而不是行内提及？

## 失败模式

| 症状 | 原因 | 处理 |
|---|---|---|
| 评审者反复问"这个会影响 X 吗" | AS IS 触发条件不具体 | 把触发条件细化到输入 / 状态 / 权限层级 |
| PR 描述要和 diff 来回对照才能看懂 | TO BE 在复述 diff，没提炼意图 | 提到"做了什么本质改动"的层级，不要列文件名 |
| 读起来像营销文案 | 没有量化、没有 trade-off | 用数字 / 接口名替换形容词，补一句"代价是什么" |
| AS IS 和 TO BE 内容互相渗透 | 没有保持"问题 / 方案"分离 | 现状和触发归 AS IS，改动和验证归 TO BE |
| 评审者看不出风险 | 没提 blast radius | 说明影响边界、向后兼容性、是否需要迁移 |
| 一份 PR 描述比 diff 还长 | 把背景、思路、迭代过程都塞进去了 | 删元信息、删过程、保留状态对比 |
