---
layout: post
title:  "keepalived搭建"
date:   2016-10-27 11:10:38 +0800

---
<img src="/images/fulls/03.jpg" class="fit image"> 


什么是Keepalived呢?keepalived观其名可知保持存活在网络里面就是 保持在线 了 也就是所谓的高可用或热备用来防止单点故障(单点故障是指一旦某一点出现故障就会导致整个系统架构的不可用)的发生那说到keepalived时不得不说的一个协议就是VRRP协议可以说这个协议就是keepalived实现的基础那么首先我们来看看VRRP协议

注搞运维的要有足够的耐心哦不理解协议就很难透彻的掌握keepalived的了
一VRRP协议
在VRRP中有两组重要的概念：VRRP路由器和虚拟路由器，主控路由器和备份路由器。
VRRP 路由器是指运行VRRP的路由器，是物理实体，虚拟路由器是指VRRP协议创建的，
是逻辑概念。一组VRRP路由器协同工作，共同构成一台虚拟路由器。 
Vrrp中存在着一种选举机制，用以选出提供服务的路由即主控路由，
其他的则成了备份路由。当主控路由失效后，
备份路由中会重新选举出一个主控路由，来继续工作，来保障不间断服务.

我们来配置keepalived
第一步，下载安装keepalived
   

    wget http://www.keepalived.org/software/keepalived-1.2.22.tar.gz  //下载keepalived  
    tar zxvf keepalived-1.2.22.tar.gz  //解压  
    cd keepalived-1.2.22/    //进入目录  
    ./configure    //配置  
    make && make install   //编译安装  

第二步 ，配置keepalived
[php]  

    vi /usr/local/etc/keepalived/keepalived.conf  

主服务器配置如下：

    global_defs {  
            router_id NodeA  
        }  
        vrrp_instance VI_1 {  
            state MASTER    #设置为主服务器  
            interface eno16777736  #监测网络接口  
            virtual_router_id 60  #主、备必须一样  
            priority 100   #(主、备机取不同的优先级，主机值较大，备份机值较小,值越大优先级越高)  
            advert_int 1   #VRRP Multicast广播周期秒数  
            authentication {  
            auth_type PASS  #VRRP认证方式，主备必须一致  
            auth_pass 1234   #(密码)  
        }  
        virtual_ipaddress {  
            192.168.1.214/24  #VRRP HA虚拟地址  
        }  
    }  


备用服务器配置如下：
   
    global_defs {  
            router_id NodeB  
        }  
        vrrp_instance VI_1 {  
            state SLAVE   #设置为备用服务器  
            interface eno16777736  #监测网络接口  
            virtual_router_id 60  #主、备必须一样  
            priority 90   #(主、备机取不同的优先级，主机值较大，备份机值较小,值越大优先级越高)  
            advert_int 1   #VRRP Multicast广播周期秒数  
            authentication {  
            auth_type PASS  #VRRP认证方式，主备必须一致  
            auth_pass 1234   #(密码)  
        }  
        virtual_ipaddress {  
            192.168.1.214/24  #VRRP HA虚拟地址  
        }  
    }  


启动keepalived：
   

    keepalived -D -f /usr/local/etc/keepalived/keepalived.conf  


第三步：验证配置是否成功。
在两个服务器的web根目录中各自建立一个test.html文件。
主服务器：
   

    this is master server  
    <br>    
    from server 192.168.1.211  

备用服务器：
   

    this is slave server  
    <br>    
    from server 192.168.1.211  

首先请确保通过各自的IP/test.html能够正常访问到。
然后通过之前设置的虚拟IP地址/test.html访问，这时应该访问到的是主服务器的内容，即：
   

    this is master server  
    from server 192.168.1.212  

接下来关闭主服务器的keepalived：
   

    killall keepalived  

这时再访问虚拟IP地址/test.html，显示的内容应该为
   

    this is slave server  
    from server 192.168.1.211  

这就说明配置成功。