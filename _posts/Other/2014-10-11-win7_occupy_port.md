---
layout: post
title: win7下查看被占用端口
category: Other
tags: 
keywords: 
description: 
---

今天启动Jetty时，发现端口80被占用，经查看原来是Apache占用了。

步骤：

	$ netstat -aon|findstr 80 
	TCP    0.0.0.0:80       0.0.0.0:0    LISTENING    9092
	TCP    0.0.0.0:8081     0.0.0.0:0    LISTENING    2400
	...

	$ tasklist|findstr 9092
	httpd.exe    9092 Services   0     19,536 K

将其关闭