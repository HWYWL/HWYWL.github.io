---
title: Caddy 反向代理
date: 2023-02-07 16:49:39
tags: 
 - 反向代理
categories: 
 - caddy
keywords: "caddy"
description: Caddy 反向代理
---

## 基本介绍和使用

**Caddy** 是一款使用 **Go** 语言开发的 **Web** 服务器。其配置更为简洁，并可以自动申请及配置 **SSL** 证书、**OCSP** 装样、静态文件服务、反向代理、**Kubernetes** 入口等，强力推荐。

### 安装 Caddy
```
# 安装 Caddy 软件包
yum install caddy -y
```

### 配置 Caddy
```
# 下载 Halo 官方的 Caddy 配置模板
curl -o /etc/caddy/conf.d/Caddyfile.conf --create-dirs https://dl.halo.run/config/Caddyfile
```

下载完成之后，我们还需要对其进行修改。
```
# 使用 vim 编辑 Caddyfile
vim /etc/caddy/conf.d/Caddyfile.conf
```

打开之后我们可以看到
```
https://www.simple.com {
 gzip
 tls xxxx@xxx.xx
 proxy / localhost:port {
  transparent
 }
}
```

- 请把 https://www.simple.com 改为自己的域名。
- tls 后面的 xxxx@xxx.xx 改为自己的邮箱地址，这是用于自动申请 SSL 证书用的。需要注意的是，不需要你自己配置 SSL 证书，而且会自动帮你续签。
- localhost:port 请将 port 修改为你需要的运行端口，默认为 8090。

修改完成之后启动 **Caddy** 服务即可。
```
# 开启自启 Caddy 服务
systemctl enable caddy

# 启动 Caddy
service caddy start

# 停止运行 Caddy
service caddy stop

# 重启 Caddy
service caddy restart

# 查看 Caddy 运行状态
service caddy status
```

> 如果 **Caddy** 启动出现诸如 **[/usr/lib/systemd/system/caddy.service:23] Unknown lvalue 'AmbientCapabilities' in section 'Service'** 这样的问题，请使用 **yum update -y** 更新系统。然后再使用 **service caddy restart** 重启，已知 **CentOS 7.3** 会出现该问题。

## 进阶设置

多网址重定向到主网址，比如访问 simple.com 跳转到 www.simple.com
```
# 使用 vim 编辑 Caddyfile
vim /etc/caddy/conf.d/Caddyfile.conf
```

打开之后我们在原有的基础上添加以下配置
```
https://simple.com {
  redir https://www.simple.com{url}
}
```

将 **https://simple.com** 和 **https://www.simple.com{url}** 修改为自己需要的网址就行了，比如我要求访问 **hwy.ac.cn** 跳转到 **www.hwy.ac.cn**，完整的配置如下：
```
https://hwy.ac.cn {
  redir https://www.hwy.ac.cn{url}
}

https://www.hwy.ac.cn {
  gzip
  tls hwy@ac.cn
  proxy / localhost:8090 {
    transparent
  }
}
```

最后我们重启 **Caddy** 即可。

除此之外它还支持**自定义负载平衡和活动运行状况检查、JSONAPI更新配置等等。。。**