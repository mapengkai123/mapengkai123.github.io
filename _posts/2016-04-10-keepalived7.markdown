---
layout: post
title:  "双机热备"
date:   2016-04-10 16:05:38 +0800
---
<img src="/images/fulls/08.jpg" class="fit image">


通常说的双机热备是指两台机器都在运行，但并不是两台机器都同时在提供服务。
当提供服务的一台出现故障的时候，另外一台会马上自动接管并且提供服务，而且切换的时间非常短。

keepalived的工作原理是VRRP（VirtualRouter Redundancy Protocol）虚拟路由冗余协议。
在VRRP中有两组重要的概念：VRRP路由器和虚拟路由器，主控路由器和备份路由器。
VRRP 路由器是指运行VRRP的路由器，是物理实体，虚拟路由器是指VRRP协议创建的，是逻辑概念。一组VRRP路由器协同工作，共同构成一台虚拟路由器。 Vrrp中存在着一种选举机制，用以选出提供服务的路由即主控路由，其他的则成了备份路由。当主控路由失效后，备份路由中会重新选举出一个主控路由，来继续工作，来保障不间断服务。

1.准备两台服务器

主服务器：192.168.1.111

从服务器：192.168.1.199

虚拟ip:192.168.1.223

两台机器安装

2.安装keepalived需要的依赖包

yum install openssl-devel
yum install popt-devel
yum install ipvsadm
yum install libnl*

3.下载keepalived

yum install keepalived

4.修改主服务器配置文件

vim /etc/keepalived/keepalived.conf

! Configuration File for keepalived

 

global_defs {

  notification_email {

    #acassen@firewall.loc没有服务器配置邮箱可将其注释掉

    #failover@firewall.loc

    #sysadmin@firewall.loc

   }

  #notification_email_from Alexandre.Cassen@firewall.loc

   #smtp_server192.168.200.1

  #smtp_connect_timeout 30

   router_idLVS_DEVEL

}

 

vrrp_instance VI_1 {

    state MASTER

    interface eno16777736

   virtual_router_id 51#和slave一样

    priority 100#主机高于slave

    advert_int 1#检测服务器状态间隔时间

    authentication{

        auth_typePASS

        auth_pass 1111

    }

   virtual_ipaddress {

        192.168.1.223#虚拟IP地址，可以为多个

    }

}

 

开启服务

systemctl start keepalived

5.修改slave配置

vim /etc/keepalived/keepalived.conf

! Configuration File for keepalived

 

global_defs {

 #notification_email {

  #  644856452@qq.com

  #}

 #notification_email_from Alexandre.Cassen@firewall.loc

  #smtp_server127.0.0.1

 #smtp_connect_timeout 30

  router_idLVS_DEVEL

}

 

vrrp_instance VI_1 {

   state SLAVE

  interface eno16777736

  virtual_router_id 51

   priority 99#低于主服务器100

   advert_int 1

   authentication {

       auth_typePASS

       auth_pass1111#验证密码，两台机器保持一致

   }

 

  virtual_ipaddress {

      192.168.1.223

   }

}

开启服务

systemctl start keepalived

6.在两台服务器web根目录下建立一个index.PHP文件，写上本机ip地址

7.在两台机器上使用 "ipa" 查看虚拟 IP 信息

可以看到，虚拟Ip此时绑定在主机上

在浏览器中输入虚拟ip192.168.1.223此时将看到

访问的是master,那么将master的服务关闭呢？在192.168.1.111上运行systemctl stop keepalived

此时再看两台机器的虚拟ip信息

此时可以看出虚拟ip绑定到了slave服务器上。

在浏览器中输入192.168.1.223可以看到

主机服务挂掉了，此时访问的是slave.

此时在主服务器上打开keepalived服务，systemctl start keepalived

再次访问192.168.1.223，将看到

主服务器又活了

那么如何根据服务某个端口的开与关来进行虚拟Ip的绑定呢？

Vim /usr/share/doc/keepalived-1.2.13/samples/keepalived.conf.vrrp.localcheck

参考提供的例子

! Configuration File for keepalived

vrrp_script chk_http_port {

       script"</dev/tcp/127.0.0.1/80" # connects and exits

       interval1                      # check everysecond

       weight-2                       # default prio:-2 if connect fails

}

vrrp_script chk_mysql_port {

       script"</dev/tcp/127.0.0.1/80" # connects and exits

       interval1                      # check everysecond

       weight-2                       # default prio:-2 if connect fails

}

 

    track_script {

      chk_http_port

       chk_mysql_port

}

将将以上信息复制到两台服务器的/etc/keepalived/keepalived.conf文件里

变成如下，参考，注意从机不一样，为了讲清楚上面的信息放入的位置。

! Configuration File for keepalived

 

global_defs {

  notification_email {

    acassen@firewall.loc

    failover@firewall.loc

    sysadmin@firewall.loc

  }

  notification_email_from Alexandre.Cassen@firewall.loc

  smtp_server 192.168.200.1

  smtp_connect_timeout 30

  router_id LVS_DEVEL

}

vrrp_script chk_http_port {

       script"</dev/tcp/127.0.0.1/80" # connects and exits

       interval 1                      # check every second

       weight -2                       # default prio: -2 ifconnect fails

}

 

vrrp_script chk_mysql_port {

       script"</dev/tcp/127.0.0.1/3306" # connects and exits

       interval 1                      # check every second

       weight -2                   # default prio: -2 ifconnect fails

}

vrrp_instance VI_1 {

   state MASTER

   interface eno16777736

   virtual_router_id 51

   priority 100

   advert_int 1

   authentication {

        auth_type PASS

        auth_pass aaa

   }

   virtual_ipaddress {

        192.168.1.229

   }

   track_script {

       chk_http_port

chk_mysql_port

   }

}

此时再重启两台机器的keepalived服务

Systemctl restart keepalived

此时分别关闭主机80和3306端口服务

可以发现虚拟Ip绑定到了从机上。