---
title: Nginx+tomcat+redis实现负载与session
date: 2017-02-20 00:52:11
tags: 
    - nginx
    - tomcat
categories: 
    - web
    - nginx
copyright: true
---

可实现nginx的负载,session共享,后台健康检测,以达到故障后实现快速切换
<!--more-->
安装需要准备的包

```
commons-pool2-2.2.jar
jedis-2.7.2.jar
tomcat-redis-session-manage-tomcat7.jar
#目前上面这些组件不支持tomcat8.0
apache-tomcat-7.0.75.tar.gz
jdk-8u45-linux-x64.tar.gz    #用以支持JAVA
nginx-1.7.8.tar.gz
nginx_upstream_check_module-master.zip   #后台健康监测插件，需要安装Nginx时编译进去
```

**YUM源为epel**

规划---(测试机的配置基本一致，本文只书写其一)

| IP              | 备注            |
| --------------- | ------------- |
| 192.168.247.151 | Nginx+Redis   |
| 192.168.247.152 | tomcat（test1） |
| 192.168.247.153 | tomcat（test2） |

>   192.168.247.151

##### 安装Nginx

为了支持Nginx的rewrite功能，首先安装pcre*模块

```
yum -y install pcre*
```

为了进行后台的健康检测，所以下载淘宝的检测插件，安装Nginx直接编译进去

/usr/local/src

**nginx_upstream_check_module-master.zip 和nginx-1.7.8.tar.gz**

```
unzip  nginx_upstream_check_module-master.zip
tar -zxvf nginx-1.7.8.tar.gz
cd nginx-1.7.8
./configure --prefix=/usr/local/nginx --with-http_ssl_module --with-http_spdy_module --with-http_stub_status_module --with-pcre --add-module=/usr/local/src/nginx_upstream_check_module-master
make
make install
```

注意：如果是Nginx安装后进行的编译

```
cd nginx-1.7.8
patch -p1 < ../nginx_http_upstream_check_module/check_1.7.?+.patch    #版本根据Nginx选择
./configure --prefix=/usr/local/nginx --with-http_ssl_module --with-http_spdy_module --with-http_stub_status_module --with-pcre --add-module=/usr/local/src/nginx_upstream_check_module-master
make     #千万不能  make install  不然就真的覆盖了
mv /usr/local/nginx/sbin/nginx /usr/local/nginx/sbin/nginx-1.7.0.bak
cp ./objs/nginx /usr/local/nginx/sbin/
/usr/local/nginx/sbin/nginx    #启动Nginx
```

###### 配置Nginx

/usr/local/nginx/conf/nginx.conf

```
http {
    include       mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    #access_log  logs/access.log  main;
    sendfile        on;
    keepalive_timeout  65;
    #开启负载均衡，指向后台tomcat集群
    upstream test {
        server 192.168.247.152:8080;
        server 192.168.247.153:8080;
        #开启健康检测机制
        check interval=3000 rise=2 fall=5 timeout=1000 type=http;
        check_http_send "HEAD /test HTTP/1.0\r\n\r\n";
        check_http_expect_alive http_2xx http_3xx;
        }
        
    server{
        listen  80;
        #server_name test.test.com; #设置域名
        #健康检测界面
        location = /nstatus {
            check_status;
            access_log off;
            allow all;
            }
        #测试页面 自己在tomcat上新建的
        location /test {
            proxy_pass http://test;
            proxy_set_header Host $host; 
            proxy_redirect off; 
            proxy_set_header X-Real-IP $remote_addr; 
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; 
           }
        }
}
```

启动Nginx 输入健康检测地址即可看到后台tomcat状态



##### 安装Redis  实现session共享

下载redis包官网[http://redis.io](http://redis.io/)

注：redis的test需要tcl的支持，所以可先检查下是否安装了tcl

```
yum - y install
wget http://download.redis.io/redis-stable.tar.gz
tar –zxvf redis-stable.tar.gz
cd redis-stable
make
```

完毕后 src下会多出几个文件

 redis-benchmark redis-check-aof redis-check-rdb redis-cli redis-sentinel  redis-server 

可手动将其复制到/usr/local/bin目录下，也可执行make install

此处选择make install

```
make install
```

注意：若此时执行redis-server –v (查看版本命令)，若提示redis-server command not found，则需要将/usr/local/bin目录加到环境变量，如何添加，此处不做详细介绍，可查看修改/etc/profile，(查看环境变量命令：echo $PATH)

**redis安装完毕**

###### 修改redis配置文件

创建redis目录用以存放redis 日志 数据库 进程

```
mkdir -p /var/redis/{data,log,run}
```

拷贝解压包下的redis.conf文件至/etc/redis

```
cp -p /usr/local/src/redis-stable/redis.conf /etc/redis.conf
vim /etc/redis.conf
```

```
port 6379
daemonize yes #开启后台进程
pidfile /var/redis/run/6379.pid
logfile /var/redis/log/redis.log
dbfilename dump.rdb
dir /var/redis/data  #数据库路径 默认是./
requirepass centos #设置密码为centos
#bind 127.0.0.1  默认是开启的，只允许本地登陆，所以，要不添加IP，要不给注释了 
```

启动redis

```
redis-server /etc/redis.conf
ps -aux | grep redis
redis-cli    #客户端连接，进入redis

AUTH centos  #密码认证
SHUTDOWN
exit

service redis start
```



###### 设置redis开机启动

```
cp -p /usr/local/src/redis-stable/utils/redis_init_script /etc/init.d/redis
ll    #看有没有执行权限
```

修改脚本的pid,conf等路径,添加开机启动权限

```
#开机启动
#chkconfig: 2345 90 10
#description: Redis is a persistent ket-value database
#added by zili on 20170218  

REDISPORT=6379
EXEC=/usr/local/bin/redis-server
CLIEXEC=/usr/local/bin/redis-cli

PIDFILE=/var/redis/run/6379.pid
CONF="/etc/redis.conf"
...
#如果是在生产环境，那么就规划好端口等，尽量使用变量去实现，加以区分
比如
PIDFILE=/var/redis/run/${REDISPORT}.pid
CONF="/etc/redis_${REDISPORT}.conf"
```

```
service redis star | stop #等均可使用了，不能使用就查看权限 ll
chkconfig redis on
```


> 192.168.254.152/153

##### 安装tomcat

tomcat 安装依赖JAVA的JDK 所以判断JDK是否安装并进行安装

```
rpm -qa | grep java
```

###### 删除openjdk

```
rpm -e --nodeps java-1.7....... # -e 删除 --nodeps强行删除
```

下载 jdk-8u45-linux-x64.tar.gz 解压 到   /usr/java/

配置全局变量 vim /etc/profile  在末尾添家，注意路径！特别是JDK文件名字

```
export JAVA_HOME=/usr/java/jdk1.8.0_45
export JRE_HOME=/usr/java/jdk1.8.0_45/jre
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib:$CLASSPATH
export PATH=$PATH:$JAVA_HOME/bin
```

```
java -version  #不报错，能看版本，安装成功
```



下载tomcat包 apache-tomcat-7.0.75.tar.gz  （注意redis组件目前不支持tomcat8.0）

以及三个是jar插件 commons-pool2-2.2.jar     jedis-2.7.2.jar    tomcat-redis-session-manage-tomcat7.jar

解压 tomcat安装包 到/usr/local/tomcat

```
tar -zxvf apache-tomcat-7.0.75.tar.gz 
mv apache-tomcat-7.0.75 /usr/local/tomcat
mv  3个插件  /usr/local/tomcat/lib
/usr/local/tomcat/bin/startup.sh
```

浏览器即可访问tomcat  默认端口8080

###### tomcat配置

manager-gui等配置,去conf文件下修改 tomcat-users.xml

添加相应权限  注！只可以**本地访问manager**

```xml
 <role rolename="tomcat"/>
  <role rolename="role1"/>
  <user username="tomcat" password="tomcat" roles="tomcat"/>
  <user username="both" password="tomcat" roles="tomcat,role1"/>
  <user username="role1" password="tomcat" roles="role1"/>
  <role rolename="admin"/>
  <role rolename="admin-gui"/>
  <role rolename="manager-gui"/>
  <user username="vic" password="tomcat" roles="manager-gui,admin-gui,admin"/>
</tomcat-users>
```

重启生效

编辑test下的测试页面 主要测试session

```jsp
<%@ page language="java" %><html>
  <head><title>152</title></head>
  <body>
    <h1><font color="green">152</font></h1>
    <table align="centre" border="1">
      <tr>
       <td>Session ID</td>
        <% session.setAttribute("tomcat.suzf.net","tomcat.suzf.net"); %>
        <td><%= session.getId() %></td>
      </tr>
      <tr>
        <td>Created on</td>
        <td><%= session.getCreationTime() %></td>
      </tr>
    </table>
  </body>
</html> 
```

###### session共享保存设置

```
vim /usr/local/tomcat/conf/context.xml
```

```
<Valve className="com.orangefunction.tomcat.redissessions.RedisSessionHandlerValve" />        
<Manager className="com.orangefunction.tomcat.redissessions.RedisSessionManager" 
    host="192.168.247.151"             <!-- Redis地址 -->
    port="6379"                        <!-- Redis端口 -->
    password="centos"                  <!-- Redis密码 -->
    database="0"                       <!-- 存储Session的Redis库编号 -->
    maxInactiveInterval="60"           <!-- Session失效的间隔（秒） -->
/>
```

重启生效

**如若出现500错误 查看防火墙，selinux，iptables等以及redis是否启动**

*完毕*
