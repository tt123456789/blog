Title: ISCSI存储配置
Date: 2015-05-03 14:28
Modified: 2015-05-03 18:28
Category: Servers
Tags: ISCSI
Slug: 
Author: allposs


## 简介##
&#160; &#160; &#160; &#160;iSCSI（internet SCSI）技术由IBM公司研究开发，是一个供硬件设备使用的、可以在IP协议的上层运行的SCSI指令集，这种指令集合可以实现在IP网络上运行SCSI协议，使其能够在诸如高速千兆以太网上进行路由选择。iSCSI技术是一种新储存技术，该技术是将现有SCSI接口与以太网络(Ethernet)技术结合，使服务器可与使用IP网络的储存装置互相交换资料.

&#160; &#160; &#160; &#160;iSCSI是一种基于TCP/IP 的协议，用来建立和管理IP存储设备、主机和客户机等之间的相互连接，并创建存储区域网络（SAN）。SAN 使得SCSI 协议应用于高速数据传输网络成为可能，这种传输以数据块级别（block-level）在多个数据存储网络间进行。SCSI 结构基于C/S模式，其通常应用环境是：设备互相靠近，并且这些设备由SCSI 总线连接。

&#160; &#160; &#160; &#160;iSCSI 的主要功能是在TCP/IP 网络上的主机系统（启动器 initiator）和存储设备（目标器 target）之间进行大量数据的封装和可靠传输过程。

## 环境##

+ 操作系统：CentOS7.1 X86_64
+ Yum源：163源
+ IP地址：
			target: 10.199.200.15
		 	client: 10.199.200.40	

+ DNS：
+ 主机名：

## 软件包##

###1. Yum源###

	[root@Monitor ~]# yum -y install targetcli iscsi-initiator-utils* 
	[root@node1 ~]# yum -y install iscsi-initiator-utils*
###2. 源码包###

##拓扑图##


## 正文##
服务器端：
安装相关软件包，并修改iqn

	[root@Monitor ~]# yum -y install targetcli iscsi-initiator-utils*
修改iqn号
	[root@Monitor ~]# vi /etc/iscsi/initiatorname.iscsi 
修改为：iqn.1994-05.com.allposs:monitor

配置target

	[root@Monitor ~]# targetcli 
	/> cd backstores/block/
	/backstores/block> create desk1 /dev/sdc1
	/backstores/block> cd /iscsi/
	/iscsi> create iqn.1994-05.com.allposs:monitor
	/iscsi> cd iqn.1994-05.com.allposs:monitor/tpg1/portals/
	/iscsi/iqn.19.../tpg1/portals> delete 0.0.0.0 ip_port=3260 
	/iscsi/iqn.19.../tpg1/portals> create 10.199.200.15 ip_port=3260
	/iscsi/iqn.19.../tpg1/portals> cd /iscsi/iqn.1994-05.com.allposs:monitor/tpg1/acls 
	/iscsi/iqn.19...tor/tpg1/acls> create iqn.1994-05.com.allposs.node1:ovirth
	/iscsi/iqn.19...tor/tpg1/acls> cd /iscsi/iqn.1994-05.com.allposs:monitor/tpg1/luns 
	/iscsi/iqn.19...tor/tpg1/luns> cd /
	/> saveconfig

设置开机启动

	[root@Monitor ~]# service tgtd start
	[root@Monitor ~]# chkconfig tgtd on


客户端:
安装并配置iqn
1、安装iscsi-initiators

	[root@node1 ~]# yum -y install iscsi-initiator-utils*

修改iqn号

	[root@node1 ~]# vi /etc/iscsi/initiatorname.iscsi 
	修改为：iqn.1994-05.com.allposs.node1:ovirth

设置开机启动

	[root@node1 ~]# service tgtd start
	[root@node1 ~]# chkconfig tgtd on

发现targets

	[root@node1 ~]# iscsiadm -m discovery -t sendtargets -p 10.199.200.15:3260
	扫描信息都写入了/var/lib/iscsi/nodes/iqn.1994-05.com.allposs.node1:ovirth/10.199.200.15,3260,1/default 文件中了。

连接存储

	[root@node1 ~]# iscsiadm -m node -T iqn.1994-05.com.allposs:monitor -p 10.199.200.15:3260 -l


检验

	[root@node1 ~]# lsblk -l

## 结束##