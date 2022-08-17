---
title: websocket 与 socket 非阻塞通信
date: 2020-06-12 11:52:05
tags: socket python epoll
categories: python加油鸭
---

<!--more-->

 记录一下

```python
import select
import socket
import threading

from flask import Flask
from flask_sockets import Sockets
from gevent import pywsgi
from geventwebsocket.handler import WebSocketHandler


app = Flask(__name__)
sockets = Sockets(app)


class Config(object):
    SOCKET_HOST = '127.0.0.1'
    SOCKET_PORT = 2020


def read_thread_method(sock,socket_lock,ws):
    while True:
        if not sock:  # 如果socket关闭，退出
            break
        # 使用select监听客户端（这里客户端需要不停接收服务端的数据，所以监听客户端）
        # 第一个参数是要监听读事件列表，因为是客户端，我们只监听创建的一个socket就ok
        # 第二个参数是要监听写事件列表，
        # 第三个参数是要监听异常事件列表，
        # 最后一个参数是监听超时时间，默认永不超时。如果设置了超时时间，过了超时时间线程就不会阻塞在select方法上，会继续向下执行
        # 返回参数 分别对应监听到的读事件列表，写事件列表，异常事件列表
        rs, _, _ = select.select([sock], [], [], 10)
        for r in rs:  # 我们这里只监听读事件，所以只管读的返回句柄数组
            socket_lock.acquire()  # 在读取之前先加锁，锁定socket对象（sock是主线程和子线程的共享资源，锁定了sock就能保证子线程在使用sock时，主线程无法对sock进行操作）

            if not sock:  # 这里需要判断下，因为有可能在select后到加锁之间socket被关闭了
                socket_lock.release()
                break
            try:
                data = r.recv(1024)  # 读数据，按自己的方式读
            except Exception as e:
                data = None

            socket_lock.release()  # 读取完成之后解锁，释放资源

            if not data:
                print('server close')
            else:
                print(data)
                ws.send(data.decode())


def start_client(ws):

    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock.connect((Config.SOCKET_HOST, Config.SOCKET_PORT))
    # 创建线程锁，防止主线程socket被close了，子线程还在recv而引发的异常
    socket_lock = threading.Lock()

    # 创建一个线程去读取数据
    read_thread = threading.Thread(target=read_thread_method, args=(sock,socket_lock,ws))
    read_thread.setDaemon(True)
    read_thread.start()


    return sock,socket_lock,read_thread

    # # 测试不断写数据
    # for x in range(5):
    #     print(x)
    #     sock.send(str(x).encode())
    #     sleep(1)  # 交出CPU时间，否则其他线程只能看着

    # 清理socket，同样道理，这里需要锁定和解锁
    # socket_lock.acquire()
    # sock.close()
    # socket_lock.release()


# socket 路由
@sockets.route('/')
def echo_socket(ws):

    print('当前/:', ws)
    sock,socket_lock,read_thread = start_client(ws)
    print("连接了poll")
    while not ws.closed:


        message = ws.receive()
        print("\n message:",message,type(message))
        print("ws.closed:", ws.closed)
        # print("线程id={},\n线程名称={},\n正在执行的线程列表:{},\n正在执行的线程数量={},\n当前激活线程={}".format(
        #     read_thread.ident, read_thread.getName(), threading.enumerate(), threading.active_count(),
        #     read_thread.isAlive)
        # )
        if message:
            ws.send("callback")
            sock.send(message.encode())
        if ws.closed:
            print("关闭了 sock "
            socket_lock.acquire()
            sock.close()
            socket_lock.release()




if __name__ == "__main__":

    try:
        server = pywsgi.WSGIServer(('0.0.0.0', 2021), application=app, handler_class=WebSocketHandler)
        print("websocket server start ... ")
        server.serve_forever()
    except Exception as e:
        print('error:',e)



```