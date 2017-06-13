Title: jenkins配置安装
Date: 11:05 2017/3/21
Modified: 11:05 2017/3/21
Category: Servers
Tags: jenkins
Slug: 
Author: allposs


## 简介

&#160; &#160; &#160; &#160;Jenkins是一个开源软件项目，旨在提供一个开放易用的软件平台，使软件的持续集成变成可能。

## 环境
+ 操作系统：CentOS 7.2 x64
+ Yum源：163源
+ IP地址：
			7Node1 10.199.200.101

+ DNS：
+ 主机名：
			7Node1.example.com


## 软件包

###1. Yum源

	[root@7Node1 ~]# wget http://mirrors.hust.edu.cn/apache/tomcat/tomcat-8/v8.5.12/bin/apache-tomcat-8.5.12.tar.gz
	[root@7Node1 ~]# wget http://mirrors.jenkins.io/war-stable/latest/jenkins.war
	[root@7Node1 ~]# wget http://download.oracle.com/otn/java/jdk/8u112-b15/jdk-8u112-linux-x64.rpm

###2. 源码包


##拓扑图



## 正文

###1.安装准备

	[root@7Node1 ~]# wget http://mirrors.hust.edu.cn/apache/tomcat/tomcat-8/v8.5.12/bin/apache-tomcat-8.5.12.tar.gz
	[root@7Node1 ~]# wget http://mirrors.jenkins.io/war-stable/latest/jenkins.war
	[root@7Node1 ~]# wget http://download.oracle.com/otn/java/jdk/8u112-b15/jdk-8u112-linux-x64.rpm
	[root@7Node1 ~]# yum install jdk-7u80-linux-x64.rpm
	[root@7Node1 ~]# tar xf apache-tomcat-8.5.12.tar.gz
	[root@7Node1 ~]# mv apache-tomcat-8.5.12 /opt/tomcat
	[root@7Node1 ~]# rm -rf /opt/tomcat/webapps/*
	[root@7Node1 ~]# cp jenkins.war /opt/tomcat/webapps/
	[root@7Node1 ~]# vim /etc/hosts
		10.199.200.101  7Node1.example.com



###2.开始安装

	[root@7Node1 ~]# cd /opt/tomcat/bin/
	[root@7Node1 bin]# ./startup.sh 
		Using CATALINA_BASE:   /opt/tomcat
		Using CATALINA_HOME:   /opt/tomcat
		Using CATALINA_TMPDIR: /opt/tomcat/temp
		Using JRE_HOME:        /usr
		Using CLASSPATH:       /opt/tomcat/bin/bootstrap.jar:/opt/tomcat/bin/tomcat-juli.jar
		Tomcat started.
	[root@7Node1 bin]# firewall-cmd --permanent --add-port=8080/tcp
	[root@7Node1 bin]# firewall-cmd --reload
	[root@7Node1 bin]# cat /root/.jenkins/secrets/initialAdminPassword
	cc99ab5eab40446d95654dddb0202a8b

访问http://10.199.200.101:8080/jenkins/login?from=%2Fjenkins%2F

输入初始化密码
![](http://images.allposs.com/20170319131314.png)

安装插件方法
![](http://images.allposs.com/20170319131339.png)

安装插件
![](http://images.allposs.com/20170319131354.png)

创建用户与密码
![](http://images.allposs.com/20170321103304.png)


![](http://images.allposs.com/20170321103340.png)

安装完成
![](http://images.allposs.com/20170321105322.png)


## 结束