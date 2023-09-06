
---
title: (五)数据流重导向
date: 2022-08-08 10:15:22
author: Taniz Hugo
language: zh-CN
abbrlink: linux5
top: false
is_draft: false
summary: 继续跟着鸟哥学Linux,第五章
categories: 
  - Linux
tags:
  - Linux指令
---



何为数据流重导向：将数据重新传递到其他地方去
那我们的数据一般传输去哪？屏幕啊！大多数情况下我们执行的命令数据最后都会传递到我们的屏幕，那我们可以传递到其他地方吗？当然可以，这就是我们这章要学的

## 什么是数据流重导向

要理解什么是数据路重导向，首先需要了解一下命令的运行结果，下图5.1展示了命令运行过程的数据传输情况

<center>
<img src="https://i.imgs.ovh/i/2023/08/20/64e1ff00d09c9.png"/><br/>
<font size=2>图5.1</font>
</center>


我们在Bash当中运行一个命令的时候，这个命令可能会由文件读取数据，经过处理之后，再将数据输出到屏幕上。图中 **standard output** 和 **standard output error** 分别代表了 **标准输出** 和 **标准错误输出** 

### Standard output 与 Standard error output

简单理解
标准输出：命令运行所回传的正确的信息
标准错误输出：命令运行失败后，所回传的错误信息

不管是正确或者是错误的数据都是默认输出到屏幕上，这不是我们想要的，一班正确的内容就是我们想看到的内容，而错误的内容我们就像保留下方便修改，由此可以通过特殊字符将信息传递到别的地方去：

    # 例子1：将错误的输出信息存储到 stderr文件中
    [tanzitao@master ~]$ cat test.txt an.txt 2> stderr  <-- 将错误的数据重导向到文件中
    this is a test file                                 <-- 正确的内容打印在了屏幕上
    [tanzitao@master ~]$ cat stderr                     <-- 查看保留错误的文件
    cat: an.txt: 没有那个文件或目录                      <-- 错误的内容保留在了文件内

一般常用的几个数据导向需要记住：

* 1>: 以覆盖的方法将[正确的数据] 输出到指定的文件或者装置上 
* 1>>: 以累加的方法将[正确的数据] 输出到指定的文件或者装置上 
* 2>: 以覆盖的方法将[错误的数据] 输出到指定的文件或者装置上 
* 2>>: 以累累加的方法将[错误的数据] 输出到指定的文件或者装置上 

需要注意的是 **1>>** 和 **2>>** 中间是没有空格的

### /dev/null 垃圾桶黑洞装置与特殊写法

想象一下，如果我知道错误信息会发生，所以就是要将错误信息忽略掉不显示呢？这个时候黑洞装置 **/dev/null** 就很重要了,因为所有的内容都会被扔掉

    # 例子2：将错误的输出信息存扔掉不显示
    [tanzitao@master ~]$ find /home -name .bashrc               <-- 直接查询所有创建的账号
    find: ‘/home/tzz’: 权限不够                                  <-- 因为这些账号不是我们创建的所以不能进入
    /home/tanzitao/.bashrc
    [tanzitao@master ~]$ find /home -name .bashrc 2> /dev/null   <-- 附带条件，将错误的信息全部扔到 /dev/null 里
    /home/tanzitao/.bashrc                                       <-- 只显示一条


如果我们既要保留正确命令又要保留错误的命令，并且写入同一文件，那么就需要参考以下正确写法

    # 例子3：将输出的内容输入到同一文件
    [tanzitao@master ~]$ find /home -name .bashrc > list 2> list    <-- 错误写法 
    [tanzitao@master ~]$ find /home -name .bashrc > list 2>&1  2>&1 <-- 语法正确
    [tanzitao@master ~]$ find /home -name .bashrc &> list           <-- 正确

### standard input : < 与 <<

在了解到 stderr与stdout后，那个 **<** 和 **<<** 又代表什么意思呢？
简单来说就是[将原本需要键盘输入的数据，改为有文件内容来取代]。我们先来了解下什么是键盘输入

    # 例子4：使用键盘输入创建文件
    [tanzitao@master ~]$ cat > catfile     <-- 利用 [>] 创建文件并向里输入内容
    test                                   <-- 一下都是输入的内容
    this
    file
    is  
    catfile

由于加入 > 在cat之后，所以catile会被主动创建，而内容就是键盘敲的数据，只不过

    # 例子5：使用[cat]直接将输入信息输出到[catfile]中，且当由键盘输入eof的时候，该次输入就结束
    [tanzitao@master ~]$ cat > catfile << "eof"    
    > test              <-- 以下是输入内容
    > this is a file
    > eof               <-- 遇到暂停符就退出输入流入

## 命令运行的判断依据

在某些情况下，许多的命令我想要一次输入去运行，而不是分多次运行，那该怎么办呢？一种可以通过 **shell script** 撰写脚本运行，另一种是通过 **;** 或者 **\|** 间隔开内容达到一次输入都条指令的形式

### [cmd ; cmd] （不考虑命令相关性的连续命令下达）

在命令与命令之间利用分号(;)隔开，这样一来，分号前的命令运行完后就会立刻执行后面的任务，往往这些命令之间没有太大的关联性，例如我们需要先更新我们的下载器，再进行模块的下载

    例子1：更新下载apt-get；利用apt-get下载内容
    [tanzitao@master ~]$ apt-get update; apt install ninja-build <-- 以顿号分隔，不用考略之间的连续性

### $? （命令回传值）与 && 和 ||

"有相关联性的连续命令"主要判断依据就是：前一个命令是否正确运行的对于后一指令是否有影响

若上一条命令运行结果正确，那么Linux底下会回传一个 **$? = 0** 的回传值。有了回传值的结果就能用 **&&** 和 **||** 判断后续的命令是否运行

| 命令下达情况   | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| cmd1 && cmd2   | 1. 若cmd1运行完毕且正确运行 （$?=0），则开始运行cmd2<br/>2. 若cmd1运行完毕且为错误 （$?!=0），则cmd2不运行 |
| cmd1 \|\| cmd2 | 1. 若cmd1运行完毕且正确运行（$?=0），则cmd2不运行<br/>2. 若cmd1运行完毕且为错误，则开始运行cmd2 |

* ps cmd1和cmd2都是命令

下面做一个测试：
（1）先判断一个目录是否存在；
（2) 若存在才在该目录底下创建一个文件

    例子1：使用ls查阅目录 /test/abc 是否存在，如果存在就在该目录下创建 touch.txt 文件
    [tanzitao@master ~]$ ls test/abc && touch test/abc/touch.txt
    ls: 无法访问test/abc: 没有那个文件或目录            <-- 直接报错 显示没有该文件，但是没有touch的错误，表示touch并没有运行
    [tanzitao@master ~]$ mkdir test/abc               <-- 在test目录下创建abc文件夹
    [tanzitao@master ~]$ ls test/abc && touch test/abc/touch.txt  <-- 成功执行且没有报错
    [tanzitao@master ~]$ ls test/abc
    touch.txt                                          <-- 文件夹下创建了 touch.txt 文件


    例子2：紧接着上面一步，报错的时文件不是不存在吗？那我们换一条命令，如果文件夹不存在，那就生成一个文件夹
    [tanzitao@master ~]$ ls test/abc || mkdir test/abc
    ls: 无法访问test/abc: 没有那个文件或目录
    [tanzitao@master ~]$ ls test/
    abc


    例子3：我不清楚 test/abc 文件夹是否存在，但是我想创建 test/abc/touch.txt 
    [tanzitao@master ~]$ ls test/abc || mkdir test/abc && touch test/abc/touch.txt     <-- 结合例子1和2的语句
    ls: 无法访问test/abc: 没有那个文件或目录                                             <-- 会报错因为 ls查询错误
    [tanzitao@master ~]$ ls test/abc/
    touch.txt                                                                           <-- 命令执行成功

例子3无论在什么情况下都能成功运行，我们可以分析下这是为什么？

* （1）如果 **test/abc** 文件夹不存在，查询文件夹命令: **ls test/abc** 执行错误，返回 \$?!=0,（2）因为 **||** 存在的缘故，执行创建文件夹命令: **mkdir test/abc**所以回传 \$?=0,（3）因为 **&&** 遇到 **$?=0** 的缘故，会执行创建文件语句: **touch test/abc/touch.txt**,（4）最终文件创建成功
* （2）如果 **test/abc** 文件夹存在，查询文件夹命令: **ls test/abc** 执行正确，返回 \$?=0,（2）因为 **||** 存在的缘故，不会执行创建文件夹命令,（3）因为 **&&** 遇到 **$?=0** 的缘故，会执行创建文件语句: **touch test/abc/touch.txt**,（4）最终文件创建成功


总的来说，这样的语句能够极大的方便我们对文件或者信息查询进行操作，不用走一步看一步，在特别庞大的程序中，往往我们对于命令注重的只有结果，因此一定要好好记牢这两种命令运行的判断依据

