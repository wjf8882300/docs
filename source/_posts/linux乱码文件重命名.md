---
title: linux乱码文件重命名
date: 2021-12-23 20:39:04
tags: linux
categories: 操作系统
---


#### 1.每个文件有唯一的索引号

#### 2.获得索引号(命令：ls -i)
```
$ ls -i
 281474977250287 _config.landscape.yml  3940649674488860 public/
 281474977250288 _config.yml             281474977250290 scaffolds/
1970324837515388 db.json                 281474977250294 source/
 562949953960870 node_modules/           281474977250297 themes/
3940649674489031 package.json            281474977257164 yarn.lock
3377699721068038 package-lock.json
```
#### 3.find命令重命名：
```
# -exec后为shell命令，{}代表当前文件名，\;表示shell命令结束
find . -inum 索引号 -exec mv {} newname \;
```

#### 4.批量重命名：
```
# awk的printf命令与C语言类似，$1表示已空格分隔的第一个参数，++i变量未初始化，默认为0
ls -i | awk '{printf("find . -inum %s -exec mv {} %03d.txt \;\n",$1,++i)}' | sh
```
