---
title: Python 各种进制相互转换 16进制转换成2进制 不够用0补齐 前面补0
date: 2020-03-27 15:18:15
tags: 字符串 python
categories: python加油鸭
---

<!--more-->

<table border="2" cellpadding="2" cellspacing="2"><tbody><tr><td>&nbsp;</td><td>2进制</td><td>8进制</td><td>10进制</td><td>16进制</td></tr><tr><td>2进制</td><td>-</td><td>bin(int(x, 8))</td><td>bin(int(x, 10))</td><td>bin(int(x, 16))</td></tr><tr><td>8进制</td><td>oct(int(x, 2))</td><td>-</td><td>oct(int(x, 10))</td><td>oct(int(x, 16))</td></tr><tr><td>10进制</td><td>int(x, 2)</td><td>int(x, 8)</td><td>-</td><td>int(x, 16)</td></tr><tr><td>16进制</td><td>hex(int(x, 2))</td><td>hex(int(x, 8))</td><td>hex(int(x, 10))</td><td>-</td></tr></tbody></table>

bin\(\)、oct\(\)、hex\(\)的返回值均为字符串，且分别带有0b、0o、0x前缀。

比如：2进制转换成8进制 执行

```
oct(int(x, 2))  
```

以此类推

还可以通过format方法来实现

10进行十进制，十六进制，八进制，二进制的转换：  
\(#：保留进制前缀\)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190518151155582.png)

 16进制转换成2进制 想保留8位，前面用0补齐，可以采用以下方法：

```
state_16 = "1C"
state_10 = int(state_16,16)
state_2 = '{:08b}'.format(state_10)
```

 ![](https://img-blog.csdnimg.cn/20200327153344141.png)