---
title: JDBC-DBUtil使用
date: 2017-01-11 18:23:54
tags: 
 - JDBC
categories: 
 - 数据库
keywords: "MySQL"
description: JDBC-DBUtil使用。
---


### 学习目标
- 1. 能够描述什么是连接池
- 2. 能够实现自定义连接池
- 3. 能够解决自定义Connection的close释放连接的问题
- 4. 能够运用第三方连接池实现技术 DBCP C3P0
    - 第一步：导入c3p0的jar包；
    - 第二步：创建连接池对象(ComboPooledDataSource)；
    - 第三步：设置参数；（配置文件）
        - c3p0-config.xml（优先级更高）
        - c3p0.properties
    - 第四步：调用getConnection获取数据库连接；
    - 第五步：调用Connection对象的close方法释放Connection；
- 5. 能够描述出三种数据库元数据
- 6. 能够运用数据库表元数据


# 一、 数据库连接池
之前我们访问数据库：
- 第一步：获取数据库连接
- 第二步：创建PreparedStatement对象；
- 第三步：设置参数；
- 第四步：遍历结果集；
- 第五步：关闭资源；

获取数据库连接是一个比较耗时的操作。

## 1.1 什么数据库连接池
数据库连接池就就是一个用来存储了数据库连接的集合对象。
使用数据库连接池的好处：减少在获取Connection对象的等待时间，从而可以提高访问数据库的效率。
- 应用程序直接获取数据库连接的缺点：
- 
![image](6842DEDE225343B7962DD019055278AB)

- 数据库连接池：

![image](C94854159CCC4AE8820DFECC76DBB652)

## 1.2 自定义连接池的实现
1) 第一步：创建一个类，实现DataSouce接口；
2) 第二步：添加一些属性；


```java
//存储所有Connection对象
LinkedList<MyConnection> pool = new LinkedList<MyConnection>();
//数据库的连接参数
private static String driverClass = "com.mysql.jdbc.Driver";
private static String url = "jdbc:mysql://localhost:3306/itheima";
private static String user = "root";
private static String password = "root";
//数据库连接池的参数
private int initialPoolSize = 3; //初始化的连接数
private int maxPoolSize = 5; //最大连接数
private int curPoolSize = 0; //当前的连接数
```
3) 第三步：创建一个方法生成数据库连接；

```
//生成数据库连接
public MyConnection createConnection() throws SQLException {
    Connection conn = DriverManager.getConnection(url, user, password);
    return new MyConnection(this, conn);
}
```
4) 第四步：创建一个方法获取数据库连接；

```
//获取数据库连接
public Connection getConnection() throws SQLException {
    if (pool.size() > 0) { //数据库连接池中有Connection对象
        //删除并返回集合中的Connection对象
        return pool.removeLast();
    }
    if (curPoolSize < maxPoolSize) { //如果数据库连接池中没有了Connection，但是当前的连接数没有超过可允许的最大连接数据
        Connection conn = createConnection();
        curPoolSize++;
        return conn;
    }
    //如果数据库连接池中没有Connection,而且当前连接数大于等于连接池可允许的最大连接数
    throw new RuntimeException("已经达到最大的连接数！");
}
```
5) 第五步：创建一个方法释放数据库连接；

```
//释放数据库连接
public void releaseConnection(MyConnection conn) {
    //把Connection对象返回给集合中
    pool.add(conn);
}
```
## 1.3 装饰者开发模式
问题：调用conn.close()方法的时候，不是关闭数据库连接，而是把数据库的连接释放到连接池中，如何实现?使用装饰者模式。
 
作用：可以对一些类进行装饰，装饰后的类称之为装饰类。所谓的装饰类就是对原先类的一些方法进行增强处理。
 
实现步骤：
- 第一步：多个装饰类要实现一个共同的接口，或者继承一个共同的父类；
- 第二步：装饰类要保存被装饰类的一个引用；
 
需求：读取一个文件的内容，把读取到的内容输出到控制台。
1) 输出内容的每一行要有行号；
2) 输出内容的每一行的末尾要有换行标签；
3) 输入内容的每一行的要有行号和换行标签；

## 1.4 DBCP连接池技术
1) 第一步：下载DBCP连接池压缩包，并解压缩；
2) 第二步：把DBCP的核心jar包复制到项目中(commons-dbcp-1.4.jar、commons-pool-1.5.6.jar)；
3) 第三步：创建一个BasicDataSource对象；
4) 第四步：设置数据库连接池的参数；
5) 第五步：调用该对象的getConnection方法；


```
public static void main(String[] args) throws SQLException {
        //创建BasicDataSource对象
        BasicDataSource dataSource = new BasicDataSource();
        //设置连接池参数
        //连接数据库的参数
        dataSource.setDriverClassName("com.mysql.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql://localhost:3306/itheima");
        dataSource.setUsername("root");
        dataSource.setPassword("root");
        //连接池的参数
        dataSource.setInitialSize(5); //设置连接池的初始连接数
        dataSource.setMaxActive(10); //连接池的最大连接数
        dataSource.setMaxIdle(10); //设置最大空闲的连接数
        dataSource.setMinIdle(5); //最小空闲的连接数
        dataSource.setMaxWait(3000); //最大等待的时间（毫秒）
 
        System.out.println(dataSource.getConnection());
        System.out.println(dataSource.getConnection());
        System.out.println(dataSource.getConnection());
        System.out.println(dataSource.getConnection());
        System.out.println(dataSource.getConnection());
        System.out.println(dataSource.getConnection());
        System.out.println(dataSource.getConnection());
        System.out.println(dataSource.getConnection());
        System.out.println(dataSource.getConnection());
        Connection conn = dataSource.getConnection();
        System.out.println(conn);
        conn.close();
        System.out.println(dataSource.getConnection());
    }
```
## 1.5 C3P0连接池技术
### 1.5.1 使用C3P0的步骤
1) 第一步：下载c3p0压缩包，解压缩；
2) 第二步：把c3p0的核心jar包导入工程中（c3p0-0.9.5.2.jar、mchange-commons-java-0.2.11.jar）；
3) 第三步：创建连接池对象；
4) 第四步：设置连接池参数；
5) 第五步：获取Connection对象；


```
public static void main(String[] args) throws PropertyVetoException, SQLException {
        //创建连接池对象
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        //数据库连接参数
        dataSource.setDriverClass("com.mysql.jdbc.Driver");
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/itheima");
        dataSource.setUser("root");
        dataSource.setPassword("root");
        //数据库连接池参数
        dataSource.setInitialPoolSize(3); //连接池初始连接数
        dataSource.setMaxPoolSize(5); //连接池的最大连接数
        dataSource.setMinPoolSize(3); //数据库的最小连接数
        dataSource.setAcquireIncrement(2); //每次创建的连接数
        dataSource.setCheckoutTimeout(3000); //最大等待时间（毫秒）
 
        System.out.println(dataSource.getConnection());
        System.out.println(dataSource.getConnection());
        System.out.println(dataSource.getConnection());
        System.out.println(dataSource.getConnection());
        Connection conn = dataSource.getConnection();
        System.out.println(conn);
        conn.close(); //不是关闭连接，是释放连接
        System.out.println(dataSource.getConnection());
    }
```
1.5.1 使用配置文件
1) 方式一：在src目录下创建一个c3p0-config.xml文件（该名字是固定的）；
![image](3083D9606F184F98985DF67F93485630)


```
<?xml version="1.0" encoding="UTF-8"?>
<c3p0-config>
    <!-- 默认配置 -->
    <default-config>
        <!-- 配置参数 -->
        <property name="driverClass">com.mysql.jdbc.Driver</property>
        <property name="jdbcUrl">jdbc:mysql://localhost:3306/itheima?generateParameterMetadata=true</property>
        <property name="user">root</property>
        <property name="password">root</property>
        <property name="initialPoolSize">3</property>
        <property name="maxPoolSize">5</property>
        <property name="minPoolSize">3</property>
        <property name="acquireIncrement">2</property>
        <property name="checkoutTimeout">3000</property>
    </default-config>
</c3p0-config>
```

2) 方式二：在src目录下创建一个c3p0.properties文件（该名字是固定的）；
![image](A5F8F26D34A8488B822F7718E74E7A8E)
![image](25CDF7CD3D3746C5A2CE724711893B7D)

**==注意==：使用XML文件配置方式的优先级更高。**

# 二、 自定义数据库工具类
- 1.1 什么是元数据
元数据就是数据中的数据。例如：获取数据库的版本信息，获取表的字段等等。
- 1.2 获取数据库元数据
- 1.2.1 获取数据库的元数据

DatabaseMetaData：数据库的元数据对象；
![image](D18611EB56884BCD9F30E837957BA6CC)

**1.2.2 获取用户操作的元数据**

ParameterMetaData：参数的元数据对象；
![image](38A1574B7E9B40A697F6179BD3C5CF63)

ResultSetMetaData：获取结果集的元数据对象；

![image](BFD6ABAB8EF94594A74490159E4FD652)

### 1.3 自定义框架实现
1.3.1 增删改功能
BaseDao.java

```
//增删改操作
public void update(String sql, Object[] params) throws SQLException {
    //获取数据库连接
    Connection conn = DbUtil.getConnection();
    //创建PreparedStatement对象
    PreparedStatement pstmt = conn.prepareStatement(sql);

    //问题：如果用户传参数的个数与实际需要传入参数的个数不一致，咋办？
    ParameterMetaData metadata = pstmt.getParameterMetaData();
    //获取参数的个数
    int count = metadata.getParameterCount();
    if (params != null) {
        if (count != params.length) {
            throw new RuntimeException("参数个数不正确！");
        }
        //设置参数
        for (int i = 1; i <= params.length; i++) {
            pstmt.setObject(i, params[i-1]);
        }
    }
    //执行更新操作
    pstmt.executeUpdate();
    //释放连接
    conn.close();
}
```
1.3.2 查询功能
BaseDao.java


```
//查询
public Object find(String sql, Object[] params, ResultSetHandler handler) throws Exception {
    //数据库连接
    Connection conn = DbUtil.getConnection();
    //创建PrepardStatement对象
    PreparedStatement pstmt = conn.prepareStatement(sql);

    ParameterMetaData metadata = pstmt.getParameterMetaData();
    //获取参数的个数
    int count = metadata.getParameterCount();
    //设置参数
    if (params != null) {
        if (count != params.length) {
            throw new RuntimeException("参数个数不正确！");
        }
        //设置参数
        for (int i = 1; i <= params.length; i++) {
            pstmt.setObject(i, params[i-1]);
        }
    }
    //执行查询
    ResultSet rs = pstmt.executeQuery();
    //问题：怎么处理结果集？结果集有可能只返回一条数据，也有可能返回多行数据。
    return handler.handle(rs);
}
```

ResultSetHandler.java

```
public interface ResultSetHandler {
 
    //专门处理结果集的方法
    public Object handle(ResultSet rs) throws Exception;
 
}
```

BeanHandler.java

```
/*
    结果集处理类（专门处理单行的结果）
 
        规则：数据库表的列名必须要与Bean对象的属性名要一致。
*/
public class BeanHandler implements ResultSetHandler {
 
    private Class clazz;
 
    public BeanHandler(Class clazz) {
        this.clazz = clazz;
    }
 
    //对结果集进行处理，处理完成后返回一个Bean对象
    public Object handle(ResultSet rs) throws Exception {
        //先移动指针
        if (rs.next()) {
            //通过class类创建对象
            Object o = clazz.newInstance();
            //获取结果集的元数据对象
            ResultSetMetaData metadata = rs.getMetaData();
            //获取列的数量
            int count = metadata.getColumnCount();
            //获取所有列的数据
            for (int i = 1; i <= count; i++) {
                //获取了每一列的数据，把每一列数据设置class对象中？
                //获取列名
                String fieldName = metadata.getColumnLabel(i);
                //使用反射设置对象的属性
                Field field = clazz.getDeclaredField(fieldName);
                //暴力反射
                field.setAccessible(true);
                //设置对象的属性值
                field.set(o, rs.getObject(i));
            }
            return o;
        }
        return null;
    }
 
}
```

BeanListHandler.java

```
/*
    结果集处理类（专门处理多行结果的处理类）
*/
public class BeanListHandler implements ResultSetHandler {
 
    private Class clazz;
 
    public BeanListHandler(Class clazz) {
        this.clazz = clazz;
    }
 
    //对结果集进行处理，处理完成后返回一个Bean对象
    public List handle(ResultSet rs) throws Exception {
        List result = new ArrayList();
        //遍历结果集
        while (rs.next()) {
            //通过class类创建对象
            Object o = clazz.newInstance();
            //获取结果集的元数据对象
            ResultSetMetaData metadata = rs.getMetaData();
            //获取列的数量
            int count = metadata.getColumnCount();
            //获取所有列的数据
            for (int i = 1; i <= count; i++) {
                //获取了每一列的数据，把每一列数据设置class对象中？
                //获取列名
                String fieldName = metadata.getColumnLabel(i);
                //使用反射设置对象的属性
                Field field = clazz.getDeclaredField(fieldName);
                //暴力反射
                field.setAccessible(true);
                //设置对象的属性值
                field.set(o, rs.getObject(i));
            }
            result.add(o);
        }
        return result;
    }
 
}
```

# 三、 DbUtil
**3.1 DbUtil介绍**

作用：简化一些数据库的操作。

**3.2 DbUtil的使用步骤**
- 第一步：导入DbUtil的核心jar包；
- 第二步：创建QueryRunner对象；
- 第三步：调用该对象一些方法访问数据库；


```
public class Demo5 {
    //不带事务
    private static QueryRunner queryRunner = new QueryRunner(new ComboPooledDataSource());
    //带事务
    private static QueryRunner queryRunner2 = new QueryRunner();
 
    public static void main(String[] args) throws SQLException {
        String sql = "select * from employees where id = 2";
        List<Employee> empList = (List<Employee>) queryRunner.query(
                sql, new BeanListHandler(Employee.class));
        System.out.println(empList);
 
        Employee emp = (Employee) queryRunner.query(
                sql, new BeanHandler(Employee.class));
        System.out.println(emp);
 
        sql = "insert into employees(name, age, sal) values(?, ?, ?)";
        queryRunner.update(sql, new Object[]{"大大宝", 60,100000});
 
        sql = "delete from employees where id = ?";
        queryRunner.update(sql, 5);
```

带事务的QueryRunner：

```
//开启事务
    Connection conn = DbUtil.getConnection();
    try {
        conn.setAutoCommit(false);
        String sql = "insert into employees(name, age, sal) values(?, ?, ?)";
        queryRunner2.update(conn, sql, new Object[]{"大大宝1号", 60, 100000});
        sql = "insert into employees(name, age, sal) values(?, ?, ?)";
        queryRunner2.update(conn, sql, new Object[]{"大大宝2号", 60, 100000});
        sql = "insert into employees(name, age, sal) values(?, ?, ?)";
        queryRunner2.update(conn, sql, new Object[]{"大大宝3号", 60, 100000});
        int i = 10 / 0;
        conn.commit();
    } catch (Exception e) {
        conn.rollback();
        System.out.println("事务回滚了！");
        //throw new RuntimeException(e);
    } finally {
        conn.close();
    }
}
```

### 3.4 批处理
执行批处理操作需要传入一个二维数组作为参数。第一维就是参数个数，第二维存储了每个参数的内容；


```
package cn.yi.test;
 
 
import java.sql.SQLException;
import java.util.Random;
 
import org.apache.commons.dbutils.QueryRunner;
import org.junit.Test;
 
import cn.yi.util.DbUtil;
/*
 * 代码测试
 */
public class TestCode {
    private static QueryRunner queryRunner = new QueryRunner(DbUtil.cpds);
 
    //数据库数据准备,使用批处理
    @Test
    public void dataPrepare() throws SQLException{
        Random random = new Random();
        long starTime=System.currentTimeMillis();
        String sql = "INSERT INTO student(NAME,age,result) VALUES(?,?,?)";
        Object[][] params = new Object[1500][3];
        String[] str = new String[]{"张三", "李四", "王五", "赵六", "小明", "田七"};
        int i;
        for (i = 0; i < 1500; i++){
            int randomNum = random.nextInt(str.length);
            params[i][0] = str[randomNum];
            params[i][1] = "21";
            params[i][2] = i;
        }
        queryRunner.batch(sql, params);
        long endTime=System.currentTimeMillis();
        long Time=(endTime-starTime);
        System.out.println("运行时间："+Time + "ms");
    }
}
```
