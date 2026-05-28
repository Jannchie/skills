---
name: readme-writing
description: Write or rewrite a project README that converts a 10-second visitor into a user. Use when the user asks to write/rewrite/improve a README, mentions "写 README", "README 写作", "项目说明", or wants a README generated from existing code or notes. Front-loads identity → value → minimum-runnable example; cuts marketing fluff, premature API dumps, and decorative TOCs.
---

# README 写作

读者是 10 秒内决定要不要继续看的访客，不是已被说服的用户。每一行回答「我为什么还在读」；前三屏决定一切。

## README 骨架

按从上到下顺序。可选块没素材就跳过，不要硬凑。

**居中视觉块**（仅当有 icon / logo 时启用，用 `<p align="center">` 包裹整块）：

- icon / logo（单张、紧凑）
- H1 项目名
- badge 行
- 可选 demo 截图 / GIF / asciinema

无 icon 时全部左对齐，跳过居中包裹。

**正文**（左对齐）：

- **简介**：一句话身份（`项目名 + 动词 + 对象`，不要 "modern / blazing-fast" 形容词）+ 为什么用它（回答「我已经有 X，凭什么换」），合计 3–5 行
- **安装**：一条命令；多平台 / 多语言分列
- **快速上手**：最小可跑的真实场景，目标 5 行内（必需的 import / context 不计入，做不到说明示例选错了）
- **常用用法**：2–4 个真实场景，不是 API 穷举
- **配置 / API**：链接到独立文档，README 留 ~5 个最常用入口
- **License**：一行

**底部一行**：FAQ / Roadmap / Contributing / Changelog 链接，全部外链。

## 读者最小信息量

每一句话问：删了读者损失什么？

- 损失「能不能用」→ 留
- 损失「适不适合我」→ 留
- 损失「作者很认真的感觉」→ 删

## 示例：真实 vs 占位

占位符让读者判断不出适不适合自己。真实示例 = 文档 + 营销 + 烟囱测试。

- 占位：`client.do_thing(YOUR_API_KEY, "your_data_here")`
- 真实：`client.translate("Hello", target="ja")  # → "こんにちは"`

## 段落：臃肿 vs 精简

臃肿版：

> Welcome to FooLib! 🎉 FooLib is a modern, blazing-fast, developer-friendly library for parsing configuration files. Born out of our team's frustration with existing tools, FooLib aims to provide a delightful experience for developers of all skill levels. Whether you're building a small CLI or a large distributed system, FooLib has you covered.

精简版：

> FooLib parses YAML / TOML / JSON configs into typed structs. Faster than PyYAML on files > 10MB, and unlike Dynaconf it has no runtime env-var magic — what you load is what you get.

差异：身份明确（动词 + 对象）、给出两个真实痛点 + 已知替代对照、零形容词堆、零迎宾语。

## 不要放进 README 的东西

**装饰类**：

- Badge 海洋。最多 3–4 个，每个回答一个具体问题（build / version / license / downloads）；Stars badge、"made with love"、"PRs welcome" 全删
- Emoji。小节标题前缀（🚀 Quick Start、📦 Install）、feature 列表的 ✅ / ❌、装饰性的 ✨ 全不用，用纯文本 bullet 即可。例外：项目本身就是 emoji / 字符 / 角色相关工具
- ASCII 艺术、分隔线、装饰横幅

**应外链的文档类**：

- API 全集 → docs 站 / `docs/` / docstring
- 完整配置参考 → 独立文档
- 架构图谱 → `ARCHITECTURE.md`
- 路线图细节 → `ROADMAP.md`
- 贡献指南正文 → `CONTRIBUTING.md`
- 变更历史 → `CHANGELOG.md`

**套话与冗余**：

- "Welcome to..." / "Thanks for checking out..."
- "This project was born out of..."（项目起源放博客）
- 复述 install 命令做了什么
- "Feel free to open an issue"（GitHub 已经有按钮）
- 每个特性都配一段散文（应该是 bullet）

判断标准：第一次来的访客不需要的东西就不放。

## 变体与例外

- **多语言**：面向中文用户时 README 中文，配 `README.en.md`，互相在顶部链接。命令、路径、API 名不翻译
- **极小库**（≤ 10 个 API / 单文件）：可以在 README 完整列 API，不强制拆 `docs/`
- **企业 / 合规**：可保留 disclaimer / 法律声明块，但放在 License 之后，不放在前三屏
- **Tutorial / cookbook 类项目**：示例代码比例可远高于普通项目，但「身份 + 价值」开场仍适用
- **CLI / TUI 工具**：demo 优先用终端 GIF / asciinema；GUI / Web 用截图或短 GIF（< 10s，< 5MB）

## 返工诊断

| 反馈 | 违反 | 处理 |
|---|---|---|
| "看不出能干什么" | 缺一句话身份 | 重写简介首句，动词 + 对象说清 |
| "太长" | API 全集塞进来了 | 拆 `docs/`，README 留入口示例 |
| "太硬广" | 形容词堆 / 没痛点 | 用具体场景替 "blazing-fast" 类词 |
| "示例跑不起来" | 占位符 | 换可复制运行的真实片段 |
| "找不到 install" | 装饰物把前两屏顶下去 | 砍 badge / banner，install 提到前两屏 |
| "看着很 cheap" | emoji / 装饰横幅 | 全删，用纯文本 |

## 收尾检查

- 只看前 200 字，能否答出「是什么 / 为什么用 / 怎么开始」？
- 第一段无形容词堆、无 emoji、无迎宾语？
- 示例能直接复制运行？
- 全文 < 300 行？（超出说明在做 docs 的事，挪出去）
