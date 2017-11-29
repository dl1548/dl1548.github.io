---
title: zabbix监控MySQL性能
date: 2017-05-01 20:01:51
tags:
    - zabbix
    - mysql
categories: zabbix
copyright: true
---

套用自带模板监控mysql性能

<!--more-->

#### 安装agent 
[查看此链接](http://dl1548.site/2017/04/18/zabbix-agent%E5%AE%89%E8%A3%85/)

####  DB权限
1. 在客户端的mysql里添加权限,使用zabbix账号连接本地的mysql
```
mysql> grant all on *.* to zabbix@'localhost' identified by "zabbix”;
mysql> flush privileges;
```
#### 编辑 my.cnf
`/etc/zabbix/etc/my.cnf` (需新建)
```
#zabbix Agent
[mysql]
host=localhost
user=zabbix
password=zabbix
socket=/var/lib/mysql/mysql.sock  #具体根据个人情况
[mysqladmin]
host=localhost
user=zabbix
password=zabbix
socket=/var/lib/mysql/mysql.sock
```
#### 修改agentd
`/etc/zabbix/zabbix_agentd.d/userparameter_mysql.conf`
`HOME=/var/lib/zabbix ` 修改为 ` HOME=/etc/zabbix/etc/ `

其他
`zabbix_agentd.conf`配置文件中
`Include`选择要包含 `zabbix_agentd.d`

`service zabbix-agent restart`

然后去web端口,添加主机 套用内置mysql模板就可以了.


#### 关于userparameter_mysql.conf
 还有一种配置

#####  新建脚本
`vim /etc/zabbix/chk_mysql.sh`
```
#!/bin/bash
# 用户名
MYSQL_USER='zabbix'
# 密码
MYSQL_PWD='zabbix'
# 主机地址/IP
MYSQL_HOST='127.0.0.1'
# 端口
MYSQL_PORT='3306'
# 数据连接
MYSQL_CONN="/usr/bin/mysqladmin -u${MYSQL_USER} -p${MYSQL_PWD} -h${MYSQL_HOST} -P${MYSQL_PORT}"
# 参数是否正确
if [ $# -ne "1" ];then 
    echo "arg error!" 
fi 
# 获取数据
case $1 in 
    Uptime) 
        result=`${MYSQL_CONN} status|cut -f2 -d":"|cut -f1 -d"T"` 
        echo $result 
        ;; 
    Com_update) 
        result=`${MYSQL_CONN} extended-status |grep -w "Com_update"|cut -d"|" -f3` 
        echo $result 
        ;; 
    Slow_queries) 
        result=`${MYSQL_CONN} status |cut -f5 -d":"|cut -f1 -d"O"` 
        echo $result 
        ;; 
    Com_select) 
        result=`${MYSQL_CONN} extended-status |grep -w "Com_select"|cut -d"|" -f3` 
        echo $result 
                ;; 
    Com_rollback) 
        result=`${MYSQL_CONN} extended-status |grep -w "Com_rollback"|cut -d"|" -f3` 
                echo $result 
                ;; 
    Questions) 
        result=`${MYSQL_CONN} status|cut -f4 -d":"|cut -f1 -d"S"` 
                echo $result 
                ;; 
    Com_insert) 
        result=`${MYSQL_CONN} extended-status |grep -w "Com_insert"|cut -d"|" -f3` 
                echo $result 
                ;; 
    Com_delete) 
        result=`${MYSQL_CONN} extended-status |grep -w "Com_delete"|cut -d"|" -f3` 
                echo $result 
                ;; 
    Com_commit) 
        result=`${MYSQL_CONN} extended-status |grep -w "Com_commit"|cut -d"|" -f3` 
                echo $result 
                ;; 
    Bytes_sent) 
        result=`${MYSQL_CONN} extended-status |grep -w "Bytes_sent" |cut -d"|" -f3` 
                echo $result 
                ;; 
    Bytes_received) 
        result=`${MYSQL_CONN} extended-status |grep -w "Bytes_received" |cut -d"|" -f3` 
                echo $result 
                ;; 
    Com_begin) 
        result=`${MYSQL_CONN} extended-status |grep -w "Com_begin"|cut -d"|" -f3` 
                echo $result 
                ;; 

        *) 
        echo "Usage:$0(Uptime|Com_update|Slow_queries|Com_select|Com_rollback|Questions|Com_insert|Com_delete|Com_commit|Bytes_sent|Bytes_received|Com_begin)" 
        ;; 
esac
```

##### 修改配置
```
vim /etc/zabbix/zabbix_agentd.d/userparameter_mysql.conf

#UserParameter=mysql.status[*],echo "show global status where Variable_name='$1';" | HOME=/var/lib/zabbix mysql -N | awk '{print $$2}'
UserParameter=mysql.status[*],/etc/zabbix/chk_mysql.sh $1

#UserParameter=mysql.size[*],bash -c 'echo "select sum($(case "$3" in both|"") echo "data_length+index_length";; data|index) echo "$3_length";; free) echo "data_free";; esac)) from information_schema.tables$([[ "$1" = "all" || ! "$1" ]] || echo " where table_schema=\"$1\"")$([[ "$2" = "all" || ! "$2" ]] || echo "and table_name=\"$2\"");" | HOME=/var/lib/zabbix mysql -N'

#UserParameter=mysql.ping,HOME=/var/lib/zabbix mysqladmin ping | grep -c alive
UserParameter=mysql.ping,mysqladmin -uzabbix -pzabbix -P3306 -h127.0.0.1  ping | grep -c alive
UserParameter=mysql.version,mysql -V

```

___