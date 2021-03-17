---
title: JDK 16 GA 正式发布： 17个特性，一网打尽！
date: 2021-03-17 09:54:65
tags: 
 - Java
categories: 
 - Java
keywords: "Java"
description: JDK 16 GA 正式发布： 17个特性，一网打尽
---


### 官方给出的JDK16所有特性一览如下，总计17个特性： 
- 338: Vector API (Incubator) 
- 347: Enable C++14 Language Features 
- 357: Migrate from Mercurial to Git 
- 369: Migrate to GitHub 
- 376: ZGC: Concurrent Thread-Stack Processing 
- 380: Unix-Domain Socket Channels 
- 386: Alpine Linux Port 
- 387: Elastic Metaspace 
- 388: Windows/AArch64 Port 
- 389: Foreign Linker API (Incubator) 
- 390: Warnings for Value-Based Classes 
- 392: Packaging Tool 
- 393: Foreign-Memory Access API (Third Incubator) 
- 394: Pattern Matching for instanceof 
- 395: Records 
- 396: Strongly Encapsulate JDK Internals by Default 
- 397: Sealed Classes (Second Preview)

接下来，我们一一对其进行解读。

### 338: Vector API
Java提供了一些Vector API, 那到底什么是Vector API呢？废话不多说，给你举个例子。我们先写一段普通的Java代码:
```java
void scalarComputation(float[] a, float[] b, float[] c) {
   for (int i = 0; i < a.length; i++) {
        c[i] = (a[i] * a[i] + b[i] * b[i]) * -1.0f;
   }
}
```

那么，用Vector API实现等价逻辑的代码如下所示。是不是更复杂了， CRY～，嗯，没关系，反正要用到Vector API起码得是10年以后的事情了:

```java
static final VectorSpecies<Float> SPECIES = FloatVector.SPECIES_256;

void vectorComputation(float[] a, float[] b, float[] c) {

    for (int i = 0; i < a.length; i += SPECIES.length()) {
        var m = SPECIES.indexInRange(i, a.length);
  // FloatVector va, vb, vc;
        var va = FloatVector.fromArray(SPECIES, a, i, m);
        var vb = FloatVector.fromArray(SPECIES, b, i, m);
        var vc = va.mul(va).
                    add(vb.mul(vb)).
                    neg();
        vc.intoArray(c, i, m);
    }
}
```

### 347: Enable C++14 Language Features
一句话概括就是JDK16的C++源码可以使用C++14的语法特性。并且如果在HotSpot源码中确实有用到的话，会给出这些特性的特殊指导说明。

### 369: Migrate to GitHub
OpenJDK终于也迁移到GitHub中，这个和 JEP 357: Migrate from Mercurial to Git 一起完成的。之前OpenJDK的源代码都是用Mercuial维护的，没听说过？没听说过就对了，也不用浪费时间去了解。

OpenJDK还解释了为什么选择GitHub，主要有3个原因：



1. 相比其他产品，GitHub的性能更好。
2. GitHub是全球最大的源码托管服务，并且有超过5kw用户。
3. GitHub拥有最多用于扩展的API。这些API支撑了很多文本编辑器(例如: Emacs, VS Code, Atom等), IDE工具(IDEA, Eclipse等), 命令行等对它的支持。


最后，附上GitHub中OpenJDK项目地址：https://github.com/openjdk/。
[![66VYsP.png](https://s3.ax1x.com/2021/03/17/66VYsP.png)](https://imgtu.com/i/66VYsP)

### 376: ZGC: Concurrent Thread-Stack Processing
这个需求的意思是ZGC把线程栈处理从safepoint挪到并发阶段，从而减少GC的停顿时间。ZGC的目标就是消灭GC停顿和扩展性问题。为此，ZGC做了很多的工作，在把Thread-Stack从safepoint挪到并发阶段之前，还有很多其他的GC操作也做了这些优化，比如：marking, relocation, reference processing, class unloading, 以及大多数的 root processing。

### 386: Alpine Linux Port
支持一些新的平台，比如Alpine Linux。Alpine Linux是一个Linux发行版, 它是一个由社区开发的Linux操作系统,该操作系统以安全为理念,面向x86路由器、防火墙、虚拟专用网、IP电话盒及服务器而设计。官方用3个词来介绍它：Small. Simple. Secure。确实很小，它的发行包不到6MB。

[![66VDRs.png](https://s3.ax1x.com/2021/03/17/66VDRs.png)](https://imgtu.com/i/66VDRs)

此外，JEP 388: Windows/AArch64 Port 也是类似的特性，即支持Windows平台和ARM64(AArch64)平台。

### 392: Packaging Tool
这个特性在JDK14中已经被介绍过(JEP 343)，它的目的是创建一个打包工具jpackage，jpackage将Java应用程序打包到特定平台的程序包中，该程序包包含所有必需的依赖项。该应用程序可以作为普通JAR文件的集合或作为模块的集合提供。受支持的特定平台的软件包格式有：Linux: deb and rpm macOS: pkg and dmg Windows: msi and exe

打包命令参考如下：
```
$ jpackage --name myapp --input lib --main-jar main.jar
# 如果MANIFEST.MF中没有Main-Class，也可以使用如下的打包命令
$ jpackage --name myapp --input lib --main-jar main.jar \
  --main-class myapp.Main
```

### 393: Foreign-Memory Access API
引入API允许Java程序安全的、高效的访问堆外内存。这个需求的目标是希望做到如下几点：

- Generality: 通用性，一个简单的API就可以操作很多外部内存，例如本地内存、托管的堆内存等。
- Safety: 安全性, 这些API不会破坏JVM的安全，可以放心的使用。
- Control: 可控性，客户端应具有关于如何显示或者隐式控制重新分配内存段的选项。
- Usability: 可用性, 对那些需要访问堆外内的程序来说，比如使用Unsafe接口操作堆外内存，新设计的API应该是更好的替代者。


JDK16将这些API放到孵化的名为jdk.incubator.foreign的模块中，有3个主要的抽象: MemorySegment, MemoryAddress 和 MemoryLayout。

### 394: Pattern Matching for instanceof
这个特性最先出现在JDK14中(JEP 305), JDK15中(JEP 375)是第二轮预览。此次加到JDK16中是彻底完成这个特性。这个特性是为了增强Java语法，允许在instanceof中加入表达式匹配。以前我们写代码可能需要这样:

```java
if (obj instanceof String) {
    String s = (String) obj;    // grr...
    ...
}
```

现在我们只需要这样：

```java
if (obj instanceof String s) {
    // Let pattern matching do the work!
    ...
}
```

我们还可以这样:
```java
if (obj instanceof String s && s.length() > 5) {
    flag = s.contains("jdk");
}
```

### 395: Records
很多人抱怨Java太冗余了，很多代码太形式主义了。比如如下这段代码所示，除了x和y的属性定义，其他的构造方法，equals、hashCode、toString方法基本上都借助IDE工具生成：

```java
class Point {
    private final int x;
    private final int y;

    Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    int x() { return x; }
    int y() { return y; }

    public boolean equals(Object o) {
        if (!(o instanceof Point)) return false;
        Point other = (Point) o;
        return other.x == x && other.y == y;
    }

    public int hashCode() {
        return Objects.hash(x, y);
    }

    public String toString() {
        return String.format("Point[x=%d, y=%d]", x, y);
    }
}
```

因此Java发明了一种新的Java类，我们只需要简单的几行代码即可。有点类似于lombok给我们代码带来的简洁性：

```java
record Point(int x, int y) { }
```

### 397: Sealed Classes
中文翻译的含义是密封类，具体是什么意思呢？举个栗子，如下这段代码所示，Expr这个接口用sealed进行了修饰，那么只有permits后面的实现类实现这个接口，不允许其他实现类：

```
package com.example.expression;

public sealed interface Expr
    permits ConstantExpr, PlusExpr, TimesExpr, NegExpr { ... }

public final class ConstantExpr implements Expr { ... }
public final class PlusExpr     implements Expr { ... }
public final class TimesExpr    implements Expr { ... }
public final class NegExpr      implements Expr { ... }
```

sealed不仅可以修饰interface，也可以修饰类class。JEP 397 的目的是限制接口的实现，以及限制类的继承。如此一来，类和接口的作者对其写的类和接口有更好的控制权。

sealed修饰的类有3大约束：



1. sealed修饰的类和它permits的子类必属属于相同的module(JDK9中的模块化)。如果是没有命名的module，那么需要在相同的包名下。
2. 每一个permits的子类必须直接实现sealed的类，而不能间接实现。
3. 每一个permits的子类必须用一个修饰符描述它是如何传播从父类继承过来的密封性的。比如子类用final修饰，表示不允许其他类继承。再比如用sealed修饰，从而继续限制子类的子类。