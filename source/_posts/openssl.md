---
title: openssl相关介绍 
date: 2019-08-01 14:21:18
tags:
  - Linux  
categories:
  - 技术
---

##### 概述
openssl是一个开源的软件包, 我们可以使用它用来进行安全通信, 它主要分为三个功能部分，SSL协议库、应用程序和密码算法库。
< !--more-->

##### openssl的应用场景是什么？
在https协议中， http下层的传输协议由TCP/IP变成了SSL/TLS。 说到这里，需要了解下SSL(安全套接层协议)和TLS(传输层安全)他们之间的联系。在我认为TLS其实就是SSL，只不过一开始不存在TLS，在SSL发展到v3版本的时候，在1999年的被IETF改名了，正式成为了一个安全传输协议的标准。所以，TLSv1.0应该是基本等价SSLv3.1。
那openssl到底应用在哪里呢？ 上面有提到，openssl其实包含了3部分内容，其中一部分就是SSL协议库，它几乎支持所有的加密算法和加密协议，当然就包括SSL协议咯。所以，很多应用软件都会使用它作为底层来实现TLS的功能，例如nginx, apache等。

##### openssl有哪些功能，命令?
因为加密技术分为对称加密和非对称加密，而openssl既然支持几乎所有的加密算法，所以openssl肯定也是支持对称加密和非对称加密的。
#####  对称加密算法，加密解密使用同一个密钥
```
openssl enc 
  -e  -des3  使用des3加密
  -d  -des3  使用des3解密
  -a  使用base64编码格式
  -salt 加入随机数
  -out  输出加密文件路径
  -in   指定加密文件路径

如：openssl enc -e -des3 -a -salt -in johnny -out johnny.jiami
```

##### 非对称加密算法，公钥从私钥中提取出来，使用对方的公钥来加密数据，保证数据安全，用自己的私钥加密来证明数据的来源。

- 生成私钥
```
openssl genrsa [-out filename] [-passout arg] [-des] [-des3] [-idea] [-f4] [-3] [-rand file(s)] [-engine id] [numbits]

例如，生成一个长度为4096的私钥
openssl genrsa -out johnny.key 4096  

```
生成私钥的常用命令
```
-out filename 将生成的私钥保存到文件
-des -des3 -idea 指的是加密算法
numbits 指的是生成密钥的大写，默认2048
```
- 从公钥中提取私钥
```
openssl rsa [-inform PEM|NET|DER] [-outform PEM|NET|DER] [-in filename] [-passin arg] [-out filename] [-passout arg] [-sgckey] [-des] [-des3][-idea] [-text] [-noout] [-modulus] [-check] [-pubin] [-pubout] [-engine id]

例如，提取一个私钥
openssl rsa -in johnny.key -out johnny.pub -pubout
```
提取私钥的几个命令
```
-in 指定私钥文件
-out 提取的公钥写入文件
- pubout 提取公钥
```

##### 单向加密
```
openssl dgst -md5 johnny.key

几种家用的加密算法
-md5
-md4
-sha1
-sha
-dss1
```

##### 生成随机数
```
openssl rand -格式 num
openssl rand -base64 10

-base64 以base64格式编码
-hex 16进制编码
-out 输出到文件
后面的num表示生成多少bytes长度的随机数
```

##### 生成密码
```
openssl passwd 
生成密码(标准输入) echo -n "123456" | openssl passwd -crypt stdin
生成密码(文件输入) openssl passwd -1 -in johnny.mingwen

-1 指定md5算法
-crypt Linux标准密码算法格式
-in 需要加密的输入文件
stdin 通过标准输入
```

##### openssl还可以生成证书，等
