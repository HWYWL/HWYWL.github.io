---
title: 使用docker安装kafka
date: 2019-08-30 09:16:31
tags: 
 - 消息队列
 - kafka
categories: 
 - 消息队列
keywords: "消息队列,kafka"
description: 使用docker安装kafka。
---

### 下载kafkaManager
源码下载地址：https://github.com/yahoo/kafka-manager/
下载源码，然后上传解压准备编译


```
cd /export/servers/kafka-manager-1.3.3.15
unzip kafka-manager-1.3.3.15.zip -d  ../servers/
./sbt clean dist
```

编译完成之后，我们需要的安装包就在这个路径之下

```
/export/servers/kafka-manager-1.3.3.15/target/universal
```

将我们编译好的kafkamanager的压缩包解压到指定目录
```
cd  /export/servers/kafka-manager-1.3.3.15/target/universal
unzip kafka-manager-1.3.3.15.zip -d /export/servers/
```

修改配置文件

```
cd /export/servers/kafka-manager-1.3.3.15/
vim  conf/application.conf
```
配置修改为如下
```
kafka-manager.zkhosts="node01:2181,node02:2181,node03:2181"
```

为kafkamanager的启动脚本添加执行权限
```
cd /export/servers/kafka-manager-1.3.3.15/bin
chmod u+x ./*
```

启动kafkamanager进程
```
cd /export/servers/kafka-manager-1.3.3.15
nohup bin/kafka-manager  -Dconfig.file=/export/servers/kafka-manager-1.3.3.15/conf/application.conf -Dhttp.port=8070   2>&1 &
```

浏览器页面访问
http://node01:8070/

在页面中的Cluster配置好指定的kafka就可以监控了

![](https://s2.ax1x.com/2020/02/26/3Ufphq.png)
