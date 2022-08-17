---
title: selector Python selectors模块 I/O多路复用
date: 2020-04-29 10:18:04
tags: python socket epoll
categories: python加油鸭
---

<!--more-->

### **selectors模块**

此模块允许高级和高效的I / O多路复用，构建在select模块原语上。鼓励用户使用此模块，除非他们需要精确控制所使用的操作系统级原语。（ 默认使用epoll，但由于Windows不支持epoll，如果在你的Windows上找不到epoll的话，就会用select）  它定义了一个抽象基类，有几个具体的实现工具\(KqueueSelector, EpollSelector...\),可以用于等待多个文件对象的I / O就绪通知。在下文中，file object” 指的是任何fileno\(\) method，或一个原始文件的描述符。请参阅文件对象。BaseSelectorKqueueSelectorEpollSelectorfileno\(\)
 　**DefaultSelector** 是别名到当前平台上可用的最有效的实现：这应该是大多数用户的默认选择
   注意支持的文件对象类型取决于平台：在Windows上，支持套接字，但不支持管道，而在Unix上，支持两者（也可以支持其他类型，例如fifos或特殊文件设备）

### 实例- server端

```python
import selectors
from socket import *

def accept(sk,mask):
    conn,addr=sk.accept()
    sel.register(conn,selectors.EVENT_READ,read)

def read(conn,mask):
    try:
        data=conn.recv(1024)
        if not data:
            print('closing',conn)
            sel.unregister(conn)
            conn.close()
            return
        conn.send(data.upper()+b'_SB')
    except Exception:
        print('closing', conn)
        sel.unregister(conn)
        conn.close()

sk=socket()
sk.setsockopt(SOL_SOCKET,SO_REUSEADDR,1)
sk.bind(('127.0.0.1',8008))
sk.listen(5)
sk.setblocking(False) #设置socket的接口为非阻塞
sel=selectors.DefaultSelector()   # 选择一个适合我的IO多路复用的机制
sel.register(sk,selectors.EVENT_READ,accept)
#相当于网select的读列表里append了一个sk对象,并且绑定了一个回调函数accept
# 说白了就是 如果有人请求连接sk,就调用accept方法

while True:
    events=sel.select() #检测所有的sk,conn，是否有完成wait data阶段
    for sel_obj,mask in events:  # [conn]
        callback=sel_obj.data #callback=read
        callback(sel_obj.fileobj,mask) #read(conn,1)
```

###  实例-client端

```python
import time
import socket
import threading
def func():
    sk = socket.socket()
    sk.connect(('127.0.0.1',8008))
    sk.send(b'hello')
    time.sleep(3)
    print(sk.recv(1024))
    sk.close()

for i in range(20):
    threading.Thread(target=func).start()
```