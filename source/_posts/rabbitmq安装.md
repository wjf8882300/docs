---
title: rabbitmq安装
date: 2021-12-23 20:59:07
tags: rabbitmq
categories: 工具
---

#### docker安装

```
docker run -d \
--hostname my-rabbit \
--name rabbitmq \
--restart=always \
-e RABBITMQ_DEFAULT_USER=rabbitmq \
-e RABBITMQ_DEFAULT_PASS=IJH1Acq7F8mVgot5 \
-p 15672:15672 \
-p 5672:5672 \
rabbitmq:management
```
> RABBITMQ_DEFAULT_USER 用户名
> RABBITMQ_DEFAULT_PASS 密码
> 15672: web管理端口号
> 5672: TCP端口

#### windows上安装
windows下载安装包，解压即可，然后切换到目录下

1）设置默认自启动
```
rabbitmq-service start
```

2）安装管理插件，通过http://localhost:15672可以访问
```
rabbitmq-plugins enable rabbitmq_management
```

3）创建用户，并设置权限
```
rabbitmqctl add_user rabbitmq IJH1Acq7F8mVgot5
rabbitmqctl set_user_tags rabbitmq administrator
rabbitmqctl set_permissions -p / rabbitmq ".*" ".*" ".*"
```