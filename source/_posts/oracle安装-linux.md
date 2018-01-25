---
title: oracle-11g安装-linux
date: 2017-09-22 21:00:32
tags: oracle
categories: 数据库
copyright: true
---
需要图形化

| 内存大小 | swap空间需求 |
|---|---|
| 4G < mem <8G | 2*mem |
| 8G < mem <32G | 1.5*mem |
| 32G < mem | 32G |

<!--more-->
#### 硬件环境
系统需要图形化
##### 内存
内存: 大于4G ` grep MemTotal /proc/meminfo `
swap:  `grep SwapTotal /proc/meminfo`

| 内存大小 | swap空间需求 |
|---|---|
| 4G < mem <8G | 2*mem |
| 8G < mem <32G | 1.5*mem |
| 32G < mem | 32G |

##### 硬盘
/tmp 空间大于1G `df -h /tmp`
空间需求

|安装模式|软件所需空间|数据文件所需空间|
|---|---|---|
|企业版 Enterprise Edition|4.35G|1.68G|
|标准版 Standard Edition|3.73G|1.48G|

#### 软件环境
##### hosts
```
vim /etc/hosts
#添加信息格式如下
IP hostname  #10.1.27.25 oracle
```
##### 软件包
```
binutils
compat-libcap1
compat-libstdc++-33
elfutils-libelf
elfutils-libelf-devel
gcc-4.1.2 
gcc-c++-4.1.2 
glibc-2.5-24 
glibc-2.5-24 (32 bit) 
glibc-common-2.5 
glibc-devel-2.5 
glibc-devel-2.5 (32 bit) 
glibc-headers-2.5 
ksh-20060214 
libaio-0.3.106 l
ibaio-0.3.106 (32 bit)
libaio-devel-0.3.106 
libaio-devel-0.3.106 (32 bit) 
libgcc-4.1.2libgcc-4.1.2 (32 bit) 
libstdc++-4.1.2 
libstdc++-4.1.2 (32 bit) 
libstdc++-devel 4.1.2 
make-3.81 
numactl-devel-0.9.8.x86_64 
sysstat-7.0.2 
unixODBC-2.2.11 (32-bit) or later
unixODBC-devel-2.2.11 (64-bit) or later
unixODBC-2.2.11 (64-bit) or later 
检查方法：#rpm -q 包名称    //不需要写后面的版本号
安装方法：#rpm –ivh 包名称
也可以通过yum安装
```
#### 用户和组
可在/etc/groups 查看
```
groupadd oinstall –g 1000 #指定组ID
groupadd dba –g 1001
groupadd oper –g 1002
useradd -g oinstall -G dba oracle #oracle所属组和附加组
passwd oracle #设置密码
```
#### 核心参数
**vim /etc/sysctl.conf**
```
添加以下内容：
fs.aio-max-nr = 1048576
fs.file-max = 6815744
kernel.shmall = 2097152
kernel.shmmax = 536870912
kernel.shmmni = 4096
kernel.sem = 250 32000 100 128 
net.ipv4.ip_local_port_range = 9000 65500 
net.core.rmem_default = 262144 
net.core.rmem_max = 4194304 
net.core.wmem_default = 262144 
net.core.wmem_max = 1048586 

#安装oracle 12C 内核有变动，值如下。
shmmax  67286788096
shmall 13141950


2. 使核心参数生效
# /sbin/sysctl –p
```
**vim /etc/security/limits.conf**
```
#添加以下内容：
oracle soft nproc 2047 
oracle hard nproc 16384 
oracle soft nofile 1024 
oracle hard nofile 65536 
oracle soft stack 10240 
```
**/etc/pam.d/login**
```
添加以下内容：
session required pam_limits.so
```
**vi /etc/profile**
```
添加以下内容：
if [ $USER = "oracle" ]; then
  if [ $SHELL = "/bin/ksh" ]; then
    ulimit -p 16384
    ulimit -n 65536
  else
    ulimit -u 16384 
    ulimit -n 65536
  fi
fi
```
##### 配置说明
[原链接更为详细](http://www.jianshu.com/p/23ee9db2a620)
ulimit
>  1.只对当前tty（终端有效），若要每次都生效的话，可以把ulimit参数放到对应用户的.bash_profile里面或/etc/profile；
2.ulimit命令本身就有分软硬设置，加-H就是硬，加-S就是软；
3.默认显示的是软限制，如果运行ulimit命令修改的时候没有加上的话，就是两个参数一起改变.生效；
```
命令参数
-H 设置硬件资源限制.
-S 设置软件资源限制.
-a 显示当前所有的资源限制.
-c size:设置core文件的最大值.单位:blocks
-d size:设置数据段的最大值.单位:kbytes
-f size:设置创建文件的最大值.单位:blocks
-l size:设置在内存中锁定进程的最大值.单位:kbytes
-m size:设置可以使用的常驻内存的最大值.单位:kbytes
-n size:设置内核可以同时打开的文件描述符的最大值.单位:n
-p size:设置管道缓冲区的最大值.单位:kbytes
-s size:设置堆栈的最大值.单位:kbytes
-t size:设置CPU使用时间的最大上限.单位:seconds
-v size:设置虚拟内存的最大值.单位:kbytes
unlimited 是一个特殊值，用于表示不限制
```
>/etc/security/limit.conf 和vim /etc/sysctl.conf
一个是针对用户的,一个是针对系统的
要使 limits.conf 文件配置生效，必须要确保 pam_limits.so 文件被加入到启动文件中,所以修改/etc/pam.d/login,并添加相关内容

#### 创建目录
```
mkdir -p /u01/app/
chown -R oracle:oinstall /u01/app/ 
chmod -R 775 /u01/app/ 
```

#### oracle用户环境变量
vi /home/oracle/.bash_profile
```
export TMP=/tmp
export TMPDIR=$TMP
export ORACLE_BASE=/u01/app/oracle
export ORACLE_HOME=$ORACLE_BASE/product/11.2.0/db_1
export ORACLE_SID=db11g
export ORACLE_TERM=xterm
export PATH=/usr/sbin:$PATH
export PATH=$ORACLE_HOME/bin:$PATH
export LD_LIBRARY_PATH=$ORACLE_HOME/lib:/lib64:/usr/lib64:/usr/local/lib64:/usr/X11R6/lib64/
export CLASSPATH=$ORACLE_HOME/JRE:$ORACLE_HOME/jlib:$ORACLE_HOME/rdbms/jlib
export NLS_LANG="SIMPLIFIED CHINESE_CHINA.ZHS16GBK"
umask 022
```

#### 下载解压包

```
链接: https://pan.baidu.com/s/1nvIGppJ
密码: whna
```
#### 安装oracle
切换oracle 用户，进入解压路径下 database 目录，运行./runInstaller 命令，开始安装

- Next----> Yes
- 选择"Skip Software updates",点击"Next"按钮
- 选择"Install database software only",点击"Next"按钮
- 选择"Single instance database installation"，点击"Next"按钮
- 将"Simplified Chinese"通过">"按钮添加到"Selected Languages"，点击"Next"按钮
- 选择"Enterprise Edition",点击"Next"按钮
- 确认"Oracle Base","Software Location"路径,点击"Next"按钮
- 确认"Inventory Directory"路径和"oraInventory Group Name"用户组,点击"Next"按钮
- 确认 database 相关的用户组，第二行选择 oper 用户组，点击"Next"按钮
  - 如果出现缺少pdksh-5.2.14 忽略即可.新的oracle都使用ksh包了,这个安装了就好
- 点击"Install"按钮，开始安装
- 等待安装完成(会提示登录root,执行脚本)
- 用 root 用户先执行orainstRoot.sh脚本，完成之后再用 root 用户执行 root.sh 脚本
- 点击"OK"按钮
- 点击"Close"按钮 --完成

#### 创建监听

使用 oracle 用户执行 netca 命令创建监听
- 选择"Listener configuration",点击"Next"按钮
- 选择"Add",点击"Next"按钮
- Listener name(可默认) 点击"Next"按钮
- 选择tcp协议.点击"Next"按钮
- 选择"Use the standard port number of 1521",点击"Next"按钮
- 选择"No",点击"Next"按钮
- 点击"Next"按钮
- 点击"Finish"按钮

#### 创建数据库

使用 oracle 用户执行 dbca 命令创建数据库

- 点击"Next"按钮 
- 选择"Create a Database" ，点击“Next” 按钮
- 选择"General Purpose or Transaction Processing"类型。生成环境按需求选择,一般选择"Custom Database"类型。 点击"Next"按钮 
- 输入"Golbal Database Name","SID Prefix":db11g 点击"Next"按钮
- 不勾选"Configure Enterprise Manager",点击"Next"按钮
- 勾选use the same ...输入 sys,system 统一密码:oracle 点击"Next"按钮
- 提示密码不符合 Oracle 推荐要求，忽略，点击"Yes"按钮
- 选择"Storage Type"为"File System"选择"使用 Oracle-Managed Files",在"Database File Location"输入:{ORACLE_BASE}/oradata 点击"Next"按钮
- 不勾选"Specify Fast Recovery Area"和"Enable Archiving",点击"Next"按钮
- 把复选框都去掉勾,点击"Next"按钮
- 在"Memory"选项卡选择"Typical" 自动分配内存
- 在"调整内存"选项卡中，设置最大进程数为500
- 在"Character Sets"选项卡选择"Choose from the list of character sets", 选择 “ZHS16GBK”，“Default Territory”选项卡选择 China，点击“Next”按钮
- 将重做日志组调整为5组，每组2个大小为128m 的重做日志文件，点击“下一步”
- 勾选create database 和 Generate Database Create Scripts  点击"Finish"按钮
- 点击"OK"按钮
- 脚本创建完成，点击"OK"按钮
- 点击"Exit"按钮退出，至此，数据库创建完成。

#### 其他配置

##### 取消密码限制

```
sqlplus “/as sysdba”
SQL> ALTER PROFILE DEFAULT LIMIT COMPOSITE_LIMIT UNLIMITED;
ALTER PROFILE DEFAULT LIMIT SESSIONS_PER_USER UNLIMITED;
ALTER PROFILE DEFAULT LIMIT CPU_PER_SESSION UNLIMITED;
ALTER PROFILE DEFAULT LIMIT CPU_PER_CALL UNLIMITED;
ALTER PROFILE DEFAULT LIMIT LOGICAL_READS_PER_SESSION UNLIMITED;
ALTER PROFILE DEFAULT LIMIT LOGICAL_READS_PER_CALL UNLIMITED;
ALTER PROFILE DEFAULT LIMIT IDLE_TIME UNLIMITED;
ALTER PROFILE DEFAULT LIMIT CONNECT_TIME UNLIMITED;
ALTER PROFILE DEFAULT LIMIT PRIVATE_SGA UNLIMITED;
ALTER PROFILE DEFAULT LIMIT FAILED_LOGIN_ATTEMPTS UNLIMITED;
ALTER PROFILE DEFAULT LIMIT PASSWORD_LIFE_TIME UNLIMITED;
ALTER PROFILE DEFAULT LIMIT PASSWORD_REUSE_TIME UNLIMITED;
ALTER PROFILE DEFAULT LIMIT PASSWORD_REUSE_MAX UNLIMITED;
ALTER PROFILE DEFAULT LIMIT PASSWORD_LOCK_TIME UNLIMITED;
ALTER PROFILE DEFAULT LIMIT PASSWORD_GRACE_TIME UNLIMITED;
```

##### 关闭数据库审计

```
1、查看审计功能是否开启
su – oracle
sqlplus “/as sysdba”
SQL> show parameter audit_trail
NAME          TYPE     VALUE
-------------------- ----------- ------------------------------
audit_trail     string      DB
说明：VALUE值为DB，表面审计功能为开启的状态

2、关闭oracle的审计功能
SQL> alter system set audit_trail=FALSE scope=spfile;
System altered.

3、重启数据库
SQL> shutdown immediate;
SQL> startup;
 
4、验证审计是否已经被关闭
SQL> show parameter audit_trail
NAME      TYPE       VALUE
------------- ----------- ------------------------------
audit_trail   string      FALSE
说明：VALUE值为FALSE，表面审计功能为关闭的状态
lsnrctl status   监听状态查看
SQL> show user --显示当前连接用户 
SQL> show error　　 --显示错误
sqlplus /nolog       SQL>connect / as sysdba ;
查看当前的所有数据库: select * from v$database;   select name from v$database;
进入test数据库：database test;  查看所有的数据库实例：select * from v$instance;
更改数据库用户的密码：(将sys与system的密码改为test.)
alter user sys indentified by test;
alter user system indentified by test;
```
