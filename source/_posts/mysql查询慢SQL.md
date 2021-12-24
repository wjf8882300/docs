---
title: mysql查询慢SQL
date: 2021-12-23 13:31:27
tags: mysql
categories: 数据库
---

#### 1.查看日志是否打开
```
SHOW VARIABLES LIKE '%slow%';

Variable_name              Value                                              
-------------------------  ---------------------------------------------------
log_slow_admin_statements  OFF                                                
log_slow_slave_statements  OFF                                                
slow_launch_time           2                                                  
slow_query_log             ON                                                 
slow_query_log_file        /data/usr-data/mysql-5.7.25/2dc304b65a14-slow.log  
```

#### 2.设置打开日志
如果slow_query_log不是on，则设置为on
```
SET GLOBAL slow_query_log = on;
```
slow_query_log_file表示日志路径

#### 3.设置时间大于1s的记录到慢日志中

```
SET GLOBAL long_query_time = 1;
```