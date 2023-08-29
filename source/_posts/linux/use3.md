---
title: Linux的Bash——（三）命名别名与历史命令
tags: Linux
author: Taniz Hugo
language: zh-CN
timezone: Asia/Shanghai
abbrlink: linux3
date: 2022-07-27 23:10:56
---



在早期DOS年代，可以使用 **cls** 清楚我们屏幕上的信息，但是在Linux当中使用的是 **clear** 清楚画面。那么我们可不可以让cls等于clear呢？在Bash当然可以，这样的操作成为 **命令别名**，我们输入的每一条指令都会被记录下来，成为 **历史命令** 

## 3.1 命令别名配置：alias unalias

命令别名的意思就是：将系统存在的命令赋予新的别名，可以一同使用
一般在一下几种情况会用到我们的命令别名：

### （1）管用命令特别长的时候：

举个例子来说，让我们要查询隐藏的文档，并且需要长的列出一页一页的翻看，那我们下达的命令为 **ls -al | more** ， 每次都这样输入真的很麻烦，那我们可以通过 **命令别名进行配置**

    # 例子1：使用alias简洁命令
    [tanzitao@node03 ~]$ alias lm='ls -al | more'  <-- 雀食节省了很多的空间


### （2）可以预防我们误操作：

我们经常会去删除文件等，可能我们有时候脑子一抽就把整个文件夹里的所有文件都删除了

    # 例子2：使用alias预防误操作
    [tanzitao@node03 ~]$ alias rm='rm -i'       <-- 这样设置后每次删除都会询问再次确认删除

### （3）查看我们所有的命令别名

    # 例子3：alias查看所有的命令别名
    [tanzitao@node03 ~]$ alias
    alias egrep='egrep --color=auto'
    alias fgrep='fgrep --color=auto'
    alias grep='grep --color=auto'
    alias l.='ls -d .* --color=auto'
    alias ll='ls -l --color=auto'
    alias lm='ls -al | more'
    alias ls='ls --color=auto'
    alias rm='rm -i'            <-- 刚刚设置的内容就存在啦
    alias vi='vim'              <-- vi和vim 不太一样 vim可以额外做一些语法检测和颜色显示，默认的root是单纯使用vi而已
    alias which='alias | /usr/bin/which --tty-only --read-alias --show-dot --show-tilde'
    
    # 可以看到系统内置了许多的命名别名

### （4）取消命令别名

        # 例子4：unalias取消命令别名
    
        [tanzitao@node03 ~]$ rm 123.txt                 <-- 保留例2中的别名
        rm: remove regular empty file '123.txt'? yes    <-- 删除前需要用户确认
        [tanzitao@node03 ~]$ unalias rm                 <-- 使用unalias取消命名
        [tanzitao@node03 ~]$ rm 456.txt                 <-- 直接删除，没有询问


## 3.2 历史命令：history

在bash当中提供有命令历史的服务，可以查询我们曾经下达过的命令，使用 **history** 就可以完成操作，不过这里我们先用命令别名简化下我们的命令

    [tanzitao@node03 ~]$ alias h='history'              <-- 命令别名 活学活用
    
    [tanzitao@node03 ~]$ history [n]
    [tanzitao@node03 ~]$ history [-c]
    [tanzitao@node03 ~]$ history [-raw] histfiles
    选项与参数：
    n   ：数字，意思是『要列出最近的 n 笔命令行表』的意思！
    -c  ：将目前的 shell 中的所有 history 内容全部消除
    -a  ：将目前新增的 history 命令新增入 histfiles 中，若没有加 histfiles ，
        则默认写入 ~/.bash_history
    -r  ：将 histfiles 的内容读到目前这个 shell 的 history 记忆中；
    -w  ：将目前的 history 记忆内容写入 histfiles 中！

### （1）使用history查看所有的历史

    # 例子1：列出目前内存的所有history记忆 注意！当前内存
    
    [tanzitao@node03 ~]$ h
    ...                         <-- 前面省略
    159  history
    160  alias h='history'
    161  h                      <-- 当前命令

列出的信息信息当中一共有两行：
第一行：命令所在当前shell当中为第几个命令
第二行：命令本身

### （2）使用history n查看进n调命令

    # 例子2：列出近5条命令
    [tanzitao@node03 ~]$ h 5
    158  rm 456.txt
    159  history
    160  alias h='history'
    161  h
    162  h 5

### （3）使用history -w写入数据

    # 例子3：立刻将当前数据写入histfile当中
    [tanzitao@node03 ~]$ h -w               <-- 默认会写入到 "./bash_history"当中
    [tanzitao@node03 ~]$ echo $HISTSIZE     <-- 查看HISTESIZE最多保留多少条记录
    1000


在正常的情况下，历史命令的读取与记录是这样的：


* 当我们以 bash 登录 Linux 主机之后，系统就会自动的调动文件 **~/.bash_history** 读取曾经的命令，至于命令的数量就有 环境变量 **HISTFILESIZE** 决定了

* 假设我这次登陆主机后，共下达过 100 次命令，等我注销时， 系统就会将 101~1100 这总共 1000 笔历史命令更新到 **~/.bash_history** 当中 也就是说，历史命令在我注销时，会将最近的 HISTFILESIZE 笔记录到我的纪录文件当中

* 当然，也可以用 history -w 强制立刻写入的，但是 **~/.bash_history** 记录的笔数永远都是 HISTFILESIZE 那么多，旧的信息会被主动的拿掉， 仅保留最新的


### （4）"!"命令的使用

    [tanzitao@node03 ~]$ !number
    [tanzitao@node03 ~]$ !command
    [tanzitao@node03 ~]$ !!
    选项与参数：
    number  ：运行第几笔命令的意思；
    command ：由最近的命令向前搜寻『命令串开头为 command』的那个命令，并运行；
    !!      ：就是运行上一个命令(相当于按↑按键后，按 Enter
    
    [tanzitao@node03 ~]$ h 5
    192  alias h='history'
    193  h
    194  h 5
    195  h 3
    196  h 5
    [tanzitao@node03 ~]$ !194  <-- 运行第194的命令
    [tanzitao@node03 ~]$ !!    <-- 运行上一个命令
    [tanzitao@node03 ~]$ !h    <-- 运行最近一 al 为开头的命令