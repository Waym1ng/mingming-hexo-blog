---
title: Celery 的用法介绍
date: 2020-08-18 17:35:35
tags: python 数据库
categories: python加油鸭
---

<!--more-->

## **celery介绍**

### **什么是celery**

**这次我们来介绍一下Python的一个第三方模块celery，那么celery是什么呢？**

- `celery是一个灵活且可靠的，处理大量消息的分布式系统，可以在多个节点之间处理某个任务。`
- `celery是一个专注于实时处理的任务队列，支持任务调度。`
- `celery是开源的，有很多使用者。`
- `celery完全基于Python语言编写。`

**所以celery是一个任务调度框架，类似于Apache的airflow，当然airflow也是基于Python语言编写。不过有一点需要注意，celery是用来调度任务的，它本身并不具备任务的存储功能，而我们说在调度任务的时候肯定是要把任务存起来的，因此在使用celery的时候还需要搭配一些具备存储、访问功能的工具，比如：消息队列、Redis缓存、数据库等等。官方推荐的是消息队列RabbitMQ，个人认为有些时候使用Redis也是不错的选择，当然我们都会介绍。**

**那么celery都可以在哪些场景中使用呢？**

- `异步任务：一些耗时的操作可以交给celery异步执行，而不用等着程序处理完才知道结果。比如：视频转码、邮件发送、消息推送等等。`
- `定时任务：比如定时推送消息、定时爬取数据、定时统计数据等等`

参考：[这是一篇很详细的celery用法介绍](https://www.cnblogs.com/traditional/p/11788756.html)