# Hexo 博客源码仓库

本仓库存放 Hexo 博客的源码，部署后的静态网站托管在 [Waym1ng.github.io](https://github.com/Waym1ng/Waym1ng.github.io)。

## 仓库说明

- **源码仓库**（本仓库）：存放博客源码、配置文件、文章内容等
  - 地址：https://github.com/Waym1ng/mingming-hexo-blog.git
- **部署仓库**：存放由 Hexo 生成的静态网页
  - 地址：https://github.com/Waym1ng/Waym1ng.github.io.git

---

## 环境准备

### 1. 配置 SSH 密钥

#### 检查是否已有 SSH 密钥
打开终端，运行以下命令检查是否已存在 SSH 密钥：
```bash
ls ~/.ssh
```

如果看到类似 `id_ed25519` 和 `id_ed25519.pub` 或 `id_rsa` 和 `id_rsa.pub` 的文件，说明已存在密钥，可以跳过生成步骤。

#### 生成新的 SSH 密钥
如果没有密钥，运行以下命令生成：
```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

按提示操作：
- 可以直接按回车使用默认路径
- 可以设置密码（也可以直接回车跳过）

#### 将公钥添加到 GitHub

**复制公钥内容**：
```bash
cat ~/.ssh/id_ed25519.pub
```
或使用 Windows 系统时：
```bash
type %userprofile%\.ssh\id_ed25519.pub
```

**在 GitHub 添加 SSH Keys**：
1. 登录 GitHub，点击右上角头像 → **Settings**
2. 左侧菜单选择 **SSH and GPG keys**
3. 点击 **New SSH key** 按钮
4. 填写 Title（如 "My PC"）
5. 将复制的公钥粘贴到 Key 文本框中
6. 点击 **Add SSH key**

#### 测试 SSH 连接
```bash
ssh -T git@github.com
```

如果看到类似 `Hi Waym1ng! You've successfully authenticated` 的消息，说明配置成功。

#### 常见问题
- 如果提示 `Connection timed out`，请检查网络连接或代理设置
- 如果提示 `Permission denied`，请确认公钥是否正确添加到 GitHub

---

### 2. 安装依赖

#### 安装 Node.js 和 npm
访问 [Node.js 官网](https://nodejs.org/) 下载并安装 LTS 版本。

安装完成后检查版本：
```bash
node -v
npm -v
```

#### 安装 Hexo
```bash
npm install -g hexo-cli
```

#### 安装部署插件
```bash
npm install hexo-deployer-git --save
```

---

## 项目初始化

### 1. 克隆源码仓库
```bash
git clone https://github.com/Waym1ng/mingming-hexo-blog.git
cd mingming-hexo-blog
```

### 2. 安装项目依赖
```bash
npm install
```

### 3. 配置部署信息
在项目根目录的 `_config.yml` 文件末尾配置部署信息：

```yaml
deploy:
  type: git
  repo: git@github.com:Waym1ng/Waym1ng.github.io.git
  branch: main
```

### 4. 建立远程仓库连接
```bash
git remote add blog https://github.com/Waym1ng/mingming-hexo-blog.git
```

---

## 工作流程

### 本地开发

#### 新建文章
```bash
hexo new "文章标题"
```
文章会创建在 `source/_posts/` 目录下，使用 Markdown 编辑器编辑内容。

#### 新建页面
```bash
hexo new page "页面名称"
```

#### 本地预览
```bash
hexo s
```
打开浏览器访问 `http://localhost:4000` 查看效果，按 `Ctrl + C` 关闭服务器。

---

### 源码管理（更新源码到 mingming-hexo-blog 仓库）

当修改了文章、配置等源码后，提交到源码仓库：

```bash
git add .
git commit -m "提交说明"
git push -u blog main
```

---

### 部署发布（发布到 Waym1ng.github.io 仓库）

当需要部署博客到线上时，执行以下命令：

```bash
hexo clean    # 清理缓存和静态文件
hexo generate # 生成静态文件到 public 目录
hexo deploy   # 部署到 GitHub Pages
```

**命令简写**：
```bash
hexo c  # hexo clean
hexo g  # hexo generate
hexo d  # hexo deploy
```

也可以合并执行：
```bash
hexo clean && hexo generate && hexo deploy
```

或使用简写：
```bash
hexo c && hexo g && hexo d
```

部署完成后，访问 `https://waym1ng.github.io` 即可查看博客。

---

## 常用命令速查

### 基础命令
```bash
hexo new "postName"        # 新建文章
hexo new page "pageName"   # 新建页面
hexo clean                 # 清理静态文件
hexo generate              # 生成静态页面至 public 目录
hexo server                # 开启预览访问端口（默认端口 4000，Ctrl + C 关闭）
hexo deploy                # 部署到 GitHub
hexo help                  # 查看帮助
hexo version               # 查看 Hexo 的版本
```

### 命令简写
```bash
hexo n  ==> hexo new
hexo g  ==> hexo generate
hexo s  ==> hexo server
hexo d  ==> hexo deploy
hexo c  ==> hexo clean
```

### 完整部署流程
```bash
hexo clean && hexo generate && hexo deploy
```

---

## 常见问题

### 部署时提示 Permission denied
确保已正确配置 SSH 密钥，并测试连接：`ssh -T git@github.com`

### 部署后页面未更新
- 检查 `_config.yml` 中的 `deploy` 配置是否正确
- 确保 GitHub Pages 设置中的分支为 `main`
- GitHub Pages 部署可能需要几分钟时间

### 本地预览图片无法显示
确保图片路径正确，建议使用相对路径或放在 `source/images/` 目录下
