---
title: elasticsearch单机与集群
date: 2021-12-23 20:43:31
tags: elasticsearch
categories: 工具
---

#### 单机

docker-compose.yml

```
version: '2.2'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.7.2
    container_name: elasticsearch
    environment:
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esdata1:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
    networks:
      - esnet
  elasticsearch2:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.7.2
    container_name: elasticsearch2
    environment:
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - "discovery.zen.ping.unicast.hosts=elasticsearch"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esdata2:/usr/share/elasticsearch/data
    networks:
      - esnet
 
volumes:
  esdata1:
    driver: local
  esdata2:
    driver: local
 
networks:
  esnet:
```

 
#### 集群
1、设置工作目录
```
mkdir -p /data/elasticsearch/data /data/elasticsearch/logs
chown -R 1000:1000 /data/elasticsearch
```

2、进入sysctl.conf文件添加一行（解决容器内存权限过小问题）
```
vim /etc/sysctl.conf
vm.max_map_count=262144
sysctl -p
```
3、配置文件elasticsearch.yml 
es-node-0(172.19.115.18):
```
cluster.name: elasticsearch-cluster
node.name: es-node0
network.host: 172.19.115.18
http.port: 9200
transport.tcp.port: 9300
http.cors.enabled: true
http.cors.allow-origin: "*"
node.master: false 
node.data: true
discovery.seed_hosts: ["172.19.115.62:9300","172.19.115.64:9300","172.19.115.65:9300"]
cluster.initial_master_nodes: ["es-node1","es-node2","es-node3"]
gateway.recover_after_nodes: 3
gateway.recover_after_time: 5m
gateway.expected_nodes: 5
```

es-node-1(172.19.115.62):
```
#集群名(广播形式构建集群，所以集群名称必须一致才会相互发现)
cluster.name: elasticsearch-cluster
#节点名:节点一：es-node1，节点二：es-node2，节点三：es-node3
node.name: es-node1
#设置绑定的ip地址，可以是ipv4或ipv6的，默认为0.0.0.0，
network.bind_host: 0.0.0.0
#设置其它节点和该节点交互的ip地址，如果不设置它会自动判断，
#值必须是个真实的ip地址
network.publish_host: 172.19.115.62
#设置对外服务的http端口，默认为9200
http.port: 9200
#设置节点之间交互的tcp端口，默认是9300
transport.tcp.port: 9300
#是否允许跨域REST请求（配置这个可以解决elasticsearch-head访问）
http.cors.enabled: true
#允许 REST 请求来自何处
http.cors.allow-origin: "*"
#节点角色设置
node.master: true
node.data: true
#有成为主节点资格的节点列表
discovery.zen.ping.unicast.hosts: ["172.19.115.62:9300","172.19.115.64:9300","172.19.115.65:9300"]
#集群中一直正常运行的，有成为master节点资格的最少节点数（默认为1）
discovery.zen.minimum_master_nodes: 2
```

es-node-2(172.19.115.64):
```
cluster.name: elasticsearch-cluster
node.name: es-node2
network.bind_host: 0.0.0.0
network.publish_host: 172.19.115.64
http.port: 9200
transport.tcp.port: 9300
http.cors.enabled: true
http.cors.allow-origin: "*"
node.master: true
node.data: true
discovery.zen.ping.unicast.hosts: ["172.19.115.62:9300","172.19.115.64:9300","172.19.115.65:9300"]
discovery.zen.minimum_master_nodes: 2
```

es-node-3(172.19.115.65):
```
cluster.name: elasticsearch-cluster
node.name: es-node3
network.bind_host: 0.0.0.0
network.publish_host: 172.19.115.65
http.port: 9200
transport.tcp.port: 9300
http.cors.enabled: true
http.cors.allow-origin: "*"
node.master: true
node.data: true 
discovery.zen.ping.unicast.hosts: ["172.19.115.62:9300","172.19.115.64:9300","172.19.115.65:9300"]
discovery.zen.minimum_master_nodes: 2
```

4、启动容器
```
# 172.19.115.18上执行
docker run --restart always -e ES_JAVA_OPTS="-Xms1200m -Xmx1200m" -d -p 9200:9200 -p 9300:9300 -v /data/elasticsearch/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml -v /data/elasticsearch/data:/usr/share/elasticsearch/data --name es-node0 elasticsearch:5.6.12
# 172.19.115.62上执行
docker run --restart always -e ES_JAVA_OPTS="-Xms1200m -Xmx1200m" -d -p 9200:9200 -p 9300:9300 -v /data/elasticsearch/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml -v /data/elasticsearch/data:/usr/share/elasticsearch/data --name es-node1 elasticsearch:5.6.12
# 172.19.115.64上执行
docker run --restart always -e ES_JAVA_OPTS="-Xms1200m -Xmx1200m" -d -p 9200:9200 -p 9300:9300 -v /data/elasticsearch/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml -v /data/elasticsearch/data:/usr/share/elasticsearch/data --name es-node2 elasticsearch:5.6.12
# 172.19.115.65上执行
docker run --restart always -e ES_JAVA_OPTS="-Xms1200m -Xmx1200m" -d -p 9200:9200 -p 9300:9300 -v /data/elasticsearch/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml -v /data/elasticsearch/data:/usr/share/elasticsearch/data --name es-node3 elasticsearch:5.6.12
```