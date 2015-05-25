---
layout: post
title: CentOS使用Eclipse开发Hadoop应用程序 
category: Hadoop
tags: Hadoop
keywords: 
description: 
---

## 简介
在CentOS6.5上开发Hadoop应用程序，例子采用Hadoop提供的WordCount。

## 启动Hadoop
本机Hadoop目录：~/Hadoop/hadoop1.2.1
	
	$ cd ~/Hadoop/hadoop1.2.1
	$ ./bin/start-all.sh

如果NameNode没启动成功，则可能是因为${hadoop.tmp.dir}目录的内容被清除了。
hadoop.tmp.dir是hadoop文件系统依赖的基础配置，很多路径都依赖它。它默认的位置是在/tmp/{$user}下面，但是在/tmp路径下的存储是不安全的，因为linux一次重启，文件就可能被删除。

建议重新配置hadoop.tmp.dir，编辑${HADOOP_HOME}/conf/core-site.xml，目录的位置可自行定义。

	<property>
        <name>hadoop.tmp.dir</name>
        <value>/home/xandy/Hadoop/hadoop-1.2.1/tmp</value>
    </property>

则需重新格式化名字节点，然后启动：

	$ cd ~/Hadoop/hadoop1.2.1
	$./bin/hadoop namenode -format
	$ ./bin/start-all.sh

## 新建WordCount项目

Eclipse中选择`File -> New -> Other -> Map/Reduce Project`新建名为WordCount的Map/Reduce项目。

创建`net.cocloud.hadoop`包，将${HADOOP_HOME}/src/examples/org/apache/hadoop/examples/WordCount.java复制到`net.cocloud.hadoop`下面。

在Map/Reduce视图中的Map/Reduce Locations中New Hadoop Location。填写相应的参数：

* Location name:XandyHadoop
* Map/Reduce Master Host:localhost; Port:9001
* DFS Master Host:localhost; Port：9000
* User name:xandy

## 准备数据

	$ vi~/wordcount.txt 

将Hadoop简介的一段文字填入其中。

	what Is Apache Hadoop?
	The Apache™ Hadoop® project develops open-source software for reliable, scalable, distributed computing.

	The Apache Hadoop software library is a framework that allows for the distributed processing of large data sets across clusters of computers using simple programming models. It is designed to scale up from single servers to thousands of machines, each offering local computation and storage. Rather than rely on hardware to deliver high-availability, the library itself is designed to detect and handle failures at the application layer, so delivering a highly-available service on top of a cluster of computers, each of which may be prone to failures.


将wordcount.txt复制到hdfs/input目录下面

	./bin/hadoop fs -mkdir input
	./bin/hadoop fs -put ~/wordcount.txt input

在DFS Locations下面可以看到/usr/xandy/input/wordcount.txt文件

## 运行程序

WordCount项目右键，`Run as -> Run Configurations`
	
	Main:
		Project: WordCount
		Main Class: net.cocloud.hadoop.WordCount
	Arguments:
		Program arguments: hdfs://localhost:9000/user/xandy/input/wordcount.txt hdfs://localhost:9000/user/xandy/output/

点击Run按钮，得到相应的输出：

	14/08/24 01:59:33 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
	14/08/24 01:59:33 WARN mapred.JobClient: No job jar file set.  User classes may not be found. See JobConf(Class) or JobConf#setJar(String).
	14/08/24 01:59:33 INFO input.FileInputFormat: Total input paths to process : 1
	14/08/24 01:59:33 WARN snappy.LoadSnappy: Snappy native library not loaded
	14/08/24 01:59:33 INFO mapred.JobClient: Running job: job_local1283277468_0001
	14/08/24 01:59:33 INFO mapred.LocalJobRunner: Waiting for map tasks
	14/08/24 01:59:33 INFO mapred.LocalJobRunner: Starting task: attempt_local1283277468_0001_m_000000_0
	14/08/24 01:59:33 INFO util.ProcessTree: setsid exited with exit code 0
	...
	14/08/24 01:59:34 INFO mapred.JobClient:     Map output records=103
	14/08/24 01:59:34 INFO mapred.JobClient:     Combine input records=103
	14/08/24 01:59:34 INFO mapred.JobClient:     Reduce input records=78

从DFS Locations中查看WordCount项目生成的输出文件`/user/xandy/output/part-r-00000`内容：

	Apache	2
	Apache™	1
	Hadoop	1
	Hadoop?	1
	Hadoop®	1
	...
	the	3
	thousands	1
	to	5
	top	1
	up	1
	using	1
	what	1
	which	1


## Eclipse调试

* F5-Step Into：表示进入当前方法。
* F6-Step Over：表示运行下一行代码。如果当前行有方法调用，这个方法将被执行完毕返回，然后到下一行。
* F7-Step Return：表示退出当前方法，返回到调用层。
* F8-表示当前实现继续运行直到下一个断点。