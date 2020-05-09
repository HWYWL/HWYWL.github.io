---
title: Spring MVC数据自动绑定
date: 2017-05-27 16:11:35
tags: 
 - Spring
categories: 
 - Spring
keywords: "Spring"
description: Spring MVC数据自动绑定。
---

通过例子介绍几种自动绑定方式：

SpringmvcServletAPI .java

```
package com.yi.controller;

import com.yi.entity.User;
import com.yi.entity.Users;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpSession;

/**
 * 数据自动绑定
 * Created by Administrator on 2017/4/17.
 */
@Controller
public class SpringmvcServletAPI {

    /**
     * 将字段与表单字段一一对应就能自动绑定
     * @param session
     * @return
     */
    @RequestMapping(value = "login")
    public String login(HttpSession session, String username){
        System.out.println("用户名：" + username);
        session.setAttribute("username",username);
        return "index.jsp";
    }

    /**
     * 将pojo(Javabean)的字段与前端字段一一对应就能自动绑定
     * @param request
     * @param user
     * @return
     */
    @RequestMapping(value = "getup")
    public String login(HttpServletRequest request, User user){
        System.out.println("账号：" + user.getUsername());
        System.out.println("密码：" + user.getPassword());

        request.setAttribute("user",user);
        return "index.jsp";
    }

    /**
     * 将list集合中元素的字段与前端字段一一对应就能自动绑定
     * @param request
     * @param users
     * @return
     */
    @RequestMapping(value = "batchinput")
    public String login(HttpServletRequest request,Users users){
        System.out.println("长度：" + users.getUsers().size());
        for (User user:users.getUsers()
             ) {
            System.out.println("账号：" + user.getUsername());
            System.out.println("密码：" + user.getPassword());
        }
        request.setAttribute("list",users.getUsers());

        return "index.jsp";
    }

    /**
     * 将数组的字段与前端字段对应就能自动绑定
     * @param request
     * @param username
     * @return
     */
    @RequestMapping(value = "arrayinput")
    public String login(HttpServletRequest request,String[] username){
        for (String name:username
             ) {
            System.out.println("姓名：" + name);
        }

        request.setAttribute("nameArr",username);
        return "index.jsp";
    }
}
```

User.java


```
package com.yi.entity;

/**
 * 用户信息pojo
 * Created by Administrator on 2017/4/18.
 */
public class User {
    private Integer uid;
    private String username;
    private String password;

    public User() {
        super();
    }

    public User(Integer uid, String username, String password) {
        this.uid = uid;
        this.username = username;
        this.password = password;
    }

    public User(String username, String password) {
        this.username = username;
        this.password = password;
    }

    public Integer getUid() {
        return uid;
    }

    public void setUid(Integer uid) {
        this.uid = uid;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    @Override
    public String toString() {
        return "User{" +
                "uid=" + uid +
                ", username='" + username + '\'' +
                ", password='" + password + '\'' +
                '}';
    }
}
```

Users.java


```
package com.yi.entity;

import java.util.ArrayList;
import java.util.List;

/**
 * 封装User到集合中，用户数据绑定
 * Created by Administrator on 2017/4/18.
 */
public class Users {
    List<User> users = new ArrayList<>();

    public List<User> getUsers() {
        return users;
    }

    public void setUsers(List<User> users) {
        this.users = users;
    }
}
```

index.jsp


```
<%--
  Created by IntelliJ IDEA.
  User: Administrator
  Date: 2017/4/17
  Time: 20:28
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<html>
  <head>
    <title>$Title$</title>
  </head>
  <body>
    <h3>SpringMVC自动绑定字段</h3><hr>
    ${username},欢迎回来...<br>

    <h3>SpringMVC自动绑定javabean</h3><hr>
    ${requestScope.user.username},欢迎回来...<br>
    ${requestScope.user.password},这是密码<br>

    <h3>SpringMVC自动绑定集合</h3><hr>
    <c:forEach items="${list}" var="user">
        ${user.username},欢迎回来...<br>
        ${user.password},这是密码<br>
    </c:forEach>

    <h3>SpringMVC自动绑定数组</h3><hr>
    <c:forEach items="${nameArr}" var="name">
        ${name},欢迎回来...<br>
    </c:forEach>
  </body>
</html>
```

login.jsp


```
<%--
  Created by IntelliJ IDEA.
  User: Administrator
  Date: 2017/4/17
  Time: 21:45
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>登录</title>
</head>
<body>
    <h3>SpringMVC自动绑定字段</h3><hr>
    <form action="${pageContext.request.contextPath}/login.mvc" method="post">
        账号：<input type="text" name="username"><br><br>
        <input type="submit">
    </form>

    <h3>SpringMVC自动绑定javabean</h3><hr>
    <form action="${pageContext.request.contextPath}/getup.mvc" method="post">
        账号：<input type="text" name="username"><br><br>
        密码：<input type="text" name="password"><br><br>
        <input type="submit">
    </form>

    <h3>SpringMVC自动绑定集合</h3><hr>
    <form action="${pageContext.request.contextPath}/batchinput.mvc" method="post">
        账号：<input type="text" name="users[0].username"><br><br>
        密码：<input type="text" name="users[0].password"><br><br>
        账号：<input type="text" name="users[1].username"><br><br>
        密码：<input type="text" name="users[1].password"><br><br>
        账号：<input type="text" name="users[2].username"><br><br>
        密码：<input type="text" name="users[2].password"><br><br>
        <input type="submit">
    </form>

    <h3>SpringMVC自动绑定数组</h3><hr>
    <form action="${pageContext.request.contextPath}/arrayinput.mvc" method="post">
        账号：<input type="text" name="username"><br><br>
        账号：<input type="text" name="username"><br><br>
        <input type="submit">
    </form>
</body>
</html>
```

结果：

![image](https://s2.ax1x.com/2020/02/27/3d7b0P.png)
<hr>
![image](https://s2.ax1x.com/2020/02/27/3dHM0x.png)