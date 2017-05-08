Title: tomcat日志配置
Date: 2016/7/14 14:25:15 
Modified: 2016/7/14 14:25:19   
Category: Servers
Tags: tomcat
Slug: 
Author: allposs


## 简介
&#160; &#160; &#160; &#160;Tomcat默认生成的日志文件catalina.out，随着时间的推移，逐渐增大，可能达到G数量级。当catalina.out文件过大，我们将无法使用过常规编辑工具查看，严重影响系统维护工作。
## 环境

+ 操作系统：CentOS6.5 X86_64
+ Yum源：163源
+ IP地址：
+ DNS：
+ 主机名：

## 软件包

###1. Yum源

	[root@node1 ~]# yum -y groupinstall "Development Tools"

###2. 源码包

	URL:http://pkgs.fedoraproject.org/repo/pkgs/cronolog/cronolog-1.6.2.tar.gz/a44564fd5a5b061a5691b9a837d04979/cronolog-1.6.2.tar.gz
##拓扑图

无
	
## 正文


###1. 修改日志输出级别

####1.1  日志类型与级别

Tomcat 日志分为下面５类：


	catalina 、 localhost 、 manager 、 admin 、 host-manager


每类日志的级别分为如下 7 种：

	SEVERE (highest value) > WARNING > INFO > CONFIG > FINE > FINER > FINEST (lowest value)
####1.2  日志级别的设定方法

&#160; &#160; &#160; &#160;修改 conf/logging.properties 中的内容，设定某类日志的级别

示例：

设置 catalina 日志的级别为： FINE

	1catalina.org.apache.juli.FileHandler.level = FINE

禁用 catalina 日志的输出：

	1catalina.org.apache.juli.FileHandler.level = OFF

输出 catalina 所有的日志消息均输出：

	1catalina.org.apache.juli.FileHandler.level = ALL

###2. 日志分割
####2.1 使用cronolog工具分割
#####2.1.1 cronolog工具下载
	URL:http://pkgs.fedoraproject.org/repo/pkgs/cronolog/cronolog-1.6.2.tar.gz/a44564fd5a5b061a5691b9a837d04979/cronolog-1.6.2.tar.gz
#####2.1.2 cronolog编译安装
	[root@node1 ~]# yum -y groupinstall "Development Tools"
	[root@node1 ~]# tar xvf cronolog-1.6.2.tar.gz
	[root@node1 ~]# ./configure --prefix=/usr/local/cronolog
	[root@node1 ~]# make
	[root@node1 ~]# sudo make install
#####2.1.3 修改Tomcat启动脚本catalina.sh 

a. 修改输出日志路径
修改：

		if [ -z "$CATALINA_OUT" ] ; then
       		CATALINA_OUT="$CATALINA_BASE"/logs/catalina.out
	fi
为：

		if [ -z "$CATALINA_OUT" ] ; then
	      CATALINA_OUT="$CATALINA_BASE"/logs/catalina."$(date +%Y-%m-%d-%H-%M)".out
	fi
b. 删除生成日志文件
注释：

	touch "$CATALINA_OUT"
   为：

	#touch "$CATALINA_OUT"
c. 修改启动脚本参数
修改：

      org.apache.catalina.startup.Bootstrap "$@" start \
      >> "$CATALINA_OUT" 2>&1 "&"

   为：

      org.apache.catalina.startup.Bootstrap "$@" start 2>&1 \
      | /usr/local/sbin/cronolog "$CATALINA_OUT" >> /dev/null &
d. 重启Tomcat
Tomcat输出日志文件分割成功，输出log文件格式为：catalina.2014-08-15.out类型。

	
## 结束##