# LangGraph Agent Server 从零实现：create_agent + Threads 持久化

> 文档源：综合自 [Run a local server](https://docs.langchain.com/oss/python/langgraph/local-server)、[Use threads](https://docs.langchain.com/langsmith/use-threads)、LangChain 官方论坛代码示例。
> 时间：2026-04-08

---

## 一、两条路的本质区别

| | `create_agent` 独立调用 | LangGraph + Agent Server + Threads |
|---|---|---|
| 状态管理 | 自己维护 `messages` list | Checkpointer 自动持久化 |
| 多轮上下文 | 每次调用要手动传历史 | 自动累积，无需手动处理 |
| Threads API | ❌ 不支持 | ✅ 完整支持 |
| Studio 可视化 | ❌ | ✅ |
| 适用场景 | 一次性任务、简单场景 | 生产级多轮对话 |

**核心结论：** `create_agent` 创建的 Agent 可以作为 LangGraph 的一个节点来使用，从而获得 Threads 持久化能力。只需把 `create_agent` 的逻辑包装进 `StateGraph`，加上 `checkpointer` 即可。

---

## 二、项目创建

```bash
# Python >= 3.11
pip install -U "langgraph-cli[inmem]" langgraph-sdk
pip install -U langchain-openai langchain-core langgraph

# 从模板创建标准项目
langgraph new ./my-agent-app --template new-langgraph-project-python
cd my-agent-app
pip install -e .
```

生成结构：

```
my-agent-app/
├── .env                        # API Key 配置
├── langgraph.json              # Server 配置（注册 graph）
├── requirements.txt
└── agent_app/
    ├── __init__.py
    └── agent.py                # 👈 核心：Agent 逻辑
```

---

## 三、Agent 代码详解（agent.py）

这是整个流程最核心的文件。

```python
# agent_app/agent.py

import os
from langgraph.graph import StateGraph, MessagesState, START, END
from langgraph.checkpoint.memory import MemorySaver
from langgraph.prebuilt import create_react_agent
from langchain_openai import ChatOpenAI
from langchain_core.messages import HumanMessage, AIMessage

# ─────────────────────────────────────────────
# 第一步：创建底层的 Agent
#   这里的逻辑等价于你原来用 create_agent() 创建的 Agent
#   只是这里用 prebuilt 的 create_react_agent
# ─────────────────────────────────────────────
llm = ChatOpenAI(
    model="gpt-4o",
    api_key=os.getenv("OPENAI_API_KEY"),
    temperature=0,
    streaming=True,
)

# 如果有工具，加上工具
# tools = [search_tool, calculate_tool, ...]
# agent = create_react_agent(llm, tools)
# 如果没有工具，直接用 llm（纯对话）
agent = create_react_agent(llm, tools=None)


# ─────────────────────────────────────────────
# 第二步：包装成带状态的 Graph 节点
#   node 函数是每次 Run 执行的单位
#   state["messages"] 自动携带之前所有历史
# ─────────────────────────────────────────────
def call_agent(state: MessagesState) -> dict:
    """
    每次调用时，state['messages'] 包含：
      - 前几轮的用户消息
      - 前几轮的 AI 回复
    agent.invoke() 内部会"看到"完整上下文
    """
    result = agent.invoke({"messages": state["messages"]})
    # 返回更新的 messages，LangGraph 会自动合并到状态
    return {"messages": result["messages"]}


# ─────────────────────────────────────────────
# 第三步：组装成 StateGraph
# ─────────────────────────────────────────────
graph = StateGraph(MessagesState)
graph.add_node("agent", call_agent)  # 注册节点
graph.add_edge(START, "agent")        # 入口
graph.add_edge("agent", END)          # 出口

# ─────────────────────────────────────────────
# 第四步：编译（关键！挂上 Checkpointer）
#   MemorySaver = 内存持久化，重启丢失（开发用）
#   生产用：PostgresSaver / SqliteSaver
# ─────────────────────────────────────────────
checkpointer = MemorySaver()
compiled_graph = graph.compile(checkpointer=checkpointer)
```

---

## 四、Server 配置（langgraph.json）

```json
{
  "dependencies": ["./requirements.txt"],
  "graphs": {
    "agent": "./agent_app/agent.py:compiled_graph"
  },
  "env": ".env"
}
```

- `"agent"` — 对外暴露的 graph ID，SDK 调用时用
- `"./agent_app/agent.py:compiled_graph"` — 从哪个文件导出编译好的 graph

---

## 五、环境变量（.env）

```bash
LANGSMITH_API_KEY=lsv2_xxxxx      # 必填（免费注册 https://smith.langchain.com）
OPENAI_API_KEY=sk-xxxxx            # 模型 API Key
```

---

## 六、启动 Agent Server

```bash
langgraph dev
```

输出：

```
🚀 API:        http://127.0.0.1:2024
🎨 Studio UI:  https://smith.langchain.com/studio/?baseUrl=http://127.0.0.1:2024
📚 API Docs:   http://127.0.0.1:2024/docs
```

> **注意：** `langgraph dev` 是内存模式，重启后 Threads 状态丢失，仅用于开发测试。

---

## 七、客户端完整代码（client.py）

### 7.1 安装 SDK

```bash
pip install -U langgraph-sdk
```

### 7.2 完整调用示例

```python
# client.py
import asyncio
from langgraph_sdk import get_client

async def main():
    # ── 连接 Agent Server ──
    client = get_client(url="http://127.0.0.1:2024")

    # ── 创建线程（会话容器）──
    thread = await client.threads.create(
        metadata={"user_id": "alice"}   # 可选：打标签方便管理
    )
    thread_id = thread["thread_id"]
    print(f"线程已创建: {thread_id}")

    # ── 第一轮对话 ──
    print("\n=== 第一轮：问天气 ===")
    async for chunk in client.runs.stream(
        thread_id=thread_id,
        graph_id="agent",                        # 对应 langgraph.json 里的 key
        input={
            "messages": [
                {"role": "user", "content": "北京今天天气怎么样？"}
            ]
        },
        stream_mode="values"
    ):
        print(f"事件: {chunk.event}")

    # ── 第二轮对话（追问）──
    # Thread 自动记忆上下文，Agent 能理解"明天呢？"指的是北京天气
    print("\n=== 第二轮：追问明天 ===")
    async for chunk in client.runs.stream(
        thread_id=thread_id,
        graph_id="agent",
        input={
            "messages": [
                {"role": "user", "content": "明天呢？"}
            ]
        },
        stream_mode="values"
    ):
        print(f"事件: {chunk.event}")

    # ── 查看当前状态 ──
    state = await client.threads.get_state(thread_id)
    print("\n=== 当前状态 ===")
    for msg in state["values"]["messages"]:
        print(f"  [{msg['type']}] {msg['content']}")

    # ── 查看完整执行历史 ──
    history = await client.threads.get_history(thread_id)
    print(f"\n=== 历史快照（共 {len(history)} 个 checkpoint）===")
    for snapshot in history:
        step = snapshot["metadata"]["step"]
        ckpt = snapshot["checkpoint_id"]
        print(f"  Step {step}: {ckpt}")


asyncio.run(main())
```

### 7.3 输出示例

```
线程已创建: 3c90c3cc-0d44-4b50-8888-8dd25736052a

=== 第一轮：问天气 ===
事件: updates  →  messages: [HumanMessage("北京今天..."), AIMessage("北京今天晴，25°C")]

=== 第二轮：追问明天 ===
事件: updates  →  messages: [HumanMessage("北京今天..."), AIMessage("北京今天晴"),
                               HumanMessage("明天呢？"), AIMessage("明天多云，22°C")]

=== 当前状态 ===
  [human] 北京今天天气怎么样？
  [ai] 北京今天晴，25°C
  [human] 明天呢？
  [ai] 明天多云，22°C

=== 历史快照（共 2 个 checkpoint）===
  Step 1: 1f02f46f-7308-616c-8000-1b158a9a6955
  Step 2: 1f02f46f-733f-6b58-8001-ea90dcabb1bd
```

---

## 八、执行流程图

```
┌─────────────────────────────────────────────────────────┐
│  client.py                                               │
│                                                         │
│  ① client.threads.create()                              │
│       ↓                                                 │
│  ② client.runs.stream(thread_id, msg1)                  │
│       ↓                                                 │
│  ③ client.runs.stream(thread_id, msg2)  ← 同一个 thread_id │
│       ↓                                                 │
│  ④ client.threads.get_state() / get_history()          │
└───────────────────────┬─────────────────────────────────┘
                        │ HTTP / WebSocket
                        ▼
┌─────────────────────────────────────────────────────────┐
│  Agent Server (langgraph dev)                           │
│                                                         │
│  ┌──────────────────────────────────────────────────┐  │
│  │  compiled_graph (StateGraph)                      │  │
│  │                                                   │  │
│  │  START → [call_agent 节点] → END                  │  │
│  │               ↓                                   │  │
│  │        agent.invoke({"messages": state["messages"]})│  │
│  └───────────────────┬──────────────────────────────┘  │
│                      ↓                                   │
│  ┌──────────────────────────────────────────────────┐  │
│  │  MemorySaver (Checkpointer)                       │  │
│  │                                                   │  │
│  │  Run 1: 保存 messages = [user, ai1]              │  │
│  │  Run 2: 读取 messages → 合并 → 保存 [user,ai1,  │  │
│  │          user2, ai2]                              │  │
│  └──────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

---

## 九、关键设计点解释

### 为什么原来 `create_agent` 不能直接用 Threads？

```python
# 原来每次调用是独立的，不共享状态
result1 = agent.invoke({"messages": [user_msg1]})
result2 = agent.invoke({"messages": [user_msg2]})
# result2 看不到 result1 的对话，agent 是"失忆"的

# 因为你没有持久化中间状态，MessagesState 没有跨调用累积
```

### 为什么加了 StateGraph + Checkpointer 就行了？

```python
# call_agent 节点每次收到的 state["messages"]
#   = 之前所有 Run 累积的消息（由 Checkpointer 管理）

# Checkpointer 相当于一个中间件：
#   Run 开始前：从存储读取当前状态
#   Run 结束后：把新状态写入存储
#   → 同一个 thread_id 每次都能"接着上次的记忆"继续
```

### `thread_id` 在哪里发挥作用？

```python
# 客户端传 thread_id → Server 知道操作哪个会话
# → Checkpointer 找到对应的持久化存储
# → 状态读写都针对这个 thread

# 不同的 thread_id = 完全独立的对话历史
# alice 的 thread 和 bob 的 thread 互不干扰
```

---

## 十、生产部署（简单说明）

开发模式（`langgraph dev`）状态存内存，重启丢失。

生产需要 Docker Compose：

```bash
# docker-compose.yml 包含：
#   - PostgreSQL（持久化存储）
#   - Redis（任务队列）
#   - LangGraph API Server
#   - MongoDB（可选）
```

```bash
# 启动生产级服务
docker compose up -d

# API 变成
client = get_client(url="http://your-server:8123")
```

详见 [Self-host standalone servers](https://docs.langchain.com/langsmith/deploy-standalone-server)。

---

## 参考资料

- [Run a local server](https://docs.langchain.com/oss/python/langgraph/local-server)
- [Use threads](https://docs.langchain.com/langsmith/use-threads)
- [Persistence - LangGraph](https://docs.langchain.com/oss/python/langgraph/persistence)
- [LangGraph SDK - PyPI](https://pypi.org/project/langgraph-sdk/)
- [create_react_agent](https://docs.langchain.com/oss/python/langgraph/prebuilt)
