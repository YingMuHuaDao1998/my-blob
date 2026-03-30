# OnlyOffice Document Builder Python SDK 获取文档位置 ID

## 概述

ONLYOFFICE Document Builder 支持通过 **Python SDK**（`document-builder` 库）在服务端生成和操作 Office 文档。本文档总结在文档中获取**位置 ID** 的几种方案。

> **说明：** Document Builder 的 Python SDK 本质上是通过 `CDocBuilder` 类调用内部 JavaScript API 来操作文档对象模型（DOM），因此获取位置 ID 的能力等同于 Office JavaScript API。

---

## 环境准备

### 安装

```bash
pip3 install document-builder
```

### 基础初始化

```python
import os
import docbuilder

# 初始化
builder = docbuilder.CDocBuilder()
builder.Initialize()

# 创建/打开文档
builder.CreateFile("docx")  # 或 OpenFile("docx", "/path/to/file.docx")
```

---

## 方案一：通过元素索引获取位置

### 获取段落/元素总数和索引

```python
context = builder.GetContext()
globalObj = context.GetGlobal()
api = globalObj["Api"]

document = api.GetDocument()

# 获取文档中所有元素（段落、表格等）的数量
elementsCount = document.GetElementsCount()
print(f"文档元素总数: {elementsCount}")

# 按索引获取元素
for i in range(elementsCount):
    element = document.GetElement(i)
    elementType = element.GetClassType()
    print(f"索引 {i}: {elementType}")
```

### 获取当前活动元素（光标位置）

```python
# 获取选区或光标所在元素
selection = document.GetSelection()
# 配合 GetElementByRange 使用
```

> **适用场景：** 遍历文档结构、按索引定位元素。

---

## 方案二：通过书签（Bookmark）获取位置 ID

书签是最常见的"位置标记"方式，每个书签有唯一 ID。

### 创建书签

```python
document = api.GetDocument()
paragraph = document.GetElement(0)  # 假设在第一个段落

# 添加书签，AddBookmark 返回书签对象
bookmark = document.AddBookmark({
    "Text": "第一章",
    "Name": "chapter1"
})
bookmarkId = bookmark.GetId()
print(f"书签ID: {bookmarkId}")
```

### 读取所有书签

```python
bookmarks = document.GetBookmarks()
bookmarksCount = bookmarks.GetCount()

for i in range(bookmarksCount):
    bookmark = bookmarks.GetItem(i)
    bookmarkId = bookmark.GetId()
    bookmarkName = bookmark.GetName()
    print(f"书签ID: {bookmarkId}, 名称: {bookmarkName}")
```

> **适用场景：** 在文档中标记特定位置，通过 ID 快速跳转或替换内容。

---

## 方案三：通过内容控件（Content Control）获取位置 ID

内容控件是一种可嵌入文档的富文本容器，每个控件有唯一 ID。

### 创建内容控件

```python
document = api.GetDocument()
paragraph = document.GetElement(0)

# 创建文本内容控件
contentControl = document.AddContentControl({
    "Type": "text",  # text / checkbox / dropdown / picture / combobox
    "Tag": "myControl",
    "Lock": False
})
controlId = contentControl.GetId()
print(f"内容控件ID: {controlId}")
```

### 获取所有内容控件

```python
contentControls = document.GetContentControls()
count = contentControls.GetCount()

for i in range(count):
    cc = contentControls.GetItem(i)
    ccId = cc.GetId()
    ccTag = cc.GetTag()
    print(f"控件ID: {ccId}, Tag: {ccTag}")
```

### 按 Tag 查找

```python
# 通过 Tag 定位特定控件
targetControl = document.GetContentControlByTag("myControl")
if targetControl:
    print(targetControl.GetId())
```

> **适用场景：** 表单、模板填充、数据绑定等需要精确定位的内容区域。

---

## 方案四：通过批注（Comment）获取位置 ID

```python
comments = document.GetComments()
count = comments.GetCount()

for i in range(count):
    comment = comments.GetItem(i)
    commentId = comment.GetId()
    print(f"批注ID: {commentId}")
```

> **适用场景：** 提取文档中的所有批注及其位置信息。

---

## 方案五：通过目录（Table of Contents）获取位置

```python
# 获取文档中所有目录
toc = document.GetTablesOfContents()
count = toc.GetCount()

for i in range(count):
    tocItem = toc.GetItem(i)
    print(f"目录项: {tocItem.GetText()}")
```

---

## 综合示例：打开文档并汇总所有位置信息

```python
import os
import docbuilder

builder = docbuilder.CDocBuilder()
builder.Initialize()

# 打开已有文档
builder.OpenFile("docx", "/path/to/document.docx")

context = builder.GetContext()
globalObj = context.GetGlobal()
api = globalObj["Api"]
document = api.GetDocument()

print("=== 文档位置信息汇总 ===\n")

# 1. 元素总数
print(f"📄 元素总数: {document.GetElementsCount()}")

# 2. 书签
bookmarks = document.GetBookmarks()
print(f"🔖 书签数: {bookmarks.GetCount()}")
for i in range(bookmarks.GetCount()):
    b = bookmarks.GetItem(i)
    print(f"   - ID: {b.GetId()}, Name: {b.GetName()}")

# 3. 内容控件
contentControls = document.GetContentControls()
print(f"📦 内容控件数: {contentControls.GetCount()}")
for i in range(contentControls.GetCount()):
    cc = contentControls.GetItem(i)
    print(f"   - ID: {cc.GetId()}, Tag: {cc.GetTag()}")

# 4. 批注
comments = document.GetComments()
print(f"💬 批注数: {comments.GetCount()}")

builder.CloseFile()
```

---

## 方案对比

| 方案 | 获取方式 | 是否有持久 ID | 适用场景 |
|------|---------|-------------|---------|
| 元素索引 | `GetElement(i)` | ❌ 索引会随文档变化 | 遍历文档结构 |
| 书签 | `AddBookmark` / `GetBookmarks()` | ✅ ID 持久 | 位置标记、跳转 |
| 内容控件 | `AddContentControl` | ✅ ID 持久 | 表单、模板、数据绑定 |
| 批注 | `GetComments()` | ✅ ID 持久 | 提取批注信息 |

---

## 附录：两种调用方式的区别

OnlyOffice Document Builder 支持两种调用模式：

### 方式 A：Python SDK（Builder Framework）

通过 `docbuilder.CDocBuilder` Python 库直接调用，安装后即可使用。

### 方式 B：HTTP API（Document Builder API）

如果调用方希望不直接依赖 SDK，也可以通过 HTTP POST 请求调用远程 Document Server：

```bash
curl -X POST "https://documentserver/docbuilder" \
  -H "Content-Type: application/json" \
  -d '{
    "async": false,
    "url": "https://example.com/script.js"
  }'
```

> 此方式需要已部署 ONLYOFFICE Docs 服务器，`.js` 脚本中同样可以使用 `Api.GetDocument().GetBookmarks()` 等接口。

---

## 参考资料

- [ONLYOFFICE Document Builder 官方文档](https://api.onlyoffice.com/docs/document-builder/get-started/overview/)
- [Python Samples Guide](https://api.onlyoffice.com/docs/document-builder/samples/python-samples-guide/)
- [CDocBuilder 类参考](https://api.onlyoffice.com/docs/document-builder/builder-framework/CDocBuilder/)
- [GitHub: document-builder-samples](https://github.com/ONLYOFFICE/document-builder-samples)
