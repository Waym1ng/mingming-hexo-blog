## 搭建hexo博客
### 这里是存放源码的仓库
- 首先需要配置好ssh
- 建立链接
> git remote add blog https://github.com/Waym1ng/mingming-hexo-blog.git
- 代码更新时
> git push -u blog main

### 另一个仓库部署用 [Waym1ng](https://github.com/Waym1ng/Waym1ng.github.io)
- 安装部署插件 
> npm install hexo-deployer-git --save
- 需要在hexo根目录下配置 _config.yml 文件
    ```
    deploy:
    type: 'git'
    repo: git@github.com:Waym1ng/Waym1ng.github.io.git
    branch: main
    ```
- 建立链接
> git remote add origin https://github.com/Waym1ng/Waym1ng.github.io.git
- 代码更新时
> hexo c

> hexo g

> hexo d
 
 ### hexo常用命令
 ```
hexo new "postName" # 新建文章
hexo new page "pageName" # 新建页面
hexo clean # 清理静态文件
hexo generate # 生成静态页面至public目录
hexo server # 开启预览访问端口（默认端口4000，'ctrl + c'关闭server）
hexo deploy # 部署到GitHub
hexo help  # 查看帮助
hexo version  # 查看Hexo的版本

这里的命令都可简写，如：
hexo n ==> hexo new
hexo g ==> hexo generate
hexo s ==> hexo server
hexo d ==> hexo deploy
 ```

