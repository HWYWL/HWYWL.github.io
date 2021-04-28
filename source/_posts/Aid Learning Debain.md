---
title: Aid Learning 让你的废旧手机成为Linux服务器
date: 2021-04-28 09:57:56
tags: 
 - 操作系统
 - Linux
categories: 
 - Linux
keywords: "Linux,分区"
description: Aid Learning 让你的废旧手机成为Linux服务器。
---

## 平台介绍

- AidLearning是一个强大的移动端的AI开发平台，它几乎支持所有深度学习神经网络开发的框架和工具。
- 移动端（手机）上最好的，环境最全的Linux模拟器，唯一支持图形化桌面的Linux模拟器…
- 唯一支持AI开发环境的模拟器、内置全球最流行Top 7的深度学习框架，内置大量深度学习的模型、例子和开发组件
-  唯一支持python图形化开发和调试的模拟器，支持触摸拖拽式界面设计，提高你的开发效率
- 支持用python开发可运行在手机的App，支持python代码直接编译生成可部署的apk文件
- 一键式安装，无任何依赖，你只需在手机上要安装一个10M的引导App，就可以自动完成所有环境的安装。
- 跨平台开发，支持云桌面（手机桌面和电脑桌面相同），既可以在手机或平版上或其他嵌入式主板上运行，也可以在电脑端基于web直接访问和开发。
- 支持加速库openblas，支持多线程和多进程，运行流畅、不卡顿，充分发挥ARM CPU和GPU的算力
- 最新版本支持python直接调用手机的gpu加速，一般深度学习的tflite模型，30fps～80fps，轻松达到（在主流手机上）

通过**lsb_release -a**命令我们可以发现它是一个**Debian 10**的发行版
```
demo@localhost:~/coder$ lsb_release -a
No LSB modules are available.
Distributor ID: Debian
Description:    Debian GNU/Linux 10 (buster)
Release:        10
Codename:       buster
demo@localhost:~/coder$ 
```

## 软件安装
### 手机软件安装
手机安装完成**AidLearning**之后点击软件进入，根据提示它会自动安装linux，时间会比较久，可以耐心等待。安装完毕之后可以点击页面上的
**Cloud_ip**那个应用。此时如果你的电脑和手机在同一个网络可以直接在电脑上访问显示的**IP地址**，通过电脑操作。

![gPSEUf.jpg](https://z3.ax1x.com/2021/04/28/gPSEUf.jpg)

电脑浏览器访问，到此一个Linux安装完成

![gPSQrn.png](https://z3.ax1x.com/2021/04/28/gPSQrn.png)

### Linux常用软件安装
刚安装完成的Linux其实是欠缺很多软件的，比如ssh、sudo、curl、lrzsz等等。
安装这些软件之前我们先使用网页版的终端命令行来更新一个自带的软件包。
```
# 更新软件包
apt-get update
apt-get upgrade

# 安装sudo
apt-get install sudo

# 安装curl
apt-get install curl

# 安装lrzsz
apt-get install lrzsz
```

### 安装ssh服务
```
# 安装ssh
apt-get install ssh

# 修改sshd_config文件
vim /etc/ssh/sshd_config

1.将#PasswordAuthentication no的注释去掉，并且将no修改为yes
2.将#PermitRootLogin prohibit-password的注释去掉，将prohibit-password改为yes
PasswordAuthentication yes
PermitRootLogin yes
或echo -e "PasswordAuthentication yes\nPermitRootLogin yes" >> /etc/ssh/sshd_config
3.启动SSH服务，命令为：/etc/init.d/ssh start // 或者service ssh start
4.验证SSH服务状态，命令为：/etc/init.d/ssh status
5. 添加开机自启动 update-rc.d ssh enable
```

系统出了默认的root用户之外，还有一个demo用户，可以使用这个用户登陆ssh，不知道为何ssh的root登陆不上去。
```
用户名：demo
密码：demo
```


### 安装JDK8
通过上面软件更新之后应该是会有一个jdk 11的版本，但是目前国内用的最多的还是JDK 8。
```
# 自带的JDK版本
demo@localhost:~$ java -version
openjdk version "11.0.11" 2021-04-20
OpenJDK Runtime Environment (build 11.0.11+9-post-Debian-1deb10u1)
OpenJDK 64-Bit Server VM (build 11.0.11+9-post-Debian-1deb10u1, mixed mode)
```

```
# 安装JDK 8
sudo apt install apt-transport-https ca-certificates wget dirmngr gnupg software-properties-common

wget -qO - https://adoptopenjdk.jfrog.io/adoptopenjdk/api/gpg/key/public | sudo apt-key add -

sudo add-apt-repository --yes https://adoptopenjdk.jfrog.io/adoptopenjdk/deb/

sudo apt update

sudo apt install adoptopenjdk-8-hotspot
```

在命令行终端中输入如下命令，检查安装结果。
```
# 此时系统上显示的还是JDK 11的版本
java -version
```

切换默认JDK版本
```
# 默认JDK切换命令
sudo update-alternatives --config java

# 输出列表如下：
demo@localhost:~$ sudo update-alternatives --config java
There are 2 choices for the alternative java (providing /usr/bin/java).

  Selection    Path                                                Priority   Status
------------------------------------------------------------
* 0            /usr/lib/jvm/java-11-openjdk-arm64/bin/java          1111      auto mode
  1            /usr/lib/jvm/adoptopenjdk-8-hotspot-arm64/bin/java   1081      manual mode
  2            /usr/lib/jvm/java-11-openjdk-arm64/bin/java          1111      manual mode

Press <enter> to keep the current choice[*], or type selection number: 
```

输入编号1然后回车，这是OpenJDK 8 就为当前默认版本，在此输入**java -version**查看输出，如
```
demo@localhost:~$ java -version
openjdk version "1.8.0_292"
OpenJDK Runtime Environment (build 1.8.0_292-b10)
OpenJDK 64-Bit Server VM (build 25.292-b10, mixed mode)
demo@localhost:~$ 
```

卸载默认的JDK
```
sudo apt remove default-jdk
```
卸载刚装上的JDK8
```
sudo apt remove adoptopenjdk-8-hotspot
```

### 运行Java程序
这里使用随便新建一个Spring Boot项目，使用**rz**命令把**jar**包上传到Linux，
```
# 启动jar
java -jar xxx.jar
```

![gPFxeJ.png](https://z3.ax1x.com/2021/04/28/gPFxeJ.png)

完美运行。

## 其他说明
系统还自带了Python，所以不需要自己安装
```
demo@localhost:~$ python --version
Python 3.7.3
demo@localhost:~$ pip --version
pip 18.1 from /usr/lib/python3/dist-packages/pip (python 3.7)
demo@localhost:~$ 
```