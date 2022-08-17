---
title: Python实现通用装饰器
date: 2020-03-02 17:03:39
tags: python
categories: python加油鸭
---

<!--more-->

```python
from functools import wraps

def decorator(func):
    @wraps(func)
    def inner(*args, **kwargs):
		
		# 这里可以写你额外希望增加功能的代码
            
        return func(*args, **kwargs)
    return inner
```