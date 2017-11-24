---
title: iSCSI存储
date: 2017-05-15 15:36:20
tags: iSCSI
categories: linux
copyright: true
---

临时需要个挂个iSCSI,发现基本想不起来,果然好记性不如烂笔头,还是写一下比较好-。-

<!--more-->
>摘:
iSCSI是一种基于TCP/IP 的协议，用来建立和管理IP存储设备、主机和客户机等之间的相互连接，并创建存储区域网络（SAN）。SAN 使得SCSI 协议应用于高速数据传输网络成为可能，这种传输以数据块级别（block-level）在多个数据存储网络间进行。SCSI 结构基于C/S模式，其通常应用环境是：设备互相靠近，并且这些设备由SCSI 总线连接。
·   
简单说
iSCSI就是把SCSI指令通过TCP/IP协议封装起来，在以太网中传输。iSCSI 可以实现在IP网络上传递和运行SCSI协议，使其能够在诸如高速千兆以太网上进行数据存取，实现了数据的网际传递和管理。基于iSCSI建立的存储区域网（SAN）与基于光纤的FC-SAN相比，具有很好的性价比。


关闭了selinux 和防火墙，centos7
target （服务器）： 192.168.247.149
Initiator（客户端）：192.168.247.154

#### 服务端
##### 新加硬盘做存储
```
Disk /dev/sdb: 21.5 GB, 21474836480 bytes, 41943040 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```
##### 创建磁盘并格式化

> 一个硬盘主 分区至少有1个，最多4个，扩展分区可以没有，最多1个。且主分区+扩展分区总共不能超过4个。
为了突破这最多四个主分区的限制，管理员可以把其中一个主分区设置为扩展分区(注意只能够使用一个扩展分区)来进行扩充。而在扩充分区下，又可以建立多个逻辑分区。也就是说，扩展分区是无法直接使用的，必须在细分成逻辑分区才可以用来存储数据。通常情况下，逻辑分区的起始位置及结束位置记录在每个逻辑分区的第一个扇区，这也叫做扩展分区表。在扩展分区下，系统管理员可以根据实际情况建立多个逻辑分区，将一个扩展分区划割成多个区域来使用。

格式化为LVM（LVM是 Logical Volume Manager逻辑卷管理）
LVM的好处就是可以按需分配，动态管理
```bash
Command (m for help): n
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p       #必须要一个主分区
Partition number (1-4, default 1): 
First sector (2048-41943039, default 2048): 
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-41943039, default 41943039): 
Using default value 41943039
Partition 1 of type Linux and of size 20 GiB is set

Command (m for help): t    #类型
Selected partition 1        #编号 可按  L  查看要创建的格式
Hex code (type L to list all codes): 8e
Changed type of partition 'Linux' to 'Linux LVM'

Command (m for help): p      #查看

Disk /dev/sdb: 21.5 GB, 21474836480 bytes, 41943040 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0xea844b86

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048    41943039    20970496   8e  Linux LVM

保存即可

partprobe #是配置生效
```

##### 创建逻辑卷

```
[root~] ]$pvcreate /dev/sdb1
  Physical volume "/dev/sdb1" successfully created
[root~] ]$pvs
  PV         VG     Fmt  Attr PSize  PFree 
  /dev/sda2  centos lvm2 a--  59.78g     0 
  /dev/sdb1         lvm2 ---  20.00g 20.00g
[root~] ]$vgcreate vg_iscsi /dev/sdb1
  Volume group "vg_iscsi" successfully created
[root~] ]$vgs
  VG       #PV #LV #SN Attr   VSize  VFree 
  centos     1   2   0 wz--n- 59.78g     0 
  vg_iscsi   1   0   0 wz--n- 20.00g 20.00g
[root~] ]$lvcreate -l 100%FREE -n /dev/vg_iscsi/lv_iscsi
  Logical volume "lv_iscsi" created.
[root~] ]$lvs
  LV       VG       Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root     centos   -wi-ao---- 57.78g                                                    
  swap     centos   -wi-ao----  2.00g                                                    
  lv_iscsi vg_iscsi -wi-a----- 20.00g       

```

##### 安装配置target
```
[root~] ]$yum -y install targetcli
[root~] ]$targetcli 
#最初状态
/> ls
o- / ......................................................................................................................... [...]
  o- backstores .............................................................................................................. [...]
  | o- block .................................................................................................. [Storage Objects: 0]
  | o- fileio ................................................................................................. [Storage Objects: 0]
  | o- pscsi .................................................................................................. [Storage Objects: 0]
  | o- ramdisk ................................................................................................ [Storage Objects: 0]
  o- iscsi ............................................................................................................ [Targets: 0]
  o- loopback ......................................................................................................... [Targets: 0]
/> cd backstores/block 
/backstores/block> create sdb1_iscsi /dev/vg_iscsi/lv_iscsi
Created block storage object sdb1_iscsi using /dev/vg_iscsi/lv_iscsi.
切换到iscsi下
/iscsi> create iqn.2017-05.com.master:ip149   #注意全局唯一，格式iqn.日期（日期只能有年月并且月份必须加0.颠倒的domain：自定义标识
/iscsi/iqn.20...149/tpg1/acls> create iqn.2017-05.com.master:ip149
Created Node ACL for iqn.2017-05.com.master:ip149
/iscsi/iqn.20...149/tpg1/acls> cd ../luns 
/iscsi/iqn.20...149/tpg1/luns> create /backstores/block/sdb1_iscsi
/iscsi/iqn.20...149/tpg1/luns> cd ../portals/
/iscsi/iqn.20.../tpg1/portals> create 192.168.247.149
Using default IP port 3260
/iscsi/iqn.20.../tpg1/portals> cd /
#完成状态
/> ls
o- / ......................................................................................................................... [...]
  o- backstores .............................................................................................................. [...]
  | o- block .................................................................................................. [Storage Objects: 1]
  | | o- sdb1_iscsi ........................................................ [/dev/vg_iscsi/lv_iscsi (20.0GiB) write-thru activated]
  | o- fileio ................................................................................................. [Storage Objects: 0]
  | o- pscsi .................................................................................................. [Storage Objects: 0]
  | o- ramdisk ................................................................................................ [Storage Objects: 0]
  o- iscsi ............................................................................................................ [Targets: 1]
  | o- iqn.2017-05.com.master:ip149 ...................................................................................... [TPGs: 1]
  |   o- tpg1 ............................................................................................... [no-gen-acls, no-auth]
  |     o- acls .......................................................................................................... [ACLs: 1]
  |     | o- iqn.2017-05.com.master:ip149 ......................................................................... [Mapped LUNs: 1]
  |     |   o- mapped_lun0 ............................................................................ [lun0 block/sdb1_iscsi (rw)]
  |     o- luns .......................................................................................................... [LUNs: 1]
  |     | o- lun0 ...................................................................... [block/sdb1_iscsi (/dev/vg_iscsi/lv_iscsi)]
  |     o- portals .................................................................................................... [Portals: 1]
  |       o- 0.0.0.0:3260 ..................................................................................................... [OK]
  o- loopback ......................................................................................................... [Targets: 0]
/> exit
```
##### 启动服务
```
[root~] ]$systemctl start target.service 
[root~] ]$systemctl enable target.service 
Created symlink from /etc/systemd/system/multi-user.target.wants/target.service to /usr/lib/systemd/system/target.service.
[root~] ]$
```



#### 客户端

```
 [root~] ]$yum -y install iscsi-initiator*
 [root~] ]$vim /etc/iscsi/initiatorname.iscsi 
InitiatorName=iqn.2017-05.com.master:ip149 #服务器的iqn

[root@minion-02 /]# systemctl restart iscsi
[root@minion-02 /]# iscsiadm -m discovery -t st -p 192.168.247.149
192.168.247.149:3260,1 iqn.2017-05.com.master:ip149
[root@minion-02 /]# iscsiadm -m node -T iqn.2017-05.com.master:ip149 -p 192.168.247.149 -l
Logging in to [iface: default, target: iqn.2017-05.com.master:ip149, portal: 192.168.247.149,3260] (multiple)
Login to [iface: default, target: iqn.2017-05.com.master:ip149, portal: 192.168.247.149,3260] successful.
[root@minion-02 /]# fdisk -l
#可以看到iscsi已经挂上 为sdb
Disk /dev/sdb: 21.5 GB, 21470642176 bytes, 41934848 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 4194304 bytes
[root@minion-02 /]# 


#sdb就像是新加的硬盘，我们可以fdisk 它
 n  p 1 enter enter w
我们可以对这个盘进行单独的挂载。切记不可扩展到某个分区
```

##### 挂载
```
[root@minion-02 /]#  mkfs.xfs /dev/sdb1
[root@minion-02 /]#  mkdir /mnt/iscsi
[root@minion-02 /]#  mount /dev/sdb1 /mnt/iscsi/
[root@minion-02 /]# blkid
/dev/sda1 : UUID="64f589c2-2ac4-47f7-8d35-170913a2563f" TYPE=”xfs”
[root@minion-02 /]# vim /etc/fstab
UUID="64f589c2-2ac4-47f7-8d35-170913a2563f"     /mnt/iscsi   xfs     defaults,_netdev        0 0

fstab一定呀写对，不然系统重启会启动不起来，特别注意。
万一 启动不起来怎么办？
重启虚拟机  按 e 进行编辑
修改ro  rd........等  为 rd.break

switch_root:/# mount –o remount,rw /sysroot/
switch_root:/# chroot /sysroot/
sh-4.2#vi /etc/fstab    #重新编辑
##这里还可以修改root密码。
passwd root

如果开了seLinux.记得 touch /.autorelabel 这句是为了selinux生效,否则系统无法正常启动
如果忘记密码也可用这个办法修改
```

___
