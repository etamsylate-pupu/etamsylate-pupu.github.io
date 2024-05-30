---
title: 项目开发-Redis篇
tags: [redis]
date: 2024-04-24 10:22:23
categories: technique
urlname: 33
---

> Redis is an **in-memory data store** used by millions of developers as a cache, **vector database**, **document database**, **streaming engine**, and message broker. Redis has built-in replication and different levels of **on-disk persistence**. It supports complex data types (e.g., strings, hashes, lists, sets, sorted sets, and JSON), with atomic operations defined on those data types.

小林Coding对Resdis的介绍如下：

[REmote DIctionary Server(Redis)][1] 是一个开源（BSD许可）的基于内存的非关系型数据库，对数据的读写操作都是在内存中完成，因此**读写速度非常快**，常用于**缓存，消息队列、分布式锁等场景**。

Redis 提供了多种数据类型来支持不同的业务场景，比如 String(字符串)、Hash(哈希)、 List (列表)、Set(集合)、Zset(有序集合)、Bitmaps（位图）、HyperLogLog（基数统计）、GEO（地理信息）、Stream（流），并且对数据类型的操作都是**原子性**的，因为执行命令由**单线程**负责的，不存在并发竞争的问题。

除此之外，Redis 还支持**事务 、持久化、Lua 脚本、多种集群方案（主从复制模式、哨兵模式、切片机群模式）、发布/订阅模式，内存淘汰机制、过期删除机制**等等。


Redis高性能的原因：

- 基于内存，内存的访问速度比磁盘快很多
- 基于 Reactor 模式设计开发了一套高效的事件处理模型，主要是单线程事件循环和 IO 多路复用
- 内置了多种优化过后的数据类型/结构实现，性能非常高
- 通信协议实现简单且解析高效


> 




参考资料：

[小林Coding-Redis 常见面试题][2]
[JavaGuide- Redis常见面试题总结(上)][4]
[高性能网络编程之 Reactor 网络模型（彻底搞懂）][5]

[1]: https://redis.io/
[2]: https://www.xiaolincoding.com/redis/base/redis_interview.html#%E4%BB%80%E4%B9%88%E6%98%AF-redis
[3]: https://try.redis.io/
[4]: https://javaguide.cn/database/redis/redis-questions-01.html
[5]: https://juejin.cn/post/7092436770519777311
