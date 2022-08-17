---
title: python之sqlalchemy详解
date: 2020-03-25 09:28:14
tags: python mysql orm 数据库
categories: python加油鸭
---

<!--more-->

## **python之sqlalchemy详解**

## **SQLAlchemy连接数据库**

### **可以使用ORM框架操作数据库**

> ORM框架 = object relationship mappiy 框架

### **学习前提（软件）**

> mysql、pymysql（或者MySQLdb）、SQLAlchemy

### **SQLAlchemy连接数据库**

### **步骤**

- 首先导入sqlalchemy.create\_engine
- 输入配置信息（服务器ip，端口号，数据库名，账户，密码）
- 传递一个满足某种格式的字符串
  - dialect+driver://username:password\@host:port/database\?charset=utf8  
  
    dialect：数据库的实现，比如MySQL、PostgreSQL、SQLite，并且转换成小写。  
    driver：Python对应的驱动，python3对应sqlalchemy,python2对应MySQLdb。  
    username：连接数据库的用户名  
    password：连接数据库的密码  
    host：连接数据库的域名
- 创建数据库引擎
- 创建连接

### **代码实现**

```
from sqlalchemy import create_engine
​
#准备连接数据库基本信息
# 代表哪一台计算机，ip地址是多少
HOSTNAME = '127.0.0.1'
# 端口号
PORT = '3306'
# 数据库的名字，连接那个数据库
DATABASE = 'first_sqlalchemy'
# 数据库的账号和密码
USERNAME = 'root'
PASSWORD = 'root'
# 按照要求组织成一定的字符串
DB_URI = 'mysql+pymysql://{username}:{pwd}@{host}:{port}/{db}?charset=utf8'\
    .format(username =USERNAME,pwd = PASSWORD,host = HOSTNAME,port=PORT,db = DATABASE)
# 创建数据库引擎
engine = create_engine(DB_URI)
# 创建连接,如果运行之后，输入1则连接成功
with engine.connect()as con:
    rs = con.execute('SELECT 1')
    print (rs.fetchone())
```

### **拓展：用SQLAlchemy执行原生SQL**

```
from sqlalchemy import create_engine
from constants import DB_URI
#连接数据库
engine = create_engine(DB_URI,echo=True)
# 使用with语句连接数据库，如果发生异常会被捕获
with engine.connect() as con:
    # 先删除users表
    con.execute('drop table if exists authors')
    # 创建一个users表，有自增长的id和name
    con.execute('create table authors(id int primary key auto_increment,'name varchar(25))')
    # 插入两条数据到表中
    con.execute('insert into persons(name) values("abc")')
    con.execute('insert into persons(name) values("xiaotuo")')
    # 执行查询操作
    results = con.execute('select * from persons')
    # 从查找的结果中遍历
    for result in results:
        print(result)
```

## **ORM框架**

> ORM：Object Relationship Mapping（对象模型与数据库表的映射）

### **Python代码和SQL代码的对应关系**

> **py代码：**  
> class Person\(object\):  
> name = 'xx'  
> age = 18  
> country ='xx  
> 对应关系：  
> Person类 -> 数据库中的一张表  
> Person类中的属性 -> 数据库中一张表字段  
> Person类的一个对象 -> 数据库中表的一条数据

### **使用原生sql语句的缺点**

1.  SQL语句重复利用率不高，越复杂的SQL语句条件越多，代码越长，会出现很多相近的SQL语句。
2.  大型的项目是在业务逻辑中拼接出来的，如果数据库需要更改，就要去修改这些逻辑，这会很容易漏掉对某些SQL语句的修改。
3.  写SQL时容易忽略web安全问题，造成隐患。

### **SQLAlchemy的优势**

- 易用性：使用ORM做数据库开发可以有效减少重复SQL语句的概率，写出来的模型也更加直观、清晰
- 性能损耗小：任何操作都要通过ORM转化，确实会损耗性能，但综合考虑开发效率、代码阅读性，带来的好处远大于性能损耗，而且项目越大作用越明显。
- 设计灵活：可以轻松的写出复杂的查询
- 可移植：SQLAlchemy封装了底层的数据库实现，支持多个关系数据库引擎，可以非常轻松的切换数据库

## **SQLAlchemy操作数据库**

> 操作前，必须要先构建session对象！！

### **定义ROM模型映射到数据库中**

### **步骤：**

1.  用**declarative\_base**根据**engine**创建一个**ORM基类**。
2.  用这个**Base**类作为基类来写自己的ORM类。要定义`__tablename__`类属性，来指定这个模型映射到数据库中的表名。
3.  创建属性来映射到表中的字段，所有需要映射到表中的属性都应该为**Column类型**
4.  使用`Base.metadata.create_all()`来将模型映射到数据库中。

### **注意：**

使用create\_all函数创建的表映射到数据库之后，即使改变了模型的字段，也不会重新映射了。

体现为代码中，后加的**nick**项，没有办法实现。

解决方法：每次运行时，先把之前的表全部删除之后，在创建新的表。

### **代码实现：**

```
# 需求：创建好一个ORM类模型,并且映射到指定的数据库中成为表
​
# 1、用`declarative_base`根据`engine`创建一个ORM基类Base。
​
Base = declarative_base(engine)
​
# 2、用这个`Base`类作为基类来写自己的ORM类。要定义`__tablename__`类属性，来指定这个模型映射到数据库中的表名。
​
class Person(Base):
​
    __tablename__ = 't_person'
​
# 3、在这个ORM模型中创建一些属性，来跟表中的字段进行 一一 映射。这些属性必须是sqlalchemy给我们提供好的数据类型
​
    id = Column(Integer,primary_key=True,autoincrement= True)
​
    # nullable = True  可以为空
​
    name = Column(String(50),nullable=False)
​
    age = Column(Integer)
​
    country = Column(String(50))
​
    # 后加入的nick项
​
    nick = Column(String(20))
​
# 4、使用`Base.metadata.create_all()`来将模型映射到数据库中
​
# Base.metadata.create_all()
​
# 注意点：一旦使用`Base.metadata.create_all()`将模型映射到数据库中后，即使改变了模型的字段，也不会重新映射了。
# 解决方法：先将表全部删除之后再创建新的表
# 将Base上的ORM类模型对应的数据表都删除
Base.metadata.drop_all()
# 创建Base上的ORM类到数据库中成为表
Base.metadata.create_all()
```

### **对数据的增删改查操作（crud→增删改查）**

> crud功能指的就是增删改查。

### **步骤：**

1.  构建session对象，所有的所有和数据库的ORM操作都必须通过一个叫做`session`的会话对象来实现
2.  对表添加数据**\{add\(\),add\_all\(\[\]\)\}**
3.  查询表中数据
    1.  全表查询 查找某个模型对应的那个表中所有的数据  
     all\_person = session.query\(Person\).all\(\)
    2.  条件查询 filter\_by  
     all\_person = session.query\(Person\).filter\_by\(name='momo1'\).all\(\)
    3.  条件查询 filter  
     all\_person = session.query\(Person\).filter\(Person.name=='momo1'\).all\(\)
    4.  条件查询 get方法是根据id来查找的，只会返回一条数据或者None  
     person = session.query\(Person\).get\(primary\_key\)
    5.  条件查询 使用first方法获取结果集中的第一条数据  
     person = session.query\(Person\).first\(\)

 

1.  修改对象  
    首先从数据库中**查找对象**，然后将这条数据**修改**为你想要的数据，最后做**commit**操作就可以修改数据了
2.  删除对象  
    将需要删除的数据从数据库中**查找出来**，然后使用**session.delete**方法将这条数据从session中删除，最后做**commit**操作就可以了。

### **代码实现：**

```
'''
    需求2：crud的操作
        0、前提：获取到根数据库，进行会话的对象 
        1、对表，添加数据
        2、查询表数据
        3、修改表，指定行数据
        4、删除表数据
'''
# 获取到根数据库，进行会话的对象 ，构建session对象，所有的所有和数据库的ORM操作都必须通过一个叫做`session`的会话对象来实现
# Session = sessionmaker(engine)
# session = Session()
#  下一行代码相当于上两行
session = sessionmaker(engine)()
​
# 1、对表，添加数据  session.add() session.add_all([])
p = Person(name='MGorz',age =18,country ='China',nick='诚哥')
session.add(p) # 一次只能插入一条数据
session.commit()
# 添加多条数据 
p2 = Person(name='Mrz',age =18,country ='China',nick='ha')
p3 = Person(name='MGz',age =18,country ='China',nick='e')
session.add_all([p2,p3])
session.commit()
​
# 2、查询表中数据
# 2.1   全表查询  ()中填类名,
ps = session.query(Person).all()
for p in ps :
    print(p)
# 2.2   条件查询    filter_by
# filter_by(),括号中填入想查询的。
ps = session.query(Person).filter_by(name = "MGorz").all()
for p in ps :
    print(p)
# 2.3   条件查询    filter
# 注意和filter_by的区别
ps = session.query(Person).filter(Person.name == "MGorz").all()
for p in ps :
    print(p)
# 2.4   条件查询    get
ps = session.query(Person).get(2)
print(ps)
# 2.5   条件查询    first
ps = session.query(Person).first()
print(ps)
​
# 3、修改制定行数据
person = session.query(Person).first()
print(person)
# 修改表数据，对象.属性 = '   '
person.name = 'hahaha'
# 别忘记要提交哦~
session.commit()
​
# 4、删除表数据
person = session.query(Person).get(3)
print(person)
session.delete(person)
session.commit()
```

### **Column常用数据类型**

常用的数据类型，一共是有11个。

> 注：在SQLAlchemy中不存在double数据类型，使用 **DECIMAL**类型替代

让我们来分别看下这11个数据类型都有哪些？

1.  **Integer**：整型，映射到数据库中是**int**类型。
2.  **Float**：浮点类型，映射到数据库中是**float**类型。他占据的32位。  
    浮点类型，有可能会造成精度丢失，特别是在money方面，不可原谅。
3.  Double（**SQLAlchemy中没有，代替品为DECIMAL**）：双精度浮点类型，映射到数据库中是double类型，占据64位
4.  **String**：可变字符类型，映射到数据库中是**varchar**类型.
5.  **Boolean**：布尔类型，映射到数据库中的是**tinyint**类型。
6.  **DECIMAL**：定点类型。是专门为了**解决浮点类型精度丢失**的问题的。在存储money相关的字段的时候建议大家都使用这个数据类型。并且这个类型使用的时候需要传递两个参数，**第一个参数是用来标记这个字段总能能存储多少个数字**，**第二个参数表示小数点后有多少位**。
7.  **Enum**：枚举类型。指定某个字段只能是枚举中指定的几个值，不能为其他值。在ORM模型中，使用**Enum**来作为枚举。
8.  **Date**：存储时间，只能**存储年月日**。映射到数据库中是date类型。在Python代码中，可以使用`datetime.date`来指定。
9.  **DateTime**：存储时间，可以**存储年月日时分秒毫秒等**。映射到数据库中也是datetime类型。在Python代码中，可以使用`datetime.datetime`来指定。
10.  **Time**：存储时间，可以**存储时分秒**。映射到数据库中也是time类型。在Python代码中，可以使用`datetime.time`来指定  
    ps：注意区分Date/DateTime/Time的储存信息！！！
11.  **Text**：存储长字符串。一般可以存储**6W**多个字符
12.  **LONGTEXT**：长文本类型，映射到数据库中是longtext类型（**不过这个只有mysql有，orcale没有**）

上述就是**11个**数据类型，因为这里把不存在的double也添加上了，所以看起来是12个。

下面就让咱们一起来进行验证。

### **代码实现**

```
# 依次导入常用的数据类型
from sqlalchemy import create_engine, Column, Integer, String,Float,DECIMAL,Boolean,Date,DateTime,Time,Text,Enum
​
# 从sqlalchemy的方言模块dialects导入mysql专有的LONGTEXT,长文本类型
from sqlalchemy.dialects.mysql import LONGTEXT
​
# 在python3.x中有enum模块
import enum
​
#导入时间了
from datetime import datetime,date,time
​
​
# 用`declarative_base`根据`engine`创建一个ORM基类。
from sqlalchemy.ext.declarative import declarative_base
​
# 引入创建py和数据库连接的sessionmaker函数
from sqlalchemy.orm import sessionmaker
​
​
​
# 准备连接数据库基本信息
HOSTNAME = '127.0.0.1'
PORT = '3306'
DATABASE = 'first_sqlalchemy'
USERNAME = 'root'
PASSWORD = 'root'
DB_URI = 'mysql+pymysql://{username}:{pwd}@{host}:{port}/{db}?charset=utf8' \
    .format(username=USERNAME, pwd=PASSWORD, host=HOSTNAME, port=PORT, db=DATABASE)
​
​
​
# 创建数据库引擎
engine = create_engine(DB_URI)
​
Base = declarative_base(engine)
​
​
​
# 需求：常用的数据类型
​
# 枚举另外一种写法，导入enum模块，定义枚举类
class TagEnum(enum.Enum):
    python = "PYTHON"
    flask = 'FLASK'
    django = 'DJANGO'
​
​
​
class News(Base):
    __tablename__ = 'news'
    id = Column(Integer,primary_key=True,autoincrement=True)
    price1 = Column(Float)   # 不过存储数据时存在精度丢失的问题
    price2 = Column(DECIMAL(10,4))
    title = Column(String(50))
    is_delete = Column(Boolean)
    tag1 = Column(Enum('PYTHON','FLASK','DJANGO'))  # 枚举的常规写法（推荐）
    tag2 = Column(Enum(TagEnum))  # 另一种写法
    create_time1 = Column(Date)
    create_time2 = Column(DateTime)
    create_time3 = Column(Time)
    content1 = Column(Text)
    content2 = Column(LONGTEXT)
​
​
​
​
# 将Base上的ORM类模型对应的数据表都删除
# Base.metadata.drop_all()
# 创建Base上的ORM类到数据库中成为表
# Base.metadata.create_all()
​
​
# 新增数据到表news中
session = sessionmaker(engine)()
# 注意float出现的精度丢失问题      is_delete,布尔类型true：1，false:0
news1 = News(price1=1000.0078,price2=1000.0078,title='测试数据',is_delete=True,tag1="PYTHON",tag2=TagEnum.flask,    create_time1=date(2018,12,12),create_time2=datetime(2019,2,20,12,12,30),create_time3=time(hour=11,minute=12,second=13),content1="hello",content2 ="hello   hi   nihao")
​
​
session.add(news1)
session.commit()
```

### **Column常用约束参数**

在给数据库表指定key的时候，必然要给它们添加，例如：不可空，字节长度等等的限制，这就需要约束参数的出场了。常用的约束参数一共有**7**种

### **常见约束参数**

**约束参数描述，功能**primary\_keyTrue设置某个字段为主键autoincrementTrue设置这个字段为自动增长的default设置某个字段的默认值。在发表时间这些字段上面经常用nullable指定某个字段是否为空。默认值是True，就是可以为空unique指定某个字段的值是否唯一。默认是False。onupdate在**数据更新**的时候会**调用**这个参数指定的**值或者函数**。在_第一次插入这条数据的时候，不会用onupdate的值，只会使用default的值_。常用于是`update_time`字段（每次更新数据的时候都要更新该字段值）。name指定ORM模型中某个属性映射到表中的字段名。如果**不指定**，那么会**使用这个属性的名字来作为字段名**。这个参数也可以当作位置参数，在第1个参数来指定。

### **代码实现**

```
# 注意千万不能忘记要先创建session对象啊！！
# 需求：sqlalchemy中Column常用的约束参数
class News(Base):
    __tablename__ = 'news'
    id = Column(Integer,primary_key=True,autoincrement=True)
    # 这个Datetime是sqlalchemy中的
    create_time = Column(DateTime,default=datetime.now)
    read_count = Column(Integer,default=11)
    title1 = Column(String(50),name='my_title',nullable=False)
    title2 = Column('my_title22',String(50), nullable=False)# name 属性的两种方式
    telephone = Column(String(11),unique=True)
    update_time = Column(DateTime,onupdate=datetime.now,default=datetime.now)#onupdate是在数据更新的时候才会起作用，插入数据时候不起作用
​
# Base.metadata.drop_all()
# Base.metadata.create_all()
​
news1 = News(title1 ='hahah',title2 = 'womingsad')
session.add(news1)
#结束之后，必须要提交
session.commit()
```

### **Query查询函数，可以传递的参数有哪些？**

Query查询函数传递的参数无非就是分为**三类**，分别是**模型名、模型中的属性、聚合函数**，那下面就简单的说明，

### **模型名**

> 指定查找该模型中所有属性（对应查询为全表查询）

### **模型中的属性**

> 可以指定只查找某个模型的其中几个属性

### **聚合函数**

> 聚合函数一共有五个，都被放到了sqlalchemy的func模块中。

1.  func.count：统计行的数量
2.  func.avg：求平均值
3.  func.max：求最大值
4.  func.min：求最小值
5.  func.sum：求和

### **代码实现**

```
# 新增测试数据
​
for x in range(1,6):
​
    a = News(title = "标题%s"%x,price = random.randint(1,100))
​
    session.add(a)
# 提交
session.commit()
​
​
​
# 三种参数
# 模型名
result = session.query(News).all()  #全表查询
print(result)
​
# 模型名的属性名,返回的列表中的元素是元组类型数据
result = session.query(News.title,News.price).all()
print(result)
​
# 聚合函数
# 统计行的数量
r = session.query(func.count(News.id)).first()
print(r)
#最大值
r = session.query(func.max(News.price)).first()
print(r)
#最小值
r = session.query(func.min(News.price)).first()
print(r)
#平均值
r = session.query(func.avg(News.price)).first()
print(r)
#求和
r = session.query(func.sum(News.price)).first()
print(r)
```

 转自：<https://zhuanlan.zhihu.com/p/71134861>