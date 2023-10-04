---
title: 安装Anaconda环境
date: 2023-9-30 11:13:43
author: Taniz Hugo
language: zh-CN
abbrlink: 
top: false
is_draft: false
summary: 最近换了台新电脑，打算上班之余学习下爬虫，不过电脑没有Python环境，所以趁着国庆假期有空就来搞一下环境
categories: 
  - Python
tags:
  - PyCharm
  - Conda
  - Python
  - 开发环境
---

#### 前言

最近换了台新电脑，打算上班之余学习下爬虫，不过电脑没有Python环境，所以趁着国庆假期有空就来搞一下环境。

在PC端最好的Python IDE我觉得就是PyCharm莫属了吧，如果说要管理Python环境，除了Python自带的`venv`，我更喜欢的还是用Anaconda。

闲话少说下面就直接进入安装的过程



#### 安装Anaconda

##### 1. 下载

Anaconda官方下载地址：https://www.anaconda.com/download/

Anaconda是免费下载的，选择自己的系统下载对于的Python

##### 2. 安装

安装的过程也很简单，基本都是下一步，下面是具体的安装过程：

![image-20230930115246257](https://s2.loli.net/2023/10/04/lsQMcAuifwSpRr6.png)

![image-20230930115314029](https://s2.loli.net/2023/10/04/WVSm9vo4Ht7fjIP.png)

![image-20230930115340560](https://s2.loli.net/2023/10/04/AtRGYHLKUDElzyI.png)

![image-20230930115522857](https://s2.loli.net/2023/10/04/KOZ8l1mBk27XoHU.png)

这一步推荐全部都勾上

![image-20230930115938405](https://s2.loli.net/2023/10/04/TOPDcdzKHwkStVn.png)

安装完之后一直按Next到这一步，全不勾上

![image-20230930120544102](https://s2.loli.net/2023/10/04/YeJ5j2UT4Z8E7mb.png)

##### 3. 验证

如果是完全按照前面的安装过程安装，那么正常来说Anaconda的程序已经会加入到环境变量当中，不用手动添加。

如果需要手动添加就打开：**控制面板\系统和安全\系统\高级系统设置\环境变量\用户变量\PATH** 中添加 Anaconda的安装目录的Scripts文件夹。

`win + R` 输入`cmd`，打开命令行，输入以下命令：

```shell
 conda --version
```

成功返回Anaconda版本号就是代表安装成功

![image-20230930121428913](https://s2.loli.net/2023/10/04/5lk6yupYQtL3BOg.png)



#### Anaconda的使用

下面列出一些Anaconda常用的命令，满足日常的版本管理需求。

##### 1.更新

为了避免可能发生的错误, 在第一次允许Anaconda前，先把所有工具包进行升级

```shell
conda upgrade --all -y
```



##### 2. 创建环境

这里的环境指的就是一个个独立的Python环境，每个Python环境都有自己独立的内核版本，以及各自所需的开发环境。假如我这里现在需要创建一个Python环境用与爬虫，需要Python内核版本是3.7，将它命名为my_request，以下是执行命令：

```shell
conda create --name my_request python=3.7 -y
```



##### 3. 激活环境

激活名为`my_request`的环境

```shell
conda activate myenv
```

成功激活环境后，前面会显示当前环境名

![image-20230930124335845](https://s2.loli.net/2023/10/04/Vdj3ugpSkaUrcPA.png)

在不同环境下，我们利用pip工具下载各种Python工具的时候都是互不干扰的，所以要安装某个工具的时候要先计入到对应的环境下载工具，否则有可能不会生效。



##### 4.   退出环境

这个不必多说，就是退出当前Python环境

```shell
conda deactivate
```



###### 5. 移除某环境

删除不需要或者是临时的环境，假设我要删除刚刚创建的`my_request`的环境

```shell
conda env remove --name myenv
```



#### 导入PyCharm

打开PyCharm，如果你曾经允许过某个程序，右下角会显示使用Python内核，如果没有就会显示：无解释器。点击添加解释器。

![image-20230930131903200](https://s2.loli.net/2023/10/04/Z3GfRYkAHPVidOC.png)

选择Conda环境，点击现有环境，在解释器那里选择Anaconda的安装目录下的`envs`文件夹，每个文件夹的名字对应的就是创建的Python环境。

选择对应的环境的名文件夹 下的`python.exe`文件

![image-20230930131547449](https://s2.loli.net/2023/10/04/ix75pUrZkXvDKTh.png)

现在可以看到右下角现实的就是刚刚创建的Python环境，这样环境就算创建成功了。

![image-20230930131921217](https://s2.loli.net/2023/10/04/rUuMsLlZnpGwWRk.png)
