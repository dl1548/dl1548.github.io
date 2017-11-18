---
title: ansible基础
date: 2017-02-25 12:30:45
tags: ansible
categories:
     - 运维工具
     - 自动化
copyright: true
---
<img src="/images/ansible-logo.jpg" width = "30%" height = "20%" alt="ansble-logo" align=center />
#### 前言
**Ansible is Simple IT Automation——简单的自动化IT工具，可以实现 批量系统配置、批量程序部署、批量运行命令等功能，简而言之，就是** <span style='color:red'> **分布式集中管理工具**</span>， **通俗的讲就是批量在远端服务器上执行命令。其实，ansible自身不具备部署能力的，只是提供框架，其核心为模块**
<!--more-->

#### 什么是ansible?

![ansible架构.JPG](http://upload-images.jianshu.io/upload_images/2511748-ee48dcc8af53cf00.JPG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



| 五大部分               | 功能                             |
| :----------------- | :----------------------------- |
| connection plugins | 远程连接插件                         |
| hosts              | 定义管理主机或主机组                     |
| modules            | 包含各个核心模块及自定义模块                 |
| Plugin             | 完成模块功能的补充，如日志插件、邮件插件等          |
| Playbook           | ansible的任务配置文件，将多个任务定义在剧本中进行管理 |



​    

​    

​    

#### ansible 的工作流程





![ansible流程.JPG](http://upload-images.jianshu.io/upload_images/2511748-1001b5832ebba246.JPG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)







---

​    

​    

​    

#### 安装ansible

```shell
#配置源  ansible默认不在yum仓库中
rpm -iUvh http://ftp.jaist.ac.jp/pub/Linux/Fedora/epel//7/x86_64/e/epel-release-7-9.noarch.rpm

#此源主要是为了安装PyYAML
wget http://mirrors.163.com/.help/CentOS7-Base-163.repo          
mv CentOS7-Base-163.repo /etc/yum.repos.d/

yum -y install ansible

ansible --version
#可查看当前ansible版本
```

#### 配置登录

>   ansible 使用ssh登录，所以主奴之间要配置密钥进行认证，这样才能开始正常工作

```shell
ssh-keygen -t rsa #回车
#将会生成密钥/root/.ssh/id_rsa.pub

ssh-copy-id -i root@ipaddress 
#公钥将会被cp到各个ipaddress节点

#至此 已经实现了master与各节点的连接
```

#### 定义Ansible的节点清单

```
vim /etc/ansible/hosts
[testgroup]-----#服务器组的名字，方便统一管理，划分和命名要有规划
192.168.1.XX----#组内节点的地址
192.168.1.XXX
等等

[websever]
....
[DBserver]
....
```





#### 简单的远程操作

**通过执行who，查看服务器登录信息**

```shell
ansible testgroup -m command -a 'who' #组
ansible all -m command -a 'who'   # 所有
ansible 192.168.1.XX -m command -a 'who'  #单个ip

##
192.168.247.152 | SUCCESS | rc=0 >>
root     :0           2017-02-15 22:33 (:0)
root     pts/0        2017-02-15 22:34 (:0)
root     pts/1        2017-02-18 13:08 (192.168.247.1)
root     pts/2        2017-03-07 22:52 (:0)
root     pts/3        2017-03-07 22:54 (192.168.247.156)


# 以ashin用户身份ping .134
ansible 192.168.1.134 -m ping -u zili

# 以用户zili身份使用sudo来ping 组testgroup
# -K是输入root密码
ansible v1 -m ping -u zili --sudo -K
```



#### 定义变量

##### 定义主机变量

```YAML
    [web]
    192.168.247.152  http_port=80
    ...............  http_port=303
    [mysql]
    192.168.247.152
    ...
    
    #组名以及ip根本自己需求定义
    #主机指定变量，以便后面供palybooks配置使用。
    #定义两台web服务器的apache参数http_port，可以让两台服务器产生的apache配置文件httpd.conf差异化
    
```

##### 定义组变量

```yaml
    [web]
    192.168.247.152
    [web:vars]
    http_port=80
    
    #组变量的作用域是覆盖组所有成员，通过定义一个新块，块名由组名+ ":vars"组成。
```

##### 嵌套组

```yaml
    [web]
    192.168.247.152
    [mysql]
    192.168.247.152
    ...
    [nested：children]
    web
    mysql
    [nested：vars]
    ntp_server=s1b.time.edu.cn   
    
##
嵌套组定义一个新块，块名由 组名+":chilren" 组成。
同是嵌套组也可以定义组变量，作用域是嵌套组里的所有组,  
嵌套组只能在/usr/bin/ansible-playbook中，
在/usr/bin/ansible中不起作用，下面会介绍playbook
```

##### 分离主机和组特定数据

>   为更好的规范定义的**主机与组变量**，我们实际是不会在hosts里直接写的var，将定义的主机名与组变量单独剥离出来放到指定的文件中，将采用YAML格式存放
全局的变量放在group_vars/all中，局部变量放在group_vars/x中，特定的host使用特定的变量可以使用host_vars/x，子group中的变量会覆盖上级变量，hosts变量总是覆盖groups变量
存放位置规定："/etc/ansible/group\_vars/名"和"/etc/ansible/host\_vars/主机名"分别存放指定组名或主机名定义的变量，如
`/etc/ansible/group\_vars/mysql.yml`
`/etc/ansible/host\_vars/192.168.11.1.yml`
使用变量要用jinja语法去引用

```yaml
cat mysql.yml

   ---
    ntp_server: s1b.time.edu.cn
    database_server: 192.168.247.152

##
规范变量名字，是因为，ansible会自动加载这目录下的变量，
否则无法调用，当然也有解决不放此目录的方法
```
例如
```
[root@master ansible]# tree
├── create_user.yml
├── group_vars
│   └── t1.yml
├── hosts

[root@master ansible]# cat hosts
[t1]
10.1.27.24

所以
├── group_vars
│   └── t1.yml  #他的内容就是t1的变量

[root@master ansible]# cat group_vars/t1.yml 
---
user: ansibleTest1

[root@master ansible]# cat create_user.yml 
# create user
---
- name: create user
  hosts: t1
  user: root
  tasks:
  - name: useradd {{ user }}  #引用t1变量
    user: name="{{ user }}"

返回结果如下
[root@master ansible]# ansible-playbook create_user.yml 

PLAY [create user] ***********************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************
ok: [10.1.27.24]

TASK [useradd ansibleTest1] **************************************************************************************************
changed: [10.1.27.24]

PLAY RECAP *******************************************************************************************************************
10.1.27.24                 : ok=2    changed=1    unreachable=0    failed=0   

```


#### 命令参数

>    ansible <host-pattern> [options]  

```basic
    -m MODULE_NAME, --module-name=MODULE_NAME     要执行的模块，默认为 command  
    -a MODULE_ARGS, --args=MODULE_ARGS      模块的参数  
    -u REMOTE_USER, --user=REMOTE_USER ssh      连接的用户名，默认用 root，ansible.cfg 中可以配置
    -k, --ask-pass      提示输入 ssh 登录密码，当使用密码验证登录的时候用     
    -s, --sudo      sudo 运行
    -U SUDO_USER, --sudo-user=SUDO_USER     sudo 到哪个用户，默认为 root
    -K, --ask-sudo-pass     提示输入 sudo 密码，当不是 NOPASSWD 模式时使用
    -B SECONDS, --background=SECONDS       run asynchronously, failing after X seconds(default=N/A)
    -P POLL_INTERVAL, --poll=POLL_INTERVAL      set the poll interval if using
    -B (default=15)
    -C, --check     只是测试一下会改变什么内容，不会真正去执行
    -c CONNECTION   连接类型(default=smart)
    -f FORKS, --forks=FORKS     fork 多少个进程并发处理，默认 5
    -i INVENTORY, --inventory-file=INVENTORY      指定hosts文件路径默认 default =/etc/ansible/hosts
    -l SUBSET, --limit=SUBSET       指定一个 pattern，对<host_pattern>已经匹配的主机中再过滤一次
    --list-hosts        只打印有哪些主机会执行这个 playbook 文件：不是实际执行该 playbook
    -M MODULE_PATH, --module-path=MODULE_PATH       要执行的模块的路径，默认为/usr/share/ansible/
    -o, --one-line      压缩输出，摘要输出
    --private-key=PRIVATE_KEY_FILE      私钥路径
    -T TIMEOUT, --timeout=TIMEOUT   ssh 连接超时时间，默认 10 秒
    -t TREE, --tree=TREE            日志输出到该目录，日志文件名会以主机名命名
    -v, --verbose   verbose mode (-vvv for more, -vvvv to enable connection debugging)
```
#### Pattern
可以直接指定ip或hosts中的组名，同时指定多个组或者多个`ip`使用`:`分割
```
ansible group1:group2 -m ping
ansible ip1:ip2 -m ping

#all  或者 *  代表全部
ansible all -m ping

# 感叹号 ! 表示非
g1:!g2   #表示在g1分组中，但是不在g2中的hosts

# &符号表示交集
g1:&g2  #表示在g1分组中，也在g2中的hosts

#使用下标
g1[2]  #组的第三个
g1[0:3] #组的前四个
```



#### 常用模块

##### copy模块

```BASH
目的：把主控端/root目录下的a.sh文件拷贝到到指定节点上  
命令：ansible 192.168.247.152 -m copy -a 'src=/root/a.sh dest=/tmp/ owner=root group=root mode=0755'
```

##### file模块

```BASH
目的：更改指定节点上/tmp/t.sh的权限为755，属主和属组为root  
命令：ansible all -m file -a "dest=/tmp/t.sh mode=755 owner=root group=root"
```

##### cron模块

```BASH
    目的：在指定节点上定义一个计划任务，每隔3分钟到主控端更新一次时间  
    命令：ansible all -m cron -a 'name="custom job" minute=*/3 hour=* day=* month=* weekday=* job="/usr/sbin/ntpdate 192.168.247.152"'
```

##### group模块

```BASH
目的：在所有节点上创建一个组名为nolinux，gid为2014的组  
命令：ansible all -m group -a 'gid=2014 name=nolinux'
```

##### uesr模块

```bash
目的：在所有节点上创建一个用户名为nolinux，组为nolinux的用户  
命令：ansible all -m user -a 'name=nolinux groups=nolinux state=present'
删除用户  
命令：ansible all -m user -a 'name=nolinux state=absent remove=yes'
```

##### yum模块

```BASH
目的：在指定节点上安装 apache 服务  
命令：ansible all -m yum -a "state=present name=httpd"
#state=latest 安装最新版本
```

##### shell模块

```SHELL
目的：在指定节点上安装 apache 服务  
命令：ansible testgroup -m shell -a 'yum -y install httpd'
```

##### command模块

```
目的：在指定节点上运行hostname命令
命令：ansible 192.168.247.152 -m command -a 'hostname'
```

##### raw模块

```
目的：在192.168.247.152节点上运行ifconfig命令
命令：ansible 192.168.247.152 -m raw-a 'ifconfig|eth0'
```

##### script模块

```SHELL
目的：在指定节点上执行/root/a.sh脚本(该脚本是在ansible主控端)  
命令：ansible 10.1.1.113 -m script -a '/root/a.sh'
```

##### command,script,shell,raw的区别

>   思考：四者有何区别？

 command模块 [执行远程命令]
```
ansible client -m command -a "uname -n" -s
```
script模块 [在远程主机执行主控端的shell/python脚本]
 ```
ansible client -m script -a "/soft/ntpdate.py" -s
 ```

 shell模块 [执行远程主机的shell/python脚本]
 ```
ansible client -m shell -a "/soft/file.py" -s
```
raw模块 [类似于command模块、支持管道传递]
 ```
ansible client -m raw -a "ifconfig eth0|sed -n 2p|awk '{print \$2}'" -s
  ```
    

##### service模块

```SHELL
目的：启动指定节点上的 httpd 服务，并让其开机自启动  
命令：ansible 192.168.247.152 -m service -a 'name=httpd state=restarted enabled=yes'
```

##### ping模块

```SHELL
目的：检查指定节点机器是否还能连通  
命令：ansible 192.168.247.152 -m ping
```

##### get_url

```
目的：下载百度下的图标文件到节点的/tmp文件下
命令：ansible testgroup -m get_url -a 'url=https://www.baidu.com/favicon dest=/tmp'
#结果为error.html，但是证明了模块是可用的
```

##### stat模块

```
目的：获取远程文件状态信息，包括atime、ctime、mtime、md5、uid、gid等信息
ansible web -m stat -a 'path=/etc/sysctl.conf'
```

##### template模块

```
template使用了Jinja2格式作为文件模版，进行文档内变量的替换的模块。它的每次使用都会被ansible标记为”changed”状态。
```

#### 模块参数
 | 参数名    | 是否必须 | 默认值  | 选项     | 说明|
 | ---- | ---- | ---- | ---- | ----|
 | backup | no   | no   | yes/no | 建立个包括timestamp在内的文件备份，以备不时之需.            |
 | dest   | yes  |      |        | 远程节点上的绝对路径，用于放置template文件。               |
| group  | no   |      |        | 设置远程节点上的的template文件的所属用户组                |
| mode   | no   |      |        | 设置远程节点上的template文件权限。类似[Linux](http://www.2cto.com/os/linux/)中*chmod*的用法 |
 | owner  | no   |      |        | 设置远程节点上的template文件所属用户                   |
 | src    | yes  |      |        | 本地Jinjia2模版的template文件位置                 |

#### 模块参数案例
把/mytemplates/foo.j2文件经过填写参数后，复制到远程节点的/etc/file.conf，文件权限相关略过
```
 - template: src=/mytemplates/foo.j2 dest=/etc/file.conf owner=bin group=wheel mode=0644
```
 跟上面一样的效果，不一样的文件权限设置方式
```
 - template: src=/mytemplates/foo.j2 dest=/etc/file.conf owner=bin group=wheel mode="u=rw,g=r,o=r"
```

 ```BASH
#详细说明 
 roles/templates/server.xml中的template文件关键部分如下：
 <user username="{{ admin_username }}" password="{{ admin_password }}" roles="manager-gui" />
 #当这个文件还没被template执行的时候，本地的admin_username及admin_password 都是变量状态。 
 #当playbook执行完template的时候，远程的admin_username*及admin_password 会变成变量所对应的值。

 #例 
 #前面的那个Playbook,如果我们在tomcat-servers设置了这两个变量如下：
 dmin_username: admin
 admin_password: adminsecret

#那么在执行这个Playbook前，对应的那个template文件（俗称模版），
将在本地保持{{ admin_username }}及{{ admin_password }}的状态。
在Ansible调用template模版执行的时候，这里将由Jinjia2从”tomcat-servers”读取对应的值，
然后替换掉模版里的变量，然后把这个替换变量值后的文件拷贝到远程节点。
#这个就是template的意义所在。
```

##### 更多模块
ansible-doc -l 查询





---





#### playbook的配置和使用

配置文件后缀名为.yml

##### 官网demo说明

```yaml
#这个是你选择的主机
- hosts: webservers
#这个是变量
  vars:
    http_port: 80
    max_clients: 200
#远端的执行权限
  remote_user: root
  tasks:
#利用yum模块来操作
  - name: ensure apache is at the latest version
    yum: pkg=httpd state=latest
  - name: write the apache config file
    template: src=/srv/httpd.j2 dest=/etc/httpd.conf
#触发重启服务器
    notify:
    - restart apache
  - name: ensure apache is running
    service: name=httpd state=started
#这里的restart apache 和上面的触发是配对的。这就是handlers的作用。相当于tag
  handlers:
    - name: restart apache
      service: name=httpd state=restarted
```

```YAML
#有的系统做了sudo限制，所以需要在playbook中开启权限，如下
- hosts: web
  remote_user: vic
  tasks:
    - service: name=nginx state=started
      sudo: yes
```

##### 脚本实例

```YAML
#create_user.yml
- name: create user
  hosts: testgroup
  user: root
  ##facts可以调用client的变量来使用，后面变量里会详细介绍
  gather_facts: false
  vars:
  - user: "usertest1"
  tasks:
  - name: create {{ user }}
    user: name="{{ user }}"
    
#返回结果如下
[root/] ]$ansible-playbook create_user.yml

PLAY [create user] *************************************************************

TASK [create usertest1] ********************************************************
ok: [192.168.247.152]

PLAY RECAP *********************************************************************
192.168.247.152            : ok=1    changed=0    unreachable=0    failed=0
```

###### 给脚本添加service的调用

```YAML
#create_user.yml
- name: create user
  hosts: testgroup
  user: root
  gather_facts: false
  vars:
  - user: "usertest1"
  tasks:
  - name: create {{ user }}
    user: name={{ user }}
  - name: start httpd
    service: name=httpd state=startd
  #添加了httpd服务的开启
  
  #返回结果如下 
PLAY [create user] *************************************************************

TASK [create usertest2] ********************************************************
ok: [192.168.247.152]
#可以注意到 TASK [service]显示已经开启了
TASK [service] *****************************************************************
changed: [192.168.247.152]

PLAY RECAP *********************************************************************
192.168.247.152            : ok=2    changed=1    unreachable=0    failed=0

```

###### 给脚本添加copy模块的调用

```yaml
#create_user.yml
- name: create user
  hosts: testgroup
  user: root
  gather_facts: false
  vars:
  - user: "usertest1"
  tasks:
  - name: create {{ user }}
    user: name={{ user }}
  - name: start httpd
    service: name=httpd state=startd
   #添加了httpd服务的开启
  - name: Copy file to client
    copy: src=/tmp/test.test dest=/tmp/
    #添加了copy服务的开启
    
 #返回结果如下
[root/ansible_yml] ]$ansible-playbook create_user.yml

PLAY [create user] ************************************************************

TASK [create usertest2] *******************************************************
ok: [192.168.247.152]

TASK [service] ****************************************************************
ok: [192.168.247.152]
#可以注意到 TASK [copy file to clent]已成功
TASK [copy file to clent] *****************************************************
changed: [192.168.247.152]

PLAY RECAP ********************************************************************
192.168.247.152            : ok=3    changed=1    unreachable=0    failed=0
```

**copy传送的时候，可能报错**

```
afewbug | FAILED >> {
    "checksum": "4ee72f7427050dcd97068734d35ca7b2c651bc88", 
    "failed": true, 
    "msg": "Aborting, target uses selinux but python bindings (libselinux-python) aren‘t installed!"
    
是因为ansible需要libselinux-python包。（被控端需要安装libselinux-python**）  
可以在copy前先调用yum模块，安装libselinux-python
```

###### template模板（支持jinja2）


```YAML
#create_user.yml
- name: create user
  hosts: testgroup
  user: root
  gather_facts: false
  vars:
  - user: "usertest1"
  - temp: temptest
  tasks:
  - name: create {{ user }}
    user: name={{ user }}
  - name: start httpd
    service: name=httpd state=startd
   #添加了httpd服务的开启
  - name: Copy file to client
    copy: src=/tmp/test.test dest=/tmp/
    #添加了copy服务的开启
  - name: template test
    template: src=/tmp/temp dest=/tmp/{{temp}} #{{temp}}变量来自vars
    #添加了template模板使用
    
#返回结果
[root/ansible_yml] ]$ansible-playbook create_user.yml

PLAY [create user] *************************************************************

TASK [create usertest2] ********************************************************
ok: [192.168.247.152]

TASK [start httpd] *************************************************************
ok: [192.168.247.152]

TASK [copy file to clent] ******************************************************
ok: [192.168.247.152]
#可以注意到 返回结果显示成功，去client相关目录即可看到文件
TASK [template test] ***********************************************************
changed: [192.168.247.152]

PLAY RECAP *********************************************************************
192.168.247.152            : ok=4    changed=1    unreachable=0    failed=0


##
#template模块可以引用变量到源文件
/tmp/temp
{{user}}
{{temp}}

#执行yml后

[root/ansible_yml] ]$ansible testgroup -m command -a 'cat /tmp/temptest'
192.168.247.152 | SUCCESS | rc=0 >>
##client返回的就是主机源文件中引入的变量
usertest1
temptest

```



###### 执行外部命令的模块

```YAML
#create_user.yml
- name: create user
  hosts: testgroup
  user: root
  gather_facts: false
  vars:
  - user: "usertest1"
  - temp: temptest
  tasks:
  - name: create {{ user }}
    user: name={{ user }}
  - name: start httpd
    service: name=httpd state=startd
   #添加了httpd服务的开启
  - name: Copy file to client
    copy: src=/tmp/test.test dest=/tmp/
    #添加了copy服务的开启
  - name: template test
    template: src=/tmp/temp dest=/tmp/{{temp}} #{{temp}}变量来自vars
    #添加了template模板使用
  - name: run shell
    shell: /usr/bin/ls /tmp/ || /bin/true
    #/bin/true 防中断
  - name: run  command
    command: mkdir /tmp/command-test
  #添加了两个执行外部命令模块shell和command

#返回结果如下
[root/ansible_yml] ]$ansible-playbook create_user.yml

PLAY [create user] *************************************************************

TASK [create usertest2] ********************************************************
ok: [192.168.247.152]

TASK [start httpd] *************************************************************
ok: [192.168.247.152]

TASK [copy file to client] *****************************************************
changed: [192.168.247.152]

TASK [template test] ***********************************************************
changed: [192.168.247.152]

TASK [shell~] ******************************************************************
changed: [192.168.247.152]

TASK [run this command] ********************************************************
changed: [192.168.247.152]
 [WARNING]: Consider using file module with state=directory rather than running
mkdir


PLAY RECAP *********************************************************************
192.168.247.152            : ok=6    changed=4    unreachable=0    failed=0


###
#1./usr/bin/...||/bin/true  前面失败的话/bin/true：返回true。防止中断，继续执行。类似的判断还有chenge_when参数
```

#### 变量功能

##### facts

>   一个常用的组件，可实现对远程自己系统信息的获取，比如：主机名，IP地址，操作系统，分区情况，硬件信息等，配合playbook使用，更加的灵活和个性化定制

`ansible ip/group -m setup`可以获取clients的facts信息

```BASH
[root~] ]$ansible testgroup -m setup
192.168.247.152 | SUCCESS => {
    "ansible_facts": {
        "ansible_all_ipv4_addresses": [
            "192.168.122.1",
            "192.168.247.152"
        ],
        "ansible_all_ipv6_addresses": [
            "fe80::20c:29ff:feb5:6e6"
        ],
        "ansible_architecture": "x86_64",
        "ansible_bios_date": "07/02/2015",
        "ansible_bios_version": "6.00",
        
        ......
```

**脚本中开启Facts功能**

```YAML
#create_user.yml
- name: create user
  hosts: testgroup
  user: root
  gather_facts: false #开启facts
  tasks:
  - name: template test
    template: src=/tmp/temp dest=/tmp/{{temp}} #{{temp}}变量来自vars

#返回结果，
[root/ansible_yml] ]$ansible-playbook create_user.yml

PLAY [create user] *************************************************************

TASK [setup] *******************************************************************
ok: [192.168.247.152]

TASK [template test] ***********************************************************
changed: [192.168.247.152]

PLAY RECAP *********************************************************************
192.168.247.152            : ok=2    changed=1    unreachable=0    failed=0

#查看
[root/ansible_yml] ]$ansible testgroup -m raw -a "ls /tmp | grep 192*"
192.168.247.152 | SUCCESS | rc=0 >>
[u'192.168.122.1', u'192.168.247.152']
Shared connection to 192.168.247.152 closed.
#IP地址有两个所以文件名很奇怪
```

**当然，我们也可以在主机的/tmp/temp下调用facts变量**

```yaml
#修改/tmp/temp文件
test
{{ansible_all_ipv4_addresses}}

#create_user.yml
- name: create user
  hosts: testgroup
  user: root
  gather_facts: false #开启facts
  tasks:
  - name: template test
    template: src=/tmp/temp dest=/tmp/factstest

#结果如下
[root/ansible_yml] ]$ansible-playbook create_user.yml

PLAY [create user] *************************************************************

TASK [setup] *******************************************************************
ok: [192.168.247.152]

TASK [template test] ***********************************************************
changed: [192.168.247.152]

PLAY RECAP *********************************************************************
192.168.247.152            : ok=2    changed=1    unreachable=0    failed=0

[root/ansible_yml] ]$ansible testgroup -m raw -a "cat /tmp/factstest"
192.168.247.152 | SUCCESS | rc=0 >>
test
[u'192.168.122.1', u'192.168.247.152']
Shared connection to 192.168.247.152 closed.
```

##### 自定义变量

>   如何facts的变量并不能满足需求的，就可以自定义facts模板来实现
>
>    
>
>   另外可以通过本地facts来实现，只需在client的/etc/ansible/facts.d目录定义JSON,INI或可执行的JSON输出，后缀名一定要用.fact，那么这些文件就可以作为本地的facts

######在client定义变量，供ansible主机使用

```BASH
[root@test1 ~]# mkdir /etc/ansible/facts.d -p
[root@test1 ~]# cd /etc/ansible/facts.d/
[root@test1 facts.d]# cat client.fact
[general]
name=zili
```

**ansible主机**

```JSON
[root/ansible_yml] ]$ansible 192.168.247.152 -m setup -a "filter=ansible_local" 192.168.247.152 | SUCCESS => {
    "ansible_facts": {
        "ansible_local": {      #本地facts
            "client": {       #文件名
                "general": {       #节点名
                    "name": "zili"    #key-value
                }
            }
        }
    },
    "changed": false
}
```

**那么就可以通过如下方式去调用自定义的facts变量**

`{{ansible_local.client.general.name}}`

```BASH
[root/ansible_yml] ]$ansible-playbook create_user.yml

PLAY [create user] *************************************************************

TASK [setup] *******************************************************************
ok: [192.168.247.152]

TASK [template test] ***********************************************************
changed: [192.168.247.152]

PLAY RECAP *********************************************************************
192.168.247.152            : ok=2    changed=1    unreachable=0    failed=0
#调用成功，并可看到client内容已变化
[root/ansible_yml] ]$ansible testgroup -m raw -a "cat /tmp/factstest2"          192.168.247.152 | SUCCESS | rc=0 >>
zili #
Shared connection to 192.168.247.152 closed.

```

###### 在操作主机定义变量，来控制client

思路就是在执行playbook的时候将本地的facts推送到client相关目录下

```YAML
      - name: create directory for ansible custom facts
        file: state=directory recurse=yes path=/etc/ansible/facts.d
      - name: install custom facts
        copy: src=/etc/ansible/host.fact dest=/etc/ansible/facts.d
      - name: re-read facts after adding custom fact
        setup: filter=ansible_local
  #如此相当于批量在client创建了facts变量
  #然后就可以主机调用了!
```

##### 注册变量

>   变量可以将一条命令的返回值进行保存，然后提供给playbook使用

```yaml
[root@ansible ansible]# cat user1.yml
- hosts: testgroup
  remote_user: root
  tasks:
  - shell: /usr/bin/foo
    register: z     #注册了一个foo\_resul变量，变量值为shell: /usr/bin/foo的运行结果;  
    ignore_errors: True  #ignore\_errors: True为忽略错误
  - shell: touch /tmp/LLL #当变量注册完成后，就可以在后面的playbook中使用了
    when: z.rc == 5 
#当条件语句when: z.rc == 5成立时，shell: touch /tmp/LLL命令才会执行  
```

```BASH
#可以注意到command是skipping的。因为返回值是127，所以client肯定还是没有创建LLL的
[root/ansible_yml] ]$ansible-playbook user1.yml

PLAY [testgroup] ***************************************************************

TASK [setup] *******************************************************************
ok: [192.168.247.152]

TASK [command] *****************************************************************
fatal: [192.168.247.152]: FAILED! => {"changed": true, "cmd": "/usr/bin/foo", "delta": "0:00:00.003488", "end": "2017-03-11 13:08:45.996549", "failed": true, "rc": 127, "start": "2017-03-11 13:08:45.993061", "stderr": "/bin/sh: /usr/bin/foo: No such file or directory", "stdout": "", "stdout_lines": [], "warnings": []}
...ignoring

TASK [command] *****************************************************************
skipping: [192.168.247.152]

PLAY RECAP *********************************************************************
192.168.247.152            : ok=2    changed=1    unreachable=0    failed=0

##所以我门修改z.rc的返回值为127在执行
#以为返回值是对的，所以执行了touch，warning是友情提示，最好用线管模块进行文件的操作
[root/ansible_yml] ]$ansible-playbook user1.yml

PLAY [testgroup] ***************************************************************

TASK [setup] *******************************************************************
ok: [192.168.247.152]

TASK [command] *****************************************************************
fatal: [192.168.247.152]: FAILED! => {"changed": true, "cmd": "/usr/bin/foo", "delta": "0:00:00.003539", "end": "2017-03-11 13:11:01.955975", "failed": true, "rc": 127, "start": "2017-03-11 13:11:01.952436", "stderr": "/bin/sh: /usr/bin/foo: No such file or directory", "stdout": "", "stdout_lines": [], "warnings": []}
...ignoring

TASK [command] *****************************************************************
changed: [192.168.247.152]
 [WARNING]: Consider using file module with state=touch rather than running touch


PLAY RECAP *********************************************************************
192.168.247.152            : ok=3    changed=2    unreachable=0    failed=0

[root/ansible_yml] ]$


```

#### 语句

##### 条件语句

>   playbook的执行结果取决于变量，不管是facts还是tasks结果赋值的，而变量的值可以依赖于其他变量，当然一会印象ansible的执行
>
>   有时候我们，想要跳过某些主机的执行步骤，比如，某些client不安装某个软件包，不清理垃圾等等
>
>   就要使用判断了

###### when

```yaml
- name: when
  hosts: testgroup
  remote_user: root
  gather_facts: true
  tasks:
  - name: shutdown centos
    command: /sbin/shutdown -t now
    when: ansible_hostname == 'test1'
#when返回bool值，为true是执行，false则不执行
```

**when 针对不同分支的二级处理**

```YAML
- name: when
  hosts: web
  remote_user: root
  gather_facts: true
  tasks:
  - command: /sbin/ip a
    register: result
    ignore_errors: True
  - command: /bin/something
    when: result|failed
  - command: /bin/something_else
    when: result|success
  - command: /bin/still/something_else
    when: result|skipped
    
# when: result|success"的意思为当变量result执行结果为成功
#将执行/bin/something_else命令，其他同理。
#其中success为Ansible内部过滤器方法，返回True代表命令运行成功。
```

##### 循环语句

```YAML
- name: whell
  hosts: testgroup
  remote_user: root
  gather_facts: true
  tasks:
  - name: "add user"
    user: name={{ item }} state=present groups=wheel
    with_items:
    - tiger1
    - tiger2
    
#创建用户的。with_items会自动循环执行上面的语句"user: name={{ item }} state=present groups=wheel"，循环的次数为with_items的元素个数。这里有2个元素，分别为tiger1、tiger2，会分别替换{{ item }}项
#等同于
- name: whell
  hosts: testgroup
  remote_user: root
  gather_facts: true
  tasks:
  - name: "add user tiger1"
    user: name=tiger1 state=present groups=wheel
  - name: "add user tiger2"
    user: name=tiger2 state=present groups=wheel
```

###### 循环元素支持列表

```YAML
#首先定义好列表 list.yml
packages_base:
  - [ 'vsftpd', 'vim' ]
packages_apps:
  - [[ 'mysql',httpd' ]]

#然后引入使用
- name: whell
  hosts: testgroup
  remote_user: root
  gather_facts: true
  var_files: 
  - /etc/ansible/list.yml
  tasks:
  - name: "install rpm"
    yum: name={{ item }} state=installed
    with_flattened: #此语句 用来循环定义好的列表
    - packages_base
    - packages_apps
```



##### handlers 和 include

>   当多个playbook涉及复用的任务列表时，可以将复用的内容剥离出来，写到独立的文件里，需要的地方include进来即可
>
>    
>
>   除了tasks之外，还有一个handlers的命令，handlers是在执行tasks之后服务器发生变化之后可供调用的handler

```YAML
- name: write the httpd config file
  hosts: testgroup
  remote_user: root
  gather_facts: true
  tasks:
  - name: write the httpd.conf to client
    template: src=/httpd.conf.j2 dest=/etc/httpd/conf/httpd.conf
    notify:    # 如果copy执行完之后/etc/httpd/conf/httpd.conf文件发送了变化，则执行
    - restart httpd  # 调用handler
    - include: playbook/tasks/httpd.yml
    
    handlers:
    - name: restart httpd #此处的标识必须和notify一样才可以引起触发
      service: name=httpd state=restarted
      
 #注意上面使用的- include: playbook/tasks/httpd.yml，看一下这个文件的内容
- name: ensure httpd is running
  service: name=httpd state=started
```



>   notify这个action可用于在每个play的最后被触发，这样可以避免多次有改变发生时每次都执行指定的操作，取而代之，仅在所有的变化发生完成后一次性地执行指定操作。
>
>    
>
>   Handlers 是由通知者进行 notify, 如果没有被 notify,handlers 不会执行。
>
>   Handlers 最佳的应用场景是用来重启服务,或者触发系统重启操作.除此以外很少用到了。


#### roles 使用
前面的所有都在一个文件内.还有一种方法可以进行更好的组织架构
使用roles
```
[root@master roles]# tree
.
├── test
│   └── tasks
│       └── main.yml
└── test.yml
test.yml为入口文件,每次执行它即可.他的内容如下

[root@master roles]# cat test.yml 
---
- hosts: all
  roles:
    - role: test
定义了主机/主机组,然后定义了要使用的roles,(也就是roles下的文件夹的名字)

test文件夹下定义了tasks,内有 **main.yml**  这个命名是规定好的.必须是main


main.yml 书写了tasks的任务.
[root@master roles]# cat test/tasks/main.yml 
---
- name: test role ping
  ping:

[root@master roles]# 

```
结果如下:
```
[root@master roles]# ansible-playbook test.yml 

PLAY [all] ************************************************************************************

TASK [Gathering Facts] ************************************************************************
ok: [10.1.27.28]
ok: [10.1.27.24]

TASK [test : test role ping] ******************************************************************
ok: [10.1.27.28]
ok: [10.1.27.24]

PLAY RECAP ************************************************************************************
10.1.27.24                 : ok=2    changed=0    unreachable=0    failed=0   
10.1.27.28                 : ok=2    changed=0    unreachable=0    failed=0   

```
roles有很多结构,ansible可以根据其进行解析.

```
├── defaults
├── files
├── handlers
├── meta
├── tasks
├── templates
└── vars
```
如果`roles/x/tasks/main.yml`存在,则自动将里面的tasks添加到play中。
如果`roles/x/handlers/main.yml`存在,则自动将里面的handlers添加到play中。
如果`roles/x/vars/main.yml`存在, 则自动将其中的variables添加到play中。
如果`roles/x/meta/main.yml`存在,则添加role的依赖关系roles中。
任何`copy`任务、`script`任务都可以引用`roles/x/files`中的文件，无论是使用绝对或相对路径都可以。
任何`template`任务都可以引用`roles/x/templates`中的文件，无论绝对或相对路径。
任何`include`任务都可以引用`roles/x/tasks/`中的文件，无论相对或绝对路径

具体可以参见文档：[http://docs.ansible.com/playbooks_intro.html](http://docs.ansible.com/playbooks_intro.html)

#### ansible和saltstack的对比

>   1、salt要安装agent , ansible通过ssh连接。
>
>   2、salt在server端要启进程；ansible不需要。
>
>   3、salt与ansible都有模块，可使用任意语言开发模块。
>
>   4、salt与ansible都使用yaml语言格式编写剧本。
>
>
>   ansible走的是ssh,所以它有认证以及加密码的过程，使得ansible非常慢，不适用于大规模环境（指上千台）

___