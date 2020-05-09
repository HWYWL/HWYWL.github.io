---
title: 重磅：Redis 5.0 正式版发布了，19 个新特性！
date: 2019-11-02 16:32:05
tags: 
 - 缓存
 - Redis
categories: 
 - 缓存
keywords: "缓存,Redis"
description: 重磅：Redis 5.0 正式版发布了，19 个新特性！。
---

![](https://i.imgur.com/kxrr5OC.jpg)

# Redis 5.0 GA 正式版发布了！


```
下载地址：
download.redis.io/releases/redis-5.0.0.tar.gz
源码下载：
github.com/antirez/redis/releases/tag/5.0.0
```

先看一下 Redis 5 带来的更新内容：

- 1. 新的流数据类型(Stream data type) https://redis.io/topics/streams-intro
- 2.新的 Redis 模块 API：定时器、集群和字典 API(Timers, Cluster and Dictionary APIs)
- 3. RDB 现在可存储 LFU 和 LRU 信息
- 4.redis-cli 中的集群管理器从 Ruby (redis-trib.rb) 移植到了 C 语言代码。执行 `redis-cli --- cluster help` 命令以了解更多信息
- 5. 新的有序集合(sorted set)命令：ZPOPMIN/MAX 和阻塞变体(blocking variants)
- 6. 升级 Active defragmentation 至 v2 版本
- 7. 增强 HyperLogLog 的实现
- 8. 更好的内存统计报告
- 9. 许多包含子命令的命令现在都有一个 HELP 子命令
- 10. 客户端频繁连接和断开连接时，性能表现更好
- 11. 许多错误修复和其他方面的改进
- 12. 升级 Jemalloc 至 5.1 版本
- 13.  引入 CLIENT UNBLOCK 和 CLIENT ID
- 14.  新增 LOLWUT 命令 http://antirez.com/news/123
- 15.  在不存在需要保持向后兼容性的地方，弃用 "slave" 术语
- 16.  网络层中的差异优化
- 17.  Lua 相关的改进：
```
将 Lua 脚本更好地传播到 replicas / AOF
Lua 脚本现在可以超时并在副本中进入 -BUSY 状态
```

- 18.  引入动态的 HZ(Dynamic HZ) 以平衡空闲 CPU 使用率和响应性
- 19.  对 Redis 核心代码进行了重构并在许多方面进行了改进

Redis 5 是 Redis 引入流数据类型(Stream data type)的第一个版本。按照官方的说法，不使用该特性的用户在生产环境中使用 Redis 5 会有更好的体验 —— 虽然开发团队尚未发现关于这项特性的关键错误。

此外，因为许多内部结构与 Redis 4 共享，因此在内部工作方式方面，变化不会很大。