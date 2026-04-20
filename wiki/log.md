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
- **备注**: 基于 karpathy-llm-wiki-vault 改造，将原有 68 个 .md 文件的知识体系纳入 vault 管理
