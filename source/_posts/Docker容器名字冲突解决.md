---
title: Docker容器名字冲突解决
date: 2018-08-07 08:11:31
tags: 
 - Docker
categories: 
 - Docker
keywords: "Docker"
description: Docker容器名字冲突解决。
---
Docker容器名字冲突解决

<!--more-->
-- Docker容器名字冲突解决
```js
docker: Error response from daemon: Conflict. The container name "/tracker" is already in use by container "73b9fc481e0316195ab89d4c4faa38c5a1012a84ce859a65488e983e9b415255". You have to remove (or rename) that container to be able to reuse that name.
See 'docker run --help'.
```  

以上是错误的提示，我们发现有一个名为tracker的容器冲突了，此时我们使用命令进行查看
```js
docker ps
```  

如果容器存在直接使用一下命令停止、删除
```js
docker stop id
docker rm id
``` 
如果容器不存在使用以下命令查看,就会找到存在的镜像
```js
docker ps -a
``` 
此时你可以使用删除，或者把他重启，记得后面跟上id参数
```js
docker restart & docker stop & docker start
```  
注意：因为存在数据卷的原因，所以以前的数据不会被删除，可以放心大胆的删除镜像！！！