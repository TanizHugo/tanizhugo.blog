---
title: 搭建Nginx
tags: 项目搭建
author: Taniz Hugo
language: zh-CN
timezone: Asia/Shanghai
abbrlink: porject1
date: 2023-06-03 17:34:09
---


#### 安装Docker

```shell
curl -fsSL https://get.docker.com | sh
```



#### 创建Nginx目录结构

```shell
mkdir -p /etc/nginx/html
touch /etc/nginx/nginx.conf
```



### 部署Nginx容器

```shell
docker run -d --name nginx -p 80:80 -p 443:443 \
	-v /etc/nginx/nginx.conf:/etc/nginx/nginx.conf \
	-v /etc/cert:/etc/cert \
	-v /etc/nginx/html:/etc/nginx/html \
	nginx:latest
```



#### 编辑nginx.conf文件

* 打开防火墙

```shell
ufw allow 443
ufw allow 80
```



* **全局配置**

```shell
user nginx;             # Nginx运行的用户
worker_processes auto;  # 自动检测CPU核心数，用于处理并发请求

# 错误日志和访问日志
error_log /var/log/nginx/error.log;
pid /var/run/nginx.pid;

# 事件模块配置
events {
    worker_connections  1024;
}

```



* **重定向配置，跳转到域名**

```shell
http {

  server {

    listen 80;

    server_name www.ddd.com;					 # 你的域名    

    return 301 https://www.google.com$request_uri;		# 跳转域名

  }


  server {

    listen 443 ssl http2;

    server_name www.ddd.com;					# 你的域名 

    ssl_certificate /etc/cert/server.crt;		 # 证书位置

    ssl_certificate_key /etc/cert/server.key;	 # 证书位置

    return 301 https://www.google.com$request_uri;		# 跳转域名

  }

}
```



* **反向代理配置，代理指定IP加端口**

```shell
  client_max_body_size 1000m;  	 #上传限制参数1G以内文件可上传

  server {

    listen 80;

    server_name www.ddd.com;		# 你的域名 

    return 301 https://$host$request_uri;

  }
  
  server {

    listen 443 ssl http2;			# 你的域名 

    server_name www.ddd.com;

    ssl_certificate /etc/cert/server.crt;		 # 证书位置

    ssl_certificate_key /etc/cert/server.key;	 # 证书位置

    location / {

      proxy_pass http://{本机IP}:{端口};			# 反相代理的地址（代理本地服务不要用127.0.0.1)

      proxy_set_header Host $host;

      proxy_set_header X-Real-IP $remote_addr;

      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

    }

    location ~ .*\.(js|css)$ {
       proxy_pass http://{本机IP}:{端口};		# 加载js和css文件	
    }

  }

```



* **代理静态网页的配置方式**

```shell
http {

  server {

    listen 80;

    server_name www.ddd.com;					 # 你的域名    

    return 301 https://$server_name$request_uri;

}

  server {

    listen 443 ssl http2;

    server_name www.ddd.com;					 # 你的域名    
	
    include /etc/nginx/default.d/*.conf;

    add_header Access-Control-Allow-Origin '*';
    add_header Access-Control-Allow-Methods 'GET, POST, OPTIONS, PUT, DELETE';
    add_header Access-Control-Allow-Headers 'DNT,X-Mx-ReqToken,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Authorization, X-Total-Count';
    add_header Access-Control-Expose-Headers 'X-Total-Count';
	
    ssl_certificate /etc/cert/server.crt;		 # 证书位置

    ssl_certificate_key /etc/cert/server.key;	 # 证书位置
    
    charset utf-8;  # 添加这行来指定编码

    location / {
		# 对于css文件解析进行配置
		include mime.types;  	
		default_type application/octet-stream;

        root /etc/nginx/html/dist;			 # 静态页面目录

        index index.html;					# 静态页面名字

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