# LangChain 生态三件套使用指南

## 概述

LangChain 是当前 AI Agent 开发领域最活跃的生态之一，核心由三部分组成：

- **DeepAgent** — 深度 Agent 执行层
- **LangGraph** — 有状态工作流编排层
- **LangSmith** — 可观测性与调试层

三者各司其职，共同构成从开发到生产的完整闭环。

```
┌─────────────────────────────────────────────────────────┐
│                      LangChain 生态架构                  │
│                                                         │
│   DeepAgent     →  Agent 执行层（调用模型、工具、子代理）   │
│   LangGraph     →  编排层（工作流怎么走、状态怎么流转）     │
│   LangSmith     →  观测层（跑完回头看，哪里慢、哪里错）     │
└─────────────────────────────────────────────────────────┘
```

---

## 一、DeepAgent — 深度 Agent

### 1.1 定位

DeepAgent 是 LangChain 官方提供的**深度 Agent 框架**，用于构建能规划、子代理拆分、文件系统协作的复杂 Agent。与简单的 ReAct Agent 不同，DeepAgent 内置了任务分解、长期记忆、虚拟文件系统、子代理生成等高级能力。

### 1.2 核心用法

```python
from langchain.chat_models import init_chat_model
from deepagents import create_deep_agent

# 初始化 Agent
agent = create_deep_agent(
    model="anthropic:claude-sonnet-4-6",
    tools=[web_search],
    system_prompt="你是一个专业研究员，擅长分解复杂问题"
)

# 调用
result = agent.invoke({
    "messages": [{"role": "user", "content": "研究量子计算最新进展"}]
})

# 取结果
print(result["messages"][-1].content)
```

### 1.3 支持的模型

```python
agent = create_deep_agent(model="openai:gpt-5.2")
agent = create_deep_agent(model="azure_openai:gpt-5.2")
agent = create_deep_agent(model="google_genai:gemini-2.5-flash-lite")
agent = create_deep_agent(model="openrouter:anthropic/claude-sonnet-4-6")
```

### 1.4 核心能力

#### 内置 Todo 规划

DeepAgent 自动使用 `write_todos` 工具将复杂任务拆解为子任务：

```python
# Agent 接收 "研究量子计算最新进展"
# 自动拆解为：
#   - Todo 1: 搜索量子计算最新论文
#   - Todo 2: 筛选高质量来源
#   - Todo 3: 整理成报告
# 每个 Todo 由子代理独立完成
```

#### 大结果卸载（核心创新）

DeepAgent 通过虚拟文件系统解决上下文溢出问题：

```python
# 搜索结果等大中间数据自动写入虚拟文件系统
# 后续节点用 read_file 读取，不占上下文 Token
# 典型模式：
#   1. web_search 搜索 → 10万字结果
#   2. write_file 存入 /tmp/search_results.txt
#   3. 后续节点 read_file 获取，Token 消耗大幅降低
```

#### 多后端支持

```python
# Filesystem 后端 — 文件落地
from deepagents.backends import FilesystemBackend
backend = FilesystemBackend(base_dir="/tmp/agent_work")

# Store 后端 — 内存 KV
from deepagents.backends import StoreBackend
store = InMemoryStore()

# LocalShell 后端 — 本地命令执行
from deepagents.backends import LocalShellBackend

# Composite 后端 — 组合使用
from deepagents.backends import CompositeBackend, StateBackend, StoreBackend
```

#### 子代理 Spawn

```python
# 同一个任务可以并行 spawn 多个子代理处理不同子问题
# 典型场景：
#   主代理：分析市场趋势
#     → 子代理A：搜索A股数据
#     → 子代理B：搜索美股数据
#     → 子代理C：搜索宏观新闻
#   主代理：汇总三路结果
```

#### Skills 注入

```python
from deepagents.backends import StoreBackend
from deepagents.backends.utils import create_file_data

skill_url = "https://raw.githubusercontent.com/langchain-ai/deepagents/main/skills/langgraph-docs/SKILL.md"

store = StoreBackend()
store.set("/skills/langgraph-docs/SKILL.md", create_file_data(skill_content))
# Agent 运行时可读取 /skills/ 下的 Skill MD 文件，获取自定义指令
```

#### Middleware 中间件

```python
from langchain.agents.middleware import wrap_tool_call

def log_tool_call(tool_call):
    print(f"Calling tool: {tool_call['name']}")
    return tool_call

# 所有工具调用经过中间件统一处理（日志、脱敏、转换等）
agent = create_deep_agent(
    model="...",
    tools=[...],
    middleware=[log_tool_call]
)
```

### 1.5 适用场景

- 复杂多步骤研究任务（搜索→分析→写作）
- 需要长期记忆和上下文管理的场景
- 并行子代理处理的场景
- 作为 DeerFLOW 的核心执行单元

---

## 二、LangGraph — 工作流编排

### 2.1 定位

LangGraph 是 LangChain 官方的工作流编排库，用于构建**有状态、多步骤、条件分支、循环**的 AI 应用。与纯 LCEL 链式调用不同，LangGraph 将工作流建模为图结构，适合复杂 Agent 逻辑。

### 2.2 核心概念

```
图（Graph）
  ├── 节点（Node）— Python 函数，处理特定逻辑
  ├── 边（Edge） — 连接节点的路径
  ├── 状态（State）— 在节点间流转的共享数据
  └── 条件边（Conditional Edge）— 根据状态决定下一步走哪条路径
```

### 2.3 基本用法

```python
from langgraph.graph import StateGraph, END
from typing import TypedDict

# 1. 定义状态
class AgentState(TypedDict):
    messages: list
    next_action: str

# 2. 定义节点函数
def analyzer(state: AgentState):
    last_msg = state["messages"][-1]["content"]
    return {"next_action": "search" if "搜索" in last_msg else "respond"}

def searcher(state: AgentState):
    return {"messages": [...]}  # 调用搜索工具

def responder(state: AgentState):
    return {"messages": [...]}

# 3. 建图
graph = StateGraph(AgentState)
graph.add_node("analyzer", analyzer)
graph.add_node("searcher", searcher)
graph.add_node("responder", responder)

# 4. 定义边
graph.add_edge("analyzer", "searcher")
graph.add_edge("analyzer", "responder")  # 条件路由
graph.add_edge("searcher", "responder")
graph.add_edge("responder", END)

# 5. 编译并运行
app = graph.compile()
result = app.invoke({"messages": [{"role": "user", "content": "你好"}]})
```

### 2.4 条件分支

```python
from langgraph.graph import START

graph.add_edge(START, "analyzer")
graph.add_conditional_edges(
    "analyzer",
    lambda state: "searcher" if state.get("next_action") == "search" else "responder"
)
```

### 2.5 循环支持

LangGraph 原生支持工作流内的循环，这是对比链式调用的核心优势：

```python
def reasoning_loop(state: AgentState):
    # 反复调用 LLM 反思，直到满足退出条件
    if should_continue(state):
        return {"next": "reasoning_loop"}  # 循环回自己
    else:
        return {"next": "finalize"}         # 退出循环
```

### 2.6 Checkpointing（断点恢复）

生产环境务必开启，实现中断后从任意状态恢复：

```python
from langgraph.checkpoint.memory import MemorySaver

checkpointer = MemorySaver()  # 内存，生产环境换 PostgreSQL
app = graph.compile(checkpointer=checkpointer)

# 任意时刻保存快照
snapshot = app.get_state(config)
app.update_state(config, {"messages": [...], "next_action": "..."})
```

### 2.7 可视化

```python
# 导出 Mermaid 图
app.get_graph().draw_mermaid()
# 或生成 PNG
app.get_graph().draw_mermaid_png(output_file_path="graph.png")
```

### 2.8 最佳实践

| 维度 | 建议 |
|------|------|
| **State 设计** | 保持 State 小而类型安全，用 TypedDict，不要塞大对象 |
| **节点设计** | 每个节点单一职责，"保持 state 无聊" |
| **边（Edges）** | 优先用简单边，条件边只在真正需要路由时用 |
| **循环** | LangGraph 原生支持，这是对比链式调用的核心优势 |
| **Checkpointing** | 生产环境务必开启 MemorySaver，实现断点恢复 |
| **流式输出** | 用 `.stream()` 而非 `.invoke()` 实时看 Agent 思考过程 |
| **何时不用** | 简单 prompt→LLM→输出，用 LCEL 就够了，不要过度工程化 |

---

## 三、LangSmith — 可观测性平台

### 3.1 定位

LangSmith 是 LangChain 生态的**可观测性平台**，提供追踪（Trace）、评估（Eval）、数据集管理三大功能。不依赖 LangChain，可以独立追踪任意 LLM 应用。

### 3.2 核心用法

#### 装饰器方式（最常用）

```python
from langsmith.traceable import traceable

@traceable(run_type="chain")
def my_agent(input: str) -> str:
    # 每次调用自动记录完整 trace
    result = chain.invoke(input)
    return result
```

#### 上下文管理器（细粒度控制）

```python
from langsmith.traceable import TraceContext

def complex_function(data):
    with TraceContext() as trace:
        trace.set_metadata({"version": "v2", "user_id": "123"})
        trace.set_tags(["production", "experiment-a"])
        result = process(data)
        return result
```

### 3.3 核心功能

#### Tracing — 不只是 Debug

记录每一次 LLM 调用的完整输入输出、工具调用、思考过程：

```
User Query → LLM (prompt) → Tool Call (search) → LLM (with results) → Final Answer
     │              │                  │                    │
     └──────────────┴───────── 全程 trace 记录到 LangSmith ────┘
```

```python
from langsmith.traceable import traceable

@traceable(
    run_type="agent",
    tags=["production", "v2"],
    metadata={"prompt_version": "3.1"}
)
def research_agent(query: str):
    ...
```

#### 过滤与脱敏

```python
# 过滤敏感字段
from langsmith import traceable

@traceable(
    run_type="chain",
   过滤=["password", "api_key", "token"]
)
def process_user_data(data):
    ...
```

#### 自动评估（Auto Eval）

```python
# 1. 创建数据集
from langsmith import Client
client = Client()

dataset = client.create_dataset(
    dataset_name="agent-eval",
    description="Agent 评估数据集"
)

# 2. 添加测试用例
client.create_examples(
    inputs=[{"query": "你好"}],
    outputs=[{"expected": "你好，我是..."}],
    dataset_id=dataset.id
)

# 3. 配置评估器并运行
# 在 LangSmith UI 中点点点即可完成
# 支持：精确匹配、模糊匹配、LLM 作为裁判、自定义 Python Evaluator
```

#### Tagging 与实验对比

```python
# 给每次运行打标签
@traceable(tags=["feature:search-v2"])
def search_agent(query):
    ...

# 在 LangSmith UI 中按 tag 对比不同版本的 trace
# 验证 behavior change 是否符合预期
```

### 3.4 生产工作流

```
tag every run  →  evaluate what you trace  →  ship behind tags and experiments
```

核心习惯：
1. **每跑一次都 trace** — 积累数据
2. **真实失败案例加入 Dataset** — 形成回归测试集
3. **Human-in-the-loop** — AI 难以判断质量的 case 人工补充
4. **批量 Eval** — 验证 prompt / model 变更不破坏已有能力

### 3.5 使用技巧

| 技巧 | 说明 |
|------|------|
| `tags` 分组 | 按 feature / prompt 版本 / 实验名分组，方便对比 |
| `metadata` | 记录版本号、用户类型等结构化信息 |
| `filter` | 过滤敏感字段（密码、API Key、用户隐私） |
| `TraceContext` | 细粒度控制 trace 范围，不必全函数追踪 |
| `Dataset + Eval` | 把真实失败案例固化为测试用例，防止回归 |
| `compare traces` | 对比不同 prompt / model 版本的 trace，验证效果 |

---

## 四、三者协作实战

### 4.1 典型架构

```
┌─────────────────────────────────────────────────────────┐
│                     用户请求                            │
└─────────────────┬───────────────────────────────────────┘
                  │
         ┌───────▼────────┐
         │   LangGraph    │  ← 工作流编排（状态流转、条件分支、循环）
         │   (编排层)      │
         └───────┬─────────┘
                 │
         ┌───────▼────────┐
         │  DeepAgent    │  ← Agent 执行（模型调用、工具使用、子代理）
         │  (执行层)      │
         └───────┬─────────┘
                 │
         ┌───────▼────────┐
         │   LangSmith    │  ← 可观测性（trace、eval、生产监控）
         │  (观测层)      │
         └────────────────┘
```

### 4.2 在 DeerFLOW 中的应用

DeerFLOW 底层使用 LangGraph 实现 `lead_agent`，典型工作流：

```
User Message
    ↓
[analyzer] 节点 ← 分析用户意图，决定走哪条路
    ↓
[planner] 节点 ← 规划子任务列表
    ↓
[executor] 节点（DeepAgent）← 执行具体工具调用
    ↓
[aggregator] 节点 ← 汇总结果，生成最终回复
    ↓
User Response
```

### 4.3 开发调试闭环

1. **改代码** — 修改 `backend/deerflow/agents/` 里的节点逻辑
2. **本地启动** — `cd ~/deer-flow/backend && PYTHONPATH=. uv run langgraph dev --no-browser --allow-blocking --no-reload`
3. **打开调试 UI** — 浏览器访问 `http://localhost:2024`，可视化查看状态图
4. **手动发消息** — 在 UI 中测试任意输入，观察每个节点的输入输出
5. **用 LangSmith** — 若配置了 LangSmith API Key，自动记录每次 trace，方便复盘
6. **改完上线** — rebuild Docker，更新生产环境

---

## 五、环境搭建

### 5.1 安装依赖

```bash
pip install langgraph langchain langchain-anthropic langsmith
```

### 5.2 环境变量

```bash
export ANTHROPIC_API_KEY="sk-..."
export OPENAI_API_KEY="sk-..."
export TAVILY_API_KEY="tvly-..."
export LANGCHAIN_API_KEY="ls-..."       # LangSmith
export LANGCHAIN_TRACING_V2="true"       # 开启 LangSmith trace
export LANGCHAIN_PROJECT="my-project"    # 项目名
```

### 5.3 LangGraph Dev 启动

```bash
cd ~/deer-flow/backend

# 启动 LangGraph 开发服务器
PYTHONPATH=. uv run langgraph dev --no-browser --allow-blocking --no-reload

# 启动后访问：
#   - LangGraph Studio: http://localhost:2024
#   - Gateway API:      http://localhost:8001
```

---

## 六、注意事项

1. **DeepAgent 和 LangGraph 是互补关系**，DeepAgent 是执行单元，LangGraph 是编排框架，DeerFLOW 两者都用
2. **不要用 LangGraph 做简单链式调用** — 简单场景用 LCEL 就够了，上 LangGraph 有过度工程化之嫌
3. **LangSmith 不依赖 LangChain** — 任意 LLM 应用都可以接入
4. **Checkpointing 生产必开** — 否则 Agent 进程崩溃后无法恢复状态
5. **LangGraph Dev 用于开发调试** — 生产环境建议用 Docker 部署完整架构

---

## 七、参考资料

- [LangChain DeepAgent 官方文档](https://docs.langchain.com/oss/python/deepagents/overview)
- [LangGraph 官方文档](https://python.langchain.com/docs/langgraph)
- [LangSmith 官方文档](https://docs.smith.langchain.com)
- [LangGraph Best Practices - Swarnendu De](https://www.swarnendu.de/blog/langgraph-best-practices/)
- [DigitalOcean - LangSmith Debugging Guide](https://www.digitalocean.com/community/tutorials/langsmith-debudding-evaluating-llm-agents)
