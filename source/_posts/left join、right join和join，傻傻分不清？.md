---
title: left join、right join和join，傻傻分不清？
date: 2020-04-17 15:08:26
tags: 
 - MySQL
categories: 
 - 数据库
keywords: "MySQL"
description: left join、right join和join，傻傻分不清？。
---

说到SQL，很多人可能用了挺久，但依然有个问题一直困扰着，那就是 **left join**、 **join**、 **right join**和 **inner join**等等各种 join的区别。网上搜，最常见的就是一张图解图，如下：

![YeAVTP.png](https://s1.ax1x.com/2020/05/07/YeAVTP.png)

接下来就来实际自己动手实验，彻底搞懂图中的含义。

首先，先来建两张表，第一张表命名为 **kemu**，第二张表命名为 **score**：

![YeA3mn.png](https://s1.ax1x.com/2020/05/07/YeA3mn.png)

## left join 
顾名思义，就是“左连接”，表1左连接表2，以左为主，表示以表1为主，关联上表2的数据，查出来的结果显示左边的所有数据，然后右边显示的是和左边有交集部分的数据。如下：

```sql
SELECT
	* 
FROM
	kemu
	LEFT JOIN score ON kemu.ID = score.ID
```

结果集：

![YeAXcQ.png](https://s1.ax1x.com/2020/05/07/YeAXcQ.png)

## right join 
“右连接”，表1右连接表2，以右为主，表示以表2为主，关联查询表1的数据，查出表2所有数据以及表1和表2有交集的数据，如下：

```sql
SELECT
	* 
FROM
	kemu
	RIGHT JOIN score ON kemu.ID = score.ID
```

结果集：

![YeEUEt.png](https://s1.ax1x.com/2020/05/07/YeEUEt.png)

##  join（inner join）
join，其实就是“inner join”，为了简写才写成join，两个是表示一个的，内连接，表示以两个表的交集为主，查出来是两个表有交集的部分，其余没有关联就不额外显示出来，这个用的情况也是挺多的，如下

```sql
SELECT
	* 
FROM
	kemu
	JOIN score ON kemu.ID = score.ID
```

结果集：

![YeE6Ds.png](https://s1.ax1x.com/2020/05/07/YeE6Ds.png)

以后对于这三者应该再也不模糊了吧！