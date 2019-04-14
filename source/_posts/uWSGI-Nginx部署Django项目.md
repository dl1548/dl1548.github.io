---
title: uWSGI+Nginx部署Django项目
date: 2018-06-22 11:01:02
tags: django
categories:
    - web
    - django
copyright: true
---

nginx + uwsgi socket 的方式来部署 Django
请求流程:
the web client <-> the web server <-> the socket <-> uwsgi <-> Django
nginx是可选的.
<!--more-->

#### 安装

`yum install python-devel`
如果没有,先安装 `epel-release`.或下载源码安装.

安装 [nginx](http://blog.dl1548.site/2017/02/20/nginx-tomcat-redis%E5%AE%9E%E7%8E%B0%E8%B4%9F%E8%BD%BD%E4%B8%8Esession/)
```
具体参数,按需添加

./configure --prefix=/usr/local/nginx/ --with-http_v2_module --with-http_ssl_module --with-http_sub_module --with-http_stub_status_module --with-http_gzip_static_module --with-pcre
```

安装 `pip install uwsgi --upgrade`


#### uwsgi

uWSGI是一个Web服务器，它实现了WSGI协议、uwsgi、http等协议。uWSGI支持许多不同的语言，因此也支持许多Web应用程序,Nginx中HttpUwsgiModule的作用是与uWSGI服务器进行交换.

测试uWSGI是否能正常运行
新建文件`uwsgi_test.py`
```
def application(env, start_response):
        start_response('200 OK',[('Content-Type', 'text/html')])
        return ['Hello world'] # Python2
        #return [b'Hello world'] # Python3
```
在文件目录下执行` uwsgi --http 0.0.0.0:8001 --wsgi-file uwsgi_test.py`
使用浏览器访问 ip:8001返回hello world 即表示,uwsgi是可运行的


使用uWSGI运行django项目

`uwsgi --http 0.0.0.0:8001 --chdir /path/to/project --home=/path/to/env --module project.wsgi --master --processes 4 --threads 2`

--home 指定virtualenv 路径，如果没有可以去掉
project.wsgi 指的是 project/wsgi.py 文件(比如项目叫test,那就是test.wsgi)
或者直接指定入口文件

`uwsgi --http 0.0.0.0:8001 --chdir /path/to/project --wsgi-file project/wsgi.py --master --processes 4 --threads 2`

执行这个命令会产生4个uwsgi进程（每个进程2个线程），1个master进程，当有子进程死掉时再产生子进程，1个 the HTTP router进程，共6  个进程。
这个Http route进程的地位有点类似nginx，(可以认为与nginx同一层)负责路由http请求给worker, Http route进程和worker之间使用的是uwsgi协议


访问ip:8001 可正常访问,表明项目已通过uwsgi启动成功

##### 编写uwsgi配置文件
参数过多,可以通过ini配置文件来解决.

通过配置文件启动django项目,
此配置是,通过uwsgi来启动django项目 `uwsgi django.ini`

```
[uwsgi]
#绑定地址
http=0.0.0.0:8001
#项目路径,不是应用路径
chdir=/opt/cmdb
#后台启动,输出log
#daemonize=/var/log/uwsgi.log 
master=true
#processes=4
#threads=2

#并发处理进程
work=20 

# 并发的socket 连接数。默认为100,具体视情况
listen=2048 # 和系统配置有关,系统配置见后
socket-timeout=60
die-on-term = true

module=cmdb.wsgi

#指定pid和status路径,目录需新建
stats=%(chdir)/uwsgi/uwsgi.status
pidfile=%(chdir)/uwsgi/uwsgi.pid

```


##### 其他建议
通过配置文件,我们开启了多进程,多并发连接.但是这些还受限于其他的配置.比如nginx 和系统自身

系统配置
`vim /etc/sysctl.conf` 添加如下
`net.core.somaxconn = 2048`net.core.somaxconn是Linux中的一个kernel参数，表示socket监听（listen）的backlog上限
backlog就是socket的监听队列，当一个请求（request）尚未被处理或建立时，他会进入backlog。而socket server可以一次性处理backlog中的所有请求，处理后的请求不再位于监听队列中。当server处理请求较慢，以至于监听队列被填满后，新来的请求会被拒绝。默认 128 

`vim /etc/security/limits.conf`

```
* soft nofile 65535
* hard nofile 65535

#
“*”表示所有用户都生效
soft（应用软件）级别限制的最大可打开文件数的限制
hard表示操作系统级别限制的最大可打开文件数的限制
重启成效

或者 

ulimit -n 65535 临时性生效
```


nginx 配置

```
worker_processes  xx;  #可以设置成cpu个数，或者auto
worker_rlimit_nofile 65535; #worker进程打开最大连接数 和系统配置有关


events {
    use epoll;
    worker_connections  9000;  # 不可超过worker_rlimit_nofile
    multi_accept on;    #告诉nginx收到一个新连接通知后接受尽可能多的连接
}

```


`uwsgi --ini uwsgin.ini` 启动
`uwsgi --reload uwsgi/uwsgi.pid` 重启
`uwsgi --stop uwsgi/uwsgi.pid` 停止
`uwsgi --connect-and-read uwsgi/uwsgi.status` 查看状态


```
常用选项：

http ： 协议类型和端口号

processes ： 开启的进程数量

workers ： 开启的进程数量，等同于processes（官网的说法是spawn the specified number of workers / processes）

chdir ： 指定运行目录（chdir to specified directory before apps loading）

wsgi-file ： 载入wsgi-file（load .wsgi file）

stats ： 在指定的地址上，开启状态服务（enable the stats server on the specified address）

threads ： 运行线程。由于GIL的存在，觉得这个真心没啥用。（run each worker in prethreaded mode with the specified number of threads）

master ： 允许主进程存在（enable master process）

daemonize ： 使进程在后台运行，并将日志打到指定的日志文件或者udp服务器（daemonize uWSGI）。实际上最常用的，还是把运行记录输出到一个本地文件上。

pidfile ： 指定pid文件的位置，记录主进程的pid号。

vacuum ： 当服务器退出的时候自动清理环境，删除unix socket文件和pid文件（try to remove all of the generated file/sockets）

max-requests=500  #为每个工作进程设置请求数的上限。当一个工作进程处理的请求数达到这个值，那么该工作进程就会被回收重用（重启）。你可以使用这个选项来默默地对抗内存泄漏

daemonize : 日志

socket-timeout #为所有的socket操作设置内部超时时间（默认4秒）。

```

至此,项目已经可正常运行.下面要做的是与nginx结合.

#### nginx+uwsgi+django
通过sock文件或者http都是可以的.不同方式,nginx配置有所改变.

`tuoch uwsgi_http.ini`

```
[uwsgi]
#这里不要使用http了,否则访问nginx会出现502.
socket=0.0.0.0:18001
chdir = /opt/cmdb
wsgi-file = cmdb/wsgi.py
daemonize = /opt/cmdb/uwsgi/cmdb.log
touch-reload = /opt/cmdb/uwsgi/reload
#processes = 2
#threads = 4
workers=20
listen=2048
stats=%(chdir)/uwsgi/uwsgi.status
pidfile=%(chdir)/uwsgi/uwsgi.pid
socket-timeout=60
die-on-term=true

-------------------
建议使用此配置.将ini文件当到项目下,与manage.py同级

[uwsgi]
socket=0.0.0.0:18001
chdir=%d
#virtualenv=%d../venv/
module=cmdb.wsgi:application
master=True
pidfile=%(chdir)/uwsgi/uwsgi.pid
touch-reload=%(chdir)/uwsgi/reload
enable-threads=True
vacuum=True
max-requests=500
daemonize=%(chdir)/uwsgi/cmdb.log
processes=%k
threads=2
buffer-size=32768
#socket-timeout=60
```

`touch-reload`是指,reload文件被修改就重新加载进程.项目下新建reload文件,只要touch下这个文件（touch reload) 项目就会重启


##### nginx配置
新建nginx配置文件比如.`cmdb.conf`

```
server {
    listen      8008;
    server_name cmdb;
    charset     utf-8; 
    client_max_body_size 75M;
    
    proxy_connect_timeout 300;
    proxy_read_timeout 300;
    proxy_send_timeout 300;
    proxy_buffer_size 64k;
    proxy_buffers   4 32k;
    proxy_busy_buffers_size 64k;
    proxy_temp_file_write_size 64k;

    error_log  logs/cmdb_error.log;
    access_log logs/cmdb_access.log; 
 
    location /media  {
        alias /opt/cmdb/media;
    }
 
    location /static {
        alias /opt/cmdb/static;
    }
 
    location / {
        #uwsgi_pass的意思动态内容请求都通过名为django的upstream传递给uWSGI
        uwsgi_pass  0.0.0.0:8001;
        uwsgi_send_timeout 600; # 指定向uWSGI传送请求的超时时间，完成握手后向uWSGI传送请求的超时时间。
        uwsgi_connect_timeout 600; # 指定连接到后端uWSGI的超时时间。
        uwsgi_read_timeout 600;        # 指定接收uWSGI应答的超时时间，完成握手后接收uWSGI应答的超时时间。
        #uwsgi_params文件是Nginx向uWSGI传递的参数
        include     /usr/local/nginx-1.12.2/conf/uwsgi_params;
    }
}
```


#### 服务自启

##### nginx自启

###### centos6
`vim /etc/init.d/nginx`

参考官网[Red Hat NGINX Init Script](https://www.nginx.com/resources/wiki/start/topics/examples/redhatnginxinit/)
修改的地方:
nginx="/usr/local/nginx-1.12.2/sbin/nginx" #修改成nginx执行程序的路径。
NGINX_CONF_FILE="/usr/local/nginx-1.12.2/conf/nginx.conf" #修改成nginx.conf文件的路径。

chmod a+x /etc/init.d/nginx 
chkconfig --add /etc/init.d/nginx
chkconfig nginx on

###### centos7
`vi /lib/systemd/system/nginx.service`

```
[Unit]
Description=nginx
After=network.target
  
[Service]
Type=forking
ExecStart=/usr/local/nginx-1.12.2/sbin/nginx
ExecReload=/usr/local/nginx-1.12.2/sbin/nginx -s reload
ExecStop=/usr/local/nginx-1.12.2/sbin/nginx -s quit
PrivateTmp=true
  
[Install]
WantedBy=multi-user.target
```

`systemctl enable nginx.service`


##### uwsgi自启
chmod +x /etc/rc.d/rc.local
追加内容如下:(路径根据实际情况设置)
`cd /opt/cmdb/uwsgi && /usr/bin/uwsgi --ini uwsgi_http.ini`

需要注意的是,ini的配置要设置下后台`daemonize = /opt/cmdb/uwsgi/uwsgi_http.log`
`

看起来是完美了.重启的时候会发现,界面一直卡着...
```
[ *] A stop job is running for /etc/rc.d...........Compatibility
```

是因为没有配置`rc-local.service` 导致

`vim /etc/systemd/system/rc-local.service`

```
#  This file is part of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it
#  under the terms of the GNU Lesser General Public License as published by
#  the Free Software Foundation; either version 2.1 of the License, or
#  (at your option) any later version.

# This unit gets pulled automatically into multi-user.target by
# systemd-rc-local-generator if /etc/rc.d/rc.local is executable.
[Unit]
Description=/etc/rc.d/rc.local Compatibility
ConditionFileIsExecutable=/etc/rc.d/rc.local
After=network.target

[Service]
Type=forking
ExecStart=/etc/rc.d/rc.local start
TimeoutSec=5
RemainAfterExit=yes
```

systemctl daemon-reload     #重新加载systemctl
systemctl enable rc-local   #开机自启
