---
title: zabbix-proxy搭建
date: 2017-3-29 22:25:57
tags: 
    - zabbix-proxy
    - zabbix
categories: zabbix
copyright: true
---
**流程**
>   安装proxy--->监控proxy--->配置代理的mariadb--->添加proxy--->使用

<!-- more -->




#### 安装proxy

```bash
#添加zabbix 源
rpm -ivh  http://repo.zabbix.com/zabbix/3.2/rhel/7/x86_64/zabbix-release-3.2-1.el7.noarch.rpm

#安装相关软件
yum -y install fping zabbix-proxy-mysql
```

#### 监控proxy(配置agent)

```bash
#安装并配置zabbix-agent
yum install zabbix-agent

#vim /etc/zabbix/zabbix_agent.conf 进行修改
Server=zabbix server IP
Server active=zabbix server IP
Host name=zabbixproxy #(hsotname在server上添加监控的时候是要用到的)

#开启并开机启动服务

#然后去server上添加主机
配置---> 主机---> 创建主机---> 
主机名:上述的hostname
agent 接口 就是代理的IP地址
#其余按需配置,不多赘述

```

#### 配置proxy的mariadb

```bash
#安装mariadb
yum groupinstall mariadb mariadb-client 
systemctl start mariadb
systemctl enable mariadb

#设置密码等
mysql_secure_installation
#登录并创建库
create database zabbix_proxy character set utf8;
#创建用户并授权
grant all privileges on zabbix_proxy.* to zabbix@localhost identified by 'centos';
#刷新生效
flush privileges

#导入zabbix数据库到mariadb
rpm -ql zabbix-proxy-mysql #查找schema.sql.gz文件位置
cd /usr/share/doc/zabbix-proxy-mysql-3.2.4/
gunzip schema.sql.gz
#导入
mysql -uroot -p zabbix_proxy < schema.sql



```

#### 添加proxy

```bash
#修改配置文件
#vim /etc/zabbix/zabbix_proxy.conf
Server=zabbix server IP
Hostname=zabbixproxy


#DB 设定档
DBName=zabbix
DBUser=zabbix
DBPassword=111111
ProxyLocalBuffer=0 #设定为0小时，除非有其他第三方应用和插件需要调用
ProxyOfflineBuffer=1 #proxy或者server无法连接时，保留离线的监控数据的时间，单位小时
ConfigFrequency=60 #server和proxy配置修改同步时间间隔，设定5-10分钟即可。
DataSenderFrequency=10 #数据发送时间间隔，10-30s；
#网络传输质量越好，可以设定间隔时间越短，监控效果也越迅速；
StartPollers=10 #开启多线程数，一般不要超过30个；
StartPollersUnreachable=1 #该线程用来单独监控无法连接的主机，1个即可；
StartTrappers=10 #trapper线程数
StartPingers=1 #fping线程数
CacheSize=64M #用来保存监控数据的缓存数，根据监控主机数量适当调整；
Timeout=10 #超时时间，设定不要超过30s，不然会拖慢其他监控数据抓取时间；
TrapperTimeout=30 #同上
FpingLocation=/usr/sbin/fping #配合simple check icmp检测使用，如不需要可关闭；
其他配置默认即可；

#开启并开机启动服务

```

回到web端

>     Administration-->Proxies-->Create proxy

hostname 与上述的要一致



>   当我们再次创建主机的时候就可以通过 修改 *由agent代理程序监测(Monitored by proxy)* 选项进行选择了


___