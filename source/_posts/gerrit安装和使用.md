---
title: gerrit安装和使用
date: 2019-06-21 16:02:52
tags: gerrit
categories: 
    - 运维工具
    - git
copyright: true
---



个人记录
<!--more-->



#### 安装

##### 新建用户

```
# 创建专用账户
useradd gerrit
# 为账户设置密码
passwd gerrit
```

##### 安装java

```
略
$ java -version
java version "1.8.0_45"
Java(TM) SE Runtime Environment (build 1.8.0_45-b14)
Java HotSpot(TM) 64-Bit Server VM (build 25.45-b02, mixed mode)
```

##### 安装mysql

如果不打算使用`gerrit` 自带的H2数据库的话

```
安装-----略

# 创建db user
mysql> CREATE USER 'gerrit'@'%' IDENTIFIED BY 'password';

#创建要用的DB
mysql> CREATE DATABASE GerritDB;

#授权
mysql> GRANT ALL ON GerritDB.* TO 'gerrit'@'%';
```



##### 安装git

```
# yum -y install git
```



##### 安装gerrit

```
# 切换用户
su - gerrit

# 下载
wget https://gerrit-releases.storage.googleapis.com/gerrit-2.15.5.war

#安装  review_site 为定义的安装目录，默认安装在当前目录下（家目录）
 java -jar gerrit-2.15.war init -d review_site

```

一路回车默认安装 (其中的认证方式处改为 HTTP)

```
Authentication method          [OPENID/?]: HTTP
```

另外，建议这几个插件也顺便安装上，全部选y

```
Installing plugins.
Install plugin commit-message-length-validator version v2.15.5 [y/N]?
Install plugin download-commands version v2.15.5 [y/N]?
Install plugin hooks version v2.15.5 [y/N]?
Install plugin replication version v2.15.5 [y/N]?
Install plugin reviewnotes version v2.15.5 [y/N]?
Install plugin singleusergroup version v2.15.5 [y/N]?
```

文件夹授权

```
# chown -R gerrit:gerrit review_site
```

修改配置

```
$ vim review_site/etc/gerrit.config

[gerrit]
	basePath = git #指定被gerrit管理git库存放位置：review_site_project/git/
	serverId = 1048a788-9fb9-4d7d-8da4-aa44be83aa7a
	canonicalWebUrl = http://192.168.1.66:8088/  #指定web访问gerrit的网址，填自己的ip和端口号
[database] 
	type = h2 	#指定gerrit所默认数据库类型，h2 已经够用了 可以选用mysql，安装并创建gerrit账户
	database = /home/gerrit/review_site/db/GerritDB
[index]
	type = LUCENE
[auth]
	type = HTTP				#指定浏览器登录gerrit时的认证方式
[sendemail]
	smtpServer = localhost
[container]
    user = gerrit 			#指定gerrit所在机器的用户身份与上文创建的用户对应一致,可以是root
	javaHome = /usr/local/jdk1.8.0_45/jre
[sshd]
	listenAddress = *:29418                        #指定sshd服务监听的端口号
[httpd]
	listenUrl = http://*:8088/                        #指定http代理地址
[cache]
	directory = cache                        #缓存位置
```

启动

```
review_site/bin/gerrit.sh start
```



##### htpasswd

```
 #工具安装
yum -y install httpd-tools  

#生成密码文件
htpasswd -c /home/gerrit/review_site/etc/gerrit.password gerrit  #然后输入密码
```

注意，如果此时你是用`root`生成的密码文件，记得更改权限，粗暴点

```
chown -R gerrit:gerrit /home/gerrit
```



##### 安装gitweb

```
yum -y install gitweb
```

修改配置

```
/etc/gitweb.conf

# 更改路径为你的路径
our $projectroot = "/home/gerrit/review_site/git";
```

修改 /home/gerrit/review_site/etc/gerrit.config,追加内容

```
[gitweb]
        cgi = /var/www/git/gitweb.cgi
        type = gitweb
```



##### 安装nginx

```
# 安装依赖
yum -y install gcc-c++ pcre pcre-devel zlib zlib-devel openssl

安装 ----略
```

配置

```
# 添加basic认证，修改location

server {
        listen       80;
        server_name  192.168.1.66;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;
        auth_basic "Welcomme!";
        auth_basic_user_file /home/gerrit/review_site/etc/gerrit.password;
        
        location / {
            root   html;
            index  index.html index.htm;
            proxy_pass http://192.168.1.66:8088;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            #proxy_set_header X-Forwarded-For $remote_addr;
        }

```



##### 重启服务

```
/home/gerrit/review_site/bin/gerrit.sh restart
/usr/local/nginx/sbin/nginx -s reload 
```

 

访问 http://192.168.1.xxx 既可



#### 使用

` Projects –> List `能看到两个默认的项目`All-Projects`和 `All-Users`，这两个工程是两个基础的工程，我们新建的工程默认都是继承自`All-Projects`的权限



设置：

 `右上角-用户名--setting` - `Preferences`  偏好设置，`Show Change Number In Changes Table` 勾选

`右上角-用户名--setting` - `SSH Public Keys` 将客户机的`公钥`添加上

 `右上角-用户名--setting` - `HTTP Password`  生成一个访问密码，可用来访问API接口



##### 创建项目

老版UI

` Projects –> List ` 点进项目可查看到项目的一些配置，无需更改太多

`Require Change-Id in commit message: `  true ，其余可默认



点`access` 点 `edit` 可对项目进行单独的授权等

点`Add Reference` ，将Reference设置为 `refs/*`

点`Add Permission` ,选`read` 然后进行相应的赋权



##### 添加普通用户

添加普通用户的访问

```
# 密钥文件可提前收集好
cat ~/home/zili.pub | ssh gerrit gerrit create-account --full-name zili --email zili@zili.com --ssh-key - zili

此命令意味着 管理员把用户zili加入到了Anonymous Users用户组中，并且设置了他的全称，邮件以及公钥文件
```

然后我们就可以去

`People  - List Groups` 创建组，将人员划分到组内，同时 创建项目的时候就能添加相应的组了。



##### 克隆项目

```
git clone ssh://gerrit@192.168.1.66:29418/test && scp -p -P 29418 gerrit@192.168.1.66:hooks/commit-msg test/.git/hooks/


git config user.name test
git config user.email test@test.com

```



##### 修改push分支

```
git config remote.origin.push refs/heads/*:refs/for/*
```

当执行`push`命令时，将会推送到`refs/for/当前head所在的分支`上



然后我们做的所有操作都将会被gerrit记录下来了



##### review

略



#### api使用

安装 `pygerrit2`

```
from pygerrit2 import GerritRestAPI, HTTPBasicAuth
auth = HTTPBasicAuth('gerrit', '2X8XonAccDXFY7mLFfFcadHMuS1QIA2FHfPQyIXUvg')
#auth = HTTPBasicAuth('gerrit', 'gerrit')
print(auth)
rest = GerritRestAPI(url='http://192.168.1.66:8088', auth=auth)
changes = rest.get("/access/?project=All-Projects")

print(changes)


# 注意使用的是基本认证还是摘要认证，还有版本 2.14 前后区别,见官网
```





详见<https://github.com/dpursehouse/pygerrit2>