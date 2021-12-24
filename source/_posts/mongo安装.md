---
title: mongo安装
date: 2021-12-23 21:31:50
tags: mongo
categories: 数据库
---

#### 安装

```
mkdir -p /data/mongo
chown -R 1000:1000 /data/mongo
 
docker run --restart always \
-e MONGO_INITDB_ROOT_USERNAME=root \
-e MONGO_INITDB_ROOT_PASSWORD=LWOqa2FtfCZ9As6h \
-d -p 27017:27017 \
-v /data/mongo:/data/db \
--name mongo \
mongo:latest
```

- MONGO_INITDB_ROOT_USERNAME：用户名
- MONGO_INITDB_ROOT_PASSWORD：密码
- 27017：端口号
- /data/mongo：存放数据目录


#### 创建用户
1、授权普通用户
```
db.createUser({user: "smartx-oa",pwd: "FPob40Ddk7Wz",roles: [{ role: "readWrite", db: "smartx-oa" }]})
```


2、创建admin用户
```
db.createUser({ user: "root", pwd: "LWOqa2FtfCZ9As6h", roles: [{ role: "userAdminAnyDatabase", db: "admin" }] })
```


3、删除用户
```
db.dropUser("smartx-oa")
```