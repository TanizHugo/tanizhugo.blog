---
title: Linux系统开启root ssh登录
date: 2023-9-12 17:36:21
author: Taniz Hugo
language: zh-CN
abbrlink: linux6
top: false
is_draft: false
summary: 薅到了亚马逊云的服务后，第一次接触到ubuntu系统无法用ssh登录，用密钥登录稍显麻烦，所以就找了个方法打开ssh登录的限制
categories: 
  - Linux
tags:
  - Linux指令
  - ssh

---



私钥登录我觉得会比较麻烦，如果出门在外在陌生平台登录，我想我大概率身边不会有随时保存有的私钥的文件，因此我就想开启root用户密码登录，因此就有了这篇文章。



##### 进入环境

利用VPS服务供应商为你创建的私钥，在本地先进入到环境当中，或者在浏览器通过IPv4地址进行连接。

输入以下命令进入root用户

```shell
sudo -i
```



##### 编辑配置文件

输入以下命令打开ssh配置文件

```shell
vim /etc/ssh/sshd_config
```



PS：进入vim后 在正常模式下 通过 "\" 快速定位需要找的内容，例如我需要找到`PermitRootLogin `所在的位置 可以输入"\PermitRootLogin "，通过按 'n'可以跳转下个符合查询内容的位置

1. 修改参数`PermitRootLogin`

找到以下内容：`PermitRootLogin`，大概在第33行左右

将内容修改为`PermitRootLogin yes`，如下图所示

![image-20230913122029019](https://s2.loli.net/2023/09/13/MHmTYsDGBJo47rU.png)

这样就开启了root的登录权限



2. 修改参数`PasswordAuthentication`

找到以下内容：`PasswordAuthentication`，大概在第57行左右

将内容修改为`PasswordAuthentication yes`，如下图所示

![image-20230913122631956](https://s2.loli.net/2023/09/13/5iLf8HSec6hsDzv.png)

这样就开启了密码登录



##### 修改root密码

输入以下指令修改root密码

```shell
echo 'root:{你的密码}' | chpasswd
```

重启ssh服务

```shell
systemctl restart ssh
```





通过以上的设置现在就可以通过ssh工具连接到这个VPS了