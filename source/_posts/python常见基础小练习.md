---
title: python常见基础小练习
date: 2019-02-20 19:38:19
tags: 
categories: python加油鸭
---

<!--more-->

## python常见基础小练习（新手学习，有错请谅解）

1.输入一个年份，判断是否为闰年。

条件1：不能被100整除且能被4整除

条件2：被400整除【世纪年】

```
year = int(input("请输入一个年份:"))
if year %4 == 0 and year %100 != 0:
    print("%d年是闰年"%year)
elif year %400 == 0:
    print("%d年是闰年" % year)
else:
    print("%d年不是闰年"%year)
```

2.输入一位三位数，判断是否为水仙花数153  
153 = 13+53+3\^3

```
num = int(input("请输入一个三位数:\n"))
bai = int(num/100)
shi = int((num-bai*100)/10)
ge = int(num - bai*100 - shi*10)
print(bai,shi,ge)
if num == pow(bai,3) + pow(shi,3) +pow(ge,3):
    print("%d是一个水仙花数"%num)
else:
    print("%d不是一个水仙花数"%num)
```

4.摇色子，

提示:押大还押小 ： 大 或者 小

开始摇色子，

【1\~6】取值【1， 2， 3】 小

取值【4， 5， 6】大

若押中，则打印“”庄家喝酒。。。。“

若没押中，则打印”先干为敬。。。“

```
import random
guess = input("请押注：大or小 \n")
num = random.choice(range(1,7))
print(num)
if num <=3:
    guess1 = '小'
else:
    guess1 ='大'
if guess1 == guess:
    print("庄家喝酒！")
else:
    print("先干为敬！")
```