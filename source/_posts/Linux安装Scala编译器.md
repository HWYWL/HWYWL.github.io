---
title: Linux安装Scala编译器
date: 2019-09-25 11:43:34
tags: 
 - Scala
 - Linux
categories: 
 - Scala
keywords: "Scala,Linux"
description: Linux安装Scala编译器。
---
- 下载Scala地址https://www.scala-lang.org/download/2.11.8.html
- 解压Scala到指定目录
```
tar -zxvf scala-2.11.8.tgz -C /usr/java
```
- 配置环境变量，将scala加入到PATH中
```
vim /etc/profile
export JAVA_HOME=/usr/java/jdk1.8
export PATH=$PATH:$JAVA_HOME/bin:/usr/java/scala-2.11.8/bin
```
