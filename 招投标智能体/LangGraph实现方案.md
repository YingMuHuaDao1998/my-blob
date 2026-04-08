# 自动生成投标文件 — LangGraph 实现方案

> 基于 LangGraph + OpenAI Assistants / ChatModel 实现
> 更新：2026-04-08

---

## 一、整体架构

```
用户上传招标文件 + 投标模板
          │
          ▼
┌─────────────────────────────────────────┐
│           StateGraph                      │
│                                         │
│   start ──▶ init_session ──▶ fill_loop │
│                            ▲           │
│                            │ (循环 N 次) │
│                            ▼           │
│                        save_result ──▶ end │
└─────────────────────────────────────────┘
```

**三个核心节点：**
1. `init_session`：创建 LangChain Runnable，将招标文件+模板内容发送给 LLM，建立会话上下文
2. `fill_loop`：ConditionNode，遍历所有章节，对每个章节调用 LLM 填充
3. `save_result`：收集所有填充结果，拼装文档，写入 OSS，触发 callback

---

## 二、安装依赖

```bash
cd backend
pip install langchain langgraph openai python-docx oss2
```

---

## 三、完整实现

### 3.1 项目结构

```
backend/app/
├── agents/
│   ├── __init__.py
│   ├── bid_generator.py      # LangGraph 定义
│   ├── nodes.py               # 节点函数
│   ├── state.py               # 状态定义
│   └── prompts.py             # Prompt 模板
├── services/
│   ├── __init__.py
│   ├── file_parser.py         # PDF/DOCX → Markdown
│   ├── storage.py             # OSS 对象存储
│   └── callback.py            # Callback 回调
└── api/
    ├── __init__.py
    └── bid_generate.py         # FastAPI 路由
```

### 3.2 状态定义 `state.py`

```python
from typing import Annotated, Literal
from langgraph.graph import add_node
from pydantic import BaseModel, Field
from datetime import date


class BidGenerationState(BaseModel):
    """LangGraph 状态：贯穿整个填充流程"""

    bid_id: int
    user_id: int

    # 文件内容
    tender_text: str = ""
    template_text: str = ""
    tender_parsed_summary: str = ""  # 步骤1的解析摘要

    # 会话上下文（LLM Memory / ConversationChain）
    messages: list[dict] = Field(default_factory=list)

    # 章节填充状态
    section_index: int = 0
    sections: list[dict] = Field(default_factory=list)  # [{id, title, placeholder, filled_content, status}]
    filled_results: list[str] = Field(default_factory=list)

    # 结果
    result_url: str = ""
    error: str = ""

    # 控制
    status: Literal["init", "filling", "completed", "failed"] = "init"
```

### 3.3 Prompt 模板 `prompts.py`

```python
SYSTEM_TENDER_ANALYZER = """
你是一个专业的招投标文档分析专家。用户将提供一份招标文件（markdown格式）。
请提取以下关键信息并以结构化方式输出：
- 项目名称、甲方、标的金额
- 工期要求
- 资质要求（证书、认证等）
- 评分要点（技术标、商务标分值分布）
- 关键技术要求
- 特殊条款

输出格式：JSON
"""

SYSTEM_CHAPTER_FILLER = """
你是一个专业的投标文件撰写助手。用户有一份投标文件模板（markdown格式）和招标文件的解析结果。
当前任务是为模板的指定章节（section）填充内容。

填充规则：
1. 严格遵循招标文件的各项要求
2. 内容必须真实、可信，数据有据可查
3. 符合政府招标投标文件的行文规范（正式、严谨）
4. 只输出该章节的填充内容，不要输出其他章节
5. 如果模板中有占位符（如【】），请替换为实际内容

参考素材库信息（如有）：{material_context}
"""

USER_CHAPTER_FILL_TEMPLATE = """
【招标文件解析摘要】
{tender_summary}

【当前章节】
{chapter_title}

【模板原文/占位符】
{chapter_placeholder}

请输出填充后的章节内容（仅内容，不要其他说明）：
"""
```

### 3.4 节点函数 `nodes.py`

```python
import json
from .state import BidGenerationState
from .prompts import SYSTEM_TENDER_ANALYZER, SYSTEM_CHAPTER_FILLER, USER_CHAPTER_FILL_TEMPLATE
from app.services.file_parser import extract_text_from_file
from app.services.storage import upload_to_oss


def init_session(state: BidGenerationState, cfg) -> BidGenerationState:
    """
    节点1：初始化会话
    - 解析招标文件 → 生成摘要
    - 解析模板文件 → 提取章节列表
    - 建立会话上下文
    """
    from app.services.llm import llm  # 注入的 LLM 实例

    # 1. 分析招标文件，提取关键信息
    analyzer_chain = SYSTEM_TENDER_ANALYZER | llm | StrOutputParser()
    tender_summary_raw = analyzer_chain.invoke({"input": state.tender_text})
    try:
        tender_summary = json.loads(tender_summary_raw)
    except Exception:
        # 如果 LLM 返回的不是严格 JSON，降级为纯文本
        tender_summary = {"raw": tender_summary_raw}

    state.tender_parsed_summary = tender_summary

    # 2. 解析模板，提取章节列表
    sections = parse_template_sections(state.template_text)
    state.sections = sections
    state.section_index = 0

    # 3. 在 messages 中建立会话上下文
    system_msg = SYSTEM_CHAPTER_FILLER.format(
        material_context="见各章节素材"
    )
    state.messages = [
        {"role": "system", "content": system_msg},
        {"role": "user", "content": f"【招标文件摘要】\n{json.dumps(tender_summary, ensure_ascii=False)}"},
    ]
    state.status = "filling"
    return state


def parse_template_sections(template_text: str) -> list[dict]:
    """
    按 ## 一级标题拆分模板章节。
    生产环境：用 python-docx 解析真实 .docx 文件的章节结构。
    """
    import re
    chapters = []
    parts = re.split(r"(?=^##\s+)", template_text, flags=re.MULTILINE)
    for i, part in enumerate(parts):
        part = part.strip()
        if not part:
            continue
        lines = part.split("\n", 1)
        title = lines[0].replace("##", "").strip()
        body = lines[1].strip() if len(lines) > 1 else ""
        chapters.append({
            "id": f"section_{i}",
            "index": i,
            "title": title or f"第{i+1}节",
            "placeholder": body[:300] if body else "",
            "filled_content": "",
            "status": "pending",
        })
    return chapters


def fill_single_chapter(state: BidGenerationState, cfg) -> BidGenerationState:
    """
    节点2（可循环）：填充单个章节
    - 从模板取出当前 chapter_index 对应章节
    - 组装 prompt，调用 LLM 填充
    - 更新章节状态和 filled_results
    """
    from app.services.llm import llm

    idx = state.section_index
    if idx >= len(state.sections):
        return state  # 全部完成

    chapter = state.sections[idx]

    # 从素材库检索当前章节相关的素材
    from app.services.material_retriever import retrieve_materials
    material_context = retrieve_materials(
        query=chapter["title"],
        category_hint=chapter["title"],
        top_k=5,
    )

    # 组装 prompt
    user_prompt = USER_CHAPTER_FILL_TEMPLATE.format(
        tender_summary=json.dumps(state.tender_parsed_summary, ensure_ascii=False),
        chapter_title=chapter["title"],
        chapter_placeholder=chapter["placeholder"],
    )

    # 使用 ChatOpenAI（带记忆）调用
    chain = llm  # 直接用 llm.invoke
    response = chain.invoke(state.messages + [{"role": "user", "content": user_prompt}])

    filled_content = response.content if hasattr(response, "content") else str(response)

    # 更新状态
    state.sections[idx]["filled_content"] = filled_content
    state.sections[idx]["status"] = "filled"
    state.filled_results.append(filled_content)

    # 将本次填充内容追加到会话历史（用于后续章节的上下文连贯性）
    state.messages.append({"role": "assistant", "content": filled_content})

    return state


def should_continue(state: BidGenerationState) -> str:
    """
    条件边：判断是否继续填充下一个章节
    """
    if state.section_index >= len(state.sections):
        return "save_result"
    state.section_index += 1
    return "fill_chapter"


def save_result(state: BidGenerationState, cfg) -> BidGenerationState:
    """
    节点3：保存结果
    - 拼装完整 markdown 文档
    - 写入 OSS
    - 触发 callback_url
    """
    import uuid
    from app.services.storage import upload_to_oss
    from app.services.callback import send_callback

    # 拼装完整文档
    full_doc = f"# 投标文件\n\n"
    for ch in state.sections:
        full_doc += f"## {ch['title']}\n\n{ch.get('filled_content', '[未填充]')}\n\n"

    # 上传 OSS
    filename = f"bid_{state.bid_id}_generated_{uuid.uuid4().hex[:8]}.md"
    result_url = upload_to_oss(filename, full_doc.encode("utf-8"))
    state.result_url = result_url

    # 触发 callback
    if cfg.get("callback_url"):
        send_callback(cfg["callback_url"], {
            "bid_id": state.bid_id,
            "result_url": result_url,
            "sections_count": len(state.sections),
        })

    state.status = "completed"
    return state
```

### 3.5 LangGraph 定义 `bid_generator.py`

```python
from langgraph.graph import StateGraph, END
from .state import BidGenerationState
from .nodes import (
    init_session,
    fill_single_chapter,
    should_continue,
    save_result,
)


def build_bid_generation_graph():
    """
    构建投标文件自动生成的 LangGraph
    """
    workflow = StateGraph(BidGenerationState)

    # 节点
    workflow.add_node("init_session", init_session)
    workflow.add_node("fill_chapter", fill_single_chapter)
    workflow.add_node("save_result", save_result)

    # 边
    workflow.set_entry_point("init_session")

    # fill_chapter 完成后，判断是继续填充还是保存
    workflow.add_conditional_edges(
        "fill_chapter",
        should_continue,
        {
            "fill_chapter": "fill_chapter",  # 继续填充下一章节（循环）
            "save_result": "save_result",    # 全部完成，保存结果
        },
    )

    workflow.add_edge("init_session", "fill_chapter")
    workflow.add_edge("save_result", END)

    return workflow.compile()


# 全局单例，运行时实例化
_bid_graph = None

def get_bid_graph():
    global _bid_graph
    if _bid_graph is None:
        _bid_graph = build_bid_generation_graph()
    return _bid_graph


def run_bid_generation(
    bid_id: int,
    user_id: int,
    tender_text: str,
    template_text: str,
    callback_url: str | None = None,
) -> dict:
    """
    对外入口：启动 LangGraph 流程
    """
    graph = get_bid_graph()

    initial_state = BidGenerationState(
        bid_id=bid_id,
        user_id=user_id,
        tender_text=tender_text,
        template_text=template_text,
    )

    # config 用来传递 callback_url 等外部参数
    config = {"callback_url": callback_url} if callback_url else {}

    result = graph.invoke(initial_state, config=config)

    return {
        "status": result.status,
        "result_url": result.result_url,
        "sections_completed": result.section_index,
        "error": result.error,
    }
```

### 3.6 LLM 实例 `services/llm.py`

```python
import os
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(
    model=os.getenv("LLM_MODEL", "gpt-4o"),
    api_key=os.getenv("OPENAI_API_KEY"),
    temperature=0.3,  # 文档生成用低随机性
    max_retries=2,
)
```

### 3.7 FastAPI 路由 `api/bid_generate.py`

```python
from fastapi import APIRouter, Depends, UploadFile, File
from pydantic import BaseModel
from sqlalchemy.orm import Session

from app.core.deps import get_current_user
from app.database import get_db
from app.models.user import User
from app.agents.bid_generator import run_bid_generation
from app.services.file_parser import extract_text_from_file

router = APIRouter(prefix="/api/bid-generate", tags=["bid-generate"])


class StartGenerateRequest(BaseModel):
    bid_id: int
    tender_markdown: str
    template_markdown: str
    callback_url: str | None = None


class StartGenerateResponse(BaseModel):
    status: str
    result_url: str = ""
    sections_count: int = 0


@router.post("/start", response_model=StartGenerateResponse)
async def start_bid_generation(
    req: StartGenerateRequest,
    current_user: User = Depends(get_current_user),
    db: Session = Depends(get_db),
):
    result = run_bid_generation(
        bid_id=req.bid_id,
        user_id=current_user.id,
        tender_text=req.tender_markdown,
        template_text=req.template_markdown,
        callback_url=req.callback_url,
    )
    return StartGenerateResponse(
        status=result["status"],
        result_url=result.get("result_url", ""),
        sections_count=result.get("sections_completed", 0),
    )
```

---

## 四、与直接用 OpenAI Assistant API 的对比

| 对比项 | OpenAI Assistant API（直接方案） | LangGraph 方案 |
|--------|-------------------------------|----------------|
| 代码量 | 少（约 200 行） | 多（约 400-500 行）|
| 上下文管理 | Thread 自动管理 | messages 需手动维护 |
| 循环控制 | 前端驱动 | 图的 conditional_edges 自动循环 |
| 状态持久化 | OpenAI 服务端 | 可结合 Checkpointing 持久化 |
| 多 Agent 协作 | 需自己拼接 | LangGraph 原生支持 |
| 监控/调试 | OpenAI 后台 | LangGraph 有可视化 debugger |
| 依赖 | openai 包 | langchain + langgraph（较重）|

---

## 五、建议

**如果这个流程比较确定、不需要经常改，用 OpenAI Assistant API 更轻量简洁。**

**如果未来会有这些需求，推荐 LangGraph：**
- 步骤之间要加审核节点（人工确认后再继续）
- 多个 Agent 协作（一个分析招标文件、一个写技术方案、一个写商务标）
- 需要中途暂停/恢复生成流程
- 流程图需要产品/运营可见可改

你们现在的场景其实 OpenAI Assistant API 就够了，LangGraph 有点杀鸡用牛刀。当然如果团队想练手或者后续要扩展，LangGraph 架构更清晰。
