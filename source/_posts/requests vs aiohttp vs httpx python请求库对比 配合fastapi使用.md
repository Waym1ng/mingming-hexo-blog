一份 **面向 FastAPI 的、工程级别的对比结论**，站在**异步架构 / 并发 / 生产可维护性**角度来选型。

---

# 一句话结论（先给结论）

> ✅ **FastAPI 项目：优先选择 `httpx`**
> ❌ **不要在 FastAPI 的接口里直接用 `requests`**
> ⚠️ `aiohttp` 可用，但仅在特定场景下才值得选

排序（FastAPI 场景）：

```
httpx   >   aiohttp   >>>   requests
```

---

# 一、三者核心定位对比

| 库           | 核心定位                                      |
| ------------ | --------------------------------------------- |
| **requests** | 同步 HTTP 客户端（经典库，非异步）            |
| **aiohttp**  | 早期异步 HTTP + Web 框架                      |
| **httpx**    | 现代化 requests 兼容的同步 + 异步 HTTP 客户端 |

---

# 二、与 FastAPI 的「本质兼容性」

FastAPI 本质是：

-   ASGI
-   asyncio
-   高并发、非阻塞 I/O

### 是否“天然适配” FastAPI？

| 库       | 是否阻塞事件循环 | 结论           |
| -------- | ---------------- | -------------- |
| requests | ❌ 是            | **严重不适配** |
| aiohttp  | ✅ 否            | 适配           |
| httpx    | ✅ 否            | **完美适配**   |

---

# 三、详细对比表（工程级）

## 1️⃣ 并发 / 异步能力

| 项目             | requests | aiohttp | httpx |
| ---------------- | -------- | ------- | ----- |
| 异步支持         | ❌       | ✅      | ✅    |
| asyncio 原生     | ❌       | ✅      | ✅    |
| async/await      | ❌       | ✅      | ✅    |
| FastAPI 并发友好 | ❌       | ✅      | ✅    |

🚨 **FastAPI 中用 requests = 阻塞 worker**

---

## 2️⃣ API 设计 & 可维护性

| 项目              | requests   | aiohttp | httpx      |
| ----------------- | ---------- | ------- | ---------- |
| API 易用性        | ⭐⭐⭐⭐⭐ | ⭐⭐⭐  | ⭐⭐⭐⭐⭐ |
| requests 迁移成本 | 原生       | ❌      | ✅ 100%    |
| 心智负担          | 极低       | 偏高    | 低         |
| 代码可读性        | 高         | 中      | 高         |

### 示例对比

#### requests

```python
requests.get(url)
```

#### aiohttp

```python
async with aiohttp.ClientSession() as session:
    async with session.get(url) as resp:
        data = await resp.json()
```

#### httpx

```python
async with httpx.AsyncClient() as client:
    r = await client.get(url)
    data = r.json()
```

👉 **httpx = requests + async**

---

## 3️⃣ FastAPI 官方生态支持

| 能力               | requests | aiohttp | httpx |
| ------------------ | -------- | ------- | ----- |
| FastAPI 官方推荐   | ❌       | ⚠️      | ✅    |
| ASGI 集成测试      | ❌       | ❌      | ✅    |
| Starlette 原生支持 | ❌       | ❌      | ✅    |

### FastAPI 官方测试方式（httpx）

```python
from httpx import AsyncClient

async with AsyncClient(app=app, base_url="http://test") as ac:
    res = await ac.get("/ping")
```

⚠️ **aiohttp 无等价能力**

---

## 4️⃣ 性能 & 稳定性（真实生产）

| 场景     | requests | aiohttp  | httpx    |
| -------- | -------- | -------- | -------- |
| 高并发   | ❌       | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| TLS 性能 | ⭐⭐⭐   | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| HTTP/2   | ❌       | ⚠️       | ✅       |
| 连接池   | ⭐⭐⭐   | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |

> aiohttp 理论性能略高
> **但 httpx 的差距 < 5%，生产无感**

---

## 5️⃣ 错误处理 & 工程能力

| 能力         | requests | aiohttp | httpx    |
| ------------ | -------- | ------- | -------- |
| 异常体系     | 简单     | 复杂    | 清晰     |
| Timeout 粒度 | 粗       | 细      | **极细** |
| 重试策略     | ❌       | 手写    | 易封装   |
| 类型提示     | 一般     | 一般    | **优秀** |

### httpx 的 timeout 控制

```python
timeout = httpx.Timeout(
    connect=5.0,
    read=10.0,
    write=5.0,
    pool=5.0,
)
```

---

## 6️⃣ 在 FastAPI 中的「正确姿势」

### ❌ 错误示例（requests）

```python
@app.get("/data")
def data():
    return requests.get(url).json()
```

问题：

-   阻塞 event loop
-   QPS 急剧下降
-   worker 被卡死

---

### ⚠️ 勉强可用（aiohttp）

```python
@app.get("/data")
async def data():
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as resp:
            return await resp.json()
```

问题：

-   Session 不好统一管理
-   API 风格割裂

---

### ✅ 正确姿势（httpx）

```python
client = httpx.AsyncClient(timeout=10)

@app.get("/data")
async def data():
    resp = await client.get(url)
    return resp.json()
```

👉 **推荐：全局 AsyncClient + 生命周期管理**

---

## 7️⃣ 什么时候选 aiohttp？

只有这几种情况：

-   已有大量 aiohttp 代码
-   你需要 aiohttp.web 自带服务端
-   你在写爬虫 / socket 级别控制
-   极端追求性能（差距很小）

否则 ❌ 不推荐新项目选 aiohttp

---

# 四、最终选型建议

## ✅ FastAPI 项目标准答案

| 场景                    | 推荐                  |
| ----------------------- | --------------------- |
| FastAPI 调用第三方 API  | **httpx.AsyncClient** |
| 微服务间调用            | **httpx**             |
| FastAPI 单元 / 集成测试 | **httpx**             |
| 同步脚本                | requests / httpx      |
| 老 aiohttp 项目         | aiohttp               |

---
