---
title: redis安装
date: 2021-12-21 19:54:09
tags: redis
categories: 工具
---
#### 后端启动：

```
./redis-server &
```

#### windows服务：

```
# 注册redis服务
redis-server --service-install redis.conf --service-name redis
# 启动redis服务
net start redis
```

#### docker方式启动：

```
mkdir -p /data/redis
chown -R 1000:1000 /data/redis
 
docker run -d --name redis --restart=always -p 6379:6379 -v /data/redis:/data wjf8882300/redis:latest
```

#### Dockerfile

```
FROM redis
MAINTAINER wangjingfeng2008@163.com
COPY redis.conf /usr/local/etc/redis/redis.conf
EXPOSE 6379
CMD [ "redis-server", "/usr/local/etc/redis/redis.conf" ]
```



#### 修改配置文件 redis.conf

```
#bind 127.0.0.1
 
requirepass n2dasUhH0Iziq18Z
 
port 6379
 
notify-keyspace-events "Ex"
```