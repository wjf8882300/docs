---
title: docker安装mysql
date: 2021-12-23 13:27:42
tags: mysql
categories: 数据库
---

#### 1、创建目录
```
mkdir -p /data/usr-data
```
#### 2、启动实例
```
docker run -d \
--name="mysql" \
--restart always \
-e MYSQL_ROOT_HOST=% \
-e MYSQL_ROOT_PWD=KijVtGsAWN2hxLUu \
-v /data/usr-data:/data/usr-data \
-p 3306:3306 \
qq79428316/club-mysql:5.7.25
```

#### 3、创建数据库和普通用户并分配权限
```
CREATE DATABASE IF NOT EXISTS live DEFAULT CHARSET utf8 COLLATE utf8_general_ci;

CREATE USER 'dev-ue-live'@'%' IDENTIFIED BY 'fiJW6t9m27spYZd0';

GRANT ALL ON live.* TO 'dev-ue-live'@'%';
```