---
title: Python中开启多线程的装饰器
date: 2020-03-02 16:57:49
tags: python thread 多进程
categories: python加油鸭
---

<!--more-->

## Python中开启多线程的装饰器

```python
import random
import threading
import time


def run_async(func):
    def wrapper(*args, **kwargs):
        thr = threading.Thread(target = func, args = args, kwargs = kwargs)
        thr.start()
        thr.setName("func-{}".format(func.__name__))
        # thr.join()
        print("线程id={},\n线程名称={},\n正在执行的线程列表:{},\n正在执行的线程数量={},\n当前激活线程={}".format(
            thr.ident,thr.getName(),threading.enumerate(),threading.active_count(),thr.isAlive)
        )
    return wrapper



@run_async
def test(param):
    while True:
        time.sleep(3)
        t = random.randint(1,param)
        print("/test:" + str(t))


@run_async
def test2(param):
    while True:
        time.sleep(2)
        t = random.randint(1,param)
        print("/test2:" + str(t))



# test(10)
# test2(1000)
```