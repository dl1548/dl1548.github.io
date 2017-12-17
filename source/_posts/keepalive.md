---
title: keepalive
date: 2017-06-06 20:16:08
tags: keepalive
categories:
    - linux
copyright: true
---

>Keepalived使用的vrrp协议方式，虚拟路由冗余协议 (Virtual Router Redundancy Protocol，简称VRRP).
Keepalived模拟路由器的高可用，Heartbeat或Corosync的目的是实现Service的高可用。

<!--more-->

#### 安装 
如无特殊需求,直接yum安装`yum install keepalived`
```
主配置文件：/etc/keepalived/keepalived.conf
主程序文件：/usr/sbin/keepalived
Unit File：keepalived.service
Unit File的环境配置文件：/etc/sysconfig/keepalived
```
#### 单主模式
10.1.27.23 主,27.24 备,vip:27.21
##### master
```bash
[root@master keepalived]# cat keepalived.conf
! Configuration File for keepalived

global_defs {   #全局配置
   notification_email {
    lizili@xxxxxx.com   #出问题是,接收人邮件
   }
   notification_email_from lizili@xxxxxx.com #发件人邮箱
   smtp_server smtp.xxxxxx.com
   smtp_connect_timeout 30
   router_id LVS_DEVEL
}
# 这个邮件比较鸡肋,建议通过脚本发送.

vrrp_instance VI_1 {       #vrrp命名,多个的时候,命名要不一致.
    state MASTER         #虚拟机路由器状态MASTER/BACKUP
    interface eno16777984  #通过那个网卡发送vrrp广播
    virtual_router_id 51 #虚拟路由器ID,如果有多个VI要注意区分这个ID
    priority 100  #优先级,越大越优先(取值范围1-255)
    advert_int 1 #广播时间间隔,默认1s
    authentication {  #传递信息认证方式,密码仅支持8位
        auth_type PASS  
        auth_pass lizili
    }
    virtual_ipaddress {  #虚拟路由地址
        10.1.27.21
    }
}

```
##### slave
```
[root@slave keepalived]# cat keepalived.conf
! Configuration File for keepalived

global_defs {
   notification_email {
    lizili@xxxxxx.com
   }
   notification_email_from lizili@xxxxxx.com
   smtp_server smtp.xxxxxx.com
   smtp_connect_timeout 30
   router_id LVS_DEVEL
}

vrrp_instance VI_1 {
    state BACKUP
    interface eno16777984
    virtual_router_id 51
    priority 99
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass lizili
    }
    virtual_ipaddress {
        10.1.27.21
    }
}

```
##### 测试
两台都安装httpd服务`yum -y install httpd`
`vi /var/www/html/index.html`
内容分别写上本机IP,然后通过浏览器访问vip,应该能查看到master的IP地址.
然后关闭master的keepalived服务,刷新网页,应该出现slave的地址

#### 邮件告警
安装mailx
```
#安装mailx邮件服务
yum install mailx -y

#配置文件追加信息（/etc/mail.rc）
vim /etc/mail.rc
#发件人信息
set from=lizili@xxxxxx.com
set smtp=smtp.xxxxxx.com
set smtp-auth-user=lizili
set smtp-auth-password=xxxxxx
set smtp-auth=login

#测试发送
echo "hello world" | mail -s "hello" lizili@xxxxxx.com
#echo "邮件内容" | mail -s "标题" 邮箱地址
#邮件策略上,把账号加如白名单,以防被拉黑.
```
配置keepalived
```
#在VRRP实例中虚拟IP下配置添加以下信息
vrrp_instance VI_1 {
    #Keepalived进入MASTER状态执行脚本
    notify_master "/etc/keepalived/mail_notify.sh master"
    #Keepalived进入BACKUP状态执行脚本
    notify_backup "/etc/keepalived/mail_notify.sh backup"
    #Keepalived进入FAULT状态执行脚本
    notify_fault "/etc/keepalived/mail_notify.sh fault"
｝    
```
新建脚本 755权限
```
vim /etc/keepalived/mail_notify.sh
#!/bin/bash
echo "keepalived 10.1.27.23 $1 状态被激活，请确认服务运行状态"|mail -s "keepalived状态切换" lizili@wondersgroup.com
```

#### 双主模式
配置并没有太大的变化,在添加一个vrrp实例即可,配置如下
master
```shell
vrrp_instance VI_2 {       #vrrp命名,多个的时候,命名要不一致.
    state BACKUP         #修改为backup
    interface eno16777984  #通过那个网卡发送vrrp广播
    virtual_router_id 52 #虚拟路由器ID,如果有多个VI要注意区分这个ID
    priority 99  #优先级,越大越优先(取值范围1-255)
    advert_int 1 #广播时间间隔,默认1s
    authentication {  #传递信息认证方式,密码仅支持8位
        auth_type PASS  
        auth_pass lizili
    }
    virtual_ipaddress {  #虚拟路由地址
        10.1.27.11
    }
}

```
slave
```shell
vrrp_instance VI_1 {
    state MASTER
    interface eno16777984
    virtual_router_id 52
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass lizili
    }
    virtual_ipaddress {
        10.1.27.11
    }
}
```
其实就是增加新的配置 VI_2 使用Server B 做主，如此 Server A、B 各自拥有主虚拟 IP，同时备份对方的虚拟 IP, 这个方案可以是不同的服务，或者是同一服务的访问分流(配合 DNS 使用)


>未完待续
___


