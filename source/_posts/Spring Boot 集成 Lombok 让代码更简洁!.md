---
title: Spring Boot 集成 Lombok 让代码更简洁!
date: 2018-06-05 08:11:31
tags: 
 - Spring Boot
 - Lombok
categories: 
 - Java
keywords: "Spring Boot,Lombok"
description: Spring Boot 集成 Lombok 让代码更简洁!
---

### lombok的威力
![alt](https://i.loli.net/2019/03/11/5c85b64e19a51.png)
![简化代码](https://i.loli.net/2019/03/11/5c85b5f797450.png)

### IntelliJ IDEA安装lombok插件
1、菜单栏 File > Settings > Plugins > Browse repositories…
![alt](https://i.loli.net/2019/03/11/5c85b636a981e.png)

2、搜索 Lombok Plugin 安装后，重启IDEA即可生效
![alt](https://i.loli.net/2019/03/11/5c85b659d61c8.png)

### Spring Boot项目中使用lombok.
1、添加lombok依赖
![alt](https://i.loli.net/2019/03/11/5c85b67b0e9f0.png)

2、编写一个实体类 User，使用@Data注解
![alt](https://i.loli.net/2019/03/11/5c85b68b392cc.png)

3、编写测试方法，测试@Data的作用
![alt](https://i.loli.net/2019/03/11/5c85b69fcbb30.png)
![alt](https://i.loli.net/2019/03/11/5c85b6aae397b.png)

### 其它简化代码的特性介绍
```java
val : 最终局部变量
@NonNull : 让你不在担忧并且爱上NullPointerException
@CleanUp : 自动资源管理：不用再在finally中添加资源的close方法
@Setter/@Getter : 自动生成set和get方法
@ToString : 自动生成toString方法
@EqualsAndHashcode : 从对象的字段中生成hashCode和equals的实现
@NoArgsConstructor/@RequiredArgsConstructor/@AllArgsConstructor
自动生成构造方法
@Data : 自动生成set/get方法，toString方法，equals方法，hashCode方法，不带参数的构造方法
@Value : 用于注解final类
@Builder : 产生复杂的构建器api类
@SneakyThrows : 异常处理（谨慎使用）
@Synchronized : 同步方法安全的转化
@Getter(lazy=true) :
@Log : 支持各种logger对象，使用时用对应的注解，如：@Log4j
```

### 推荐用法
1、在 Bean / Entity 类上使用 @Data 注解。
2、需要使用 Log 对象的地方使用 @Log4j（依项目日志框架决定）。

**注意：lombok 的注解不能被继承。**

原文链接：https://www.jianshu.com/p/dd5349ac8473