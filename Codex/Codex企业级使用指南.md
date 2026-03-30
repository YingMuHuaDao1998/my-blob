# Codex CLI 企业级使用指南

> 整理时间：2026-03-27
> 来源：GitHub 官方文档、OpenAI 帮助中心、GitHub Discussions

---

## 一、概述

**Codex** 是 OpenAI 推出的 AI 编码代理，可以帮助开发者更快地编写、审查和发布代码。

### 核心能力

- **本地配对编程**：终端、IDE、桌面应用
- **云端任务委派**：隔离沙箱环境
- **自动化代码审查**：GitHub 集成

### 客户端选择

| 客户端 | 适用场景 |
|--------|----------|
| Codex CLI | 终端本地使用 |
| IDE 扩展 | VS Code、Cursor、Windsurf |
| Codex Web | 云端代理，访问 chatgpt.com/codex |
| Codex App | 桌面应用，支持多代理并行、worktree |

---

## 二、安装和配置

### 2.1 安装

```bash
# npm 安装
npm install -g @openai/codex

# 或使用其他包管理器
yarn global add @openai/codex
pnpm add -g @openai/codex
bun install -g @openai/codex

# macOS Homebrew
brew install --cask codex
```

### 2.2 环境变量设置

```bash
# 设置 API Key（当前会话）
export OPENAI_API_KEY="your-api-key-here"

# 或在项目根目录创建 .env 文件
echo "OPENAI_API_KEY=your-api-key-here" > .env
```

### 2.3 配置文件

配置文件位置：`~/.codex/`

**YAML 格式 (`~/.codex/config.yaml`)**:

```yaml
model: gpt-5.1-codex
approvalMode: suggest
fullAutoErrorMode: ask-user
notify: true
```

**JSON 格式 (`~/.codex/config.json`)**:

```json
{
  "model": "gpt-5.1-codex",
  "approvalMode": "suggest",
  "fullAutoErrorMode": "ask-user",
  "notify": true
}
```

### 2.4 配置参数说明

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `model` | string | o4-mini | AI 模型 |
| `approvalMode` | string | suggest | 权限模式 |
| `fullAutoErrorMode` | string | ask-user | Full Auto 错误处理 |
| `notify` | boolean | true | 桌面通知 |

---

## 三、CLI 基础命令

### 3.1 启动方式

```bash
# 交互模式
codex

# 带初始提示启动
codex "解释这个代码库"

# 全自动模式
codex --approval-mode full-auto "创建最精致的待办事项应用"

# 非交互模式（CI/脚本使用）
codex -q "修复 lint 错误"
codex -q --json "解释 utils.ts"

# Shell 补全
codex completion bash > ~/.codex-completion.bash
codex completion zsh > ~/.codex-completion.zsh
codex completion fish > ~/.codex-completion.fish
```

### 3.2 关键参数

| 参数 | 说明 | 示例 |
|------|------|------|
| `-m, --model` | 指定模型 | `codex -m gpt-5.1-codex-mini` |
| `-a, --approval-mode` | 审批模式 | `codex -a full-auto` |
| `-q, --quiet` | 静默模式 | `codex -q "task"` |
| `--notify` | 桌面通知 | `codex --notify` |
| `--no-project-doc` | 禁用 AGENTS.md | `codex --no-project-doc` |
| `--provider` | 使用其他模型 | `codex --provider azure` |

### 3.3 CLI 命令速查

| 命令 | 用途 | 示例 |
|------|------|------|
| `codex` | 交互式对话 | `codex` |
| `codex "..."` | 带初始提示启动 | `codex "修复 lint 错误"` |
| `codex -q "..."` | 非交互静默模式 | `codex -q --json "解释 utils.ts"` |
| `codex completion` | Shell 补全脚本 | `codex completion bash` |

---

## 四、审批模式详解

### 4.1 模式对比

| 模式 | 自动权限 | 需要审批 |
|------|----------|----------|
| **suggest** (默认) | 读取仓库文件 | 写文件、执行命令 |
| **auto-edit** | 读文件 + 写文件 | 执行 shell 命令 |
| **full-auto** | 读/写文件 + 执行命令 | - |

### 4.2 Full Auto 安全机制

- 网络被禁用
- 写入限制在当前工作目录
- 建议在 Git 追踪目录中使用（有安全网）

### 4.3 使用示例

```bash
# 建议模式
codex -a suggest "分析这段代码"

# 自动编辑模式
codex -a auto-edit "重构 utils.ts"

# 全自动模式（CI/自动化场景）
codex -a full-auto "运行所有测试并修复失败项"
```

---

## 五、企业级配置

### 5.1 多 Provider 配置

```json
{
  "model": "gpt-5.1-codex",
  "provider": "openai",
  "providers": {
    "openai": {
      "name": "OpenAI",
      "baseURL": "https://api.openai.com/v1",
      "envKey": "OPENAI_API_KEY"
    },
    "azure": {
      "name": "AzureOpenAI",
      "baseURL": "https://YOUR_PROJECT.openai.azure.com/openai",
      "envKey": "AZURE_OPENAI_API_KEY"
    },
    "deepseek": {
      "name": "DeepSeek",
      "baseURL": "https://api.deepseek.com",
      "envKey": "DEEPSEEK_API_KEY"
    },
    "ollama": {
      "name": "Ollama",
      "baseURL": "http://localhost:11434/v1",
      "envKey": "OLLAMA_API_KEY"
    },
    "openrouter": {
      "name": "OpenRouter",
      "baseURL": "https://openrouter.ai/api/v1",
      "envKey": "OPENROUTER_API_KEY"
    },
    "gemini": {
      "name": "Gemini",
      "baseURL": "https://generativelanguage.googleapis.com/v1beta/openai",
      "envKey": "GEMINI_API_KEY"
    },
    "mistral": {
      "name": "Mistral",
      "baseURL": "https://api.mistral.ai/v1",
      "envKey": "MISTRAL_API_KEY"
    },
    "xai": {
      "name": "xAI",
      "baseURL": "https://api.x.ai/v1",
      "envKey": "XAI_API_KEY"
    },
    "groq": {
      "name": "Groq",
      "baseURL": "https://api.groq.com/openai/v1",
      "envKey": "GROQ_API_KEY"
    }
  }
}
```

### 5.2 Provider 配置参数

| 参数 | 类型 | 说明 | 示例 |
|------|------|------|------|
| `name` | string | Provider 显示名称 | "OpenAI" |
| `baseURL` | string | API 服务 URL | "https://api.openai.com/v1" |
| `envKey` | string | API Key 环境变量名 | "OPENAI_API_KEY" |

### 5.3 AGENTS.md 自定义指令

**文件位置**：`~/.codex/AGENTS.md`

```markdown
# ~/.codex/AGENTS.md

- 总是使用表情符号回复
- 使用 TypeScript 严格模式
- 遵循项目的 ESLint 规则
- 仅在明确要求时使用 git 命令
- 为新函数编写测试
- 优先使用函数式编程模式
```

### 5.4 AGENTS.md 加载优先级

1. `~/.codex/AGENTS.md` - 全局个人指令
2. 项目根目录 `AGENTS.md` - 项目共享指令
3. 当前目录 `AGENTS.md` - 子目录/功能特定指令

**禁用加载**：

```bash
codex --no-project-doc
# 或
CODEX_DISABLE_PROJECT_DOC=1 codex
```

---

## 六、CI/CD 集成

### 6.1 GitHub Actions 示例

```yaml
name: Codex Auto-update

on:
  schedule:
    - cron: '0 9 * * 1'  # 每周一 9:00

jobs:
  update-changelog:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
      
      - name: Install Codex
        run: npm install -g @openai/codex
      
      - name: Update changelog
        env:
          OPENAI_API_KEY: ${{ secrets.OPENAI_KEY }}
        run: |
          codex -a auto-edit --quiet "更新 CHANGELOG 为下一版本准备"
      
      - name: Commit changes
        run: |
          git config --local user.email "action@github.com"
          git config --local user.name "GitHub Action"
          git add CHANGELOG.md
          git diff --quiet && git diff --staged --quiet || git commit -m "docs: update CHANGELOG"
          git push
```

### 6.2 环境变量

```bash
# 静默模式
CODEX_QUIET_MODE=1 codex

# 调试模式
DEBUG=true codex

# 禁用项目文档
CODEX_DISABLE_PROJECT_DOC=1 codex
```

---

## 七、企业级开发场景

### 7.1 代码审查

```bash
# 自动审查代码
codex -a suggest "审查 src/api/ 的变更并建议改进"

# 安全审计
codex "查找漏洞并创建安全审查报告"
```

### 7.2 重构

```bash
# React Hooks 重构
codex "将 Dashboard 组件重构为 React Hooks"

# 批量重命名
codex "使用 git mv 批量重命名 *.jpeg -> *.jpg"

# 代码清理
codex "移除 src/ 中未使用的导入和死代码"
```

### 7.3 测试生成

```bash
# 生成单元测试
codex "为 utils/date.ts 编写单元测试"

# 生成集成测试
codex "为 API 端点创建集成测试"

# 测试覆盖率
codex "提高 auth 模块的测试覆盖率"
```

### 7.4 数据库迁移

```bash
# 生成 SQL 迁移
codex "生成添加 users 表的 SQL 迁移脚本"
```

### 7.5 PR 提案

```bash
# 分析代码库并建议 PR
codex "仔细审查这个仓库，提出 3 个高影响力且范围明确的 PR 建议"
```

### 7.6 文档生成

```bash
# 生成 API 文档
codex "为 src/routes/ 中的所有端点生成 API 文档"

# 更新 README
codex "用最新功能更新 README.md"
```

---

## 八、Worktree（Codex App）

> **注意**：Worktree 是 Codex App（桌面应用）的功能，不是 CLI 功能。

### 8.1 什么是 Worktree

Codex App 内置了 **git worktree** 支持，允许：
- 同时运行多个 Codex 代理
- 在不同分支上并行工作
- 无需切换分支或创建多个仓库克隆

### 8.2 使用方式

1. 打开 Codex App
2. 在应用内创建新的 worktree
3. 每个 worktree 是独立的开发环境

### 8.3 适用场景

- 同时开发多个功能
- 在不同分支上进行代码审查
- 并行测试多个方案

---

## 九、Subagent

> **当前状态**：正在开发中，目前是内部机制，还未作为用户可见功能暴露。

根据 GitHub Discussions 的官方回复：

> "这是正在进行的工作——目前只是一个我们可以在此基础上构建的内部机制。它尚未作为用户可见的功能暴露出来。一旦它达到可以进行实验的状态，我们将添加文档。"

---

## 十、企业级功能

### 10.1 企业管理功能

| 功能 | 说明 |
|------|------|
| 企业管理指南 | 完整的企业部署文档 |
| RBAC | 基于角色的访问控制 |
| 数据驻留 | 符合数据保留和驻留政策 |
| 合规 API | 云端使用数据可通过 Compliance API 获取 |
| 企业分析 | Enterprise 工作区可访问 Analytics |

### 10.2 数据隐私

| 计划类型 | 数据使用政策 |
|----------|--------------|
| Business/Enterprise/Edu | 默认不使用输入/输出来训练模型 |
| Pro/Plus | 对话可能用于改进模型（可在设置中关闭） |

### 10.3 插件系统

- 封装可复用工作流
- 可组合多个 Skills、应用集成或 MCP 服务器配置
- 适合团队间共享稳定工作流
- Business/Enterprise 工作区可通过工作区应用控制管理

---

## 十一、平台沙箱机制

### 11.1 平台支持

| 平台 | 沙箱机制 |
|------|----------|
| macOS 12+ | Apple Seatbelt (`sandbox-exec`) |
| Linux | Docker 容器（推荐） |
| Windows | WSL2 |

### 11.2 macOS 沙箱机制

- 使用 Apple Seatbelt (`sandbox-exec`)
- 所有内容放在只读沙箱中
- 可写目录：`$PWD`、`$TMPDIR`、`~/.codex` 等
- 出站网络完全被阻止

### 11.3 Linux Docker 沙箱

```bash
# 使用脚本启动
./codex-cli/scripts/run_in_container.sh
```

---

## 十二、故障排查

### 12.1 调试模式

```bash
# 调试模式
DEBUG=true codex

# 查看完整请求/响应
DEBUG=true codex -q "测试"
```

### 12.2 常见问题

**问题：API Key 无效**

```bash
# 检查 API Key 是否设置
echo $OPENAI_API_KEY

# 重新设置
export OPENAI_API_KEY="your-api-key-here"
```

**问题：权限被拒绝**

```bash
# 检查审批模式
codex -a suggest  # 默认模式，需要审批

# 使用更高的权限模式
codex -a auto-edit  # 自动编辑
codex -a full-auto  # 全自动
```

**问题：网络请求失败**

- Full Auto 模式下网络被禁用
- 使用 suggest 或 auto-edit 模式

---

## 十三、参考资源

- **GitHub 仓库**: https://github.com/openai/codex
- **帮助中心**: https://help.openai.com/en/articles/11369540-codex-in-chatgpt
- **讨论区**: https://github.com/openai/codex/discussions
- **企业管理指南**: https://developers.openai.com/codex/enterprise

---

## 十四、更新日志

| 日期 | 更新内容 |
|------|----------|
| 2026-03-27 | 初始版本，整理 CLI 命令、企业配置、使用场景 |