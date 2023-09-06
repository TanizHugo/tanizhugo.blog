---
title:  Nginx(重定向 反代 静态页面)
date: 2023-06-03 19:09:33
author: Taniz Hugo
language: zh-CN
abbrlink: project1
top: false
is_draft: false
summary: Nginx是非常常用的开源Web服务器和反向代理服务器，在本章我收集了三种基本用法的配置文件
categories: 
  - 项目
tags:
  - Nginx
  - 重定向
  - 反向代理
  - 静态页面代理
  - Linux
  - 配置文件
---

#### 前言

Nginx的功能我想不用过多赘述，接触过WEB开发的人基本都不会陌生，开源且好用。

本篇直接给出干货，Nginx三种常用方法的配置文件的方式

以下命令操作环境为`Ubuntu 22.04 x86_64`

#### 安装Nginx

Nginx已经有软件包，存在APT仓库，因此通过系统命令下载即可

```shell
sudo apt install nginx
sudo systemctl start nginx

```

开放80和443的防火墙（分别用于HTTP和HTTPS）

```shell
ufw allow 443
ufw allow 80
```



#### 重定向配置，跳转到域名

这个很好理解，就是将自己的域名指向别人的域名，通过将客户端请求重定向到新的URL来实现。

##### 主要应用

* **HTTP到HTTPS重定向**：确保用户的数据在传输过程中得到加密保护，增强网站的安全性。
* **域名规范化**：通过将不同形式的域名重定向到一个标准的主要域名，例如在浏览器输入`www.baidu.com`和`baidu.com`都会跳转到`www.baidu.com`。
* **处理旧链接**：将已更改的URL或者已删除的页面重新定向到新的页面或者资源。
* **防止恶意请求**：将来自恶意爬虫的请求重定向到一个空白页面，以减轻不必要的服务器负担。

##### 配置文件

以下是重定向的配置文件，自行修改具有备注的地方

```shell
user nginx;
worker_processes auto;

error_log /var/log/nginx/error.log;
pid /var/run/nginx.pid;

events {
    worker_connections 1024;
}

http {
    server {
        listen 80;
        server_name www.ddd.com;   # 你的域名
        return 301 https://www.google.com$request_uri;    # 跳转域名
    }

    server {
        listen 443 ssl http2;
        server_name www.ddd.com;   # 你的域名

        ssl_certificate /etc/cert/server.crt;   # 证书位置
        ssl_certificate_key /etc/cert/server.key;   # 证书位置

        return 301 https://www.google.com$request_uri;    # 跳转域名
    }
}

```



#### 反向代理配置，代理指定IP加端口

反向代理的主要目的是在客户端和服务器之间充当中间层，Nginx充当一个代理人，辅助我们的服务器更好的与客户端连接。

##### 主要应用

* **负载均衡**：分发客户端请求到多个后端服务器，以均衡负载。这有助于确保每个服务器都能够有效处理请求，而不会过载，提高了网站的性能和可用性。
* **提高性能**：通过缓存、压缩、连接池管理等技术，提高性能。它可以缓存静态内容，减少对后端服务器的请求，加速响应时间。
* **SSL终端**：充当SSL终端，处理HTTPS连接。
* **高可用性**：通过设置多个后端服务器，反向代理可以实现高可用性配置。如果一个服务器出现故障，反向代理可以自动将流量切换到另一个可用的服务器上。

##### 配置文件

以下是反向代理的配置文件，自行修改具有备注的地方

```shell
user nginx;
worker_processes auto;

error_log /var/log/nginx/error.log;
pid /var/run/nginx.pid;

events {
    worker_connections 1024;
}

http {
    client_max_body_size 1000m;    # 上传限制参数1G以内文件可上传

    server {
        listen 80;
        server_name www.ddd.com;    # 你的域名
        return 301 https://$host$request_uri;
    }

    server {
        listen 443 ssl http2;
        server_name www.ddd.com;

        ssl_certificate /etc/cert/server.crt;    # 证书位置
        ssl_certificate_key /etc/cert/server.key; # 证书位置

        location / {
            proxy_pass http://{本机IP}:{端口};     # 反向代理的地址（代理本地服务不要用127.0.0.1)
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }

        location ~ .*\.(js|css)$ {
            proxy_pass http://{本机IP}:{端口};     # 加载js和css文件
        }
    }
}

```



#### 代理静态网页的配置方式

代理静态页面就更常见了，比如我们自己的写的网站，就可以用Nginx进行代理。

##### 主要应用

* **加速静态资源访问**：Nginx可以缓存和代理静态资源，如HTML、CSS、JavaScript、图像和视频等。通过代理这些静态资源，Nginx可以更快地响应客户端请求。
* **节省服务器资源**：前后端分离项目常用，通过将静态资源的处理交给Nginx，后端服务器可以专注于处理动态内容和业务逻辑，从而充分利用服务器资源。

##### 配置文件

以下是代理静态网页的配置文件，自行修改具有备注的地方

```shell
user nginx;
worker_processes auto;

error_log /var/log/nginx/error.log;
pid /var/run/nginx.pid;

events {
    worker_connections 1024;
}

http {
    server {
        listen 80;
        server_name www.ddd.com; # 你的域名
        return 301 https://$server_name$request_uri;
    }

    server {
        listen 443 ssl http2;
        server_name www.ddd.com; # 你的域名

        include /etc/nginx/default.d/*.conf;
        add_header Access-Control-Allow-Origin '*';
        add_header Access-Control-Allow-Methods 'GET, POST, OPTIONS, PUT, DELETE';
        add_header Access-Control-Allow-Headers 'DNT,X-Mx-ReqToken,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Authorization,X-Total-Count';
        add_header Access-Control-Expose-Headers 'X-Total-Count';

        ssl_certificate /etc/cert/server.crt; # 证书位置
        ssl_certificate_key /etc/cert/server.key; # 证书位置
        charset utf-8; # 添加这行来指定编码

        location / {
            # 对于css文件解析进行配置
            include mime.types;
            default_type application/octet-stream;

            root /etc/nginx/html/dist; # 静态页面目录
            index index.html; # 静态页面名字
        }

        error_page 404 /404.html;
        location = /404.html {
        }

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
        }
    }
}

```



以上就是Nginx三种常见用法的配置文件，应该能够满足一般的要求，作为新手上路算是比较友好的简单的了。