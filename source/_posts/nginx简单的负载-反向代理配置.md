---
title: Nginx负载/反向代理
date: 2017-08-18 15:18:06
tags: nginx
categories:
    - web
    - nginx
copyright: true
---

用Nginx实现简单的负载和代理转发.
<!--more-->
#### 安装Nginx

nginx默认使用80端口，请确保80未被使用
```
Nginx
# wget http://www.nginx.org/download/nginx-[version].tar.gz
Nginx cache purge 模块(可选)
# wget http://labs.frickle.com/files/ngx_cache_purge-[version].tar.gz

#编译安装
./configure \
--prefix=/usr/local/nginx-[version] \
--with-http_stub_status_module \
--with-http_ssl_module \
--with-http_realip_module \ 
--add-module=../ngx_cache_purge-1.3
# make
# make install

#启动，停止，重载nginx
/usr/local/nginx-[version]/sbin/nginx   #启动
/usr/local/nginx-[version]/sbin/nginx -t #测试，检测配置
/usr/local/nginx-[version]/sbin/nginx -s stop
/usr/local/nginx-[version]/sbin/nginx -s reload

打开浏览器，访问nginx地址，出现welcome nginx则配置成功

```

#### nginx 负载

新建配置文件blance-test.conf***然后在nginx.conf 中include blance-test.conf***
```
upstream blance-test {
    server 192.168.1.11:80;
    server 192.168.1.22:80;
}
server
{
    listen 80;
    server_name www.example.com; 
    location / {
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; 
        proxy_pass http://blance-test;
    }
    access_log logs/blance-test_access.log;
}
```
##### proxy_set_header

```
重点看下proxy_set_header X-Forwarded-For
当使用了代理时，web服务器无法取得真实IP，为了避免这个情况，
代理服务器通常会增加一个叫做x_forwarded_for的头信息，
把连接它的客户端IP（即客户端IP）加到这个头信息里，
这样就能保证网站的web服务器能获取到真实IP


$host 
 #请求中的主机头(Host)字段，如果请求中的主机头不可用或者空，
则为处理请求的server名称(处理请求的server的server_name的值)。小写，不含端口。

$remote_addr;   
 #客户端的IP地址（中间无代理则是真实客户端IP，有代理则是代理IP）

$proxy_add_x_forwarded_for;  
#就是$http_x_forwarded_for加上$remote_addr 
官方解释 
$proxy_add_x_forwarded_for
the “X-Forwarded-For” client request header field with the $remote_addr variable appended to it, separated by a comma. If the “X-Forwarded-For” field is not present in the client request header, the $proxy_add_x_forwarded_for variable is equal to the $remote_addr variable.
```


#### nginx 反向代理

***可选配置，与http同级***
```
   client_max_body_size 50m; #缓冲区代理缓冲用户端请求的最大字节数
    client_body_buffer_size 256k;
    client_header_timeout 3m;
    client_body_timeout 3m;
    send_timeout 3m;
    proxy_connect_timeout 300s; #nginx跟后端服务器连接超时时间(代理连接超时)
    proxy_read_timeout 300s; #连接成功后，后端服务器响应时间(代理接收超时)
    proxy_send_timeout 300s;
    proxy_buffer_size 64k; #设置代理服务器（nginx）保存用户头信息的缓冲区大小
    proxy_buffers 4 32k; #proxy_buffers缓冲区，网页平均在32k以下的话，这样设置
    proxy_busy_buffers_size 64k; #高负荷下缓冲大小（proxy_buffers*2）
    proxy_temp_file_write_size 64k; #设定缓存文件夹大小，大于这个值，将从upstream服务器传递请求，而不缓冲到磁盘
    proxy_ignore_client_abort on; #不允许代理端主动关闭连接

```
新建配置文件reverse-proxy.conf***然后在nginx.conf 中include reverse-proxy.conf***
```
    server {
        listen          7001;
        server_name     192.168.1.202:7001;
        charset utf-8;
        location /console {
           # proxy_set_header Host $host:$proxy_port;
            proxy_set_header Host $http_host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; 
            proxy_pass http://192.168.1.11:7001;
        }
        access_log logs/192.168.1.11:7001_access.log;
    }
    server {
        listen          8001;
        server_name     192.168.1.202:8001;
        charset utf-8;
        location / {
            proxy_set_header Host $host:$proxy_port;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; 
            proxy_pass http://192.168.1.11:8001;
        }
        access_log logs/192.168.1.11:8001_access.log;
    }

```

这里要说明下
```
server_name 
#主要用于配置基于名称的虚拟主机，可匹配正则，如果没有域名可填写IP，不填或随便填写一个
#nginx 根据 server_name 匹配 HTTP 请求头的 host，去决定使用那个 server
#如果都没有匹配则使用默认的。如果没有默认就是第一个server

$http_host 和 $host的区别

$host 上边描述过，主机头(Host)字段，如果不可用或空就是server_name的值。
$http_host  可以理解请求地址，即浏览器中你输入的地址（IP或域名）
那么这两个使用哪个好一些呢？
如果Host请求头部没有出现在请求头中，则$http_host值为空，但是$host值为主域名，
一般而言，会用$host代替$http_host，避免http请求中丢失Host头部的情况下Host不被重写。
如果对端口又要求可加上 :$proxy_port
```

[官方关于nginx 代理模块的文章](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#.24proxy_add_x_forwarded_for)

> Nginx访问日志 IP统计
awk '{print $1}' access.log | sort | uniq -c|sort -n

___
