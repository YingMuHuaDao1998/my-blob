# OpenSpec 中文模板配置指南

## 概述

OpenSpec 是一个 AI 原生的规范驱动开发（Spec-Driven Development）工具，通过 `proposal → specs → design → tasks` 四阶段工作流，帮助团队系统化管理需求和变更。

**问题**：OpenSpec 默认模板和指令为英文，对中文项目不友好，生成文档全是英文。

**解决方案**：将 OpenSpec 的模板文件和 schema 配置汉化，使 AI 能够生成中文文档。

---

## 环境信息

| 项目 | 版本/路径 |
|------|----------|
| OpenSpec CLI | `/opt/homebrew/bin/openspec` |
| OpenSpec 版本 | 1.2.0 |
| Node 模块位置 | `/opt/homebrew/lib/node_modules/@fission-ai/openspec/` |
| 默认模板位置 | `/opt/homebrew/lib/node_modules/@fission-ai/openspec/schemas/spec-driven/templates/` |
| 中文模板位置 | `~/.openspec-cn/` |

---

## OpenSpec 工作流

```
proposal（提案）→ specs（规格）→ design（设计）→ tasks（任务）→ apply（执行）→ archive（归档）
```

| 阶段 | 英文 | 中文 | 输出文件 |
|------|------|------|---------|
| 提案 | Proposal | 提案 | `proposal.md` |
| 规格 | Specs | 规格 | `specs/**/*.md` |
| 设计 | Design | 设计 | `design.md` |
| 任务 | Tasks | 任务 | `tasks.md` |

---

## 模板文件对照

### proposal.md — 提案文档

| 原文 | 中文 |
|------|------|
| Why | 为什么 |
| What Changes | 变更内容 |
| Capabilities | 功能 |
| New Capabilities | 新增功能 |
| Modified Capabilities | 修改功能 |
| Impact | 影响范围 |
| ADDED Requirements | 新增需求 |
| Context | 背景 |
| Goals / Non-Goals | 目标 / 非目标 |
| Decisions | 决策 |
| Risks / Trade-offs | 风险 / 权衡 |
| Task Group | 任务组 |

### spec.md — 规格说明书

| 原文 | 中文 |
|------|------|
| Feature Overview | 功能概述 |
| Users & Use Cases | 使用者与使用场景 |
| Requirements | 需求 |
| Requirement | 需求 |
| Data Model | 数据模型 |
| Edge Cases | 边界情况 |
| External Dependencies | 外部依赖 |
| Acceptance Criteria | 验收标准 |

### design.md — 技术设计文档

| 原文 | 中文 |
|------|------|
| Overview | 概述 |
| Architecture Decision | 架构决策 |
| Technical Approach | 技术方案 |
| Changes | 改动点 |
| Files | 新增 / 修改的文件 |
| API Design | API 设计（如涉及）|
| Database Changes | 数据库变更（如涉及）|
| Testing Strategy | 测试策略 |
| Deployment Notes | 部署注意事项 |

### tasks.md — 任务清单

| 原文 | 中文 |
|------|------|
| Implementation Tasks | 实现任务 |
| Task Group | 任务组 |
| Completion Criteria | 完成标准 |

---

## 文件结构

```
~/.openspec-cn/
├── schema-cn.yaml       # 中文 schema（完整工作流 + AI 指令）
├── templates/           # 中文模板
│   ├── proposal.md      # 提案模板
│   ├── spec.md          # 规格模板
│   ├── design.md        # 设计模板
│   └── tasks.md         # 任务模板
└── README.md            # 使用说明
```

---

## 配置方法

### 方法一：创建变更时指定 schema（推荐）

```bash
openspec change new 我的需求变更 \
  --schema ~/.openspec-cn/schema-cn.yaml
```

### 方法二：复制模板覆盖原版

> ⚠️ 注意：此方法会影响所有使用 OpenSpec 的项目，仅推荐在纯中文项目中使用。

```bash
# 备份原版模板
cp -r /opt/homebrew/lib/node_modules/@fission-ai/openspec/schemas/spec-driven/templates/ \
      /opt/homebrew/lib/node_modules/@fission-ai/openspec/schemas/spec-driven/templates-en/

# 覆盖为中文模板
cp -r ~/.openspec-cn/templates/* \
      /opt/homebrew/lib/node_modules/@fission-ai/openspec/schemas/spec-driven/templates/
```

### 方法三：恢复英文模板

```bash
cp -r /opt/homebrew/lib/node_modules/@fission-ai/openspec/schemas/spec-driven/templates-en/* \
      /opt/homebrew/lib/node_modules/@fission-ai/openspec/schemas/spec-driven/templates/
```

---

## 关键配置说明

### Schema 指令汉化

`schema-cn.yaml` 中包含了 AI 生成各阶段文档的**指令（instruction）**，已全部翻译为中文。AI 在读取 schema 时会获得中文指令，从而生成中文内容。

关键指令对照：

| artifact | instruction 核心含义（英→中）|
|----------|---------------------------|
| proposal | 创建提案文档，阐明为什么需要此变更 → 创庺提案文档，阐明为什么需要此变更 |
| specs | 创建规格文件，定义系统应该做什么 → 创建规格文件，定义系统应该做什么 |
| design | 创建设计文档，说明如何实现此变更 → 创建设计文档，说明如何实现此变更 |
| tasks | 创建任务清单，细化实现工作 → 创建任务清单，细化实现工作 |

### 保留的规范写法

以下内容**保留英文**，是 OpenSpec 的格式要求：

- **WHEN / THEN**：场景格式，保留英文
- **4 个井号（`####`）**：场景标题，OpenSpec 规范要求
- **SHALL / MUST**：规范性需求语言，OpenSpec 规范要求
- **kebab-case 命名**：如 `user-auth`、`data-export`

---

## 使用示例

```bash
# 1. 在项目中初始化 OpenSpec
cd /我的项目
openspec init

# 2. 创建新变更（使用中文 schema）
openspec change new 添加用户认证功能 \
  --schema ~/.openspec-cn/schema-cn.yaml

# 3. 查看生成的提案文档
cat openspec/changes/添加用户认证功能/proposal.md

# 4. 让 AI 生成规格文档
# AI 会根据中文 schema 生成中文内容
openspec change proposal 添加用户认证功能

# 5. 继续后续阶段
openspec change specs 添加用户认证功能
openspec change design 添加用户认证功能
openspec change tasks 添加用户认证功能
openspec change apply 添加用户认证功能
openspec change archive 添加用户认证功能
```

---

## 相关资源

- **OpenSpec GitHub**：https://github.com/Fission-AI/OpenSpec
- **OpenSpec 官方文档**：https://openspec.dev
- **中文模板套件**：https://github.com/你的用户名/openspec-cn（需创建）

---

## 参考资料

- OpenSpec CLI 帮助：`openspec --help`
- OpenSpec 默认模板：`/opt/homebrew/lib/node_modules/@fission-ai/openspec/schemas/spec-driven/templates/`
- OpenSpec schema 规范：`/opt/homebrew/lib/node_modules/@fission-ai/openspec/schemas/spec-driven/schema.yaml`
