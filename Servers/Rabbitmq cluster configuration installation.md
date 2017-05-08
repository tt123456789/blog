Title: Rabbitmq集群配置安装
Date: 9:38 2017/3/17
Modified: 9:38 2017/3/17
Category: Servers
Tags: rabbitmq
Slug: 
Author: allposs


## 简介
&#160; &#160; &#160; &#160;通过 Erlang 的分布式特性（magic cookie 认证节点）进行 RabbitMQ 集群，各 RabbitMQ 服务为对等节点，即每个节点都提供服务给客户端连接，进行消息发送与接收。
&#160; &#160; &#160; &#160;这些节点通过 RabbitMQ HA 队列（镜像队列）进行消息队列结构复制。本文中搭建 3 个节点，并且都是磁盘节点（所有节点状态保持一致，节点完全对等），只要有任何一个节点能够工作，RabbitMQ 集群对外就能提供服务。
## 环境
+ 操作系统：CentOS 7.2 x64
+ Yum源：163源
+ IP地址：
			7Node1 10.199.200.101
			7Node2 10.199.200.102
			7Node3 10.199.200.103
+ DNS：
+ 主机名：
			7Node1.example.com
			7Node2.example.com
			7Node3.example.com

## 软件包

###1. Yum源
	
	[root@7Node* ~]# wget http://www.rabbitmq.com/releases/rabbitmq-server/v3.6.7/rabbitmq-server-3.6.7-1.el7.noarch.rpm

###2. 源码包
	
	[root@7Node* ~]# wget http://www.rabbitmq.com/releases/rabbitmq-server/v3.6.7/rabbitmq-server-3.6.7.tar.xz
##拓扑图



## 正文
###1. 准备配置（所有节点）
	[root@7Node* ~]# wget http://www.rabbitmq.com/releases/rabbitmq-server/v3.6.7/rabbitmq-server-3.6.7-1.el7.noarch.rpm
	[root@node* ~]# vim /etc/hosts
	
		127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
		::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
		10.199.200.101   7Node1 node1.example.com
		10.199.200.102   7Node2 node2.example.com
		10.199.200.103   7Node3 node3.example.com
	[root@7Node* ~]# vi /etc/sysctl.conf
		fs.file-max = 100000
	[root@7Node* ~]# vi /etc/security/limits.conf
		*              soft     nofile          65536
		*              hard     nofile          65536
	[root@7Node* ~]# sysctl -p
###2. 安装Rabbitmq单节点

####2.1 安装配置Rabbitmq
	[root@7Node1 ~]# wget http://www.rabbitmq.com/releases/rabbitmq-server/v3.6.7/rabbitmq-server-3.6.7-1.el7.noarch.rpm
	[root@7Node1 ~]# wget http://www.rabbitmq.com/releases/erlang/erlang-19.0.4-1.el7.centos.x86_64.rpm
	[root@7Node1 ~]# yum install erlang-19.0.4-1.el7.centos.x86_64.rpm rabbitmq-server-3.6.7-1.el7.noarch.rpm 
	[root@7Node1 ~]# firewall-cmd --permanent --add-port=15672/tcp
	[root@7Node1 ~]# firewall-cmd --permanent --add-port=25672/tcp
	[root@7Node1 ~]# firewall-cmd --permanent --add-port=5672/tcp
	[root@7Node1 ~]# firewall-cmd --reload
	[root@7Node1 ~]# systemctl enable rabbitmq-server.service 
	[root@7Node1 ~]# systemctl start rabbitmq-server.service
	[root@7Node1 ~]# rabbitmq-plugins enable rabbitmq_management
		The following plugins have been enabled:
		  amqp_client
		  cowlib
		  cowboy
		  rabbitmq_web_dispatch
		  rabbitmq_management_agent
		  rabbitmq_management

		Applying plugin configuration to rabbit@7Node1... started 6 plugins.
####2.2 RabbitMQ用户管理
	添加用户（用户名admin，密码admin）
	[root@7Node1 ~]# rabbitmqctl add_user admin admin
		Creating user "admin" ...

	设置用户角色（设置admin用户为管理员角色）
	[root@7Node1 ~]# rabbitmqctl set_user_tags admin administrator
		Setting tags for user "admin" to [administrator] ...

	设置用户权限（设置admin用户配置、写、读的权限）
	[root@7Node1 ~]# rabbitmqctl set_permissions -p / admin ".*" ".*" ".*"
		Setting permissions for user "admin" in vhost "/" ...

	删除用户（删除guest用户）
	[root@7Node1 ~]# rabbitmqctl delete_user guest
		Deleting user "guest" ...

	注意：rabbitmq从3.3.0开始禁止使用guest/guest权限通过除localhost外的访问。

	如果想使用guest/guest通过远程机器访问，需要在rabbitmq配置文件中(/etc/rabbitmq/rabbitmq.config)中设置loopback_users为[]。
	[{rabbit, [{loopback_users, []}]}].
###2.3 RabbitMQ监控
	[root@7Node1 ~]# rabbitmqctl status
		Status of node rabbit@7Node1 ...
		[{pid,2456},
		 {running_applications,
			 [{rabbitmq_management,"RabbitMQ Management Console","3.6.7"},
			  {rabbitmq_web_dispatch,"RabbitMQ Web Dispatcher","3.6.7"},
			  {rabbitmq_management_agent,"RabbitMQ Management Agent","3.6.7"},
			  {amqp_client,"RabbitMQ AMQP Client","3.6.7"},
			  {cowboy,"Small, fast, modular HTTP server.","1.0.3"},
			  {ssl,"Erlang/OTP SSL application","8.0.1"},
			  {public_key,"Public key infrastructure","1.2"},
			  {inets,"INETS  CXC 138 49","6.3.2"},
			  {asn1,"The Erlang ASN1 compiler version 4.0.3","4.0.3"},
			  {cowlib,"Support library for manipulating Web protocols.","1.0.1"},
			  {crypto,"CRYPTO","3.7"},
			  {rabbit,"RabbitMQ","3.6.7"},
			  {mnesia,"MNESIA  CXC 138 12","4.14"},
			  {rabbit_common,
				  "Modules shared by rabbitmq-server and rabbitmq-erlang-client",
				  "3.6.7"},
			  {os_mon,"CPO  CXC 138 46","2.4.1"},
			  {xmerl,"XML parser","1.3.11"},
			  {ranch,"Socket acceptor pool for TCP protocols.","1.2.1"},
			  {compiler,"ERTS  CXC 138 10","7.0.1"},
			  {syntax_tools,"Syntax tools","2.0"},
			  {sasl,"SASL  CXC 138 11","3.0"},
			  {stdlib,"ERTS  CXC 138 10","3.0.1"},
			  {kernel,"ERTS  CXC 138 10","5.0.1"}]},
		 {os,{unix,linux}},
		 {erlang_version,
			 "Erlang/OTP 19 [erts-8.0.3] [source] [64-bit] [async-threads:64] [hipe] [kernel-poll:true]\n"},
		 {memory,
			 [{total,63875560},
			  {connection_readers,0},
			  {connection_writers,0},
			  {connection_channels,0},
			  {connection_other,2688},
			  {queue_procs,2688},
			  {queue_slave_procs,0},
			  {plugins,1012968},
			  {other_proc,26185000},
			  {mnesia,58576},
			  {metrics,51688},
			  {mgmt_db,342144},
			  {msg_index,39624},
			  {other_ets,2322304},
			  {binary,879560},
			  {code,24521005},
			  {atom,1033401},
			  {other_system,7472914}]},
		 {alarms,[]},
		 {listeners,[{clustering,25672,"::"},{amqp,5672,"::"},{http,15672,"::"}]},
		 {vm_memory_high_watermark,0.4},
		 {vm_memory_limit,409573785},
		 {disk_free_limit,50000000},
		 {disk_free,27631472640},
		 {file_descriptors,
			 [{total_limit,924},{total_used,2},{sockets_limit,829},{sockets_used,0}]},
		 {processes,[{limit,1048576},{used,322}]},
		 {run_queue,0},
		 {uptime,630},
		 {kernel,{net_ticktime,60}}]
		 	 
		[root@7Node1 ~]# rabbitmq-plugins list
			 Configured: E = explicitly enabled; e = implicitly enabled
			 | Status:   * = running on rabbit@7Node1
			 |/
			[e*] amqp_client                       3.6.7
			[e*] cowboy                            1.0.3
			[e*] cowlib                            1.0.1
			[  ] rabbitmq_amqp1_0                  3.6.7
			[  ] rabbitmq_auth_backend_ldap        3.6.7
			[  ] rabbitmq_auth_mechanism_ssl       3.6.7
			[  ] rabbitmq_consistent_hash_exchange 3.6.7
			[  ] rabbitmq_event_exchange           3.6.7
			[  ] rabbitmq_federation               3.6.7
			[  ] rabbitmq_federation_management    3.6.7
			[  ] rabbitmq_jms_topic_exchange       3.6.7
			[E*] rabbitmq_management               3.6.7
			[e*] rabbitmq_management_agent         3.6.7
			[  ] rabbitmq_management_visualiser    3.6.7
			[  ] rabbitmq_mqtt                     3.6.7
			[  ] rabbitmq_recent_history_exchange  3.6.7
			[  ] rabbitmq_sharding                 3.6.7
			[  ] rabbitmq_shovel                   3.6.7
			[  ] rabbitmq_shovel_management        3.6.7
			[  ] rabbitmq_stomp                    3.6.7
			[  ] rabbitmq_top                      3.6.7
			[  ] rabbitmq_tracing                  3.6.7
			[  ] rabbitmq_trust_store              3.6.7
			[e*] rabbitmq_web_dispatch             3.6.7
			[  ] rabbitmq_web_mqtt                 3.6.7
			[  ] rabbitmq_web_mqtt_examples        3.6.7
			[  ] rabbitmq_web_stomp                3.6.7
			[  ] rabbitmq_web_stomp_examples       3.6.7
			[  ] sockjs                            0.3.4

###2.4 RabbitMQ默认配置
	[root@7Node1 ~]# cat /usr/lib/rabbitmq/lib/rabbitmq_server-3.6.7/sbin/rabbitmq-defaults 		 
		#!/bin/sh -e
		##  The contents of this file are subject to the Mozilla Public License
		##  Version 1.1 (the "License"); you may not use this file except in
		##  compliance with the License. You may obtain a copy of the License
		##  at http://www.mozilla.org/MPL/
		##
		##  Software distributed under the License is distributed on an "AS IS"
		##  basis, WITHOUT WARRANTY OF ANY KIND, either express or implied. See
		##  the License for the specific language governing rights and
		##  limitations under the License.
		##
		##  The Original Code is RabbitMQ.
		##
		##  The Initial Developer of the Original Code is GoPivotal, Inc.
		##  Copyright (c) 2012-2015 Pivotal Software, Inc.  All rights reserved.
		##

		### next line potentially updated in package install steps
		SYS_PREFIX=

		### next line will be updated when generating a standalone release
		ERL_DIR=

		CLEAN_BOOT_FILE=start_clean
		SASL_BOOT_FILE=start_sasl

		if [ -f "${RABBITMQ_HOME}/erlang.mk" ]; then
			# RabbitMQ is executed from its source directory. The plugins
			# directory and ERL_LIBS are tuned based on this.
			RABBITMQ_DEV_ENV=1
		fi

		## Set default values

		BOOT_MODULE="rabbit"

		CONFIG_FILE=${SYS_PREFIX}/etc/rabbitmq/rabbitmq
		LOG_BASE=${SYS_PREFIX}/var/log/rabbitmq
		MNESIA_BASE=${SYS_PREFIX}/var/lib/rabbitmq/mnesia
		ENABLED_PLUGINS_FILE=${SYS_PREFIX}/etc/rabbitmq/enabled_plugins

		从上面看出：

		- 系统prefix是空
		- 配置文件路径是 /etc/rabbitmq/rabbitmq.config    (erlang会自动加上.config后缀)
		- 环境配置文件是 /etc/rabbitmq/rabbitmq-env.conf
		- 日志文件目录是 /var/log/rabbitmq
		- 插件文件目录是 安装目录下的plugins，这里RPM安装方式下是 /usr/lib/rabbitmq/lib/rabbitmq_server-3.6.7/plugins

### 3. RabbitMQ集群安装配置	
	集群配置在单机配置完成的基础上进行，所以单机配置每台都要操作。
#### 3.1 设置每个节点Cookie
	Rabbitmq的集群是依赖于erlang的集群来工作的，所以必须先构建起erlang的集群环境。Erlang的集群中各节点是通过一个magic cookie来实现的，这个cookie存放在 /var/lib/rabbitmq/.erlang.cookie 中，文件是400的权限。所以必须保证各节点cookie保持一致，否则节点之间就无法通信。建议在RabbitMQ服务启动前修改过cookie，如果RabbitMQ服务已经启动，修改cookie值后，必须重启RabbitMQ服务，这步很关键
	[root@7Node1 ~]# cat /var/lib/rabbitmq/.erlang.cookie
		FUYGVRAANZSYOEOIXPEK
	[root@7Node1 ~]# scp /var/lib/rabbitmq/.erlang.cookie root@7Node2:/var/lib/rabbitmq/
	[root@7Node2 ~]# cat /var/lib/rabbitmq/.erlang.cookie 
		FUYGVRAANZSYOEOIXPEK
	[root@7Node2 ~]# chown rabbitmq. /var/lib/rabbitmq/.erlang.cookie 
	[root@7Node2 ~]# chcon -u system_u /var/lib/rabbitmq/.erlang.cookie 
	[root@7Node1 ~]# scp /var/lib/rabbitmq/.erlang.cookie root@7Node3:/var/lib/rabbitmq/
	[root@7Node3 ~]# cat /var/lib/rabbitmq/.erlang.cookie 
		FUYGVRAANZSYOEOIXPEK
	[root@7Node3 ~]# chown rabbitmq. /var/lib/rabbitmq/.erlang.cookie 
	[root@7Node3 ~]# chcon -u system_u /var/lib/rabbitmq/.erlang.cookie 
	[root@7Node* ~]#  firewall-cmd --permanent --add-port={4369/tcp,25672/tcp}
	[root@7Node* ~]#  firewall-cmd --reload
	[root@7Node* ~]# systemctl start rabbitmq-server
#### 3.2 节点加入集群
	将 7Node1、7Node2 、7Node3组成集群：
		默认是磁盘节点，如果是内存节点的话，需要加--ram参数
	7Node2节点
	[root@7Node2 ~]# rabbitmqctl stop_app
		Stopping rabbit application on node rabbit@7Node2 ...
	[root@7Node2 ~]# rabbitmqctl join_cluster rabbit@7Node1
		Clustering node rabbit@7Node2 with rabbit@7Node1 ...
	[root@7Node2 ~]# rabbitmqctl start_app
		Starting node rabbit@7Node2 ...
	7Node3节点
	[root@7Node3 ~]# rabbitmqctl stop_app
		Stopping rabbit application on node rabbit@7Node3 ...
	[root@7Node3 ~]# rabbitmqctl join_cluster rabbit@7Node1
		Clustering node rabbit@7Node3 with rabbit@7Node1 ...
	[root@7Node3 ~]# rabbitmqctl start_app
		Starting node rabbit@7Node3 ...
	设置镜像策略
	[root@7Node* ~]#   rabbitmqctl set_policy ha-all "^" '{"ha-mode":"all","ha-sync-mode":"automatic"}'
	将所有队列设置为镜像队列，即队列会被复制到各个节点，各个节点状态保持一直。
	启用RabbitMQ监控插件
	cluster搭建起来后若在web管理工具中rabbitmq_management的Overview的Nodes部分看到"Node statistics not available"的信息，说明在该节点上web管理插件还未启用。
	[root@node* ~]# rabbitmq-plugins enable rabbitmq_management
	集群配置好后，可以在 RabbitMQ 任意节点上执行下面的命令来查看是否集群配置成功。
	
## 结束