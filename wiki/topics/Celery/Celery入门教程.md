# Celery 分布式任务队列入门教程

## 概述

Celery 是一个 Python 实现的分布式任务队列，用于处理异步任务、定时任务和延时任务。广泛应用于发送邮件、数据处理、图像处理等耗时操作，避免阻塞主请求。

**一句话理解：** 把要干的活丢给 Celery，它帮你排队、分配工人、完成后通知你。

---

## 核心概念

```
┌──────────────┐     ┌─────────────┐     ┌──────────────────┐
│   Django/    │     │             │     │                  │
│   Flask      │────▶│   Broker    │────▶│   Celery Worker  │
│   (Producer) │     │  (RabbitMQ/  │     │   (Consumer)     │
│              │     │   Redis)    │     │                  │
└──────────────┘     └─────────────┘     └──────────────────┘
       │                                           │
       │              ┌─────────────┐              │
       └─────────────▶│   Result    │◀─────────────┘
                      │   Backend   │
                      │  (Redis/DB) │
                      └─────────────┘
```

| 组件 | 说明 | 常见选择 |
|------|------|----------|
| **Producer** | 任务发起方（Django/Flask） | Web 应用 |
| **Broker** | 消息队列，负责接收和分发任务 | Redis、RabbitMQ |
| **Worker** | 实际执行任务的服务进程 | Celery 自带 |
| **Backend** | 存储任务执行结果 | Redis、MySQL、MongoDB |

---

## 环境准备

### 安装依赖

```bash
# 安装 celery + 消息代理(redis) + web框架(django)
pip install celery[redis] django
```

### 配置说明

```bash
# 本教程使用 Redis 作为 Broker 和 Backend（最简单）
# 确保本地 Redis 已运行（默认端口 6379）
redis-server
```

### 创建 Django 项目（已有可跳过）

```bash
django-admin startproject myproject
cd myproject
python manage.py startapp tasks
```

---

## 快速上手（5 步搞定）

### 第 1 步：创建 Celery 配置

在 Django 项目根目录创建 `celery.py`：

```python
# myproject/celery.py
import os
from celery import Celery

# 1. 加载 Django settings
os.environ.setdefault("DJANGO_SETTINGS_MODULE", "myproject.settings")

# 2. 创建 Celery 实例
app = Celery("myproject")

# 3. 从 Django settings 读取配置
app.config_from_object("django.conf:settings", namespace="CELERY")

# 4. 自动发现所有 app 下的 tasks
app.autodiscover_tasks()
```

### 第 2 步：修改 `__init__.py`

让 Django 启动时自动加载 Celery：

```python
# myproject/__init__.py
from .celery import app as celery_app

__all__ = ("celery_app",)
```

### 第 3 步：添加任务（写你要干的活）

在 `tasks/tasks.py` 中定义任务：

```python
# tasks/tasks.py
from celery import shared_task
import time

@shared_task
def send_email_task(email, content):
    """模拟发送邮件（实际项目中替换为真实发送逻辑）"""
    print(f"正在发送邮件到 {email}...")
    time.sleep(3)  # 模拟耗时操作
    print(f"邮件发送成功: {email}")
    return f"邮件已发送至 {email}"

@shared_task
def heavy_computation_task(a, b):
    """耗时的计算任务"""
    result = sum(range(a, b))
    return result
```

### 第 4 步：在 Django View 中调用任务

```python
# tasks/views.py
from django.http import JsonResponse
from .tasks import send_email_task, heavy_computation_task

def trigger_task(request):
    # 1. 触发异步任务（立即返回 task_id）
    task = send_email_task.delay("user@example.com", "Hello!")

    # 2. 也可以调用定时任务/延时任务
    # task = heavy_computation_task.apply_async(args=[1, 1000000], countdown=10)

    return JsonResponse({
        "status": "任务已提交",
        "task_id": task.id
    })

def check_task_result(request):
    """查询任务执行结果"""
    task_id = request.GET.get("task_id")
    from django_celery_results.models import TaskResult

    result = TaskResult.objects.filter(task_id=task_id).first()
    if result:
        return JsonResponse({
            "task_id": task_id,
            "status": result.status,
            "result": result.result
        })
    return JsonResponse({"status": "任务不存在"})
```

### 第 5 步：启动 Worker 监听任务

```bash
# 方式一：在 Django 项目目录启动
celery -A myproject worker --loglevel=info

# 方式二：后台运行（Linux/Mac）
celery -A myproject worker --loglevel=info --detach

# 方式三：指定日志文件
celery -A myproject worker --loglevel=info --logfile=celery.log
```

---

## 三种任务调用方式

| 方式 | 代码 | 说明 |
|------|------|------|
| **异步调用** | `task.delay()` | 立即提交，立即返回 |
| **延时调用** | `task.apply_async(countdown=10)` | 指定秒数后执行 |
| **定时调用** | `task.apply_async(eta=时间对象)` | 指定具体时间执行 |

```python
# 立即异步调用
task = send_email_task.delay("a@b.com", "hello")

# 10 秒后执行
task = heavy_computation_task.apply_async(args=[1, 1000000], countdown=10)

# 明天早上 9 点执行
from datetime import datetime, timedelta
execute_time = datetime(2026, 4, 18, 9, 0, 0)
task = send_email_task.apply_async(args=["a@b.com", "hi"], eta=execute_time)
```

---

## 定时任务（Celery Beat）

Celery Beat 负责按时触发周期性任务，类似 Linux 的 cron。

### 1. 安装 django-celery-beat

```bash
pip install django-celery-beat
```

### 2. 配置 Django settings

```python
# settings.py
INSTALLED_APPS = [
    ...
    "django_celery_beat",
    "django_celery_results",
]

CELERY_BROKER_URL = "redis://localhost:6379/0"
CELERY_RESULT_BACKEND = "django-db"
CELERY_CACHE_BACKEND = "django-cache"
CELERYBEAT_SCHEDULER = "django_celery_beat.schedulers:DatabaseScheduler"
```

### 3. 迁移数据库

```bash
python manage.py migrate
```

### 4. 启动 Beat

```bash
# 启动定时任务调度器
celery -A myproject beat --loglevel=info

# 或者分开窗口同时运行 worker + beat
celery -A myproject worker --loglevel=info
celery -A myproject beat --loglevel=info
```

### 5. 通过 Django Admin 配置定时任务

访问 `http://127.0.0.1:8000/admin/`，在 django_celery_beat 下添加 Periodic Task：

- **Name**: 任务名称
- **Task name**: `tasks.tasks.send_email_task`
- **Interval**: 每隔多久执行（如每 5 分钟）
- **Arguments**: `["admin@example.com", "定时任务测试"]`

---

## 查看任务状态（Result Backend）

### 方式一：命令行查看

```bash
# 查看任务结果
celery -A myproject result <task_id>
```

### 方式二：Django Admin 查看

安装 `django-celery-results` 后，自动在 Admin 显示任务执行结果。

### 方式三：代码中查询

```python
from celery.result import AsyncResult

task = AsyncResult("<task_id>")
print(f"状态: {task.state}")        # PENDING / STARTED / SUCCESS / FAILURE
print(f"结果: {task.result}")       # 返回值（任务成功时）
print(f"错误: {task.traceback}")   # 异常堆栈（失败时）
```

---

## 完整项目结构

```
myproject/
├── myproject/
│   ├── __init__.py        # 加载 celery_app
│   ├── celery.py          # Celery 配置
│   ├── settings.py        # Django settings（含 CELERY_* 配置）
│   └── urls.py
├── tasks/
│   ├── __init__.py
│   ├── tasks.py           # 定义的所有任务
│   ├── views.py           # 调用任务的视图
│   └── urls.py
└── manage.py
```

---

## 常见错误排查

| 错误 | 原因 | 解决 |
|------|------|------|
| `Connection refused` | Redis 未启动 | `redis-server` 先启动 Redis |
| `kombu.exceptions ImproperlyConfigured` | 未设置 `DJANGO_SETTINGS_MODULE` | 确认环境变量 |
| 任务卡住不执行 | Worker 未启动 | 新开窗口运行 `celery -A myproject worker` |
| 任务结果查不到 | 未配置 Result Backend | 配置 `CELERY_RESULT_BACKEND` |
| 定时任务不触发 | Beat 未启动 | 确认 Beat 和 Worker 同时运行 |

---

## 注意事项

1. **Broker 必须稳定**：Celery 依赖 Redis/RabbitMQ 做消息传递，Broker 挂了所有任务都丢
2. **任务要有幂等性**：同一个任务可能被执行多次，设计时考虑重复执行的影响
3. **结果不过期**：Result Backend 中任务结果默认不删除，定期清理
4. **Worker 水平扩展**：多台机器跑 Worker，Celery 自动负载均衡

---

## 参考资料

- [Celery 官方文档](https://docs.celeryproject.org/)
- [Django-Celery-Beat 文档](https://django-celery-beat.readthedocs.io/)
- [Redis 官网](https://redis.io/)
