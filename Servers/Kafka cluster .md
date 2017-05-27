
+ Title: kafka集群
+ Date: 2017/5/22 17:07:59  
+ Modified: 2017/5/22 17:07:59  
+ Category: Servers
+ Tags: kafka
+ Slug: 
+ Author: Temp



##简介

&#160; &#160; &#160; &#160;Kafka是一种高吞吐量的分布式发布订阅消息系统，它可以处理消费者规模的网站中的所有动作流数据。 这种动作（网页浏览，搜索和其他用户的行动）是在现代网络上的许多社会功能的一个关键因素。 这些数据通常是由于吞吐量的要求而通过处理日志和日志聚合来解决。 对于像Hadoop的一样的日志数据和离线分析系统，但又要求实时处理的限制，这是一个可行的解决方案。Kafka的目的是通过Hadoop的并行加载机制来统一线上和离线的消息处理，也是为了通过集群机来提供实时的消费。




##说明

&#160; &#160; &#160; &#160;这套环境尝试以3zookeeper+kafka的方式部署

		

##环境


+ 主机IP：172.17.117.116-172.17.117.121
+ 主机名：TLSKMQ01-TLSKMQ06
+ 主机配置：4CPU 8G 100G
+ 系统版本：Redhat6.5

##软件包

		# yum install wget vim net-tools sysstat telnet bash-com*
		# yum localinstall jdk-7u67-linux-x64.rpm	
		# zookeeper-3.4.10.tar.gz 
		# kafka_2.12-0.10.2.0.tgz
##安装

###1.主机初始配置

		# yum install wget vim net-tools sysstat telnet bash-com*
		# vim /etc/hosts
			127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
			::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
			172.17.117.116   TLSKMQ01
			172.17.117.117   TLSKMQ02
			172.17.117.118   TLSKMQ03
			172.17.117.119	 TLSKMQ04
			172.17.117.120	 TLSKMQ05
			172.17.117.121	 TLSKMQ06
		# yum localinstall jdk-7u67-linux-x64.rpm
		
###2.安装zookeeper集群

   zookeeper要在主机TLSKMQ01-TLSKMQ06上安装，下面演示一台安装方法：
   
	[root@TLSKMQ01 ~]# tar xf zookeeper-3.4.10.tar.gz 
	[root@TLSKMQ01 ~]# cd zookeeper-3.4.10/
	[root@TLSKMQ01 zookeeper-3.4.10]# cp conf/zoo_sample.cfg conf/zoo.cfg 
	[root@TLSKMQ01 zookeeper-3.4.10]# vim conf/zoo.cfg
		tickTime=2000
		initLimit=10
		syncLimit=5
		dataDir=/ane/zookeeper/data
		dataLogDir=/ane/zookeeper/logdata
		clientPort=2181
		server.1 = TLSKMQ01:2888:3888
		server.2 = TLSKMQ02:2888:3888
		server.3 = TLSKMQ03:2888:3888		
		server.4 = TLSKMQ04:2888:3888
		server.5 = TLSKMQ05:2888:3888
		server.6 = TLSKMQ06:2888:3888	

	[root@TLSKMQ01 ~]# mv zookeeper-3.4.10 /ane/zookeeper
	[root@TLSKMQ01 ~]# mkdir data
	[root@TLSKMQ01 ~]# vim /ane/zookeeper/data/myid
		1	
	[root@TLSKMQ01 ~]# chown -R ane.ane /ane
	[root@TLSKMQ01 ~]# su ane
	[ane@TLSKMQ01 root]# cd /ane
	[root@TLSKMQ01 ~]$ /ane/zookeeper/bin/zkServer.sh start
	
		
###3.安装kafka

	[root@TLSKMQ01 ~]# tar xf kafka_2.12-0.10.2.0.tgz
	[root@TLSKMQ01 ~]# mv kafka_2.12-0.10.2.0 /ane/kafka
	[root@TLSKMQ01 ~]# chown -R ane.ane /ane
	[root@TLSKMQ01 ~]# su ane
	[ane@TLSKMQ01 root]# cd /ane
	[ane@TLSKMQ01 ~]$ vim kafka/config/server.properties 
		broker.id=0
		port=9090
		host.name=172.17.117.116
		num.network.threads=6
		num.io.threads=16
		socket.send.buffer.bytes=102400
		socket.receive.buffer.bytes=102400
		socket.request.max.bytes=104857600
		log.dirs=/ane/kafka/data/kafka0-logs
		num.partitions=64
		num.recovery.threads.per.data.dir=1
		#log.flush.interval.messages=10000
		#log.flush.interval.ms=1000
		log.retention.hours=168
		#log.retention.bytes=1073741824
		log.segment.bytes=1073741824
		log.retention.check.interval.ms=300000
		log.cleaner.enable=false
		zookeeper.connect=172.17.117.116:2181,172.17.117.117:2181,172.17.117.118:2181,172.17.117.119:2181,172.17.117.120:2181
		zookeeper.connection.timeout.ms=6000
	[ane@TLSKMQ01 ~]$ vim kafka/start.sh
		#!/bin/bash
		export KAFKA_HEAP_OPTS="-Xmx4G -Xms4G -Xmn2G -XX:PermSize=64m -XX:MaxPermSize=128m  -XX:SurvivorRatio=6  -XX:CMSInitiatingOccupancyFraction=70 -XX:+UseCMSInitiatingOccupancyOnly"
		export JMX_PORT=10002
		pkill -15 -f kafka
		/ane/kafka/bin/kafka-server-start.sh /ane/kafka/config/server.properties &> /ane/kafka/logs/kafka.logs &
		tail -500f /ane/kafka/logs/kafka.logs

##优化

###1.zookeeper优化
	
	1.1. 快照文件和事务日志文件分别挂在不同磁盘。zoo.cfg文件中，dataDir是存放快照数据的，dataLogDir是存放事务日志的。zookeeper更新操作过程：先写事务日志，再写内存，周期性落到磁盘（刷新内存到快照文件）。事务日志的对写请求的性能影响很大，保证dataLogDir所在磁盘性能良好、没有竞争者
	
	1.2. 默认jvm没有配置Xmx、Xms等信息，可以在conf目录下创建Java.env文件（内存堆空间一定要小于机器内存，避免使用swap）
	
		vim conf/java.env
			JAVA_OPTS="-Dcom.sun.management.jmxremote.local.only=false -Djava.rmi.server.hostname=172.17.117.116 -Dcom.sun.management.jmxremote.port=10001 -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false"
			export JVMFLAGS="-Xms5120m -Xmx5120m $JVMFLAGS $JAVA_OPTS"
	
	1.3. 按天出zookeeper日志，避免zookeeper.out文件过大。
			zkEnv.sh文件日志输出方式从CONSOLE改为ROLLINGFILE；
			
			# vim zkEnv.sh
				if [ "x${ZOO_LOG_DIR}" = "x" ]
				then
					#ZOO_LOG_DIR="."
					ZOO_LOG_DIR="${ZOOKEEPER_PREFIX}/logs"
				fi

				if [ "x${ZOO_LOG4J_PROP}" = "x" ]
				then
					#ZOO_LOG4J_PROP="INFO,CONSOLE"
					ZOO_LOG4J_PROP="INFO,ROLLINGFILE"
				fi
			
			conf/log4j.properties设置为按天生成文件DailyRollingFileAppender
			
			# vim conf/log4j.properties
				#zookeeper.root.logger=INFO, CONSOLE
				zookeeper.root.logger=INFO, ROLLINGFIL
				
				log4j.appender.ROLLINGFILE=org.apache.log4j.DailyRollingFileAppender
				log4j.appender.ROLLINGFILE.Threshold=${zookeeper.log.threshold}
				log4j.appender.ROLLINGFILE.File=${zookeeper.log.dir}/${zookeeper.log.file}
				log4j.appender.ROLLINGFILE.DatePattern='.'yyyy-MM-dd
				
				# Max log file size of 10MB

	1.4. zoo.cfg文件中skipACL=yes，忽略ACL验证，可以减少权限验证的相关操作，提升一点性能。
	
	1.5. zoo.cfg文件中forceSync=no，这个对写请求的性能提升很有帮助，是指每次写请求的数据都要从pagecache中固化到磁盘上，才算是写成功返回。当写请求数量到达一定程度的时候，后续写请求会等待前面写请求的forceSync操作，造成一定延时。如果追求低延时的写请求，配置forceSync=no，数据写到pagecache后就返回。但是机器断电的时候，pagecache中的数据有可能丢失。
		默认为forceSync=yes，为yes可以设置fsync.warningthresholdms=50 如果数据固化到磁盘的操作fsync超过50ms的时候，将会在zookeeper.out中输出一条warn日志（forceSync=yes有效）。
	
	1.6. globalOutstandingLimit=100000 客户端连接过多，限制客户端请求，避免OOM
	
	1.7. zoo.cfg文件中preAllocSize=64M 日志文件预分配大小; snapCount=100,000 多少次写事务，生成一个快照如果快照生成频繁可适当调大该参数。一般zk的应用提倡读大于写，性能较好（10:1），存储元数据用来协调分布式数据最终一致。写过于频繁使用缓存更好
	
	1.8. 日志文件自动清除（如果追求性能，可手动清除）
			autopurge.snapRetainCount=3 # The number of snapshots to retain in dataDir
			autopurge.purgeInterval=1 # Purge task interval in hours Set to "0" to disable auto purge feature
			
###2.kafka优化
	
####2.1 JVM参数配置优化

&#160; &#160; &#160; &#160;如果使用的CMS GC算法，建议JVM Heap不要太大，在4GB以内就可以。JVM太大，导致Major GC或者Full GC产生的“stop the world”时间过长，导致broker和zk之间的session超时，比如重现选举controller节点和提升follow replica为leader replica。
&#160; &#160; &#160; &#160;JVM也不能过小，否则会导致频繁地触发gc操作，也影响Kafka的吞吐量。另外，需要避免CMS GC过程中的发生promotion failure和concurrent failure问题。CMSInitiatingOccupancyFraction=70可以预防concurrent failure问题，提前出发Major GC。
&#160; &#160; &#160; &#160;Kafka JVM参数可以直接修改启动脚本bin/kafka-server-start.sh 中的变量值。下面是一些基本参数，也可以根据实际的gc状况和调试GC需要增加一些相关的参数。

			export KAFKA_HEAP_OPTS="-Xmx4G -Xms4G -Xmn2G -XX:PermSize=64m -XX:MaxPermSize=128m  -XX:SurvivorRatio=6  -XX:CMSInitiatingOccupancyFraction=70 -XX:+UseCMSInitiatingOccupancyOnly"
			
&#160; &#160; &#160; &#160;需要关注gc日志中的YGC时间以及CMS GC里面的CMS-initial-mark和CMS-remark两个阶段的时间，这些GC过程是“stop the world”方式完成的。

####2.2 打开JMX端口

&#160; &#160; &#160; &#160;主要是为了通过JMX端口监控Kafka Broker信息。可以在bin/kafka-server-start.sh中打开JMX端口变量。

		export JMX_PORT=9999
		
####2.3 调整log4j的日志级别

&#160; &#160; &#160; &#160;如果集群中topic和partition数量较大时，因为log4j的日志级别太低，导致进程持续很长的时间在打印日志。日志量巨大，导致很多额外的性能开销。特别是contoller日志级别为trace级别，这点比较坑。

&#160; &#160; &#160; &#160;Tips通过JMX端口设置log4j日志级别，不用重启broker节点

		设置日志级别：
		java -jar cmdline-jmxclient-0.10.3.jar - localhost:9999 kafka:type=kafka.Log4jController setLogLevel=kafka.controller,INFO
		java -jar cmdline-jmxclient-0.10.3.jar - localhost:9999 kafka:type=kafka.Log4jController setLogLevel=state.change.logger,INFO

		检查日志级别：
		java -jar cmdline-jmxclient-0.10.3.jar - localhost:9999 kafka:type=kafka.Log4jController getLogLevel=kafka.controller
		java -jar cmdline-jmxclient-0.10.3.jar - localhost:9999 kafka:type=kafka.Log4jController 	
