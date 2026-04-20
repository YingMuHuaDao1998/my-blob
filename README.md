# my-blob

个人知识库 — LLM Wiki Vault

## 架构

```
my-blob/
├── {Topic}/                     ← 已结构化的主题知识（Celery/LangChain/...）
├── assets/                      ← 统一媒体资源
│
├── raw/                        ← 原始资料收件箱（只读）
│   ├── 01-articles/            ← 技术文章剪藏
│   ├── 02-papers/              ← 论文/PDF
│   ├── 03-transcripts/         ← 视频转录
│   ├── 04-meeting_notes/       ← 会议纪要
│   └── 09-archive/             ← 已处理归档
│
├── wiki/                       ← 知识编译输出层（AI 可写）
│   ├── index.md                ← 全局页面字典
│   ├── log.md                  ← 操作日志
│   ├── concepts/               ← 概念层
│   ├── entities/              ← 实体层
│   ├── sources/               ← 摘要层
│   └── syntheses/             ← 综合报告层
│
├── .claude/skills/            ← Agent Skills
│   ├── ingest/                ← 摄入原始资料
│   ├── query/                 ← 检索知识库
│   └── lint/                  ← 知识库体检
│
└── CLAUDE.md                   ← 全局心智规范
```

## Agent Skills

在 Codex / Claude Code 中打开本目录，使用以下命令：

- `/ingest` — 将 `raw/` 中的原始资料编译到 `wiki/`，并自动归档
- `/query <问题>` — 在知识库中检索，综合回答并标注引用
- `/lint` — 检查知识库健康度（死链、孤岛页面等）

## 设计理念

参考 [Karpathy LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)：

- **raw/** = 不可篡改的事实来源层
- **wiki/** = AI 编译提炼的知识输出层
- 一切皆需双向链接，不产生孤岛页面
- 知识冲突不覆盖，显式对比保留
