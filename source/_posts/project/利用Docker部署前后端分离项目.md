---
title: 利用Docker部署前后端分离项目
tags: 项目搭建
author: Taniz Hugo
language: zh-CN
timezone: Asia/Shanghai
abbrlink: porject2
date: 2023-07-09 11:12:17
---


**本文将采用编写各模块dockerfile的方式，利用docker-compose创建镜像和容器**

这里以前端为Vue，后端为Django进行演示



#### 安装docker以及docker-compose

```shell
# 安装Docker
curl -fsSL https://get.docker.com | sh

# 安装docker-compose
apt-get install	docker-compose -y
```



#### 域名申请证书

添加DNS记录

![image-20230815205400075](https://s2.loli.net/2023/08/24/oiQ8yrfs9EAv5jR.png)

域名申请证书

```shell
#安装证书工具：
curl https://get.acme.sh | sh; apt install socat -y || yum install socat -y; ~/.acme.sh/acme.sh --set-default-ca --server letsencrypt

#添加软链接：
ln -s  /root/.acme.sh/acme.sh /usr/local/bin/acme.sh

#切换CA机构： 
acme.sh --set-default-ca --server letsencrypt

#申请证书方式1： 
acme.sh  --issue -d 替换为你的域名 --standalone -k ec-256
#申请证书方式2： 
acme.sh --register-account -m "${RANDOM}@chacuo.net" --server buypass --force --insecure && ~/.acme.sh/acme.sh  --issue -d 你的域名 --standalone -k ec-256 --force --insecure --server buypass
#申请证书方式3：
acme.sh --register-account -m "${RANDOM}@chacuo.net" --server zerossl --force --insecure && ~/.acme.sh/acme.sh  --issue -d 你的域名 --standalone -k ec-256 --force --insecure --server zerossl

#安装证书：
acme.sh --install-cert -d 你的域名 --ecc --key-file /opt/wendy/nginx/cert/wendy-server.key --fullchain-file /opt/wendy/nginx/cert/wendy-server.crt
```



#### 创建部署目录

在/opt目录下创建自己的项目文件夹

```shell
mkdir -p /opt/wendy; cd /opt/wendy
touch mysql-dockerfile python-dockerfile nginx-dockerfile docker-compose.yml
mkdir mysql-db; mkdir python
mkdir -p nginx/cert
mkdir -p nginx/web
touch nginx/nginx.conf

# 防火墙开启端口
ufw allow from any to any port 80,8000,443,3306 proto tcp

# 将项目所需的文件放在该目录下
root@vultr:ls /opt/wendy
docker-compose.yml  mysql-db  mysql-dockerfile  python  python-dockerfile nginx nginx-dockerfile

```

ps: 
mysql-db目录下存放‘.sql’文件

python目录下存放 python后端文件

nginx目录下存放 nginx.conf文件、证书、前端文件



编辑**docker-compose.yml**文件

```yaml
version: '3'
services:
  python:
    build:
      context: .
      dockerfile: python-dockerfile
    image: wendy-python-image:latest
    container_name: wendy-python-container
    ports:
      - "8000:8000"
    volumes:
      - ./python:/opt/wendy/python
    depends_on:
      - mysql

  mysql:
    build:
      context: .
      dockerfile: mysql-dockerfile
    image: wendy-mysql-image:latest
    container_name: wendy-mysql-container
    ports:
      - "3306:3306"
    volumes:
      - ./mysql-db:/docker-entrypoint-initdb.d

  nginx:
    build:
      context: .
      dockerfile: nginx-dockerfile  
    image: wendy-nginx-image:latest
    container_name: wendy-nginx-container
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./nginx/cert:/opt/wendy/nginx/cert
      - ./nginx/web:/opt/wendy/nginx/web

```



编辑mysql-dockerfile，填入以下内容

```dockerfile
# 创建mysql-dockerfile
# 使用 MySQL 5.7 镜像作为基础镜像
FROM mysql:5.7

# 设置环境变量，指定初始化数据库时的用户名、密码和数据库名称
ENV MYSQL_ROOT_PASSWORD=123456
ENV MYSQL_DATABASE=wendy_django_db

# 对外暴露 MySQL 默认端口 3306（可以省略，根据实际情况）
EXPOSE 3306

```



编辑python-dockerfile，填入以下内容

```dockerfile
#创建python-dockerfile
# 使用 Python 3.6 镜像作为基础镜像
FROM python:3.6

# 更新 pip
RUN pip install --upgrade pip

# 设置工作目录
WORKDIR /opt/wendy/python

# 复制项目文件到容器中
COPY ./python /opt/wendy/python

# 安装项目依赖
RUN pip install -r requirements.txt

# 对外暴露 8000 端口（Django需要）
EXPOSE 8000

# 在容器启动时运行的命令
CMD python3 manage.py runserver 0.0.0.0:8000

```



因为django项目需要内映射mysql容器，需要修改数据库的参数

```python
# setiing文件

ALLOWED_HOSTS = ['43.136.62.137', '0.0.0.0', '*', 'wendy-mysql-container'] 	# 添加本机ip地址 和 mysql容器

ATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'wendy_django_db',
        'USER': 'root',
        'PASSWORD': '123456',
        'HOST': 'wendy-mysql-container',	# 填写内联mysql容器名
        'PORT': '3306',
    }
}
```



编辑nginx-dockerfile，填入以下内容

```dockerfile
# 创建Nginx容器
# 使用官方的 Nginx 镜像作为基础镜像
FROM nginx:latest

# 将自定义的 Nginx 配置文件复制到容器内
COPY ./nginx/nginx.conf /etc/nginx/nginx.conf

# 将证书复制到容器内
COPY ./nginx/cert /opt/wendy/nginx/cert

# 将项目复制到容器内
COPY ./nginx/web /opt/wendy/nginx/web

# 暴露 Nginx 的默认 HTTP 端口
EXPOSE 80
EXPOSE 443

# 在容器启动时运行 Nginx 服务器
CMD ["nginx", "-g", "daemon off;"]

```



编辑nginx.conf文件

```nginx
# 全局配置
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /var/run/nginx.pid;

# 事件模块配置
events {
    worker_connections 1024;
}

http {
    # 静态页面（wendy项目）
    server {
        listen 80;
        server_name wendy.xxx.com;
        return 301 https://$server_name$request_uri;
    }

    server {
        listen 443 ssl http2;
        server_name wendy.xxx.com;

        add_header Access-Control-Allow-Origin '*';
        add_header Access-Control-Allow-Methods 'GET, POST, OPTIONS, PUT, DELETE';
        add_header Access-Control-Allow-Headers 'DNT,X-Mx-ReqToken,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Authorization,X-Total-Count';
        add_header Access-Control-Expose-Headers 'X-Total-Count';

        # 证书存放位置
        ssl_certificate /opt/wendy/nginx/cert/wendy-server.crt;
        ssl_certificate_key /opt/wendy/nginx/cert/wendy-server.key;

        charset utf-8;

        location / {
            include mime.types;
            default_type application/octet-stream;
            
            # 前端项目所在文件夹
            root /opt/wendy/web/;		
            index index.html;
        }

        error_page 404 /404.html;
        location = /404.html {}

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {}

        include /etc/nginx/default.d/*.conf;
    }
}

```



#### docker-compose建立镜像并启动容器

```shell
docker-compose up -d
```

运行成功

![image-20230816013835707](https://s2.loli.net/2023/08/24/tvSH7LgwTAMkDh3.png)











