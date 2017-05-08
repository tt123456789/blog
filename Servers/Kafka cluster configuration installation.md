Title: kafka集群配置安装
Date: 17:01 2016/8/16
Modified: 17:01 2016/8/16
Category: Servers
Tags: kafka
Slug: 
Author: allposs


## 简介
&#160; &#160; &#160; &#160;Kafka是一种高吞吐量的分布式发布订阅消息系统，它可以处理消费者规模的网站中的所有动作流数据。 这种动作（网页浏览，搜索和其他用户的行动）是在现代网络上的许多社会功能的一个关键因素。 这些数据通常是由于吞吐量的要求而通过处理日志和日志聚合来解决。 对于像Hadoop的一样的日志数据和离线分析系统，但又要求实时处理的限制，这是一个可行的解决方案。Kafka的目的是通过Hadoop的并行加载机制来统一线上和离线的消息处理，也是为了通过集群机来提供实时的消费。

## 环境
+ 操作系统：CentOS 7.2 x64
+ Yum源：163源
+ IP地址：
			node1 10.199.200.101
			node2 10.199.200.102
			node3 10.199.200.103
+ DNS：
+ 主机名：
			node1.example.com
			node2.example.com
			node3.example.com

## 软件包

###1. Yum源


###2. 源码包
	[root@node1 ~]# wget http://apache.fayea.com/zookeeper/zookeeper-3.4.6/zookeeper-3.4.6.tar.gz
	[root@node1 ~]# wget http://mirrors.cnnic.cn/apache/kafka/0.8.2.2/kafka_2.11-0.8.2.2.tgz

##拓扑图



## 正文
###1. 准备配置（所有节点）

	[root@node1 ~]# yum localinstall jdk-8u101-linux-x64.rpm
	[root@node1 ~]# vim /etc/hosts
	
		127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
		::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
		10.199.200.101   node1 node1.example.com
		10.199.200.102   node2 node2.example.com
		10.199.200.102   node2 node2.example.com

###2. 安装zookeeper集群

####2.1 安装配置zookeeper
	[root@node1 ~]# wget http://apache.fayea.com/zookeeper/zookeeper-3.4.6/zookeeper-3.4.6.tar.gz
	[root@node1 ~]# tar xf zookeeper-3.4.6.tar.gz 
	[root@node1 ~]# cd zookeeper-3.4.6/
	[root@node1 zookeeper-3.4.6]# cp conf/zoo_sample.cfg conf/zoo.cfg 
	[root@node1 zookeeper-3.4.6]# vim conf/zoo.cfg
		# The number of milliseconds of each tick
		tickTime=2000
		# The number of ticks that the initial 
		# synchronization phase can take
		initLimit=10
		# The number of ticks that can pass between 
		# sending a request and getting an acknowledgement
		syncLimit=5
		# the directory where the snapshot is stored.
		# do not use /tmp for storage, /tmp here is just 
		# example sakes.
		dataDir=/opt/zookeeper/data
		# the port at which the clients will connect
		clientPort=2181
		# the maximum number of client connections.
		# increase this if you need to handle more clients
		#maxClientCnxns=60
		#
		# Be sure to read the maintenance section of the 
		# administrator guide before turning on autopurge.
		#
		# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
		#
		# The number of snapshots to retain in dataDir
		#autopurge.snapRetainCount=3
		# Purge task interval in hours
		# Set to "0" to disable auto purge feature
		autopurge.purgeInterval=1

		server.1 = node1:2888:3888
		server.2 = node2:2888:3888
		server.3 = node3:2888:3888

	tickTime：这个时间是作为 Zookeeper 服务器之间或客户端与服务器之间维持心跳的时间间隔，也就是每个 tickTime 时间就会发送一个心跳。
	dataDir：顾名思义就是 Zookeeper 保存数据的目录，默认情况下，Zookeeper 将写数据的日志文件也保存在这个目录里。
	clientPort：这个端口就是客户端连接 Zookeeper 服务器的端口，Zookeeper 会监听这个端口，接受客户端的访问请求。
	initLimit：这个配置项是用来配置 Zookeeper 接受客户端（这里所说的客户端不是用户连接 Zookeeper 服务器的客户端，而是 Zookeeper 服务器集群中连接到 Leader 的 Follower 服务器）初始化连接时最长能忍受多少个心跳时间间隔数。当已经超过 5个心跳的时间（也就是 tickTime）长度后 Zookeeper 服务器还没有收到客户端的返回信息，那么表明这个客户端连接失败。总的时间长度就是 5*2000=10 秒
    syncLimit：这个配置项标识 Leader 与Follower 之间发送消息，请求和应答时间长度，最长不能超过多少个 tickTime 的时间长度，总的时间长度就是2*2000=4 秒
    server.A=B：C：D：其中 A 是一个数字，表示这个是第几号服务器；B 是这个服务器的 ip 地址；C 表示的是这个服务器与集群中的 Leader 服务器交换信息的端口；D 表示的是万一集群中的 Leader 服务器挂了，需要一个端口来重新进行选举，选出一个新的 Leader，而这个端口就是用来执行选举时服务器相互通信的端口。如果是伪集群的配置方式，由于 B 都是一样，所以不同的 Zookeeper 实例通信端口号不能一样，所以要给它们分配不同的端口号
	注意:dataDir,dataLogDir中的wwb是当前登录用户名，data，logs目录开始是不存在，需要使用mkdir命令创建相应的目录。并且在该目录下创建文件myid,serve1,server2,server3该文件内容分别为1,2,3。
	针对服务器server2,server3可以将server1复制到相应的目录，不过需要注意dataDir,dataLogDir目录,并且文件myid内容分别为2,3	
	
	[root@node1 zookeeper-3.4.6]# mkdir data
	[root@node1 ~]# mv zookeeper-3.4.6 /opt/zookeeper
	[root@node1 ~]# vim /opt/zookeeper/data/myid
		1
	[root@node1 ~]# cd /opt/
	[root@node1 opt]# scp -r zookeeper root@node2:/opt/
	[root@node1 opt]# scp -r zookeeper root@node3:/opt/
	
####2.2 启动Zookeeper集群

	[root@node1 opt]#  /opt/zookeeper/bin/zkServer.sh start
	JMX enabled by default
	Using config: /opt/zookeeper/bin/../conf/zoo.cfg
	Starting zookeeper ... STARTED
	[root@node1 ~]# /opt/zookeeper/bin/zkServer.sh status
		JMX enabled by default
		Using config: /opt/zookeeper/bin/../conf/zoo.cfg
		Mode: follower

	[root@node2 ~]# vim /opt/zookeeper/data/myid
		2
	[root@node2 ~]# /opt/zookeeper/bin/zkServer.sh start
		JMX enabled by default
		Using config: /opt/zookeeper/bin/../conf/zoo.cfg
		Starting zookeeper ... STARTED
	[root@node2 ~]# /opt/zookeeper/bin/zkServer.sh status
		JMX enabled by default
		Using config: /opt/zookeeper/bin/../conf/zoo.cfg
		Mode: leader

	[root@node3 ~]# vim /opt/zookeeper/data/myid
		3
	[root@node3 ~]# /opt/zookeeper/bin/zkServer.sh start
		JMX enabled by default
		Using config: /opt/zookeeper/bin/../conf/zoo.cfg
		Starting zookeeper ... STARTED
	[root@node3 ~]# /opt/zookeeper/bin/zkServer.sh status
		JMX enabled by default
		Using config: /opt/zookeeper/bin/../conf/zoo.cfg
		Mode: follower

####2.3 测试Zookeeper集群
	[root@node1 opt]# /opt/zookeeper/bin/zkCli.sh -server10.199.200.101:2181
	
###3. 安装配置kafka集群

####3.1 安装kafka
	[root@node1 ~]# wget http://mirrors.cnnic.cn/apache/kafka/0.8.2.2/kafka_2.11-0.8.2.2.tgz
	[root@node1 ~]# tar xf kafka_2.11-0.8.2.2.tgz
	[root@node1 ~]# mv kafka_2.11-0.8.2.2 /opt/kafka
	[root@node1 ~]# cd /opt/kafka/
	
####3.2 配置kafka

	[root@node1 kafka]# vim config/server.properties 

		# Licensed to the Apache Software Foundation (ASF) under one or more
		# contributor license agreements.  See the NOTICE file distributed with
		# this work for additional information regarding copyright ownership.
		# The ASF licenses this file to You under the Apache License, Version 2.0
		# (the "License"); you may not use this file except in compliance with
		# the License.  You may obtain a copy of the License at
		# 
		#    http://www.apache.org/licenses/LICENSE-2.0
		# 
		# Unless required by applicable law or agreed to in writing, software
		# distributed under the License is distributed on an "AS IS" BASIS,
		# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
		# See the License for the specific language governing permissions and
		# limitations under the License.
		# see kafka.server.KafkaConfig for additional details and defaults

		############################# Server Basics #############################

		# The id of the broker. This must be set to a unique integer for each broker.
		broker.id=1

		############################# Socket Server Settings #############################

		# The port the socket server listens on
		port=9092

		# Hostname the broker will bind to. If not set, the server will bind to all interfaces
		host.name=10.199.200.101

		# Hostname the broker will advertise to producers and consumers. If not set, it uses the
		# value for "host.name" if configured.  Otherwise, it will use the value returned from
		# java.net.InetAddress.getCanonicalHostName().
		#advertised.host.name=<hostname routable by clients>

		# The port to publish to ZooKeeper for clients to use. If this is not set,
		# it will publish the same port that the broker binds to.
		#advertised.port=<port accessible by clients>

		# The number of threads handling network requests
		num.network.threads=3

		# The number of threads doing disk I/O
		num.io.threads=8

		# The send buffer (SO_SNDBUF) used by the socket server
		socket.send.buffer.bytes=102400

		# The receive buffer (SO_RCVBUF) used by the socket server
		socket.receive.buffer.bytes=102400

		# The maximum size of a request that the socket server will accept (protection against OOM)
		socket.request.max.bytes=104857600


		############################# Log Basics #############################

		# A comma seperated list of directories under which to store log files
		log.dirs=/opt/kafka/data/kafka-logs

		# The default number of log partitions per topic. More partitions allow greater
		# parallelism for consumption, but this will also result in more files across
		# the brokers.
		num.partitions=1

		# The number of threads per data directory to be used for log recovery at startup and flushing at shutdown.
		# This value is recommended to be increased for installations with data dirs located in RAID array.
		num.recovery.threads.per.data.dir=1

		############################# Log Flush Policy #############################

		# Messages are immediately written to the filesystem but by default we only fsync() to sync
		# the OS cache lazily. The following configurations control the flush of data to disk. 
		# There are a few important trade-offs here:
		#    1. Durability: Unflushed data may be lost if you are not using replication.
		#    2. Latency: Very large flush intervals may lead to latency spikes when the flush does occur as there will be a lot of data to flush.
		#    3. Throughput: The flush is generally the most expensive operation, and a small flush interval may lead to exceessive seeks. 
		# The settings below allow one to configure the flush policy to flush data after a period of time or
		# every N messages (or both). This can be done globally and overridden on a per-topic basis.

		# The number of messages to accept before forcing a flush of data to disk
		#log.flush.interval.messages=10000

		# The maximum amount of time a message can sit in a log before we force a flush
		#log.flush.interval.ms=1000

		############################# Log Retention Policy #############################

		# The following configurations control the disposal of log segments. The policy can
		# be set to delete segments after a period of time, or after a given size has accumulated.
		# A segment will be deleted whenever *either* of these criteria are met. Deletion always happens
		# from the end of the log.

		# The minimum age of a log file to be eligible for deletion
		log.retention.hours=168

		# A size-based retention policy for logs. Segments are pruned from the log as long as the remaining
		# segments don't drop below log.retention.bytes.
		#log.retention.bytes=1073741824

		# The maximum size of a log segment file. When this size is reached a new log segment will be created.
		log.segment.bytes=1073741824

		# The interval at which log segments are checked to see if they can be deleted according 
		# to the retention policies
		log.retention.check.interval.ms=300000

		# By default the log cleaner is disabled and the log retention policy will default to just delete segments after their retention expires.
		# If log.cleaner.enable=true is set the cleaner will be enabled and individual logs can then be marked for log compaction.
		log.cleaner.enable=false

		############################# Zookeeper #############################

		# Zookeeper connection string (see zookeeper docs for details).
		# This is a comma separated host:port pairs, each corresponding to a zk
		# server. e.g. "127.0.0.1:3000,127.0.0.1:3001,127.0.0.1:3002".
		# You can also append an optional chroot string to the urls to specify the
		# root directory for all kafka znodes.
		zookeeper.connect=node01:2181,node02:2181,node03:2181

		# Timeout in ms for connecting to zookeeper
		zookeeper.connection.timeout.ms=6000



	[root@node1 ~]# scp -r /opt/kafka root@node2:/opt/
	[root@node2 ~]# vim /opt/kafka/config/server.properties 
		修改：
		broker.id=2
		host.name=10.199.200.102

	[root@node1 ~]# scp -r /opt/kafka root@node3:/opt/
	[root@node3 ~]# vim /opt/kafka/config/server.properties 
		修改：
		broker.id=3
		host.name=10.199.200.103

####3.3 启动kafka

	[root@node1 kafka]#  bin/kafka-server-start.sh config/server.properties
	[root@node2 kafka]#  bin/kafka-server-start.sh config/server.properties
	[root@node3 kafka]#  bin/kafka-server-start.sh config/server.properties

####3.4 配置启动脚本

	[root@node1 kafka]# vim /opt/startKafka.sh
		#!/bin/bash
		#pkill -9 kafka
		/opt/kafka/bin/kafka-server-start.sh /opt/kafka/config/server.properties &> /opt/kafka/logs/kafka.logs &
		tail -500f /opt/kafka/logs/kafka.logs

	[root@node2 kafka]# vim /opt/startKafka.sh
		#!/bin/bash
		#pkill -9 kafka
		/opt/kafka/bin/kafka-server-start.sh /opt/kafka/config/server.properties &> /opt/kafka/logs/kafka.logs &
		tail -500f /opt/kafka/logs/kafka.logs

	[root@node3 kafka]# vim /opt/startKafka.sh
		#!/bin/bash
		#pkill -9 kafka
		/opt/kafka/bin/kafka-server-start.sh /opt/kafka/config/server.properties &> /opt/kafka/logs/kafka.logs &
		tail -500f /opt/kafka/logs/kafka.logs

####3.5 测试集群

	#建立一个topic为summer
	[root@node1 ~]# /opt/kafka/bin/kafka-topics.sh --create --zookeeper 10.199.200.101:2181 --replication-factor 3 --partitions 1 --topic summer
	#列出集群中所有的topic
	[root@node1 ~]# /opt/kafka/bin/kafka-topics.sh --list --zookeeper 10.199.200.102:2181 
		summer
	#查看summer这个主题的详情
	[root@node1 ~]# /opt/kafka/bin/kafka-topics.sh --describe --zookeeper 10.199.200.103:2181 --topic summer 
		Topic:summer	PartitionCount:1	ReplicationFactor:3	Configs:
		Topic: summer	Partition: 0	Leader: 2	Replicas: 2,3,1	Isr: 2,3,1
	注：
	#Topic主题名称：summer

	#Partition:只有一个，从0开始

	#leader ：id为2的broker

	#Replicas 副本存在于broker id为2,3,4的上面

	#Isr:活跃状态的broker
	
	使用node1发送一条消息，这里的node1为生产者角色
	[root@elkkafak01 ~]# /opt/kafka/bin/kafka-console-producer.sh --broker-list 10.199.200.101:2181 --topic summer
		[2016-08-16 17:29:07,225] WARN Property topic is not valid (kafka.utils.VerifiableProperties)
		tis
		kisyys
		paut
	
	使用node3接收消息
	[root@node3 ~]# /opt/kafka/bin/kafka-console-consumer.sh --zookeeper  10.199.200.103:2181 --topic summer --from-beginning
		tis
		kisyys
		paut

###4. kafka常用命令	
	
	1.列出集群中所有的topic
		/opt/kafka/bin/kafka-topics.sh --list --zookeeper 10.199.200.102:2181 
		
	2.创建TOPIC kafka-create-topic.sh
		bin/kafka-topic.sh   --replica 2 --partition 8 --topic summer  --zookeeper 10.199.200.101:2181
		创建名为test的topic， 8个分区分别存放数据，数据备份总共2份
	
	3.重新分配分区kafka-reassign-partitions.sh
	这个命令可以分区指定到想要的--broker-list上，broker-list是指节点id列表
		1.json文件内容：
		
		{  
			  
			"topics":  
			  
			[  
			  
			{  
			  
			"topic": "summer"  
			  
			}  
			  
			],  
			  
			"version":1  
  
		}
		./bin/kafka-reassign-partitions.sh --zookeeper 10.199.200.101:2181 --topics-to-move-json-file  1.json  --broker-list  "3,4,5"  --generate  
		这表示如果我要把summer迁移到3,4,5节点上kafka-reassign-partitions.sh所要出的方案，执行成功会显示两段json，一段为原来的json（可以用来备份数据），一段为迁移后的json(用来迁移)，然后把迁移的json脚本保存为2.json
		执行迁移任务：
		bin/kafka-reassign-partitions.sh --zookeeper 10.199.200.101:2181 --reassignment-json-file 2.json --execute 
		查询迁移状态：
		bin/kafka-reassign-partitions.sh --zookeeper 10.199.200.101:2181 --reassignment-json-file 2.json --verify 
		
	4.为Topic增加 partition数目kafka-add-partitions.sh
	
		./kafka-topics.sh -alter --topic test --partition 2  --zookeeper  192.168.197.170:2181,192.168.197.171:2181 （为topic test增加2个分区）
		
	
## 结束