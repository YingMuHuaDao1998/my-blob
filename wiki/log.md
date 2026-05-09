---
title: Wiki Log
type: log
last_updated: 2026-05-09
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

- **检索**: [[AI-Agent/AI Agent开发实践指南：经典范式与实例.md]]、[[AgentFramework/Agent构建核心方法对比：LangChain_LangGraph_DeepAgents.md]]、[[AgentFramework/构建支持Skill的多智能体系统.md]]
- **输出**: 结合知识库与 LangChain / LangGraph 官方文档，回答多子代理高质量搭建方案

## [2026-04-21] archive | 归档 LangChain + LangGraph 多子代理设计文档

- **变更**: 新增 [[AgentFramework/LangChain+LangGraph高质量多子代理智能体设计文档.md]]
- **变更**: 更新 [[wiki/index.md]] AgentFramework 索引
- **冲突**: 无
- **备注**: 由本次对话总结生成，偏通用任务型多子代理架构设计

## [2026-04-21] query | 什么是 LangChain

- **检索**: [[LangChain/LangChain学习总结.md]]、[[LangChain/LangChain最佳实践.md]]、[[LangChain/LangChain生态三件套使用指南.md]]
- **输出**: 说明 LangChain 的定义、定位、核心组成及与 LangGraph 的关系

## [2026-04-23] archive | 整理 Prebuilt middleware 中文笔记

- **变更**: 新增 [[LangChain/LangChain预置Middleware使用指南.md]]
- **变更**: 更新 [[wiki/index.md]] LangChain 索引
- **冲突**: 无
- **备注**: 基于 [[raw/01-articles/Prebuilt middleware]] 翻译整理

## [2026-04-24] archive | 整理 DeepAgent Skill 与 Subagent 示例

- **变更**: 新增 [[LangChain/DeepAgent加载Skill与Subagent示例.md]]
- **变更**: 更新 [[wiki/index.md]] LangChain 索引
- **冲突**: 无
- **备注**: 归档本次关于 DeepAgent、skills 与自定义 subagent 的示例说明

## [2026-04-27] archive | 整理 DeepAgent Skills 与 Middleware 关系图解

- **变更**: 新增 [[LangChain/DeepAgent Skills与Middleware关系图解.md]]
- **变更**: 更新 [[wiki/index.md]] LangChain 索引
- **冲突**: 无
- **备注**: 归档本次关于 DeepAgent Skills 与 Middleware 分工关系的说明与图示

## [2026-04-27] ingest | 导入 LangChain 预置 Middleware 资料

- **变更**: 新增 [[wiki/sources/摘要-langchain-prebuilt-middleware]]
- **变更**: 新增 [[wiki/concepts/Agent Middleware 治理]]、[[wiki/concepts/上下文治理中间件]]
- **变更**: 新增 [[wiki/entities/LangChain Middleware]]、[[wiki/entities/Deep Agents]]
- **变更**: 更新 [[wiki/index.md]] 摘要层、概念层、实体层索引
- **冲突**: 无
- **备注**: 已归档源文件 `raw/01-articles/Prebuilt middleware.md`

## [2026-04-28] lint | 知识库体检

- **孤岛**: 0个（已修复0个，剩余0个待处理）
- **死链**: 24个（已修复24个）
- **index不同步**: 6个（已修复）
- **未 ingest**: 0个
- **备注**: 忽略 `.claude/skills/` 与 `CLAUDE.md` 中的示例 wikilink，不作为真实死链

## [2026-04-28] query | 什么是 Superpowers

- **检索**: [[Superpowers/使用指南.md]]、[[Superpowers/Superpowers与OpenSpec结合使用指南.md]]、[[Superpowers/Codex集成指南.md]]
- **输出**: 说明 Superpowers 的定义、工作流定位、技能机制与与 OpenSpec 的关系

## [2026-04-29] query | LangChain createAgent 主从 Agent 与异步 Subagents

- **检索**: [[LangChain/LangChain学习总结.md]]、[[AgentFramework/构建支持Skill的多智能体系统.md]]、LangChain 官方 Subagents 文档
- **输出**: 说明 createAgent 中 supervisor/subagents 模式、同步工具封装与后台异步 job 三工具模式

## [2026-05-09] archive | 归档 create_agent 主从 Agent 异步化指南

- **变更**: 新增 [[LangChain/LangChain create_agent主从Agent异步化指南.md]]
- **变更**: 更新 [[wiki/index.md]] LangChain 索引
- **冲突**: 无
- **备注**: 基于 LangChain Python Subagents 官方文档与本次对话整理，聚焦 LangGraph supervisor + async worker 方案

## [2026-05-09] query | create_agent 先规划再执行

- **检索**: [[AgentFramework/构建支持Skill的多智能体系统.md]]、[[LangChain/LangChain学习总结.md]]、LangChain 官方 Subagents / Custom workflow / Structured output 文档
- **输出**: 说明 planner-executor 模式应使用 create_agent + LangGraph，自定义 planner 节点输出 plan，再派发子代理执行

## [2026-05-09] archive | 归档 LangGraph fan-out / fan-in 并发子智能体指南

- **变更**: 新增 [[LangChain/LangGraph fan-out与fan-in并发子智能体指南.md]]
- **变更**: 更新 [[wiki/index.md]] LangChain 索引
- **冲突**: 无
- **备注**: 基于本次关于 Subagents、single dispatch tool 与 LangGraph 并发模式的对话整理

