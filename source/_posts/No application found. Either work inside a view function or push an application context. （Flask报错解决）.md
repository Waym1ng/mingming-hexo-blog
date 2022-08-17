---
title: No application found. Either work inside a view function or push an application context. （Flask报错解决）
date: 2020-04-03 17:59:40
tags: python 设计模式
categories: python加油鸭
---

<!--more-->

**No application found. Either work inside a view function or push an application context.**

flask 报了这个错,字面意思是说没有应用上下文,字面给的解决意见是要么放置在一个视图内,要么提供一个应用\(flask\)上下文.

这是采用crate\_app\(\)来创建Flask app 的一种错误

可以采用装饰器来解决，通过app.app\_context\(\).push\(\)来推入一个上下文

```python
def sqlalchemy_context(app):
    def add_context(func):
        @wraps(func)
        def do_job(*args, **kwargs):
            app.app_context().push()
            result = func(*args,**kwargs)
            return result
        return do_job
    return add_context
```

**官方解释**  
应用上下文目的  
应用上下文存在的主要原因是，在过去，没有更好的方式来在请求上下文中附加一堆函数， 因为 Flask 设计的支柱之一是你可以在一个 Python 进程中拥有多个应用。

那么代码如何找到“正确的”应用？在过去，我们推荐显式地到处传递应用，但是这导致没有用这种想法设计的库的问题，因为让库实现这种想法太不方便。

解决上述问题的常用方法是使用后面将会提到的 current\_app 代理，它被限制在当前请求的应用引用。 既然无论如何在没有请求时创建一个这样的请求上下文是一个没有必要的昂贵操作，那么就引入了应用上下文。