---
title: pip install 安装速度很慢问题解决方案
date: 2020-03-11 17:40:32
tags: pip python
categories: python加油鸭
---

<!--more-->

#### 临时方案：

- 清华：https://pypi.tuna.tsinghua.edu.cn/simple
- 阿里云：http://mirrors.aliyun.com/pypi/simple/
- 中国科技大学 https://pypi.mirrors.ustc.edu.cn/simple/
- 华中理工大学：http://pypi.hustunique.com/
- 山东理工大学：http://pypi.sdutlinux.org/
- 豆瓣：http://pypi.douban.com/simple/  
  ————————————————————————  
  把上边的地址替换成 -i 后面即可，个人感觉豆瓣源比较快  
  pip install Flask -i https://pypi.douban.com/simple/

#### 永久方案：

```bash
mkdir ~/.pip
vim ~/.pip/pip.conf
```

**添加以下内容**

```bash
[global]
index-url = https://mirrors.cloud.tencent.com/pypi/simple
```