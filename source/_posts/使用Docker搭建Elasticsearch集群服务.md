---
title: 使用Docker搭建Elasticsearch集群服务
date: 2020-03-31 11:22:33
tags: 
 - Docker
 - Elasticsearch
categories: 
 - 数据库
keywords: "Elasticsearch,Docker"
description: 使用Docker搭建Elasticsearch集群服务。
---

## Elasticsearch安装

### 使用安装包的方式进行单机部署
下载安装包:
```
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.4.1-darwin-x86_64.tar.gz
```

解压安装包:
```
tar -xvf elasticsearch-7.4.1-darwin-x86_64.tar.gz
```

运行：
```
cd elasticsearch-7.4.1/bin
./elasticsearch
```

###使用安装包的方式进行集群部署
使用安装包的方式进行集群部署其实也很简单，你可以使用如下脚本来启动集群服务:
```
#!/bin/bash
case $1 in
"start") {
    /Users/pengli/software/middle-software/elasticsearch-7.4.1/bin/elasticsearch -E node.name=node0 -E cluster.name=elasticserch -E path.data=node0_data -d
    /Users/pengli/software/middle-software/elasticsearch-7.4.1/bin/elasticsearch -E node.name=node1 -E cluster.name=elasticserch -E path.data=node1_data -d
    /Users/pengli/software/middle-software/elasticsearch-7.4.1/bin/elasticsearch -E node.name=node2 -E cluster.name=elasticserch -E path.data=node2_data -d
    /Users/pengli/software/middle-software/elasticsearch-7.4.1/bin/elasticsearch -E node.name=node3 -E cluster.name=elasticserch -E path.data=node3_data -d
    echo "**********elasticsearch cluster start**********"
};;
esac
```
**注意：请使用自己的安装路径替换如上脚本中的路径**

### 使用Docker的方式进行单机部署
拉取镜像:
```
docker pull elasticsearch:7.4.1
```

如果你想用其他版本，可以搜索镜像，拉取你喜欢的版本：
```
docker search elasticsearch
```

运行镜像:
```
docker run -d --name es -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:7.4.1
```

### 使用Docker的方式进行集群部署

创建docker-compose.yml文件:
```
version: '2.2'
services:
  es01:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.4.1
    container_name: es01
    environment:
      - node.name=es01
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es02,es03
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - D:/Elasticsearch-Cluster/data01:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
    networks:
      - elastic
  es02:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.4.1
    container_name: es02
    environment:
      - node.name=es02
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es01,es03
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - D:/Elasticsearch-Cluster/data02:/usr/share/elasticsearch/data
    networks:
      - elastic
  es03:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.4.1
    container_name: es03
    environment:
      - node.name=es03
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es01,es02
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - D:/Elasticsearch-Cluster/data03:/usr/share/elasticsearch/data
    networks:
      - elastic

networks:
  elastic:
    driver: bridge
```
docker-compose.yml文件的最后使用的是**bridge**网络，可以使用如下命令查看是否存在这个网络：
```
docker network ls
```

如果你没这个网络，可以使用如下命令新建一个
```
docker network create bridge
```

配置说明:
- image：指定elasticsearch镜像
- container_name：执行运行容器的名称，可以随意指定
- volumes：配置卷宗映射关系，比如将本地的/Users/pengli/software/docker/elasticsearch/data03数据目录映射到docker容器的/usr/share/elasticsearch/data，这样可以保证重启容器不会导致elasticsearch数据丢失

常见错误:
如果你在使用docker部署elasticsearch集群服务中出现如下错误：java.net.UnknownHostException，那么请调大分配给docker的内存空间至4个G，官网给出的解决方案就是提升docker可用内存至4GB

启动elasticsearch集群服务:
在docker-compose.yml目录中执行**docker-compose up -d**来启动集群服务

关闭elasticsearch集群服务:
在docker-compose.yml目录中执行**docker-compose down**来关闭集群服务

查看集群信息:
在浏览器中输入**http://localhost:9200/_cat/nodes**查看集群服务
