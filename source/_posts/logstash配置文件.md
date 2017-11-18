---
title: logstash配置文件
date: 2017-10-22 16:09:02
tags:
    - logstash
    - elk
categories:
    - 运维工具
    - elk
copyright: true
---

logstash以文件启动--配置文件编写
<!--more-->

___

> 备忘

#### redis配置
主要分三大块,input,filter,output,三块为同一级
``` sh
#这里的key值,来源filebeat采集时output的定义.
input {
    redis {
        host => '10.1.27.31'
        data_type => 'list'
        key => 'network'
    }
    redis {
        host => '10.1.27.31'
        data_type => 'list'
        key => 'nginx-202'
    }
}

#这里的type来源于filebeat采集时input中的定义(document_type)

filter {
    if [type] == '93.218-zabbix' {
        grok{
            match => {
                "message" => "%{IP:clientIP}\|%{IP:serverIP}\|%{DATA:req_addr}\|%{DATA:req_url}\|%{DATA:req_status}\|%{DATA:req_browder}\|%{DATA:req_duration}\|%{DATA:req_user}"
           }
        }
    }
#这里的match要根据日志格式进行匹配,这个主要是Nginx的日志
#Nginx中对日志进行了自定的格式化.nginx参考格式
#'$remote_addr|$http_host|$request|$http_referer|$status|$http_user_agent|$request_time|$remote_user';


    if [type] == '93.218-it' {
        grok{
            match => {
                "message" => "%{IP:clientIP}\|%{IP:serverIP}\|%{DATA:req_addr}\|%{DATA:req_status}\|%{DATA:req_browder}\|%{DATA:req_duration}\|%{DATA:req_user}"
           }
        }
    }
#同上,参考格式
#'$remote_addr|$http_host|$request|$status|$http_user_agent|$request_time|$remote_user';
}


#通过type判断,定义不同的index,kibana就能建立不同的map
output {
    if [type] == '93.218-zabbix' {
        elasticsearch {
            hosts => '10.1.27.31'
            codec => 'json'
            index => '93.218-zabbix-%{+YYYY.MM.dd}'
        }
    }
    if [type] == '93.218-it' {
        elasticsearch {
            hosts => '10.1.27.31'
            codec => 'json'
            index => '93.218-it-%{+YYYY.MM.dd}'
        }
    }
}
```
过滤做好之后,可根据match的字段进行匹配,看自己想看的结果,以及后续的画图等

![kibana.jpg](http://upload-images.jianshu.io/upload_images/2511748-e0e646aa68140545.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

___
