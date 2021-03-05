---
title: Nginx反向代理本地目录
date: 2018-08-24 10:11:36
tags: 
 - 反向代理
categories: 
 - Nginx
keywords: "Nginx"
description: Nginx反向代理本地目录。
---

```
server {
    listen 80;
    server_name www.avenger.com;    //在host中把这个地址设置为127.0.0.1
    index index.html index.htm;

    autoindex on;
    autoindex_exact_size on;
    autoindex_localtime on;

    location / {
        root D:/work/code/fed-static;
    }
}
```
