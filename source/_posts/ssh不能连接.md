---
title: ssh不能连接(yum被破坏)
date: 2017-08-09 14:41:07
tags:
    - ssh
    - yum
categories: linux
copyright: true
---
同事反应，云主机能ping但ssh不上。
本地登录后，检查防火墙,ssh服务，发现服务没有了，然后就想安装`openssh-server`,结果yum不能用了
<!--more-->

```
YumRepo Error: All mirror URLs are not using ftp, http[s] or file. 
/Eg. 
removing mirrorlist with no valid mirrors: /var/cache/yum/x86_64/6/base/mirrorlist.txt 
Error: Cannot find a valid baseurl for repo: base
```

查看防火墙，网络状态，以及DNS。能否ping通yum源。`mirrorlist.centos.org`
**能通则可以排除网络问题**

检查仓库是否正常,检查变量等信息是否正常。
```
lsb_release -a
能否正常显示版本号等信息

LSB Version:    :base-4.0-amd64:base-4.0-noarch:core-4.0-amd64:core-4.0-noarch:graphics-4.0-amd64:graphics-4.0-noarch:printing-4.0-amd64:printing-4.0-noarch
Distributor ID: CentOS
Description:    CentOS release 6.7 (Final)
Release:    6.7
Codename:   Final

# rpm -q --qf %{version} centos-release;echo
6
# rpm -q --qf %{arch} centos-release;echo
x86_64


不正常则备份仓库配置，并手动修改*.repo文件，将$releasever变量全替换成6即可正常yum了。

如果是7版本则换成7

```

___
