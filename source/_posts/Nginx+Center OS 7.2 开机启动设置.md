---
title: Nginx+Center OS 7.2 开机启动设置
date: 2019-05-16 11:18:31
tags: 
 - nginx
 - Linux
categories: 
 - Nginx
keywords: "Nginx"
description: Nginx+Center OS 7.2 开机启动设置。
---

centos 7以上是用Systemd进行系统初始化的，Systemd 是 Linux 系统中最新的初始化系统（init），它主要的设计目标是克服 sysvinit 固有的缺点，提高系统的启动速度。关于Systemd的详情介绍在[这里](www.yourlink.com)。

Systemd服务文件以.service结尾，比如现在要建立nginx为开机启动，如果用yum install命令安装的，yum命令会自动创建nginx.service文件，直接用命令：
```
systemcel enable nginx.service
```
设置开机启动即可。

在这里我是用源码编译安装的，所以要手动创建nginx.service服务文件。
开机没有登陆情况下就能运行的程序，存在系统服务（system）里，即：
```
/lib/systemd/system/
```

### 在系统服务目录里创建nginx.service文件
```
vim /lib/systemd/system/nginx.service
```
内容如下:
```
[Unit]
Description=nginx
After=network.target
  
[Service]
Type=forking
ExecStart=/usr/local/nginx/sbin/nginx
ExecReload=/usr/local/nginx/sbin/nginx -s reload
ExecStop=/usr/local/nginx/sbin/nginx -s quit
PrivateTmp=true
  
[Install]
WantedBy=multi-user.target
```

- [Unit]:服务的说明
- Description:描述服务
- After:描述服务类别
- [Service]服务运行参数的设置
- Type=forking是后台运行的形式
- ExecStart为服务的具体运行命令
- ExecReload为重启命令
- ExecStop为停止命令
- PrivateTmp=True表示给服务分配独立的临时空间

注意：[Service]的启动、重启、停止命令全部要求使用绝对路径
[Install]运行级别下服务安装的相关设置，可设置为多用户，即系统运行级别为3

保存退出。

### 设置开机启动
```
systemctl enable nginx.service
```

### 其他命令
启动nginx服务
```
systemctl start nginx.service　
```

设置开机自启动
```
systemctl enable nginx.service
```

停止开机自启动
```
systemctl disable nginx.service
```

查看服务当前状态
```
systemctl status nginx.service
```

重新启动服务
```
systemctl restart nginx.service　
```

查看所有已启动的服务
```
systemctl list-units --type=service
```

### Systemd 命令和 sysvinit 命令的对照表
![alt](https://i.loli.net/2019/05/16/5cdd17a24c56278117.png)

### Sysvinit 运行级别和 systemd 目标的对应表
![alt](https://i.loli.net/2019/05/16/5cdd17b62eaf678001.png)
