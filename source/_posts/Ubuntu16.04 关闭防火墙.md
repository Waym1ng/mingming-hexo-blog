---
title: Ubuntu16.04 关闭防火墙
date: 2020-04-17 16:32:18
tags: systemd
categories: Linux
---

<!--more-->

Ubuntu16.04 关闭防火墙](https://blog.csdn.net/weixin_42474540)

一、关闭防火墙

1\. 先查看防火墙状态

<table border="0" cellpadding="0" cellspacing="0"><tbody><tr><td><p>1</p></td><td><p><code>systemctl&nbsp;status&nbsp;firewalld</code></p></td></tr></tbody></table>

```
firewalld.service - firewalld - dynamic firewall daemon

   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled)

   Active: active (running) since

 Main PID: 979 (firewalld)

   CGroup: /system.slice/firewalld.service

           └─979 /usr/bin/python -Es /usr/sbin/firewalld --nofork --nopid

5月 25 22:53:54 localhost.localdomain systemd[1]: Started firewalld - dynami...

Hint: Some lines were ellipsized, use -l to show in full.
```

2\. 关闭防火墙

 

<table border="0" cellpadding="0" cellspacing="0"><tbody><tr><td><p>1</p></td><td><p><code>systemctl&nbsp;stop&nbsp;firewalld</code></p></td></tr></tbody></table>


<table border="0" cellpadding="0" cellspacing="0"><tbody><tr><td><p>1</p></td><td><p><code>systemctl&nbsp;status&nbsp;firewalld</code></p></td></tr></tbody></table>


3\. 查看防火墙服务是否开机启动

 

<table border="0" cellpadding="0" cellspacing="0"><tbody><tr><td><p>1</p></td><td><p><code>systemctl&nbsp;is-enabled&nbsp;firewalld</code></p></td></tr></tbody></table>

enabled  #开启

4\. 关闭防火墙开机启动

 

<table border="0" cellpadding="0" cellspacing="0"><tbody><tr><td><p>1</p></td><td><p><code>systemctl&nbsp;disable&nbsp;firewalld</code></p></td></tr></tbody></table>

```
rm '/etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service'

rm '/etc/systemd/system/basic.target.wants/firewalld.service'
```

 

<table border="0" cellpadding="0" cellspacing="0"><tbody><tr><td><p>1</p></td><td><p><code>systemctl&nbsp;is-enabled&nbsp;firewalld</code></p></td></tr></tbody></table>

disabled