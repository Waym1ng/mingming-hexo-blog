---
title: 进程间通信-Queue
date: 2020-06-19 17:20:22
tags: python 队列 多进程
categories: python加油鸭
---

<!--more-->

##  1. Queue的使用

可以使用multiprocessing模块的Queue实现多进程之间的数据传递，Queue本身是一个消息列队程序，首先用一个小实例来演示一下Queue的工作原理：

```python
import multiprocessing
import time

if __name__ == '__main__':
    # 创建消息队列, 3:表示队列中最大消息个数
    queue = multiprocessing.Queue(3)
    # 放入数据
    queue.put(1)
    queue.put("hello")
    queue.put([3,5])
    # 总结: 队列可以放入任意数据类型
    # 提示： 如果队列满了，需要等待队列有空闲位置才能放入数据，否则一直等待
    # queue.put((5,6))
    # 提示： 如果队列满了，不等待队列有空闲位置，如果放入不成功直接崩溃
    # queue.put_nowait((5,6))
    # 建议： 向队列放入数据统一使用put

    # 查看队列是否满了
    # print(queue.full())

    # 注意点：queue.empty()判断队列是否空了不可靠
    # 查看队列是否空了
    # print(queue.empty())

    # 解决办法: 1. 加延时操作 2. 使用判断队列的个数,不使用empty
    # time.sleep(0.01)
    if queue.qsize() == 0:
        print("队列为空")
    else:
        print("队列不为空")

    # 获取队列的个数
    size = queue.qsize()
    print(size)

    # 获取数据
    value = queue.get()
    print(value)
    # 获取队列的个数
    size = queue.qsize()
    print(size)
    # 获取数据
    value = queue.get()
    print(value)
    # 获取数据
    value = queue.get()
    print(value)

    # 获取队列的个数
    size = queue.qsize()
    print(size)

    # 提示：如果队列空了，再取值需要等待，只有队列有值以后才能获取队列中数据
    # value = queue.get()
    # print(value)
    # 提示： 如果队列空了 ，不需要等待队列有值，但是如果取值的时候发现队列空了直接崩溃
    # 建议大家: 向队列取值使用get
    # value = queue.get_nowait()
    # print(value)
```

运行结果:

```
队列不为空
3
1
2
hello
[3, 5]
0
```

说明

初始化Queue\(\)对象时（例如：q=Queue\(\)），若括号中没有指定最大可接收的消息数量，或数量为负值，那么就代表可接受的消息数量没有上限（直到内存的尽头）；

- Queue.qsize\(\)：返回当前队列包含的消息数量；

- Queue.empty\(\)：如果队列为空，返回True，反之False , 注意这个操作是不可靠的。

- Queue.full\(\)：如果队列满了，返回True,反之False；

- Queue.get\(\[block\[, timeout\]\]\)：获取队列中的一条消息，然后将其从列队中移除，block默认值为True；

1）如果block使用默认值，且没有设置timeout（单位秒），消息列队如果为空，此时程序将被阻塞（停在读取状态），直到从消息列队读到消息为止，如果设置了timeout，则会等待timeout秒，若还没读取到任何消息，则抛出"Queue.Empty"异常；

2）如果block值为False，消息列队如果为空，则会立刻抛出"Queue.Empty"异常；

- Queue.get\_nowait\(\)：相当Queue.get\(False\)；

- Queue.put\(item,\[block\[, timeout\]\]\)：将item消息写入队列，block默认值为True；

1）如果block使用默认值，且没有设置timeout（单位秒），消息列队如果已经没有空间可写入，此时程序将被阻塞（停在写入状态），直到从消息列队腾出空间为止，如果设置了timeout，则会等待timeout秒，若还没空间，则抛出"Queue.Full"异常；

2）如果block值为False，消息列队如果没有空间可写入，则会立刻抛出"Queue.Full"异常；

- Queue.put\_nowait\(item\)：相当Queue.put\(item, False\)；

## 2\. 消息队列Queue完成进程间通信的演练

我们以Queue为例，在父进程中创建两个子进程，一个往Queue里写数据，一个从Queue里读数据：

```python
import multiprocessing
import time


# 写入数据
def write_data(queue):
    for i in range(10):
        if queue.full():
            print("队列满了")
            break
        queue.put(i)
        time.sleep(0.2)
        print(i)


# 读取数据
def read_data(queue):
    while True:
        # 加入数据从队列取完了，那么跳出循环
        if queue.qsize() == 0:
            print("队列空了")
            break
        value = queue.get()
        print(value)


if __name__ == '__main__':
    # 创建消息队列
    queue = multiprocessing.Queue(5)

    # 创建写入数据的进程
    write_process = multiprocessing.Process(target=write_data, args=(queue,))
    # 创建读取数据的进程
    read_process = multiprocessing.Process(target=read_data, args=(queue,))

    # 启动进程
    write_process.start()
    # 主进程等待写入进程执行完成以后代码再继续往下执行
    write_process.join()
    read_process.start()
```

运行结果：

```
0
1
2
3
4
队列满了
0
1
2
3
4
队列空了
```