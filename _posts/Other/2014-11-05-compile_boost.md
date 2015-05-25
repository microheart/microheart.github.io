---
layout: post
title: 编译BOOST1.55.0
category: Other
tags: [C++,Boost]
keywords: 
description: 
---

## 简介
项目中需要用到Boost。

## 下载
* http://www.boost.org/users/download/

## 编译
* 解压至d:\dev\c++\ (可为任意目录)
* 打开VS2012 x64 Native Tools Command Prompt终端
* 进入解压后的boost_1_55_0目录，
* 执行bootstrap.bat
    $ d:\dev\c++\boost_1_55_0>bootstrap.bat
	Building Boost.Build engine

	Bootstrapping is done. To build, run:

	    .\b2

	To adjust configuration, edit 'project-config.jam'.
	Further information:

	    - Command line help:
	    .\b2 --help

	    - Getting started guide:
	    http://boost.org/more/getting_started/windows.html

	    - Boost.Build documentation:
	    http://www.boost.org/boost-build2/doc/html/index.html

* 运行b2
	d:\dev\c++\boost_1_55_0>.\b2