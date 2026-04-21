# RAGFlow 知识库配置指南

## 概述

RAGFlow 中的知识库（Dataset）是其核心功能之一。每个知识库作为知识源，将本地上传的文件和 RAGFlow 文件系统中的文件引用解析为真正的"知识"，用于后续的 AI 对话。

> **重要说明**：每次创建知识库时，会在 `root/.knowledgebase` 目录下生成同名文件夹。

## 创建知识库

拥有多个知识库可以构建更灵活、多样化的问答系统。

**创建步骤**：
1. 点击页面顶部的 **Dataset** 标签
2. 点击 **Create dataset**
3. 输入知识库名称，点击 **OK** 确认

创建后会自动进入知识库的 **Configuration** 页面。

---

## 配置知识库

知识库的正确配置对后续 AI 对话至关重要。选择错误的嵌入模型或分块方法会导致语义丢失或答案匹配错误。

### 1. 选择分块方法

RAGFlow 提供多种内置分块模板，以适应不同文档布局和文件格式，确保语义完整性。

#### 分块模板对照表

| 模板 | 描述 | 支持格式 | 适用场景 |
|------|------|----------|----------|
| **General** | 按预设 token 数连续分块 | MD, MDX, DOCX, XLSX, XLS, PPT, PDF, TXT, JPEG, JPG, PNG, TIF, GIF, CSV, JSON, EML, HTML | 通用文档，适用于大多数场景 |
| **Q&A** | 检索相关信息并生成答案回答问题 | XLSX, XLS, CSV/TXT | FAQ 文档、问答对数据 |
| **Resume** | 简历解析（企业版功能） | DOCX, PDF, TXT | HR 简历筛选场景 |
| **Manual** | 产品手册解析 | PDF | 产品说明书、操作手册 |
| **Table** | 使用 TSI 技术高效解析表格数据 | XLSX, XLS, CSV/TXT | 表格数据、数据集 |
| **Paper** | 学术论文解析 | PDF | 科研论文、学术文献 |
| **Book** | 书籍解析 | DOCX, PDF, TXT | 电子书、长篇文档 |
| **Laws** | 法律文档解析 | DOCX, PDF, TXT | 法律条文、法规文件 |
| **Presentation** | 演示文稿解析 | PDF, PPTX | PPT 演讲稿 |
| **Picture** | 图片解析 | JPEG, JPG, PNG, TIF, GIF | 图片内容提取 |
| **One** | 整个文档作为一个分块 | DOCX, XLSX, XLS, PDF, TXT | 需要完整上下文的短文档 |
| **Tag** | 作为其他知识库的标签集 | XLSX, CSV/TXT | 跨知识库标签管理 |

#### 实际应用场景建议

| 场景类型 | 推荐分块方法 | 说明 |
|----------|--------------|------|
| 企业知识库 | General | 适合混合文档类型 |
| FAQ 系统 | Q&A | 专为问答设计，检索精准 |
| 法律合规系统 | Laws | 保持法律条文完整性 |
| 科研论文库 | Paper | 理解学术文档结构 |
| 产品手册库 | Manual | 解析产品说明结构 |
| 表格数据库 | Table | 高效处理表格数据 |
| HR 系统简历库 | Resume | 自动解析简历结构 |

> **提示**：从 v0.21.0 起，RAGFlow 支持 **Ingestion Pipeline**，用于自定义数据摄入和清洗工作流。

> **注意**：你也可以在 Files 页面更改单个文件的分块方法。

---

### 2. 选择嵌入模型

嵌入模型将分块转换为向量。**选择后不可更改**，除非删除知识库中所有现有分块。

#### 关键注意事项

1. **不可更改原则**：确保同一知识库中的所有文件使用相同的嵌入模型，保证它们在同一向量空间中比较。

2. **语言匹配**：某些嵌入模型针对特定语言优化，用于其他语言文档可能影响性能。

3. **推荐做法**：根据文档语言选择合适的嵌入模型。

#### 常用嵌入模型选择建议

| 文档语言 | 推荐模型 |
|----------|----------|
| 中文 | BGE 系列、text2vec-chinese |
| 英文 | OpenAI text-embedding、BGE-base-en |
| 多语言 | multilingual-e5、BGE-m3 |

---

### 3. 上传文件

#### 文件上传方式

1. **直接上传到知识库**
   - 单文件或文件夹批量上传
   - 知识库持有文件副本

2. **通过 RAGFlow 文件系统上传（推荐）**
   - 文件可链接到多个知识库
   - 每个目标知识库持有文件引用
   - 避免误删文件的风险

> **强烈推荐**：先上传文件到 RAGFlow 文件系统，再链接到目标知识库。这样可避免永久删除直接上传到知识库的文件。

#### 支持的文件格式

| 类型 | 格式 |
|------|------|
| 文档 | PDF, DOC, DOCX, TXT, MD, MDX |
| 表格 | CSV, XLSX, XLS |
| 图片 | JPEG, JPG, PNG, TIF, GIF |
| 演示 | PPT, PPTX |

---

### 4. 解析文件

文件解析是知识库配置的核心环节，包含两层含义：
1. 根据文档布局分块
2. 在分块上构建嵌入和全文（关键词）索引

#### 解析流程

1. 选择分块方法和嵌入模型
2. 上传文件
3. 点击播放按钮启动解析

#### 灵活配置

- RAGFlow 允许为特定文件选择不同的分块方法
- 可以启用或禁用单个文件，精细控制对话

---

### 5. 选择 PDF 解析器

从 v0.17.0 起，RAGFlow 将 PDF 的数据提取任务与分块方法解耦，允许自主选择视觉模型执行 OCR、TSR 和 DLR 任务。

#### PDF 解析器选项

| 解析器 | 描述 | 适用场景 |
|--------|------|----------|
| **DeepDoc** | 默认视觉模型，执行 OCR、TSR、DLR | 复杂 PDF、格式化文档（耗时较长） |
| **Naive** | 跳过 OCR、TSR、DLR 任务 | 纯文本 PDF（速度快） |
| **MinerU** | 开源 PDF 转 Markdown 工具 | 需要高质量格式转换 |
| **Docling** | 开源文档处理工具 | 生成式 AI 文档处理 |
| 第三方视觉模型 | 来自模型提供商的 VLM | 根据需求选择轻量或高性能模型 |

#### MinerU 配置（v0.22.0+）

RAGFlow 作为 MinerU 的远程客户端调用其 API。

**配置参数**：

| 参数 | 描述 | 示例 |
|------|------|------|
| `MINERU_APISERVER` | MinerU API 端点 | `http://mineru-host:8886` |
| `MINERU_BACKEND` | MinerU 后端类型 | `"pipeline"`（默认） |
| `MINERU_SERVER_URL` | 下游 vLLM HTTP 服务器 | `http://vllm-host:30000` |
| `MINERU_OUTPUT_DIR` | 输出文件存储目录 | `/path/to/output` |
| `MINERU_DELETE_OUTPUT` | 是否删除临时输出 | `1`（删除）/ `0`（保留） |

#### Docling 配置

使用外部 Docling Serve 实例时，设置：
```
DOCLING_SERVER_URL=http://docling-host:5001
```

> **警告**：第三方视觉模型标记为 **Experimental**，尚未完全测试。

---

### 6. 设置上下文窗口大小（v0.23.0+）

**Image & table context window** 功能可提高长上下文 RAG 性能。

#### 功能说明

- 图片和表格周围的文本通常描述它们
- 通过上下文窗口，可将相邻文本和视觉元素合并为一个分块
- 显著提高图表和表格的召回准确度

#### 配置步骤

1. 在知识库 **Configuration** 页面
2. 找到 **Image & table context window** 滑块
3. 调整上下文 token 数量

> **说明**：数值表示从图片/表格上下方各捕获约 N tokens 文本作为上下文信息。捕获过程在标点符号处智能优化边界，保持语义完整性。

---

### 7. 配置父子分块策略（v0.23.0+）

父子分块机制解决传统 "分块-嵌入-检索" 流程的结构矛盾。

#### 问题背景

传统流程中，单个文本分块需同时承担：
- **语义匹配（召回）**：需要精细、准确的分块
- **上下文理解（利用）**：需要连贯、信息完整的上下文

这两个目标存在内在冲突。

#### 解决方案

- 文档先分割为较大的**父分块**，保持相对完整的语义单元
- 每个父分块进一步细分为多个**子分块**，用于精确召回
- 检索时，系统根据子分块定位相关文本，同时自动关联并召回父分块
- 保证高召回相关性，同时为生成阶段提供充足语义背景

#### 实际案例

**合规手册场景**：
- 用户查询："违约责任"
- 子分块精确召回："违约罚金为合同总额的 20%"
- 但无法区分适用于"轻微违约"还是"重大违约"
- 通过父子分块，返回子分块及其父分块（完整条款章节）
- LLM 可基于更广上下文做出准确判断

#### 配置步骤

1. 在知识库 **Configuration** 页面
2. 找到 **Child chunk are used for retrieval** 开关
3. 设置子分块的分隔符

---

### 8. 干预解析结果

RAGFlow 具备可视性和可解释性，允许查看分块结果并人工干预。

#### 操作流程

1. 点击完成解析的文件查看分块结果
2. 进入 **Chunk** 页面
3. 悬停每个快照快速查看分块
4. 双击分块文本：
   - 添加关键词、问题、标签
   - 手动修改内容

> **提示**：添加关键词可提高该分块在包含这些关键词的查询中的排名。此操作增加其关键词权重，可改善搜索列表中的位置。

---

### 9. 运行检索测试

RAGFlow 在对话中使用全文搜索和向量搜索的多重召回。

#### 关键参数

| 参数 | 描述 | 默认值 | 调整建议 |
|------|------|--------|----------|
| **Similarity threshold** | 相似度阈值，低于此值的分块将被过滤 | 0.2 | 提高值可减少噪声，降低值可增加召回 |
| **Vector similarity weight** | 向量相似度对总分的贡献百分比 | 0.3 | 提高值重视语义匹配，降低值重视关键词匹配 |

#### 测试步骤

1. 在 **Retrieval testing** 区域
2. 在 **Test text** 中输入测试问题
3. 查看返回结果是否包含正确引用

---

## 搜索知识库

v0.24.0 版本的搜索功能仍处于初步阶段，仅支持按名称搜索知识库。

---

## 删除知识库

删除知识库会自动移除 `root/.knowledge` 目录下的关联文件夹。

#### 删除后果

| 上传方式 | 删除后状态 |
|----------|------------|
| 直接上传到知识库 | 文件永久删除 |
| 通过文件系统链接 | 文件引用删除，原文件仍存在 |

**删除操作**：悬停知识库卡片的三点图标，选择 **Delete**。

---

## 最佳实践总结

### 按场景的配置建议

#### 企业知识库（混合文档）

```
分块方法：General
嵌入模型：BGE-m3（多语言支持）
PDF 解析器：DeepDoc
上下文窗口：中等（约 500 tokens）
父子分块：启用（提高复杂查询准确度）
```

#### 法律合规系统

```
分块方法：Laws
嵌入模型：BGE-zh（中文优化）
PDF 解析器：DeepDoc
父子分块：启用（保持条款完整性）
相似度阈值：0.25（减少噪声）
```

#### FAQ 系统

```
分块方法：Q&A
嵌入模型：text2vec-chinese
PDF 解析器：Naive（纯文本）
上下文窗口：关闭
```

#### 科研论文库

```
分块方法：Paper
嵌入模型：multilingual-e5
PDF 解析器：MinerU 或 DeepDoc
上下文窗口：较高（图表说明重要）
父子分块：启用
```

#### 表格数据库

```
分块方法：Table
嵌入模型：BGE-base
PDF 解析器：DeepDoc（表格结构识别）
```

### 常见问题处理

| 问题 | 可能原因 | 解决方案 |
|------|----------|----------|
| 解析进度卡在 1% 以下 | 系统资源不足或配置问题 | 检查服务器状态，参考 FAQ |
| 解析接近完成时卡住 | 文件格式特殊 | 参考 FAQ 处理 |
| 答案匹配错误 | 分块方法不合适 | 尝试不同分块模板 |
| 图表无法召回 | 缺少上下文 | 启用上下文窗口功能 |
| 答案缺少完整上下文 | 分块过小 | 启用父子分块策略 |

---

---

## 实战案例

以下案例来自 RAGFlow 官方博客，展示了不同业务场景下的知识库配置和 Agent 构建。

---

### 案例一：公司研报深度分析 Agent

**场景**：金融机构投研部门分析师需要处理大量行业/公司研报、第三方数据、实时市场动态，目标是快速形成投资建议。

#### 技术流程

```
用户提问 → 提取股票代码 → 获取财务指标 → 整合研报信息 → 输出综合分析
```

#### 实现要点

| 步骤 | 组件 | 功能 |
|------|------|------|
| 股票代码提取 | Agent + TavilySearch | 从自然语言识别股票名称/代码 |
| 条件判断 | Conditional Node | 判断是否成功识别股票代码 |
| 财务数据获取 | Yahoo Finance Tools | 获取核心财务指标 |
| 数据格式化 | Code Node (Python) | 生成 Markdown 财务表格 |
| 研报检索 | Agent + Retrieval + MCP | 调用 AlphaVantage API + 内部知识库 |
| 研报生成 | Agent | 整合信息，生成专业投研报告 |

#### 分块策略

- **Paper** 分块方法：研报包含摘要、核心观点、专题分析、财务预测表、风险提示等模块
- 按章节或逻辑段落分块，保持结构完整性

#### 核心亮点

- **保留差异化观点**：当不同报告有分歧时，不合并为单一结论，而是对比分析差异
- 整个流程约 5 分钟完成

#### 数据来源

- Hugging Face Datasets: `InfiniFlow/company_financial_research_agent`

#### 参考链接

- [RAGFlow in Practice - Building an Agent for Deep-Dive Analysis of Company Research Reports](https://ragflow.io/blog/ragflow-in-practice-building-an-agent-for-deep-dive-analysis-of-company-research-reports)

---

### 案例二：电商客服 Agent

**场景**：电商零售平台的智能客服，处理用户复杂多样的需求。

#### 三大需求场景

| 场景 | 描述 | 处理方式 |
|------|------|----------|
| 产品功能对比 | 购买前对比不同型号功能差异 | Retrieval + Agent |
| 使用指导 | 丢失说明书，需要使用帮助 | Retrieval + Agent |
| 安装预约 | 预约上门安装服务 | 多轮对话 Agent |

#### 实现要点

| 组件 | 功能 |
|------|------|
| Categorize | 意图识别，路由到不同工作流 |
| Retrieval | 检索产品信息/用户指南知识库 |
| Agent | 生成回复/收集信息 |
| Message | 输出结果给用户 |

#### 分块策略

- **Manual** 分块方法：产品手册图文并茂，按最小标题分块
- 确保每个文本段落及其配图在同一分块内

#### 核心亮点

- 使用 **工作流编排** 而非纯 Agent，响应更快
- 适合电商售后客服场景（高响应速度 + 相对简单任务）
- 可扩展：用户评论分析、个性化邮件营销

#### 数据来源

- Hugging Face Datasets: `InfiniFlow/Ecommerce-Customer-Service-Workflow`

#### 参考链接

- [Tutorial - Build an E-Commerce Customer Support Agent Using RAGFlow](https://ragflow.io/blog/tutorial-build-an-e-commerce-customer-support-agent-using-ragflow)

---

### 案例三：SQL 助手工作流

**场景**：让非技术人员（市场、产品经理）用自然语言查询数据库，减少对数据分析师的依赖。

#### 核心流程

```
自然语言问题 → 检索相关信息 → Agent 生成 SQL → 执行 SQL → 返回结果
```

#### 三个知识库

| 知识库 | 内容 | 分块方法 | 分块大小 | 分隔符 |
|--------|------|----------|----------|--------|
| Schema | 数据库表结构定义 | General | 2 tokens | 分号 (;) |
| Question to SQL | 问题-SQL 示例对 | Q&A | - | - |
| Database Description | 字段含义说明 | General | 2 tokens | ### |

#### 工作流组件

1. **三个 Retrieval 组件** - 并行检索三个知识库
2. **Agent 组件 (SQL Generator)** - 整合信息生成 MySQL 查询
3. **ExeSQL 组件** - 执行 SQL 语句
4. **Message 组件** - 输出结果

#### 注意事项

- Schema 字段命名避免下划线等特殊字符
- NL2SQL 无法完全准确，建议将标准化操作封装为 API/MCP

#### 数据来源

- Hugging Face Datasets: `InfiniFlow/text2sql`

#### 参考链接

- [Tutorial - Building a SQL Assistant Workflow](https://ragflow.io/blog/tutorial-building-a-sql-assistant-workflow)

---

### 案例四：GraphRAG 知识图谱

**场景**：处理多跳查询、嵌套逻辑问题，传统 RAG 无法很好回答的场景。

#### 为什么需要知识图谱？

- 传统 RAG 只检索相似内容，可能答非所问
- 知识图谱能有效聚合相关内容，适合 Query-Focused Summarization (QFS)
- 提供更多上下文信息，让 LLM 生成更可解释的答案

#### RAGFlow 对 GraphRAG 的改进

| 改进点 | 原版 GraphRAG | RAGFlow 改进 |
|--------|---------------|--------------|
| 实体去重 | 无去重，同义词被视为不同实体 | 使用 LLM 进行实体去重 |
| Token 消耗 | 文档多次发送给 LLM，消耗大 | 文档只发送一次，最小化消耗 |

#### 使用方法

1. 解析文档时选择 **Knowledge Graph** 分块方法
2. 定义要提取的实体类型（如：组织、人物、地点）
3. 触发 LLM 提取实体并构建知识图谱
4. 可视化展示：节点名称、描述、社区、思维导图

#### 演示效果（《权力的游戏》）

- GraphRAG：对多跳嵌套逻辑问题给出深入、全面的答案
- General 方法：无结果

#### 当前限制

- 仅支持文档级别知识图谱
- 不支持跨文档知识图谱链接

#### 参考链接

- [How Our GraphRAG Reveals the Hidden Relationships of Jon Snow and the Mother of Dragons](https://ragflow.io/blog/ragflow-support-graphrag)

---

### 案例五：RAGFlow x OpenClaw 集成

**场景**：为 OpenClaw 提供企业私有数据访问能力，让 Agent 从通用助手升级为业务专家。

#### 背景

- OpenClaw 拥有强大的执行框架和多样化操作能力
- 但在企业应用中，**无法直接访问和处理内部私有数据**
- RAGFlow 专注于深度文档理解和精准信息检索，将分散的私有知识转化为 Agent 可用的数据集

#### 功能概览

| 功能类别 | 具体能力 |
|----------|----------|
| **数据集操作** | 创建、查看、更新、删除数据集；修改名称、解析方法、描述 |
| **文档处理** | 上传/解析 PDF、TXT、DOCX；启用/禁用文档；更改分块方法 |
| **语义搜索** | 跨数据集搜索、指定数据集搜索、指定文档内搜索 |

#### 快速接入指南

1. **从 ClawHub 下载 RAGFlow Skill**
   - https://clawhub.ai/yingfeng/ragflow-skill

2. **获取 RAGFlow API Key 和 URL**
   - RAGFlow 个人主页 → 点击 API

3. **配置 .env 文件**
   ```bash
   RAGFLOW_API_URL=http://your-ragflow-ip
   RAGFLOW_API_KEY=ragflow-your-api-key-here
   ```

4. **重启 OpenClaw Gateway 生效**

#### 未来规划

- 推出专用 ContextEngine
- 可作为 System Prompt 注入 OpenClaw
- 直接对接 OpenClaw 的 ContextEngine API

#### 参考链接

- [RAGFlow x OpenClaw - The Enterprise-aware Claw](https://ragflow.io/blog/ragflow-x-openclaw-the-enterprise-aware-claw)

---

### 案例总结对照表

| 案例 | 场景 | 核心技术 | 推荐分块方法 |
|------|------|----------|--------------|
| 金融研报分析 | 投研分析 | Agent + Retrieval + MCP + Code Node | Paper |
| 电商客服 | 售后服务 | Categorize + Retrieval + Agent | Manual |
| SQL 助手 | 数据查询 | 多知识库 + Text-to-SQL | General + Q&A |
| GraphRAG | 多跳查询 | 知识图谱构建与可视化 | Knowledge Graph |
| OpenClaw 集成 | 企业级 AI 助手 | Skill 集成 + 语义搜索 | 根据场景选择 |

---

## 参考资料

- [RAGFlow 官方文档](https://ragflow.io/docs/)
- [RAGFlow GitHub](https://github.com/infiniflow/ragflow)
- [MinerU GitHub](https://github.com/opendatalab/MinerU)
- [Docling GitHub](https://github.com/docling-project/docling)
- [RAGFlow Discord](https://discord.gg/NjYzJD3GM3)
- [RAGFlow 官方博客](https://ragflow.io/blog)

---

**文档版本**：基于 RAGFlow v0.24.0 文档整理  
**更新日期**：2026-03-28