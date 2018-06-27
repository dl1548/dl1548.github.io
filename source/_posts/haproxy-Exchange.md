---
title: haproxy-Exchange
date: 2018-03-21 17:28:08
tags:
    - haproxy
    - exchange
categories: linux
copyright: true
password: woshizhu
---
使用haproxy负载exchange,可搭配keepalive做高可用
<!--more-->

keepalive [建议阅读](http://blog.dl1548.site/2017/06/06/keepalive/)

exchange负载[官方文档](https://www.haproxy.com/documentation/aloha/7-0/deployment-guides/microsoft-exchange-2013/)

#### 下载
[下载地址](https://www.haproxy.org/#down)


#### 安装
解压并cd
`make TARGET=linux2628  ARCH=x86_64 PREFIX=/usr/local/haproxy`
`make install PREFIX=/usr/local/haproxy`

复制启动脚本到sbin
`cp /usr/local/haproxy/sbin/haproxy /usr/sbin/`

复制启动脚本到init.d
`cp ./examples/haproxy.init /etc/init.d/haproxy`
`chmod 755 /etc/init.d/haproxy`

创建haproxy账号
`useradd -r haproxy`
配置文件可指定进程用户

#### haproxy配置文件

`mkdir /etc/haproxy`
`vim /etc/haproxy/haproxy.cfg`
负载exchange配置
```
defaults
    log 127.0.0.1 local3 info #配合rsyslog
    option dontlognull  
    option redispatch  
    option contstats  
    retries 3  
    timeout connect 5s  
    timeout http-keep-alive 1s
    timeout http-request 15s
    timeout queue 30s  
    timeout tarpit 1m  
    backlog 10000  

    balance roundrobin  
    mode tcp
    option tcplog  
    log global  
    timeout client 300s
    timeout server 300s
    default-server inter 3s rise 2 fall 3

frontend ft_exchange_HTTP
    bind 10.0.2.32:80 name web
    maxconn 10000
    default_backend bk_exchange_HTTP

backend bk_exchange_HTTP
    server mail01 10.0.2.51:80 maxconn 10000 check
    server mail02 10.0.2.52:80 maxconn 10000 check backup

frontend ft_exchange_SSL
    bind 10.0.2.32:443 name ssl
    maxconn 10000
    default_backend bk_exchange_SSL

backend bk_exchange_SSL
    server mail01 10.0.2.51:443 maxconn 10000 check
    server mail02 10.0.2.52:443 maxconn 10000 check backup

frontend ft_exchange_SMTP
    bind 10.0.2.32:25 name smtp
    maxconn 10000
    default_backend bk_exchange_SMTP

backend bk_exchange_SMTP
    server mail01 10.0.2.51:25 maxconn 10000 check
    server mail02 10.0.2.52:25 maxconn 10000 check backup

frontend ft_exchange_SMTP_Secure
    bind 10.0.2.32:587 name smtpssl
    maxconn 10000
    default_backend bk_exchange_SMTP_Secure

backend bk_exchange_SMTP_Secure
    server mail01 10.0.2.51:587 maxconn 10000 check
    server mail02 10.0.2.52:587 maxconn 10000 check backup

frontend ft_exchange_IMAP
    bind 10.0.2.32:143 name imap
    maxconn 10000
    default_backend bk_exchange_IMAP

backend bk_exchange_IMAP
    server mail01 10.0.2.51:143 maxconn 10000 check
    server mail02 10.0.2.52:143 maxconn 10000 check backup

frontend ft_exchange_IMAP_Secure
    bind 10.0.2.32:993 name imapssl
    maxconn 10000
    default_backend bk_exchange_IMAP_Secure

backend bk_exchange_IMAP_Secure
    server mail01 10.0.2.51:993 maxconn 10000 check
    server mail02 10.0.2.52:993 maxconn 10000 check backup

frontend ft_exchange_POP3
  bind 10.0.2.32:110 name pop3
  maxconn 10000
  default_backend bk_exchange_POP3

backend bk_exchange_POP3
  server mail01 10.0.2.51:110 maxconn 10000 check
  server mail02 10.0.2.52:110 maxconn 10000 check backup

frontend ft_exchange_POP3_Secure
  bind 10.0.2.32:995 name pop3ssl
  maxconn 10000
  default_backend bk_exchange_POP3_Secure

backend bk_exchange_POP3_Secure
  server mail01 10.0.2.51:995 maxconn 10000 check
  server mail02 10.0.2.52:995 maxconn 10000 check backup

```

#### 开启日志
`vim /etc/rsyslog.conf`

```
#去掉注释
$ModLoad imudp
$UDPServerRun 514

#添加如下
local3.* /var/log/haproxy.log
```

#### 启动服务
`systemctl restart rsyslog  / haproxy`
`service rsyslog restart / haproxy`
