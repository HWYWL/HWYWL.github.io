---
title: JDBC基础
date: 2017-01-11 19:23:54
tags: 
 - JDBC
categories: 
 - 数据库
keywords: "MySQL"
description: JDBC基础。
---

学习目标
1. 能够说出什么是JDBC
2. 能够说出JDBC连接数据库的四个参数。

```java
Class.forName(“com.mysql.jdbc.Driver”);
DriverManager.getConnection(url, name, password)
url:  jdbc:mysql://localhost:3306/数据库
```

3. 能够说出JDBC的核心API

```
DriverManager: 1) 加载驱动；2）获取数据库连接；
Connection: 数据库的一个连接；
Statement
execute()
executeQuery() executeUpdate()
ResultSet: 可滚动结果集。
next() previous() first() last() absolute(int n)
getXxx(int columnIndex)  getXxx(String columnName)
```

4. 能够运用Statement执行SQL操作
5. 能够运用PreparedStatement执行SQL操作
- 第一步：获取数据库连接；
- 第二步：创建PreparedStatement对象;
PreparedStatement pstmt = conn.prearedStatement(sql);
- 第三步：如果有参数，就要设置参数；
- 第四步：调用该对象executeQuery或executeUpdate方法；
- 第五步：遍历结果；
- 第六步：释放资源；（ResultSet、Statement、Connection）
6. 能够区别Statement与PreparedStatement

```
PreparedStatement是Statement的子接口；
PreparedStatement的执行率比Statement更高；
PreparedStatement的可读性比Statement更好；
PreparedStatement的安全性比Statement更高；
```

# 一、 JDBC概述
JDBC（Java Database Connectivity）：java数据库连接技术。它的作用就是通过java代码访问和操作数据库。
JDBC是一个接口规范。它定义了一些访问数据库的规则。具体数据库的实现是由各个数据库厂商负责实现。

![image](F14813315E3741EB9AA0C5B2DDD882A5)

不同的数据库厂商都必须要提供自己的驱动程序。其实，该驱动程序就是实现了JDBC规范的程序。

# 二、JDBC的使用
## 2.1 JDBC的核心API
- DriverManger: 数据库的驱动管理类。作用：1）加载驱动；2）获取数据库的连接；
- Conection：代表一个数据库连接。访问数据库之前必须要先获取Connction对象；
- Statement：代表一个SQL语句，它负责把SQL的命令发送MySQL服务器执行；
- ResultSet：代表一个结果集对象，该对象封装了所有查询结果；

## 2.2 JDBC连接MySQL数据库
1) 第一步：加载驱动（mysql-connector-java-5.1.13-bin.jar）；
2) 第二步：获取数据库连接；
MySQL：jdbc:mysql://localhost:3306/itheima
Oracle：jdbc:oracle:thin:localhost:1521:orcl
格式：jdbc:数据库名:其他
jdbc: 固定的
数据库名：例如：mysql
其他：不同数据库的写法会不一样；
3) 第三步：创建Statement对象；
4) 第四步：调用Statement对象一些方法；
execute(): 可以执行查询或更新操作；
executeQuery(): 执行查询；
executeUpdate(): 执行更新；
5) 第五步：遍历结果结果集（执行查询才有结果集）；

ResultSet常用的方法：
- next()：把游标向下移动一行，如果该行有数据，则返回true，否则返回false；
- previous()：把游标向上移动一行，如果该行有数据，则返回true，否则返回false；
- first()：把游标移动到第一行，如果该行有数据，则返回true，否则返回false；
- last()：把游标移动到最后一行，如果该行有数据，则返回true，否则返回false；
- getXxx(int columnIndex)：根据字段的索引获取字段内容；
- getXxx(String columnName)：根据字段名或字段别名获取字段内容；
6) 第六步：关闭资源；

**==注意==：关闭资源的时候，要先开后关，后开先关。**


```
public class Demo {
 
    public static void main(String[] args) throws ClassNotFoundException, SQLException {
        //加载驱动
        Class.forName("com.mysql.jdbc.Driver");
        //获取数据库连接
        Connection conn = DriverManager.getConnection(
                "jdbc:mysql://localhost:3306/itheima",
                "root",
                "root");
        //创建Statement对象
        String sql = "insert into students values(14, 'jacky', 100, 100, 100, 'java054')";
//      String sql = "select * from students";
        Statement stat = conn.createStatement();
        //调用Statement的方法
        int rows = stat.executeUpdate(sql);
        System.out.println("返回结果：" + rows);
        //关闭资源（注意：先开后关，后开先关）
        stat.close();
        conn.close();
    }
 
}
```

## 2.3 JDBC的增删查该操作
2.2.1 添加

```
//添加
String insertSql = "insert into user(name, age) values('jacky', 18),('joe', 19);";
int rows = stmt.executeUpdate(insertSql);
System.out.println("结果：" + rows);
```

2.2.2 删除

```
//删除
String deleteSql = "delete from user where name = 'joe'";
int rows = stmt.executeUpdate(deleteSql);
System.out.println("结果：" + rows);
```

2.2.4 查询

```
//查询
String selectSql = "select id, name as FullName, age from user";
ResultSet rs = stmt.executeQuery(selectSql);
while (rs.next()) { //把游标向下移动一行

    //通过索引值获取字段的值
    int id = rs.getInt(1); //获取第一个字段，索引值从1开始计算。
    String name = rs.getString(2);
    int age = rs.getInt(3);

    //通过字段名或字段别名获取字段的值
    int id = rs.getInt("id");
    String name = rs.getString("FullName"); //字段名或别名都可以
    int age = rs.getInt("age");
    System.out.println("编号：" + id + "，姓名：" + name + "，年龄：" + age);

}
```

# 三、 SQL注入
所谓的SQL注入，就是用户把SQL命令通过web表单提交到服务器，服务器把SQL命令与已有SQL命令一起被执行，从而达到欺骗数据库服务器的目的。

![image](0B278DBD7CA74979B6C04E43B8EBF9F7)

解决办法：
1) 使用js验证，规定用户输入的内容不能够出现特殊的字符；
2) 不能够把用户输入的内容直接放在SQL语句一起执行； 

# 四、预编译对象
PreparedStatement其实就是Statement的一个子接口。它的主要作用：
1) 防止SQL注入；
2) 提高程序的可读性和可维护性；
3) 提高程序的执行效率；

![image](54C3C1B58C0B450E962C28F6EDF010DA)

![image](FFCB56D2DF804D8DA15C7035CDD5FF4A)

PreparedStatement提供一个预编译的功能。但是从MySQL4.1之后默认预编译的功能是关闭的。如果要启用预编译的功能，就需要在URL的后面田间两个参数：

![image](96F38638A15146649E80C5266EE62FC7)