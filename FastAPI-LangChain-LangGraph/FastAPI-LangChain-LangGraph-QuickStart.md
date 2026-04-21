# FastAPI + LangChain + LangGraph 快速上手

> 目标：让你能看懂公司项目代码、参与到实际开发中。

---

## 1. 技术栈全景

```
FastAPI        → Web 框架，处理 HTTP 请求/响应
LangChain      → LLM 应用开发框架，封装 Prompt、Chain、Tool
LangGraph      → 构建有状态、多步骤的 LLM 工作流（Agent）
```

简单理解：
- **FastAPI** = 你的"API 服务员"，接收请求、返回结果
- **LangChain** = 你的"LLM 胶水"，把模型、Prompt、工具粘起来
- **LangGraph** = 你的"工作流引擎"，让 Agent 能循环、记忆、分支

---

## 2. 核心概念速查

### 2.1 FastAPI 基础

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/hello")
def hello(name: str = "World"):
    return {"message": f"Hello, {name}!"}
```

**关键概念：**
- `app` — FastAPI 实例
- `@app.get/post/put/delete` — 路由装饰器
- `def endpoint(params)` — 路径参数、查询参数自动从请求中解析
- `FastAPI(run_in_executor)` — 异步处理，默认 async

**公司项目常见写法：**
```python
from fastapi import APIRouter, Depends, HTTPException
from pydantic import BaseModel

router = APIRouter(prefix="/api/v1", tags=["agent"])

class ChatRequest(BaseModel):
    query: str
    session_id: str | None = None

class ChatResponse(BaseModel):
    answer: str
    session_id: str

@router.post("/chat", response_model=ChatResponse)
async def chat(req: ChatRequest):
    # 这里调 LangChain / LangGraph
    ...
```

### 2.2 LangChain 核心三件套

#### Model（模型）
```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(
    model="gpt-4o",
    temperature=0.7,
    api_key="your-api-key"  # 或从环境变量读取
)
```

#### Prompt（提示词模板）
```python
from langchain_core.prompts import ChatPromptTemplate

prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一个专业的助手，擅长{domain}。"),
    ("human", "请回答：{question}"),
])

chain = prompt | llm  # 管道式组合
response = chain.invoke({"domain": "编程", "question": "什么是闭包？"})
```

#### OutputParser（输出解析）
```python
from langchain_core.output_parsers import StrOutputParser

chain = prompt | llm | StrOutputParser()
```

### 2.3 LangChain Expression Language (LCEL)

```
chain = prompt | llm | output_parser
```

`|` 是管道操作符，把上一节的输出传给下一节。

**公司项目典型 Chain：**
```python
from langchain_core.runnables import RunnableSerializable

class RAGChain(RunnableSerializable):
    def __init__(self, retriever, llm, prompt):
        self.retriever = retriever
        self.llm = llm
        self.prompt = prompt

    def invoke(self, input_data, **kwargs):
        docs = self.retriever.invoke(input_data["question"])
        context = "\n".join([doc.page_content for doc in docs])
        return self.llm.invoke(
            self.prompt.format(context=context, question=input_data["question"])
        )
```

### 2.4 LangGraph — 有状态的工作流

LangGraph 用于构建 Agent，让 LLM 能：
- 循环执行（不是一次调用就结束）
- 维护状态（session、memory）
- 分支判断（if-else 路由）

**最小示例：**
```python
from langgraph.graph import StateGraph, END
from typing import TypedDict, Annotated
import operator

# 定义状态
class AgentState(TypedDict):
    messages: list
    next_action: str

# 节点函数
def agent(state):
    """LLM 决定下一步"""
    messages = state["messages"]
    response = llm.invoke(messages)
    return {"messages": [response], "next_action": "should_end"}

def should_continue(state) -> str:
    return "end" if state["next_action"] == "should_end" else "agent"

# 建图
workflow = StateGraph(AgentState)
workflow.add_node("agent", agent)
workflow.set_entry_point("agent")
workflow.add_conditional_edges("agent", should_continue, {
    "end": END,
    "agent": "agent"  # 循环
})
workflow.add_edge("end", END)

app = workflow.compile()
```

---

## 3. 公司项目常见目录结构

```
project/
├── app/
│   ├── __init__.py
│   ├── main.py          # FastAPI 入口
│   ├── api/             # 路由
│   │   └── v1/
│   │       └── agent.py
│   ├── core/            # 核心配置
│   │   ├── config.py    # 环境变量、API Key
│   │   └── security.py
│   ├── agents/          # LangGraph Agent 定义
│   │   └── graph.py
│   ├── chains/          # LangChain Chain 定义
│   │   └── rag.py
│   ├── tools/           # LangChain Tools
│   │   └── search.py
│   └── schemas/         # Pydantic 模型
│       └── request.py
├── tests/
├── requirements.txt / pyproject.toml
└── .env
```

---

## 4. 开发流程速记

### 读代码顺序
1. `main.py` — 看入口，了解有哪些 API 路由
2. `core/config.py` — 看环境变量，理解依赖
3. `chains/` 或 `agents/` — 看核心业务逻辑
4. `api/` — 看请求/响应格式

### 改代码顺序
1. 找到对应的 Chain 或 Node
2. 改完后用 `chain.invoke({"key": "value"})` 本地测试
3. 再接上 FastAPI 路由

### 本地调试技巧
```python
# 直接调 Chain
result = my_chain.invoke({"query": "你好"})
print(result)

# 带内存的 Agent
from langgraph.checkpoint.memory import MemorySaver
checkpointer = MemorySaver()
app = workflow.compile(checkpointer=checkpointer)
config = {"configurable": {"thread_id": "user-123"}}
result = app.invoke({"messages": [("user", "你好")]}, config)
```

---

## 5. 常见坑 & 解答

| 问题 | 解答 |
|------|------|
| 返回 `coroutine` 对象而不是结果 | 检查是否漏了 `.invoke()` 或 `await` |
| LangChain 版本冲突 | 用 `langchain-core`、`langchain-openai` 分离管理版本 |
| LangGraph 状态不保存 | 检查是否传了 `config`（thread_id） |
| FastAPI 异步不生效 | 确认 `async def` 且调用 `await` |
| `.env` 变量读不到 | 用 `python-dotenv`，确认文件在项目根目录 |

---

## 6. 建议学习路径

```
第1天：FastAPI 基础（路由、Request/Response、Pydantic）
第2天：LangChain 核心（Model、Prompt、Chain、LCEL）
第3天：LangChain Retrieval（RAG、Retriever、VectorStore）
第4天：LangGraph 入门（State、Node、Edge、条件分支）
第5天：读懂公司项目代码，尝试小改动
```

---

## 7. 推荐资料

- [FastAPI 官方文档](https://fastapi.tiangolo.com/zh/tutorial/) — 中文友好
- [LangChain Python 教程](https://python.langchain.com/docs/tutorials/) — 官方 Quick-start
- [LangGraph 官方教程](https://langchain-ai.github.io/langgraph/tutorials/) — 核心是 StateGraph
- [LCEL 文档](https://python.langchain.com/docs/concepts/lcel/) — 管道语法的完整参考

---

*有问题随时问我，可以针对具体模块深入讲。*
