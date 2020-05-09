---
title: 集合 List、Set、Map
date: 2019-09-26 16:43:36
tags: 
 - Scala
categories: 
 - Scala
keywords: "Scala"
description: 集合 List、Set、Map。
---

## List
在Scala中列表要么为空（Nil表示空列表），要么是一个head元素加上一个tail列表。

```scala
object ListApp {
  def main(args: Array[String]): Unit = {
    val list1 = List("Hello", "Scala", "Hadoop")
    println(list1)

    val list2 = "Spark" :: "Storm" :: "Kylin" :: "Scala" :: Nil
    println(list2)

    val list3 = scala.collection.mutable.ListBuffer[String]()
    // 判断list是否为空
    if (list3.isEmpty){
      println("我是空")
    }

    list3 ++= list1
    list3 += "Hadoop"
    println(list3 + " 长度：" + list3.size)
  }
}
```
**结果**
```
List(Hello, Scala, Hadoop)
List(Spark, Storm, Kylin, Scala)
我是空
ListBuffer(Hello, Scala, Hadoop, Hadoop) 长度：4
```

## Set
Set代表一个没有重复元素的集合；将重复元素加入Set是没有用的，而且 Set 是不保证插入顺序的，即 Set 中的元素是乱序的。

```scala
object SetApp {
  def main(args: Array[String]): Unit = {
    var set1 = Set(2, 5, 1, 1, -1, 0)
    set1 += 9

    println(set1)
  }
}

```
**结果**

```
HashSet(0, 5, 1, 9, 2, -1)
```

## Map

```scala
object MapApp {
  def main(args: Array[String]): Unit = {

    // 构建映射
    var scores1 = Map("NO1" -> "校花", "NO2" -> "美女", "NO3" -> "女神")
    val scores2 = Map(("NO1" -> "校花"), ("NO2" -> "美女"), ("NO3" -> "女神"))
    scores1 +=("NO4" -> "萝莉")

    //获取映射的指
    println(scores1("NO2"))
    println(scores2.getOrElse("NO3", "不在榜单中"))
  }
}
```
**结果**

```
美女
女神
```
