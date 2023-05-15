---
title: 基于acme.sh脚本自动化白嫖SSL证书
date: 2023-05-15 10:25:35
tags: 
 - nginx
categories: 
 - nginx
keywords: "nginx,https,ssl"
description: 基于acme.sh脚本自动化白嫖SSL证书断机制。
---

## 简单介绍
acme.sh 实现了 acme 协议, 可以从 Let's Encrypt 生成免费的证书。支持一键脚本和 docker 部署.支持 http 和 DNS 两种域名验证方式。

## acme部署

通过使用域名服务商提供的 API 密钥,让acme.sh自动创建域名验证记录以申请域名证书. acme.sh 支持全球各种域名服务商的 API ,本文将以**腾讯云**（我的服务器在这）。其他例如阿里云、Cloudflare等 DNS API 支持，可以查看官方的文档：https://github.com/acmesh-official/acme.sh/wiki/dnsapi。


### 获取腾讯云dnsapi
登录DNSPod控制台访问api密钥https://console.dnspod.cn/account/token/apikey。获取DNSPod Token，**注意：是DNSPod Token，不是腾讯云API密钥**，如下截图。

![](https://hwy-figure-bed.oss-cn-hangzhou.aliyuncs.com/blog/image/1684118591585.png)

记录下id和token

```
export DP_Id="1234"
export DP_Key="sADDsdasdgdsf"
```

更多云服务商的填写方式请看官方文档：https://github.com/acmesh-official/acme.sh/wiki/dnsapi

![](https://hwy-figure-bed.oss-cn-hangzhou.aliyuncs.com/blog/image/1684118798209.png)

### 安装脚本
```
wget -O -  https://get.acme.sh | sh
```

### 配置 DNS API
acme.sh 程序目录为隐藏目录.acme.sh存放在root下.执行以下命令进入目录,并编辑account.conf,根据上文获取的 API 格式,复制粘贴到文件中保存.
```
cd ~/.acme.sh
vim account.conf
```

![](https://hwy-figure-bed.oss-cn-hangzhou.aliyuncs.com/blog/image/1684119098155.png)

直接将配置放在account.conf文件开头保存即可

![](https://hwy-figure-bed.oss-cn-hangzhou.aliyuncs.com/blog/image/1684119174714.png)

### 申请证书

```
acme.sh --issue --server letsencrypt --dns dns_dp -d example.com -d www.example.com
```
其中dns_dp为 腾讯云DNSPod.cn 服务商，自行根据官方dnsapi修改。例如:dns_ali为阿里云，dns_cf为CLoudflare。

复制
以我的的域名为例：

```
acme.sh --issue --server letsencrypt --dns dns_dp -d mj.hwy.ac.cn
```
当然你一次性申请多个域名的证书也是可以的只要遵循** -d 空格 域名 **的格式，如下:

```
acme.sh --issue --server letsencrypt --dns dns_dp -d abc.com -d *.abc.com -d def.com -d *.def.com
```

### 安装证书到指定文件夹
```
acme.sh --install-cert -d mj.hwy.ac.cn \
--key-file       /home/certificate/nginx/key.pem  \
--fullchain-file /home/certificate/nginx/cert.pem \
--reloadcmd     "/usr/local/nginx/sbin/nginx -s reload"
```
mj.hwy.ac.cn：申请的域名
key-file：密钥文件的存放路径（可以自定义）
fullchain-file：验证证书的存放路径（可以自定义）
reloadcmd：重启nginx的命令，这个根据你的nginx重启方式更改


安装完毕之后acme.sh一般会自动续签，你可以使用如下命令查看它的定时任务：
```
crontab -l
```

![](https://hwy-figure-bed.oss-cn-hangzhou.aliyuncs.com/blog/image/1684119957575.png)

### 配置nginx
接下来就是配置**nginx**对于你们老司机来说这就很简单了，找到你们nginx配置文件**nginx.conf**

在监听443那里配置上刚刚我们设置的文件路径即可：

```
ssl_certificate      /home/certificate/nginx/cert.pem;
ssl_certificate_key  /home/certificate/nginx/key.pem;
```

![](https://hwy-figure-bed.oss-cn-hangzhou.aliyuncs.com/blog/image/1684120130283.png)

最后在监听80端口配置那里设置一下转发：
```
server_name  mj.hwy.ac.cn;
rewrite ^(.*)$ https://${server_name}$1 permanent;
```

![](https://hwy-figure-bed.oss-cn-hangzhou.aliyuncs.com/blog/image/1684120291866.png)

保存文件，重启nginx，我的重启命令是：
```
/usr/local/nginx/sbin/nginx -s reload
```

等待一分钟左右，就可以在浏览器看到https了

![](https://hwy-figure-bed.oss-cn-hangzhou.aliyuncs.com/blog/image/1684120400555.png)

## 总结

快来使用acme.sh一起白嫖吧，这个脚本太方便了，感谢一直位它提供更新维护的各路大神，膜拜。