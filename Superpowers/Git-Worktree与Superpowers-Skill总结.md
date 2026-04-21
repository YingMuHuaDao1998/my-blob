# Git Worktree 与 Superpowers Skill 总结

> 整理时间：2026-04-02

---

## 1. Git Worktree 是什么

Git Worktree 是 Git 2.5（2016年）引入的内置功能，允许**同一个仓库的多个分支同时存在在不同目录**，无需切换分支。

### 原生命令

```bash
# 创建 worktree（同时创建新分支）
git worktree add <path> -b <branch-name>

# 查看所有 worktree
git worktree list

# 清理失效的 worktree
git worktree prune
```

### 解决的问题

| 场景 | 没有 Worktree | 有 Worktree |
|------|-------------|-------------|
| 同时开发两个功能 | 反复切换分支，stash 暂存 | 两个目录同时并行 |
| Code Review 时需要临时修 bug | 暂存当前修改，切分支 | 开一个新 worktree，不打扰当前工作 |
| 编译/测试耗时很长 | 切分支后要重新编译 | 可以在主目录继续，worktree 做其他事 |

---

## 2. Superpowers 的 using-git-worktrees Skill

所在路径：`~/.codex/superpowers/skills/using-git-worktrees/SKILL.md`

Superpowers 将 Git Worktree 封装成了一个开箱即用的 Skill，降低使用门槛。

### 核心功能

| 功能 | 说明 |
|------|------|
| **智能目录选择** | 优先 `.worktrees/` > `worktrees/` > 用户偏好 > 询问用户 |
| **安全检查** | 自动验证目录是否被 .gitignore 忽略，未忽略则自动修复 |
| **自动初始化** | 根据项目类型（Node/Python/Rust/Go）自动安装依赖 |
| **基线验证** | 创建后自动跑测试，确保工作区干净 |
| **分支创建** | 一步完成 worktree 创建 + 新分支创建 |

### 工作流程

```
Agent 调用 using-git-worktrees Skill
  ↓
检查目录优先级：.worktrees/ > worktrees/ > 全局位置
  ↓
验证 .gitignore（项目内目录必须被忽略，防止误提交）
  ↓
执行 git worktree add <path> -b <branch-name>
  ↓
自动检测项目类型（package.json / requirements.txt / Cargo.toml 等）
  ↓
安装依赖（npm install / pip install / cargo build 等）
  ↓
运行测试验证基线（npm test / pytest / cargo test 等）
  ↓
报告：Worktree ready at <path>
```

### 典型输出示例

```
Worktree ready at /Users/xuchaoyue/project/.worktrees/feature-auth
Tests passing (47 tests, 0 failures)
Ready to implement auth feature
```

---

## 3. 两者的关系

| 层级 | 内容 |
|------|------|
| **底层能力** | Git Worktree（Git 内置，2016年已有） |
| **封装层** | Superpowers using-git-worktrees Skill |
| **用户交互** | Agent 调用 Skill，自动完成全流程 |

```
Git Worktree（系统底层）
    ↓ 封装
Superpowers Skill（自动化包装）
    ↓ 调用
Agent 执行开发任务
```

类比：Git 的 `git worktree` 是原始命令，Superpowers Skill 相当于给你做了一个**一键启动脚本**，省去手动敲命令和记细节的麻烦。

---

## 4. 为什么 Worktree 重要

### 多分支并行开发
不再需要 `git checkout` 来回切换，每个功能在独立目录。

### 隔离性
一个 worktree 里跑测试/编译，不会影响其他目录的状态。

### 不丢修改
如果主目录有未完成的修改，可以开一个 worktree 继续工作，不需要 stash。

---

## 5. 常见坑

| 坑 | 原因 | 解法 |
|------|------|------|
| worktree 目录被 git 追踪 | 项目内目录没加 .gitignore | Superpowers Skill 自动检查并修复 |
| 忘记删失效的 worktree | 分支被删但目录残留 | 定期 `git worktree prune` |
| 在主分支创建 worktree | 误解了 worktree 的用途 | worktree 应该在任意分支上都能建 |

---

## 6. 参考

- Git 官方文档：`git worktree` — `man git-worktree`
- Superpowers Skill：`~/.codex/superpowers/skills/using-git-worktrees/SKILL.md`
- 相关 Skill：**finishing-a-development-branch**（开发完成后清理 worktree）
