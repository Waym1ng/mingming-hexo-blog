---
title: 常见几种加密算法的Python实现
date: 2020-05-29 09:17:58
tags: python 加密解密 算法
categories: python加油鸭
---

<!--more-->

生活中我们经常会遇到一些加密算法，今天我们就聊聊这些加密算法的Python实现。部分常用的加密方法基本都有对应的Python库，基本不再需要我们用代码实现具体算法。

 

**一、MD5加密**

全称：MD5消息摘要算法（英语：MD5 Message-Digest Algorithm），一种被广泛使用的密码散列函数，可以产生出一个128位（16字节）的散列值（hash value），用于确保信息传输完整一致。md5加密算法是不可逆的，所以解密一般都是通过暴力穷举方法，通过网站的接口实现解密。

**Python代码：**

```
import hashlib
m = hashlib.md5()
m.update(str.encode("utf8"))
print(m.hexdigest())
```

 

**二、SHA1加密**

全称：安全哈希算法（Secure Hash Algorithm）主要适用于数字签名标准（Digital Signature Standard DSS）里面定义的数字签名算法（Digital Signature Algorithm DSA），SHA1比MD5的安全性更强。对于长度小于2\^ 64位的消息，SHA1会产生一个160位的消息摘要。

**Python代码：**

```
import hashlib
sha1 = hashlib.sha1()
data = '2333333'
sha1.update(data.encode('utf-8'))
sha1_data = sha1.hexdigest()
print(sha1_data)
```

 

**三、HMAC加密**

全称：散列消息鉴别码（Hash Message Authentication Code）， HMAC加密算法是一种安全的基于加密hash函数和共享密钥的消息认证协议。实现原理是用公开函数和密钥产生一个固定长度的值作为认证标识，用这个标识鉴别消息的完整性。使用一个密钥生成一个固定大小的小数据块，即 MAC，并将其加入到消息中，然后传输。接收方利用与发送方共享的密钥进行鉴别认证等。

**Python代码：**

```
import hmac
import hashlib
# 第一个参数是密钥key，第二个参数是待加密的字符串，第三个参数是hash函数
mac = hmac.new('key','hello',hashlib.md5)
mac.digest()  # 字符串的ascii格式
mac.hexdigest()  # 加密后字符串的十六进制格式
```

 

**四、DES加密**

全称：数据加密标准（Data Encryption Standard），属于对称加密算法。DES是一个分组加密算法，典型的DES以64位为分组对数据加密，加密和解密用的是同一个算法。它的密钥长度是56位（因为每个第8 位都用作奇偶校验），密钥可以是任意的56位的数，而且可以任意时候改变。

**Python代码：**

```
import binascii
from pyDes import des, CBC, PAD_PKCS5
# 需要安装 pip install pyDes

def des_encrypt(secret_key, s):
    iv = secret_key
    k = des(secret_key, CBC, iv, pad=None, padmode=PAD_PKCS5)
    en = k.encrypt(s, padmode=PAD_PKCS5)
    return binascii.b2a_hex(en)

def des_decrypt(secret_key, s):
    iv = secret_key
    k = des(secret_key, CBC, iv, pad=None, padmode=PAD_PKCS5)
    de = k.decrypt(binascii.a2b_hex(s), padmode=PAD_PKCS5)
    return de

secret_str = des_encrypt('12345678', 'I love YOU~')
print(secret_str)
clear_str = des_decrypt('12345678', secret_str)
print(clear_str)
```

 

**五、AES加密**

全称：高级加密标准（英语：Advanced Encryption Standard），在密码学中又称Rijndael加密法，是美国联邦政府采用的一种区块加密标准。这个标准用来替代原先的DES，已经被多方分析且广为全世界所使用。

**Python代码：**

```
import base64
from Crypto.Cipher import AES

'''
AES对称加密算法
'''
# 需要补位，str不是16的倍数那就补足为16的倍数
def add_to_16(value):
    while len(value) % 16 != 0:
        value += '\0'
    return str.encode(value)  # 返回bytes
# 加密方法
def encrypt(key, text):
    aes = AES.new(add_to_16(key), AES.MODE_ECB)  # 初始化加密器
    encrypt_aes = aes.encrypt(add_to_16(text))  # 先进行aes加密
    encrypted_text = str(base64.encodebytes(encrypt_aes), encoding='utf-8')  # 执行加密并转码返回bytes
    return encrypted_text
# 解密方法
def decrypt(key, text):
    aes = AES.new(add_to_16(key), AES.MODE_ECB)  # 初始化加密器
    base64_decrypted = base64.decodebytes(text.encode(encoding='utf-8'))  # 优先逆向解密base64成bytes
    decrypted_text = str(aes.decrypt(base64_decrypted), encoding='utf-8').replace('\0', '')  # 执行解密密并转码返回str
    return decrypted_text
```

 

**六、RSA加密**

全称：Rivest-Shamir-Adleman，RSA加密算法是一种非对称加密算法。在公开密钥加密和电子商业中RSA被广泛使用。它被普遍认为是目前最优秀的公钥方案之一。RSA是第一个能同时用于加密和数字签名的算法，它能够抵抗到目前为止已知的所有密码攻击。

**Python代码：**

```
# -*- coding: UTF-8 -*-
# reference codes: https://www.jianshu.com/p/7a4645691c68

import base64
import rsa
from rsa import common

# 使用 rsa库进行RSA签名和加解密
class RsaUtil(object):
    PUBLIC_KEY_PATH = 'xxxxpublic_key.pem'  # 公钥
    PRIVATE_KEY_PATH = 'xxxxxprivate_key.pem'  # 私钥

    # 初始化key
    def __init__(self,
                 company_pub_file=PUBLIC_KEY_PATH,
                 company_pri_file=PRIVATE_KEY_PATH):

        if company_pub_file:
            self.company_public_key = rsa.PublicKey.load_pkcs1_openssl_pem(open(company_pub_file).read())
        if company_pri_file:
            self.company_private_key = rsa.PrivateKey.load_pkcs1(open(company_pri_file).read())

    def get_max_length(self, rsa_key, encrypt=True):
        """加密内容过长时 需要分段加密 换算每一段的长度.
            :param rsa_key: 钥匙.
            :param encrypt: 是否是加密.
        """
        blocksize = common.byte_size(rsa_key.n)
        reserve_size = 11  # 预留位为11
        if not encrypt:  # 解密时不需要考虑预留位
            reserve_size = 0
        maxlength = blocksize - reserve_size
        return maxlength

    # 加密 支付方公钥
    def encrypt_by_public_key(self, message):
        """使用公钥加密.
            :param message: 需要加密的内容.
            加密之后需要对接过进行base64转码
        """
        encrypt_result = b''
        max_length = self.get_max_length(self.company_public_key)
        while message:
            input = message[:max_length]
            message = message[max_length:]
            out = rsa.encrypt(input, self.company_public_key)
            encrypt_result += out
        encrypt_result = base64.b64encode(encrypt_result)
        return encrypt_result

    def decrypt_by_private_key(self, message):
        """使用私钥解密.
            :param message: 需要加密的内容.
            解密之后的内容直接是字符串，不需要在进行转义
        """
        decrypt_result = b""

        max_length = self.get_max_length(self.company_private_key, False)
        decrypt_message = base64.b64decode(message)
        while decrypt_message:
            input = decrypt_message[:max_length]
            decrypt_message = decrypt_message[max_length:]
            out = rsa.decrypt(input, self.company_private_key)
            decrypt_result += out
        return decrypt_result

    # 签名 商户私钥 base64转码
    def sign_by_private_key(self, data):
        """私钥签名.
            :param data: 需要签名的内容.
            使用SHA-1 方法进行签名（也可以使用MD5）
            签名之后，需要转义后输出
        """
        signature = rsa.sign(str(data), priv_key=self.company_private_key, hash='SHA-1')
        return base64.b64encode(signature)

    def verify_by_public_key(self, message, signature):
        """公钥验签.
            :param message: 验签的内容.
            :param signature: 对验签内容签名的值（签名之后，会进行b64encode转码，所以验签前也需转码）.
        """
        signature = base64.b64decode(signature)
        return rsa.verify(message, signature, self.company_public_key)
```

 

**七、ECC加密**

全称：椭圆曲线加密（Elliptic Curve Cryptography），ECC加密算法是一种公钥加密技术，以椭圆曲线理论为基础。利用有限域上椭圆曲线的点构成的Abel群离散对数难解性，实现加密、解密和数字签名。将椭圆曲线中的加法运算与离散对数中的模乘运算相对应，就可以建立基于椭圆曲线的对应密码体制。

**Python代码：**

```
# -*- coding:utf-8 *-
# author: DYBOY
# reference codes: https://blog.dyboy.cn/websecurity/121.html
# description: ECC椭圆曲线加密算法实现
"""
    考虑K=kG ，其中K、G为椭圆曲线Ep(a,b)上的点，n为G的阶（nG=O∞ ），k为小于n的整数。
    则给定k和G，根据加法法则，计算K很容易但反过来，给定K和G，求k就非常困难。
    因为实际使用中的ECC原则上把p取得相当大，n也相当大，要把n个解点逐一算出来列成上表是不可能的。
    这就是椭圆曲线加密算法的数学依据
    点G称为基点（base point）
    k（k<n）为私有密钥（privte key）
    K为公开密钥（public key)
"""

def get_inverse(mu, p):
    """
    获取y的负元
    """
    for i in range(1, p):
        if (i*mu)%p == 1:
            return i
    return -1

def get_gcd(zi, mu):
    """
    获取最大公约数
    """
    if mu:
        return get_gcd(mu, zi%mu)
    else:
        return zi

def get_np(x1, y1, x2, y2, a, p):
    """
    获取n*p，每次+p，直到求解阶数np=-p
    """
    flag = 1  # 定义符号位（+/-）

    # 如果 p=q  k=(3x2+a)/2y1mod p
    if x1 == x2 and y1 == y2:
        zi = 3 * (x1 ** 2) + a  # 计算分子      【求导】
        mu = 2 * y1    # 计算分母

    # 若P≠Q，则k=(y2-y1)/(x2-x1) mod p
    else:
        zi = y2 - y1
        mu = x2 - x1
        if zi* mu < 0:
            flag = 0        # 符号0为-（负数）
            zi = abs(zi)
            mu = abs(mu)

    # 将分子和分母化为最简
    gcd_value = get_gcd(zi, mu)     # 最大公約數
    zi = zi // gcd_value            # 整除
    mu = mu // gcd_value
    # 求分母的逆元  逆元： ∀a ∈G ，ョb∈G 使得 ab = ba = e
    # P(x,y)的负元是 (x,-y mod p)= (x,p-y) ，有P+(-P)= O∞
    inverse_value = get_inverse(mu, p)
    k = (zi * inverse_value)

    if flag == 0:                   # 斜率负数 flag==0
        k = -k
    k = k % p
    # 计算x3,y3 P+Q
    """
        x3≡k2-x1-x2(mod p)
        y3≡k(x1-x3)-y1(mod p)
    """
    x3 = (k ** 2 - x1 - x2) % p
    y3 = (k * (x1 - x3) - y1) % p
    return x3,y3

def get_rank(x0, y0, a, b, p):
    """
    获取椭圆曲线的阶
    """
    x1 = x0             #-p的x坐标
    y1 = (-1*y0)%p      #-p的y坐标
    tempX = x0
    tempY = y0
    n = 1
    while True:
        n += 1
        # 求p+q的和，得到n*p，直到求出阶
        p_x,p_y = get_np(tempX, tempY, x0, y0, a, p)
        # 如果 == -p,那么阶数+1，返回
        if p_x == x1 and p_y == y1:
            return n+1
        tempX = p_x
        tempY = p_y

def get_param(x0, a, b, p):
    """
    计算p与-p
    """
    y0 = -1
    for i in range(p):
        # 满足取模约束条件，椭圆曲线Ep(a,b)，p为质数，x,y∈[0,p-1]
        if i**2%p == (x0**3 + a*x0 + b)%p:
            y0 = i
            break

    # 如果y0没有，返回false
    if y0 == -1:
        return False

    # 计算-y（负数取模）
    x1 = x0
    y1 = (-1*y0) % p
    return x0,y0,x1,y1

def get_graph(a, b, p):
    """
    输出椭圆曲线散点图
    """
    x_y = []
    # 初始化二维数组
    for i in range(p):
        x_y.append(['-' for i in range(p)])

    for i in range(p):
        val =get_param(i, a, b, p)  # 椭圆曲线上的点
        if(val != False):
            x0,y0,x1,y1 = val
            x_y[x0][y0] = 1
            x_y[x1][y1] = 1

    print("椭圆曲线的散列图为：")
    for i in range(p):              # i= 0-> p-1
        temp = p-1-i        # 倒序

        # 格式化输出1/2位数，y坐标轴
        if temp >= 10:
            print(temp, end=" ")
        else:
            print(temp, end="  ")

        # 输出具体坐标的值，一行
        for j in range(p):
            print(x_y[j][temp], end="  ")
        print("")   #换行

    # 输出 x 坐标轴
    print("  ", end="")
    for i in range(p):
        if i >=10:
            print(i, end=" ")
        else:
            print(i, end="  ")
    print('\n')

def get_ng(G_x, G_y, key, a, p):
    """
    计算nG
    """
    temp_x = G_x
    temp_y = G_y
    while key != 1:
        temp_x,temp_y = get_np(temp_x,temp_y, G_x, G_y, a, p)
        key -= 1
    return temp_x,temp_y

def ecc_main():
    while True:
        a = int(input("请输入椭圆曲线参数a(a>0)的值："))
        b = int(input("请输入椭圆曲线参数b(b>0)的值："))
        p = int(input("请输入椭圆曲线参数p(p为素数)的值："))   #用作模运算

        # 条件满足判断
        if (4*(a**3)+27*(b**2))%p == 0:
            print("您输入的参数有误，请重新输入！！！\n")
        else:
            break

    # 输出椭圆曲线散点图
    get_graph(a, b, p)

    # 选点作为G点
    print("user1：在如上坐标系中选一个值为G的坐标")
    G_x = int(input("user1：请输入选取的x坐标值："))
    G_y = int(input("user1：请输入选取的y坐标值："))

    # 获取椭圆曲线的阶
    n = get_rank(G_x, G_y, a, b, p)

    # user1生成私钥，小key
    key = int(input("user1：请输入私钥小key（<{}）：".format(n)))

    # user1生成公钥，大KEY
    KEY_x,kEY_y = get_ng(G_x, G_y, key, a, p)

    # user2阶段
    # user2拿到user1的公钥KEY，Ep(a,b)阶n，加密需要加密的明文数据
    # 加密准备
    k = int(input("user2：请输入一个整数k（<{}）用于求kG和kQ：".format(n)))
    k_G_x,k_G_y = get_ng(G_x, G_y, k, a, p)                         # kG
    k_Q_x,k_Q_y = get_ng(KEY_x, kEY_y, k, a, p)                     # kQ

    # 加密
    plain_text = input("user2：请输入需要加密的字符串:")
    plain_text = plain_text.strip()
    #plain_text = int(input("user1：请输入需要加密的密文："))
    c = []
    print("密文为：",end="")
    for char in plain_text:
        intchar = ord(char)
        cipher_text = intchar*k_Q_x
        c.append([k_G_x, k_G_y, cipher_text])
        print("({},{}),{}".format(k_G_x, k_G_y, cipher_text),end="-")


    # user1阶段
    # 拿到user2加密的数据进行解密
    # 知道 k_G_x,k_G_y，key情况下，求解k_Q_x,k_Q_y是容易的，然后plain_text = cipher_text/k_Q_x
    print("\nuser1解密得到明文：",end="")
    for charArr in c:
        decrypto_text_x,decrypto_text_y = get_ng(charArr[0], charArr[1], key, a, p)
        print(chr(charArr[2]//decrypto_text_x),end="")

if __name__ == "__main__":
    print("*************ECC椭圆曲线加密*************")
    ecc_main()
```

\- 来源于Python乱炖 ，作者 \- 我被狗咬了