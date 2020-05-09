---
title: 五分钟学会 Java 开发效率神器 Lombok！
date: 2020-03-09 17:58:05
tags: 
 - Java
 - Lombok
categories: 
 - Java
keywords: "Java,Lombok"
description: 五分钟学会 Java 开发效率神器 Lombok！。
---

## 简介
Lombok 是一个 Java 第三方库，可以透过简单的注解省略 Java 的代码，像是 setter、getter、logger...等，目的在消除冗长的代码和提高开发效率

假设你在类上加上了一个 `@Getter` 和 `@Setter` 注解，那你就不用在写烦人的 getter 和 setter，lombok 会自动帮你产生出来啦！

![](http://www.wazhi.com.cn//school/image/1166/20200308/20200308072342560.png)

之所以加个 lombok 的 @Getter 注解就可以帮我们自动生成所有变量的 getter，是因为 lombok 参与了 Java 在 compile 阶段生成 `.class` 档的过程，lombok 会帮我们自动写一堆 getter，然后塞进 `.class` 档，所以真正被编译出来的 User.class 档桉，是包含完整的 getter 的

简单的说，lombok可以算是一种语法糖，只是在帮我们增进开发效率而已，实际上所产生出来的`.class`档仍然是完全正常的

### 安装 Lombok

要在 project 中使用 lombok，除了要在 maven 中加入 lombok 依赖，还要安装 IDEA lombok 插件

#### **1. 加入 maven 依赖**

```
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.12</version>
    <scope>provided</scope>
</dependency>
```

#### 2. 在 IDEA中安装 lombok 插件

> 我使用的 IDEA 版本是 2019.3.3，可能会因为版本差异导致安装方式有改变

先点选左上角 Intellij IDEA -> Preferences

![](http://www.wazhi.com.cn//school/image/1166/20200308/20200308072342697.png)

然后点击左边的 Plugins，再点击上面的 Marketplace tab，然后就可以在搜寻栏中输入`lombok`，并且找到lombok插件并安装它

![](http://www.wazhi.com.cn//school/image/1166/20200308/20200308072342718.png)

#### 为什么需要特地安装 IDEA lombok 插件？

其实在 maven 加入 lombok 依赖之后，使用 `mvn clean package` 就可以正常 build 过，在 IDEA 中点击绿色按钮也可以运行程式

之所以还要特地安装 IDEA lombok 插件，是因为如果不安装 lombok 插件的话，IDEA 就会没办法自动提示出 lombok 产生的方法，所以就会发生你的代码一片红，但是运行却可以通过的奇妙现象

像是下面这段代码中，因为对 IDEA 来说，代码裡并不存在 setter，所以没办法自动提示 `setId()`、`setName()` 等方法，但是又因为我们在maven中有加入 lombok 依赖，所以点击第 13 行的绿色箭头运行程式的话，是可以正常运行成功的

![](http://www.wazhi.com.cn//school/image/1166/20200308/20200308072342841.png)

所以 lombok 算侵入性很高的一个第三方库，只要团队中有一个人用 lombok 开发，那么所有的人都必须得安装上 lombok 插件，才不会在 IDEA 中一打开 project 时，整片都是痛苦的红字

### Lombok 用法

lombok 官网提供了许多注解，但是「劲酒虽好，可不能贪杯」，你用了越多 lombok 的进阶用法，会让整个团队的学习曲线上升，反而会造成反效果，所以在此处只解释最最常见、并且我认为最必要的注解使用方式，其他的用法就不介绍了

![](http://www.wazhi.com.cn//school/image/1166/20200308/20200308072342875.png)

#### 1. @Getter/@Setter

自动产生 getter/setter

![](http://www.wazhi.com.cn//school/image/1166/20200308/20200308072342898.png)

#### 2. @ToString

自动重写 `toString()` 方法，会印出所有变量

![](http://www.wazhi.com.cn//school/image/1166/20200308/20200308072342923.png)

#### 3. @EqualsAndHashCode

自动生成 `equals(Object other)` 和 `hashcode()` 方法，包括所有非静态变量和非 transient 的变量

![](http://www.wazhi.com.cn//school/image/1166/20200308/20200308072342955.png)

如果某些变量不想要加进判断，可以透过 exclude 排除，也可以使用 of 指定某些字段

![](http://www.wazhi.com.cn//school/image/1166/20200308/20200308072342974.png)

Q : 为什么只有一个整体的 `@EqualsAndHashCode` 注解，而不是分开的两个 `@Equals` 和 `@HashCode`？

A : 在 Java 中有规定，当两个对象 equals 时，他们的 hashcode 一定要相同，反之，当 hashcode 相同时，对象不一定 equals。所以 equals 和 hashcode 要一起实现，免得发生违反 Java 规定的情形发生

#### 4. @NoArgsConstructor, @AllArgsConstructor, @RequiredArgsConstructor

这三个很像，都是在自动生成该类的构造器，差别只在生成的构造器的参数不一样而已

**@NoArgsConstructor** : 生成一个没有参数的构造器

![](http://www.wazhi.com.cn//school/image/1166/20200308/20200308072342995.png)

**@AllArgsConstructor** : 生成一个包含所有参数的构造器

![](http://www.wazhi.com.cn//school/image/1166/20200308/20200308072343018.png)

这里注意一个 Java 的小坑，当我们没有指定构造器时，Java 编译器会帮我们自动生成一个没有任何参数的构造器给该类，但是如果我们自己写了构造器之后，Java 就不会自动帮我们补上那个无参数的构造器了

然而很多地方（像是 Spring Data JPA），会需要每个类都一定要有一个无参数的构造器，所以你在加上 `@AllArgsConstructor` 时，一定要补上 `@NoArgsConstrcutor`，不然会有各种坑等着你

    @AllArgsConstructor
    @NoArgsConstructor
    publicclassUser{
        private Integer id;
        private String name;
    }
    

**@RequiredArgsConstructor** : 生成一个包含 "特定参数" 的构造器，特定参数指的是那些有加上 final 修饰词的变量们

![](http://www.wazhi.com.cn//school/image/1166/20200308/20200308072343058.png)

补充一下，如果所有的变量都是正常的，都没有用 final 修饰的话，那就会生成一个没有参数的构造器

#### 5. @Data

整合包，只要加了 @Data 这个注解，等于同时加了以下注解

- 
@Getter/@Setter

- 
@ToString

- 
@EqualsAndHashCode

- 
@RequiredArgsConstructor

![](http://www.wazhi.com.cn//school/image/1166/20200308/20200308072343091.png)

@Data 是使用频率最高的 lombok 注解，通常 @Data 会加在一个值可以被更新的对象上，像是日常使用的 DTO 们、或是 JPA 裡的 Entity 们，就很适合加上 @Data 注解，也就是 @Data for mutable class

#### 6. @Value

也是整合包，但是他会把所有的变量都设成 final 的，其他的就跟 @Data 一样，等于同时加了以下注解

- 
@Getter (注意没有setter)

- 
@ToString

- 
@EqualsAndHashCode

- 
@RequiredArgsConstructor

![](http://www.wazhi.com.cn//school/image/1166/20200308/20200308072343118.png)

上面那个 @Data 适合用在 POJO 或 DTO 上，而这个 @Value 注解，则是适合加在值不希望被改变的类上，像是某个类的值当创建后就不希望被更改，只希望我们读它而已，就适合加上 @Value 注解，也就是 @Value for immutable class

另外注意一下，此 lombok 的注解 @Value 和另一个 Spring 的注解 @Value 撞名，在 import 时不要 import 错了

#### 7. @Builder

自动生成流式 set 值写法，从此之后再也不用写一堆 setter 了

![](http://www.wazhi.com.cn//school/image/1166/20200308/20200308072343147.png)

注意，虽然只要加上 @Builder 注解，我们就能够用流式写法快速设定对象的值，但是 setter 还是必须要写不能省略的，因为 Spring 或是其他框架有很多地方都会用到对象的 getter/setter 对他们取值/赋值

所以通常是 @Data 和 @Builder 会一起用在同个类上，既方便我们流式写代码，也方便框架做事

    @Data
    @Builder
    publicclassUser{
        private Integer id;
        private String name;
    }
    

#### 8. @Slf4j

自动生成该类的 log 静态常量，要打日志就可以直接打，不用再手动 new log 静态常量了

![](http://www.wazhi.com.cn//school/image/1166/20200308/20200308072343164.png)

除了 @Slf4j 之外，lombok 也提供其他日志框架的变种注解可以用，像是 @Log、@Log4j...等，他们都是帮我们创建一个静态常量 log，只是使用的库不一样而已

    @Log//对应的log语句如下
    privatestaticfinal java.util.logging.Logger log = java.util.logging.Logger.getLogger(LogExample.class.getName());
    
    @Log4j //对应的log语句如下
    privatestaticfinal org.apache.log4j.Logger log = org.apache.log4j.Logger.getLogger(LogExample.class);
    

SpringBoot默认支持的就是 slf4j + logback 的日志框架，所以也不用再多做啥设定，直接就可以用在 SpringBoot project上，log 系列注解最常用的就是 @Slf4j

> 转自：古古说

