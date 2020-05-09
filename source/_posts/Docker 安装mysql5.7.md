---
title: Docker 安装mysql5.7
date: 2018-07-27 10:25:36
tags: 
 - Docker
 - MySQL
categories: 
 - 数据库
keywords: "MySQL,Docker"
description: Docker 安装mysql5.7。
---

1 安装

docker pull docker.io/mysql

<!--more-->

<font face="Arial Black" color="#73a5ad"><span style="font-size: 16px;">**Docker 安装mysql5.7**</span></font>  

1 安装

 docker pull docker.io/mysql  

    [root@iZuf6boi8ejfovwda7q1ynZ ~]# docker pull docker.io/mysqlUsing default tag: latestTrying to pull repository docker.io/library/mysql ... latest: Pulling from docker.io/library/mysqlf49cf87b52c1: Pull complete 78032de49d65: Pull complete 837546b20bc4: Pull complete 9b8316af6cc6: Pull complete 1056cf29b9f1: Pull complete 86f3913b029a: Pull complete 4cbbfc9aebab: Pull complete 8ffd0352f6a8: Pull complete 45d90f823f97: Pull complete ca2a791aeb35: Pull complete Digest: sha256:1f95a2ba07ea2ee2800ec8ce3b5370ed4754b0a71d9d11c0c35c934e9708dcf1

2 启动  
[root@iZuf6boi8ejfovwda7q1ynZ ~]# docker run --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -d mysql  
3c995c988a94ce38a5ade6dcce7cc9168b349051ec51dc5e8a11c8f210658c04

如果需要把数据存储在宿主机器 加参数-v

    [root@iZuf6boi8ejfovwda7q1ynZ home]# docker run --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -v /home/mysql/data:/var/lib/mysql -v /home/mysql/my.cnf:/etc/mysql/my.cnf -d mysql/usr/bin/docker-current: Error response from daemon: Conflict. The name "/mysql" is already in use by container 3c995c988a94ce38a5ade6dcce7cc9168b349051ec51dc5e8a11c8f210658c04\. You have to remove (or rename) that container to be able to reuse that name..See '/usr/bin/docker-current run --help'.

<span style="color: rgb(51, 51, 51); font-family: -apple-system, &quot;SF UI Text&quot;, Arial, &quot;PingFang SC&quot;, &quot;Hiragino Sans GB&quot;, &quot;Microsoft YaHei&quot;, &quot;WenQuanYi Micro Hei&quot;, sans-serif, SimHei, SimSun;">有容器用了mysql这个名称，需要先停止，再删除镜像</span>  

    [root@iZuf6boi8ejfovwda7q1ynZ home]# docker psCONTAINER ID        IMAGE                               COMMAND                  CREATED             STATUS                  PORTS                                                             NAMES3c995c988a94        mysql                               "docker-entrypoint.sh"   8 hours ago         Up 8 hours              0.0.0.0:3306->3306/tcp                                            mysql[root@iZuf6boi8ejfovwda7q1ynZ home]# docker stop mysqlmysql[root@iZuf6boi8ejfovwda7q1ynZ home]# docker rm mysqlmysql

<span style="color: rgb(51, 51, 51); font-family: -apple-system, &quot;SF UI Text&quot;, Arial, &quot;PingFang SC&quot;, &quot;Hiragino Sans GB&quot;, &quot;Microsoft YaHei&quot;, &quot;WenQuanYi Micro Hei&quot;, sans-serif, SimHei, SimSun;">再次启动</span>  

    [root@iZuf6boi8ejfovwda7q1ynZ home]# docker run --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -v /home/mysql/data:/var/lib/mysql -v /home/mysql/my.cnf:/etc/mysql/my.cnf -d mysqla086c00b114a744e5f3b9f64357aef15e46cc7face8dca0878be37e34e0eb240/usr/bin/docker-current: Error response from daemon: invalid header field value "oci runtime error: container_linux.go:247: starting container process caused \"process_linux.go:364: container init caused \\\"rootfs_linux.go:54: mounting \\\\\\\"/home/mysql/my.cnf\\\\\\\" to rootfs \\\\\\\"/var/lib/docker/devicemapper/mnt/54b2f88d4d6b504e68cdc8dc41e9bf229ecc739bbdce4e23b1253cec6ea62e1e/rootfs\\\\\\\" at \\\\\\\"/var/lib/docker/devicemapper/mnt/54b2f88d4d6b504e68cdc8dc41e9bf229ecc739bbdce4e23b1253cec6ea62e1e/rootfs/etc/mysql/mysql.cnf\\\\\\\" caused \\\\\\\"not a directory\\\\\\\"\\\"\"\n".[root@iZuf6boi8ejfovwda7q1ynZ home]# docker psCONTAINER ID        IMAGE                               COMMAND                  CREATED             STATUS                  PORTS                                                             NAMES[root@iZuf6boi8ejfovwda7q1ynZ home]# docker ps -aCONTAINER ID        IMAGE                               COMMAND                  CREATED             STATUS                    PORTS                                                             NAMESa086c00b114a        mysql                               "docker-entrypoint.sh"   38 seconds ago      Created                                                                                     mysql

<span style="color: rgb(51, 51, 51); font-family: -apple-system, &quot;SF UI Text&quot;, Arial, &quot;PingFang SC&quot;, &quot;Hiragino Sans GB&quot;, &quot;Microsoft YaHei&quot;, &quot;WenQuanYi Micro Hei&quot;, sans-serif, SimHei, SimSun;">再次启动发现，没有配置文件/home/mysql/my.cnf  但是容器还是创建成功了</span>  

    [root@iZuf6boi8ejfovwda7q1ynZ home]# docker rm mysqlmysql[root@iZuf6boi8ejfovwda7q1ynZ home]# docker ps -aCONTAINER ID        IMAGE                               COMMAND                  CREATED             STATUS                    PORTS                                                             NAMES[root@iZuf6boi8ejfovwda7q1ynZ home]# docker run --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -v /home/mysql/data:/var/lib/mysql -d mysql825f0c86efe9fa16e909ac2444ae077a8c68667b3ae6760220971d6f2cda5f19[root@iZuf6boi8ejfovwda7q1ynZ home]# 

<span style="color: rgb(51, 51, 51); font-family: -apple-system, &quot;SF UI Text&quot;, Arial, &quot;PingFang SC&quot;, &quot;Hiragino Sans GB&quot;, &quot;Microsoft YaHei&quot;, &quot;WenQuanYi Micro Hei&quot;, sans-serif, SimHei, SimSun;">所以还是需要删除镜像，重新启动，简单点把配置文件去掉就好了</span>  

3 进入容器  
[root@iZuf6boi8ejfovwda7q1ynZ ~]# docker exec -it mysql  bash  
root@3c995c988a94:/# mysql                                                                                                                                                                                                   
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: NO)

4 进入客户端  

    [root@3c995c988a94:/# mysql -uroot -pEnter password: Welcome to the MySQL monitor.  Commands end with ; or \g.Your MySQL connection id is 4Server version: 5.7.20 MySQL Community Server (GPL)  Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.  Oracle is a registered trademark of Oracle Corporation and/or itsaffiliates. Other names may be trademarks of their respectiveowners.  Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.  mysql> select version(); +-----------+| version() |+-----------+| 5.7.20    |+-----------+1 row in set (0.00 sec)