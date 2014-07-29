---
layout: post
title: Hadoop HDFS 漫画
category: Hadoop
tags: HDFS
keywords: HDFS
description: 
---

## 简介
从网上找到关于HDFS的漫画，感觉挺好玩，对HDFS的基本流程和容错机制进行了介绍，特从PDF中截图上传，找不到文件原始出处，在此对作者Maneesh Varshney表示感谢。

## HDFS角色登场

![](/public/upload/hadoop/HDFS_Role.jpg)

HDFS有三类角色：名字节点、数据节点和客户端。名字节点和数据节点为主从关系，一个名字节点管理多个数据节点。

## 往HDFS集群写数据

![](/public/upload/hadoop/HDFS_WriteData1.jpg)

![](/public/upload/hadoop/HDFS_WriteData2.jpg)

![](/public/upload/hadoop/HDFS_WriteData3.jpg)


## 从HDFS集群中读数据

![](/public/upload/hadoop/HDFS_ReadData.jpg)

## HDFS容错场景

![](/public/upload/hadoop/HDFS_Fault_Tolerance_OW.jpg)

![](/public/upload/hadoop/HDFS_Fault_Tolerance1.jpg)

![](/public/upload/hadoop/HDFS_Fault_Tolerance2.jpg)

## 读写数据容错

![](/public/upload/hadoop/HDFS_Fault_Tolerance_RW.jpg)

## 数据节点容错

![](/public/upload/hadoop/HDFS_Fault_Tolerance_DN.jpg)

## HDFS副本放置策略

![](/public/upload/hadoop/HDFS_Replica.jpg)

## JustDoIt

![](/public/upload/hadoop/HDFS_JustDoIt.jpg)