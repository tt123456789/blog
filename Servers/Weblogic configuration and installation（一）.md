Title: weblogic配置与安装（一）
Date: 2016/7/5 17:07:59  
Modified: 2016/7/11 9:17:17   
Category: Servers
Tags: Weblogic
Slug: 
Author: allposs



## 简介##
&#160; &#160; &#160; &#160;由于公司很多业务都是使用weblogic,所以这里把自己学习weblogic的经历与实践都写下来，有什么写的不好的请下方留言！这期讲的是weblogic的基本安装，没有太多理论，只是一些操作

## 环境##

+ 操作系统：CentOS6.5 X86_64
+ Yum源：163源
+ IP地址：10.199.200.201
+ DNS：10.199.200.15
+ 主机名：node1

## 软件包##

###1. Yum源###

###2. 源码包###
	jdk-8u92-linux-x64.tar.gz
	wls1036_generic.jar
##拓扑图##

无

## 正文##

###1. 预备知识

什么是Domain和Server?

Domain

&#160; &#160; &#160; &#160;Domain是WebLogic Server实例的基本管理单元。所谓Domain就是，由配置为Administrator Server的WebLogic Server实例管理的逻辑单元，这个单元是有所有相关资源的集合。

Server

server是一个相对独立的，为实现某些特定功能而结合在一起的单元。

Domain and Server的关系

&#160; &#160; &#160; &#160;一个Domain 可以包含一个或多个WebLogic Server实例，甚至是Server集群。一个Domain中有一个且只能有一个Server 担任管理Server的功能，其它的Server具体实现一个特定的逻辑功能。

### 2. 安装jdk

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
### 3. 优化系统
#### 3.1 查看nproc(max user processes)
	
	[root@node1 weblogic]# ulimit -a
	core file size          (blocks, -c) 0
	data seg size           (kbytes, -d) unlimited
	scheduling priority             (-e) 0
	file size               (blocks, -f) unlimited
	pending signals                 (-i) 7231
	max locked memory       (kbytes, -l) 64
	max memory size         (kbytes, -m) unlimited
	open files                      (-n) 1024
	pipe size            (512 bytes, -p) 8
	POSIX message queues     (bytes, -q) 819200
	real-time priority              (-r) 0
	stack size              (kbytes, -s) 8192
	cpu time               (seconds, -t) unlimited
	max user processes              (-u) 7231
	virtual memory          (kbytes, -v) unlimited
	file locks                      (-x) unlimited
	暂时修改：
	[root@node1 weblogic]# ulimit -n 10240
	永久性修改
	[root@node1 ~]# vim /etc/security/limits.conf
	*       soft    nofile  10240
	*       hard    nofile  10240
	当然这里也可以修改其它的参数

### 4. 安装weblogic
	
&#160; &#160; &#160; &#160;在安装之前，如果需要，可以添加专门的用户或者用户组，然后为WebLogic指定安装目录等，但不是必须的，根据个人需求。我这里账户名称是root，我就按照默认的来了。安装包放在/root/weblogic/下.
	
	界面安装，一般选择这个方法就可以
	java -jar wls1036_generic.jar  即可启动安装过程, 默认启动的是图形界面的安装向导.
	控制台安装，这个方法一般用在linux等没有安装桌面环境的服务器上，命令行方式的安装命令用如下方式启动:
	java -jar wls1036_generic.jar -mode=console
	静默安装用如下方式启动:
	java -jar wls1036_generic.jar -mode=console -silent_xml=/path_to_silent.xml

	[root@node1 weblogic]# java -jar -d64 wls1036_generic.jar --mode=console

&#160; &#160; &#160; &#160;回车后，进入欢迎界面，输入Next或者直接回车；
![](http://images.allposs.com/20160705133917.png)
&#160; &#160; &#160; &#160;然后会出现安装路径，输入路径，回车；
![](http://images.allposs.com/20160705134139.png)	
&#160; &#160; &#160; &#160;确认选择新的路径，这里选择1，回车;
![](http://images.allposs.com/20160705134156.png)
&#160; &#160; &#160; &#160;再次确认使用新路径，回车；
![](http://images.allposs.com/20160705134215.png)
&#160; &#160; &#160; &#160;注册安全更新，有三项，其中第三项默认是Yes，因为我们不准备接收安全更新，所以我们输入3，回车；
![](http://images.allposs.com/20160705134401.png)
&#160; &#160; &#160; &#160;确认选择，输入No；
![](http://images.allposs.com/20160705134415.png)
&#160; &#160; &#160; &#160;提示是否绕过配置管理器启动过程并不接收安全更新通知，输入Yes；
![](http://images.allposs.com/20160705134804.png)
&#160; &#160; &#160; &#160;回车后出现刚才的三项列表，此时第三项已经更改为No,输入Next，回车；
![](http://images.allposs.com/20160705134951.png)
&#160; &#160; &#160; &#160;选择安装方式，这里选择自定义安装，2，回车；
![](http://images.allposs.com/20160705135204.png)
&#160; &#160; &#160; &#160;选择2，回车；
![](http://images.allposs.com/20160705141149.png)	
&#160; &#160; &#160; &#160;默认，回车；
![](http://images.allposs.com/20160705141253.png)
&#160; &#160; &#160; &#160;选择jdk，如果已经操作过前面安装jdk，并配置环境变量，这里显示的就是java_home目录，回车；
![](http://images.allposs.com/20160705141436.png)
&#160; &#160; &#160; &#160;确认安装路径，如果没问题直接回车;
![](http://images.allposs.com/20160705142330.png)
&#160; &#160; &#160; &#160;确认安装内容，如果没问题直接回车;
![](http://images.allposs.com/20160705142443.png)
&#160; &#160; &#160; &#160;开始安装

。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。

&#160; &#160; &#160; &#160;安装完成如下：
![](http://images.allposs.com/20160705142520.png)

### 5. 配置Domain

	[root@node1 weblogic]# cd /opt/weblogic/weblogic/wlserver_10.3/common/bin/
	[root@node1 bin]# ./config.sh 


&#160; &#160; &#160; &#160;欢迎界面，选择是新建domain还是扩展，因为第一次使用，输入1，即新建；
![](http://images.allposs.com/20160705150746.png)
&#160; &#160; &#160; &#160;选择domain源，也是使用默认配置，即键入1回车；
![](http://images.allposs.com/20160705150937.png)
&#160; &#160; &#160; &#160;应用程序模板选择，这里使用默认，即输入Next回车；
![](http://images.allposs.com/20160705151059.png)
&#160; &#160; &#160; &#160;编辑domain信息，域名base_domain，默认，输入Next回车；
![](http://images.allposs.com/20160705151123.png)
&#160; &#160; &#160; &#160;为domain选择目标或目录，默认，输入Next回车；
![](http://images.allposs.com/20160705151151.png)
&#160; &#160; &#160; &#160;配置管理员用户名和口令，这里按需要输入选项，并根据提示操作，比较简单，配置好输入Next回车（注意这里的密码最好为8位）；
![](http://images.allposs.com/20160705151442.png)
![](http://images.allposs.com/20160705151808.png)
![](http://images.allposs.com/20160705151823.png)
![](http://images.allposs.com/20160705151845.png)
![](http://images.allposs.com/20160705151904.png)
&#160; &#160; &#160; &#160;domain模式选择，我选择的生产模式，输入2回车；
![](http://images.allposs.com/20160705152021.png)
&#160; &#160; &#160; &#160;选择JDK，使用我们之前安装的，输入1回车；
![](http://images.allposs.com/20160705152045.png)
&#160; &#160; &#160; &#160;选择可选配置，我们暂不配置，输入Next回车；
![](http://images.allposs.com/20160705152113.png)
&#160; &#160; &#160; &#160;开始执行创建，之后显示创建成功。

&#160; &#160; &#160; &#160;这样domain创建完成，domain类似于Tomcat中的webapps，用于放web应用

&#160; &#160; &#160; &#160;启动weblogic

	[root@node1 bin]# /opt/weblogic/weblogic/user_projects/domains/base_domain/bin/startWebLogic.sh 

&#160; &#160; &#160; &#160;中间会提示提示输入用户名密码，就是我们创建domain时配置的用户名与密码。然后使用浏览器，输入http://10.199.200.201:7001/console/login/LoginForm.jsp就可以访问了

注意：

	如果启动时候报：
	java.lang.AssertionError: Could not obtain the localhost address. The most likely cause is an error in the network configuration of this machine.
	则因为主机名得不到解析，直接修改hosts文件可以解决
	

### 6. 配置WebLogic启动参数

#### 6.1 重启服务后不需要输入密码
	
	[root@node1 bin]# cd /opt/weblogic/weblogic/user_projects/domains/base_domain/servers/AdminServer/
	[root@node1 AdminServer]# mkdir security
	输入
	username=weblogic
	password=weblogic123
	[root@node1 AdminServer]# mv boot.properties security/
	重启服务

#### 6.2 配置启动脚本
&#160; &#160; &#160; &#160;因为bin目录下的启动脚本只是让服务在前台工作，一但离开前台或者但是ssh连接就会造成服务断开，所以需要重写一个启动脚本

	[root@node1 weblogic]# vim startweblogic.sh
	#!/bin/sh
	DOMAIN_HOME="/opt/weblogic/weblogic/user_projects/domains/base_domain/"
	nohup ${DOMAIN_HOME}/bin/startWebLogic.sh $* > /dev/null 2>&1 &
	
至此weblogic基本安装完成

## 结束##