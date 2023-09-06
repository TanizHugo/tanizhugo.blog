---
title: 开启ISP静态IP代理
date: 2023-09-05 10:38:01
author: Taniz Hugo
language: zh-CN
abbrlink: so4
top: false
is_draft: true
summary: 跟着不良林大神学习节点搭建，实现科学上网
categories: 
  - 科学上网
tags:
  - 住宅IP
  - ISP
  - x-ui
---

ISP

在出站`outbounds`列队填以下内容

```json
{
  "tag": "us_isp",
  "protocol": "socks",
  "settings": {
    "servers": [
      {
        "address": "156.233.21.63",
        "port": 2333,
        "users": [
          {
            "user": "hugo1024",
            "pass": "123456"
          }
        ]
      }
    ]
  }
}

```

