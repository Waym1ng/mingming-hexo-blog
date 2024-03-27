---
title: feapder爬虫框架之轻量AirSpider用法示例
date: 2024-03-27 19:39:59
tags: 爬虫
categories: 爬虫


---

<!--more-->

## 安装

精简版

```shell
pip install feapder
```

浏览器渲染版：

```shell
pip install "feapder[render]"
```

完整版：

```shell
pip install "feapder[all]"
```

三个版本区别：

1. 精简版：不支持浏览器渲染、不支持基于内存去重、不支持入库mongo
2. 浏览器渲染版：不支持基于内存去重、不支持入库mongo
3. 完整版：支持所有功能

## 使用

> AirSpider是一款轻量爬虫，学习成本低。面对一些数据量较少，无需断点续爬，无需分布式采集的需求，可采用此爬虫

创建模板命令：`feapder create -s air_spider_test` 

请选择爬虫模板  *AirSpider* 

```
# -*- coding: utf-8 -*-
"""
Created on 2024-03-26 11:57:14
---------
@summary:
---------
@author: zhangmingwei
"""

import feapder


class AirSpiderTest(feapder.AirSpider):
    def start_requests(self):
        yield feapder.Request("https://spidertools.cn")

    def parse(self, request, response):
        # 提取网站title
        print(response.xpath("//title/text()").extract_first())
        # 提取网站描述
        print(response.xpath("//meta[@name='description']/@content").extract_first())
        print("网站地址: ", response.url)


if __name__ == "__main__":
    AirSpiderTest().start()
```

