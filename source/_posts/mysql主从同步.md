---
title: mysql主从同步
date: 2017-09-04 14:49:47
tags: mysql
categories: database
copyright: true
---

数据库,还是做个备份吧...
前提：数据库版本一致，初始化数据一致（数据库一致）
<!--more-->
### 主服务器
#### 新建同步用户
```
GRANT REPLICATION SLAVE ON *.* TO backup@'%' IDENTIFIED BY 'backup';
#新建backupDB用户，%代表任何ip都可连接， 密码backup
```
#### 修改配置
`/etc/my.cnf`

```
[mysqld]
log-bin=/var/lib/mysql/mysql-bin#二进制文件保存位置,这些文件是mysql的事务日志记录。
server-id=1  #唯一标示
binlog-do-db=zabbix # /要同步的数据库,可指定多个
binlog-do-db=it

#要同步多个数据库，就在slave多加几个replicate-db-db=数据库名

binlog_ignore_db=mysql   #忽略的数据库
binlog_ignore_db=information_schema
binlog_ignore_db=performance_schema

#auto-increment-increment = 10
#auto-increment-offset=1
#这俩设置标示这台服务器上插入的第一个id就是 1， 第二行的id就是 11了,主主备份可以避免重复
```
#### mysql-bin
```
systemctl restart mariadb.service

MariaDB [(none)]> show master status;
+------------------+----------+--------------+---------------------------------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB                            |
+------------------+----------+--------------+---------------------------------------------+
| mysql-bin.000001 |      245 | it           | mysql,information_schema,performance_schema |
+------------------+----------+--------------+---------------------------------------------+
1 row in set (0.00 sec)
```

*这里要注意* `| mysql-bin.000001 |      245 ` 后面的从服务会用到
所以，此时，就不要对主数据库进行操作了，以免值变化。

### 从服务器
#### 修改配置
`/etc/my.cnf`

```
server-id=2  #唯一标示
replicate-do-db=it


#下面这些主要是做主主配置
#log-bin=/var/lib/mysql/mysql-bin #二进制文件保存位置
#log_slave_updates = 1 #slave将复制事件写进自己的二进制日志
```
#### 重启并进入数据库
```
systemctl restart mariadb.service

#这里用到了master数据库的状态值
change master to master_host='10.1.*.*(master的IP)',master_user='backup',master_password='backup',master_log_file='mysql-bin.000001',master_log_pos=245;

#开启slave
start slave
#查看状态

MariaDB [(none)]> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 10.1.。。。
                  Master_User: backup
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000001
          Read_Master_Log_Pos: 468
               Relay_Log_File: mariadb-relay-bin.000002
                Relay_Log_Pos: 529
        Relay_Master_Log_File: mysql-bin.000001
             Slave_IO_Running: Yes   #这里要是YES
            Slave_SQL_Running: Yes   #这里要是YES
              Replicate_Do_DB: it
略....
```
然后主服务器又任何操作，从服务器就同步过来了。


### 其他

#### 解除主从
两个办法
1. 彻底解除主从复制关系
- stop slave;
- reset slave; #或直接删除master.info和relay-log.info这两个文件
- 修改my.cnf删除主从相关配置参数

2. 让slave不随MySQL自动启动
修改my.cnf
在[mysqld]中增加 skip-slave-start 选项


#### mysqldump需注意

`mysqldump --master-data --single-transaction --user=username --password=password dbname> dumpfilename`

这样就可以保留 `file` 和 `position` 的信息，在新搭建一个slave的时候，还原完数据库
 file 和 position 的信息也随之更新，接着再start slave 就可以很迅速的完成增量同步！


 #### 链条式同步
 如果想实现 主-从（主）-从 这样的链条式结构，需要设置：
`log-slave-updates` #只有加上它，从前一台机器上同步过来的数据才能同步到下一台机器

二进制日志也是必须开启的：
`log-bin=/opt/mysql/binlogs/bin-log`
`log-bin-index=/opt/mysql/binlogs/bin-log.index`

还可以设置一个log保存周期：
`expire_logs_days=14`

___
