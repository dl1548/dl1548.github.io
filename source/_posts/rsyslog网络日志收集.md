---
title: rsyslog网络日志收集
date: 2017-10-18     13:07:16
tags: 
    - rsyslog
    - elk
categories:
    - 运维工具
    - elk
copyright: true
---

网络设备的日志较分散,集中管理一下,看日志就方便多了
<!--more-->
___

> 软件rsyslog
centos7 默认安装

#### 编辑参数
```
vim /etc/sysconfig/rsyslog
修改如下
SYSLOGD_OPTIONS="-m 0 -r"
-m 0表示不在日志中添加时间戳消息，-r 表示允许接收外来日
```

#### 修改配置
```
vim /etc/rsyslog.conf
修改如下
# Provides UDP syslog reception
$ModLoad imudp #去掉注释
$UDPServerRun 514#去掉注释

# Provides TCP syslog reception
$ModLoad imtcp#去掉注释
$InputTCPServerRun 514#去掉注释


#### GLOBAL DIRECTIVES ####
#添加日志接收模板：
$template IpTemplate,"/net-log/%FROMHOST-IP%_%$YEAR%-%$MONTH%-%$DAY%.log"
:FROMHOST-IP, !isequal, "127.0.0.1" ?IpTemplate #本地的不保存
#*.* ?IpTemplate   #表示所有的日志都通过模板进行保存
& ~

###
$template IpTemplate 指令让rsyslog进程把日志文件写入到/net-log下指定的log文件中，指定的log文件使用客户端的IP地址命名。
& ~表示的是重定向规则，告知rsyslog进程无需进一步处理日志消息，无需写入本地日志文件。

#如果要把不同服务器发送过来的日志保存到不同的文件, 可以这样操作:  
#精确分类
:fromhost-ip, isequal, “1.1.1.1″ /var/log/a.log  
:FROMHOST-IP, isequal, “2.2.2.2″ /var/log/2.log  
#根据网段分类
:FROMHOST-IP, startswith, “3.3.3.” /var/log/c.log  
:FROMHOST-IP, startswith, “4.4.4.” /var/log/d.log  

```
#### 重启服务
```
systemctl restart rsyslog 
#查看进程状态。
netstat -tulpn | grep rsyslog 
```
#### 配置交换机
```
全局模式中：
 logging <ip-address|ipv6-address|hostname>
 logging on
```
