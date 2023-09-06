---
title: Docker搭建个人ChatGPT(潘多拉)
date: 2023-07-05 14:06:33
author: Taniz Hugo
language: zh-CN
abbrlink: other1
top: false
is_draft: false
summary: 因为种种原因，国内的没有方法访问ChatGPT，我身边也有许多朋友也想用，于是我就用自己的服务器，搭建了个FakeGPT，让他们也能涌上
categories: 
  - AI
tags:
  - ChatGPT
  - Docker
  - 潘多拉
---



#### 安装Docker

```shell
curl -fsSL https://get.docker.com | sh
```



#### 启动ChatGPT容器

```shell
# 开发8899端口访问
ufw allow 8899

docker run  -e PANDORA_CLOUD=cloud -e PANDORA_SERVER=0.0.0.0:8899 -p 8899:8899 -d pengzhile/pandora
```



#### 域名添加记录

打开[cloudflare](https://dash.cloudflare.com/)  域名->DNS->记录
添加一条A类型的DNS名称

![image-20230814200348017](https://s2.loli.net/2023/08/21/NpziYf7yjcolGEX.png)



#### 申请域名

```shell
#安装证书工具：
curl https://get.acme.sh | sh; apt install socat -y || yum install socat -y; ~/.acme.sh/acme.sh --set-default-ca --server letsencrypt

#添加软链接：
ln -s  /root/.acme.sh/acme.sh /usr/local/bin/acme.sh

#切换CA机构： 
acme.sh --set-default-ca --server letsencrypt

# 开放80和443端口提供访问
ufw allow 80
ufw allow 443

#申请证书方式1： 
acme.sh  --issue -d 替换为你的域名 --standalone -k ec-256
#申请证书方式2： 
acme.sh --register-account -m "${RANDOM}@chacuo.net" --server buypass --force --insecure && ~/.acme.sh/acme.sh  --issue -d 你的域名 --standalone -k ec-256 --force --insecure --server buypass
#申请证书方式3：
acme.sh --register-account -m "${RANDOM}@chacuo.net" --server zerossl --force --insecure && ~/.acme.sh/acme.sh  --issue -d 你的域名 --standalone -k ec-256 --force --insecure --server zerossl

#安装证书：
acme.sh --install-cert -d 你的域名 --ecc --key-file /opt/cert/gpt-server.key --fullchain-file /opt/cert/gpt-server.crt
```



#### 通过Nginx反向代理

创建Nginx目录结构

```shell
mkdir -p /opt/nginx/
mkdir -p /opt/cert/
touch /opt/nginx/nginx.conf
```

修改**nginx.conf**内容

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
    client_max_body_size 1000m;  # 上传限制参数1G以内文件可上传

    server {
        listen 80;
        server_name www.ddd.com;  # 你的域名
        return 301 https://$host$request_uri;
    }

    server {
        listen 443 ssl http2;  # 你的域名
        server_name www.ddd.com;

        ssl_certificate /etc/cert/server.crt;  # 证书位置
        ssl_certificate_key /etc/cert/server.key;  # 证书位置

        location / {
            proxy_pass http://{本机IP}:8899;  # 反向代理本地ChatGPT容器，端口一般为8899
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }

        location ~ .*\.(js|css)$ {
            proxy_pass http://{本机IP}:8899;  # 加载js和css文件
        }
    }
}

```



#### 启动Nginx容器

```shell
# 做好挂载和映射启动容器
docker run -d --name nginx-continer -p 80:80 -p 443:443 \
	-v /opt/nginx/nginx.conf:/etc/nginx/nginx.conf \
	-v /opt/cert:/etc/cert \
	-v /opt/nginx/html:/etc/nginx/html \
	nginx:latest
```



#### 访问ChatGPT

浏览器访问域名 ：gpt.xxx.com

![image-20230814200821845](https://s2.loli.net/2023/08/21/pxPNVOrAbcLeEkI.png)



#### 获取Token

访问[ChatGPT官网](http://chat.openai.com/api/auth/session)，获取自己的 Access TOKEN：

![image-20230814201310998](https://s2.loli.net/2023/08/21/MyJItHrnsuR9Cel.png)



#### 完毕

填入Token，即可完成

如果访问失效：
重新申请Token或chatgpt官方发送一条信息

