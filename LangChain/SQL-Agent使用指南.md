# LangChain SQL Agent 使用指南

## 概述

SQL Agent 是 LangChain 提供的一个强大的智能代理，能够自动回答关于 SQL 数据库的问题。它使用大语言模型（LLM）来理解用户问题、生成 SQL 查询、执行查询并返回结果。

### 核心能力

- **自动推理**：理解自然语言问题并转换为 SQL 查询
- **错误自修复**：当 SQL 执行失败时自动修正查询
- **多步推理**：先获取表结构，再生成精确查询
- **安全可控**：支持人工审核机制

## 工作原理

### 完整工作流程

```mermaid
sequenceDiagram
    participant U as 用户
    participant A as SQL Agent
    participant T as Tools
    participant DB as 数据库
    
    U->>A: 提出问题
    A->>T: 获取可用表列表
    T->>DB: 查询表名
    DB-->>T: 返回表列表
    T-->>A: 表信息
    
    A->>A: 决定相关表
    A->>T: 获取相关表 schema
    T->>DB: 查询表结构
    DB-->>T: 返回 schema
    T-->>A: schema 信息
    
    A->>A: 生成 SQL 查询
    A->>T: 检查 SQL 正确性
    T-->>A: SQL 验证结果
    
    A->>T: 执行 SQL
    T->>DB: 运行查询
    
    alt 查询成功
        DB-->>T: 查询结果
        T-->>A: 结果数据
        A-->>U: 自然语言回答
    else 查询失败
        DB-->>T: 错误信息
        T-->>A: 错误详情
        A->>A: 分析错误并修正 SQL
        A->>T: 重新执行修正后的 SQL
    end
```

### Agent 执行步骤

SQL Agent 的工作分为以下 8 个步骤：

1. **获取可用表**：从数据库获取所有可用的表和 schema
2. **决定相关表**：判断哪些表与用户问题相关
3. **获取表结构**：获取相关表的 schema 定义
4. **生成查询**：基于问题和 schema 生成 SQL 查询
5. **查询检查**：使用 LLM 检查 SQL 的常见错误
6. **执行查询**：执行 SQL 并返回结果
7. **错误修正**：如果执行失败，根据错误信息修正 SQL
8. **生成回答**：基于查询结果生成自然语言回答

## 核心概念

### 1. Tools（工具）

SQL Agent 使用以下工具与数据库交互：

| 工具名称 | 功能描述 |
|---------|---------|
| `sql_db_query` | 执行 SQL 查询并返回结果 |
| `sql_db_schema` | 获取指定表的结构和示例数据 |
| `sql_db_list_tables` | 获取数据库中所有表的列表 |
| `sql_db_query_checker` | 在执行前检查 SQL 查询的正确性 |

### 2. Agents（代理）

使用 ReAct（Reasoning + Acting）模式的智能代理：
- 理解用户意图
- 决定使用哪个工具
- 解析工具返回结果
- 循环执行直到完成任务

### 3. Human-in-the-loop（人工介入）

支持在执行关键操作前暂停，等待人工审核：
- 检查 SQL 查询是否符合预期
- 防止危险操作
- 提供审批或拒绝的机制

## 使用方法

### 安装依赖

```bash
pip install langchain langgraph langchain-community
```

### 完整示例

#### Step 1: 选择 LLM

```python
import os
from langchain.chat_models import init_chat_model

# 设置 API Key
os.environ["OPENAI_API_KEY"] = "sk-..."

# 初始化模型（需要支持 tool-calling）
model = init_chat_model("gpt-4o")
```

**支持的 LLM**：
- OpenAI (GPT-4o, GPT-4)
- Anthropic (Claude)
- Azure OpenAI
- Google Gemini
- AWS Bedrock
- HuggingFace

#### Step 2: 配置数据库

```python
from langchain_community.utilities import SQLDatabase

# 连接数据库
db = SQLDatabase.from_uri("sqlite:///Chinook.db")

# 查看数据库信息
print(f"Dialect: {db.dialect}")
print(f"Available tables: {db.get_usable_table_names()}")
```

**支持的数据源**：
- SQLite
- PostgreSQL
- MySQL
- Oracle
- SQL Server

#### Step 3: 添加工具

```python
from langchain_community.agent_toolkits import SQLDatabaseToolkit

# 创建工具包
toolkit = SQLDatabaseToolkit(db=db, llm=model)
tools = toolkit.get_tools()

# 查看可用工具
for tool in tools:
    print(f"{tool.name}: {tool.description}\n")
```

#### Step 4: 创建 Agent

```python
from langchain.agents import create_agent

# 定义系统提示词
system_prompt = """
You are an agent designed to interact with a SQL database.
Given an input question, create a syntactically correct {dialect} query to run,
then look at the results of the query and return the answer. Unless the user
specifies a specific number of examples they wish to obtain, always limit your
query to at most {top_k} results.

You can order the results by a relevant column to return the most interesting
examples in the database. Never query for all the columns from a specific table,
only ask for the relevant columns given the question.

You MUST double check your query before executing it. If you get an error while
executing a query, rewrite the query and try again.

DO NOT make any DML statements (INSERT, UPDATE, DELETE, DROP etc.) to the
database.

To start you should ALWAYS look at the tables in the database to see what you
can query. Do NOT skip this step.
Then you should query the schema of the most relevant tables.
""".format(
    dialect=db.dialect,
    top_k=5,
)

# 创建 Agent
agent = create_agent(
    model,
    tools,
    system_prompt=system_prompt,
)
```

#### Step 5: 运行 Agent

```python
# 提出问题
question = "Which genre on average has the longest tracks?"

# 流式执行
for step in agent.stream(
    {"messages": [{"role": "user", "content": question}]},
    stream_mode="values",
):
    step["messages"][-1].pretty_print()
```

**执行流程示例**：

```
================================ Human Message =================================
Which genre on average has the longest tracks?
================================== Ai Message ==================================
Tool Calls:
  sql_db_list_tables (call_BQsWg8P65apHc8BTJ1NPDvnM)
================================= Tool Message =================================
Name: sql_db_list_tables
Album, Artist, Customer, Employee, Genre, Invoice, InvoiceLine, MediaType, Playlist, PlaylistTrack, Track
================================== Ai Message ==================================
Tool Calls:
  sql_db_schema (call_i89tjKECFSeERbuACYm4w0cU)
  Args:
    table_names: Track, Genre
================================= Tool Message =================================
Name: sql_db_schema
[Schema information...]
================================== Ai Message ==================================
Tool Calls:
  sql_db_query_checker (call_G64yYm6R6UauiVPCXJZMA49b)
  Args:
    query: SELECT Genre.Name, AVG(Track.Milliseconds) AS AverageLength 
           FROM Track INNER JOIN Genre ON Track.GenreId = Genre.GenreId 
           GROUP BY Genre.Name ORDER BY AverageLength DESC LIMIT 5;
================================= Tool Message =================================
Name: sql_db_query_checker
[Validated query...]
================================== Ai Message ==================================
Tool Calls:
  sql_db_query (call_AnO3SrhD0ODJBxh6dHMwvHwZ)
================================= Tool Message =================================
Name: sql_db_query
[('Sci Fi & Fantasy', 2911783.0384615385), ('Science Fiction', 2625549.076923077), ...]
================================== Ai Message ==================================
On average, the genre with the longest tracks is "Sci Fi & Fantasy" with an 
average track length of approximately 2,911,783 milliseconds.
```

### Human-in-the-Loop 实现

#### 配置人工审核

```python
from langchain.agents import create_agent
from langchain.agents.middleware import HumanInTheLoopMiddleware
from langgraph.checkpoint.memory import InMemorySaver

# 创建带人工审核的 Agent
agent = create_agent(
    model,
    tools,
    system_prompt=system_prompt,
    middleware=[
        HumanInTheLoopMiddleware(
            interrupt_on={"sql_db_query": True},
            description_prefix="Tool execution pending approval",
        ),
    ],
    checkpointer=InMemorySaver(),
)
```

#### 执行并暂停审核

```python
question = "Which genre on average has the longest tracks?"
config = {"configurable": {"thread_id": "1"}}

# 第一次执行：会暂停等待审核
for step in agent.stream(
    {"messages": [{"role": "user", "content": question}]},
    config,
    stream_mode="values",
):
    if "__interrupt__" in step:
        print("INTERRUPTED:")
        interrupt = step["__interrupt__"][0]
        for request in interrupt.value["action_requests"]:
            print(request["description"])
    elif "messages" in step:
        step["messages"][-1].pretty_print()
```

**输出**：

```
INTERRUPTED:
Tool execution pending approval
Tool: sql_db_query
Args: {'query': 'SELECT g.Name AS Genre, AVG(t.Milliseconds) AS AvgTrackLength 
       FROM Track t JOIN Genre g ON t.GenreId = g.GenreId 
       GROUP BY g.Name ORDER BY AvgTrackLength DESC LIMIT 1;'}
```

#### 批准并继续执行

```python
from langgraph.types import Command

# 批准并继续执行
for step in agent.stream(
    Command(resume={"decisions": [{"type": "approve"}]}),
    config,
    stream_mode="values",
):
    if "messages" in step:
        step["messages"][-1].pretty_print()
```

## 工具详解

### sql_db_query

**功能**：执行 SQL 查询并返回结果

**输入**：详细的、正确的 SQL 查询语句

**输出**：查询结果或错误信息

**使用建议**：
- 如果查询错误，会返回错误信息
- 遇到 "Unknown column" 错误时，使用 `sql_db_schema` 查询正确的字段名
- 执行前先用 `sql_db_query_checker` 检查

### sql_db_schema

**功能**：获取表的结构和示例数据

**输入**：逗号分隔的表名列表（如：`table1, table2, table3`）

**输出**：表的 schema 定义和示例行数据

**使用建议**：
- 先用 `sql_db_list_tables` 确认表是否存在
- 只查询相关的表，避免获取过多信息

### sql_db_list_tables

**功能**：获取数据库中所有表的列表

**输入**：空字符串

**输出**：逗号分隔的表名列表

**使用建议**：
- 这是 Agent 的第一步操作
- 用于了解数据库中有哪些表可用

### sql_db_query_checker

**功能**：在执行前检查 SQL 查询的正确性

**输入**：SQL 查询语句

**输出**：验证后的查询或修正建议

**使用建议**：
- **总是**在执行查询前使用此工具
- 帮助发现常见错误
- 提高查询成功率

## 系统提示词最佳实践

### 推荐模板

```python
system_prompt = """
You are an agent designed to interact with a SQL database.

## 基本规则
- Given an input question, create a syntactically correct {dialect} query
- Always limit results to at most {top_k} rows unless user specifies otherwise
- Only query relevant columns, never use SELECT *
- DO NOT make DML statements (INSERT, UPDATE, DELETE, DROP)

## 执行流程
1. ALWAYS start by listing available tables
2. Query schema of relevant tables
3. Generate and check the query
4. Execute the query
5. If error occurs, analyze and retry

## 安全要求
- Double check query before executing
- If error occurs, rewrite and try again
- Never modify database structure
"""
```

### 关键配置参数

| 参数 | 说明 | 推荐值 |
|------|------|--------|
| `dialect` | 数据库方言 | `db.dialect` |
| `top_k` | 默认返回行数 | 5-10 |
| `max_iterations` | 最大迭代次数 | 10-15 |
| `handle_parsing_errors` | 处理解析错误 | True |

## 安全注意事项

### ⚠️ 风险警告

**构建 SQL 数据库问答系统需要执行模型生成的 SQL 查询，这存在固有风险。**

### 安全建议

1. **最小权限原则**
   ```python
   # 给 Agent 的数据库用户只授予 SELECT 权限
   db = SQLDatabase.from_uri(
       "postgresql://readonly_user:pass@localhost/db"
   )
   ```

2. **限制危险操作**
   - 在系统提示中明确禁止 DML 操作
   - 使用只读数据库用户
   - 设置行数限制防止全表扫描

3. **人工审核**
   - 对关键操作启用 human-in-the-loop
   - 审查 SQL 查询后再执行
   - 记录所有查询日志

4. **数据脱敏**
   - 隐藏或脱敏敏感字段
   - 限制可访问的表
   - 使用视图而非直接访问表

### 权限配置示例

```python
# 只允许访问特定表
db = SQLDatabase.from_uri(
    "sqlite:///mydb.db",
    include_tables=["users", "orders"],  # 白名单
    sample_rows_in_table_info=3,  # 限制采样行数
)
```

## 性能优化

### Token 控制

```python
# 限制表和采样行数
db = SQLDatabase.from_uri(
    "postgresql://user:pass@localhost:5432/mydb",
    include_tables=["users", "orders"],  # 只包含特定表
    sample_rows_in_table_info=3,  # 每表采样 3 行
)
```

### 缓存策略

- 缓存相似查询的 SQL
- 缓存表结构信息
- 使用 LangSmith 跟踪和优化

### 异步执行

```python
# 对于长查询，使用异步处理
import asyncio

async def run_agent_async(question):
    result = await agent.ainvoke(
        {"messages": [{"role": "user", "content": question}]}
    )
    return result
```

## 监控与调试

### 使用 LangSmith

```bash
# 设置环境变量
export LANGSMITH_TRACING="true"
export LANGSMITH_API_KEY="..."
```

### 查看执行轨迹

在 LangSmith 中可以查看：
- 执行步骤
- 工具调用
- LLM 提示词
- 返回结果
- 错误信息

### 调试模式

```python
# 启用详细输出
agent = create_agent(
    model,
    tools,
    system_prompt=system_prompt,
    verbose=True,  # 显示思考过程
)
```

## 常见问题

### 1. Token 超限

**原因**：表结构过大，schema 信息占用过多 token

**解决方案**：
```python
db = SQLDatabase.from_uri(
    "sqlite:///mydb.db",
    include_tables=["table1", "table2"],  # 限制表
    sample_rows_in_table_info=2,  # 减少采样行
)
```

### 2. 查询错误

**原因**：生成的 SQL 不符合语法或引用了不存在的列

**解决方案**：
- 使用 `sql_db_query_checker` 工具
- 在系统提示中强调"必须先查看 schema"
- 启用错误重试机制

### 3. 性能慢

**原因**：多次 LLM 调用导致延迟

**解决方案**：
- 使用更快的模型（如 GPT-4o-mini）
- 缓存表结构信息
- 减少不必要的工具调用

## 架构图

```mermaid
graph TD
    A[用户问题] --> B[SQL Agent]
    B --> C{获取表列表}
    C --> D[sql_db_list_tables]
    D --> E[决定相关表]
    E --> F[获取表 Schema]
    F --> G[sql_db_schema]
    G --> H[生成 SQL]
    H --> I[检查 SQL]
    I --> J[sql_db_query_checker]
    J --> K{SQL 正确?}
    K -->|否| H
    K -->|是| L[执行查询]
    L --> M[sql_db_query]
    M --> N{执行成功?}
    N -->|否| O[分析错误]
    O --> H
    N -->|是| P[生成回答]
    P --> Q[返回结果]
```

## 进阶用法

### 使用 LangGraph 自定义 Agent

对于更复杂的场景，可以使用 LangGraph 直接实现 SQL Agent：

```python
from langgraph.graph import StateGraph, END

# 定义状态
class AgentState(TypedDict):
    question: str
    query: str
    result: Any
    answer: str

# 构建图
workflow = StateGraph(AgentState)
workflow.add_node("get_tables", get_tables_node)
workflow.add_node("get_schema", get_schema_node)
workflow.add_node("generate_query", generate_query_node)
workflow.add_node("execute_query", execute_query_node)
workflow.add_node("generate_answer", generate_answer_node)

# 定义边
workflow.add_edge("get_tables", "get_schema")
workflow.add_edge("get_schema", "generate_query")
workflow.add_edge("generate_query", "execute_query")
workflow.add_edge("execute_query", "generate_answer")
workflow.add_edge("generate_answer", END)
```

**优势**：
- 更细粒度的控制
- 自定义流程
- 添加复杂逻辑
- 集成其他工具

## 参考资料

- [LangChain SQL Agent 官方文档](https://docs.langchain.com/oss/python/langchain/sql-agent)
- [LangGraph SQL Agent 教程](https://docs.langchain.com/oss/python/langgraph/sql-agent)
- [Human-in-the-Loop 指南](https://docs.langchain.com/oss/python/langchain/human-in-the-loop)
- [LangSmith 追踪文档](https://docs.smith.langchain.com/)
- [ReAct 论文](https://arxiv.org/pdf/2210.03629)

## 总结

LangChain SQL Agent 是一个功能强大的智能数据库查询工具，通过以下特性简化了 Text-to-SQL 的实现：

✅ **自动推理**：理解自然语言并生成 SQL  
✅ **错误自修复**：自动修正查询错误  
✅ **安全可控**：支持人工审核机制  
✅ **易于使用**：几行代码即可实现  
✅ **生产就绪**：支持监控、调试、优化  

适用于内部 BI 工具、数据分析平台、智能问答系统等场景。