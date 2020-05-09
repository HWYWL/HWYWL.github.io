---
title: Scala面向对象编程之对象
date: 2019-09-27 17:43:36
tags: 
 - Scala
categories: 
 - Scala
keywords: "Scala"
description: Scala面向对象编程之对象。
---

## 对象
### Scala中的object
object 相当于 class 的单个实例，通常在里面放一些静态的 field 或者 method；

在Scala中没有静态方法和静态字段，但是可以使用object这个语法结构来达到同样的目的。

object作用：
- 存放工具方法和常量
- 高效共享单个不可变的实例
- 单例模式

举例说明：

```scala
package com.yi.clasz

class Session {}

object SessionFactory{
  //该部分相当于java中的静态块
  val session = new Session()

  //在object中的方法相当于java中的静态方法
  def getSession(): Session ={
    session
  }
}

object SessionDemo{
  def main(args: Array[String]): Unit = {
    //单例对象，不需要new，用【单例对象名称.方法】调用对象中的方法
    val session1 = SessionFactory.getSession()
    val session2 = SessionFactory.getSession()
    
    println(session1)
    println(session2)
  }
}

```
**结果**

```
com.yi.clasz.Session@2be94b0f
com.yi.clasz.Session@2be94b0f
```

### Scala中的伴生对象
如果有一个class文件，还有一个与class同名的object文件，那么就称这个object是class的伴生对象，class是object的伴生类。

- 伴生类和伴生对象必须存放在一个.scala文件中；
- 伴生类和伴生对象的最大特点是，可以相互访问；


举例说明：

```scala
package com.yi.clasz

// 伴生类
class Dao {
  val id = 1
  private var name = "旺财"

  def printName(): Unit = {
    // 在Dao类中可以访问伴生类对象的的私有属性
    println(Dao.COUSTANT + name)
  }
}

// 伴生对象
object Dao {
  // 伴生对象中的私有属性
  private val COUSTANT = "汪汪汪。。。"
  def main(args: Array[String]): Unit = {
    val p = new Dao

    // 访问伴生类的私有属性
    p.name = "大旺财"
    p.printName()
  }
}

```
**结果**

```
汪汪汪。。。大旺财
```

### Scala中的apply方法
- object 中非常重要的一个特殊方法，就是apply方法；
- apply方法通常是在伴生对象中实现的，其目的是，通过伴生类的构造函数功能，来实现伴生对象的构造函数功能；
- 通常我们会在类的伴生对象中定义apply方法，当遇到类名(参数1,...参数n)时apply方法会被调用；//ctrl+n
- 在创建伴生对象或伴生类的对象时，通常不会使用new class/class() 的方式，而是直接使用 class()，隐式的调用伴生对象的 apply 方法，这样会让对象创建的更加简洁；

```scala
/**
 * 通过伴生类的构造函数功能，来实现伴生对象的构造函数功能
 */
object ApplyApp {
  def main(args: Array[String]): Unit = {
    val ap = new ApplyTest

    for (_ <- 1.to(10)){
      ApplyTest.incr1()
      ap.incr2()
    }

    println(ApplyTest.count1)
    println(ap.count2)
  }
}

/**
 * class是object的伴生类
 */
object ApplyTest {
  var count1 = 0

  def incr1(): Unit ={
    count1 += 1
  }
}

/**
 * object是class的伴生对象
 */
class ApplyTest{
  var count2 = 0

  def incr2(): Unit ={
    count2 += 1
  }
}
```
**结果**

```
10
10
```
