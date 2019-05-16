---
title: nginx+lua获取get和post参数
date: 2019-05-15 14:39:25
tags: 
  - nginx
  - lua
categories: linux
copyright: true
---

<img src="/images/nginx-lua.png" alt="nginx-lua" align=center />

nginx 获取GET参数，或者POST参数，进行动态的转发等

<!--more-->

mkdir /opt/packages

#### 安装nginx

```
# 安装依赖
yum -y install gcc gcc-c++ glib* make autoconf openssl openssl-devel \
libxslt-devel gd gd-devel pcre pcre-devel  readline-devel  wget curl

cd /opt/packages
wget http://nginx.org/download/nginx-1.12.2.tar.gz

#新建用户，如果不新建，configure时则不要指定用户
useradd -M -s /sbin/nologin nginx 

#编译，此时可以不编译安装，因为后续需添加模块到nginx
./configure --user=nginx --group=nginx --prefix=/usr/local/nginx --with-file-aio \
--with-http_ssl_module --with-http_realip_module --with-http_addition_module \
--with-http_xslt_module --with-http_image_filter_module --with-http_sub_module \
--with-http_dav_module --with-http_flv_module --with-http_mp4_module \
--with-http_gunzip_module --with-http_gzip_static_module \
--with-http_auth_request_module --with-http_random_index_module \
--with-http_secure_link_module --with-http_degradation_module \
--with-http_stub_status_module

#安装
make -j2 && make install

```



#### 安装LuaJIT

```
cd /opt/packages
wget http://luajit.org/download/LuaJIT-2.0.5.tar.gz
tar -zxvf  LuaJIT-2.0.5.tar.gz
cd LuaJIT-2.0.5
make install PREFIX=/usr/local/LuaJIT
```

输出 以下内容则成功

`==== Successfully installed LuaJIT 2.0.5 to /usr/local/LuaJIT ====`

修改环境变量

```
export LUAJIT_LIB=/usr/local/LuaJIT/lib
export LUAJIT_INC=/usr/local/LuaJIT/include/luajit-2.0

#退出后
source /etc/profile
```



#### ngx_devel_kit

```
cd /opt/packages
wget https://github.com/simplresty/ngx_devel_kit/archive/v0.3.0.tar.gz
tar zxvf v0.3.0.tar.gz

#放着，一会编译ngxin时导入。
```



#### 安装lua-nginx-module

 ```
cd /opt/packages
wget https://github.com/openresty/lua-nginx-module/archive/v0.10.13.tar.gz
tar zxvf v0.10.13.tar.gz

#放着，一会编译ngxin时导入。
 ```



#### 编译模块至nginx

```
#如果nginx已经安装 执行nginx -V 可查看编译参数

./configure --user=nginx --group=nginx --prefix=/usr/local/nginx --with-file-aio \
--with-http_ssl_module --with-http_realip_module --with-http_addition_module \
--with-http_xslt_module --with-http_image_filter_module --with-http_sub_module \
--with-http_dav_module --with-http_flv_module --with-http_mp4_module \
--with-http_gunzip_module --with-http_gzip_static_module \
--with-http_auth_request_module --with-http_random_index_module \
--with-http_secure_link_module --with-http_degradation_module \
--with-http_stub_status_module \
--with-ld-opt=-Wl,-rpath,/usr/local/LuaJIT/lib \
--add-module=/opt/packages/ngx_devel_kit-0.3.0 \
--add-module=/opt/packages/lua-nginx-module-0.10.13

make -j2 && make install
```

编译主要添加的是 这三条数据

```
--with-ld-opt=-Wl,-rpath,/usr/local/LuaJIT/lib \
--add-module=/opt/packages/ngx_devel_kit-0.3.0 \
--add-module=/opt/packages/lua-nginx-module-0.10.13
```



#### 测试lua模块

修改`nginx.conf` 的`server` 添加如下

```
location /lua { 
    default_type 'text/plain'; 
    content_by_lua 'ngx.say("lua install success")'; 
}

# 重加载nginx
nginx -s reload

curl http://127.0.0.1/lua
#返回 ： lua install success

```



lua 模块安装完毕。



#### 安装 cjson

如果需要lua解析json，可安装此模块

```
cd /opt/packages
http://www.kyne.com.au/~mark/software/download/lua-cjson-2.1.0.tar.gz
make
```

如若报错如下

```
cc -c -O3 -Wall -pedantic -DNDEBUG  -I/usr/local/include -fpic -o lua_cjson.o lua_cjson.c
lua_cjson.c:43:17: 致命错误：lua.h：没有那个文件或目录
 #include <lua.h>
                 ^
编译中断。
make: *** [lua_cjson.o] 错误 1
```

修改`Makefile` 这两处 

```
PREFIX =            /usr/local/LuaJIT
LUA_INCLUDE_DIR =   $(PREFIX)/include/luajit-2.0
```



复制`cjson.so`

```
 cp cjson.so /usr/local/LuaJIT/lib/lua/5.1/
 chmod 755 /usr/local/LuaJIT/lib/lua/5.1/cjson.so 
```



#### 使用lua 去rewrite路由

```
location /zili {
	# 请求体的size大于nginx配置里的client_body_buffer_size
	# 则会导致请求体被缓冲到磁盘临时文件里，client_body_buffer_size默认是8k或者16k
	# 如果考虑性能，可以使用ngx.req.get_body_file()，见后续
	
	client_max_body_size  100m ;
    client_body_buffer_size  100m ;
    set $proxy '';
        rewrite_by_lua_block {
            local request_method = ngx.var.request_method
            if request_method == "GET" then
                local arg = ngx.req.get_uri_args()["proxy"] or 0
                ngx.var.proxy = arg
            elseif request_method == "POST" then
                ngx.req.read_body()
                local jkdata = ngx.req.get_body_data()
                --ngx.print(jkdata)
                if jkdata then
                	--如果传进来的是 json 通过这种方式解析，
                    --cjson = require "cjson"
                    --jkval = cjson.decode(jkdata)
                    --ngx.var.proxy = valp["proxy"]
                    
                    --这里比较糙，使用正则去匹配了需要的ip字符串
                    ngx.var.proxy = jkdata:match("(%d+%.%d+%.%d+%.%d+)")
                    --ngx.var.proxy = jkval["proxy"]
                end
            end
        }
    include     /usr/local/nginx/conf/uwsgi_params;
    proxy_pass  $proxy:18001;
    access_log logs/aaa_access.log zili;
}

```

ngx.req.get_body_file()

```
ngx.req.read_body()
body_data = ngx.req.get_body_data()
if not body_data then
    local datafile = ngx.req.get_body_file()
    if not datafile then
        error_code = 1
        error_msg = "no request body found"
    else
        local fh, err = io.open(datafile, "r")
        if not fh then
            error_code = 2
            error_msg = "failed to open " .. tostring(datafile) .. "for reading: " .. tostring(err)
        else
            fh:seek("set")
            body_data = fh:read("*a")
            fh:close()
            if body_data == "" then
                error_code = 3
                error_msg = "request body is empty"
            end
        end
    end
 end
```



#### 相关包

```
链接: https://pan.baidu.com/s/1nq9pOQO7uj88DwfHaT0EDg 提取码: p3fh
```



完

---

