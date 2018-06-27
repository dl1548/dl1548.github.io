---
title: linux个人备忘手册
date: 1111-11-11 11:11:12
tags: -个人备忘手册
categories: linux
copyright: true
password: woshizhu
---

> 一些小命令的记录

<!--more-->

#### 按当前日期命名文件
```
创建备份Shell脚本:
输入/粘贴以下内容：
#!/bin/bash
mysqldump -uusername -ppassword DatabaseName > /home/backup/DatabaseName_$(date +%Y%m%d_%H%M%S).sql
对备份进行压缩：
#!/bin/bash
mysqldump -uusername -ppassword DatabaseName | gzip > /home/backup/DatabaseName_$(date +%Y%m%d_%H%M%S).sql.gz
```

#### 定时备份

```
#!/usr/bin/bash
cd /var/log/
tar -zcf /var/log/haproxy_log_bk/$(date +%Y%m%d_%H%M%S).tar.gz haproxy.log
echo "" > haproxy.log
```

#### -mtime
```
查找 并移动3天前的
find /net-log/ -mtime +3 -name "*.log" -exec mv {} /tmp/ \;
删除
find /net-log/ -mtime +3 -name "*.log" -exec rm -rf {} \;

命令可写到定时任务中,对长时间不读取文件进行删除
crontab -e
* * * * *  + 程序 +命令或脚本
#如果是bash命令 可直接写.
例:每天0点执行test.py
0 0 * * *  /usr/bin/python3   /script/test.py
```

#### ssh快捷登录
```
zili@Ubuntu:~$ cat ~/.ssh/config
ServerAliveInterval 60 #60S发送一次存活信息,以免断开
Host study
User root
Hostname 10.1.1.11
Port 21131
#默认为22 有修改则填写
#若有多个主机,直接复制修改即可
zili@Ubuntu:~$ ssh study
root@10.1.1.11's password:
```

#### mysql开启远程

`GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'youpassword' WITH GRANT OPTION;`


```
mysql> select user,host from mysql.user;
+--------+-----------+
| user   | host      |
+--------+-----------+
| root   | 127.0.0.1 |
| root   | localhost |
+--------+-----------+

mysql> show status like 'Threads%';
+-------------------+-------+
| Variable_name     | Value |
+-------------------+-------+
| Threads_cached    | 58    |
| Threads_connected | 57    |   ###这个数值指的是打开的连接数
| Threads_created   | 3676  |
| Threads_running   | 4     |   ###这个数值指的是激活的连接数，这个数值一般远低于connected数值
+-------------------+-------+

Threads_connected 跟show processlist结果相同，表示当前连接数。准确的来说，Threads_running是代表当前并发数

这是是查询数据库当前设置的最大连接数
mysql> show variables like '%max_connections%';
+-----------------+-------+
| Variable_name   | Value |
+-----------------+-------+
| max_connections | 1000  |
+-----------------+-------+

可以在/etc/my.cnf里面设置数据库的最大连接数
[mysqld]
max_connections = 10000
```

#### 开启snmp
`yum –y install net-snmp net-snmp-devel`
若要使用snmpwalk进行安装检测，则还需要
`yum –y install net-snmp-utils`

`vi /etc/snmp/snmpd.conf`
把`62`行中的`systemview`改为`mib2`
把`89行的`#`去掉。
然后在最后一行添加 `rwcommunity  ge.` 保存退出。
防火墙添加策略,重启服务即可
`snmpwalk -v 2c -c public localhost sysName.0`可做验证,默认社区号是`public`
若需要修改则`41`行中`public`换为指定字符串即可


#### openssl自签
Key是私用秘钥，通常是RSA算法
Csr是证书请求文件，用于申请证书。在制作csr文件时，必须使用自己的私钥来签署申，还可以设定一个密钥。
crt是CA认证后的证书文，签署人用自己的key给你签署凭证。
```
# 生成一个RSA密钥
openssl genrsa -des3 -out 33iq.key 1024

# 拷贝一个不需要输入密码的密钥文件
openssl rsa -in 33iq.key -out 33iq_nopass.key

# 生成一个证书请求
openssl req -new -key 33iq.key -out 33iq.csr

# 自己签发证书
openssl x509 -req -days 365 -in 33iq.csr -signkey 33iq.key -out 33iq.crt

```

#### pip源更换

pip国内的一些镜像
    - 阿里云 http://mirrors.aliyun.com/pypi/simple/
    - 中国科技大学 https://pypi.mirrors.ustc.edu.cn/simple/
    - 豆瓣(douban) http://pypi.douban.com/simple/
    - 清华大学 https://pypi.tuna.tsinghua.edu.cn/simple/
    - 中国科学技术大学 http://pypi.mirrors.ustc.edu.cn/simple/

临时使用：
可以在使用pip的时候在后面加上-i参数，指定pip源
`pip install scrapy -i https://pypi.tuna.tsinghua.edu.cn/simple`

永久修改：
    linux:
    修改 ~/.pip/pip.conf (没有就创建一个)，如下：
```
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
```


















___
