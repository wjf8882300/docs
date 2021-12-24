---
title: redis批量删除key
date: 2021-12-21 19:54:27
tags: redis
categories: 工具
---

比如删除com:uecent:udata:basketball*

```
python3 redis_delete.py -d 3 -k com:uecent:udata:basketball
```
参数说明：
-d 表示第几个数据库
-k 表示前缀


redis_delete.py源码
```
# -*- coding:UTF-8 -*-
import redis
import sys
import getopt
 
############################################
# example:
# delete db0: python3 redis_delete.py  -k 'udata.climb' -d 0
# delete db1: python3 redis_delete.py  -k 'udata.climb' -d 1
############################################
 
host = '127.0.0.1'
port = '6379'
password = 'n2dasUhH0Iziq18Z'
 
def delete(key, database):
    r = redis.Redis(host=host, port=port, db=database, password=password, decode_responses=True)
    list_keys = r.keys("%s*" % key)
 
    for key in list_keys:
        r.delete(key)
 
if __name__ == "__main__":
    root_path = ""
    is_rename = True
 
    argv = sys.argv[1:]
    if len(argv) < 1:
        print('redis_delete.py -k <Key> -d <Database>')
        sys.exit()
 
    # 获取命令行参数
    try:
        opts, args = getopt.getopt(argv, "hk:d:", ["kKey=", "dDatabase="])
    except getopt.GetoptError:
        print('redis_delete.py -k <Key> -d <Database>')
        sys.exit(2)
 
    for opt, arg in opts:
        if opt == '-h':
            print('redis_delete.py -k <Key> -d <Database>')
            sys.exit()
        elif opt in ("-k", "--kKey"):
            key = arg
        elif opt in ("-d", "--dDatabase"):
            database = arg
    delete(key, int(database))
```