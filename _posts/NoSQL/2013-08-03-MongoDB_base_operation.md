---
layout: post
title: MongoDB基本操作
category: NoSQL
tags: NoSQL
keywords: MongodDB
description: 
---

##添加
1 **连接，选择数据库**

    >mongo 
    >use mydb
	switched to db mydb
mongo默认连接本地的mongodb服务器的27017端口，如果要连接到不同服务器和端口，使用`--host`和`--port`选项，如`mongo --host localhost --port 27017`。
**注**：若不存在mydb数据库，`use mydb`并不会创建数据库。数据库和集合的创建是“lazy”的，即只有在第一个document被插入时集合和数据库才真正创建——这时在磁盘的文件系统里才能看见。

2 **添加记录**

用法：`db.collection.insert(<document>)`

	> var man = {name:"jack"}
	> db.person.insert(man)
	> db.person.find()
	{ "_id" : ObjectId("51f0e5af4d50ac57eaa3fa35"), "name" : "jack" }
插入man文档到集合person中，也可一次插入多条记录`db.collection.insert([d1,d2,...dn])`。

**MongoDB VS 关系型数据库**

* MongoDB的文档(document)，相当于关系数据库中的一行记录；
* 多个文档组成一个集合(collection)，相当于关系数据库的表；
* 多个集合(collection)，逻辑上组织在一起，就是数据库(database)；
* 一个 MongoDB实例支持多个数据库(database)。

如果文档中不包含`_id`属性，则`db.collection.save(<document>)`相当于`insert()`.

**添加操作注意事项**

* 若插入的文档中没有`_id`属性，则mongod会添加`_id`属性，并保证其值唯一。
* 若插入的文档中指定了`_id`的值，则必须保证其值在集合中唯一。
* 最大的BSON文档大小为16MB。
* 命名限制，属性名不能以`$`开始，且不能包含`.`字符

##更新
update()操作接受一个upsert标志，如果更新的文档不存在且upsert为true，则相当于insert()操作。
update()用法：

	db.collection.update(<query>,<update>,{upsert: true})
"_id" : ObjectId("51f0e5af4d50ac57eaa3fa35")
##查询
    
    >var cnt = 50;
    >for(var i = 0; i < cnt; ++i) {
    	var tempName = "james" + i;
    	var tempAge = 16 + i;
    	db.person.insert({name:tempName,age:tempAge});
     }
添加50条记录到集合person中,

	> db.person.find()
	{ "_id" : ObjectId("51f0e5af4d50ac57eaa3fa35"), "name" : "jack" }
	{ "_id" : ObjectId("51f0e5f54d50ac57eaa3fa36"), "name" : "james0", "age" : 16 }
	{ "_id" : ObjectId("51f0e5f54d50ac57eaa3fa37"), "name" : "james1", "age" : 17 }
	{ "_id" : ObjectId("51f0e5f54d50ac57eaa3fa38"), "name" : "james2", "age" : 18 }
	{ "_id" : ObjectId("51f0e5f54d50ac57eaa3fa39"), "name" : "james3", "age" : 19 }
	{ "_id" : ObjectId("51f0e5f54d50ac57eaa3fa3a"), "name" : "james4", "age" : 20 }
	{ "_id" : ObjectId("51f0e5f54d50ac57eaa3fa3b"), "name" : "james5", "age" : 21 }
	{ "_id" : ObjectId("51f0e5f54d50ac57eaa3fa3c"), "name" : "james6", "age" : 22 }
	{ "_id" : ObjectId("51f0e5f54d50ac57eaa3fa3d"), "name" : "james7", "age" : 23 }
	{ "_id" : ObjectId("51f0e5f54d50ac57eaa3fa3e"), "name" : "james8", "age" : 24 }
	{ "_id" : ObjectId("51f0e5f54d50ac57eaa3fa3f"), "name" : "james9", "age" : 25 }
	{ "_id" : ObjectId("51f0e5f54d50ac57eaa3fa40"), "name" : "james10", "age" : 26 }
	{ "_id" : ObjectId("51f0e5f54d50ac57eaa3fa41"), "name" : "james11", "age" : 27 }
	{ "_id" : ObjectId("51f0e5f54d50ac57eaa3fa42"), "name" : "james12", "age" : 28 }
	{ "_id" : ObjectId("51f0e5f54d50ac57eaa3fa43"), "name" : "james13", "age" : 29 }
	{ "_id" : ObjectId("51f0e5f54d50ac57eaa3fa44"), "name" : "james14", "age" : 30 }
	{ "_id" : ObjectId("51f0e5f54d50ac57eaa3fa45"), "name" : "james15", "age" : 31 }
	{ "_id" : ObjectId("51f0e5f54d50ac57eaa3fa46"), "name" : "james16", "age" : 32 }
	{ "_id" : ObjectId("51f0e5f54d50ac57eaa3fa47"), "name" : "james17", "age" : 33 }
	{ "_id" : ObjectId("51f0e5f54d50ac57eaa3fa48"), "name" : "james18", "age" : 34 }
	Type "it" for more
当你查询一个集合时，MongoDB返回一个游标对象，这个对象包含查询的结果。`mongo shell`迭代显示这些结果，它并不是一次返回所有的结果，它一次显示前20条，正如上列所述，然后等待显示剩余结果的请求。在`mongo shell`中通过`it`命令来显示下一文档集合。

**条件查询**

	> db.person.find({name:"james8"})
	{ "_id" : ObjectId("51f0e5f54d50ac57eaa3fa3e"), "name" : "james8", "age" : 24 }
    > db.person.find({age:{$lte:20}})
    { "_id" : ObjectId("51f0e5f54d50ac57eaa3fa36"), "name" : "james0", "age" : 16 }
    { "_id" : ObjectId("51f0e5f54d50ac57eaa3fa37"), "name" : "james1", "age" : 17 }
    { "_id" : ObjectId("51f0e5f54d50ac57eaa3fa38"), "name" : "james2", "age" : 18 }
    { "_id" : ObjectId("51f0e5f54d50ac57eaa3fa39"), "name" : "james3", "age" : 19 }
    { "_id" : ObjectId("51f0e5f54d50ac57eaa3fa3a"), "name" : "james4", "age" : 20 }
* $lt,$lte,$gt,$gte分别对应<,<=,>,>=
* $in,$or,$not分别对应关系型数据库中的in，or，not
