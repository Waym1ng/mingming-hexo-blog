---
title: 用python 获取当前时间(年-月-日 时:分:秒)，并且返回当前时间的下一秒
date: 2022-06-23 21:40:49
tags:
- 基础
categories:
- python
---

# 获取当前时间，并且返回当前时间的下一秒
因为存在年-月-日 时:分:秒
考虑到用split的方法做的话非常麻烦
所以引入time和datetime模块
当然 也可以改写成输入一个时间

```python
import datetime,time
# 获取当前时间
now_time = datetime.datetime.now()
# 格式化时间字符串
str_time = now_time.strftime("%Y-%m-%d %X")
tup_time = time.strptime(str_time,"%Y-%m-%d %X")
time_sec = time.mktime(tup_time)
# 转换成时间戳 进行计算
time_sec += 1
tup_time2 = time.localtime(time_sec)
str_time2 = time.strftime("%Y-%m-%d %X",tup_time2)
print(str_time)
print(str_time2)

```
