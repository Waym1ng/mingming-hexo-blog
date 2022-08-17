---
title: Linux ubuntu pip install mysqlclient 报错问题解决
date: 2020-07-13 11:58:23
tags: linux ubuntu python
categories: Linux
---

<!--more-->

 -    ### 报错

```bash
Looking in indexes: https://pypi.douban.com/simple
Collecting mysqlclient
  Using cached https://pypi.doubanio.com/packages/a5/e1/e5f2b231c05dc51d9d87fa5066f90d1405345c54b14b0b11a1c859020f21/mysqlclient-2.0.1.tar.gz (87 kB)
    ERROR: Command errored out with exit status 1:
     command: /home/zunder/Desktop/env/djenv/bin/python3 -c 'import sys, setuptools, tokenize; sys.argv[0] = '"'"'/tmp/pip-install-1qfymr2e/mysqlclient/setup.py'"'"'; __file__='"'"'/tmp/pip-install-1qfymr2e/mysqlclient/setup.py'"'"';f=getattr(tokenize, '"'"'open'"'"', open)(__file__);code=f.read().replace('"'"'\r\n'"'"', '"'"'\n'"'"');f.close();exec(compile(code, __file__, '"'"'exec'"'"'))' egg_info --egg-base /tmp/pip-pip-egg-info-9lw6d9f3
         cwd: /tmp/pip-install-1qfymr2e/mysqlclient/
    Complete output (12 lines):
    /bin/sh: 1: mysql_config: not found
    /bin/sh: 1: mariadb_config: not found
    /bin/sh: 1: mysql_config: not found
    Traceback (most recent call last):
      File "<string>", line 1, in <module>
      File "/tmp/pip-install-1qfymr2e/mysqlclient/setup.py", line 15, in <module>
        metadata, options = get_config()
      File "/tmp/pip-install-1qfymr2e/mysqlclient/setup_posix.py", line 65, in get_config
        libs = mysql_config("libs")
      File "/tmp/pip-install-1qfymr2e/mysqlclient/setup_posix.py", line 31, in mysql_config
        raise OSError("{} not found".format(_mysql_config_path))
    OSError: mysql_config not found
    ----------------------------------------
ERROR: Command errored out with exit status 1: python setup.py egg_info Check the logs for full command output.
```

- ### 解决

> apt-get install libmysqlclient-dev

 

```bash
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following additional packages will be installed:
  libmysqlclient20 libssl-dev libssl-doc libssl1.0.0 zlib1g zlib1g-dev
The following NEW packages will be installed:
  libmysqlclient-dev libmysqlclient20 libssl-dev libssl-doc zlib1g-dev
The following packages will be upgraded:
  libssl1.0.0 zlib1g
2 upgraded, 5 newly installed, 0 to remove and 721 not upgraded.
Need to get 4,260 kB/5,394 kB of archives.
After this operation, 20.5 MB of additional disk space will be used.
Do you want to continue? [Y/n] Y
Get:1 https://mirrors.tuna.tsinghua.edu.cn/ubuntu xenial-updates/main amd64 libmysqlclient20 amd64 5.7.30-0ubuntu0.16.04.1 [685 kB]
Get:2 https://mirrors.tuna.tsinghua.edu.cn/ubuntu xenial-updates/main amd64 zlib1g-dev amd64 1:1.2.8.dfsg-2ubuntu4.3 [167 kB]
Get:3 https://mirrors.tuna.tsinghua.edu.cn/ubuntu xenial-updates/main amd64 libssl-dev amd64 1.0.2g-1ubuntu4.16 [1,344 kB]
Get:4 https://mirrors.tuna.tsinghua.edu.cn/ubuntu xenial-updates/main amd64 libmysqlclient-dev amd64 5.7.30-0ubuntu0.16.04.1 [986 kB]
Get:5 https://mirrors.tuna.tsinghua.edu.cn/ubuntu xenial-updates/main amd64 libssl-doc all 1.0.2g-1ubuntu4.16 [1,078 kB]
Fetched 4,260 kB in 2s (1,654 kB/s)
Preconfiguring packages ...
(Reading database ... 214443 files and directories currently installed.)
Preparing to unpack .../zlib1g_1%3a1.2.8.dfsg-2ubuntu4.3_amd64.deb ...
Unpacking zlib1g:amd64 (1:1.2.8.dfsg-2ubuntu4.3) over (1:1.2.8.dfsg-2ubuntu4) ...
Processing triggers for libc-bin (2.23-0ubuntu3) ...
Setting up zlib1g:amd64 (1:1.2.8.dfsg-2ubuntu4.3) ...
Processing triggers for libc-bin (2.23-0ubuntu3) ...
(Reading database ... 214443 files and directories currently installed.)
Preparing to unpack .../libssl1.0.0_1.0.2g-1ubuntu4.16_amd64.deb ...
Unpacking libssl1.0.0:amd64 (1.0.2g-1ubuntu4.16) over (1.0.2g-1ubuntu4) ...
Selecting previously unselected package libmysqlclient20:amd64.
Preparing to unpack .../libmysqlclient20_5.7.30-0ubuntu0.16.04.1_amd64.deb ...
Unpacking libmysqlclient20:amd64 (5.7.30-0ubuntu0.16.04.1) ...
Selecting previously unselected package zlib1g-dev:amd64.
Preparing to unpack .../zlib1g-dev_1%3a1.2.8.dfsg-2ubuntu4.3_amd64.deb ...
Unpacking zlib1g-dev:amd64 (1:1.2.8.dfsg-2ubuntu4.3) ...
Selecting previously unselected package libssl-dev:amd64.
Preparing to unpack .../libssl-dev_1.0.2g-1ubuntu4.16_amd64.deb ...
Unpacking libssl-dev:amd64 (1.0.2g-1ubuntu4.16) ...
Selecting previously unselected package libmysqlclient-dev.
Preparing to unpack .../libmysqlclient-dev_5.7.30-0ubuntu0.16.04.1_amd64.deb ...
Unpacking libmysqlclient-dev (5.7.30-0ubuntu0.16.04.1) ...
Selecting previously unselected package libssl-doc.
Preparing to unpack .../libssl-doc_1.0.2g-1ubuntu4.16_all.deb ...
Unpacking libssl-doc (1.0.2g-1ubuntu4.16) ...
Processing triggers for libc-bin (2.23-0ubuntu3) ...
Processing triggers for man-db (2.7.5-1) ...
Setting up libssl1.0.0:amd64 (1.0.2g-1ubuntu4.16) ...
Setting up libmysqlclient20:amd64 (5.7.30-0ubuntu0.16.04.1) ...
Setting up zlib1g-dev:amd64 (1:1.2.8.dfsg-2ubuntu4.3) ...
Setting up libssl-dev:amd64 (1.0.2g-1ubuntu4.16) ...
Setting up libmysqlclient-dev (5.7.30-0ubuntu0.16.04.1) ...
Setting up libssl-doc (1.0.2g-1ubuntu4.16) ...
Processing triggers for libc-bin (2.23-0ubuntu3) ...
```

> apt-get install libssl-dev

```bash
Reading package lists... Done
Building dependency tree       
Reading state information... Done
libssl-dev is already the newest version (1.0.2g-1ubuntu4.16).
libssl-dev set to manually installed.
0 upgraded, 0 newly installed, 0 to remove and 721 not upgraded.
```

- ### 执行

>  pip install mysqlclient \-i https://pypi.douban.com/simple

 成功安装！

![](https://img-blog.csdnimg.cn/20200713115437135.png)