---
layout: post
title:  "MemcachedQ"
date:   2015-02-17 11:10:38 +0800
---
<img src="/images/fulls/03.jpg" class="fit image"> 


memcacheq是memcache基础上的一个功能插件，q是queue的意思，是一个列队;
持久化消息队列memcacheq(简称mcq)是一个轻量级的消息队列。mcq依赖于Berkeley DB和libevent。Berkeley DB用于持久化存储队列的数据，避免在mcq崩溃或这服务器当掉时候，不至于数据丢失。

	1.简单易用
	2.处理速度快
	3.多条队列
	4.并发性能好
	5.与memcache的协议兼容

使用场景：

	高并发、数据可以弱一致性.(每次Http请求都是一个事务，每个表每一行都要立即更新，每次改变都要所有状态立即切换).运行与使用

参数列表：

    -p <num>      TCP监听端口(default: 22201)  
    -U <num>      UDP监听端口(default: 0, off)  
    -s <file>     unix socket路径(不支持网络)  
    -a <mask>     unix socket访问掩码(default 0700)  
    -l <ip_addr>  监听网卡  
    -d            守护进程  
    -r            最大化核心文件限制  
    -u <username> 以用户身份运行(only when run as root)  
    -c <num>      最大并发连接数(default is 1024)  
    -v            详细输出 (print errors/warnings while in event loop)  
    -vv           更详细的输出 (also print client commands/reponses)  
    -i            打印许可证信息  
    -P <file>     PID文件  
    -t <num>      线程数(default 4)

    --------------BerkeleyDB Options------------------

    -m <num>      BerkeleyDB内存缓存大小, default is 64MB  
    -A <num>      底层页面大小, default is 4096, (512B ~ 64KB, power-of-two)  
    -H <dir>      数据库家目录, default is '/data1/memcacheq'  
    -L <num>      日志缓冲区大小, default is 32KB  
    -C <num>      多少秒checkpoint一次, 0 for disable, default is 5 minutes  
    -T <num>      多少秒memp_trickle一次, 0 for disable, default is 30 seconds  
    -S <num>      多少秒queue stats dump一次, 0 for disable, default is 30 seconds  
    -e <num>      达到缓存百分之多少需要刷新, default is 60%  
    -E <num>      一个单一的DB文件有多少页, default is 16*1024, 0 for disable  
    -B <num>      指定消息体的长度,单位字节, default is 1024  
    -D <num>      多少毫秒做一次死锁检测(deadlock detecting), 0 for disable, default is 100ms  
    -N            开启DB_TXN_NOSYNC获得巨大的性能改善, default is off  
    -R            自动删除不再需要的日志文件, default is off  

MemcacheQ 的安装与使用

1、安装libevent

官网：http://www.libevent.org/

	$ wget https://github.com/downloads/libevent/libevent/libevent-2.0.21-stable.tar.gz --no-check-certificate
	$ tar -zxvf libevent-2.0.21-stable.tar.gz
	$ cd libevent-2.0.21-stable
	$ ./configure --prefix=/usr/local/libevent
	$ sudo make && make install

2、安装 BerkeleyDB

官网：http://www.oracle.com/technetwork/products/berkeleydb/downloads/index.html（下载需要登录）

安装：

	$ wget http://download.oracle.com/berkeley-db/db-6.0.30.tar.gz
	$ tar -zxvf db-6.0.30.tar.gz (根据自身的情况解压到目录)
	$ cd db-6.0.30/build_unix
	$ ../dist/configure --prefix=/usr/local/berkeleyDB
	$ sudo make && make install

安装完成之后：

	$ export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/libevent/lib:/usr/local/berkeleyDB/lib

或者：

	$ vim /etc/ld.so.conf

添加：

	/usr/local/libevent/lib
	/usr/local/berkeleyDB/lib

并执行：

$ /sbin/ldconfig

3、安装 MemcacheQ

	官网：http://memcachedb.org/memcacheq/
	$ wget https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/memcacheq/memcacheq-0.2.0.tar.gz
	$ tar -zxvf memcacheq-0.2.0.tar.gz
	$ cd memcacheq-0.2.0
	$ ./configure --prefix=/usr/local/memcacheq --enable-threads --with-libevent=/usr/local/libevent --with-bdb=/usr/local/berkeleyDB
	$ sudo make && make install

测试是否安装成功：

	/usr/local/memcacheq/bin/memcacheq -h

4、启动服务

建立相关目录：

	mkdir -p 755 /var/memcacheq/logs/
	mkdir -p 755 /var/memcacheq/data/

启动服务：

	/usr/local/memcacheq/bin/memcacheq -d -r -uroot -p11212 -H /var/memcacheq/data -N -R -v -L 1024 -B 1024 > /var/memcacheq/logs/mq_error.log 2>&1

参数说明：

    -d : 以后台服务方式运行
    -l : 设置监听地址及端口（默认是22201）
    -A : 数据页大小
    -H : 数据保存目录
    -B : 队列中每条数据的最大长度（字节）
    -N : 使用内存缓冲方式保存数据至磁盘，从而获得极高性能。若无此参数，性能会很差
    -R : 自动清理过期的日志
    -u : 设置memcacheq进程账号
    -L ：日志缓存大小（默认是32K，1024表示1024K）

添加到开机启动：

vim /etc/rc.local

添加：

/usr/local/memcacheq/bin/memcacheq -d -r -uroot -p11212 -H /var/memcacheq/data -N -R -v -L 1024 -B 1024 > /var/memcacheq/logs/mq_error.log 2>&1

5、使用 PHP 连接

	<?php
    $memcache_obj = new Memcache;
    $memcache_obj->connect('127.0.0.1', 11212);
    $memcache_obj->set('pushtask','38a7d8a504d69d1e2ac843b5804c04',0,0);
    $memcache_obj->close();
	?>

	<?php
    $memcache_obj = new Memcache;
    $memcache_obj->connect('127.0.0.1', 11212);
    $task = $memcache_obj->get('pushtask');
    //do your task here
    ...
    $memcache_obj->close();
	?>

6、使用命令行连接

连接：

telnet 127.0.0.1 11212

添加：

格式：

	set <queue name> <flags> 0 <message_len>
	<put your message body here>
	STORED

示例：

	set test_queue 0 0 6
	abcdef
	STORED

获取：

格式：

	get <queue name>
	VALUE <queue name> <flags> <message_len>
	<your message body will come here>
	END

示例：

	get test_queue
	VALUE test_queue 0 13
	first_message
	END

查看队列：

	stats queue
	STAT test_queue 20/10    #表示队列test_queue里面有20条信息,读取了10条
	END

删除整个队列：

	格式：
	delete <queue name>

	示例：
	delete test_queue
	DELETED

利用memcacheQ怎么解决秒杀高并发，抢购高并发。
使用消息队列来实现

可以基于例如MemcacheQ等这样的消息队列，具体的实现方案这么表述吧
比如有100张票可供用户抢，那么就可以把这100张票放到缓存中，读写时不要加锁。 当并发量大的时候，可能有500人左右抢票成功，这样对于500后面的请求可以直接转到活动结束的静态页面。进去的500个人中有400个人是不可能获得商品的。所以可以根据进入队列的先后顺序只能前100个人购买成功。后面400个人就直接转到活动结束页面。当然进去500个人只是举个例子，至于多少可以自己调整。而活动结束页面一定要用静态页面，不要用数据库。这样就减轻了数据库的压力。