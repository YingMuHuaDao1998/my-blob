# LangChain 预置 Middleware 使用指南

> 来源：[[raw/01-articles/Prebuilt middleware]]
> 主题：LangChain / Deep Agents 内置中间件
> 整理时间：2026-04-23

## 概述

LangChain 与 Deep Agents 提供了一组**预置 Middleware（中间件）**，用于解决 Agent 在生产环境中的常见问题，例如：

- 上下文过长
- 工具调用失控
- 模型调用成本过高
- 缺少人工审批
- 敏感信息处理
- 长任务规划与上下文隔离

这些中间件的定位不是“锦上添花”，而是构建**高质量、可控、可恢复、可观测 Agent** 的基础设施。

---

## 一、什么是 Middleware

在 LangChain 里，Middleware 可以理解为：

- 在 **模型调用前后** 注入额外逻辑
- 在 **工具调用前后** 做控制、修改或拦截
- 在 **上下文流转过程中** 做裁剪、摘要、重试、脱敏、审批等处理

也就是说，Middleware 是 Agent 的“运行时治理层”。

---

## 二、Provider-agnostic Middleware（与模型厂商无关）

这类中间件可用于任意 LLM Provider。

| Middleware | 中文理解 | 主要作用 |
|------------|----------|----------|
| Summarization | 对话摘要 | 接近 token 上限时自动压缩历史对话 |
| Human-in-the-loop | 人在回路 | 工具执行前插入人工审批 |
| Model call limit | 模型调用限流 | 限制模型调用次数，防止死循环和超预算 |
| Tool call limit | 工具调用限流 | 限制工具调用次数 |
| Model fallback | 模型回退 | 主模型失败时自动切换到备用模型 |
| PII detection | 敏感信息检测 | 检测并处理个人敏感信息 |
| To-do list | 待办规划 | 给 Agent 加任务规划和进度跟踪能力 |
| LLM tool selector | LLM 工具选择器 | 先选工具，再让主模型工作 |
| Tool retry | 工具重试 | 工具失败后自动指数退避重试 |
| Model retry | 模型重试 | 模型调用失败后自动指数退避重试 |
| LLM tool emulator | 工具模拟器 | 用 LLM 模拟工具执行，便于测试 |
| Context editing | 上下文编辑 | 清理旧工具输出，压缩上下文 |
| Filesystem middleware | 文件系统中间件 | 给 Agent 提供文件系统用于上下文存储 |
| Subagent middleware | 子代理中间件 | 让主代理可以分派任务给子代理 |

---

## 三、重点 Middleware 详解

### 1. Summarization（摘要中间件）

**作用：**
当对话接近 token 上限时，自动把旧消息压缩成摘要，同时保留最近的若干消息。

**适用场景：**
- 长对话
- 多轮任务执行
- 需要保留会话脉络，但上下文窗口有限

**核心配置：**
- `trigger`：什么时候触发摘要
- `keep`：摘要后保留多少近期上下文
- `model`：谁来负责生成摘要

**实践理解：**
它解决的是“历史太长，但不能直接丢”的问题。

---

### 2. Human-in-the-loop（人工审批）

**作用：**
在工具真正执行前暂停，让人类进行：

- approve（批准）
- edit（修改）
- reject（拒绝）

**适用场景：**
- 发邮件
- 写数据库
- 金融交易
- 高风险生产操作

**关键点：**
需要 `checkpointer`，因为中断后要能恢复现场。

**实践理解：**
这类 Middleware 是 Agent 进入生产环境时最重要的安全阀之一。

---

### 3. Model call limit（模型调用限额）

**作用：**
限制 Agent 在一次运行或一个线程中的模型调用次数。

**适用场景：**
- 防止 ReAct / Loop 死循环
- 控制预算
- 压测与测试环境限额

**核心参数：**
- `threadLimit`：一个线程里总共最多调用几次
- `runLimit`：单次执行最多调用几次
- `exitBehavior`：超限后是结束还是报错

**实践理解：**
这是成本控制和防失控治理的关键中间件。

---

### 4. Tool call limit（工具调用限额）

**作用：**
限制所有工具或某个指定工具的调用次数。

**适用场景：**
- 限制昂贵 API
- 限制 web search、数据库查询次数
- 防止代理陷入搜索循环

**常见策略：**
- 对所有工具设全局上限
- 对某个工具单独设上限
- 超限后 `continue / error / end`

**实践理解：**
当 Agent 的工具很多时，这个中间件几乎是必备的。

---

### 5. Model fallback（模型回退）

**作用：**
当主模型失败时，自动切换到备用模型。

**适用场景：**
- 主模型超时或不可用
- 多厂商容灾
- 成本优化（失败后回退到更便宜模型）

**实践理解：**
它提升的是系统韧性，不是单纯提升效果。

---

### 6. PII detection（敏感信息检测）

**作用：**
检测对话中的个人敏感信息，并按策略处理。

**支持策略：**
- `block`：发现就报错阻断
- `redact`：直接替换为脱敏标记
- `mask`：部分打码
- `hash`：替换成稳定哈希

**内置类型：**
- email
- credit_card
- ip
- mac_address
- url

**自定义方式：**
- 正则字符串
- `RegExp`
- 自定义 detector 函数

**实践理解：**
这在金融、医疗、客服系统中尤其重要。

---

### 7. To-do list（待办规划）

**作用：**
自动给 Agent 注入 `write_todos` 工具和相应提示词，让 Agent 能把复杂任务拆成待办并追踪进度。

**适用场景：**
- 多步骤任务
- 长运行任务
- 需要让用户可见任务进度

**实践理解：**
它其实是把“规划能力”前置出来，让 Agent 更像真正的任务执行器。

---

### 8. LLM tool selector（LLM 工具选择器）

**作用：**
在调用主模型前，先用一个较小模型挑选“这次真正相关的工具”。

**适用场景：**
- 工具很多（10+）
- 大部分工具对当前问题无关
- 希望减少 token 消耗

**关键参数：**
- `model`：用于选工具的模型
- `maxTools`：最多保留几个工具
- `alwaysInclude`：总是保留的工具

**实践理解：**
这是一种典型的 Context Engineering：先裁剪工具上下文，再做主推理。

---

### 9. Tool retry / Model retry（重试中间件）

**作用：**
在工具调用或模型调用失败时自动重试，并支持指数退避。

**关键参数：**
- `maxRetries`
- `backoffFactor`
- `initialDelayMs`
- `maxDelayMs`
- `jitter`
- `retryOn`
- `onFailure`

**适用场景：**
- 网络抖动
- 外部 API 临时失败
- 模型服务 429 / 503

**实践理解：**
这不是为了掩盖真正错误，而是为了提高系统对短暂故障的恢复能力。

---

### 10. LLM tool emulator（工具模拟器）

**作用：**
不真正执行工具，而是让 LLM 模拟工具返回结果。

**适用场景：**
- 开发期原型验证
- 外部工具不可用
- 工具真实调用成本过高
- 想测试 Agent 工作流而不依赖真实系统

**实践理解：**
它非常适合在“先验证流程，再接真实工具”的阶段使用。

---

### 11. Context editing（上下文编辑）

**作用：**
当上下文过长时，自动清理旧的工具输出，只保留最近若干条工具结果。

**典型能力：**
- 监控 token 数
- 清除旧工具结果
- 保留最近 N 条
- 可选清除工具输入参数
- 为被清除内容填充 placeholder

**与 Summarization 的区别：**
- Summarization：把旧内容压缩成摘要
- Context editing：直接清理旧工具输出

**实践理解：**
这更适合“工具输出很长但历史价值不高”的场景。

---

### 12. Filesystem middleware（文件系统中间件）

**作用：**
给 Agent 提供一个可读写文件系统，用来存放中间结果、上下文和长期记忆。

**内置工具：**
- `ls`
- `read_file`
- `write_file`
- `edit_file`

**核心价值：**
解决工具返回结果长度不可控的问题。比如：
- Web Search 结果很长
- RAG 检索内容很多
- 中间分析结果不适合一直塞在上下文里

**短期 / 长期文件系统：**
- 默认写入 graph state 中，属于短期上下文
- 配合 `CompositeBackend + StoreBackend` 可将 `/memories/` 下路径持久化为长期记忆

**实践理解：**
这是 Deep Agents 在 Context Engineering 上非常关键的一层。

---

### 13. Subagent middleware（子代理中间件）

**作用：**
允许主代理通过 `task` 工具把任务转交给子代理。

**子代理定义通常包含：**
- `name`
- `description`
- `systemPrompt`
- `tools`
- `model`
- `middleware`

**核心价值：**
- 隔离上下文
- 让主代理专注调度
- 让子代理专注深入某项任务

**额外特性：**
主代理默认还有一个 `general-purpose` 子代理，可用来做通用任务委派，减少主代理上下文膨胀。

**实践理解：**
这类中间件本质上服务于多代理架构与上下文隔离。

---

## 四、Provider-specific Middleware

文章最后还提到 **Provider-specific middleware（厂商专属中间件）**。

当前页面给出的示例是：

### Anthropic

为 Claude 模型提供：
- Prompt caching
- bash tool
- text editor
- memory
- file search

这说明 LangChain 的 Middleware 体系有两层：

1. **通用中间件**：所有 provider 都可用
2. **厂商专属中间件**：针对某个模型能力做优化

---

## 五、如何选择这些 Middleware

可以按目标来选：

### 1. 如果你想解决“上下文过长”
优先考虑：
- `summarizationMiddleware`
- `contextEditingMiddleware`
- `createFilesystemMiddleware`

### 2. 如果你想解决“安全与可控”
优先考虑：
- `humanInTheLoopMiddleware`
- `modelCallLimitMiddleware`
- `toolCallLimitMiddleware`
- `piiMiddleware`

### 3. 如果你想解决“稳定性与容错”
优先考虑：
- `modelFallbackMiddleware`
- `modelRetryMiddleware`
- `toolRetryMiddleware`

### 4. 如果你想解决“复杂任务协作”
优先考虑：
- `todoListMiddleware`
- `llmToolSelectorMiddleware`
- `createSubAgentMiddleware`

---

## 六、我的理解：这篇文章的核心价值

这篇文章最重要的价值，不是单独介绍某个 API，而是揭示了一个更重要的事实：

> **高质量 Agent = 模型能力 + 工具能力 + Middleware 治理能力**

Middleware 在这里扮演的是 **Harness / Runtime Governance** 的角色，它决定了 Agent 是否：

- 可控
- 稳定
- 成本可管理
- 上下文不会迅速爆炸
- 能适应真实生产环境

所以从工程角度看，Middleware 不是“附属功能”，而是 Agent 系统从 Demo 走向 Production 的关键层。

---

## 七、与现有知识的关联

- [[LangChain/LangChain学习总结]]
- [[LangChain/LangChain最佳实践]]
- [[LangChain/LangChain生态三件套使用指南]]
- [[AgentFramework/LangChain+LangGraph高质量多子代理智能体设计文档]]
- [[wiki/concepts/Agent Harness]]
- [[wiki/concepts/Harness Engineering]]
- [[wiki/concepts/Agent 可控性]]

---

## 八、可直接记住的结论

1. **Summarization / Context editing / Filesystem** 主要解决上下文问题。  
2. **Human-in-the-loop / Limit / PII** 主要解决安全与治理问题。  
3. **Retry / Fallback** 主要解决稳定性问题。  
4. **To-do / Tool selector / Subagent** 主要解决复杂任务组织问题。  
5. Middleware 是 Agent 从“能跑”走向“可生产”的关键层。
