---
layout: post
title:  "session入redis  "
date:   2016-06-19 18:05:38 +0800
---
<img src="/images/fulls/10.jpg" class="fit image">


Session信息入Redis

Session简介

session，中文经常翻译为会话，其本来的含义是 指有始有终的一系列动作/消息，比如打电话时从拿起电话拨号到挂断电话这中间的一系列过程可以称之为一个session。有时候我们可以看到这样的话“在 一个浏览器会话期间，...”，这里的会话一词用的就是其本义，是指从一个浏览器窗口打开到关闭这个期间①。最混乱的是“用户（客户端）在一次会话期间”这样一句话，它可能指用户的一系列动作（般情况下是同某个具体目的相关的一系列动作，比如从登录到选购商品到结账登出这样一个网上购物的过程，有时候也 被称为一个transaction），然而有时候也可能仅仅是指一次连接，也有可能是指含义①，其中的差别只能靠上下文来推断②。 

    然而当session一词与网络协议相关联时，它又往往隐含了“面向连接”和/或“保持状态”这样两个含义，“面向连接”指的是在通信双方在通信之前要先 建立一个通信的渠道，比如打电话，直到对方接了电话通信才能开始，与此相对的是写信，在你把信发出去的时候你并不能确认对方的地址是否正确，通信渠道不一 定能建立，但对发信人来说，通信已经开始了。“保持状态”则是指通信的一方能够把一系列的消息关联起来，使得消息之间可以互相依赖，比如一个服务员能够认 出再次光临的老顾客并且记得上次这个顾客还欠店里一块钱。这一类的例子有“一个TCP session”或者“一个POP3 session”③。 

    而到了web服务器蓬勃发展的时代，session在web开发语境下的语义又有了新的扩展，它的含义是指一类用来在客户端与服务器之间保持状态的解决方 案④。有时候session也用来指这种解决方案的存储结构，如“把xxx保存在session里”⑤。由于各种用于web开发的语言在一定程度上都提供 了对这种解决方案的支持，所以在某种特定语言的语境下，session也被用来指代该语言的解决方案，比如经常把Java里提供的javax.servlet.http.HttpSession简称为session⑥。 

    鉴于这种混乱已不可改变，本文中session一词的运用也会根据上下文有不同的含义，请大家注意分辨。 
在本文中，使用中文“浏览器会话期间”来表达含义①，使用“session机制”来表达含义④，使用“session”表达含义⑤，使用具体的“HttpSession”来表达含义⑥ 

为什么要把SESSION保存在缓存

就PHP来说，语言本身支持的session是以文件的方式保存到磁盘文件中，保存在指定的文件夹中，保存的路径可以在配置文件中设置或者在程序中使用函数session_save_path()进行设置，但是这么做有弊端，

第一就是保存到文件系统中，效率低，只要有用到session就会从好多个文件中查找指定的sessionid，效率很低。

第二就是当用到多台服务器的时候可能会出现，session丢失问题（其实是保存在了其他服务器上）。

当然了，保存在缓存中可以解决上面的问题，如果使用php本身的session函数，可以使用 session_set_save_handler()函数很方便的对session的处理过程进行重新控制。如果不用php的session系列函数， 可以自己编写个类似的session函数，也是可以的，我现在做的这个项目就是这样，会根据用户的mid、登录时间进行求hash作为 sessionId，每次请求的时候都必须加上sessionId才算合法（第一次登录的时候是不需要的，这个时候会创建sessionId，返回给客户 端），这么做也很方便、简洁高效的。当然了，我这篇文章主要说的是在php自身的SESSION中”做做手脚”。

SESSION保存在缓存中

php将缓存保存到redis中，可以使用配置文件，对session的处理和保存做修改，当然了，在程序中使用ini_set()函数去修改也可以，这个很方便测试，我这里就使用这种方式，当然了，要是生产环境还是建议使用配置文件。

如果想简单操作session入redis操作可以将一下代码运行一下

    <?php

    ini_set("session.save_handler", "redis");

    ini_set("session.save_path", "tcp://localhost:6379");

    session_start();

    header("Content-type:text/html;charset=utf-8");

    $_SESSION['view'] = 'zhangsan';

    echo $_SESSION['view'];

    这里设置session.save_handler方式为redis，session.save_path为redis的地址和端口，设置之后刷新，再回头查看redis，会发现redis中的生成了sessionId，sessionId和浏览器请求的是一样的，

     如果是memcache

    <?php

    ini_set("session.save_handler", "memcache");

    ini_set("session.save_path", "tcp://localhost:11211");

    session_start();

    header("Content-type:text/html;charset=utf-8");

    $_SESSION['view'] = 'zhangsan';

    echo $_SESSION['view'];

    也可以使用

    Session_set_save_handler(‘open’,’close’,’ read’,’ write’,’ destory’,’ gc’);

    用法如下自定义一个Redis_session类

    <?php

    class RedisSession{

        private $_redis = array(

            'handler' => null, //数据库连接句柄

            'host' => null,   //redis端口号

            'port' => null,

        );

        public function __construct($array = array()){

            isset($array['host'])?$array['host']:"false";

            isset($array['port'])?$array['host']:"false";

            $this->_redis = array_merge($this->_redis, $array);

        }

        public function begin(){

            //设置session处理函数

            session_set_save_handler(

                array($this, 'open'),

                array($this, 'close'),

                array($this, 'read'),

                array($this, 'write'),

                array($this, 'destory'),

                array($this, 'gc')

            );

        }

        public function open(){

            $redis = new Redis();

            $redis->connect($this->_redis['host'], $this->_redis['port']);

            if(!$redis){

                return false;

            }

     

            $this->_redis['handler'] = $redis;

            $this->gc(null);

            return true;

        }

        //关

        public function close(){

            return $this->_redis['handler']->close();

        }

        //读

        public function read($session_id){

            return $this->_redis['handler']->get($session_id);

        }

        //写

        public function write($sessionId, $sessionData){

            return $this->_redis['handler']->set($sessionId, $sessionData);

        }

        public function destory($sessionId){

            return $this->_redis['handler']->delete($sessionId) >= 1 ? true : false;

        }

        public function gc(){

            //获取所有sessionid，让过期的释放掉

            $this->_redis['handler']->keys("*");

            return true;

        }

    }

    $ses = new RedisSession(array('host'=>'127.0.0.1','port'=>'6379'));

    $ses->begin();

    session_start();

    $_SESSION['name']='zhangsan';

    echo $_SESSION['name'];

 

 

这样就可以实现session数据如redis代码执行过程中必须安装redis才可以
