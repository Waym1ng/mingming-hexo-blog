---
title: SQLAlchemy 中的 Session,sessionmaker,scoped_session详解
date: 2020-03-25 17:48:54
tags: python 数据库 mysql sql
categories: python加油鸭
---

<!--more-->

## 一、关于 SQLAlchemy

什么是SQLAlchemy？

答：SQLAlchemy是一个关系型数据库框架，它提供了高层的 ORM 和底层的原生数据库的操作，让开发者不用直接和 SQL 语句打交道，而是通过 Python 对象来操作数据库，在舍弃一些性能开销的同时，换来的是开发效率的较大提升。

一句话：就是对数据库的抽象！

---

Session 其实 就是一个会话, 可以和数据库打交道的一个会话

在一般的意义上， 会话建立与数据库的所有对话，并为你在其生命周期中加载或关联的所有对象表示一个“等待区”。他提供了一个入口点获得查询对象， 向数据库发送查询，使用会话对象的当前数据库连接， 将结果行填充在对象中， 然后存储在会话中， 在这种结构中称为身份映射 – 这种数据结构维护了每一个副本的唯一， 这种唯一意味着一个对象只能有一个特殊的唯一主键。

会话以基本无状态的形式开始，一旦发出查询或其他对象被持久化，它就会从一个引擎申请连接资源，该引擎要么与会话本身相关联，要么与正在操作的映射对象相关联。此连接标识正在进行的事务， 在会话提交或回滚其挂起状态之前，该事务一直有效。

会话中维护的所有变化的对象都会被跟踪 \- 在再次查询数据库或提交当前事务之前， 它将刷新对数据库的所有更改， 这被称为工作模式单元。

在使用会话时候，最重要的是要注意与它相关联的对象是会话所持有的事务的代理对象 \- 为了保持同步，有各种各样的事件会导致对象重新访问数据库。可能从会话中分离对象并继续使用他们，尽管这种做法有其局限性。但是通常来说，当你希望再次使用分离的对象时候，你会将他们与另一个会话重新关联起来， 以便他们能够恢复表示数据库状态的正常任务

### 1\. Session是缓存吗？

```bash
可能会将这里的session与http中的session搞混，需要注意的是，它有点用作缓存，因为它实现了 身份映射 模式，并存储了键入其主键的对象。但是，它不执行任何类型的查询缓存。
此外，默认情况下，Session使用弱引用存储对象实例。这也违背了将Session用作缓存的目的。关于session强应用下次再讨论。
```

### 2\. Session作用：

```
1. session创建和管理数据库连接的会话
2. model object 通过session对象访问数据库，并把访问到的数据以 Identity Map的方式，映射到Model object中
```

### 3\. Session生命周期：

```
1. session在刚被创建的时候，还没有和任何model object 绑定，可认为是无状态的
2. session 接受到query查询语句, 执行的结果或保持或者关联到session中
3. 任意数量的model object被创建，并绑定到session中，session会管理这些对象
4. 一旦session 里面的objects 有变化，那可是要commit/rollback提交或者放弃changs
```

### 4\. Session什么时候创建，提交，关闭？

```
一般来说，session在需要访问数据库的时候创建，在session访问数据库的时候，准确来说，应该是“add/update/delete”数据库的时候，会开启database transaction。

假设没有修改autocommit的默认值(False), 那么，database transaction 一直会保持，只有等到session发生rolled back、committed、或者closed的时候才结束，一般建议，当database transaction结束的时候，同时close session,以保证，每次发起请求，都会创建一个新的session

特别是对web应用来说，发起一个请求，若请求使用到Session访问数据库，则创建session，处理完这个请求后，关闭session
```

### 4\. 获取一个Session：

```
Session 是一个直接实例化的常规的Python 类。然而， 为了标准会会话的配置和获取方式， sessionmaker 类通常用于创建顶级会话配置， 然后可以在整个应用程序中使用它， 就不需要重复配置参数。
```

下面是sessionmaker 的使用方式

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

# 创建连接数据库的引擎，session连接数据库需要
my_engine = create_engine('mysql+pymysql://root:123456@localhost/my_db')

# 创建一个配置过的Session类
Session = sessionmaker(bind=my_engine)

# 实例化一个session
db_session = Session()

# 使用session
myobject = MyObject('foo', 'bar')
db_session.add(myobject)
db_session.commit()

```

在上面，该\*\*sessionmaker\(\)创建了一个工厂类，在创建这个工厂类时我们配置了参数绑定了引擎。将其赋值给Session。每次实例化Session都会创建一个绑定了引擎的Session。\*\*这样这个session在访问数据库时都会通过这个绑定好的引擎来获取连接资源  
当你编写应用程序时， 请将sessionmaker 工厂放在全局级别，视作应用程序配置的一部分。例如：应用程序包中有三个.py文件，您可以将该sessionmaker行放在\_\_init\_\_.py文件中; 在其他模块“from mypackage import Session”。这样，所有的Session\(\)的配置都由该配置中心控制。

### 5\. 关于SQLAlchemy 的 create\_engine：

直接只用 create\_engine 时，就会创建一个带连接池的引擎：

```
my_engine = create_engine('mysql+pymysql://root:123456@localhost/my_db')
```

创建一个session，连接池会分配一个connection。当session在使用后显示地调用 session.close\(\)，也不能把这个连接关闭，而是由由QueuePool连接池管理并复用连接。

确保 session 在使用完成后用 session.close、session.commit 或 session.rollback 把连接还回 pool，这是一个必须在意的习惯。

**关于SQLAlchemy 数据库连接池：**

```
session 和 connection 不是相同的东西， session 使用连接来操作数据库，一旦任务完成 session 会将数据库 connection 交还给 pool。

在使用 create_engine 创建引擎时，如果默认不指定连接池设置的话，一般情况下，SQLAlchemy 会使用一个 QueuePool 绑定在新创建的引擎上。并附上合适的连接池参数
```

create\_engine\(\) 函数和连接池相关的参数有：

- pool\_recycle, 默认为 -1, 推荐设置为 7200, 即如果 connection 空闲了 7200 秒，自动重新获取，以防止 connection 被 db server 关闭。
- pool\_size=5, 连接数大小，默认为 5，正式环境该数值太小，需根据实际情况调大
- max\_overflow=10, 超出 pool\_size 后可允许的最大连接数，默认为 10, 这 10 个连接在使用过后，不放在 pool 中，而是被真正关闭的。
- pool\_timeout=30, 获取连接的超时阈值，默认为 30 秒

SQLAlchemy不使用连接池：  
在创建引擎时指定参数 poolclass=NullPool 即禁用了SQLAlchemy提供的数据库连接池。SQLAlchemy 就会在执行 session.close\(\) 后立刻断开数据库连接。当然，如果没有被调用 session.close\(\)，则数据库连接不会被断开，直到程序终止。

```
my_engine = create_engine('mysql+pymysql://root:123456@localhost/my_db'，poolclass=NullPool)
```

关于 SQLAlchemy 的 engine ,这里有一篇文章写的很好：<http://sunnyingit.github.io/book/section_python/SQLalchemy-engine.html>

### 6\. 关于线程安全：

session不是线程安全的，在多线程的环境中，默认情况下，多个线程将会共享同一个session。试想一下，假设A线程正在使用session处理数据库，B线程已经执行完成，把session给close了，那么此时A在使用session就会报错，怎么避免这个问题？

```
1. 可以考虑在这些线程之间共享Session及其对象。但是应用程序需要确保实现正确的锁定方案，以便多个线程不会同时访问Session或其状态。SQLAlchemy 中的 scoped_session 就可以证线程安全，下面会有讨论。
2. 为每个并发线程维护一个会话，而不是将对象从一个Session复制到另一个Session，通常使用Session.merge()方法将对象的状态复制到一个不同Session的新的本地对象中。
```

---

## 二、单线程下 scoped\_session 对创建 Session 的影响

上面简单介绍了sessionmaker的作用，下面开始探讨 scoped\_session 对创建 Session 的影响。现在先探讨单线程情况。

---

先声明待会实验用的模型：

```python
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Column, Integer, String
from sqlalchemy import create_engine


Base = declarative_base
engine = create_engine("mysql+pymysql://root:123456@127.0.0.1:3306/my_db?charset=utf8mb4")


class Person(Base):
    __tablename__ = 'Person'

    id = Column(Integer, primary_key=True, autoincrement=True)
    name = Column(String(length=64), comment='姓名')
    mobile = Column(String(length=13), comment='手机号')
    id_card_number = Column(String(length=64), comment='身份证')

    def __str__(self):
        return '%s(name=%r,mobile=%r,id_card_number=%r)' % (
            self.__class__.__name__,
            self.name,
            self.mobile,
            self.id_card_number
        )

# 在数据库中创建模型对象的表
Base.metadata.create_all(engine)

```

### 1\. 两个 Session 添加同一个对象

1.1 在 commit 之前添加：

```python

from sqlalchemy.orm import sessionmaker, scoped_session


session_factory = sessionmaker(bind=engine)
# engine 在上面已经创建好了


person = Person(name='frank-' + 'job3', mobile='111111', id_card_number='123456789')

Session= session_factory()
s1= session_factory()
# s1 : <sqlalchemy.orm.session.Session object at 0x107ec8c18>

s2 = session_factory() 
# s2 : <sqlalchemy.orm.session.Session object at 0x107ee3ac8>

print(s1 is s2)
# False

id(s1),id(s2)
# 4427910168, 4428020424

s1.add(person)
s2.add(person)
# 会报错！
# sqlalchemy.exc.InvalidRequestError: Object '<Person at 0x22beb14bf60>' is already attached to session '2' (this is '3')

```

**结论：**

```
通过 sessionmaker 工厂创建了两个 Session ，而且可以看到 s1 s2 是两个不同的 Session 。
在 s1 添加 person 后，继续使用 s2 添加 person 报错. 说 person 这个对象 已经和 另一个 Session 关联一起来了, 所以再次关联另一个 Session 就会报错。
```

1.2 在 commit 之后添加：

即在上面代码的 s1.add\(person\) 之后， s1.commit\(\) ,然后再 s2.add\(persion\)  
这里就没帖代码了。

**结论：**

```
即使在 s1 提交之后，s2 再去添加 person 也会发生错误,但 s1 的提交是成功了的，数据 person 已经存放在数据库了。
当 s1 添加 person 并提交，然后关闭 s1 ,s2再去添加并提交 person 数据库,这不会报错，但是数据库也不会出现两条 person 数据。
```

1.3 再 close 之后添加：

```python
p = Person(name='frank', mobile='11111111', id_card_number='123456789')

s1.add(p)
s1.commit()

s2.add(p)
s2.commit()

# 也会报错！！！
# sqlalchemy.exc.InvalidRequestError: Object '<Person at 0x21207e3e128>' is already attached to session '2' (this is '3')


p = Person(name='frankcc', mobile='1111111122', id_card_number='1234567890')

s1.add(p)
s1.commit()
s1.close()
s2.add(p)
s2.commit()

# 不会报错

```

**结论：**

```
s1 关闭之后， s2再去添加提交同一个对象，不会报错，但是数据库值有一条 person 数据。
```

### 2\. 不同的 Session 添加不同的对象

```python
person4 = Person(name='frank-' + 'job4', mobile='4444444444', id_card_number='123456789')
person1 = Person(name='frank-' + 'job1', mobile='111111', id_card_number='123456789')

s1.add(person1)
s2.add(person4)
s1.commit()  # 提交数据
s2.commit()  # 提交数据, 写入数据库

```

**结论：**

```
当然，s1 ,s2 添加提交不同的对象，不会出错。在数据库成功新增数据。 
```

```bash

mysql> select * from person;
+----+------------+------------+----------------+
| id | name       | mobile     | id_card_number |
+----+------------+------------+----------------+
|  1 | frank-job1 | 111111     | 123456789      |
|  2 | frank-job4 | 4444444444 | 123456789      |
+----+------------+------------+----------------+
2 rows in set (0.00 sec)

```

---

**以上说明：**

```
一个对象一旦被一个 Session 添加，除非关闭这个 Session ，不然其他的 Session 无法添加这个对象。
一个 Session 添加并提交一个对象，然后关闭该 Session ，其他的 Session 可以添加并提交这个对象，但是数据库并不会有这条数据。
```

---

### 3\. 用 scoped\_session 创建 Session

```python

session_factory = sessionmaker(bind=engine)
Session = scoped_session(session_factory)


s1 = Session()
# <sqlalchemy.orm.session.Session object at 0x0000020E58690240>

s2 = Session()
# <sqlalchemy.orm.session.Session object at 0x0000020E58690240>

print(s1 is s2)
# True


p = Person(name='frankaaabb', mobile='1111111122233', id_card_number='12345678900099')

s1.add(p)
s2.add(p)
s2.commit()

```

**结论：**

```
可以看到，通过scoped_session再去创建 Session ，返回的是同一个 Session 。
scoped_session类似单例模式，当我们调用使用的时候，会先在Registry里找找之前是否已经创建Session，未创建则创建 Session ，已创建则直接返回。
```

---

## 三、多线程下 scoped\_session 对创建 Session 的影响

这里探讨在多线程下使用 scoped\_session 与不使用 scoped\_session 的情况

---

### 1\. 当不用 scoped\_session 时：

当不使用 scoped\_session 时，也分两种情况，是否创建全局性 Session

1.1 多线程下不设置 Session 为全局变量

```python

session_factory = sessionmaker(bind=engine)
Session = session_factory


def job(name):
    session = Session()

    print(f"id session:{id(session)}")

    person = Person(name='frank-' + name, mobile='111111', id_card_number='123456789')
    print(f"{name} person is add..")
    session.add(person)

    time.sleep(1)
    if name == 'job3':
        # 线程3 提交, 其他线程不提交.
        session.commit()
        session.close()


if __name__ == '__main__':

    thread_list = []

    # 创建5个线程
    for i in range(5):
        name = 'job' + str(i)
        t = threading.Thread(target=job, name=name, args=(name,))

        thread_list.append(t)

    for t in thread_list:
        t.start()

    for t in thread_list:
        t.join()

```

**结论：**

```
每个线程下的 Session 都是不同的 Session
数据库成功新增了线程3提交的数据，其他的线程中的数据并没有提交到数据库中去。
```

```bash

id session:2557871997392
job0 person is add..
id session:2557871998064
job1 person is add..
id session:2557871998568
job2 person is add..
id session:2557871999072
job3 person is add..
id session:2557871999688
job4 person is add..


mysql> select * from person;
+----+------------+--------+----------------+
| id | name       | mobile | id_card_number |
+----+------------+--------+----------------+
| 14 | frank-job3 | 111111 | 123456789      |
+----+------------+--------+----------------+
1 row in set (0.00 sec)

```

1.2 在多线程下用全局 Session

```python

session_factory = sessionmaker(bind=engine)
Session = session_factory
session = Session()


def job(name):
    global session

    print(f"id session:{id(session)}")

    person = Person(name='frank-' + name, mobile='111111', id_card_number='123456789')
    print(f"{name} person is add..")
    session.add(person)

    time.sleep(1)
    if name == 'job3':
        # 线程3 提交, 其他线程不提交.
        session.commit()
        session.close()


if __name__ == '__main__':

    thread_list = []

    # 创建5个线程
    for i in range(5):
        name = 'job' + str(i)
        t = threading.Thread(target=job, name=name, args=(name,))

        thread_list.append(t)

    for t in thread_list:
        t.start()

    for t in thread_list:
        t.join()

```

**结论：**

```
全部线程下的 Session 都时同一个 Session
每个线程下的数据都被提交到了数据库
```

```bash

id session:2737857674824
job0 person is add..
id session:2737857674824
job1 person is add..
id session:2737857674824
job2 person is add..
id session:2737857674824
job3 person is add..
id session:2737857674824
job4 person is add..


mysql> select * from person;
+----+------------+--------+----------------+
| id | name       | mobile | id_card_number |
+----+------------+--------+----------------+
| 15 | frank-job0 | 111111 | 123456789      |
| 16 | frank-job2 | 111111 | 123456789      |
| 17 | frank-job1 | 111111 | 123456789      |
| 18 | frank-job3 | 111111 | 123456789      |
| 19 | frank-job4 | 111111 | 123456789      |
+----+------------+--------+----------------+
5 rows in set (0.00 sec)

```

### 2\. 当使用 scoped\_session 时：

2.1 多线程下不设置 Session 为全局变量

```python

session_factory = sessionmaker(bind=engine)
Session = scoped_session(session_factory)


def job(name):
    session = Session()

    print(f"id session:{id(session)}")

    person = Person(name='frank-' + name, mobile='111111', id_card_number='123456789')
    print(f"{name} person is add..")
    session.add(person)

    time.sleep(1)
    if name == 'job3':
        # 线程3 提交, 其他线程不提交.
        session.commit()
        session.close()


if __name__ == '__main__':

    thread_list = []

    # 创建5个线程
    for i in range(5):
        name = 'job' + str(i)
        t = threading.Thread(target=job, name=name, args=(name,))

        thread_list.append(t)

    for t in thread_list:
        t.start()

    for t in thread_list:
        t.join()


```

**结论：**

```
每个线程下的 Session 都不相同
只有线程3下的数据被提交到了数据库
```

```bash

id session:2173841850832
job0 person is add..
id session:2173841851504
job1 person is add..
id session:2173841851896
job2 person is add..
id session:2173841852008
job3 person is add..
id session:2173841853128
job4 person is add..


mysql> select * from person;
+----+------------+--------+----------------+
| id | name       | mobile | id_card_number |
+----+------------+--------+----------------+
| 32 | frank-job3 | 111111 | 123456789      |
+----+------------+--------+----------------+
1 row in set (0.00 sec)


```

2.2 多线程下设置 Session 为全局变量

```python

session_factory = sessionmaker(bind=engine)
Session = scoped_session(session_factory)
session = Session()


def job(name):
    global session

    print(f"id session:{id(session)}")

    person = Person(name='frank-' + name, mobile='111111', id_card_number='123456789')
    print(f"{name} person is add..")
    session.add(person)

    time.sleep(1)
    if name == 'job3':
        # 线程3 提交, 其他线程不提交.
        session.commit()
        session.close()


if __name__ == '__main__':

    thread_list = []

    # 创建5个线程
    for i in range(5):
        name = 'job' + str(i)
        t = threading.Thread(target=job, name=name, args=(name,))

        thread_list.append(t)

    for t in thread_list:
        t.start()

    for t in thread_list:
        t.join()


```

**结论：**

```
每个线程下的 Session 是同一个 Session
每个线程下的数据都没提交到了数据库
```

```bash

id session:2810724382032
job0 person is add..
id session:2810724382032
job1 person is add..
id session:2810724382032
job2 person is add..
id session:2810724382032
job3 person is add..
id session:2810724382032
job4 person is add..


mysql> select * from person;
+----+------------+--------+----------------+
| id | name       | mobile | id_card_number |
+----+------------+--------+----------------+
| 33 | frank-job0 | 111111 | 123456789      |
| 34 | frank-job2 | 111111 | 123456789      |
| 35 | frank-job1 | 111111 | 123456789      |
| 36 | frank-job3 | 111111 | 123456789      |
| 37 | frank-job4 | 111111 | 123456789      |
+----+------------+--------+----------------+
5 rows in set (0.00 sec)

```

---

**以上说明：**

```
在同一个线程中，有 scoped_session 的时候，返回的是同一个 Session 对象。
在多线程下，即使通过  scoped_session 创建Session，每个线程下的 Session 都是不一样的，每个线程都有一个属于自己的 Session 对象，这个对象只在本线程下共享。 
scoped_session 只有在单线程下才能发挥其作用。在多线程下显得没有什么作用
```

 转自：<https://www.cnblogs.com/ChangAn223/p/11277468.html>