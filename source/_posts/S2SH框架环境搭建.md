---
title: S2SH框架环境搭建
date: 2017-12-14 10:16:15
tags: 
 - ORM
categories: 
 - ORM
keywords: "ORM"
description: S2SH框架环境搭建。
---

# 1.使用HibernateDaoSupport简化整合代码
原理：HibernateDaoSupport类有一个HibernateTemplate成员变量，提供一个setSessionFactory的方法，用于注入SessionFactory，把HibernateTemplate创建出来。

![image](https://s2.ax1x.com/2020/02/27/3doan0.png)

1）让dao类继承HibernateDaoSupport类


```
public class ProductDao extends HibernateDaoSupport implements IProductDao {
 
@Override
public List<Product> findAll() {
return this.getHibernateTemplate().loadAll(Product.class);
}
 
@Override
public void save(Product p) {
this.getHibernateTemplate().save(p);
}
}
```

2）简化applicationContext.xml配置文件


```
<bean id="sessionFactory" class="org.springframework.orm.hibernate5.LocalSessionFactoryBean">
<!-- configLocation：用于加载hibernate.cfg.xml文件 -->
<property name="configLocation" value="classpath:/hibernate.cfg.xml"/>
</bean>
<!-- 创建dao -->
<bean id="productDao" class="gz.itheima.dao.impl.ProductDao">
<property name="sessionFactory" ref="sessionFactory"/>
</bean>
```

# 2.Spring整合Hibernate 的两套方案
2.1 存在hibernate.cfg.xml文件

```
<bean id="sessionFactory" class="org.springframework.orm.hibernate5.LocalSessionFactoryBean">
<!-- configLocation：用于加载hibernate.cfg.xml文件 -->
<property name="configLocation" value="classpath:/hibernate.cfg.xml"/>
</bean>
```

2.2 不需要hibernate.cfg.xml文件（重点看）


```
<bean id="sessionFactory" class="org.springframework.orm.hibernate5.LocalSessionFactoryBean">
<!-- 注入连接池 -->
<property name="dataSource" ref="dataSource"/>
<!-- 注入hibernate属性 -->
<property name="hibernateProperties">
<props>
<prop key="hibernate.dialect">org.hibernate.dialect.MySQL5Dialect</prop>
<prop key="hibernate.show_sql">true</prop>
<prop key="hibernate.format_sql">true</prop>
<prop key="hibernate.hbm2ddl.auto">update</prop>
</props>
</property>
<!-- 注入映射文件的信息 -->
<property name="mappingResources">
<array>
<value>gz/itheima/entity/Category.hbm.xml</value>
<value>gz/itheima/entity/Product.hbm.xml</value>
</array>
</property>
</bean>
```

# 3.Spring整合Struts2
3.1 单独开发业务层，把事务切换到业务层

1)类

```
public class ProductBiz implements IProductBiz {
    //注入dao
    private IProductDao productDao;
        public void setProductDao(IProductDao productDao) {
        this.productDao = productDao;
    }
     
    @Override
        public List<Product> findAll() {
        return productDao.findAll();
    }
     
    @Override
        public void save(Product p) {
        productDao.save(p);
    }
     
}
```

2）配置

```
<!-- 创建biz -->
<bean id="prductBiz" class="gz.itheima.biz.impl.ProductBiz">
<property name="productDao" ref="productDao"/>
</bean>
```

修改事务配置，把切入点换成业务层类


```
<!-- 配置Spring的事务管理 -->
<!-- 1.创建事务管理器 -->
<bean id="transactionManager" class="org.springframework.orm.hibernate5.HibernateTransactionManager">
<property name="sessionFactory" ref="sessionFactory"/>
</bean>
<!-- 2.创建事务通知 -->
<tx:advice id="txAdvice" transaction-manager="transactionManager">
<tx:attributes>
<tx:method name="*" isolation="DEFAULT" propagation="REQUIRED"/>
</tx:attributes>
</tx:advice>
<!-- 3.配置切面 -->
<aop:config>
<aop:pointcut expression="execution(* gz.itheima.biz.impl.*Biz.*(..))" id="myPT"/>
<aop:advisor advice-ref="txAdvice" pointcut-ref="myPT"/>
</aop:config>
```

3）测试


```
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration({"classpath:/applicationContext.xml"})
public class Demo1 {
    //注入biz
    @Resource(name="productBiz")
    private IProductBiz productBiz;
    @Test
    public void test1(){
        Product p  =new Product();
        p.setName("vivo");
        productBiz.save(p);
    }
}
```

注意：一旦把事务换到了业务层后，dao层的测试代码就不可用了！因为dao层没有事务啦!

3.2 单独使用Struts2框架

1）包

Struts2最小包（12）

**注意：不要导入javassist-3.11.0.GA.jar,因为hibernate框架有了这个包，并且版本更高！**

2）类和页面

Add.jsp页面


```
<s:form action="product_add">
  <table border="1">
  	<tr>
  	<td>产品名称</td>
  	<td><s:textfield name="name"/></td>
  	</tr>
  	<tr>
  	<td><input type="submit" value="保存"/></td>
  	</tr>
  </table>
  </s:form>
```

ProductAction


```
public class ProductAction extends ActionSupport implements ModelDriven<Product>{
//接收参数
    private Product p = new Product();
    /**
     * 保存数据
     */
    public String add(){
        System.out.println(p);
        //调用业务
        //回显信息页面
        ActionContext ac = ActionContext.getContext();
        ac.put("msg", "添加成功");
        return SUCCESS;
    }
     
    @Override
        public Product getModel() {
        return p;
    }
}
```

3）配置

Web.xml


```
<!-- struts的核心过滤器 -->
  <filter>
  	<filter-name>struts2</filter-name>
<filter-class>org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter</filter-class>
  </filter>
  <filter-mapping>
  	<filter-name>struts2</filter-name>
  	<url-pattern>/*</url-pattern>
  </filter-mapping>
```
Struts.xml


```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE struts PUBLIC
"-//Apache Software Foundation//DTD Struts Configuration 2.3//EN"
"http://struts.apache.org/dtds/struts-2.3.dtd">
 
<struts>
<!-- 修改struts的UI模块 ,该为简单模板-->
<constant name="struts.ui.theme" value="simple"/>
 
<package name="ssh" extends="struts-default" namespace="/">
<action name="product_*" class="gz.itheima.web.ProductAction" method="{1}">
<result>/succ.jsp</result>
</action>
</package>
 
</struts>
```

4）测试

部署tomcat，浏览器访问：

http://localhost:8080/day38_04_ssh_1/add.jsp

3.3 Spring框架整合Strut2

1）在Spring没有整合Struts2的情况下，需要手动到spring的IOC工厂获取biz对象


```
//1.不整合的情况下，直接调用spring的IOC工厂的biz的对象
ApplicationContext context = new ClassPathXmlApplicationContext("/applicationContext.xml");
productBiz = (IProductBiz)context.getBean("productBiz");
productBiz.save(p);
```


```
注意：这种情况下，action每次都需要重新读取applicationContext.xml文件，重新创建IOC工厂，效率是非常低的！
```

2）Spring框架整合struts2（重点）
 
原理：Spring框架提供了一个web监听器，放在spring-web的jar里面。

![image](https://s2.ax1x.com/2020/02/27/3doBAU.png)

这个web监听器（ContextLoaderListener），主要是用于在项目启动的时候，一次性地把applicationContext.xml文件加载进内存，方便表现层去使用spring的对象。

![image](https://s2.ax1x.com/2020/02/27/3doccR.png)

整合步骤：

1）导入spring-web的jar包

![image](https://s2.ax1x.com/2020/02/27/3doWB6.png)

2）在web.xml配置web监听器


```
 <!-- 配置spring的web监听器，用于加载applicationContext.xml -->
  <!-- 默认情况下，到WEB-INF目录下读取applicationContext.xml -->
  <listener>
  	<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>
  <!-- 修改web监听器读取文件的路径 -->
  <context-param>
  	<param-name>contextConfigLocation</param-name>
  	<param-value>classpath:applicationContext.xml</param-value>
  </context-param>
```

3）修改action代码，从内存取出biz对象


```
//2.Spring整合struts2
//从内存中取出biz对象
ApplicationContext context = 
WebApplicationContextUtils.getWebApplicationContext(ServletActionContext.getServletContext());
productBiz = (IProductBiz)context.getBean("productBiz");
productBiz.save(p);
```

注意：这种情况下，action的代码比较麻烦，所以struts2提供了整合spring的方案，简化action获取biz对象的代码

3.4 Struts2框架整合Spring

1）导入struts2-spring-plugin的jar包

![image](https://s2.ax1x.com/2020/02/27/3dTpCQ.png)

Struts-plugin.xml : 编写了一个拦截器。主要用于让action获取到spring环境的对象

![image](https://s2.ax1x.com/2020/02/27/3do73d.png)

2）修改action类和配置文件
 
方案一：Action对象是由Struts2创建的（根据spring的名称（bean的id）自动匹配）


```
private IProductBiz productBiz;
public void setProductBiz(IProductBiz productBiz) {
this.productBiz = productBiz;
}
<!-- 创建biz -->
<bean id="productBiz" class="gz.itheima.biz.impl.ProductBiz">
<property name="productDao" ref="productDao"/>
</bean>
```

方案二：Action对象是由Spring创建  （推荐使用）
 
在applicationContext.xml创建action对象


```
<!-- 创建 action -->
<bean id="productAction" class="gz.itheima.web.ProductAction">
<property name="prdoBiz" ref="productBiz"/>
</bean>
```

修改struts.xml文件的class属性


```
<action name="product_*" class="productAction" method="{1}">
<result>/succ.jsp</result>
</action>
```
![image](https://s2.ax1x.com/2020/02/27/3dTVET.png)

注意：在方案二的情况，spring创建出来的action必须是设置成多例，否则会出现线程并发问题！


```
<bean id="productAction" class="gz.itheima.web.ProductAction" scope="prototype">
<property name="prdoBiz" ref="productBiz"/>
</bean>
```

# 4.解决Hibernate延迟加载数据显示问题
原理：spring框架设计了一个过滤器：OpenSessionInViewFilter

![image](https://s2.ax1x.com/2020/02/27/3dTK29.png)

![image](https://s2.ax1x.com/2020/02/27/3dT1Dx.png)

配置步骤：

在web.xml配置过滤器


```
 <!-- 注意：OpenSessionInViewFilter必须配置在struts2的前面 -->
  <filter>
    <filter-name>OpenSessionInViewFilter</filter-name>
    <filter-class>org.springframework.orm.hibernate5.support.OpenSessionInViewFilter</filter-class>
  </filter>
  <filter-mapping>
    <filter-name>OpenSessionInViewFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
```

注意：必须把这个过滤器配置在struts2的配置前面才会生效！


### 问题建议

- 联系我的邮箱：ilovey_hwy@163.com
- 我的博客：http://hwy.ac.cn
- GitHub：https://www.github.com/HWYWL