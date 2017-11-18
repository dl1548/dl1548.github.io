---
title: git客户端最新版安装
date: 2017-07-12 20:11:32
tags:
    - git
categories:
    - 运维工具
    - git
copyright: true
---
升级Git为最新版
<!--more-->
centos7环境
> 系统默认安装为1.8版本，源码安装2.9

#### 卸载并安装

```
#卸载默认版本
yum remove git -y

#安装依赖库
yum -y install curl-devel expat-devel gettext-devel openssl-devel zlib-devel
yum -y install gcc perl-ExtUtils-MakeMaker

#新建文件夹，下载git源码包
mkdir /usr/local/git
cd 进去
wget https://github.com/git/git/archive/v2.9.2.tar.gz

#解压包
tar -zxvf  包名

#安装git
make prefix=/usr/local/git all
make prefix=/usr/local/git install

#添加环境变量
vi /etc/profile  
export PATH="/usr/local/git/bin:$PATH" 
source /etc/profile

#查看版本
git --version   #应该是git version 2.9.2

#设置git默认路径
 ln -s /usr/local/git/bin/git-upload-pack /usr/bin/git-upload-pack 
ln -s /usr/local/git/bin/git-receive-pack /usr/bin/git-receive-pack 
安装完毕！
```
#### 创建git用户和组

```
  groupadd git
  useradd git -g git 
  passwd git 

#切换git用户 避免权限问题
su - git
```

#### Git SSH 密钥认证

```
#生成密钥
ssh-keygen -t rsa -C "****@sina.com"
#会多出两个密钥文件
id_rsa  id_rsa.pub
#复制.pub的内容到你的git账户下
```
![sshkey.png](http://upload-images.jianshu.io/upload_images/2511748-865aa385b099b9d8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 测试连接
```
ssh -T git@github.com

#oschina的
ssh -T git@git.oschina.net
输入yes 会在当前目录生成known_hosts，认证成功！

至此，git实现免密连接
```
可以做先关git的操作了

#### 禁止git用户shell登录

```
vim /etc/password
git:x:502:502::/home/git:/bin/bash
修改为
git:x:502:502::/home/git:/usr/local/git/bin/git-shell
```

完
___
