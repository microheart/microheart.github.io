---
layout: post
title: Linux core dump调试案例
category: Linux
tags: Linux
keywords: 
description: 
---

## 简介
核心文件（core file），也称核心转储（core dump），是操作系统在进程收到某些信号而终止运行时，将此时进程地址空间的内容以及有关进程状态的其他信息写出的一个磁盘文件。这种信息往往用于调试。
核心文件一词来源于磁芯内存（core memory）。来源于[维基百科](http://zh.wikipedia.org/zh/%E6%A0%B8%E5%BF%83%E6%96%87%E4%BB%B6)

## 案例
	#include <stdio.h>

	static void err_core(void);

	int main(void)
	{
	     err_core();
	     return 0;
	}

	static void err_core(void)
	{
	     int *p = NULL;
	     /* derefernce a null pointer, expect core dump. */
	     printf("%d", *p);
	}

程序因为空指针异常，执行到`printf("%d", *p);`将出错，在产生core dump文件之前需对系统参数进行设置。

### 查看机器[Optional]
	$ uname -a
	Linux xandy-vm 3.2.0-23-generic #36-Ubuntu SMP Tue Apr 10 20:39:51 UTC 2012 x86_64 x86_64 x86_64 GNU/Linux

### core文件大小设置
	$ ulimit -c unlimited
	表示core文件大小不收限制。 `ulimit -c filesize`，filesize的单位为kbyte。`ulimit -c`查看core文件的生成开关。为0，则表示关闭了此功能，不会生成core文件。


### core文件的名称[Optional]
	$ sudo sh -c "echo 1 > /proc/sys/kernel/core_uses_pid"
	表示生成的文件名形式为core.pid，默认为core
	使用$ sudo echo "1" > /proc/sys/kernel/core_uses_pid可能会提示Permission denied。

### core文件生成路径[Optional]
	$ sudo sh -c "echo ~/corefile/core-%e-%p-%t > /proc/sys/kernel/core_pattern"`  
	将core文件统一生成到~/corefile目录下，产生的文件名为core-命令名-pid-时间戳，默认在当前目录。
   
### 编译&运行
	$ gcc -Wall -o test_core_dump -g core_dump.c
	$ ./test_core_dump
	Segmentation fault (core dumped)
	

### core dump调试
	$ gdb -c ~/corefile/core-test_core_dump-4109-1402463675 ./test_core_dump  
	(gdb)bt
	#0  0x0000000000400518 in err_core () at core_dump.c:15
	#1  0x00000000004004fd in main () at core_dump.c:7
	gdb -c core.pid program_name调试core dump文件。


## 附

### 正常程序core dump
	$ kill -s SIGSEGV pid


### 设置proc/sys/kernel/core_pattern 参数列表:
* %p – insert pid into filename 添加pid
* %u – insert current uid into filename 添加当前uid
* %g – insert current gid into filename 添加当前gid
* %s – insert signal that caused the coredump into the filename 添加导致产生core的信号
* %t – insert UNIX time that the coredump occurred into filename 添加core文件生成时的unix时间
* %h – insert hostname where the coredump happened into filename 添加主机名
* %e – insert coredumping executable name into filename 添加命令名

### ulimit -- 用户资源限制命令
ulimit用于shell启动进程所占用的资源.  
语法格式 :ulimit [-acdfHlmnpsStvw] [size]  

* -a     列出所有当前资源极限。
* -c     以 512 字节块为单位，指定核心转储的大小。
* -d     以 K 字节为单位指定数据区域的大小。
* -f     使用 Limit 参数时设定文件大小极限（以块计），或者在未指定参数时报告文件大小极限。缺省值为 -f 标志。
* -H     指定设置某个给定资源的硬极限。如果用户拥有 root 用户权限，可以增大硬极限。任何用户均可减少硬极限。
* -m     以 K 字节为单位指定物理存储器的大小。
* -n     指定一个进程可以拥有的文件描述符的数量的极限。
* -s     以 K 字节为单位指定堆栈的大小。
* -S     指定为给定的资源设置软极限。软极限可增大到硬极限的值。如果 -H 和 -S 标志均未指定，极限适用于以上二者。
* -t     指定每个进程所使用的秒数 。