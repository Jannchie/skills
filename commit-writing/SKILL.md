---
name: commit-writing
description: 写符合 Conventional Commits 规范的 git commit message。保持精简：subject 一行讲清意图，body 只在 diff 看不出来时才补。Use when creating git commits, writing commit messages, splitting a working tree into commits, or when the user mentions "写 commit"、"commit message"、"提交信息"、"提交规范".
---

# Commit Writing

## 目标

commit message 服务两类读者：

1. **未来的你 / 同事**——在 `git log` / `git blame` 里追溯"这行为什么这么改"
2. **自动化工具**——从规范化 type 生成 changelog / 触发 release / 决定 semver bump

不要复述 diff，diff 自己会说话。要写的是 diff 看不出的**意图**。

## 格式

遵循 [Conventional Commits](https://www.conventionalcommits.org/)：

```
<type>(<scope>): <subject>

<body>

<footer>
```

只有 subject 一行是必填，body / footer 按需。

## 语言

始终英语。Conventional Commits 的 type 本身基于英文，且英文 commit 在国际化协作、CI 工具、changelog 自动化里通用。中文团队也不破例。命令、路径、API 名保持原样。

## type

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

破坏性变更用 `!`：`feat(api)!: rename /v1/users to /v2/users`，并在 body 加 `BREAKING CHANGE: <迁移说明>`。

## scope

可选但推荐。命名受影响的模块 / 包 / 子系统，如 `auth`、`api`、`ui`、`db`。monorepo 用包名（`@foo/bar` → `bar`）。跨多个不相关模块就别写 scope，别硬凑。

## subject 规则

- 动词原形开头（`add` / `fix` / `remove`，不是 `added` / `fixing`）
- 小写开头，不加句号
- 说"做了什么"，不说"为什么"（why 留给 body）
- ≤ 70 字符（含 type / scope）
- 单独看能判断要不要点开

对照：

- ❌ `fixed bug` / `update auth` / `various improvements` / `WIP`
- ✅ `fix(auth): race condition in token refresh under concurrent login`
- ✅ `feat(api): add cursor pagination to /search`
- ✅ `refactor(db): extract connection pool from request middleware`

## 什么时候写 body

绝大多数 commit **不需要 body**。只在 diff 看不出来时才补：

- 选了非显然方案——一句话说为什么不是更直观的另一种
- 修复牵涉的具体触发条件（race / 特定输入 / 权限边界）
- 关联 issue / 工单（`Refs: #123`、`Fixes: SEC-431`）
- 性能 / 安全的具体数据（`reduces p95 from 800ms to 120ms`）

body 与 subject 之间留空行，每行 ≤ 72 字符。

## 拆 commit 还是合 commit

每个 commit 应该是一个**可独立 review、可独立 revert** 的单元。

- 一次改动跨多个独立关注点（重构 + 新功能 + 修 bug）→ 拆，用 `git add -p`
- 一连串小动作服务同一意图（rename + 更新 import + 修 type）→ 合
- 中间状态会让 CI 红 / 让测试编不过 → 不要在那里切

## 不要写进 commit message

- 无信息的 subject（"更新一些代码"、"小修改"、"WIP"）
- 复述 diff（"修改了 foo.py 的 bar 函数"）
- 工具链元信息（"用 X 生成"、"AI 协助"），除非影响后续追溯
- emoji / 装饰字符
- 多余的迎宾或致谢

## 失败模式

| 症状 | 原因 | 处理 |
|---|---|---|
| `git log --oneline` 看不出做了什么 | subject 太笼统 | 加 scope，把"哪里坏了 / 加了什么"写进 subject |
| 一个 commit 改了五个不相关的事 | 没拆 | `git reset HEAD~` 后用 `git add -p` 分批 |
| body 比 diff 还长 | 在复述代码 | 砍掉所有 diff 能自证的部分 |
| reviewer 反复问"为什么这么改" | 缺意图 | body 补一句"为什么不是另一种方案" |
| 标题里堆形容词（"重大改进"） | 在写营销词 | 换成具体接口名 / 数字 / 行为 |
