---
title: 使用IDEA插件Alibaba Cloud Toolkit工具一键部署本地应用到ECS服务器
date: 2018-12-13 08:50:08
tags: 
 - IDEA
 - Alibaba Cloud Toolkit
categories: 
 - 其他
keywords: "IDEA,其他"
description: 使用IDEA插件Alibaba Cloud Toolkit工具一键部署本地应用到ECS服务器。
---


# 使用IDEA插件Alibaba Cloud Toolkit工具一键部署本地应用到ECS服务器

最近看到阿里云发布了一款名为 Alibaba Cloud Toolkit 的插件，可以帮助开发者高效开发并部署适合在云端运行的应用，瞬间击中了我的小心脏，这个对于个人开发者来说超级棒啊，终于不需要再手动 scp/ftp 上传应用到服务器了，连启动的命令都是可以自行编写的，棒棒！
PS：个人开发者项目不多也不大，如果使用jenkins等工具会比较麻烦，不如直接用手扔来得直接。

## 什么是 Alibaba Cloud Toolkit

Alibaba Cloud Toolkit （后文简称 Cloud Toolkit）是阿里云针对 IDE 平台为开发者提供的一款插件，用于帮助开发者高效开发并部署适合在云端运行的应用。
您在本地完成应用程序的开发、调试和测试后，可以使用在 IED （如 Eclipse 或 IntelliJ）中安装的 Cloud Toolkit 插件，通过图形配置的方式连接到云端部署环境并将应用程序快如部署到云端。
说明：目前 Cloud Toolkit 仅支持 Eclipse、Intellij 等其它开发环境开发中，请您持续关注 Cloud Tookit 动态。
官方有提供简单版的文档说明，小伙伴也可以参考下面链接：
```
https://help.aliyun.com/product/29966.html
```
## 使用IDEA安装和配置Cloud Toolkit

作者手动在idea上安装了一下这个工具，并测试完成，对这个工具可以说非常满意，下面是安装和配置的流程，主要有以下几步：
- 在idea上安装Alibaba Cloud Toolkit插件
- 重启idea应用
- 配置Cloud Toolkit插件中的Accout信息
- 在阿里云中获取用户AccessKey相关信息（AccessKey ID、	Access Key Secret）
- 配置发布到ECS的相关服务器及命令信息
- 测试并成功发布

主要流程为以上6步，下面我们一步步来配置，上图：
![](https://i.imgur.com/hPX0T0T.jpg)

如果插件下载速度比较慢，稍等一会，作者测试时也下载失败了一次，下载完成后需要重启idea应用后生效。

首先，需要先配置Alibaba Cloud Toolkit的Account，位置见下图：
![](https://i.imgur.com/e2NM5oN.jpg)

![](https://i.imgur.com/4BDmwJN.jpg)

上图中的AccessKey需要在阿里云的控制台中配置，如果是新用户，需要手动创建一个AccessKey，如下图：
![](https://i.imgur.com/V7KK3kf.jpg)

创建完成并配置好Account后，就可以着手配置对应的项目发布到ECS信息，官方文档见以下链接：
```
https://help.aliyun.com/document_detail/98762.html
```
![](https://i.imgur.com/1uSOgIR.jpg)

如果你的Account配置没有问题，则会自动账户显示对应的ECS服务器，在发布时，需要手动选择某台服务器，一定要选择哦！

对于Command的编写，可以参考官方文档（点击下图中的蓝色字体：Learn Sample直达）：
```
https://yq.aliyun.com/articles/665693
```
![](https://i.imgur.com/RRd7HIQ.jpg)

配置成功后，可以点击Run运行程序，此时会自动为我们编译并上传到阿里云服务器中，发布到地址就是上图中的Deploy Location中的路径，发布前如果需要Maven执行，一定不要忘记配置上图中Maven的命令，中间的Command是在上传到服务器成功后执行的命令，主要用于应用的启动停止重启等。
下面是发布成功的示例：

![](https://i.imgur.com/s9dr40F.jpg)

服务器的显示结果如下：

![](https://i.imgur.com/25pm03e.jpg)

## 结语
以上是对IDEA插件Alibaba Cloud Toolkit的安装配置及使用案例，如果小伙伴还有遇到其他的问题，可以根据一下链接，加入Alibaba Cloud Toolkit 官方唯一指定支持群，提交你的需求&Bug哦。
```
https://yq.aliyun.com/articles/656292
```

```

作者：YClimb
链接：https://juejin.im/post/5c0a748ff265da610f6389a8
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```
