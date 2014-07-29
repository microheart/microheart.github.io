---
layout: post
title: 协议格式
category: Redis
tags: [Redis,协议]
keywords: 协议
description: 
---

## 协议格式简单介绍

* *参数的个数 CRLF
* $第一个参数的长度CRLF
* 第一个参数CRLF
* ...
* $第N个参数的长度CRLF
* 第N个参数CRLF
* 例如在redis_cli里键入get a，经过协议组装后的请求为

2\r\n$3\r\nget\r\n$1\r\na\r\n



