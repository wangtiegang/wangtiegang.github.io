---
layout:     post
title:      "nginx配置https免费证书及更新"
subtitle:   ""
date:       2020-07-20 21:23:00
author:     "wangtiegang"
header-img: ""
catalog: false
tags:
    - linux
---

#### 申请免费证书

目前常用的免费证书一般是从 ```Let's Encrypt v2``` 获取，申请之后可以拿到证书、中间证书、私钥三个字符串。此处省略申请过程。

#### 配置nginx证书

登录服务器，进入到 nginx 的配置目录，新建三个配置文件。

* ```xx.xxx.com.crt``` 证书

该证书其实就是将证书和中间证书合并为一个文件

```
# 将证书字符串复制到这
# 将中间证书字符串复制到这
```

* xx.xxx.com.key 

```
# 将私钥字符串复制到这
```

* https_xx.xxx.com.conf

```
    # 证书路径
    ssl_certificate /etc/nginx/xx.xxx.com.crt;
    # 私钥路径
    ssl_certificate_key /etc/nginx/xx.xxx.com.key;
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:50m;
    ssl_session_tickets off;

    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers 'ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES256-SHA:ECDHE-ECDSA-DES-CBC3-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:DES-CBC3-SHA:!DSS';
    ssl_prefer_server_ciphers on;
```

* 在域名配置文件中使用 ssl

```
# 省略其他配置
server {
    listen 443 ssl;
    # 必须为证书的子域名
    server_name xx.xx.xxx.com;
    # ssl 证书配置文件
    include https_xx.xxx.com.conf;
    # ...
}
```

* 重启服务，登录网址就可以看到浏览器已经显示安全网址了

#### 更新免费证书

因为免费 ssl 证书只有三个月有效期，所以得定时更新证书，如果每次都手动更新，就太麻烦了，所以可以利用 crontab 定时脚本去更新证书

*  ```Let's Encrypt v2``` 下载官方更新脚本，或者自己定义脚本

* 配置 crontab 

```
0 8 * * * root python /etc/nginx/auto_update_ssl_nginx.py -t CBebTdW4u -d /etc/nginx/xx.xxx.com.crt -k /etc/nginx/xx.xxx.com.key -r yes >> /etc/nginx/auto_update_ssl_nginx.log 2>&1
```
