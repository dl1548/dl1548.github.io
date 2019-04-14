---
title: saltstack基础
date: 2017-03-25 18:21:55
tags: saltstack
categories:
    - 运维工具
    - 自动化
copyright: true
---


<img src="/images/salt-logo.png" width = "50%" height = "40%" alt="salt-logo" align=center />

#### salt简介

  SaltStack是一个服务器基础架构集中化管理平台，具备配置管理、远程执行、监控等功能，基于Python语言实现，结合轻量级消息队列（ZeroMQ）与Python第三方模块（Pyzmq、PyCrypto、Pyjinjia2、python-msgpack和PyYAML等）构建。

  通过部署SaltStack，我们可以在成千万台服务器上做到批量执行命令，根据不同业务进行配置集中化管理、分发文件、采集服务器数据、操作系统基础及软件包管理等，SaltStack是运维人员提高工作效率、规范业务配置与操作的利器。
<!--more-->


#### salt基本原理

>SaltStack 采用 C/S模式，server端就是salt的master，client端就是minion，minion与master之间通过ZeroMQ消息队列通信
>
>minion上线后先与master端联系，把自己的pub key发过去，这时master端通过salt-key -L命令就会看到minion的key，接受该minion-key后，也就是master与minion已经互信
>
>master可以发送任何指令让minion执行了，salt有很多可执行模块，比如说cmd模块，在安装minion的时候已经自带了，它们通常位于你的python库中，`locate salt | grep /usr/` 可以看到salt自带的所有东西。
>
>这些模块是python写成的文件，里面会有好多函数，如cmd.run，当我们执行`salt '*' cmd.run 'uptime'`的时候，master下发任务匹配到的minion上去，minion执行模块函数，并返回结果。master监听4505和4506端口，4505对应的是ZMQ的PUB system，用来发送消息，4506对应的是REP system是来接受消息的。

具体步骤如下

1.  Salt stack的Master与Minion之间通过ZeroMq进行消息传递，使用了ZeroMq的发布-订阅模式，连接方式包括tcp，ipc
2.  salt命令，将`cmd.run ls`命令从`salt.client.LocalClient.cmd_cli`发布到master，获取一个Jodid，根据jobid获取命令执行结果。
3.  master接收到命令后，将要执行的命令发送给客户端minion。
4.  minion从消息总线上接收到要处理的命令，交给`minion._handle_aes`处理
5.  `minion._handle_aes`发起一个本地线程调用cmdmod执行ls命令。线程执行完ls后，调用`minion._return_pub`方法，将执行结果通过消息总线返回给master
6.  master接收到客户端返回的结果，调用`master._handle_aes`方法，将结果写的文件中
7.  `salt.client.LocalClient.cmd_cli`通过轮询获取Job执行结果，将结果输出到终端。

#### 安装salt

>   导入salt 密钥

```bash
7版本
rpm --import https://repo.saltstack.com/yum/redhat/7/x86_64/latest/SALTSTACK-GPG-KEY.pub


6版本
rpm --import https://repo.saltstack.com/yum/redhat/6/x86_64/latest/SALTSTACK-GPG-KEY.pub


#新增 /etc/yum.repos.d/saltstack.repo
7 & 6版本

[saltstack-repo] 
name = RHEL / CentOS $ releasever的
SaltStack repo baseurl = https://repo.saltstack.com/yum/redhat/$releasever/$basearch/latest 
enabled = 1 
gpgcheck = 1 
gpgkey = https：// repo.saltstack.com/yum/redhat/$releasever/$basearch/latest/SALTSTACK-GPG-KEY.pub
```

>   安装 salt-minion, salt-master,或Salt components:

```bash
yum install salt-master
yum install salt-minion
yum install salt-ssh
yum install salt-syndic
yum install salt-cloud
```

#### 配置salt

##### master

>   一般使用默认就好   (/etc/salt/master)

```BASH
#指定master，冒号后有一个空格
master: 192.168.2.22
user: root

#-------以下为可选--------------
# salt运行的用户，影响到salt的执行权限
user: root
#s alt的运行线程，开的线程越多一般处理的速度越快，但一般不要超过CPU的个数
worker_threads: 10
# master的管理端口
publish_port : 4505
# master跟minion的通讯端口，用于文件服务，认证，接受返回结果等
ret_port : 4506
# 如果这个master运行的salt-syndic连接到了一个更高层级的master,那么这个参数需要配置成连接到的这个高层级master的监听端口
syndic_master_port : 4506
# 指定pid文件位置
pidfile: /var/run/salt-master.pid
# saltstack 可以控制的文件系统的开始位置
root_dir: /
# 日志文件地址
log_file: /var/log/salt_master.log
# 分组设置
nodegroups:
  group_all: '*'
# salt state执行时候的根目录
file_roots:
  base:
    - /srv/salt/
# 设置pillar 的根目录
pillar_roots:
  base:
    - /srv/pillar
```

启动master

```
systemctl start salt-master
systemctl enable salt-master
```

##### minion

(/etc/salt/minion)

```BASH
#指定master，冒号后有一个空格
master: 192.168.2.22
id: minion-01
user: root

#-------以下为可选--------------
# minion的识别ID，可以是IP，域名，或是可以通过DNS解析的字符串
id: 192.168.0.100
# salt运行的用户权限
user: root
# master的识别ID，可以是IP，域名，或是可以通过DNS解析的字符串
master : 192.168.0.100
# master通讯端口
master_port: 4506
# 备份模式，minion是本地备份，当进行文件管理时的文件备份模式
backup_mode: minion
# 执行salt-call时候的输出方式
output: nested 
# minion等待master接受认证的时间
acceptance_wait_time: 10
# 失败重连次数，0表示无限次，非零会不断尝试到设置值后停止尝试
acceptance_wait_time_max: 0
# 重新认证延迟时间，可以避免因为master的key改变导致minion需要重新认证的syn风暴
random_reauth_delay: 60
# 日志文件位置
log_file: /var/logs/salt_minion.log
# 文件路径基本位置
file_roots:
  base:
    - /etc/salt/minion/file
# pillar基本位置
pillar_roots:
  base:
    - /data/salt/minion/pillar
```

启动minion

```
systemctl start salt-master
systemctl enable salt-master
```



#### 添加key

>   master 端查看key

```BASH
[root@master salt]# salt-key 
Accepted Keys:
Denied Keys:
Unaccepted Keys:   #可看到 minion已经检测到，没有认证key
minion-01
Rejected Keys:

[root@master salt]# salt-key -a minion-01
The following keys are going to be accepted:
Unaccepted Keys:
minion-01
Proceed? [n/Y] y    #Y确认添加
Key for minion minion-01 accepted.  #添加成功
[root@master salt]# salt-key 
Accepted Keys:
minion-01
Denied Keys:
Unaccepted Keys:
Rejected Keys:
[root@master salt]#
```

##### salt-key常用参数

| -a   | 添加指定ID 的key |
| :--- | :---------- |
| -A   | 添加全部        |
| -R   | 拒绝全部        |
| -d   | 删除指定ID的     |
| -D   | 删除全部        |

##### 测试连通性

```BASH
[root@master salt]# salt 'minion-01' test.ping
minion-01:
    True   #返回结果表示成功
[root@master salt]# 

```

##### 简单服务的安装

```bash
[root/] ]$salt 'minion-01' pkg.install ftp  #解释
minion-01:
    ----------
    ftp:
        ----------
        new:
            0.17-67.el7
        old:
[root/] ]$

#去minion查看
[root@minion-01 tmp]# rpm -qa ftp
ftp-0.17-67.el7.x86_64

#salt 'minion-01' pkg.install ftp
#1.'*' 代表的是target是指在那些minion上操作
#2. 'pkg' 是一个执行模块,就像'test' 
#3.'install' 是执行模块下面的函数，像test下的ping
#4.'ftp' 是函数的参数(arg)，有的函数需要参数，有的不需要比如test.ping就不需要参数
```

```bash
 ##查看所有执行模块的doc
 salt 'minion' sys.doc
 ##查看test模块的帮助
 salt 'minion' sys.doc test  
 ##查看test.ping函数的帮助
 salt 'minion' sys.doc test.ping 
```

#### salt常用命令

##### salt

>   该命令执行salt的执行模块,通常在master端运行.常用命令

```bash
salt [option] '<target>' <function> [arguments]

#例如
salt 'minion-01' cmd.run 'ip addr'
```

##### salt-run

>   该命令执行runner(salt自带或者自定义的，)，通常在master端执行，比如经常用到的manage

```bash
salt-run [options] [runner.func]

#例如
salt-run manage.status   ##查看所有minion状态
salt-run manage.down     ##查看所有没在线minion
salt-run manged.up       ##查看所有在线minion
```

##### salt-key

>   密钥管理，通常在master端执行

```BASH
salt-key [options]
salt-key -L              ##查看所有minion-key
salt-key -a <key-name>   ##接受某个minion-key
salt-key -d <key-name>   ##删除某个minion-key
salt-key -A              ##接受所有的minion-key
salt-key -D              ##删除所有的minion-key
```

##### salt-call

>   该命令通常在minion上执行，minion自己执行可执行模块，不通过master下发job

```BASH
salt-call [options] <function> [arguments]
salt-call test.ping           ##自己执行test.ping命令
salt-call cmd.run 'ifconfig'  ##自己执行cmd.run函数
```

##### salt-cp

>   分发文件到minion上,不支持目录分发.运行在master

```bash
salt-cp [options] '<target>' SOURCE DEST
#例如
salt-cp '*' testfile.html /tmp
salt-cp 'test*' index.html /tmp/a.html

#如果minion 是windows端 默认/ 指的是 C:\   /test = C:\test
```

##### salt-master

````BASH
salt-master [options]
salt-master            ##前台运行master
salt-master -d         ##后台运行master
salt-master -l debug   ##前台debug输出
````

##### salt-minion

```BASH
salt-minion [options]
salt-minion            ##前台运行
salt-minion -d         ##后台运行
salt-minion -l debug   ##前台debug输出
```

#### 普通用户执行salt

两种方法

>   1: ACL(修改master)

```bash
    client_acl:
    monitor: #uonghu
     - test*: #权限
    - test.*
    dev:
     - service.*
    sa:
     - .*
#重启master
     
#给予目录和文件权限
chmod +r /etc/salt/master
chmod +x /var/run/salt
chmod +x /var/cache/salt
```

>   2 external_auth(修改master)

```bash
  pam:
    fred:
      - test.*
#重启master
     
#给予目录和文件权限
chmod +r /etc/salt/master
chmod +x /var/run/salt
chmod +x /var/cache/salt
```

使用Token不必每次都输入账号密码，使用external_auth每次都是需要密码的，这样多麻烦，这里引入了Token，它会保存一串字符到在当前用户家目录下.salt_token中，在有效时间内使用external_auth是不需要输入密码的，默认时间12hour，可以通过master配置文件修改

```
salt -T -a pam '*' test.ping
```

#### target

>   target也就是目标,目的.指定master命令应该对谁执行

-   正则匹配

```bash
[root@master /]# salt -E  'mini*' test.ping
minion-02:
    True
minion-01:
    True
```

-   列表匹配

```BASH
[root@master ~]# salt -L minion-01,minion-02 test.ping
minion-02:
    True
minion-01:
    True
```

-   grains匹配

```BASH
[root@master ~]# salt -G 'os:CentOs' test.ping
minion-02:
    True
minion-01:
    True
```

-   组匹配

```bash
#开启master 的default_include
vim /etc/salt/master.d/nodegroup.conf 
#写到master中也是这个格式
nodegroups:
 test1: 'L@test1,test2 or test3*'
 test2: 'G@os:CenOS or test2'

salt -N test1 test.ping   #-N指定groupname

在top file中使用nodegroups

'test1':
 - match: nodegroup     ##没s,匹配的是文件
 - webserver
```

```bash
[root@master ~]# salt -N nodegroups test.ping
minion-02:
    True
minion-01:
    True
#组需要在master中预先定义
```

-   复合匹配  `salt -C 'G@os:MacOS or L@Minion1'` 
-   Pillar匹配 `salt -I 'key:value' test.ping`
-   CIDR匹配 `salt -S '192.168.1.0/24' test.ping`

>   在top文件中匹配 grains

```BASH
'node_type:web':
  - match: grain         #没有s
  - webserver
```

>   top文件中使用jinja

```BASH
{% set self = grains['node_type'] %}
    - match: grain
- {{ self }}
```

+   一次在n个minion上执行

```bash
-b n
--batch-size n
#例：
salt '*' -b 5 test.ping
#5个5个的ping
```

#### 多master

>   >   2个master并不会共享Minion keys，一个master删除了一个key不会影响另一个
>
>   >   不会自动同步File_roots,所以需要手动去维护，如果用git就没问题了
>
>   >   不会自动同步Pillar_Roots，所以需要手工去维护，也可以用git
>
>   >   Master的配置文件也是独立的

```BASH
#安装 salt-master

#原master的密钥cp一份到新的master
scp /etc/salt/pki/master/master* newmaster:/etc/salt/pki/master/
#启动新的Master

#修改配置minion的配置
master:
  - master1
  - master2
#重启minion

#新master接受所有的key
salt-key -L
salt-key -A
```

#### YAML

>   语法风格

+   空格和TAB

    yaml两个空格为缩进, TAB不要使用!

+ 冒号: 和减号-

    : 和- 后面要跟上一个空格在写

+ 数字解析

    mode: 0644 会解析成为mode: 644 最好使用mode: (0644)

+ 简写

    ```bash
    vim:
      pkg.installed #第一个简写
      user.present #第二个简写.不被支持,因为不支持双简写

    #建议规范书写
    vim:
      pkg:
        - installed
      user:
        - present
        
    ```

#### Jinja

>   Jinja 基于Python模板引擎开发,saltstack默认使用yaml_jinja渲染器,渲染流程时先jinja在yaml解析.所以在开始解析yaml的时候可以使用jinja"偷个腥"

+   区分模板文件

在salt中,files和templates都使用file这个state模块.那么如何区分模板是什么文件呢.

```bash
  - templates: jinja
  
file.managed:
  - name: /tmp/test
  - source: salt://tmp/test
  - template: jinja
  - defaults:
    Server: {{ pillar['.....'] }}
  
```

+   jinja中使用grains

```
{{ grains['os'] }}
```

+   jinja中使用执行模块

```BAHS
{{ salt['network.hw_addr']('eth0') }}
```

+   jinja中使用Pillar

```
{{ pillar['apache']['PORT'] }}
```

**Jinja的逻辑关系**

```
{% if grains['os'] == 'RedHat' %}
apache: httpd
{% elif grains['os'] == 'Debian' %}
apache: apache2
{% endif %}
```

**更多使用自行研究**



#### salt常用模块和API

##### 查看支持的所有modules

```BASH
[root/] ]$salt 'minion-01' sys.list_modules
minion-01:
    - acl
...
```

##### salt.client调用API举例

**`[root/] ]$cd /usr/lib/python2.7/site-packages/salt/modules/`** 模块path

**API调用示例**

```python
[root/] ]$cat test.py 
#!/usr/bin/python
import salt.client
client = salt.client.LocalClient()

res = client.cmd('*','test.ping')
print res
[root/] ]$./test.py 
{'minion-02': True, 'minion-01': True}

##解释一下
#当我们调用salt.client.LocalClient的时候,其实就等于我们执行了 salt '*' test.ping

```

**API调用：**

```
client.cmd('*','file.remove',['/tmp/foo'])
```





>   salt  <target> sys.doc  module
>
>   可以查看模块支持那些命令

##### Archive

>   实现对系统曾经的压缩包调用支持gzip,gunzip.rar,tar,unrar,unzip等

```BASH
#采用gunzip解压sourcefile.txt.gz包
salt '*' archive.gunzip sourcefile.txt.gz

#采用gzip压缩sourcefile.txt文件
salt '*' archive.gzip sourcefile.txt
```

**API调用：**

```
client.cmd('*','archive.gunzip',['sourcefile.txt.gz'])
```



##### cmd

>   实现对远程命令的调用执行,(默认具备root权限!谨慎使用)

```BASH
#获取所欲被控主机的内存使用情况
salt '*' cmd.run 'free -m'

#在wx主机上运行test.py脚本，其中script/test.py存放在file_roots指定的目录（默认是在/srv/salt,自定义在/etc/salt/master文件中定义），
#该命令会做2个动作：首先同步test.py到minion的cache目录；起床运行该脚本
salt 'minion-01' cmd.script salt://script/test.py
```

**API调用：**

```
client.cmd('*','cmd.run',['free -m'])
```

##### cp

>   实现远程文件目录的复制,以及下载URL文件等操作

```BASH
#将被控主机的/etc/hosts文件复制到被控主机本地的salt cache目录（/var/cache/salt/minion/localfiles/）
salt '*' cp.cache_local_file /etc/hosts

#将主控端file_roots指定位置下的目录复制到被控主机/minion/目录下
salt '*' cp.get_dir salt://script/ /minion/

#将主控端file_roots指定位置下的文件复制到被控主机/minion/test.py文件(file为文件名)
salt '*' cp.get_dir salt://script/test.py /minion/test.py

#下载URL内容到被控主机指定位置(/tmp/index.html)
salt '*' cp.get_url http://www.slashdot.ort /tmp/index.html
```

**API调用：**

```
client.cmd('*','cp.get_file',['salt://script/test.py','/minion/test.py'])
```

 

##### cron

>   实现对minion的crontab控制

```BASH
#查看指定被控主机、root用户的crontab操作
salt 'minion-01' cron.raw_cron root

#为指定被控主机、root用户添加/usr/local/weekly任务zuoye
salt 'minion-01' cron.set_job root '*' '*' '*' '*' 1 /usr/local/weekly 

#删除指定被控主机、root用户crontab的/usr/local/weekly任务zuoye
salt 'minion-01' cron.rm_job root /usr/local/weekly 
```

**API调用：**

```bash
client.cmd('wx','cron.set_job',['root','*','*','*','*',1,'/usr/local/weekly'])
```

##### file

>   对minion的文件操作,包括文件读写,权限,查找校验

```BASH
#校验所有被控主机/etc/fstab文件的md5值是否为xxxxxxxxxxxxx,一致则返回True值
salt '*' file.check_hash /etc/fstab md5=xxxxxxxxxxxxxxxxxxxxx

#校验所有被控主机文件的加密信息，支持md5、sha1、sha224、shs256、sha384、sha512加密算法
salt '*' file.get_sum /etc/passwd md5

#修改所有被控主机/etc/passwd文件的属组、用户权限、等价于chown root:root /etc/passwd
salt '*' file.chown /etc/passwd root root

#复制所有被控主机/path/to/src文件到本地的/path/to/dst文件
salt '*' file.copy /path/to/src /path/to/dst

#检查所有被控主机/etc目录是否存在，存在则返回True,检查文件是否存在使用file.file_exists方法
salt '*' file.directory_exists /etc

#获取所有被控主机/etc/passwd的stats信息
salt '*' file.stats /etc/passwd

#获取所有被控主机/etc/passwd的权限mode，如755，644
salt '*' file.get_mode /etc/passwd

#修改所有被控主机/etc/passwd的权限mode为0644
salt '*' file.set_mode /etc/passwd 0644

#在所有被控主机创建/opt/test目录
salt '*' file.mkdir /opt/test

#将所有被控主机/etc/httpd/httpd.conf文件的LogLevel参数的warn值修改为info
salt '*' file.sed /etc/httpd/httpd.conf 'LogLevel warn' 'LogLevel info'

#给所有被控主机的/tmp/test/test.conf文件追加内容‘maxclient 100’
salt '*' file.append /tmp/test/test.conf 'maxclient 100'

#删除所有被控主机的/tmp/foo文件
salt '*' file.remove /tmp/foo
```

##### network

>   返回minion的主机信息

```bash
#在指定被控主机获取dig、ping、traceroute目录域名信息
salt 'minion-01' network.dig www.qq.com
salt 'minion-01' network.ping www.qq.com
salt 'minion-01' network.traceroute www.qq.com

#获取指定被控主机的mac地址
salt 'minion-01' network.hwaddr eth0

#检测指定被控主机是否属于10.0.0.0/16子网范围，属于则返回True
salt 'minion-01' network.in_subnet 10.0.0.0/16

#获取指定被控主机的网卡配置信息
salt 'minion-01' network.interfaces

#获取指定被控主机的IP地址配置信息
salt 'minion-01' network.ip_addrs

#获取指定被控主机的子网信息
salt 'minion-01' network.subnets
```

**API调用：**

```bash
client.cmd('minion-01','network.ip_addrs')
```

 ##### pkg

>   minion的程序包管理,如yum, apt-get等

```BASH
#为所有被控主机安装PHP环境，根据不同系统发行版调用不同安装工具进行部署，如redhat平台的yum，等价于yum -y install php
salt '*' pkg.install php

#卸载所有被控主机的PHP环境
salt '*' pkg.remove php

#升级所有被控主机的软件包
salt '*' pkg.upgrade
```

**API调用：**

```
client.cmd('*','pkg.remove',['php'])
```

##### status

```
salt '*' status.version
```

API

```python
import salt.client
client = salt.client.LocalClient()
client.cmd('*','status.uptime')
```

##### system

>   用来日常操作计算机

```BASH
system.halt        #停止正在运行的系统
system.init 3      #切换到字符界面，5是图形界面
system.poweroff
system.reboot
system.shutdown
```

##### systemd(service)

```
  service.available sshd            #查看服务是否可用
  service.disable <service name>    #设置开机启动的服务
  service.enable <service name>
  service.disabled <service name>   #查看服务是不是开机启动
  service.enabled <service name>
  service.get_disabled              #返回所有关闭的服务
  service.get_enabled               #返回所有开启的服务
  service.get_all                   #返回所有服务
  service.reload <service name>     #重新载入指定的服务
  service.restart <service name>    #重启服务
  service.start <service name>
  service.stop <service name>
  service.status <service name>
  service.force_reload <service name>  #强制载入指定的服务
```

**使用**

```BASH
[root@mail python]# salt '*' service.available sshdmonitor:    True

api调用:
>>> client.cmd('*','service.available',['sshd']){'monitor': True}
```





#### grains

>   服务器的一些静态信息，强调的是静态，就是不会变的东西，比如说os是centos，不会变化，除非重新安装系统

##### grains的使用

```BASH
#查询所有grains信息
[root@master salt]# salt 'minion-01' grains.items 
minion-01:
    ----------
    SSDs:
    biosreleasedate:
        09/21/2015
    biosversion:
        6.00
    cpu_flags:
        - fpu
        - vme
        - de
.....

#查询grains指定项
[root@master salt]# salt '*' grains.item os
minion-02:
    ----------
    os:
        CentOS
minion-01:
    ----------
    os:
        CentOS
[root@master salt]# 


[root@master salt]# salt -G 'os:CentOS' test.ping
minion-01:
    True

#对系统是CentOS的服务器进行ping测试操作
#os:CentOS ; 就是对应上面grains.items显示出来的os值是CentOs的对象进行匹配 

 
#对cpu架构是x86_64的服务器显示CPU的个数
salt -G 'cpuarch:x86_64' grains.item num_cpus
 
#对字典值的对象进行匹配
salt -G 'ip_interfaces:eno16777728:192.168.2.*' test.ping
```

**在SLS中用grains**

```BASH
# 在xxx.sls中使用grains
'os:CentOS':
    - match: grain
    - webserver
```



##### 自定义grains(两种方法)

**1 . minion端修改**  重启生效

>   修改配置文件 /etc/salt/minion  或者写在/etc/salt/grains中
>
>   打开 default_include: minion.d/*.conf   或者直接添加此命令 
>
>   在minion端的/etc/salt/minion.d/ 目录下新建并编辑.conf后缀文件

```BASH
grains: #如果是/etc/salt/grains中,不需此行
  roles:
    - webserver
  sex: boy  #名字：值
  age:      #名字：多个值
    - 33
    - 44
 # 重启生效
[root@master ~]# salt 'minion-01' grains.item age
minion-01:
    ----------
    age:
        - 33
        - 44
[root@master ~]# 
```

**2 . minion端修改 ** 同步之后生效

>   base目录（在/etc/salt/master中配置的file_roots项，默认在/srv/salt）下生成**_grains** 目录,新建文件,用python来写

编写文件,需要返回一个字典

```bash
 vim test1.py
def hello(): ##函数名字无所谓，应该是所有函数都会运行
    agrain = {}
    agrain['hello'] = 'lzl' 
    return agrain   ##返回这个字典


========================

#!/usr/bin/python
# -*- coding:utf-8 -*-
import os
def file():
    grains={}#初始化一个字典，
    file = os.popen('ulimit -n').read()
    grains['my_file']=file
    return grains


#注意文件赋予权限
chmod a+x .py
```

```BASH
#同步到各个minion中去
salt '*' saltutil.sync_all

#查看
[root/srv/salt/_grains] ]$salt 'minion-01' grains.item hello
minion-01:
    ----------
    hello:
        lzl
```

#### pillar

>   Pillar在salt中是非常重要的组成部分，利用它可以完成很强大的功能，它可以指定一些信息到指定的minion上，不像grains一样是分发到所有Minion上的，它保存的数据可以是动态的,Pillar以sls来写的，格式是键值
>
>   适用
>
>   1.比较敏感的数据，比如密码，key等
>
>   2.特殊数据到特定Minion上
>
>   3.动态的内容
>
>   4.其他数据类型

##### pillar基本使用

**查看所有**

```
salt '*' pillar.items
```

**查看某个**

```BASH
salt '*' pillar.item KEY
#可以取到更小粒度的
salt '*' pillar.get <key>:<key> 
```

##### 编写pillar

>   指定pillar_roots
>
>   默认是/srv/pillar/(可通过修改master配置文件修改),建立目录

**top.sls**

```BASH
base:           #指定环境
  '*':          #target
    - test1     #引用test1.sls 或者test1/init.sls
    
#通过分组名匹配，
base:
  group1:
    - match: nodegroup    #必须要有 - match: nodegroup  
    - webserver  

#通过grain模块匹配的示例
base:
  'os:CentOS':
    - match: grain   #必须要有- match: grain
    - webserver
    

```

**test1.sls**

```bash
name: test1
user: lzl
```

**刷新**  pillar数据

```
salt '*' saltutil.refresh_pillar
```

**查看结果**

```BASH
[root/srv/pillar] ]$salt 'minion-01' pillar.items
minion-01:
    ----------
    name:
        test1
    user:
        lzl
[root/srv/pillar] ]$

```

#### 在state中通过jinja使用pillar

默认state文件位置/src/salt/

```BASH
user.sls

{% for user, uid in pillar.get('users', {}).items() %}  
 ##pillar.get('users',{})可用pillar['users']代替，前者在没有得到值的情况下，赋默认值
{{user}}:
  user.present:
    - uid: {{uid}}
{% endfor %}
 

```

#### jinja配合grains 指定pillar数据

```BASH
{% if grains['os_family'] == 'RedHat' %}
apache: httpd
{% elif grains['os'] == 'CentOS' %}
apache: httpd
vim: vim
{% elif grains['os'] == 'Arch' %}
apache: apache
vim: vim
{% endif %}
```

#### 使用salt state

>   它的核心是写sls(SaLt State file)文件,sls文件默认格式是YAML格式，并默认使用jinja模板，jinja是根据django的模板语言发展而来的语言，简单并强大，支持for if 等循环判断。salt state主要用来描述系统，软性，服务，配置文件的状态，常常被称为配置管理！

>   通常state，pillar,top file会用sls文件来编写。state文件默认是放在/srv/salt中，它与你的master配置文件中的file_roots设置有关

##### 简单的state文件配置&介绍

```bash
#/srv/salt/apahce.sls

apache:           ##state ID，全文件唯一,如果模块没跟-name默认用的ID作为-name
 pkg:             ##模块
   #- name: apache ##函数参数，可以省略
   - installed    ##函数
 service:         ##模块
   - running      ##函数
  #- name: apache ##函数参数，这个是省略的，也可以写上
   - require:     ##依赖系统
     - pkg: apache  ##表示依赖id为apache的pkg状态
     

#声明一个叫apache的状态id,该id可以随意，最好能表示一定意思

#pkg代表的是pkg模块

#installed是pkg模块下的一个函数，描述的是状态，该函数表示apache是否部署，返回值为True或者False，为真时，表示状态OK，否则会去满足该状态(下载安装apache)，如果满足不了会提示error,在该模块上面省略了参数-name: apache,因为ID为apache,这些参数是模块函数需要的（可以去查看源码）

#service是指的service模块
#这个模块下主要是描述service状态的函数，running状态函数表示apache在运行，省略-name不在表述，-require表示依赖系统，依赖系统是state system的重要组成部分，在该处描述了apache服务的运行需要依赖apache软件的部署，这里就要牵涉到sls文件的执行，sls文件在salt中执行时无序(如果没有指定顺序，后面会讲到order)，假如先执行了service这个状态，它发现依赖pkg包的安装，会去先验证pkg的状态有没有满足，如果没有依赖关系的话，我们可以想象，如果没有安装apache，apache 的service肯定运行会失败的，我们来看看怎么执行这个sls文件:
     
salt '*' state.sls apache  

#在命令行里这样执行，.sls不写，如果在目录下，将目录与文件用’.’隔开，
#如： httpd/apache.sls –> httpd.apache

#或者
salt '*' state.highstate 
#前提是存在top.sls 去指定minion运行的是哪个文件
#top.sls
base:
  '*':
    - webserver
```

>   state.sls默认的运行环境是base环境，但是它并不读取top.sls（top.sls定义了运行环境以及需要运行的sls）
>
>   state.sls也可以指定读取哪个环境：state.sls  salt_env='prod' xxxx.sls，这个xxxx.sls可以不在top.sls中记录。
>
>   state.highstate: 这个是全局的所有环境，以及所有状态都生效。它会读取每一个环境的top.sls，并且对所有sls都生效。不在top.sls文件里面记录的sls则不会被执行；
>
>   

阅读后写的版本

```bash
webserver:
  pkg:
    - name: httpd
    - installed
  service:
    - name: httpd
    - running
    - reqire:
      -pkg: httpd

[root/srv/salt] ]$salt 'minion-02' state.sls webserver
minion-02:
----------
          ID: webserver
    Function: pkg.installed
        Name: httpd
      Result: True
     Comment: The following packages were installed/updated: httpd
     Started: 18:24:07.033564
    Duration: 65091.443 ms
     Changes:   
              ----------
              httpd:
                  ----------
                  new:
                      2.4.6-45.el7.centos
                  old:
              httpd-tools:
                  ----------
                  new:
                      2.4.6-45.el7.centos
                  old:
              mailcap:
                  ----------
                  new:
                      2.1.41-2.el7
                  old:
----------
          ID: webserver
    Function: service.running
        Name: httpd
      Result: True
     Comment: Started Service httpd
     Started: 18:25:12.142495
    Duration: 5599.171 ms
     Changes:   
              ----------
              httpd:
                  True

Summary
------------
Succeeded: 2 (changed=2)
Failed:    0
------------
Total states run:     2
[root/srv/salt] ]$

```

##### 较复杂的state

**/srv/salt/ssh/init.sls**

```bash
openssh-client:
  pkg.installed
/etc/ssh/ssh_config:
  file.managed:
    - user: root
    - group: root
    - mode: 644
    - source: salt://ssh/ssh_config
    - require:
      - pkg: openssh-client
#ssh/init.sls 意思是当执行 salt '*' state.sls ssh的时候其实就是执行init.sls
#第一行:文件名,全文件唯一,如果pkg等模块没跟- name 包名, 默认用的ID作为-name
#第二行: 简写,意思pkg下的installed函数
#第三行: ID 告诉minion下载的文件应该放哪里!
#第四行:简写
#第八行:source是告诉minion从哪里下载源文件!
#salt://ssh/ssh_config其实就是/srv/salt/ssh/ssh_config 前面/srv/salt这个路径和file_roots的配置有关
```

**/srv/salt/ssh/server.sls**

```BASH
include:
  - ssh
#include表示包含意思，就是把ssh/init.sls直接包含进来

openssh-server:
 pkg.installed

sshd:
  service.running:
    - require:
      - pkg: openssh-client
      - pkg: openssh-server
      - file: /etc/ssh/banner
      - file: /etc/ssh/sshd_config

/etc/ssh/sshd_config:
  file.managed:
    - user: root
    - group: root
    - mode: 644
    - source: salt://ssh/sshd_config
    - require:
      - pkg: openssh-server
/etc/ssh/banner:
  file:
    - managed
    - user: root
    - group: root
    - mode: 644
    - source: salt://ssh/banner
    - require:
      - pkg: openssh-server

```

>   此时的目录结构应该是

```
├── ssh
│   ├── banner
│   ├── init.sls
│   ├── server.sls
│   ├── ssh_config
│   └── sshd_config
```

**关于include**古官网的demo

```bash
include:
  - ssh.server
extend:
  /etc/ssh/banner:
    file:
      - source: salt://ssh/custom-banner
 
#包含ssh/server.sls,扩展/etc/ssh/banner，重新其source而其它的如user,group等不变，与include一致。

include:
  - apache
extend:
  apache:
  service:
    - watch:
      - pkg: mod_python
#把apache.sls包含进来，想apache-service是追加了依赖关系(watch也是依赖系统的函数).
```

#### 关于渲染器 render system

>   salt默认是用的yaml_jinja渲染器处理ss文件,会优先使用jinjia处理,然后传给yaml处理然后生成salt需要的python数据类型.

**apache/init.sls**

```bash
apache:
  pkg:installed:
    {% if grains['os'] == 'CentoOS' %}
    - name: httpd
    {% endif %}
  service.running:
    {% if grains['os'] == 'CentoOS' %}
    - name: httpd
    {% endif %}
    - watch:
      - pkg: apache
      
#简单的例子,使用jinja结合grains进行判断
```

**user/init.sls**

```BASH
{% set users = ['jerry','tom','gaga'] %}
{% for user in users %}
{{ user }}:
 user.present:
   - shell: /bin/bash
   - home: /home/{{ user }}
{% endfor %}

---------------------------

{% if salt['cmd.run']('uname -i') == 'x86_64' %}
hadoop:
 user.present:
   - shell: /bin/bash
   - home: /home/hadoop
{% elif salt['cmd.run']('uname -i') == 'i386' %}
openstack:
 user.present:
   - shell: /bin/bash
- home: /home/openstack
{% else %}
django:
 user.present:
   - shell: /sbin/nologin
{% endif %}
```

#### py渲染器

>   纯python写的sls文件.如果使用其他的渲染器,需要在文件开头声明,!py就是声明用的py渲染器,
>
>    py中可用的变量有**salt**,**grains**,**pillar**,**opts**,**env**,**sls**,前三个分别对应jinja里的salt,grains,pillar,**opts**是minion的配置文件的字典，**env**对应的是环境如base,**sls**对应的是sls的文件名

```PYTHON
#!py
import os
def run():
   '''add user hadoop'''
platform = os.popen('uname -a').read().strip()
if platform == 'x86_64':
   return {'hadoop': {'user': ['present',{'shell': '/bin/bash'}, {'home': '/home/hadoop'}]}}
elif platform == 'i386':
       return {'openstack': {'user': ['present', {'shell': '/bin/bash'}, {'home': '/home/openstack'}]}}
else:
   return {'django': {'user': ['present', {'shell': '/sbin/nologin'}]}}

#注意的是return的数据结构{ID: {module: [func, arg1,arg2,...,]}} 或 {ID: {module.func: [arg1,arg2,..,]}} 。表示的内容与“示例；salt字典”表达的相同
```

#### state的执行顺序

>   stata执行,也就是.sls文件的执行是无序的.为了保证每次的顺序是一致的,就加入了state order ,
>
>   先了解下高级数据(High Data)和低级数据(Low Data).
>
>   高级数据就是指编写的sls文件的数据
>
>   低级数据就是经过render和parser编译过的数据

```bash
[root~] ]$salt 'minion-01' state.show_highstate
minion-01:
    ----------
    webserver:
        ----------
        __env__:
            base
        __sls__:
            webserver
        pkg:
            |_
              ----------
              name:
                  httpd
            - installed
            |_
              ----------
              order:
                  10000
        service:
            |_
              ----------
              name:
                  httpd
            - running
            |_
              ----------
              -pkg:
                  httpd
              reqire:
                  None
            |_
              ----------
              order:
                  10001
[root~] ]$salt 'minion-01' state.show_lowstate
minion-01:
    |_
      ----------
      __env__:
          base
      __id__:
          webserver
      __sls__:
          webserver
      fun:
          installed
      name:
          httpd
      order:
          10000
      state:
          pkg
    |_
      ----------
      -pkg:
          httpd
      __env__:
          base
      __id__:
          webserver
      __sls__:
          webserver
      fun:
          running
      name:
          httpd
      order:
          10001
      reqire:
          None
      state:
          service
[root~] ]$

```

>   查看可知,里面有个order,这个是默认salt 会自动设置,从10000开始.可通过修改master `state_auto_order: False`来关闭

#### order的设定

+   include

>   被include的文件Order靠前,先执行

+   手动定义order

```
httpd:
  pkg:
    - installed
    - order: 1
#order的值越小,优先级越高.但是-1 是最后!
```

+   依赖关系系统

就是前面使用过的 - require 

#### 依赖关系系统 requisite system

>   我们已经使用过依赖关系系统了,就是定义状态和状态之间的依赖关系,常用的函数有 `require`和`watch` 以及他们的变种`require_in`和`watch-in`
>
>   四者有何区别?
>
>   require,watch是指依赖，require_in,watch_in是指被依赖

>   watch 常用于service,而且当依赖条件发生变化的时候会执行一些动作

```bash
/etc/httpd/httpd.conf:
  file:
    - managed
    - source: salt://httpd/httpd.conf
  pkg.installed
  service:
    - running
    - require:
      - pkg: httpd
    - watch:
      - file://etc/httpd/httpd.conf #当httpd.conf改变时，重启httpd服务
    
============================    

/etc/httpd/httpd.conf:
  file:
    - managed
    - source: salt://httpd/httpd.conf   
    - watch_in:
      - service: httpd
  httpd:
    pkg:
      - installed
      - require_in:
        - service: httpd
    service:
      - running
    
    
    
    
    
    
    
```

#### salt state多环境

>   针对不同的环境,应用不同state的file,比如开发,测试,生产等.
>
>   通过修改master对不同的环境应用不通过的目录

```BASH
#官方demo
Example:
  file_roots:
    base:
      - /srv/salt/
    dev:
      - /srv/salt/dev/services
      - /srv/salt/dev/states
    prod:
      - /srv/salt/prod/services
      - /srv/salt/prod/states
#file_roots 配置salt配置的存放目录, 其中base环境是必要的, 指定top.sls存放的位置.
#默认没指定环境时则从base目录获取文件
#其它则是一些自定义的, 可以通过环境变量指定.
#这样可以逻辑上隔离一些环境配置.
#每一个环境都可以定义多个目录, 优先级关系由定义目录的顺序决定.
file_roots:
  base:
    - /srv/salt/foo
    - /srv/salt/bar
#如果寻找 salt://file.sls, 如果都存在/srv/salt/foo/file.sls和/srv/salt/bar/file.sls, 则使用第一个找到的.    
```

另一个例子

```BASH
file_roots:
  base:
    - /srv/salt/prod
  qa:
    - /srv/salt/qa
    - /srv/salt/prod
  dev:
    - /srv/salt/dev
    - /srv/salt/qa
    - /srv/salt/prod
#/srv/salt/prod 里的配置是在三种环境下都可以, /srv/salt/qa 只在qa和dev环境下可用, /srv/salt/dev则只在dev环境下可用.
```

简答你的实施案例

```bash
#master配置
file_roots:
  base:
    - /home/base/
  dev:
    - /home/dev/
    - /home/base/
    
#base环境   
#/home/base
├── envtest.sls
└── top.sls

#cat /home/base/envtest.sls
envtest:
  cmd.run:
    - name: "echo '[base] env'"    

#dev环境
#/home/dev/
├── mytest.sls
└── top.sls

#cat /home/dev/mytest.sls
envtest:
  cmd.run:
    - name: "echo '[dev] env'"



##执行效果如下,如果不添加环境变量,则提示找不到文件
[root/srv/salt/dev] ]$salt 'minion-01' state.sls mytest  test=True
minion-01:
    Data failed to compile:
----------
    No matching sls found for 'mytest' in env 'base'
ERROR: Minions returned with non-zero exit code

#加上环境变量执行
[root/srv/salt/dev] ]$salt 'minion-01' state.sls mytest saltenv='dev' test=True
minion-01:
----------
          ID: mytest
    Function: cmd.run
        Name: echo dev-env
      Result: None
     Comment: Command "echo dev-env" would have been executed
     Started: 23:54:46.298421
    Duration: 0.422 ms
     Changes:   

Summary
------------
Succeeded: 1 (unchanged=1)
Failed:    0
------------
Total states run:     1
[root/srv/salt/dev] ]$
```

#### salt schedule(salt中的crontab)

>   周期性的执行一些函数,需要注意的是: 在minion上执行salt可执行模块里的函数,在master执行的是runner模块的函数.
>
>   共有三种方式:master minion  pillar

+   master端
+   minion端
+   pillar

>   一般而言,尤其是在minion端配置,基本不会用到的,主要还是一pillar为主

**修改top.sls**

```bash
#添加
  - schedule
```

**/srv/pillar/schedule.sls**

```BASH
schedule:
  test-job:
    function: cmd.run
    seconds: 10
    args:
      - 'date >> /date.log'
      
#没隔10S 在/目录的date.log文件中记录一条时间
```

```bash
salt "*" saltutil.refresh_pillar
#刷新pillar到minion

#回到minion 可以查看到
[root@minion-01 /]# ls
bin  boot  date.log  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
[root@minion-01 /]# cat date.log 
Fri Mar 24 02:27:40 CST 2017
Fri Mar 24 02:27:50 CST 2017
Fri Mar 24 02:28:00 CST 2017
....
```

#### salt ssh

>   salt-ssh 是 0.17.0 新出现的一个功能.对于有些不能安装minion的机器,ssh不失为一种好的选择但是SSH并不能取代minion,salt的有些功能不支持ssh.而且走的是SSH 并不是ZeroMQ,所以速度会有所影响

```bash
#首先安装salt-ssh.
yum -y install salt-ssh
```

```BASH
[root~] ]$cat /etc/salt/roster #roster文件名和路径!
minion-01:
  host: 192.168.247.153
  user: root
  passwd: centos
minion-02:
  host: 192.168.247.154
  user: root
  passwd: centos
  sudo: True

#如果不给passwd的话,执行salt-ssh会提示输入密码
#普通用户给sudo权限
```

```BASH
#第一次使用记得加参数 -i 否则报错如下
[root~] ]$salt-ssh 'minion-01' test.ping
minion-01:
    ----------
    retcode:
        254
    stderr:
    stdout:
        The host key needs to be accepted, to auto accept run salt-ssh with the -i flag:
        The authenticity of host '192.168.247.153 (192.168.247.153)' can't be established.
        ECDSA key fingerprint is 16:f6:f5:49:24:9c:91:da:d7:02:58:a2:14:08:e4:15.
        Are you sure you want to continue connecting (yes/no)? 
        
#第一次运行 添加-i参数
[root~] ]$salt-ssh 'minion-01' test.ping -i
minion-01:
    True
[root~] ]$salt-ssh 'minion-01' test.ping
minion-01:
    True
[root~] ]$

```



#### Returners

略



#### 扩展salt



略
