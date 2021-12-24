---
title: mysql分区表
date: 2021-12-23 13:23:00
tags: mysql
categories: 数据库
---
mysql 表分区的几种方式：

RANGE分区：基于属于一个给定连续区间的列值，把多行分配给分区。
LIST分区：类似于按RANGE分区，区别在于LIST分区是基于列值匹配一个离散值集合中的某个值来进行选择。
HASH分区：基于用户定义的表达式的返回值来进行选择的分区，该表达式使用将要插入到表中的这些行的列值进行计算。这个函数可以包含MySQL中有效的、产生非负整数值的任何表达式。
KEY分区：类似于按HASH分区，区别在于KEY分区只支持计算一列或多列，且MySQL服务器提供其自身的哈希函数。必须有一列或多列包含整数值。
以xxl-job为例，这是一张日志表，每天产上约30万条记录

#### 1. 创建表xxl_job_log并创建分区

```
/* 注意：需要id和trigger_time作为双主键，有主键的表但分区字段（trigger_time）不是主键，新建分区表时会报A PRIMARY KEY must include all columns in the table’s partitioning function */
CREATE TABLE `xxl_job_log` (
  `id` BIGINT(20) NOT NULL AUTO_INCREMENT,
  `job_group` INT(11) NOT NULL COMMENT '执行器主键ID',
  `job_id` INT(11) NOT NULL COMMENT '任务，主键ID',
  `executor_address` VARCHAR(255) DEFAULT NULL COMMENT '执行器地址，本次执行的地址',
  `executor_handler` VARCHAR(255) DEFAULT NULL COMMENT '执行器任务handler',
  `executor_param` VARCHAR(512) DEFAULT NULL COMMENT '执行器任务参数',
  `executor_sharding_param` VARCHAR(20) DEFAULT NULL COMMENT '执行器任务分片参数，格式如 1/2',
  `executor_fail_retry_count` INT(11) NOT NULL DEFAULT '0' COMMENT '失败重试次数',
  `trigger_time` DATETIME NOT NULL COMMENT '调度-时间',
  `trigger_code` INT(11) NOT NULL COMMENT '调度-结果',
  `trigger_msg` TEXT COMMENT '调度-日志',
  `handle_time` DATETIME DEFAULT NULL COMMENT '执行-时间',
  `handle_code` INT(11) NOT NULL COMMENT '执行-状态',
  `handle_msg` TEXT COMMENT '执行-日志',
  `alarm_status` TINYINT(4) NOT NULL DEFAULT '0' COMMENT '告警状态：0-默认、1-无需告警、2-告警成功、3-告警失败',
  PRIMARY KEY (`id`, `trigger_time`),
  KEY `I_trigger_time` (`trigger_time`),
  KEY `I_handle_code` (`handle_code`),
  KEY `I_alarm_status` (`alarm_status`),
  KEY `I_trigger_code` (`trigger_code`),
  KEY `I_job_group` (`job_group`),
  KEY `I_job_id` (`job_id`)
) ENGINE=INNODB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4;
 
/* 依托字段trigger_time新建分区表，每天一个分区，此处手工创建，需再建一个定时创建分区的任务 */
ALTER TABLE xxl_job_log PARTITION BY RANGE COLUMNS(trigger_time)(
    PARTITION p20211212 VALUES LESS THAN('2021-12-12'),
    PARTITION p20211213 VALUES LESS THAN('2021-12-13'),
    PARTITION p20211214 VALUES LESS THAN('2021-12-14'),
    PARTITION p20211215 VALUES LESS THAN('2021-12-15'),
    PARTITION p20211216 VALUES LESS THAN('2021-12-16')
);
 
/* 查询分区表情况 */
SELECT partition_name, partition_description AS val FROM information_schema.partitions
WHERE table_name='xxl_job_log' AND table_schema='xxl_job';     
 
partition_name  val          
--------------  --------------
p20211212       '2021-12-12' 
p20211213       '2021-12-13' 
p20211214       '2021-12-14' 
p20211215       '2021-12-15' 
p20211216       '2021-12-16'
 
/* 查询分区表情况 */
EXPLAIN PARTITIONS SELECT * FROM xxl_job_log;
    id  select_type  table        partitions                                                             type    possible_keys  key     key_len  ref       rows  filtered  Extra  
------  -----------  -----------  ---------------------------------------------------------------------  ------  -------------  ------  -------  ------  ------  --------  --------
     1  SIMPLE       xxl_job_log  p20211212,p20211213,p20211214,p20211215,p20211216,p20211217,p20211218  ALL     (NULL)         (NULL)  (NULL)   (NULL)  583426    100.00  (NULL)

```

#### 2.创建自动分区

##### 2.1 创建自动分区存储过程pro_xxl_job_log

创建存储过程，每天新增3个分区，并删除15天前的分区
```
DELIMITER $$
DROP PROCEDURE IF EXISTS pro_xxl_job_log
$$
CREATE PROCEDURE pro_xxl_job_log()
BEGIN
  DECLARE v_sysdate DATE;
  DECLARE v_mindate DATE;
  DECLARE v_maxdate DATE;
  DECLARE v_pt VARCHAR(20);
  DECLARE v_maxval VARCHAR(20);
  DECLARE i INT;
   
  /*增加新分区代码，执行时，不要复制此行*/
  SELECT MAX(CAST(REPLACE(partition_description, '''', '') AS DATE)) AS val
  INTO   v_maxdate
  FROM   INFORMATION_SCHEMA.PARTITIONS
  WHERE  TABLE_NAME = 'xxl_job_log' AND TABLE_SCHEMA = 'xxl_job';
   
  SET v_sysdate = SYSDATE();
   
  WHILE v_maxdate <= (v_sysdate + INTERVAL 3 DAY) DO
    SET v_pt = DATE_FORMAT(v_maxdate+ INTERVAL 1 DAY ,'%Y%m%d');
    SET v_maxval = DATE_FORMAT(v_maxdate + INTERVAL 1 DAY, '%Y-%m-%d');
    SET @sql = CONCAT('alter table xxl_job_log add partition (partition p', v_pt, ' values less than(''', v_maxval, '''))');
    -- SELECT @sql;
    PREPARE stmt FROM @sql;
    EXECUTE stmt;
    DEALLOCATE PREPARE stmt;
    SET v_maxdate = v_maxdate + INTERVAL 1 DAY;
  END WHILE;
   
  /*删除旧分区，执行时，不要复制此行*/
  SELECT MIN(CAST(REPLACE(partition_description, '''', '') AS DATE)) AS val
  INTO   v_mindate
  FROM   INFORMATION_SCHEMA.PARTITIONS
  WHERE  TABLE_NAME = 'xxl_job_log' AND TABLE_SCHEMA = 'xxl_job';
     
  WHILE v_mindate <= (v_sysdate - INTERVAL 15 DAY) DO
    SET v_pt = DATE_FORMAT(v_mindate - INTERVAL 1 DAY,'%Y%m%d');
    SET @sql = CONCAT('alter table xxl_job_log drop partition p', v_pt);
    -- SELECT @sql;
    PREPARE stmt FROM @sql;
    EXECUTE stmt;
    DEALLOCATE PREPARE stmt;
    SET v_mindate = v_mindate + INTERVAL 1 DAY;
  END WHILE;
  
END$$
DELIMITER ;
```

测试存储过程

```
/* 调用存储过程*/
CALL pro_xxl_job_log;
 
 /* 查询分区表情况 */
SELECT partition_name, partition_description AS val FROM information_schema.partitions
WHERE table_name='xxl_job_log' AND table_schema='xxl_job';     
 
partition_name  val          
--------------  --------------
p20211212       '2021-12-12' 
p20211213       '2021-12-13' 
p20211214       '2021-12-14' 
p20211215       '2021-12-15' 
p20211216       '2021-12-16' 
p20211217       '2021-12-17' 
p20211218       '2021-12-18'
 
由上可见新增了p20211217、p20211218两个分区
```



##### 2.2 开启事件

```
/* 查询是否开启事件 */
show variables like "event_scheduler";
Variable_name    Value  
---------------  --------
event_scheduler  OFF   
 
/* 临时开启事件 */
SET GLOBAL event_scheduler = ON;
永久开启事件：修改mysql配置文件my.cnf，在[mysqld]下面增加：event_scheduler=ON

```

##### 2.3 编写事件

设置从2021-12-15 00:00:00开始，每24小时执行一次

```
DELIMITER $$
DROP EVENT IF EXISTS auto_pt $$
CREATE EVENT auto_pt
ON SCHEDULE
EVERY 24 HOUR
STARTS '2021-12-15 00:00:00'
DO
BEGIN
    CALL pro_xxl_job_log();
END$$
DELIMITER ;
```

查看事件

```
show events;
 
Db       Name     Definer  Time zone  Type       Execute at  Interval value  Interval field  Starts               Ends    Status   Originator  character_set_client  collation_connection  Database Collation 
-------  -------  -------  ---------  ---------  ----------  --------------  --------------  -------------------  ------  -------  ----------  --------------------  --------------------  --------------------
xxl_job  auto_pt  root@%   SYSTEM     RECURRING  (NULL)      24              HOUR            2021-12-15 00:00:00  (NULL)  ENABLED           1  utf8                  utf8_general_ci       utf8mb4_unicode_ci
```