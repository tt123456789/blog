Title: LVS+Keepalived NET模式配置与安装
Date: 2016/5/4 11:30:03 
Modified: 2016/5/4 11:30:00 
Category: Servers
Tags: LVS+Keepalived
Slug: 
Author: allposs

## 简介##
&#160; &#160; &#160; &#160;LVS的英文全称是Linux Virtual Server，即Linux虚拟服务器。它是我们国家的章文嵩博士的一个开源项目。在linux内存2.6中，它已经成为内核的一部分，在此之前的内核版本则需要重新编译内核。
&#160; &#160; &#160; &#160;LVS主要用于多服务器的负载均衡。它工作在网络层，可以实现高性能，高可用的服务器集群技术。它廉价，可把许多低性能的服务器组合在一起形成一个超级服务器。它易用，配置非常简单，且有多种负载均衡的方法。它稳定可靠，即使在集群的服务器中某台服务器无法正常工作，也不影响整体效果。另外可扩展性也非常好。
&#160; &#160; &#160; &#160;keepalived观其名可知，保持存活，在网络里面就是保持在线了，也就是所谓的高可用或热备，用来防止单点故障(单点故障是指一旦某一点出现故障就会导致整个系统架构的不可用)的发生.
## 环境##

+ 操作系统：Redhat7.1 X86_64
+ Yum源：本地yum源
+ IP地址：
		+ node1 10.199.200.201
		+ node2 10.199.200.202
		+ node3 10.199.200.203
		+ node4 10.199.200.204
		+ VIP 
+ DNS：10.199.200.15
+ NTP:10.199.200.15
+ 主机名：
		+ node1
		+ node2
		+ node3
		+ node4

## 软件包##

###1. Yum源###
	[root@node1 ~]# yum install ipvsadm keepalived
	[root@node2 ~]# yum install ipvsadm keepalived
	[root@node3 ~]# yum install httpd
	[root@node4 ~]# yum install httpd
###2. 源码包###

##拓扑图##
![](http://7xt2ce.com2.z0.glb.clouddn.com/20160118001.png)


## 正文##
###1.node1配置###
####1.1安装lvs+keepalived####


	[root@node1 ~]# yum install ipvsadm keepalived
	[root@node1 ~]# systemctl enable keepalived.service
	[root@node1 ~]# systemctl start keepalived.service 
	[root@node1 ~]# systemctl stop firewalld.service 
	[root@node1 ~]# systemctl disable firewalld.service 
	[root@node1 ~]# sed -i 's/SELINUX=enforcing/#SELINUX=enforcing/'/etc/selinux/config
	[root@node1 ~]#  sed -i 's/SELINUXTYPE=targeted/#SELINUXTYPE=targeted/'/etc/selinux/config
	[root@node1 ~]# echo -e "SELINUX=disabled" >>/etc/selinux/config

开启路由转发

	[root@node1 ~]# echo "net.ipv4.ip_forward=1" >  /etc/sysctl.conf
	[root@node1 ~]# sysctl -p


####1.2配置keepalived####

	[root@node1 ~]# vim /etc/keepalived/keepalived.conf 

修改为

	! Configuration File for keepalived

	global_defs {
   		router_id LVS_MASTER
	}

	vrrp_sync_group VG1 {
		group {
			VIP1
			DIP1
	      	}
	}

	vrrp_instance VIP1 {
    	state MASTER
    	interface eno33554968
    	virtual_router_id 50
    	priority 100
    	advert_int 1
    	authentication {
        	auth_type PASS
        	auth_pass 1111
    	}
    	virtual_ipaddress {
        	172.16.10.10/24

    	}
	}


	vrrp_instance DIP1 {
    	state MASTER
    	interface eno16777728
    	virtual_router_id 60
    	priority 100
    	advert_int 1
    	authentication {
        	auth_type PASS
        	auth_pass 1111
    	}
    	virtual_ipaddress {
        	10.199.200.10/24

    	}
	}
#DIP1
	virtual_server 172.16.10.10 80 {
    	delay_loop 6
    	lb_algo rr
    	lb_kind NAT
    	persistence_timeout 5
    	protocol TCP

    	real_server 172.16.10.201 80 {
        	weight 3
        	TCP_CHECK {
            	connect_timeout 3
            	nb_get_retry 3
            	delay_before_retry 3
            	connect_port 80
        	}
    	}

    	real_server 172.16.10.202 80 {
        	weight 3
        	TCP_CHECK {
            	connect_timeout 3
            	nb_get_retry 3
            	delay_before_retry 3
            	connect_port 80
        	}
    	}
	}

#VIP1
	virtual_server 10.199.200.10 80 {
    	delay_loop 6
    	lb_algo rr
    	lb_kind NAT
    	persistence_timeout 5
    	protocol TCP

    	real_server 172.16.10.203 80 {
        	weight 3
        	TCP_CHECK {
            	connect_timeout 3
            	nb_get_retry 3
            	delay_before_retry 3
            	connect_port 80
        	}
    	}

    	real_server 172.16.10.204 80 {
        	weight 3
        	TCP_CHECK {
            	connect_timeout 3
            	nb_get_retry 3
            	delay_before_retry 3
            	connect_port 80
        	}
    	}
	}


###2.node2配置###
####2.1安装lvs+keepalived####


	[root@node2 ~]# yum install ipvsadm keepalived
	[root@node2 ~]# systemctl enable keepalived.service
	[root@node2 ~]# systemctl start keepalived.service 
	[root@node2 ~]# systemctl stop firewalld.service 
	[root@node2 ~]# systemctl disable firewalld.service 
	[root@node2 ~]# sed -i 's/SELINUX=enforcing/#SELINUX=enforcing/'/etc/selinux/config
	[root@node2 ~]#  sed -i 's/SELINUXTYPE=targeted/#SELINUXTYPE=targeted/'/etc/selinux/config
	[root@node1 ~]# echo -e "SELINUX=disabled" >>/etc/selinux/config

开启路由转发

	[root@node2 ~]# echo "net.ipv4.ip_forward=1" >  /etc/sysctl.conf
	[root@node2 ~]# sysctl -p


####2.2配置keepalived####

	[root@node2 ~]# vim /etc/keepalived/keepalived.conf 

修改为

	! Configuration File for keepalived

	global_defs {
   		router_id LVS_BACKUP
	}
	vrrp_sync_group VG1 {
		group {
			VIP1
			DIP1
	      }
	}

	vrrp_instance VIP1 {
    	state BACKUP
    	interface eno33554968
    	virtual_router_id 50
    	priority 80
    	advert_int 1
    	authentication {
        	auth_type PASS
        	auth_pass 1111
    	}
    	virtual_ipaddress {
        	172.16.10.10/24

    	}
	}


	vrrp_instance DIP1 {
    	state BACKUP
    	interface eno16777728
    	virtual_router_id 60
    	priority 80
    	advert_int 1
    	authentication {
        	auth_type PASS
        	auth_pass 1111
    	}
    	virtual_ipaddress {
        	10.199.200.10/24

    	}
	}

#DIP1
	virtual_server 172.16.10.10 80 {
    	delay_loop 6
    	lb_algo rr
    	lb_kind NAT
    	persistence_timeout 5
    	protocol TCP

    	real_server 172.16.10.201 80 {
        	weight 3
        	TCP_CHECK {
            	connect_timeout 3
            	nb_get_retry 3
            	delay_before_retry 3
            	connect_port 80
        	}
    	}

    	real_server 172.16.10.202 80 {
        	weight 3
        	TCP_CHECK {
            	connect_timeout 3
            	nb_get_retry 3
           	 	delay_before_retry 3
            	connect_port 80
        	}
    	}
	}

#VIP1
	virtual_server 10.199.200.10 80 {
    	delay_loop 6
    	lb_algo rr
    	lb_kind NAT
    	persistence_timeout 5
    	protocol TCP

    	real_server 172.16.10.203 80 {
        	weight 3
        	TCP_CHECK {
            	connect_timeout 3
            	nb_get_retry 3
            	delay_before_retry 3
            	connect_port 80
        	}
    	}

    	real_server 172.16.10.204 80 {
        	weight 3
        	TCP_CHECK {
            	connect_timeout 3
            	nb_get_retry 3
            	delay_before_retry 3
            	connect_port 80
        	}
    	}
	}


###3.node3配置网络###
####3.1 修改网络配置####
修改网关为DIP

	[root@node3 ~]# nmcli connection edit eno33554968
	nmcli> set ipv4.method 
	Allowed values for 'method' property: auto, link-local, manual, shared, disabled
	Enter 'method' value: manual
	nmcli> set ipv4.addresses 172.16.10.203/24
	nmcli> set ipv4.gateway 172.16.10.10
	nmcli> save
	Connection 'eno33554968' (db411fc5-1c3a-480c-a2a3-17e0968c45e6) successfully updated.
	nmcli> quit
	[root@node3 ~]# nmcli connection up eno33554968 


####3.2 安装配置httpd####

	[root@node3 ~]# yum install httpd
	[root@node3 ~]# firewall-cmd --permanent --add-service=http
	[root@node3 ~]# firewall-cmd --reload 
	[root@node3 ~]# systemctl enable httpd.service 
	[root@node3 ~]# systemctl start httpd.service

	[root@node3 ~]# vim /var/www/html/index.html

写入

	<h1>WEB1/10,199,200.203</h1>



###4.node4配置网络###
####4.1 修改网络配置####
修改网关为DIP

	[root@node4 ~]# nmcli connection edit eno33554968
	nmcli> set ipv4.method 
	Allowed values for 'method' property: auto, link-local, manual, shared, disabled
	Enter 'method' value: manual
	nmcli> set ipv4.addresses 172.16.10.204/24
	nmcli> set ipv4.gateway 172.16.10.10
	nmcli> save
	Connection 'eno33554968' (db411fc5-1c3a-480c-a2a3-17e0968c45e6) successfully updated.
	nmcli> quit
	[root@node4 ~]# nmcli connection up eno33554968 

####4.2安装配置httpd####

	[root@node4 ~]# yum install httpd
	[root@node4 ~]# firewall-cmd --permanent --add-service=http
	[root@node4 ~]# firewall-cmd --reload 
	[root@node4 ~]# systemctl enable httpd.service 
	[root@node4 ~]# systemctl start httpd.service

	[root@node4 ~]# vim /var/www/html/index.html

写入

	<h1>WEB1/10,199,200.204/h1>






## 结束##