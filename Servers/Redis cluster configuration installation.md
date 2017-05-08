Title: redis 集群配置安装
Date: 14:42 2016/8/3
Modified: 14:41 2016/8/3
Category: Servers
Tags: redis
Slug: 
Author: allposs

## 简介
&#160; &#160; &#160; &#160;Redis是一个开源（BSD许可），内存存储的数据结构服务器，可用作数据库，高速缓存和消息队列代理。它支持字符串、哈希表、列表、集合、有序集合，位图，hyperloglogs等数据类型。内置复制、Lua脚本、LRU收回、事务以及不同级别磁盘持久化功能，同时通过Redis Sentinel提供高可用，通过Redis Cluster提供自动分区。

&#160; &#160; &#160; &#160;Redis Cluster  是Redis的集群实现，内置数据自动分片机制，集群内部将所有的key映射到16384个Slot中，集群中的每个Redis Instance负责其中的一部分的Slot的读写。集群客户端连接集群中任一Redis Instance即可发送命令，当Redis Instance收到自己不负责的Slot的请求时，会将负责请求Key所在Slot的Redis Instance地址返回给客户端，客户端收到后自动将原请求重新发往这个地址，对外部透明。一个Key到底属于哪个Slot由crc16(key) % 16384 决定。

&#160; &#160; &#160; &#160;关于负载均衡，集群的Redis Instance之间可以迁移数据，以Slot为单位，但不是自动的，需要外部命令触发。

&#160; &#160; &#160; &#160;关于集群成员管理，集群的节点(Redis Instance)和节点之间两两定期交换集群内节点信息并且更新，从发送节点的角度看，这些信息包括：集群内有哪些节点，IP和PORT是什么，节点名字是什么，节点的状态(比如OK，PFAIL，FAIL，后面详述)是什么，包括节点角色(master 或者 slave)等。

&#160; &#160; &#160; &#160;关于可用性，集群由N组主从Redis Instance组成。主可以没有从，但是没有从 意味着主宕机后主负责的Slot读写服务不可用。一个主可以有多个从，主宕机时，某个从会被提升为主，具体哪个从被提升为主，协议类似于Raft，参见这里。如何检测主宕机？Redis Cluster采用quorum+心跳的机制。从节点的角度看，节点会定期给其他所有的节点发送Ping，cluster-node-timeout(可配置，秒级)时间内没有收到对方的回复，则单方面认为对端节点宕机，将该节点标为PFAIL状态。通过节点之间交换信息收集到quorum个节点都认为这个节点为PFAIL，则将该节点标记为FAIL，并且将其发送给其他所有节点，其他所有节点收到后立即认为该节点宕机。从这里可以看出，主宕机后，至少cluster-node-timeout时间内该主所负责的Slot的读写服务不可用。

## 环境

+ 操作系统：CentOS7 X86_64
+ Yum源：163源
+ IP地址：10.199.255.201 10.199.255.202
+ DNS：10.199.255.15
+ 主机名：moniter

## 软件包

###1. Yum源



###2. 源码包

[root@node1 ~]# wget http://download.redis.io/releases/redis-3.2.1.tar.gz

&#160; &#160; &#160; &#160;在选用版本的时候最好选择最新版本，我在后来测试安装的时候有发现编译出错的，但是选择新版本时就可以，请各位注意

## 正文

###1. 在node1上配置

####1.1 下载安装redis

	[root@node1 ~]# wget http://download.redis.io/releases/redis-3.2.1.tar.gz
	[root@node1 ~]# tar xf redis-3.2.1.tar.gz 
	[root@node1 ~]# mkdir /opt/redis_cluster/{7000,7001,7002} -p
	[root@node1 ~]# cd redis-3.2.1/
	[root@node1 redis-3.2.1]# make MALLOC=libc
	[root@node1 redis-3.2.1]# make install

####1.2 配置redis节点

	[root@node1 redis-3.2.1]# vim redis.conf 
		daemonize    yes                          //redis后台运行
		pidfile  /var/run/redis_7000.pid    //pidfile文件对应7000
		port  7000  	//端口7000
		bind 192.168.3.85                              
		cluster-enabled  yes                    //开启集群  把注释#去掉
		cluster-config-file  nodes.conf      //集群的配置  配置文件首次启动自动生成
		cluster-node-timeout  5000      //请求超时  设置5秒够了
		appendonly  yes                        //aof日志开启  有需要就开启，它会每次写操作都记录一条日志
		
	[root@node1 redis-3.2.1]# cp redis.conf /opt/redis_cluster/7000/
	[root@node1 redis-3.2.1]# cp redis.conf /opt/redis_cluster/7001/
	[root@node1 redis-3.2.1]# cp redis.conf /opt/redis_cluster/7002/
	[root@node1 redis-3.2.1]# vim /opt/redis_cluster/7000/redis.conf 
		port 7000
	[root@node1 redis-3.2.1]# vim /opt/redis_cluster/7001/redis.conf 
		port 7001
	[root@node1 redis-3.2.1]# vim /opt/redis_cluster/7002/redis.conf 
		port 7002

####1.3 启动redis节点
	[root@node1 redis-3.2.1]# cd /opt/redis_cluster/7000/
	[root@node1 7000]# redis-server  redis.conf
	[root@node1 7000]# cd /opt/redis_cluster/7001/
	[root@node1 7001]# redis-server  redis.conf
	[root@node1 7001]# cd /opt/redis_cluster/7002/
	[root@node1 7002]# redis-server  redis.conf

####1.4 配置redis节点启动脚本
	[root@node1 redis_cluster]# vim startredis.sh 
		#!/bin/bash

		pkill -9 -f 7000
		cd /opt/redis_cluster/7000/
		/usr/local/bin/redis-server redis.conf

		pkill -9 -f 7001
		cd /opt/redis_cluster/7001/
		/usr/local/bin/redis-server redis.conf

		pkill -9 -f 7002
		cd /opt/redis_cluster/7002/
		/usr/local/bin/redis-server redis.conf
	[root@node1 redis_cluster]# chmod u+x startredis.sh





###2. 在node2上配置

####2.1 下载安装redis

	[root@node2 ~]# wget http://download.redis.io/releases/redis-3.2.1.tar.gz
	[root@node2 ~]# tar xf redis-3.2.1.tar.gz 
	[root@node2 ~]# mkdir /opt/redis_cluster/{7003,7004,7005} -p
	[root@node2 ~]# cd redis-3.2.1/
	[root@node2 redis-3.2.1]# make MALLOC=libc
	[root@node2 redis-3.2.1]# make install

####2.2 配置redis节点

	[root@node2 redis-3.2.1]# vim redis.conf 
		daemonize    yes                          //redis后台运行
		pidfile  /var/run/redis_7000.pid    //pidfile文件对应7000
		bind 192.168.3.86
		port  7000                                  //端口7000
		cluster-enabled  yes                    //开启集群  把注释#去掉
		cluster-config-file  nodes.conf      //集群的配置  配置文件首次启动自动生成
		cluster-node-timeout  5000      //请求超时  设置5秒够了
		appendonly  yes                        //aof日志开启  有需要就开启，它会每次写操作都记录一条日志
	[root@node2 redis-3.2.1]# cp redis.conf /opt/redis_cluster/7003
	[root@node2 redis-3.2.1]# cp redis.conf /opt/redis_cluster/7004
	[root@node2 redis-3.2.1]# cp redis.conf /opt/redis_cluster/7005
	[root@node2 redis-3.2.1]# vim /opt/redis_cluster/7003/redis.conf 
		port 7003
	[root@node2 redis-3.2.1]# vim /opt/redis_cluster/7004/redis.conf 
		port 7004
	[root@node2 redis-3.2.1]# vim /opt/redis_cluster/7005/redis.conf 
		port 7005

####2.3 启动redis节点

	[root@node2 redis-3.2.1]# cd /opt/redis_cluster/7003/
	[root@node2 7003]# redis-server redis.conf 
	[root@node2 7003]# redis-server redis.conf 
	[root@node2 7003]# cd /opt/redis_cluster/7004/
	[root@node2 7004]# redis-server redis.conf 
	[root@node2 7004]# cd /opt/redis_cluster/7005/
	[root@node2 7005]# redis-server redis.conf

	//[root@node2 7005]# yum -y install ruby ruby-devel rubygems rpm-build
	//[root@node2 7005]# gem install redis


####2.4 配置redis节点启动脚本

	[root@node2 redis_cluster]# vim startredis.sh

		#!/bin/bash

		pkill -9 -f 7003
		cd /opt/redis_cluster/7003/
		/usr/local/bin/redis-server redis.conf

		pkill -9 -f 7004
		cd /opt/redis_cluster/7004/
		/usr/local/bin/redis-server redis.conf

		pkill -9 -f 7005
		cd /opt/redis_cluster/7005/
		/usr/local/bin/redis-server redis.conf
		
	[root@node2 redis_cluster]# chmod u+x startredis.sh



###3. 配置redis-cluster

	[root@node1 7002]# yum -y install ruby ruby-devel rubygems rpm-build 
	[root@node1 7002]# gem install redis
	[root@node1 7002]# /root/redis-3.2.1/src/redis-trib.rb create --replicas 1 192.168.3.85:7000 192.168.3.85:7001 192.168.3.85:7002 192.168.3.86:7003 192.168.3.86:7004 192.168.3.86:7005
	如果ruby下载不了，可以去官方网站下载一个，然后安装https://rubygems.org/gems/redis/versions/3.3.1
	[root@node1 7002]# wget https://rubygems.org/downloads/redis-3.3.1.gem
	[root@node1 7002]# gem install -l ./redis-3.3.1.gem

###3. Redis集群命令
	集群  
	CLUSTER INFO 打印集群的信息  
	CLUSTER NODES 列出集群当前已知的所有节点（node），以及这些节点的相关信息。  
	节点  
	CLUSTER MEET <ip> <port> 将 ip 和 port 所指定的节点添加到集群当中，让它成为集群的一份子。  
	CLUSTER FORGET <node_id> 从集群中移除 node_id 指定的节点。  
	CLUSTER REPLICATE <node_id> 将当前节点设置为 node_id 指定的节点的从节点。  
	CLUSTER SAVECONFIG 将节点的配置文件保存到硬盘里面。  
	槽(slot)  
	CLUSTER ADDSLOTS <slot> [slot ...] 将一个或多个槽（slot）指派（assign）给当前节点。  
	CLUSTER DELSLOTS <slot> [slot ...] 移除一个或多个槽对当前节点的指派。  
	CLUSTER FLUSHSLOTS 移除指派给当前节点的所有槽，让当前节点变成一个没有指派任何槽的节点。  
	CLUSTER SETSLOT <slot> NODE <node_id> 将槽 slot 指派给 node_id 指定的节点，如果槽已经指派给另一个节点，那么先让另一个节点删除该槽>，然后再进行指派。  
	CLUSTER SETSLOT <slot> MIGRATING <node_id> 将本节点的槽 slot 迁移到 node_id 指定的节点中。  
	CLUSTER SETSLOT <slot> IMPORTING <node_id> 从 node_id 指定的节点中导入槽 slot 到本节点。  
	CLUSTER SETSLOT <slot> STABLE 取消对槽 slot 的导入（import）或者迁移（migrate）。  
	键  
	CLUSTER KEYSLOT <key> 计算键 key 应该被放置在哪个槽上。  
	CLUSTER COUNTKEYSINSLOT <slot> 返回槽 slot 目前包含的键值对数量。  
	CLUSTER GETKEYSINSLOT <slot> <count> 返回 count 个 slot 槽中的键。  