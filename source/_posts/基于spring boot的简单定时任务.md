---
title: 基于spring boot的简单定时任务
date: 2018-11-05 08:11:31
tags: 
 - Spring Boot
 - Scheduled
categories: 
 - Spring Boot
keywords: "Spring Boot,Scheduled"
description: 基于spring boot的简单定时任务。
---

# spring-boot-scheduled

### 说明
使用自带的定时任务非常简单，我们只需要打开定时器
```java
/**
 * 定时任务配置类
 * @author YI
 * @date 2018-11-5 10:04:06
 */
@Configuration
@EnableScheduling
public class Config {

}
```

然后编写我们的任务即可完成一个简单的定时任务
```java
/**
 * 定时任务
 * @author YI
 * @date 2018-11-5 10:30:08
 */
@Component
public class ScheduledTasks {
    private static final Logger log = LoggerFactory.getLogger(ScheduledTasks.class);

    private static final SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

    /**
     * 5秒钟执行一次
     */
    @Scheduled(fixedRate = 5000)
    public void reportCurrentTime() {
        log.info("现在时间是 {}", dateFormat.format(new Date()));
    }
}
```

最后我们来看看执行效果

![](https://i.imgur.com/xbCIXBv.jpg)

### 问题建议

- 联系我的邮箱：ilovey_hwy@163.com
- 我的博客：http://www.hwy.ac.cn
- GitHub：[源码](https://github.com/HWYWL/spring-boot-2.x-examples/tree/master/spring-boot-scheduled)