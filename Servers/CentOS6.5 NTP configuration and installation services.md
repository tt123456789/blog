Title: CentOS6.5 配置与安装NTP服务
Date: 2016/5/7 17:36:24   
Modified: 2016/5/7 17:36:27  
Category: Servers
Tags: NTP
Slug: 
Author: allposs
# 简介##
&#160; &#160; &#160; &#160; NTP（Network Time Protocol，网络时间协议）是用来使网络中的各个计算机时间同步的一种协议。它的用途是把计算机的时钟同步到世界协调时UTC，其精度在局域网内可达0.1ms，在互联网上绝大多数的地方其精度可以达到1-50ms。

&#160; &#160; &#160; &#160;它可以使计算机对其服务器或时钟源（如石英钟，GPS等等）进行时间同步，它可以提供高精准度的时间校正，而且可以使用加密确认的方式来防止恶毒的协议攻击。

## 环境##

+ 操作系统：CentOS6.5 X86_64
+ Yum源：153源
+ IP地址：10.199.255.15
+ DNS：10.199.255.15
+ 主机名：NAS

## 软件包##

###1. Yum源###

	[root@NAS ~]# yum -y install ntp ntpdata

###2. 源码包###



## 正文##

###1. 安装NTP软件包###

	[root@NAS ~]#yum -y install ntp ntpdata/*yum安装NTP服务*/
	[root@NAS ~]# chkconfig --add ntpd /*添加NTP*/
	[root@NAS ~]# chkconfig ntpd on /*开机自启动NTP*/
修改防火墙
	[root@NAS ~]# vi /etc/sysconfig/iptables
	-A INPUT -p tcp -m tcp --dport 123 -j ACCEPT

###2.修改NTP配置文件###

	vi /etc/ntp.conf
***************************************************************

	# For more information about this file, see the man pages
	# ntp.conf(5), ntp_acc(5), ntp_auth(5), ntp_clock(5), ntp_misc(5), ntp_mon(5).

	driftfile /var/lib/ntp/drift
	restrict default ignore 设置默认策略为拒绝所有访问方式的请求
	# Permit time synchronization with our time source, but do not
	# permit the source to query or modify the service on this system.
	restrict default kod nomodify notrap nopeer noquery
	restrict -6 default kod nomodify notrap nopeer noquery

	# Permit all access over the loopback interface. This could
	# be tightened as well, but to do so would effect some of
	# the administrative functions.
	restrict 127.0.0.1
	restrict -6 ::1

	# Hosts on local network are less restricted.
	restrict 10.199.255.0 mask 255.255.255.0 nomodify notrap 允许局域网内机器同步时间

	# Use public servers from the pool.ntp.org project.
	# Please consider joining the pool (http://www.pool.ntp.org/join.html).
	server 210.72.145.44 perfer   # 中国国家受时中心
	server 202.112.10.36             # 1.cn.pool.ntp.org
	server 59.124.196.83             # 0.asia.pool.ntp.org

	#broadcast 192.168.1.255 autokey # broadcast server
	#broadcastclient # broadcast client
	#broadcast 224.0.1.1 autokey # multicast server
	#multicastclient 224.0.1.1 # multicast client
	#manycastserver 239.255.254.254 # manycast server
	#manycastclient 239.255.254.254 autokey # manycast client
	# allow update time by the upper server 

	# 允许上层时间服务器主动修改本机时间
	restrict 210.72.145.44 nomodify notrap noquery
	restrict 202.112.10.36 nomodify notrap noquery
	restrict 59.124.196.83 nomodify notrap noquery

	# Undisciplined Local Clock. This is a fake driver intended for backup
	# and when no outside source of synchronized time is available.
	# 外部时间服务器不可用时，以本地时间作为时间服务
	server  127.127.1.0     # local clock
	fudge   127.127.1.0 stratum 10

	# Enable public key cryptography.
	#crypto

	includefile /etc/ntp/crypto/pw

	# Key file containing the keys and key identifiers used when operating
	# with symmetric key cryptography.
	keys /etc/ntp/keys

	# Specify the key identifiers which are trusted.
	#trustedkey 4 8 42

	# Specify the key identifier to use with the ntpdc utility.
	#requestkey 8

	# Specify the key identifier to use with the ntpq utility.
	#controlkey 8

	# Enable writing of statistics records.
	#statistics clockstats cryptostats loops

***************************************************************

配置文件修改完成，保存退出，启动服务。

	[root@NAS ~]# service ntpd start

启动后，一般需要5-10分钟左右的时候才能与外部时间服务器开始同步时间。可以通过命令查询NTPD服务情况。
查看服务连接和监听

	[root@NAS ~]# netstat -tlunp | grep ntp

ntpq -p 查看网络中的NTP服务器，同时显示客户端和每个服务器的关系

	[root@NAS ~]# ntpq -p
	[root@NAS ~]# ntpq -p

位置	标志	含义

	符号
	\*				响应的NTP服务器和最精确的服务器
	\+				响应这个查询请求的NTP服务器	
	blank(空格)		没有响应的NTP服务器	
	标题	remote		响应这个请求的NTP服务器的名称
	refid			NTP服务器使用的更高一级服务器的名称	
	st				正在响应请求的NTP服务器的级别	
	when			上一次成功请求之后到现在的秒数	
	poll			本地和远程服务器多少时间进行一次同步，单位秒，在一开始运行NTP的时候这个poll值会比较小，服务器同步的频率大，可以尽快调整到正确的时间范围，之后poll值会逐渐增大，同步的频率也就会相应减小	
	reach			用来测试能否和服务器连接，是一个八进制值，每成功连接一次它的值就会增加	
	delay			从本地机发送同步要求到ntp服务器的往返时间	
	offset			主机通过NTP时钟同步与所同步时间源的时间偏移量，单位为毫秒，offset越接近于0，主机和ntp服务器的时间越接近	
	jitter			统计了在特定个连续的连接数里offset的分布情况。简单地说这个数值的绝对值越小，主机的时间就越精确	

ntpstat 命令查看时间同步状态，这个一般需要5-10分钟后才能成功连接和同步。所以，服务器启动后需要稍等下。
刚启动的时候，一般是：

	[root@NAS ~]# ntpstat
	unsynchronised
	time server re-starting
	polling server every 64 s
连接并同步后:

	synchronised to NTP server (202.112.10.36) at stratum 3
	time correct to within 275 ms
	polling server every 256 s

内网的NTPD服务已经配置完成，如果所有正常后，开始配置内网的其他设备与这台服务器作为时间同步服务。

## 结束##