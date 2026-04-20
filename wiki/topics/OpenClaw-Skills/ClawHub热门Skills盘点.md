# ClawHub 热门 Skills 盘点（2026）

> 基于 ClawHub 市场数据、社区反馈及 GitHub 下载量统计，整理目前最受欢迎、最实用的编程类 Skills。

---

## 📊 生态系统总览

| 平台 | Skill 总量 | 累计下载量 |
|------|-----------|-----------|
| ClawHub | 13,700+ | 150万+ |
| awesome-agent-skills (VoltAgent) | 100+ 精选 | 13.1K ⭐ |
| ComposioHQ/awesome-claude-skills | 49.7K ⭐ | — |
| alirezarezvani/claude-skills | **223 skills** | v2.2.0 (2026-03-31) |

> SKILL.md 格式**跨平台通用**：Claude Code / Codex / Cursor / Gemini CLI / OpenClaw 均支持

---

## 🏆 编程类 Skills 排行榜

### 按下载量排序

| 排名 | Skill | 下载量 | 类型 | 核心功能 |
|:---:|-------|--------|------|---------|
| 1 | **frontend-design**（Anthropic 官方）| 277K+ | 设计 | 强制设计方向规范，先设计再写代码 |
| 2 | **agent-browser** | 14K ⭐ | 浏览器自动化 | Web 自动化、抓取、UI 测试 |
| 3 | **Wacli** | 16K | CLI 工具 | 通用命令行封装 |
| 4 | **ByteRover** | 16K | 开发伴侣 | AI 代码开发和调试 |
| 5 | **Self-Improving Agent** | 15K | AI 进化 | 运行时自我优化 |
| 6 | **Gog**（Google Workspace）| 14K | 办公自动化 | Google Docs/Sheets/Drive |
| 7 | **ATXP** | 14K | 跨平台自动化 | 连接各种服务构建流水线 |
| 8 | **GitHub** | 10K | 开发工作流 | Issues/PR/仓库管理 |
| 9 | **Summarize** | 10K | 内容处理 | 长文摘要 |
| 10 | **agent-sandbox-skill** | — | 云沙箱 | E2B 隔离环境构建/测试全栈应用 |

---

## 🔥 深度推荐：真正值得关注的 Skills

### 1️⃣ Anthropic 官方 Skills（17个）

GitHub: [anthropics/skills](https://github.com/anthropics/skills)

Anthropic 在 GitHub 发布了 **17 个官方 Skill**，是质量最有保证的一批：

| Skill | 功能 |
|-------|------|
| **frontend-design**（⭐ 277K+）| 先定设计方向再写代码，强制设计规范 |
| **document-skills** | PDF/Word/PPT 处理 |
| **测试相关** | 单元测试、E2E 测试自动化 |
| **代码质量** | Lint、格式化、架构检查 |

**安装方式：**
```bash
# Claude Code 内
/plugin marketplace add anthropics/skills
# 然后启用 frontend-design

# 或手动
cp -r skills/skills/$s ~/.claude/skills/
```

---

### 2️⃣ agent-browser（14K ⭐）

> 兼容：Claude Code / Cursor / Codex / Gemini CLI / Copilot

**最成熟的 Web 自动化 Skill**，支持：
- 打开网页、截图、点击、填表
- 抓取 JS 渲染内容
- UI 测试、网页数据采集
- 与 Playwright 无缝集成

```bash
npx clawhub@latest install agent-browser
```

---

### 3️⃣ agent-sandbox-skill（E2B 云沙箱）

> 支持：Gemini CLI / Claude Code / Codex CLI

让 AI 在**完全隔离的云沙箱**中构建、测试、部署全栈应用，**永不触碰本地文件系统**。

| 能力 | 说明 |
|------|------|
| 隔离执行 | 每个 Agent 分支在独立 E2B 沙箱中运行 |
| 全栈支持 | 可创建 Next.js/Express/任何技术栈 |
| 规模化 | 可同时运行多个独立沙箱 |
| 免本地污染 | 对本地文件和生成环境完全安全 |

```bash
# Claude Code / Codex 中使用
\agent-sandboxes:plan-full-stack  # 生成详细实现计划
\agent-sandboxes:test             # 运行验证测试
```

---

### 4️⃣ alirezarezvani/claude-skills（223个 Skills）

> ⭐ 活跃更新：v2.2.0（2026-03-31），含安全套件

**最全面的多领域 Skill 集合**，持续更新：

**工程团队套件（engineering-team）：**
- 敏捷工作流（Scope → Epic → Story → Task → 执行 → 质量门禁）
- 安全套件（新增 v2.2.0）：adversarial-reviewer / ai-security / cloud-security / incident-response / red-team / threat-detection
- 自动化测试流水线
- 技术债清理

**覆盖领域（部分）：**
| 类别 | 数量 | 代表 Skill |
|------|------|-----------|
| 工程团队 | 完整敏捷流程 | ln-200~ln-1000 |
| 安全 | 8个 | adversarial-reviewer 等 |
| 数据库 | Snowflake 等 | snowflake-development |
| 自评系统 | self-eval | 运行时自我评估 |

---

### 5️⃣ levnikolaevich/claude-code-skills（完整流水线）

专注**软件工程全生命周期**的 Skill 集合：

```
Scope 分解 → Epic 创建 → Story 规划 → Task 执行 → 代码质量门禁 → 测试
     ↓           ↓           ↓           ↓           ↓            ↓
 ln-200     ln-210/220   ln-221/222   ln-301   ln-510~514    ln-523
```

| 层级 | Skills | 核心功能 |
|------|--------|---------|
| 规划层 | ln-200~230 | 需求分解、优先级排序 |
| 执行层 | ln-300~404 | 任务创建、执行、审查、重做 |
| 质量层 | ln-500~523 | 代码质量检查、技术债、回归测试 |
| 编排层 | ln-1000 | 元编排器，串联全流程 |

---

### 6️⃣ 安全类 Skills（重要！）

来自 alirezarezvani v2.2.0 安全套件：

| Skill | 功能 |
|-------|------|
| **adversarial-reviewer** | 对抗性代码审查 |
| **ai-security** | AI 安全专项审查 |
| **cloud-security** | 云安全配置检查 |
| **incident-response** | 安全事件响应流程 |
| **red-team** | 红队攻击模拟 |
| **threat-detection** | 威胁检测 |

> ⚠️ **建议**：安装任何外部 Skill 之前，先用 **Skill Vetter** 做安全审计

---

### 7️⃣ Playwright Skill（浏览器自动化）

专为 AI Agent 优化的 Playwright Skill：
- 结构化 SKILL.md
- 完整的测试自动化工作流
- MCP 兼容设置
- 支持真实场景的测试管道

---

### 8️⃣ ComposioHQ/awesome-claude-skills（49.7K ⭐）

按类别整理的最大 Skill 集合：

| 类别 | 代表 Skill |
|------|---------|
| 文档处理 | PDF/Word/Google Docs |
| 开发工具 | GitHub/Git/代码审查 |
| 数据分析 | 数据库/SQL |
| 自动化 | n8n/IFTTT/Zapier |
| 安全 | 渗透测试/威胁检测 |
| 媒体 | 图像/视频生成 |
| 企业 | Jira/Linear/Notion |

---

## 📂 技能库资源地图

| 资源 | 地址 | 说明 |
|------|------|------|
| **ClawHub** | clawhub.ai | 官方 Skill 市场（13,700+） |
| **VoltAgent/awesome-agent-skills** | GitHub 13.1K ⭐ | 按类别整理的精选列表 |
| **ComposioHQ/awesome-claude-skills** | GitHub 49.7K ⭐ | 最大 Claude Skills 集合 |
| **alirezarezvani/claude-skills** | GitHub | 223 Skills，完整工程团队套件 |
| **heilcheng/awesome-agent-skills** | GitHub 3.3K ⭐ | Agent Skills 精选列表 |
| **Antigravity Awesome Skills** | — | 1,234+ 兼容 Skills |

---

## 🛠 安装命令速查

```bash
# 通过 clawhub 安装
npx clawhub@latest install <skill-name>

# 示例
npx clawhub@latest install agent-browser
npx clawhub@latest install github
npx clawhub@latest install playwright-skill

# Claude Code 官方市场
/plugin marketplace add anthropics/skills

# 查看已安装
npx clawhub@latest list

# 安全审计
npx clawhub@latest audit --local
```

---

## 💡 实用建议

### 必装清单（编程场景）
| Skill | 理由 |
|-------|------|
| **agent-browser** | Web 自动化必备 |
| **GitHub** | 开发工作流闭环 |
| **playwright-skill** | 浏览器测试 |
| **frontend-design** | 保证前端设计质量 |
| **安全套件**（adversarial-reviewer 等）| 代码安全审查 |

### 重要提醒
> ⚠️ 一个工程师测试了 47 个 Skill，发现 **40 个反而让输出变差**——增加 Token 开销、延迟，并缩小 AI 发挥空间。
>
> **最好的 Skill 往往是自己写的**——当你在多个会话里反复解释同一套工作流时，就该把它写成 Skill 了。

### Skill 质量判断标准
1. **描述精准**：描述模糊的 Skill，AI 永远不会主动触发
2. **体积控制**：完整指令在 5k tokens 以内
3. **触发明确**：description 写清楚触发场景
4. **先测试再生产**：用 test prompts 验证触发率

---

## 数据来源

- ClawHub 官方注册表 (clawhub.ai)
- GitHub 各 Skill 仓库 Stars 和更新频率
- awesome-agent-skills (VoltAgent, 13.1K ⭐)
- ComposioHQ/awesome-claude-skills (49.7K ⭐)
- alirezarezvani/claude-skills (v2.2.0, 2026-03-31)
- Anthropic 官方 skills 仓库 (github.com/anthropics/skills)
- 各社区评测文章（Reddit、AI Tools Kit、Composio.dev 等）
