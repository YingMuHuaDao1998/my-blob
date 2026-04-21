# LangChain 框架最佳实践

> 整理时间：2026-03-27
> 来源：官方文档、社区实践、生产经验总结

---

## 一、LangChain 核心架构

```
┌─────────────────────────────────────────────────┐
│                   Application                    │
├─────────────────────────────────────────────────┤
│  Chains │ Agents │ Memory │ Retrieval │ Tools   │
├─────────────────────────────────────────────────┤
│              LangChain Expression Language       │
│                    (LCEL)                        │
├─────────────────────────────────────────────────┤
│     Model I/O │ Output Parsers │ Prompts        │
└─────────────────────────────────────────────────┘
```

---

## 二、LCEL（LangChain Expression Language）

### 2.1 为什么使用 LCEL

LCEL 是 LangChain 的核心编排语言，提供：

- **组合性**：链式组装组件
- **流式支持**：自动处理流式输出
- **并行执行**：自动优化并行调用
- **可观测性**：内置 LangSmith 集成

### 2.2 基本语法

```python
from langchain_core.runnables import RunnableParallel, RunnablePassthrough
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI

# 基础链
chain = prompt | model | output_parser

# 带 RAG 的链
chain = (
    {"context": retriever, "question": RunnablePassthrough()}
    | prompt
    | model
    | StrOutputParser()
)

# 并行执行
chain = RunnableParallel(
    summary=summarize_chain,
    keywords=keyword_chain,
    translation=translate_chain
)
```

### 2.3 最佳实践

**✅ 使用 LCEL 而非传统 Chain 类**

```python
# ❌ 旧方式（不推荐）
from langchain.chains import LLMChain
chain = LLMChain(llm=model, prompt=prompt)

# ✅ 新方式（推荐）
chain = prompt | model | output_parser
```

**✅ 使用 RunnablePassthrough 传递数据**

```python
# 保留原始输入
chain = {
    "original": RunnablePassthrough(),
    "processed": preprocessing_chain
} | merge_chain
```

**✅ 使用 RunnableLambda 包装自定义函数**

```python
from langchain_core.runnables import RunnableLambda

def format_docs(docs):
    return "\n\n".join(doc.page_content for doc in docs)

chain = {
    "context": retriever | RunnableLambda(format_docs),
    "question": RunnablePassthrough()
} | prompt | model
```

---

## 三、Prompt 最佳实践

### 3.1 使用 ChatPromptTemplate

```python
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder

# 基础模板
prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一个专业的{role}。"),
    ("human", "{input}")
])

# 带历史消息的模板
prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一个有帮助的助手。"),
    MessagesPlaceholder(variable_name="chat_history"),
    ("human", "{input}")
])
```

### 3.2 Prompt 模板设计原则

**✅ 变量命名清晰**

```python
# ❌ 模糊
prompt = ChatPromptTemplate.from_messages([
    ("human", "{x}")
])

# ✅ 清晰
prompt = ChatPromptTemplate.from_messages([
    ("human", "请解释以下概念：{concept}，用{language}回答")
])
```

**✅ System Prompt 完整规范**

```python
SYSTEM_PROMPT = """
你是一个专业的代码审查助手。

## 职责
- 审查代码质量和安全性
- 提供具体的改进建议
- 遵循项目的编码规范

## 输出格式
1. 问题列表（带代码位置）
2. 改进建议
3. 重构示例（如需要）

## 约束
- 不要修改业务逻辑
- 保持代码风格一致
"""
```

### 3.3 Few-Shot 示例

```python
from langchain_core.prompts import FewShotChatMessagePromptTemplate

examples = [
    {"input": "高兴", "output": "😊 心情不错！"},
    {"input": "难过", "output": "😢 需要安慰吗？"},
]

example_prompt = ChatPromptTemplate.from_messages([
    ("human", "{input}"),
    ("ai", "{output}")
])

few_shot_prompt = FewShotChatMessagePromptTemplate(
    example_prompt=example_prompt,
    examples=examples,
)

final_prompt = ChatPromptTemplate.from_messages([
    ("system", "你是情感分析助手。"),
    few_shot_prompt,
    ("human", "{input}")
])
```

---

## 四、RAG（检索增强生成）最佳实践

### 4.1 标准架构

```
文档 → 分割 → 向量化 → 存储 → 检索 → 重排序 → 生成
```

### 4.2 文档分割策略

```python
from langchain_text_splitters import RecursiveCharacterTextSplitter, MarkdownHeaderTextSplitter

# 通用分割器
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=200,  # 重叠避免信息丢失
    length_function=len,
    separators=["\n\n", "\n", "。", "！", "？", " ", ""]
)

# Markdown 结构化分割
markdown_splitter = MarkdownHeaderTextSplitter(
    headers_to_split_on=[
        ("#", "header1"),
        ("##", "header2"),
        ("###", "header3"),
    ]
)

# 代码分割
from langchain_text_splitters import Language
code_splitter = RecursiveCharacterTextSplitter.from_language(
    language=Language.PYTHON,
    chunk_size=1000,
    chunk_overlap=100
)
```

**最佳实践：**

- **chunk_size**：根据模型上下文窗口和检索精度平衡，通常 500-1500
- **chunk_overlap**：10-20% 的 chunk_size，确保上下文连贯
- **保留元数据**：来源、标题、页码等

### 4.3 向量存储选择

| 向量库 | 适用场景 | 特点 |
|--------|----------|------|
| Chroma | 开发/原型 | 轻量、易用、本地 |
| FAISS | 大规模本地 | Facebook 开源、高性能 |
| Pinecone | 生产环境 | 托管服务、可扩展 |
| Weaviate | 企业级 | 混合检索、GraphQL |
| Milvus | 大规模 | 高性能、云原生 |
| Qdrant | 生产环境 | Rust 实现、高效 |

### 4.4 检索策略

**基础检索：**

```python
from langchain_community.vectorstores import Chroma
from langchain_openai import OpenAIEmbeddings

vectorstore = Chroma.from_documents(documents, OpenAIEmbeddings())
retriever = vectorstore.as_retriever(
    search_type="similarity",  # 或 "mmr", "similarity_score_threshold"
    search_kwargs={"k": 4}
)
```

**MMR（最大边际相关性）检索：**

```python
# 平衡相关性和多样性
retriever = vectorstore.as_retriever(
    search_type="mmr",
    search_kwargs={
        "k": 4,
        "fetch_k": 20,  # 先获取更多候选
        "lambda_mult": 0.5  # 多样性权重
    }
)
```

**带阈值过滤：**

```python
retriever = vectorstore.as_retriever(
    search_type="similarity_score_threshold",
    search_kwargs={
        "k": 4,
        "score_threshold": 0.8  # 只返回高相关结果
    }
)
```

### 4.5 重排序（Reranking）

```python
from langchain.retrievers import ContextualCompressionRetriever
from langchain.retrievers.document_compressors import CrossEncoderReranker
from langchain_community.cross_encoders import HuggingFaceCrossEncoder

# 使用 Cross-Encoder 重排序
base_retriever = vectorstore.as_retriever(search_kwargs={"k": 20})
compressor = CrossEncoderReranker(
    model=HuggingFaceCrossEncoder(model_name="BAAI/bge-reranker-base"),
    top_n=4
)
retriever = ContextualCompressionRetriever(
    base_compressor=compressor,
    base_retriever=base_retriever
)
```

### 4.6 完整 RAG 链

```python
from langchain_core.runnables import RunnableParallel, RunnablePassthrough
from langchain_core.output_parsers import StrOutputParser

def format_docs(docs):
    return "\n\n---\n\n".join(doc.page_content for doc in docs)

# RAG 链
rag_chain = (
    {
        "context": retriever | RunnableLambda(format_docs),
        "question": RunnablePassthrough()
    }
    | prompt
    | model
    | StrOutputParser()
)

# 带来源追踪
rag_chain_with_sources = RunnableParallel(
    answer=rag_chain,
    sources=lambda x: [doc.metadata for doc in retriever.invoke(x)]
)
```

---

## 五、Agent 最佳实践

### 5.1 工具设计

**✅ 工具定义规范**

```python
from langchain_core.tools import tool

@tool
def search_database(query: str, limit: int = 10) -> str:
    """
    搜索公司内部知识库。
    
    Args:
        query: 搜索关键词
        limit: 返回结果数量，默认10
        
    Returns:
        搜索结果的字符串表示
    """
    results = db.search(query, limit=limit)
    return format_results(results)
```

**工具设计原则：**

| 原则 | 说明 |
|------|------|
| 单一职责 | 每个工具只做一件事 |
| 清晰描述 | LLM 能理解何时使用 |
| 参数验证 | 使用 Pydantic 定义类型 |
| 错误处理 | 返回有意义的错误信息 |
| 幂等性 | 相同输入产生相同结果 |

**❌ 不好的工具设计：**

```python
@tool
def do_stuff(action: str) -> str:
    """执行各种操作"""
    # 职责不清晰，LLM 无法判断何时使用
    pass
```

### 5.2 Agent 类型选择

| Agent 类型 | 适用场景 | 特点 |
|-----------|----------|------|
| **Tool Calling Agent** | 工具调用 | 推荐，支持 Function Calling |
| **ReAct Agent** | 通用推理 | 思考-行动-观察循环 |
| **Structured Chat Agent** | 多工具协作 | 支持多输入工具 |
| **OpenAI Functions Agent** | OpenAI 模型 | 原生 Function Calling |

### 5.3 使用 LangGraph 构建 Agent

```python
from langgraph.prebuilt import create_react_agent
from langgraph.checkpoint.memory import MemorySaver

# 创建带记忆的 Agent
agent = create_react_agent(
    model=model,
    tools=[search_tool, calculator_tool],
    checkpointer=MemorySaver()
)

# 执行
config = {"configurable": {"thread_id": "conversation-1"}}
result = agent.invoke(
    {"messages": [("user", "帮我分析这个问题")]},
    config=config
)
```

### 5.4 Human-in-the-Loop

```python
from langgraph.prebuilt import create_react_agent
from langgraph.checkpoint.memory import MemorySaver

# 创建 Agent
agent = create_react_agent(
    model=model,
    tools=[sensitive_operation_tool],
    checkpointer=MemorySaver(),
    interrupt_before=["tools"]  # 工具调用前暂停
)

# 执行到暂停点
result = agent.invoke(
    {"messages": [("user", "删除文件 test.txt")]},
    config=config
)

# 人工审核后继续
if user_approves():
    result = agent.invoke(None, config=config)
```

---

## 六、Memory 最佳实践

### 6.1 记忆类型

| 类型 | 用途 | 实现 |
|------|------|------|
| **短期记忆** | 当前会话 | 消息历史 |
| **长期记忆** | 跨会话 | 向量存储 |
| **实体记忆** | 特定实体 | 知识图谱 |

### 6.2 消息历史管理

```python
from langchain_core.messages import BaseMessage
from langchain_community.chat_message_histories import ChatMessageHistory
from langchain_core.runnables.history import RunnableWithMessageHistory

# 存储历史
store = {}

def get_session_history(session_id: str) -> ChatMessageHistory:
    if session_id not in store:
        store[session_id] = ChatMessageHistory()
    return store[session_id]

# 带历史的链
chain_with_history = RunnableWithMessageHistory(
    chain,
    get_session_history,
    input_messages_key="input",
    history_messages_key="chat_history"
)
```

### 6.3 长期记忆（RAG 风格）

```python
from langchain.memory import VectorStoreRetrieverMemory

# 向量存储作为长期记忆
vectorstore = Chroma(embedding_function=embeddings)
memory = VectorStoreRetrieverMemory(
    retriever=vectorstore.as_retriever(search_kwargs={"k": 3})
)

# 保存重要信息
memory.save_context(
    {"input": "我叫张三"},
    {"output": "好的，我记住了，您叫张三。"}
)

# 记忆会持久化到向量存储
```

### 6.4 记忆窗口策略

```python
from langchain_core.messages import trim_messages

# 限制消息数量
def filter_messages(messages: list[BaseMessage]) -> list[BaseMessage]:
    return trim_messages(
        messages,
        max_tokens=4000,
        strategy="last",  # 保留最近的
        token_counter=len,  # 或使用实际 tokenizer
        include_system=True,
        start_on="human"
    )

chain = filter_messages | prompt | model
```

---

## 七、输出解析最佳实践

### 7.1 结构化输出

```python
from langchain_core.output_parsers import PydanticOutputParser
from pydantic import BaseModel, Field

class Person(BaseModel):
    name: str = Field(description="姓名")
    age: int = Field(description="年龄")
    interests: list[str] = Field(description="兴趣爱好")

parser = PydanticOutputParser(pydantic_object=Person)

# 使用解析器
prompt = ChatPromptTemplate.from_messages([
    ("system", "提取人物信息。\n{format_instructions}"),
    ("human", "{input}")
]).partial(format_instructions=parser.get_format_instructions())

chain = prompt | model | parser
```

### 7.2 自动修复解析错误

```python
from langchain.output_parsers import OutputFixingParser

# 自动修复解析失败
fixing_parser = OutputFixingParser.from_llm(
    parser=parser,
    llm=model
)

chain = prompt | model | fixing_parser
```

### 7.3 带重试的解析

```python
from langchain.output_parsers import RetryWithErrorOutputParser

retry_parser = RetryWithErrorOutputParser.from_llm(
    parser=parser,
    llm=model
)

# 解析失败时会带上错误信息重试
```

---

## 八、错误处理与容错

### 8.1 重试机制

```python
from langchain_core.runnables import RunnableRetry

# 自动重试
chain_with_retry = chain.with_retry(
    stop_after_attempt=3,
    wait_exponential_jitter=True,
    retry_if_exception_type=(RateLimitError, TimeoutError)
)
```

### 8.2 降级策略

```python
from langchain_core.runnables import RunnableFallback

# 降级到更便宜的模型
expensive_chain = expensive_prompt | gpt4 | parser
fallback_chain = simple_prompt | gpt35 | parser

chain = expensive_chain.with_fallbacks([fallback_chain])
```

### 8.3 超时控制

```python
from langchain_core.runnables import RunnableConfig

# 设置超时
result = chain.invoke(
    {"input": "..."},
    config=RunnableConfig(timeout=30)  # 30秒超时
)
```

---

## 九、性能优化

### 9.1 批处理

```python
# 批量处理
results = chain.batch([
    {"input": "问题1"},
    {"input": "问题2"},
    {"input": "问题3"}
])

# 带配置的批处理
results = chain.batch(
    inputs,
    config={"max_concurrency": 5}  # 并发控制
)
```

### 9.2 流式输出

```python
# 流式处理
for chunk in chain.stream({"input": "..."}):
    print(chunk, end="", flush=True)

# 异步流式
async for chunk in chain.astream({"input": "..."}):
    print(chunk, end="", flush=True)
```

### 9.3 缓存

```python
from langchain_core.caches import InMemoryCache
from langchain_core.globals import set_llm_cache

# 设置缓存
set_llm_cache(InMemoryCache())

# 相同请求会命中缓存
```

### 9.4 模型选择策略

```python
# 根据任务复杂度选择模型
def get_model_for_task(complexity: str):
    if complexity == "simple":
        return ChatOpenAI(model="gpt-3.5-turbo")
    elif complexity == "complex":
        return ChatOpenAI(model="gpt-4-turbo")
    else:
        return ChatOpenAI(model="gpt-4o")
```

---

## 十、LangGraph vs LangChain

### 10.1 何时使用 LangGraph

| 场景 | 推荐 |
|------|------|
| 简单链式调用 | LangChain LCEL |
| 需要循环/分支 | LangGraph |
| 复杂状态管理 | LangGraph |
| 多 Agent 协作 | LangGraph |
| Human-in-the-loop | LangGraph |
| 需要持久化和恢复 | LangGraph |

### 10.2 LangGraph 示例

```python
from langgraph.graph import StateGraph, END
from typing import TypedDict

class State(TypedDict):
    messages: list
    next_action: str

def agent_node(state: State) -> State:
    # Agent 逻辑
    return {"messages": [...], "next_action": "tools"}

def tools_node(state: State) -> State:
    # 工具执行
    return {"messages": [...], "next_action": "agent"}

# 构建图
graph = StateGraph(State)
graph.add_node("agent", agent_node)
graph.add_node("tools", tools_node)
graph.add_edge("agent", "tools")
graph.add_conditional_edges("tools", lambda s: s["next_action"])
graph.set_entry_point("agent")

app = graph.compile()
```

---

## 十一、常见反模式

### ❌ 反模式 1：过度复杂的 Chain

```python
# ❌ 嵌套太深
chain = (
    (chain1 | chain2 | (chain3 | chain4))
    | (chain5 | (chain6 | chain7))
)

# ✅ 拆分为独立组件
def build_pipeline():
    preprocess = preprocess_chain
    process = process_chain
    postprocess = postprocess_chain
    return preprocess | process | postprocess
```

### ❌ 反模式 2：工具描述不清晰

```python
# ❌ LLM 无法理解何时使用
@tool
def process(data: str) -> str:
    """处理数据"""
    pass

# ✅ 描述清晰
@tool
def analyze_sentiment(text: str) -> str:
    """
    分析文本的情感倾向。
    当用户需要了解文本是正面、负面还是中性时使用。
    
    Args:
        text: 需要分析的文本内容
        
    Returns:
        情感分析结果：positive/negative/neutral
    """
    pass
```

### ❌ 反模式 3：忽略错误处理

```python
# ❌ 直接调用，无错误处理
result = chain.invoke({"input": user_input})

# ✅ 完善的错误处理
try:
    result = chain.invoke({"input": user_input})
except RateLimitError:
    result = fallback_chain.invoke({"input": user_input})
except Exception as e:
    logger.error(f"Chain failed: {e}")
    result = "抱歉，处理您的请求时出现问题。"
```

### ❌ 反模式 4：无限制的消息历史

```python
# ❌ 历史无限增长
memory = ConversationBufferMemory()

# ✅ 限制窗口
memory = ConversationBufferWindowMemory(k=10)
# 或使用 trim_messages 过滤
```

---

## 十二、生产部署清单

### 12.1 可观测性

- [ ] 集成 LangSmith 或其他追踪工具
- [ ] 记录请求/响应日志
- [ ] 监控延迟和成功率
- [ ] 设置告警阈值

### 12.2 安全性

- [ ] 输入验证和清洗
- [ ] 敏感信息过滤
- [ ] 速率限制
- [ ] 用户权限控制

### 12.3 可靠性

- [ ] 重试机制
- [ ] 降级策略
- [ ] 超时控制
- [ ] 熔断机制

### 12.4 成本控制

- [ ] Token 使用监控
- [ ] 缓存策略
- [ ] 模型选择优化
- [ ] 请求批处理

---

## 十三、参考资源

- **官方文档**: https://python.langchain.com/docs/
- **LangGraph 文档**: https://langchain-ai.github.io/langgraph/
- **LangSmith**: https://www.langchain.com/langsmith
- **Cookbook**: https://github.com/langchain-ai/langchain/tree/master/cookbook

---

## 十四、更新日志

| 日期 | 更新内容 |
|------|----------|
| 2026-03-27 | 初始版本，整理 LCEL、RAG、Agent、Memory 等最佳实践 |