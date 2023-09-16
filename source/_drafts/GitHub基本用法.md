---
title: GitHub基本用法
date: 2023-08-31 12:18:23
author: Taniz Hugo
language: zh-CN
abbrlink: github1
top: false
is_draft: false
summary: GitHub就不用多说了吧，最大的项目开源库，丰富的学习资源，学习必备，不过学习之前也要学会工具啊，不然光看书不会记笔记怎么行
categories: 
  - GitHub
tags:
  - GitHub基本指令
  - 项目管理
---



GitHub 是一个面向开发者的协作平台，主要用于版本控制、源代码管理和协作开发项目。它提供了一个云端的代码托管服务，允许开发者将他们的代码存储在云端仓库中，并能够轻松地与团队成员合作、跟踪代码更改、解决问题和管理软件项目。

![github](https://s2.loli.net/2023/09/02/hemvJ1UzcTCqxoA.png)

对于程序员来说无论去到那，公司项目管理也好，自己的代码托管存储平台也好，GitHub及其衍生应用绝对是首选。想要用好GitHub，肯定离不开git这个工具，这个博客就是为了能够更快上手git而写的，目的也和简单嘛，给别人看看，也给自己看看咯。

在一切开始之前必须先确保有git的环境，可以从Git官方网站（https://git-scm.com/）下载适用的操作系统的Git版本。

#### 创建SSH密钥

首先这是在进行一切操作之前都必须要要先弄的一个步骤，这是为了更建立更安全、更快速的链接，管理自己的平台。

为了在使用Git时进行SSH密钥身份验证，需要生成SSH密钥对，并将公钥添加到你的Git托管服务帐户GitHub中。

##### 1. 本地创建密钥

在本地终端用以下命令生成SSH密钥，分别为`Ed25519`和`RSA`,其中`your_email@example.com`替换为你的电子邮件地址。这个电子邮件地址将与SSH密钥相关联，用于标识你的密钥。

```shell
# Ed25519
ssh-keygen -t ed25519 -C "your_email@example.com"

# RSA
ssh-keygen -t rsa -C "your_email@example.com"

```

生成密钥过程中系统会提示以下信息：

（1）需要你选择一个保存密钥的位置（默认为用户主目录下的`.ssh`目录）。

（2）提示改邮箱账号已经存在密钥，提示是否需要替换。

（3）如果你选择设置密码，将会提示你输入密码。这个密码用于保护你的私钥，通常建议设置密码以提高安全性。

```shell
$ ssh-keygen.exe -t rsa -b 4096 -C "your_email@example.com"	# 生成命令
Generating public/private rsa key pair.
Enter file in which to save the key (/c/Users/17514/.ssh/id_rsa):	# 密钥保存路径，按Enter键接受默认值
/c/Users/17514/.ssh/id_rsa already exists.	
Overwrite (y/n)? y	# 提示已经存在密钥，选择是否覆盖
Enter passphrase (empty for no passphrase):	# 提示输入密码（可以不输入）
Enter same passphrase again:	# 重复输入密码

```

##### 2. 拷贝本地生成密钥

密钥生成后可以选择访问默认路径，打开默认路径下的`id_rsa.pub`文件，复制内容。如果你在Windows下也可以执行以下命令复制直接复制内容。

```shell
 clip < ~/.ssh/id_rsa.pub
```

##### 3. 将公钥添加到Git托管服务

打开GitHub，点击右上角头像->点击弹出框的`Settings`->选择`SSH and GPG keys`

点击 ![image-20230902163209245](https://s2.loli.net/2023/09/02/K7ioDvPMx5tYOwB.png) 

输入你的密钥标题 ![image-20230902163334377](https://s2.loli.net/2023/09/02/u6GjDc7eJPN59wE.png)

粘贴密钥 点击保存。

![image-20230902165351425](https://s2.loli.net/2023/09/02/ukKpniljhzbIfmG.png)

##### 4. 测试SSH连接

可以使用以下链接测试SSH连接是否正常。

```shell
$ ssh -T git@github.com
Hi TanizHugo! You've successfully authenticated, but GitHub does not provide shell access.

```

如果一切设置正确，应该会看到自己的用户名，代表已经成功了。

#### 创建仓库

GitHub的仓库（Repository，通常简称为"repo"）是一个存储和管理代码、文档和其他项目资源的地方。它是Git版本控制系统的主要概念之一，GitHub则是一个基于Git的代码托管平台，它使得创建、协作和分享仓库变得更加容易和可视化。

#### 1. GitHub中创建仓库

 转到[GitHub的网站](https://github.com/)，登录到GitHub帐户。点击右上角头像，选择`Your repository`(你的仓库)，跳转到你的仓库当中。

![image-20230903065621203](https://s2.loli.net/2023/09/03/wAzt5brIOlsxq2V.png)

点击右上角的绿色图标![image-20230903065752719](https://s2.loli.net/2023/09/03/AiTXd4FyagUw7Ph.png) 开始创建仓库



填写仓库信息： 在创建仓库的页面上，您需要提供一些关于您的仓库的信息，包括：

- Repository name（仓库名称）：输入您的仓库的名称。
- Description（描述）：可选，用于描述您的仓库的简短说明。
- Visibility（可见性）：选择是公共仓库（Public）还是私有仓库（Private）。请注意，私有仓库可能需要付费。
- Initialize this repository with（使用以下选项初始化仓库）：您可以选择初始化仓库，添加README文件、.gitignore文件和许可证文件，或者您可以稍后手动添加它们。

![image-20230903070233052](https://s2.loli.net/2023/09/03/KmzCykLSfQVtBbr.png)

填写完所有信息后，点击绿色的"Create repository"按钮来创建您的仓库。到此仓库现在已经创建成功了！你将被重定向到您的新仓库的页面，可以在这里找到仓库的URL、仓库设置和其他相关信息。

##### 2. 关联本地

要在本地编辑GitHub仓库，首先需要将仓库克隆到您的计算机上。在命令行中导航到您要存储仓库的目录，并运行`git clone`，将GitHub仓库克隆到本地。例如，如果要克隆名为 "Wendy" 的仓库，可以运行：

```shell
# 采用ssh密钥方式拉去项目，前提是已经配置好密钥
git clone git@github.com:TanizHugo/Wendy.git

```

![image-20230903071030996](https://s2.loli.net/2023/09/03/l4qI6OnBcupS2Ak.png)

无论是HTTPS还是SSH，直接在前面加上`git clone`，然后复制url即可。



#### 编辑和提交更改

每次当我们对于文件进行了修改之后，都要将我们的修改内容同步到GitHub上面去。以下是一些基本的Git命令来完成这些操作：

- `git pull origin 分支名`:拉取仓库内容，确保您的本地仓库是最新的
- `git add <文件>`：将更改的文件添加到Git的暂存区。
- `git commit -m "提交消息"`：提交已添加到暂存区的更改，并附带一条描述提交内容的消息。
- `git push`：将本地的更改推送到GitHub仓库。

这些步骤是基本的操作，最好每次提交代码之前都进行如上操作，以下是详细的操作步骤。





（未完。。。待续。。。）

