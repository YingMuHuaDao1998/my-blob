# FastAPI 入门指南：从小白到跑通第一个真实项目

> 假设你：会用 Python 写函数、懂一点 HTTP 是什么、有过 GET/POST 的概念，其他不懂没关系

---

## 一、FastAPI 是什么

FastAPI 是一个** Python 的 Web 框架**，用来快速构建 API（接口服务）。

**它解决什么问题？**

传统 Python 写 API：Flask 轻量但 Validation 全靠自己，Django 全面但太重。FastAPI 在两者之间——既简单，又能自动校验数据、自动生成文档、性能还高。

**为什么用它？**

| 特点 | 说明 |
|------|------|
| 🚀 性能高 | 接近 Node.js 和 Go（感谢 Starlette） |
| 📄 自动文档 | 写好接口，浏览器直接打开交互式文档 |
| ✅ 数据校验 | 用 Pydantic，类型不对直接报错，不用自己写 if |
| ⚡ 异步支持 | 原生 async/await，高并发场景天然支持 |
| 🔧 依赖注入 | 写数据库连接、认证等公共逻辑极其方便 |

Netflix、Microsoft、Uber 都在用。

---

## 二、安装与环境

```bash
pip install fastapi uvicorn
# uvicorn 是运行 FastAPI 的服务器（类似 node_modules）

# 也可以一起装
pip install "fastapi[all]"
```

**运行项目**：

```bash
uvicorn main:app --reload
# main 是文件名，app 是 FastAPI 实例名
# --reload 表示改代码后自动重启（开发模式）
```

运行后访问：
- `http://127.0.0.1:8000/docs` — **Swagger 交互式文档**（最重要的！）
- `http://127.0.0.1:8000/redoc` — ReDoc 文档
- `http://127.0.0.1:8000/openapi.json` — OpenAPI 规范（给程序用）

---

## 三、第一个接口：Hello World

```python
# main.py
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"message": "你好，世界！"}
```

访问 `http://127.0.0.1:8000/` 就能看到 JSON 返回。访问 `/docs` 可以用网页直接测试这个接口。

---

## 四、快速上手 5 分钟：核心概念

### 4.1 路由参数（Path Parameters）—— URL 里拿数据

```python
@app.get("/items/{item_id}")
def get_item(item_id: int):
    return {"item_id": item_id, "name": "这个东西"}
```

- 访问 `/items/42` → `item_id = 42`
- 访问 `/items/abc` → **自动报错**（因为声明了 `int` 类型）

### 4.2 查询参数（Query Parameters）—— URL ? 后面拿数据

```python
@app.get("/search")
def search_items(q: str = "", limit: int = 10):
    return {"query": q, "returned": limit}
```

- 访问 `/search?q=苹果&limit=5` → 返回 `{"query": "苹果", "returned": 5}`
- `q: str = ""` 表示可选，默认空字符串
- `limit: int = 10` 表示可选，默认 10

### 4.3 请求体（Request Body）—— POST 数据

用 **Pydantic** 定义数据结构：

```python
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    price: float
    is_offer: bool | None = None  # 可选字段

@app.post("/items/")
def create_item(item: Item):
    return {"item": item, "status": "创建成功"}
```

测试：`/docs` 页面可以直接填表单测试，FastAPI 会自动校验数据类型。

### 4.4 响应模型（Response Model）—— 控制输出格式

```python
class UserOut(BaseModel):
    id: int
    name: str
    email: str

@app.get("/users/{user_id}", response_model=UserOut)
def get_user(user_id: int):
    # 从数据库取数据
    return {"id": user_id, "name": "张三", "email": "zhangsan@example.com"}
```

好处：即使数据库返回了密码字段，`response_model` 会**只输出声明的字段**，防止泄露。

---

## 五、Pydantic 核心用法

Pydantic 是 FastAPI 数据校验的基础。

### 5.1 必填 vs 可选

```python
class Product(BaseModel):
    name: str              # 必填
    price: float           # 必填
    description: str | None = None  # 可选
    tags: list[str] = []  # 可选，默认空列表
```

### 5.2 嵌套模型

```python
class Address(BaseModel):
    city: str
    street: str

class User(BaseModel):
    name: str
    address: Address  # 嵌套

# 对应 JSON：
# {"name": "李四", "address": {"city": "北京", "street": "中关村"}}
```

### 5.3 列表返回

```python
@app.get("/items/", response_model=list[Item])
def get_items():
    return [
        Item(name="苹果", price=5.0),
        Item(name="香蕉", price=3.0)
    ]
```

---

## 六、依赖注入：FastAPI 最强大的功能

### 6.1 为什么需要依赖注入

很多接口都需要相同的逻辑：数据库连接、用户认证、获取公共参数。如果每个接口都写一遍，代码又臭又长。依赖注入让你**把公共逻辑抽出来**，需要的时候声明一下就行。

### 6.2 第一个依赖

```python
# dependencies.py
def get_query_param(x: str = "默认值"):
    return x

# main.py
from fastapi import Depends

@app.get("/items/")
def read_items(shared_param: str = Depends(get_query_param)):
    return {"got": shared_param}
```

### 6.3 数据库连接依赖

```python
from database import SessionLocal

def get_db():
    db = SessionLocal()
    try:
        yield db  # 实际使用时才建立连接
    finally:
        db.close()  # 用完自动关闭

@app.get("/users/{user_id}")
def get_user(user_id: int, db=Depends(get_db)):
    user = db.query(User).get(user_id)
    return user
```

### 6.4 认证依赖

```python
def verify_token(token: str = Depends(get_header_token)):
    if token != "secret-token":
        raise HTTPException(status_code=401, detail="未授权")
    return token

@app.get("/admin/dashboard")
def admin_panel(token: str = Depends(verify_token)):
    return {"data": "管理员专属内容"}
```

**依赖注入 vs 中间件**：中间件处理**所有请求**，而 `Depends` 只处理**声明了它的接口**。一句话：认证用 Depends 比中间件更灵活。

---

## 七、错误处理

### 7.1 HTTPException

```python
from fastapi import HTTPException

@app.get("/items/{item_id}")
def get_item(item_id: int):
    if item_id not in items:
        raise HTTPException(status_code=404, detail="找不到这个东西")
    return items[item_id]
```

### 7.2 自定义全局异常处理器

```python
from fastapi import Request
from fastapi.responses import JSONResponse

@app.exception_handler(ItemNotFoundError)
async def item_not_found_handler(request: Request, exc: ItemNotFoundError):
    return JSONResponse(
        status_code=404,
        content={"message": "物品不存在"}
    )
```

---

## 八、中间件

中间件是**每个请求都会经过**的拦截器。

### 8.1 自定义中间件

```python
import time
from fastapi import FastAPI, Request

app = FastAPI()

@app.middleware("http")
async def add_process_time(request: Request, call_next):
    start = time.perf_counter()
    response = await call_next(request)  # 等待下游处理完
    process_time = time.perf_counter() - start
    response.headers["X-Process-Time"] = str(process_time)
    return response
```

### 8.2 常用内置中间件

```python
from fastapi.middleware.cors import CORSMiddleware
from fastapi.middleware.gzip import GZipMiddleware

app = FastAPI()

# 允许跨域（前端调 API 必须）
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Gzip 压缩，减少传输量
app.add_middleware(GZipMiddleware, minimum_size=1000)
```

---

## 九、项目结构：一个真实可用的架子

```
my_project/
├── main.py              # 应用入口
├── database.py          # 数据库连接
├── models.py            # SQLAlchemy 模型（表结构）
├── schemas.py           # Pydantic 模型（数据校验）
├── dependencies.py      # 依赖注入（认证、数据库连接）
├── routers/
│   ├── __init__.py
│   ├── items.py         # items 相关接口
│   └── users.py         # users 相关接口
└── requirements.txt
```

### main.py

```python
from fastapi import FastAPI
from routers import items, users

app = FastAPI(title="我的 API", version="1.0.0")

app.include_router(items.router, prefix="/items", tags=["物品"])
app.include_router(users.router, prefix="/users", tags=["用户"])

@app.get("/")
def read_root():
    return {"message": "API 正在运行"}
```

### database.py

```python
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

SQLALCHEMY_DATABASE_URL = "sqlite:///./myapp.db"
# PostgreSQL: "postgresql://user:password@localhost/dbname"

engine = create_engine(SQLALCHEMY_DATABASE_URL, connect_args={"check_same_thread": False})
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()
```

### models.py（数据库表结构）

```python
from sqlalchemy import Column, Integer, String, Boolean
from database import Base

class Item(Base):
    __tablename__ = "items"

    id = Column(Integer, primary_key=True, index=True)
    name = Column(String, index=True)
    description = Column(String, nullable=True)
    price = Column(Integer)  # 存整数（分）避免浮点精度问题
    in_stock = Column(Boolean, default=True)
```

### schemas.py（Pydantic 模型）

```python
from pydantic import BaseModel

class ItemBase(BaseModel):
    name: str
    description: str | None = None
    price: int  # 前端传"分"，避免精度问题

class ItemCreate(ItemBase):
    pass

class ItemOut(ItemBase):
    id: int
    in_stock: bool

    class Config:
        from_attributes = True  # ORM 模式：sqlalchemy → pydantic 自动映射
```

### routers/items.py（具体接口）

```python
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.orm import Session
from typing import list

import models, schemas, database

router = APIRouter()

def get_db():
    db = database.SessionLocal()
    try:
        yield db
    finally:
        db.close()

# ── CREATE ──
@router.post("/", response_model=schemas.ItemOut, status_code=201)
def create_item(item: schemas.ItemCreate, db: Session = Depends(get_db)):
    db_item = models.Item(**item.model_dump())
    db.add(db_item)
    db.commit()
    db.refresh(db_item)
    return db_item

# ── READ ALL ──
@router.get("/", response_model=list[schemas.ItemOut])
def read_items(skip: int = 0, limit: int = 100, db: Session = Depends(get_db)):
    return db.query(models.Item).offset(skip).limit(limit).all()

# ── READ ONE ──
@router.get("/{item_id}", response_model=schemas.ItemOut)
def read_item(item_id: int, db: Session = Depends(get_db)):
    item = db.query(models.Item).get(item_id)
    if not item:
        raise HTTPException(status_code=404, detail="物品不存在")
    return item

# ── UPDATE ──
@router.put("/{item_id}", response_model=schemas.ItemOut)
def update_item(item_id: int, item: schemas.ItemCreate, db: Session = Depends(get_db)):
    db_item = db.query(models.Item).get(item_id)
    if not db_item:
        raise HTTPException(status_code=404, detail="物品不存在")
    for key, value in item.model_dump().items():
        setattr(db_item, key, value)
    db.commit()
    db.refresh(db_item)
    return db_item

# ── DELETE ──
@router.delete("/{item_id}", status_code=204)
def delete_item(item_id: int, db: Session = Depends(get_db)):
    db_item = db.query(models.Item).get(item_id)
    if not db_item:
        raise HTTPException(status_code=404, detail="物品不存在")
    db.delete(db_item)
    db.commit()
    return None  # 204 状态码不返回内容
```

---

## 十、运行 + 测试

**启动服务**：

```bash
uvicorn main:app --reload
```

**在 `/docs` 测试**：

打开 `http://127.0.0.1:8000/docs`，你会看到所有接口的列表。点击任意接口 → "Try it out" → 填参数 → Execute。不用 Postman，直接在浏览器里完成。

**写测试**：

```python
# test_main.py
from fastapi.testclient import TestClient
from main import app

client = TestClient(app)

def test_read_items():
    response = client.get("/items/")
    assert response.status_code == 200

def test_create_item():
    response = client.post("/items/", json={
        "name": "测试物品",
        "price": 1000,
        "description": "测试"
    })
    assert response.status_code == 201
    assert response.json()["name"] == "测试物品"
```

```bash
pip install pytest httpx
pytest test_main.py -v
```

---

## 十一、生产部署

### 方式一：直接用 uvicorn

```bash
uvicorn main:app --host 0.0.0.0 --port 8000 --workers 4
# --workers 4：用 4 个进程（多核利用）
```

### 方式二：Docker

```dockerfile
FROM python:3.11
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

```bash
docker build -t my-fastapi-app .
docker run -p 8000:8000 my-fastapi-app
```

### 方式三：直接部署到云

```bash
pip install fastapi-cloud
fastapi deploy
```

---

## 十二、常见问题

**Q：选 FastAPI 还是 Flask？**

- 需要**自动文档 + 数据校验 + 高性能** → FastAPI
- 只是简单脚本、小工具 → Flask 也可以

**Q：要不要用 async？**

- 如果接口里有 **I/O 操作**（数据库、HTTP 请求、文件读写）→ 用 `async def`
- 如果只是 CPU 计算 → 普通 `def` 就行
- 混用也可以：同步函数在异步框架里会自动被包装成线程

```python
# 正确的 async 用法
@app.get("/items/{item_id}")
async def get_item(item_id: int, db: Session = Depends(get_db)):
    # SQLAlchemy 的 session 在 async 里需要特殊处理
    # 建议用 asyncpg 或者直接用同步方式（FastAPI 会自动处理）
    item = db.query(Item).get(item_id)
    return item
```

**Q：SQLAlchemy 同步还是异步？**

初学者用**同步版本**就够了，FastAPI 会自动在线程池里运行，不影响性能。只有在真正遇到瓶颈时才升级到 async sqlalchemy + `async_sessionmaker`。

**Q：生产环境还需要什么？**

- **Gunicorn** + Uvicorn workers（多进程）
- **PostgreSQL**（生产数据库）
- **Redis**（缓存、限流）
- **Docker**（容器化）
- **CI/CD**（自动测试 + 部署）
- **日志**：`logging` + 结构化日志（JSON 格式）

---

## 参考资料

- [FastAPI 官方文档](https://fastapi.tiangolo.com/tutorial/first-steps/)
- [Real Python - Get Started With FastAPI](https://realpython.com/get-started-with-fastapi/)
- [Auth0 - FastAPI Best Practices](https://auth0.com/blog/fastapi-best-practices/)
- [GitHub - FastAPI Best Practices (zhanymkanov)](https://github.com/zhanymkanov/fastapi-best-practices)
- [Real Python - FastAPI Example Application](https://realpython.com/fastapi-python-web-apis/)
- [OneUptime - FastAPI Authentication Middleware](https://oneuptime.com/blog/post/2026-01-25-fastapi-authentication-middleware/view)
