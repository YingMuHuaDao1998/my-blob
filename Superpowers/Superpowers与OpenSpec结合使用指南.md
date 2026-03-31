# Superpowers 与 OpenSpec 结合使用指南

> 打造完整的设计→规划→执行→验证工作流

## 概述

**Superpowers** 和 **OpenSpec** 是两个互补的 AI 编程辅助工具，结合使用可以实现：

- **Superpowers**：提供系统化的开发流程（需求澄清、任务分解、TDD、代码审查）
- **OpenSpec**：在代码编写之前建立人机共识的规范（Spec），确保实现方向正确

两者结合的核心思路：**OpenSpec 解决"做什么"的问题，Superpowers 解决"怎么做"的问题**。

---

## 核心概念对比

| 维度 | Superpowers | OpenSpec |
|------|-------------|----------|
| **核心职能** | 开发流程框架 | 规范驱动开发（SDD） |
| **介入时机** | 全流程（设计→实现→审查） | 代码编写之前 |
| **产出物** | 任务列表、测试、审查意见 | Spec 文档（proposal、tasks） |
| **工作模式** | 代理自主执行工作流 | 人机对齐后交付任务 |
| **API Key** | 不需要 | 不需要 |
| **支持平台** | Claude Code、Codex、Cursor 等 | 20+ AI 助手（斜杠命令） |

---

## 工作流整合

### 完整开发流程

```mermaid
flowchart TD
    A[用户需求] --> B[OpenSpec: 提出变更]
    B --> C[Human + AI 对齐规范]
    C --> D[生成 specs + tasks.md]
    D --> E[Superpowers: brainstorming]
    E --> F[Superpowers: writing-plans]
    F --> G[Superpowers: subagent-driven-dev]
    G --> H[Superpowers: TDD]
    H --> I[Superpowers: code-review]
    I --> J[Superpowers: finishing-branch]
    J --> K[OpenSpec: archive 变更]
```

### 阶段详解

#### Phase 1：需求澄清（OpenSpec + Superpowers Brainstorming）

**OpenSpec** 的 `/opsx:propose` 启动结构化提案流程：

```
用户: /opsx:propose add-user-auth
AI: Created openspec/changes/add-user-auth/
  ✓ proposal.md — why we're doing this
  ✓ specs/ — requirements and scenarios
  ✓ design.md — technical approach
  ✓ tasks.md — implementation checklist
Ready for implementation!
```

**Superpowers Brainstorming** 进行苏格拉底式需求深挖：

- 提出澄清性问题
- 探索替代方案
- 分段展示设计供人工批准

#### Phase 2：规范确认（OpenSpec specs）

OpenSpec 生成的 `tasks.md` 是具体实现检查清单：

```markdown
## tasks.md 示例

- [ ] Create User model with email/password fields
- [ ] Add password hashing with bcrypt
- [ ] Implement JWT token generation
- [ ] Create login API endpoint POST /auth/login
- [ ] Add input validation middleware
- [ ] Write unit tests for auth service
```

这些任务直接作为 Superpowers `writing-plans` 的输入。

#### Phase 3：执行与验证（Superpowers）

```mermaid
graph LR
    A[按 tasks 执行] --> B[TDD RED-GREEN-REFACTOR]
    B --> C[Code Review]
    C --> D[验证修复有效]
    D --> E[提交/合并]
```

---

## 安装与配置

### 安装 OpenSpec

```bash
npm install -g @fission-ai/openspec@latest
```

验证安装：

```bash
opsx --version
```

### 安装 Superpowers（以 Claude Code 为例）

```bash
/plugin install superpowers@claude-plugins-official
```

### 目录结构

结合使用时，推荐的项目结构：

```
项目目录/
├── openspec/                    # OpenSpec 规范目录
│   ├── specs/                   # 当前规范（source of truth）
│   └── changes/                 # 变更提案
│       ├── add-feature-x/
│       │   ├── proposal.md
│       │   ├── design.md
│       │   ├── specs/
│       │   └── tasks.md
│       └── archive/             # 已完成的变更
├── .claude/                     # Claude Code 配置
│   └── skills/                  # Superpowers 技能
└── src/                         # 源代码
```

---

## 实用技巧

### 技巧 1：OpenSpec 作为 Superpowers 的输入层

**问题**：Superpowers 的 brainstorming 有时过于发散

**方案**：先用 OpenSpec `/opsx:propose` 固定范围，再交给 Superpowers 执行

```
# 1. 先用 OpenSpec 定义范围
/opsx:propose add-payment-integration

# 2. AI 生成结构化提案后，交给 Superpowers
help me plan this feature using the tasks.md
```

### 技巧 2：用 Superpowers 审查 OpenSpec 生成的 Tasks

**问题**：OpenSpec 生成的 tasks 可能过于简单

**方案**：用 Superpowers 的 `writing-plans` 进一步细化

```bash
# 将 tasks.md 的任务细化为 2-5 分钟的原子任务
help me expand these tasks into a detailed plan with 2-5 min subtasks
```

### 技巧 3：OpenSpec archive 作为项目历史

OpenSpec 的 archive 功能保留了完整的变更历史：

```
openspec/changes/archive/2025-01-23-add-dark-mode/
├── proposal.md
├── design.md
├── specs/
└── tasks.md (checked)
```

这些归档可以直接作为项目的技术决策文档。

### 技巧 4：Superpowers TDD 验证 OpenSpec Tasks

OpenSpec 的 tasks 只定义"做什么"，Superpowers 的 TDD 确保"做得对"：

```mermaid
graph TD
    A[OpenSpec task: 创建用户模型] --> B[Superpowers: 写失败测试]
    B --> C[写最小实现]
    C --> D[Superpowers: 重构]
    D --> E[OpenSpec: 标记任务完成]
```

### 技巧 5：双层代码审查

**OpenSpec design.md** → 检查是否按照规范实现
**Superpowers code-review** → 检查代码质量

```
# OpenSpec 层面：是否符合 design.md？
- [ ] 使用了 bcrypt 而非 MD5
- [ ] JWT secret 从环境变量读取

# Superpowers 层面：代码是否优雅？
- [ ] 错误处理是否完善
- [ ] 是否有内存泄漏
```

---

## 命令对照表

### OpenSpec 常用命令

| 命令 | 作用 |
|------|------|
| `/opsx:propose <name>` | 创建新变更提案 |
| `/opsx:status` | 查看当前变更状态 |
| `/opsx:diff` | 查看规范差异 |
| `/opsx:archive` | 归档已完成变更 |
| `/opsx:list` | 列出所有变更 |

### Superpowers 常用命令

| 命令 | 触发技能 |
|------|----------|
| `help me plan this feature` | brainstorming + writing-plans |
| `let's debug this issue` | systematic-debugging |
| `run tests` | test-driven-development |
| `review this code` | requesting-code-review |
| `finish this branch` | finishing-a-development-branch |

---

## 典型使用场景

### 场景 1：新功能开发

```mermaid
flowchart LR
    A["/opsx:propose 新功能"] --> B[对齐 specs]
    B --> C[Superpowers brainstorming]
    C --> D[细化 tasks]
    D --> E[TDD 实现]
    E --> F[Code Review]
    F --> G["/opsx:archive"]
```

### 场景 2：Bug 修复

```
# 用 OpenSpec 记录修复方案
/opsx:propose fix-login-redirect

# 用 Superpowers 验证修复
help me debug why login redirect isn't working
run tests to verify the fix
```

### 场景 3：代码重构

```
# OpenSpec 定义重构范围
/opsx:propose refactor-auth-module

# Superpowers 执行重构
help me plan the refactoring with test coverage
```

---

## 常见问题

### Q1：两个工具会不会冲突？

**不会**。它们作用于不同阶段：

- OpenSpec → 代码编写之前（对齐）
- Superpowers → 代码编写及之后（执行）

### Q2：可以只用 OpenSpec 不用 Superpowers 吗？

可以。OpenSpec 独立完整，Superpowers 是增强选项。

### Q3：支持哪些 AI 助手？

- **OpenSpec**：20+ AI 助手（Claude Code、Cursor、Copilot、Windsurf 等）
- **Superpowers**：Claude Code、Codex、Cursor、OpenCode、Gemini CLI 等

### Q4：需要网络吗？

都不需要 API Key，纯本地运行。

---

## 参考资料

- [OpenSpec GitHub](https://github.com/Fission-AI/OpenSpec)
- [Superpowers GitHub](https://github.com/obra/superpowers)
- [OpenSpec Medium 文章](https://medium.com/coding-nexus/openspec-a-spec-driven-workflow-for-ai-coding-assistants-no-api-keys-needed-d5b3323294fa)
- [Spec-Driven Development 指南](https://aliirz.com/getting-started-with-sdd)

---

## 更新日志

- **2026-03-31**: 初始版本，整合 Superpowers 与 OpenSpec 结合使用技巧
