---
title: 使用docker安装kafka
date: 2018-08-07 08:11:31
tags: 
 - Docker
 - kafka
categories: 
 - 消息队列
keywords: "Docker,kafka"
description: 使用docker安装kafka。
---

1. 启动zookeeper容器
2. 启动kafka容器
3. 测试kafka
4. 集群搭建
5. 创建Replication为2，Partition为2的topic

<!--more-->
### 1\. 启动zookeeper容器

    docker pull wurstmeister/zookeeper
    docker run -d --name zookeeper -p 2181:2181 -t wurstmeister/zookeeper

### <a name="t1" style="outline-style: initial; outline-width: 0px; color: rgb(78, 161, 219); cursor: pointer; word-break: break-all;"></a>2\. 启动kafka容器

    docker pull wurstmeister/kafka

    docker run  -d --name kafka -p 9092:9092 -e KAFKA_BROKER_ID=0 -e KAFKA_ZOOKEEPER_CONNECT=192.168.99.100:2181 -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://192.168.99.100:9092 -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 -t wurstmeister/kafka 

这里面主要设置了4个参数

    KAFKA_BROKER_ID=0               
    KAFKA_ZOOKEEPER_CONNECT=192.168.99.100:2181
    KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://192.168.99.100:9092
    KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092

中间两个参数的192.168.99.100改为宿主机器的IP地址，如果不这么设置，可能会导致在别的机器上访问不到kafka。

### <a name="t2" style="outline-style: initial; outline-width: 0px; color: rgb(78, 161, 219); cursor: pointer; word-break: break-all;"></a>3\. 测试kafka

进入kafka容器的命令行

    docker exec -ti kafka /bin/bash

进入kafka所在目录

    cd opt/kafka_2.11-2.0.0/

### <a name="t3" style="outline-style: initial; outline-width: 0px; color: rgb(78, 161, 219); cursor: pointer; word-break: break-all;"></a>4\. 集群搭建

使用docker命令可快速在同一台机器搭建多个kafka，只需要改变brokerId和端口

    docker run -d --name kafka1 \
    -p 9093:9093 \
    -e KAFKA_BROKER_ID=1 \
    -e KAFKA_ZOOKEEPER_CONNECT=192.168.99.100:2181 \
    -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://192.168.99.100:9093 \
    -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9093 -t wurstmeister/kafka

### <a name="t4" style="outline-style: initial; outline-width: 0px; color: rgb(78, 161, 219); cursor: pointer; word-break: break-all;"></a>5\. 创建Replication为2，Partition为2的topic

在kafka容器中的opt/<span style="background-color: transparent; color: inherit; font-family: Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; font-size: inherit; white-space: pre-wrap;">kafka_2.11-2.0.0</span>/目录下输入

    bin/kafka-topics.sh --create --zookeeper 192.168.99.100:2181 --replication-factor 2 --partitions 2 --topic partopic

### <a name="t5" style="outline-style: initial; outline-width: 0px; color: rgb(78, 161, 219); cursor: pointer; word-break: break-all;"></a>6\. 查看topic的状态

在kafka容器中的opt/<span style="background-color: transparent; color: inherit; font-family: Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; font-size: inherit; white-space: pre-wrap;">kafka_2.11-2.0.0</span>/目录下输入

    bin/kafka-topics.sh --describe --zookeeper 192.168.99.100:2181 --topic partopic

输出结果：

    Topic:partopic  PartitionCount:2    ReplicationFactor:2 Configs:
        Topic: partopic Partition: 0    Leader: 0   Replicas: 0,1   Isr: 0,1
        Topic: partopic Partition: 1    Leader: 0   Replicas: 1,0   Isr: 0,1

显示每个分区的Leader机器为broker0，在broker0和1上具有备份，Isr代表存活的备份机器中存活的。  
当停掉kafka1后，

    docker stop kafka1

再查看topic状态，输出结果：

    Topic:partopic  PartitionCount:2    ReplicationFactor:2 Configs:
        Topic: partopic Partition: 0    Leader: 0   Replicas: 0,1   Isr: 0
        Topic: partopic Partition: 1    Leader: 0   Replicas: 1,0   Isr: 0