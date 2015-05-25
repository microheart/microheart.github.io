---
layout: post
title: Windows编译OpenSSL
category: Linux
tags: OpenSSL
keywords: 
description: 
---


##软件准备

ActivePerl
* 从[http://www.activestate.com/activeperl/downloads](http://www.activestate.com/activeperl/downloads)下载符合本地平台的ActivePerl，并安装，并将安装后bin路径加入系统环境变量path中。

NASM
* 从[http://sourceforge.net/projects/nasm/](http://sourceforge.net/projects/nasm/)下载NASM，并安装，并将安装路径加入系统环境变量path中。


* 从[https://www.openssl.org/source/](https://www.openssl.org/source/)下载最新版本的OpenSSL，当前1.0.1最新版本为1.0.1m.

* 安装了Visual Studio，如Visual Studio 2008.

## 编译

**编译32位的OpenSSL**

	从VS中进入Visual Studio 2008 Command Prompt, 2010， 2012也一样。
	Start->All Programs->Microsoft Visual Studio 2008->Visual Studio Tools->Visual Studio 2008 Command Prompt
	这样的话，就不需要配置vs的环境变量了，可以直接使用nmake等命令。
	>cd D:\dev\OpenSSL\openssl-1.0.1m
	>perl  Configure  VC-WIN32 
	Configuring for VC-WIN32
	...
	IsMK1MF=1
	CC            =cl
	CFLAG         =-DOPENSSL_THREADS  -DDSO_WIN32 -W3 -Gs0 -GF -Gy -nologo -DOPENSSL_SYSNAME_WIN32 -DWIN32_LEAN_AND_MEAN -DL_ENDIAN -D_CRT_SECURE_NO_DEPRECATE -DOPENSSL_BN_ASM_PART_WORDS -DOPENSSL_IA32_SSE2 -DOPENSSL_BN_ASM_MONT -DOPENSSL_BN_ASM_GF2m -DSHA1_ASM -DSHA256_ASM -DSHA512_ASM -DMD5_ASM -DRMD160_ASM -DAES_ASM -DVPAES_ASM -DWHIRLPOOL_ASM -DGHASH_ASM
	...

	Configured for VC-WIN32.
	>ms\do_ms
	>nmake  –f  ms\nt.mak
	在out32dll中生成静态库、动态库和.exe文件，如libeay32.lib，ssleay32.lib nt.mak生成静态库
	>nmake  -f  ms\ntdll.mak test
	1 handshakes of 256 bytes done
	passed all tests
	完成测试

**编译64位的OpenSSL**

	进入VS的64位命令行界面
	Start->All Programs->Microsoft Visual Studio 2008->Visual Studio Tools->Visual Studio 2008 x64 Win64 Command Prompt
	>perl Configure VC-WIN64A
	Configuring for VC-WIN64A
	    no-ec_nistp_64_gcc_128 [default]  OPENSSL_NO_EC_NISTP_64_GCC_128 (skip dir)
	...
	CC            =cl
	CFLAG         =-DOPENSSL_THREADS  -DDSO_WIN32 -W3 -Gs0 -Gy -nologo -DOPENSSL_SYS
	NAME_WIN32 -DWIN32_LEAN_AND_MEAN -DL_ENDIAN -DUNICODE -D_UNICODE -D_CRT_SECURE_N
	O_DEPRECATE -DOPENSSL_IA32_SSE2 -DOPENSSL_BN_ASM_MONT -DOPENSSL_BN_ASM_MONT5 -DO
	PENSSL_BN_ASM_GF2m -DSHA1_ASM -DSHA256_ASM -DSHA512_ASM -DMD5_ASM -DAES_ASM -DVP
	AES_ASM -DBSAES_ASM -DWHIRLPOOL_ASM -DGHASH_ASM
	Configured for VC-WIN64A.

	>ms\do_nasm
	>ms\do_win64a 
	>nmake  -f  ms\nt.mak
	>nmake  -f  ms\nt.mak test

