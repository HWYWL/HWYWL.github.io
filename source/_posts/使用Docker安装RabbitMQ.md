---
title: 使用Docker安装RabbitMQ
date: 2018-08-18 09:15:31
tags: 
 - Docker
 - RabbitMQ
categories: 
 - 消息队列
keywords: "Docker,RabbitMQ"
description: 使用Docker安装RabbitMQ。
---
获取rabbit镜像：
docker pull rabbitmq:management

<!--more-->
获取rabbit镜像：
```js
docker pull rabbitmq:management
```  

创建并运行容器：
```js
docker run -d --hostname my-rabbit --name rabbit -p 8080:15672 rabbitmq:management  
--hostname：指定容器主机名称  
--name:指定容器名称  
-p:将mq端口号映射到本地  

或在运行时设置用户和密码
docker run -d --hostname my-rabbit --name rabbit -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin -p 15672:15672 -p 5672:5672 -p 25672:25672 -p 61613:61613 -p 1883:1883 rabbitmq:management

15672：控制台端口号

5672：应用访问端口号
```  

查看rabbit运行状况：

![](https://images2017.cnblogs.com/blog/1081448/201710/1081448-20171027152847117-1098449262.png)

```js
容器运行正常，使用http://192.168.99.100:15672访问RabbitMQ控制台
```