---
layout: post
title: Mac下编译OpenSSL
category: Linux
tags: OpenSSL
keywords: 
description: 
---


## 简介
这两年来，OpenSSL的漏洞越来越多，如CVE-2014-0160，CVE-2015-0289。
在公司所在部门开发的更新模块基于OpenSSL，每出现OpenSSL漏洞，都需评估其影响，是否要出Hotfix.
更新模块覆盖了绝大多数平台，包含：Windows, Linux, Mac, Android, AIX, FreeBSD等
公司所有产品都用到此模块，此模块静态链接OpenSSL。

## 编译OpenSSL
从[http://www.openssl.org](http://www.openssl.org/)下载OpenSSL，Mac中用到的为OpenSSL 0.9.8
最新的版本为OpenSSL 0.9.8zf

编译不同平台下的版本，需要配置。32位的对应`darwin-i386-cc`, 64位对应`darwin64-x86-64-cc`

### i386版本编译

	# tar zxvf openssl-0.9.8zf.tar.gz
	# cd openssl-0.9.8zf
	# ./Configure darwin-i386-cc
	# make


执行`./Configure darwin-i386-cc`生成的部分log，其中的CFLAG中包含了`-arch i386`：

	Configuring for darwin-i386-cc
	    no-camellia     [default]  OPENSSL_NO_CAMELLIA (skip dir)
	 	...
	CC            =cc
	CFLAG         =-DOPENSSL_THREADS -D_REENTRANT -DDSO_DLFCN -DHAVE_DLFCN_H -arch i386 -O3 -fomit-frame-pointer -DL_ENDIAN
	...
	making links in crypto...
	crypto.h => ../include/openssl/crypto.h
	...
	make[1]: Nothing to be done for `generate'.

	Configured for darwin-i386-cc.


### x86-64版本编译
先清理编译i386生成的文件

	# make clean
	# cd openssl-0.9.8zf
	# ./Configure darwin64-x86-64-cc
	# make


执行`./Configure darwin64-x86-64-cc`生成的部分log，其中的CFLAG中包含了`-arch x86_64`：

	Configuring for darwin64-x86_64-cc
	    no-camellia     [default]  OPENSSL_NO_CAMELLIA (skip dir)
	   	...
	IsMK1MF=0
	CC            =cc
	CFLAG         =-DOPENSSL_THREADS -D_REENTRANT -DDSO_DLFCN -DHAVE_DLFCN_H -arch x86_64 -O3 -fomit-frame-pointer -DL_ENDIAN -DMD32_REG_T=int -Wall
	...
	Configured for darwin64-x86_64-cc.

生成libssl.a和libcrypto.a

可用lipo命令查看编译后生成的类型

	# lipo -info libssl.a
	input file libssl.a is not a fat file
	Non-fat file: libssl.a is architecture: x86_64