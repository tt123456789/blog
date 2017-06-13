Title: Nginx 反向代理配置与安装
Date: 2016/5/14 10:35:29 
Modified: 2016/5/14 10:35:32 
Category: Servers
Tags: nginx
Slug: 
Author: allposs


## 简介##
&#160; &#160; &#160; &#160;Nginx proxy 是 Nginx 的王牌功能,利用 proxy 基本可以实现一个完整的 7 层负载均。
1. 功能强大,性能卓越,运行稳定。
2. 配置简单灵活。
3. 能够自动剔除工作不正常的后端服务器。
4. 上传文件使用异步模式。
5. 支持多种分配策略,可以分配权重,分配方式灵活。

## 环境##

+ 操作系统：CentOS7.1 X86_64
+ Yum源：163源
+ IP地址：如拓扑图
+ DNS：10.199.255.15
+ 主机名：node1,node2,node3

## 软件包##

###1. Yum源

	[root@node2 ~]# yum install httpd
	[root@node3 ~]# yum install httpd
	[root@node1 nginx-1.9.9]# yum -y groupinstall "Development Tools"
	[root@node1 nginx-1.9.9]# yum install pcre-devel openssl openssl-devel

###2. 源码包

	[root@node1 ~]# wget http://yum.allposs.com/source/nginx-1.9.9.tar.gz


##拓扑图
Nginx 静态反向代理：
![](http://images.allposs.com/20151229001.png)

Nginx PHP反向代理：
![](http://images.allposs.com/20151229002.png)
## 正文
###Nginx 静态反向代理

####1. 配置node2


	[root@node2 ~]# yum install httpd httpd-devel mariadb mariadb-server mariadb-devel php php-mysql php-common php-gd php-mbstring php-mcrypt php-devel php-xml 
	[root@node2 ~]# systemctl enable httpd.service 
	[root@node2 ~]# systemctl enable mariadb.service
	[root@node2 ~]# vim /etc/httpd/conf/httpd.conf 

修改主页为index.php index.html

	[root@node2 ~]# systemctl start httpd.service 
	[root@node2 ~]# systemctl start mariadb.service 
	[root@node2 ~]# firewall-cmd --permanent --add-service=http
	[root@node2 ~]# firewall-cmd --reload 


初始化mariadb

	[root@node2 ~]# mysql_secure_installation 
	/usr/bin/mysql_secure_installation: line 379: find_mysql_client: command not found

	NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      	SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

	In order to log into MariaDB to secure it, we'll need the current
	password for the root user.  If you've just installed MariaDB, and
	you haven't set the root password yet, the password will be blank,
	so you should just press enter here.

	Enter current password for root (enter for none): 
	OK, successfully used password, moving on...

	Setting the root password ensures that nobody can log into the MariaDB
	root user without the proper authorisation.

	Set root password? [Y/n] y
	New password: 
	Re-enter new password: 
	Password updated successfully!
	Reloading privilege tables..
	 ... Success!


	By default, a MariaDB installation has an anonymous user, allowing anyone
	to log into MariaDB without having to have a user account created for
	them.  This is intended only for testing, and to make the installation
	go a bit smoother.  You should remove them before moving into a
	production environment.

	Remove anonymous users? [Y/n] y
 	... Success!

	Normally, root should only be allowed to connect from 'localhost'.  This
	ensures that someone cannot guess at the root password from the network.

	Disallow root login remotely? [Y/n] y
 	... Success!

	By default, MariaDB comes with a database named 'test' that anyone can
	access.  This is also intended only for testing, and should be removed
	before moving into a production environment.

	Remove test database and access to it? [Y/n] y
	 - Dropping test database...
	 ... Success!
	 - Removing privileges on test database...
	 ... Success!

	Reloading the privilege tables will ensure that all changes made so far
	will take effect immediately.

	Reload privilege tables now? [Y/n] y
	 ... Success!

	Cleaning up...

	All done!  If you've completed all of the above steps, your MariaDB
	installation should now be secure.

	Thanks for using MariaDB!


测试php

	[root@node2 ~]# vim /var/www/html/index.php


写入

	this is php1 !
	<?php phpinfo(); ?>
	this is php1 !
                    


	[root@node2 ~]# vim /var/www/html/index.html

写入

	This is node2 !


####2. 配置node3



	[root@node3 ~]# yum install httpd httpd-devel mariadb mariadb-server mariadb-devel php php-mysql php-common php-gd php-mbstring php-mcrypt php-devel php-xml 
	[root@node3 ~]# systemctl enable httpd.service 
	[root@node3 ~]# systemctl enable mariadb.service
	[root@node3 ~]# vim /etc/httpd/conf/httpd.conf 


修改主页为index.php index.html

	[root@node3 ~]# systemctl start httpd.service 
	[root@node3 ~]# systemctl start mariadb.service 
	[root@node3 ~]# firewall-cmd --permanent --add-service=http
	[root@node3 ~]# firewall-cmd --reload 
	

初始化mariadb

	[root@node3 ~]# mysql_secure_installation 
	/usr/bin/mysql_secure_installation: line 379: find_mysql_client: command not found

	NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      	SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

	In order to log into MariaDB to secure it, we'll need the current
	password for the root user.  If you've just installed MariaDB, and
	you haven't set the root password yet, the password will be blank,
	so you should just press enter here.

	Enter current password for root (enter for none): 
	OK, successfully used password, moving on...

	Setting the root password ensures that nobody can log into the MariaDB
	root user without the proper authorisation.

	Set root password? [Y/n] y
	New password: 
	Re-enter new password: 
	Password updated successfully!
	Reloading privilege tables..
	 ... Success!


	By default, a MariaDB installation has an anonymous user, allowing anyone
	to log into MariaDB without having to have a user account created for
	them.  This is intended only for testing, and to make the installation
	go a bit smoother.  You should remove them before moving into a
	production environment.

	Remove anonymous users? [Y/n] y
 	... Success!

	Normally, root should only be allowed to connect from 'localhost'.  This
	ensures that someone cannot guess at the root password from the network.

	Disallow root login remotely? [Y/n] y
 	... Success!

	By default, MariaDB comes with a database named 'test' that anyone can
	access.  This is also intended only for testing, and should be removed
	before moving into a production environment.

	Remove test database and access to it? [Y/n] y
	 - Dropping test database...
	 ... Success!
	 - Removing privileges on test database...
	 ... Success!

	Reloading the privilege tables will ensure that all changes made so far
	will take effect immediately.

	Reload privilege tables now? [Y/n] y
	 ... Success!

	Cleaning up...

	All done!  If you've completed all of the above steps, your MariaDB
	installation should now be secure.

	Thanks for using MariaDB!

测试php

	[root@node3 ~]# vim /var/www/html/index.php


写入

	this is php2 !
	<?php phpinfo(); ?>
	this is php2!
                    

	[root@node3 ~]# vim /var/www/html/index.html

写入

	This is node3 !

####3. 配置node1


	[root@node1 ~]# wget http://yum.allposs.com/source/nginx-1.9.9.tar.gz
	[root@node1 ~]# tar -zxvf nginx-1.9.9.tar.gz 
	[root@node1 ~]# cd nginx-1.9.9/
	[root@node1 nginx-1.9.9]# 
	[root@node1 ~]# cd nginx-1.9.9/
	[root@node1 nginx-1.9.9]# yum -y groupinstall "Development Tools" "Development Libraries" 
	[root@node1 nginx-1.9.9]# yum install pcre-devel openssl openssl-devel
	[root@node1 nginx-1.9.9]# groupadd -r nginx
	[root@node1 nginx-1.9.9]# useradd -s /sbin/nologin -g nginx -r nginx
	[root@node1 nginx-1.9.9]# ./configure --user=nginx --group=nginx --prefix=/usr/local/nginx/ --with-http_stub_status_module --with-http_ssl_module --with-cc-opt='-O2' --with-cpu-opt=opteron --conf-path=/etc/nginx/nginx.conf
	[root@node1 nginx-1.9.9]# make && make install
	[root@node1 nginx-1.9.9]# firewall-cmd --permanent --add-service=http
	[root@node1 nginx-1.9.9]# firewall-cmd --reload
	[root@node1 nginx-1.9.9]# vim /etc/nginx/nginx.conf

添加：

        location / {
            root   /usr/share/nginx/html;
            index  index.html index.htm;
                if ($request_uri ~* \.html$) {
                        proxy_pass http://htmlserver;
                                }
                if ($request_uri ~* \.php$)  {
                        proxy_pass http://phpserver;
                                }
        }

在最尾添加，注意，要与location在同一层次。

	upstream htmlserver {
        server 10.199.255.202;
        server 10.199.255.203;

        }

	upstream phpserver {
        server 10.199.255.202;
        server 10.199.255.203;
        }

启动nginx

	[root@node1 nginx-1.9.9]# /usr/local/nginx/sbin/nginx

在客户端测试

	[root@node4 ~]# curl 10.199.255.201/index.html
	[root@node4 ~]# curl 10.199.255.201
	[root@node4 ~]# curl 10.199.255.201/index.php


## 结束##