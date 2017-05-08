Title: CentOS7.1 编译安装Tomcat服务
Date: 2016/5/4 11:34:12 
Modified: 2016/5/4 11:34:15 
Category: Servers
Tags: Tomcat
Slug: 
Author: allposs
## 简介##
&#160; &#160; &#160; &#160;Tomcat 服务器是一个免费的开放源代码的Web 应用服务器，属于轻量级应用服务器，在中小型系统和并发访问用户不是很多的场合下被普遍使用，是开发和调试JSP 程序的首选。对于一个初学者来说，可以这样认为，当在一台机器上配置好Apache 服务器，可利用它响应对HTML 页面的访问请求。实际上Tomcat 部分是Apache 服务器的扩展，但它是独立运行的，所以当你运行tomcat 时，它实际上作为一个与Apache 独立的进程单独运行的。诀窍是，当配置正确时，Apache 为HTML页面服务，而Tomcat 实际上运行JSP 页面和Servlet。另外，Tomcat和IIS、Apache等Web服务器一样，具有处理HTML页面的功能，另外它还是一个Servlet和 JSP容器，独立的Servlet容器是Tomcat的默认模式。不过，Tomcat处理静态HTML的能力不如Apache服务器。  
&#160; &#160; &#160; &#160;Tomcat 很受广大程序员的喜欢，因为它运行时占用的系统资源小，扩展性好，支持负载平衡与邮件服务等开发应用系统常用的功能；而且它还在不断的改进和完善中，任何一个感兴趣的程序员都可以更改它或在其中加入新的功能。
## 环境##
+ 操作系统：CentOS7.1 x86_64  
+ Yum源：163源  
+ IP地址：10.199.255.15  
+ DNS：10.199.255.15  
+ 主机名：node1

## 软件包##

Yum源

源码包

	java下载地址：
	http://www.oracle.com/technetwork/java/javase/downloads/index.html

	tomcat下载地址：
	http://tomcat.apache.org/download-90.cgi

## 正文##
###1. 安装java###

	[root@node1 ~]# tar -zxvf jdk-8u65-linux-x64.tar.gz   
	[root@node1 ~]# cp -rp jdk1.8.0_65/ /usr/local/  
	[root@node1 ~]# vim /etc/profile

用vim打开profile添加环境变量  

	JAVA_HOME=/usr/local/jdk1.8.0_65/
	CLASSPATH=$CLASSPATH:$JAVA_HOME/lib
	PATH=$JAVA_HOME/bin:$PATH

重新加载profile文件  

	[root@node1 ~]# . /etc/profile
	[root@node1 ~]# chown -R root\: /usr/local/jdk1.8.0_65/
	[root@node1 ~]# java -version

###2. 安装tomcat###

	[root@node1 ~]# tar -zxvf apache-tomcat-9.0.0.M1.tar.gz
	[root@node1 ~]# cp -rp apache-tomcat-9.0.0.M1 /usr/local/Tomcat

编辑系统环境变量

	[root@node1 ~]# vim /etc/profile

写入系统环境变量

	#tomcat环境变量
	CATALINA_HOME=/usr/local/Tomcat/
	PATH=$JAVA_HOME/bin:/usr/local/Tomcat/bin/:$PATH

重新加载环境变量

	[root@node1 ~]# . /etc/profile

启动tomcat  

	[root@node1 ~]# /usr/local/Tomcat/bin/catalina.sh start

## 结束##