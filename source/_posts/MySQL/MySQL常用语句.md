---
title: MySQL常用语句
date: 2023-10-10 10:53:55
author: Taniz Hugo
language: zh-CN
abbrlink: 
top: false
is_draft: false
summary: 最近上班要接触到数据库的查询，这时候才发现，自己平时的SQL的知识如此匮乏，大学没有好好学习的下场啊~ 现在有空就来记录下一些常用的命令
categories: 
  - MySQL
tags:
  - MySQL常用语句
---



#### 前言

最近上班要接触到数据库的查询，这时候才发现，自己平时的SQL的知识如此匮乏，大学没有好好学习的下场啊~ 现在有空就来记录下一些常用的命令。



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





#### 外键





#### 索引



#### 创建用户

基本命令

```mysql
CREATE USER '新用户'@'主机' IDENTIFIED BY '密码';	
```

例子：创建一个主机IP为'12.34.56.78'，名为'test'的用户，设置密码'123456'

```mysql
CREATE USER 'test'@'12.34.56.78' IDENTIFIED BY '123456';
```

#### 授权

在MySQL中，授权是通过GRANT语句来实现的。

GRANT语句允许给用户或用户组授予特定的权限，以便执行各种操作，如SELECT、INSERT、UPDATE等。

##### 授权所有权限给用户

授予用户在所有数据库（`*.*`）上的所有权限，只能从本地主机（'localhost'）连接，并使用指定的用户名和密码。

```mysql
GRANT ALL PRIVILEGES ON *.* TO 'username'@'localhost' IDENTIFIED BY 'password';
```

##### **授予特定权限给用户**

授予用户在指定数据库（`database_name`）上的SELECT、INSERT和UPDATE权限。

```mysql
GRANT SELECT, INSERT, UPDATE ON database_name.* TO 'username'@'localhost';
```

|    授权权限    |                 描述                  |
| :------------: | :-----------------------------------: |
|     SELECT     |        允许用户从表中查询记录         |
|     INSERT     |        允许用户从表中添加记录         |
|     UPDATE     |        允许用户从表中修改记录         |
|     DELETE     |        允许用户从表中删除记录         |
|     CREATE     |      允许用户创建新的数据库或表       |
|      DROP      |        允许用户删除数据库或表         |
|     ALTER      |       允许用户修改现有表的结构        |
| SHOW DATABASES |        允许用户列出所有数据库         |
|  CREATE USER   | 允许用户创建、更改和删除MySQL用户账户 |
|  GRANT OPTION  | 允许用户将自己拥有的权限授予其他用户  |
| ALL PRIVILEGES |             包含所有权限              |
