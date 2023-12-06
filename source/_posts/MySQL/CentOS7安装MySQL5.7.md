---
title: CentOS7安装MySQL5.7
date: 2023-10-25 18:13:39
author: Taniz Hugo
language: zh-CN
abbrlink: 
top: false
is_draft: false
summary: 
categories: 
  - MySQL
tags:
  - MySQL常用语句
---



#### 前言

很久没有写项目了，写项目的时候肯定离不开的就是数据库，原本我是选择用Docker，拉一个镜像搭建的数据库，可是我发现每次都要进入到Docker里面对数据库进行操作太麻烦了，所以我还是打算在搭建一个MySQL的服务。



#### 查询表B的数据 更新表A

##### 命令

```sql
UPDATE rpa_tdtgl t
JOIN rpa_asbll a ON t.bh = a.bh
SET t.ejcbdw = a.ejcbdw, t.jtjbdw = a.jtjbdw;
```

##### 知识点

JOIN --- ON ---



#### 条件查询筛选特定列

##### 命令

```sql

```



##### 知识点



#### 查询字段名



##### 命令

返回表字段所有信息

```sql
DESCRIBE your_table_name;

```

查询所有字段名，以逗号形式返回

```sql
SELECT GROUP_CONCAT(column_name) AS FieldNames
FROM information_schema.columns
WHERE table_name = 'your_table_name';

```

