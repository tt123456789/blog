
+ Title: kafka��Ⱥ
+ Date: 2017/5/22 17:07:59  
+ Modified: 2017/5/22 17:07:59  
+ Category: Servers
+ Tags: kafka
+ Slug: 
+ Author: Temp



##���

&#160; &#160; &#160; &#160;Kafka��һ�ָ��������ķֲ�ʽ����������Ϣϵͳ�������Դ��������߹�ģ����վ�е����ж��������ݡ� ���ֶ�������ҳ����������������û����ж��������ִ������ϵ������Ṧ�ܵ�һ���ؼ����ء� ��Щ����ͨ����������������Ҫ���ͨ��������־����־�ۺ�������� ������Hadoop��һ������־���ݺ����߷���ϵͳ������Ҫ��ʵʱ��������ƣ�����һ�����еĽ��������Kafka��Ŀ����ͨ��Hadoop�Ĳ��м��ػ�����ͳһ���Ϻ����ߵ���Ϣ����Ҳ��Ϊ��ͨ����Ⱥ�����ṩʵʱ�����ѡ�




##˵��

&#160; &#160; &#160; &#160;���׻���������3zookeeper+kafka�ķ�ʽ����

		

##����


+ ����IP��172.17.117.116-172.17.117.121
+ ��������TLSKMQ01-TLSKMQ06
+ �������ã�4CPU 8G 100G
+ ϵͳ�汾��Redhat6.5

##�����

		# yum install wget vim net-tools sysstat telnet bash-com*
		# yum localinstall jdk-7u67-linux-x64.rpm	
		# zookeeper-3.4.10.tar.gz 
		# kafka_2.12-0.10.2.0.tgz
##��װ

###1.������ʼ����

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
		
###2.��װzookeeper��Ⱥ

   zookeeperҪ������TLSKMQ01-TLSKMQ06�ϰ�װ��������ʾһ̨��װ������
   
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
	
		
###3.��װkafka

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

##�Ż�

###1.zookeeper�Ż�
	
	1.1. �����ļ���������־�ļ��ֱ���ڲ�ͬ���̡�zoo.cfg�ļ��У�dataDir�Ǵ�ſ������ݵģ�dataLogDir�Ǵ��������־�ġ�zookeeper���²������̣���д������־����д�ڴ棬�������䵽���̣�ˢ���ڴ浽�����ļ�����������־�Ķ�д���������Ӱ��ܴ󣬱�֤dataLogDir���ڴ����������á�û�о�����
	
	1.2. Ĭ��jvmû������Xmx��Xms����Ϣ��������confĿ¼�´���Java.env�ļ����ڴ�ѿռ�һ��ҪС�ڻ����ڴ棬����ʹ��swap��
	
		vim conf/java.env
			JAVA_OPTS="-Dcom.sun.management.jmxremote.local.only=false -Djava.rmi.server.hostname=172.17.117.116 -Dcom.sun.management.jmxremote.port=10001 -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false"
			export JVMFLAGS="-Xms5120m -Xmx5120m $JVMFLAGS $JAVA_OPTS"
	
	1.3. �����zookeeper��־������zookeeper.out�ļ�����
			zkEnv.sh�ļ���־�����ʽ��CONSOLE��ΪROLLINGFILE��
			
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
			
			conf/log4j.properties����Ϊ���������ļ�DailyRollingFileAppender
			
			# vim conf/log4j.properties
				#zookeeper.root.logger=INFO, CONSOLE
				zookeeper.root.logger=INFO, ROLLINGFIL
				
				log4j.appender.ROLLINGFILE=org.apache.log4j.DailyRollingFileAppender
				log4j.appender.ROLLINGFILE.Threshold=${zookeeper.log.threshold}
				log4j.appender.ROLLINGFILE.File=${zookeeper.log.dir}/${zookeeper.log.file}
				log4j.appender.ROLLINGFILE.DatePattern='.'yyyy-MM-dd
				
				# Max log file size of 10MB

	1.4. zoo.cfg�ļ���skipACL=yes������ACL��֤�����Լ���Ȩ����֤����ز���������һ�����ܡ�
	
	1.5. zoo.cfg�ļ���forceSync=no�������д����������������а�������ָÿ��д��������ݶ�Ҫ��pagecache�й̻��������ϣ�������д�ɹ����ء���д������������һ���̶ȵ�ʱ�򣬺���д�����ȴ�ǰ��д�����forceSync���������һ����ʱ�����׷�����ʱ��д��������forceSync=no������д��pagecache��ͷ��ء����ǻ����ϵ��ʱ��pagecache�е������п��ܶ�ʧ��
		Ĭ��ΪforceSync=yes��Ϊyes��������fsync.warningthresholdms=50 ������ݹ̻������̵Ĳ���fsync����50ms��ʱ�򣬽�����zookeeper.out�����һ��warn��־��forceSync=yes��Ч����
	
	1.6. globalOutstandingLimit=100000 �ͻ������ӹ��࣬���ƿͻ������󣬱���OOM
	
	1.7. zoo.cfg�ļ���preAllocSize=64M ��־�ļ�Ԥ�����С; snapCount=100,000 ���ٴ�д��������һ�����������������Ƶ�����ʵ�����ò�����һ��zk��Ӧ���ᳫ������д�����ܽϺã�10:1�����洢Ԫ��������Э���ֲ�ʽ��������һ�¡�д����Ƶ��ʹ�û������
	
	1.8. ��־�ļ��Զ���������׷�����ܣ����ֶ������
			autopurge.snapRetainCount=3 # The number of snapshots to retain in dataDir
			autopurge.purgeInterval=1 # Purge task interval in hours Set to "0" to disable auto purge feature
			
###2.kafka�Ż�
	
####2.1 JVM���������Ż�

&#160; &#160; &#160; &#160;���ʹ�õ�CMS GC�㷨������JVM Heap��Ҫ̫����4GB���ھͿ��ԡ�JVM̫�󣬵���Major GC����Full GC�����ġ�stop the world��ʱ�����������broker��zk֮���session��ʱ����������ѡ��controller�ڵ������follow replicaΪleader replica��
&#160; &#160; &#160; &#160;JVMҲ���ܹ�С������ᵼ��Ƶ���ش���gc������ҲӰ��Kafka�������������⣬��Ҫ����CMS GC�����еķ���promotion failure��concurrent failure���⡣CMSInitiatingOccupancyFraction=70����Ԥ��concurrent failure���⣬��ǰ����Major GC��
&#160; &#160; &#160; &#160;Kafka JVM��������ֱ���޸������ű�bin/kafka-server-start.sh �еı���ֵ��������һЩ����������Ҳ���Ը���ʵ�ʵ�gc״���͵���GC��Ҫ����һЩ��صĲ�����

			export KAFKA_HEAP_OPTS="-Xmx4G -Xms4G -Xmn2G -XX:PermSize=64m -XX:MaxPermSize=128m  -XX:SurvivorRatio=6  -XX:CMSInitiatingOccupancyFraction=70 -XX:+UseCMSInitiatingOccupancyOnly"
			
&#160; &#160; &#160; &#160;��Ҫ��עgc��־�е�YGCʱ���Լ�CMS GC�����CMS-initial-mark��CMS-remark�����׶ε�ʱ�䣬��ЩGC�����ǡ�stop the world����ʽ��ɵġ�

####2.2 ��JMX�˿�

&#160; &#160; &#160; &#160;��Ҫ��Ϊ��ͨ��JMX�˿ڼ��Kafka Broker��Ϣ��������bin/kafka-server-start.sh�д�JMX�˿ڱ�����

		export JMX_PORT=9999
		
####2.3 ����log4j����־����

&#160; &#160; &#160; &#160;�����Ⱥ��topic��partition�����ϴ�ʱ����Ϊlog4j����־����̫�ͣ����½��̳����ܳ���ʱ���ڴ�ӡ��־����־���޴󣬵��ºܶ��������ܿ������ر���contoller��־����Ϊtrace�������ȽϿӡ�

&#160; &#160; &#160; &#160;Tipsͨ��JMX�˿�����log4j��־���𣬲�������broker�ڵ�

		������־����
		java -jar cmdline-jmxclient-0.10.3.jar - localhost:9999 kafka:type=kafka.Log4jController setLogLevel=kafka.controller,INFO
		java -jar cmdline-jmxclient-0.10.3.jar - localhost:9999 kafka:type=kafka.Log4jController setLogLevel=state.change.logger,INFO

		�����־����
		java -jar cmdline-jmxclient-0.10.3.jar - localhost:9999 kafka:type=kafka.Log4jController getLogLevel=kafka.controller
		java -jar cmdline-jmxclient-0.10.3.jar - localhost:9999 kafka:type=kafka.Log4jController 	
