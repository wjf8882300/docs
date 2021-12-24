---
title: elasticsearch查询
date: 2021-12-23 20:44:11
tags: elasticsearch
categories: 工具
---

#### 查询索引mapper
```
GET uecent-20200813
 
输出：
{
  "uecent-20200813": {
    "aliases": {},
    "mappings": {
      "java_log": {
        "properties": {
          "@log_name": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "@source": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "@timestamp": {
            "type": "date"
          },
          "args": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "city_name": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "client": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "container_id": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "container_name": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "country_name": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "domain": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "fluentd-folder": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "fluentd-ip": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "fluentd-ip-in": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "fluentd-project": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "hostname": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "http_user_agent": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "https": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "ip": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "level": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "location": {
            "type": "geo_point"
          },
          "log": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "logdate": {
            "type": "date"
          },
          "partial_id": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "partial_last": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "partial_message": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "partial_ordinal": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "pid": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "referer": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "region_name": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "request": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "request_method": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "responsetime": {
            "type": "float"
          },
          "scheme": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "size": {
            "type": "long"
          },
          "source": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "status": {
            "type": "long"
          },
          "thread": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "upstreamaddr": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "upstreamtime": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          }
        }
      }
    },
    "settings": {
      "index": {
        "creation_date": "1597285570606",
        "number_of_shards": "5",
        "number_of_replicas": "1",
        "uuid": "L1_5Cf0GSlWbTYyOt9Dm3g",
        "version": {
          "created": "5061299"
        },
        "provided_name": "uecent-20200813"
      }
    }
  }
}
```

#### 普通查询

```
# 查询索引uecent-20200813
# size表示返回前500条
# query表示查询条件
#      filter过滤
#          range表示范围，下面例子中表示查询时间戳字段为@timestamp，大于等于1597288423982且小于等于1597288723982的数据
#          query_string查询字符串
#              analyze_wildcard表示是否分析通配符查询或前缀查询，true表示是
#              query：domain.keyword表示关键字domain的字段（精确匹配），fluentd-project:模糊匹配
# sort表示按照时间排序
GET /uecent-20200813/_search
{
    "size": 500,
    "query": {
        "bool": {
            "filter": [
                {
                    "range": {
                        "@timestamp": {
                            "gte": "1597288423982",
                            "lte": "1597288723982",
                            "format": "epoch_millis"
                        }
                    }
                },
                {
                    "query_string": {
                        "analyze_wildcard": true,
                        "query": "domain.keyword:api.uecent.com AND fluentd-project:'nginx' AND status:(\"101\" OR \"200\" OR \"302\" OR \"304\" OR \"400\" OR \"404\" OR \"499\" OR \"500\") AND NOT http_user_agent:'Zabbix'"
                    }
                }
            ]
        }
    },
    "sort": {
        "@timestamp": {
            "order": "desc",
            "unmapped_type": "boolean"
        }
    },
    "script_fields": {},
    "docvalue_fields": [
        "@timestamp"
    ]
}
 
输出：
{
  "took": 17,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1805,
    "max_score": null,
    "hits": [
      {
        "_index": "uecent-20200813",
        "_type": "java_log",
        "_id": "NTc1NTJlZjYtNWU0Yy00ZTM1LWIyYTAtODE0MTdmYjE1YzQ4",
        "_score": null,
        "_source": {
          "fluentd-ip-in": "172.19.115.16",
          "container_id": "c5a0cc3c416b582d21d3161e9b1f77dbd739d5028284d5a5ad0838e1cdf0c854",
          "container_name": "/nginx",
          "source": "stdout",
          "log": """{"@timestamp":"2020-08-13T11:18:43+08:00","@source":"172.17.0.8","hostname":"c5a0cc3c416b","ip":"114.95.120.93","client":"114.95.120.93","request_method":"GET","scheme":"http","domain":"api.uecent.com","referer":"-","request":"/udata/api/v1/football/common/detail/255719476464328704","args":"-","size":2580,"status": 200,"responsetime":0.283,"upstreamtime":"0.282","upstreamaddr":"172.19.115.50:80","http_user_agent":"Java/1.8.0_181","https":""}""",
          "fluentd-folder": "dev-live",
          "fluentd-project": "nginx",
          "@source": "172.17.0.8",
          "hostname": "c5a0cc3c416b",
          "ip": "114.95.120.93",
          "client": "114.95.120.93",
          "request_method": "GET",
          "scheme": "http",
          "domain": "api.uecent.com",
          "referer": "-",
          "request": "/udata/api/v1/football/common/detail/255719476464328704",
          "args": "-",
          "size": 2580,
          "status": 200,
          "responsetime": 0.283,
          "upstreamtime": "0.282",
          "upstreamaddr": "172.19.115.50:80",
          "http_user_agent": "Java/1.8.0_181",
          "https": "",
          "logdate": "2020-08-13T11:18:43.000000000+0800",
          "@timestamp": "2020-08-13T11:18:43.000000000+08:00",
          "@log_name": "nginx"
        },
        "fields": {
          "@timestamp": [
            1597288723000
          ]
        },
        "sort": [
          1597288723000
        ]
      },
...
```


#### 聚合查询
```
# aggs为聚合条件
#    terms表示以city_name.keyword这个条件进行分组
#    geohash_grid表示在上面的基础再分组
GET /uecent-20200813/_search
{
    "size": 0,
    "query": {
        "bool": {
            "filter": [
                {
                    "range": {
                        "@timestamp": {
                            "gte": "1597305930213",
                            "lte": "1597306230213",
                            "format": "epoch_millis"
                        }
                    }
                },
                {
                    "query_string": {
                        "analyze_wildcard": true,
                        "query": "domain.keyword:grafana.uecent.com AND fluentd-project:\"nginx\" AND status:(\"101\" OR \"200\" OR \"304\" OR \"404\" OR \"499\" OR \"500\") AND NOT http_user_agent:'Zabbix'"
                    }
                }
            ]
        }
    },
    "aggs": {
        "4": {
            "terms": {
                "field": "city_name.keyword",
                "size": 10,
                "order": {
                    "_term": "desc"
                },
                "min_doc_count": 1
            },
            "aggs": {
                "3": {
                    "geohash_grid": {
                        "field": "location",
                        "precision": 3
                    },
                    "aggs": {}
                }
            }
        }
    }
}
 
输出：
{
  "took": 36,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 11,
    "max_score": 0,
    "hits": []
  },
  "aggregations": {
    "4": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "3": {
            "buckets": [
              {
                "key": "wtw",
                "doc_count": 11
              }
            ]
          },
          "key": "上海",
          "doc_count": 11
        }
      ]
    }
  }
}
```

#### 查询当前日志最多的项目
```
GET /_search
{
    "size": 0,
    "query": {
        "bool": {
            "filter": [
                {
                    "range": {
                        "@timestamp": {
                            "gte": "1599098400000",
                            "lte": "1599102000000",
                            "format": "epoch_millis"
                        }
                    }
                }
            ]
        }
    },
    "aggs": {
        "3": {
            "terms": {
                "field": "fluentd-project.keyword",
                "size": 15,
                "order": {
                    "_count": "desc"
                },
                "min_doc_count": 1
            }
        }
    }
}
```