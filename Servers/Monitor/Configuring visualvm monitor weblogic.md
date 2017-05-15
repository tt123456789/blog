Title: 配置visualvm监控weblogic
Date: 2016/7/8 14:24:10   
Modified: 2016/7/8 14:24:13   
Category: Servers
Tags: Weblogic
Slug: 
Author: allposs


## 简介
&#160; &#160; &#160; &#160;JDK为我们提供了可以监控服务器性能的工具，例如JConsole、Java VisualVM，这里讲如果使用Java VisualVM远程监控weblogic服务器的性能（内存，线程，垃圾回收等）

&#160; &#160; &#160; &#160;通过这些指标可以观察服务器的运行状态，分析错误原因，例如内存溢出等。

## 环境

+ 操作系统：CentOS6.5 X86_64
+ Yum源：163源
+ IP地址：10.199.200.201
+ DNS：10.199.200.15
+ 主机名：

## 软件包##

###1. Yum源


###2. 源码包


##拓扑图


## 正文

&#160; &#160; &#160; &#160;visualvm必须通过jstatd服务来取得远程机器上Java应用程序的运行数据。所以我们得先在要监控的机器上启动jstatd服务（这个服务是在远程机器上启动的）

###1. 配置好Java环境

	[root@node1 weblogic]# tar xf jdk-8u92-linux-x64.tar.gz
	[root@node1 weblogic]# mkdir /opt/weblogic
	[root@node1 weblogic]# mv jdk1.8.0_92/ /opt/weblogic/jdk
	[root@node1 weblogic]# vi /etc/profile
	增加
	export JAVA_HOME=/opt/weblogic/jdk
	export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
	export PATH=$PATH:$JAVA_HOME/bin
	
	[root@node1 weblogic]# source /etc/profile
	[root@node1 weblogic]# java -version
	java version "1.8.0_92"
	Java(TM) SE Runtime Environment (build 1.8.0_92-b14)
	Java HotSpot(TM) 64-Bit Server VM (build 25.92-b14, mixed mode)


###2. 创建jstatd.all.policy文件

	[root@node1 weblogic]# cd /opt/weblogic/jdk/bin/
	[root@node1 bin]# vim jstatd.all.policy

	grant codebase "file:${java.home}/../lib/tools.jar" {

       permission java.security.AllPermission;

	};


&#160; &#160; &#160; &#160;这个文件的作用是让jstatd服务能够读取机器上的java应用程序的运行数据


###3. 运行jstatd

&#160; &#160; &#160; &#160;把目录切到：%JAVA_HOME%\bin目录下，然后执行如下命令：

	[root@node1 bin]# jstatd -J-Djava.security.policy=jstatd.all.policy


###4. 配置weblogic

&#160; &#160; &#160; &#160;使用VM监控服务器内存变化，首先需要配置服务器启动参数，进入weblogic安装目录，找到所在domain的bin文件，打开setDomainEnv.sh文件：

&#160; &#160; &#160; &#160;在JAVA_OPTIONS=的后面加上以下配置内容：

	 -XX:+HeapDumpOnOutOfMemoryError -Dcom.sun.management.jmxremote.port=6001 -Dcom.sun.management.jmxremote.authenticate=false  -Dcom.sun.management.jmxremote.ssl=false

注释：

	+HeapDumpOnOutOfMemoryError 
	当服务器发生内存溢出时会自动产生一个dump文件。该文件记录服务器崩溃原因，你可以通过该文件分析问题。


	-Dcom.sun.management.jmxremote.port=6001
	指远程监控的端口，务必保证该端口是可用端口。

	-Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=falseg 
	禁用SSL

	-Dcom.sun.management.jmxremote.ssl=false 
	禁用密码验证

配置完毕后，重启weblogic服务。

###5. 监控weblogic

####5.1 打开visualvm
![](http://image.allposs.cn/20160708145406.png)
####5.2 添加远程主机

远程-右键添加远程主机-输入远程主机IP地址
![](http://image.allposs.cn/20160708145524.png)
添加完成，这只有基本的监控功能
![](http://image.allposs.cn/20160708145536.png)

5.3 添加jmx连接
远程主机名10.199.200.201-右键添加jmx连接-输入远程主机IP地址与jmx端口
![](http://image.allposs.cn/20160708145637.png)

5.5 完成
![](http://image.allposs.cn/20160708145708.png)
	
## 结束##