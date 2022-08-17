---
title: Python定时任务 每隔一定时间运行一次函数/任务 python的定时任务
date: 2020-03-06 14:43:33
tags: 
categories: python加油鸭
---

<!--more-->

#### 每隔一定时间运行一次函数/任务 例子中每10秒执行一次…

```python
def func_task():
    print('执行任务中...')

def func_timer():

    func_task()
    global timer  # 定义全局变量
    # 定时器构造函数主要有2个参数，第一个参数为时间，第二个参数为函数名
    timer = threading.Timer(10, func_timer)   # 10秒调用一次函数

    print("线程名称={},\n正在执行的线程列表:{},\n正在执行的线程数量={},\n当前激活线程={}\n".format(
        timer.getName(), threading.enumerate(), threading.active_count(), timer.isAlive)
    )

    timer.start()    #启用定时器

timer = threading.Timer(1, func_timer)
timer.start()
print('定时器启动成功-----')
```