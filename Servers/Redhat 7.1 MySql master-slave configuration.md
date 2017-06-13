Title: Redhat 7.1 MySql主从配置
Date: 2016/5/14 10:48:41 
Modified: 2016/5/14 10:48:43 
Category: Servers
Tags: Mysql
Slug: 
Author: allposs

## 简介##
&#160; &#160; &#160; &#160;MySQL[1]  是一个关系型数据库管理系统，由瑞典 MySQL AB 公司开发，目前属于 Oracle 旗下公司。MySQL 最流行的关系型数据库管理系统，在 WEB 应用方面 MySQL 是最好的 RDBMS (Relational Database Management System，关系数据库管理系统) 应用软件之一。MySQL 是一种关联数据库管理系统，关联数据库将数据保存在不同的表中，而不是将所有数据放在一个大仓库内，这样就增加了速度并提高了灵活性。MySQL 所使用的 SQL 语言是用于访问数据库的最常用标准化语言。MySQL 软件采用了双授权政策（本词条“授权政策”），它分为社区版和商业版，由于其体积小、速度快、总体拥有成本低，尤其是开放源码这一特点，一般中小型网站的开发都选择 MySQL 作为网站数据库。由于其社区版的性能卓越，搭配 PHP 和 Apache 可组成良好的开发环境。

## 环境##

+ 操作系统：redhat7.1 X86_64
+ Yum源：163源
+ IP地址： 10.199.200.201
          10.199.200.202
+ DNS： 10.199.200.15
+ 主机名： node1
		  node2

## 软件包##

###1. Yum源###

###2. 源码包###
	[root@node1 ~]# wget http://10.199.200.15/source/mysql-5.6.29-linux-glibc2.5-x86_64.tar.gz
	[root@node2 ~]# wget http://10.199.200.15/source/mysql-5.6.29-linux-glibc2.5-x86_64.tar.gz
##拓扑图##
![](http://images.allposs.com/20160302001.png)

## 正文##
###1.node1配置###
####1.node1准备工作####
#####1.1创建data分区#####

	[root@node1 ~]# fdisk /dev/sdb
	Command (m for help): n
	Partition type:
   		p   primary (0 primary, 0 extended, 4 free)
  		e   extended
	Select (default p): p
	Partition number (1-4, default 1): 1
	First sector (2048-62914559, default 2048): 
	Using default value 2048
	Last sector, +sectors or +size{K,M,G} (2048-62914559, default 62914559): 
	Using default value 62914559
	Partition 1 of type Linux and of size 30 GiB is set
	Command (m for help): t
	Hex code (type L to list all codes): 8e
	Changed type of partition 'Linux' to 'Linux LVM'
	Command (m for help): w
	The partition table has been altered!

	[root@node1 ~]# pvcreate /dev/sdb1
	[root@node1 ~]# vgcreate data /dev/sdb2
	[root@node1 ~]# lvcreate -L 10G -n mysqldata data
	[root@node1 ~]# mkfs.ext4 /dev/data/mysqldata 
	[root@node1 ~]# lsblk -f
复制UUID
	[root@node1 ~]# vim /etc/fstab 

写入相关配置

#####1.2配置相关权限#####

	[root@node1 ~]# useradd -M -s /sbin/nologin mysql
	[root@node1 ~]# chown -R mysql.mysql /data/mysql/
	[root@node1 ~]# firewall-cmd --permanent --add-service=mysql
	[root@node1 ~]# firewall-cmd --reload


####2.配置mysql####
#####2.1前提#####

	[root@node1 ~]# wget http://10.199.200.15/source/mysql-5.6.29-linux-glibc2.5-x86_64.tar.gz
	[root@node1 ~]# tar xf mysql-5.6.29-linux-glibc2.5-x86_64.tar.gz 
	[root@node1 ~]# mv mysql-5.6.29-linux-glibc2.5-x86_64/ /usr/local/mysql
	[root@node1 ~]# cd /usr/local/mysql/
	[root@node1 mysql]# chown -R root.mysql ./*
	[root@node1 mysql]# scripts/mysql_install_db --user=mysql --datadir=/data/mysql
	[root@node1 mysql]# cp -f support-files/my-default.cnf /etc/my.cnf
	[root@node1 ~]# vim /etc/my.cnf

#####2.2修改配置文件#####
	
	# Example MySQL config file for large systems.
	#
	# This is for a large system with memory = 512M where the system runs mainly
	# MySQL.
	#
	# MySQL programs look for option files in a set of
	# locations which depend on the deployment platform.
	# You can copy this option file to one of those
	# locations. For information about these locations, see:
	# http://dev.mysql.com/doc/mysql/en/option-files.html
	#
	# In this file, you can use all long options that a program supports.
	# If you want to know which options a program supports, run the program
	# with the "--help" option.

	# The following options will be passed to all MySQL clients
	[client]
	#password       = your_password
	port            = 3306
	socket          = /tmp/mysql.sock

	# Here follows entries for some specific programs
	
	# The MySQL server
	[mysqld]
	port            = 3306
	socket          = /tmp/mysql.sock
	skip-external-locking
	key_buffer_size = 256M
	max_allowed_packet = 1M
	table_open_cache = 256
	sort_buffer_size = 1M
	read_buffer_size = 1M
	read_rnd_buffer_size = 4M
	myisam_sort_buffer_size = 64M
	thread_cache_size = 8
	query_cache_size= 16M
	# Try number of CPU's*2 for thread_concurrency
	thread_concurrency = 8

	datadir = /data/mysql
	#添加data_dir
	sync-binlog = 1
	#同步二进制日志

	# Don't listen on a TCP/IP port at all. This can be a security enhancement,
	# if all processes that need to connect to mysqld run on the same host.
	# All interaction with mysqld must be made via Unix sockets or named pipes.
	# Note that using this option without enabling named pipes on Windows
	# (via the "enable-named-pipe" option) will render mysqld useless!
	# 
	#skip-networking

	# Replication Master Server (default)
	# binary logging is required for replication
	log-bin=master-bin
	#修改二进制日志
	log-bin-index=master-bin.index
	#添加log-bin-index
	# binary logging format - mixed recommended
	binlog_format=mixed

	# required unique id between 1 and 2^32 - 1
	# defaults to 1 if master-host is not set
	# but will not function as a master if omitted
	server-id       = 1
	#注意SERVERID
	# Replication Slave (comment out master section to use this)
	#
	# To configure this host as a replication slave, you can choose between
	# two methods :
	#
	# 1) Use the CHANGE MASTER TO command (fully described in our manual) -
	#    the syntax is:
	#
	#    CHANGE MASTER TO MASTER_HOST=<host>, MASTER_PORT=<port>,
	#    MASTER_USER=<user>, MASTER_PASSWORD=<password> ;
	#
	#    where you replace <host>, <user>, <password> by quoted strings and
	#    <port> by the master's port number (3306 by default).
	#
	#    Example:
	#
	#    CHANGE MASTER TO MASTER_HOST='125.564.12.1', MASTER_PORT=3306,
	#    MASTER_USER='joe', MASTER_PASSWORD='secret';
	#
	# OR
	#
	# 2) Set the variables below. However, in case you choose this method, then
	#    start replication for the first time (even unsuccessfully, for example
	#    if you mistyped the password in master-password and the slave fails to
	#    connect), the slave will create a master.info file, and any later
	#    change in this file to the variables' values below will be ignored and
	#    overridden by the content of the master.info file, unless you shutdown
	#    the slave server, delete master.info and restart the slaver server.
	#    For that reason, you may want to leave the lines below untouched
	#    (commented) and instead use CHANGE MASTER TO (see above)
	#
	# required unique id between 2 and 2^32 - 1
	# (and different from the master)
	# defaults to 2 if master-host is set
	# but will not function as a slave if omitted
	#server-id       = 2
	#
	# The replication master for this slave - required
	#master-host     =   <hostname>
	#
	# The username the slave will use for authentication when connecting
	# to the master - required
	#master-user     =   <username>
	#
	# The password the slave will authenticate with when connecting to
	# the master - required
	#master-password =   <password>
	#
	# The port the master is listening on.
	# optional - defaults to 3306
	#master-port     =  <port>
	#
	# binary logging - not required for slaves, but recommended
	#log-bin=mysql-bin

	# Uncomment the following if you are using InnoDB tables
	#innodb_data_home_dir = @localstatedir@
	#innodb_data_file_path = ibdata1:10M:autoextend
	#innodb_log_group_home_dir = @localstatedir@
	# You can set .._buffer_pool_size up to 50 - 80 %
	# of RAM but beware of setting memory usage too high
	#innodb_buffer_pool_size = 256M
	#innodb_additional_mem_pool_size = 20M
	# Set .._log_file_size to 25 % of buffer pool size
	#innodb_log_file_size = 64M
	#innodb_log_buffer_size = 8M
	#innodb_flush_log_at_trx_commit = 1
	#innodb_lock_wait_timeout = 50

	[mysqldump]
	quick
	max_allowed_packet = 16M

	[mysql]
	no-auto-rehash
	# Remove the next comment character if you are not familiar with SQL
	#safe-updates

	[myisamchk]
	key_buffer_size = 128M
	sort_buffer_size = 128M
	read_buffer = 2M
	write_buffer = 2M

	[mysqlhotcopy]
	interactive-timeout

#####2.3配置systemd文件#####

	[root@node1 ~]# vim /usr/lib/systemd/system/mysql.service

	[Unit]
	Description=mysql rdbms

	[Service]
	User=mysql
	Group=mysql
	Type=simple
	GuessMainPID=yes
	ExecStart=/usr/local/mysql/bin/mysqld --defaults-file=/etc/my.cnf
	ExecStop=kill /data/mysql/mysql.pid
	Restart=systemctl stop mysql && systemctl start mysql

	[Install]
	WantedBy=multi-user.target


#####2.4配置环境变量#####

	[root@node1 ~]# vim /etc/profile.d/mysql.sh
	export PATH=$PATH:/usr/local/mysql/bin
	[root@node1 ~]# . /etc/profile.d/mysql.sh 

#####2.5测试Mysql运行#####
 
	[root@node1 ~]# systemctl enable mysql.service 
	[root@node1 ~]# systemctl start mysql.service
	[root@node1 ~]# netstat -tlnp | grep mysql


![](http://images.allposs.com/20160302002.png)
####3.配置主从关系####
#####3.1创建主从用户#####

	[root@node1 ~]# mysql
	mysql> grant replication slave on *.* TO 'repluser'@'10.199.200.%' identified by 'replpasss';
	mysql> flush privileges;
	mysql> quit

#####3.2查看二进制日志信息#####

	mysql> show master status;


![](http://images.allposs.com/20160302003.png)
#####3.3查看二进制日志#####

	mysql> show binlog events in 'master-bin.000004'\G
![](http://images.allposs.com/20160302005.png)
###配置node2###
####1.node2配置####
####1.1创建data分区####

	[root@node2 ~]# fdisk /dev/sdb
	Command (m for help): n
	Partition type:
   		p   primary (0 primary, 0 extended, 4 free)
   		e   extended
	Select (default p): p
	Partition number (1-4, default 1): 1
	First sector (2048-62914559, default 2048): 
	Using default value 2048
	Last sector, +sectors or +size{K,M,G} (2048-62914559, default 62914559): 
	Using default value 62914559
	Partition 1 of type Linux and of size 30 GiB is set
	Command (m for help): t
	Hex code (type L to list all codes): 8e
	Changed type of partition 'Linux' to 'Linux LVM'
	Command (m for help): w
	The partition table has been altered!

	[root@node2 ~]# pvcreate /dev/sdb1
	[root@node2 ~]# vgcreate data /dev/sdb2
	[root@node2 ~]# lvcreate -L 10G -n mysqldata data
	[root@node2 ~]# mkfs.ext4 /dev/data/mysqldata 
	[root@node2 ~]# lsblk -f
	复制UUID
	[root@node1 ~]# vim /etc/fstab 
	写入相关配置


#####1.2配置相关权限#####

	[root@node2 ~]# useradd -M -s /sbin/nologin mysql
	[root@node2 ~]# chown -R mysql.mysql /data/mysql/
	[root@node2 ~]# firewall-cmd --permanent --add-service=mysql
	[root@node2 ~]# firewall-cmd --reload


####2.配置mysql####
#####2.1前提#####

	[root@node2 ~]# wget http://10.199.200.15/source/mysql-5.6.29-linux-glibc2.5-x86_64.tar.gz
	[root@node2 ~]# tar xf mysql-5.6.29-linux-glibc2.5-x86_64.tar.gz -C /usr/local/ 
	[root@node2 ~]# cd /usr/local/

	[root@node2 local]# mv mysql-5.6.29-linux-glibc2.5-x86_64 mysql
	[root@node2 local]# cd mysql/
	[root@node2 mysql]# chown -R root.mysql ./*
	[root@node2 mysql]# scripts/mysql_install_db --user=mysql --datadir=/data/mysql
	[root@node1 mysql]# cp -f support-files/my-default.cnf /etc/my.cnf
	[root@node2 ~]# vim /etc/my.cnf

#####2.2修改配置文件#####
	
	# MySQL programs look for option files in a set of
	# locations which depend on the deployment platform.
	# You can copy this option file to one of those
	# locations. For information about these locations, see:
	# http://dev.mysql.com/doc/mysql/en/option-files.html
	#
	# In this file, you can use all long options that a program supports.
	# If you want to know which options a program supports, run the program
	# with the "--help" option.
	
	# The following options will be passed to all MySQL clients
	[client]
	#password       = your_password
	port            = 3306
	socket          = /tmp/mysql.sock

	# Here follows entries for some specific programs

	# The MySQL server
	[mysqld]
	port            = 3306
	socket          = /tmp/mysql.sock
	skip-external-locking
	key_buffer_size = 256M
	max_allowed_packet = 1M
	table_open_cache = 256
	sort_buffer_size = 1M
	read_buffer_size = 1M
	read_rnd_buffer_size = 4M
	myisam_sort_buffer_size = 64M
	thread_cache_size = 8
	query_cache_size= 16M
	# Try number of CPU's*2 for thread_concurrency
	thread_concurrency = 8
	datadir = /data/mysql
	# Don't listen on a TCP/IP port at all. This can be a security enhancement,
	# if all processes that need to connect to mysqld run on the same host.
	# All interaction with mysqld must be made via Unix sockets or named pipes.
	# Note that using this option without enabling named pipes on Windows
	# (via the "enable-named-pipe" option) will render mysqld useless!
	# 
	#skip-networking

	# Replication Master Server (default)
	# binary logging is required for replication
	#log-bin=master-bin
	#log-bin-index=master-bin.index
	#去掉二进制日志
	# binary logging format - mixed recommended
	binlog_format=mixed


	relay-log = relay-log
	relay-log-index = relay-log.index



	# required unique id between 1 and 2^32 - 1
	# defaults to 1 if master-host is not set
	# but will not function as a master if omitted
	server-id       = 21
	#修改serverid
	# Replication Slave (comment out master section to use this)
	#
	# To configure this host as a replication slave, you can choose between
	# two methods :
	#
	# 1) Use the CHANGE MASTER TO command (fully described in our manual) -
	#    the syntax is:
	#
	#    CHANGE MASTER TO MASTER_HOST=<host>, MASTER_PORT=<port>,
	#    MASTER_USER=<user>, MASTER_PASSWORD=<password> ;
	#
	#    where you replace <host>, <user>, <password> by quoted strings and
	#    <port> by the master's port number (3306 by default).
	#
	#    Example:
	#
	#    CHANGE MASTER TO MASTER_HOST='125.564.12.1', MASTER_PORT=3306,
	#    MASTER_USER='joe', MASTER_PASSWORD='secret';
	#
	# OR
	#
	# 2) Set the variables below. However, in case you choose this method, then
	#    start replication for the first time (even unsuccessfully, for example
	#    if you mistyped the password in master-password and the slave fails to
	#    connect), the slave will create a master.info file, and any later
	#    change in this file to the variables' values below will be ignored and
	#    overridden by the content of the master.info file, unless you shutdown
	#    the slave server, delete master.info and restart the slaver server.
	#    For that reason, you may want to leave the lines below untouched
	#    (commented) and instead use CHANGE MASTER TO (see above)
	#
	# required unique id between 2 and 2^32 - 1
	# (and different from the master)
	# defaults to 2 if master-host is set
	# but will not function as a slave if omitted
	#server-id       = 2
	#
	# The replication master for this slave - required
	#master-host     =   <hostname>
	#
	# The username the slave will use for authentication when connecting
	# to the master - required
	#master-user     =   <username>
	#
	# The password the slave will authenticate with when connecting to
	# the master - required
	#master-password =   <password>
	#
	# The port the master is listening on.
	# optional - defaults to 3306
	#master-port     =  <port>
	#
	# binary logging - not required for slaves, but recommended
	#log-bin=mysql-bin

	# Uncomment the following if you are using InnoDB tables
	#innodb_data_home_dir = @localstatedir@
	#innodb_data_file_path = ibdata1:10M:autoextend
	#innodb_log_group_home_dir = @localstatedir@
	# You can set .._buffer_pool_size up to 50 - 80 %
	# of RAM but beware of setting memory usage too high
	#innodb_buffer_pool_size = 256M
	#innodb_additional_mem_pool_size = 20M
	# Set .._log_file_size to 25 % of buffer pool size
	#innodb_log_file_size = 64M
	#innodb_log_buffer_size = 8M
	#innodb_flush_log_at_trx_commit = 1
	#innodb_lock_wait_timeout = 50

	[mysqldump]
	quick
	max_allowed_packet = 16M

	[mysql]
	no-auto-rehash
	# Remove the next comment character if you are not familiar with SQL
	#safe-updates

	[myisamchk]
	key_buffer_size = 128M
	sort_buffer_size = 128M
	read_buffer = 2M
	write_buffer = 2M

	[mysqlhotcopy]
	interactive-timeout


#####2.3配置systemd文件#####

	[root@node2 ~]# vim /usr/lib/systemd/system/mysql.service

	[Unit]
	Description=mysql rdbms

	[Service]
	User=mysql
	Group=mysql
	Type=simple
	GuessMainPID=yes
	ExecStart=/usr/local/mysql/bin/mysqld --defaults-file=/etc/my.cnf
	ExecStop=kill /data/mysql/mysql.pid
	Restart=systemctl stop mysql && systemctl start mysql

	[Install]
	WantedBy=multi-user.target


#####2.4测试Mysql#####

	[root@node2 ~]# systemctl enable mysql.service 
	[root@node2 ~]# systemctl start mysql.service 
	[root@node1 ~]# netstat -tlnp | grep mysql


#####2.5配置环境变量#####

	[root@node2 ~]# vim /etc/profile.d/mysql.sh
	export PATH=$PATH:/usr/local/mysql/bin
	[root@node2 ~]# . /etc/profile.d/mysql.sh 

####3.配置主从关系####
#####3.1配置主从复制用户#####

	[root@node1 ~]# mysql
	mysql> change master to master_host='10.199.200.201',master_user='repluser',master_password='replpass',master_log_file='master-bin.000004',master_log_pos=412;
	mysql> show slave status\G
	#查看从服务器信息

![](http://images.allposs.com/20160302006.png)

	mysql> start slave;
	#启动从服务器
	mysql> show slave status\G

![](http://images.allposs.com/20160302007.png)

	mysql> show global variables like 'read_only';
	#查看从服务是否可读写
	mysql> quit

![](http://images.allposs.com/20160302008.png)
#####3.2配置从服务器为只读#####

	[root@node2 ~]# vim /etc/my.cnf
	添加
	read_only = on
	[root@node2 ~]# systemctl restart mysql.service 
	[root@node2 ~]# mysql
	mysql> show global variables like 'read_only';
	mysql> quit
![](http://images.allposs.com/20160302009.png)

###3.测试###
在主服务器上创建数据库，然后再从服务器上看看是否存在。
## 结束##
