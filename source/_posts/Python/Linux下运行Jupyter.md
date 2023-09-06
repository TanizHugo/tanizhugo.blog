---
title: Linux下运行Jupyter
date: 2023-8-24 12:11:09
author: Taniz Hugo
language: zh-CN
abbrlink: python1
top: false
is_draft: false
summary: 闲来无事琢磨怎么将自己的本地的开发环境全部卸载，但是这不代表说我就放弃了写代码，想想自己有服务器为什么不好好利用起来呢？
categories: 
  - Python
tags:
  - Linux
  - Jupyter
  - Python
  - 开发环境
---


当我拥有了自己的第一台VPS，我就开始考虑：“我是不是应该将一些开发环境部署在我的VPS呢，这样我本地就能省下很多的存储空间，~~安装更多有意思的东西~~”。其实在开发中这是非常正常的，要将工作环境和运行环境给区分开来。

Python的开发工具很多，可我独钟的还是PyCharm和Jupyter，前者我觉得适合用于Windows和Mac系统中大的项目开发，后者适合用于Linux进行数据分析、脚本、爬虫或者学习。

![image-202308231756345](https://s2.loli.net/2023/08/23/XVsKUyCh8mLt9Tr.png)

谈到部署环境，那肯定离不开的就是`Docker`，近些找不到工作的日子里，没事就买了VPS捣腾了下Docker，发现其实还是非常实用的，那我就基于Docker来快速安装Jupyter，



#### 安装Docker

首先安装Docker到环境当中

```shell
# 一个sh的脚本，通过判断你的电脑系统自动安装新版docker，当然也可以使用系统命令安装
curl -fsSL https://get.docker.com | sh

```

如果直接拉镜像会有点慢，因此我们需要改用国内的 Docker Hub 镜像服务器

1. 编辑/etc/docker/daemon.json 配置文件

```shell
# 编辑配置文件，如果文件不存在，以下命令会自动创建。
vi /etc/docker/daemon.json

```

将配置信息粘贴到配置文件中，配置信息为 json 格式。

```shell
# 可以根据实际需要设置多个国内的镜像服务器
{
  "registry-mirrors": ["https://n14or9zx.mirror.aliyuncs.com",
  "https://mirror.ccs.tencentyun.com",
  "http://registry.docker-cn.com",
  "http://docker.mirrors.ustc.edu.cn",
  "http://hub-mirror.c.163.com"],

  "log-driver": "json-file",
  "log-opts": {
    "max-size": "500m"
  }
}
```

2. 重启Docker服务

```shell
systemctl daemon-reload 
systemctl restart docker
```

3. 检查设置是否生效

```
docker info
```

结果中显示了我们设置的镜像服务器地址，则说明设置已经生效，返回的信息类似下面这样：

```shell
 Registry Mirrors:
  https://mirror.baidubce.com/
 Live Restore Enabled: false
```



#### 创建Jupyter容器

docker创建容器，如果镜像库没有镜像会自动拉取

```shell
# 创建工作目录
mkdir -p /opt/jupyter/python3.6
chmod -R 777 /opt/jupyter/python3.6

# 设置token为123456
docker run -u root -d \
	--name jupyter-python3.6 \
	-e JUPYTER_ENABLE_LAB=yes \
	-p 8888:8888 \
	-v /opt/jupyter/python3.6:/home/jovyan/work \
	jupyter/scipy-notebook:latest start-notebook.sh --NotebookApp.token='123456'

# 或不设置token直接进入
docker run -d -u root \
	--name jupyter-python3.6 \
	-e JUPYTER_ENABLE_LAB=yes \
	-p 8888:8888 \
	-v /opt/jupyter/python3.6:/home/jovyan/work \
jupyter/scipy-notebook:latest start-notebook.sh --NotebookApp.token=''

```



#### 创建环境

进入jupyter的容器，进行接下来的操作

```shell
docker exec -it jupyter-python3.6 bash
```

使用 `conda` 安装特定版本的 Python，并加入Jupyter内核：

```shell
# 创建环境
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --set show_channel_urls yes
conda create -n req python=3.6 -y 

# 激活环境
conda activate req

# 安装 ipykernel
conda install -c conda-forge ipykernel -y
conda install jupyter_client jupyter_core -y

# 向 jupyter notebooke 添加当前环境
python -m ipykernel install --user --name=req

# Contrl + D 退出容器
```



#### 查看结果

打开浏览器输入ip:8888

![image-20230823175630374](https://s2.loli.net/2023/08/23/JxWu34BmnolCUys.png)

测试环境内核

![image-20230823234041616](https://s2.loli.net/2023/08/23/rIWOlKcYMS7TdXj.png)



下载你需要的python环境

```shell
docker exec -it jupyter-python3.6 /bin/bash -c "source activate req && pip install Matplotlib -i https://pypi.tuna.tsinghua.edu.cn/simple"

```

