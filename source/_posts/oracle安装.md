---
title: oracle安装
date: 2021-12-23 21:32:13
tags: oracle
categories: 数据库
---


#### oracle安装

1.安装依赖包
```
yum -y install gcc gcc-c++ make binutils compat-libstdc++-33 elfutils-libelf \
elfutils-libelf-devel glibc glibc-common glibc-devel \
libaio libaio-devel libgcc libstdc++ libstdc++-devel \
unixODBC unixODBC-devel
```

2.修改内核参数
```
[root@oracledb ~]# vi/etc/sysctl.conf #末尾添加如下

fs.aio-max-nr = 1048576
fs.file-max = 6553600

kernel.shmall = 10523004
kernel.shmmax = 107374182400
kernel.shmmni = 4096
kernel.sem = 610 86620 100 142

net.ipv4.ip_local_port_range= 9000 65500
net.core.rmem_default=262144
net.core.wmem_default=4194304
net.core.rmem_max=4194304
net.core.wmem_max=1048576

[root@oracledb ~]# sysctl -p (备注：用于输出配置后的结果，如果有错误会提示)
```

3.修改系统资源限制(打开进程数和文件数)
```
[root@oracledb ~]# vi/etc/security/limits.conf #末尾添加如下

[plain] view plain copy 在CODE上查看代码片派生到我的代码片
oracle soft nproc 2047
oracle hard nproc 16384
oracle soft nofile 1024
oracle hard nofile 65536

grid soft nproc 2047
grid hard nproc 16384
grid soft nofile 1024
grid hard nofile 65536

[root@oracledb ~]# vi /etc/pam.d/login
session required pam_namespace.so #下面添加一条pam_limits.so
session required /lib64/security/pam_limits.so
session required /lib/security/pam_limits.so
session required pam_limits.so
```

4.创建用户和组
```
[root@oracledb ~]# groupadd oinstall
[root@oracledb ~]# groupadd dba
[root@oracledb ~]# groupadd oper
[root@oracledb ~]# useradd -u600 -g oinstall oracle
[root@oracledb ~]# usermod -G dba,oper oracle
[root@oracledb ~]# id oracle
[root@oracledb ~]# passwd oracle
```

5.创建安装目录并赋权
```
[root@oracledb ~]# mkdir /u01
[root@oracledb ~]# mkdir /u02
[root@oracledb ~]# chown -R oracle:oinstall /u01
[root@oracledb ~]# chown -R oracle:oinstall /u02
[root@oracledb ~]# su oracle
[root@oracledb ~]# mkdir -p /u01/app/oracle/product/11.2.0/db_1
[root@oracledb ~]# mkdir -p /u02/oradata
[root@oracledb ~]# mkdir -p /u02/oradata/oracledb #oracledb为你数据库实例名
```

6.设置oracle环境变量(使用oracle帐号登录桌面，并开启terminal窗口文件最后最后加入如下环境变量的设置行)
```
[oracle@oracledb ~]# vi /home/oracle/.bash_profile
ORACLE_BASE=/u01/app/oracle; export ORACLE_BASE
ORACLE_HOME=$ORACLE_BASE/product/11.2.0/db_1; export ORACLE_HOME
LD_LIBRARY_PATH=$ORACLE_HOME/lib; export LD_LIBRARY_PATH
ORACLE_SID=oracledb; export ORACLE_SID
ORA_NLS33=$ORACLE_HOME/nls/admin/data; export ORA_NLS33
NLS_LANG=american_america.zhs16gbk;export NLS_LANG
PATH=$ORACLE_HOME/bin:$PATH; export PATH
[oracle@oracledb ~]# source /home/oracle/.bash_profile（使配置立即生效）
[oracle@oracledb ~]# env（检查环境变量设置是否OK）
```

7.上传安装文件

8.解压oracle安装文件(进入：/home/oracle/Downloads目录)
```
[oracle@oracledb~]# unzip -o -d /home/oracle/Downloads/linuxamd64_12c_database_1of2.zip
[oracle@oracledb~]# unzip -o -d /home/oracle/Downloads/linuxamd64_12c_database_2of2.zip
```

9.静默安装oracle

8、安装oracle软件
```
$ cp -R /home/oracle/database/response /home/oracle //复制一份模板
$ cd /home/oracle/response
$ vi db_install.rsp //修改安装应答文件
```
三个文件作用分别是：
db_install.rsp：安装应答
dbca.rsp：创建数据库应答
netca.rsp：建立监听、本地服务名等网络设置应答

特别是组件配置事后请用如右语句查询核实(select comp_id, comp_name, version, status from dba_registry)

配置安装应答文件db_install.rsp，如下：
```
oracle.install.option=INSTALL_DB_SWONLY
ORACLE_HOSTNAME=nmgdboracle //通过hostname命令获取
UNIX_GROUP_NAME=oinstall
INVENTORY_LOCATION=/home/oracle/oraInventory
SELECTED_LANGUAGES=en,zh_CN
ORACLE_HOME=/home/oracle/app/oracle/product/11.2.0/dbhome_1
ORACLE_BASE=/home/oracle/app
oracle.install.db.InstallEdition=EE
oracle.install.db.EEOptionsSelection=true
oracle.install.db.optionalComponents=oracle.rdbms.partitioning:11.2.0.3.0,oracle.oraolap:11.2.0.3.0,oracle.rdbms.dm:11.2.0.3.0,oracle.rdbms.dv:11.2.0.3.0,oracle.rdbms.lbac:11.2.0.3.0,oracle.rdbms.rat:11.2.0.3.0
oracle.install.db.DBA_GROUP=dba
oracle.install.db.OPER_GROUP=oinstall
oracle.install.db.config.starterdb.type=GENERAL_PURPOSE
oracle.install.db.config.starterdb.globalDBName=xtwl
oracle.install.db.config.starterdb.SID=xtwl
oracle.install.db.config.starterdb.characterSet=AL32UTF8
oracle.install.db.config.starterdb.memoryOption=true
oracle.install.db.config.starterdb.memoryLimit=10240 //物理内存的60%左右
oracle.install.db.config.starterdb.password.ALL=oracle //注意修改
oracle.install.db.config.starterdb.control=DB_CONTROL
SECURITY_UPDATES_VIA_MYORACLESUPPORT=false
DECLINE_SECURITY_UPDATES=true //一定要是true
```

```
$cd /home/oracle/database
$./runInstaller -silent -responseFile /home/oracle/response/db_install.rsp //了解安装进度 tail -f /home/oracle/oraInventory/logs/installActions*log
```

//当安装界面出现如下信息的时候
The installation of Oracle Database 11g was successful.
Please check '/home/oracle/oraInventory/logs/silentInstall2016-02-04_09-21-13AM.log' for more details.

As a root user, execute the following script(s):
1. /home/oracle/oraInventory/orainstRoot.sh
2. /home/oracle/app/oracle/product/11.2.0/dbhome_1/root.sh


Successfully Setup Software.

//在新打开的root登录的窗口中执行下面的脚本
```
#/home/oracle/oraInventory/orainstRoot.sh
#/home/oracle/app/oracle/product/11.2.0/dbhome_1/root.sh
```
//执行完上面的脚本后回到安装界面按下Enter键以继续


9、配置oracle监听：
```
$cd /home/oracle/response

$netca /silent /responsefile /home/oracle/response/netca.rsp
```
成功运行后，在/home/oracle/app/oracle/product/11.2.0/dbhome_1/network/admin目录下生成sqlnet.ora和listener.ora两个文件。
通过 netstat -tlnp 命令，看到
tcp 0 0 0.0.0.0:1521 0.0.0.0:* LISTEN 22494/tnslsnr
说明监听器已经在1521端口上开始工作了


10、安装oracle数据库
```
$cd /home/oracle/response
$vi dbca.rsp //修改创建数据库应答文件

RESPONSEFILE_VERSION = "11.2.0"
OPERATION_TYPE = "createDatabase"
#-----------------------*** End of GENERAL section ***------------------------
GDBNAME = "xtwl"
SID = "xtwl"
TEMPLATENAME = "General_Purpose.dbc"
SYSPASSWORD = "oracle"
SYSTEMPASSWORD = "oracle"
DATAFILEDESTINATION = /home/oracle/app/oradata
#RECOVERYAREADESTINATION=/home/oracle/backup //该项参数设置无效，默认恢复表空间仍然是$ORACLE_BASE/flash_recovery_area
CHARACTERSET = "AL32UTF8"
TOTALMEMORY = "10240" //物理内存的60%左右
#-----------------------*** End of CREATEDATABASE section ***------------------------

$dbca -silent -responseFile /home/oracle/response/dbca.rsp (静默卸载：dbca -silent -deleteDatabase -sourcedb xtwl -sid xtwl)
```

看到下面语句说明创建成功
Look at the log file "/home/oracle/app/cfgtoollogs/dbca/xtwl/xtwl.log" for further details.

查看创建情况
```
$cat /home/oracle/app/cfgtoollogs/dbca/xtwl/xtwl.log
```

建库后实例检查：
```
$ps -ef | grep ora_ | grep -v grep
oracle 39754 1 0 10:20 ? 00:00:00 ora_pmon_xtwl
oracle 39756 1 0 10:20 ? 00:00:00 ora_vktm_xtwl
oracle 39760 1 0 10:20 ? 00:00:00 ora_gen0_xtwl
oracle 39762 1 0 10:20 ? 00:00:00 ora_diag_xtwl
oracle 39764 1 0 10:20 ? 00:00:00 ora_dbrm_xtwl
oracle 39766 1 0 10:20 ? 00:00:00 ora_psp0_xtwl
oracle 39768 1 0 10:20 ? 00:00:00 ora_dia0_xtwl
oracle 39770 1 0 10:20 ? 00:00:00 ora_mman_xtwl
oracle 39772 1 0 10:20 ? 00:00:00 ora_dbw0_xtwl
oracle 39774 1 0 10:20 ? 00:00:00 ora_lgwr_xtwl
oracle 39776 1 0 10:20 ? 00:00:00 ora_ckpt_xtwl
oracle 39778 1 0 10:20 ? 00:00:00 ora_smon_xtwl
oracle 39780 1 0 10:20 ? 00:00:00 ora_reco_xtwl
oracle 39782 1 0 10:20 ? 00:00:00 ora_mmon_xtwl
oracle 39784 1 0 10:20 ? 00:00:00 ora_mmnl_xtwl
oracle 39786 1 0 10:20 ? 00:00:00 ora_d000_xtwl
oracle 39788 1 0 10:20 ? 00:00:00 ora_s000_xtwl
oracle 39798 1 0 10:20 ? 00:00:00 ora_qmnc_xtwl
oracle 39813 1 0 10:20 ? 00:00:00 ora_cjq0_xtwl
oracle 39815 1 0 10:20 ? 00:00:00 ora_q000_xtwl
oracle 39817 1 0 10:20 ? 00:00:00 ora_q001_xtwl
```

查看监听状态
```
$lsnrctl status
```

三、参数修改：

需要手动备份spfile文件：
```
cp $ORACLE_HOME/dbs/spfilextwl.ora $ORACLE_HOME/dbs/spfilextwl_bak.ora
```

1、修改最大连接数：
```
sql> show parameter processes;
sql> alter system set processes=2000 scope = spfile;
```

2、禁止回收站功能：
```
SQL> show parameter recyclebin;
SQL> alter system set recyclebin=off scope=spfile;
```

3、关闭审计功能：
```
SQL> show parameter audit;
SQL> alter system set audit_trail=NONE scope=spfile;
```

4、修改用户密码用不过期：
```
SQL> select * from dba_profiles s where s.profile='DEFAULT' and resource_name='PASSWORD_LIFE_TIME';
PROFILE RESOURCE_NAME RESOURCE
------------------------------ -------------------------------- --------
LIMIT
----------------------------------------
DEFAULT PASSWORD_LIFE_TIME PASSWORD
180
SQL> alter profile default limit password_life_time unlimited;
Profile altered.
SQL> select * from dba_profiles s where s.profile='DEFAULT' and resource_name='FAILED_LOGIN_ATTEMPTS';
PROFILE RESOURCE_NAME RESOURCE
------------------------------ -------------------------------- --------
LIMIT
----------------------------------------
DEFAULT FAILED_LOGIN_ATTEMPTS PASSWORD
10
SQL> alter profile default limit failed_login_attempts unlimited;
Profile altered.
```

5、修改控制文件里可重复使用的记录所能保存的最小天数：（一般设置为45天）
```
SQL> show parameter control;

NAME TYPE VALUE
------------------------------------ ----------- ------------------------------
control_file_record_keep_time integer 7
control_files string /home/oracle/app/oradata/xtwl/control01.ctl, /home/oracle/app/flash_recovery_area/xtwl/control02.ctl
control_management_pack_access string DIAGNOSTIC+TUNING

SQL> alter system set control_file_record_keep_time=45 scope=spfile;

System altered.
```

6、设置数据库为自动内存管理模式：

（1）修改数据库为自动内存管理模式：
```
SQL> alter system set memory_target=10240M scope=spfile; //物理内存的60%左右。
System altered.

SQL> alter system set memory_max_target=10240M scope=spfile; //物理内存的60%左右。
System altered.

SQL> alter system set sga_target=0 scope=spfile;
System altered.

SQL> alter system set sga_max_size=7168M scope=spfile; //实例内存的70%左右，即memory_max_target*70%，也即物理内存*60%*70%。
System altered.

SQL> alter system set pga_aggregate_target=0 scope=spfile;
System altered.

SQL> alter system set pre_page_sga=FALSE scope=spfile;
System altered.
```
（2）重启数据库：
```
SQL> shutdown immediate;
Database closed.
Database dismounted.
ORACLE instance shut down.
SQL> startup;
ORACLE instance started.

Total System Global Area 2254802944 bytes
Fixed Size 2215344 bytes
Variable Size 1073742416 bytes
Database Buffers 1174405120 bytes
Redo Buffers 4440064 bytes
Database mounted.
Database opened.
```

（3）查看各个内存参数设置：
```
SQL> show parameter sga;

NAME TYPE VALUE
------------------------------------ ----------- ------------------------------
lock_sga boolean FALSE
pre_page_sga boolean FALSE
sga_max_size big integer 7G
sga_target big integer 0
SQL> show parameter pga;

NAME TYPE VALUE
------------------------------------ ----------- ------------------------------
pga_aggregate_target big integer 0
SQL> show parameter memory;

NAME TYPE VALUE
------------------------------------ ----------- ------------------------------
hi_shared_memory_address integer 0
memory_max_target big integer 10G
memory_target big integer 10G
shared_memory_address integer 0
```


四、修改redo log组以及大小：--为防止日志频繁切换，引起数据库性能低下问题。

1、创建redo日志存放目录：
```
$ mkdir -p /home/oracle/app/oradata/xtwllog/
$ chmod 750 /home/oracle/app/oradata/xtwllog/
```

2、查询日志信息：
```
SQL> select group#,bytes/1024/1024,status from v$log;

GROUP# BYTES/1024/1024 STATUS
---------- --------------- ----------------
1 50 ACTIVE
2 50 CURRENT
3 50 ACTIVE
```

3、查询日志目录：
```
SQL> select * from v$logfile order by group#;

GROUP# STATUS TYPE MEMBER IS_
---------- ------- ------- ------------------------------------------ ----------------
1 ONLINE /home/oracle/app/oradata/xtwl/redo01.log NO
2 ONLINE /home/oracle/app/oradata/xtwl/redo02.log NO
3 ONLINE /home/oracle/app/oradata/xtwl/redo03.log NO
```

4、新增两组日志组，每组500M：
```
SQL> alter database add logfile group 4 '/home/oracle/app/oradata/xtwllog/redo04.log' size 500M;

Database altered.

SQL> alter database add logfile group 5 '/home/oracle/app/oradata/xtwllog/redo05.log' size 500M;

Database altered.
```

5、查询4、5两组日志是否成功添加：
```
SQL> select group#,bytes/1024/1024,status from v$log;

GROUP# BYTES/1024/1024 STATUS
---------- --------------- ----------------
1 50 INACTIVE
2 50 CURRENT
3 50 INACTIVE
4 500 UNUSED
5 500 UNUSED
```

6、删除日志组1：
```
SQL> alter database drop logfile group 1;

Database altered.
```

7、删除日志组2报错：
```
SQL> alter database drop logfile group 2;
alter database drop logfile group 2
*
ERROR at line 1:
ORA-01623: log 2 is current log for instance xtwl (thread 1) - cannot drop
ORA-00312: online log 2 thread 1: '/home/oracle/app/oradata/xtwl/redo02.log'
```

8、需要手动切换日志多次，使新建的日志组能够应用：
```
SQL> alter system switch logfile;

System altered.


SQL> select group#,bytes/1024/1024,status from v$log;

GROUP# BYTES/1024/1024 STATUS
---------- --------------- ----------------
2 50 ACTIVE
3 50 INACTIVE
4 500 CURRENT
5 500 UNUSED

SQL> alter system switch logfile;

System altered.

SQL> select group#,bytes/1024/1024,status from v$log;

GROUP# BYTES/1024/1024 STATUS
---------- --------------- ----------------
2 50 ACTIVE
3 50 INACTIVE
4 500 ACTIVE
5 500 CURRENT
```

9、使用alter system checkpoint将Active的日志状态置为INACTIVE：
```
SQL> alter system checkpoint;

System altered.

SQL> select group#,bytes/1024/1024,status from v$log;

GROUP# BYTES/1024/1024 STATUS
---------- --------------- ----------------
2 50 INACTIVE
3 50 INACTIVE
4 500 INACTIVE
5 500 CURRENT
```

10、删除原2，3日志组：
```
SQL> alter database drop logfile group 2;

Database altered.

SQL> alter database drop logfile group 3;

Database altered.
```

11、新增1，2，3日志组，每组500M：
```

SQL> alter database add logfile group 1 '/home/oracle/app/oradata/xtwllog/redo01.log' size 500M;

Database altered.

SQL> alter database add logfile group 2 '/home/oracle/app/oradata/xtwllog/redo02.log' size 500M;

Database altered.

SQL> alter database add logfile group 3 '/home/oracle/app/oradata/xtwllog/redo03.log' size 500M;

Database altered.

SQL> SELECT group#, members, bytes/1024/1024 byte_mb, status FROM v$log;


GROUP# MEMBERS BYTE_MB STATUS
---------- ---------- ---------- ----------------
1 1 500 UNUSED
2 1 500 UNUSED
3 1 500 UNUSED
4 1 500 INACTIVE
5 1 500 CURRENT
```

12、多次执行切换日志操作，使新建的日志组都能正常应用：
```
SQL> alter system switch logfile;

System altered.

SQL> alter system switch logfile;

System altered.

SQL> alter system switch logfile;

System altered.

SQL> SELECT group#, members, bytes/1024/1024 byte_mb, status FROM v$log;

GROUP# MEMBERS BYTE_MB STATUS
---------- ---------- ---------- ----------------
1 1 500 ACTIVE
2 1 500 ACTIVE
3 1 500 CURRENT
4 1 500 INACTIVE
5 1 500 ACTIVE


SQL> select * from v$logfile order by group#;
GROUP# STATUS TYPE MEMBER IS_
---------- ------- ------- ------------------------------------------ ----------------
1 ONLINE /home/oracle/app/oradata/xtwllog/redo01.log NO
2 ONLINE /home/oracle/app/oradata/xtwllog/redo02.log NO
3 ONLINE /home/oracle/app/oradata/xtwllog/redo03.log NO
4 ONLINE /home/oracle/app/oradata/xtwllog/redo04.log NO
5 ONLINE /home/oracle/app/oradata/xtwllog/redo05.log NO
```

13、删除原redo日志文件，释放磁盘空间：
```
$ rm /home/oracle/app/oradata/xtwl/redo0*.log
```

五、修改数据库为归档日志模式：
1、新建归档日志存放目录：
```
$ mkdir -p /home/oracle/app/archlog
$ chmod 750 /home/oracle/app/archlog
```

2、停止数据库：
```
SQL> shutdown immediate
Database closed.
Database dismounted.
ORACLE instance shut down.
```

3、启动数据库到mount状态：
```
SQL> startup mount
ORACLE instance started.
Total System Global Area 1603411968 bytes
Fixed Size 2213776 bytes
Variable Size 1023412336 bytes
Database Buffers 570425344 bytes
Redo Buffers 7360512 bytes
Database mounted.
```

4、修改日志模式为归档模式：
```
SQL> alter database archivelog;
Database altered.
```

5、修改归档日志格式：
```
SQL> alter system set log_archive_format='xtwldb_%t_%s_%r.log' scope=spfile;
System altered.
```

6、修改归档日志路径：
```
SQL> alter system set log_archive_dest1='location=/home/oracle/app/archlog' scope=spfile;
System altered.
```

7、打开数据库：
```
SQL> alter database open;
Database altered.
```

8、重启数据库使各参数生效：
```
SQL> shutdown immediate;
Database closed.
Database dismounted.
ORACLE instance shut down.


SQL> startup;
ORACLE instance started.

Total System Global Area 1603411968 bytes
Fixed Size 2213776 bytes
Variable Size 570427504 bytes
Database Buffers 1023410176 bytes
Redo Buffers 7360512 bytes
Database mounted.
Database opened.
```

#### 日常维护

(1)启动监听
```
[oracle@oracledb~]lsnrctl start
```

(2)启动数据库
```
sqlplus system/system as sysdba
SQL> startup
SQL> alter system register;
```
(3)查看监听状态
```
lsnrctl status
```

(3)启动管理平台
```
[oracle@oracledb~]emctl start dbconsole
```