# my-blob

个人 LLM Wiki 知识库

## 目录结构

```
my-blob/
│
├── 🧠 wiki/                      ← 知识核心层
│   ├── topics/                  ← 人类整理的主题知识（可链接引用）
│   │   ├── Celery/
│   │   ├── LangChain/
│   │   ├── Redis/
│   │   ├── AI-Agent/
│   │   └── ...（26个主题目录）
│   ├── sources/                 ← 摘要层：raw/ 原始资料的提炼摘要
│   ├── concepts/                ← 概念层：方法论、框架、第一性原理
│   ├── entities/                ← 实体层：人名、公司、工具、产品
│   ├── syntheses/               ← 综合层：复杂问题的深度研究报告
│   ├── index.md                 ← 全局页面字典
│   └── log.md                   ← 操作日志（Append-only）
│
├── 📥 raw/                      ← 原始资料收件箱（只读）
│   ├── 01-articles/            ← 技术文章剪藏
│   ├── 02-papers/              ← 论文/PDF
│   ├── 03-transcripts/         ← 视频转录
│   ├── 04-meeting_notes/       ← 会议纪要
│   └── 09-archive/            ← ingest 后自动归档
│
├── 🖼️ assets/                  ← 统一媒体资源
│
├── 🤖 CLAUDE.md                 ← 全局心智规范
└── ⚙️ .claude/skills/          ← Agent Skills
    ├── ingest/                  ← 编译 raw → wiki/sources
    ├── query/                   ← 检索并综合回答
    └── lint/                    ← 知识库体检
```

## 层级说明

| 层级 | 定位 | 示例 |
|------|------|------|
| `wiki/topics/` | 人类整理的原始知识 | `topics/Celery/Celery入门教程.md` |
| `wiki/sources/` | AI 提炼的资料摘要 | `sources/摘要-celery-best-practices.md` |
| `wiki/concepts/` | 抽象概念层 | `concepts/分布式任务队列.md` |
| `wiki/entities/` | 实体（工具/框架/模型） | `entities/Celery.md` |
| `wiki/syntheses/` | 综合分析报告 | `syntheses/Agent开发选型对比.md` |

## Agent Skills

在 Codex / Claude Code 中打开本目录：

- **`/ingest`** — 把 `raw/` 中的材料编译到 `wiki/sources/`，并自动归档
- **`/query <问题>`** — 检索知识库，综合回答并标注双链引用
- **`/lint`** — 检查知识库健康度（死链、孤岛页面等）

## 设计理念

基于 [Karpathy LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)：

- **raw/** = 不可篡改的事实来源层
- **wiki/** = AI 编译提炼的知识输出层
- 一切皆需双向链接，不产生孤岛页面
- 知识冲突不覆盖，显式对比保留
