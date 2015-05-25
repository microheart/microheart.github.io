---
layout: post
title: MySQL主从复制
category: Java
tags: MySQL
keywords: 
description: 
---

## 简介
阅读《大型网站技术架构》书籍，试着搭建数据库的主从架构。

## Master设置
编辑mysql配置文件my.cnf或者my.ini(Windows)

	[mysqld]
	log-bin=mysql-bin
	server-id=1

server-id要唯一，范围从1到2^^32-1，如果server-id没有设置，master则会拒绝slave的所有连接。
log-bin必须设置，日志文件名字的前缀。

配置完成后，需要重启server。

## slave设置

	[mysqld]
	server-id=2
server-id要唯一，不能与master或者其他slave相同，如果server-id没有设置，则不会去连接master.
在slave上不一定要enable 二进制日志(binary logging)，如果设置了log-bin，则可以用于数据备份和灾难恢复。

## 为同步(Replication)创建一个账号
每个slave必须使用mysql账号连接master。赋予这个账号REPLICATION_SLAVE权限。可以为每个slave创建一个账号，也可以多个slave共用一个账号。

这个账号的信息将以明文的形式保存在slave数据库中，为了安全起见，最好是创建一个单独的账号，专门用于同步。
创建一个用户名为repl，密码为slavepass的账号，赋予REPLICATION权限。可以从任何mydomain.com的主机上连接master。

	mysql> CREATE USER 'repl'@'%.mydomain.com' IDENTIFIED BY 'slavepass';
	mysql> GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%.mydomain.com';

## 获取master二进制日志的协调点
获取协调点是为了决定slave将从master哪个位置进行事件读取。
如果已经有数据在master中，要在replication之前同步数据。需要停止在master继续执行语言。

给master加上读锁

	mysql> FLUSH TABLES WITH READ LOCK;

获取当前二进制文件名和位置

	mysql > SHOW MASTER STATUS;
	+------------------+----------+--------------+------------------+
	| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
	+------------------+----------+--------------+------------------+
	| mysql-bin.000003 | 73       | test         | manual,mysql     |
	+------------------+----------+--------------+------------------+

## 释放master中的读锁
	mysql> UNLOCK TABLES;

## 在slave上配置master信息
为了slave能够和master通信，需要将master用于同步的账号告诉slave。

	mysql> CHANGE MASTER TO
	    ->     MASTER_HOST='master_host_name',
	    ->     MASTER_USER='replication_user_name',
	    ->     MASTER_PASSWORD='replication_password',
	    ->     MASTER_LOG_FILE='recorded_log_file_name',
	    ->     MASTER_LOG_POS=recorded_log_position;

你可以用SHOW SLAVE STATUS语句查看slave的设置是否正确：

	mysql> SHOW SLAVE STATUS \G

## 开始slave线程
	mysql> START SLAVE;

当slave在同步时，将生成master.info和relay-log.info两个文件，用于跟踪master的bin-log.

## 参考资料
> http://blog.csdn.net/qmhball/article/details/8233769
> http://blog.csdn.net/hguisu/article/details/7325124
> http://www.cnblogs.com/Ihaveadream/p/4136373.html
> https://www.centos.bz/2012/05/amoeba-for-mysql/
> http://dev.mysql.com/doc/refman/5.5/en/replication-howto.html
