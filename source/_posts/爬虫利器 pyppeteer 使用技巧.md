## 一、拦截器简单用法

拦截器作用于单个Page，即浏览器中的一个标签页。每初始化一个Page都要添加一下拦截器。拦截器实际上是

通过给各种事件添加回调函数来实现的。

事件列表可参见：pyppeteer.page.Page.Events

常用拦截器：

- request：发出网络请求时触发
- response：收到网络响应时触发
- dialog：页面有弹窗时触发

使用request拦截器修改请求：

```python
# coding:utf8
import asyncio
from pyppeteer import launch

from pyppeteer.network_manager import Request


launch_args = {
    "headless": False,
    "args": [
        "--start-maximized",
        "--no-sandbox",
        "--disable-infobars",
        "--ignore-certificate-errors",
        "--log-level=3",
        "--enable-extensions",
        "--window-size=1920,1080",
        "--user-agent=Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.97 Safari/537.36",
    ],
}


async def modify_url(request: Request):
    if request.url == "https://www.baidu.com/":
        await request.continue_({"url": "https://www.baidu.com/s?wd=ip&ie=utf-8"})
    else:
        await request.continue_()


async def interception_test():
    # 启动浏览器
    browser = await launch(**launch_args)
    # 新建标签页
    page = await browser.newPage()
    # 设置页面打开超时时间
    page.setDefaultNavigationTimeout(10 * 1000)
    # 设置窗口大小
    await page.setViewport({"width": 1920, "height": 1040})

    # 启用拦截器
    await page.setRequestInterception(True)

    # 设置拦截器
    # 1. 修改请求的url
    if 1:
        page.on("request", modify_url)
        await page.goto("https://www.baidu.com")

    await asyncio.sleep(10)

    # 关闭浏览器
    await page.close()
    await browser.close()
    return


if __name__ == "__main__":
    loop = asyncio.get_event_loop()
    loop.run_until_complete(interception_test())
```

使用response拦截器获取某个请求的响应：

```python
async def get_content(response: Response):
    """
        # 注意这里不需要设置 page.setRequestInterception(True)
        page.on("response", get_content)
    :param response:
    :return:
    """
    if response.url == "https://www.baidu.com/":
        content = await response.text()
        title = re.search(b"<title>(.*?)</title>", content)
        print(title.group(1))
```

清除页面所有弹窗:

```python
async def handle_dialog(dialog: Dialog):
    """
        page.on("dialog", get_content)
    :param dialog: 
    :return: 
    """
    await dialog.dismiss()
```

## 二、拦截器实现切换代理

一般情况下浏览器添加代理的方法为设置启动参数：

> --proxy-server=http://user:password@ip:port

例如：

```
launch_args = {
    "headless": False,
    "args": [        "--proxy-server=http://localhost:1080",
        "--user-agent=Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.97 Safari/537.36",
    ],
}
```

但此种方式的缺点很明显，只能在浏览器启动时设置。当需要切换代理时，只能重启浏览器，这个代价

就太高了，所以我们可以想想其他办法。

思路很简单：

- request拦截器可以修改请求属性并且返回自定义响应内容
- 使用第三方库来发送网络请求，并设置代理。然后封装响应内容返回给浏览器

```python
import aiohttp

aiohttp_session = aiohttp.ClientSession(loop=asyncio.get_event_loop())

proxy = "http://127.0.0.1:1080"
async def use_proxy_base(request: Request):
    """
        # 启用拦截器
        await page.setRequestInterception(True)
        page.on("request", use_proxy_base)
    :param request:
    :return:
    """
    # 构造请求并添加代理
    req = {
        "headers": request.headers,
        "data": request.postData,
        "proxy": proxy,  # 使用全局变量 则可随意切换
        "timeout": 5,
        "ssl": False,
    }
    try:
        # 使用第三方库获取响应
        async with aiohttp_session.request(
            method=request.method, url=request.url, **req
        ) as response:
            body = await response.read()
    except Exception as e:
        await request.abort()
        return

    # 数据返回给浏览器
    resp = {"body": body, "headers": response.headers, "status": response.status}
    await request.respond(resp)
    return	
```

或者再增加一些缓存来节约一下带宽：

```python
# 静态资源缓存
static_cache = {}
async def use_proxy_and_cache(request: Request):
    """
        # 启用拦截器
        await page.setRequestInterception(True)
        page.on("request", use_proxy_base)
    :param request:
    :return:
    """
    global static_cache
    if request.url not in static_cache:
        # 构造请求并添加代理
        req = {
            "headers": request.headers,
            "data": request.postData,
            "proxy": proxy,  # 使用全局变量 则可随意切换
            "timeout": 5,
            "ssl": False,
        }
        try:
            # 使用第三方库获取响应
            async with aiohttp_session.request(
                method=request.method, url=request.url, **req
            ) as response:
                body = await response.read()
        except Exception as e:
            await request.abort()
            return

        # 数据返回给浏览器
        resp = {"body": body, "headers": response.headers, "status": response.status}
        # 判断数据类型 如果是静态文件则缓存起来
        content_type = response.headers.get("Content-Type")
        if content_type and ("javascript" in content_type or "/css" in content_type):
            static_cache[request.url] = resp
    else:
        resp = static_cache[request.url]

    await request.respond(resp)
    return
```

## 最后

参考：[pyppeteer进阶技巧](https://www.cnblogs.com/dyfblog/p/10887940.html)