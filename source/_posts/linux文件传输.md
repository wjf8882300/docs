---
title: linux文件传输
date: 2021-12-23 20:26:06
tags: linux
categories: 操作系统
---

#### 1.Linux下scp的用法（不支持断点续传）
##### 1.1.将本机文件复制到远程服务器上

```
# 将当前服务器/data目录下面的test.txt复制到远程服务器192.168.6.129的tmp目录下面
scp /data/test.txt root@192.168.6.129:/tmp
```
参数说明：
- /data      本地文件的绝对路径
- test.txt   要复制到服务器上的本地文件
- root       通过root用户登录到远程服务器（也可以使用其他拥有同等权限的用户）
- 192.168.6.129 远程服务器的ip地址（也可以使用域名或机器名）
- /etc/squid    将本地文件复制到位于远程服务器上的路径



##### 1.2.将远程服务器上的文件复制到本机

```
# 将远程服务器192.168.6.129的tmp目录下面的test.txt复制到当前服务器/data目录
scp root@192.168.6.129:/tmp/test.txt /data
```
参数说明：
- root  通过root用户登录到远程服务器（也可以使用其他拥有同等权限的用户）
- 192.168.6.129 远程服务器的域名（当然也可以使用该服务器ip地址）
- /tmp/test.txt  欲复制到本机的位于远程服务器上的文件
- /data 将远程文件复制到本地的绝对路径

#### 2.Linux下rsync用法（支持断点续传）
##### 2.1文件断点下载

```
# 将远程服务器192.168.6.129的/data/large.tar.gz文件复制到当前服务器/tmp目录
rsync -P --rsh=ssh root@192.168.6.129:/data/large.tar.gz /tmp/targe.tar.gz
```


##### 2.2文件断点上传
```
# 将当前服务器的/tmp/targe.tar.gz文件上传到远程服务器192.168.6.129的/data目录下
rsync -P --rsh=ssh /tmp/targe.tar.gz root@192.168.6.129:/data/large.tar.gz
```

##### 2.3文件目录断点下载
```
# 将远程服务器192.168.6.129的/data目录复制到当前服务器/data目录
rsync -P --rsh=ssh -r root@192.168.6.129:/data /data
```

##### 2.4文件目录断点上传
```
# 将当前服务器的/data目录上传到远程服务器192.168.6.129的/data目录
rsync -P --rsh=ssh -r /data root@192.168.6.129:/data
```
