---
title: spring boot druid监控
date: 2021-12-23 13:50:10
tags: java
categories: 语言
---

#### 1、配置文件
```
datasource:
  druid:
    driver-class-name: oracle.jdbc.driver.OracleDriver
    url: jdbc:oracle:thin:@xxx.xxx.xxx.xxx:1521:orcl
    username: bt_test
    password: bt_test2
    filters: stat
    maxActive: 20
    initialSize: 1
    maxWait: 60000
    minIdle: 1
    timeBetweenEvictionRunsMillis: 60000
    minEvictableIdleTimeMillis: 300000
    validationQuery: select 'x' from dual
    testWhileIdle: true
    testOnBorrow: false
    testOnReturn: false
    poolPreparedStatements: true
    maxOpenPreparedStatements: 20
    # 下面开启监控功能
    web-stat-filter:
      enabled: true
      url-pattern: /*
      exclusions: /*.js,/*.gif,/*.jpg,/*.bmp,/*.png,/*.css,/*.ico,/druid/*
    filter:
      stat:
        enabled: true
        log-slow-sql: true
        slow-sql-millis: 10000
        merge-sql: true
    aop-patterns: com.btong.*
    stat-view-servlet:
      enabled: true
      login-username: btong
      login-password: 
      allow:
```

#### 2.访问监控日志

##### 2.1.登录网站
http://xxx.xxx.xxx.xxx/druid/login.html
输入用户名：btong
输入密码：btong@123

##### 2.2.监控截图
![这是druid监控页面截图](druid.png)	  