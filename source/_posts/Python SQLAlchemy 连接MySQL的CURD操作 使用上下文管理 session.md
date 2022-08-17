---
title: Python SQLAlchemy 连接MySQL的CURD操作 使用上下文管理 session
date: 2020-07-08 17:20:59
tags: mysql 数据库 orm
categories: python加油鸭
---

<!--more-->

 -    ### 使用 contextmanager 来管理

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import scoped_session,sessionmaker


db_connect = "mysql+pymysql://root:password@localhost:3306/db_name?charset=utf8"

create=create_engine(db_connect)
SessionType=scoped_session(sessionmaker(bind=create,expire_on_commit=False))

def GetSession():
    return SessionType()

from contextlib import contextmanager

@contextmanager
def session_socpe():
    session=GetSession()
    try:
        yield session
        session.commit()
    except:
        session.rollback()
        raise
    finally:
        session.close()
```

 -    ###  用法
 -     查询

```python
with session_socpe() as session:
    obj = session.query(Model类).filter(Model类.字段==参数).first()
```

 -    增加

```python
with session_socpe() as session:
    obj = Model类(字段1=值1, 字段2=值2, 字段3=值3)
    session.add(obj)
```

 -     删除

```python
with session_socpe() as session:
    obj = session.query(Model类).filter(Model类.字段==参数).delete()
```

 -    修改

```python
with session_socpe() as session:
    obj = session.query(Model类).filter(Model类.字段==参数).update({"修改字段":"修改值"})
```

> 每次修改时，不用频繁地session.commit\(\)
> 
> 也方便了 万一出错后 的 rollback 回滚