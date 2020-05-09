---
title: Nginx基础配置
date: 2018-08-24 10:25:36
tags: 
 - nginx
categories: 
 - Nginx
keywords: "Nginx"
description: Nginx基础配置。
---
# Nginx基础配置
以我的博客为例我们分为两种配置，一种普通的反向代理http，另一种是https

### 配置http
```
server {
    listen 80;
    server_name www.hwy.ac.cn; #将 www.yourdomain.com 替换为之前注册并解析的域名
    root /root/firekylin;
    set $node_port 此处替换为项目端口号;

    index index.js index.html index.htm;

    location ^~ /.well-known/acme-challenge/ {
      alias /root/firekylin/ssl/challenges/;
      try_files $uri = 404;
    }

    location / {
        proxy_http_version 1.1;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_set_header X-NginX-Proxy true;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_pass http://127.0.0.1:$node_port$request_uri;
        proxy_redirect off;
    }

    location = /development.js {
        deny all;
    }
    location = /testing.js {
        deny all;
    }

    location = /production.js {
        deny all;
    }
}
```

### 配置https
```
server{
    listen 443;
    server_name hwy.ac.cn www.hwy.ac.cn;
    root /root/firekylin;
    set $node_port 此处替换为项目端口号;

    ssl on;
    ssl_certificate  /xxx/xxx/xxx.crt;
    ssl_certificate_key /xxx/xxx/xxx.key;

    ssl_session_timeout 5m;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA256:ECDHE-RSA-AES256-SHA:ECDHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA;
    ssl_session_cache shared:SSL:50m;
    ssl_dhparam %path/ssl/dhparams.pem;
    ssl_prefer_server_ciphers on;


    index index.js index.html index.htm;

    location ^~ /.well-known/acme-challenge/ {
      alias /root/firekylin/ssl/challenges/;
      try_files $uri = 404;
    }

   location / {
        proxy_http_version 1.1;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_set_header X-NginX-Proxy true;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_pass http://127.0.0.1:$node_port$request_uri;
        proxy_redirect off;
    }

    location = /development.js {
        deny all;
    }

    location = /testing.js {
        deny all;
    }

    location = /production.js {
        deny all;
   }
}
server {
    listen 80;
    server_name hwy.ac.cn www.hwy.ac.cn;
    rewrite ^(.*) https://hwy.ac.cn$1 permanent;
}
```

其他项目可以参考配置，大部分配置是一样的。