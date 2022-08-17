---
title: 基于Python Flask 详解 OAuth 2.0 (以github为例)
date: 2020-04-16 17:03:07
tags: python jwt
categories: python加油鸭
---

<!--more-->

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcubWVpd2VuLmNvbS5jbi9pMTcwODkxMS8zNTBlODg5ZDQxYTc0YmM1LmpwZWc?x-oss-process=image/format,png)

_OAuth2流程图_

OAuth2 对于我来说是一个神秘的东西，我想初步的弄懂中间的整个流程，于是就去google搜索相关的文档资料。

在浏览了参差不齐的各种文章后，[简述 OAuth 2.0 的运作流程](https://www.barretlee.com/blog/2016/01/10/oauth2-introduce/) 基本对于小白来说是最浅显明了的。

这篇文章以用户使用 github 登录网站留言为例，详述 OAuth 2.0 的运作流程。

整个OAuth2 的流程分为三个阶段：

1.  网站和 Github 之间的协商
2.  用户和 Github 之间的协商
3.  网站和 Github 用户数据之间的协商

由于这篇文章是简述，所以并不涉及代码相关的东西，我在原来的文章基础上添加了代码相关的具体实现和一些关键网络交互截图说明方便理解。对于一些文字，由于原文已经写的很流畅严谨，我直接就从原来的博文中复制过来了。

---

假如我有一个网站，你是我网站上的访客，看了文章想留言表示「朕已阅」，留言时发现有这个网站的帐号才能够留言，此时给了你两个选择：一个是在我的网站上注册拥有一个新账户，然后用注册的用户名来留言；一个是使用 github 帐号登录，使用你的 github 用户名来留言。前者你觉得过于繁琐，于是惯性地点击了 github 登录按钮，此时 OAuth 认证流程就开始了。

需要明确的是，即使用户刚登录过 github，我的网站也不可能向 github 发一个什么请求便能够拿到访客信息，这显然是不安全的。就算用户允许你获取他在 github 上的信息，github 为了保障用户信息安全，也不会让你随意获取。所以操作之前，我的网站与 github 之间需要要有一个协商。

1\. 网站和 Github 之间的协商

Github 会对用户的权限做分类，比如读取仓库信息的权限、写入仓库的权限、读取用户信息的权限、修改用户信息的权限等等。如果我想获取用户的信息，Github 会要求我，先在它的平台上注册一个应用，在申请的时候标明需要获取用户信息的哪些权限，用多少就申请多少，并且在申请的时候填写你的网站域名，Github 只允许在这个域名中获取用户信息。

此时我的网站已经和 Github 之间达成了共识，Github 也给我发了两张门票，一张门票叫做 Client Id，另一张门票叫做 Client Secret。

我先去阅读了一下github上相关OAuth2的资料，然后在[这里](https://github.com/settings/developers)注册了一个应用。

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcubWVpd2VuLmNvbS5jbi9pMTcwODkxMS8wYWI5ZTRjMWYxMDBlZmI4LnBuZw?x-oss-process=image/format,png)![](https://img-blog.csdnimg.cn/20200416170217566.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjQ3NDU0MA==,size_16,color_FFFFFF,t_70)

其中最后一个callback URL表示用户授权之后github默认要跳转的url地址，在代码中需要添加一个路由来处理针对这个地址的请求。

创建好之后就会显示在OAuth Apps的列表中。

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcubWVpd2VuLmNvbS5jbi9pMTcwODkxMS80ZTA4MWY5OTkxYzZhNzNhLnBuZw?x-oss-process=image/format,png)

这一步非常简单，github生成了两个钥匙，Client ID和Client Secret。现在我的网站就可以使用合法的使用github提供的OAuth登陆机制了。

2\. 用户和 Github 之间的协商

用户进入我的网站，点击 github 登录按钮的时候，我的网站会把上面拿到的 Client Id 交给用户，让他进入到 Github 的授权页面，Github 看到了用户手中的门票，就知道这是我的网站让他过来的，于是它就把我的网站想要获取的权限摆出来，并询问用户是否允许我获取这些权限。

如果用户觉得我的网站要的权限太多，或者压根就不想我知道他这些信息，选择了拒绝的话，整个 OAuth 2.0 的认证就结束了，认证也以失败告终。如果用户觉得 OK，在授权页面点击了确认授权后，页面会跳转到我预先设定的 `redirect_uri` 并附带一个盖了章的门票 code。

这个时候，用户和 Github 之间的协商就已经完成，Github 也会在自己的系统中记录这次协商，表示该用户已经允许在我的网站访问上直接操作和使用他的部分资源。

这个中间会涉及到非常多的流程，我选择使用python基于flask来演示整个流程。

```python
# github生成的两把钥匙
client_id = '1f93ab8ba338b032b8e7'
client_secret = 'f0cf5600d2749d1651f2d5f7225c81f562******'


@app.route('/', methods=['GET', 'POST'])
def index():
    url = 'https://github.com/login/oauth/authorize'
    params = {
        'client_id': client_id,
        # 如果不填写redirect_uri那么默认跳转到oauth中配置的callback url。
        # 'redirect_uri': 'http://dig404.com/oauth2/github/callback',
        'scope': 'read:user',
        # 随机字符串，防止csrf攻击
        'state': 'An unguessable random string.',
        'allow_signup': 'true'
    }
    url = furl(url).set(params)
    return redirect(url, 302)
```

当用户在浏览器中访问127.0.0.1:5000的时候，flask会将请求重定向到github的oauth服务页面，重定向的url会携带上两个主要的参数，一个是client\_id，一个是scope，这两个参数可以让github知道这个请求是从哪里过来的，并且想要获取的权限。

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcubWVpd2VuLmNvbS5jbi9pMTcwODkxMS9lZWRjNzZmMjY0ODE1OWU0LnBuZw?x-oss-process=image/format,png)

访问125.0.0.1:5000后flask重定向到github授权页面![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcubWVpd2VuLmNvbS5jbi9pMTcwODkxMS84ZjM3NWVkN2VkZmNiNzJiLnBuZw?x-oss-process=image/format,png)

如果没有登陆github那么首先github会先跳转到用户的登陆页面![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcubWVpd2VuLmNvbS5jbi9pMTcwODkxMS8yYTExNDhjNjc0N2JmNDA4LnBuZw?x-oss-process=image/format,png)

用户登陆自己的github账号

![](https://img-blog.csdnimg.cn/20200416165941162.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjQ3NDU0MA==,size_16,color_FFFFFF,t_70)

登陆成功后跳转到授权页面

![](https://img-blog.csdnimg.cn/20200416165956689.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjQ3NDU0MA==,size_16,color_FFFFFF,t_70)

点击授权后github跳转到之前设置的callback页面

其中最终重定向url中的code参数就是github分配的针对当前登陆用户的授权码，也就是一张门票。在github的后台，这个code和client\_id，user是对应的。

到这里用户和github之间的协商就完成了，剩下的事情就是网站和github之间的事情了。

3\. 从 Github 获取用户的信息

第二步中，已经拿到了盖过章的门票 code，但这个 code 只能表明，用户允许我的网站从 github 上获取该用户的数据，如果我直接拿这个 code 去 github 访问数据一定会被拒绝，因为任何人都可以持有 code，github 并不知道 code 持有方就是我本人。

还记得之前申请应用的时候 github 给我的两张门票么，Client Id 在上一步中已经用过了，接下来轮到另一张门票 Client Secret。

创建一个处理callback路由的处理函数，首先是获取github返回的code。

```python
@app.route('/oauth2/<service>/callback')
def oauth2_callback(service):
    print(service)

    code = request.args.get('code')
    # 根据返回的code获取access token
    access_token_url = 'https://github.com/login/oauth/access_token'
    payload = {
        'client_id': client_id,
        'client_secret': client_secret,
        'code': code,
        # 'redirect_uri':
        'state': 'An unguessable random string.'
    }
    r = requests.post(access_token_url, json=payload, headers={'Accept': 'application/json'})
    access_token = json.loads(r.text).get('access_token')
    # 拿到access token之后就可以去读取用户的信息了
    access_user_url = 'https://api.github.com/user'
    r = requests.get(access_user_url, headers={'Authorization': 'token ' + access_token})
    return jsonify({
        'status': 'success',
        'data': json.loads(r.text)
    })
```

拿着用户盖过章的 code 和能够标识个人身份的 client\_id、client\_secret 去拜访 github，拿到最后的绿卡 access\_token。

有了access\_token之后就可以读取用户授权的信息了，最后为了演示我把读取到的信息回显到了网页上。  
这其中的过程对于用户来说不可见的，用户最终在浏览器中的url还是第二步重定向的url。

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcubWVpd2VuLmNvbS5jbi9pMTcwODkxMS80NzlhZTllYzRiMGZkMTY4LnBuZw?x-oss-process=image/format,png)

读取到的用户信息

拿到用户信息后其实就相当于用户已经登陆了，下一步就可以基于获取到的用户信息对用户做一些业务相关的处理了。

---

整个 OAuth2 流程在这里也基本完成了，文章中的表述很粗糙，比如 access\_token 这个绿卡是有过期时间的，如果过期了需要使用 refresh\_token 重新签证。重点是让读者理解整个流程，细节部分可以阅读 [RFC6749 文档](http://www.rfcreader.com/#rfc6749)。

希望对你理解 OAuth 2.0 有帮助。

转自：<https://www.meiwen.com.cn/subject/iqncpftx.html>