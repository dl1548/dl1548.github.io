---
title: linux文件共享
date: 2017-03-25 13:27:05
tags:
    - nfs
    - samba
categories: linux
copyright: true
---

> 文章包括NFS和samba


<!--more-->
#### NFS

##### 安装相关服务
```
yum install nfs-utils rpcbind
```

##### 创建共享文件夹
```
mkdir -p /nfs_share/share-one
mkdir -p /nfs_share/share-two
```

##### 编辑配置文件

`vi /etc/exports`
```bash
# 所有的IP都可访问
/nfs_share/share-one *(rw,async,no_root_squash)

# 某网段/IP可访问
/nfs_share/share-two 192.168.1.0/24(rw,async,no_root_squash)
```
然后输入`exportfs -r`重新共享所有目录

配置文件还是要解释下:
`rw`:read-write，可读写
`ro`:read-only，只读
`sync`:文件同时写入硬盘和内存
`async`:文件暂存于内存，而不是直接写入内存
`no_root_squash`:NFS客户端连接服务端时如果使用的是root的话，那么对服务端分享的目录来说，也拥有root权限。显然开启这项是不安全的
`root_squash`:NFS客户端连接服务端时如果使用的是root的话，那么对服务端分享的目录来说，权限降低至拥有匿名用户权限，通常他将使用nobody或nfsnobody身份
`all_squash`:不论NFS客户端连接服务端时使用什么用户，对服务端分享的目录来说都是匿名用户权限；

##### 启动服务

```
service rpcbind start  #这个先启动
service nfs start
chkconfig rpcbind on
chkconfig nfs on
```

##### 客户端挂载
同服务器,安装服务创建挂载目录并开启服务,然后
```bash
#查看共享目录
showmount -e server_ip

#挂载
mount -t nfs 192.168.1.123:/nfs_share/share-two /mnt
#如果网络不稳,换成tcp协议挂载,默认使用udp
mount -t nfs 192.168.1.123:/nfs_share/share-two /mnt -o proto=tcp -o nolock
```

##### 开机自动挂载
```
vi /etc/fstab

server_ip:/nfs_share/share-two /mnt nfs default 0 0
```


#### samba

##### 安装服务

yum -y install samba samba-client samba-common

##### 配置文件
vim /etc/samba/smb.conf
```bash
[smbshare]  #命名
comment = samba share #注释
path    = /smbshare #路径
read list = harry #可读用户
#这里也可以用组来表示 read list = @samba_group
#writeable=yes
#write list = harry  #可写用户,可写默认有可读权限.
hosts allow = 172.24.1.  #ip限制,注意点.

```

##### 添加用户
添加组: `groupadd samba_group`
先添加系统用户`useradd harry -s /sbin/nologin -g samba_group`
添加samba用户`smbpasswd -a harry` 然后输入密码即可

#####  文件夹&权限
```
mkdir /smbshare
setfacl -m u:harry:rwx /smbshare 
#setfacl -m g:samba_group:rwx /smbshare #根据组修改文件acl
#getfacl filename #查看文件夹的权限
```

##### 开启服务
```bash
systemctl restart smb
systemctl restart nmb
systemctl enable smb
systemctl enable nmb
```

##### 客户端挂载

安装服务
`yum -y install cifs-utils`
vim /etc/fstab
```
//192.168.1.(smb服务地址)/multishare /mnt/multishare cifs sec=ntlmssp,user=harry,password=harry,multiuser       0 0
```
`mount –a`
`df -h`

___