---
title: python小项目 用函数写一个简单的ATM系统（满足登陆注册查询转账改密冻结解锁的功能）
date: 2019-03-02 11:50:33
tags: python函数 python基础 python小练习
categories: python加油鸭
---

<!--more-->

# 简单的ATM系统

使用函数调用的方法，适合初学者学习参考

```
import random

display='''
**********************
*                    *
*  welcome to bank   *
*                    *
**********************
'''
display2='''
**********************
*  1.登陆   2.开户    *
*  3.查询   4.取款    *
*  5.存款   6.退出    *
*  7.转账   8.改密    *
*  9.锁卡   0.解锁   *
**********************
'''
def getkamun(userdict):
    userlist = list(userdict)
    kanum = userdict.keys()
    while 1:
        kalist =[]
        while len(kalist)<3:
            i = random.randrange(10)
            if str(i) not in kalist:
                kalist.append(str(i))
        kanum="".join(kalist)
        if kanum in userlist:
            continue
        else:
            break
    return kanum

def register(userdict):
    idcard =input("身份证号：")
    name = input("用户名：")
    phone =input("电话号码：")
    money=int(input("预存款："))
    while 1:
        psd1 =input("密码：")
        psd2 =input("确认密码：")
        if psd1==psd2:
            psd=psd2
            break
        else:
            print("密码不一致，重新输入")
    kanum=getkamun(userdict)
    user ={'idcard':idcard,'name':name,'phone':phone,'money':money,'psd':psd,'kanum':kanum,'suo':False}
    return user

def login(userdict):
    usernum = input("请输入卡号：")
    user =userdict.get(usernum)
    if user==None:
        print("卡号不存在")
        return
    else:
        if user['suo']:
            print("已锁定，请解锁后登陆")
            return
        for i in range(4):
            psd = input("请输入密码：")
            if usernum ==user.get('kanum') and psd ==user.get('psd'):
                print("登陆成功")
                return user.get('kanum')
            else:
                print("登陆失败,还有%d次机会"%i)
                continue
        else:
            user['suo']=True
            print("登陆次数超过三次,已锁定")

def refer(kanum):
    user = userdict.get(kanum)
    money = user['money']
    print("当前余额为%d"%money)
    return None

def fund(kanum):
    user = userdict.get(kanum)
    money=user['money']
    money += eval(input("输入金额"))
    user['money']=money
    print("成功！当前余额为%d" % money)
    return None

def draw(kanum):
    user = userdict.get(kanum)
    money=user['money']
    while 1:
        money -= eval(input("输入金额"))
        if money>=0:
            user['money']=money
            print("成功！当前余额为%d"%money)
            break
        else:
            print("输入大于余额，重新输入")
            break
    return None

def tran(kanum,userdict):
    user = userdict[kanum]
    print(user)
    obj = input("请输入卡号：")
    if obj not in userdict:
        print("卡号不存在")
    else:
        money2 = int(input("请输入转账金额："))
        if money2<user['money']:
            user['money'] -= money2
            userdict.get(obj)['money'] +=money2
            print("转账成功，余额为%d"%user['money'])
            return
        else:
            print("余额不足")

def changepsd(kanum,userdict):
    user = userdict[kanum]
    oldpsd = input("请输入当前密码：")
    if oldpsd == user['psd']:
        newpsd = input("请输入新密码：")
        user['psd'] = newpsd
        print("修改成功")
    else:
        print("密码错误")

def lock(kanum,userdict):
    kanum = input("请输入您的卡号：")
    if kanum in userdict:
        idcard = input("请输入身份证号码：")
        name = input("请输入姓名：")
        phone = input("请输入电话：")
        user = userdict[kanum]
        if idcard==user['idcard'] and name==user['name'] and phone==user['phone']:
            user['suo']=True
            print("锁定成功")
            return
        else:
            print("信息有误")
    else:
        print("卡号不存在")
        return

def unlock(userdict):
    kanum = input("请输入卡号：")
    if kanum in userdict:
        idcard = input("请输入身份证：")
        name = input("请输入姓名：")
        phone = input("请输入电话：")
        user = userdict[kanum]
        if idcard==user['idcard'] and name==user['name'] and phone==user['phone']:
            user['suo']=False
            print("解锁成功")
            return
        else:
            print("信息有误")
    else:
        print("卡号不存在")
        return

def welcome():
    print(display)

welcome()
userdict={}
login_state = None
money =0
n = 0
while 1:
    print(display2)
    n =input("用户选择要操作：")
    if n=='1':
        login_state = login(userdict)
        print(userdict)
    elif n=='2':
        user=register(userdict)
        kanum=user.get('kanum')
        userdict[kanum]=user
        print('卡号为：%s'%kanum)
    elif n=='3':
        print(userdict)
        if login_state:
            refer(login_state)
        else:
            print("请先登陆")
    elif n=='4':
        if login_state:
            draw(login_state)
        else:
            print("请先登陆")
    elif n=='5':
        if login_state:
            fund(login_state)
        else:
            print("请先登陆")
    elif n=='6':
        login_state = None
        print('退出登陆')
    elif n=='7':
        if login_state:
            tran(login_state, userdict)
        else:
            print("请先登陆")
    elif n=='8':
        if login_state:
            changepsd(login_state, userdict)
        else:
            print("请先登陆")
    elif n=='9':
        n=lock(login_state, userdict)
    elif n=='0':
        unlock(userdict)
    else:
        print("输入有误，退出系统")
        break
```