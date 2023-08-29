---
title: Slurm的基本指令
date: 2022-08-08 10:15:22
author: Taniz Hugo
language: zh-CN
timezone: Asia/Shanghai
abbrlink: slurm1
top: false
tags: 
    - HPC
    - Slurm
summary: Slurm
categories: HPC
---

这是我在公司实习，参与HPC项目开发时候所接触的知识。
`Slurm`是一个开源的作业调度和集群管理系统。它被设计用于管理大规模的计算集群，如超级计算机、大规模服务器集群等，以实现资源的高效利用和作业的调度。它提供了命令行和可视化界面，使管理员和用户能够管理和监控集群资源的使用情况。

关于理论我可能懂得不多，下面是我个人在使用过程中整理出来，非常常用的一些命令，能够快速上手`Slurm`，或者是帮我自己回顾这部分知识。

### 一些文件的位置

* slurmd、slurmdbd、slurmctld日志文件: /var/log/
* slurmdbd服务的配置文件: /etc/slurm/slurmdbd.conf 
* slurmctld、slurmd服务的配置文件: /etc/slurm/slurm.conf  
* 建立slurmctld服务存储其状态等的目录，由 **slurm.conf** 中 `StateSaveLocation`参数定义： /var/spool/slurmctld

### 服务质量(QOS)配置

对资源进行限制，比如限制用户的单个作业CPU核数、运行中作业总CPU核数、作业时长等，除了利用队列分区等外，还可以利用服务质量(Quality of Service, QOS)。利用Slurm提交作业时，可以为每个作业赋予一个QOS，与作业相关的QOS有三种途径：

* 作业调度优先级：Job Scheduling Priority (qos中的参数`Priority`) priority越大优先级越高

* 作业抢占：Job Preemption (slurm.conf)

* 作业限制：Job Limits (QOS设置)

**部署服务的命令**



### service：服务

slurmdbd(记账服务)
slurmctld(控制管理服务)
slurmd(作业调度服务)

**管理结点** 需要开启以上三种服务， **计算节点** 只需要开启`slurmd 作业调度服务`即可

### NODESTATE：节点状态

节点的状态可以通过命令：**sinfo** 进行查看

节点的状态包括：

* allocated、alloc：已分配，正在使用
* completing、comp：已完成
* idle：未分配，可使用
* mixed：混合，节点在运行作业，但有些空闲 CPU 核，可接受新作业
* drained、drain：故障
* down：宕机
* fail：已失败
* reserved、resv：资源预留
* unknown、unk：未知原因
* 如果状态带有后缀 *，表示节点没有响应

### JOBSTATE：作业状态

作业状态可以通过命令：**squeue** 查看最近正在运行以及正在排队的任务
也可以通过命令：**sacct** 查看今天一天的任务

任务的状态包括：

* R\RUNNING：正在运行
* PD\PENDING：正在排队
* CG\COMPLENTING：一个无法终止的作业
* CD\COMPLETED：已完成
* FAILED：执行失败


## 2. sacctmgr：用于查看和修改Slurm账户信息

**sacctmgr** 用于查看或者修改Slurm账户信息，Slurm账户信息在数据库中维护，接口由slurmdbd（Slurm Database守护程序）提供，该数据库可以作为单个站点上多台计算机的用户和计算机信息的中央仓库。基于形成所谓的关联的四个参数来记录Slurm账户信息
记录的Slurm账户信息包括：用户、集群、分区和账户

* 用户（user）：系统登录名
* 集群（cluster）：集群名称，是在slurm.conf配置的参数：**ClusterName**
* 分区（partition）：该Slurm集群上的分支
* 账户（account）：银行账户，计费所用

**sacctmgr** 的命令格式一般是 **sacctmgr[options][command]**

## 3. sbatch：用于批处理作业提交。此脚本一般也可以含有一个或者多个srun命令启动并行任务

准备作业脚本然后通过 `sbatch` 提交是slurm提交任务的最常用手段，为了将作业脚本提交给作业系统，slurm使用

    [root@master tanzitao_slurm]# sbatch while.slurm <-- 其中 while.slurm 是自己编写的脚本任务

### 脚本任务内容

|              **options**              | **function**                                                 | **case**                                                     |
| :-----------------------------------: | ------------------------------------------------------------ | ------------------------------------------------------------ |
|         -D,--chdir [work_dir]         | 指定该任务的工作目录<br/>直接影响提交Job的目录               | #SBATCH -D /opt/work `需要运行的job在work目录下`             |
|        -J,--job-name [jobname]        | 指定作业名                                                   | #SBATCH -J test `此作业名称为 test`                          |
|        -A,--account [account]         | 指定计费账户                                                 | #SBATCH -A aaa `指定计费账户为 aaa`                          |
|       -p,--partition [parition]       | 指定作业队列                                                 | #SBACTH -p master `指定作业工作队列在 master`                |
|        -o,--output [file_name]        | 输出文件重定向                                               | #SBATCH -o test.out `内容输出到:test.out`                    |
|        -e,--error [file_name]         | 错误信息文件重定向                                           | #SBATCH -o test.err `报错信息输出到:test.err`                |
| -t,--time [time] `(xx:xx:xx)个人推荐` | 作业最大运行时间                                             | #SBATCH -t 00:00:10 `作业最多运行10s`                        |
|         -N,--nodes [num_node]         | 作业申请节点数                                               | #SBATCH -N 2 `申请两个节点完成作业`                          |
|         -w,--nodelist [hosts]         | 指定作业节点（可多个）                                       | #SBATCH -w master,node[01-02] `指定作业偏好在master、node01和node02上执行`<br/> **ps：作业申请节点数需要对于为n** |
|         -x,--exclude [hosts]          | 指定作业排除的作业节点（可多个）                             | #SBATCH -x master,node01 `指定作业不在master和node01两个节点上执行` |
|          -q,--qos [qos_name]          | 指定qos服务                                                  | #SBATCH -q normal `指定名为 normal的qos服务`                 |
|       --ntask-per-node [number]       | 每个节点上运行的任务数量n，默认情况下可以理解为：<br/>每个节点使用的n个核心<br/>如果程序不支持多线程，这个数不应该超过1 | #SBATCH -n 1 `每个节点上运行的任务数量为n`                   |
|     -c,--cpus-per-task [num_cpu]      | 作业请求cpus数                                               | #SBATCH -c 2 `指定作业使用两个cpu核心`                       |
|             --gres [list]             | 指定作业使用gpu这类资源                                      | #SBATCH --gres=gup:2 `指定作业使用两块gpu`                   |
|          --priority [level]           | 指定作业的运行优先等级                                       | #SBATCH --priority 10 `指定作业优先级为10`                   |

每个 **options** 前面都需要跟一个 `#SBATCH` ，以下是一个脚本的例子

    #!/bin/bash                 <-- 必须引入Bash程序
    #SBATCH -J while            <-- 作业名 while
    #SBATCH -o while.out        <-- 作业输出 while.out 文件
    #SBACTH -e while.err        <-- 作业错误信息输出 while.err
    #SBATCH -p master           <-- 指定分区\队列 master
    #SBATCH -w node01           <-- 指定作业运行偏好 node01 节点
    #SBATCH -N 1                <-- 作业申请节点数为1
    
    python while.py             <-- 执行的作业内容


## 4. scontrol：显示或设定slurm作业、队列、结点等状态

### 查看信息

**sontrol show** 是slurm查询内容的开头，以此可以查询许多的内容

* scontrol show partition [PARTITION-NAME]（可忽略）: 查看分区信息
* scontrol show node [NODE-NAME]（可忽略） : 查看指定节点信息
* scontrol show job [JOBID] : 查看某个任务信息

### 更新作业

在任务开始之前发现作业的属性写错了（例如提交错分区，分配资源有误），如果取消重新排队似乎很不划算。如果作业恰好 **正在排队** 可以通过 `scontrol` 命令更新作业的属性

    scontrol update jobid=JOBID ... #...为下面参数
    reqnodelist=<nodes>
    reqcores=<count>
    name=<name>
    nodelist=<nodes>
    excnodelist=<nodes>
    numcpus=<min_count-max_count>
    numnodes=<min_count-max_count>
    numtasks=<count>
    starttime=yyyy-mm-dd
    partition=<name>
    timelimit=d-h:m:s
    mincpusnode=<count>
    minmemorycpu=<megabytes>
    minmemorynode=<megabytes>

## 5. sacct：显示激活的或者已经完成作业或作业步的记账信息

**查询所有任务信息（默认只能查询一天的任务）**

    [root@master slurm]# sacct --allocations
           JobID    JobName  Partition    Account  AllocCPUS      State ExitCode
    ------------ ---------- ---------- ---------- ---------- ---------- --------
    753               while     master       root          1  CANCELLED     0:15
    754               while      test1       root          1 CANCELLED+      0:0
    755               while      test2       root          1     FAILED      2:0

**查询指定时间段所有任务信息**

    [root@master slurm]#sacct --starttime 2022-08-10T00:00:00 --endtime 2022-08-10T22:00:00
           JobID    JobName  Partition    Account  AllocCPUS      State ExitCode
    ------------ ---------- ---------- ---------- ---------- ---------- --------
    751.batch         batch                  root          1     FAILED      2:0
    752               while     master       root          1 CANCELLED+      0:0
    752.batch         batch                  root          1  CANCELLED     0:15

**查询详情作业详情**

    [root@master tanzitao_slurm]# sacct --format=JobId,state,user,account,partition,jobname,nodelist,nnodes,submit,start,end,suspended,ncpus,MaxRss,elapsed,allocgres
        JobID      State      User    Account  Partition    JobName        NodeList   NNodes              Submit               Start                 End  Suspended      NCPUS     MaxRSS    Elapsed    AllocGRES
    ------------ ---------- --------- ---------- ---------- ---------- --------------- -------- ------------------- ------------------- ------------------- ---------- ---------- ---------- ---------- ------------
    753           CANCELLED      root       root     master      while          master        1 2022-08-11T09:50:21 2022-08-11T09:50:21 2022-08-11T09:59:55   00:00:00          1              00:09:34

## 6. sinfo：查看集群状态(常用)

|     **options**      | **function**         | **case**                                                     |
| :------------------: | -------------------- | ------------------------------------------------------------ |
|        sinfo         | 查看总体资源信息     | <img src="https://s2.loli.net/2023/08/20/ij62LtvPUWHsqez.png"/> |
|       sinfo -N       | 查看节点信息         | <img src="https://s2.loli.net/2023/08/20/mVCDLYqaiPGpEFZ.png"/> |
| sinfo -t=[NODESTATE] | 查看指定节点状态信息 | <img src="https://s2.loli.net/2023/08/20/Jh5pWlCqXPwBMcj.png"/> |
| sinfo -p=[PARTITION] | 查看指定队列信息     | <img src="https://s2.loli.net/2023/08/20/aSTHqLFCmhV1uU2.png"/> |


## 7. squeue：查看作业信息(常用)

|     **options**      | **function**         | **case**                                                     |
| :------------------: | -------------------- | ------------------------------------------------------------ |
|        squeue        | 查看所有作业信息     | <img src="https://s2.loli.net/2023/08/20/Jh8XPLkN7OiSwHc.png"/> |
|      squeue -l       | 查看所有作业更多细节 | <img src="https://s2.loli.net/2023/08/20/Ml8LSxQXRI3Nym1.png"/> |
| squeue -n [TAKENAME] | 查看特定节点作业信息 | <img src="https://s2.loli.net/2023/08/20/gPjmc51SnXNGoAV.png"/> |
|  squeue -j [JOBID]   | 查看指定作业信息     | <img src="https://s2.loli.net/2023/08/20/TiRmr6dohOwQFNs.png"/> |
| squeue -t=[JOBSTATE] | 查看指定状态作业     | <img src="https://s2.loli.net/2023/08/20/m8SwyX92okBHnV3.png"/> |






