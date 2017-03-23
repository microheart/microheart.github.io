---
layout: post
title: Memcached安装与调试
category: Memcached
tags: Memcached
keywords:
description:
---


## 安装Memcached

	// 安装libevent
	$ wget https://github.com/downloads/libevent/libevent/libevent-1.4.14b-stable.tar.gz
	$ tar zxvf libevent-1.4.14b-stable.tar.gz
	$ cd libevent-1.4.14b-stable
	$./configure --prefix=/usr/local
	$ make
	# make install

	// 安装memcached
	$wget http://Memcached.googlecode.com/files/Memcached-1.4.22.tar.gz
	$ tar zxvf Memcached-1.4.22.tar.gz
	$ cd Memcached-1.4.22
	$ ./configure --with-libevent=/usr/local
	$ make
	# make install


## 设置代理
yum 代理

	# vi /etc/profile
	  在文件末尾添加
	  proxy=http://trend\xandy_xu:shun#1211@cnnjproxy-ha.tw.trendnet.org:8080

## 安装telnet

	#yum install telnet-server
	#yum install telnet

## 测试
	$ /usr/local/bin/memcached -d -vvv
	$ telnet localhost 11211

	[root@nj-grey-guo xandy]# telnet localhost 11211
	Trying ::1...
	Connected to localhost.
	Escape character is '^]'.


注：ping和ssh都可以连得上，若telnet连接memcached server连接不上，则需要将防火墙关闭。
网上找了好久的资料，终于将这个问题解决了。

	# iptables -F

### 存储命令

命令格式： &lt;command name&gt; &lt;key&gt; &lt;flags&gt; &lt;exptime&gt; &lt;bytes&gt; &lt;data block&gt;
命令解释:
&lt;command name&gt;  set/add/replace
&lt;key&gt;  查找关键字
&lt;flags&gt; 客户机使用它存储关于键值对的额外信息
&lt;exptime&gt; 该数据的存活时间,0为永远
&lt;bytes&gt; 存储字节数
&lt;data block&gt; 存储的数据块

	set server 0 0 9
	localhost
	STORED
	add port 0 0 4
	8080
	STORED
	get server
	VALUE server 0 9
	localhost
	END
	get port
	VALUE port 0 4
	8080
	END
	replace server 0 0 11
	10.64.40.12
	STORED
	get server port
	VALUE server 0 11
	10.64.40.12
	VALUE port 0 4
	8080
	END

set 无论如何都进行存储
add 只有数据不存在时进行添加
repalce 只有数据存在时进行替换


### 读取命令
get &lt;key&gt;
&lt;key&gt;可以表示一个或多个键值，由空格隔开的字串

### 删除命令
delete &lt;key&gt;
删除键值为key的数据。

	delete port
	DELETED

### 状态命令
stats

	stat
	STAT pid 2769
	STAT uptime 3347
	STAT time 1423311919
	STAT version 1.4.22
	STAT libevent 1.4.14b-stable
	STAT pointer_size 64
	STAT rusage_user 0.068989
	STAT rusage_system 0.109983
	STAT curr_connections 11
	STAT total_connections 15
	STAT connection_structures 12
	STAT reserved_fds 20
	STAT cmd_get 26
	STAT cmd_set 11
	STAT cmd_flush 0
	STAT cmd_touch 0
	STAT get_hits 22
	STAT get_misses 4
	STAT delete_misses 0
	STAT delete_hits 1
	STAT incr_misses 0
	STAT incr_hits 0
	STAT decr_misses 0
	STAT decr_hits 0
	STAT cas_misses 0
	STAT cas_hits 0
	STAT cas_badval 0
	STAT touch_hits 0
	STAT touch_misses 0
	STAT auth_cmds 0
	STAT auth_errors 0
	STAT bytes_read 1016
	STAT bytes_written 1125
	STAT limit_maxbytes 67108864
	STAT accepting_conns 1
	STAT listen_disabled_num 0
	STAT threads 4
	STAT conn_yields 0
	STAT hash_power_level 16
	STAT hash_bytes 524288
	STAT hash_is_expanding 0
	STAT malloc_fails 0
	STAT bytes 155
	STAT curr_items 2
	STAT total_items 8
	STAT expired_unfetched 0
	STAT evicted_unfetched 0
	STAT evictions 0
	STAT reclaimed 0
	STAT crawler_reclaimed 0
	STAT lrutail_reflocked 0
	END


## debug Memecached

	$ cd Memcached-1.4.22
	#make uninstall
	$ ./configure --with-libevent=/usr/local
	$make  CFLAGS="-g -O0"
    $make install


## Memcached对每一个存取操作都是加锁的。

	(gdb) i breakpoint
	Num     Type           Disp Enb Address            What
	1       breakpoint     keep y   0x000000000040f478 in main at memcached.c:5027
	        breakpoint already hit 1 time
	2       breakpoint     keep y   0x000000000041376c in do_item_alloc at items.c:95
	        breakpoint already hit 5 times
	3       breakpoint     keep y   0x000000000041418d in do_item_link at items.c:317
	        breakpoint already hit 5 times
	4       breakpoint     keep y   0x000000000041458c in do_item_update at items.c:395
	        breakpoint already hit 1 time
	5       breakpoint     keep y   0x0000000000414600 in do_item_replace at items.c:413
	        breakpoint already hit 2 times
	6       breakpoint     keep y   0x00000000004153e8 in do_item_get at items.c:576
	        breakpoint already hit 7 times
	7       breakpoint     keep y   0x0000000000416453 in assoc_init at assoc.c:63
	        breakpoint already hit 1 time
	8       breakpoint     keep y   0x0000000000416514 in assoc_find at assoc.c:81
	        breakpoint already hit 7 times
	9       breakpoint     keep y   0x0000000000416879 in assoc_insert at assoc.c:160
	        breakpoint already hit 5 times
	10      breakpoint     keep y   0x00000000004169be in assoc_delete at assoc.c:180
	        breakpoint already hit 2 times
	11      breakpoint     keep y   0x0000000000417327 in create_worker at thread.c:307
	        breakpoint already hit 4 times
	12      breakpoint     keep y   0x000000000041739e in accept_new_conns at thread.c:320
	13      breakpoint     keep y   0x00000000004173cc in setup_thread at thread.c:330
	        breakpoint already hit 4 times
	14      breakpoint     keep y   0x00000000004175dc in thread_libevent_process at thread.c:388
	        breakpoint already hit 9 times
	15      breakpoint     keep y   0x00000000004178e3 in item_get at thread.c:493
	        breakpoint already hit 2 times
	16      breakpoint     keep y   0x0000000000417b54 in item_update at thread.c:562
	        breakpoint already hit 1 time
	17      breakpoint     keep y   0x0000000000417d61 in item_stats at thread.c:625
	18      breakpoint     keep y   0x0000000000418ade in memcached_thread_init at thread.c:772
	        breakpoint already hit 1 time
	19      breakpoint     keep y   0x0000000000411401 in slabs_init at slabs.c:96
	        breakpoint already hit 1 time
	20      breakpoint     keep y   0x0000000000411877 in do_slabs_newslab at slabs.c:196
	        breakpoint already hit 1 time


	(gdb) bt
	#0  process_get_command (c=0x7fffe00261c0, tokens=0x7ffff7d24b50, ntokens=3, return_cas=false) at memcached.c:2877
	#1  0x000000000040bc55 in process_command (c=0x7fffe00261c0, command=0x7fffe00263d0 "get") at memcached.c:3433
	#2  0x000000000040cba6 in try_read_command (c=0x7fffe00261c0) at memcached.c:3768
	#3  0x000000000040d8b9 in drive_machine (c=0x7fffe00261c0) at memcached.c:4113
	#4  0x000000000040e427 in event_handler (fd=40, which=2, arg=0x7fffe00261c0) at memcached.c:4358
	#5  0x00007ffff7de8c29 in event_process_active (base=0x63c870, flags=&lt;value optimized out&gt;) at event.c:395
	#6  event_base_loop (base=0x63c870, flags=&lt;value optimized out&gt;) at event.c:547
	#7  0x00000000004175bf in worker_libevent (arg=0x62fa60) at thread.c:378
	#8  0x0000003bce0079d1 in start_thread () from /lib64/libpthread.so.0
	#9  0x0000003bcdce89dd in clone () from /lib64/libc.so.6





	(gdb) i threads
	  6 Thread 0x7ffff5521700 (LWP 6074)  0x0000003bce00b5bc in pthread_cond_wait@@GLIBC_2.3.2 ()
	   from /lib64/libpthread.so.0
	  5 Thread 0x7ffff5f22700 (LWP 6073)  0x0000003bcdce8fd3 in epoll_wait () from /lib64/libc.so.6
	  4 Thread 0x7ffff6923700 (LWP 6071)  0x0000003bcdce8fd3 in epoll_wait () from /lib64/libc.so.6
	  3 Thread 0x7ffff7324700 (LWP 6069)  0x0000003bcdce8fd3 in epoll_wait () from /lib64/libc.so.6
	* 2 Thread 0x7ffff7d25700 (LWP 6068)  process_get_command (c=0x7fffe00261c0, tokens=0x7ffff7d24b50, ntokens=3,
	    return_cas=false) at memcached.c:2877
	  1 Thread 0x7ffff7dd1700 (LWP 5794)  0x0000003bcdce8fd3 in epoll_wait () from /lib64/libc.so.6


	set hi 0 0 7
	welcome
	(gdb) bt
	#0  do_item_get (key=0x7ffff4b20e28 "hi", nkey=2, hv=1421556408) at items.c:576
	#1  0x0000000000407e5e in do_store_item (it=0x7ffff4b20df0, comm=2, c=0x7fffe00261c0, hv=1421556408) at memcached.c:2290
	#2  0x0000000000417cd8 in store_item (item=0x7ffff4b20df0, comm=2, c=0x7fffe00261c0) at thread.c:595
	#3  0x000000000040473b in complete_nread_ascii (c=0x7fffe00261c0) at memcached.c:896
	#4  0x0000000000407dda in complete_nread (c=0x7fffe00261c0) at memcached.c:2276
	#5  0x000000000040d9ec in drive_machine (c=0x7fffe00261c0) at memcached.c:4151
	#6  0x000000000040e427 in event_handler (fd=40, which=2, arg=0x7fffe00261c0) at memcached.c:4358
	#7  0x00007ffff7de8c29 in event_process_active (base=0x63c870, flags=&lt;value optimized out&gt;) at event.c:395
	#8  event_base_loop (base=0x63c870, flags=&lt;value optimized out&gt;) at event.c:547
	#9  0x00000000004175bf in worker_libevent (arg=0x62fa60) at thread.c:378
	#10 0x0000003bce0079d1 in start_thread () from /lib64/libpthread.so.0
	#11 0x0000003bcdce89dd in clone () from /lib64/libc.so.6

	(gdb) bt
	#0  assoc_insert (it=0x7ffff4b20df0, hv=1421556408) at assoc.c:166
	#1  0x000000000041427f in do_item_link (it=0x7ffff4b20df0, hv=1421556408) at items.c:329
	#2  0x000000000040854c in do_store_item (it=0x7ffff4b20df0, comm=2, c=0x7fffe00261c0, hv=1421556408) at memcached.c:2385
	#3  0x0000000000417cd8 in store_item (item=0x7ffff4b20df0, comm=2, c=0x7fffe00261c0) at thread.c:595
	#4  0x000000000040473b in complete_nread_ascii (c=0x7fffe00261c0) at memcached.c:896
	#5  0x0000000000407dda in complete_nread (c=0x7fffe00261c0) at memcached.c:2276
	#6  0x000000000040d9ec in drive_machine (c=0x7fffe00261c0) at memcached.c:4151
	#7  0x000000000040e427 in event_handler (fd=40, which=2, arg=0x7fffe00261c0) at memcached.c:4358
	#8  0x00007ffff7de8c29 in event_process_active (base=0x63c870, flags=&lt;value optimized out&gt;) at event.c:395
	#9  event_base_loop (base=0x63c870, flags=&lt;value optimized out&gt;) at event.c:547
	#10 0x00000000004175bf in worker_libevent (arg=0x62fa60) at thread.c:378
	#11 0x0000003bce0079d1 in start_thread () from /lib64/libpthread.so.0
	#12 0x0000003bcdce89dd in clone () from /lib64/libc.so.6

	(gdb)
	930               out_string(c, "STORED");
	(gdb) bt
	#0  complete_nread_ascii (c=0x7fffe00261c0) at memcached.c:930
	#1  0x0000000000407dda in complete_nread (c=0x7fffe00261c0) at memcached.c:2276
	#2  0x000000000040d9ec in drive_machine (c=0x7fffe00261c0) at memcached.c:4151
	#3  0x000000000040e427 in event_handler (fd=40, which=2, arg=0x7fffe00261c0) at memcached.c:4358
	#4  0x00007ffff7de8c29 in event_process_active (base=0x63c870, flags=&lt;value optimized out&gt;) at event.c:395
	#5  event_base_loop (base=0x63c870, flags=&lt;value optimized out&gt;) at event.c:547
	#6  0x00000000004175bf in worker_libevent (arg=0x62fa60) at thread.c:378
	#7  0x0000003bce0079d1 in start_thread () from /lib64/libpthread.so.0
	#8  0x0000003bcdce89dd in clone () from /lib64/libc.so.6


	(gdb) bt
	#0  drive_machine (c=0x7fffe00261c0) at memcached.c:4269
	#1  0x000000000040e427 in event_handler (fd=40, which=2, arg=0x7fffe00261c0) at memcached.c:4358
	#2  0x00007ffff7de8c29 in event_process_active (base=0x63c870, flags=&lt;value optimized out&gt;) at event.c:395
	#3  event_base_loop (base=0x63c870, flags=&lt;value optimized out&gt;) at event.c:547
	#4  0x00000000004175bf in worker_libevent (arg=0x62fa60) at thread.c:378
	#5  0x0000003bce0079d1 in start_thread () from /lib64/libpthread.so.0
	#6  0x0000003bcdce89dd in clone () from /lib64/libc.so.6

	(gdb) i threads
	  6 Thread 0x7ffff5521700 (LWP 6074)  0x0000003bce00b5bc in pthread_cond_wait@@GLIBC_2.3.2 ()
	   from /lib64/libpthread.so.0
	  5 Thread 0x7ffff5f22700 (LWP 6073)  0x0000003bcdce8fd3 in epoll_wait () from /lib64/libc.so.6
	  4 Thread 0x7ffff6923700 (LWP 6071)  0x0000003bcdce8fd3 in epoll_wait () from /lib64/libc.so.6
	  3 Thread 0x7ffff7324700 (LWP 6069)  0x0000003bcdce8fd3 in epoll_wait () from /lib64/libc.so.6
	* 2 Thread 0x7ffff7d25700 (LWP 6068)  drive_machine (c=0x7fffe00261c0) at memcached.c:4269
	  1 Thread 0x7ffff7dd1700 (LWP 5794)  0x0000003bcdce8fd3 in epoll_wait () from /lib64/libc.so.6


	delete hi
	3413            fprintf(stderr, "&lt;%d %s\n", c-&gt;sfd, command);
	(gdb) p command
	$33 = 0x7fffe00263d0 "delete hi"
	(gdb) bt
	#0  process_command (c=0x7fffe00261c0, command=0x7fffe00263d0 "delete hi") at memcached.c:3413
	#1  0x000000000040cba6 in try_read_command (c=0x7fffe00261c0) at memcached.c:3768
	#2  0x000000000040d8b9 in drive_machine (c=0x7fffe00261c0) at memcached.c:4113
	#3  0x000000000040e427 in event_handler (fd=40, which=2, arg=0x7fffe00261c0) at memcached.c:4358
	#4  0x00007ffff7de8c29 in event_process_active (base=0x63c870, flags=&lt;value optimized out&gt;) at event.c:395
	#5  event_base_loop (base=0x63c870, flags=&lt;value optimized out&gt;) at event.c:547
	#6  0x00000000004175bf in worker_libevent (arg=0x62fa60) at thread.c:378
	#7  0x0000003bce0079d1 in start_thread () from /lib64/libpthread.so.0
	#8  0x0000003bcdce89dd in clone () from /lib64/libc.so.6

	(gdb) bt
	#0  process_delete_command (c=0x7fffe00261c0, tokens=0x7ffff7d24b50, ntokens=3) at memcached.c:3324
	#1  0x000000000040beb8 in process_command (c=0x7fffe00261c0, command=0x7fffe00263d0 "delete") at memcached.c:3462
	#2  0x000000000040cba6 in try_read_command (c=0x7fffe00261c0) at memcached.c:3768
	#3  0x000000000040d8b9 in drive_machine (c=0x7fffe00261c0) at memcached.c:4113
	#4  0x000000000040e427 in event_handler (fd=40, which=2, arg=0x7fffe00261c0) at memcached.c:4358
	#5  0x00007ffff7de8c29 in event_process_active (base=0x63c870, flags=&lt;value optimized out&gt;) at event.c:395
	#6  event_base_loop (base=0x63c870, flags=&lt;value optimized out&gt;) at event.c:547
	#7  0x00000000004175bf in worker_libevent (arg=0x62fa60) at thread.c:378
	#8  0x0000003bce0079d1 in start_thread () from /lib64/libpthread.so.0
	#9  0x0000003bcdce89dd in clone () from /lib64/libc.so.6

	(gdb) bt
	#0  do_item_unlink (it=0x7ffff4b20df0, hv=1421556408) at items.c:339
	#1  0x0000000000417b36 in item_unlink (item=0x7ffff4b20df0) at thread.c:553
	#2  0x000000000040b99c in process_delete_command (c=0x7fffe00261c0, tokens=0x7ffff7d24b50, ntokens=3)
	    at memcached.c:3357
	#3  0x000000000040beb8 in process_command (c=0x7fffe00261c0, command=0x7fffe00263d0 "delete") at memcached.c:3462
	#4  0x000000000040cba6 in try_read_command (c=0x7fffe00261c0) at memcached.c:3768
	#5  0x000000000040d8b9 in drive_machine (c=0x7fffe00261c0) at memcached.c:4113
	#6  0x000000000040e427 in event_handler (fd=40, which=2, arg=0x7fffe00261c0) at memcached.c:4358
	#7  0x00007ffff7de8c29 in event_process_active (base=0x63c870, flags=&lt;value optimized out&gt;) at event.c:395
	#8  event_base_loop (base=0x63c870, flags=&lt;value optimized out&gt;) at event.c:547
	#9  0x00000000004175bf in worker_libevent (arg=0x62fa60) at thread.c:378
	#10 0x0000003bce0079d1 in start_thread () from /lib64/libpthread.so.0
	#11 0x0000003bcdce89dd in clone () from /lib64/libc.so.6
	(gdb) p *it
	$45 = {next = 0x7ffff4b20e50, prev = 0x0, h_next = 0x0, time = 1723, exptime = 0, nbytes = 9, refcount = 2,
	  nsuffix = 6 '\006', it_flags = 11 '\v', slabs_clsid = 1 '\001', nkey = 2 '\002', data = 0x7ffff4b20df0}
	(gdb) ptype it
	type = struct _stritem {
	    struct _stritem *next;
	    struct _stritem *prev;
	    struct _stritem *h_next;
	    rel_time_t time;
	    rel_time_t exptime;
	    int nbytes;
	    short unsigned int refcount;
	    uint8_t nsuffix;
	    uint8_t it_flags;
	    uint8_t slabs_clsid;
	    uint8_t nkey;
	    union {
	        uint64_t cas;
	        char end;
	    } data[];
	} *


	Breakpoint 10, assoc_delete (key=0x7ffff4b20e28 "hi", nkey=2, hv=1421556408) at assoc.c:180
	180         item **before = _hashitem_before(key, nkey, hv);
	(gdb) bt
	#0  assoc_delete (key=0x7ffff4b20e28 "hi", nkey=2, hv=1421556408) at assoc.c:180
	#1  0x00000000004143b3 in do_item_unlink (it=0x7ffff4b20df0, hv=1421556408) at items.c:346
	#2  0x0000000000417b36 in item_unlink (item=0x7ffff4b20df0) at thread.c:553
	#3  0x000000000040b99c in process_delete_command (c=0x7fffe00261c0, tokens=0x7ffff7d24b50, ntokens=3)
	    at memcached.c:3357
	#4  0x000000000040beb8 in process_command (c=0x7fffe00261c0, command=0x7fffe00263d0 "delete") at memcached.c:3462
	#5  0x000000000040cba6 in try_read_command (c=0x7fffe00261c0) at memcached.c:3768
	#6  0x000000000040d8b9 in drive_machine (c=0x7fffe00261c0) at memcached.c:4113
	#7  0x000000000040e427 in event_handler (fd=40, which=2, arg=0x7fffe00261c0) at memcached.c:4358
	#8  0x00007ffff7de8c29 in event_process_active (base=0x63c870, flags=&lt;value optimized out&gt;) at event.c:395
	#9  event_base_loop (base=0x63c870, flags=&lt;value optimized out&gt;) at event.c:547
	#10 0x00000000004175bf in worker_libevent (arg=0x62fa60) at thread.c:378
	#11 0x0000003bce0079d1 in start_thread () from /lib64/libpthread.so.0
	#12 0x0000003bcdce89dd in clone () from /lib64/libc.so.6
	(gdb) ptype key
	type = const char *
	(gdb) p key
	$46 = 0x7ffff4b20e28 "hi"


分配空间，然后再读取。

	set comany 0 0 11
	Trend Micro
	(gdb) p *c
	$48 = {sfd = 40, sasl_conn = 0x0, authenticated = false, state = conn_parse_cmd, substate = bin_no_state,
	  last_cmd_time = 4651, event = {ev_next = {tqe_next = 0x0, tqe_prev = 0x7fffe0013560}, ev_active_next = {
	      tqe_next = 0x0, tqe_prev = 0x63d290}, ev_signal_next = {tqe_next = 0x0, tqe_prev = 0x0},
	    min_heap_idx = 4294967295, ev_base = 0x63c870, ev_fd = 40, ev_events = 18, ev_ncalls = 0,
	    ev_pncalls = 0x7ffff7d24e4e, ev_timeout = {tv_sec = 0, tv_usec = 0}, ev_pri = 0,
	    ev_callback = 0x40e3a8 &lt;event_handler&gt;, ev_arg = 0x7fffe00261c0, ev_res = 2, ev_flags = 130}, ev_flags = 18,
	  which = 2, rbuf = 0x7fffe00263d0 "set company 0 0 11", rcurr = 0x7fffe00263d0 "set company 0 0 11", rsize = 2048,
	  rbytes = 20, wbuf = 0x7fffe0026be0 "ERROR\r\n\r\nROR bad command line format\r\n",
	  wcurr = 0x7fffe00296d0 "STAT pid 5794\r\nSTAT uptime 3112\r\nSTAT time 1423478618\r\nSTAT version 1.4.22\r\nSTAT libevent 1.4.14b-stable\r\nSTAT pointer_size 64\r\nSTAT rusage_user 0.017997\r\nSTAT rusage_system 3.215511\r\nSTAT curr_connec"..., wsize = 2048, wbytes = 1106, write_and_go = conn_new_cmd, write_and_free = 0x0, ritem = 0x7ffff4b20e3a "",
	  rlbytes = 0, item = 0x0, sbytes = 0, iov = 0x7fffe0027af0, iovsize = 400, iovused = 4, msglist = 0x7fffe0029400,
	  msgsize = 10, msgused = 1, msgcurr = 1, msgbytes = 37, ilist = 0x7fffe00273f0, isize = 200, icurr = 0x7fffe00273f0,
	  ileft = 0, suffixlist = 0x7fffe0027a40, suffixsize = 20, suffixcurr = 0x7fffe0027a40, suffixleft = 0,
	  protocol = ascii_prot, transport = tcp_transport, request_id = 0, request_addr = {sin6_family = 2, sin6_port = 49642,
	    sin6_flowinfo = 86523914, sin6_addr = {__in6_u = {__u6_addr8 = '\000' &lt;repeats 15 times&gt;, __u6_addr16 = {0, 0, 0,
	          0, 0, 0, 0, 0}, __u6_addr32 = {0, 0, 0, 0}}}, sin6_scope_id = 0}, request_addr_size = 16, hdrbuf = 0x0,
	  hdrsize = 0, noreply = false, stats = {buffer = 0x0, size = 2048, offset = 1106}, binary_header = {request = {
	      magic = 0 '\000', opcode = 0 '\000', keylen = 0, extlen = 0 '\000', datatype = 0 '\000', reserved = 0,
	      bodylen = 0, opaque = 0, cas = 0}, bytes = '\000' &lt;repeats 23 times&gt;}, cas = 7, cmd = -1, opaque = 0, keylen = 0,
	  next = 0x0, thread = 0x62fa60}
	(gdb) bt
	#0  process_command (c=0x7fffe00261c0, command=0x7fffe00263d0 "set company 0 0 11") at memcached.c:3412
	#1  0x000000000040cba6 in try_read_command (c=0x7fffe00261c0) at memcached.c:3768
	#2  0x000000000040d8b9 in drive_machine (c=0x7fffe00261c0) at memcached.c:4113
	#3  0x000000000040e427 in event_handler (fd=40, which=2, arg=0x7fffe00261c0) at memcached.c:4358
	#4  0x00007ffff7de8c29 in event_process_active (base=0x63c870, flags=&lt;value optimized out&gt;) at event.c:395
	#5  event_base_loop (base=0x63c870, flags=&lt;value optimized out&gt;) at event.c:547
	#6  0x00000000004175bf in worker_libevent (arg=0x62fa60) at thread.c:378
	#7  0x0000003bce0079d1 in start_thread () from /lib64/libpthread.so.0
	#8  0x0000003bcdce89dd in clone () from /lib64/libc.so.6
