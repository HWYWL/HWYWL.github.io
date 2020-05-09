---
title: Docker nginx安装与配置挂载并实现负载均衡
date: 2018-07-27 10:25:36
tags: 
 - Docker
 - nginx
categories: 
 - Nginx
keywords: "Nginx,Docker"
description: docker安装nginx。
---
在Docker下载Nginx镜像</span>

docker pull nginx docker images

<!--more-->

### **<font color="#a5c6ce" style="" face="Arial Black">Docker nginx安装与配置挂载并实现负载均衡</font>**

*   <span style="outline-style: initial; outline-width: 0px; font-weight: 700; word-break: break-all;">在Docker下载Nginx镜像</span>

    docker pull nginx
    docker images

![这里写图片描述](https://img-blog.csdn.net/2018070213224659?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI2NjQxNzgx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

*   <span style="outline-style: initial; outline-width: 0px; font-weight: 700; word-break: break-all;">创建挂载目录</span>

    mkdir -p /data/nginx/{conf,conf.d,html,logs}

![这里写图片描述](https://img-blog.csdn.net/20180702132449202?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI2NjQxNzgx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

*   <span style="outline-style: initial; outline-width: 0px; font-weight: 700; word-break: break-all;">编写nginx,conf配置文件，并放在文件夹中</span>

    # For more information on configuration, see:
    #   * Official English Documentation: http://nginx.org/en/docs/
    #   * Official Russian Documentation: http://nginx.org/ru/docs/

    user nginx;
    worker_processes auto;
    error_log /var/log/nginx/error.log;
    pid /run/nginx.pid;

    # Load dynamic modules. See /usr/share/nginx/README.dynamic.
    include /usr/share/nginx/modules/*.conf;

    events {
        worker_connections 1024;
    }

    http {
        log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                          '$status $body_bytes_sent "$http_referer" '
                          '"$http_user_agent" "$http_x_forwarded_for"';

        access_log  /var/log/nginx/access.log  main;

        sendfile            on;
        tcp_nopush          on;
        tcp_nodelay         on;
        keepalive_timeout   65;
        types_hash_max_size 2048;

        include             /etc/nginx/mime.types;
        default_type        application/octet-stream;

        # Load modular configuration files from the /etc/nginx/conf.d directory.
        # See http://nginx.org/en/docs/ngx_core_module.html#include
        # for more information.
        include /etc/nginx/conf.d/*.conf;

        server {
            listen       80 default_server;
            listen       [::]:80 default_server;
            server_name  182.254.161.54;
            root         /usr/share/nginx/html;

            # Load configuration files for the default server block.
            include /etc/nginx/default.d/*.conf;

            location / {
            proxy_pass http://pic; 
            }

            error_page 404 /404.html;
                location = /40x.html {
            }

            error_page 500 502 503 504 /50x.html;
                location = /50x.html {
            }
        }

        upstream pic{
                    server 182.254.161.54:8088 weight=5;
                    server 182.254.161.54:8089 weight=5;
        }

    }

*   <span style="outline-style: initial; outline-width: 0px; font-weight: 700; word-break: break-all;">启动容器</span>

    docker run --name mynginx -d -p 82:80  -v /data/nginx/conf/nginx.conf:/etc/nginx/nginx.conf  -v /data/nginx/logs:/var/log/nginx -d docker.io/nginx

*   <span style="outline-style: initial; outline-width: 0px; font-weight: 700; word-break: break-all;">查看启动的容器</span>

    docker ps 

![这里写图片描述](https://img-blog.csdn.net/20180702132832236?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI2NjQxNzgx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

*   <span style="outline-style: initial; outline-width: 0px; font-weight: 700; word-break: break-all;">先前已经在Docker部署两个tomcat，一个是8088端口，另一个是8089端口，并进入两个容器里编写了简单的页面</span>

![这里写图片描述](https://img-blog.csdn.net/20180702133106139?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI2NjQxNzgx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

<span style="outline-style: initial; outline-width: 0px; font-weight: 700; word-break: break-all;">访问8088端口</span>   
![这里写图片描述](https://img-blog.csdn.net/20180702133728495?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI2NjQxNzgx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

<span style="outline-style: initial; outline-width: 0px; font-weight: 700; word-break: break-all;">访问8089端口</span>   
![这里写图片描述](https://img-blog.csdn.net/20180702133805719?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI2NjQxNzgx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

*   <span style="outline-style: initial; outline-width: 0px; font-weight: 700; word-break: break-all;">现在通过Nginx访问两个tomcat的内容，实现负载均衡的功能，出于区别，更能体现负载均衡的功能，两个页面的内容不一样，但是访问路径都一样，只是通过Nginx反向代理去轮换访问</span>

![这里写图片描述](https://img-blog.csdn.net/20180702133452332?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI2NjQxNzgx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![这里写图片描述](https://img-blog.csdn.net/20180702133509870?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI2NjQxNzgx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)