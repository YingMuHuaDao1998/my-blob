# 标书管理模块 PRD

## 1. 产品概述

**产品名称：** 招投标智能体（AixBidder）  
**模块名称：** 标书管理  
**核心功能：** 管理所有标书项目，支持创建、编辑、删除、搜索、筛选、查看详情

---

## 2. 功能清单

### 2.1 列表管理
- 分页展示（默认10条/页）
- 支持上一页/下一页翻页
- 切换每页条数（通过下拉选择）

### 2.2 搜索过滤
- 按**标书名称**或**甲方**关键字搜索
- 按**标书状态**筛选（下拉）
- 按**日期范围**筛选（开始日期 ~ 截止日期）

### 2.3 视图切换
- **列表视图**：表格形式展示
- **卡片视图**：卡片形式展示

### 2.4 标书 CRUD
- **创建标书**：点击「创建标书」按钮
- **查看详情**：跳转详情页
- **编辑标书**：跳转编辑页
- **删除标书**：确认后删除

---

## 3. 标书状态流转

```
待开始 → 编写中 → 待提交 → 已提交 → 已开标 → 已中标/未中标
```

| 状态 | 说明 |
|------|------|
| 待开始 | 刚创建，尚未启动 |
| 编写中 | 正在编写标书 |
| 待提交 | 等待提交 |
| 已提交 | 已投出 |
| 已开标 | 已开标，等待结果 |
| 已中标 | 中标 |
| 未中标 | 未中标 |

---

## 4. 页面结构

### 4.1 列表页 /bid-management
- 顶部：搜索框 + 状态筛选 + 日期筛选
- 视图切换：列表 / 卡片
- 操作按钮：创建标书
- 表格/卡片列表
- 底部分页

### 4.2 详情页 /bid-management/:id
- 标书基本信息
- 进度信息
- 相关操作

### 4.3 创建/编辑页 /bid-management/edit 或 /bid-management/create
- 表单填写

---

## 5. 数据模型

### 5.1 表：bid_projects（标书项目表）

| 字段 | 类型 | 说明 |
|------|------|------|
| id | BIGINT PK | 主键 |
| name | VARCHAR(255) | 标书名称 |
| party_a | VARCHAR(255) | 甲方（招标方） |
| bid_amount | DECIMAL(15,2) | 标的金额（元） |
| status | ENUM | 状态 |
| progress | INT | 进度百分比（0-100） |
| start_date | DATE | 开始日期 |
| end_date | DATE | 截止日期 |
| created_by | BIGINT FK | 创建人 |
| created_at | DATETIME | 创建时间 |
| updated_at | DATETIME | 更新时间 |

### 5.2 表：bid_status_history（状态变更记录表）

| 字段 | 类型 | 说明 |
|------|------|------|
| id | BIGINT PK | 主键 |
| bid_id | BIGINT FK | 关联标书 |
| from_status | ENUM | 原状态 |
| to_status | ENUM | 新状态 |
| changed_by | BIGINT FK | 变更人 |
| changed_at | DATETIME | 变更时间 |

### 5.3 表：bid_attachments（标书附件表）

| 字段 | 类型 | 说明 |
|------|------|------|
| id | BIGINT PK | 主键 |
| bid_id | BIGINT FK | 关联标书 |
| file_name | VARCHAR(255) | 文件名 |
| file_path | VARCHAR(500) | 存储路径 |
| file_size | BIGINT | 文件大小（字节） |
| uploaded_by | BIGINT FK | 上传人 |
| uploaded_at | DATETIME | 上传时间 |

### 5.4 表：users（用户表）

| 字段 | 类型 | 说明 |
|------|------|------|
| id | BIGINT PK | 主键 |
| username | VARCHAR(100) | 用户名 |
| password_hash | VARCHAR(255) | 密码（加密存储） |
| display_name | VARCHAR(100) | 展示名称 |
| role | ENUM | 角色：admin/member |
| created_at | DATETIME | 创建时间 |

---

## 6. 核心 API 概要

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | /api/bids | 获取标书列表（支持分页/搜索/筛选） |
| GET | /api/bids/:id | 获取标书详情 |
| POST | /api/bids | 创建标书 |
| PUT | /api/bids/:id | 编辑标书 |
| DELETE | /api/bids/:id | 删除标书 |
| GET | /api/bids/:id/history | 获取状态变更记录 |
| POST | /api/bids/:id/attachments | 上传附件 |

---

## 7. 当前页面数据（测试数据）

- **名称：** 测试
- **甲方：** 测试
- **标的：** 12万元
- **状态：** 待开始
- **进度：** 0%
- **开始日期：** 2026-04-03
- **截止日期：** 2026-04-04
