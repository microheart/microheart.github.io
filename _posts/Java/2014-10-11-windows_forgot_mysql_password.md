---
layout: post
title: windows环境中mysql忘记root密码的解决办法
category: Java
tags: MySQL
keywords: 
description: 
---

最近对一个轻量级的JFinal比较感兴趣，想使用它做一个小网站，由于很久没有使用电脑上的MySQL，忘记了密码。查了一些资料，将解决过程记下来。

## 平台
* Windows 7
* MySQL 5.1
* 安装路径："D:\Program Files (x86)\MySQL\MySQL Server 5.1"

## 步骤

### 停止mysql服务

	$ net stop mysql

### 跳过权限安全检查

	$ cd D:\Program Files (x86)\MySQL\MySQL Server 5.1\bin
	$ mysqld --defaults-file="D:\Program Files (x86)\MySQL\MySQL Server 5.1\my.ini" --console --skip-grant-tables

该命令通过跳过权限安全检查，开启mysql服务，当连接mysql时，可以不用输入用户密码。保留窗口，不要关闭。

### 修改密码

打开第二个cmd窗口

	$ mysql -u root -p
	$ Enter password:

直接回车，不需要输入密码，即可登录。

	$ show databases;
	$ use mysql;
	$ UPDATE user SET Password=PASSWORD('123456') where USER='root';
	$ FLUSH PRIVILEGES;
	$ quit

### 测试
关闭第一个cmd窗口，即开启mysql服务的窗口

	$ net start mysql
	$ mysql -u root -p
	$ Enter password: ******

Okay.