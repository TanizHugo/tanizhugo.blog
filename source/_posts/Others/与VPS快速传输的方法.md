---
title: 与VPS快速传输的方法
date: 2023-08-26 23:16:09
author: Taniz Hugo
language: zh-CN
abbrlink: other1
top: false
is_draft: false
summary: 我是个比较注重效率的人，当我发现SFTP传输真的很慢的时候，于是我就开始找别的方法
categories: 
  - 其他
tags:
  - SFTP
  - CuteHttpFileServer
---

今天我的博客所在的VPS要到期了，我也没打算续费，所以需要迁移到新的VPS上，所以我需要将我的论文和哪些主题文件直接迁移到新的主机上。不过呢因为我新的VPS和旧的VPS之间线路很差（一个国内一个国外），直接`scp`很慢很慢，于是我就想反正都是要迁移的不如先下载到本机，反正也就留个底，以后还要更新的。
然后我就觉得这个速度也太慢了，因为利用WinSCP这些工具或者是其他的，基本都是采用sftp，如果文件很小，sftp是完全够用的，一旦文件比较大，sftp就不行了，而我又是注重效率的人，我就想找个方法解决这个问题。

![image-20230906231412781](https://s2.loli.net/2023/09/06/T5iJokCj8U9aGKS.png)

首先就是提出一个很简单的解决思路，既然sftp的方法不好，那我走http不好吗？所以方向很明确，有没有一个服务可以帮我帮我的文件传输通过http完成。



### 提速软件CuteHttpFileServer

官网：[http://iscute.cn/chfs](https://link.zhihu.com/?target=http%3A//iscute.cn/chfs)

CuteHttpFileServer/CHFS 是一个免费的 HTTP 服务器程序，支持平台广泛，除了 Windows 平台，对 MIPS 和 ARM 架构的 Linux 也都有支持。并且本身界面也很简洁方便，支持上传、下载、二维码下载等。

![image-20230826165731938](https://s2.loli.net/2023/08/26/bEAuKCVITfFRoh1.png)

程序本身也非常简单，就一个单文件，Windows 版本是有 GUI 界面的版本的，这里就说一下 Linux 下的使用。



### 下载软件

在官网选择适合的版本下载，这里我选择`hfs-linux-amd64-2.0.zip`

![image-20230826165854960](https://s2.loli.net/2023/08/26/ZAeGUBx6ERjWunc.png)

```shell
# curl下载zip文件
curl -o /opt/chfs-linux-amd64-2.0.zip http://iscute.cn/tar/chfs/2.0/chfs-linux-amd64-2.0.zip && cd /opt

# 解压文件
mkdir chfs && unzip chfs-linux-amd64-2.0.zip -d chfs
```



### 使用方法

解压下来可以看到其实这个就是一个文件，使用的方法也很简单，以下是基本命令

| 参数        | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| **help**    | 显示帮助信息                                                 |
| **path**    | 你要共享的目录，默认为程序运行目录。如果需要共享多个目录，则用“\|”符号隔开。**注意：如果路径带有空格，则需要将整个路径用引号包住。** |
| **port**    | 程序使用的端口号，默认为80                                   |
| **allow**   | IP地址过滤，可使用白名单模式或黑名单模式                     |
| **rule**    | 账户及访问权限，允许一个账户多点登陆，默认情况下匿名用户具有读写权限，其语法为：  **RULEITEM1[\|RULEITEM2\|RULEITEM3...]**  每个RULEITEM代表一个账户信息及其访问权限，多个RULEITEM则用'\|'进行分割，RULEITEM的语法为：  **USER:PWD:MASK[:DIR:MASK...]**  每个项由“:”来分隔，前三个项是必须的，分别对应：账户名、账户密码、共享目录根目录的访问权限。后面的可选的项，必须成对出现，用来设定根目录下面的子级目录的访问权限。一些规定：  * 对于匿名用户，前两个项都为空 * 访问权限分为四种：""(不可访问)，"R"(只读)，"W"(读写)，"D"(写+删除)。读权限指的是下载，写权限指上传、新建等操作，删除权限是在写权限的基础上加上删除权限。 * 各项的值应避免出现空白键，':'及'\|'（目录名除外） |
| **log**     | 用户操作日志存放目录，默认是程序所在目录下的logs中。禁用日志功能只需将其赋值为空即可。 |
| **file**    | 配置文件，该文件可配置上述配置项，语法相同，如果配置有效则覆盖对应配置项。另外，一些功能需要通过配置文件进行配置，比如页面自定义和SSL证书设置。[下载配置文件模板](http://iscute.cn/asset/chfs.ini) |
| **version** | 显示程序版本号                                               |



使用命令前先做一些准备，优化体验

```shell
# 修改权限
chmod -R 755 /opt/chfs/chfs

# 添加软连接
ln -s /opt/chfs/chfs /usr/local/bin/chfs

```



假设我们在/opt/test目录下有我们需要下载或上传的文件，以下是一些命令例子

```shell
# 打开新的终端
tmux

# 使用chfs打开连接
chfs --path /opt/test --port 8000

# 开放防火墙
ufw allow 8000
```

在浏览器访问就可以看到我们指定目录下的所有内容

![image-20230826172456167](https://s2.loli.net/2023/08/26/HgUGFIXbqNVydm7.png)

