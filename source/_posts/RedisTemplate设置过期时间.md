---
title: RedisTemplate设置过期时间
date: 2019-09-28 16:32:05
tags: 
 - 缓存
 - Redis
categories: 
 - 缓存
keywords: "缓存,Redis"
description: RedisTemplate设置过期时间。
---


# 通过spring boot 中的Redis模板设置数据的过期时间
```
//向redis里存入数据和设置缓存时间  
stringRedisTemplate.opsForValue().set("baike", "100", 60 * 10, TimeUnit.SECONDS);

//val做-1操作  
stringRedisTemplate.boundValueOps("baike").increment(-1);

//根据key获取缓存中的val  
stringRedisTemplate.opsForValue().get("baike")

//val +1  
stringRedisTemplate.boundValueOps("baike").increment(1);

//根据key获取过期时间  
stringRedisTemplate.getExpire("baike");

//根据key获取过期时间并换算成指定单位  
stringRedisTemplate.getExpire("baike",TimeUnit.SECONDS);

//根据key删除缓存  
stringRedisTemplate.delete("baike");

//检查key是否存在，返回boolean值  
stringRedisTemplate.hasKey("baike");

//向指定key中存放set集合  
stringRedisTemplate.opsForSet().add("baike", "1","2","3");

//设置过期时间  
stringRedisTemplate.expire("baike",1000 , TimeUnit.MILLISECONDS);

//根据key查看集合中是否存在指定数据  
stringRedisTemplate.opsForSet().isMember("baike", "1");

//根据key获取set集合  
stringRedisTemplate.opsForSet().members("baike");

//验证有效时间
Long expire = redisTemplate.boundHashOps("baike").getExpire();
System.out.println("redis有效时间："+expire+"S");
```