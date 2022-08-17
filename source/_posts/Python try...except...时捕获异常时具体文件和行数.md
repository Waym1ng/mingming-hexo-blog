---
title: Python try...except...时捕获异常时具体文件和行数
date: 2021-07-08 18:49:59
tags: python debug
categories: python加油鸭
---

<!--more-->

```python
def try_exception_test():
    try:
        a = 0
        b = 1/a
        print(b)
    except Exception as e:
        print(e)
        # 发生异常所在的文件
        print(e.__traceback__.tb_frame.f_globals["__file__"])
        # 发生异常所在的行数
        print(e.__traceback__.tb_lineno)

if __name__ == '__main__':
    try_exception_test()
```

模拟一段会抛出异常的代码

执行结果：

> division by zero  
> C:/Users/admin01/Desktop/script/demo.py  
> 4

可以看到报错原因为**division by zero**

文件位置为**C:/Users/admin01/Desktop/script/demo.py**

行数为**第4行**

 要是系统中不能实时打印出来的话，可以考虑加上 **flush=True**

```python
print(str(e), flush=True)
```