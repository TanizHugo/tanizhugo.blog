
---
title: (四)Bash shell 的操作环境
date: 2022-08-04 16:10:57
author: Taniz Hugo
language: zh-CN
abbrlink: linux4
top: false
is_draft: false
summary: 继续跟着鸟哥学Linux,第四章
categories: 
  - Linux
tags:
  - Linux指令
---


每当我们登录主机的时候，屏幕上总会有一些说明文字，有的会告诉我们上一次登录是什么时候如图 4.1 

<center>
<img src="https://i.imgs.ovh/i/2023/08/20/64e1fda5647a4.png"/><br/>
<font size=2>图4.1</font>
</center>


其实聪明的你肯定会想到，这些信息都是可以设置的，因此我们也可以设置欢迎文字等等。此外，在我们开发的过程中，某些运行环境所需的变量以及为了方便开发的命名别名，是不是都可以在登录的时候就主动地帮我们配置好？

由此进入本章的学习当中，Bash shell 的操作环境

## 4.1 路径与命令搜寻顺序

在前面的学习 alias 与 bash 的内建命令过程中，我们都知道系统里其实有不少的ls命令，或者是包括内建的echo命令，那我们是否有过疑惑：Bash怎么知道到底拿那一条命令来执行？

其实在Bash里面命令的运行顺序可以这样看：
(1) 以相对/绝对路径运行命令，例如 [ /bin/ls ] 或者 [ ./ls ];
(2) 有 alias 找到命令来运行;
(3) 由 bash 内建的 (builtin) 命令来运行;
(4) 透过 $PATH 这个变量的顺序搜寻到第一个命令来运行;

如图4.2，当我们直接运行 **/bin/ls** 会发现使用 **ls** 没有用颜色区分文件和文件夹，但是我们直接运行 **ls** 就有颜色区分。

<center>
<img src="https://i.imgs.ovh/i/2023/08/20/64e1fda4519ba.png"/><br/>
<font size=2>图4.2</font>
</center>


这是因为 **/bin/ls** 是直接去用该命令下达，而 **ls** 会因为 [ alias 'ls --color=auto' ] 这个命令别名而先使用

如果想要了解命令的搜寻的顺序，其实可以透过 **type** 命令进行查询

    # 例子1：查看 "echo" 命令的执行方式
    [tanzitao@master ~]$ alias echo='echo -n'
    [tanzitao@master ~]$ type -a echo
    echo 是 `echo -n' 的别名
    echo 是 shell 内嵌
    echo 是 /usr/bin/echo

## 4.2 bash的进站与欢迎信息： /etc/issue, /etc/motd

本章开始我们提到了进站会有欢迎信息，其实这些进站信息就卸载了 **/etc/issue** 这个文件里面

    # 例子2：查看 /etc/issue 文件中的欢迎内容
    [tanzitao@master ~]$ cat /etc/issue
    \S
    Kernel \r on an \m
    welcome!

具体的内容可以参考以下表，详细记录了各个代码的意义


| 代码 | 意义                              |
| :--: | --------------------------------- |
|  \d  | 本地端时间的日期                  |
|  \l  | 显示第几个终端机接口              |
|  \m  | 显示硬件登记(i386/i486/i586/i686) |
|  \n  | 显示主机的网络名称                |
|  \o  | 显示 domian name                  |
|  \r  | 操作系统的版本(相当于uname -r)    |
|  \t  | 显示本地端时间的时间              |
|  \s  | 操作系统的名称                    |
|  \v  | 操作系统的版本                    |

不过其实这些都无伤大雅，设置来美化一下自己的系统也不错

## 4.3 环境配置文件: 

在我们的Bash系统当中有一些环境配置文件的存在，让Bash在启动时直接读取这些配置文件，用来正常启动我们的Bash。而这些配置文件又可以分为 **全体系统的配置文件** 以及 **用户个人偏好配置文件**。 在前面小几章提到的命名别名与自定义变量都会在我们注销Bash后失效，因此如果我们要长期保留这些内容，就必须写入到配置文件当中才行。

### login与non-login shell

* login shell：取得Bash时，需要完整的登录流程的（完整登录流程指需要输入用户名和密码），就称为loigin shell

* non-login shell：取得Bash接口的方法就不需要重复登录，举个简单的例子：在bash当中开启另一个bash（子线程）的时候，就不是完整的登录流程，这样就称为non-login shell了

### /etc/profile

**/etc/profile** 这个配置文件里面有许多重要的变量数据，这个文件在每个用户登录取得Bash程序后都回去读的配置文件（仅限于 login shell登录）
因此如果我们想修改我们配置的环境就是改这个部分。但是！千万不要乱改，下表展示了几个重要的配置文件

|  变量名  | 描述                                                        |
| :------: | ----------------------------------------------------------- |
|   PATH   | 回一句UID决定PATH变量要不要包含有 **sbin** 的系统命令目录   |
|   MAIL   | 依据账号配置好使用者的mailbox 到 **/var/spool/main/账户名** |
|   USER   | 根据用户的账号配置此变量的内容                              |
| HOSTNAME | 依据主机的hostname配置此变量的内容                          |
| HISTSIZE | 历史记录保留的最大数量上限                                  |

总结来说，我们只需要记住，用户每次登录Bash都会从 **/etc/profile**里面获取配置，并且在里面还会呼叫其他的配置文件，别轻易乱动这个文件就对了！

### ~/bash_profile

Bash在读完了 **/etc/profile** 的整体环境配置后，接下来就是读取使用者user个人的配置文件（仅限于 login shell登录），主要分为一下三个，依次的顺序分别是：

  1. ~/.bash_profile
  2. ~/.bash_login
  3. ~/.profile

其实Bash的login shell配置只会读取上面三个文件的其中一个，而读取的顺序就是按上面所说，举例来说：只要读取 ~/.bash_profile 成功，剩下的两个文件就不会去读取

    # 例子1：查看 ~/.bash_profile 文件的内容
    [root@master ~]# cat ~/.bash_profile
    # .bash_profile
    
    # Get the aliases and functions
    if [ -f ~/.bashrc ]; then          <-- 判断并读取 ~/.bashrc
            . ~/.bashrc
    fi
    
    # User specific environment and startup programs
    
    PATH=$PATH:$HOME/bin:/root/miniconda/bin    <-- 下面的内容是在处理个人配置
    export PATH
    WORKON_HOME=~/.virtualenvs
    export WORKON_HOME
    source /usr/local/bin/virtualenvwrapper.sh

发现了吗？在个人配置下存在 **PATH** 这个变量，和我们在 **/etc/profile** 下的PATH变量是不冲突的，而是属于累加形式，合并在一起，这也意味着如果我们将运行的文件放在自家目录下 **~/bin** 程序也是可以被执行的，这就不用次次都使用相对/绝对路径运行程序

在 **./bash_profile** 文件中存在一段用 **if...then...** 完成的配置，这一段的内容就是说：如果在家目录下 **~/.bashrc** 文件存在，就读取 **~/bashrc** 文件

最后我们说了这么多，login shell的读取流程如图4.3

<center>
<img src="https://i.imgs.ovh/i/2023/08/20/64e1fda5648bd.png"/><br/>
<font size=2>图4.3</font>
</center>


### soucrce

在上述所提到 Bash当中 **/etc/profile** 全用户配置文件和 **~/.bashrc** 本用户配置文件都是在 login shell的时候才会去读取，当我们将自己的编号配置写入上述文件后，通常都需要注销后再等了，配置才会生效。那我们可不可以跳过注销登录的步骤，直接使他生效，那就需要用到 **source** 这个变量命令了

你会疑惑：这有什么用？
当我们需要用到不同的配置环境文件的时候，一个人需要负责不同的任务开发，每个任务的环境都不一样，那么我们可以每一个环境都创建属于自己的 source变量文件，操作之前直接soucrce一下，就方便啦。

###  ~/.bashrc

说完了 login shell的情况后，接下来我么聊一聊 non-login shell非登录的情况Bash在 non-login shell的时候，读取的配置文件只有 **~/.bashrc** 

    # 例子2：查看 ~/.bashrc 的内容
    [tanzitao@master ~]$ cat ~/.bashrc
    # .bashrc
    
    # User specific aliases and functions
    alias rm='rm -i'
    alias rp='cp -i'
    alias mv='mv -i'


    # Source global definitions
    if [ -f /etc/bashrc ]; then
            . /etc/bashrc
    fi
    
    # Uncomment the following line if you don't like systemctl's auto-paging feature:
    # export SYSTEMD_PAGER=
    
    # User specific aliases and functions
    
    alias sudo='sudo env PATH=$PATH'

特别注意root用户和一般用户下面的 **~/.bashrc** 内容是不一样的


## 4.4 终端机的环境配置： stty, set

stty是什么，我们且按下不表，直接使用命令： **stty -a** 将所有参数打印出来

    # 例子1：列出所有的按键与按键内容
    [tanzitao@master ~]$ stty -a
    speed 9600 baud; rows 30; columns 102; line = 0;
    intr = ^C; quit = ^\; erase = ^?; kill = ^U; eof = ^D; eol = <undef>; eol2 = <undef>; swtch = <undef>;
    start = ^Q; stop = ^S; susp = ^Z; rprnt = ^R; werase = ^W; lnext = ^V; flush = ^O; min = 1; time = 0;
    -parenb -parodd -cmspar cs8 -hupcl -cstopb cread -clocal -crtscts
    -ignbrk -brkint -ignpar -parmrk -inpck -istrip -inlcr -igncr icrnl ixon -ixoff -iuclc -ixany -imaxbel
    -iutf8
    opost -olcuc -ocrnl onlcr -onocr -onlret -ofill -ofdel nl0 cr0 tab0 bs0 vt0 ff0
    isig icanon iexten echo echoe echok -echonl -noflsh -xcase -tostop -echoprt echoctl echoke


stty 就是可以帮配置我们的终端机的输入按键，举个例子来说: **^** 代表 [Ctrl] 按键，**intr = ^C** 就代表 **intr命令** 通过 **[Ctrl] + C** 进行了代替

  * eof:End of file代表输入结束
  * erase：向后删除字符
  * intr：给出一个终端的信号到正在run的程序
  * kill：删除在目前命令上的所有文字
  * quit：送出一个quit的讯号目前正在run的程序
  * start：在某个程序停止后，重新启动他的output
  * stop：停止目前屏幕的输出
  * susp：送出一个 terminal stop 的信息给到正在run的程序

一下列出Bash默认的组合键：

| 组合按键 | 运行结果                            |
| :------: | ----------------------------------- |
| Ctrl + C | 终止目前的命令                      |
| Ctrl + D | 输入结束（EOF），例如邮件结束的时候 |
| Ctrl + M | 就是Enter                           |
| Ctrl + S | 暂停屏幕的输出                      |
| Ctrl + Q | 恢复屏幕的输出                      |
| Ctrl + U | 在提示字符下，将整列命令删除        |
| Ctrl + Z | [暂停]目前的命令                    |

## 4.5 通配符与特殊字符

在Bash当中如果我们需要分析任务，查看任务状态等，为了简简便内让那个那我们少不了的就是用通配符协助我们处理，以下是一些常用的通配符：

|        符号         | 意义                                                       |
| :-----------------: | ---------------------------------------------------------- |
|          *          | 代表[0个到无穷个]任意字符                                  |
|         ？          | 代表[有且只有一个]字符                                     |
|         []          | 代表 [在括号内一定有一个] 的字符：[abc]一定有一个字符      |
| 可能是a,b,c中的一个 |                                                            |
|         [-]         | 代表 [在编码顺序内的所有字符]：[0-9]代表0到9之间的所有数字 |
|         [^]         | 代表 [反向选择]：[^abc]一定有一个不在abc中的               |

    # 例子1：查找所有 txt文件
    [tanzitao@master ~]$ ls *.txt
    123.txt  456.txt  abc.txt  chpass.txt
    # 例子2：查找所有 带数字的txt文件
    [tanzitao@master ~]$ ls *[0-9].txt
    123.txt  456.txt
    # 例子3：查找所有 3个字符的txt文件
    [tanzitao@master ~]$ ls ???.txt
    123.txt  456.txt  abc.txt   
    # 例子4：查找所有 不带数字的txt文件
    [tanzitao@master ~]$ ls *[^0-9].txt
    abc.txt  chpass.txt

除此以外，以下还汇总了Bash环境中的特殊字符

| 符号 | 内容                                                         |
| :--: | ------------------------------------------------------------ |
|  #   | 批注符号：这个最常被使用在script当中，视为说明！在后的数据均不运行 |
|  \\  | 跳脱字符：将特殊字符或者通配符还原成一般字符                 |
|  \|  | 管线(pipe):分割两个管线命令的界定                            |
|  ;   | 连续命令下达的分隔符：连续性命令的界定                       |
|  ~   | 用户的家目录                                                 |
|  $   | 取用变量前导符：就是在变量之前需要加的变量取代值             |
|  &   | 工作控制：将命令编程背景下工作                               |
|  !   | 逻辑意义上的 [非] 的意思                                     |
|  /   | 目录符号：路径分隔符                                         |
| >,>> | 数据流重导向：输出导向，分别是[取代] 和 [累加]               |
| <,<< | 数据流冲导向：输入导向                                       |
|  ''  | 不具备变量的置换功能                                         |
|  ""  | 具备变量的置换功能                                           |
|  ``  | 反引号之间有限运行的命令，可以用$()代替                      |
|  ()  | 在中间为字shell（）的起始与结束                              |
|  {}  | 在中间命令区块的组合                                         |