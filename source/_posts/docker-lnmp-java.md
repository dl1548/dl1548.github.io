---
title: docker-lnmp-java
date: 2018-07-31 08:31:21
tags: docker
categories: 
    - linux
copyright: true
password: woshizhu
---

java 项目,前端nginx.
<!--more-->

涉及知识点较多,本文主要写的是容器化的部分.
其余的知识点并不会做过多的解释.主要是容器相关的配置文件解释.
阅读前提,熟悉linux系统,具有一定docker知识.对compose编排有了解.熟悉zabbix,nginx的编译安装.清楚相关配置参数,依赖等.
如果对以上提到的几点有不清楚的,那么不建议阅读.

#### 构建rpm包

由于在容器内编译是需要很多组件以及依赖和工具.所以导致生成的容器以及镜像体积非常大.

所以,建议下载相关的源码包,构建完毕后在容器内安装,这样整个包体积就缩小非常多!

当然如果对整个编译依赖`非常非常清楚`,也可以在容器内编译,然后编译完毕后删除不需要的依赖包以及缓存等.

具体rpm构建方式不赘述了.有兴趣可以[点此阅读 ](http://blog.dl1548.site/2018/02/28/rpm%E6%89%93%E5%8C%85/),相关的rpm构建配置文档如下

##### nginx rpm包构建

nginx.spec 文件配置

```
Name:   nginx                              
Version:     1.12.2                           
Release:    1%{?dist}                       
Summary:    nginx-1.12.2                   
Group:      nginx                          
License:    GPL

URL:        http://blog.dl1548.site

Source0:    nginx-1.12.2.tar.gz
Source1:    nginx.conf
Source2:    fastcgi_params
Source3:    index.php


BuildRequires:  openssl,openssl-devel,pcre,pcre-devel         
#制作rpm包时，所依赖的基本库,很多,并未写完,具体的在dockerfile体现

Requires:   openssl,openssl-devel,pcre,pcre-devel

#BuildRoot: %{_tmpdir}/%{name}-%{version}

%description                                
creata by lizili

%pre                                        
grep nginx /etc/passwd > /dev/null
if [ $? != 0 ] 
then useradd nginx -M -s /sbin/nologin
fi
[ -d /usr/local/zabbix ]||rm -rf /usr/local/nginx*

%post                                       

%preun                                      
/usr/local/nginx/sbin/nginx -s stop

%postun                                     
userdel  nginx
rm -rf /usr/local/nginx

%prep
%setup -q                                   

#这里就是nginx编译安装的相关配置信息
%build                                      
./configure --user=nginx --group=nginx --prefix=/usr/local/%{name} --with-file-aio --with-http_ssl_module \
--with-http_realip_module --with-http_addition_module --with-http_xslt_module --with-http_image_filter_module \
--with-http_sub_module --with-http_dav_module --with-http_flv_module --with-http_mp4_module --with-http_gunzip_module \
--with-http_gzip_static_module --with-http_auth_request_module --with-http_random_index_module --with-http_secure_link_module \
--with-http_degradation_module --with-http_stub_status_module
make -j8 %{?_smp_mflags}


%install                                    
test -L %{buildroot}/usr/local/%{name} && rm -f %{buildroot}/usr/local/%{name}
make install DESTDIR=%{buildroot}

install -p -D -m 0755 %{SOURCE1}        %{buildroot}/usr/local/%{name}/conf/nginx.conf
install -p -D -m 0755 %{SOURCE2}        %{buildroot}/usr/local/%{name}/conf/fastcgi_params
install -p -D -m 0755 %{SOURCE3}        %{buildroot}/usr/local/%{name}/html/index.php

%files
%defattr (-,root,root,0755)                                      

/usr/local/%{name}/*

%changelog                                  
#主要用于软件的变更日志。该选项可有可无

%clean 
rm -rf %{buildroot}                         
#清理临时文件

```

##### zabbix_server rpm包构建

zabbxi_server.spec

```
%define zabbix_user zabbix                  
Name:   zabbix                              
Version:    3.0.5                          
Release:    1%{?dist}                       
Summary:    zabbix_server                 
Group:      zabbix                          
License:    GPL
URL:        http://blog.dl1548.site
Source0:    zabbix-3.0.5.tar.gz
BuildRequires: unixODBC 
#制作rpm包时，所依赖的基本库,依赖很多,简单写一个

Requires: unixODBC

#BuildRoot: %{_tmpdir}/%{name}-%{version}-%{release}

%description                                
Zabbix server 3.0.5
creata by lizili

%pre                                        
grep zabbix /etc/passwd > /dev/null
if [ $? != 0 ] 
then useradd zabbix -M -s /sbin/nologin
fi
[ -d /usr/local/zabbix ]||rm -rf /usr/local/zabbix*
rm -rf /etc/zabbix*


%post                                       
#修改启动文件路径
sed -i "/BASEDIR=\/usr\/local/c\BASEDIR=\/usr\/local\/%{name}" /etc/init.d/zabbix_agentd
sed -i "/BASEDIR=\/usr\/local/c\BASEDIR=\/usr\/local\/%{name}" /etc/init.d/zabbix_server

%preun                                      
/etc/init.d/zabbix_server stop

%postun                                     
userdel  zabbix
rm -rf /usr/local/zabbix*

%prep

%setup -q                                   

%build                                      
#定义编译软件包时的操作
./configure --prefix=/usr/local/%{name} --enable-server --enable-agent --enable-ipv6 --with-mysql --with-net-snmp  --with-libcurl --with-openipmi --with-ldap --with-ssh2 --enable-java --with-libxml2
make -j8 %{?_smp_mflags}

%install                                    
test -L %{buildroot}/usr/local/%{name} && rm -f %{buildroot}/usr/local/%{name}
install -d %{buildroot}/etc/profile.d
install -d %{buildroot}/etc/init.d
make install DESTDIR=%{buildroot}

#复制启动脚本
cp %{_builddir}/%{name}-%{version}/misc/init.d/fedora/core/zabbix_agentd  %{buildroot}/etc/init.d/zabbix_agentd
cp %{_builddir}/%{name}-%{version}/misc/init.d/fedora/core/zabbix_server  %{buildroot}/etc/init.d/zabbix_server

%files
%defattr (-,root,root,0755)                                      
#定义rpm包安装时创建的相关目录及文件
/usr/local/%{name}/*
/etc/init.d/zabbix_agentd
/etc/init.d/zabbix_server

%changelog                                  
#主要用于软件的变更日志。该选项可有可无

%clean 
rm -rf %{buildroot}                         
#清理临时文件

```

事后会得到两个编译好的rpm包. 保存下来,给容器使用.



#### 编排

本文使用的docker官网的编排工具docker-compose

编排的前提是,先要创建一个私有网络`docker network create -d bridge --subnet 172.1.0.0/16 jk`  

这样,可以不用映射某些不必要的端口.`容器内可直接访问相关的地址`.

当然,也可以配置通过映射访问.那么需要更改几个文件.

建议容器内互联,这样,可不用进入容器内更改配置.而且不必要的端口不用曝露出来,增加安全性

编排结构如下

```
├── docker-compose.yml
├── monitor
│   └── apache-tomcat-8.5.24.tar.gz
│   └── Dockerfile                  
│   └── jdk-8u45-linux-x64.tar.gz
│   └── monitor.war
├── mysql
│   ├── cnf
│   │   ├── 3.0.5
│   │   │   ├── data.sql
│   │   │   ├── images.sql
│   │   │   ├── monitor.sql
│   │   │   └── schema.sql
│   │   ├── init_database.sql
│   │   ├── init_db.sh
│   │   └── mysqld.cnf
│   ├── datadir
│   └── Dockerfile
├── nginx
│   ├── Dockerfile
│   ├── nginx-1.12.2-1.el7.centos.x86_64.rpm
│   ├── nginx.conf
│   └── zabbix
│       ├── 3.0.5
│       │   ├── logs
│       │   │   └── zabbix_server.log
│       │   ├── web
│       │   │   └── zabbix.conf.php
│       │   └── zabbix_server.conf
│       └── zabbix-3.0.5-1.el7.centos.x86_64.rpm
└── readme

```

`monitor` 此文件夹下是monitor项目的Dockerfile需要的文件以及配置

`mysql` 此文件夹下是mysql项目的Dockerfile需要的文件以及配置

`nginx` 此文件夹下是nginx项目的Dockerfile需要的文件以及配置(包括zabbix)

`docker-compose.yml` 是编排文件,通过它来构建整个monitor的容器化

##### monitor

此文件夹下的先关配置用来构建公司monitor项目.

Dockerfile如下:

```
FROM centos

ADD jdk-8u45-linux-x64.tar.gz /usr/local

ENV RUN_AS_USER=root
ENV JAVA_HOME /usr/local/jdk1.8.0_45
ENV CLASS_HOME=/usr/local/jdk1.8.0_45/lib:$JAVA_HOME/jre/lib
ENV CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib:$CLASSPATH
ENV PATH=$PATH:$JAVA_HOME/bin

ADD apache-tomcat-8.5.24.tar.gz /usr/local

EXPOSE 8080
ENTRYPOINT ["/usr/local/apache-tomcat-8.5.24/bin/catalina.sh", "run"]
```



##### mysql

`cnf文件夹` 数据库相关配置文件.数据库配置就是`mysqld.cnf` Dockerfile 将此文件进行映射并赋权.

这样在生成镜像的时候,相关的数据库配置就能生效了.不用去容器内修改.注意权限!

`3.0.5`  针对的是zabbix版本,存放的也是zabbix需要预先生成并导入的表.

`init_database.sql` 初始化数据的文件 

```
CREATE DATABASE IF NOT EXISTS zabbix default charset utf8 COLLATE utf8_general_ci;
CREATE DATABASE IF NOT EXISTS monitor default charset utf8 COLLATE utf8_general_ci;
```

`init_db.sh` 初始化数据库的脚本

```
#init db
mysql -uroot -p$MYSQL_ROOT_PASSWORD << EOF
source $WORK_PATH/$FILE_0;
EOF

# init zbx
mysql -uroot -p$MYSQL_ROOT_PASSWORD zabbix << EOF
source $WORK_PATH/$FILE_1;
source $WORK_PATH/$FILE_2;
source $WORK_PATH/$FILE_3;
EOF

#init monitor
mysql -uroot -p$MYSQL_ROOT_PASSWORD monitor << EOF
source $WORK_PATH/$FILE_4;
EOF
```

`datadir` 用来做数据持久化使用

Dockerfile如下:

```
FROM mysql:5.7.21

MAINTAINER lizili(blog.dl1548.site)

ENV WORK_PATH /usr/local/work

#此目录下的脚本在生成容器是会自动执行.用来做初始化使用
ENV AUTO_RUN_DIR /docker-entrypoint-initdb.d

#定义sql文件名
ENV FILE_0 init_database.sql 
ENV FILE_1 schema.sql
ENV FILE_2 images.sql
ENV FILE_3 data.sql
ENV FILE_4 monitor.sql
#定义shell文件名
ENV INSTALL_DB_SHELL init_db.sh

#把数据库初始化数据的文件复制到工作目录下
COPY cnf/$FILE_0 $WORK_PATH/
COPY cnf/3.0.5/$FILE_1 $WORK_PATH/
COPY cnf/3.0.5/$FILE_2 $WORK_PATH/
COPY cnf/3.0.5/$FILE_3 $WORK_PATH/
COPY cnf/3.0.5/$FILE_4 $WORK_PATH/
#把要执行的shell文件放到/docker-entrypoint-initdb.d/目录下，容器会自动执行这个shell
COPY cnf/$INSTALL_DB_SHELL $AUTO_RUN_DIR/

#mysql配置文件
COPY cnf/mysqld.cnf /etc/mysql/mysql.conf.d/mysqld.cnf

#给执行文件增加可执行权限.权限一定要更改.否则配置不生效.这个权限困扰我一段时间..有点坑.
RUN mkdir -p $WORK_PATH \
    && chown mysql:mysql /etc/mysql/mysql.conf.d/mysqld.cnf \
    && chmod 600 /etc/mysql/mysql.conf.d/mysqld.cnf \
    && chmod a+x $AUTO_RUN_DIR/$INSTALL_DB_SHELL
```

##### nginx

`nginx-1.12.2-1.el7.centos.x86_64.rpm` 前面构建的rpm安装包.

`nginx.conf` nginx配置文件,需要注意这里的nginx路径是由构建rpm是决定的.

```
#user  nobody;
worker_processes  auto;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;
events {
    use epoll;
    worker_connections 51200;
    multi_accept on;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    #access_log  logs/access.log  main;
    sendfile        on;
    client_max_body_size  500m;

    tcp_nopush     on;
    tcp_nodelay on;
    keepalive_timeout  65;

    fastcgi_connect_timeout 300;
    fastcgi_send_timeout 300;
    fastcgi_read_timeout 300;
    fastcgi_buffer_size 64k;
    fastcgi_buffers 4 64k;
    fastcgi_busy_buffers_size 128k;
    fastcgi_temp_file_write_size 256k;


    gzip on;
    gzip_min_length  1k;
    gzip_buffers     4 16k;
    gzip_http_version 1.1;
    gzip_comp_level 2;
    gzip_types     text/plain application/javascript application/x-javascript text/javascript text/css application/xml application/xml+rss;
    gzip_vary on;
    gzip_proxied   expired no-cache no-store private auth;
    gzip_disable   "MSIE [1-6]\.";


    server {
        listen       80;
        server_name  localhost;
        index index.html index.htm index.php;
        root /usr/local/nginx/html;

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        location ~ [^/]\.php(/|$) {
            #try_files $uri =404;
            root           /usr/local/nginx/html;
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            #fastcgi_param  SCRIPT_FILENAME  /usr/local/nginx/html/$fastcgi_script_name;
            include        fastcgi_params;
        }

        location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$ {
            expires      30d;
        }

        location ~ .*\.(js|css)?$ {
            expires      12h;
        }

        location ~ /\. {
            deny all;
        }
    
        access_log logs/php_access.log;
        error_log  logs/php_error.log  notice;
    }

}
```

`zabbix` 此文件夹是用来搭建监控服务.

`3.0.5` 是为了zabbix版本分类使用

`web` 存放的是zabbix官方提供的监控前端php等文件.

`zabbix.conf.php` zabbix前端的配置文件.主要是指定连接的数据库地址,以及监控server地址

此文件可直接在官网提供的前端文件中,复制,重命名,然后更改.

注意我指定的数据库地址,就是容器内地址,可不通过端口映射访问,一定程度上讲,访问速度也有所提升.

```
<?php
// Zabbix GUI configuration file.
global $DB;

$DB['TYPE']     = 'MYSQL';
$DB['SERVER']   = '172.1.0.33';
$DB['PORT']     = '3306';
$DB['DATABASE'] = 'zabbix';
$DB['USER']     = 'root';
$DB['PASSWORD'] = '123qweASD';

// Schema name. Used for IBM DB2 and PostgreSQL.
$DB['SCHEMA'] = '';

$ZBX_SERVER      = '192.168.1.244';
$ZBX_SERVER_PORT = '10051';
$ZBX_SERVER_NAME = '';
$IMAGE_FORMAT_DEFAULT = IMAGE_FORMAT_PNG;
```

`zabbix_server.conf`  zabbix_server服务配置.

这个文件主要是要指定数据库为远程数据库.以及配置相关库和账户密码等

主要更改的几个点

```
DBHost=172.1.0.33  #这里指定的也是容器内的地址.如果是端口映射,那么指定os地址.
DBName=zabbix
DBUser=root
DBPassword=123qweASD
```

`zabbix-3.0.5-1.el7.centos.x86_64.rpm`  前面构建的rpm安装包.

Dockerfile 配置如下:

这个dockerfile是最复杂的,安装了nginx,zabbix,php-fpm的运行环境以及依赖.

安装了容器中文语言环境支持,用来支持zabbix的中文语言环境(需要做相关配置.具体公司操作文档有.可直接在web内的前端文件修改,docker会自动映射到容器内部),修改了一下配置文件,使zabbix可正常运行.

```
FROM centos

MAINTAINER zili.li(blog.dl1548.site)

COPY nginx-1.12.2-1.el7.centos.x86_64.rpm /tmp/nginx-1.12.2-1.el7.centos.x86_64.rpm
COPY zabbix/zabbix-3.0.5-1.el7.centos.x86_64.rpm /tmp/zabbix-3.0.5-1.el7.centos.x86_64.rpm
#zabbix rpm

WORKDIR /tmp

#install server
RUN nginxDeps='glib* autoconf openssl openssl-devel libxslt-devel gd gd-devel pcre pcre-devel ' \
    && phpDeps='php php-mysql php-common php-fpm php-gd php-ldap \
            php-odbc php-pear php-xml php-mcrypt php-xmlrpc php-devel \
            php-mbstring php-snmp php-soap curl curl-devel php-bcmath mod_ssl' \
    && zbxDeps='fping mysql-connector-odbc mysql-devel libdbi-dbd-mysql \
            libssh2 libxml2 libxml2-devel libssh2-devel unixODBC unixODBC-devel \
            pam-devel net-snmp-devel net-snmp-utils  OpenIPMI OpenIPMI-devel rpm-build \
            openldap openldap-devel libcurl-devel  java java-devel' \
    && useradd -M -s /sbin/nologin nginx \
    && useradd -M -s /sbin/nologin php \
    && yum -y install epel-release \
    && yum clean all \
    && yum -y install $nginxDeps $phpDeps $zbxDeps kde-l10n-Chinese \ 
    && rpm -ivh nginx-1.12.2-1.el7.centos.x86_64.rpm \
    && rpm -ivh zabbix-3.0.5-1.el7.centos.x86_64.rpm \
    && rm -rf /tmp/* \
    && rm -rf /var/cache/* \
    && sed -i 's/upload_max_filesize = 2M/upload_max_filesize = 50M/g' /etc/php.ini \
    && sed -i 's/;date.timezone =/date.timezone =PRC/' /etc/php.ini \
    && sed -i 's/max_execution_time = 30/max_execution_time = 600/g' /etc/php.ini \
    && sed -i 's/max_input_time = 60/max_input_time = 600/g' /etc/php.ini \
    && sed -i 's/memory_limit = 128M/memory_limit = 256M/g' /etc/php.ini \
    && sed -i 's/post_max_size = 8M/post_max_size = 16M/g' /etc/php.ini \
    && rm -rf /etc/localtime && ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
    && localedef -c -f UTF-8 -i zh_CN zh_CN.utf8

#ENV TIME_ZONE Asia/Shanghai
ENV PATH /usr/local/nginx/sbin:$PATH                
ENV LC_ALL zh_CN.utf8

EXPOSE 80

#ENTRYPOINT ["nginx"]

CMD /usr/local/nginx/sbin/nginx && /etc/init.d/zabbix_server start && /etc/init.d/zabbix_agentd start && /usr/sbin/php-fpm
```



##### docker-compose.yml

```
version: "3"
services:
  backserver:
    hostname: MonitorBack
    build: ./nginx
    volumes:
      - ./nginx/nginx.conf:/usr/local/nginx/conf/nginx.conf:rw
      - ./nginx/zabbix/3.0.5/web/:/usr/local/nginx/html/monitor:rw
      - ./nginx/zabbix/3.0.5/zabbix_server.conf:/usr/local/zabbix/etc/zabbix_server.conf:rw
    ports:
      - 80:80
      - 10050:10050 
      - 10051:10051
    networks:
      jk:
        ipv4_address: 172.1.0.80
    restart: always
    depends_on:
      - dbserver
      - webserver

  webserver:
    hostname: JKMonitor
    build: ./monitor
    restart: always
    volumes:
      - ./monitor/monitor.war:/usr/local/apache-tomcat-8.5.24/webapps/monitor.war
    ports:
      - 8080:8080
    networks:
      jk:
        ipv4_address: 172.1.0.88
    depends_on:
      - dbserver

  dbserver:
    hostname: MonitorDB
    build: ./mysql
    volumes:
      #- ./my.cnf:/etc/my.cnf.d/zili.cnf:rw #在dockerfile中配置并赋权
      - ./mysql/datadir:/var/lib/mysql:rw
    #ports:
    #  - 3306:3306
    networks:
      jk:
        ipv4_address: 172.1.0.33
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: '123qweASD'
      MYSQL_ROOT_HOST: '%'

networks:
  jk:
    external: true
```



进入jk下 执行`docker-compose up --build` 即可,后台加`-d`

完.