---
title: Ubuntu安装Django-mysql
date: 2017-04-29 14:21:07
tags: 
    - django
    - mysql
categories: web
copyright: true
---

#### 安装mysql
```
sudo apt install mysql-server mysql-client libmysqlclient-dev
```
#### 启动mysql
**并重置密码 修改配置**
```
/etc/init.d/mysql start
#重置密码,注意符合复杂度
mysql_secure_installation
#注释 vi /etc/mysql/mysql.conf.d/mysqld.cnf  
bind-address = 127.0.0.1
#连接数据库开启root远程
mysql -u -p  xxxxx
grant all privileges on *.* to 'root'@'%' identified by '******';
flush privileges
或退出重启
/etc/init.d/mysql restart
```
#### 安装pip3
```
sudo apt-get install python3-pip
```
#### 安装django
```
sudo apt-get install django==1.10
```
#### 使用django
```
sudo mkdir django
cd django
sudo django-admin startproject study
cd study
sudo python3 manage.py startapp app1
sudo python3 manage.py runserver 0.0.0.0:8000

#本机打开127.0.0.1:8000即可看到 
It worked!
Congratulations on your first Django-powered page.
```

___
