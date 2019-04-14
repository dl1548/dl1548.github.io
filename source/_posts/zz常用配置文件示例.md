---
title: 常用配置文件示例
date: 1111-11-11 11:11:11
tags: 
categories:
copyright:
password: woshizhu
---

常用配置文件示例
<!--more-->


#### nginx

##### 反代,跨域,session

一个tomcat下跑多个应用

```
#user  nobody;
worker_processes  auto;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;

events {
    use epoll;
    worker_connections  1024;
    multi_accept on;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;
    server_names_hash_bucket_size 128;
    client_header_buffer_size 32k;
    large_client_header_buffers 4 32k;
    client_max_body_size 50m;

    sendfile        on;
    tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    gzip  on;
    gzip_min_length  1k;
    gzip_buffers     4 16k;
    gzip_http_version 1.1;
    gzip_comp_level 2;
    gzip_types     text/plain application/javascript application/x-javascript   text/javascript text/css application/xml application/xml+rss;
    gzip_vary on;
    gzip_proxied   expired no-cache no-store private auth;
    gzip_disable   "MSIE [1-6]\.";
    

    server {
        listen       80;
        server_name  192.168.1.42;

        error_log  logs/error.log;
        access_log logs/access.log;

        default_type 'text/html';
        charset utf-8;
        #charset koi8-r;

        #access_log  logs/host.access.log  main;
    
        #root    /usr/local/nginx/html;
    
        location / {
            root   /usr/local/nginx/html;
            #index  index.html index.htm;
            add_header 'Access-Control-Allow-Origin' *;
        try_files $uri $uri/ @router;
            index  index.html index.htm;
        }

        location /itsm/ {
                root   /usr/local/nginx/html;
                add_header 'Access-Control-Allow-Origin' *;
                #try_files $uri $uri/ @router;
                try_files $uri $uri/ /itsm/index.html;
                index  index.html index.htm;
                #error_log  logs/vue_itsm_error.log;
                #access_log logs/vue_itsm_access.log; 
            }

        location @router {
                rewrite ^.*$ /index.html last;
            }


            #error_page  404              /404.html;

            # redirect server error pages to the static page /50x.html
            #
            error_page   500 502 503 504  /50x.html;
            location = /50x.html {
                root   html;
            }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        location ~ \.php$ {
            root           /usr/local/nginx/html;
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME  /usr/local/nginx/html/$fastcgi_script_name;
            include        fastcgi_params;
            access_log logs/php_access.log;
            error_log  logs/php_error.log  notice;
        }

        location /cmdb {
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_pass http://192.168.1.42:8080/cmdb/;
            proxy_ignore_client_abort on;
            #error_log  logs/cmdb_error.log;
                #access_log logs/cmdb_access.log;
            }
        
        location /itsm/cmdb {
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_pass http://192.168.1.42:8080/cmdb/;
                proxy_ignore_client_abort on;
                #error_log  logs/cmdb_error.log;
                #access_log logs/cmdb_access.log;
            }
    
        location  /itsmboot {
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_pass http://192.168.1.42:8080/itsmboot/;
                proxy_cookie_path /cmdb /itsmboot;  #处理跳转的
                proxy_ignore_client_abort on;
            #    error_log  logs/itsm_error.log;
            #    access_log logs/itsm_access.log;
            }
        
        location  /itsm/itsmboot {
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_pass http://192.168.1.42:8080/itsmboot/;
                proxy_ignore_client_abort on;
            #    error_log  logs/itsm_error.log;
            #    access_log logs/itsm_access.log;
            }
    
        #websocket ping
        location /cmdb/wsping {
            	proxy_pass http://192.168.1.155:8080/cmdb/webSocket;
                proxy_redirect off;
            	proxy_read_timeout 300s;
                proxy_send_timeout 300s;
                proxy_pass_request_headers on;
                #proxy_cookie_path /cmdb /itsmboot;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection Upgrade;
        }

    
    
        location /api {
            uwsgi_pass  0.0.0.0:18001;
            include     /usr/local/nginx/conf/uwsgi_params;
            error_log  logs/api_error.log;
            access_log logs/api_access.log; 
        }
        location /monitor {
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_pass http://192.168.1.36:8080/monitor/;
                proxy_ignore_client_abort on;
                error_log  logs/monitor_error.log;
                access_log logs/monitor_access.log;
            proxy_redirect off;
              }
    }
}

```


#### redis

文件中的路径根据实际情况修改

##### /etc/redis.conf

```
port 6379
protected-mode no #保护模式 开启则需要配置bind和密码,否则关闭
daemonize yes #开启后台进程
pidfile /var/redis/run/6379.pid
logfile /var/redis/log/redis.log
dbfilename dump.rdb
dir /var/redis/data  #数据库路径 默认是./
requirepass centos #设置密码为centos
#bind 127.0.0.1  默认是开启的，只允许本地登陆，所以，要不添加IP，要不给注释了 
```

##### /etc/init.d/redis

```
#!/bin/sh
REDISPORT=6379
EXEC=/usr/local/redis/bin/redis-server
CLIEXEC=/usr/local/redis/bin/redis-cli

PIDFILE=/var/redis/run/${REDISPORT}.pid
CONF="/etc/redis.conf"

case "$1" in
    start)
        if [ -f $PIDFILE ]
        then
                echo "$PIDFILE exists, process is already running or crashed"
        else
                echo "Starting Redis server..."
                $EXEC $CONF
        fi
        ;;
    stop)
        if [ ! -f $PIDFILE ]
        then
                echo "$PIDFILE does not exist, process is not running"
        else
                PID=$(cat $PIDFILE)
                echo "Stopping ..."
                $CLIEXEC -p $REDISPORT shutdown
                while [ -x /proc/${PID} ]
                do
                    echo "Waiting for Redis to shutdown ..."
                    sleep 1
                done
                echo "Redis stopped"
        fi
        ;;
    *)
        echo "Please use start or stop as first argument"
        ;;
esac

```



