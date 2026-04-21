# OpenSpec 定制化配置指南：中文 Spec + 自定义模板

> 让 OpenSpec 生成的 proposal/design/tasks/spec 文档全程使用中文，并按团队自定义模板输出

---

## 一、核心机制总览

OpenSpec 支持三类定制化：

| 层级 | 文件位置 | 控制内容 |
|------|---------|---------|
| **语言配置** | `openspec/config.yaml` | 生成文档的语言 |
| **Schema 定制** | `openspec/schemas/*.yaml` | 生成哪些文档、顺序、字段 |
| **模板内容** | `openspec/templates/` | 每种文档的具体格式和引导词 |

三层可以独立使用，也可以组合。实现"中文 + 自定义模板"需要同时配置语言和模板。

---

## 二、语言配置：让所有产物输出中文

### 2.1 配置方法

在 `openspec/config.yaml`（项目根目录）添加 `context` 配置：

```yaml
context: |
  Language: Chinese (zh-CN)
  所有 artifacts（proposal、design、specs、tasks）必须使用简体中文。
  技术术语（如 API、REST、GraphQL、HTTP）保留英文。
  代码示例和文件路径保留原样。
  日期格式使用 YYYY-MM-DD。
```

### 2.2 验证配置是否生效

```bash
openspec show <change-id>
# 检查输出内容是否为中文
```

### 2.3 多语言场景示例

```yaml
# 中文项目
context: |
  Language: Chinese (zh-CN)
  所有 artifacts 必须使用简体中文。

# 日文项目（保留技术术语英文）
context: |
  Language: Japanese
  Write in Japanese, but:
  - Keep technical terms like "API", "REST", "GraphQL" in English
  - Code examples and file paths remain in English

# 葡萄牙语项目
context: |
  Language: Portuguese (pt-BR)
  All artifacts must be written in Brazilian Portuguese.
```

### 2.4 与其他 context 组合

```yaml
context: |
  Language: Chinese (zh-CN)
  所有 artifacts 必须使用简体中文。

  ## 项目上下文
  技术栈：Next.js 14 + TypeScript + TailwindCSS
  代码规范：组件文件使用 PascalCase，工具函数使用 camelCase
  测试要求：所有 API 必须有对应的 integration test
```

---

## 三、Schema 定制：自定义生成的文档组合

### 3.1 默认 Schema：`spec-driven`

OpenSpec 默认的 `spec-driven` Schema 按顺序生成：

```
proposal.md → specs/*.md → design.md → tasks.md
```

### 3.2 创建自定义 Schema

在 `openspec/schemas/` 目录下创建 YAML 文件，例如 `openspec/schemas/chinese-minimal.yaml`：

```yaml
name: chinese-minimal
description: 轻量中文变更流程，适用于小型需求

# 定义该 schema 生成的 artifacts 及其顺序
artifacts:
  - id: proposal
    filename: proposal.md
    description: 变更提案

  - id: spec
    filename: spec.md
    description: 需求规格（合并为一个文件）'
    single_file: true       # 合并为单个文件而非 specs/ 目录

  - id: tasks
    filename: tasks.md
    description: 实施检查清单
```

### 3.3 在 config.yaml 中设置默认 Schema

```yaml
default_schema: chinese-minimal

context: |
  Language: Chinese (zh-CN)
  所有 artifacts 必须使用简体中文。
```

之后运行 `openspec new <change-id>` 时，自动使用该 Schema。

### 3.4 常用 Schema 变体

**最小 Schema（仅 proposal + tasks）**：

```yaml
name: minimalist
description: 最小变更流程，仅包含提案和任务清单

artifacts:
  - id: proposal
    filename: proposal.md

  - id: tasks
    filename: tasks.md
```

**完整 Schema（含 design 审查）**：

```yaml
name: chinese-full
description: 完整中文变更流程，包含设计评审

artifacts:
  - id: proposal
    filename: proposal.md

  - id: specs
    description: 详细需求规格

  - id: design
    filename: design.md

  - id: tasks
    filename: tasks.md
```

---

## 四、模板定制：自定义文档格式和引导词

### 4.1 默认模板位置

OpenSpec 的内置 prompt 模板定义在各 skill 的 `SKILL.md` 中（位于 `~/.claude/skills/openspec-*/`）。模板内容由 AI 在生成文档时参考，可通过覆写 `AGENTS.md` 或 skill 文件来影响输出格式。

### 4.2 覆写 proposal.md 模板

在项目根目录的 `AGENTS.md`（或 `CLAUDE.md`）中添加自定义指令：

```markdown
# OpenSpec 文档生成规则

## proposal.md 格式要求

```markdown
# [变更名称]

## 背景与动机
- 为什么需要做这个变更？
- 解决了什么业务问题或技术痛点？

## 变更范围
### 包含
- ...

### 不包含
- ...

## 影响评估
- 性能影响：
- 安全影响：
- 向后兼容性：

## 成功标准
- [ ] 标准一
- [ ] 标准二
```

## design.md 格式要求

```markdown
# 技术设计方案：<变更名称>

## 概述
用一段话描述整体技术方案。

## 架构图
（在此处插入架构图或 Mermaid 图）

## 数据模型
| 字段 | 类型 | 说明 |
|------|------|------|
| ... | ... | ... |

## API 设计
### 接口列表
| 方法 | 路径 | 说明 |
|------|------|------|
| GET | /api/... | ... |

## 技术选型
- 依赖库/框架：
- 选型理由：

## 风险与对策
| 风险 | 应对措施 |
|------|---------|
| ... | ... |
```

## tasks.md 格式要求

```markdown
# 实施任务清单：<变更名称>

## 任务总览
- 预计工时：
- 优先级：
- 负责人：

## 实施步骤

### 阶段一：基础设施
- [ ] 任务 1.1
- [ ] 任务 1.2

### 阶段二：核心功能
- [ ] 任务 2.1
- [ ] 任务 2.2

### 阶段三：测试与部署
- [ ] 任务 3.1
- [ ] 任务 3.2
```

## 通用规则
- 所有文档必须使用中文（技术术语保留英文）
- 代码块必须标注语言：\`\`\`typescript
- 使用 Mermaid 语法绘制架构图
- 链接使用相对路径
```

### 4.3 自定义 spec.md（需求规格）格式

```markdown
## spec.md（需求规格）格式要求

```markdown
# 需求规格：<功能名称>

## 功能描述
一段话描述本功能要做什么。

## 用户故事
**作为** [角色]
**我希望** [功能]
**以便于** [收益]

## 验收标准（Acceptance Criteria）

### AC-1：[场景描述]
**Given** [前置条件]
**When** [操作]
**Then** [预期结果]

### AC-2：[场景描述]
...

## 功能性需求

### FR-1
- 描述：
- 优先级：P0 / P1 / P2

### FR-2
...

## 非功能性需求
- 性能：
- 可用性：
- 安全：

## 边界条件
- 正常流程：
- 异常流程：
- 极端情况：
```
```

---

## 五、完整配置示例

### 5.1 目录结构

```
项目根目录/
├── openspec/
│   ├── config.yaml              # 语言 + 默认 Schema 配置
│   ├── schemas/
│   │   └── chinese-full.yaml    # 自定义 Schema
│   └── specs/                   # 系统的 Source of Truth
├── openspec/
├── AGENTS.md                    # 自定义模板格式指令
└── src/
```

### 5.2 config.yaml 完整示例

```yaml
# openspec/config.yaml

# 设置默认 Schema
default_schema: chinese-full

# 语言和项目上下文
context: |
  Language: Chinese (zh-CN)
  所有 artifacts 必须使用简体中文。
  技术术语（API、REST、GraphQL、HTTP、JSON）保留英文。
  代码示例和文件路径保留原样。
  日期格式使用 YYYY-MM-DD。

  ## 项目上下文
  技术栈：Next.js 14 + TypeScript + TailwindCSS + Prisma
  代码规范：
  - 组件文件：PascalCase（如 UserProfile.tsx）
  - 工具函数：camelCase（如 formatDate.ts）
  - 样式文件：kebab-case（如 button-styles.css）
  测试要求：核心业务逻辑必须编写单元测试
```

### 5.3 自定义 Schema 完整示例

```yaml
# openspec/schemas/chinese-full.yaml

name: chinese-full
description: 完整中文变更流程（提案 → 规格 → 设计 → 任务）

artifacts:
  - id: proposal
    filename: proposal.md
    required: true

  - id: specs
    description: 需求规格（delta specs）
    required: true

  - id: design
    filename: design.md
    required: false
    enabled: true

  - id: tasks
    filename: tasks.md
    required: true
```

---

## 六、LobeHub 中文模板参考

**重要发现**：LobeHub 上已有开发者发布了**中文 OpenSpec 变更提案 Skill**（`openspec-new-change`），其模板要求：

> All generated documents (proposal, design, specs, tasks) must be written in Chinese

这证明中文模板需求是真实存在的，且已有社区实现。可作为模板设计的参考起点。

---

## 七、常见问题

### Q1：设置了语言但输出仍是英文？

**原因**：`config.yaml` 的 `context` 只是给 AI 的指令，不是强制约束。

**解决方法**：
1. 确认 `config.yaml` 位于项目根目录
2. 在 `AGENTS.md` 中明确强化语言指令
3. 在每次 `/opsx:propose` 时，在提示词中再次强调语言要求

### Q2：自定义模板不生效？

**原因**：OpenSpec 的 skill prompt 由 AI 执行，自定义模板是对 AI 的"引导"而非"强制"。

**解决方法**：
- 模板越具体、越结构化，AI 遵循度越高
- 使用完整的 Markdown 格式框架（包含占位符说明）
- 在 `AGENTS.md` 中用"必须"、"禁止"等强约束词汇

### Q3：Schema 和 config.yaml 的区别？

| 维度 | Schema | config.yaml |
|------|--------|------------|
| **控制内容** | 生成哪些文档、顺序 | 语言、默认 Schema、项目上下文 |
| **粒度** | artifacts 组合 | 全局行为 |
| **适用场景** | 不同类型变更用不同流程 | 统一规范 |

### Q4：如何让 AI 在 propose 时自动使用自定义模板？

在 `AGENTS.md` 的 `## OpenSpec` 部分加入：

```markdown
## OpenSpec 使用规范

每次执行 `/opsx:propose` 时：
1. proposal.md 按「## proposal.md 格式要求」生成
2. design.md 按「## design.md 格式要求」生成
3. tasks.md 按「## tasks.md 格式要求」生成
4. 所有内容使用简体中文
5. 禁止生成英文文档
```

---

## 八、进阶：结合 Superpowers 使用中文 Spec

当 OpenSpec 与 Superpowers 结合时，中文配置需要在**两层都设置**：

```
OpenSpec 层：openspec/config.yaml → Language: Chinese
    ↓ OpenSpec 生成中文 tasks.md
Superpowers 层：AGENTS.md → TDD prompt 使用中文
    ↓ Superpowers 按 tasks.md 执行 TDD
最终产出：中文注释 + 中文测试描述 + 中文代码
```

在 Superpowers 的 `AGENTS.md` 中添加：

```markdown
## TDD 规则（中文）

- 写失败的测试 → 测试描述使用中文
- 写最小实现 → 代码使用中文注释说明关键逻辑
- 重构 → 保持原有中文注释
```

---

## 参考资料

- [OpenSpec 官方文档 - Multi-Language Guide](https://github.com/Fission-AI/OpenSpec/blob/main/docs/multi-language.md)
- [OpenSpec 官方文档 - Customization Guide](https://github.com/Fission-AI/OpenSpec/blob/main/docs/customization.md)
- [LinkedIn - Customising OpenSpec workflow with schemas and config.yaml](https://www.linkedin.com/pulse/customising-openspec-workflow-schemas-configyaml-hari-krishnan--10fhc)
- [LobeHub - openspec-new-change Skill](https://lobehub.com/skills/sdotyxz-playspecg-openspec-new-change)
- [GitHub Issue #299 - Add Comprehensive Multilingual Support](https://github.com/Fission-AI/OpenSpec/issues/299)
- [GitHub Issue #614 - openspec init no longer creates AGENTS.md](https://github.com/Fission-AI/OpenSpec/issues/614)
