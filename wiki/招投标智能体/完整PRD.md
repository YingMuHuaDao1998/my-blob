# 招投标智能体（AixBidder）- 完整产品需求文档

> 基于源码分析 + 页面调研生成
> 更新时间：2026-04-05

---

## 1. 产品概述

**产品名称：** 招投标智能体（AixBidder）  
**产品定位：** 面向投标企业的智能化标书制作与管理平台，覆盖从"拿到招标文件"到"提交标书"的完整流程  
**目标用户：** 投标企业的项目经理、行政人员、市场人员

---

## 2. 系统功能架构

```
招投标智能体
├── 工作台（Dashboard）
├── 标书管理
│   ├── 列表页
│   ├── 创建标书
│   ├── 详情页
│   │   ├── 招标文件解析（步骤1）
│   │   ├── 自动生成投标文件（步骤2）
│   │   └── 标书合规校验（步骤3）
├── 标书检查
│   ├── 标书合规检查（上传招标文件+投标文件）
│   └── 标书比对（两份投标文件同屏比对）
└── 素材库管理
    ├── 基础信息（公司简介、营业执照、组织架构图）
    ├── 公司资质（ISO认证、CMMI等）
    ├── 专利软著（软件著作权、发明专利）
    ├── 合同业绩（政府项目、教育项目、医疗项目等）
    └── 人员信息（项目经理、技术人员等）
```

---

## 3. 业务流程

### 3.1 核心业务流程（制标流程）

```
步骤1：招标文件解析
    ↓ AI解析招标文件，提取关键信息
步骤2：自动生成投标文件
    ↓ 基于知识库自动填充投标文件内容
步骤3：标书合规校验
    ↓ 智能检测标书合规性，规避废标风险
```

### 3.2 状态派生规则

标书状态（status）和进度（progress）由三个阶段状态自动计算得出：

| 分析状态 | 生成状态 | 校验状态 | → 标书状态 | → 进度 |
|---------|---------|---------|-----------|-------|
| 0 | 0 | 0 | 待开始 | 0% |
| 1 | 0 | 0 | 进行中 | 30% |
| 1 | 1 | 0 | 进行中 | 80% |
| 1 | 1 | 1 | 已完成 | 100% |

---

## 4. 功能模块详述

### 4.1 工作台 `/dashboard`

**统计指标：**
- 总项目数（支持环比上月）
- 素材条目数（支持环比上月）
- 本月生成标书数（支持环比上月）
- 合规校验通过率（支持环比上月）

**快捷入口：**
- 标书管理
- 素材库管理
- 标书检查
- 标书比对

**近期项目列表：**
- 显示项目名称、状态、截止日期、进度条
- 最多显示 4 条

### 4.2 标书管理 `/bid-management`

#### 4.2.1 列表页

**篮选条件：**
- 关键字搜索（标书名称 / 甲方名称）
- 标书状态筛选（待开始 / 进行中 / 已完成）
- 日期范围筛选（开始日期 ~ 截止日期）

**视图切换：** 列表视图 / 卡片视图

**分页：** 默认 10 条/页，支持切换每页条数

**操作：** 创建标书 / 查看详情 / 编辑 / 删除（软删除）

#### 4.2.2 创建标书

**必填字段：**
- 标书名称（name, max 200）
- 甲方名称（party_a_name, max 200）
- 预算金额（budget_amount, Decimal(14,2)）
- 开始日期（start_date）
- 截止日期（end_date, >= start_date）

**选填字段：**
- 描述（description, max 2000）

**校验规则：**
- 截止日期不能早于开始日期
- 名称和甲方名称自动去除首尾空格

#### 4.2.3 详情页

**基本信息展示：**
- 标书名称
- 标的金额
- 截止日期
- 甲方名称
- 创建时间
- 描述

**制标流程（三个步骤卡片）：**

| 步骤 | 名称 | 功能 | 状态说明 |
|------|------|------|---------|
| 步骤1 | 招标文件解析 | 上传招标文件，AI智能解析关键信息 | 进行中 / 待开始 |
| 步骤2 | 自动生成投标文件 | 基于知识库自动填充投标文件 | 进行中 / 待开始 |
| 步骤3 | 标书合规校验 | 智能检测标书合规性，规避废标风险 | 进行中 / 待开始 |

**下一步操作引导：**
- 根据当前步骤状态提示用户下一步操作
- 提供"开始招标文件解析"按钮

#### 4.2.4 编辑标书

支持修改：
- 标书名称、甲方名称、预算金额
- 描述
- 开始日期、截止日期
- 三个阶段状态（analysis_status / generation_status / review_status）

### 4.3 标书检查 `/bid-inspection/bid-check`

**功能：** 上传招标文件和投标文件，AI 检测标书合规性

**流程：**
1. **上传阶段**：上传招标文件 + 投标文件（PDF/DOCX）
2. **检查阶段**（3步）：
   - 解析招标文件要求
   - 比对投标文件内容
   - 生成问题报告
3. **结果阶段**：展示错误项 / 警告项 / 提示项

**问题分类：**
- **错误项**（error）：必须修改，可能导致废标
- **警告项**（warning）：建议修改，可能影响评分
- **提示项**（info）：优化建议，提升中标概率

**每个合规问题的结构：**
- 类型、分类、标题、描述
- 位置（章节）
- 页码
- 整改建议
- 对应招标文件要求（原文 + 页码）

**结果操作：** 重新检查 / 导出报告

### 4.4 标书比对 `/bid-inspection/bid-comparison`

**功能：** 上传两份投标文件进行同屏比对，分析相似度

**相似度分类：**
- **完全相同**（identical）：内容完全一致
- **高度相似**（similar）：表述略有差异但含义相同
- **存在差异**（different）：内容差异较大需要关注

**结果展示：**
- 每条比对结果含：标题、相似度百分比、文档A页码、文档B页码、两侧原文内容
- 支持点击条目在两侧文档中快速定位

### 4.5 素材库管理 `/material/*`

**五类素材：**

| 类别 | 说明 | 示例 |
|------|------|------|
| 基础信息 | 企业基础资料 | 公司简介、营业执照、组织架构图 |
| 公司资质 | 资质认证证书 | ISO9001、ISO27001、CMMI5 |
| 专利软著 | 知识产权 | 软件著作权、发明专利 |
| 合同业绩 | 历史项目案例 | 智慧政务、教育信息化、医疗大数据 |
| 人员信息 | 团队成员 | 项目经理、技术负责人、高级工程师 |

**素材通用字段：**
- 标题（title）
- 类型（type）
- 分类（category）
- 状态（已审核 / 待审核）
- 更新日期
- 文件大小（可选）

**素材操作：** 新增 / 编辑 / 删除

---

## 5. 数据库设计

### 5.1 ER 关系概览

```
users (1) ──── (N) bids (1) ──── (N) bid_analysis_result
                              (1) ──── (N) bid_generation_result
                              (1) ──── (N) bid_review_result
                              (1) ──── (N) compliance_issues
```

### 5.2 表：users（用户表）

| 字段 | 类型 | 约束 | 说明 |
|------|------|------|------|
| id | BIGINT | PK, AUTO | 主键 |
| username | VARCHAR(50) | UNIQUE, NOT NULL | 用户名 |
| password_hash | VARCHAR(255) | NOT NULL | 密码（bcrypt） |
| email | VARCHAR(100) | NULLABLE | 邮箱 |
| is_active | BOOLEAN | DEFAULT TRUE | 是否激活 |
| last_login | DATETIME | NULLABLE | 最后登录时间 |
| login_failed_count | INT | DEFAULT 0 | 登录失败次数 |
| locked_until | DATETIME | NULLABLE | 锁定截止时间 |
| created_at | DATETIME | NOT NULL | 创建时间 |
| updated_at | DATETIME | NOT NULL | 更新时间 |

### 5.3 表：bids（标书主表）

| 字段 | 类型 | 约束 | 说明 |
|------|------|------|------|
| id | BIGINT | PK, AUTO | 主键 |
| owner_id | BIGINT | FK→users.id, INDEX | 标书所有者 |
| name | VARCHAR(200) | NOT NULL | 标书名称 |
| party_a_name | VARCHAR(200) | NOT NULL | 甲方名称 |
| budget_amount | DECIMAL(14,2) | NOT NULL | 预算金额 |
| description | TEXT | NULLABLE | 描述 |
| start_date | DATE | NOT NULL | 开始日期 |
| end_date | DATE | NOT NULL | 截止日期 |
| status | SMALLINT | DEFAULT 0 | 标书状态（0待开始/1进行中/2已完成） |
| progress | SMALLINT | DEFAULT 0 | 进度百分比（0-100） |
| analysis_status | SMALLINT | DEFAULT 0 | 招标文件解析状态（0未开始/1进行中/2完成） |
| generation_status | SMALLINT | DEFAULT 0 | 自动生成状态（0未开始/1进行中/2完成） |
| review_status | SMALLINT | DEFAULT 0 | 合规校验状态（0未开始/1进行中/2完成） |
| is_deleted | SMALLINT | DEFAULT 0 | 软删除标记 |
| deleted_at | DATETIME | NULLABLE | 删除时间 |
| created_at | DATETIME | NOT NULL | 创建时间 |
| updated_at | DATETIME | NOT NULL | 更新时间 |

### 5.4 表：bid_analysis_results（招标文件解析结果表）

| 字段 | 类型 | 约束 | 说明 |
|------|------|------|------|
| id | BIGINT | PK, AUTO | 主键 |
| bid_id | BIGINT | FK→bids.id | 关联标书 |
| parsed_data | JSON/TEXT | NULLABLE | AI解析出的结构化数据 |
| source_file | VARCHAR(500) | NULLABLE | 原始招标文件名 |
| page_count | INT | NULLABLE | 解析的页数 |
| created_at | DATETIME | NOT NULL | 解析时间 |

### 5.5 表：bid_generation_results（投标文件生成结果表）

| 字段 | 类型 | 约束 | 说明 |
|------|------|------|------|
| id | BIGINT | PK, AUTO | 主键 |
| bid_id | BIGINT | FK→bids.id | 关联标书 |
| generated_file | VARCHAR(500) | NULLABLE | 生成的投标文件路径 |
| sections | JSON | NULLABLE | 各章节内容摘要 |
| created_at | DATETIME | NOT NULL | 生成时间 |

### 5.6 表：bid_review_results（合规校验结果表）

| 字段 | 类型 | 约束 | 说明 |
|------|------|------|------|
| id | BIGINT | PK, AUTO | 主键 |
| bid_id | BIGINT | FK→bids.id | 关联标书 |
| score | DECIMAL(5,2) | NULLABLE | 合规评分 |
| error_count | INT | DEFAULT 0 | 错误项数量 |
| warning_count | INT | DEFAULT 0 | 警告项数量 |
| info_count | INT | DEFAULT 0 | 提示项数量 |
| report_file | VARCHAR(500) | NULLABLE | 生成的报告文件路径 |
| created_at | DATETIME | NOT NULL | 校验时间 |

### 5.7 表：compliance_issues（合规问题记录表）

| 字段 | 类型 | 约束 | 说明 |
|------|------|------|------|
| id | BIGINT | PK, AUTO | 主键 |
| bid_review_result_id | BIGINT | FK→bid_review_results.id | 关联校验结果 |
| issue_type | ENUM | NOT NULL | 问题类型（error/warning/info） |
| category | VARCHAR(100) | NOT NULL | 问题分类 |
| title | VARCHAR(200) | NOT NULL | 问题标题 |
| description | TEXT | NULLABLE | 问题描述 |
| location | VARCHAR(200) | NULLABLE | 问题位置 |
| page | INT | NULLABLE | 页码 |
| suggestion | TEXT | NULLABLE | 整改建议 |
| bid_requirement | TEXT | NULLABLE | 对应招标文件要求 |
| bid_requirement_page | INT | NULLABLE | 要求所在页码 |

### 5.8 表：materials（素材表）

| 字段 | 类型 | 约束 | 说明 |
|------|------|------|------|
| id | BIGINT | PK, AUTO | 主键 |
| owner_id | BIGINT | FK→users.id | 所属用户 |
| title | VARCHAR(200) | NOT NULL | 素材标题 |
| type | VARCHAR(100) | NOT NULL | 素材类型 |
| category | ENUM | NOT NULL | 分类（基础信息/公司资质/专利软著/合同业绩/人员信息） |
| status | ENUM | DEFAULT 待审核 | 审核状态（已审核/待审核） |
| content | TEXT | NULLABLE | 文本内容 |
| file_path | VARCHAR(500) | NULLABLE | 文件存储路径 |
| file_size | BIGINT | NULLABLE | 文件大小（字节） |
| metadata | JSON | NULLABLE | 扩展元数据 |
| created_at | DATETIME | NOT NULL | 创建时间 |
| updated_at | DATETIME | NOT NULL | 更新时间 |

---

## 6. API 设计

### 6.1 认证相关 `/api/auth`

| 方法 | 路径 | 说明 |
|------|------|------|
| POST | /api/auth/login | 登录 |
| POST | /api/auth/register | 注册 |
| POST | /api/auth/logout | 登出 |

### 6.2 标书相关 `/api/bids`

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | /api/bids | 获取标书列表（支持关键字/状态/日期筛选 + 分页） |
| POST | /api/bids | 创建标书 |
| GET | /api/bids/:bid_id | 获取标书详情 |
| PATCH | /api/bids/:bid_id | 更新标书（含阶段状态更新） |
| DELETE | /api/bids/:bid_id | 删除标书（软删除） |

**列表查询参数：**
```
?keyword=&status=&start_date=&end_date=&page=1&page_size=10
```

**状态枚举值：** 0=待开始, 1=进行中, 2=已完成

### 6.3 素材相关 `/api/materials`

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | /api/materials | 获取素材列表 |
| POST | /api/materials | 创建素材 |
| PUT | /api/materials/:id | 更新素材 |
| DELETE | /api/materials/:id | 删除素材 |

---

## 7. 安全设计

### 7.1 认证
- 用户密码 bcrypt 加密存储
- 登录失败锁定机制（login_failed_count + locked_until）
- Token 认证（JWT 或 Session）

### 7.2 授权
- 所有业务 API 需登录认证（get_current_user）
- 标书数据按 owner_id 隔离，用户只能操作自己的标书

### 7.3 数据保护
- 删除操作使用软删除（is_deleted + deleted_at），数据可恢复

---

## 8. 技术栈

| 层级 | 技术 |
|------|------|
| 前端 | React + TypeScript + Ant Design |
| 后端 | FastAPI (Python) + SQLAlchemy |
| 数据库 | SQLite（开发）|
| 认证 | JWT / bcrypt |
| 文件存储 | 本地文件系统（file_path） |

---

## 9. 页面路由

| 路径 | 页面 |
|------|------|
| /login | 登录页 |
| /register | 注册页 |
| /dashboard | 工作台 |
| /bid-management | 标书管理列表 |
| /bid-management/detail/:id | 标书详情 |
| /bid-management/detail/:id/bid-analysis | 招标文件解析 |
| /bid-management/detail/:id/auto-generate | 自动生成投标文件 |
| /bid-management/detail/:id/compliance-check | 标书合规校验 |
| /bid-inspection/bid-check | 标书合规检查 |
| /bid-inspection/bid-comparison | 标书比对 |
| /material/basic-info | 基础信息 |
| /material/company-cert | 公司资质 |
| /material/patent | 专利软著 |
| /material/contract | 合同业绩 |
| /material/staff | 人员信息 |

---

## 10. 后续可扩展功能

1. **PDF/DOCX 文件解析**：当前标书分析/生成/检查模块多为前端 mock 数据，可接入真实文件解析（Python docling、pdfplumber 等）
2. **AI 生成**：标书内容 AI 自动撰写、素材库智能匹配
3. **消息通知**：标书截止日期临近提醒、校验结果通知
4. **多用户协作**：团队成员管理、标书操作权限细分
5. **标书模板**：常用标书模板管理，一键生成
6. **中标记录**：历史中标/未中标记录分析
