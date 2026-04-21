# Python PDF 转图片方案对比

将 PDF 文档分割成多张图片的几种主流方案。

---

## 方案一览

| 方案        | 外部依赖    | 速度  | 易用性   | 推荐场景        |
| --------- | ------- | --- | ----- | ----------- |
| pdf2image | poppler | 中等  | ⭐⭐⭐⭐⭐ | 快速上手、简单需求   |
| PyMuPDF   | 无       | 快   | ⭐⭐⭐⭐  | 批量处理、追求性能   |
| pdf2pic   | poppler | 中等  | ⭐⭐⭐⭐  | 友好 API、批量处理 |
| pypdfium2 | 无       | 快   | ⭐⭐⭐   | 跨平台、无外部依赖   |

---

## 1. pdf2image（最常用）

最流行的 PDF 转图片库，API 简洁直观。

### 安装

```bash
# macOS
brew install poppler

# Ubuntu/Debian
sudo apt-get install poppler-utils

# pip
pip install pdf2image
```

### 基础用法

```python
from pdf2image import convert_from_path

# 转换所有页面
images = convert_from_path('input.pdf', dpi=150)

# 保存为图片
for i, img in enumerate(images, 1):
    img.save(f'page_{i}.png', 'PNG')
```

### 常用参数

```python
images = convert_from_path(
    'input.pdf',
    dpi=300,              # 分辨率，默认 200
    first_page=1,         # 起始页
    last_page=5,          # 结束页
    fmt='png',            # 输出格式
    thread_count=4,       # 多线程处理
    output_folder='./out' # 输出目录
)
```

### 优缺点

**优点：**
- 文档完善，社区活跃
- API 设计简洁直观
- 支持多线程处理

**缺点：**
- 依赖系统安装 poppler
- 大文件处理较慢

---

## 2. PyMuPDF / fitz（性能最优）

基于 MuPDF 的高性能 PDF 处理库，速度最快。

### 安装

```bash
pip install PyMuPDF
```

> 无外部依赖，pip 安装即可使用。

### 基础用法

```python
import fitz  # PyMuPDF

doc = fitz.open('input.pdf')

for i, page in enumerate(doc, 1):
    # 渲染为图片
    pix = page.get_pixmap(dpi=150)
    pix.save(f'page_{i}.png')

doc.close()
```

### 高级用法

```python
import fitz

doc = fitz.open('input.pdf')

for page_num in range(len(doc)):
    page = doc[page_num]
    
    # 设置渲染参数
    mat = fitz.Matrix(2, 2)  # 2x 缩放，相当于 144 DPI (72*2)
    pix = page.get_pixmap(matrix=mat)
    
    # 或直接指定 DPI
    pix = page.get_pixmap(dpi=300)
    
    # 裁剪区域
    rect = fitz.Rect(0, 0, 500, 800)
    pix = page.get_pixmap(clip=rect)
    
    # 保存
    pix.save(f'page_{page_num + 1}.png')

doc.close()
```

### 导出其他格式

```python
pix.save('page.jpg', 'JPEG', jpg_quality=95)
pix.save('page.webp', 'WEBP', webp_quality=90)
```

### 优缺点

**优点：**
- 速度极快
- 无外部依赖
- 功能丰富（支持裁剪、旋转、注释等）

**缺点：**
- API 相对复杂
- 包体积较大

---

## 3. pdf2pic

pdf2image 的封装，提供更友好的 API。

### 安装

```bash
brew install poppler  # macOS
pip install pdf2pic
```

### 基础用法

```python
from pdf2pic import Converter

converter = Converter('input.pdf')
images = converter.convert(
    dpi=150,
    output_folder='output',
    fmt='png'
)
```

### 批量处理

```python
from pdf2pic import Converter

converter = Converter('input.pdf')

# 转换指定页面
converter.convert(
    dpi=150,
    first_page=1,
    last_page=10,
    output_folder='./images',
    fmt='png'
)
```

### 优缺点

**优点：**
- API 友好，适合批量处理
- 自动处理文件命名

**缺点：**
- 同样依赖 poppler
- 灵活性不如 pdf2image

---

## 4. pypdfium2

基于 Chrome PDFium 引擎，跨平台稳定。

### 安装

```bash
pip install pypdfium2
```

> 无外部依赖。

### 基础用法

```python
import pypdfium2 as pdfium

pdf = pdfium.PdfDocument('input.pdf')

for i in range(len(pdf)):
    page = pdf[i]
    
    # 渲染页面（scale=2 约等于 144 DPI）
    bitmap = page.render(scale=2)
    
    # 保存为 PNG
    bitmap.save(f'page_{i + 1}.png')
```

### 高级用法

```python
import pypdfium2 as pdfium

pdf = pdfium.PdfDocument('input.pdf')

for i in range(len(pdf)):
    page = pdf[i]
    
    # 设置渲染参数
    bitmap = page.render(
        scale=4,              # 4x 缩放，高质量
        crop=(0, 0, 500, 800) # 裁剪区域
    )
    
    # 导出
    bitmap.save(f'page_{i + 1}.png')
    # 或获取 PIL Image
    pil_img = bitmap.to_pil()
```

### 优缺点

**优点：**
- 无外部依赖
- 基于 Chrome PDFium，稳定可靠
- 输出质量高

**缺点：**
- 相对小众，文档较少
- API 不够直观

---

## 推荐选择

| 场景              | 推荐方案                        |
| --------------- | --------------------------- |
| 快速原型、简单需求       | **pdf2image**               |
| 批量处理、追求性能       | **PyMuPDF**                 |
| 生产环境、无外部依赖      | **PyMuPDF** 或 **pypdfium2** |
| 跨平台部署、Docker 容器 | **PyMuPDF** 或 **pypdfium2** |

---

## 常见问题

### DPI 如何选择？

| DPI | 用途 |
|-----|------|
| 72 | 屏幕预览 |
| 150 | 一般阅读 |
| 300 | 打印输出 |
| 600 | 高清扫描件 |

### 如何处理大文件？

```python
# 方案 1: 逐页处理（pdf2image）
from pdf2image import convert_from_path

for i, img in enumerate(convert_from_path('large.pdf', dpi=150)):
    img.save(f'page_{i + 1}.png')
    # 及时释放内存
    del img

# 方案 2: 多线程（pdf2image）
images = convert_from_path('large.pdf', dpi=150, thread_count=4)

# 方案 3: PyMuPDF（内存效率最高）
import fitz

doc = fitz.open('large.pdf')
for i in range(len(doc)):
    pix = doc[i].get_pixmap(dpi=150)
    pix.save(f'page_{i + 1}.png')
    # 自动释放
doc.close()
```

### 如何处理加密 PDF？

```python
# PyMuPDF
doc = fitz.open('protected.pdf', password='your_password')

# pdf2image
from pdf2image import convert_from_path
images = convert_from_path('protected.pdf', password='your_password')
```

---

## 参考资料

- [pdf2image 文档](https://pdf2image.readthedocs.io/)
- [PyMuPDF 文档](https://pymupdf.readthedocs.io/)
- [pdf2pic GitHub](https://github.com/Belval/pdf2pic)
- [pypdfium2 文档](https://pypdfium2.readthedocs.io/)