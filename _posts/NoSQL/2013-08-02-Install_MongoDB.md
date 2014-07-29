---
layout: post
title: Win7下安装MongoDB
category: NoSQL
tags: [NoSQL,安装]
keywords: MongodDB
description: 
---

##MongoDB安装流程

1 **MongoDB简介**

[MongoDB](http://www.mongodb.org)是一个基于分布式文件存储的数据库，由C++语言编写，旨在为WEB应用提供可扩展的高性能数据存储解决方案。

2 **下载**

从[MongoDB主页](http://www.mongodb.org/downloads)选择合适的版本，如**mongodb-win32-x86_64-2.4.5.zip**

解压到D盘(可为任意目录)， 则路径为`D:\mongodb-win32-x86_64-2.4.5`，重命名mongodb-win32-x86_64-2.4.5为mongodb，则路径为`D:\mongodb`，将`D:\mongodb\bin`添加到path。

3 **创建目录**

在`d:\mongodb`下创建`data\db`二级目录,也可以直接新建文件夹等操作完成。

    >d:
    >cd d:\mongodb
    >md data
    >md data\db   

4 **测试**

    >mongod.exe –dbpath d:\mongodb\data
启动mongod服务

重新打开另一个命令行窗口

    >mongo.exe
    >db.test.save({name: 'jack' })
    >db.test.find()
	{ "_id" : ObjectId("51ee49fbd7c7ce2423c66f33"), "name" : "jack" }
    
##安装MongoDB服务
1 **创建mongod.cfg**

在mongodb目录(d:\mongodb)下创建mongod.cfg文件，内容如下：

    logpath= D:\mongodb\log\mongo.log 
    logappend=true
    dbpath=D:\mongodb\data\db
    directoryperdb=true

2 **安装服务**

**Windows Start->All Programs->Accessories->Command Prompt->Run as Administrator(right click)**

打开具有管理员权限的命令行窗口，安装服务需要管理员权限，

    >mongod.exe --config  d:\mongodb\mongod.cfg –-install
**注**：需要指定mongod.cfg的全路径，若配置错误，可通过**sc delete MongoDB**命令删除服务

运行MongoDB服务

    >net start MongoDB

3 **停止或者移除MongoDB服务**

停止MongoDB 服务:

    >net stop MongoDB
    
删除MongoDB 服务:

    >mongod.exe --remove
