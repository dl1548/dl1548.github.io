---
title: zabbix自定义key
date: 2017-04-25 18:42:57
tags: zabbix
categories: zabbix
copyright: true
---

当zabbix的key不能满足需求的时候,可以通过自定义key来解决.在客户端配置文件zabbix_angentd.conf里面配置UserParameter.

<!--more-->

参数格式
`UserParameter=key[*],command`

|参数|    描述|
|---|---|
|Key|   唯一. [*]表示里面可以传递多个参数|
|Command    |需要执行的脚本，key的[]里面的参数一一对应$1到$9，一共9个参数。$0表示脚本命令.|
首先去配置文件中,开启include
`Include=/etc/zabbix/zabbix_agentd.d/*.conf`
然后去目录下新建配置文件
```
#新建配置文件
[root@elk zabbix_agentd.d]# ls
mykey.conf  userparameter_mysql.conf
#编辑配置文件
[root@elk zabbix_agentd.d]# cat mykey.conf 
UserParameter = mykey,/etc/zabbix/script/test.sh

#编辑脚本(路径自定义)
[root@elk zabbix_agentd.d]# cat /etc/zabbix/script/test.sh 
#!/usr/bin/env bash
echo '1234324' 
```
重启服务
`systemctl restart zabbix-agent.service`

添加监控项
去前端web页面,相关主机上`创建监控项`即可,键值使用定义的key,如这里的`mykey`.信息类型根据实际情况指定

其他
```
UserParameter=ping[*],echo $1
web键值输入
ping[0] - 将一直返回0
ping[aaa] - 将一直返回 'aaa'
```
```
统计一个文件中有多少行被匹配?
UserParameter=wc[*],grep -c "$2" $1
如下方法将会返回文件中出现指定字符的行数
wc[/etc/passwd,root]
wc[/etc/services,zabbix]
```

