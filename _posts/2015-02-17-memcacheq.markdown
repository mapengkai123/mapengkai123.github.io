---
layout: post
title:  "MemcachedQ"
date:   2015-02-17 11:10:38 +0800
---
<img src="/images/fulls/03.jpg" class="fit image"> 



持久化消息队列memcacheq(简称mcq)是一个轻量级的消息队列。mcq依赖于Berkeley DB和libevent。Berkeley DB用于持久化存储队列的数据，避免在mcq崩溃或这服务器当掉时候，不至于数据丢失。

	1.damn simple （简单易用）
	2.very fast （处理速度快）
	3.multiple queue （多条队列）
	4.concurrent well （并发性能好）
	5.memcache protocol compatible （与memcache的协议兼容）
	使用场景：

		高并发、数据可以弱一致性.(每次Http请求都是一个事务，每个表每一行都要立即更新，每次改变都要所有状态立即切换).
	运行与使用
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
		    --------------------BerkeleyDB Options-------------------------------  
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

启动MemcacheQ

	    memcacheq -d -r -H /data1/memcacheq -N -R -v -L 1024 -B 1024 > /data1/mq_error.log 2>&1  
	    memcacheq -h 查看更多设置  
	    <?php  
	    /* 连接memcacheq server */  
	    $memcache_obj = new Memcache();  
	    $memcache_obj->connect('localhost', 22201); // default port 22201   
	    /* 添加到对列 */  
	    $memcache_obj->set('demoqueue1', 'message body here1', MEMCACHE_COMPRESSED, 0);  
	    $memcache_obj->set('demoqueue1', 'message body here2', MEMCACHE_COMPRESSED, 0);  
	    $memcache_obj->set('demoqueue1', 'message body here3', MEMCACHE_COMPRESSED, 0);  
	    $memcache_obj->set('demoqueue1', 'message body here4', MEMCACHE_COMPRESSED, 0);  
	    /* 移出对列 */  
	    echo $memcache_obj->get('demoqueue1').'<br>';  
	    echo $memcache_obj->get('demoqueue1').'<br>';  
	    echo $memcache_obj->get('demoqueue1').'<br>';  

怎么解决秒杀高并发，抢购高并发。
使用消息队列来实现

可以基于例如MemcacheQ等这样的消息队列，具体的实现方案这么表述吧
比如有100张票可供用户抢，那么就可以把这100张票放到缓存中，读写时不要加锁。 当并发量大的时候，可能有500人左右抢票成功，这样对于500后面的请求可以直接转到活动结束的静态页面。进去的500个人中有400个人是不可能获得商品的。所以可以根据进入队列的先后顺序只能前100个人购买成功。后面400个人就直接转到活动结束页面。当然进去500个人只是举个例子，至于多少可以自己调整。而活动结束页面一定要用静态页面，不要用数据库。这样就减轻了数据库的压力。