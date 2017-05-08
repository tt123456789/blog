Title: Nginx tomcat memcached 集群安装与配置
Date: 2016/5/6 11:09:05  
Modified: 2016/5/6 11:09:08  
Category: Servers
Tags: tomcat
Slug: 
Author: allposs
## 简介##
&#160; &#160; &#160; &#160;memcache来存储session，tomcat作为servlet容器，nginx作为tomcat代理，对外接口。这个时候只要保证memcache服务器不停，tomcat之间可以任意热插拔。实现了session的独立存储，需要的话，还可以存储到数据库或者文件中，也不用考虑是否使用黏性session。




## 环境##

+ 操作系统：CentOS7.1 X86_64
+ Yum源：163源
+ IP地址：10.199.200.201 10.199.200.202 10.199.200.203 10.199.200.204
+ DNS：10.199.200.15
+ 主机名：node1,node2,node3,node4

## 软件包##

###1. Yum源###
	[root@node1 ~]# yum -y groupinstall "Development Tools"
	[root@node1 ~]# yum install make apr* autoconf automake curl-devel gcc gcc-c++ zlib-devel openssl openssl-devel pcre-devel gd kernel keyutils patch perl kernel-headers compat* mpfr cpp glibc libgomp libstdc++-devel ppl cloog-ppl keyutils-libs-devel libcom_err-devel libsepol-devel  libselinux-devel krb5-devel zlib-devel libXpm* freetype libjpeg* libpng* php-common php-gd ncurses* libtool* libxml2 libxml2-devel patch libmcrypt libmcrypt-devel

###2. 源码包###
	
	[root@node4 ~]# wget https://github.com/downloads/libevent/libevent/libevent-2.0.21-stable.tar.gz
	[root@node4 ~]# wget http://memcached.org/files/memcached-1.4.25.tar.gz

	jar包
	https://code.google.com/archive/p/memcached-session-manager/downloads
##拓扑图##

![](http://image.allposs.cn/20160506132217.png)

## 正文##

###1.node1配置

####1.1安装nginx

	[root@node1 ~]# yum -y groupinstall "Development Tools"
	[root@node1 ~]# yum install make apr* autoconf automake curl-devel gcc gcc-c++ zlib-devel openssl openssl-devel pcre-devel gd kernel keyutils patch perl kernel-headers compat* mpfr cpp glibc libgomp libstdc++-devel ppl cloog-ppl keyutils-libs-devel libcom_err-devel libsepol-devel  libselinux-devel krb5-devel zlib-devel libXpm* freetype libjpeg* libpng* php-common php-gd ncurses* libtool* libxml2 libxml2-devel patch libmcrypt libmcrypt-devel
    [root@node1 ~]# tar xf nginx-1.9.10.tar.gz
	[root@node1 ~]# cd nginx-1.9.10/
	[root@node1 nginx-1.9.10]# ./configure --prefix=/usr/local/nginx --conf-path=/etc/nginx/nginx.conf --pid-path=/var/run/nginx/nginx.pid --user=nginx --group=nginx
	[root@node1 nginx-1.9.10]# make && make install
	[root@node1 nginx-1.9.10]# /usr/local/nginx/sbin/nginx -t
	测试配置文件是否正常
	[root@node1 nginx-1.9.10]# /usr/local/nginx/sbin/nginx

启动nginx并在浏览器里测试是否正常

####1.2修改配置文件

	user  nginx nginx; #运行nginx所在的用户名和用户组
	worker_processes  8;#启动进程数
	error_log  logs/error.log;
	#error_log  logs/error.log  notice;
	#error_log  logs/error.log  info;

	pid        /tmp/nginx.pid;

	worker_rlimit_nofile 65535;
	events {
		use epoll;
    	worker_connections  65535;#工作进程的最大连接数量
	}


	http {
    	include       mime.types;
    	default_type  application/octet-stream;
    	sendfile        on;
    	keepalive_timeout  65;
    	gzip  on;
		#设定请求缓冲    
      	server_names_hash_bucket_size 128;
      	client_header_buffer_size 32k;
      	large_client_header_buffers 4 32k;
      	#client_max_body_size 8m;
		tcp_nopush     on;
		tcp_nodelay on;

    	upstream node1.allposs.com {
        	server  10.199.200.202:8080;
        	server  10.199.200.203:8090;
        }
    server {
        listen       80;
        server_name  node1.allposs.com;
        charset utf-8;

        location / {
            root   html;
            index  index.html index.htm;
            proxy_pass http://node1.allposs.com;
            proxy_set_header X-Real-IP $remote_addr;
            client_max_body_size 100m;

        }
        location ~^/(WEB-INF)/ {
            deny all;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        
    }

###2.node2配置
####2.1安装tomcat

	[root@node2 ~]# tar xf jdk-8u77-linux-x64.tar.gz 
	[root@node2 ~]# mv jdk1.8.0_77/ /usr/local/jdk
		JAVA_HOME=/usr/local/jdk/
		CLASSPATH=$CLASSPATH:$JAVA_HOME/lib
		PATH=$JAVA_HOME/bin:$PATH
	[root@node2 ~]# source /etc/profile
	[root@node2 ~]# java -version	
	[root@node2 ~]# tar xf apache-tomcat-7.0.67.tar.gz
	[root@node2 ~]# mv apache-tomcat-7.0.67 /usr/local/tomcat
	[root@node2 ~]# vim /etc/profile	
		CATALINA_BASE=/usr/local/tomcat
		CATALINA_HOME=/usr/local/tomcat
	[root@node2 ~]# export CATALINA_BASE CATALINA_HOME
	[root@node2 ~]# cd /usr/local/tomcat/bin/
	[root@node2 bin]# ./startup.sh
	[root@node2 bin]# firewall-cmd --permanent --add-port=8080/tcp
	[root@node2 bin]# firewall-cmd --reload

在浏览器中输入ip地址：8080测试tomcat是否成功

####2.2配置tomcat与memcached连接

#####2.2.1下载jar包
&#160; &#160; &#160; &#160;Tomcat集群配置，集群中各个结点通过共享存储在缓存Memcached中session来实现session的共享：如果有一台机器上的Tomcat服务停掉了，对于其他对等服务器上的session数据仍然可以从Memcached缓存中读取，从而不会发生session丢失的问题。

&#160; &#160; &#160; &#160;我们使用的是memcached，所以要使用spymemcached-2.11.1.jar这个包

&#160; &#160; &#160; &#160;注意1：如果你想使用couchbase形式的，则需要以下jar包： couchbase-client-1.4.0.jar jettison-1.3.jar, commons-codec-1.5.jar, httpcore-4.3.jar,httpcore-nio-4.3.jar, netty-3.5.5.Final.jar

&#160; &#160; &#160; &#160;注意2：如果你使用java内置的序列化方式，把jar放在$CATALINA_HOME/lib/里即可。如果为了更好的性能，使用自定义的序列化方式，就要把其它jar包部署在具体java项目工程下的WEB-INF/lib里。以下是四种session序列化方式对应需要的jar包

    kryo-serializer: msm-kryo-serializer, kryo-serializers-0.11 (0.11 is needed, as 0.20+ is for kryo2), kryo, minlog, reflectasm, asm-3.2

    javolution-serializer: msm-javolution-serializer, javolution-5.4.3.1

    xstream-serializer: msm-xstream-serializer, xstream, xmlpull, xpp3_min

    flexjson-serializer: msm-flexjson-serializer, flexjson

&#160; &#160; &#160; &#160;可以在[https://code.google.com/p/memcached-session-manager](https://code.google.com/p/memcached-session-manager)和[https://spymemcached.googlecode.com](https://spymemcached.googlecode.com)上下载相关的jar文件。基于不同的序列化方案，可以有多种配置方法，下面是选择Javolution序列化框架，需要提供如下库文件：

    https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/memcached-session-manager/memcached-session-manager-1.6.5.jar
    https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/memcached-session-manager/memcached-session-manager-tc7-1.6.5.jar
    https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/memcached-session-manager/msm-javolution-serializer-1.6.5.jar
    http://memcached-session-manager.googlecode.com/svn/maven/javolution/javolution/5.4.3.1/javolution-5.4.3.1.jar
    https://spymemcached.googlecode.com/files/spymemcached-2.8.4.jar

#####2.2.2配置server.xml,修改根目录

	<host....>
	.....
	<Context docBase="/var/www/html" path="" reloadable="true"/>
	.....

	</Host>

#####2.2.3配置context.xml，配置msm。

	[root@node2 conf]# vim context.xml

	<?xml version='1.0' encoding='utf-8'?>

		<Context>
		<WatchedResource>WEB-INF/web.xml</WatchedResource>
			<Manager className="de.javakaffee.web.msm.MemcachedBackupSessionManager"   
    			memcachedNodes="n1:10.199.200.204:11211"   
    			requestUriIgnorePattern=".*\.(png|gif|jpg|css|js)$"   
    			sessionBackupAsync="false"   
    			sessionBackupTimeout="100" 
				transcoderFactoryClass="de.javakaffee.web.msm.serializer.javolution.JavolutionTranscoderFactory"   
    		copyCollectionsForSerialization="false" 
	/>
	</Context>


#####2.2.4配置测试jsp页面

	[root@node2 lib]# vim /var/www/html/test.jsp 

	SessionID:<%=session.getId()%>  
	<BR>  
	SessionIP:<%=request.getServerName()%>  
	<BR>  
	SessionPort:<%=request.getServerPort()%>  
	<%  
	out.println("This is Tomcat Server node2！");  
	%>  




###3.node3配置
####3.1安装tomcat
	[root@node3 ~]# tar xf jdk-8u77-linux-x64.tar.gz 
	[root@node3 ~]# tar xf apache-tomcat-7.0.67.tar.gz 
	[root@node3 ~]# mv jdk1.8.0_77/ /usr/local/jdk
	[root@node3 ~]# mv apache-tomcat-7.0.67 /usr/local/tomcat
	[root@node3 ~]# vim /etc/profile
		CATALINA_BASE=/usr/local/tomcat
		CATALINA_HOME=/usr/local/tomcat
		JAVA_HOME=/usr/local/jdk/
		CLASSPATH=$CLASSPATH:$JAVA_HOME/lib
		PATH=$JAVA_HOME/bin:$PATH
	[root@node3 ~]# source /etc/profile
	[root@node3 ~]# java -version
	[root@node3 ~]# cd /usr/local/tomcat/bin/
	[root@node3 bin]# ./startup.sh
	[root@node3 bin]# firewall-cmd --permanent --add-port=8080/tcp
	[root@node3 bin]# firewall-cmd --reload

在浏览器中输入ip地址：8080测试tomcat是否成功

####3.2配置tomcat与memcached连接

#####3.2.1下载jar包
	[root@node3 ~]# cd /usr/local/tomcat/lib/
	[root@node3 lib]# wget https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/memcached-session-manager/memcached-session-manager-1.6.5.jar
	[root@node3 lib]# wget https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/memcached-session-manager/memcached-session-manager-tc7-1.6.5.jar
	[root@node3 lib]# wget http://memcached-session-manager.googlecode.com/svn/maven/javolution/javolution/5.4.3.1/javolution-5.4.3.1.jar
	[root@node2 lib]# wget https://spymemcached.googlecode.com/files/spymemcached-2.8.4.jar

#####3.2.2配置server.xml,修改根目录
	[root@node3 conf]# vim context.xml
	<host....>
	.....
	<Context docBase="/var/www/html" path="" reloadable="true"/>
	.....

	</Host>

#####3.2.3配置context.xml，配置msm。

	[root@node2 conf]# vim context.xml

	<?xml version='1.0' encoding='utf-8'?>

		<Context>
		<WatchedResource>WEB-INF/web.xml</WatchedResource>
			<Manager className="de.javakaffee.web.msm.MemcachedBackupSessionManager"   
    			memcachedNodes="n1:10.199.200.204:11211"   
    			requestUriIgnorePattern=".*\.(png|gif|jpg|css|js)$"   
    			sessionBackupAsync="false"   
    			sessionBackupTimeout="100" 
				transcoderFactoryClass="de.javakaffee.web.msm.serializer.javolution.JavolutionTranscoderFactory"   
    		copyCollectionsForSerialization="false" 
	/>
	</Context>


#####3.2.4配置测试jsp页面

	[root@node2 lib]# vim /var/www/html/test.jsp 

	SessionID:<%=session.getId()%>  
	<BR>  
	SessionIP:<%=request.getServerName()%>  
	<BR>  
	SessionPort:<%=request.getServerPort()%>  
	<%  
	out.println("This is Tomcat Server node3！");  
	%>  

###4node4配置
####4.1安装memcached
下载libevent，安装memcached需要libevent支持

	[root@node4 ~]# wget https://github.com/downloads/libevent/libevent/libevent-2.0.21-stable.tar.gz
	[root@node4 ~]#	tar xf libevent-2.0.21-stable.tar.gz 
	[root@node4 ~]# cd libevent-2.0.21-stable/
	[root@node4 libevent-2.0.21-stable]# ./configure --prefix=/usr/local/libevent
	[root@node4 libevent-2.0.21-stable]# make && make install
	[root@node4 ~]# wget http://memcached.org/files/memcached-1.4.25.tar.gz
	[root@node4 ~]# tar xf memcached-1.4.25.tar.gz 
	[root@node4 ~]# cd memcached-1.4.25/
	[root@node4 memcached-1.4.25]# ./configure --prefix=/usr/local/memcached --with-libevent=/usr/local/libevent
	[root@node4 memcached-1.4.25]# make && make install
	[root@node4 memcached-1.4.25]# firewall-cmd --permanent --add-port=11211/tcp
	[root@node4 memcached-1.4.25]# firewall-cmd --reload
	[root@node4 ~]# /usr/local/memcached/bin/memcached -d -m 512  -p 11211 -c 256 -P /tmp/memcached.pid -u root
		-d选项是启动一个守护进程.
		-m是分配给Memcache使用的内存数量，单位是MB，我这里是10MB.
		-u是运行Memcache的用户，我这里是root.
		-l是监听的服务器IP地址，如果有多个地址的话，我这里指定了服务器的IP地址192.168.0.200.
		-p是设置Memcache监听的端口，我这里设置了12000，最好是1024以上的端口.
		-c选项是最大运行的并发连接数，默认是1024，我这里设置了256，按照你服务器的负载量来设定.
		-P是设置保存Memcache的pid文件，我这里是保存在 /tmp/memcached.pid.

####4.2测试memcached

	[root@node4 ~]# telnet 127.0.0.1 11211
	Trying 127.0.0.1...
	Connected to 127.0.0.1.
	Escape character is '^]'.
	set key1 0 60 4
	zhou
	STORED
	get key1
	VALUE key1 0 4
	zhou
	END

####4.3配置memcached服务脚本

centos6

	#!/bin/sh   
	#   
	# memcached:    MemCached Daemon   
	#   
	# chkconfig:    - 90 25  
	# description:  MemCached Daemon   
	#   
	# Source function library.   
	. /etc/rc.d/init.d/functions   
	. /etc/sysconfig/network   
	#[ ${NETWORKING} = "no" ] && exit 0  
	#[ -r /etc/sysconfig/dund ] || exit 0  
	#. /etc/sysconfig/dund   
	#[ -z "$DUNDARGS" ] && exit 0  
	start()   
	{   
        echo -n $"Starting memcached: "  
        daemon $MEMCACHED -u daemon -d -m 1024  -p 11211  
        echo   
	}   
	stop()   
	{   
        echo -n $"Shutting down memcached: "  
        killproc memcached   
        echo   
	}   
	MEMCACHED="/usr/local/memcached/bin/memcached"  
	[ -f $MEMCACHED ] || exit 1  
	# See how we were called.   
	case "$1" in   
  	start)   
        	start   
        ;;   
  	stop)   
        stop   
        ;;   
  	restart)   
        stop   
        sleep 3  
        start   
        ;;   
    *)   
        echo $"Usage: $0 {start|stop|restart}"  
        exit 1  
	esac   
	exit 0 



###5.测试
####5.1重启node2与node3
#####5.1.1重启node2并测试
	[root@node2 bin]# ./shutdown.sh
	[root@node2 bin]# ./catalina.sh start

启动日志如下：

![](http://image.allposs.cn/20160506102337.png)

通过浏览器http://10.199.200.202:8080/test.jsp访问

![](http://image.allposs.cn/20160506103915.png)

#####5.1.2重启node3并测试

	[root@node3 bin]# ./shutdown.sh
	[root@node3 bin]# ./catalina.sh start

启动日志如下：
![](http://image.allposs.cn/20160506104107.png)

通过浏览器http://10.199.200.203:8080/test.jsp访问
	
![](http://image.allposs.cn/20160506104128.png)
5.2通过nginx测试
通过浏览器http://10.199.200.201/test.jsp访问

第一次刷新

![](http://image.allposs.cn/20160506104353.png)

第二次刷新

![](http://image.allposs.cn/20160506104417.png)

## 结束##