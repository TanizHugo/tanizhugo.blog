
---
title: (二)Shell的变量
date: 2022-07-25 21:26:21
author: Taniz Hugo
language: zh-CN
abbrlink: linux2
top: false
is_draft: false
summary: 继续跟着鸟哥学Linux,第二章
categories: 
  - Linux
tags:
  - Linux指令
---


## 2.1 什么是变量？

**变量就是以特定的字符串代表不固定的内容**

变量的优点又有那些？

### (1) 可变性与方便性

关于可变性和方便性，我相信如果有接触过至少一门编程语言就能很快Get到这个点，所谓的： **一改全改** ,特别是在脚本程序设计(shell script)中起了非常大的作用
对于要反复引入的而又复杂的内容，使用变量真的太合适了，例如我们的：路径、版本名等

* 补充：硬件环境下，一般也会用到变量来代替值，使得某些值更具有表象意义，例如： LED_ON=0; LED_OFF=1;

### (2) 影响bash环境操作的变量

在我们的bash当中，某些特定的变量也会影响我们的命令是否能正常运行。在前面我们也提到过，bash的命令分为内嵌和外部。
例如我们在 第一章所提到过的 **ls** 这条指令的路径就是在 "/etc/bin" 当中，其中变量 **PATH** 就决定我们能不能在任何的目录下运行命令。

深入来说，由于在Linux System所有的线程都需要一个运行码，如同上面所提到的，真正shell 来跟Linux 沟通，是在正确登录Linux系统之后，而在这之前，由于系统需要一些变量在提供数据的存取，所以一些所谓的 **环境变量** 就需要读入到系统中，例如： **PATH、HOEM、MAIL、SHELL**

* 为了与普通变量区分，一般环境变量都是大写的（国际通用规则啦）

## 2.2 echo，变量配置守则

变量与变量代表的内容之间的关系是怎么样的呢？这就必须要引入一下命令帮助我们更好的理解

### (1) 变量的取用：echo

* 取用&获取
  取用变量或者说获取变量内容的方法很简单，就是在变量名称前面加上\$，或者是以\${变量}的方式来取都是可以的

* 配置&修改
  更为简单，只需要用等号(=)连接变量与内容即可配置或者修改变量成功

* 例子

      [tanzitao@manager ~]$ echo $PATH          <-- 获取环境变量
      /opt/torque/bin:/opt/torque/sbin:/home/tanzitao/.vscode-server/  <-- 环境变量内容 
      [tanzitao@manager ~]$ TZT=test      <-- 配置变量（TZT）
      [tanzitao@manager ~]$ echo $TZT     <-- 查询变量
      test                                <-- 获取结果


* 补充：当一个变量名没有给配置的时候都是空的

### (2) 变量的配置守则

* 变量与变量的内容以一个等号连接，且等号两边都不能有空格

      [tanzitao@manager ~]$ tag = tanzitao   <-- 有空格，错误
      bash: tag: 未找到命令
      [tanzitao@manager ~]$ tag=tanzitao     <-- 无空格，正确

* 变量名只能由英文和中文，但是不能以数字开头

      [tanzitao@manager ~]$ 1tag=tanzitao    <-- 数字开头
      bash: 1tag=tanzitao: 未找到命令         <-- 报错
      [tanzitao@manager ~]$ tag=tanzitao     <-- 字母开头 没报错

* 变量的如果有空格符可以使用双引号["] 或 [']将变量的内容结合起来，但是两者之间却又存在差异

      [tanzitao@manager ~]$ var="lang is $LANG"  <-- 使用了["]
      [tanzitao@manager ~]$ echo $var
      lang is zh_CN.UTF-8                        <-- 成功输出结果
      [tanzitao@manager ~]$ var='lang is $LANG'  <-- 使用了[']
      [tanzitao@manager ~]$ echo $var
      lang is $LANG                              <-- 输出字符串本身
      
      ps: 两者的区别就在于：双引号["]会高级点，能够将变量中的的变量值读取出来，而单引号[']内的字符为一班字符

* 可以跳脱的字节 [ \ ]将特殊符号(如\[Enter],$,\,空格符,'等) 变成一般字符

      [tanzitao@manager ~]$ var='lang \
      > ^C
      [tanzitao@manager ~]$ var="lang is \
      > $LANG"

* 在一串命令当中如果还需要用到其他的命令提供信息，可以使用反引号[\`命令\`] 或者 [$(命令)]的形式在命令当中嵌入命令 

      [tanzitao@manager ~]$ version="version`uname -r`" | echo $version    <-- 使用[\`命令\`]方式嵌入命令
      version=3.10.0-1160.15.2.el7.x86_64
      [tanzitao@manager ~]$ version="version=$(uname -r)" | echo $version  <-- 使用[$(命令)]方式嵌入命令
      version=3.10.0-1160.15.2.el7.x86_64

* 若变量为扩增变量的内容时，可以用 "\$变量名称" 或者 \${变量}累加内容

      [tanzitao@manager ~]$ echo $sum
      1
      [tanzitao@manager ~]$ sum="$sum"+2  <-- 直接跟在后面 特别注意！ '+' 不代表连接符
      [tanzitao@manager ~]$ echo $sum
      1+2

* 若变量需要在其他的子程序当中运行，可以使用 **export** 

      [tanzitao@manager ~]$ name=tanzitao <-- 初始化变量
      [tanzitao@manager ~]$ echo $$       <-- 查看当前程序的PID
      6227                                <-- 当前Bash程序的PID
      [tanzitao@manager ~]$ bash          <-- 开启子程序
      [tanzitao@manager ~]$ echo $$       <-- 查看当前程序的PID
      3177                                <-- 当前Bash程序的PID
      [tanzitao@manager ~]$ echo $name    <-- 获取变量内容（无结果）
      
      [tanzitao@manager ~]$ exit          <-- 退出子程序
      exit
      [tanzitao@manager ~]$ echo $$       <-- 查看程序PID
      6227                                <-- 确认回到当前程序
      [tanzitao@manager ~]$ export name   <-- 变量转换为环境变量
      [tanzitao@manager ~]$ bash          <-- 再次进入子程序
      [tanzitao@manager ~]$ echo $name    <-- 获取变量内容
      tanzitao                            <-- 显示结果

* 大写：系统默认变量；小写：自行配置变量

* 取消变量使用 unset

## 2.3 环境变量的功能

环境变量的主要功能：home目录的变换、提示字符的显示、运行文件搜索的路径

### (1) 用 env 观察环境变量与常见环境变量的说明

      [tanzitao@manager ~]# env
      HOSTNAME=manager               <-- 这部主机的主机名
      TERM=xterm-256color            <-- 这个终端机使用的环境是什么类型
      SHELL=/bin/bash                 <-- 目前这个环境下，使用的 Shell 是哪一个程序？
      HISTSIZE=1000                  <-- 记录命令的笔数在 CentOS 默认可记录 1000 笔
      USER=tanzitao                  <-- 使用者的名称
      LS_COLORS=...                  <-- 一些颜色显示
      MAIL=/var/spool/mail/tanzitao  <-- 这个用户所取用的 mailbox 位置
      PATH=/home/tanzitao/bin       <-- 运行文件命令搜寻路径
      INPUTRC=/etc/inputrc           <-- 与键盘按键功能有关。可以配置特殊按键！
      PWD=/home/tanzitao             <-- 目前用户所在的工作目录 (利用 pwd 取出！)
      LANG=en_US                     <-- 这个与语系有关，底下会再介绍！
      HOME=/home/tanzitao            <-- 这个用户的家目录啊！
      _=/home/tanzitao               <-- 上一次使用的命令的最后一个参数(或命令本身)

* RANDOM

这是一个 **随机的随机数变量** 可以通过 \$RANDOM的方式生成随机数。在一般情况的\$RANDOM的变量范围在0-32767范围内

      [tanzitao@manager ~]$ declare -i number=$RANDOM*10/32768; echo $number
      7                                    <-- 取出0-9范围内的随机变量

### (2) 用 set 观察所有变量（环境变量+自定义变量）

      [tanzitao@manager ~]# set
      BASH=/bin/bash                                   <-- bash 的主程序放置路径
      BASH_VERSINFO=("x86_64-redhat-linux-gnu")        <-- bash 的版本
      BASH_VERSION='4.2.46(2)-release'                 <-- 也是 bash 的版本啊！
      COLUMNS=156                                      <-- 在目前的终端机环境下，使用的字段有几个字符长度
      HISTFILE=/home/tanzitao/bash_history             <-- 历史命令记录的放置文件(隐藏文档)
      HISTFILESIZE=1000                                <-- 存起来(与上个变量有关)的文件之命令的最大纪录笔数。
      HISTSIZE=1000                                    <-- 目前环境下，可记录的历史命令最大笔数。
      HOSTTYPE=x86_64                                  <-- 主机安装的软件主要类型。
      IFS=$' \t\n'                                     <-- 默认的分隔符
      LINES=35                                         <-- 目前的终端机下的最大行数
      MAILCHECK=60                                     <-- 与邮件有关。每 60 秒去扫瞄一次信箱有无新信！
      OLDPWD=/home                                     <-- 上个工作目录。我们可以用 cd - 来取用这个变量。
      OSTYPE=linux-gnu                                 <-- 操作系统的类型！
      PPID=20025                                       <-- 父程序的PID (会在后续章节才介绍)
      PS1='[\u@\h \W]\$ '                              <-- PS1命令提示字符，也就是我们常见的[root@www ~]# 或 [dmtsai ~]$ 的配置值
      PS2='> '                                         <-- 如果你使用跳脱符号 (\) 第二行以后的提示字符也
      name=VBird                                       <-- 刚刚配置的自定义变量也可以被列出来喔！
      $                                                <-- 目前这个 shell 所使用的 PID
      ?                                                <-- 刚刚运行完命令的回传值。      

#### PS1：（提示字符的配置）

首先需要的注意的是PS1后面不是应英文字母，是数字1，其实PS1就相当于我们的 **命令提示字符** 当我们每次按下 **Enter** 去运行某些命令都会出现

      [tanzitao@manager ~]$ echo $PS1  <-- 查看我们的提示字符
      [\u@\h \W]\$                     <-- 具体内容(符号意义在下面讲)
      [tanzitao@manager ~]$            <-- 每一行都会存在的命令提示字符


`源格式：[\u@\h \W]\$`

* \d : 可显示出 **星期 月 日** 的日期格式

      [tanzitao@manager SCOW]$ PS1="[\d@\u \h]\$"
      [二 7月 26@tanzitao manager]$
      下面的可以自主尝试

* \H : 完整的主机名

* \h : 仅取主机名在第一个小数点之前的名字

* \t : 显示时间，为24小时的 HH：MM：SS

* \T : 显示时间，为12小时的 HH：MM：SS

* \A : 显示时间，为24小时的 HH：MM

* \@ : 显示时间，为12小时的 am/pm 样式

* \u : 目前使用者的账号名称如：root

* \v : BASH的版本信息

* \w : 完整的工作目录名称

* \W : 利用basename函数取得工作目录名称，列出最后一个目录的名字

* \\# : 下达的第几个命令

* \\$ : 提示字符，如果是root时，提示字符为 **\#** 否则是 **\$**

#### \$:(关于本地shell的PID)

所谓的PID就是(Process ID)，通过 **echo \$$** 可以读取当前Shell的PID

#### ?:(关于上个运行命令的回传值)

* 只要上一句话成功执行就会回传一个0
* 只要上一句话运行发生错误就会回传一个非0的错误代码

### (3) export： 自定义变量转化为环境变量

经过对命令 **env** 和 **set** 的认识，了解到了什么环境变量和自定变量，那么这两者具体的差异是什么呢？总的来说概括为一句话：`该变量是否会被子程序所继续应用`

当我们登录Linux并取得一个bash程序之后，接下来的任何命令都是由这个bash所衍生出来的，那些被下达的命令就成为子程序。下图展示了父程序与子程序之间的概念：

<center><img src='https://i.imgs.ovh/i/2023/08/20/64e1e44e38aa6.png'><br/><font size=2>图1：父程序与子程序</font></center>


原本在bash下运行另外一个bash，结果操作的环境就会跑道第二个bash当中，原本的bash就被暂停。只要暂定子程序的bash就能会到父程序当中

至于 **export** 的用法以及父程序和子程序的演示在本章前面部分已经演示过了，就不再重复

## 2.4 变量的有效范围

变量的有效范围在学习 **export** 的时候就已经提及了，在程序运行的时候，有父程序和子程序之间的关系，则变量是否能被应用和export有很大的关系

* 在某些不同的数据会谈论到[全局变量]与[局部变量]，可以这样看待：
  `环境变量=全局变量`
  `自定义变量=局部变量`

为什么环境变量的数据可以被子程序所应用呢？
因为这是内存分配的关系，操作过程如下

  * 当启动一个shell，操作系统会自动分配给一些给予区块给shell，这个内存变量和一让字程序免费用
  * 若在附程序利用了export，可以昂自定义的变量内容写在上述的记忆区域中（环境变量）
  * 当加载另外一个shell，子shell可以将父shell的环境变量所在记忆块导入记得环境变量当中


## 2.5 变量键盘读取、数组与宣告： read，array，declare

变量配置都是由命令直接配置的，那么可不可以由用户经过键盘进行输入呢？就是说，在某些程序运行的过程中需要用户输入“yes/no”之类的信息，在bash里也有相对应的功能

### (1) read

要读取来自键盘输入的变量，那就需要用到read这个命令了。这个命令最常被用在shell script当中。

      [tanzitao@manager SCOW]# read [-pt] variable
      选项与参数：
      -p  ：后面可以接提示字符
      -t  ：后面可以接等待的秒数
      
      # 例子1：read的使用
      [tanzitao@manager SCOW]$ read test        <-- 使用read命令 会等待用户输入
      This is a test                            <-- 输入"test"变量的内容
      [tanzitao@manager SCOW]$ echo $test       <-- 取出变量内容
      This is a test
    
      # 例子2：read参数的使用
      [tanzitao@manager SCOW]$ read -p "input your name:" -t 20 name  <-- "-p"后跟提示字符 "-t"后跟等待时间(s)
      input your name:tzt
      [tanzitao@manager SCOW]$ echo $name
      tzt


read的命令就像是c语言当中 scanf() 一样，在程序中实现程序和使用者交流


### (2) declare / typeset

**declare** 和 **typeset** 的功能是一样的，都是用来宣告变量的类型

      [tanzitao@manager SCOW]$ declare [-aixr] variable
      选项与参数：
      -a  ：将后面名为 variable 的变量定义成为数组 (array) 类型
      -i  ：将后面名为 variable 的变量定义成为整数数字 (integer) 类型
      -x  ：用法与 export 一样，就是将后面的 variable 变成环境变量；
      -r  ：将变量配置成为 readonly 类型，该变量不可被更改内容，也不能 unset
    
      # 例子1：declare -i的使用
      [tanzitao@manager SCOW]$ sum=1+3+5              <-- 初始化变量为"计算式"形式的"字符串"
      [tanzitao@manager SCOW]$ echo $sum
      1+3+5                                           <-- 结果为"字符串"
      [tanzitao@manager SCOW]$ declare -i sum=1+3+5   <-- 定义sum变量为整数数字类型
      [tanzitao@manager SCOW]$ echo $sum
      9                                               <-- 结果为计算式的结果

在默认的情况下，bash对于变量有几个基本的定义：

* 变量得类型默认为 **字符串** ，如果不指定类型，呢么1+2就是一个 **字符串** 而不是 **计算式** 

* bash环境中的数值运算，默认最多仅能达到整数的形态，所以`1/3结果是0`

      # 例子2：declare -x的使用
      [tanzitao@manager SCOW]$ declare -x sum         <-- 将变量转化为环境变量 
      [tanzitao@manager SCOW]$ export | grep sum
      declare -ix sum="9" 
      
      # 例子3：declare -r的使用
      [tanzitao@manager SCOW]$ declare -r sum         <-- 变量转化为只读，内容不可更改
      [tanzitao@manager SCOW]$ sum=123
      -bash: sum: 只读变量
      [tanzitao@manager SCOW]$ declare +x sum         <-- 使用"+"可以取消变量的设置
      [tanzitao@manager SCOW]$ declare -p sum
      declare -ir sum="9"

### (3) 数组(array)变量类型

在某些时候，我们必须使用该数组来宣告一些变量，这既需要用到数组的变量类型，首先明确一点，在bash里头，数组的配置方式是：`var[index]=content`

* var:数组名

* \[index]:下标

* content:变量内容
      

      [tanzitao@manager SCOW]$ var[1]="small min"
      [tanzitao@manager SCOW]$ var[2]="big max"
      [tanzitao@manager SCOW]$ var[3]="nice max"
      [tanzitao@manager SCOW]$ echo "${var[1]}, ${var[2]}, ${var[3]}"               <-- 需要注意数组的取出变量的方式
      small min, big max, nice max


数组的读取方式需要注意：建议直接以 **${数组}** 的方式来读取

## 2.6 与文件系统以及程序的限制关系：ulimit

bash可以通过限制用户的某些系统资源，从而预防系统的资源溢出


      [tanzitao@manager SCOW]$ ulimit [-SHacdfltu] [配额]
      选项与参数：
      -H  ：hard limit ，严格的配置，必定不能超过这个配置的数值；
      -S  ：soft limit ，警告的配置，可以超过这个配置值，但是若超过则有警告信息。
            在配置上，通常 soft 会比 hard 小，举例来说，soft 可配置为 80 而 hard 
            配置为 100，那么你可以使用到 90 (因为没有超过 100)，但介于 80~100 之间时，
            系统会有警告信息通知你！
      -a  ：后面不接任何选项与参数，可列出所有的限制额度；
      -c  ：当某些程序发生错误时，系统可能会将该程序在内存中的信息写成文件(除错用)，
            这种文件就被称为核心文件(core file)。此为限制每个核心文件的最大容量。
      -f  ：此 shell 可以创建的最大文件容量(一般可能配置为 2GB)单位为 Kbytes
      -d  ：程序可使用的最大断裂内存(segment)容量；
      -l  ：可用于锁定 (lock) 的内存量
      -t  ：可使用的最大 CPU 时间 (单位为秒)
      -u  ：单一用户可以使用的最大程序(process)数量。
    
      [tanzitao@manager SCOW]$ ulimit -a
      core file size          (blocks, -c) 0          <-- 只要是 0 就代表没限制
      data seg size           (kbytes, -d) unlimited
      scheduling priority             (-e) 0
      file size               (blocks, -f) unlimited  <-- 可创建的单一文件的大小
      pending signals                 (-i) 31191
      max locked memory       (kbytes, -l) 64
      max memory size         (kbytes, -m) unlimited
      open files                      (-n) 1024      <-- 同时开启文件的数量
      pipe size            (512 bytes, -p) 8
      POSIX message queues     (bytes, -q) 819200
      real-time priority              (-r) 0
      stack size              (kbytes, -s) 8192
      cpu time               (seconds, -t) unlimited
      max user processes              (-u) 4096
      virtual memory          (kbytes, -v) unlimited
      file locks                      (-x) unlimited


## 2.7 变量内容的删除、取代与替换

变量除了可以直接配置来修改原本的内容以外，还可以对变量的内容进行删除、取代与替换

### (1) 变量内容的删除与代替

      例1： 对于\#和\##的使用
      [tanzitao@manager SCOW]$ var="123 1234 12345"   <-- 初始化变量
      [tanzitao@manager SCOW]$ echo ${var#*3}         <-- 使用 "#" 检索变量 "*"代表任何内容且不限制
      1234 12345

对于变量：${var#*3}
\${}: 关键词：再删除模式必须存在
var:  原本变量的内容
\#:   删除or替代的关键字："#"代表从变量内容最前面开始删除，且只删除最短的部分
\*3:  需要删除的部分 "*"是通配符

      例子2： #和##的区别
      对于%和%%的使用
      [tanzitao@manager SCOW]$ echo ${var#*3}
      1234 12345
      [tanzitao@manager SCOW]$ echo ${var##*3}
      45
    
      #：删除从头开始搜索符合变量最短的哪一个
      ##：删除从头开始搜索符合变量最长的哪一个
    
      例子3：对于%和%的使用
      [tanzitao@manager SCOW]$ echo ${var%3*}
      123 1234 12
      [tanzitao@manager SCOW]$ echo ${var%%3*}
      12
    
      %：删除从尾开始搜索符合变量最短的哪一个
      %%：删除从尾开始搜索符合变量最长的哪一个
    
      例子4： 对于对于/和//的使用
      [tanzitao@manager SCOW]$ echo ${var/3/9}
      129 1234 12345
      [tanzitao@manager SCOW]$ echo ${var//3/9}
      129 1294 12945

|                      变量配置方式                       |                             内容                             |
| :-----------------------------------------------------: | :----------------------------------------------------------: |
|         \${变量#关键词} <br/> \${变量##关键词}          | 变量内容从头开始寻找数据，若符合关键词，则将符合的最短数据删除 <br/> 变量内容从头开始寻找数据，若符合关键词，则将符合的最长数据删除 |
|         \${变量\%关键词} <br/> \${变量%%关键词}         | 变量内容从尾开始寻找数据，若符合关键词，则将符合的最短数据删除 <br/> 变量内容从尾开始寻找数据，若符合关键词，则将符合的最长数据删除 |
| \${旧字符串/新字符串} <br/> \${变量//旧字符串/新字符串} | 若变量内容符合旧字符串，则将第一个旧字符串替换为新字符串 <br/> 若变量内容符合旧字符串，则将所有旧字符串替换为新字符串 |

### (2) 变量的测试与内容替换

在某些时候，我们常常会需要判断某个变量是否存在，如果变量存在就使用既有的配置，如果变量不存在就给予一个常用的配置。

      例子1： 测试一下是否存在 username 这个变量，若不存在则给予 username 内容为 root
      [tanzitao@manager SCOW]$ echo $username               <-- 读取空变量
    
      [tanzitao@manager SCOW]$ username=${username-root}    <-- 如果username有内容就保留，如果不存在就赋值为"root"
      [tanzitao@manager SCOW]$ echo $username
      root`                                                 <-- 赋值为root
      [tanzitao@manager SCOW]$ username="tzt"               <-- 变量赋值
      [tanzitao@manager SCOW]$ username=${usern ame-root}   <-- 如果username有内容就保留，如果不存在就赋值为"root" 
      [tanzitao@manager SCOW]$ echo $username
      tzt

对于命令：new_var=${old_var-content}
new_var: 新的变量，主要是来取代旧变量，一般来说名称是一样的
\${}: 关键词部分，必须要存在
old_var: 旧的变量，被测试的项目
content: 变量的内容，这部分是在给予为配置变量的内容

不过这样的配置也有可能会出现问题，就是说username可能已经被配置为了 **空字符串** 不过只需要加入一点小改动就能避免这种情况的发生

      例子2：若 username 未配置或为空字符串，则将 username 内容配置为 root
      [tanzitao@manager SCOW]$ username=""                  <-- 变量配置空字符串
      [tanzitao@manager SCOW]$ username=${username-root} 
      [tanzitao@manager SCOW]$ echo $username
                                          <-- 因为username 被配置为了空字符串，所以保留为空字符串
      [tanzitao@manager SCOW]$ username=${username:-root}
      [tanzitao@manager SCOW]$ echo $username
      root                          <-- 加上 ":" 后边亮的内容若是空或者未配置都能够被后面的内容替换

