---
title: docker
date: 2021-12-21 18:18:40
tags: docker
categories: 工具
---
#### 1.docker概念与好处
docker由镜像与容器构成，镜像包含了要运行的所有环境、代码与数据。容器是运行时态生成的运行环境。
1. 环境隔离  需要的环境都在一个镜像中，运行一个容器就有了所需环境。
2. 快速部署  制作好镜像之后，可以方便部署。
3. 配置简单  不管是命令还是配置都简洁高效。

#### 2. docker安装
##### 1. centos 7及以上
```
# step 1: 安装必要的一些系统工具
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
# Step 2: 添加软件源信息
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# Step 3: 更新并安装 Docker-CE
sudo yum makecache fast
sudo yum -y install docker-ce
```
##### 2. Ubuntu 14.04 16.04
```
# step 1.docker概念与好处1: 安装必要的一些系统工具
sudo apt-get update
sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common
# step 2: 安装GPG证书
curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
# Step 3: 写入软件源信息
sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
# Step 4: 更新并安装 Docker-CE
sudo apt-get -y update
sudo apt-get -y install docker-ce
```
#### 3.docker开机启动
```
# 添加开机启动
systemctl enable docker.service
# 启动docker服务
systemctl start docker.service
```
#### 4.docker常用命令
```
# 查看镜像
docker images

# 拉取镜像
docker pull xxx

# 删除镜像
docker rmi xxx

# 查看运行的容器
docker ps

# 启动容器
docker start xxx

# 停止容器
docker stop xxx

# 重启容器
docker restart xxx

# 进入容器
docker exec -it xxx bash

# 查看日志
docker logs -f xxx --tail=10

# 查看容器
docker inspect xxx

# 查看容器状态
docker stats
```

#### 5.制作镜像
##### 1. 镜像加速
```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://w029dasz.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

##### 2. 准备Dockerfile
```
# java镜像
FROM openjdk:8-jdk-alpine
 
# 将本地文件夹挂载到当前容器
# 创建/tmp目录并持久化到Docker数据文件夹，因为Spring Boot使用的内嵌Tomcat容器默认使用/tmp作为工作目录。
VOLUME ["/tmp"]
 
# 拷贝文件到容器
COPY *.jar /opt/app.jar
 
# 解决容器时间和宿主主机时间不一致问题
RUN echo 'http://mirrors.aliyun.com/alpine/v3.6/community/'>/etc/apk/repositories && echo 'http://mirrors.aliyun.com/alpine/v3.6/main/'>>/etc/apk/repositories && apk add --no-cache tzdata
ENV TZ Asia/Shanghai
RUN /bin/cp /usr/share/zoneinfo/${TZ} /etc/localtime && echo '${TZ}' >/etc/timezone
 
RUN apk add  --no-cache ffmpeg
 
# 打开服务端口
EXPOSE 8080 8080
 
# 配置环境变量 todo jvm优化参数可以设置这里
ENV JAVA_OPTS='-Xmx1024m -Xms512m -Xdebug -Xnoagent -Djava.compiler=NONE -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=8077' APP_OPTS=''
 
# 配置容器启动后执行的命令
ENTRYPOINT java $JAVA_OPTS  -server  -Dfile.encoding=UTF-8 -Duser.language=zh -Duser.region=CN -Djava.security.egd=file:/dev/./urandom $APP_OPTS -jar /opt/app.jar
```

##### 3. 制作镜像

```
# 制作镜像
docker build -t registry.uecent.com:5000/provider-auth:1.0.0 .
# 上传镜像(非必要)
docker push registry.uecent.com:5000/provider-auth:1.0.0
# 启动镜像
docker run -d --name provider-auth \ 
--restart=always  \
-p 80:80 \
-v /data/provider-auth:/tmp \
-e EUREKA_SERVER=http://172.19.115.13/eureka/ \
-e APP_OPTS=\"spring.profiles.active=dev\" \
--log-opt max-size=100m --log-opt max-file=3 \
registry.uecent.com:5000/provider-auth:1.0.0
```
参数说明：
-d 表示后台运行
--name 指定容器名称
--restart always表示总会重启
-p 映射端口号
-v 映射目录
-e 指定环境变量
--log-opt 日志相关
--privileged 允许root权限
--network=host 使用主机网络

#### 6. docker-compose

##### 1. 安装

```
sudo curl -L "https://github.com/docker/compose/releases/download/1.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose docker-compose --version
```

##### 2. docker-compose.yml

```
version: '3.2'
 
services:
  redis:
    container_name:  redis
    image: redis:latest
    restart: unless-stopped
    ports:
      - "6379:6379"
    volumes:
      - /redis/data:/data
  rabbitmq:
    container_name:  rabbitmq
    image: rabbitmq:management
    restart: unless-stopped
    ports:
      - "15672:15672"
      - "5672:5672"
    volumes:
      - /redis/data:/data   
    environment:
      - RABBITMQ_DEFAULT_USER=rabbitmq
      - RABBITMQ_DEFAULT_PASS=Qfka7ls2A40JEoMw
```
   

##### 3.启动

```
# 启动
docker-compose up -d
# 查看
docker-compse ps
```

#### 7.docker私服

##### 1.安装
```
# 拉取镜像
docker pull registry 
# 新建仓库目录
mkdir -p /data/registry
# 启动容器
docker run -d -v /data/registry:/var/lib/registry -v /data/cert:/data/cert -p 5000:5000 -e REGISTRY_STORAGE_DELETE_ENABLED=true -e REGISTRY_HTTP_TLS_CERTIFICATE=/data/cert/cert.pem -e REGISTRY_HTTP_TLS_KEY=/data/cert/cert.key --restart=always --privileged=true --name registry registry:latest
```

##### 2.私服相关操作

```
搜索镜像：
curl -X GET https://registry.uecent.com:5000/v2/_catalog

获取镜像列表：
curl -X GET https://registry.uecent.com:5000/v2/provider-auth/tags/list

查询镜像tag：
curl -v --silent -H "Accept: application/vnd.docker.distribution.manifest.v2+json" -X GET https://registry.uecent.com:5000/v2/provider-auth/manifests/latest 2>&1 | grep Docker-Content-Digest | awk '{print ($3)}'

删除镜像：
curl -v --silent -H "Accept: application/vnd.docker.distribution.manifest.v2+json" -X DELETE https://registry.uecent.com:5000/v2/provider-auth/manifests/sha256:cc21a69e5d84b78f51eb8003d1f31743fa8cfc629b5e6379e751afff1ca34544

垃圾回收（在docker服务器上执行）：
docker exec -it registry /bin/registry garbage-collect /etc/docker/registry/config.yml

```

