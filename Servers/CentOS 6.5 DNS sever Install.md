Title: CentOS 6.5 DNS服务安装
Date: 2016/5/4 11:29:33 
Modified: 9:20 2016/9/21
Category: Servers
Tags: DNS
Slug: 
Author: allposs

## 简介
&#160; &#160; &#160; &#160;DNS（Domain Name System，域名系统），因特网上作为域名和IP地址相互映射的一个分布式数据库，能够使用户更方便的访问互联网，而不用去记住能够被机器直接读取的 IP数串。通过主机名，最终得到该主机名对应的IP地址的过程叫做域名解析（或主机名解析）,有点类似操作系统里面的hosts文件，虽然功能相同，但偏 向不同，一个只是小型数据解析，一个则是大型数据解析，对于解析更有高效。DNS协议运行在UDP协议之上，使用端口号53。在RFC文档中RFC 2181对DNS有规范说明，RFC 2136对DNS的动态更新进行说明，RFC 2308对DNS查询的反向缓存进行说明。

&#160; &#160; &#160; &#160;在DNS服务器里面，每个IP地址都可以有一个主机名，主机名由一个或多个字符串组成，字符串之间用小数点隔开。有了主机名，就不要死记硬背每台IP设备的IP地址，只要记住相对直观有意义的主机名就行了。这就是DNS协议所要完成的功能。

&#160; &#160; &#160; &#160;主机名到IP地址的映射有两种方式：

&#160; &#160; &#160; &#160;1）静态映射，每台设备上都配置主机到IP地址的映射，各设备独立维护自己的映射表，而且只供本设备使用；

&#160; &#160; &#160; &#160;2）动态映射，建立一套域名解析系统（DNS），只在专门的DNS服务器上配置主机到IP地址的映射，网络上需要使用主机名通信的设备，首先需要到DNS服务器查询主机所对应的IP地址。

&#160; &#160; &#160; &#160;通过主机名，最终得到该主机名对应的IP地址的过程叫做域名解析（或主机名解析）。在解析域名时，可以首先采用静态域名解析的方法，如果静态域名解析不成功，再采用动态域名解析的方法。可以将一些常用的域名放入静态域名解析表中，这样可以大大提高域名解析效率。


## 环境

+ 操作系统：CentOS6.5 X86_64
+ Yum源：163源
+ IP地址：10.199.255.15
+ DNS：10.199.255.15
+ 主机名：moniter

## 软件包

###1. Yum源


	[root@moniter ~] yum install bind bind-chroot bind-utils bind-libs

###2. 源码包



## 正文
###1.安装BIND
使用yum安装bind相关软件包

	[root@moniter ~] yum install bind bind-chroot bind-utils bind-libs

###2. 配置BIND
######2.1.配置主配置文件
编辑bind的主配置文件，bind的主配置文件为named.conf,服务名称为named.

	[root@moniter ~] vim /etc/named.conf
	//
	// named.conf
	//
	// Provided by Red Hat bind package to configure the ISC BIND named(8) DNS
	// server as a caching only nameserver (as a localhost DNS resolver only).
	//
	// See /usr/share/doc/bind*/sample/ for example named configuration files.
	//

	options {
        	listen-on port 53 { any; };       //监听端口，
        	listen-on-v6 port 53 { ::1; };
        	directory       "/var/named";
        	dump-file       "/var/named/data/cache_dump.db";
        	statistics-file "/var/named/data/named_stats.txt";
        	memstatistics-file "/var/named/data/named_mem_stats.txt";
        	allow-query     { 10.199.255.0/24; };   //允许访问的客户端，这里是10.199.255.0网段
        	recursion yes;					//允许递归查询

        	dnssec-enable yes;
        	dnssec-validation yes;
        	dnssec-lookaside auto;

        	/* Path to ISC DLV key */
        	bindkeys-file "/etc/named.iscdlv.key";

        	managed-keys-directory "/var/named/dynamic";
	};

	logging {
        	channel default_debug {
                file "data/named.run";
                severity dynamic;
        	};
	};

	zone "." IN {
        	type hint;
        	file "named.ca";
	};
	include "/etc/named.rfc1912.zones";
	include "/etc/named.root.key";

######2.2 配置自定义区域
在二级配置文件下配置自定义区域。

	[root@moniter ~] vim /etc/named.rfc1912.zones
 
	// and http://www.ietf.org/internet-drafts/draft-ietf-dnsop-default-local-zones-02.txt
	// (c)2007 R W Franks
	//
	// See /usr/share/doc/bind*/sample/ for example named configuration files.
	//

	zone "localhost.localdomain" IN {
        	type master;
        	file "named.localhost";
        	allow-update { none; };
	};

	zone "localhost" IN {
        	type master;
        	file "named.localhost";
        	allow-update { none; };
	};

	zone "1.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.ip6.arpa" IN {
        	type master;
        	file "named.loopback";
        	allow-update { none; };
	};

	zone "1.0.0.127.in-addr.arpa" IN {
        	type master;
        	file "named.loopback";
        	allow-update { none; };
	};

	zone "0.in-addr.arpa" IN {
        	type master;
        	file "named.empty";
        	allow-update { none; };
	};

	zone "allposs.com" IN {
        	type master;
        	file "allposs.com_zone";
	};

	zone "255.199.10.in-addr.arpa" IN {
        	type master;
        	file "10.199.255.zone";
	};	

######2.3 配置解析文件
	[root@monitor ~]# vim /var/named/allposs.zone
	$TTL 1D
	@       IN SOA  @       root (
											0       ; serial
											1D      ; refresh
											1H      ; retry
											1W      ; expire
											3H )    ; minimum
	@       NS      ns.allposs.com.
	ns      A       10.199.200.15
	www     A       10.199.200.15
	[root@monitor ~]# vim /var/named/10.199.200.zone
	$TTL 1D
	@       IN SOA  @       root (
											0       ; serial
											1D      ; refresh
											1H      ; retry
											1W      ; expire
											3H )    ; minimum
	@       NS      ns.allposs.com.
	ns      A       10.199.200.15
	15      PTR     www.allposs.com.


## 结束##