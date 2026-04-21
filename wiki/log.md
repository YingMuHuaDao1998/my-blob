---
title: Wiki Log
type: log
last_updated: 2026-04-21
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

## [2026-04-21] ingest | 导入 Hermes Agent 资料

- **变更**: 新增 [[wiki/sources/摘要-hermes-agent]]
- **变更**: 新增 [[wiki/entities/Hermes Agent]]、[[wiki/entities/Nous Research]]
- **变更**: 新增 [[wiki/concepts/自我改进智能体]]、[[wiki/concepts/Agent 长期记忆与技能闭环]]
- **变更**: 更新 [[wiki/index.md]] 摘要层、概念层、实体层索引
- **冲突**: 无
- **备注**: 已归档源文件 `raw/01-articles/NousResearchhermes-agent The agent that grows with you.md`

## [2026-04-21] ingest | 导入 Harness 革命综述

- **变更**: 新增 [[wiki/sources/摘要-harness-革命]]
- **变更**: 新增 [[wiki/concepts/Agent Harness]]、[[wiki/concepts/Harness Engineering]]、[[wiki/concepts/Agent 可控性]]
- **变更**: 新增 [[wiki/entities/Claude Code]]、[[wiki/entities/Codex]]、[[wiki/entities/OpenClaw]]
- **变更**: 更新 [[wiki/index.md]] 摘要层、概念层、实体层索引
- **冲突**: 无
- **备注**: 已归档源文件 `raw/01-articles/最新！万字综述Harness革命！.md`

## [2026-04-21] query | LangChain + LangGraph 多子代理智能体搭建

- **检索**: [[topics/AI-Agent/AI Agent开发实践指南：经典范式与实例.md]]、[[topics/AgentFramework/Agent构建核心方法对比：LangChain_LangGraph_DeepAgents.md]]、[[topics/AgentFramework/构建支持Skill的多智能体系统.md]]
- **输出**: 结合知识库与 LangChain / LangGraph 官方文档，回答多子代理高质量搭建方案

## [2026-04-21] archive | 归档 LangChain + LangGraph 多子代理设计文档

- **变更**: 新增 [[AgentFramework/LangChain+LangGraph高质量多子代理智能体设计文档.md]]
- **变更**: 更新 [[wiki/index.md]] AgentFramework 索引
- **冲突**: 无
- **备注**: 由本次对话总结生成，偏通用任务型多子代理架构设计

