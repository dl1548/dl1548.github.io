---
title: elk部署
date: 2017-10-14 14:29:03
tags: elk
categories:
    - 运维工具
    - elk
copyright: true
---
<img src="/images/elk.png"  alt="elk" align=center />
ELK并不是单个软件，它是一套解决方案。
[ELK](https://www.elastic.co/cn/)由Elasticsearch、Logstash和Kibana三部分组件组成
(封面图有个歧义,kinaba是从es中取数据的,为了方便理解传值,和logstash进行了连接.)
<!--more-->
___
>Elasticsearch是个开源分布式搜索引擎
Logstash开源的工具，它可以对你的日志进行收集、分析，并将其存储供以后使用
kibana 开源和免费的工具，它可以为 Logstash 和 ElasticSearch 提供的日志分析友好的 Web 界面

**默认端口号：**
- elasticsearch：9200 9300
- logstash     : 9301
- kinaba       : 5601

*流程图*

![基本流程图.png](http://upload-images.jianshu.io/upload_images/2511748-5496327305930aee.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

[下载相关软件包](https://www.elastic.co/cn/downloads)
本次实验我使用的包版本如下：elasticsearch-5.6.2.tar.gz，kibana-5.6.2-linux-x86_64.tar.gz ，logstash-5.6.2.tar.gz，注意安装包版本于java版本的兼容行，这里使用的是java1.8+，centos7。
### logstash
#### 安装logstash
```bash
#安装java，logstash运行依赖java，直接yum安装
yum -y install java-1.8.0
java -version
#显示如下，安装成功
openjdk version "1.8.0_144"
OpenJDK Runtime Environment (build 1.8.0_144-b01)
OpenJDK 64-Bit Server VM (build 25.144-b01, mixed mode)

#解压logstash包
tar -zxvf logstash-5.6.2.tar.gz -C /usr/local/

#配置环境变量，方便敲命令
echo "export PATH=\$PATH:/usr/local/logstash-5.6.2/bin" > /etc/profile.d/logstash.sh
. /etc/profile
```
#### 运行logstash
```json
logstash -e 'input{stdin{}}output{stdout{codec=>rubydebug}}'
#等待数秒后，随便输入字符串，将会得到如下json格式返回

test
{
      "@version" => "1",
          "host" => "0.0.0.0",
    "@timestamp" => 2017-10-10T05:34:17.427Z,
       "message" => "test"
}

#常用参数解释
-e :指定logstash的配置信息，可以用于快速测试;
-f :指定logstash的配置文件；可以用于生产环境;
```
#### 以配置文件运行logstash
```
[root@master config]# cat simple.conf 
input { stdin {} }
output {
   stdout { codec=> rubydebug }
}
logstash -f simple.conf
```
#### 输出到redis
redis要提前准备
```
# cat logstash_to_redis.conf
input { stdin { } }
output {
    stdout { codec => rubydebug }
    redis {
        host => 'REDIS IP '
        data_type => 'list'
        key => 'logstash:redis'
    }
}
```

##### 安装redis
[下载redis](https://redis.io/download)
[学习地址](http://www.runoob.com/redis/redis-tutorial.html)
```
tar -zxvf redis-4.0.2.tar.gz -C /usr/local/
cd  redis-4.0.2
make
#如果要执行make test 那么需要安装tcl
#yum -y install tcl
make install

##如若报错
zmalloc.h:50:31: error: jemalloc/jemalloc.h: No such file or directory
zmalloc.h:55:2: error: #error "Newer version of jemalloc required"
make[1]: *** [adlist.o] Error 1
make[1]: Leaving directory `/data0/src/redis-version/src'
make: *** [all] Error 2
#解决办法是：
make MALLOC=libc
```
##### 运行redis
```
#修改配置
/usr/local/redis-version/redis.conf
#修改 
daemonize 为 yes #设置后台启动
bind 为 0.0.0.0 #设置允许远程的地址
cd src
redis-server ../redis.conf

#查看进行(默认为6379端口)
ps -aux | grep redis 

#通过客户端登录
/usr/local/redis-version/src/redis-cli 
127.0.0.1:6379> ping
PONG
测试redis正常
```
#### 输入到redis
```
[root@master config]# cat redis.conf 
input { stdin { } }
output {
    stdout { codec => rubydebug }
    redis {
        host => '10.1.27.24'
        data_type => 'list'
        key => 'logstash:redis'
    }
}
#运行 logstash -f redis.conf
输入test，返回如下
{
      "@version" => "1",
          "host" => "0.0.0.0",
    "@timestamp" => 2017-10-10T06:25:42.830Z,
       "message" => "test"
}
去redis查看
[root@slave redis-4.0.2]# ./src/redis-cli monitor #这个要在logstash前打开
OK
1507616746.341355 [0 10.1.27.23:59337] "rpush" "logstash:redis" "{\"@version\":\"1\",\"host\":\"0.0.0.0\",\"@timestamp\":\"2017-10-10T06:25:42.830Z\",\"message\":\"test\"}"

```

#### 后台运行
```
nohup  logstash -f /usr/local/logstash-version/config/conf.d/filename.conf  &>/dev/null &
```
### elasticsearch
#### 安装Elasticsearch
默认不能用root用户启动
```
#添加用户和组
groupadd es
useradd es -g es -p password
chown -R es:es /usr/local/elasticsearch-5.6.2
#  启动ES（3种方式）
su - es
/usr/local/elasticsearch-5.6.2/bin/elasticsearch
/usr/local/elasticsearch-5.6.2/bin/elasticsearch -d #守护进程方式启动
nohup /usr/local/elasticsearch-5.6.2/bin/elasticsearch > /var/log/es.log 2>&1 & #推荐
 ```

##### es报错解决方案

 ```
ERROR: [2] bootstrap checks failed
[1]: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65536]
[2]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]

【1】无法创建文件，用户最大可创建文件数太小
#切换到root用户
vim /etc/security/limits.conf
#添加以下内容
* soft nofile 65536
* hard nofile 131072
* soft nproc 2048
* hard nproc 4096
【2】虚拟内存太小
#切换到root
vi /etc/sysctl.conf
#添加内容：
vm.max_map_count=655360
#运行命令生效
sysctl -p
```

#### 检查es启动状态,并访问
```json
[root@master ~]# netstat -tnlp |grep java
tcp6       0      0 :::9200                 :::*                    LISTEN      14610/java          
tcp6       0      0 :::9300                 :::*                    LISTEN      14610/java


#http://IPADDRESS:9200/

{
  "name" : "-fiwSlM",
  "cluster_name" : "elasticsearch",  #默认的集群名
  "cluster_uuid" : "biIeKipDSyKCRpS84bb0Ng",
  "version" : {
    "number" : "5.6.2",
    "build_hash" : "57e20f3",
    "build_date" : "2017-09-23T13:16:45.703Z",
    "build_snapshot" : false,
    "lucene_version" : "6.6.1"
  },
  "tagline" : "You Know, for Search"
}

此时目录下会多个data文件夹，默认的数据目录
```

#### elasticsearch配置文件详解

[此链接为出处](http://www.cnblogs.com/zlslch/p/6419948.html)

```bash
elasticsearch-.yml（中文配置详解）

# ======================== Elasticsearch Configuration =========================
#
# NOTE: Elasticsearch comes with reasonable defaults for most settings.
# Before you set out to tweak and tune the configuration, make sure you
# understand what are you trying to accomplish and the consequences.
#
# The primary way of configuring a node is via this file. This template lists
# the most important settings you may want to configure for a production cluster.
#
# Please see the documentation for further information on configuration options:
# <http://www.elastic.co/guide/en/elasticsearch/reference/current/setup-configuration.html>
#
# ---------------------------------- Cluster -----------------------------------
#
# Use a descriptive name for your cluster:
# 集群名称，默认是elasticsearch
# cluster.name: my-application
#
# ------------------------------------ Node ------------------------------------
#
# Use a descriptive name for the node:
# 节点名称，默认从elasticsearch-2.4.3/lib/elasticsearch-2.4.3.jar!config/names.txt中随机选择一个名称
# node.name: node-1
#
# Add custom attributes to the node:
# 
# node.rack: r1
#
# ----------------------------------- Paths ------------------------------------
#
# Path to directory where to store the data (separate multiple locations by comma):
# 可以指定es的数据存储目录，默认存储在es_home/data目录下
# path.data: /path/to/data
#
# Path to log files:
# 可以指定es的日志存储目录，默认存储在es_home/logs目录下
# path.logs: /path/to/logs
#
# ----------------------------------- Memory -----------------------------------
#
# Lock the memory on startup:
# 锁定物理内存地址，防止elasticsearch内存被交换出去,也就是避免es使用swap交换分区
# bootstrap.memory_lock: true
#
#
#
# 确保ES_HEAP_SIZE参数设置为系统可用内存的一半左右
# Make sure that the `ES_HEAP_SIZE` environment variable is set to about half the memory
# available on the system and that the owner of the process is allowed to use this limit.
# 
# 当系统进行内存交换的时候，es的性能很差
# Elasticsearch performs poorly when the system is swapping the memory.
#
# ---------------------------------- Network -----------------------------------
#
#
# 为es设置ip绑定，默认是127.0.0.1，也就是默认只能通过127.0.0.1 或者localhost才能访问
# es1.x版本默认绑定的是0.0.0.0 所以不需要配置，但是es2.x版本默认绑定的是127.0.0.1，需要配置
# Set the bind address to a specific IP (IPv4 or IPv6):
#
# network.host: 192.168.0.1
#
#
# 为es设置自定义端口，默认是9200
# 注意：在同一个服务器中启动多个es节点的话，默认监听的端口号会自动加1：例如：9200，9201，9202...
# Set a custom port for HTTP:
#
# http.port: 9200
#
# For more information, see the documentation at:
# <http://www.elastic.co/guide/en/elasticsearch/reference/current/modules-network.html>
#
# --------------------------------- Discovery ----------------------------------
#
# 当启动新节点时，通过这个ip列表进行节点发现，组建集群
# 默认节点列表：
# 127.0.0.1，表示ipv4的回环地址。
#   [::1]，表示ipv6的回环地址
#
# 在es1.x中默认使用的是组播(multicast)协议，默认会自动发现同一网段的es节点组建集群，
# 在es2.x中默认使用的是单播(unicast)协议，想要组建集群的话就需要在这指定要发现的节点信息了。
# 注意：如果是发现其他服务器中的es服务，可以不指定端口[默认9300]，如果是发现同一个服务器中的es服务，就需要指定端口了。
# Pass an initial list of hosts to perform discovery when new node is started:
# 
# The default list of hosts is ["127.0.0.1", "[::1]"]
#
# discovery.zen.ping.unicast.hosts: ["host1", "host2"]
#
# 通过配置这个参数来防止集群脑裂现象 (集群总节点数量/2)+1
# Prevent the "split brain" by configuring the majority of nodes (total number of nodes / 2 + 1):
#
# discovery.zen.minimum_master_nodes: 3
#
# For more information, see the documentation at:
# <http://www.elastic.co/guide/en/elasticsearch/reference/current/modules-discovery.html>
#
# ---------------------------------- Gateway -----------------------------------
#
# Block initial recovery after a full cluster restart until N nodes are started:
# 一个集群中的N个节点启动后,才允许进行数据恢复处理，默认是1
# gateway.recover_after_nodes: 3
#
# For more information, see the documentation at:
# <http://www.elastic.co/guide/en/elasticsearch/reference/current/modules-gateway.html>
#
# ---------------------------------- Various -----------------------------------
# 在一台服务器上禁止启动多个es服务
# Disable starting multiple nodes on a single system:
#
# node.max_local_storage_nodes: 1
#
# 设置是否可以通过正则或者_all删除或者关闭索引库，默认true表示必须需要显式指定索引库名称
# 生产环境建议设置为true，删除索引库的时候必须显式指定，否则可能会误删索引库中的索引库。
# Require explicit names when deleting indices:
#
# action.destructive_requires_name: true
 集群名称，默认是elasticsearch

　　输入，http://192.168.80.200:9200/
```

#### logstash和elasticsearch结合

>结合有两种方法，直接结合，通过redis结合

*直接结合*
```json
#编写logstash配置文件
# cat logstash-elasticsearch.conf 
input { stdin {} }
output {
    elasticsearch { hosts => "IPADDRESS" }    #此处用hosts
    stdout { codec=> rubydebug }
｝

#通过配置文件启动logstash即可

```
*与redis结合*

```json
# cat  config/logstash-redis.conf
input {
    redis {
        host => '10.1.27.24' #redis地址
        data_type => 'list'
        key => 'logstash:redis'
    }
}
output {
    elasticsearch {
        hosts => '10.1.27.23'  #hosts
        codec => 'json'
    }
}

#http://10.1.27.23:9200/_search?pretty
#浏览器访问es地址，会发现redis的内容也会显示出来。
{
  "took" : 7,
  "timed_out" : false,
  "_shards" : {
    "total" : 10,
    "successful" : 10,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 7,
    "max_score" : 1.0,
    "hits" : [
   {
        "_index" : "logstash-2017.10.11",
        "_type" : "logs",
        "_id" : "AV8JbX8-FBpUgNjayMTu",
        "_score" : 1.0,
        "_source" : {
          "@version" : "1",
          "host" : "0.0.0.0",
          "@timestamp" : "2017-10-11T03:14:51.802Z",
          "message" : "ceshi"   #此消息为直接结合的
        }
      },
      {
        "_index" : "logstash-2017.10.11",
        "_type" : "redis-input",
        "_id" : "AV8KRKarFBpUgNjayMT2",
        "_score" : 1.0,
        "_source" : {
          "@version" : "1",
          "host" : "0.0.0.0",
          "@timestamp" : "2017-10-11T07:09:29.575Z",
          "message" : "redisredisredisredisredisredis", #此内容为redis
          "type" : "redis-input"  #这里可以看出，类型为配置里定义的类型名
        }
      },
     }
    ]
  }
}

```
### Kinaba

#### 安装kibana

```
# tar zxf kibana--VERSION -C /usr/local
```

#### 配置

```
vim /usr/local/kibana-VERSION /config/kibana.yml
#修改如下
server.host: "0.0.0.0"
elasticsearch_url: "http://es的IP地址:9200"

#访问kibana地址
http://10.1.27.23:5601
```

#### 启动

```bash
/usr/local/kibana-VERSION-linux-x64/bin/kibana
```
点击Create 创建一个索引

![image.png](http://upload-images.jianshu.io/upload_images/2511748-508cca05a115d518.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*然后点击左侧discovery 发现es的日志，选择时间段即可查看日志*


![image.png](http://upload-images.jianshu.io/upload_images/2511748-4d01152e4e9a2bfb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


ELK部署，完。
***
访问出现

![image.png](http://upload-images.jianshu.io/upload_images/2511748-4d9b7d02de047f0e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

换个浏览器访问

___
