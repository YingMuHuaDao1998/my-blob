# LangChain 学习总结

> 来源：[LangChain Python 官方文档 - Learn](https://docs.langchain.com/oss/python/learn)
> 整理日期：2026-04-10
> 目标读者：从小白出发，每个技术点都精细讲解

---

## 目录

1. [先搞清楚：LangChain 生态是什么](#1-langchain-生态是什么)
2. [核心概念](#2-核心概念)
3. [三层架构详解](#3-三层架构详解)
4. [Tutorial 实战：每个模块能做什么](#4-tutorial-实战每个模块能做什么)
5. [多 Agent 协作模式](#5-多-agent-协作模式)
6. [通用工作流程图](#6-通用工作流程图)
7. [如何选择合适的工具](#7-如何选择合适的工具)

---

## 1. LangChain 生态是什么

LangChain 是一个**帮你快速构建 AI Agent（AI 代理）**的工具集。它不是单个产品，而是一套层层叠加的生态系统：

```mermaid
graph TB
    subgraph "LangChain 生态三层"
        A["🔧 Framework 层<br>LangChain（快速上手）"] 
        B["⚙️ Runtime 层<br>LangGraph（底层编排）"]
        C["🚀 Harness 层<br>Deep Agents SDK（开箱即用）"]
    end
    
    A --> B
    B --> C
    
    style A fill:#e1f5fe
    style B fill:#fff3e0
    style C fill:#e8f5e9
```

| 层级 | 产品 | 定位 | 一句话理解 |
|------|------|------|-----------|
| **Framework** | LangChain | 高级抽象，快速起步 | "搭 Agent 的脚手架" |
| **Runtime** | LangGraph | 底层编排，精细控制 | "运行 Agent 的引擎" |
| **Harness** | Deep Agents SDK | 复杂任务，开箱即用 | "全能 Agent 工具箱" |

### 三者关系

- **Deep Agents SDK** 建立在 **LangGraph** 之上
- **LangChain 1.0** 也建立在 **LangGraph** 之上
- **LangGraph** 提供持久化、人机交互、Streaming 等底层能力
- **LangChain** 在 LangGraph 之上封装了更高级的抽象

### Feature 对比表

| 功能 | LangChain | LangGraph | Deep Agents |
|------|-----------|-----------|-------------|
| 短时记忆 | Short-term memory | Short-term memory | StateBackend |
| 长时记忆 | ✅ 有 | ✅ 有 | ✅ 有 |
| 子 Agent | Multi-agent subagents | Subgraphs | Subagents |
| 人机交互 | middleware | Interrupts | `interrupt_on` 参数 |
| Streaming | ✅ 有 | ✅ 有 | ✅ 有 |

---

## 2. 核心概念

### 2.1 Provider 和 Model（模型）

**Provider（提供商）**是托管 AI 模型的服务商，比如 OpenAI、Anthropic、Google、AWS Bedrock 等。每个 Provider 有独立的集成包（如 `langchain-openai`）。

LangChain 的核心理念：**一个接口，走遍所有模型**。

```python
# ✅ 方式一：provider:model 格式（推荐）
from langchain_openai import ChatOpenAI
llm = ChatOpenAI(model="openai:gpt-4o")

# ✅ 方式二：直接指定模型名
llm = ChatOpenAI(model="gpt-4o")

# ✅ 换模型？只改一行代码
llm = ChatOpenAI(model="anthropic:claude-3-5-sonnet")
```

**为什么重要？** 你不需要因为换了一个 AI 提供商就重写整个应用，LangChain 给你统一的抽象。

**路由器和代理**：
- **OpenRouter** / **LiteLLM**：统一 API，同时访问多个 Provider，支持自动 failover（一个模型挂了自动切另一个）

```mermaid
graph LR
    A["应用代码"] --> B["LangChain 统一接口"]
    B --> C["OpenRouter / LiteLLM"]
    C --> D["OpenAI"]
    C --> E["Anthropic"]
    C --> F["Google"]
    C --> G["...其他"]
```

### 2.2 Memory（记忆）

LangChain 的记忆系统类比人类大脑，分为**短时记忆**和**长时记忆**：

```mermaid
graph TD
    A["人类记忆类型"] --> B["短时记忆<br>（单次对话内）"]
    A --> C["长时记忆<br>（跨会话持久化）"]
    
    C --> D["语义记忆 Semantic<br>存储事实（用户信息等）"]
    C --> E["情景记忆 Episodic<br>存储经历/经验"]
    C --> F["程序记忆 Procedural<br>存储规则/指令"]
```

**短时记忆（Short-term memory）**：
- 作用域：单次对话 / 一个线程（Thread）
- 实现：通过 **LangGraph State** 的一部分，通过 **checkpointer** 持久化
- 类比：聊天的上下文窗口

```python
# LangGraph 中，短时记忆就是 state 里的 messages 字段
from langgraph.graph import MessagesState

state = MessagesState()
# 每次对话，消息追加到 state["messages"]，checkpointer 自动保存
```

**长时记忆（Long-term memory）**：
- 作用域：跨会话、跨线程
- 实现：通过 **LangGraph Store**，存储为 JSON 文档
- 组织方式：`namespace` + `key` → 值

```python
from langgraph.store.memory import MemoryStore

store = MemoryStore()
# 存记忆
await store.aput(("user", user_id), "preferences", {"theme": "dark"})
# 取记忆
prefs = await store.aget(("user", user_id), "preferences")
```

**三种记忆类型（类比人类大脑）**：

| 类型 | 存储内容 | 实现方式 |
|------|---------|---------|
| **语义记忆 Semantic** | 事实（用户信息、资料） | Profile 或 Collection 文档 |
| **情景记忆 Episodic** | 经历/经验 | Few-shot example prompting |
| **程序记忆 Procedural** | 执行任务的规则 | Agent 反思修改自身提示词 |

### 2.3 Context（上下文）

**Context engineering** 是"以正确格式提供正确信息和工具给 AI 应用"的实践。

```mermaid
graph TD
    A["Context 类型"] --> B["Static Runtime Context<br>静态·不可变·单次运行"]
    A --> C["Dynamic Runtime Context<br>动态·可变·单次运行内"]
    A --> D["Dynamic Cross-conversation<br>动态·跨会话持久化"]
    
    B --> B1["通过 invoke/stream 的<br>context 参数传入"]
    C --> C1["LangGraph State 对象<br>= 短时记忆"]
    D --> D1["LangGraph Store<br>= 长时记忆"]
```

**注意**：Runtime context ≠ LLM 的 context window（上下文窗口）。前者是代码层面的依赖注入，后者是 LLM 能吃的 token 上限。

### 2.4 Component Architecture（组件架构）

LangChain 的组件生态分为 7 大类，形成一个处理 pipeline：

```mermaid
graph LR
    A["输入处理<br>Input Processing"] --> B["Embedding & 存储<br>Embedding & Storage"]
    B --> C["检索<br>Retrieval"]
    C --> D["生成<br>Generation"]
    D --> E["编排<br>Orchestration"]
    
    B -.-> B1["Embedding 模型"]
    B -.-> B2["向量数据库<br>Chroma / Pinecone / FAISS"]
    C -.-> C1["Retrievers"]
    E -.-> E1["Agents"]
    E -.-> E2["Tools"]
    E -.-> E3["Memory"]
```

**7 类核心组件**：

| 组件类别 | 作用 | 常见例子 |
|---------|------|--------|
| **Models** | 调用 LLM | ChatOpenAI, ChatAnthropic |
| **Tools** | 赋予 Agent 外部能力 | 搜索、数据库、API 调用 |
| **Agents** | 决定"思考-行动"循环 | ReAct Agent, Tool Calling Agent |
| **Memory** | 管理对话历史 | 消息历史、自定义状态 |
| **Retrievers** | 从向量库检索相关内容 | VectorStoreRetriever |
| **Document Processing** | 文档加载和切分 | Loaders, Splitters |
| **Vector Stores** | 存储 Embedding 向量 | Chroma, Pinecone, FAISS |

---

## 3. 三层架构详解

### 3.1 LangChain（Framework 层）

**定位**：高级抽象，快速上手

**核心功能**：
- 提供 `create_agent` 等简单易用的接口
- 封装了 Model、Tool、Agent loop、Memory 的抽象
- 不需要懂 LangGraph 也能用

```python
from langchain import hub
from langchain.agents import create_tool_calling_agent, AgentExecutor
from langchain_openai import ChatOpenAI

# 加载预置提示词
prompt = hub.pull("hwchase17/openai-functions-agent")

# 创建 Agent
llm = ChatOpenAI(model="gpt-4o")
agent = create_tool_calling_agent(llm, tools, prompt)

# 运行
agent_executor = AgentExecutor(agent=agent, tools=tools)
result = agent_executor.invoke({"input": "帮我查一下北京的天气"})
```

**适用场景**：
- ✅ 快速构建简单 Agent
- ✅ 团队标准化开发
- ✅ 不需要复杂编排的场景

### 3.2 LangGraph（Runtime 层）

**定位**：底层编排引擎，适合复杂生产环境

**核心概念**：

```mermaid
graph TD
    A["StateGraph<br>状态图"] --> B["Nodes（节点）<br=程序步骤"]
    A --> C["Edges（边）<br>=流程连接"]
    A --> D["State<br>=共享数据"]
    
    B --> B1["START → 入口"]
    B --> B2["普通函数 → 同步执行"]
    B --> B3["ToolNode → 执行工具"]
    B --> B4["END → 出口"]
    
    C --> C1["普通边 → 固定流向"]
    C --> C2["条件边 → 根据state判断"]
    C --> C3["Send → map-reduce并发"]
```

**StateGraph 核心写法**：

```python
from langgraph.graph import StateGraph, MessagesState, START, END
from langgraph.store.memory import MemorySaver

# 1. 定义状态（TypedDict）
class MyState(MessagesState):
    step: str

# 2. 建图
graph = StateGraph(MyState)

# 3. 加节点
graph.add_node("process", process_function)
graph.add_node("tools", tool_node)

# 4. 连边
graph.add_edge(START, "process")
graph.add_edge("process", END)

# 5. 编译（加持久化）
checkpointer = MemorySaver()
app = graph.compile(checkpointer=checkpointer)

# 6. 运行
result = app.invoke(
    {"messages": [{"role": "user", "content": "hello"}]},
    config={"configurable": {"thread_id": "1"}}
)
```

**LangGraph vs LangChain**：LangChain 的 Agent（如 `create_agent`）底层实际上也是在 LangGraph 上跑的，LangGraph 是更底层的 Runtime。

**适用场景**：
- ✅ 需要精细控制 Agent 编排
- ✅ 长时间运行的 Stateful Agent
- ✅ 需要人机交互（Human-in-the-loop）
- ✅ 复杂工作流（结合确定性和 Agent 步骤）

### 3.3 Deep Agents SDK（Harness 层）

**定位**：复杂任务的"全能工具箱"，开箱即用

**一句话理解**：在 LangGraph 基础上，给你预置了文件系统、子 Agent、任务规划、Shell 执行等能力，不需要从零搭。

```python
# 安装：pip install deepagents
from deepagents import create_deep_agent

def get_weather(city: str) -> str:
    """获取天气"""
    return f"{city}今天晴天，25度！"

agent = create_deep_agent(
    tools=[get_weather],
    system_prompt="你是一个有帮助的助手",
)

# 直接运行
result = agent.invoke({
    "messages": [{"role": "user", "content": "北京天气怎么样？"}]
})
```

**Deep Agents 核心能力**：

| 能力 | 说明 | 代码 |
|------|------|------|
| **任务规划** | 内置 `write_todos` 工具，拆解复杂任务 | Agent 自动分解步骤 |
| **文件系统** | `ls`, `read_file`, `write_file`, `edit_file` | 防止 context window 溢出 |
| **Shell 执行** | `execute` 工具（在 sandbox 下） | 运行测试、构建、git |
| **子 Agent** | `task` 工具，spawn 专用子 Agent | 上下文隔离 |
| **长时记忆** | LangGraph Store 跨线程持久化 | 保存/读取历史信息 |
| **人机交互** | `interrupt_on` 配置 | 敏感操作需人工确认 |
| **可插拔后端** | 内存/本地磁盘/sandbox/自定义 | 灵活切换存储 |

```mermaid
graph TB
    A["Deep Agents SDK"] --> B["内置工具"]
    A --> C["任务规划<br>write_todos"]
    A --> D["虚拟文件系统<br>ls/read/write/edit"]
    A --> E["子Agent<br>task tool"]
    A --> F["长时记忆<br>Memory Store"]
    A --> G["人机交互<br>interrupt_on"]
    
    B --> B1["execute<br>Shell命令"]
    B --> B2["search<br>搜索"]
    B --> B3["custom<br>自定义"]
    
    style A fill:#e8f5e9
```

---

## 4. Tutorial 实战：每个模块能做什么

### 4.1 Semantic Search（语义搜索）

**场景**：基于 PDF 的语义搜索引擎

```mermaid
graph LR
    A["用户问题"] --> B["Embedding 模型"]
    B --> C["向量数据库检索"]
    C --> D["相关文档"]
    D --> E["LLM 生成答案"]
```

**流程**：
1. 文档加载 → PDF/TXT
2. 文本切分（chunk）→ 小段文本
3. Embedding → 向量存储到向量库
4. 用户提问 → 同样 Embedding
5. 相似度检索 → Top-K 相关文档
6. 将文档喂给 LLM → 生成答案

### 4.2 RAG Agent（检索增强生成 Agent）

**场景**：让 Agent 能够从知识库中检索信息再回答

```mermaid
graph TD
    A["用户问题"] --> B["Agent"]
    B --> C{"需要检索？"}
    C -->|是| D["向量检索"]
    C -->|否| F["直接回答"]
    D --> E["拿到相关文档"]
    E --> F
    F --> G["LLM 生成"]
```

**LangChain 做法**（简单场景）：
```python
# 使用 create_agent 快速创建 RAG Agent
agent = create_agent(llm, tools=[retriever_tool], prompt=...)
```

**LangGraph 做法**（精细控制）：
```python
# 自定义 RAG 流程中的每一步
graph.add_node("generate_query", generate_query_or_respond)
graph.add_node("grade_documents", grade_documents)  # 过滤相关文档
graph.add_node("rewrite_question", rewrite_question)  # 改写查询
graph.add_node("generate_answer", generate_answer)

# 条件边：根据分数决定下一步
graph.add_conditional_edges(
    "grade_documents",
    decide_to_generate,
    {"rewrite": "rewrite_question", "generate": "generate_answer"}
)
```

### 4.3 SQL Agent

**场景**：用自然语言查询数据库

```mermaid
graph TD
    A["用户问数据库"] --> B["SQL Agent"]
    B --> C["列出表<br>list_tables"]
    B --> D["查表结构<br>get_schema"]
    B --> E["生成SQL<br>generate_query"]
    E --> F{"人工审核?"}
    F -->|是| G["interrupt<br>暂停等待"]
    G --> H["批准后执行"]
    F -->|否| H
    H --> I["run_query<br>执行SQL"]
    I --> J["返回结果"]
```

**LangChain SQL Agent 工具集**：
- `sql_db_query` — 执行 SQL 查询
- `sql_db_schema` — 查看表结构
- `sql_db_list_tables` — 列出所有表
- `sql_db_query_checker` — 检查 SQL 语法

**LangGraph SQL Agent**（更精细）：
- 独立节点：`list_tables` → `get_schema` → `generate_query` → `check_query` → `run_query`
- 支持 `interrupt` 实现人工审核（敏感 SQL 执行前暂停）

### 4.4 Voice Agent（语音 Agent）

**场景**：能听、能说、能查数据的对话助手

```mermaid
graph TD
    A["用户语音"] --> B["Audio Transcription Tool<br>语音转文字"]
    B --> C["Agent<br>理解+决策"]
    C --> D["调用工具<br>查数据/API"]
    D --> E["LLM 生成回复"]
    E --> F["文字转语音"]
    F --> G["用户听到回复"]
    
    C -.->|"历史消息"| G
    G -.->|"下一轮"| A
```

**特点**：
- 支持多轮对话（消息历史维护）
- 工具调用获取结构化数据
- 适合做电话机器人、语音助手

### 4.5 Data Analysis Agent（数据分析 Agent）

**场景**：自动分析数据并发送报告（如 Slack）

- 内置数据分析工具
- 可对接 Slack 发送报告
- 适合运营/财务自动化场景

### 4.6 Deep Research Agent（深度研究 Agent）

**场景**：多步骤网络研究，自动委托子 Agent

- **子 Agent 委托**：主 Agent 拆解任务 → 分发给专门的研究子 Agent
- **战略反思**：定期反思当前进展，决定下一步
- 多轮搜索 → 整理 → 汇总报告

---

## 5. 多 Agent 协作模式

当单 Agent 能力不够时，需要多个 Agent 协作。

### 5.1 Supervisor Pattern（主管模式）

**思想**：一个主管 Agent 把子 Agent 当工具调用

```mermaid
graph TD
    A["用户请求"] --> B["Supervisor Agent<br>主管"]
    B --> C{"调用哪个子Agent?"}
    C -->|日历| D["Calendar Agent<br>日历Agent"]
    C -->|邮件| E["Email Agent<br>邮件Agent"]
    D --> B
    E --> B
    B --> F["汇总回复"]
```

**实现**：
- 子 Agent 用 `@tool` 包装
- Supervisor 通过 `ToolRuntime` 获取子 Agent 上下文
- 支持 Human-in-the-loop（`HumanInTheLoopMiddleware`）

```python
# 子Agent包装成工具
@tool
def calendar_agent(query: str) -> str:
    """查日历"""
    return calendar_agent_instance.run(query)

@tool
def email_agent(query: str) -> str:
    """发邮件"""
    return email_agent_instance.run(query)

# Supervisor 调用
supervisor_agent = create_agent(llm, tools=[calendar_agent, email_agent], ...)
```

### 5.2 Handoffs Pattern（交接模式）

**思想**：单 Agent 内部有状态机，不同状态走不同处理流程

```mermaid
graph TD
    A["用户进来"] --> B["warranty_collector<br>保修期收集"]
    B --> C["issue_classifier<br>问题分类"]
    C --> D{"什么问题?"}
    D -->|退款| E["refund_specialist<br>退款专员"]
    D -->|换货| F["exchange_specialist<br>换货专员"]
    D -->|维修| G["repair_specialist<br>维修专员"]
    E --> H["汇总回复"]
    F --> H
    G --> H
```

**实现**：
- 用 `SupportState` 携带 `current_step` 字段
- `@wrap_model_call` 装饰器实现中间件
- `Command` 对象触发状态转换

```python
class SupportState(TypedDict):
    messages: Annotated[list, add_messages]
    current_step: str  # warranty_collector | issue_classifier | ...

# 状态转换
Command(
    resume={"current_step": "resolution_specialist", ...}
)
```

### 5.3 Router Pattern（路由模式）

**思想**：根据用户查询类型，路由到不同的知识库/专业 Agent

```mermaid
graph TD
    A["用户问题"] --> B["Router Agent<br>判断问题类型"]
    B --> C{"类型?"}
    C -->|产品| D["产品知识库Agent"]
    C -->|技术| E["技术支持Agent"]
    C -->|售后| F["售后Agent"]
    D --> G["汇总"]
    E --> G
    F --> G
```

### 5.4 Skills Pattern（技能模式）

**思想**：Agent 渐进式加载专门技能，按需加载上下文

```mermaid
graph TD
    A["用户请求"] --> B["主Agent"]
    B --> C{"需要SQL技能?"}
    C -->|是| D["动态加载<br>SQL Skill"]
    C -->|否| F["继续"]
    D --> E["使用技能处理"]
    E --> F
    F --> G["返回结果"]
```

---

## 6. 通用工作流程图

### 6.1 Agent + Tools 工作流

```mermaid
graph TD
    A["用户请求"] --> B["Agent 思考"]
    B --> C{"需要调用工具?"}
    C -->|是| D["调用工具<br>ToolNode"]
    D --> E{"结果?"}
    E -->|"成功"| B
    E -->|"失败| F["重试或放弃"]
    C -->|否| G["返回最终答案"]
```

### 6.2 RAG 全链路

```mermaid
graph LR
    A["文档"] --> B["Loader<br>加载"]
    B --> C["Splitter<br>切分"]
    C --> D["Embedding<br>向量化"]
    D --> E["Vector Store<br>向量库"]
    
    F["用户问题"] --> G["Embedding"]
    G --> H["相似度检索"]
    E --> H
    H --> I["Top-K 相关文档"]
    I --> J["LLM 生成答案"]
```

### 6.3 LangGraph 状态机

```mermaid
graph TD
    START --> A["节点A"]
    A --> B{"条件判断"}
    B -->|"路径1"| C["节点B"]
    B -->|"路径2"| D["节点D"]
    C --> E["节点E"]
    D --> E
    E --> END
    
    style START fill:#90caf9
    style END fill:#90caf9
```

---

## 7. 如何选择合适的工具

```mermaid
flowchart TD
    A["你想做什么?"] --> B{"复杂度"}
    
    B -->|"简单Agent<br>快速上手"| C["LangChain<br>create_agent"]
    B -->|"复杂工作流<br>精细控制"| D["LangGraph<br>StateGraph"]
    B -->|"复杂多步骤任务<br>需要规划+文件系统"| E["Deep Agents SDK"]
    
    D --> F{"需要多Agent协作?"}
    F -->|"是"| G["Multi-agent patterns<br>Supervisor/Handoffs/Router"]
    F -->|"否"| H["单Agent + Tools"]
    
    E --> F
    
    C --> H
    G --> H
```

### 选择决策表

| 需求 | 推荐 | 原因 |
|------|------|------|
| 快速做个聊天机器人 | LangChain | 一行代码搞定 |
| 需要查数据库 | LangChain SQL Agent | 内置工具 |
| 复杂多步骤任务 | LangGraph | 状态机+条件边 |
| 长期运行的 Agent | LangGraph + checkpointer | 持久化+断点恢复 |
| 人工审核环节 | LangGraph interrupt | 暂停等批准 |
| 多个专业 Agent 协作 | Multi-agent patterns | Supervisor/Handoffs |
| 复杂+规划+子 Agent | Deep Agents SDK | 开箱即用 |
| 语音对话 | LangChain Voice Agent | 内置语音处理 |

---

## 更多资源

- [LangChain 官方文档](https://docs.langchain.com/oss/python/langchain/overview)
- [LangGraph 官方文档](https://docs.langchain.com/oss/python/langgraph/overview)
- [Deep Agents SDK](https://docs.langchain.com/oss/python/deepagents/overview)
- [LangChain Academy（课程）](https://academy.langchain.com/)
- [LangSmith（调试追踪）](https://smith.langchain.com/)
