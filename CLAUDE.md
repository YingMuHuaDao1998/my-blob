# LLM Wiki 全局心智规范

## 语言与角色

- **语言**：始终使用简体中文进行思考、回复和知识库编写。
- **角色**：你正在维护一个 **LLM Wiki**（基于 [Karpathy LLM Wiki 理念](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)），任务是：将碎片化信息编译成结构化、高度相互链接的知识网络。

---

## 目录结构

```
🏛️ my-blob (LLM Wiki Vault)
├── 📂 {Topic}/                  ← 已有知识主题目录（已结构化的知识）
├── 🖼️ assets/                   ← 统一媒体资源：图片、PDF、附件
│
├── 📥 raw/                      ← 原始资料收件箱（只读事实层）
│   ├── 📄 01-articles/          ← 网页剪藏、技术文章 (.md)
│   ├── 🎓 02-papers/            ← 论文、深度研报、PDF
│   ├── 🎙️ 03-transcripts/        ← 视频/播客转录文本
│   ├── 💡 04-meeting_notes/     ← 头脑风暴、会议纪要
│   └── 🗃️ 09-archive/           ← ingest 成功后自动归档
│
├── 🧠 wiki/                     ← 知识编译输出层（AI 拥有完全写权限）
│   ├── 📑 index.md              ← 全局内容字典：所有 wiki 页面索引
│   ├── 📜 log.md                ← 操作日志（Append-only）
│   ├── 🏗️ concepts/             ← 抽象层：方法论、架构模式
│   ├── 👥 entities/             ← 实体层：人名、公司、工具、项目
│   ├── 🔍 sources/              ← 摘要层：raw 文件的一对一核心提炼
│   └── 💎 syntheses/            ← 综合层：复杂问题的深度研究报告
│
├── 🤖 CLAUDE.md                 ← 本文件：全局心智规范
└── ⚙️ .claude/skills/           ← Agent Skills
    ├── ⚙️ ingest/               ← 将 raw 文件编译到 wiki
    ├── 🔎 query/                ← 检索 wiki 并生成带引用的回答
    └── 🩺 lint/                  ← 知识库体检
```

---

## 目录权限边界

| 目录 | 权限 | 说明 |
|------|------|------|
| `raw/` | **绝对只读** | 原始素材来源，禁止修改或删除 |
| `assets/` | 读写 | 媒体资源，引用时用 `![[文件.png]]` |
| `wiki/` | **AI 完全控制** | 你需要在此创建、更新、提炼知识 |
| `{Topic}/` | **已有知识** | 人类已整理的主题知识，AI 可读不可随意重写 |

---

## Wiki 核心文件契约

### 1. `wiki/index.md` — 总目录

每次新增 wiki 页面后必须同步更新。格式：

```markdown
# Wiki Index

## Sources
- [[摘要-slug]] — 资料核心主旨

## Entities
- [[EntityName]] — 实体身份或核心功能

## Concepts
- [[ConceptName]] — 概念或框架的核心定义

## Syntheses
- [[synthesis-slug]] — 回答的复杂问题
```

### 2. `wiki/log.md` — 操作日志（Append-only）

每次操作后追加记录：

```markdown
## [YYYY-MM-DD] <动作> | <简述>
- **变更**: 新增/更新了 [[PageName]]
- **冲突**: 无 / 冲突 [[PageName]] 已标注
```

### 3. 内容分类规则

- `wiki/concepts/`：方法论、框架、理论（如 `Agent_Skill.md`）
- `wiki/entities/`：人物、公司、工具、产品（如 `Claude_Code.md`）
- `wiki/sources/`：raw 文件的提炼摘要（如 `摘要-claude-code-best-practices.md`）

### 4. 强制双向链接

每个 wiki 页面必须包含 `## 关联连接` 区域，绝不能产生孤岛页面。

### 5. 矛盾处理原则

新知识与旧知识冲突时，不要静默覆盖。在页面中新建 `## 知识冲突` 区块，保留两种说法并做对比分析。

---

## 页面 Frontmatter 规范

所有 wiki 页面必须包含：

```yaml
---
title: "页面标题"
type: concept | entity | source | synthesis
tags: [标签1, 标签2]
sources: [raw/01-articles/xxx.md]  # 仅 source/synthesis 页面
last_updated: YYYY-MM-DD
---
```

---

## Skills 工作流

- **`/ingest`**：读取 `raw/` 文件，提炼核心，创建 wiki 摘要/实体/概念页面，更新 index 和 log，完成后归档到 `raw/09-archive/`
- **`/query <问题>`**：读取 `wiki/index.md` 找相关页面，深度阅读后综合回答，回答中必须用 `[[wikilink]]` 标注引用来源
- **`/lint`**：扫描 wiki/ 找孤岛页面、死链、逻辑冲突，报告给用户

---

## 已有知识说明

`{Topic}/` 目录（如 `Celery/`、`LangChain/`）是已结构化的主题知识库。AI 在回答问题时应该优先从这些目录中检索原始内容，再结合 wiki/ 的提炼层综合回答。
