---
title: zabbix tripper和zabbix_sender
date: 2017-12-24 20:50:04
tags: zabbix
categories: zabbix
copyright: true
---

>zabbix获取key值有超时时间，如果自定义的key脚本一般需要执行很长时间，这根本没法去做监控，所以使用zabbix监控类型zabbix trapper，需要配合zabbix_sender给它传递数据

<!--more-->

#### zabbix_sender安装
各版本客户端安装
centos5
`rpm -ivh http://mirrors.aliyun.com/zabbix/zabbix/3.0/rhel/5/x86_64/zabbix-sender-3.0.5-1.el5.x86_64.rpm`
centos6
`rpm -ivh http://mirrors.aliyun.com/zabbix/zabbix/3.0/rhel/6/x86_64/zabbix-sender-3.0.5-1.el6.x86_64.rpm`
centos7
`rpm -ivh http://mirrors.aliyun.com/zabbix/zabbix/3.0/rhel/7/x86_64/zabbix-sender-3.0.5-1.el7.x86_64.rpm`

#### zabbix_sender格式

格式
`zabbix_sender [-Vhv] {[-zpsI] -ko | [-zpI] -T -i <file> -r} [-c <file>]`

```bash
  -c --config <file>                   #配置文件绝对路径
  -z --zabbix-server <server>         # zabbix server的IP地址
  -p --port <server port>              #zabbix server端口.默认10051
  -s --host <hostname>                # 主机名，zabbix里面配置的主机名（不是服务器的hostname），不能使用ip地址
  -I --source-address <IP address>     #源IP
  -k --key <key>                     #  监控项的key
  -o --value <key value>               #key值
  -i --input-file <input file>        # 从文件里面读取hostname、key、value 一行为一条数据，使用空格作为分隔符，如果主机名带空格，那么请使用双引号包起来
  -T --with-timestamps              #一行一条数据，空格作为分隔符: <hostname> <key> <timestamp> <value>，#配合 --input-file option，timestamp为unix时间戳
  -r --real-time                      #将数据实时提交给服务器
  -v --verbose                         #详细模式, -vv 更详细
```
```
例一:
zabbix_sender -z server -s host -k key -o value
例二：
zabbix_sender -c config-file -k key -o value
例三：
zabbix_sender -z server -i file
```

#### 实例
`zabbix_sender -z 10.1.93.218 -s 10.1.27.31 -k mytrip -o 666`
```
[root@elk zabbix]# zabbix_sender -z 10.1.93.218 -s 10.1.27.31 -k mytrip -o 666
info from server: "processed: 1; failed: 0; total: 1; seconds spent: 0.000089"
sent: 1; skipped: 0; total: 1

-z 10.1.93.218 : zabbix server
-s 10.2.27.31 : 主机名,和web页面中要一致
-k mytrip  :定义的key ,和web页面中要一致
-o 666  : value,发送的值.
```
也可一通过某些命令来获取值发送
获取当前登录用户数
`zabbix_sender -z 10.1.93.218 -s 10.1.27.31 -k mytrip -o $(w|sed -n '1,2!p'| wc -l)`

还可以通过读取文本
```
-i  # 从文件里面读取hostname、key、value 一行为一条数据
使用空格作为分隔符，如果主机名带空格，那么请使用双引号包起来

zabbix_sender -z 10.1.93.218 -i a.txt

cat a.txt
#cat f.txt
10.1.27.31  mytrip 1
10.1.27.31  mytrip2 2
```


#### web页面
zabbix server方面
```
主机:
  - 主机名称要和 -s 一致
监控项:
  - 创建监控项
  - 类型:(zabbix tripper / 采集器)
  - 键值: 要和 -k 一致
```

`zabbix_sender` 可通过`crontab`来定时执行.可直接写,也可写成脚本后执行,建议分类编辑为脚本.



