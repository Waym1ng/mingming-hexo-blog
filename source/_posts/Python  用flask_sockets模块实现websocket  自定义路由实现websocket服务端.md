---
title: Python  用flask_sockets模块实现websocket  自定义路由实现websocket服务端
date: 2020-03-02 16:49:41
tags: python websocket socket flask
categories: python加油鸭
---

<!--more-->

## flask\_sockets模块的使用

```
	可以自定义路由
	socket路由的访问地址为ws://localhost:端口/自定义的地址
	如ws://localhost:5000/echo
```

### 当创建一个ws的时候，接收到了数据，但是却想返回\(send\)到另外一个ws，这怎么办呢

_后端代码_

```python
import random
import threading

import time

import datetime
from flask import Flask, render_template
from flask_sockets import Sockets
from gevent import pywsgi
from geventwebsocket.handler import WebSocketHandler



app = Flask(__name__)
sockets = Sockets(app)

# 多线程装饰器 没有用到 
def run_async(f):
    def wrapper(*args, **kwargs):
        thr = threading.Thread(target = f, args = args, kwargs = kwargs)
        thr.start()
        thr.setName("func-{}".format(f.__name__))
        # thr.join()
        print("线程id={},\n线程名称={},\n正在执行的线程列表:{},\n正在执行的线程数量={},\n当前激活线程={}".format(
            thr.ident,thr.getName(),threading.enumerate(),threading.active_count(),thr.isAlive)
        )
    return wrapper

@app.route('/test')
def test():
    return render_template('test2.html')


WS = {}

# socket 路由，访问url是： ws://localhost:7070/echo
@sockets.route('/echo')
def echo_socket(ws):
    global WS
    print('/echo:', ws)
    while not ws.closed:
        try:
            WS['echo'] = ws
            print('WS', WS)
            # args = ws.origin
            str_time = datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')
            # msg = 'pid:%s,fileno:%s,str_time:%s' % (pid, fileno, str_time)
            message = ws.receive()
            # print('args:',args)
            # print('message:', str_time, message)
            ws.send("time:%s,msg: %s"%(str_time,str(message)))

        except Exception as e:
            print('error:',e)


@sockets.route('/poll')
def test2_socket(ws):
    try:
        print('/poll:', ws)
        # while True:
        n = ws.receive()
        print('n-----', n)
        print('WS-----', WS)

        WS['echo'].send("receive for poll: " + n)

    except Exception as e:
        print('error:',e)

@sockets.route('/test3')
def test3_socket(ws):
    try:
        print('/test3:', ws)
        while True:
            time.sleep(3)
            t = random.randint(1,100)
            ws.send("while True number: " + str(t))
    except Exception as e:
        print('error:',e)

if __name__ == "__main__":
    try:
        server = pywsgi.WSGIServer(('0.0.0.0', 7070), application=app, handler_class=WebSocketHandler)
        print("web server start ... ")
        server.serve_forever()
    except Exception as e:
        print('error:',e)
```

_前端代码_

```python
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="utf-8">
        <title>WebSocket</title>
        <script type="text/javascript" src="//cdn.bootcss.com/jquery/3.1.1/jquery.min.js"></script>
        <script type="text/javascript">

            var socket;

            function connect() {
                var host = "ws://" + $("#serverIP").val() + ":" + $("#serverPort").val() + "/" + $("#serverRoute").val();
                socket = new WebSocket(host);
                try {

                    socket.onopen = function (msg) {
                        console.log(msg);
                        var m = $("#sendText").val();
                        socket.send(m);
                        // alert("连接成功！");
                    };

                    socket.onmessage = function (msg) {
                        if (typeof msg.data == "string") {
                            $("#t").html(msg.data)
                        }
                        else {
                            alert("非文本消息");
                        }
                    };

                    socket.onclose = function (msg) {
                        alert("socket closed!")
                    };
                }
                catch (ex) {
                    log(ex);
                }
            }

            function sendMsg() {
                var msg = $("#sendText").val();
                socket.send(msg);
            }

            function butClose() {
                socket.close();
                socket = null;
            }

            window.onbeforeunload = function () {
                try {
                    socket.close();
                    socket = null;
                }
                catch (ex) {
                }
            };

        </script>
    </head>
    <body>
        <h3>WebSocketTest</h3>
        <div id="login">
            <div>
                <input id="serverIP" type="text" placeholder="服务器IP" value="127.0.0.1" autofocus="autofocus" />
                <input id="serverPort" type="text" placeholder="服务器端口" value="7070" />
                <input id="serverRoute" type="text" placeholder="服务器路由" value="echo" />
                <input id="btnConnect" type="button" onclick="connect()" value="连接" />
            </div>
            <div>
                <input id="btnClose" type="button" onclick="butClose()" value="关闭连接" />
            </div>
            <div>
                <input id="sendText" type="text" placeholder="发送文本" value="24" />
                <input id="btnSend" onclick="sendMsg()" type="button" value="发送" />
            </div>
        </div>
        <h2 id="t"></h2>
    </body>
</html>
```