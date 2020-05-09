---
title: openVPN多配置文件管理
date: 2018-12-06 13:50:08
tags: 
 - openVPN
categories: 
 - 其他
keywords: "openVPN,其他"
description: openVPN多配置文件管理。
---


我们在使用openVPN的时候可能会有好几个不同的配置，此时如果需要切换使用默认会显得很麻烦，我们可以手动修改配置文件达到一键切换的目的。

首先把不同的配置按文件夹分类归类好，把其所在的**config.ovpn**文件**移动**到文件夹外面，如下：

![](https://i.imgur.com/aFvnCwR.jpg)

文件夹里面装的是我们生存的配置文件，比如我的那个经典网络文件A级里面：

![](https://i.imgur.com/0WPJQMF.jpg)

之后我们需要修改移动出来的**xxx.ovpn文件**，用记事本打开可能不会自动格式化就一行显示，看着不方便，所以我用Sunlime Text软件打开，把每个修改为如下的样子

![](https://i.imgur.com/6Z8r8eH.jpg)

然后我们启动**OpenVPN GUI**就可以选择我们所需要的网络了

![](https://i.imgur.com/WISrKsq.jpg)

我使用的是Windows 7系统 其他系统不知道信不信，需要自己测试。

