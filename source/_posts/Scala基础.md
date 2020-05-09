---
title: Scala基础
date: 2019-09-25 11:43:36
tags: 
 - Scala
categories: 
 - Scala
keywords: "Scala"
description: Scala基础。
---

### 声明变量

```scala
object Hello {
  def main(args: Array[String]): Unit = {
    println("Hello Scala")

    // 使用val定义的变量值是不可变的，相当于Java里面final修饰的变量
    val i = 1

    //使用var定义的变量是可变的，在Scala中鼓励使用val
    var s = "Hello Scala"

    // Scala编译器会自动推断变量的类型，必要的时候可以指定类型
    val str: String = "Hello Spark"
  }
}

```


### 常用类型
Scala和Java一样，有7种数值类型Byte、Char、Short、Int、Long、Float、Double类型和1个Boolean类型。

### 条件表达式
Scala的条件表达式比较简洁，定义变量时加上if else判断条件。例如：

```scala
/**
* 条件表达式
*
* @param num1 参数一
* @param num2 参数二
*/
def conditionalExpression(num1: Int, num2: Int): Unit = {
    // 条件表达式
    if (num1 > num2) {
      println(num1 + " > " + num2)
    } else if (num1 == num2) {
      println(num1 + " = " + num2)
    } else {
      println(num1 + " < " + num2)
    }
}
```

### 块表达式
定义变量时用 {} 包含一系列表达式，其中块的最后一个表达式的值就是块的值。

```scala
def lump (): Unit ={
    val a = 10
    val b = 20
    
    // 块表达式
    val result = {
    // 块中最后一个表达式的值,既是快表达式的返回值
      a + b
    }
    
    println(result)
}
```

### 循环
在scala中有好几种循环，其中for循环和while循环用的比较多

**for循环：**

```scala
  /**
   * foreach循环表达式
   *
   * @param num1 参数一
   * @param num2 参数二
   */
  def foreachExpression(num1: Int, num2: Int): Unit = {
    val arr: Range = num1.until(num2)
    arr.foreach(e => {
      print(e + " ")
    })
    println()
  }

  /**
   * for循环表达式
   *
   * @param num1 参数一
   * @param num2 参数二
   */
  def forExpression(num1: Int, num2: Int): Unit = {
    val arr: Range = num1.until(num2)

    for (a <- arr if a % 2 == 0) {
      println("质数: " + a)
    }
  }

```

**while循环：**

```scala
  /**
   * while循环表达式
   */
  def whileExpression(): Unit = {
    var count = 10

    while (count > 0) {
      print(count + " ")
      count -= 1
    }
    println()
  }

  /**
   * do-while循环表达式
   */
  def dowhileExpression(): Unit = {
    var count = 10

    do {
      print(count + " ")
      count -= 1
    } while (count > 0)

    println()
  }

```
**其他循环：**


```scala
  /**
   * to循环表达式
   *
   * @param num1 参数一
   * @param num2 参数二
   */
  def toExpression(num1: Int, num2: Int): Unit = {
    val arr: Inclusive = num1.to(num2)
    println(arr.toList)
  }

  /**
   * range循环表达式
   *
   * @param num1 参数一
   * @param num2 参数二
   */
  def rangeExpression(num1: Int, num2: Int): Unit = {
    val arr: Range = Range(num1, num2)
    println(arr.toList)
  }

  /**
   * until循环表达式
   *
   * @param num1 参数一
   * @param num2 参数二
   */
  def untilExpression(num1: Int, num2: Int): Unit = {
    val arr: Range = num1.until(num2)
    println(arr.toList)
  }

```

**break 终止循环**

```scala
  /**
   * break 终止循环
   */
  def breakExpression(num1: Int, num2: Int): Unit = {
    val arr: Inclusive = num1.to(num2)
    val loop = new Breaks;

    loop.breakable {
      for (a <- arr) {
        print(a + " ")
        if (a == 5) {
          // 终止循环
          loop.break()
        }
      }
    }

    println()
  }
```
