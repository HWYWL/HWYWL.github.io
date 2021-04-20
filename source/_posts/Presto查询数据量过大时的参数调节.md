---
title: Presto查询数据量过大时的参数调节
date: 2021-04-20 11:49:36
tags: 
 - 大数据
 - Presto
categories: 
 - 大数据
keywords: "大数据,Presto"
description:Presto查询数据量过大时的参数调节。
---

查询数据量过大，执行过程中途presto会报警
```
query.max-memory、query.max-memory-per-node、query.max-total-memory-per-node、jvm
```
会分别提示超出最大限制，调整参数配置防止任务被中断

### 调参过程：

应先调整jvm的大小，并重启机器，否则不生效
```
# 切换root用户
sudo -i 

# 对 jvm.config中-Xmx进行调参
cd /etc/presto/conf  
```


![c7QjXt.png](https://z3.ax1x.com/2021/04/20/c7QjXt.png)


将默认的-Xmx53369263620
调整后为-Xmx533692636200 约和497GB（近500GB）
调整 jvm后reboot重启，等待机器重连。


重复上面sudo-i 等命令vim /config.properties
分别将下图参数调整为800GB、40GB、55GB

![c7QHte.png](https://z3.ax1x.com/2021/04/20/c7QHte.png)

默认的配比是query.max-memory-per-node的值在jvm重点的Xmx的10%左右

可通过 initctl list 查看所有正在运行的服务
```
presto的服务为 presto-server，将其终止后重启
查看进程：sudo status presto-server
终止进行：sudo stop presto-server
开始进程：sudo start presto-server
```