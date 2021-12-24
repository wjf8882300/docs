---
title: redis集群
date: 2021-12-21 19:54:16
tags: redis
categories: 工具
---
一主多从集群
以三台机器为例
node1: 172.19.192.179:6379
node2: 172.19.192.179:6380
node3: 172.19.192.179:6381

```
# node1: 172.19.192.179:6379
docker run -d --name redis-node1 -p 6379:6379 -v /data/redis/node1:/data redis:5.0.5
# node2: 172.19.192.179:6380
docker run -d --name redis-node2 -p 6380:6379 -v /data/redis/node2:/data redis:5.0.5
# node3: 172.19.192.179:6381
docker run -d --name redis-node3 -p 6381:6379 -v /data/redis/node3:/data redis:5.0.5

# node2上执行
docker exec -it redis-node2 redis-cli
127.0.0.1:6379> auth n2dasUhH0Iziq18Z
OK
127.0.0.1:6379> slaveof 172.19.192.179 6379
OK
 
# node3上执行
docker exec -it redis-node3 redis-cli
127.0.0.1:6379> auth n2dasUhH0Iziq18Z
OK
127.0.0.1:6379> slaveof 172.19.192.179 6379
OK

```

#### cluster集群
##### 1.编写redis-cluster.tmpl

```
# redis端口
port ${PORT}
# 设置密码
requirepass n2dasUhH0Iziq18Z
masterauth n2dasUhH0Iziq18Z
# 关闭保护模式
protected-mode no
# 开启集群
cluster-enabled yes
# 集群节点配置
cluster-config-file nodes.conf
# 超时
cluster-node-timeout 5000
# 集群节点IP host模式为宿主机IP
cluster-announce-ip 172.19.192.179
# 集群节点端口 7001 - 7006
cluster-announce-port ${PORT}
cluster-announce-bus-port 1${PORT}
# 开启 appendonly 备份模式
appendonly yes
# 每秒钟备份
appendfsync everysec
# 对aof文件进行压缩时，是否执行同步操作
no-appendfsync-on-rewrite no
# 当目前aof文件大小超过上一次重写时的aof文件大小的100%时会再次进行重写
auto-aof-rewrite-percentage 100
# 重写前AOF文件的大小最小值 默认 64mb
auto-aof-rewrite-min-size 64mb
```

##### 2.编写redis-cluster-config.sh

```
for port in `seq 7001 7006`; do \
  mkdir -p ./redis-cluster/${port}/conf \
  && PORT=${port} envsubst < ./redis-cluster.tmpl > ./redis-cluster/${port}/conf/redis.conf \
  && mkdir -p ./redis-cluster/${port}/data; \
done
```

##### 3.执行redis-cluster-config.sh创建集群所需目录

##### 4.编写docker-compose.yaml
```
version: '2.2'
 
services:
  redis7001:
    image: 'redis'
    container_name: redis7001
    command:
      ["redis-server", "/usr/local/etc/redis/redis.conf"]
    volumes:
      - ./redis-cluster/7001/conf/redis.conf:/usr/local/etc/redis/redis.conf
      - ./redis-cluster/7001/data:/data
    ports:
      - "7001:7001"
      - "17001:17001"
    environment:
      # 设置时区为上海，否则时间会有问题
      - TZ=Asia/Shanghai
    logging:
      driver: "json-file"
      options:
        max-size: "100m"
        max-file: "3"
       
         
 
  redis7002:
    image: 'redis'
    container_name: redis7002
    command:
      ["redis-server", "/usr/local/etc/redis/redis.conf"]
    volumes:
      - ./redis-cluster/7002/conf/redis.conf:/usr/local/etc/redis/redis.conf
      - ./redis-cluster/7002/data:/data
    ports:
      - "7002:7002"
      - "17002:17002"
    environment:
      # 设置时区为上海，否则时间会有问题
      - TZ=Asia/Shanghai
    logging:
      driver: "json-file"
      options:
        max-size: "100m"
        max-file: "3"  
 
  redis7003:
    image: 'redis'
    container_name: redis7003
    command:
      ["redis-server", "/usr/local/etc/redis/redis.conf"]
    volumes:
      - ./redis-cluster/7003/conf/redis.conf:/usr/local/etc/redis/redis.conf
      - ./redis-cluster/7003/data:/data
    ports:
      - "7003:7003"
      - "17003:17003"
    environment:
      # 设置时区为上海，否则时间会有问题
      - TZ=Asia/Shanghai
    logging:
      driver: "json-file"
      options:
        max-size: "100m"
        max-file: "3"    
       
  redis7004:
    image: 'redis'
    container_name: redis7004
    command:
      ["redis-server", "/usr/local/etc/redis/redis.conf"]
    volumes:
      - ./redis-cluster/7004/conf/redis.conf:/usr/local/etc/redis/redis.conf
      - ./redis-cluster/7004/data:/data
    ports:
      - "7004:7004"
      - "17004:17004"
    environment:
      # 设置时区为上海，否则时间会有问题
      - TZ=Asia/Shanghai
    logging:
      driver: "json-file"
      options:
        max-size: "100m"
        max-file: "3"  
 
  redis7005:
    image: 'redis'
    container_name: redis7005
    command:
      ["redis-server", "/usr/local/etc/redis/redis.conf"]
    volumes:
      - ./redis-cluster/7005/conf/redis.conf:/usr/local/etc/redis/redis.conf
      - ./redis-cluster/7005/data:/data
    ports:
      - "7005:7005"
      - "17005:17005"
    environment:
      # 设置时区为上海，否则时间会有问题
      - TZ=Asia/Shanghai
    logging:
      driver: "json-file"
      options:
        max-size: "100m"
        max-file: "3"   
 
  redis7006:
    image: 'redis'
    container_name: redis7006
    command:
      ["redis-server", "/usr/local/etc/redis/redis.conf"]
    volumes:
      - ./redis-cluster/7006/conf/redis.conf:/usr/local/etc/redis/redis.conf
      - ./redis-cluster/7006/data:/data
    ports:
      - "7006:7006"
      - "17006:17006"
    environment:
      # 设置时区为上海，否则时间会有问题
      - TZ=Asia/Shanghai   
    logging:
      driver: "json-file"
      options:
        max-size: "100m"
        max-file: "3"
```
		
##### 5.启动redis集群容器

```
docker-compose up -d
```

##### 6.执行命令完成集群配置

```
docker exec -it redis7001 redis-cli -p 7001 -a n2dasUhH0Iziq18Z --cluster create 172.19.192.179:7001 172.19.192.179:7002 172.19.192.179:7003 172.19.192.179:7004 172.19.192.179:7005 172.19.192.179:7006 --cluster-replicas 1
```

##### 7.测试集群

```
docker exec -it redis7001 redis-cli -h 172.19.192.179 -p 7005 -a n2dasUhH0Iziq18Z ping
```