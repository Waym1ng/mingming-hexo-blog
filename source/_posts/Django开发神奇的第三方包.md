---
title: Django开发神奇的第三方包
date: 2020-06-19 16:53:57
tags: python
categories: Django
---

<!--more-->

 

## 1\. Python social auth

一款社交账号认证/注册机制，支持Django、Flask、Webpy等在内的多个开发框架，提供了约50多个服务商的授权认证支持，如Google、Twitter、新浪微博等站点，配置简单。

GitHub 地址：[pennersr/django-allauth](https://link.zhihu.com/?target=https%3A//github.com/pennersr/django-allauth)

文档地址：[Welcome to django-allauth\!](https://link.zhihu.com/?target=https%3A//django-allauth.readthedocs.io/en/latest/)

点评：增强 Django 内置的 django.contrib.auth 模块，提供登录、注册、邮件验证、找回密码等一切用户验证相关的功能。另外还提供 OAuth 第三方登录功能，例如国内的微博、微信登录，国外的 GitHub、Google、facebook 登录等，几乎囊括了大部分热门的第三方账户登录。配置简单，开箱即用。

```
pip install python-social-auth
```

## 2\. Django Guardian

Django默认没有提供对象（Object）级别的权限控制，我们可以通过该扩展来帮助Django实现对象级别的权限控制。

```
pip install django-guardian
```

## 3\. Django OAuth Toolkit

可以帮助Django项目实现数据、逻辑的OAuth2功能，可与Django REST框架完美整合起来。

```
pip install django-oauth-toolkit
```

## 4\. django-allauth

可用于账号注册、管理和第三方社交账号的认证。

django-allauth 是一个能够解决你的注册和认证需求的、可重用的 Django 应用。无论你需要构建本地注册系统还是社交账户注册系统，django-allauth 都能够帮你做到。

这个应用支持多种认证体系，比如用户名或电子邮件。一旦用户注册成功，它还可以提供从无需认证到电子邮件认证的多种账户验证的策略。同时，它也支持多种社交账户和电子邮件账户。它还支持插拔式注册表单，可让用户在注册时回答一些附加问题。

django-allauth 支持多于 20 种认证提供者，包括 Facebook、Google、微博 和 微信。如果你发现了一个它不支持的社交网站，很有可能通过第三方插件提供该网站的接入支持。这个项目还支持自定义后端，可以支持自定义的认证方式，对每个有定制认证需求的人来说这都很棒。

django-allauth 易于配置，且有完善的文档。该项目通过了很多测试，所以你可以相信它的所有部件都会正常运作。

```
pip install django-allauth
```

## 5\. Celery

用来管理异步、分布式的消息作业队列，可用于生产系统来处理百万级别的任务。

django-celery是django web开发中执行异步任务或定时任务的最佳选择。它的应用场景包括:

 -    异步任务: 当用户触发一个动作需要较长时间来执行完成时，可以把它作为任务交给celery异步执行，执行完再返回给用户。这点和你在前端使用ajax实现异步加载有异曲同工之妙。
 -    定时任务。假设有多台服务器，多个任务，定时任务的管理是很困难的，你要在不同电脑上写不同的crontab，而且还不好管理。Celery可以帮助我们快速在不同的机器设定不同任务。
 -    其他可以异步执行的任务。比如发送短信，邮件，推送消息，清理/设置缓存等。这点还是比较有用的。

```
pip install Celery
```

## 6\. Django REST 框架

构建REST API的优秀框架，可管理内容协商、序列化、分页等，开发者可以在浏览器中浏览构建的API。

REST API 正在迅速成为现代 Web 应用的标准功能。 API 就是简单的使用 JSON 对话而不是 HTML，当然你可以只用 Django 做到这些。你可以制作自己的视图，设置合适的 Content-Type，然后返回 JSON 而不是渲染后的 HTML 响应。这是在像 Django Rest Framework（下称 DRF）这样的 API 框架发布之前，大多数人所做的。

如果你对 Django 的视图类很熟悉，你会觉得使用 DRF 构建 REST API 与使用它们很相似，不过 DRF 只针对特定 API 使用场景而设计。一般的 API 设置只需要一点代码，所以我们没有提供一份让你兴奋的示例代码，而是强调了一些可以让你生活的更舒适的 DRF 特性：

 -    可自动预览的 API 可以使你的开发和人工测试轻而易举。你可以查看 DRF 的示例代码。你可以查看 API 响应，并且不需要你做任何事就可以支持 POST/PUT/DELETE 类型的操作。
 -    便于集成各种认证方式，如 OAuth, Basic Auth, 或API Tokens。
 -    内建请求速率限制。
 -    当与 django-rest-swagger 组合使用时，API 文档几乎可以自动生成。
 -    广泛的第三方库生态。

```
pip install djangorestframework
```

## 7\. Django stored messages

可以很好地集成在Django的消息框架中（django.contrib.messages）并让用户决定会话过程中存储在数据库中的消息。

## 8\. django-cors-headers

一款设置CORS（Cross-Origin Resource Sharing）标头的应用，基于XmlHttpRequest，对管理Django应用中的跨域请求非常有帮助。

```
pip install django-cors-headers
```

## 9\. Debug toolbar

可在设置面板显示当前请求/响应的各种调试信息。除了本身提供的操作面板外，还有来自社区的多个第三方面板。

该工具给django web开发提供了强大的调试功能，包括查看执行的sql语句，db查询次数，request，headers，调试概览等。 通过安装插件Pympler，你还可以了解内存使用情况。

```
pip install django-debug-toolbar
```

**静态资源**

## 10\. Django Storages

可使静态资源方便地存储在外部服务上。安装后只需运行“python [manage.py](https://link.zhihu.com/?target=http%3A//manage.py) collectstatic”命令就可以将全部改动的静态文件复制到选定的后端。可结合库“python-boto”一起使用，将静态文件存储到Amazon S3上。

```
pip install django-storages
```

## 11\. Django Pipeline

静态资源管理应用，支持连接和压缩CSS/Javascript文件、支持CSS和Javascript的多种编译器、内嵌JavaScript模板，可充分允许自定义。

```
pip install django-pipeline
```

## 12\. Django Compressor

可将页面中链接的以及直接编写的JavaScript和CSS打包到一个单一的缓存文件中，以减少页面对服务器的请求数，加快页面的加载速度。

```
pip install django_compressor
```

## 13\. Reversion

为模型提供版本控制功能，稍微配置后，就可以恢复已经删除的模型或回滚到模型历史中的任何一点。最新版本支持Django 1.6。

```
pip install django-reversion
```

## 14\. Django extensions

Django框架的扩展功能集合，包括management命令扩展、数据库字段扩展、admin后台扩展等。

```
pip install django-extensions
```

## 15\. Django braces

是一系列可复用的行为、视图模型、表格和其他组件的合集。

```
pip install django-braces
```

## 16.django-haystack - 全文检索引擎

全文检索不同于标题的简单匹配，是一件技术难度比较高的活。当文章很长时，你很难找到精确的匹配，同时搜索全文需要消耗大量的计算资源。有了haystack，你可以直接django中直接添加搜索功能，像搜索标题一样搜索全文，而无需关注索引建立、搜索解析等技术问题。haystack支持多种搜索引擎，不仅仅是whoosh，使用solr、elastic search等搜索，也可通过haystack，而且直接切换引擎即可，甚至无需修改搜索代码。

GitHub 地址：[Welcome to Haystack\!](https://link.zhihu.com/?target=https%3A//django-haystack.readthedocs.io/en/master/)

文档地址：[django-haystack/django-haystack](https://link.zhihu.com/?target=https%3A//github.com/django-haystack/django-haystack)

## 17.django-ckeditor - 富文本编辑器

django没有提供官方的富文本编辑器，而ckeditor恰好是内容型网站后台管理中不可或缺的控件。ckeditor是一款基于javascript，使用非常广泛的开源网页编辑器。它允许用户直接编写图文，插入列表和表格，并支持文本和HTML格式代码输入。

GitHub 地址：[django-ckeditor/django-ckeditor](https://link.zhihu.com/?target=https%3A//github.com/django-ckeditor/django-ckeditor)

## 18.django-imagekit - 自动化处理图像

现代网站开发一般免不了处理一些图片，例如头像、用户上传的图片等内容。django-imagekit 帮你配合 django 的 model 模块自动完成图片的裁剪、压缩、生成缩略图、加水印等一系列图片相关的操作。

GitHub 地址：[matthewwithanm/django-imagekit](https://link.zhihu.com/?target=https%3A//github.com/matthewwithanm/django-imagekit)

文档地址：[Installation \- ImageKit 3.2.6 documentation](https://link.zhihu.com/?target=http%3A//django-imagekit.rtfd.org/)

## 19.django-xadmin - 更美观更强大的后台

如果你不喜欢django自带后台admin简陋的样式，你可以使用xadmin。xadmin是基于bootstrap和admin的一个更强大的后台管理系统。应该会给有强迫症的你带来惊喜。

GitHub 地址：[sshwsfc/xadmin](https://link.zhihu.com/?target=https%3A//github.com/sshwsfc/xadmin)

文档地址：[Welcome to Django Xadmin’s Documentation\!](https://link.zhihu.com/?target=https%3A//xadmin.readthedocs.io/en/docs-chinese/)

## 20.django-constance - 常量管理

有时我们会在 django 的 settings 中设置一些常量，但是有可能会进行变更。利用这个包，只需简单的配置就可以自动生成 admin 管理后台可以修改管理常量。

Django 的好处就是大而全，不仅内置了 ORM、表单、模板引擎、用户系统等，而且第三方应用的生态也是十分完善，开发中大部分常见的功能都能找到对应的第三方实现。在这里给大家推荐 10 个十分优秀的 Django 第三方库（GitHub 星星数基本都在 1000 以上，而且都在持续维护与更新中）。虽然这些库很适合用于社交网站的开发，但也有很大一部分是通用的，可以用于任何用 Django 开发的项目。使用这些库将大大提高开发效率和生产力。

## 21.django-model-utils

简介：增强 Django 的 model 模块。内置了一些通用的 model Mixin，例如 TimeStampedModel 为模型提供一个创建时间和修改时间的字段，还有一些有用的 Field，几乎每个 Django 项目都能用得上。

GitHub 地址：[jazzband/django-model-utils](https://link.zhihu.com/?target=https%3A//github.com/jazzband/django-model-utils)

文档地址：[django-model-utils \- django-model-utils 3.2.0 documentation](https://link.zhihu.com/?target=http%3A//django-model-utils.readthedocs.io/en/latest/)

## 22.django-crispy-forms

简介：大大增强 Django 内置的表单功能，Django 内置的表单生成原生的 HTML 表单代码还可以，但为其设置样式是一个麻烦的事情。django-crispy-forms 帮助你使用一行代码渲染一个 Bootstrap 样式的表单，当然它还支持其它一些热门的 CSS 框架样式的渲染。

GitHub 地址：[django-crispy-forms/django-crispy-forms](https://link.zhihu.com/?target=https%3A//github.com/django-crispy-forms/django-crispy-forms)

文档地址：[Forms have never been this crispy](https://link.zhihu.com/?target=http%3A//django-crispy-forms.rtfd.org/)

## 23.django-mptt

简介：配合 Django 的 ORM 系统，为数据库的记录生成树形结构，并提供便捷的操作树型记录的 API。例如可以使用它实现一个多级的评论系统。总之，只要你的数据结构可能需要使用树来表示，django-mptt 将大大提高你的开发效率。

GitHub 地址：[django-mptt/django-mptt](https://link.zhihu.com/?target=https%3A//github.com/django-mptt/django-mptt)

文档地址：[Django MPTT documentation](https://link.zhihu.com/?target=https%3A//django-mptt.readthedocs.io/)

## 24.django-contrib-comments

简介：用于提供评论功能，最先集成在 django 的 contrib 内置库里，后来被移出来单独维护。这个评论库提供了基本的评论功能，但是只支持单级评论。好在这个库具有很好的拓展性，基于上边提到的 django-mptt，就可以构建一个支持层级评论的评论库。

GitHub 地址：[django/django-contrib-comments](https://link.zhihu.com/?target=https%3A//github.com/django/django-contrib-comments)

文档地址：[Django “excontrib” Comments](https://link.zhihu.com/?target=https%3A//django-contrib-comments.readthedocs.io/)

## 25.django-brace

简介：django 内置的 class based view 很 awesome，但还有一些通用的类视图没有包含在 django 源码中，这个库补充了更多常用的类视图。类视图是 django 的一个很重要也很优雅的特性，使用类视图可以减少视图函数的代码编写量、提高视图函数的代码复用性等。

GitHub 地址：[brack3t/django-braces](https://link.zhihu.com/?target=https%3A//github.com/brack3t/django-braces)

文档地址：[Welcome to django-braces’s documentation\!](https://link.zhihu.com/?target=http%3A//django-braces.readthedocs.io/en/latest/index.html)

点评：深入学习类视图可以看Django类视图源码分析。

## 26.django-notifications-hq

简介：为你的网站提供类似于 GitHub 这样的通知功能。未读通知数、通知列表、标为已读等等。

GitHub 地址：[django-notifications/django-notifications](https://link.zhihu.com/?target=https%3A//github.com/django-notifications/django-notifications)

文档地址：[django-notifications-hq](https://link.zhihu.com/?target=https%3A//pypi.python.org/pypi/django-notifications-hq/)

## 27.django-simple-captcha

简介：配合 django 的表单模块，方便地为表单添加一个验证码字段。对验证性要求不高的需求，例如注册表单防止机器人自动注册等使用起来非常方便。

GitHub 地址：[mbi/django-simple-captcha](https://link.zhihu.com/?target=https%3A//github.com/mbi/django-simple-captcha)

文档地址：[Django Simple Captcha](https://link.zhihu.com/?target=http%3A//django-simple-captcha.readthedocs.io/en/latest/)

## 28.django-anymail

简介：配合 django 的 email 模块，只需简单配置，就可以使用 Mailgun、SendGrid 等发送邮件。

GitHub 地址：[anymail/django-anymail](https://link.zhihu.com/?target=https%3A//github.com/anymail/django-anymail)

文档地址：[Anymail: Django email integration for transactional ESPs](https://link.zhihu.com/?target=https%3A//anymail.readthedocs.io/)

## 29.django-activity-stream

简介：社交类网站免不了关注、收藏、点赞、用户动态等功能，这一个 app 全搞定。甚至用它实现一个朋友圈也不是不可能。

GitHub 地址：[justquick/django-activity-stream](https://link.zhihu.com/?target=https%3A//github.com/justquick/django-activity-stream)

文档地址：[Django Activity Stream Documentation](https://link.zhihu.com/?target=http%3A//django-activity-stream.rtfd.io/en/latest/)

## 30.Datatables

是一款jquery表格插件。它是一个高度灵活的工具，可以将任何HTML表格添加高级的交互功能。

官网：[Table plug-in for jQuery](https://link.zhihu.com/?target=https%3A//datatables.net/) 中文网站：[Datatables 中文网](https://link.zhihu.com/?target=http%3A//datatables.club/)

 

链接：<https://zhuanlan.zhihu.com/p/76255269>  
来源：知乎