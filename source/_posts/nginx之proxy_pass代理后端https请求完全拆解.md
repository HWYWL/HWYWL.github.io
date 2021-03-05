---
title: nginx之proxy_pass代理后端https请求完全拆解
date: 2019-02-18 09:25:36
tags: 
 - 反向代理
categories: 
 - Nginx
keywords: "Nginx"
description: nginx之proxy_pass代理后端https请求完全拆解。
---

# 前言
本文解释了怎么对nginx和后端服务器组或代理服务器进行加密http通信。

### 获取SSL服务端证书
你可以从一个可信证书颁发机构(CA)购买一个服务器证书, 或者你可以使用openssl库创建一个内部CA， 并给自己颁发证书。这个服务器端证书和私钥需要部署在后端的每一个服务器上。

### 获取SSL客户端证书
nignx使用一个SSL客户端证书来对后端服务器组来标识自己。这个客户端证书必须是被一个可信CA签名的，并且和相匹配的私钥一起部署在nginx中。
你还需要在后端服务器上配置好所有的来源SSL连接都需要客户端证书，并信任这个CA颁发的nginx客户端证书。 然后当nginx连接后端时，将提供客户端证书，并且后端将会接收这个连接。

### 配置nginx
首先，改变相应URL到支持SSL连接的后端服务器组。在nginx的配置文件中，指明proxy_pass指令在代理服务器或后端服务器组中使用"https"协议:
```nginx
location /upstream {
    proxy_pass https://backend.example.com;
}
```

增加客户端证书和私钥，用于验证nginx和每个后端服务器。使用proxy_ssl_certificate 和 proxy_ssl_certificate_key指令:
```nginx
location /upstream {  
    proxy_pass                https://backend.example.com;  
    proxy_ssl_certificate     /etc/nginx/client.pem;  
    proxy_ssl_certificate_key /etc/nginx/client.key  
}  
```

如果你在后端服务器使用了自签名证书或者使用了自建CA，你需要配置proxy_ssl_trusted_certificate. 这个文件必须是PEM格式的。另外还可以配置proxy_ssl_verify和proxy_ssl_verfiy_depth指令， 用来验证安全证书：

```nginx
location /upstream {
    ...
    proxy_ssl_trusted_certificate /etc/nginx/trusted_ca_cert.crt;
    proxy_ssl_verify       on;
    proxy_ssl_verify_depth 2;
    ...
}
```

每一个新的SSL连接都需要在服务端和客户端进行一个完整的SSL握手过程，这非常耗费CPU计算资源。为了是nignx代理预先协商连接参数，使用一种简略的握手过程，增加proxy_ssl_session_reuse指令配置:

```nginx
location /upstream {
    ...
    proxy_ssl_session_reuse on;
    ...
}
```

可选的，你也可以配置使用的SSL协议和SSL秘钥算法:
```nginx
location /upstream {
        ...
        proxy_ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        proxy_ssl_ciphers   HIGH:!aNULL:!MD5;
}
```

### 配置后端服务器
每一个后端服务器都必须配置成接受https连接。每一个后端服务器需要使用ssl_certificate和ssl_certificate_key指令来指定服务器证书和私钥的文件路径:
```nginx
server {
    listen              443 ssl;
    server_name         backend1.example.com;

    ssl_certificate     /etc/ssl/certs/server.crt;
    ssl_certificate_key /etc/ssl/certs/server.key;
    ...
    location /yourapp {
        proxy_pass http://url_to_app.com;
        ...
    }
}
```

使用ssl_client_certificat指令来设定客户端证书的文件路径:
```nginx
server {
    ...
    ssl_client_certificate /etc/ssl/certs/ca.crt;
    ssl_verify_client      off;
    ...
}
```

### 完整示例
```nginx
http {
    ...
    upstream backend.example.com {
        server backend1.example.com:443;
        server backend2.example.com:443;
   }

    server {
        listen      80;
        server_name www.example.com;
        ...

        location /upstream {
            proxy_pass                    https://backend.example.com;
            proxy_ssl_certificate         /etc/nginx/client.pem;
            proxy_ssl_certificate_key     /etc/nginx/client.key
            proxy_ssl_protocols           TLSv1 TLSv1.1 TLSv1.2;
            proxy_ssl_ciphers             HIGH:!aNULL:!MD5;
            proxy_ssl_trusted_certificate /etc/nginx/trusted_ca_cert.crt;

            proxy_ssl_verify        on;
            proxy_ssl_verify_depth  2;
            proxy_ssl_session_reuse on;
        }
    }

    server {
        listen      443 ssl;
        server_name backend1.example.com;

        ssl_certificate        /etc/ssl/certs/server.crt;
        ssl_certificate_key    /etc/ssl/certs/server.key;
        ssl_client_certificate /etc/ssl/certs/ca.crt;
        ssl_verify_client      off;

        location /yourapp {
            proxy_pass http://url_to_app.com;
        ...
        }

    server {
        listen      443 ssl;
        server_name backend2.example.com;

        ssl_certificate        /etc/ssl/certs/server.crt;
        ssl_certificate_key    /etc/ssl/certs/server.key;
        ssl_client_certificate /etc/ssl/certs/ca.crt;
        ssl_verify_client      off;

        location /yourapp {
            proxy_pass http://url_to_app.com;
        ...
        }
    }
}
```

在这个示例中， proxy_pass指令设置使用了"https"协议，所以nginx转发到后端服务器的流量是安全的。

当一个安全的连接第一次从nginx转发到后端服务器，将会实施一次完整的握手过程。
- proxy_ssl_certificate指令设置了后端服务器需要的PEM格式证书的文件位置。
- proxy_ssl_certificate_key指令设置了证书的私钥位置。proxy_ssl_protocols和
- proxy_ssl_ciphers指令控制使用的协议和秘钥算法。

因为proxy_ssl_session_reuse指令配置，当下一次nginx转发一个连接到后端服务器时，会话参数会被重复使用，从而更快的建立安全连接。

**proxy_ssl_trusted_certificate**指令设置的那个可信CA证书文件是用来验证后端服务器的证书。

**proxy_ssl_verify_depth**指令指定了证书链检查的深度。proxy_ssl_verify指令验证证书的有效性。