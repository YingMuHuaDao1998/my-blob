---
title: Wiki Log
type: log
last_updated: 2026-04-20
---

# Wiki Log

Append-only 操作日志。每次 ingest/query/lint 操作后追加记录。

---

## [2026-04-20] init | LLM Wiki Vault 初始化

- **变更**: 创建 `raw/`, `wiki/`, `assets/`, `.claude/skills/` 目录结构
- **变更**: 创建 `CLAUDE.md` 全局心智规范
- **变更**: 创建 `wiki/index.md` 总目录
- **变更**: 创建 `.claude/skills/ingest/query/lint` 三个技能
- **冲突**: 无
- **备注**: 基于 karpathy-llm-wiki-vault 改造

## [2026-04-20] restructure | 将 26 个主题目录移入 wiki/topics/

- **变更**: 所有人类整理的知识文档（Celery/LangChain/Redis/...）移入 `wiki/topics/`
- **变更**: 更新 `README.md` 和 `CLAUDE.md` 说明层级架构
- **冲突**: 无
- **备注**: git 历史完整保留，git mv 操作
