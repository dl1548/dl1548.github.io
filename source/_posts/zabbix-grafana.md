---
title: zabbix-grafana
date: 2017-04-11 14:39:18
tags:
    - zabbix
    - grafana
categories: zabbix
copyright: true
---
<img src="/images/grafana.jpg" width = "70%" height = "50%" alt="模板" align=center />
**前言**
(封面图片来自网上)
Grafana是一个可视化面板（Dashboard），有非常漂亮的图表和布局展示，通过插件与zabbix结合,可以很好的弥补zabbix图形化扎眼的缺点
<!--more-->

**centos7**

>[官网demo](http://play.grafana.org/)
>[grafana-zabbix 官方文档](http://docs.grafana-zabbix.org/)
>[官方安装教程](http://docs.grafana.org/installation/rpm/)

**简单说明**
```bash
#安装grafana
 sudo yum install https://s3-us-west-2.amazonaws.com/grafana-releases/release/grafana-4.2.0-1.x86_64.rpm
#安装zabbix 插件
 grafana-cli plugins install alexanderzobnin-zabbix-app
#安装饼图插件
 grafana-cli plugins install grafana-piechart-panel

#启动grafana
systemctl start grafana-server
systemctl enable grafana-server
```
> 访问`IPADDRESS:3000` 即可打开grafana的登录界面
默认的账户密码都是admin
> 进去之后开启zabbix插件，Plugins里开启。
>然后点击数据源(Data sources),添加数据源(add data sources)
然后添加zabbix数据源

这里的URL要是zabbix网站地址 比如：http://1.1.1.1/zabbix/api_jsonrpc.php

![zabbix源.JPG](http://upload-images.jianshu.io/upload_images/2511748-205e4aa6095ef924.JPG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


[grafana-zabbix 官方文档有如何添加机器,以及如何通过template添加机器的教程](http://docs.grafana-zabbix.org/)

> 创建模板 这里简单记录如何通过模板添加主机

![新建模板.png](http://upload-images.jianshu.io/upload_images/2511748-beff502a8624b8c0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**grafana 提供了四个变量,用来确认 群组,主机,应用集,监控项,一般情况下我们只要定义下group,host,item就能进行分组/分类监控了.**

>添加模板变量


![变量.png](http://upload-images.jianshu.io/upload_images/2511748-0a4477e50738843c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 这里通过添加两个变量 帮助理解模板的变量


![变量配置1.png](http://upload-images.jianshu.io/upload_images/2511748-503e5c167ae2d381.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![变量配置2.png](http://upload-images.jianshu.io/upload_images/2511748-e17600e907f1ec0e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**这样两个变量定义下来,就能对组和主机进行一个筛选,当然还可以定义变量对应用,监控项等进行筛选.不再述**
```
query的匹配原则
        *                     returns all groups
        *.*                   returns all hosts (from all groups)
        Servers.*             returns all hosts in group Servers
        Servers.*.*           returns all applications in group Servers
        Servers.*.*.*         returns all items from hosts in group Servers
```
>保存模版后,可以给模板添加图形


![模版添加图形化.png](http://upload-images.jianshu.io/upload_images/2511748-d0ca9d9fb7f4cefb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


完
