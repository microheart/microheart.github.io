---
layout: post
title: Redis 笔记 
category: Redis
tags: 
keywords: 
description: 
---


Redis是一个高性能，基于内存的key-value存储系统。
和Memcached类似，它支持存储的value类型相对更多，且Redis会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件。

此系列文章通过阅读`Redis2.6`的源码，做了几篇笔记，希望和对Redis感兴趣的朋友们一起学习，共同进步。在阅读文章中，如有建议敬请反馈，谢谢。



参考文献：

[Redis 设计与实现](http://origin.redisbook.com/en/latest/)