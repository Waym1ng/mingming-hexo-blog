---
title: 新手学习python基础：常用数学函数
date: 2019-02-20 19:28:40
tags: 
categories: python加油鸭
---

<!--more-->

新手学习python基础：常用数学函数

import math,random  
print\(\)  
‘’’  
1.数字类型之间的转换 int\(\) float\(\) str\(\)  
2.常用的数学函数  
abs\(\) 返回绝对值  
max\(\) 返回最大值  
min\(\) 返回最小值  
pow\(x,y\) 求x的y次方的值  
round\(x,\[n\]\) 返回浮点数的四舍五入值，n代表小数点后的位数，python3中向偶数靠拢

‘’’

‘’’  
导入math模块 import math

math.ceil\(x\):返回x的向上取整数值

math.floor\(x\):返回x的向下取整的数值

math.modf\(x\):返回x的整数部分和小数部分，两部分的数值符号与x相同，整数部分以浮点数表示。

math.sqrt\(x\):反回数字的x的开平方根，返回类型为正数【浮点型】  
‘’’

```
# print(round(2.555))
# print(round(2.5))
# print(round(3.555))
# print(round(3.5))
# print(round(2.555555, 5))
# print(round(3.555555, 5))
# print(round(2.55574, 4))
# print(round(3.55555, 4))
# print(round(2.5555, 3))
# print(round(3.5555, 3))
# print(round(2.555, 2))
# print(round(3.555, 2))
# print(round(2.55, 1))
# print(round(3.55, 1))
# print(round(2.5))
# print(round(3.5))
```

‘’’  
导入random模块

random.choice\(\[1,2,3,4\]\) ：随机返回一个元素【从指定序列中挑选一个元素】

random.randrange\(n\):从0\~n-1之间选择一个随机数

random.random\(\) :随机产生\[0,1\)之间的数，结果为浮点数

l1 = \[1, 2, 4, 5\]

random.shuffle\(l1\) :将序列中的所有元素进行随机排列

random.uniform\(m, n\) :随机产生一个\[m, n\]之间的浮点数  
‘’’

```
print(random.choice(['a','s','d','f','g',1,2,3,4]))
print(random.choice(range(100)))
print(random.random())
print(int(random.random()*100+1))
```