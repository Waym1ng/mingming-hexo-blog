---
title: Flask+ nginx + gunicorn + supervisor 部署项目
date: 2022-06-23 15:59:19
tags:
    - flask
    - nginx
categories:
    - python
---

### Flask+ nginx + gunicorn + supervisor 部署项目
##### 编辑manage.py 文件 作为启动文件来管理Flask app
```python
from flask_script import Manager
from flask_migrate import Migrate, MigrateCommand

app = Flask(__name__)
manager = Manager(app)
# 数据库迁移初始化
Migrate(app, db)
# 添加迁移命令
manager.add_command("db", MigrateCommand)


if __name__=="__main__":
    # 使用manager对象启动flask项目，代替app.run()
    manager.run()

```

#####  在supervisor.conf 文件中增加 以下program
```python
[program:projects]
command=/usr/local/bin/gunicorn -w 4 -b 0.0.0.0:2020 -k gevent manage:app
directory=/home/zzz/projects/
autostart=true
redirect_stderr=true
stdout_logfile=/home/server_log/projects.log
```

> directory  这里填写项目的路径

如果没有写manage.py文件的话 command中 后面也可以直接写创建了 flask app 的文件
即 初始化了这句话的文件 app = Flask(__name__)

#### 这是gunicorn 的常用参数
> -w: 指定worker的数量（根据实际情况设定）
> -b：指定绑定的地址和端口号
> -k: 指定worker-class模式，默认为sync，这里用gevent使之变为异步协程，提高性能。 
>   最后指定app的位置。

#### nginx的简单配置
```python
  server {
      listen 80(监听的端口);
      server_name ip地址或者域名;
      
      location / {
    		proxy_pass http://127.0.0.1:2020(转发至的端口);
      }
  }
```
**nginx 的详细配置可以参考：**
[nginx 详解](https://blog.csdn.net/qq_40036754/article/details/102463099)
