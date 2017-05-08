Title: CentOS6.5 安装配置DHCP服务
Date: 2016/5/4 11:29:45 
Modified: 2016/5/4 11:29:41 
Category: Servers
Tags: DHCP
Slug: 
Author: allposs



## 简介
&#160; &#160; &#160; &#160;DHCP（Dynamic Host Configuration Protocol，动态主机配置协议）是一个局域网的网络协议，使用UDP协议工作， 主要有两个用途：给内部网络或网络服务供应商自动分配IP地址，给用户或者内部网络管理员作为对所有计算机作中央管理的手段，在RFC 2131中有详细的描述。DHCP有3个端口，其中UDP67和UDP68为正常的DHCP服务端口，分别作为DHCP Server和DHCP Client的服务端口；546号端口用于DHCPv6 Client，而不用于DHCPv4，是为DHCP failover服务，这是需要特别开启的服务，DHCP failover是用来做“双机热备”的。

## 环境

+ 操作系统：CentOS6.5 X86_64
+ Yum源：163源
+ IP地址：10.199.255.15
+ DNS：10.199.255.15
+ 主机名：moniter

## 软件包

###1. Yum源

	[root@moniter ~] yum -y install dhcp

###2. 源码包



## 正文##
###1. 安装DHCP

######1.1 使用YUM安装DHCP

	[root@moniter ~] yum -y install dhcp

###2. 配置DHCP

	[root@moniter ~] vi /etc/ntpd/ntpd.conf
	ddns-update-style none;
	default-lease-time 21600;
	max-lease-time  43200;
	option  domain-name     "allposs.com";
	option  domain-name-servers 10.199.255.1;

	subnet 10.199.255.0 netmask 255.255.255.0{
	range 10.199.255.150 10.199.255.200;
	option subnet-mask 255.255.255.0;
	option routers 10.199.255.1;

	}

全局设置：

	通常我们把subnet 10.199.255.0 netmask 255.255.255.0（不包括subnet这行内容） 以上的内容成为全局。各行含义如下：
	ddns-update-style interim;         表示dhcp服务器和dns服务器的动态信息更新模式。这行必须要有dhcp服务器才能启动以来。
	Default-lease-time 21600;          默认租约时间
	Max-lease-time 43200;         最大租约时间
	Option domain-name "allposs.com";         域名服务器的名称
	Option domaini-servers 10.199.255.254         默认域名服务的ip地址
	Sub 后从“{”开始 到最后一个“}”结束表示子网属性。其主要配置只对大括号里的内容有效。一个配置文件可以有多个子网属性。
	Sub 10.199.255.0 netmask 255.255.255.0;         （意思是我所分配的ip地址所在的网段为10.199.255.0 子网掩码为255.255.255.0 ）
	Range 10.199.255.150 10.199.255.200;         （分配的ip地址范围为10.199.255.150到10.199.255.200）
	Option subnet-mask 255.255.255.0;         （分配ip地址的子网掩码为 255.255.255.0
	Option routers 10.199.255.1;         （分给客户机的网关为10.199.255.1）有时候我们需要为某一个机器配置固定的ip地址，而下面的配置选项满足了这一要求：
	Host server01 {
	Hardware ethernet b0:c0:12:f2:a3:a4;
	Fixed-address 10.199.255.100;
	}
	具体含义和简单意思是“我们给客户机mac地址为b0;c0;12;f2;a3;a4所配置的ip地址为10.199.255.100”。

设置开机启动

	[root@moniter ~] service dhcpd start
	[root@moniter ~] chkconfig –add dhcpd
	[root@moniter ~] chkconifg dhcpd on

## 结束