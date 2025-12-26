---
title: uv 下一代 Python 包管理工具终极教程
date: 2025-12-24 15:32:13
tags: python uv
categories: python加油鸭
---

<!--more-->


# 🚀 uv: 下一代 Python 包管理工具终极教程

欢迎来到 `uv` 的世界！`uv` 是一个用 Rust 编写的、快如闪电的 Python 包管理器和解析器。它旨在成为 `pip`、`pip-tools` 和 `venv` 的直接替代品，并提供无与伦比的性能。

本教程将带您全面了解 `uv` 的核心功能，并通过与传统 `pip` 的详细对比，助您轻松上手。

> **💡 核心优势：**
>
> -   **极速 ⚡️**：比 `pip` 和 `pip-tools` 快 10-100 倍。
>
> -   **节省空间 💾**：通过全局缓存，避免重复下载和存储同一个包。
>
> -   **一体化 📦**：同时处理包安装、虚拟环境创建和依赖锁定。
>
> -   **直接替代**：与 `pip` 的命令高度兼容，学习成本低。

## 📥 安装 uv

首先，让我们将 `uv` 安装到您的系统中。

**macOS 和 Linux**

```
curl -LsSf [https://astral.sh/uv/install.sh](https://astral.sh/uv/install.sh) | sh
```

**Windows**

```
powershell -c "irm [https://astral.sh/uv/install.ps1](https://astral.sh/uv/install.ps1) | iex"
```

安装完成后，通过运行 `uv --version` 来验证是否成功。

## 🆚 核心理念对比：uv vs. pip

理解 `uv` 的强大之处，关键在于其**全局缓存**机制。

| **特性**     | **uv**                                         | **pip + venv**                                   |
| :----------- | :--------------------------------------------- | :----------------------------------------------- |
| **包存储**   | 全局统一缓存，多项目通过**链接**共享           | 每个虚拟环境都**复制**一份完整的包文件           |
| **磁盘占用** | **极低**。无论多少项目使用，同一个包只存一份。 | **较高**。项目越多，重复的包越多，占用空间越大。 |
| **安装速度** | 首次安装后，后续安装几乎是瞬时的（无需下载）。 | 每个新环境都需要重新下载或从本地缓存复制。       |
| **环境创建** | 内置 `uv venv` 命令，速度极快。                | 使用 Python 内置的 `venv` 模块，速度较慢。       |

## 🛠️ `uv --help` 核心命令详解

`uv` 的许多命令都封装在 `uv pip` 子命令下，以保持与 `pip` 的兼容性。

### 1. 虚拟环境管理 (`uv venv`)

`uv` 内置了超高速的虚拟环境创建工具。

**对比**

-   **uv**: `uv venv`

-   **传统**: `python -m venv .venv`

**示例：**

```
# 创建一个名为 .venv 的虚拟环境
uv venv

# 指定 Python 解释器版本创建
uv venv --python 3.11

# 激活虚拟环境 (与传统方式相同)
# macOS / Linux
source .venv/bin/activate
# Windows
.venv\Scripts\activate
```

### 2. 安装包 (`uv pip install`)

这是最常用的命令，`uv` 在此展现了惊人的速度。

**对比**

-   **uv**: `uv pip install <package>`

-   **传统**: `pip install <package>`

**常用参数示例：**

-   **安装单个或多个包**

    ```
    # 安装 FastAPI 和 requests
    uv pip install "fastapi[all]" requests
    ```

-   **安装指定版本**

    ```
    # 安装 0.109.x 版本的 fastapi
    uv pip install "fastapi<0.110"

    # 安装特定版本的 httpx
    uv pip install httpx==0.27.0
    ```

-   **从 `requirements.txt` 文件安装 (`-r, --requirement`)**

    ```
    # 从 requirements.txt 安装所有依赖
    uv pip install -r requirements.txt

    # 从多个文件安装
    uv pip install -r requirements.txt -r requirements-dev.txt
    ```

-   **安装可编辑包 (`-e, --editable`)** (用于本地开发)

    ```
    # 将当前目录作为一个可编辑包安装
    uv pip install -e .
    ```

### 3. 卸载包 (`uv pip uninstall`)

**对比**

-   **uv**: `uv pip uninstall <package>`

-   **传统**: `pip uninstall <package>`

**示例：**

```
# 卸载 fastapi
uv pip uninstall fastapi

# 从文件中卸载所有包
uv pip uninstall -r requirements.txt
```

### 4. 查看已安装的包 (`uv pip list`)

**对比**

-   **uv**: `uv pip list`

-   **传统**: `pip list`

**常用参数示例：**

-   **标准列表**

    ```
    uv pip list
    ```

-   **排除可编辑包 (`--exclude-editable`)**

    ```
    uv pip list --exclude-editable
    ```

### 5. 生成依赖列表 (`uv pip freeze`)

用于生成 `requirements.txt` 文件。

**对比**

-   **uv**: `uv pip freeze`

-   **传统**: `pip freeze`

**示例：**

```
# 将当前环境的依赖输出到 requirements.txt
uv pip freeze > requirements.txt
```

### 6. 同步环境 (`uv pip sync`) ⭐

这是 `uv` 的一个**杀手级功能**，它能确保你的虚拟环境与 `requirements.txt` 文件**完全一致**。它会自动安装缺失的包，并**移除**环境中存在但文件中没有的包。

**对比**

-   **uv**: `uv pip sync requirements.txt`

-   **传统**: 无直接命令。需要先 `pip install -r requirements.txt`，然后手动对比并卸载多余的包。

**示例：**
假设 `requirements.txt` 内容为：

```
fastapi==0.109.2
```

而你的环境中还安装了 `requests`。

```
# 运行同步命令
uv pip sync requirements.txt

# 结果：uv 会自动卸载 requests，因为文件中没有它
# Uninstalled 1 package: requests
```

### 7. 清理缓存 (`uv cache clean`)

管理 `uv` 的全局缓存。

**对比**

-   **uv**: `uv cache clean`

-   **传统**: `pip cache purge`

**示例：**

```
# 清理所有未使用的缓存数据
uv cache clean
```

## 🌀 完整项目工作流示例

让我们用 `uv` 从零开始搭建一个简单的 FastAPI 项目。

**1. 创建项目并初始化环境**

```
mkdir my-fastapi-app
cd my-fastapi-app

# 创建并激活虚拟环境
uv venv
source .venv/bin/activate
```

**2. 安装依赖**

```
# 安装 FastAPI, 包含所有可选依赖（如 uvicorn）
uv pip install "fastapi[all]"
```

**3. 编写应用代码**
创建一个 `main.py` 文件：

```
# main.py
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
async def read_root():
    return {"message": "Hello, World with uv and FastAPI! 🚀"}
```

**4. 运行开发服务器**

```
# 使用 uvicorn 启动应用，--reload 开启热重载
uvicorn main:app --reload
```

现在，在浏览器中访问 `http://127.0.0.1:8000` 就可以看到 FastAPI 应用了。

**5. 生成 `requirements.txt`**

```
# 将当前环境的依赖输出到 requirements.txt
uv pip freeze > requirements.txt
```

文件内容会包含 `fastapi`、`uvicorn` 及其所有子依赖。

**6. 体验 `sync` 功能**
首先，我们手动安装一个额外的包。

```
uv pip install requests
```

现在，我们的环境比 `requirements.txt` 多了一个 `requests` 包。

接着，我们使用 `sync` 来恢复环境的纯净状态。

```
uv pip sync requirements.txt
```

`uv` 会智能地发现并卸载 `requests`，让环境与依赖文件严格保持一致。

## ✨ 结语

`uv` 不仅仅是一个更快的 `pip`，它通过现代化的设计理念，为 Python 开发者带来了更高效、更整洁、更愉悦的包管理体验。从创建环境到安装、同步依赖，`uv` 的一体化解决方案无疑是 Python 开发生态的一大步进。

希望本教程能帮助您顺利地将 `uv` 集成到您的日常工作中！
