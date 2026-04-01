# AI Agent 开发实践指南：经典范式与实例

> 当前时间：2026-04-01 | 覆盖 ReAct/ReWOO/Reflection/Planning 等核心范式 + 多 Agent 架构 + 主流框架 + 生产实践

---

## 一、Agent 的核心循环

所有 AI Agent 的本质都是一个**控制循环**：

```
用户输入
    ↓
 Reason（推理）→ 决定下一步做什么
    ↓
 Act（行动）→ 调用工具 / 生成文本 / 调用其他 Agent
    ↓
 Observe（观察）→ 收集行动结果
    ↓
 循环直到达成目标或达到最大步数
```

区别不同 Agent 范式，就是区别**这个循环的组织方式**。

---

## 二、经典单 Agent 范式

### 范式一：ReAct（Reasoning + Acting）

**思想**：Thought → Action → Observation 三环紧耦合，每一步都必须先想再做再看结果。

**流程**：
```
Thought: 用户问的是天气，我需要先查北京当前温度
Action: call weather_tool(city=北京)
Observation: 温度 18°C，晴
Thought: 温度适中，适合户外活动
Action: 生成回复文本
```

**适用场景**：需要**实时外部数据**的任务（搜索、查数据库、API 调用）。

**优缺点**：
- ✅ 推理过程可追溯，每步都有日志
- ✅ 对复杂多步任务友好
- ❌ 每次 Tool call 都有延迟，成本较高
- ❌ 长的推理链容易中间"跑偏"

**经典框架实现**：LangChain 的 `agent_executor`、CrewAI 的默认 Agent 行为、AutoGPT。

---

### 范式二：ReWOO（Reasoning Without Observations）

**思想**：把推理和执行**解耦**——先统一规划出所有需要做的事，再批量执行，最后一起观察。

**流程**：
```
Planner（规划阶段）:
  Thought: 用户问天气和穿衣建议
  Plan: [Step1: 查北京天气, Step2: 查穿衣建议数据库]
  → 一次性生成完整计划

Worker（执行阶段）:
  → 批量执行 Step1 和 Step2（可并行）
  → 返回所有观察结果

Solver（求解阶段）:
  → 基于所有观察结果生成最终回答
```

**适用场景**：**可并行**的多步任务，或需要提前看到完整计划再做执行的任务。

**优缺点**：
- ✅ 规划阶段可以整体优化，减少不必要的 Tool call
- ✅ Worker 可并行，延迟更低
- ❌ 如果计划阶段出错，后续全错
- ❌ 无法根据中间结果动态调整计划

---

### 范式三：Reflection（自我反思）

**思想**：Agent 生成初始输出后，**让 Agent 自己评审并改进**这个输出，可迭代多轮。

**流程**：
```
初始生成 → 评审（是否达标？）→ 不达标则改进 → 再评审 → ...
```

**实现方式**：

1. **纯自我反思**：同一个 LLM 输出后立即评审
   ```
   原答案 → "这个答案有什么问题？" → 改进版答案
   ```

2. **带工具的反射**：用外部工具验证输出再改进
   ```
   生成代码 → 运行单元测试 → 测试失败 → 分析原因 → 修复 → 再测试
   ```

**典型案例**：Self-RAG（用 RAG 验证答案相关性 + 支持度后再决定是否采纳）、Devin（生成代码 → 测试 → 失败后反思修复）。

**优缺点**：
- ✅ 对代码生成、文字撰写等任务提升显著
- ✅ 特别适合"质量要求高、错误代价大"的场景
- ❌ 延迟和成本翻倍（需要多次 LLM 调用）
- ❌ 如果初始输出偏差太大，反思也可能走偏

---

### 范式四：Tool Use（工具调用）

**思想**：Agent 不只是生成文本，而是**拥有调用外部工具的能力**，从而完成真实世界任务。

**常见工具类型**：

| 工具类型 | 例子 |
|---------|------|
| 搜索 | Web Search、Wikipedia |
| 数据库 | SQL 查询、RAG 检索 |
| 代码执行 | Python REOL、Bash |
| API 调用 | 天气 API、邮件 API |
| 文件操作 | 读文件、写文件 |
| 浏览器 | 网页抓取、UI 操作 |

**关键工程问题**：
- 工具描述的质量（prompt 中如何描述工具功能）
- 工具返回结果如何截断（防止 context 溢出）
- 工具调用失败如何处理（超时、重试、fallback）

---

### 范式五：Planning（任务规划）

**思想**：面对复杂目标，Agent **主动拆解成子任务**，按步骤执行，必要时重新规划。

**流程**：
```
复杂目标 → 任务分解 → 生成执行计划 → 按计划执行 → 遇到问题 → 重新规划 → ...
```

**典型实现**：
- **OpenAI o1/o3**：内置隐式规划能力，通过 CoT 自我拆解
- **Superpowers**：`writing-plans` skill 自动生成含测试步骤的完整计划
- **AutoGPT/PromptEngineer**：自主拆解目标为待办清单

**优缺点**：
- ✅ 对复杂长周期任务有效
- ✅ 计划本身可被人类评审（Human-in-the-loop）
- ❌ 规划阶段 token 消耗大
- ❌ 任务分解质量依赖模型能力

---

## 三、多 Agent 架构模式

当单 Agent 无法处理复杂任务时，进入多 Agent 领域。LangChain 文档总结了 4 种核心模式：

### 模式一：Subagents（子代理）

**思想**：一个**主 Agent** 把任务拆分后，委托给专门的子 Agent 执行，主 Agent 监督结果。

```
用户 → 主 Agent → [子 Agent A] → [子 Agent B] → 主 Agent → 用户
                   ↓并行或顺序
               各自完成子任务
```

**典型案例**：
- Browser Agent：主 Agent 协调，拆分出"爬虫子 Agent"、"推理子 Agent"、"存储子 Agent"
- Code Agent：主 Agent 规划 + 代码生成子 Agent + 测试子 Agent

**何时用**：各子任务需要**不同的推理模式或工具集**。

---

### 模式二：Handoffs（交接）

**思想**：Agent 之间通过**显式交接**传递控制权，一个 Agent 完成自己部分后，明确把任务交给下一个 Agent。

```
Agent A（处理用户输入）→ 交给 → Agent B（执行具体操作）→ 交给 → Agent C（总结输出）
```

**典型案例**（CrewAI 流水线）：
```
Researcher Agent → 交给 → Writer Agent → 交给 → Editor Agent
（搜集资料）        （撰写内容）        （审核修改）
```

**何时用**：**顺序依赖**的流水线任务，每步输出是下一步的输入。

---

### 模式三：Skills（技能按需加载）

**思想**：单一 Agent 在不同任务阶段**按需加载不同的技能上下文**，而不是预先注入所有知识。

```
单一 Agent + Skill A（处理财务分析）→ 切换 → Skill B（处理法律审查）
         ↑ 按需加载                    ↑
      当前任务需要的技能上下文
```

**典型案例**：
- Superpowers 的 skill 系统：每个 skill 是一组打包的工具 + prompt
- Claude Code 的 `skills/` 目录：按需加载特定领域的操作规范

**何时用**：Agent 需要跨多个不同领域执行任务，但**不想维护多个 Agent 实例**。

---

### 模式四：Router（路由）

**思想**：有一个**分类器**根据输入类型，把任务分发到专门的 Agent。

```
用户输入 → 路由分类器 → [Agent A：财务问题]
                       [Agent B：技术支持]
                       [Agent C：一般问答]
```

**典型案例**：
- 企业客服机器人：意图识别 → 分配到订单/退款/技术等专用 Agent
- RAG 系统：问题分类 → 选择对应知识库检索路径

**何时用**：任务类型**明确可分类**，且每类需要不同的专业知识或工具。

---

## 四、四大多 Agent 架构对比

| 模式 | 模型调用次数 | 适用场景 | 复杂度 |
|------|------------|---------|--------|
| Subagents | 多（每个子任务一次） | 独立并行任务 | 中 |
| Handoffs | 顺序递减 | 流水线依赖任务 | 低 |
| Skills | 按需加载上下文 | 多领域单 Agent | 低 |
| Router | 分类 + 执行两次 | 明确分类的分发任务 | 中 |

> **大多数团队不需要多 Agent 系统**。从单 Agent + 工具开始，只有在遇到明确的扩展瓶颈时才引入多 Agent 架构。

---

## 五、主流框架生态

### 5.1 框架总览

| 框架 | 定位 | 核心优势 |
|------|------|---------|
| **LangGraph** | 可编程状态机 | 最灵活，适合复杂定制 |
| **CrewAI** | 多 Agent 编排 | 上手快，role-goal-backstory 模式清晰 |
| **AutoGen**（微软） | 多 Agent 对话 | 适合 Agent 间协商场景 |
| **Superpowers** | 单 Agent 能力扩展 | TDD + 子代理 + Skills，专注代码场景 |
| **OpenSpec** | 规范驱动 | spec-first，适合有治理要求的企业 |
| **Dify / Coze** | 无代码平台 | 快速搭建对话式 Agent |
| **OpenAI / Claude SDK** | 底层 API | 最小依赖，自己造轮子 |

### 5.2 如何选择

```
快速验证想法 → Dify / Coze（无代码）
生产级多 Agent 流水线 → CrewAI / LangGraph
复杂状态机 + 全链路可控 → LangGraph
AI 编码场景（Spec + TDD）→ Superpowers + OpenSpec
自己造轮子学习原理 → 纯 SDK（OpenAI / Anthropic）
```

---

## 六、生产环境最佳实践

### 6.1 Agent 生命周期管理

```
设计 → 训练/调优 → 测试 → 部署 → 监控 → 优化 → 迭代
```

关键环节：
- **评测集**：为每个 Agent 构建专属评测集（输入 → 期望输出）
- **轨迹回放**：记录完整 Thought-Action-Observation 链，用于复盘
- **Human-in-the-loop**：关键决策节点让人类确认，避免失控

### 6.2 核心工程问题

**Token 成本控制**
- 短回复任务：限制最大步数（max_steps=10）
- 长任务：用 summarizer 压缩中间历史
- 规划阶段：用 ReWOO 减少冗余 Tool call

**错误处理**
- Tool call 失败：重试 1-2 次后 fallback
- LLM 拒绝执行：给出更明确的引导 prompt
- 死循环检测：同一状态重复 N 次后强制终止

**安全与治理**
- 外部 API 调用要验签和限流
- 代码执行类工具要在沙箱中运行
- Agent 决策链路要可审计（记录完整轨迹）

### 6.3 RAG + Agent 配合

**知识类 Agent 的标配架构**：
```
用户问题
  ↓
检索（Embedding 相似度）→ 从向量数据库拉取相关文档
  ↓
Agent（携带检索结果 + 原始问题）→ 生成答案
  ↓
Reflection（可选）→ 验证答案是否和检索内容一致
```

Self-RAG 是这个模式的典型实现：生成回答后验证相关性，必要时重新检索。

---

## 七、真实案例

### 案例一：AI 客服机器人（Router + Handoffs）

```
用户输入 → 意图分类（Router）
  → [退款问题] → 退款 Agent → 人工审批节点 → 用户
  → [产品咨询] → RAG 检索 → 知识 Agent → 用户
  → [投诉] → 记录 → 人工客服 → 反馈用户
```

### 案例二：代码审查 Agent（Reflection + Tool Use）

```
收到 PR → 抓取代码 Diff
  → 代码审查 Agent（Reflection）
    → 生成审查意见 → 评审是否通过
    → 不通过：返回具体修改要求
  → 通过：提交合并
```

### 案例三：研究助手（CrewAI 多 Agent）

```python
# Researcher 从多个源搜集资料
researcher = Agent(role="研究分析师", goal="收集权威资料", backstory="...")
# Writer 基于研究结果撰写报告
writer = Agent(role="内容撰写师", goal="生成结构化报告", backstory="...")
# Editor 审核文章质量
editor = Agent(role="质量审核", goal="确保准确无误", backstory="...")

crew = Crew(agents=[researcher, writer, editor],
            tasks=[research_task, writing_task, editing_task])
```

### 案例四：OpenSpec + Superpowers 组合（Spec-Driven Agent）

```
OpenSpec: /opsx:propose → 生成 proposal/tasks.md（规范层）
  ↓
Superpowers: 读取 tasks.md → TDD 执行 → 子代理开发
  ↓
AI Agent: 每个 task 都有失败的测试先写 → 实现 → 验证通过
  ↓
OpenSpec: /opsx:archive → 归档完整变更链
```

---

## 八、学习路径建议

```
第一阶段：单 Agent + ReAct
  → 用 LangChain 或直接用 API 实现一个带工具调用的 Agent
  → 理解 Thought/Action/Observation 循环

第二阶段：加入 Reflection
  → 让 Agent 自我评审改进输出
  → 对代码生成类任务效果特别明显

第三阶段：多 Agent 编排
  → 用 CrewAI 实现一个简单的多 Agent 流水线
  → 理解何时需要多 Agent，何时不需要

第四阶段：生产治理
  → 加入轨迹记录、可观测性、Human-in-the-loop
  → 掌握 LangGraph 的状态机编程模式
```

---

## 参考资料

**范式来源**
- [DeepLearning.AI - Agentic Design Patterns Part 2: Reflection](https://www.deeplearning.ai/the-batch/agentic-design-patterns-part-2-reflection/)
- [Machine Learning Mastery - 7 Must-Know Agentic AI Design Patterns](https://machinelearningmastery.com/7-must-know-agentic-ai-design-patterns/)
- [Capabl - Agentic AI Design Patterns: ReAct, ReWOO, CodeAct](https://capabl.in/blog/agentic-ai-design-patterns-react-rewoo-codeact-and-beyond)
- [Source Allies - ReAct vs ReWOO](https://www.sourceallies.com/2024/08/react-vs-rewoo/)
- [Daily Dose of DS - Implementing ReAct Agentic Pattern](https://www.dailydoseofds.com/ai-agents-crash-course-part-10-with-implementation/)

**多 Agent 架构**
- [LangChain Docs - Multi-agent](https://docs.langchain.com/oss/python/langchain/multi-agent)
- [LangChain Blog - Choosing the Right Multi-Agent Architecture](https://blog.langchain.com/choosing-the-right-multi-agent-architecture/)
- [Galileo AI - Architectures for Multi-Agent Systems](https://galileo.ai/blog/architectures-for-multi-agent-systems)
- [CrewAI Blog - How to build Agentic Systems](https://blog.crewai.com/agentic-systems-with-crewai/)

**企业实践**
- [OneReach AI - Best Practices for AI Agent Implementations: Enterprise Guide 2026](https://onereach.ai/blog/best-practices-for-ai-agent-implementations/)
- [ArXiv - Evolution of Agentic AI Software Architecture (2602.10479)](https://arxiv.org/html/2602.10479)
- [Dev.to - Multi-Agent Architectures: Patterns Every AI Engineer Should Know](https://dev.to/sateesh2020/multi-agent-architectures-patterns-every-ai-engineer-should-know-jij)
