Title: Redhat7.1 编译安装LAMP服务-MariaDB
Date: 2016/5/13 20:03:05 
Modified: 2016/5/13 20:03:08 
Category: Servers
Tags: LAMP
Slug: 
Author: allposs



## 简介##
&#160; &#160; &#160; &#160;Linux+Apache+Mysql/MariaDB+Perl/PHP/Python一组常用来搭建动态网站或者服务器的开源软件，本身都是各自独立的程序，但是因为常被放在一起使用，拥有了越来越高的兼容度，共同组成了一个强大的Web应用程序平台。随着开源潮流的蓬勃发展，开放源代码的LAMP已经与J2EE和.Net商业软件形成三足鼎立之势，并且该软件开发的项目在软件方面的投资成本较低，因此受到整个IT界的关注。从网站的流量上来说，70%以上的访问流量是LAMP来提供的，LAMP是最强大的网站解决方案．

&#160; &#160; &#160; &#160;这次是是用redhat7.1+Apache2.4.17+MariaDB10.1+PHP5.6.15搭建的环境，这个只是基础环境，后面会有相关详细文档分开解剖,这篇文档的重点是展示Redhat7.X与6.X版本有哪些不同，还有最近MySql的分支MariaDB与MySql的区别。

## 环境##

+ 操作系统：RedHat7.1 X86_64
+ Yum源：163源
+ IP地址：10.199.255.15
+ DNS：10.199.255.15
+ 主机名：Monitor
+ URL：

## 软件包##

###1. Yum源###

	[root@Monitor ~]# yum  -y groupinstall  "Development Tools"
	[root@Monitor ~]# yum install make autoconf automake curl curl-devel gcc gcc-c++ zlib-devel openssl openssl-devel pcre-devel gd kernel keyutils patch perl kernel-headers compat* cpp glibc libgomp libstdc++-devel keyutils-libs-devel libsepol-devel libselinux-devel krb5-devel cmake ncurses-devel bzip2-devel libmcrypt-devel libpng-devel
	[root@moniter httpd-2.4.17]# yum install gcc bison bison-devel zlib-devel openssl-devel libxml2-devel libcurl-devel bzip2-devel readline-devel libedit-devel openssl openssl-devel  libpng-devel  bzip2 bzip2-devel curl curl-devel  readline-devel

###2. 源码包###

	[root@Monitor ~]# wget http://mirror.bit.edu.cn/apache//httpd/httpd-2.4.17.tar.gz
	[root@Monitor ~]# wget http://mirror.bit.edu.cn/apache//apr/apr-1.5.2.tar.gz
	[root@Monitor ~]# wget http://mirror.bit.edu.cn/apache//apr/apr-util-1.5.4.tar.gz
	[root@Monitor ~]# wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.37.tar.gz
	[root@Monitor ~]# wget http://cn2.php.net/get/php-5.6.15.tar.gz/from/this/mirror
	[root@Monitor ~]# wget https://github.com/MariaDB/server/archive/10.1.zip
	[root@Monitor ~]# wget http://downloads.sourceforge.net/project/libpng/zlib/1.2.8/zlib-1.2.8.tar.gz?r=http%3A%2F%2Fwww.zlib.net%2F&ts=1449822508&use_mirror=nchc

## 正文##

###1. 安装apache###

######1.1 编译Apache

首先卸载旧版本

	[root@Monitor ~]# yum remove apr-util-devel apr apr-util-mysql apr-docs apr-devel apr-util apr-util-docs

开始编译安装

	[root@Monitor ~]# tar -zxvf apr-1.5.2.tar.gz 
	[root@Monitor ~]# cd apr-1.5.2/
	[root@Monitor apr-1.5.2]# ./configure --prefix=/usr/local/apr
	[root@Monitor apr-1.5.2]# make && make install
	[root@Monitor ~]# tar -zxvf apr-util-1.5.4.tar.gz 
	[root@Monitor ~]# cd apr-util-1.5.4/
	[root@Monitor apr-util-1.5.4]# ./configure --prefix=/usr/local/apr-util --with-apr=/usr/local/apr/bin/apr-1-config
	[root@Monitor apr-util-1.5.4]# make && make install 
	[root@Monitor ~]# tar -zxvf pcre-8.37.tar.gz 
	[root@Monitor ~]# cd pcre-8.37/
	[root@Monitor pcre-8.37]# ./configure --prefix=/usr/local/pcre
	[root@Monitor pcre-8.37]# make && make install
	[root@Monitor ~]# tar -zxvf httpd-2.4.17.tar.gz 
	[root@Monitor ~]# cd httpd-2.4.17/
	[root@Monitor httpd-2.4.17]# ./configure --prefix=/usr/local/httpd --enable-so --with-gd --with-jpeg-dir=/usr --with-png-dir=/usr --with-zlib-dir=/usr --with-freetype=/usr --sysconfdir=/etc/httpd/ --with-apr=/usr/local/apr --with-apr-util=/usr/local/apr-util --with-pcre=/usr/local/pcre
	[root@Monitor httpd-2.4.17]# make && make install


######1.2 配置apache

	[root@Monitor bin]# cd /usr/lib/systemd/system
	[root@Monitor system]# vim httpd.service

写入

	[Unit]
	Description=httpd
	After=network.target

	[Service]
	Type=forking
	ExecStart=/usr/local/httpd/bin/apachectl start
	ExecReload=/usr/local/httpd/bin/apachectl restart
	ExecStop=/usr/local/httpd/bin/apachectl  stop
	PrivateTmp=true

	[Install]
	WantedBy=multi-user.target

######1.3 服务配置

	[root@Monitor system]# chmod 754 httpd.service 
	[root@Monitor system]# systemctl enable httpd.service
	[root@Monitor system]# systemctl start httpd.service
	[root@moniter ~]# firewall-cmd --permanent --add-service=http
	[root@moniter ~]# firewall-cmd --permanent --add-service=mysql
	[root@moniter ~]# firewall-cmd --permanent --add-service=https
	[root@Monitor system]# firewall-cmd --reload
	[root@Monitor system]# groupadd -g 202 apache
	[root@Monitor system]# useradd -g apache -u 202 -M -s /sbin/nologin apache

###2. 安装MariaDB

######2.1 编译MariaDB

	[root@Monitor ~]# unzip server-10.1.zip 
	[root@Monitor ~]# cd server-10.1/
	[root@Monitor server-10.1]# cmake . -DCMAKE_INSTALL_PREFIX=/usr/local/MariaDB -DMYSQL_DATADIR=/data/MariaDB -DWITH_INNOBASE_STORAGE_ENGINE=1 -DWITH_ARCHIVE_STPRAGE_ENGINE=1 -DWITH_BLACKHOLE_STORAGE_ENGINE=1 -DWIYH_READLINK=1 -DWIYH_SSL=system -DVITH_ZLIB=system -DWITH_LOBWRAP=0 -DMYSQL_UNIX_ADDR=/tmp/mysql.sock -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci -DSYSCONFDIR=/etc/MariaDB
	[root@Monitor server-10.1]# make
	[root@Monitor server-10.1]# make install

注：
cmake查看帮助是 cmake . -LH or ccmake .

######2.2 基本配置

	[root@Monitor system]# groupadd -g 203 mariadb
	[root@Monitor system]# useradd -g mariadb -u 203 -M -s /sbin/nologin mariadb
	[root@Monitor system]# cd /usr/local/MariaDB/
	[root@Monitor MariaDB]# mkdir /data/MariaDB -p
	[root@Monitor MariaDB]# cp support-files/my-small.cnf /etc/my.cnf
	[root@Monitor MariaDB]# cp support-files/mysql.server /etc/rc.d/init.d/mariadb
	[root@Monitor MariaDB]# chmod +x /etc/rc.d/init.d/mariadb 
	[root@Monitor MariaDB]# chkconfig --add mariadb
	[root@Monitor MariaDB]# chkconfig mariadb on
	[root@Monitor MariaDB]# vim /etc/my.cnf

在最底下写入

	[mysqld]
	basedir = /usr/local/MariaDB
	datadir = /data/MariaDB
	pid-file = /data/MariaDB/mariadb.pid
	character-set-server = utf8
	collation-server = utf8_general_ci
	user = mariadb
	port = 3306
	default_storage_engine = InnoDB
	innodb_file_per_table = 1
	server_id = 1
	log_bin = mysql-bin
	binlog_format = mixed
	expire_logs_days = 7
	bind-address = 0.0.0.0

	# name-resolve
	skip-name-resolve
	skip-host-cache

	#lower_case_table_names = 1
	ft_min_word_len = 1
	query_cache_size = 64M
	query_cache_type = 1

	skip-external-locking
	key_buffer_size = 16M
	max_allowed_packet = 1M
	table_open_cache = 64
	sort_buffer_size = 512K
	net_buffer_length = 8K
	read_buffer_size = 256K
	read_rnd_buffer_size = 512K
	myisam_sort_buffer_size = 8M

	# LOG
	log_error = /data/MariaDB/mariadb-error.log
	long_query_time = 1
	slow_query_log
	slow_query_log_file = /data/MariaDB/mariadb-slow.log

	# Oher
	#max_connections = 1000
	open_files_limit = 65535

######2.3 环境变量配置######

	[root@Monitor MariaDB]# chown mariadb.mariadb /data/MariaDB/
	[root@Monitor MariaDB]# /usr/local/MariaDB/scripts/mysql_install_db --user=mariadb --basedir=/usr/local/MariaDB --datadir=/data/MariaDB
	[root@Monitor MariaDB]# service mariadb start
	[root@Monitor MariaDB]# export PATH=$PATH:/usr/local/MariaDB/bin
	[root@Monitor MariaDB]# echo 'export PATH=$PATH:/usr/local/MariaDB/bin' >> /etc/profile
	[root@Monitor MariaDB]# source /etc/profile

######2.4初始化配置######

登陆MariaDB

	[root@moniter ~]# mysql

查看当前数据库的全局变量和会话变量

	MariaDB [(none)]> show global variables\G
	MariaDB [(none)]> show session variables\G

删除所有匿名用户

	MariaDB [(none)]> drop user ''@'localhost';

给root用户增加密码
第一种方法

	MariaDB [(none)]> set password for 'root'@'localhost' =password('Password123');

第二种方法

	MariaDB [(none)]> update user set password=password('Password123')where user='root' and host='127.0.0.1';

###3.安装PHP

######3.1 编译PHP

	[root@Monitor ~]# mv mirror php-5.6.15.tar.gz
	[root@Monitor ~]# tar -zxvf php-5.6.15.tar.gz 
	[root@moniter ~]# cd php-5.6.15/
	[root@moniter php-5.6.15]# ./configure --prefix=/usr/local/php --with-bz2 --with-curl --enable-ftp --enable-sockets --disable-ipv6 --enable-gd-native-ttf --enable-mbstring --enable-calendar --with-gettext --with-pdo-mysql=mysqlnd --with-mysqli=mysqlnd --with-mysql=mysqlnd --enable-dom --enable-xml --enable-bcmath --with-config-file-path=/etc/php --with-config-file-scan-dir=/etc/php/php.d --with-apxs2=/usr/local/httpd/bin/apxs --with-png-dir=/usr/local/libpng/ --with-jpeg-dir=/usr/local/jpeg/ --with-freetype-dir=/usr/local/freetype/ --with-gd=/usr/local/gd/  --with-zlib-dir=/usr/local/zlib/   --with-mysql --with-mysqli
	[root@Monitor php-5.6.15]# make
	[root@Monitor php-5.6.15]# make install

######3.2 配置集合######

	[root@moniter php-5.6.15]# vim /etc/httpd/httpd.conf

修改用户组为apache

Options Indexes FollowSymLinks  修改为：Options Includes ExecCGI FollowSymLinks（允许服务器执行CGI及SSI，禁止列出目录）
去掉AddHandler cgi-script .cgi前# （允许扩展名为.pl的CGI脚本运行）

AllowOverride None　 修改为：AllowOverride All （允许.htaccess）

DirectoryIndex index.html   修改为：DirectoryIndex index.html index.htm Default.html Default.htm index.php（设置默认首页文件，增加index.php）

在最后一行增加：Addtype application/x-httpd-php .php .phtml

	[root@moniter php-5.6.15]# mkdir /etc/php/php.d
	[root@moniter php-5.6.15]# cp cp php.ini-development /etc/php/php.d/
	[root@moniter ~]# vim .bashrc 

写入


	export PATH=/usr/local/php/bin:$PATH
	export PATH=/usr/local/php/sbin:$PATH

然后运行以下命令

	[root@moniter ~]# source ~/.bashrc
	[root@moniter ~]# php --version
	[root@moniter php-5.6.15]# systemctl restart httpd.service 
	[root@moniter php-5.6.15]# vim /usr/local/httpd/htdocs/info.php

写入

	<?php
	phpinfo();
	?>

在浏览器进行测试，如果phpinfo内容显示基本上没有问题了。
问题
## 结束##