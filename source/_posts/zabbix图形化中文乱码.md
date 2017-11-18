---
title: zabbix图形化中文乱码
date: 2017-03-29 14:12:33
tags: zabbix
categories: zabbix
copyright: true
---
主要是应为zabbix web端 默认没有中文字体库
<!--more-->

> 从 windows下控制面板->字体->选择一种中文字库,宋体 楷体等
上传至
/var/www/html/zabbix/fonts下 
修改后缀为ttf

修改   include/defines.inc.php 下
```
define('ZBX_GRAPH_FONT_NAME', 'SIMKAI'); // font file name   #SIMKAI 是字体文件名
```
