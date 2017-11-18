---
title: zabbix-server-is-not-running
date: 2017-03-30 11:50:57
tags:
    - zabbix
categories: zabbix
copyright: true
---

<img src="/images/zabbix-1.jpg" width = "70%" height = "50%" alt="shell" align=center />
### 前言
(图片来自网上)
zabbix server is not running: the infomation displayed may not be current
<!-- more -->

#### 看log 
( /tmp/zabbix_server.log)

`Connection to database 'xxx' failed: [1045] Access denied for user 'xxx'@'localhost' (using password: NO)`

#### 看端口
```
[root@localhost conf]# netstat -ntlp |grep zabbix
tcp        0      0 0.0.0.0:10050               0.0.0.0:*                   LISTEN      3672/zabbix_agentd
tcp        0      0 :::10050                    :::*                        LISTEN      3672/zabbix_agentd
```
~~我本机装了agent ,自己监控自己.~~
可以看到zabbix_server的端口10051根本就没起来,所以可能根本就不是帐号或者权限的问题.确定方向在拍错.

### 解决方法

1. 检查 zabbix_server.conf 里面 密码是否启动

2. 检查 mysql用户zabbix 是否能够正常登录

3. 检查数据库是否授权
` show grants for zabbix@'localhost';`
```
mysql> select user,host from mysql.user;
+--------+-----------+
| user   | host      |
+--------+-----------+
| root   | 127.0.0.1 |
| root   | localhost |
+--------+-----------+
```
创建,授权
`grant all privileges on zabbix.* to 'zabbix'@'%' identified by 'zabbix';`





要去检查下配置文件 zabbix_server.conf
```
DBPassword=zabbix   #注意这个配置文件 不需要加引号的!
```
-------

如果出现

### ERROR 1045
ERROR 1045 (28000): Access denied for user 'xxx'@'localhost' (using password: YES) 

首先照上 查看各权限,如果还不行 则
```
删除匿名用户.
mysql> use mysql 
mysql> delete from user where user=''; 
mysql> flush privileges; 
```


5.10  更新
今天监控突然启动不起来了.报错如出一辙,以为谁动了配置.赶紧上去看log
```
[root@localhost ~]# tailf /tmp/zabbix_server.log
 10151:20170510:083714.639 SSH2 support:              YES
 10151:20170510:083714.639 IPv6 support:              YES
 10151:20170510:083714.639 TLS support:                NO
 10151:20170510:083714.639 ******************************
 10151:20170510:083714.639 using configuration file: /usr/local/zabbix/etc/zabbix_server.conf
 10151:20170510:083714.646 current database version (mandatory/optional): 03020000/03020000
 10151:20170510:083714.646 required mandatory version: 03020000
 10151:20170510:083715.302 __mem_malloc: skipped 1 asked 244072 skip_min 35304 skip_max 35304
 10151:20170510:083715.302 [file:strpool.c,line:53] zbx_mem_realloc(): out of memory (requested 244072 bytes)
 10151:20170510:083715.302 [file:strpool.c,line:53] zbx_mem_realloc(): please increase CacheSize configuration parameter
```
显示CacheSize不足了.所以去配置文件调整下就好了.
