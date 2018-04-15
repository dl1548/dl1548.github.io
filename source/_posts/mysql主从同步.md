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

#最大连接数,按需更改
max_connections=10000

character-set-server=utf8

#每个bin-log最大大小，当此大小等于500M时会自动生成一个新的日志文件。一条记录不会写在2个日志文件中，所以有时日志文件会超过此大小。
max_binlog_size = 500M

#日志缓存大小
binlog_cache_size = 128K

#当Slave从Master数据库读取日志时更新新写入日志中，如果只启动log-bin而没有启动log-slave-updates则Slave只记录针对自己数据库操作的更新。
log-slave-updates

#设置bin-log日志文件保存的天数，此参数mysql5.0以下版本不支持。
expire_logs_days=30

#设置bin-log日志文件格式为：MIXED，可以防止主键重复。
binlog_format="MIXED"

#要同步的数据库,可指定多个,需复制此参数
#binlog-do-db=zabbix 
#binlog-do-db=it

#要同步多个数据库，就在slave多加几个replicate-db-db=数据库名

#binlog_ignore_db=mysql   #忽略的数据库
#binlog_ignore_db=information_schema
#binlog_ignore_db=performance_schema

#auto-increment-increment = 10
#auto-increment-offset=1
#这俩设置标示这台服务器上插入的第一个id就是 1， 第二行的id就是 11了,主主备份可以避免重复
```

复制的几种模式解释
```
– 基于SQL语句的复制(statement-based replication, SBR)，
– 基于行的复制(row-based replication, RBR)，
– 混合模式复制(mixed-based replication, MBR)。
相应地，binlog的格式也有三种：STATEMENT，ROW，MIXED。 MBR 模式中，SBR 模式是默认的。
在运行时可以动态改动 binlog的格式，除了以下几种情况：
1.存储流程或者触发器中间
2.启用了NDB
3.当前会话试用 RBR 模式，并且已打开了临时表
如果binlog采用了 MIXED 模式，那么在以下几种情况下会自动将binlog的模式由 SBR 模式改成 RBR 模式
```

#### mysql-bin
```
systemctl restart mariadb.service

#锁表,禁止写操作.
FLUSH TABLES WITH READ LOCK;

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

从服务如果是克隆的主服务器,需要注意 data/mysql.auto.cnf删除,并重启服务器.否则无法完成克隆
因为两台服务器mysql的server-uuid 是相同的.

#### 修改配置
`/etc/my.cnf`

```
server-id=2  #唯一标示
#只读,建议开启,防止slave写了数据导致主从出现问题.
read_only=on
#replicate-do-db=it


#下面这些主要是做主主配置
#log-bin=/var/lib/mysql/mysql-bin #二进制文件保存位置
#log_slave_updates = 1 #slave将复制事件写进自己的二进制日志

slave-skip-errors=1007,1008,1053,1062,1213,1158,1159

#error code代表的错误如下：
    1007：数据库已存在，创建数据库失败
    1008：数据库不存在，删除数据库失败
    1050：数据表已存在，创建数据表失败
    1051：数据表不存在，删除数据表失败
    1053：复制过程中主服务器宕机
    1054：字段不存在，或程序文件跟数据库有冲突
    1060：字段重复，导致无法插入
    1061：重复键名
    1062：主键冲突 Duplicate entry '%s' for key %d
    1068：定义了多个主键
    1094：位置线程ID
    1146：数据表缺失，请恢复数据库
    1158：网络错误，出现读错误，请检查网络连接状况
    1159：网络错误，读超时，请检查网络连接状况
    1213: 死锁
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



#记得解锁主库
UNLOCK TABLES;
```
然后主服务器又任何操作，从服务器就同步过来了。


### 其他
关于主从备份优缺点机制等,[推荐阅读](https://blog.csdn.net/www63912/article/details/53443490)

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
