---
title: HTTPS & TLS & 证书申请
date: 2023-08-27 14:09:38
author: Taniz Hugo
language: zh-CN
abbrlink: network2
top: false
is_draft: false
summary: 继认识完什么DNS服务器以及如何购买自己的域名后，紧接着对TLS协议以及证书申请的介绍，使用自己域名搭建网站
categories: 
  - 网络
tags:
  - TLS
  - 证书申请
  - HTTPS
  - Linux
  - ACME
---

#### 前言

紧接着介绍完什么是DNS以及购买完了自己域名后，那肯定就是想立刻将自己的域名利用起来，搭建一些网站。

不过再来回顾一下：域名其实就是将一串容易记住的英文字符如:`www.baidu.com`，映射到一个不容易记忆的IP地址如:`14.119.104.254`。

域名解析的过程发生在网络建立连接之前，那么接下来就是客户端与服务端建立安全的网络连接。我们在浏览器当中基本都是使用`HTTPS`协议进行交互，很容易理解，接下来要介绍的的就是`HTTPS`建立网络连接的过程和，以及`TLS协议`和`证书`在这个过程中充当了什么样的角色。

其实关于这一部分网络上已经有很多很多博客，就是一抓一大把。不过像这些都是一个程序员必须具备的基本知识，更加详细的内容我会后面再出，这里就是简单的介绍



#### TLS

##### 简单的介绍

TLS，是一个用于保护网络通信安全的协议，TLS的前身是SSL，不过因为在SSL 2.0版本中存在严重的漏洞，因此目前逐渐被TLS取代。

目前，TLS最新版本是TLS 1.3，也是最最为广泛的版本。

##### 工作流程

下面是简略的TLS工作流程

![image-20230906175204952](https://s2.loli.net/2023/09/06/oy3wlZVQSI74cD5.png)



#### 证书

##### 简单的介绍

数字证书简称证书，用于证明某个实体的身份，通常包括公钥和实体的相关信息。它是由权威机构（通常是证书颁发机构，CA）签发的，用于验证该实体的身份和公钥的有效性。

最重要的作用：保证通讯双方的数据安全性！

##### 证书的结构

- **主题（Subject）**：证书的主题标识了该证书所属实体的信息，如域名、组织名称等。
- **颁发者（Issuer）**：颁发者是签发该证书的CA的信息，证明了CA的身份。
- **有效期**：证书包括一个开始日期和截止日期，指示了证书的有效期。
- **公钥**：证书包含了实体的公钥，用于费对称加密算法的加密通信和验证数字签名。
- **数字签名**：证书上的数字签名是CA使用其私钥对证书内容的摘要进行加密的结果，用于验证证书的真实性。

##### 理解证书

有了证书确认之后，就能确认客户端A与服务端B之间的安全交流，假如突然出现了客户端C因为没有确认“证书”，因此拿不到对应的公钥，所以没办法插入A和B之间的交流。



#### 补充概念

由于讲证书里面有一些概念，为了方便理解，抽出来补充一下再继续往下写。

##### **公钥vs私钥**

* **公钥**：公开的密钥，可以发送给所有人知道

* **私钥**：不能给任何人知道，保存在服务器里面

##### **对称加密vs非对称加密**

* **对成加密算法**：加密密钥和解密密钥一样

* **非对称加密算法**：需要用私钥和公钥，私钥加密→公钥解密，公钥加密→私钥解密



#### HTTPS

了解了TLS的工作流程回头看HTTPS工作流程其实就没那么复杂了，因为**TLS建立连接**的工作流程是包括在**HTTPS建立连接**的过程之中的，所以说了解前者差不多就等于了解了后者，HTTPS建立连接的过程就不过多赘述了。

##### HTTP&HTTPS

HTTP最大的弊端就是**明文传输导致的不安全**，为了解决不安全的问题就引入了TLS协议。

![image-20230906182032555](https://s2.loli.net/2023/09/06/XQJNRHprjlbZC26.png)

一句话总结：HTTPS = HTTP + TLS



#### 证书申请

说完了理论知识， 接下来就是实操怎么为自己的服务器申请证书了。

为自己的服务器申请证书通常使用的是公共或私有的证书颁发机构（CA）来获得数字证书，这里我选择`ACME（Automated Certificate Management Environment）`，一种自动化证书管理协议，常用于自动化申请和更新SSL/TLS证书。

##### 操作步骤

###### 1. 添加DNS记录

首先回顾上一节提到的，添加我们的DNS记录，具体可以看之前的博客：[DNS与域名的购买和配置](https://blog.tanizhugo.top/posts/network1.html)

![image-20230824163343170](https://s2.loli.net/2023/09/06/BpfyDgOTZVz6U5s.png)

如我上图所示，添加了一条`www.tanizhugo.top`的记录，下面的操作我会以这个域名作为演示。

###### 2. 安装ACME客户端

创建一个目录存放我的证书

```shell
mkdir -p /etc/cert

```

安装证书工具

```shell
curl https://get.acme.sh | sh; apt install socat -y || yum install socat -y; ~/.acme.sh/acme.sh --set-default-ca --server letsencrypt

```

添加软连接

```shell
ln -s  /root/.acme.sh/acme.sh /usr/local/bin/acme.sh

```

切换CA机构

```shell

acme.sh --set-default-ca --server letsencrypt
```

###### 3. ACME挑战

ACME服务器将向域名所在的服务器发送挑战，这个所谓的挑战就是要验证，是否有该域名的控制权。在ACME我们只需要打开**80**端口就可以自动验证。如果挑战成功，那么就会颁发TLS证书。

开启80端口

```shell
ufw allow 80

```

发起挑战，以下有三种方法，如果不行就逐个试下。注意要将`www.tanizhugo.top`替换为你的域名。

```shell
acme.sh  --issue -d www.tanizhugo.top --standalone -k ec-256

acme.sh --register-account -m "${RANDOM}@chacuo.net" --server buypass --force --insecure && ~/.acme.sh/acme.sh  --issue -d www.tanizhugo.top --standalone -k ec-256 --force --insecure --server buypass

acme.sh --register-account -m "${RANDOM}@chacuo.net" --server zerossl --force --insecure && ~/.acme.sh/acme.sh  --issue -d www.tanizhugo.top --standalone -k ec-256 --force --insecure --server zerossl

```

当返回了证书存放路径，就证明申请证书成功了，以下是返回的例子。

![image-20230906212314189](https://s2.loli.net/2023/09/06/9DLe4jP2MHoXnaE.png)

###### 4. 安装证书

申请到证书后，已经安装到默认路径下，但是这个路径太麻烦了，因此我们选择安装到自己建的目录下。注意要将`www.tanizhugo.top`替换为你的域名，安装的文件名`www.tanizhugo.top.key`和`www.tanizhugo.top.key`需要替换为自己的进行区分。

```shell
acme.sh --install-cert -d www.tanizhugo.top --ecc --key-file /etc/cert/www.tanizhugo.top.key --fullchain-file /etc/cert/www.tanizhugo.top.crt

```

#### 应用

以上，就是整个证书申请的过程，其实还是很简单的，拿到证书后就可以开始用域名搭建自己的网页，或者反代之类的。

可以使用Nginx的
