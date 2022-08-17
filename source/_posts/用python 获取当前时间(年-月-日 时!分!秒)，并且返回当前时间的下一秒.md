---
title: 用python 获取当前时间(年-月-日 时:分:秒)，并且返回当前时间的下一秒
date: 2019-03-02 11:24:13
tags: python基础 datetime模块
categories: python加油鸭
---

<!--more-->

# 获取当前时间，并且返回当前时间的下一秒

因为存在年-月-日 时:分:秒  
考虑到用split的方法做的话非常麻烦  
所以引入time和datetime模块  
当然 也可以改写成输入一个时间

### 代码实现

```python
import datetime,time
# 获取当前时间
now_time = datetime.datetime.now()
# 格式化时间字符串
str_time = now_time.strftime("%Y-%m-%d %H:%M:%S")
tup_time = time.strptime(str_time,"%Y-%m-%d %H:%M:%S")
time_sec = time.mktime(tup_time)
# 转换成时间戳 进行计算
time_sec += 1
tup_time2 = time.localtime(time_sec)
str_time2 = time.strftime("%Y-%m-%d %H:%M:%S",tup_time2)
print(str_time)
print(str_time2)
```

### 关于格式问题

```python
%a 星期的简写。如 星期三为Web
%A 星期的全写。如 星期三为Wednesday
%b 月份的简写。如4月份为Apr
%B 月份的全写。如4月份为April
%c:  日期时间的字符串表示。（如： 04/07/10 10:43:39）
%d:  日在这个月中的天数（是这个月的第几天）
%f:  微秒（范围[0,999999]）
%H:  小时（24小时制，[0, 23]）
%I:  小时（12小时制，[0, 11]）
%j:  日在年中的天数 [001,366]（是当年的第几天）
%m:  月份（[01,12]）
%M:  分钟（[00,59]）
%p:  AM或者PM
%S:  秒
%U:  周在当年的周数当年的第几周），星期天作为周的第一天
%w:  今天在这周的天数，范围为[0, 6]，6表示星期天
%W:  周在当年的周数（是当年的第几周），星期一作为周的第一天
%x:  日期字符串（如：04/07/10）
%X:  时间字符串（如：10:43:39）
%y:  2个数字表示的年份
%Y:  4个数字表示的年份
%z:  与utc时间的间隔 （如果是本地时间，返回空字符串）
%Z:  时区名称（如果是本地时间，返回空字符串）
%%:  %% => %
```