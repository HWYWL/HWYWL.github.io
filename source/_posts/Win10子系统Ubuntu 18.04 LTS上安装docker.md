---
title: Win10子系统Ubuntu 18.04 LTS上安装docker
date: 2020-03-20 16:00:53
tags: 
 - Docker
 - Linux
categories: 
 - Docker
keywords: "Linux,Docker"
description: Win10子系统Ubuntu 18.04 LTS上安装docker。
---
子系统安装docker挺麻烦的，这篇文章也是为了做个记录，以后有需要可以翻出来瞅瞅。
这里我使用的是Ubuntu 18.04 LTS的版本，可以在应用商店搜索**Linux**下载。

![](https://s1.ax1x.com/2020/03/20/8gKiUH.png)

我们要先去docker的官网下载一个Windows的docker版本：
```
https://hub.docker.com/?overlay=onboarding
```
![8gKNrT.png](https://s1.ax1x.com/2020/03/20/8gKNrT.png)

## Linux配置
Windows 子系统Ubuntu 18.04 LTS和docker桌面版下载完毕之后，我们启动子系统，配置一些参数。

为了方便后续使用ssh工具连接linux，需要先安装ssh服务：
```
sudo apt-get install openssh-server
sudo apt-get install openssh-client
```

修改sshd配置文件
```
vim /etc/ssh/sshd_config
```
修改内容包含下面的几个配置：
```
ListenAddress 0.0.0.0 # 取消注释
#StrictModes yes # 注释
PasswordAuthentication yes # 允许密码登录
```

启动ssh,ssh服务不会跟随子系统自动启动这个比较麻烦，每次启动子系统都需要执行启动的命令
```
service ssh start
```
如果提示**sshd error: could not load host key**，则用下面的命令重新生成
```
sudo rm /etc/ssh/ssh*key
dpkg-reconfigure openssh-server
```

shell软件推荐国产的**FinalShell**，美观又好用。

## 安装docker
我们先清一下老版本的docker，虽然我们知道没有，但官方让我们清一下
```
sudo apt-get remove docker docker-engine docker.io containerd runc
```
在安装 Docker 引擎之前，您需要设置 Docker 存储库。之后，可以从存储库安装和更新 Docker。
```
sudo apt-get update
```

安装包以允许通过 HTTPS 使用存储库:
```
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
```

添加 Docker 的官方 GPG 密钥：
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

通过搜索指纹的最后8个字符，验证您现在是否拥有带有指纹的钥匙。
```
sudo apt-key fingerprint 0EBFCD88

出现类似以下信息，说明是正确的
pub   rsa4096 2017-02-22 [SCEA]
      9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid           [ unknown] Docker Release (CE deb) <docker@docker.com>
sub   rsa4096 2017-02-22 [S]

```

使用以下命令设置稳定存储库：
```
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

更新包索引:
```
sudo apt-get update
```

安装最新版本的 Docker 引擎：
```
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

看我们是否安装成功：
```
docker -v

出现以下类似的信息就说明安装成功了

Docker version 19.03.8, build afacb8b7f0
```

到此以你为就安装成功了，太天真了（我以前就是那么天真）。

启动服务：
```
sudo service docker start

它会显示
* Starting Docker：docker		[OK]
```
然而这并不可以用！！！，我来看一下现在docker的状态
```
sudo service docker status

会发现他出现这样一行字
* Docker is not running
```

如果我们这个时候使用docker拉一个**hello-world**,它会蹦出一句话
```
docker: Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?.
```

**windows10子系统有其特殊性，需要安装docker for windows，用来作为Docker的守护进程(docker daemon)，作为Docker的服务端，ubuntu下作为客户端去访问这个守护进程，这也就是为什么要下载docker for window的原因**

## 安装 Docker for Windows
Typer-V如果没有开启，启动软件的时候会报错，这里开启，重启电脑即可

![8gJZBq.png](https://s1.ax1x.com/2020/03/20/8gJZBq.png)

然后开启CPU的虚拟化

![8gJBgH.png](https://s1.ax1x.com/2020/03/20/8gJBgH.png)

如果发现是禁用的，需要去BOIS设置开启虚拟化，具体可以自行百度很多文档。

安装没什么好说的，直接使用默认就好，成功启动之后，会引导你登录官方Docker的账号(不登录也可以)，同时电脑的右下方会有个小鲸鱼，右键小鲸鱼，安装完成之后我们需要设置一些东西。

![8gtq9U.png](https://s1.ax1x.com/2020/03/20/8gtq9U.png)

我们需要把守护进程勾起来

![8gNAjH.png](https://s1.ax1x.com/2020/03/20/8gNAjH.png)


然后切换到我们linux中执行以下命令，让子进程链接宿主机Docker守护进程（每次重启系统需要刷一遍，也挺麻烦）
```
echo "export DOCKER_HOST='tcp://0.0.0.0:2375'" >> ~/.bashrc
source ~/.bashrc
```

再次使用查看版本命令，可以发现多了很多东西
```
docker version
```
这是以前的状态
![8gUuL9.png](https://s1.ax1x.com/2020/03/20/8gUuL9.png)

这是子进程链接宿主机Docker守护进程后的状态
![8gU7SU.png](https://s1.ax1x.com/2020/03/20/8gU7SU.png)

现在就真的可以启动服务了：
```
sudo service docker start

它会显示
* Starting Docker：docker		[OK]
```

看一下现在docker的状态
```
sudo service docker status

可能还是会出现docker未运行的状态，但是现在已经可以使用了，我们来运行一个测试程序
* Docker is not running
```

拉取一个hello world测试程序：
```
docker pull hello-world
```

执行我们拉取的hello-world
```
docker run hello-world
```

出现以下信息说明已经成功了，不容易呀
```
root@huangwenyi:~# docker run hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```