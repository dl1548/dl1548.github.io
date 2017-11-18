---
title: zabbix编译安装
date: 2017-03-28 10:46:27
tags: zabbix
categories: zabbix
copyright: true
---


<img src="/images/zabbix-logo.jpg" alt="salt-logo" align=center />

#### 前言

踩了不少坑,总结个文档.
<!--more-->
#### 安装基础环境(依赖)
平台在 centos 7+ （PHP 5.5）
```
yum -y install php*   
yum -y install mariadb*
yum install -y httpd-manual httpd  mod_ssl mod_perl mod_auth_mysql
yum install -y fping mysql mysql-server  mysql-connector-odbc mysql-devel libdbi-dbd-mysql libssh2 libxml2 libxml2-devel libssh2-devel unixODBC unixODBC-devel
yum install -y iksemel*
yum install -y net-snmp-devel net-snmp-utils curl-devel OpenIPMI OpenIPMI-devel rpm-build openldap openldap-devel java java-devel
yum install -y pam-devel (目的是为了安装monit来监控heartbeat等服务)
yum install -y gcc*

#如果php不支持mysql查看php-mysql是否安装成功
```

平台在 centos 6+ 
```
安装epel源
rpm -ivh http://mirrors.sohu.com/fedora-epel/6/x86_64/epel-release-6-8.noarch.rpm

然后安装支持PHP5.4 以上的源 并安装PHP 相关的服务
rpm -Uvh http://mirror.webtatic.com/yum/el6/latest.rpm
yum install -y httpd mysql mysql-server
yum install php55w  php55w-bcmath php55w-cli php55w-common  php55w-devel php55w-fpm    php55w-gd php55w-imap  php55w-ldap php55w-mbstring php55w-mcrypt php55w-mysql   php55w-odbc   php55w-pdo php55w-pear  php55w-pecl-igbinary  php55w-xml php55w-xmlrpc php55w-opcache php55w-intl php55w-pecl-memcache 

其他的安装包同7
```


**开启HTTPD MYSQL**

#### 安装zabbix
##### 下载zabbix 并解压
```
https://www.zabbix.com/download
然后解压
```

##### 编译并开启相关服务
```
./configure  --prefix=/usr/local/zabbix --enable-server --enable-proxy --enable-agent --enable-ipv6 --with-mysql --with-net-snmp  --with-libcurl --with-openipmi --with-ldap --with-ssh2 --with-jabber --enable-java --with-libxml2
```
`--with-libxml2` ` --with-libcurl ` 为了开启vm的监控
#### 创建用户和组
```
groupadd zabbix
useradd -g zabbix zabbix
```
复制文件到web目录下！
#### 创建目录
```
mkdir /var/www/html/zabbix
```
/root/Downloads/zabbix-3.0.1/frontends/php
```
cp -rp * /var/www/html/zabbix/
```


#### 创建库
```
create database zabbix default charset utf8;
```
##### 创建zabbix用户并设置密码
```
grant all privileges on zabbix.* to 'zabbix'@'%' identified by 'zabbix';
flush privileges;
```

进入解压后的zabbix目录 database/mysql 导入相应数据到zabbix数据库
##### 导入库文件

```
mysql -uroot -p zabbix < schema.sql
初始化server（顺序不能变）
mysql -uroot -p zabbix < images.sql 
mysql -uroot -p zabbix < data.sql 
```
#### 启动脚本
复制启动脚本到init.d
/root/Downloads/zabbix-3.0.1/misc/init.d/fedora/core5
```
[root@localhost core5]# ls
zabbix_agentd  zabbix_server
[root@localhost core5]# cp -rp * /etc/init.d/
```
注意：
/usr/local/zabbix-2.2.2/etc/zabbix_*
此安装路径下也两个启动程序。但是不是启动脚本。曾经使用这两个命令，但是zabbix启动不起来。所以使用了启动脚本

然后修改这两个脚本文件中zabbix的路径
ZABBIX_BIN="/usr/local/zabbix/sbin/zabbix_server"

修改/etc/php.ini，使配置达到要求
    
```
max_execution_time = 300
 memory_limit = 128M
 post_max_size = 16M
 upload_max_filesize = 2M
 max_input_time = 300
 date.timezone=Asia/Shanghai
```
启动相关服务
```
/etc/init.d/zabbix_agentd start
/etc/init.d/zabbix_server start
systemctl start httpd.service 
systemctl enable httpd
systemctl start mariadb
systemctl enable mariadb
```
访问本机地址下的zabbix网站进行设置

___
