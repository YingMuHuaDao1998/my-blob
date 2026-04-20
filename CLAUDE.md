# LLM Wiki 全局心智规范

## 语言与角色

- **语言**：始终使用简体中文进行思考、回复和知识库编写。
- **角色**：你正在维护一个 **LLM Wiki**（基于 [Karpathy LLM Wiki 理念](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)），任务是：将碎片化信息编译成结构化、高度相互链接的知识网络。

---

## 目录结构

```
🏛️ my-blob (LLM Wiki Vault)
│
├── 🧠 wiki/                     ← 知识核心层
│   ├── topics/                  ← 【人类整理的原始知识】← 人类整理的主题知识，AI 可读不可随意重写
│   │   ├── Celery/
│   │   ├── LangChain/
│   │   ├── Redis/
│   │   └── ...（26个主题目录）
│   ├── sources/                 ← AI 提炼层：raw 文件的摘要索引
│   ├── concepts/                ← AI 提炼层：方法论、架构模式
│   ├── entities/               ← AI 提炼层：工具、框架、模型
│   ├── syntheses/              ← AI 提炼层：复杂问题的深度报告
│   ├── index.md                ← 全局内容字典
│   └── log.md                  ← 操作日志（Append-only）
│
├── 📥 raw/                      ← 原始资料收件箱（只读事实层）
│   ├── 01-articles/           ← 网页剪藏、技术文章
│   ├── 02-papers/             ← 论文、PDF
│   ├── 03-transcripts/         ← 视频转录
│   ├── 04-meeting_notes/      ← 头脑风暴、会议纪要
│   └── 09-archive/            ← ingest 成功后自动归档
│
├── 🖼️ assets/                  ← 统一媒体资源
│
├── 🤖 CLAUDE.md                 ← 本文件：全局心智规范
└── ⚙️ .claude/skills/         ← Agent Skills
    ├── ingest/                 ← raw → wiki 编译
    ├── query/                  ← 检索与综合回答
    └── lint/                   ← 知识库体检
```

---

## 目录权限边界

| 目录 | 权限 | 说明 |
|------|------|------|
| `raw/` | **绝对只读** | 原始素材，禁止修改或删除 |
| `wiki/topics/` | **AI 可读，不可随意重写** | 人类整理的知识，只做增量补充 |
| `wiki/sources/` | **AI 完全控制** | 提炼摘要，AI 拥有完全写权限 |
| `wiki/concepts/` | **AI 完全控制** | 概念提炼，AI 拥有完全写权限 |
| `wiki/entities/` | **AI 完全控制** | 实体索引，AI 拥有完全写权限 |
| `assets/` | 读写 | 媒体资源 |

---

## Wiki 核心文件契约

### 1. `wiki/index.md` — 总目录

每次新增 wiki 页面后必须同步更新：

```markdown
# Wiki Index

## Topics（主题层）
- [[topics/Celery/]] — Celery 分布式任务队列
- [[topics/LangChain/]] — LangChain 开发框架
...

## Sources（摘要层）
- [[sources/摘要-slug]] — 资料核心主旨

## Concepts（概念层）
- [[concepts/ConceptName]] — 概念或框架定义

## Entities（实体层）
- [[entities/EntityName]] — 实体身份或核心功能

## Syntheses（综合层）
- [[syntheses/synthesis-slug]] — 复杂问题分析报告
```

### 2. `wiki/log.md` — 操作日志（Append-only）

```markdown
## [YYYY-MM-DD] <动作> | <简述>
- **变更**: 新增/更新了 [[PageName]]
- **冲突**: 无 / 冲突 [[PageName]] 已标注
```

### 3. 强制双向链接

每个 wiki 页面必须包含 `## 关联连接` 区域，绝不能产生孤岛页面。

### 4. 矛盾处理原则

新知识与旧知识冲突时，不静默覆盖。在页面中新建 `## 知识冲突` 区块，保留两种说法并做对比。

---

## 页面 Frontmatter 规范

```yaml
---
title: "页面标题"
type: topic | source | concept | entity | synthesis
tags: [标签1, 标签2]
sources: [raw/01-articles/xxx.md]  # source/synthesis 必填
last_updated: YYYY-MM-DD
---
```

---

## Skills 工作流

- **`/ingest`**：读取 `raw/` 文件，提炼核心，创建 wiki 摘要/概念/实体页面，更新 index 和 log，完成后归档到 `raw/09-archive/`
- **`/query <问题>`**：读取 `wiki/index.md` 找相关页面，综合回答，回答中必须用 `[[wikilink]]` 标注引用来源
- **`/lint`**：扫描 wiki/ 找孤岛页面、死链、逻辑冲突，报告给用户

---

## 知识流转

```
外部资料 → raw/ → ingest → wiki/sources/（摘要）
                            → wiki/concepts/（概念提炼）
                            → wiki/entities/（实体索引）

AI 回答问题 → 优先 wiki/syntheses/ + sources/ → topics/（原始知识）
                            → concepts/ + entities/（提炼层）
```
