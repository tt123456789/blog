Title: ovirt的虚拟机显示IP与FQDN
Date: 2016/6/7 9:37:40  
Modified: 2016/6/7 9:37:45 
Category: Servers
Tags: ovirt
Slug: 
Author: allposs


## 简介##
&#160; &#160; &#160; &#160;这是继上次ovirt安装的基本配置的补充

## 环境##

+ 操作系统：CentOS* redhat*
+ Yum源：ovirt源
+ IP地址：
+ DNS：
+ 主机名：

## 软件包##

###1. Yum源###
	[root@node0 ~]# yum localinstall http://resources.ovirt.org/pub/yum-repo/ovirt-release36.rpm -y
	[root@node0 ~]# yum install ovirt-guest-agent -y
###2. 源码包###

##拓扑图##


## 正文##
###1. 配置yum源

	[root@node0 ~]# yum localinstall http://resources.ovirt.org/pub/yum-repo/ovirt-release36.rpm -y
根据实际情况定版本号，这里最新的是3.6

###2. 安装ovirt-guest-agent

	[root@node0 ~]# yum install ovirt-guest-agent -y

###3. 开启服务并配置

	[root@node0 ~]# service ovirt-guest-agent start
	[root@node0 ~]# chkconfig ovirt-guest-agent on

## 结束##