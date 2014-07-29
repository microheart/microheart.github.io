---
layout: post
title: Redis应用场景 
category: Redis
tags: Redis
keywords: Redis,应用场景
description: 
---


## 1.取最新N个数据的操作
比如典型的取网站的最新文章，通过下面方式，我们可以将最新的500条评论的ID放在Redis的List集合中，并将超出集合部分从数据库获取。

* 使用LPUSH latest.comments<ID>命令，向list集合中插入数据 
* 插入完成后再用LTRIM latest.comments 0 500命令使其永远只保存最近500个ID 
* 然后我们在客户端获取某一页评论时可以用下面的逻辑 

伪代码

	def get_latest_comments(start,num_items):
	    id_list = redis.lrange("latest.comments",start,start+num_items-1)
	    if ( id_list.length < num_items )
	    	id_list = SQL_DB("SELECT ... ORDER BY time LIMIT ...") 
		return id_list 
	 

如果你还有不同的筛选维度，比如某个分类的最新N条，那么你可以再建一个按此分类的List，只存ID的话，Redis是非常高效的。


## 2.排行榜应用，取TOP N操作
这个需求与上面需求的不同之处在于，前面操作以时间为权重，这个是以某个条件为权重，比如按顶的次数排序，这时候可以应用sorted set，将你要排序的值设置成sorted set的score，将具体的数据设置成相应的value，每次只需要执行一条ZADD命令即可。

## 3.需要精准设定过期时间的应用
比如你可以把上面说到的sorted set的score值设置成过期时间的时间戳，那么就可以简单地通过过期时间排序，定时清除过期数据了，不仅是清除Redis中的过期数据，你完全可以把Redis里这个过期时间当成是对数据库中数据的索引，用Redis来找出哪些数据需要过期删除，然后再精准地从数据库中删除相应的记录。

## 4.计数器应用
Redis的命令都是原子性的，你可以轻松地利用INCR，DECR命令来构建计数器系统。

## 5.Uniq操作，获取某段时间所有数据排重值
这个使用Redis的set数据结构最合适了，只需要不断地将数据往set中扔就行了，set意为集合，所以会自动排重。

## 6.实时系统，反垃圾系统
通过上面说到的set功能，你可以知道一个终端用户是否进行了某个操作，可以找到其操作的集合并进行分析统计对比等。没有做不到，只有想不到。

## 7.Pub/Sub构建实时消息系统
Redis的Pub/Sub系统可以构建实时的消息系统，比如很多用Pub/Sub构建的实时聊天系统的例子。

## 8.构建队列系统
使用list可以构建队列系统，使用sorted set甚至可以构建有优先级的队列系统。

## 9.缓存
这个不必说了，性能优于Memcached，数据结构更多样化。



