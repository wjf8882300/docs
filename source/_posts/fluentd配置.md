---
title: fluentd配置
date: 2021-12-23 21:08:40
tags: elasticsearch
categories: 工具
---

> fluentd用来收集日志，然后转发给elasticsearch，这样通过kibana查看日志

![最终效果](kibana.png)

#### client配置：
```
# fluentd/etc/fluent.conf
<source>
  @type forward
  port 24224
  bind 0.0.0.0
</source>
<filter service.provider*>
  @type concat
  timeout_label @NORMAL
  flush_interval 5s
  key log
  stream_identity_key container_id
  multiline_start_regexp /\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}(.|,)\d{3}/
</filter>
<filter service.component*>
  @type concat
  timeout_label @NORMAL
  flush_interval 5s
  key log
  stream_identity_key container_id
  multiline_start_regexp /\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}(.|,)\d{3}/
</filter>
<filter service.nginx*>
  @type parser
  format json
  reserve_data true
  key_name log
</filter>
<match service.*>
   @type relabel
   @label @NORMAL
</match>
<label @NORMAL>
  <filter service.provider*>
    @type parser
    key_name log
    reserve_data true
    <parse>
       @type regexp
       expression /^(?<time>\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}.\d{3})\s{1,2}(?<level>[^\s]+) (?<pid>\S+) --- \[(?<thread>.*)\] /
    </parse>
  </filter>
  <filter service.component*>
    @type parser
    key_name log
    reserve_data true
    <parse>
       @type regexp
       expression /^(?<time>\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}.\d{3})\s{1,2}(?<level>[^\s]+) (?<pid>\S+) --- \[(?<thread>.*)\] /
    </parse>
  </filter>
  <match service.*>
    @type forward
    send_timeout 60s
    recover_wait 10s
    hard_timeout 60s
    buffer_type memory
    flush_interval 1s
    buffer_chunk_limit 256M
    buffer_queue_limit 128
    num_threads 8
    <server>
        name server-efk
        host 172.19.115.18
        port 24225
    weight 60
    </server>
    <server>
    name server-selenium
    host 172.19.115.29
    port 24225
    weight 60
    </server>
    <server>
    name server-mqtt
    host 172.19.115.20
    port 24225
    weight 60
    </server>
  </match>
</label>
```

#### server配置：

```
# fluentd/etc/fluent.conf
<source>
  @type forward
  port 24225
  bind 0.0.0.0
</source>
<filter service.nginx>
  @type parser
  format json
  reserve_data true
  key_name log
</filter>
<match service.*>
  @type forest
  subtype copy
  remove_prefix service
    <template>
      <store>
        @type elasticsearch
        host 172.19.115.18
        port 9200
        logstash_format true
        logstash_prefix fluentd
        logstash_dateformat %Y%m%d
        include_tag_key true
        type_name java_log
        tag_key @log_name
        buffer_type memory
        flush_interval 5s
        max_retry_wait 30
        disable_retry_limit
        buffer_chunk_limit 256M
        buffer_queue_limit 128
        num_threads 8
        reconnect_on_error true
        resurrect_after 5s
      </store>
        # <store>
        #       @type file
    #   path /var/log/fluent/${tag}.*.log
    #   compress gzip
    #   time_slice_format %Y%m%d
    #   time_slice_wait 10m
    #   append true
    #   message_key log
    #   format single_value
    # </store>
   </template>
</match>
```

#### Dockerfile配置
```
FROM fluent/fluentd:latest
 
USER root
 
VOLUME ["/var/log/fluent", "/fluentd/etc/"]
 
RUN apk add --no-cache --update --virtual .build-deps \
        sudo build-base ruby-dev \
 && sudo gem install fluent-plugin-elasticsearch \
 && sudo gem install fluent-plugin-record-reformer \
 && sudo gem install fluent-plugin-forest \
 && sudo gem sources --clear-all \
 && apk del .build-deps \
 && rm -rf /tmp/* /var/tmp/* /usr/lib/ruby/gems/*/cache/*.gem
 
COPY fluent.conf /fluentd/etc/
 
RUN echo 'http://mirrors.aliyun.com/alpine/v3.6/community/'>/etc/apk/repositories && echo 'http://mirrors.aliyun.com/alpine/v3.6/main/'>>/etc/apk/repositories && apk add --no-cache tzdata
ENV TZ Asia/Shanghai
RUN /bin/cp /usr/share/zoneinfo/${TZ} /etc/localtime && echo '${TZ}' >/etc/timezone
 
#RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
 
# 多行匹配插件（client）
RUN ["gem", "install", "fluent-plugin-concat", "-no-rdoc", "-no-ri", "--version", "2.1.0"]
# elasticsearch插件（server）
# RUN ["gem","install","fluent-plugin-elasticsearch","-no-rdoc","-no-ri","--version","1.9.2"]
```