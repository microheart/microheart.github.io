---
layout: post
title: Hadoop 集群OS环境配置
category: Hadoop
tags: Hadoop
keywords: 
description: 
---


## 创建相同的用户(三台主机)
	
	# useradd hadoop
	# passwd hadoop

## 确保主机之间建立信任关系，实现无密码登陆（三台主机）

生成秘钥并配置SSH无密码登陆主机
	
	$ ssh-keygen -t rsa
	$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys

将文件拷贝到另外两台主机

	$ scp ~/.ssh/authorized_keys slave1:~/.ssh/
	$ scp ~/.ssh/authorized_keys slave2:~/.ssh/

	三台主机上执行
	$ chmod 600 authorized_keys

验证
	
	$ ssh salve1
	$ ssh salve2
