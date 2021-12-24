---
title: java内存诊断
date: 2021-12-23 13:59:09
tags: java
categories: 语言
---

针对内存溢出、内存泄露的情况，需要定位哪些数据占用了大量内存，且不释放，并定位内存溢出或者泄露的代码。

#### 1.通过dump解析

java通过dump生成内存分析的文件，再通过eclipse memory analyzer工具分析定位。



##### 1.1.启动时添加
启动jar包时添加红字部分，便于通过jvm监控工具链接程序，并进行内存监控。
```
java -jar \
-Dspring.profiles.active=${PROFILE} \
-Djava.rmi.server.hostname=${IP} \
-Dcom.sun.management.jmxremote \
-Dcom.sun.management.jmxremote.port=1099 \
-Dcom.sun.management.jmxremote.rmi.port=1099 \
-Dcom.sun.management.jmxremote.authenticate=false \
-Dcom.sun.management.jmxremote.ssl=false \
bill.1.0.0.jar
```
##### 1.2.jvm监控工具
C:\Program Files (x86)\Java\jdk1.8.0_191\bin\jvisualvm.exe

###### 1）添加要监控的服务器
![](java-memory-01.png)	


###### 2）监控内存、cpu、线程使用情况

点击堆Dump即可生成内存分析文件，系统会提示文件存储在服务器上的位置。Docker容器放在设置的数据目录下。

比如生成文件/tmp/heapdump-1589453851824.hprof，把文件下载到本地，等待分析。
![](java-memory-02.png)	
![](java-memory-03.png)	



##### 1.3.eclipse memory analyzer工具
使用本工具打开上个步骤生成的hprof文件，会生成如下图所示。
![](java-memory-04.png)	

![](java-memory-05.png)	

![](java-memory-06.png)	


#### 2.通过arthas在线查看
arthas服务端
```
docker exec -i provider-auth \
/bin/sh -c 'java -jar /opt/arthas/arthas-boot.jar 6 --attach-only --session-timeout=31536000 --tunnel-server ws://172.19.115.12:7777/ws --agent-id 172.19.115.8'
```
手工调试：

```
docker exec -it provider-auth sh

java -jar /opt/arthas/arthas-boot.jar 6
```
--attach-only # 表示非交互式启动
--tunnel-server # 表示tunnel-server地址和端口
--agent-id # 表示当前arthas的标识

![](java-memory-07.png)	

常用命令：
help    # 帮助
dashboard # 查看面板
thread  # 查看所有线程
thread -b # 查看锁住的线程
thread -n 3 # 查看前3个占用cpu最高的线程

dashboard命令的使用
![](java-memory-08.png)	


thread命令的使用
下面的例子查看线程id为103的线程，显示出这个线程执行的代码为rabbitmq监听，可以通过这种方式定位出问题的代码。
![](java-memory-09.png)	


heapdum生成内存分析文件
![](java-memory-10.png)	