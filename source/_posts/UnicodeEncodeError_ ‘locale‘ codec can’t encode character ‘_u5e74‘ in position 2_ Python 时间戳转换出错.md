---
title: UnicodeEncodeError_ ‘locale‘ codec can’t encode character ‘_u5e74‘ in position 2_ Python 时间戳转换出错
date: 2022-07-22 21:39:11
tags:
    - 基础
categories:
    - python
---
- 当我们想将时间戳转换成特定格式的时间字符串，比如带有年月日，以下写法可能会出现报错

```python
datetime.strftime(datetime.fromtimestamp(1655481600), '%Y年%m月%d日 %H:%M:%S')
```

> UnicodeEncodeError: 'locale' codec can't encode character '\u5e74' in position 2: encoding error

- 解决方案

```python
from datetime import datetime
datetime_str = datetime.strftime(datetime.fromtimestamp(1655481600), '%Yn%my%dr %H:%M:%S').replace('n','年').replace('y', '月').replace('r', '日')
print(datetime_str) # '2022年06月18日 00:00:00'
datetime_str2 = datetime.strftime(datetime.fromtimestamp(1655481600), '%Y{y}%m{m}%d{d} %H:%M:%S').format(y='年', m='月', d='日')
print(datetime_str2) # '2022年06月18日 00:00:00'
```

