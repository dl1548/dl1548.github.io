---
title: nginx自签证书
date: 2018-08-02 09:34:34
tags:
    - nginx
categories:
    - web
copyright:
---


#### 证书生成

生成RSA的密钥，需填密码，后续会使用到

`openssl genrsa -des3 -out jkstack.key 1024`

生成一个不需要密码的密钥文件，要给nginx使用，若有密码，每次reload nginx配置时候都要你验证这个PAM密码的

`openssl rsa -in jkstack.key -out jkstack_nopass.key`

生成一个证书请求 

`openssl req -new -key jkstack.key -out jkstack.csr` 



```
Enter pass phrase for domain.key:                           # 之前设置的密码
-----
Country Name (2 letter code) [XX]:CN                        # 国家
State or Province Name (full name) []:ShangHai                 # 地区或省份
Locality Name (eg, city) [Default City]:ShangHai           # 地区局部名
Organization Name (eg, company) [Default Company Ltd]:JK # 机构名称
Organizational Unit Name (eg, section) []:JK            # 组织单位名称
Common Name (eg, your name or your server's hostname) []:jk.com # 网站域名
Email Address []:jk@jk.com                             # 邮箱
A challenge password []:                                    # 私钥保护密码,可直接回车
An optional company name []:                                # 可直接回车
```



使用上面的密钥和CSR对证书签名
`openssl x509 -req -days 3650 -in jkstack.csr -signkey jkstack.key -out jkstack.crt`

 

#### Ngx配置

```
nginx需编译模块 
--with-http_ssl_module --with-http_stub_status_module

nginx -V  # 查看当前模块
```



```
server {
    listen 80;
    listen 443 ssl;                # 监听443端口, 开启ssl(必须)
    server_name domain.com;
    
    # ssl on;     # 不建议使用， 该指令与listen中ssl参数功能相同.
    # 引用ssl证书(必须,如果放在nginx/conf/ssl下可以用相对路径,其他位置必须用绝对路径)
    ssl_certificate      /home/user/domain.com/conf/ssl/domain.crt;
    ssl_certificate_key  /home/user/domain.com/conf/ssl/domain_nopass.key;

    # 协议优化(可选,优化https协议,增强安全性)
    ssl_protocols        TLSv1 TLSv1.1 TLSv1.2
    ssl_ciphers          ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
    ssl_prefer_server_ciphers  on;
    ssl_session_cache    shared:SSL:10m;
    ssl_session_timeout  10m;

    # 自动跳转到HTTPS
    if ($server_port = 80) {
        rewrite ^(.*)$ https://$host$1 permanent;
    }

    # 其他配置信息...
}
```

