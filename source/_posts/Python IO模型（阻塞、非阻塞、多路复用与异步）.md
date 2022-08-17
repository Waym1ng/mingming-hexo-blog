---
title: Python IO模型（阻塞、非阻塞、多路复用与异步）
date: 2020-04-29 10:03:47
tags: epoll python linux
categories: python加油鸭
---

<!--more-->

# IO模型

　　同步IO和异步IO，阻塞IO和非阻塞IO分别是什么，到底有什么区别？不同环境下给出的答案也是不一的。所以先限定一下上下文是非常有必要的。

_`本文讨论的背景是Linux环境下的network IO。`_

**在深入了解之前，我们应先了解几个概念：**

 用户空间和内核空间  
　 \- 进程切换  
　 \- 进程的阻塞  
　 \- 文件描述符  
　 \- 缓存 I/O

**用户空间与内核空间**

　　现在操作系统都是采用虚拟存储器，那么对32位操作系统而言，它的寻址空间（虚拟存储空间）为4G（2的32次方）。操作系统的核心是内核，独立于普通的应用程序，可以访问受保护的内存空间，也有访问底层硬件设备的所有权限。为了保证用户进程不能直接操作内核（kernel），保证内核的安全，操心系统将虚拟空间划分为两部分，一部分为内核空间，一部分为用户空间。针对linux操作系统而言，将最高的1G字节（从虚拟地址0xC0000000到0xFFFFFFFF），供内核使用，称为内核空间，而将较低的3G字节（从虚拟地址0x00000000到0xBFFFFFFF），供各个进程使用，称为用户空间。

 

**进程切换**

　　为了控制进程的执行，内核必须有能力挂起正在CPU上运行的进程，并恢复以前挂起的某个进程的执行。这种行为被称为进程切换。因此可以说，任何进程都是在操作系统内核的支持下运行的，是与内核紧密相关的。

从一个进程的运行转到另一个进程上运行，这个过程中经过下面这些变化：  
　　1. 保存处理机上下文，包括程序计数器和其他寄存器。  
　　2. 更新PCB信息。

　　3. 把进程的PCB移入相应的队列，如就绪、在某事件阻塞等队列。  
　　4. 选择另一个进程执行，并更新其PCB。  
　　5. 更新内存管理的数据结构。  
　　6. 恢复处理机上下文。

总而言之就是很耗资源，具体的可以参考这篇文章：[进程切换](http://guojing.me/linux-kernel-architecture/posts/process-switch/)

_注：进程控制块（Processing Control Block），是[操作系统](http://baike.baidu.com/view/880.htm)[核心](http://baike.baidu.com/view/22680.htm)中一种数据结构，主要表示[进程](http://baike.baidu.com/view/19746.htm)状态。其作用是使一个在[多道程序](http://baike.baidu.com/view/1189611.htm)环境下不能独立运行的程序（含数据），成为一个能独立运行的基本单位或与其它进程并发执行的进程。或者说，OS是根据PCB来对并发执行的进程进行控制和管理的。 PCB通常是系统内存占用区中的一个连续存区，它存放着[操作系统](http://baike.baidu.com/view/880.htm)用于描述进程情况及控制进程运行所需的全部信息 _

**进程的阻塞**

　　正在执行的进程，由于期待的某些事件未发生，如请求系统资源失败、等待某种操作的完成、新数据尚未到达或无新工作做等，则由系统自动执行阻塞原语\(Block\)，使自己由运行状态变为阻塞状态。可见，进程的阻塞是进程自身的一种主动行为，也因此只有处于运行态的进程（获得CPU），才可能将其转为阻塞状态。`当进程进入阻塞状态，是不占用CPU资源的`。

**文件描述符fd**

　　文件描述符（File descriptor）是计算机科学中的一个术语，是一个用于表述指向文件的引用的抽象化概念。

　　文件描述符在形式上是一个非负整数。实际上，它是一个索引值，指向内核为每一个进程所维护的该进程打开文件的记录表。当程序打开一个现有文件或者创建一个新文件时，内核向进程返回一个文件描述符。在程序设计中，一些涉及底层的程序编写往往会围绕着文件描述符展开。但是文件描述符这一概念往往只适用于UNIX、Linux这样的操作系统。

 

**缓存 I/O**

　　缓存 I/O 又被称作标准 I/O，大多数文件系统的默认 I/O 操作都是缓存 I/O。在 Linux 的缓存 I/O 机制中，操作系统会将 I/O 的数据缓存在文件系统的页缓存（ page cache ）中，也就是说，数据会先被拷贝到操作系统内核的缓冲区中，然后才会从操作系统内核的缓冲区拷贝到应用程序的地址空间。

缓存 I/O 的缺点：  
　　数据在传输过程中需要在应用程序地址空间和内核进行多次数据拷贝操作，这些数据拷贝操作所带来的 CPU 以及内存开销是非常大的。

##  IO模式

对于一次IO访问（以read举例），数据会先被拷贝到操作系统内核的缓冲区中，然后才会从操作系统内核的缓冲区拷贝到应用程序的地址空间。所以说，当一个read操作发生时，它会经历两个阶段：  
　　1. 等待数据准备 \(Waiting for the data to be ready\)  
　　2. 将数据从内核拷贝到进程中 \(Copying the data from the kernel to the process\)

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWFnZXMyMDE1LmNuYmxvZ3MuY29tL2Jsb2cvMTA1MDM5My8yMDE3MDQvMTA1MDM5My0yMDE3MDQxNTIxMTkxNzg0NS04NzI2MjYyNzEucG5n?x-oss-process=image/format,png)

_详细：http://blog.csdn.net/haiross/article/details/39078853_

 

正式因为这两个阶段，linux系统产生了下面五种网络模式的方案。  
　 \- 阻塞 I/O（blocking IO）  
　 \- 非阻塞 I/O（nonblocking IO）  
　 \- I/O 多路复用（ IO multiplexing）  
　 \- 信号驱动 I/O（ signal driven IO）  
　 \- 异步 I/O（asynchronous IO）

注：由于signal driven IO在实际中并不常用，所以我这只提及剩下的四种IO Model。

 

**阻塞 I/O（blocking IO）**

阻塞I/O模型是最广泛的模型，在linux中，默认情况下所有的socket都是blocking，一个典型的读操作流程大概是这样：

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWFnZXMyMDE1LmNuYmxvZ3MuY29tL2Jsb2cvMTA1MDM5My8yMDE3MDMvMTA1MDM5My0yMDE3MDMwODE5MDAxNTI5Ny0xNDIwNDg3Nzc5LnBuZw?x-oss-process=image/format,png)

　　当用户进程调用了recvfrom这个系统调用，kernel就开始了IO的第一个阶段：准备数据（对于网络IO来说，很多时候数据在一开始还没有到达。比如，还没有收到一个完整的UDP包。这个时候kernel就要等待足够的数据到来）。这个过程需要等待，也就是说数据被拷贝到操作系统内核的缓冲区中是需要一个过程的。而在用户进程这边，整个进程会被阻塞（当然，是进程自己选择的阻塞）。当kernel一直等到数据准备好了，它就会将数据从kernel中拷贝到用户内存，然后kernel返回结果，用户进程才解除block的状态，重新运行起来。

　**　所以，blocking IO的特点就是在IO执行的两个阶段都被block了。**

** 注意：进程处于阻塞模式时，让出CPU，进入休眠状态。**

 

**非阻塞 I/O（nonblocking IO）**

linux下，可以通过设置socket使其变为non-blocking。当对一个non-blocking socket执行读操作时，流程是这个样子：

 

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWFnZXMyMDE1LmNuYmxvZ3MuY29tL2Jsb2cvMTA1MDM5My8yMDE3MDMvMTA1MDM5My0yMDE3MDMwODE5MDIxNDg3NS0xNzA4MDgwMTk2LnBuZw?x-oss-process=image/format,png)

　　当用户进程发出read操作时，如果kernel中的数据还没有准备好，那么它并不会block用户进程，而是立刻返回一个error。从用户进程角度讲 ，它发起一个read操作后，并不需要等待，而是马上就得到了一个结果。用户进程判断结果是一个error时，它就知道数据还没有准备好，于是它可以再次发送read操作。一旦kernel中的数据准备好了，并且又再次收到了用户进程的system call，那么它马上就将数据拷贝到了用户内存，然后返回。

　　**所以，nonblocking IO的特点是用户进程需要不断的主动询问kernel数据好了没有。**

阻塞IO与非阻塞IO的性能区别

　　在阻塞模式下，若从网络流中读取不到指定大小的数据量，阻塞IO就在那里阻塞着。比如，已知后面会有10个字节的数据发过来，但是我现在只收到8个字节，那么当前线程就在那傻傻地等到下一个字节的到来，对，就在那等着，啥事也不做，直到把这10个字节读取完，这才将阻塞放开通行。

　　在非阻塞模式下，若从网络流中读取不到指定大小的数据量，非阻塞IO就立即通行。比如，已知后面会有10个字节的数据发过来，但是我现在只收到8个字节，那么当前线程就读取这8个字节的数据，读完后就立即返回，等另外两个字节再来的时候再去读取。

　　从上面可以看出，阻塞IO在性能方面是很低下的，如果要使用阻塞IO完成一个Web服务器的话，那么对于每一个请求都必须启用一个线程进行处理。而使用非阻塞IO的话，一到两个线程基本上就够了，因为线程不会产生阻塞，好比一下接收A请求的数据，另一下接收B请求的数据，等等，就是不停地东奔西跑，直接到把数据接收完了。

**阻塞IO和非阻塞IO的区别就在于：应用程序的调用是否立即返回！**

****注意；非阻塞模式的使用并不普遍，因为非阻塞模式会浪费大量的CPU资源。****

 

**I/O 多路复用（ IO multiplexing）**

　　多路复用的本质是同步非阻塞I/O，多路复用的优势并不是单个连接处理的更快，而是在于能处理更多的连接。

　　I/O编程过程中，需要同时处理多个客户端接入请求时，可以利用多线程或者I/O多路复用技术进行处理。 I/O多路复用技术通过把多个I/O的阻塞复用到同一个select阻塞上，一个进程监视多个描述符，一旦某个描述符就位， 能够通知程序进行读写操作。因为多路复用本质上是同步I/O，都需要应用程序在读写事件就绪后自己负责读写。 与传统的多线程/多进程模型比，I/O多路复用的最大优势是系统开销小，系统不需要创建新的额外进程或者线程。

- 应用场景
  - 服务器需要同时处理多个处于监听状态或者多个连接状态的套接字
  - 需要同时处理多种网络协议的套接字
  - 一个服务器处理多个服务或协议

目前支持多路复用的系统调用有 `select` , `poll` , `epoll` 。

　　IO multiplexing就是我们说的select，poll，epoll，有些地方也称这种IO方式为event driven IO。select/epoll的好处就在于单个process就可以同时处理多个网络连接的IO。它的基本原理就是select，poll，epoll这个function会不断的轮询所负责的所有socket，当某个socket有数据到达了，就通知用户进程。

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWFnZXMyMDE1LmNuYmxvZ3MuY29tL2Jsb2cvMTA1MDM5My8yMDE3MDMvMTA1MDM5My0yMDE3MDMwODE5MDMzMTQ2OS0xMzk0OTM5NzY2LnBuZw?x-oss-process=image/format,png)

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWFnZXMyMDE1LmNuYmxvZ3MuY29tL2Jsb2cvMTA1MDM5My8yMDE3MDMvMTA1MDM5My0yMDE3MDMwODE5MDMzMTQ2OS0xMzk0OTM5NzY2LnBuZw?x-oss-process=image/format,png)

**`　　当用户进程调用了select，那么整个进程会被block`**，而同时，kernel会“监视”所有select负责的socket，当任何一个socket中的数据准备好了，select就会返回。这个时候用户进程再调用read操作，将数据从kernel拷贝到用户进程。

　　**所以，I/O 多路复用的特点是通过一种机制一个进程能同时等待多个文件描述符，而这些文件描述符（套接字描述符）其中的任意一个进入读就绪状态，select\(\)函数就可以返回。**

　　这个图和blocking IO的图其实并没有太大的不同，事实上，还更差一些。因为这里需要使用两个system call \(select 和 recvfrom\)，而blocking IO只调用了一个system call \(recvfrom\)。但是，用select的优势在于它可以同时处理多个connection。

　　所以，如果处理的连接数不是很高的话，使用select/epoll的web server不一定比使用multi-threading + blocking IO的web server性能更好，可能延迟还更大。select/epoll的优势并不是对于单个连接能处理得更快，而是在于能处理更多的连接。）

　　在IO multiplexing Model中，实际中，对于每一个socket，一般都设置成为non-blocking，但是，如上图所示，整个用户的process其实是一直被block的。只不过process是被select这个函数block，而不是被socket IO给block。

 

**异步 I/O（asynchronous IO）**

　　相比于IO多路复用模型，异步IO并不十分常用，不少高性能并发服务程序使用IO多路复用模型+多线程任务处理的[架构](http://lib.csdn.net/base/architecture)基本可以满足需求

linux下的asynchronous IO其实用得很少。先看一下它的流程：

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWFnZXMyMDE1LmNuYmxvZ3MuY29tL2Jsb2cvMTA1MDM5My8yMDE3MDMvMTA1MDM5My0yMDE3MDMwODE5MDUxOTczNC0yNjMyNDAxNDkucG5n?x-oss-process=image/format,png)

　　用户进程发起read操作之后，立刻就可以开始去做其它的事。而另一方面，从kernel的角度，当它受到一个asynchronous read之后，首先它会立刻返回，所以不会对用户进程产生任何block。然后，kernel会等待数据准备完成，然后将数据拷贝到用户内存，当这一切都完成之后，kernel会给用户进程发送一个signal，告诉它read操作完成了。

##  总结：

_浅析 I/O 模型及其设计模式http://blog.jobbole.com/104638/_

blocking和non-blocking的区别

调用blocking IO会一直block住对应的进程直到操作完成，而non-blocking IO在kernel还准备数据的情况下会立刻返回。

synchronous IO和asynchronous IO的区别

**同步IO和异步IO的区别就在于：数据访问的时候进程是否阻塞！**

在说明synchronous IO和asynchronous IO的区别之前，需要先给出两者的定义。POSIX的定义是这样子的：  
　 \- A synchronous I/O operation causes the requesting process to be blocked until that I/O operation completes;  
　 \- An asynchronous I/O operation does not cause the requesting process to be blocked;

　　两者的区别就在于synchronous IO做”IO operation”的时候会将process阻塞。按照这个定义，之前所述的blocking IO，non-blocking IO，IO multiplexing都属于synchronous IO。

　　有人会说，non-blocking IO并没有被block啊。这里有个非常“狡猾”的地方，定义中所指的”IO operation”是指真实的IO操作，就是例子中的recvfrom这个system call。non-blocking IO在执行recvfrom这个system call的时候，如果kernel的数据没有准备好，这时候不会block进程。但是，当kernel中数据准备好的时候，recvfrom会将数据从kernel拷贝到用户内存中，这个时候进程是被block了，在这段时间内，进程是被block的。

　　而asynchronous IO则不一样，当进程发起IO 操作之后，就直接返回再也不理睬了，直到kernel发送一个信号，告诉进程说IO完成。在这整个过程中，进程完全没有被block。

 

各个IO Model的比较如图所示：

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWFnZXMyMDE1LmNuYmxvZ3MuY29tL2Jsb2cvMTA1MDM5My8yMDE3MDMvMTA1MDM5My0yMDE3MDMwODE5MDcyMjg5MS0xMjc5Nzk2NzE5LnBuZw?x-oss-process=image/format,png)

　　通过上面的图片，可以发现non-blocking IO和asynchronous IO的区别还是很明显的。在non-blocking IO中，虽然进程大部分时间都不会被block，但是它仍然要求进程去主动的check，并且当数据准备完成以后，也需要进程主动的再次调用recvfrom来将数据拷贝到用户内存。而asynchronous IO则完全不同。它就像是用户进程将整个IO操作交给了他人（kernel）完成，然后他人做完后发信号通知。在此期间，用户进程不需要去检查IO操作的状态，也不需要主动的去拷贝数据。

## IO多路复用之select、poll、epoll

　　select，poll，epoll都是IO多路复用的机制。I/O多路复用就是通过一种机制，一个进程可以监视多个描述符，一旦某个描述符就绪（一般是读就绪或者写就绪），能够通知程序进行相应的读写操作。但select，poll，epoll本质上都是同步I/O，因为他们都需要在读写事件就绪后自己负责进行读写，也就是说这个读写过程是阻塞的，而异步I/O则无需自己负责进行读写，异步I/O的实现会负责把数据从内核拷贝到用户空间。

**select**

```html
select(rlist, wlist, xlist, timeout=None)
```

　　select 函数监视的文件描述符分3类，分别是writefds、readfds、和exceptfds。调用后select函数会阻塞，直到有描述副就绪（有数据 可读、可写、或者有except），或者超时（timeout指定等待时间，如果立即返回设为null即可），函数返回。当select函数返回后，可以 通过遍历fdset，来找到就绪的描述符。

　　select目前几乎在所有的平台上支持，其良好跨平台支持也是它的一个优点。select的一 个缺点在于单个进程能够监视的文件描述符的数量存在最大限制，在Linux上一般为1024，可以通过修改宏定义甚至重新编译内核的方式提升这一限制，但 是这样也会造成效率的降低。

```html
　　使用select库的一般步骤：创建所关注事件的描述集合。对于一个描述符，可以关注其上面的读事件、写事件以及异常发生事件，所以要创建三类事件描述符集合，分别用来收集读事件的描述符、写事件的描述符和异常事件的描述符。
　　其次，调用底层提供的select（）函数，等待事件的发生。select的阻塞与是否设置非阻塞的IO是没有关系的。
　　然后，轮询所有事件描述符集合中的每一个事件描述符，检查是否有响应的时间发生，如果有，则进行处理。
　　nginx服务器在编译过程中如果没有为其指定其他高性能事件驱动模型库，它将自动编译该库。
　　可以使用--with-select_module和--without-select_module两个参数，强制nginx是否编译该库。

实例：
```

```html
利用select通过单进程实现同时处理多个非阻塞的socket连接:
```

```python
# _*_coding:utf-8_*_

import select
import socket
import sys
import queue

# Create a TCP/IP socket
server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server.setblocking(False)

# Bind the socket to the port
server_address = ('localhost', 6666)
print(sys.stderr, 'starting up on %s port %s' % server_address)
server.bind(server_address)

# Listen for incoming connections
server.listen(5)

# Sockets from which we expect to read
inputs = [server]

# Sockets to which we expect to write
outputs = []

message_queues = {}
while inputs:
    # Wait for at least one of the sockets to be ready for processing
    print('\nwaiting for the next event')
    readable, writable, exceptional = select.select(inputs, outputs, inputs)
    # Handle inputs
    for s in readable:
        if s is server:
            # A "readable" server socket is ready to accept a connection
            connection, client_address = s.accept()
            print('new connection from', client_address)
            connection.setblocking(False)
            inputs.append(connection)

            # Give the connection a queue for data we want to send
            message_queues[connection] = queue.Queue()
        else:
            data = s.recv(1024)
            if data:
                # A readable client socket has data
                print(sys.stderr, 'received "%s" from %s' % (data, s.getpeername()))
                message_queues[s].put(data)
                # Add output channel for response
                if s not in outputs:
                    outputs.append(s)
            else:
                # Interpret empty result as closed connection
                print('closing', client_address, 'after reading no data')
                # Stop listening for input on the connection
                if s in outputs:
                    outputs.remove(s)  # 既然客户端都断开了，我就不用再给它返回数据了，所以这时候如果这个客户端的连接对象还在outputs列表中，就把它删掉
                inputs.remove(s)  # inputs中也删除掉
                s.close()  # 把这个连接关闭掉

                # Remove message queue
                del message_queues[s]
    # Handle outputs
    for s in writable:
        try:
            next_msg = message_queues[s].get_nowait()
        except queue.Empty:
            # No messages waiting so stop checking for writability.
            print('output queue for', s.getpeername(), 'is empty')
            outputs.remove(s)
        else:
            print('sending "%s" to %s' % (next_msg, s.getpeername()))
            s.send(next_msg)
    # Handle "exceptional conditions"
    for s in exceptional:
        print('handling exceptional condition for', s.getpeername())
        # Stop listening for input on the connection
        inputs.remove(s)
        if s in outputs:
            outputs.remove(s)
        s.close()

        # Remove message queue
        del message_queues[s]
```

 

```python
#!/usr/bin/env/ python
# -*-coding:utf-8 -*-
import socket,select,queue
import sys

server=socket.socket()
server.bind(('localhost',8080))

server.listen(1000)

server.setblocking(False)#设置为非阻塞模式，socket必须在非阻塞情况下才能实现IO多路复用。

#需要监听的可读对象
inputs=[server,]#存放selcct要监测的链接,一开始监测自己，有活动就表示有链接进来
outputs=[]#这里存放的是内核返回的活跃的客户端连接，就是服务器需给send data的客户端连接

message_queues={}
while inputs:
    # Wait for at least one of the sockets to be ready for processing
    print('\nwaiting for the next event')
    readable,writeable,exceptionable=select.select(inputs,outputs,inputs) #此处会被select模块阻塞，只有当监听的三个参数发生变化时，select才会返,

    # print(readable,writeable,exceptionable)
    for r in readable:
        if r is server:#代表来了一个新连接
            conn,addr=server.accept()
            print("来了一个新连接",addr)
            inputs.append(conn)#因为这个新建立的连接还没发数据过来，现在就接收的话会报错，所以要想客户端发数据过来时server端能知道，就要select监测这个连接。
            message_queues[conn]=queue.Queue()#接收到客户端的数据后,不立刻返回 ,暂存在队列里,以后发送
        else:
            data = r.recv(1024) # 注意这里是r，而不是conn，多个连接的情况
            if data:
                # A readable client socket has data
                print(sys.stderr, 'received "%s" from %s' % (data, r.getpeername()))
                # r.send(data) # 先不发，放到一个对列里，之后再发
                message_queues[r].put(data)# 往里面放数据
                # Add output channel for response
                if r not in outputs:
                    outputs.append(r)  #放入返回的连接队列里
            else:
                # Interpret empty result as closed connection
                print('closing', addr, 'after reading no data')
                # Stop listening for input on the connection
                if r in outputs:
                    outputs.remove(r)  # 既然客户端都断开了，我就不用再给它返回数据了，所以这时候如果这个客户端的连接对象还在outputs列表中，就把它删掉
                inputs.remove(r)  # inputs中也删除掉
                r.close()  # 把这个连接关闭掉

                # Remove message queue
                del message_queues[r]

    for w in writeable:  # 要返回给客户端的连接列表
        data_to_client = message_queues[w].get()  # 在字典里取数据
        w.send(data_to_client)  # 返回给客户端
        outputs.remove(w)  # 删除这个数据，确保下次循环的时候不返回这个已经处理完的连接了。

    for e in exceptionable:  # 如果连接断开，删除连接相关数据
        if e in outputs:
            outputs.remove(e)
        inputs.remove(e)
        del message_queues[e]
```

**poll**

```html
　　poll库，作为linux平台上的基本事件驱动模型，Windows平台不支持poll库。
```

```html
　　使用poll库的一般过程是：与select的基本工作方式是相同的，都是先创建一个关注事件的描述符集合，再去等待这些事件的发生，然后在轮询描述符集合，检查有没有事件发生，如果有，就进行处理。
　　与select的主要区别是select需要为读事件、写事件、异常事件分别创建一个描述符的集合，因此在轮询的时候，需要分别轮询这三个集合。而poll库只需创建一个集合，在每个描述符对应的结构上分别设置读事件，写事件和异常事件，最后轮询的时候可以同时检查这三种事件是否发生。是select库优化的实现。
    nginx服务器在编译过程中如果没有为其指定其他高性能事件驱动模型库，它将自动编译该库。可以使用--with-poll_module和--without-poll_module两个参数，强制nginx是否编译该库。
```

```html
int poll (struct pollfd *fds, unsigned int nfds, int timeout);
```

　不同与select使用三个位图来表示三个fdset的方式，poll使用一个 pollfd的指针实现。　

```html
struct pollfd {
    int fd; /* file descriptor */
    short events; /* requested events to watch */
    short revents; /* returned events witnessed */
};
```

　　pollfd结构包含了要监视的event和发生的event，不再使用select“参数-值”传递的方式。同时，pollfd并没有最大数量限制（但是数量过大后性能也是会下降）。 和select函数一样，poll返回后，需要轮询pollfd来获取就绪的描述符。

　　从上面看，select和poll都需要在返回后，`通过遍历文件描述符来获取已经就绪的socket`。事实上，同时连接的大量客户端在一时刻可能只有很少的处于就绪状态，因此随着监视的描述符数量的增长，其效率也会线性下降。

**epoll**

```html
　　epoll是在2.6内核中提出的，是之前的select和poll的增强版本。相对于select和poll来说，epoll更加灵活，没有描述符限制。epoll使用一个文件描述符管理多个描述符，将用户关系的文件描述符的事件存放到内核的一个事件表中，这样在用户空间和内核空间的copy只需一次。　  
　　epoll库是Nginx服务器支持的高性能事件之一，它是公认的非常优秀的时间驱动模型，和poll和select有很大的不同，属于poll库的一个变种，他们的处理方式都是创建一个待处理事件列表，然后把这个事件列表发送给内核，返回的时候，再去轮询检查这个列表，以判断事件是否发生。如果这样的描述符在比较多的应用中，效率就显得低下了，epoll是描述符列表的管理交给内核负责，一旦某种事件发生，内核会把发生事件的描述符列表通知给进程，这样就避免了轮询整个描述符列表，epoll库得到事件列表，就开始进行事件处理了。
　 注意：select和epoll最大的区别就是：select只是告诉你一定数目的流有事件了，至于哪个流有事件，还得你一个一个地去轮询，而 epoll会把发生的事件告诉你，通过发生的事件，就自然而然定位到哪个流了
```

一 epoll操作过程

epoll操作过程需要三个接口，分别如下：

```html
int epoll_create(int size)；//创建一个epoll的句柄，size用来告诉内核这个监听的数目一共有多大
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event)；
int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);
```

1\. int epoll\_create\(int size\);  
　　创建一个epoll的句柄，size用来告诉内核这个监听的数目一共有多大，这个参数不同于select\(\)中的第一个参数，给出最大监听的fd+1的值，`参数size并不是限制了epoll所能监听的描述符最大个数，只是对内核初始分配内部数据结构的一个建议`。  
当创建好epoll句柄后，它就会占用一个fd值，在linux下如果查看/proc/进程id/fd/，是能够看到这个fd的，所以在使用完epoll后，必须调用close\(\)关闭，否则可能导致fd被耗尽。

2\. int epoll\_ctl\(int epfd, int op, int fd, struct epoll\_event \*event\)；  
　　函数是对指定描述符fd执行op操作。  
　 \- epfd：是epoll\_create\(\)的返回值。  
　 \- op：表示op操作，用三个宏来表示：添加EPOLL\_CTL\_ADD，删除EPOLL\_CTL\_DEL，修改EPOLL\_CTL\_MOD。分别添加、删除和修改对fd的监听事件。  
　 \- fd：是需要监听的fd（文件描述符）  
　 \- epoll\_event：是告诉内核需要监听什么事

3\. int epoll\_wait\(int epfd, struct epoll\_event \* events, int maxevents, int timeout\);  
　　等待epfd上的io事件，最多返回maxevents个事件。  
参数events用来从内核得到事件的集合，maxevents告之内核这个events有多大，这个maxevents的值不能大于创建epoll\_create\(\)时的size，参数timeout是超时时间（毫秒，0会立即返回，-1将不确定，也有说法说是永久阻塞）。该函数返回需要处理的事件数目，如返回0表示已超时。

```python
import socket, logging
import select, errno

logger = logging.getLogger("network-server")

def InitLog():
    logger.setLevel(logging.DEBUG)

    fh = logging.FileHandler("network-server.log")
    fh.setLevel(logging.DEBUG)
    ch = logging.StreamHandler()
    ch.setLevel(logging.ERROR)

    formatter = logging.Formatter("%(asctime)s - %(name)s - %(levelname)s - %(message)s")
    ch.setFormatter(formatter)
    fh.setFormatter(formatter)

    logger.addHandler(fh)
    logger.addHandler(ch)


if __name__ == "__main__":
    InitLog()

    try:
        # 创建 TCP socket 作为监听 socket
        listen_fd = socket.socket(socket.AF_INET, socket.SOCK_STREAM, 0)
    except socket.error as  msg:
        logger.error("create socket failed")

    try:
        # 设置 SO_REUSEADDR 选项
        listen_fd.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    except socket.error as  msg:
        logger.error("setsocketopt SO_REUSEADDR failed")

    try:
        # 进行 bind -- 此处未指定 ip 地址，即 bind 了全部网卡 ip 上
        listen_fd.bind(('', 2003))
    except socket.error as  msg:
        logger.error("bind failed")

    try:
        # 设置 listen 的 backlog 数
        listen_fd.listen(10)
    except socket.error as  msg:
        logger.error(msg)

    try:
        # 创建 epoll 句柄
        epoll_fd = select.epoll()
        # 向 epoll 句柄中注册 监听 socket 的 可读 事件
        epoll_fd.register(listen_fd.fileno(), select.EPOLLIN)
    except select.error as  msg:
        logger.error(msg)

    connections = {}
    addresses = {}
    datalist = {}
    while True:
        # epoll 进行 fd 扫描的地方 -- 未指定超时时间则为阻塞等待
        epoll_list = epoll_fd.poll()

        for fd, events in epoll_list:
            # 若为监听 fd 被激活
            if fd == listen_fd.fileno():
                # 进行 accept -- 获得连接上来 client 的 ip 和 port，以及 socket 句柄
                conn, addr = listen_fd.accept()
                logger.debug("accept connection from %s, %d, fd = %d" % (addr[0], addr[1], conn.fileno()))
                # 将连接 socket 设置为 非阻塞
                conn.setblocking(0)
                # 向 epoll 句柄中注册 连接 socket 的 可读 事件
                epoll_fd.register(conn.fileno(), select.EPOLLIN | select.EPOLLET)
                # 将 conn 和 addr 信息分别保存起来
                connections[conn.fileno()] = conn
                addresses[conn.fileno()] = addr
            elif select.EPOLLIN & events:
                # 有 可读 事件激活
                datas = ''
                while True:
                    try:
                        # 从激活 fd 上 recv 10 字节数据
                        data = connections[fd].recv(10)
                        # 若当前没有接收到数据，并且之前的累计数据也没有
                        if not data and not datas:
                            # 从 epoll 句柄中移除该 连接 fd
                            epoll_fd.unregister(fd)
                            # server 侧主动关闭该 连接 fd
                            connections[fd].close()
                            logger.debug("%s, %d closed" % (addresses[fd][0], addresses[fd][1]))
                            break
                        else:
                            # 将接收到的数据拼接保存在 datas 中
                            datas += data
                    except socket.error as  msg:
                        # 在 非阻塞 socket 上进行 recv 需要处理 读穿 的情况
                        # 这里实际上是利用 读穿 出 异常 的方式跳到这里进行后续处理
                        if msg.errno == errno.EAGAIN:
                            logger.debug("%s receive %s" % (fd, datas))
                            # 将已接收数据保存起来
                            datalist[fd] = datas
                            # 更新 epoll 句柄中连接d 注册事件为 可写
                            epoll_fd.modify(fd, select.EPOLLET | select.EPOLLOUT)
                            break
                        else:
                            # 出错处理
                            epoll_fd.unregister(fd)
                            connections[fd].close()
                            logger.error(msg)
                            break
            elif select.EPOLLHUP & events:
                # 有 HUP 事件激活
                epoll_fd.unregister(fd)
                connections[fd].close()
                logger.debug("%s, %d closed" % (addresses[fd][0], addresses[fd][1]))
            elif select.EPOLLOUT & events:
                # 有 可写 事件激活
                sendLen = 0
                # 通过 while 循环确保将 buf 中的数据全部发送出去
                while True:
                    # 将之前收到的数据发回 client -- 通过 sendLen 来控制发送位置
                    sendLen += connections[fd].send(datalist[fd][sendLen:])
                    # 在全部发送完毕后退出 while 循环
                    if sendLen == len(datalist[fd]):
                        break
                # 更新 epoll 句柄中连接 fd 注册事件为 可读
                epoll_fd.modify(fd, select.EPOLLIN | select.EPOLLET)
            else:
                # 其他 epoll 事件不进行处理
                continue
```

---

### select \& poll \& epoll比较

- 每次调用 `select` 都需要把所有要监听的文件描述符拷贝到内核空间一次，fd很大时开销会很大。 `epoll` 会在epoll\_ctl\(\)中注册，只需要将所有的fd拷贝到内核事件表一次，不用再每次epoll\_wait\(\)时重复拷贝
- 每次 `select` 需要在内核中遍历所有监听的fd，直到设备就绪； `epoll` 通过 `epoll_ctl` 注册回调函数，也需要不断调用 `epoll_wait` 轮询就绪链表，当fd或者事件就绪时，会调用回调函数，将就绪结果加入到就绪链表。
- `select` 能监听的文件描述符数量有限，默认是1024； `epoll` 能支持的fd数量是最大可以打开文件的数目，具体数目可以在/proc/sys/fs/file-max查看
- `select` , `poll` 在函数返回后需要查看所有监听的fd，看哪些就绪，而epoll只返回就绪的描述符，所以应用程序只需要就绪fd的命中率是百分百。

表面上看epoll的性能最好，但是在连接数少并且链接都十分活跃的情况下，select和poll的性能可能比epoll好，毕竟epoll的通知机制需要很多函数回调。

select效率低是一位每次都需要轮询，但效率低也是相对的，也可通过良好的设计改善

参考：<https://www.cnblogs.com/freely/p/6522432.html>