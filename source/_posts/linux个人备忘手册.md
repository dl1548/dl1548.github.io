---
title: linux个人备忘手册
date: 1111-11-11 11:11:11
tags: -个人备忘手册
categories: linux
copyright: true
password: woshizhu
---

> 一些小命令的记录

<!--more-->

#### 按当前日期命名文件
```
创建备份Shell脚本:
输入/粘贴以下内容：
#!/bin/bash
mysqldump -uusername -ppassword DatabaseName > /home/backup/DatabaseName_$(date +%Y%m%d_%H%M%S).sql
对备份进行压缩：
#!/bin/bash
mysqldump -uusername -ppassword DatabaseName | gzip > /home/backup/DatabaseName_$(date +%Y%m%d_%H%M%S).sql.gz
```

#### -mtime
```
查找 并移动3天前的
find /net-log/ -mtime +3 -name "*.*" -exec mv {} /tmp/ \;

命令可写到定时任务中,对长时间不读取文件进行删除
crontab -e
* * * * *  + 程序 +命令或脚本
#如果是bash命令 可直接写.
例:每天0点执行test.py
0 0 * * *  /usr/bin/python3   /script/test.py 
```

#### ssh快捷登录
```
zili@Ubuntu:~$ cat ~/.ssh/config 
Host study
User root
Hostname 10.1.1.11
Port 21131
#默认为22 有修改则填写
#若有多个主机,直接复制修改即可
zili@Ubuntu:~$ ssh study
root@10.1.1.11's password: 
```














___