---
title: filebeat客户端
date: 2017-10-16 13:48:29
tags:
    - elk
    - filebeat
categories:
    - 运维工具
    - elk
copyright: true
---



Filebeat是一个日志文件托运工具，在你的服务器上filebeat后，它会监控日志目录或者指定的日志文件，追踪读取这些文件（追踪文件的变化，不停的读)并且转发这些信息到elasticsearch/logstarsh/redis/kafka/等等中
<!--more-->
___
version : 5.6

日志的搜集,选择filebeat还是logstash 根据个人情况,filebeat相对来说更轻便
#### 安装filebeat
[官网说明](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-installation.html)
前提ELK已部署完毕.个人实验是输出到redis
```
DEB:
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-5.6.3-amd64.deb
sudo dpkg -i filebeat-5.6.3-amd64.deb

RPM:
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-5.6.3-x86_64.rpm
sudo rpm -vi filebeat-5.6.3-x86_64.rpm
```
#### [filebeat.yml配置](https://www.elastic.co/guide/en/beats/filebeat/current/configuring-howto-filebeat.html)
默认路径在 /etc/filebeat/filebeat.yml
模板中有各种说明,输出到各种环境的配置,
```
#以redis为例
- input_type: log
  # Paths that should be crawled and fetched. Glob based paths.
  paths:
    - /usr/local/nginx-1.12.1/logs/*.log
  # - /usr/local/nginx-1.12.1/logs/*/*.log #子目录的log也会采集



多日志,不同源采集:
filebeat.prospectors:

- input_type: log
  paths:
    - /net-log/10.4.32.1*.log
  fields:
    log_source: 10.4.32.1
  fields_under_root: true
  document_type: 10.4.32.1

- input_type: log
  paths:
    - /net-log/127.0.0.1*.log
  fields:
    log_source: 127.0.0.1
  fields_under_root: true
  document_type: 127.0.0.1

#fields_under_root: true 
#此配置使field字典的数据以顶级格式出现,
#如果字段已存在,则覆盖.
#document_type: 127.0.0.1,定义了type值,此值可在logstash以文件启动的配置中,if['type'] == 或者 if value in ['type'] 进行判断归类.以为filter


    -
       paths:
          - /home/b/*.log
       fields:
         log_source: b

#--------------------------------redis---------------------
output.redis:
    hosts: ["10.1.27.24"]   #必须
   #password: "your pwd"
    db: 0
    key: "nginx"   #必须
    timeout: 10


##key值的作用
key值会传递给redis/es/logstash等,所以key的定义可以起到一定日志分类的作用.
logstash可以根据key值决定取出哪些值.存到es或其他存储中
```
##### 参考(logstash文件启动配置)
```
logstash的简单配置
input {
    redis {
        host => '10.1.27.24'
        data_type => 'list'
        key => 'nginx'
    }
}
filter{
    #if [] ...
    grok{
        ...
    }
}
output {
#这里可以加判断,if 'a' in [type]{.....index => 'a-...'}
#这样可以对日志更细致的划分,索引建立的更细致
    elasticsearch {
        hosts => '10.1.27.23'
        codec => 'json'
        index => 'nginx-%{+YYYY.MM.dd}'
    }

```
[输出到各环境的配置kafka,redis,es,logstash等](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-configuration-details.html)

#### 启动filebeat
```
rpm安装：

sudo /etc/init.d/filebeat start
```

___
