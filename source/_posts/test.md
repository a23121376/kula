title: test
date: 2015-01-15 11:26:05
categories:
tags:
---

以下内容基本上是配置，原理另外说明                         -----kula

  测试坏境：      192.168.1.112 controller112.wgl.com controller112
                          192.168.1.113	 controller113.wgl.com	 controller113
                          192.168.1.111	 controller.wgl.com	 controller  （VIP：虚拟IP --2台共用）

1 前提准备：
(1).各节点之间主机名互相解析(主机名与主机名之间要可以Ping通 即须在/etc/hosts进行修改）
(2).各节点之间时间同步  
(3).关闭防火墙与SELinux
    
2 sync databases:    
            两台一起：
           1） 先卸载原来的数据库：#rpm -e MySQL-server MySQL-client --nodeps
           2） 删除原来所有的数据库：#rm -rf /var/lib/mysql
           3） 安装新版本的数据库：#yum install MySQL-client-wsrep MySQL-server-wsrep galera(注意galera版本）（出错的话多看看日志信息）
           4） 添加数据库用户以便于同步（两台都得创建）：  
         mysql> grant all on *.* to sst_user@'%' identified by 'sst_pass';
         mysql> grant all on *.* to sst_user@'localhost' identified by 'sst_pass';
         mysql> flush privileges;
    mysql> show grants for sst_user;
           5） 修改my.cnf配置文件：  
           6)  重启mysql服务#/etc/init.d/mysql restart（先停掉‘从’节点的mysq服务 然后等‘主’节点服务重启完后再启动）

   这时候可以查看一下2方的数据库有木有同步
  mysql> show status like 'wsrep_%';


3 RabbitMQ集群

RabbitMQ是用erlang开发的，集群非常方便，因为erlang天生就是一门分布式语言。
RabbitMQ的集群节点包括内存节点、磁盘节点。顾名思义内存节点就是将所有数据放在内存，磁盘节点将数据放在磁盘。不过，如前文所述，如果在投递消息时，打开了消息的持久化，那么即使是内存节点，数据还是安全的放在磁盘。
良好的设计架构可以如下：在一个集群里，有3台机器，其中1台使用磁盘模式，另2台使用内存模式。2台内存模式的节点，无疑速度更快，因此客户端（consumer、producer）连接访问它们。而磁盘模式的节点，由于磁盘IO相对较慢，因此仅作数据备份使用。
配置RabbitMQ集群非常简单，就几个命令。我没有多余的测试机，就用2台测试，hostname分别是controller112和controller113，配置步骤如下：
（1）RabbitMQ集群节点必须在同一个网段里，如果是跨广域网效果就差。
（2）在两台机上都安装和启动RabbitMQ。
（3）将controller112的/var/lib/rabbitmq/.erlang.cookie这个文件，拷贝到controller113的同一位置（反过来亦可），该文件是集群节点进行通信的验证密钥，所有节点必须一致。拷完后重启下RabbitMQ。
注意：使用SCP或者其它传输工具后，记得要确保属主是rabbitmq:
#chown rabbitmq. .erlang.cookie
Starting independent nodes

Clusters are set up by re-configuring existing RabbitMQ nodes into a cluster configuration. Hence the first step is to start RabbitMQ on all nodes in the normal way:
[root@controller112 ~]# rabbitmq-server -detached
[root@controller113 ~]# rabbitmq-server -detached

最新版本貌似有问题 暂时先用3.1.5的版本
（4）假设将dwdns1和dwdns5连接起来，在controller113上，执行如下命令：(先确保服务处于开启状态）
rabbitmqctl stop_app
rabbitmqctl join_cluster --ram rabbit@controller112     （如果没有该子命令 升级一下rabbitmq-server)
rabbitmqctl start_app
rabbitmqctl cluster_status
上述命令将controller113连接到controller112，使两者成为一个集群，最后重启rabbitmq应用。在这个cluster命令下，controller113是内存节点（RabbitMQ启动后，默认是磁盘节点）。
在RabbitMQ集群里，必须至少有一个磁盘节点存在。
#rabbitmqctl set_policy ha-all "^ha\." '{"ha-mode":"all"}'


如果rabittmq服务起不来,可以先把beam有关的进程kill掉或者删除一下rm /var/lib/rabbitmq/mnesia -rf这个目录
官方文档：
http://www.rabbitmq.com/clustering.html
4 安装配置corosync与pacemaker
（参考链接 http://freeloda.blog.51cto.com/2033581/1272417）
node1:
1
2
[root@controller112 ~]# yum install -y corosync*
[root@controller112 ~]# yum install -y pacemaker*
node2:
1
2
[root@controller113 ~]# yum install -y corosync*
[root@controller113 ~]# yum install -y pacemaker*


两台配置一样，配置文件如下：
#————————————————————————————————————————————————————————————————————————————————————————————————————————
compatibility: whitetank
totem { 
    version: 2  
    secauth: on #启动认证  
    threads: 0  
     cluster_name: controller  #vip对应的主机名
    interface {  
        ringnumber: 0  
        bindnetaddr: 192.168.18.0 #修改心跳线网段  
        mcastaddr: 226.99.10.10 #组播传播心跳信息  
        mcastport: 5405  
        ttl: 1  
    }  
}
logging { 
    fileline: off  
    to_stderr: no  
    to_logfile: yes  
    to_syslog: no  
    logfile: /var/log/cluster/corosync.log #日志位置  
    debug: off  
    timestamp: on  
    logger_subsys {  
        subsys: AMF  
        debug: off  
    }  
}
amf { 
    mode: disabled  
}
#启用pacemaker
service { 
    ver: 0  
    name: pacemaker  
}
aisexec { 
    user: root  
    group: root  
}
#——————————————————————————————————————————————————————————————————————————
生成密钥文件
注：corosync生成key文件会默认调用/dev/random随机数设备，一旦系统中断的IRQS的随机数不够用，将会产生大量的等待时间，因此，为了节约时间，我们在生成key之前讲random替换成urandom，以便节约时间。
1
2
3
4
5
6
7
[root@controller112 corosync]# mv /dev/{random,random.bak}  
[root@controller112 corosync]# ln -s /dev/urandom /dev/random
[root@controller112 corosync]# corosync-keygen  
Corosync Cluster Engine Authentication key generator.  
Gathering 1024 bits for key from /dev/random.  
Press keys on your keyboard to generate entropy.  
Writing corosync key to /etc/corosync/authkey.


将key文件authkey与配置文件corosync.conf复制另一台去
corosync 到这里配置全部完成。下面我们进行启动测试！

先启动corosync服务:
(1).查看corosync引擎是否正常启动
 grep -e "Corosync Cluster Engine" -e "configuration file" /var/log/cluster/corosync.log 
(2).查看初始化成员节点通知是否正常发出
grep  TOTEM /var/log/cluster/corosync.log
(3).检查启动过程中是否有错误产生 
grep ERROR: /var/log/cluster/corosync.log 
(4).查看pacemaker是否正常启动
grep pcmk_startup /var/log/cluster/corosync.log  
查看集群状态
 #crm_mon


5 crm启动VIP并安装与启动haproxy
    1）安装haproxy:#yum install haproxy
    2 ) 进入crm开始启动VIP（确保在同一个局域网内没有被占用的一个IP）（没有crm命令时，要先安装一下crmsh这个包）


在configure模式下导入：
crm(live)configure#primitive NTP lsb:ntpd
crm(live)configure#clone ntp-clone NTP
crm(live)configure#property stonith-enabled="false"    
crm(live)configure#property no-quorum-policy="ignore"
crm(live)configure#property default-resource-stickiness="100"
crm(live)configure#primitive VIP ocf:heartbeat:IPaddr2 params ip="192.168.1.111" nic="br-eth0" cidr_netmask="24"  meta target-role="Started"
crm(live)configure#location cli-prefer-VIP VIP inf: controller112
crm(live)configure#commit                                     -------------------提交并保存，然后exit退出来
  这时候再查看一下机群状态crm_mon 会发现VIP起来了但是还没有haproxy代理 下面应该来配置openstack组件配置文件内容（主要做的是原本的IP改成VIP）


      3）接着配置一下2台的haproxy配置文件：
 
      4）先启动其中一台haproxy（一定要确保它起来）然后在这台开始配置下面文件 ｛如果起不来可以查看一下是否有相关进程占了然后kill掉 或者配置文件出错｝先修改下面这个配置文件 方便haproxy启动
    
-rw-r--r-- 1 root root 34432 Jun 9 17:33 httpd.conf    ------------这个文件主要修改的是
# prevent Apache from glomming onto all bound IP addresses (0.0.0.0)
# Listen 80       →↓
Listen 192.168.1.112:80

修改openstack组件配置文件（2台都要修改）并同步数据库（只要一台同步就行，另外一台也会有一样的信息）
主要需要修改配置文件如下：
-rwxr-x--- 1 keystone keystone 499 Jun 9 20:02 keystone.conf
-rw-r--r-- 1 root root 2888 Jun 9 17:46 config.yaml
-rwxr-xr-x 1 root root 3463 Jun 9 17:46 keystone-init.py             --------数据库同步后需写进新的表格 python  keystone-init.py config.yaml
      

-rw-r--r-- 1 glance glance 469 Jun 9 16:59 glance-api.conf
-rw-r--r-- 1 glance glance 2710 Jun 9 16:59 glance-api-paste.ini
-rw-r--r-- 1 glance glance 332 Jun 9 16:59 glance-registry.conf
-rw-r--r-- 1 glance glance 332 Jun 9 16:59 glance-registry-paste.ini -------------上传的镜像2边都要有镜像文件#cd /var/lib/glance/images/ （上传镜像时候 它会根据source算法选择一台上传）
                                  

-rw-r--r-- 1 cinder cinder 1964 Jun 9 19:22 api-paste.ini
-rw-r--r-- 1 cinder cinder 449 Jun 9 19:19 cinder.conf
  

-rw-r--r-- 1 nova nova 3852 Jun 9 17:05 api-paste.ini
-rw-r--r-- 1 nova nova 1787 Jun 9 17:05 nova.conf
    同步这个数据会比较慢

-rw-r--r-- 1 root neutron 871 Jun 9 17:10 api-paste.ini
-rw-r--r-- 1 root neutron 500 Jun 9 17:10 dhcp_agent.ini
-rw-r--r-- 1 root neutron 662 Jun 9 18:52 neutron.conf
lrwxrwxrwx 1 root neutron 55 Jun 8 15:42 plugin.ini               -----------------数据库名是ovs_neutron
      
   
-rw-r--r-- 1 root apache 14548 Jun 9 19:13 local_settings

web 访问的直接地址OPENSTACK_HOST =  vip

另外，就是注意每个文件的属主以及它所在的属主都应该与相关组件对应


接下来需要同步数据库：#openstack-db --init --service xxxx --password xxxx
除了neutron数据库 其它的都用上述命令创建
# mysql -u root -p
mysql> CREATE DATABASE ovs_neutron;
mysql> GRANT ALL PRIVILEGES ON ovs_neutron.* TO 'neutron'@'localhost' IDENTIFIED BY 'neutron';
mysql> GRANT ALL PRIVILEGES ON ovs_neutron.* TO 'neutron'@'%' IDENTIFIED BY 'neutron';

检查另一台有木有同步~


继续进入crm，开始使用代理
crm(live)configure#primitive HAPROXY lsb:haproxy op monitor interval="30s" meta target-role="Started"
crm(live)configure#colocation haproxy-with-vip inf: VIP HAPROXY
crm(live)configure#order vip-before-haproxy Mandatory: VIP HAPROXY
crm(live)configure#commit 
然后重启启动所有openstack组件服务,接下来是各种调优测试~
目前还没上HA的只剩下puppetmaster。。。后期需完善
