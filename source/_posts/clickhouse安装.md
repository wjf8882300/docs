---
title: clickhouse安装
date: 2021-12-23 21:30:46
tags: clickhouse
categories: 数据库
---

#### 1.安装服务端
##### 1.1 docker安装

```
docker run -d --restart=always \
--name clickhouse-server \
-p 8123:8123 -p 9000:9000 \
--ulimit nofile=262144:262144 \
-v /mnt/clickhouse:/var/lib/clickhouse \
-v /mnt/clickhouse/config.xml:/etc/clickhouse-server/config.xml \
-v /mnt/clickhouse/users.xml:/etc/clickhouse-server/users.xml \
yandex/clickhouse-server
```

- /mnt/clickhouse/config.xml：系统配置文件  
- /mnt/clickhouse/users.xml：用户配置文件  

##### 1.2 修改用户名
默认为default，且没有密码，

生成密码密文（明文：wRZOGtaYIykiVDJX， 密文：12b4efbcc6f2baa1a35f7b4dc1d946c86316d651d115796328b6013aa806b45c）
```
PASSWORD="wRZOGtaYIykiVDJX"; echo "$PASSWORD"; echo -n "$PASSWORD" | sha256sum | tr -d '-'
 
# 输出
wRZOGtaYIykiVDJX
12b4efbcc6f2baa1a35f7b4dc1d946c86316d651d115796328b6013aa806b45c
```

修改users.xml，然后重启下clickhouse-server

```
<users>
    <root>
        <!-- 此处保存为密文，用的时候用明文 -->
        <password_sha256_hex>12b4efbcc6f2baa1a35f7b4dc1d946c86316d651d115796328b6013aa806b45c</password_sha256_hex>
        <networks>
            <ip>::/0</ip>
        </networks>
        <profile>default</profile>
        <quota>default</quota>
        <access_management>1</access_management>
    </root>
</users>
```



#### 2. 安装客户端
##### 2.1 docker安装官方客户端
客户端直接运行即可
```
docker run -it yandex/clickhouse-client -h 172.19.99.154 -u root --password wRZOGtaYIykiVDJX
```

##### 2.2 安装redash客户端
镜像：redash-master_worker、redash-master_scheduler 需要自行编译
1）获取源代码：https://github.com/getredash/redash
2）源码下有docker-compose.yml，可以直接通过docker-compose up -d编译
3）编译之后再安装nodejs和yarn，然后执行yarn --frozen-lockfile（这一步有诸多问题，可以跳过）
前两步编译后可以得到redash-master_worker、redash-master_scheduler两个镜像，用于后续。

docker-compose.yml
```
# This configuration file is for the **development** setup.
# For a production example please refer to getredash/setup repository on GitHub.
version: "2.2"
x-redash-service: &redash-service
  build:
    context: .
    args:
      skip_frontend_build: "true"
  volumes:
    - .:/app
x-redash-environment: &redash-environment
  REDASH_LOG_LEVEL: "INFO"
  REDASH_REDIS_URL: "redis://redis:6379/0"
  REDASH_DATABASE_URL: "postgresql://postgres@postgres/postgres"
  REDASH_RATELIMIT_ENABLED: "false"
  REDASH_MAIL_DEFAULT_SENDER: "redash@example.com"
  REDASH_MAIL_SERVER: "email"
  REDASH_ENFORCE_CSRF: "true"
  REDASH_SECRET_KEY: "BOm3S2NXDUjFt6Wf"
  REDASH_COOKIE_SECRET: "BOm3S2NXDUjFt6Wf"
  GOOGLE_CLIENT_ID: ""
services:
  server:
    <<: *redash-service
    command: dev_server
    depends_on:
      - postgres
      - redis
    ports:
      - "5000:5000"
      - "5678:5678"
    environment:
      <<: *redash-environment
      PYTHONUNBUFFERED: 0
  scheduler:
    <<: *redash-service
    command: dev_scheduler
    depends_on:
      - server
    environment:
      <<: *redash-environment
  worker:
    <<: *redash-service
    command: dev_worker
    depends_on:
      - server
    environment:
      <<: *redash-environment
      PYTHONUNBUFFERED: 0
  redis:
    image: redis:3-alpine
    restart: unless-stopped
  postgres:
    image: postgres:9.5-alpine
    # The following turns the DB into less durable, but gains significant performance improvements for the tests run (x3
    # improvement on my personal machine). We should consider moving this into a dedicated Docker Compose configuration for
    # tests.
    ports:
      - "15432:5432"
    command: "postgres -c fsync=off -c full_page_writes=off -c synchronous_commit=OFF"
    restart: unless-stopped
    environment:
      POSTGRES_HOST_AUTH_METHOD: "trust"
  email:
    image: djfarrelly/maildev
    ports:
      - "1080:80"
    restart: unless-stopped
```

4）使用官方镜像redash/redash:10.1.0.b50633、与自行编译的redash-master_scheduler、redash-master_worker
docker-compose.yml
```
version: "2.2"
x-redash-environment: &redash-environment
  REDASH_LOG_LEVEL: "INFO"
  REDASH_REDIS_URL: "redis://redis:6379/0"
  REDASH_DATABASE_URL: "postgresql://postgres@postgres/postgres"
  REDASH_RATELIMIT_ENABLED: "false"
  REDASH_MAIL_DEFAULT_SENDER: "redash@example.com"
  REDASH_MAIL_SERVER: "email"
  REDASH_ENFORCE_CSRF: "true"
  REDASH_SECRET_KEY: "BOm3S2NXDUjFt6Wf"
  REDASH_COOKIE_SECRET: "BOm3S2NXDUjFt6Wf"
  GOOGLE_CLIENT_ID: ""
  # Set secret keys in the .env file
services:
  server:
    image: redash/redash:10.1.0.b50633
    command: dev_server
    depends_on:
      - postgres
      - redis
    ports:
      - "5000:5000"
      - "5678:5678"
    environment:
      <<: *redash-environment
      PYTHONUNBUFFERED: 0
  scheduler:
    image: redash-master_scheduler
    command: dev_scheduler
    depends_on:
      - server
    environment:
      <<: *redash-environment
  worker:
    image: redash-master_worker
    command: dev_worker
    depends_on:
      - server
    environment:
      <<: *redash-environment
      PYTHONUNBUFFERED: 0
  redis:
    image: redis:3-alpine
    restart: unless-stopped
  postgres:
    image: postgres:9.5-alpine
    # The following turns the DB into less durable, but gains significant performance improvements for the tests run (x3
    # improvement on my personal machine). We should consider moving this into a dedicated Docker Compose configuration for
    # tests.
    ports:
      - "15432:5432"
    command: "postgres -c fsync=off -c full_page_writes=off -c synchronous_commit=OFF"
    restart: unless-stopped
    environment:
      POSTGRES_HOST_AUTH_METHOD: "trust"
  email:
    image: djfarrelly/maildev
    ports:
      - "1080:80"
    restart: unless-stopped
```	
5）创建数据库
```
docker-compose run --rm server create_db
```
6）登录网站
http://xxx.xxx.xxx.xxx:5000/