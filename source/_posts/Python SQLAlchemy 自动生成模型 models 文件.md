---
title: Python SQLAlchemy 自动生成模型 models 文件
date: 2020-07-08 16:57:56
tags: python mysql 数据库
categories: python加油鸭
---

<!--more-->

 -     安装模块

```bash
pip3 install sqlacodegen
```

 -     执行  
 

```bash
sqlacodegen mysql+pymysql://root:password@127.0.0.1:3306/db_name > test_model.py

root:mysql 用户
password:mysql 密码
db_name: 数据库名称
test_model.py:导出的名字

--tables test : 可以指定test数据表
```

 -    查看 py 文件  得到以下

```python
from sqlalchemy import CHAR, Column, DateTime, Enum, String, Text, text
from sqlalchemy.dialects.mysql import INTEGER
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()
metadata = Base.metadata

class SaveUser(Base):
    __tablename__ = 'save_user'

    id = Column(INTEGER(11), primary_key=True)
    userName = Column(String(32), unique=True)
    password = Column(String(16), index=True)
    createTime = Column(DateTime)
```

-  成功\!