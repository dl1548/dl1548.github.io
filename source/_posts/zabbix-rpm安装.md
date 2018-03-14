---
title: zabbix-rpm安装
date: 2017-03-28 10:56:27
tags: zabbix
categories: zabbix
copyright: true
---


[官网](https://www.zabbix.com/documentation/3.4/manual/installation/install_from_packages/rhel_centos)提供不同版本的仓库文件

此次安装以centos7为基础,zabbix3.4
<!--more-->

___

安装源文件
```
rpm -ivh http://repo.zabbix.com/zabbix/3.4/rhel/7/x86_64/zabbix-release-3.4-2.el7.noarch.rpm
```
#### 安装
```
yum install zabbix-server-mysql -y
yum install zabbix-web-mysql -y
yum -y install mariadb mariadb-server mariadb-libs mariadb-devel
```


#### 启动httpd,mysql
```
 systemctl enable mariadb
 systemctl start mariadb
 systemctl enable httpd
 systemctl enable httpd
```


#### 设置mysql密码
```
mysqladmin -uroot password 'password';
```
这里建议,运行一次mysql配置向导,删除匿名用户
```
运行mysql_secure_installation会执行几个设置：
  a)为root用户设置密码
  b)删除匿名账号
  c)取消root用户远程登录
  d)删除test库和对test库的访问权限
  e)刷新授权表使修改生效
```

#### 配置php
```
vi  /etc/php.ini
date.timezone = Asia/Shanghai
max_execution_time = 300
post_max_size = 32M
max_input_time=300
memory_limit = 128M
```


#### 创建zabbix库
```
create database zabbix charset utf8;
```

#### 创建/授权
创建zabbix用户并授权
```
grant all privileges on zabbix.* to 'zabbix'@'%' identified by 'zabbix';

#刷新
flush privileges;
```

#### 导入库文件
```
cd /usr/share/doc/zabbix-server-mysql-3.4.3
gzip -d create.sql.gz
mysql -uzabbix -p zabbix < create.sql
```
#### 配置zabbix_server
```
vi /etc/zabbix/zabbix_server.conf
LogFile=/var/log/zabbix/zabbix_server.log
DBName=zabbix
DBUser=zabbix
DBPassword=zabbix
```

#### 启动zabbix
```
systemctl start zabbix-server.service 
systemctl enable zabbix-server.service 

```
#### web访问
```
http://ipaddr/zabbix
```
最后会提示创建成功,并且提示配置文件路径

>Congratulations! You have successfully installed Zabbix frontend.
>Configuration file "/etc/zabbix/web/zabbix.conf.php" created.

___
