---
title: elasticsearch操作
date: 2021-12-23 20:44:21
tags: elasticsearch
categories: 工具
---


#### 查看索引
```
curl -XGET "http://localhost:9200/_cat/indices?v"
health status index            uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   fluentd-20200305 glto7qAvQ9K6WRKXu2ymRw   5   1   25652327            0        6gb            6gb
yellow open   .kibana          nni-Brg0TaKJne5HLxywUw   1   1         17            2     34.7kb         34.7kb
yellow open   fluentd-20200304 9v0nUR5UQ0uadzL2JdNGBw   5   1   25048041            0     18.2gb         18.2gb
yellow open   fluentd-20200303 _03cCwgOTo6Sxu53x-2-eQ   5   1   19563551            0     12.3gb         12.3gb
health：red表示不可用，yellow或者green才可用
```


#### 清理日志脚本
```
#/bin/bash
 
#指定日期(5天前)
DATA=`date -d "5 day ago" +%Y%m%d`
 
#当前日期
time=`date`
 
#删除7天前的日志
curl -XDELETE http://127.0.0.1:9200/fluentd-${DATA}
 
if [ $? -eq 0 ];then
  echo $time"-->del $DATA log success.." >> /tmp/es-index-clear.log
else
  echo $time"-->del $DATA log fail.." >> /tmp/es-index-clear.log
fi
```

#### 添加到任务计划
```
crontab -e 10 1 * * * sh /root/clear_log.sh > /dev/null 2>&1
```




Elasticsearch中信息很多，同时ES也有很多信息查看命令，可以帮助开发者快速查询Elasticsearch的相关信息。

```
_cat
$ curl localhost:9200/_cat
# 查看各个节点磁盘使用情况
/_cat/allocation
shards disk.indices disk.used disk.avail disk.total disk.percent host          ip            node
    17       28.6gb    42.8gb    153.8gb    196.7gb           21 172.19.115.65 172.19.115.65 es-node3
    17       18.6gb    34.9gb    161.7gb    196.7gb           17 172.19.115.62 172.19.115.62 es-node1
    16       24.8gb    38.3gb    158.3gb    196.7gb           19 172.19.115.64 172.19.115.64 es-node2
    17       21.6gb    50.9gb    145.8gb    196.7gb           25 172.19.115.18 172.19.115.18 es-node0
 
# 查看各个节点分片情况
/_cat/shards
/_cat/shards/{index}
index                    shard prirep state      docs   store ip            node
uecent-20200622          2     p      STARTED 3208156   2.6gb 172.19.115.62 es-node1
uecent-20200622          2     r      STARTED 3208156   2.4gb 172.19.115.64 es-node2
uecent-20200622          3     r      STARTED 3207549   3.5gb 172.19.115.65 es-node3
uecent-20200622          3     p      STARTED 3207546   2.4gb 172.19.115.64 es-node2
uecent-20200622          4     p      STARTED 3210288   2.4gb 172.19.115.65 es-node3
uecent-20200622          4     r      STARTED 3210287   2.3gb 172.19.115.18 es-node0
uecent-20200622          1     p      STARTED 3211264   2.4gb 172.19.115.18 es-node0
uecent-20200622          1     r      STARTED 3211264   2.5gb 172.19.115.62 es-node1
uecent-20200622          0     p      STARTED 3208470   2.6gb 172.19.115.65 es-node3
 
# 查看主节点
/_cat/master
id                     host          ip            node
6k8UWh-jQRqPsJoiV8IHEg 172.19.115.65 172.19.115.65 es-node3
 
# 查看所有节点
/_cat/nodes
ip            heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
172.19.115.64           60          96   5    0.44    0.35     0.35 mdi       -      es-node2
172.19.115.65           39          98   9    0.69    0.60     0.56 mdi       *      es-node3
172.19.115.62           36          98   6    0.45    0.42     0.36 mdi       -      es-node1
172.19.115.18           69          98  39    1.32    1.23     1.07 mdi       -      es-node0
 
# 查看索引
/_cat/indices
/_cat/indices/{index}
health status index                    uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   smartxoffice_oax_message 58jTM6mvTqa1hliNHUpAcw   5   1         16            1    198.9kb         99.4kb
green  open   uecent-20200621          tbqfnUPiT3SMOjNgGptw9w   5   1   48523671            0     66.5gb         33.2gb
green  open   smartxoffice_oax_group   rKx-RizXRwKhcIzPyCJGFw   5   1          1            0      2.9mb          1.4mb
green  open   .kibana                  pF9hvtvRRQ24aGdM2UloVw   1   1         25           18    162.8kb         81.4kb
green  open   smartxoffice_oax_friend  dm5Xm1HMTpizvQDMwVYndA   5   1         10          814      1.8mb        954.4kb
green  open   kefu_question_info       lm5ANJsoSmiA7cxfRgn1Fw   5   0          3            0       14kb           14kb
green  open   uecent-20200622          oFySuI7mREOZBtqhwjPFUQ   5   1   16101241            0       27gb         12.6gb
green  open   smartxoffice_oax_user    nxJK9JxtQGOm0d7Eyrifjg   5   1          6         2507    278.9kb        139.4kb
 
# 查看
/_cat/segments
/_cat/segments/{index}
index                    shard prirep ip            segment generation docs.count docs.deleted    size size.memory committed searchable version compound
smartxoffice_oax_message 0     r      172.19.115.62 _1               1          1            0   6.1kb        3200 true      true       6.6.1   true
smartxoffice_oax_message 0     r      172.19.115.62 _3               3          1            0   6.1kb        3200 true      true       6.6.1   true
smartxoffice_oax_message 0     r      172.19.115.62 _7               7          1            0   6.5kb        3505 true      true       6.6.1   true
smartxoffice_oax_message 0     p      172.19.115.64 _1               1          1            0   6.1kb        3200 true      true       6.6.1   true
smartxoffice_oax_message 0     p      172.19.115.64 _3               3          1            0   6.1kb        3200 true      true       6.6.1   true
smartxoffice_oax_message 0     p      172.19.115.64 _7               7          1            0   6.5kb        3505 true      true       6.6.1   true
smartxoffice_oax_message 1     r      172.19.115.65 _1               1          1            0   6.5kb        3505 true      true       6.6.1   true
smartxoffice_oax_message 1     r      172.19.115.65 _3               3          1            0   6.1kb        3200 true      true       6.6.1   true
smartxoffice_oax_message 1     p      172.19.115.18 _1               1          1            0   6.5kb        3505 true      true       6.6.1   true
smartxoffice_oax_message 1     p      172.19.115.18 _3               3          1            0   6.1kb        3200 true      true       6.6.1   true
 
# 查看记录条数
/_cat/count
/_cat/count/{index}
epoch      timestamp count
1592816612 17:03:32  64672303
 
#
/_cat/recovery
/_cat/recovery/{index}
index                    shard time  type        stage source_host   source_node target_host   target_node repository snapshot files files_recovered files_percent files_total bytes      bytes_recovered bytes_percent bytes_total translog_ops translog_ops_recovered translog_ops_percent
smartxoffice_oax_message 0     618ms peer        done  172.19.115.18 es-node0    172.19.115.62 es-node1    n/a        n/a      10    10              100.0%        10          19610      19610           100.0%        19610       0            0                      100.0%
smartxoffice_oax_message 0     590ms peer        done  172.19.115.18 es-node0    172.19.115.64 es-node2    n/a        n/a      10    10              100.0%        10          19610      19610           100.0%        19610       0            0                      100.0%
smartxoffice_oax_message 1     56ms  peer        done  172.19.115.64 es-node2    172.19.115.65 es-node3    n/a        n/a      7     7               100.0%        7           13280      13280           100.0%        13280       0            0                      100.0%
smartxoffice_oax_message 1     106ms peer        done  172.19.115.64 es-node2    172.19.115.18 es-node0    n/a        n/a      7     7               100.0%        7           13280      13280           100.0%        13280       0            0                      100.0%
 
# 查询健康状态
/_cat/health
epoch      timestamp cluster               status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1592816675 17:04:35  elasticsearch-cluster green           4         4     67  36    0    0        0             0                  -                100.0%
 
#
/_cat/pending_tasks
/_cat/aliases
/_cat/aliases/{alias}
/_cat/thread_pool
/_cat/plugins
/_cat/fielddata
/_cat/fielddata/{fields}
/_cat/nodeattrs
/_cat/repositories
/_cat/snapshots/{repository}
```

verbose
每个命令都支持使用?v参数，来显示详细的信息：
```
$ curl localhost:9200/_cat/master?v
id                     host      ip        node
yBet3cYzQbC68FRzLZDmFg 127.0.0.1 127.0.0.1 lihao
```

help
每个命令都支持使用help参数，来输出可以显示的列：
```
$ curl localhost:9200/_cat/master?help
id   |   | node id
host | h | host name
ip   |   | ip address
node | n | node name
```

headers
通过h参数，可以指定输出的字段：
```
$ curl localhost:9200/_cat/master?v
id                     host      ip        node
yBet3cYzQbC68FRzLZDmFg  127.0.0.1 127.0.0.1 lihao
 
$ curl localhost:9200/_cat/master?h=ip,node
127.0.0.1 lihao
```

数字类型的格式化
很多的命令都支持返回可读性的大小数字，比如使用mb或者kb来表示。
```
$ curl localhost:9200/_cat/indices?v
health status index                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   aaa                       5   1          2            0      7.2kb          7.2kb
yellow open   logstash-eos-2016.09.01   5   1        297            0    202.3kb        202.3kb
yellow open   bank                      5   1       1001            1    451.6kb        451.6kb
yellow open   website                   5   1          2            0      7.8kb          7.8kb
yellow open   .kibana                   1   1          5            1     26.6kb         26.6kb
yellow open   logstash-eos-2016.09.02   5   1         11            0     33.9kb         33.9kb
yellow open   test-2016.09.01           5   1          1            0      3.9kb          3.9kb
yellow open   testst_index              5   1          0            0       795b           795b
```