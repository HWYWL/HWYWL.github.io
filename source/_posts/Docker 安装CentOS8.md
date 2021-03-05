---
title: Docker 安装CentOS8
date: 2020-06-03 15:31:44
tags: 
 - Docker
categories: 
 - OS
keywords: "CentOS,Docker"
description: Docker 安装CentOS8。
---

我本机使用的是Windows 10 Pro，因为有时候需要使用Linux，但是如果使用虚拟机的模式会麻烦，不仅要安装虚拟机还得去下载几个G的镜像。
所以这里使用docker安装CentOS8，下面是我们的docker版本。

![](https://hwy-figure-bed.oss-cn-hangzhou.aliyuncs.com/image/20200603153525.png)

目前最近的laster版本就是CentOS8的版本，如果你想使用其他版本也可以。

![](https://hwy-figure-bed.oss-cn-hangzhou.aliyuncs.com/image/20200603153651.png)

拉取最新的镜像：
```
docker pull centos
```

查看已下载的镜像：
```
λ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
centos              latest              470671670cac        4 months ago        237MB
```

用镜像创建一个容器:
```
docker run -d -p 5555:22 --name mycentos8 --privileged=true centos /usr/sbin/init
```

查看启动的容器：
```
λ docker ps -a
CONTAINER ID    IMAGE     COMMAND              CREATED       STATUS           PORTS                   NAMES
b066e3bc5eed    centos    "/usr/sbin/init"     4 hours ago   Up 15 minutes    0.0.0.0:5555->22/tcp    mycentos8
```

链接容器，进入容器内部：
```
λ docker exec -it mycentos8 /bin/bash
[root@b066e3bc5eed /]#
```

docker提供的镜像非常基础，缺少很多常用的命令,所以我们需要安装软件：
```
dnf install openssh* -y
dnf install passwd -y
yum -y install cracklib-dicts   
yum install vim
yum install net-tools.x86_64
```

我们先启动sshd：
```
systemctl start sshd
systemctl enable sshd
```

安装完成之后我们可以设置新密码，用于后续的ssh链接：
```
[root@b066e3bc5eed /]# passwd root
Changing password for user root.
New password: 
BAD PASSWORD: The password is a palindrome
Retype new password: 
passwd: all authentication tokens updated successfully.
```

我们可以使用**ifconfig**的命令查看IP地址：
```
[root@b066e3bc5eed /]# ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.2  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:ac:11:00:02  txqueuelen 0  (Ethernet)
        RX packets 27640  bytes 2381286 (2.2 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 34934  bytes 4167271 (3.9 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

因为启动容器的时候已经把**22**段口映射到了宿主机的**5555**端口，所以我们直接使用**127.0.0.1**就可以连接：
SSH远程终端工具我用的是国人开发的**FinalShell**,当然你使用**XShell**或者直接使用命令行都可以。

![](https://hwy-figure-bed.oss-cn-hangzhou.aliyuncs.com/image/20200603155817.png)

连接上之后就可以像普通Linux一样使用了

![](https://hwy-figure-bed.oss-cn-hangzhou.aliyuncs.com/image/20200603155932.png)


### 问题建议

- 联系我的邮箱：ilovey_hwy@163.com
- 我的博客：https://hwy.ac.cn
- GitHub：https://github.com/HWYWL