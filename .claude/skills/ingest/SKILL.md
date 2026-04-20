---
name: ingest
description: 将 raw/ 目录下的原始资料编译到 wiki/ 中。处理完成后将源文件自动移动到 raw/09-archive/ 归档。支持 `/ingest` (扫描 raw/ 下所有未归档文件) 或 `/ingest <path>` (处理指定文件)。当用户提到"摄取"、"导入"、"收入"资料，或要求将文件加入知识库时也应触发。绝对忽略 raw/09-archive/ 目录。
user-invocable: true
---

# ingest 技能

## 核心职责

将 `raw/` 中的原始资料编译到 `wiki/` 知识层，形成结构化、可链接的知识网络。

## 触发方式

- 用户执行 `/ingest` → 扫描 raw/ 所有未归档文件
- 用户执行 `/ingest <path>` → 处理指定文件
- 用户说"把这个资料摄入知识库"等 → 隐式触发

## 目录结构

```
raw/
├── 01-articles/        ← .md 文章
├── 02-papers/          ← .pdf 论文
├── 03-transcripts/     ← 转录文本
├── 04-meeting_notes/   ← 会议纪要
└── 09-archive/         ← ⚠️ 已归档，绝对不处理
```

## 编译流水线

### 步骤 1：发现待处理文件

```bash
# 扫描 raw/ 下所有 .md 和 .pdf（排除 archive）
find raw/ -type f \( -name "*.md" -o -name "*.pdf" \) | grep -v "09-archive"
```

### 步骤 2：读取源文件

- `.md` → 读取完整内容
- `.pdf` → 用 pdf 工具提取文本，失败则记录元信息

### 步骤 3：提炼核心

提取：
- **核心主旨**：这段资料讲什么（1-2句话）
- **实体**：人物、公司、工具、产品（→ `wiki/entities/`）
- **概念**：框架、方法论、理论（→ `wiki/concepts/`）
- **标签**：3-5 个关键词

### 步骤 4：创建来源摘要

写入 `wiki/sources/摘要-{slug}.md`：

```markdown
---
title: "摘要-{文件slug}"
type: source
tags: [来源, 原始文件, 标签1, 标签2]
sources: [raw/01-articles/xxx.md]
last_updated: YYYY-MM-DD
---

## 核心摘要
[3-5句话的核心总结]

## 关联连接
- [[EntityName]]
- [[ConceptName]]
```

### 步骤 5：创建/更新实体或概念页面

- 实体 → `wiki/entities/{EntityName}.md`
- 概念 → `wiki/concepts/{ConceptName}.md`

**处理逻辑：**
- 页面不存在 → 按 Frontmatter 规范新建
- 页面已存在 → 读取现有内容，增量合并新信息
- **发现冲突 → 立即暂停，向用户报告，等确认后再继续**

### 步骤 6：更新 index 和 log

```bash
# 更新 wiki/index.md（追加新页面条目）
# 追加 wiki/log.md（Append-only）
```

### 步骤 7：归档源文件

```bash
mv "raw/01-articles/xxx.md" "raw/09-archive/xxx.md"
```

## Frontmatter 规范

```yaml
---
title: "{标题}"
type: source | entity | concept | synthesis
tags: [标签1, 标签2]
sources: [raw/01-articles/xxx.md]  # source/synthesis 必填
last_updated: YYYY-MM-DD
---
```

## 注意事项

- **绝不修改 raw/ 下 09-archive 以外的任何文件**
- ingest 完成后立即归档，避免重复处理
- 每次 ingest 必须更新 index 和 log
- 非中文内容翻译成中文
