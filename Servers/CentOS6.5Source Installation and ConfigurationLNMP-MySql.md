Title: CentOS6.5 源码安装配置LNMP-MYSQL
Date: 2016/5/4 11:29:54 
Modified: 2016/5/4 11:29:52 
Category: Servers
Tags: LNMP
Slug: 
Author: allposs



## 简介##
&#160; &#160; &#160; &#160;LNMP是一个基于CentOS/Debian编写的Nginx、PHP、MySQL、phpMyAdmin、eAccelerator一键安装包。可以在VPS、独立主机上轻松的安装LNMP生产环境。

## 环境##

+ 操作系统：CentOS6.5 X86_64
+ Yum源：163源
+ IP地址：10.199.255.201
+ DNS：10.199.255.15
+ 主机名：moniter

## 软件包##

###1. Yum源###

	[root@moniter ~] yum install make apr* autoconf automake curl-devel gcc gcc-c++ zlib-devel \
	openssl openssl-devel pcre-devel gd kernel keyutils patch perl kernel-headers \
	compat* mpfr cpp glibc libgomp libstdc++-devel ppl cloog-ppl keyutils-libs-devel \
	libcom_err-devel libsepol-devel  libselinux-devel krb5-devel zlib-devel libXpm* \
	freetype libjpeg* libpng* php-common php-gd ncurses* libtool* libxml2 libxml2-devel \
	patch libmcrypt libmcrypt-devel

zlib:nginx提供gzip模块，需要zlib库支持，openssl:nginx提供ssl功能，pcre:支持地址重写rewrite功能
###2. 源码包###

	Nginx:http://nginx.org/download/nginx-1.8.0.tar.gz
	Boost:http://sourceforge.net/projects/boost/files/boost/1.57.0/boost_1_57_0.tar.gz	
	MySql-5.6.24: http://mirrors.sohu.com/mysql/MySQL-5.6/mysql-5.6.24.tar.gz
	Cmake:Cmake-3.2.2:http://www.cmake.org/files/v3.2/cmake-3.2.2.tar.gz
	Php5:http://cn2.php.net/get/php-5.5.25.tar.gz/from/this/mirror


## 正文##


###1. 安装Nginx###
######1.1 Nginx编译安装######

	[root@moniter ~] mkdir -p /var/www/nginx/web	#创建网页根目录
	[root@moniter ~] groupadd -r nginx	#创建nginx用户
	[root@moniter ~] useradd -s /sbin/nologin -g nginx -r nginx	#创建用户nginx并加入到nginx组，不允许nginx用户登录系统
	[root@moniter ~] mkdir /usr/sbin/nginx	#创建二进制文件目录
	[root@moniter ~] mkdir /usr/local/nginx	#创建安装目录
	[root@moniter ~] mkdir /etc/nginx/	#创建配置文件目录
	[root@moniter ~] mkdir /var/run/nginx/	#创建PID文件目录
	[root@moniter ~] tar -zxvf nginx-1.8.0.tar.gz	#解压nginx-1.8.0.tar.gz
	[root@moniter ~] cd nginx-1.8.0
	[root@moniter ~] ./configure --prefix=/usr/local/nginx --sbin-path=/usr/sbin/nginx/ \
	--conf-path=/etc/nginx/nginx.conf --pid-path=/var/run/nginx/nginx.pid \
	--user=nginx --group=nginx --with-http_ssl_module --with-http_flv_module \
	--with-http_gzip_static_module --http-log-path=/var/log/nginx/access.log \
	--http-client-body-temp-path=/var/tmp/nginx/client \
	--http-proxy-temp-path=/var/tmp/nginx/proxy \
	--http-fastcgi-temp-path=/var/tmp/nginx/fcgi --with-http_stub_status_module
	[root@moniter ~] make && mke install

备注：

	--prefix #nginx安装目录，默认在/usr/local/nginx
	--pid-path #pid问件位置，默认在logs目录
	--lock-path #lock问件位置，默认在logs目录
	--with-http_ssl_module #开启HTTP SSL模块，以支持HTTPS请求。
	--with-http_dav_module #开启WebDAV扩展动作模块，可为文件和目录指定权限
	--with-http_flv_module #支持对FLV文件的拖动播放
	--with-http_realip_module #支持显示真实来源IP地址
	--with-http_gzip_static_module #预压缩文件传前检查，防止文件被重复压缩
	--with-http_stub_status_module #取得一些nginx的运行状态
	--with-mail #允许POP3/IMAP4/SMTP代理模块
	--with-mail_ssl_module #允许POP3／IMAP／SMTP可以使用SSL／TLS
	--with-pcre=../pcre-8.11 #注意是未安装的pcre路径
	--with-zlib=../zlib-1.2.5 #注意是未安装的zlib路径
	--with-debug #允许调试日志
	--http-client-body-temp-path #客户端请求临时文件路径
	--http-proxy-temp-path #设置http proxy临时文件路径
	--http-fastcgi-temp-path #设置http fastcgi临时文件路径
	--http-uwsgi-temp-path=/var/tmp/nginx/uwsgi #设置uwsgi 临时文件路径
	--http-scgi-temp-path=/var/tmp/nginx/scgi #设置scgi 临时文件路径

1.2 设置Nginx
设置nginx自启动，加入以下脚本：

	[root@moniter ~] vi /etc/init.d/nginx

配置一

	=======================================================
	#!/bin/bash
	# nginx Startup script for the Nginx HTTP Server
	# it is v.0.0.2 version.
	# chkconfig: - 85 15
	# description: Nginx is a high-performance web and proxy server.
	#              It has a lot of features, but it's not for everyone.
	# processname: nginx
	# pidfile: /var/run/nginx.pid
	# config: /usr/local/nginx/conf/nginx.conf
	nginxd=/usr/sbin/nginx/nginx
	nginx_config=/etc/nginx/nginx.conf
	nginx_pid=/var/run/nginx/nginx.pid
	RETVAL=0
	prog="nginx"
	# Source function library.
	. /etc/rc.d/init.d/functions
	# Source networking configuration.
	. /etc/sysconfig/network
	# Check that networking is up.
	[ ${NETWORKING} = "no" ] && exit 0
	[ -x $nginxd ] || exit 0
	# Start nginx daemons functions.
	start() {
	if [ -e $nginx_pid ];then
	   echo "nginx already running...."
	   exit 1
	fi
	   echo -n $"Starting $prog: "
	   daemon $nginxd -c ${nginx_config}
	   RETVAL=$?
	echo
   		[ $RETVAL = 0 ] && touch /var/lock/subsys/nginx
   		return $RETVAL
	}
	# Stop nginx daemons functions.
	stop() {
        	echo -n $"Stopping $prog: "
        	killproc $nginxd
        	RETVAL=$?
        	echo
        	[ $RETVAL = 0 ] && rm -f /var/lock/subsys/nginx /var/run/nginx/nginx.pid
	}
	# reload nginx service functions.
	reload() {
    	echo -n $"Reloading $prog: "
    	#kill -HUP `cat ${nginx_pid}`
    	killproc $nginxd -HUP
    	RETVAL=$?
    	echo
	}
	# See how we were called.
	case "$1" in
	start)
        	start
        	;;
	stop)
        	stop
        	;;
	reload)
        	reload
        	;;
	restart)
        	stop
        	start
        	;;
	status)
        	status $prog
        	RETVAL=$?
        	;;
	*)
        	echo $"Usage: $prog {start|stop|restart|reload|status|help}"
        	exit 1
	esac
	exit $RETVAL
	=======================================================

配置二

	=======================================================

	#! /bin/bash
	# Description: Startup script for webserver on CentOS. cp it in /etc/init.d and
	# chkconfig --add nginx && chkconfig nginx on
	# then you can use server command control nginx
	#
	# chkconfig: 2345 08 99
	# description: Starts, stops nginx
	set -e
	PATH=$PATH:/usr/sbin/nginx/
	DESC="nginx daemon"
	NAME=nginx
	DAEMON=/usr/sbin/nginx/$NAME
	CONFIGFILE=/etc/nginx/nginx.conf
	PIDFILE=/var/run/nginx/nginx.pid
	SCRIPTNAME=/etc/init.d/$NAME
	# Gracefully exit if the package has been removed.
	test -x $DAEMON || exit 0
	d_start() {
	$DAEMON -c $CONFIGFILE || echo -n " already running"
	}
	d_stop() {
	kill -QUIT `cat $PIDFILE` || echo -n " not running"
	}
	d_reload() {
	kill -HUP `cat $PIDFILE` || echo -n " can't reload"
	}
	case "$1" in
	start)
	echo -n "Starting $DESC: $NAME"
	d_start
	echo "."
	;;
	stop)
	echo -n "Stopping $DESC: $NAME"
	d_stop
	echo "."
	;;
	reload)
	echo -n "Reloading $DESC configuration..."
	d_reload
	echo "reloaded."
	;;
	restart)
	echo -n "Restarting $DESC: $NAME"
	d_stop
	sleep 1
	d_start
	echo "."
	;;
	*)
	echo "Usage: $SCRIPTNAME {start|stop|restart|force-reload}" >&2
	exit 3
	;;
	esac
	exit 0
	=======================================================

配置日志分割脚本
再配置每天自动切割nginx日志脚本：

	=======================================================
	vi /usr/local/nginx/sbin/cut_nginx_log.sh

	#!/bin/bash
	# This script run at 00:00
	# The Nginx logs path
 	logs_path="/var/log/nginx/error.log"
 	logs_bak_path="/root/logs/nginx/"

 	mkdir -p ${logs_bak_path}$(date -d "yesterday" +"%Y")/$(date -d "yesterday" +"%m")/
	 cp ${logs_path} ${logs_bak_path}$(date -d "yesterday" +"%Y")/$(date -d "yesterday" +"%m")/access_$(date -d "yesterday" +"%Y%m%d").log
	 rm -rf ${logs_path}
 	kill -USR1 `cat /var/run/nginx/nginx.pid `

	=======================================================


定时自动启动任务crontab
设定cut_nginx_logs.sh启动时间。
执行命令crontab -e进入编辑状态
添加如下代码，每天0点01分启动。

	[html] view plaincopy
	01 00 * * * /bin/sh  /usr/local/nginx/sbin/cut_nginx_logs.sh
	：wq 退出


	[root@moniter ~] crontab -l 查看运行情况
	[root@moniter ~] chmod 700 /etc/init.d/nginx
	[root@moniter ~] chkconfig --add nginx
	[root@moniter ~] chkconfig --level 2345 nginx on
	[root@moniter ~] /etc/init.d/nginx start

###3.编译安装MySql###
######3.1 编译安装cmake######

	[root@moniter ~] tar -zxvf cmake-3.2.2.tar.gz编译安装cmake
	[root@moniter ~] cd cmake-3.2.2
	[root@moniter ~] ./configure
	[root@moniter ~] make && make install

######3.2 编译安装######

	[root@moniter ~] mkdir /usr/local/boost
	[root@moniter ~] cp boost_1_57_0.tar.gz /usr/local/boost/
	[root@moniter ~] groupadd mysql	编译安装mysql	#添加mysql组
	[root@moniter ~] useradd -g mysql mysql -s /bin/false	#创建用户mysql并加入到mysql组，不允许mysql用户直接登录系统
	[root@moniter ~] mkdir -p /data/mysql #创建MySQL数据库存放目录
	[root@moniter ~] chown -R mysql:mysql /data/mysql #设置权限
	[root@moniter ~] mkdir -p /usr/local/mysql #创建安装目录
	[root@moniter ~] mkdir -p /etc/mysql/	
	[root@moniter ~] tar zxvf mysql-5.6.24.tar.gz
	[root@moniter ~] cd mysql-5.6.24
	[root@moniter ~] cmake -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DINSTALL_DATADIR=/data/mysql/  -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci -DEXTRA_CHARSETS=all -DWITH_SSL=yes -DWITH_EMBEDDED_SERVER=1 -DENABLED_LOCAL_INFILE=1 -DWITH_MYISAM_STORAGE_ENGINE=1 -DWITH_INNOBASE_STORAGE_ENGINE=1 -DWITH_ARCHIVE_STORAGE_ENGINE=1 -DWITH_BLACKHOLE_STORAGE_ENGINE=1 -DWITH_FEDERATED_STORAGE_ENGINE=1 -DWITH_PARTITION_STORAGE_ENGINE=1 -DMYSQL_TCP_PORT=3306 -DENABLED_LOCAL_INFILE=1 -DSYSCONFDIR=/etc/mysql/ -DWITH_READLINE=on -DWITH_BOOST=/usr/local/boost/
	[root@moniter ~] Make
	[root@moniter ~] Make install

######3.3 配置MySql######
########3.3.1  5.6版本安装########

	[root@moniter ~] mv /etc/my.cnf /etc/my.cnf.bak
	[root@moniter ~] /usr/local/mysql/scripts/mysql_install_db --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data --user=mysql
	[root@moniter ~] cp /usr/local/mysql/support-files/mysql.server /etc/rc.d/init.d/mysqld
	[root@moniter ~] chown 755 /etc/rc.d/init.d/mysqld
	[root@moniter ~] service mysqld start

注：在启动MySQL服务时，会按照一定次序搜索my.cnf，先在/etc目录下找，找不到则会搜索"$basedir/my.cnf"，在本例中就是 /usr/local/mysql/my.cnf，这是新版MySQL的配置文件的默认位置！
注意：在CentOS 6.4版操作系统的最小安装完成后，在/etc目录下会存在一个my.cnf，需要将此文件更名为其他的名字，如：/etc/my.cnf.bak，否则，该文件会干扰源码安装的MySQL的正确配置，造成无法启动。
在使用"yum update"更新系统后，需要检查下/etc目录下是否会多出一个my.cnf，如果多出，将它重命名成别的。否则，MySQL将使用这个配置文件启动，可能造成无法正常启动等问题。
MySQL启动成功后，root默认没有密码，我们需要设置root密码。
设置之前，我们需要先设置PATH，要不不能直接调用mysql
修改/etc/profile文件，在文件末尾添加

	=======================================================
	PATH=/usr/local/mysql/bin:$PATH
	export PATH
	=======================================================

关闭文件，运行下面的命令，让配置立即生效

	[root@moniter ~] source /etc/profile

现在，我们可以在终端内直接输入mysql进入，mysql的环境了
执行下面的命令修改root密码

	[root@moniter ~] mysql -uroot  
	mysql> SET PASSWORD = PASSWORD('password');

若要设置root用户可以远程访问，执行

	mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'localhost' IDENTIFIED BY 'password' WITH GRANT OPTION;

红色的password为远程访问时，root用户的密码，可以和本地不同。
###4、PHP安装###
######4.1 编译安装PHP######

	[root@moniter ~] mkdir -p /usr/local/php5
	[root@moniter ~] mkdir -p /etc/php5
	[root@moniter ~] ./configure --prefix=/usr/local/php5 --with-config-file-path=/etc/php5 \
	--with-mysql=/usr/local/mysql --with-mysqli=/usr/local/mysql/bin/mysql_config \
	--with-mysql-sock=/tmp/mysql.sock --with-gd --with-iconv --with-zlib --enable-xml \
	--enable-bcmath --enable-shmop --enable-sysvsem --enable-inline-optimization \
	--with-curlwrappers --enable-mbregex --enable-fpm --enable-mbstring --enable-ftp \
	--enable-gd-native-ttf --with-openssl --enable-pcntl --enable-sockets --with-xmlrpc \
	--enable-zip --enable-soap --without-pear --with-gettext --enable-session --with-mcrypt \
	--with-curl --disable-fileinfo     

	[root@moniter ~] make
	[root@moniter ~] make install

 ######4.2 配置php######

	[root@moniter ~] cp php.ini-production /usr/local/php5/etc/php.ini #复制php配置文件到安装目录
	[root@moniter ~] rm -rf /etc/php.ini #删除系统自带配置文件
	[root@moniter ~] ln -s /usr/local/php5/etc/php.ini /etc/php.ini #添加软链接
	[root@moniter ~] cp /usr/local/php5/etc/php-fpm.conf.default /usr/local/php5/etc/php-fpm.conf#拷贝模板文件为php-fpm配置文件
	[root@moniter ~] vi /usr/local/php5/ etc/php-fpm.conf #编辑

	user = nginx #设置php-fpm运行账号为nginx
	group = nginx #设置php-fpm运行组为nginx
	pid = run/php-fpm.pid #取消前面的分号

设置 php-fpm开机启动

	[root@moniter ~] cp /root/php-5.6.9/sapi/fpm/init.d.php-fpm /etc/rc.d/init.d/php-fpm	#拷贝php-fpm到启动目录
	[root@moniter ~] chmod +x /etc/rc.d/init.d/php-fpm #添加执行权限
	[root@moniter ~] chkconfig php-fpm on #设置开机启动
	[root@moniter ~] vi /usr/local/php5/etc/php.ini #编辑配置文件

找到date.timezone
修改为：

	=======================================================
	date.timezone = PRC #设置时区
	=======================================================

找到：disable_functions =
修改为：

	=======================================================
	disable_functions = passthru,exec,system,chroot,scandir,chgrp,chown,shell_exec,proc_open,proc_get_status,ini_alter,ini_alter,ini_restore,dl,openlog,syslog,readlink,symlink,popepassthru,stream_socket_server,escapeshellcmd,dll,popen,disk_free_space,checkdnsrr,checkdnsrr,getservbyname,getservbyport,disk_total_space,posix_ctermid,posix_get_last_error,posix_getcwd, posix_getegid,posix_geteuid,posix_getgid, posix_getgrgid,posix_getgrnam,posix_getgroups,posix_getlogin,posix_getpgid,posix_getpgrp,posix_getpid, posix_getppid,posix_getpwnam,posix_getpwuid, posix_getrlimit, posix_getsid,posix_getuid,posix_isatty, posix_kill,posix_mkfifo,posix_setegid,posix_seteuid,posix_setgid, posix_setpgid,posix_setsid,posix_setuid,posix_strerror,posix_times,posix_ttyname,posix_uname
	=======================================================

###5. 配置nginx支持php###

	[root@moniter ~] vi /usr/local/nginx/conf/nginx.conf #编辑配置文件,需做如下修改
	[root@moniter ~] user nginx nginx; #首行user去掉注释,修改Nginx运行组为nginx nginx；必须与/usr/local/php5/etc/php-fpm.conf中的user,group配置相同，否则php运行出错

	=======================================================
	server {
	listen 8001;/*监听端口号*/
	server_name www.allposs.cn;/*域名*/
	access_log/usr/local/nginxweb/htdocs/access.log;/*站点访问日志*/
	location / {
	root /var/www/nginx/web/;/*页面文件目录*/
	index index.php index.html index.htm;
	}
	error_page 500 502 503 504 /50x.html;/*服务器错误页面*/
	location = /50x.html {
	root html;
	}
	# pass the PHP scripts to FastCGI serverlistening on 127.0.0.1:9000 
	location ~ \.php$ {
	root /var/www/nginx/web/;
	fastcgi_pass 127.0.0.1:9000; /*Nginx转发请求地址*/
	fastcgi_index index.php;
	fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
	include fastcgi_params;
	}
	location ~ /\.ht {
	deny all;
	}
	}
	=======================================================

###6.测试篇###

	[root@moniter ~] cd /var/www/nginx/web/ #进入nginx默认网站根目录
	[root@moniter ~] vi index.php
输入

	<?php
	phpinfo();
	?>

继续配置

	[root@moniter ~] chown nginx.nginx /var/www/nginx/web/ -R #设置目录所有者
	[root@moniter ~] chmod 700 /var/www/nginx/web/ -R #设置目录权限
	[root@moniter ~] shutdown -r now #重启系统

在浏览器中打开服务器IP地址，会看到下面的界面，配置成功
服务器相关操作命令

	[root@moniter ~] service nginx restart #重启nginx
	[root@moniter ~] service mysqld restart #重启mysql
	[root@moniter ~] /usr/local/php5/sbin/php-fpm #启动php-fpm
	[root@moniter ~] /etc/rc.d/init.d/php-fpm restart #重启php-fpm
	[root@moniter ~] /etc/rc.d/init.d/php-fpm stop #停止php-fpm
	[root@moniter ~] /etc/rc.d/init.d/php-fpm start #启动php-fpm


备注：

	nginx默认站点目录是：/var/www/nginx/web/
	权限设置：chown nginx:nginx /var/www/nginx/web/ -R
	MySQL数据库目录是：/data/mysql
	权限设置：chown mysql.mysql -R /data/mysql
## 结束##
参考文档

	http://www.cnblogs.com/xiongpq/p/3384681.html
	http://www.cnblogs.com/whoamme/p/3678795.html
	http://blog.csdn.net/tjssehaige/article/details/40938757
