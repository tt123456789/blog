Title: Nginx 7层负载均衡安装与配置
Date: 2016/5/10 21:49:27 
Modified: 2016/5/10 21:49:31 
Category: Servers
Tags: nginx
Slug: 
Author: allposs

## 简介##
&#160; &#160; &#160; &#160;负载平衡是一种常用的技术来优化利用资源最大化吞吐量，减少等待时间，并确保容错。

&#160; &#160; &#160; &#160;可以使用nginx的作为一种非常高效的HTTP负载平衡器，将流量分配到多个应用服务器上提高性能，可扩展性和高可用性。

## 环境##

+ 操作系统：CentOS7.1 X86_64
+ Yum源：自己配置的yum源
+ IP地址： 	
			Nod1:10.199.255.201 192.168.200.201
			Node2:10.199.255.202
			Node3:10.199.255.203
+ DNS：10.199.255.15
+ 主机名：Node1,Node2,Node3

## 软件包##

###1. Yum源###

	[root@node2 ~]# yum install httpd
	[root@node3 ~]# yum install httpd
	[root@node1 ~]# yum -y groupinstall "Development Tools"
	[root@node1 ~]# yum install pcre-devel openssl openssl-devel


###2. 源码包###

	[root@node1 ~]# wget http://yum.allposs.com/source/nginx-1.9.9.tar.gz

##拓扑图##

![](http://image.allposs.cn/20151231009.png)

## 正文##
###nginx负载均衡种类###
+ round-robin：轮询。以轮询方式将请求分配到不同服务器上
+ least-connected：最少连接数。将下一个请求分配到连接数最少的那台服务器上
+ ip-hash ：基于客户端的IP地址。散列函数被用于确定下一个请求分配到哪台服务器上

###Node1配置###

	[root@node1 ~]# yum -y groupinstall "Development Tools"
	[root@node1 ~]# yum install pcre-devel openssl openssl-devel
	[root@node1 ~]# wget http://yum.allposs.com/source/nginx-1.9.9.tar.gz
	[root@node1 ~]# firewall-cmd --permanent --add-service=http
	[root@node1 ~]# firewall-cmd --reload
	[root@node1 ~]# tar -zxvf nginx-1.9.9.tar.gz 
	[root@node1 ~]# cd nginx-1.9.9/
	[root@node1 nginx-1.9.9]# ./configure --user=nginx --group=nginx --prefix=/usr/local/nginx/ --with-http_stub_status_module --with-http_ssl_module --with-cc-opt='-O2' --with-cpu-opt=opteron --conf-path=/etc/nginx/nginx.conf --with-http_stub_status_module --with-http_ssl_module
	// --with-http_stub_status_module 安装允许状态模块 
	// --with-http_ssl_module 安装ssl模块
	[root@node1 nginx-1.9.9]# make && make install
	[root@node1 nginx-1.9.9]# vim /etc/nginx/nginx.conf

####轮询负载配置####

	http {
    	upstream myhtml {
        	server 10.199.255.202;
        	server 10.199.255.203;
    	}
 
    server {
        listen 80;
 
        location / {
            proxy_pass http://myhtml;
        }
    }


![](http://image.allposs.cn/20151231010.png)
&#160; &#160; &#160; &#160;node2和node3运行着相同的应用程序。nginx默认负载均衡是以轮询方式进行。所有的请求被代理到服务组myhtml，然后nginx负载均衡的分发请求。

&#160; &#160; &#160; &#160;nginx反向代理实现包括下面这些负载均衡HTTP、HTTPS、FastCGI、uwsgi，SCGI和memcached。

&#160; &#160; &#160; &#160;要配置HTTPS的负载均衡，只需使用“https”开头的协议。

&#160; &#160; &#160; &#160;当要设置FastCGI，uwsgi，SCGI，或者memcached的负载平衡，分别使用fastcgi_pass，uwsgi_pass，scgi_pass和memcached_pass指令。


	[root@node1 ~]# /usr/local/nginx/sbin/nginx


测试

	[root@node1 ~]# curl 192.168.200.201



![](http://image.allposs.cn/20151231014.png)
####最少连接负载均衡####


 	upstream myhtml {
        least_conn;
        server 10.199.255.202;
        server 10.199.255.203;
    	}



![](http://image.allposs.cn/20151231015.png)
测试

	[root@node1 ~]# curl 192.168.200.201

####会话持久性
&#160; &#160; &#160; &#160;以轮询或最少连接的负载均衡算法，每个后续的客户端的请求，可以潜在地分配给不同的服务器上，并不能保证相同的客户端请求将总是指向同一服务器上。

&#160; &#160; &#160; &#160;这对于有会话信息的应用场景下，会有问题的。一般的做法是需要将session信息共享，如使用memcache来存放session。

&#160; &#160; &#160; &#160;如果将客户端的会话“粘性”或总是试图选择一个特定的服务器，也是可以的。负载均衡的ip-hash机制就可以实现。

	upstream myapp1 {
    	ip_hash;
    	server 10.199.255.202;
    	server 10.199.255.203;
	}


![](http://image.allposs.cn/20151231016.png)
测试


	[root@node1 ~]# curl 192.168.200.201


![](http://image.allposs.cn/20151231017.png)
####加权负载均衡####
&#160; &#160; &#160; &#160;可以使用权重来进一步控制影响nginx负载均衡算法。

&#160; &#160; &#160; &#160;在上面的例子中，没有配置权重，这意味着所有指定的服务器都被视为同等的。

&#160; &#160; &#160; &#160;当指定的服务器的权重参数，权重占比为负载均衡决定的一部分。权重大负载就大。

	upstream myapp1 {
        	server 10.199.255.202 weight=3;
    		server 10.199.255.203;
    	}


![](http://image.allposs.cn/20151231018.png)

&#160; &#160; &#160; &#160;这种情况下，每4个新的请求将被分布如下：3请求将被引导到202，一个请求将去203

测试只能用抓包测试

###Node2配置###
安装httpd

	[root@node2 ~]# yum install httpd
	[root@node2 ~]# vim /var/www/html/index.html

写入

	This is node2!!!

开启服务并配置防火墙

	[root@node2 ~]# systemctl enable httpd.service 
	[root@node2 ~]# systemctl start httpd.service
	[root@node2 ~]# firewall-cmd --permanent --add-service=http
	[root@node2 ~]# firewall-cmd --reload

测试安装

	[root@node2 ~]# curl 10.199.255.202



###Node3配置###
安装httpd

	[root@node3 ~]# yum install httpd
	[root@node3 ~]# vim /var/www/html/index.html

写入

	This is node3!!!

开启服务并配置防火墙

	[root@node3 ~]# systemctl enable httpd.service 
	[root@node3 ~]# systemctl start httpd.service
	[root@node3 ~]# firewall-cmd --permanent --add-service=http
	[root@node3 ~]# firewall-cmd --reload

测试安装

	[root@node3 ~]# curl 10.199.255.203




## 结束##