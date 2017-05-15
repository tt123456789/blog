Title: Zabbix 源码安装配置
Date: 2016/5/4 11:30:27 
Modified: 2016/5/4 11:30:24 
Category: Servers
Tags: zabbix
Slug: 
Author: allposs


## 简介##
&#160; &#160; &#160; &#160;zabbix（音同 zæbix）是一个基于WEB界面的提供分布式系统监视以及网络监视功能的企业级的开源解决方案。
&#160; &#160; &#160; &#160;zabbix能监视各种网络参数，保证服务器系统的安全运营；并提供灵活的通知机制以让系统管理员快速定位/解决存在的各种问题。
&#160; &#160; &#160; &#160;zabbix由2部分构成，zabbix server与可选组件zabbix agent。
&#160; &#160; &#160; &#160;zabbix server可以通过SNMP，zabbix agent，ping，端口监视等方法提供对远程服务器/网络状态的监视，数据收集等功能，它可以运行在Linux，Solaris，HP-UX，AIX，Free BSD，Open BSD，OS X等平台上。

	Zabbix Server (收集数据)
	Zabbix Agent (监视)
	Net-SNMP (为了支持 SNMP(简单网络管理协议))
	Jabber (支持消息通知)
	OpenIPMI (监视)
	cURL (网页监视) 
## 环境##

+ 操作系统：RedHat7.1 X86_64
+ Yum源：本地yum源
+ IP地址：10.199.200.15
+ DNS：10.199.200.15
+ 主机名：Monitor

## 软件包##

###1. Yum源###

	[root@Monitor RedHat]# yum install httpd httpd-devel mariadb mariadb-server mariadb-devel php php-mysql php-common php-gd php-xml 

###2. 源码包###

	[root@Monitor ~]# wget http://downloads.sourceforge.net/project/zabbix/ZABBIX%20Latest%20Stable/2.4.7/zabbix-2.4.7.tar.gz?r=http%3A%2F%2Fwww.zabbix.com%2Fdownload.php&ts=1452051496&use_mirror=netix



##拓扑图##

## 正文##
1.安装基础环境
安装LAMP

	[root@Monitor ~]# yum install httpd httpd-devel mariadb mariadb-server mariadb-devel php php-mysql php-common php-gd php-xml php-bcmath php-mbstring 
	OpenIPMI-devel libssh2-devel libxml2-devel unixODBC-devel net-snmp net-snmp-*
	[root@Monitor ~]# vim /etc/httpd/conf.d/vrt.conf

写入

	<Virtualhost *:80>
	    ServerName zabbix.allposs.com
   	 	DocumentRoot /data/web/Monitor/zabbix/
    	DirectoryIndex index.php index.html
    	<Directory "/data/web/Monitor/zabbix/">
        	Options FollowSymLinks
        	Order allow,deny
        	Allow from all
        	Require all granted
    	</Directory>
		LogFormat "%h %l %u %t %T \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" common
		CustomLog /data/web/logs/Monitor/zabbix_access_log.log common
		ErrorLog /data/web/logs/Monitor/zabbix_error_log.log
	</VirtUalHost>

注：
apache配置文件请见Apache区域相关文档。

开启服务并配置防火墙

	[root@Monitor ~]# mkdir -p /data/web/Monitor/zabbix/
	[root@Monitor ~]# mkdir -p /data/web/logs/Monitor/
	[root@Monitor ~]# chcon -R -t httpd_sys_content_t /data/web/
	[root@Monitor ~]# chcon -R -t httpd_log_t  /data/web/logs/
	[root@Monitor ~]# firewall-cmd --permanent --add-service=http
	[root@Monitor ~]# firewall-cmd --permanent --add-service=mysql
	[root@Monitor ~]# firewall-cmd --reload
	[root@Monitor ~]# systemctl enable mariadb.service 
	[root@Monitor ~]# systemctl enable httpd.service 
	[root@Monitor ~]# systemctl start mariadb.service
	[root@Monitor ~]# systemctl start httpd.service  

初始化Mariadb

	[root@Monitor ~]# mysql_secure_installation
	#使用MySQL的初始化脚本，设置密码等。 
	[root@Monitor ~]# ntpdate cn.pool.ntp.org
	#同步时间，注意一定要与监控客户机一致，最好搭建一个NTP服务器，以保证时间同步。

安装zabbix

	[root@Monitor ~]# wget http://downloads.sourceforge.net/project/zabbix/ZABBIX%20Latest%20Stable/2.4.7/zabbix-2.4.7.tar.gz?r=http%3A%2F%2Fwww.zabbix.com%2Fdownload.php&ts=1452051496&use_mirror=netix
	[root@Monitor ~]# mv zabbix-2.4.7.tar.gz\?r\=http\:%2F%2Fwww.zabbix.com%2Fdownload.php zabbix-2.4.7.tar.gz
	[root@Monitor ~]# tar -zxvf zabbix-2.4.7.tar.gz
	[root@Monitor ~]# cd zabbix-2.4.7/
	[root@Monitor zabbix-2.4.7]# groupadd  -g 201  zabbix
	[root@Monitor zabbix-2.4.7]# useradd -M -s /sbin/nologin -g zabbix zabbix
	[root@Monitor zabbix-2.4.7]# mysql -uroot -p
	MariaDB [(none)]> create database zabbix character set utf8;
	MariaDB [(none)]> grant all on zabbix.* to 'zabbix'@'localhost' identified by 'Password123' with grant option;
	MariaDB [(none)]> flush privileges;
	MariaDB [(none)]> quit
![](http://7xt2ce.com2.z0.glb.clouddn.com/20160106001.png)

导入数据库

	[root@Monitor zabbix-2.4.7]# mysql -u'zabbix' -p'Password123' zabbix <database/mysql/schema.sql
	[root@Monitor zabbix-2.4.7]# mysql -u'zabbix' -p'Password123' zabbix <database/mysql/images.sql
	[root@Monitor zabbix-2.4.7]# mysql -u'zabbix' -p'Password123' zabbix <database/mysql/data.sql

2.编译安装zabbix

	[root@Monitor zabbix-2.4.7]#  yum  -y groupinstall  "Development Tools"
	注意：
		configure: error: Jabber library not found
		#yum install iksemel-devel
		configure: error: LIBXML2 library not found
		#yuminstall libxml2-devel
		configure: error: unixODBC library not found
		#yum install unixODBC-devel
		configure: error: Invalid OPENIPMI directory - unable to findipmiif.h
		#yum install OpenIPMI-devel
		configure:error: Unable to find "javac" executable in path
		需要安装jdk
		configure: error: Curl library not found
		# yum install curl-devel
	[root@Monitor zabbix-2.4.7]# ./configure --prefix=/usr/local/zabbix --sysconfdir=/etc/zabbix/ --enable-server --enable-proxy --enable-agent --with-net-snmp --with-libcurl --with-mysql=/usr/bin/mysql_config --with-net-snmp --with-ssh2 --enable-java --with-libcurl --with-libxml2 --with-ldap --with-openipmi --with-unixodbc--with-jabber --enable-ipv6 
	[root@Monitor zabbix-2.4.7]# make && make install

注：

	--prefix指定zabbix安装目录，
	--enable-server 支持zabbix服务器，
	--enable-agent支持zabbix代理，
	--enable-proxy 支持zabbix代理服务器，
	--with-mysql 使用MySQL客户端库可以选择指定路径mysql_config，
	--with-net-snmp 使用net snmp软件包,择性地指定路径NET SNMP配置, 
	--with-libcurl 使用curl包。
	--with-openipmi 开启温度监控
	--enable-java 开启java监控 注意需要安装jdk
	 安装Jabber需要依赖于iksemel包的iksemel-devel包
	要使用自己发现协议必须开启snmp和fping
配置zabbix


	root@monitor zabbix-2.4.7]# cp -Rf frontends/php/* /data/web/Monitor/zabbix/
	[root@monitor zabbix-2.4.7]# usermod -G apache zabbix
	[root@monitor zabbix-2.4.7]# chown -R apache.apache /data/web/Monitor/zabbix/
	[root@monitor zabbix-2.4.7]# chown -R zabbix.zabbix /data/web/logs/Monitor/
	[root@Monitor zabbix-2.4.7]# vim /usr/lib/systemd/system/zabbix-server.service

写入

	[Unit]
	Description=Zabbix Monitor server
	After=network.target

	[Service]
	Type=forking
	PIDFile=/tmp/zabbix_server.pid
	Environment="CONFIG=/etc/zabbix/zabbix_server.conf"
	ExecStart=/usr/local/zabbix/sbin/zabbix_server  -c ${CONFIG}
	ExecStop=/bin/kill `cat $PIDFile`
	RemainAfterExit=yes
	Restart=always

	[Install]
	WantedBy=multi-user.target

修改配置文件


	[root@Monitor zabbix-2.4.7]# vim /etc/zabbix/zabbix_server.conf

修改

	ListenPort=10051
	LogFile=/data/web/logs/Monitor/zabbix_server.log
	PidFile=/tmp/zabbix_server.pid
	DBHost=localhost
	DBName=zabbix
	DBUser=zabbix
	DBPassword=Password123
	DBPort=3306
	AlertScriptsPath=/etc/zabbix/alertscripts

修改php.ini

	[root@Monitor zabbix-2.4.7]# vim /etc/php.ini 

修改


	post_max_size = 16M
	max_execution_time = 300
	max_input_time = 300
	date.timezone = Asia/Shanghai

添加客户端

	[root@monitor Monitor]# vim /usr/lib/systemd/system/zabbix-agentd.service

写入

	[Unit]
	Description=Zabbix Monitor agentd
	After=network.target

	[Service]
	Type=forking
	PIDFile=/tmp/zabbix_agentd.pid
	Environment="CONFIG=/etc/zabbix/zabbix_agentd.conf"
	ExecStart=/usr/local/zabbix/sbin/zabbix_agentd  -c ${CONFIG}
	ExecStop=/bin/kill `cat $PIDFile`
	RemainAfterExit=yes
	Restart=always

	[Install]
	WantedBy=multi-user.target


修改配置文件

	PidFile=/tmp/zabbix_agentd.pid
	LogFile=/data/web/logs/Monitor/zabbix_agentd.log
	Server=10.199.200.15
	ListenPort=10050
	ServerActive=127.0.0.1
	Hostname=Zabbix server
	UnsafeUserParameters=1

设置防火墙

	[root@monitor ~]# firewall-cmd --permanent --add-port=10051/tcp
	[root@monitor ~]# firewall-cmd --permanent --add-port=10051/udp
	[root@monitor ~]# firewall-cmd --permanent --add-port=10050/udp
	[root@monitor ~]# firewall-cmd --permanent --add-port=10050/tcp
	[root@monitor ~]# firewall-cmd --reload 

开启服务

	[root@monitor ~]# chcon -u system_u /usr/lib/systemd/system/zabbix-*
	[root@monitor ~]# chcon -t systemd_unit_file_t /usr/lib/systemd/system/zabbix-*
	[root@monitor Monitor]# systemctl enable zabbix-agentd.service 
	[root@monitor Monitor]# systemctl start zabbix-agentd.service 

## 结束##
注意：
		1.在配置相关服务的时候一定要注意selinux和防火墙设置，还有还有文件权限的设置。