---
title: 搭建vless+xtls+回落节点
tags: Science Online
author: Taniz Hugo
language: zh-CN
timezone: Asia/Shanghai
abbrlink: SO1
date: 2023-05-06 12:00:09
---



#### 安装xray

```shell
xray官方一键安装脚本：bash -c "$(curl -L github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install -u root
启动v2ray：systemctl start xray.service 
重启v2ray：systemctl restart xray.service 
v2ray状态：systemctl status xray.service
```

#### 修改xray配置文件

配置文件：/usr/local/etc/xray/config,json

```shell
{
    "log": {
        "loglevel": "warning"
    },
    "inbounds": [
        {
            "port": 443,
            "protocol": "vless",
            "settings": {
                "clients": [
                    {
                        "id": "a547ef50-2e51-4297-a9be-ddb1e4960bd0", 
                        "flow": "xtls-rprx-vision"
                    }
                ],
                "decryption": "none",
                "fallbacks": [
                    {
                        "dest": 58554
                    }
                ]
            },
            "streamSettings": {
                "network": "tcp",
                "security": "tls",
                "tlsSettings": {
                    "alpn": [
                        "http/1.1"
                    ],
                    "certificates": [
                        {
                            "certificateFile": "/usr/local/etc/xray/server.crt", 
                            "keyFile": "/usr/local/etc/xray/server.key"
                        }
                    ]
                }
            }
        },
        {
            "port": 58554,
            "listen": "127.0.0.1",
            "protocol": "trojan",
            "settings": {
                "clients": [
                    {
                        "password": "4b907f67de10bad4429225dd9a75ab2e2c940f9b1d80f5f9cffc96245d976e38"
                    }
                ],
                "fallbacks": [
                    {
                        "dest": "127.0.0.1:80"
                    }
                ]
            },
            "streamSettings": {
                "network": "tcp",
                "security": "none"
            }
        }
    ],
    "outbounds": [
        {
            "protocol": "freedom"
        }
    ]
}

```



#### 申请证书

```
#安装证书工具：
curl https://get.acme.sh | sh; apt install socat -y || yum install socat -y; ~/.acme.sh/acme.sh --set-default-ca --server letsencrypt

#添加软链接：
ln -s  /root/.acme.sh/acme.sh /usr/local/bin/acme.sh

#切换CA机构： 
acme.sh --set-default-ca --server letsencrypt

# 开放80端口提供访问
ufw allow 80

#申请证书方式1： 
acme.sh  --issue -d 替换为你的域名 --standalone -k ec-256
#申请证书方式2： 
acme.sh --register-account -m "${RANDOM}@chacuo.net" --server buypass --force --insecure && ~/.acme.sh/acme.sh  --issue -d 你的域名 --standalone -k ec-256 --force --insecure --server buypass
#申请证书方式3：
acme.sh --register-account -m "${RANDOM}@chacuo.net" --server zerossl --force --insecure && ~/.acme.sh/acme.sh  --issue -d 你的域名 --standalone -k ec-256 --force --insecure --server zerossl

#安装证书：
acme.sh --install-cert -d 你的域名 --ecc --key-file /etc/x-ui/server.key --fullchain-file /etc/x-ui/server.crt
```



#### 添加nginx服务回落自己的页面

下载nginx服务

```shell
apt-get install nginx -y
```

nginx.conf: /etc/nginx/nginx.conf

```shell
# 全局配置
user nginx;             # Nginx运行的用户
worker_processes auto;  # 自动检测CPU核心数，用于处理并发请求

# 错误日志和访问日志
error_log /var/log/nginx/error.log;
pid /var/run/nginx.pid;

# 事件模块配置
events {
    worker_connections 1024;  # 每个工作进程可以处理的最大连接数
}

http {
    include /etc/nginx/mime.types;  # 加载MIME类型配置

    # 默认服务器设置
    server {
        listen 80;  # 监听80端口

        server_name example.com;  # 你的域名

        location / {
            root /var/www/html;  # 静态文件根目录
            index index.html;    # 默认文件
        }

        location /api {
            proxy_pass http://backend_server;  # 反向代理到后端服务器
        }

        error_page 404 /404.html;  # 处理404错误
    }
}

```

