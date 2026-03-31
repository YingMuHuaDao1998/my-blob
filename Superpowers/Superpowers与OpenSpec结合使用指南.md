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
    D --> E[Superpowers: 头脑风暴]
    E --> F[Superpowers: 编写计划]
    F --> G[Superpowers: 子代理开发]
    G --> H[Superpowers: 测试驱动]
    H --> I[Superpowers: 代码审查]
    I --> J[Superpowers: 分支完成]
    J --> K[OpenSpec: 归档变更]
```

### 阶段详解

#### 阶段一：需求澄清（OpenSpec + Superpowers 头脑风暴）

**OpenSpec** 的 `/opsx:propose` 启动结构化提案流程：

```
用户: /opsx:propose add-user-auth
AI: Created openspec/changes/add-user-auth/
  ✓ proposal.md — 为什么做这件事
  ✓ specs/ — 需求和场景
  ✓ design.md — 技术方案
  ✓ tasks.md — 实现检查清单
准备开始实现！
```

**Superpowers 头脑风暴** 进行苏格拉底式需求深挖：

- 提出澄清性问题
- 探索替代方案
- 分段展示设计供人工批准

#### 阶段二：规范确认（OpenSpec specs）

OpenSpec 生成的 `tasks.md` 是具体实现检查清单：

```markdown
## tasks.md 示例

- [ ] 创建 User 模型，包含 email/password 字段
- [ ] 使用 bcrypt 对密码加密
- [ ] 实现 JWT token 生成
- [ ] 创建登录 API 端点 POST /auth/login
- [ ] 添加输入验证中间件
- [ ] 编写 auth 服务的单元测试
```

这些任务直接作为 Superpowers `writing-plans` 的输入。

#### 阶段三：执行与验证（Superpowers）

```mermaid
flowchart LR
    A[按 tasks 执行] --> B[TDD 红-绿-重构]
    B --> C[代码审查]
    C --> D[验证修复有效]
    D --> E[提交/合并]
```

---

## 安装与配置（Codex 篇）

### 前置条件

- OpenAI Codex CLI 已安装
- Git

---

### 安装 OpenSpec（Codex）

OpenSpec 通过技能（Skills）和斜杠命令（Slash Commands）集成到 Codex。

**方法一：自动安装（推荐）**

```bash
# 初始化 OpenSpec，选择 Codex 平台
openspec init --platform codex

# 或全局安装（所有平台）
openspec init --global
```

**方法二：手动安装**

```bash
# 1. 创建 Codex 技能目录
mkdir -p ~/.codex/skills

# 2. 克隆 OpenSpec 仓库
git clone https://github.com/Fission-AI/OpenSpec.git /tmp/openspec

# 3. 复制 Codex 技能文件
cp -r /tmp/openspec/.codex/skills/openspec-* ~/.codex/skills/

# 4. 创建斜杠命令提示文件
mkdir -p ~/.codex/prompts
cp /tmp/openspec/.codex/prompts/opsx-*.md ~/.codex/prompts/
```

**验证安装**

启动 Codex 新会话，输入：

```
/opsx:help
```

如果看到 OpenSpec 帮助信息，说明安装成功。

---

### 安装 Superpowers（Codex）

让 Codex 自动执行安装：

```
Fetch and follow instructions from https://raw.githubusercontent.com/obra/superpowers/refs/heads/main/.codex/INSTALL.md
```

或手动安装：

```bash
# 1. 克隆 Superpowers 仓库
git clone https://github.com/obra/superpowers.git ~/.codex/superpowers

# 2. 创建技能符号链接
mkdir -p ~/.agents/skills
ln -s ~/.codex/superpowers/skills ~/.agents/skills/superpowers

# 3. 重启 Codex 会话
```

**验证安装**

```
help me plan this feature
```

Codex 应该触发 Superpowers 的 brainstorming 和 writing-plans 技能。

---

### 启用多代理功能（可选）

Superpowers 的部分技能依赖多代理特性，在 `~/.codex/config.toml` 中添加：

```toml
[features]
multi_agent = true
```

### 目录结构

结合使用时，Codex 项目推荐结构：

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
├── .codex/                      # Codex 配置
│   ├── skills/                  # OpenSpec 技能
│   └── AGENTS.md                # 全局指令（可选）
├── ~/.agents/skills/superpowers # Superpowers 技能（符号链接）
└── src/                         # 源代码
```

---

### Codex 整合后的完整工作流

当 OpenSpec 和 Superpowers 都安装好后，Codex 中的标准开发流程：

```mermaid
flowchart TD
    A[用户提出需求] --> B["用 OpenSpec 提出变更<br/>/opsx:propose <功能名>"]
    B --> C[Human + AI 对齐 specs]
    C --> D[OpenSpec 生成 design.md + tasks.md]
    D --> E["用 Superpowers 规划任务<br/>help me plan this feature"]
    E --> F[Superpowers brainstorming 细化]
    F --> G[Superpowers writing-plans 拆分任务]
    G --> H[Superpowers TDD 执行]
    H --> I[Superpowers code-review 审查]
    I --> J["OpenSpec 归档变更<br/>/opsx:archive"]
    J --> K[提交 PR / Merge]
```

**Codex 中的命令序列示例：**

```
# 1. 开启新功能规范流程
/opsx:propose add-user-auth

# 2. 等待 specs 对齐完成后，用 Superpowers 规划
help me plan this feature using tasks.md

# 3. 执行 TDD
let's test-drive this implementation

# 4. 代码审查
review the auth module

# 5. 完成分支
finish this development branch

# 6. 归档 OpenSpec 变更
/opsx:archive
```

---

## 实用技巧

### 技巧一：OpenSpec 作为 Superpowers 的输入层

**问题**：Superpowers 的 brainstorming 有时过于发散

**方案**：先用 OpenSpec `/opsx:propose` 固定范围，再交给 Superpowers 执行

```
# 1. 先用 OpenSpec 定义范围
/opsx:propose add-payment-integration

# 2. AI 生成结构化提案后，交给 Superpowers
help me plan this feature using the tasks.md
```

### 技巧二：用 Superpowers 审查 OpenSpec 生成的 Tasks

**问题**：OpenSpec 生成的 tasks 可能过于简单

**方案**：用 Superpowers 的 `writing-plans` 进一步细化

```bash
# 将 tasks.md 的任务细化为 2-5 分钟的原子任务
help me expand these tasks into a detailed plan with 2-5 min subtasks
```

### 技巧三：OpenSpec archive 作为项目历史

OpenSpec 的 archive 功能保留了完整的变更历史：

```
openspec/changes/archive/2025-01-23-add-dark-mode/
├── proposal.md
├── design.md
├── specs/
└── tasks.md (已勾选)
```

这些归档可以直接作为项目的技术决策文档。

### 技巧四：Superpowers TDD 验证 OpenSpec Tasks

OpenSpec 的 tasks 只定义"做什么"，Superpowers 的 TDD 确保"做得对"：

```mermaid
flowchart TD
    A[OpenSpec task: 创建用户模型] --> B[Superpowers: 写失败测试]
    B --> C[写最小实现]
    C --> D[Superpowers: 重构]
    D --> E[OpenSpec: 标记任务完成]
```

### 技巧五：双层代码审查

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
| `/opsx:propose <名称>` | 创建新变更提案 |
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

### 场景一：新功能开发

```mermaid
flowchart LR
    A["/opsx:propose 新功能"] --> B[对齐 specs]
    B --> C[Superpowers 头脑风暴]
    C --> D[细化 tasks]
    D --> E[TDD 实现]
    E --> F[代码审查]
    F --> G["/opsx:archive"]
```

### 场景二：Bug 修复

```
# 用 OpenSpec 记录修复方案
/opsx:propose fix-login-redirect

# 用 Superpowers 验证修复
help me debug why login redirect isn't working
run tests to verify the fix
```

### 场景三：代码重构

```
# OpenSpec 定义重构范围
/opsx:propose refactor-auth-module

# Superpowers 执行重构
help me plan the refactoring with test coverage
```

---

## 常见问题

### 问一：两个工具会不会冲突？

**不会**。它们作用于不同阶段：

- OpenSpec → 代码编写之前（对齐）
- Superpowers → 代码编写及之后（执行）

### 问二：可以只用 OpenSpec 不用 Superpowers 吗？

可以。OpenSpec 独立完整，Superpowers 是增强选项。

### 问三：支持哪些 AI 助手？

- **OpenSpec**：20+ AI 助手（Claude Code、Cursor、Copilot、Windsurf 等）
- **Superpowers**：Claude Code、Codex、Cursor、OpenCode、Gemini CLI 等

### 问四：需要网络吗？

都不需要 API Key，纯本地运行。

---

## 实际项目案例（两者结合使用）

> 以下案例均为 **OpenSpec + Superpowers 组合使用**的真实实践

---

### 案例一：用户直接在 GitHub 提问如何结合（Issue #859）

**来源**：[GitHub Issue #859](https://github.com/Fission-AI/OpenSpec/issues/859) — 2026年3月19日

**背景**：有用户在 OpenSpec 官方仓库提问："How can I use OpenSpec and Superpowers together? Could you create a skill?"

这是目前能找到的最直接的关于两者结合使用的官方讨论。核心观点：

> **"Superpowers provides individual productivity skills (TDD, debugging, planning, etc.) · OpenSpec provides structured workflow governance (proposal, review, approval)"**

即：
- **Superpowers** → 个体生产力技能（TDD、调试、规划等）
- **OpenSpec** → 结构化流程治理（提案、审查、审批）

**衍生进展**：[Issue #780](https://github.com/Fission-AI/OpenSpec/issues/780) 提出了将 OpenSpec 作为 Superpowers 技能分发的功能请求——让用户可以通过 Superpowers 插件市场直接安装 OpenSpec，实现一键集成。

---

### 案例二：Claude Code 用户真实工作流（Builder.io 博客）

**来源**：[Builder.io 官方博客 - The Superpowers Plugin](https://www.builder.io/blog/claude-code-superpowers-plugin)

**项目背景**：使用 Superpowers 插件在 Claude Code 中实现 Ding v1 配置验证功能

**关键实践经验**：

1. **Superpowers plan 包含完整可运行代码**
   - Plan 生成后，每个任务都包含失败的测试用例
   - 实现代码必须让测试通过才算完成

2. **三条不可妥协的规则**：
   ```
   ① Spec 是所有决策的唯一依据
   ② 每个 plan 任务在写实现代码之前必须先有失败测试
   ③ 任务复选框是会话恢复机制
   ```

3. **OpenSpec 在此场景的对应作用**：
   - 如果用 OpenSpec 替代纯文本 Spec，proposal/design.md 就是"Spec"
   - OpenSpec 的 Delta Specs 让变更可审计、可对比
   - 提案-审批流程让实现有据可依

**两者结合的实践路径**：

```
OpenSpec: 提出变更 → 生成 design.md（技术方案）
    ↓
Superpowers: 读取 design.md 作为 spec
    ↓
help me plan this feature using design.md
    ↓
Superpowers TDD: 写失败测试 → 写最小实现 → 重构
    ↓
OpenSpec: /opsx:archive 归档变更
```

---

### 案例三：st0012.dev 的 Ruby 项目工作流

**来源**：[st0012.dev](https://st0012.dev/links/2026-01-15-a-claude-code-workflow-with-the-superpowers-plugin/)

**项目背景**：Ruby on Rails 项目中使用 Superpowers 插件

**具体用法**：

```
# 用 Superpowers brainstorm 捕获任务上下文
/superpowers:brainstorm I want to build a prototype for <issue>
```

**与 OpenSpec 的对应关系**：

| Superpowers 步骤 | OpenSpec 对应物 |
|-----------------|----------------|
| brainstorm | `/opsx:propose` + 人工对齐 |
| 生成 plan | design.md + tasks.md |
| TDD 执行 | 按 tasks.md 逐项实现 |
| code-review | 人工审批 / opsx:review |
| 完成分支 | `/opsx:archive` |

---

### 案例四：OpenSpec 作为 Superpowers 技能的提案（Issue #780）

**来源**：[GitHub Issue #780](https://github.com/Fission-AI/OpenSpec/issues/780)

**核心提案**：将 OpenSpec 作为 Superpowers marketplace 中的一个技能分发

**优势**：
1. 用户可以在 Superpowers 生态中直接发现和安装 OpenSpec
2. 安装体验统一，不需要分别配置两个工具
3. OpenSpec 的提案-审查流程可以作为 Superpowers 工作流的入口层

**提案的工作流整合**：

```mermaid
flowchart LR
    A[Superpowers<br/>marketplace] --> B[安装 OpenSpec 技能]
    B --> C[Codex / Claude Code]
    C --> D{开发场景}
    D -->|新功能| E["OpenSpec /opsx:propose"]
    D -->|Bug 修复| F["OpenSpec /opsx:propose"]
    E --> G[Superpowers brainstorm]
    F --> G
    G --> H[Superpowers TDD]
    H --> I[Superpowers code-review]
    I --> J["OpenSpec /opsx:archive"]
```

---

### 案例五：Framework 对比中的定位（Rick Hightower）

**来源**：[Medium - The Great Framework Showdown](https://medium.com/@richardhightower/the-great-framework-showdown-superpowers-vs-bmad-vs-speckit-vs-gsd-360983101c10)

在五大 AI 编程框架（BMAD、SpecKit、OpenSpec、GSD、Superpowers）对比中，对两者的定位：

| 框架 | 核心定位 |
|------|---------|
| OpenSpec | **brownfield-first delta specs**（增量规格，擅长存量项目） |
| Superpowers | **TDD-enforced discipline**（强制 TDD 纪律） |

**组合后的竞争力**：

> OpenSpec 解决"改什么"的问题（基于现有代码库的增量规范）
> Superpowers 解决"怎么改"的问题（强制测试先行 + 子代理执行）

这意味着组合使用特别适合：**在已有代码库上进行系统化功能迭代**，既能保持规范可追溯，又能保证代码质量。

---

## 参考资料

- [OpenSpec GitHub](https://github.com/Fission-AI/OpenSpec)
- [Superpowers GitHub](https://github.com/obra/superpowers)
- [OpenSpec GitHub Issue #859 - 官方讨论两者结合](https://github.com/Fission-AI/OpenSpec/issues/859)
- [OpenSpec GitHub Issue #780 - 作为 Superpowers 技能分发提案](https://github.com/Fission-AI/OpenSpec/issues/780)
- [Builder.io - The Superpowers Plugin for Claude Code](https://www.builder.io/blog/claude-code-superpowers-plugin)
- [st0012.dev - Claude Code workflow with Superpowers](https://st0012.dev/links/2026-01-15-a-claude-code-workflow-with-the-superpowers-plugin/)
- [Medium - The Great Framework Showdown](https://medium.com/@richardhightower/the-great-framework-showdown-superpowers-vs-bmad-vs-speckit-vs-gsd-360983101c10)
- [AI Plain English - Framework 对比](https://ai.plainenglish.io/the-great-framework-showdown-superpowers-vs-bmad-vs-speckit-vs-gsd-360983101c10)

---

## 更新日志

- **2026-03-31**：添加实际组合使用案例 — Issue #859 官方讨论、Builder.io 工作流、st0012 Ruby 项目、Issue #780 技能分发提案、Rick Hightower 框架对比定位
- **2026-03-31**：初始版本，整合 Superpowers 与 OpenSpec 结合使用技巧
