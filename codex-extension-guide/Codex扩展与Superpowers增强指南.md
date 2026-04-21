# Codex 扩展与 Superpowers 增强指南

> 基于 Codex 配置实践总结，帮助你扩展和增强 Codex 的功能

## 目录

- [概述](#概述)
- [Codex 配置结构](#codex-配置结构)
- [AGENTS.md 全局指令](#agentsmd-全局指令)
- [Superpowers 架构](#superpowers-架构)
- [创建自定义技能](#创建自定义技能)
- [技能测试方法](#技能测试方法)
- [最佳实践](#最佳实践)
- [实际案例](#实际案例)
- [常见问题](#常见问题)

---

## 概述

Codex 是一个强大的编码代理工具，通过以下方式可以扩展和增强其功能：

1. **全局指令 (AGENTS.md)** - 设置全局行为偏好
2. **技能系统 (Skills)** - 添加特定任务的处理流程
3. **Superpowers 插件** - 一套完整的软件开发工作流
4. **MCP 服务器** - 集成外部工具和服务

本指南将帮助你理解这些扩展机制，并创建适合自己的增强配置。

---

## Codex 配置结构

### 目录结构

```
~/.codex/                          # Codex 主目录
├── AGENTS.md                      # 全局指令（自定义）
├── config.toml                    # 主配置文件
├── auth.json                      # 认证信息
├── history.jsonl                  # 历史记录
├── sessions/                      # 会话数据
├── skills/                        # 自定义技能（可选）
├── superpowers/                   # Superpowers 插件
└── .codex-global-state.json       # 全局状态

~/.agents/skills/                  # 技能发现目录
└── superpowers -> ~/.codex/superpowers/skills  # 符号链接
```

### 配置文件优先级

```
项目级 > 用户级 > 默认值
```

1. **项目级**: `项目目录/.codex/AGENTS.md` 或 `项目目录/AGENTS.md`
2. **用户级**: `~/.codex/AGENTS.md`
3. **默认值**: Codex 内置行为

### config.toml 配置示例

```toml
# 人格设置
personality = "pragmatic"
model_provider = "custom"
model = "gpt-5.4"
model_reasoning_effort = "high"

# MCP 服务器配置
[mcp_servers.pencil]
command = "/Applications/Pencil.app/Contents/Resources/app.asar.unpacked/out/mcp-server-darwin-arm64"
args = [ "--app", "desktop" ]

# 自定义模型提供商
[model_providers.custom]
name = "custom"
wire_api = "responses"
requires_openai_auth = true
base_url = "https://api.example.com/v1"

# 项目信任级别
[projects."/path/to/trusted/project"]
trust_level = "trusted"
```

---

## AGENTS.md 全局指令

### 什么是 AGENTS.md？

AGENTS.md 是 Codex 启动时自动读取的指令文件，用于设置全局行为偏好。每次会话都会加载这些指令，作为基础上下文。

### 创建 AGENTS.md

```bash
# 创建用户级全局指令
touch ~/.codex/AGENTS.md

# 或创建项目级指令
touch 项目目录/AGENTS.md
```

### 基本结构

```markdown
# AGENTS.md - Codex 全局指令

## 语言偏好
[设置代码注释语言、文档语言等]

## 代码风格
[设置编码规范、命名约定等]

## 工作流程
[设置特定的开发流程要求]

## 工具偏好
[设置工具使用偏好]
```

### 常用配置示例

#### 1. 中文注释配置

```markdown
# AGENTS.md - Codex 全局指令

## 语言偏好

**所有生成的代码必须使用中文注释。**

包括但不限于：
- 函数注释/文档字符串
- 行内注释
- README 和文档
- 提交信息（commit messages）

### 示例

```python
def calculate_total(items: list) -> float:
    """计算订单总金额
    
    Args:
        items: 订单商品列表
        
    Returns:
        总金额
    """
    total = 0.0
    for item in items:
        # 累加每件商品的价格
        total += item.price * item.quantity
    return total
```

## 代码风格

- 注释使用中文
- 变量名和函数名使用英文（保持代码可读性）
- 文档字符串使用中文
- 错误消息使用中文（除非是技术性错误）
```

#### 2. 测试驱动开发配置

```markdown
# AGENTS.md - TDD 强化指令

## 开发流程

**强制执行测试驱动开发 (TDD)**

1. 先写失败的测试
2. 运行测试确认失败
3. 编写最小代码使测试通过
4. 运行测试确认通过
5. 重构代码
6. 重复

### 禁止行为

- ❌ 在没有测试的情况下编写生产代码
- ❌ 跳过测试"仅此一次"
- ❌ 保留未测试的代码

### 测试覆盖率要求

- 新功能：必须有测试
- Bug 修复：必须先写回归测试
- 重构：必须有现有测试保护
```

#### 3. Git 提交规范配置

```markdown
# AGENTS.md - Git 提交规范

## 提交信息格式

```
<类型>(<范围>): <简短描述>

[可选的详细描述]

[可选的注脚]
```

### 类型

- `feat`: 新功能
- `fix`: Bug 修复
- `docs`: 文档变更
- `style`: 代码格式（不影响功能）
- `refactor`: 重构
- `test`: 测试相关
- `chore`: 构建/工具变更

### 示例

```
feat(auth): 添加用户登录功能

- 实现 JWT 认证
- 添加登录/登出 API
- 添加单元测试

Closes #123
```

## 提交规则

1. 每次提交只做一件事
2. 提交信息使用中文描述（类型和范围用英文）
3. 提交前运行测试
4. 不提交敏感信息
```

#### 4. 项目特定配置

```markdown
# AGENTS.md - 项目特定配置

## 项目信息

- 项目名称: MyAwesomeProject
- 技术栈: React + TypeScript + Node.js
- 数据库: PostgreSQL
- 部署: Docker + Kubernetes

## 架构约定

### 目录结构
```
src/
├── components/     # React 组件
├── hooks/          # 自定义 Hooks
├── services/       # API 服务
├── utils/          # 工具函数
└── types/          # TypeScript 类型定义
```

### 命名约定

- 组件: PascalCase (e.g., `UserProfile.tsx`)
- 函数: camelCase (e.g., `fetchUserData`)
- 常量: UPPER_SNAKE_CASE (e.g., `API_BASE_URL`)
- 文件: 与主要导出项同名

### API 调用

使用 `src/services/apiClient.ts` 中的统一 API 客户端

```typescript
// 正确 ✓
import { apiClient } from '@/services/apiClient';

const data = await apiClient.get('/users/123');

// 错误 ✗
const data = await fetch('https://api.example.com/users/123');
```

## 代码审查清单

在提交代码前检查：

- [ ] 所有测试通过
- [ ] 代码符合项目风格
- [ ] 添加了必要的注释
- [ ] 更新了相关文档
- [ ] 没有调试代码遗留
```

### AGENTS.md 最佳实践

1. **保持简洁**: 每条指令都应该有价值，避免冗余
2. **使用示例**: 好的示例比长篇说明更有效
3. **分层次**: 用户级设置通用偏好，项目级设置特定规则
4. **定期维护**: 随着项目发展更新指令
5. **避免冲突**: 确保项目级和用户级指令不矛盾

---

## Superpowers 架构

### 什么是 Superpowers？

Superpowers 是一套完整的软件开发工作流系统，基于"技能"(Skills)构建，为编码代理提供：

- 需求分析和设计流程
- 测试驱动开发 (TDD)
- 代码审查流程
- Git 工作流自动化
- 子代理协作

### 安装 Superpowers

```bash
# 1. 克隆 Superpowers 仓库
git clone https://github.com/obra/superpowers.git ~/.codex/superpowers

# 2. 创建技能目录和符号链接
mkdir -p ~/.agents/skills
ln -s ~/.codex/superpowers/skills ~/.agents/skills/superpowers

# 3. 重启 Codex 让其发现新技能
```

### 核心工作流程

```
1. brainstorming (头脑风暴)
   └── 理解需求，探索方案，生成设计文档

2. using-git-worktrees (Git 工作树)
   └── 创建隔离的工作分支，验证测试基线

3. writing-plans (编写计划)
   └── 将设计拆分成小任务（2-5分钟/任务）

4. subagent-driven-development (子代理驱动开发)
   └── 分派子代理执行任务，进行代码审查

5. test-driven-development (测试驱动开发)
   └── 强制执行 RED-GREEN-REFACTOR 循环

6. requesting-code-review (代码审查请求)
   └── 任务间的代码审查

7. finishing-a-development-branch (完成开发分支)
   └── 合并、创建 PR 或清理工作树
```

### 技能目录结构

```
~/.codex/superpowers/skills/
├── brainstorming/
│   └── SKILL.md              # 需求分析和设计
├── writing-plans/
│   └── SKILL.md              # 编写实施计划
├── test-driven-development/
│   └── SKILL.md              # TDD 流程
├── systematic-debugging/
│   └── SKILL.md              # 系统性调试
├── writing-skills/
│   ├── SKILL.md              # 如何编写技能
│   ├── anthropic-best-practices.md
│   ├── testing-skills-with-subagents.md
│   └── examples/
├── using-git-worktrees/
│   └── SKILL.md
├── subagent-driven-development/
│   └── SKILL.md
└── ... 其他技能
```

### 更新 Superpowers

```bash
cd ~/.codex/superpowers && git pull
```

由于使用符号链接，更新后技能立即生效，无需重启 Codex。

---

## 创建自定义技能

### 技能是什么？

技能是编码代理的参考指南，帮助代理：

- 应用经过验证的技术和模式
- 遵循最佳实践
- 保持一致性

**技能是：**
- 可重用的技术、模式、工具
- 参考指南
- 流程文档

**技能不是：**
- 一次性解决方案的描述
- 标准实践的重述
- 项目特定的约定（这些应该放在 AGENTS.md）

### 何时创建技能

**应该创建：**
- 技术对你来说不够直观
- 你会在多个项目中参考
- 模式适用广泛
- 其他人也会受益

**不应该创建：**
- 一次性解决方案
- 其他地方已文档化的标准实践
- 项目特定约定
- 可用正则/验证器强制执行的约束

### 技能类型

| 类型 | 描述 | 示例 |
|------|------|------|
| **Technique** | 具体的方法步骤 | condition-based-waiting, root-cause-tracing |
| **Pattern** | 思考问题的方式 | flatten-with-flags, test-invariants |
| **Reference** | API 文档、语法指南 | office docs, framework API |

### 技能目录结构

```
skills/
└── skill-name/
    ├── SKILL.md              # 主文档（必需）
    ├── supporting-file.*     # 支持文件（可选）
    └── examples/             # 示例（可选）
```

### SKILL.md 结构

#### Frontmatter

```yaml
---
name: skill-name                    # 技能名称（字母、数字、连字符）
description: Use when ...           # 描述何时使用（第三人称）
---
```

**限制：**
- Frontmatter 总长度 ≤ 1024 字符
- `description` 应该描述"何时使用"，不是"做什么"
- 以 "Use when..." 开头，关注触发条件

#### 文档主体

```markdown
# Skill Name

## Overview
[简要概述，解释这个技能解决什么问题]

## When to Use
[触发条件，什么情况下应该使用这个技能]

## Core Principles
[核心原则，简明扼要]

## Process
[具体步骤，清晰可执行]

## Examples
[代码示例，展示最佳实践]

## Anti-Patterns
[常见错误，需要避免]

## References
[参考链接，延伸阅读]
```

### 创建技能示例

#### 示例 1：API 文档技能

```markdown
---
name: api-documentation
description: Use when documenting REST APIs, creating API reference docs, or writing endpoint documentation
---

# API Documentation

## Overview

为 REST API 创建清晰、一致、易于理解的文档。

## When to Use

- 新增 API 端点
- 更新 API 行为
- 为团队创建 API 参考文档

## Endpoint Documentation Format

每个端点应包含：

### 基本信息

```markdown
## Endpoint Name

`METHOD /api/path/{param}`

Brief description of what this endpoint does.

### Parameters

| Name | Type | In | Required | Description |
|------|------|-----|----------|-------------|
| param | string | path | Yes | Description |
| field | number | body | No | Description |

### Response

```json
{
  "status": "success",
  "data": { }
}
```

### Status Codes

| Code | Meaning | When |
|------|---------|------|
| 200 | OK | Successful request |
| 400 | Bad Request | Invalid parameters |
| 404 | Not Found | Resource not found |

### Examples

```bash
curl -X POST https://api.example.com/endpoint \
  -H "Content-Type: application/json" \
  -d '{"field": "value"}'
```
```

## Best Practices

1. **一致性**: 所有端点使用相同格式
2. **完整性**: 包含所有必需参数和响应码
3. **示例**: 提供可运行的示例
4. **版本**: 标注 API 版本

## Anti-Patterns

- ❌ 过时或不准确的文档
- ❌ 缺少错误响应示例
- ❌ 没有实际的请求示例
```

#### 示例 2：React 组件技能

```markdown
---
name: react-component-patterns
description: Use when creating React components, refactoring component structure, or establishing component conventions
---

# React Component Patterns

## Overview

定义 React 组件的标准模式和最佳实践。

## When to Use

- 创建新组件
- 重构现有组件
- 建立团队组件规范

## Component Structure

```typescript
// 1. 导入
import React from 'react';
import { useQuery } from '@tanstack/react-query';

// 2. 类型定义
interface Props {
  userId: string;
  onUserUpdate?: (user: User) => void;
}

interface User {
  id: string;
  name: string;
  email: string;
}

// 3. 组件
export function UserProfile({ userId, onUserUpdate }: Props) {
  // 3.1 Hooks
  const { data: user, isLoading, error } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
  });

  // 3.2 事件处理
  const handleClick = () => {
    if (user && onUserUpdate) {
      onUserUpdate(user);
    }
  };

  // 3.3 渲染
  if (isLoading) return <LoadingSpinner />;
  if (error) return <ErrorMessage error={error} />;
  
  return (
    <div className="user-profile">
      <h2>{user.name}</h2>
      <button onClick={handleClick}>
        更新用户信息
      </button>
    </div>
  );
}

// 4. 辅助函数
async function fetchUser(id: string): Promise<User> {
  const response = await fetch(`/api/users/${id}`);
  if (!response.ok) throw new Error('Failed to fetch user');
  return response.json();
}
```

## Naming Conventions

- **组件**: PascalCase (`UserProfile.tsx`)
- **Props 接口**: `Props`
- **事件处理**: `handle` + 动作 (`handleClick`, `handleSubmit`)
- **布尔状态**: `is`/`has` 前缀 (`isLoading`, `hasError`)

## Component Types

| 类型 | 用途 | 示例 |
|------|------|------|
| 展示组件 | 只渲染 UI | `Button`, `Card` |
| 容器组件 | 处理逻辑和状态 | `UserProfileContainer` |
| 高阶组件 | 复用逻辑 | `withAuth` |
| 自定义 Hook | 复用状态逻辑 | `useUserProfile` |

## Best Practices

1. **单一职责**: 每个组件只做一件事
2. **Props 向下，事件向上**: 单向数据流
3. **条件渲染优先**: 使用三元运算符和 `&&`
4. **避免过度优化**: 只在需要时优化性能
```

### 安装自定义技能

```bash
# 方法 1: 直接放在技能目录
mkdir -p ~/.agents/skills/my-skills
cp -r my-skill ~/.agents/skills/my-skills/

# 方法 2: 创建符号链接
ln -s /path/to/my-skills ~/.agents/skills/my-skills

# 方法 3: 项目级技能
mkdir -p 项目目录/.codex/skills
cp -r my-skill 项目目录/.codex/skills/
```

---

## 技能测试方法

### 测试驱动技能开发

技能开发遵循 TDD 原则：

```
RED → GREEN → REFACTOR
```

| TDD 阶段 | 技能测试 | 说明 |
|----------|----------|------|
| **RED** | 基线测试 | 不加载技能运行场景，观察代理失败 |
| **GREEN** | 编写技能 | 针对失败编写技能 |
| **REFACTOR** | 压力测试 | 寻找漏洞，加强技能 |

### 测试流程

#### 1. 基线测试 (RED)

**目标**: 运行场景 WITHOUT 技能，观察代理失败

```markdown
## 基线场景

场景: 用户要求"快速添加一个功能"

### 预期失败模式

1. 代理直接开始编写代码
2. 跳过测试"节省时间"
3. 不考虑边缘情况
4. 提交未测试的代码

### 测试命令

在不加载技能的情况下运行：

```
请帮我快速添加一个用户登录功能
```

### 记录观察

- 代理是否跳过了设计？
- 代理是否写了测试？
- 代理是否考虑了安全性？
```

#### 2. 编写技能 (GREEN)

**目标**: 针对基线失败编写技能

技能应该：
- 直接解决观察到的失败模式
- 清晰陈述规则
- 提供反例

#### 3. 压力测试 (REFACTOR)

**目标**: 寻找技能的漏洞

```markdown
## 压力场景

尝试各种"绕过"技能的方式：

1. "这很紧急，跳过测试吧"
2. "这只是原型，不需要文档"
3. "我知道这不符合规范，但是..."
4. "用户明确要求快速完成"

### 预期

技能应该强制执行规则，即使有压力也不妥协
```

### 何时测试技能

**应该测试：**
- 强制纪律的技能 (TDD, 测试要求)
- 有合规成本的技能 (时间、精力、返工)
- 可能被合理化绕过的技能 ("仅此一次")
- 与即时目标冲突的技能 (速度 vs 质量)

**不需要测试：**
- 纯参考技能 (API 文档, 语法指南)
- 没有规则可违反的技能
- 代理没有动机绕过的技能

---

## 最佳实践

### 1. 简洁原则

上下文窗口是公共资源，每个 token 都有成本。

**好的例子 (约 50 tokens):**

```markdown
## 提取 PDF 文本

使用 pdfplumber：

```python
import pdfplumber

with pdfplumber.open("file.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```
```

**不好的例子 (约 150 tokens):**

```markdown
## 提取 PDF 文本

PDF (Portable Document Format) 文件是一种常见的文件格式，包含文本、图像和其他内容。
要从 PDF 中提取文本，你需要使用一个库。有很多 PDF 处理库可用，但我们推荐
pdfplumber，因为它易于使用，并且能很好地处理大多数情况。首先，你需要使用
pip 安装它。然后你可以使用下面的代码...
```

### 2. 适当的自由度

**高自由度 (文本指令):**

用于：
- 多种方法都有效
- 决策依赖上下文
- 启发式方法

```markdown
## 代码审查流程

1. 分析代码结构和组织
2. 检查潜在 bug 或边缘情况
3. 建议可读性和可维护性改进
4. 验证是否符合项目约定
```

**低自由度 (详细规范):**

用于：
- 容易出错的任务
- 需要精确性
- 关键路径

```markdown
## Git 提交格式

必须使用以下格式：

```
<类型>(<范围>): <描述>

[可选正文]

[可选注脚]
```

类型必须是: feat, fix, docs, style, refactor, test, chore
```

### 3. 提供示例

好的示例比长篇说明更有效。

```markdown
## 正确 ✓

```python
def calculate_discount(price: float, rate: float) -> float:
    """计算折扣后价格
    
    Args:
        price: 原价
        rate: 折扣率 (0.0-1.0)
        
    Returns:
        折扣后价格
    """
    if not 0 <= rate <= 1:
        raise ValueError("折扣率必须在 0 和 1 之间")
    return price * (1 - rate)
```

## 错误 ✗

```python
def calc(p, r):
    return p * (1 - r)  # 缺少类型注解、文档字符串和验证
```
```

### 4. 说明负面模式

告诉代理不要做什么。

```markdown
## 避免的做法

### ❌ 在没有测试的情况下编写代码

```python
# 错误: 没有测试就写实现
def process_data(data):
    return [item * 2 for item in data]
```

### ❌ 保留调试代码

```python
# 错误: 提交调试打印语句
def process_data(data):
    print(f"Processing {len(data)} items")  # 删除这个!
    return [item * 2 for item in data]
```
```

### 5. 分层配置

```
用户级 AGENTS.md
├── 通用偏好 (语言、风格)
├── 常用工具配置
└── 适用于所有项目的规则

项目级 AGENTS.md
├── 项目特定约定
├── 技术栈相关规则
└── 覆盖用户级设置
```

### 6. 定期维护

- **更新技能**: 随着工具和最佳实践变化
- **删除过时**: 不再相关的技能
- **优化简洁性**: 定期精简冗余内容

---

## 实际案例

### 案例 1: 添加中文注释支持

**问题**: Superpowers 生成的代码注释是英文

**解决方案**: 创建 `~/.codex/AGENTS.md`

```markdown
# AGENTS.md - Codex 全局指令

## 语言偏好

**所有生成的代码必须使用中文注释。**

包括但不限于：
- 函数注释/文档字符串
- 行内注释
- README 和文档
- 提交信息（commit messages）

## 代码风格

- 注释使用中文
- 变量名和函数名使用英文（保持代码可读性）
- 文档字符串使用中文
- 错误消息使用中文（除非是技术性错误）
```

**效果**: Codex 现在会生成中文注释的代码

### 案例 2: 创建项目特定技能

**需求**: 为 React + TypeScript 项目创建组件模板技能

**步骤**:

1. 创建技能目录

```bash
mkdir -p ~/.agents/skills/my-skills/react-component-template
```

2. 创建 SKILL.md

```markdown
---
name: react-component-template
description: Use when creating new React components in TypeScript projects
---

# React Component Template

## Overview

定义项目中 React 组件的标准模板和约定。

## Component Template

```typescript
// 导入
import React from 'react';
import styles from './ComponentName.module.css';

// 类型定义
interface Props {
  // 定义 props
}

// 组件
export function ComponentName({ }: Props) {
  // Hooks 在这里
  
  // 事件处理在这里
  
  // 渲染
  return (
    <div className={styles.container}>
      {/* 内容 */}
    </div>
  );
}
```

## File Structure

```
ComponentName/
├── ComponentName.tsx       # 组件
├── ComponentName.module.css # 样式
├── ComponentName.test.tsx   # 测试
└── index.ts                 # 导出
```

## Naming Rules

- 组件: PascalCase
- 文件: 与组件同名
- Props 接口: `Props`
- 样式: CSS Modules
```

3. 使用技能

Codex 会自动发现并使用这个技能，当创建 React 组件时。

### 案例 3: 强化 TDD 流程

**问题**: 代理有时跳过测试

**解决方案**: 创建 TDD 强化技能

```markdown
---
name: strict-tdd
description: Use when implementing features, fixing bugs, or any code changes. Activates BEFORE any code is written.
---

# Strict Test-Driven Development

## Iron Law

```
没有失败测试先行的代码 = 0 行代码
```

## RED-GREEN-REFACTOR 循环

### 1. RED: 写失败测试

```python
def test_user_login():
    # 这个测试 MUST 失败
    user = create_user("test@example.com", "password123")
    result = login("test@example.com", "password123")
    assert result.success == True
```

运行测试，确认失败。

### 2. GREEN: 写最小代码

```python
def login(email, password):
    # 只写让测试通过的最小代码
    return LoginResult(success=True)
```

运行测试，确认通过。

### 3. REFACTOR: 重构

改进代码，保持测试通过。

## 强制规则

1. **看到测试失败**: 必须运行测试并看到失败
2. **最小实现**: 只写让测试通过的最小代码
3. **删除未测试代码**: 如果代码写在了测试之前，删除重来

## 禁止行为

- ❌ "这很简单，不需要测试"
- ❌ "我之后会补测试"
- ❌ "这只是原型"
- ❌ "时间紧急，跳过测试"

所有这些都是合理的借口，但测试先行是不可妥协的。
```

---

## 常见问题

### Q1: AGENTS.md 和 SKILL.md 有什么区别？

| 特性 | AGENTS.md | SKILL.md |
|------|-----------|----------|
| **作用** | 全局行为偏好 | 特定任务的处理流程 |
| **范围** | 所有会话 | 特定触发条件 |
| **位置** | `~/.codex/` 或项目根目录 | `~/.agents/skills/` |
| **内容** | 偏好设置、风格约定 | 流程、步骤、示例 |
| **示例** | "使用中文注释" | "如何实现 TDD" |

### Q2: 如何让技能在特定条件下触发？

在 `description` 字段中明确定义触发条件：

```yaml
---
name: api-security-review
description: Use when creating or modifying API endpoints, especially those handling authentication, authorization, or sensitive data
---
```

### Q3: 技能更新后需要重启 Codex 吗？

**不需要**。由于使用符号链接，技能文件更新后立即生效。

但如果修改了 `AGENTS.md`，需要重启 Codex 或开始新会话。

### Q4: 如何调试技能不被触发？

1. **检查文件位置**: 确保在 `~/.agents/skills/` 目录下
2. **检查文件名**: 必须是 `SKILL.md`（大写）
3. **检查 frontmatter**: `name` 和 `description` 必须正确
4. **检查符号链接**: `ls -la ~/.agents/skills/`

```bash
# 验证技能目录
ls -la ~/.agents/skills/

# 验证符号链接
ls -la ~/.agents/skills/superpowers
```

### Q5: Superpowers 和自定义技能可以共存吗？

**可以**。它们都在 `~/.agents/skills/` 目录下，Codex 会同时发现和使用它们。

```
~/.agents/skills/
├── superpowers -> ~/.codex/superpowers/skills
└── my-skills/
    └── my-custom-skill/
        └── SKILL.md
```

### Q6: 如何更新 Superpowers？

```bash
cd ~/.codex/superpowers && git pull
```

由于使用符号链接，更新后立即生效。

### Q7: 如何临时禁用某个技能？

重命名 `SKILL.md` 文件：

```bash
mv ~/.agents/skills/my-skills/my-skill/SKILL.md ~/.agents/skills/my-skills/my-skill/SKILL.md.disabled
```

要重新启用，改回原名即可。

### Q8: 技能可以有支持文件吗？

**可以**。技能目录可以包含支持文件：

```
skills/
└── my-skill/
    ├── SKILL.md              # 主文档（必需）
    ├── reference.md          # 详细参考
    ├── examples.md           # 更多示例
    └── templates/
        └── component.ts      # 模板文件
```

在 `SKILL.md` 中引用：

```markdown
## 模板

参考 `templates/component.ts` 获取完整模板。
```

### Q9: 项目级 AGENTS.md 会覆盖用户级吗？

**会**。项目级设置优先于用户级：

```
优先级: 项目级 > 用户级 > 默认值
```

建议：
- 用户级: 设置通用偏好（语言、风格）
- 项目级: 设置项目特定约定（技术栈、架构）

### Q10: 如何知道哪些技能被加载了？

Codex 启动时会预加载所有技能的元数据（name 和 description）。你可以：

```bash
# 列出所有技能
ls ~/.agents/skills/
ls ~/.agents/skills/*/
```

或在对话中询问 Codex："你有哪些可用的技能？"

---

## 参考资料

### 官方文档

- [Codex Documentation](https://github.com/openai/codex)
- [Superpowers GitHub](https://github.com/obra/superpowers)
- [Agent Skills Overview](https://platform.claude.com/docs/agents-and-tools/agent-skills/overview)

### 相关技能

在 Superpowers 中查看：

- `writing-skills` - 如何编写技能
- `testing-skills-with-subagents` - 如何测试技能
- `anthropic-best-practices` - 技能编写最佳实践

### 社区

- [Superpowers Discord](https://discord.gg/Jd8Vphy9jq)
- [GitHub Issues](https://github.com/obra/superpowers/issues)

---

## 更新日志

- **2024-03-28**: 初始版本，基于 Codex 扩展实践创建
  - 添加 AGENTS.md 全局指令
  - 创建自定义技能
  - Superpowers 集成

---

> 本指南基于实际使用经验总结，会随着工具更新而持续改进。