---
title: Linux的Bash——(一)认识Bash
date: 2022-07-21 21:18:54
author: Taniz Hugo
language: zh-CN
abbrlink: linux1
top: false
is_draft: false
summary: 第一次接触Linux系统是在大二的课堂上，还记得那时候上课的老师真的很喜欢吹水，同时那时候也没有提起什么兴趣学，直到我的第一份实习，直接上手在Linux系统操作。为了更好的上手，于是我跟着鸟哥开始了Linux的学习~
categories: 
  - Linux
tags:
  - Linux指令
---



## 1.1 硬件、核心与Shell

我们都知道对于操作系统来说，肯定离不开Shell编程，为了理解Shell是干什么的，首先理解一下计算机的运行情况。举个例子来说：**当我想要电脑发出【音乐】的时候，计算机是怎么运转的呢？** 

    1. 硬件：具备声卡，只有硬件有【声卡芯片】才能发出声音
    2. 核心：操作系统的核心就是支持这个芯片组，提供芯片的驱动
    3. 应用程序：需要用户输入发出声音的命令


以上就是一个基本的步骤，也就是说：
1.我必须要【输入】一个命令之后，【硬件】才会通过下达的命令进行工作；
2.而【硬件】又怎么会知道我的下达的命令呢？就是通过【核心】（kernel）的控制工作；
3.用户输入命令【Shell】和【核心】（kernel）沟通，好让【核心】（kernel）可以控制硬件准确无误的工作。


<center>
<img src=https://i.imgs.ovh/i/2023/08/20/64e1decf6f04a.png width=200>


<font size=2>图1.1.1 硬件、核心与用户之间的关系</font><br/>
</center>

看完上面这幅图其实不难发现，作为用户，我们一般能接触的都是应用程序，就想鸡蛋的外壳一样，包含着我们整个系统，因此这个**应用程序**也被称呼为**壳编程(shell)**<br/>

**壳编程(shell)**的功能就是给用户提供一个操作系统的接口，只要你能够操作应用程序的接口都能称之为**壳程序(shell)**。<br/>

    1. 狭义壳程序：通过命令行运行的软件，包括Bash等
    2. 广义壳程序：包括图形化接口的软件，因为图形化接口也能的各种应用程序能够呼叫核心工作


## 1.2 为什么要学习文字接口的shell？

1. shell具有通用性
   无论操作系统再怎么个改变，shell的内核都不会发生很大的改变，不同的Linux版本下，所使用的Bash都是一样的


2. 在远程管理情况下，文字接口就是比较快
   Linux的管理常常都是需要通过远程联机的，而远程的联机文字接口的传输速度一定更快，而且不容易出现断线或者是信息外流的问题。

3. Liunx的核心
   想要真正搞懂一个系统的运行、管理系统的话，shell是绝对离不开的，而且shell能够帮助我们高效的处理文件信息，不需借助外部的任何手段

## 1.3 系统的合法shell 与 /etc/shells 功能

在我们了解到**Shell**在整个系统架构中担任的是什么角色，但是也是**Shell**也分为很多不同的版本<br/>
例如：Bourne SHell(sh)、 C SHell、商业上常用的K SHell，各有特点。至于在Linux使用的这一个版本的就成为**Bourne Again SHell （简称 BASH）**<br/>

其实在我们的Linux系统下也是兼容了不同版本的Shell，可以通过 **/etc/shells**文件查看
    

    cat /etc/shells   <-- 使用cat查看shells文件的内容
    /bin/sh         <-- 已经被 /bin/bash所取代
    /bin/bash       <-- 就是Linux默认的Shell
    <...等等就不一一列出来了>

**/etc/shells**这个文件为什么会存在？这是因为系统的某些服务在运行的过程中，回去检车使用者能够使用的shells，而这些shell的查询就是借由 **/etc/shells**这个文件。<br/>
同时在用户登录Linux的使用，系统就会分配一个shell进行工作，而这个登录取得shell就记录在 **/etc/passwd**这个文件内

    cat /etc/passwd | head -n 3  
    root:x:0:0:root:/root:/bin/bash
    bin:x:1:1:bin:/bin:/sbin/nologin
    daemon:x:2:2:daemon:/sbin:/sbin/nologin

## 1.4 Bash shell 的功能

1. 命令编辑功能（history）
   Bash可以记忆近1000个命令，用户可以通键盘的 **[上下键]** 查询命令。
   每条命令都被记录在 `~/.bash_history` ，不过需要注意的是，`~/.bash_history` 记录的是前一次登录所运行过的命令，这一次登录所运行的命令都被缓存在内存当中，当成功注销系统后，才会将新的命令记忆录到 `~/.bash_history` 当中。


2. 命令与文件补全功能（**[tab]**]的好处）
   Bash可以在输入命令的时候，只要输出前几个关键字按下 **[tab]** 自动将命令或者文件补齐
   (1) 少打很多字，提高效率 
   (2) 输入的数据绝对是正确的！
   同时如果想知道在当前环境下所有可以运行的命令有几个，可以直接在bash的提示字符后面连续按两下 **[tab]**,如果想知道"c"开头的命令有什么？直接按下 **c[tab][tab]**就可以了

3. 命令别名配置功能（alias）
   Bash可以将命令重新赋予别名，使用关键字 **alias**命令执行，例如 : alias lm='ls -al' 
   (1) 简化代码，针对特别长，需要用到 **管线**的命令，起到简化作用 
   (2) 防止某些代码的直接执行，避免错误，例如: alias rm='rm -r',这用做就能在每次删除文件或者文件夹之前再次确认是否删除

4. 工作控制、前景背景控制（job control, foreground, background）
   Bash可以提供用户在单一登录的环境中，达到多任务运行的目的。并且尅随时将工作人道背景当中运行，而不会因为使用 **[ctrl]+c** 使得程序终止

5. 程序化脚本（shell scripts）
   Bash可以将日常管理下打的连续命令写成一个文件，整个设计就想是一个小型的程序语言，而在Bash仅仅使用简单的shell scripts就可以完成了

6. 通配符（Wildcard）
   Bash支持许多的通配符来帮助用户查询与下达命令，举个例子如果我想知道 ~/下有多少t开头的文件，可以通过 **ls -l ~/t\*** 命令来查询

## 1.5 Bash shell的内建命令：type

在Bash当中，**内建**了许多的命令，例如: **cd、umask**
通过 **type** 可以知道命令是来自外部还是内建在bash当中

    type [-tpa] name
    # 选项和参数：
        ： 在不加入任何参数的时候，type会显示出name是外部命令还是内建命令
    -t  ： 当加入了-t的参数后，type会将name以下的内容显示出他的意义：
            file ： 表示为外部命令
            alias： 表示改命令为命令别名所配置的名称
            builtin： 表示改命令为bash内建的命令功能
    -p  ： 如果后面接的name为外部命令，才能显示完整的文件名
    -a  ： 会由PATH变量定义的路径中，将所有含有name的命令都列出来，包含了alias
    
    范例一：查询一下ls这个命令是否有内建
    [tanzitao@master ~]$ type ls              
    ls 是 `ls --color=auto' 的别名    <-- 没有加任何的参数，列出最主要的使用情况
    [tanzitao@master ~]$ type -t ls   <-- 仅仅列出ls运行时候的依据
    alias
    [tanzitao@master ~]$ type -a ls   <-- 列出了所有的依据
    ls 是 `ls --color=auto' 的别名     <-- 先试用aliase
    ls 是 /usr/bin/ls                 <-- 还有外部命令存在 /bin/ls
           
    范例二： 查询一下cd和alias这个命令是否有内建
    [tanzitao@master ~]$ type cd
    cd 是 shell 内嵌
    [tanzitao@master ~]$ type alias
    alias 是 shell 内嵌
    [tanzitao@master ~]$

## 1.6 Bash命令的下达

在前面我们提及过，为什么在我们以 **文本模式** 进入了我们的Linux系统后，我们就能输入命令与核心(kernal)进行沟通。这是因为用户登录后就会在系统文件 **/etc/bash** 加载我们的程序，这个程序我们称为壳(Shell)。

回顾了Bash命令的意义后，接下来就是了解Bash的命令下达方式，举个例子：

    [tanzitao@master ~]$ command [-options] parameter1 parameter2 ... 
    
    command(命令): ls      <-- 查看当前目录内容
    options(选项)：ls -a   <-- 查看当前目录详细内容
    paramenter(参数): ls -a ./test   <-- 查看test目录下所有文件和文件夹的详细内容

说明：
（1） 一行命令的第一个输入关键字一定是命令或者可运行文案、
（2） 选项部分可有可无，由实际命令来决定，且每个命令所携带的选项都不尽相同
（3） 命令、选项 、参数，这三部分之间都是由空格之间区分，且不论之间空几个格运行结果都一样
（4） 如果命令太长可以使用 **反斜杠(\\)** 跳脱 **[Enter]** 符号，使得命令可以连续下一行
    * 注意： **反斜杠(\\)** 是立即跟 **[Enter]** 才能换行，甚至一个空格都能有！！！
（5） 命令区分大小

