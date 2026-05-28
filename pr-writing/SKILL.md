---
name: pr-writing
description: 写或重写 GitHub Pull Request 描述。默认使用 AS IS / TO BE 结构让评审者快速判断"现在哪里坏了"和"这个 PR 改了什么"；并给出 bug 修复、安全修复、新功能、重构、依赖升级、回滚等场景的变体。Use when drafting or rewriting PR descriptions, preparing security/bugfix PR bodies, release-review notes, or reviewer-facing change summaries. Also use when the user mentions "写 PR"、"PR 描述"、"PR body"、"PR summary"、"review notes".
---

# PR Writing

## 目标

PR 描述只服务一种读者：**评审者和经理**。他们要在最短时间内回答三个问题：

1. 这个 PR 改了什么？
2. 为什么要改？
3. 怎么相信它没坏？

任何不直接服务这三个问题的句子都应该被砍。代码 diff 本身已经回答"逐行怎么改"，PR 描述不需要复述。

## 语言

- **标题永远英语**——Conventional Commits 规范本身基于英文 type，且英文标题在国际化协作 / CI 工具 / 搜索中通用。即使中文团队也不破例。
- **正文跟作者已有 PR 的语言**——先扫一眼作者在本仓库的历史 PR（`gh pr list --author @me --repo <owner>/<repo> --state all --limit 10` 看 title + body），用 ta 已经在用的语言。没有历史就跟仓库主流语言（看最近合入的 10 个 PR），仍无法判断再退到英文。
- 命令、路径、接口名、代码标识符无论正文什么语言都保持原样不翻译。

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

<当前行为，以及它在什么条件下出问题。>

## TO BE

<这个 PR 之后的行为，以及如何验证。>
```

只有两个一级小节，因为读者只需要"前 / 后"两个状态的对比。其余内容应该折叠进这两段，不要平行新增 "Background"、"Motivation"、"Implementation"、"Future Work" 等小节。

## AS IS：现状

描述 PR 之前的状态。四个要素，缺一不可：

- **影响面**：命名受影响的接口、函数、页面、流程。颗粒度匹配评审者能定位的层级，不要笼统说"登录功能"，要说"`/api/auth/refresh` 在 token 过期时的处理"。
- **失败模式**：具体的错误或不期望的行为。不要说"有 bug"，要说"在 X 条件下返回 Y，预期是 Z"。
- **触发条件**：什么样的输入、状态、权限边界、并发场景会触发问题。评审者据此判断是否影响他们关心的代码路径。
- **影响**：用评审者熟悉的语言描述后果——数据丢失、权限越界、性能退化、用户可见错误。避免"严重"、"潜在风险"这类形容词，用具体后果替代。

AS IS 只描述"坏了的现状"。**不要在这里写修复方案**，那是 TO BE 的内容。两边内容互相渗透是这个格式最常见的失败模式。

## TO BE：改动后

描述 PR 之后的状态。四个要素：

- **最小有意义的改动**：解释这个改动的本质（新增 guard、收紧权限、修正状态机、改变持久化时机），而不是逐行讲解 diff。
- **原本坏的场景现在怎么走**：直接对应 AS IS 里的触发条件，说明在新行为下会发生什么。
- **验证路径**：跑了什么测试、加了什么测试、人工怎么验证。如果只跑了局部测试就明说，不要含糊成 "tests passed"。
- **风险与 blast radius**：是否向后兼容、是否需要数据迁移、是否影响其他依赖该路径的功能。如果完全无风险也明说一句。

CI 状态只在你最近确实看过且通过时再提，并指明哪个 check。

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
> `POST /api/auth/refresh` 在并发刷新 token 时存在 race condition：两个请求同时到达会各自生成新 token 写入 Redis，旧 token 残留 5 分钟未被 invalidate。
>
> 触发：单用户同时打开两个 tab 都触发自动刷新；mobile 客户端在弱网下重试。
>
> 影响：5 分钟会话劫持窗口。
>
> ## TO BE
>
> 用 `SETNX` 取代 `SET`，token 写入失败则复用已有 token；旧 token 即时 invalidate。完全向后兼容，无需迁移。
>
> 验证：
> - 新增 `test_concurrent_refresh_keeps_single_token`
> - CI 全量 green（pytest + integration）
> - 手动两个 curl 并发触发，Redis 始终只有一份有效 token
>
> ---
>
> - resolves #842
> - refs SEC-431

## 砍掉的内容

以下内容**默认不要写**，除非读者真的需要：

- **长 Background 段落**——评审者打开 PR 时已经有上下文，压缩到一句话或直接进入 AS IS。
- **完整 diff 复述**——他们会读代码。PR 描述提供的是"代码看不出来的意图"。
- **工具链元信息**（"用 X 生成"、"参考了 Y 讨论"）——除非该信息对评审决策有用。
- **Future work 清单**——除非必须和这个 PR 配对发布。开 issue 跟踪，别塞进描述。
- **形容词式成果**（"显著提升"、"大幅简化"、"全面优化"）——用数字或具体行为替代，否则删掉。

## 评审前检查

发出 PR 或标记 ready for review 之前过一遍：

- AS IS 是否同时说清了"现状"和"失败 / 缺失场景"？
- TO BE 是否同时说清了"改动"和"验证方式"？
- 每段长度控制在 1–3 个短段落或要点；超过就拆或砍。
- 有没有"提升了安全性"这种空话？后面有没有跟上"具体收紧了哪个边界"？
- 是否过度承诺？只跑了 unit test 就别说 "all tests pass"，写实际跑了什么。
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
