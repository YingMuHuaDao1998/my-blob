# ClawHub 热门 Skills 盘点（2026）

> 基于 ClawHub 市场数据、社区反馈及下载量统计，整理目前最受欢迎、最实用的 OpenClaw Skills。

---

## 📊 总览

| 数据来源 | Skill 总量 | 累计下载量 |
|---------|-----------|-----------|
| ClawHub 官方 | 3,286+ | 150万+ |
| 另一口径 | 13,700+ | — |

---

## 🏆 Top 10 安装量排行

| 排名 | Skill | 下载量 | 类型 | 核心功能 |
|:---:|-------|--------|------|---------|
| 1 | **Capability Evolver** | 35K | AI 能力进化 | 让 Agent 自我反思、自我改进、自我记忆 |
| 2 | **Wacli** | 16K | CLI 工具 | 通用命令行封装，扩展终端能力 |
| 3 | **ByteRover** | 16K | 开发工具 | AI 代码开发和调试辅助 |
| 4 | **Self-Improving Agent** | 15K | AI 能力进化 | 运行时自我优化，积累执行经验 |
| 5 | **ATXP** | 14K | 自动化 | 跨平台任务自动化工作流 |
| 6 | **Gog** | 14K | Google Workspace | 驱动 Google Docs/Sheets/Drive 等 |
| 7 | **Agent Browser** | 11K | 浏览器自动化 | 让 Agent 操控浏览器进行 Web 操作 |
| 8 | **Summarize** | 10K | 内容处理 | 长文摘要、要点提取 |
| 9 | **GitHub** | 10K | 开发工具 | 管理 Issues、PRs、仓库工作流 |
| 10 | **Sonoscli** | 10K | 媒体控制 | 控制 Sonos 音响系统 |

> 数据来源：ClawHub 下载排行 / ClawOneClick 统计

---

## 📂 按类别详细说明

### 1️⃣ AI 能力进化（最火方向）

#### Capability Evolver — 星球级别 🌟
- **安装量**：35K（排名第一）
- **定位**：让 AI Agent 拥有自我反思、自我批评、自我学习的能力
- **核心能力**：
  - 任务完成后自动评估效果
  - 记录失败教训到长期记忆
  - 随使用持续提升表现
- **为什么火**：这是唯一一个让 Agent 越用越聪明的 Skill，符合"通用 AI  Agent"的核心预期

#### Self-Improving Agent
- **安装量**：15K
- **定位**：运行时自我优化，积累偏好和工作流经验
- **与 Capability Evolver 的区别**：更偏重执行层面的经验积累，比如记住用户的偏好、常用命令、工作风格

---

### 2️⃣ 开发工作流

#### GitHub
- **安装量**：10K
- **定位**：用自然语言管理 GitHub 所有功能
- **核心能力**：
  - `创建 / 评论 / 关闭` Issues
  - `审查 PR` 代码变更
  - `管理分支`、`处理 CI/CD`
  - `搜索代码`、`查看提交历史`
- **安装**：`npx clawhub@latest install github`

#### Agent Browser
- **安装量**：11K（最热浏览器自动化 Skill）
- **定位**：让 Agent 操控真实浏览器
- **核心能力**：
  - 打开网页、截图、点击、填表
  - 抓取动态内容（JS 渲染页面）
  - 做 UI 测试、网页数据采集
- **为什么火**：解决了 AI 无法操作 Web 的最后一公里问题

#### ByteRover
- **安装量**：16K
- **定位**：AI 代码开发和调试伴侣
- **核心能力**：辅助写代码、Debug、重构建议

---

### 3️⃣ 办公与生产力

#### Summarize
- **安装量**：10K
- **定位**：一键摘要任何长内容
- **核心能力**：
  - 网页、文章、文档摘要
  - 会议记录提炼
  - 多文档对比摘要

#### Gog（Google Workspace）
- **安装量**：14K
- **定位**：驱动 Google 全家桶
- **支持**：Google Docs / Sheets / Drive / Calendar / Gmail
- **使用场景**：让 Agent 帮你读写 Google 文档、操作表格、管理文件

#### ATXP
- **安装量**：14K
- **定位**：跨平台自动化工作流
- **核心能力**：连接各种服务，构建自动化流水线

---

### 4️⃣ 通信与协作

| Skill | 下载量 | 支持平台 |
|-------|--------|---------|
| **Slack** | 高 | Slack 消息、频道管理 |
| **Discord** | 高 | Discord 服务器、消息 |
| **Telegram** | 高 | Telegram Bot |
| **Feishu（飞书）** | 中高 | 飞书消息、文档、云空间 |
| **Email / Gmail** | 中 | 邮件读取和发送 |

> OpenClaw 默认支持飞书/Telegram/Discord/Slack 等主流 IM，开箱即用

---

### 5️⃣ 云与服务集成

| Skill | 场景 |
|-------|------|
| **AWS / Aliyun** | 云服务器管理（我目前就在用 `aliyun-server` skill 管阿里云 ECS） |
| **Docker** | 容器管理、镜像操作 |
| **Monday** | 项目管理面板，任务创建和状态更新 |
| **Notion** | Notion 数据库和页面管理 |
| **Feishu Wiki** | 飞书知识库 |
| **RAGFlow** | 知识库 RAG（我之前帮你配置过） |
| **DeerFlow** | 多 Agent 工作流（本地部署在 2026-03-29） |

---

### 6️⃣ AI 媒体生成

| Skill | 能力 |
|-------|------|
| **PixVerse** | AI 图片和视频生成 |
| **ElevenLabs Agent** | 语音交互、TTS |
| **Suno** | AI 音乐生成 |

---

### 7️⃣ 信息获取与研究

| Skill | 能力 |
|-------|------|
| **Tavily Web Search** | ClawHub 最热搜索 Skill，AI 优化的搜索引擎 |
| **Browser Search** | 通用网页搜索 |
| **Weather** | 天气查询（OpenClaw 内置） |
| **Calendar** | 日历事件管理 |

---

### 8️⃣ 安全类（必装）

#### Skill Vetter
- **定位**：安装 Skill 前的安全审计
- **功能**：自动检查 Skill 是否存在恶意代码、过度权限、可疑模式
- **建议**：从 ClawHub 安装任何 Skill 之前，先用 Vetter 过一遍

#### Predicate Snapshot
- **效果**：节省 90% Token 成本
- **原理**：缓存常用 Skill 输出，避免重复调用

---

## 🛠 安装方式

```bash
# 通过 clawhub CLI 安装
npx clawhub@latest install <skill-name>

# 示例
npx clawhub@latest install github
npx clawhub@latest install agent-browser
npx clawhub@latest install capability-evolvers

# 查看已安装
npx clawhub@latest list

# 安全审计
npx clawhub@latest audit --local
```

---

## 💡 实用建议

### 必装清单（根据我的使用经验）
| Skill | 理由 |
|-------|------|
| **GitHub** | 开发工作流必备 |
| **Agent Browser** | Web 自动化抓取 |
| **Self-Improving Agent** | 让 Agent 记住我的偏好 |
| **Summarize** | 处理长文档必备 |
| **AliYun Server**（已有） | ECS 运维管理 |

### 我的当前 Skill 配置（供参考）
```
~/.openclaw/skills/
├── aliyun-server/        ← 阿里云 ECS 管理
├── excalidraw/           ← Excalidraw 图表
├── feishu-doc/           ← 飞书文档
├── feishu-drive/         ← 飞书云盘
├── feishu-wiki/          ← 飞书知识库
├── feishu-task/          ← 飞书任务
├── file-organizer-skill/ ← 文件整理
├── github/               ← GitHub 工作流
├── self-improving-1-2-16/← 自我改进
├── tavily-search/        ← Web 搜索
├── tech-doc-writer/      ← 技术文档生成
└── weather/              ← 天气查询
```

---

## 📌 安全提醒

> ⚠️ 安装任何 Skill 之前：
> 1. 用 **Skill Vetter** 过一遍
> 2. 检查 Skill 申请的权限范围
> 3. 不确定时不安装——先问我

---

## 数据来源

- ClawHub 官方注册表 (clawhub.ai)
- ClawOneClick 2026 下载量排行
- awesome-openclaw-skills (GitHub 42.8k ⭐)
- 各社区评测文章（Reddit、AI Tools Kit、Growexx 等）
