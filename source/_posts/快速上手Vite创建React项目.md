---
title: 快速上手Vite创建React项目
date: 2025-12-25 12:52:13
tags: 前端
categories: 前端
---

<!--more-->

### 准备工作 (Prerequisites)

在开始之前，请确保你的电脑上已经安装了 **Node.js**。Vite 需要 Node.js 版本 `18+` 或 `20+`。

你可以通过在终端（Terminal）或命令提示符（Command Prompt）中运行以下命令来检查你的 Node.js 版本：

```bash
node -v
```

如果未安装或版本过低，请从 [Node.js 官网](https://nodejs.org/) 下载并安装。

---

### 一、初始化项目 (Initialization)

你只需要一个命令就可以开始。打开你的终端，然后运行：

```bash
npm create vite@latest
```

`yarn` 或 `pnpm` 用户也可以使用：

```bash
# yarn
yarn create vite

# pnpm
pnpm create vite
```

---

### 二、交互式步骤详解 (Interactive Steps)

运行上述命令后，Vite 会通过一系列交互式问题来引导你完成项目创建。

1.  **Project name:** 首先，它会提示你输入项目名称。这将会是你项目的文件夹名称。

    ```
    ✔ Project name: … <your-project-name>
    ```

    输入项目名后按回车，例如 `my-react-app`。

2.  **Select a framework:** 接下来，Vite 会让你选择一个前端框架。使用键盘的**上下箭头**移动，选择 **React**，然后按回车。

    ```
    ✔ Select a framework: › - Use arrow keys. Return to submit.
      Vanilla
    ❯ React
      Vue
      ...
    ```

3.  **Select a variant:** 然后，你需要选择一个变体（variant）。这决定了你的项目是使用 JavaScript 还是 TypeScript。

    ```
    ✔ Select a variant: › - Use arrow keys. Return to submit.
    ❯ TypeScript
      TypeScript + SWC
      JavaScript
      JavaScript + SWC
    ```

    -   **TypeScript / TypeScript + SWC**: 如果你想使用 TypeScript (推荐用于大型项目)。
    -   **JavaScript / JavaScript + SWC**: 如果你只想使用纯 JavaScript。
    -   **SWC 是什么？** SWC 是一个基于 Rust 的超高速编译器，可以替代 Babel。对于大多数项目，选择不带 SWC 的标准 `TypeScript` 或 `JavaScript` 就完全足够了。

    选择你需要的版本后按回车。

---

### 三、启动项目 (Running the Project)

项目文件创建完成后，终端会显示接下来的三个步骤。

1.  **进入项目目录：**

    ```bash
    cd my-react-app
    ```

2.  **安装项目依赖：**

    ```bash
    npm install
    ```

    这个命令会读取 `package.json` 文件并下载所有必需的库（如 React, ReactDOM 等）到 `node_modules` 文件夹中。

3.  **启动开发服务器：**

    ```bash
    npm run dev
    ```

运行后，你会看到类似下面的输出，告诉你开发服务器已经成功启动：

```
  VITE v5.2.10  ready in 320 ms

  ➜  Local:   http://localhost:5173/
  ➜  Network: use --host to expose
  ➜  press h + enter to show help
```

现在，打开你的浏览器并访问 `http://localhost:5173/`，你就能看到一个全新的 React 应用正在运行了！

### 命令总结 (TL;DR)

如果你只需要快速复制粘贴命令，这里是全过程：

```bash
# 1. 创建项目 (跟随交互式提示选择 React 和 JavaScript/TypeScript)
npm create vite@latest

# 2. 进入项目目录 (将 my-react-app 替换为你的项目名)
cd my-react-app

# 3. 安装依赖
npm install

# 4. 启动！
npm run dev
```

享受 Vite 带来的闪电般的开发体验吧！
