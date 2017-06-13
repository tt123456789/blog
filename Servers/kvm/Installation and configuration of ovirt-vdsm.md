Title: ovrit安装与配置
Date: 2016/5/19 9:48:54 
Modified: 2016/5/19 9:48:57 
Category: Servers
Tags: Ovirt
Slug: 
Author: allposs


## 简介##
&#160; &#160; &#160; &#160;oVirt 是个管理虚拟化的应用程序。言下之意就是你可利用 oVirt 的管理界面（oVirt 引擎）来管理硬件节点、存储及网络资源，并部署及监控在你的数据中心内运行的虚拟机器。oVirt 在理念上与 vSphere 类同。

## 环境##

+ 操作系统：
	node1 centos 7.1
	node2 centos 7.1
	engine centos 6.5 or cento 7.1
+ Yum源：163源 ovirt源 epl源
+ IP地址：
	node1 192.168.1.111/24
	node2 192.168.1.112/24
	engine 192.168.1.110/24
+ DNS：
	node1 114.114.114.114
	node2 114.114.114.114
	engine 114.114.114.114
+ 主机名：
	node1.allposs.com node2.allposs.com engine.allposs.com

## 软件包##

###1. Yum源###


	[root@engine ~]# yum install http://resources.ovirt.org/pub/yum-repo/ovirt-release36.rpm
	[root@engine ~]# yum -y install ovirt-engine
	[root@node1 ~]# yum install -y bridge-utils
	[root@node1 network-scripts]# yum groupinstall "x window system"
	[root@node1 network-scripts]# yum install gnome-classic-session gnome-terminal dejavu-sans-mono-fonts nautilus-open-termina
	[root@node1 network-scripts]# yum install cjkuni-uming-fonts
	[root@node1 network-scripts]# yum install tigervnc-server
	[root@node1 network-scripts]# yum -y install libcanberra-gtk2 qemu-kvm.x86_64 qemu-kvm-tools.x86_64    libvirt.x86_64 libvirt-cim.x86_64 libvirt-client.x86_64 libvirt-java.noarch  libvirt-python.x86_64 libiscsi-1.7.0-5.el6.x86_64  dbus-devel  virt-clone tunctl virt-manager libvirt libvirt-python python-virtinst libvirt-devel
	[root@node1 network-scripts]# yum install http://resources.ovirt.org/releases/ovirt-release/ovirt-release-master.rpm
	[root@node1 network-scripts]# yum install -y vdsm vdsm-cli
###2. 源码包###

##拓扑图##

![](http://images.allposs.com/20160519092940.png)


## 正文##



###1.node0

####1.1 安装Engine

	[root@engine ~]# mkdir /data/iso -p
	[root@engine ~]# yum install http://resources.ovirt.org/pub/yum-repo/ovirt-release36.rpm
	[root@engine ~]# yum -y install ovirt-engine
	[root@engine ~]# engine-setup

如果没有部署dns，可以修改hosts文件

	[root@engine ~]# vim /etc/hosts
	10.199.200.100  engine.allposs.com


![](http://images.allposs.com/20160517221408.png)
![](http://images.allposs.com/20160517222848.png)
![](http://images.allposs.com/20160517222848.png)
![](http://images.allposs.com/20160517223335.png)
![](http://images.allposs.com/20160517223412.png)
![](http://images.allposs.com/20160517223455.png)
![](http://images.allposs.com/20160517223532.png)

	[root@engine ~]# engine-setup
	[ INFO  ] Stage: Initializing
	[ INFO  ] Stage: Environment setup
          Configuration files: ['/etc/ovirt-engine-setup.conf.d/10-packaging-jboss.conf', '/etc/ovirt-engine-setup.conf.d/10-packaging.conf']
          Log file: /var/log/ovirt-engine/setup/ovirt-engine-setup-20160517222643-yyikfb.log
          Version: otopi-1.4.1 (otopi-1.4.1-1.el6)
	[ INFO  ] Stage: Environment packages setup
	[ INFO  ] Yum Downloading: repomddx0tAbtmp.xml (0%)
	[ INFO  ] Stage: Programs detection
	[ INFO  ] Stage: Environment setup
	[ INFO  ] Stage: Environment customization
         
          --== PRODUCT OPTIONS ==--
         
          Configure Engine on this host (Yes, No) [Yes]: 
          Configure VM Console Proxy on this host (Yes, No) [Yes]: 
          Configure WebSocket Proxy on this host (Yes, No) [Yes]: 
         
          --== PACKAGES ==--
         
	[ INFO  ] Checking for product updates...
	[ INFO  ] No product updates found
         
          --== ALL IN ONE CONFIGURATION ==--
         
         
          --== NETWORK CONFIGURATION ==--
         
          Host fully qualified DNS name of this server [engine.allposs.com]: 
          Setup can automatically configure the firewall on this system.
          Note: automatic configuration of the firewall may overwrite current settings.
          Do you want Setup to configure the firewall? (Yes, No) [Yes]: 
	[ INFO  ] iptables will be configured as firewall manager.
         
          --== DATABASE CONFIGURATION ==--
         
          Where is the Engine database located? (Local, Remote) [Local]: 
          Setup can configure the local postgresql server automatically for the engine to run. This may conflict with existing applications.
          Would you like Setup to automatically configure postgresql and create Engine database, or prefer to perform that manually? (Automatic, Manual) [Automatic]: 
         
          --== OVIRT ENGINE CONFIGURATION ==--
         
          Application mode (Virt, Gluster, Both) [Both]: 
          Engine admin password: 
          Confirm engine admin password: 
	[WARNING] Password is weak: it is based on a dictionary word
          Use weak password? (Yes, No) [No]: yes
         
          --== STORAGE CONFIGURATION ==--
         
          Default SAN wipe after delete (Yes, No) [No]: 
         
          --== PKI CONFIGURATION ==--
         
          Organization name for certificate [allposs.com]: 
         
          --== APACHE CONFIGURATION ==--
         
          Setup can configure apache to use SSL using a certificate issued from the internal CA.
          Do you wish Setup to configure that, or prefer to perform that manually? (Automatic, Manual) [Automatic]: 
          Setup can configure the default page of the web server to present the application home page. This may conflict with existing applications.
          Do you wish to set the application as the default page of the web server? (Yes, No) [Yes]: 
         
          --== SYSTEM CONFIGURATION ==--
         
          Configure an NFS share on this server to be used as an ISO Domain? (Yes, No) [Yes]: 
          Local ISO domain path [/var/lib/exports/iso]: /data/iso
         
          Please provide the ACL for the Local ISO domain.
          See the exports(5) manpage for the format.
          Examples:
          - To allow access for host1, host2 and host3, input: host1(rw) host2(rw) host3(rw)
          - To allow access to the entire Internet, input: *(rw)
         
          For more information, see: http://www.ovirt.org/Troubleshooting_NFS_Storage_Issues
         
          Local ISO domain ACL: 
	[ ERROR ] Please specify value
         
          Please provide the ACL for the Local ISO domain.
          See the exports(5) manpage for the format.
          Examples:
          - To allow access for host1, host2 and host3, input: host1(rw) host2(rw) host3(rw)
          - To allow access to the entire Internet, input: *(rw)
         
          For more information, see: http://www.ovirt.org/Troubleshooting_NFS_Storage_Issues
         
          Local ISO domain ACL: *(rw)
          Local ISO domain name [ISO_DOMAIN]: isos
         
          --== MISC CONFIGURATION ==--
         
         
          --== END OF CONFIGURATION ==--
         
	[ INFO  ] Stage: Setup validation
          Generated iptables rules are different from current ones.
          Do you want to review them? (Yes, No) [No]: 
	[WARNING] Warning: Not enough memory is available on the host. Minimum requirement is 4096MB, and 16384MB is recommended.
          Do you want Setup to continue, with amount of memory less than recommended? (Yes, No) [No]: yes
         
          --== CONFIGURATION PREVIEW ==--
         
          Application mode                        : both
          Default SAN wipe after delete           : False
          Firewall manager                        : iptables
          Update Firewall                         : True
          Host FQDN                               : engine.allposs.com
          Engine database secured connection      : False
          Engine database host                    : localhost
          Engine database user name               : engine
          Engine database name                    : engine
          Engine database port                    : 5432
          Engine database host name validation    : False
          Engine installation                     : True
          NFS setup                               : True
          PKI organization                        : allposs.com
          NFS export ACL                          : *(rw)
          NFS mount point                         : /data/iso
          Configure local Engine database         : True
          Set application as default page         : True
          Configure Apache SSL                    : True
          Configure VMConsole Proxy               : True
          Engine Host FQDN                        : engine.allposs.com
          Configure WebSocket Proxy               : True
         
          Please confirm installation settings (OK, Cancel) [OK]: 



安装完成

![](http://images.allposs.com/20160517224109.png)

这个时候就可以通过浏览器访问


###2 node1

####2.1 配置网卡
	
	[root@node1 ~]# yum install -y bridge-utils
	[root@node1 ~]# cd /etc/sysconfig/network-scripts/

em1

	[root@node1 network-scripts]# vi ifcfg-em1

修改为：

	DEVICE=em1
	ONBOOT=yes
	#BRIDGE=ovirtmgmt
   	MASTER=bond1
    SLAVE=yes

em2

	[root@node1 network-scripts]# vi ifcfg-em2

修改为：

	DEVICE=em2
	ONBOOT=yes
	#BRIDGE=ovirtmgmt
   	MASTER=bond1
    SLAVE=yes

em3

	[root@node1 network-scripts]# vi ifcfg-em3

修改为：

	DEVICE=em3
	ONBOOT=yes
	#BRIDGE=ovirtmgmt
   	MASTER=bond1
    SLAVE=yes

em4

	[root@node1 network-scripts]# vi ifcfg-em4

修改为：

	DEVICE=em4
	ONBOOT=yes
	#BRIDGE=ovirtmgmt
   	MASTER=bond1
    SLAVE=yes


p6p1

	[root@node1 network-scripts]# vi ifcfg-p6p1 

写入

	DEVICE=p6p1
	ONBOOT=yes
	#BRIDGE=ovirtmgmt
	MASTER=bond2
	SLAVE=yes


p6p2

	[root@node1 network-scripts]# vi ifcfg-p6p2

写入

	DEVICE=p6p2
	ONBOOT=yes
	#BRIDGE=ovirtmgmt
	MASTER=bond2
	SLAVE=yes


####2.2 增加Bonding

	[root@node1 network-scripts]# echo "alias netdev-bond0 bonding" > /etc/modprobe.d/bonding.conf
	[root@node1 network-scripts]# vi ifcfg-bond1


写入


	DEVICE=bond1
	TYPE=bond
	ONBOOT=yes
	BOOTPROTO=none
	BONDING_OPTS="miimon=80 mode=4"
	BRIDGE=ovirtmgmt


	[root@node1 network-scripts]# vi ifcfg-bond2


写入


	DEVICE=bond2
	TYPE=bond
	ONBOOT=yes
	BOOTPROTO=none
	BONDING_OPTS="miimon=80 mode=4"




	[root@node1 network-scripts]# vi ifcfg-ovirtmgmt

写入

	DEVICE=ovirtmgmt
	TYPE=Bridge
	ONBOOT=yes
	DELAY=0
	BOOTPROTO=static
	IPADDR=192.168.1.111
	NETMASK=255.255.255.0
	GATEWAY=192.168.1.1

配置网络

	[root@node1 network-scripts]# systemctl disable NetworkManager.service
	[root@node1 network-scripts]# chkconfig network on

重启

####2.3 安装桌面
	[root@node1 network-scripts]# yum groupinstall "x window system"
	[root@node1 network-scripts]# yum install gnome-classic-session gnome-terminal dejavu-sans-mono-fonts nautilus-open-termina
	[root@node1 network-scripts]# yum install cjkuni-uming-fonts
	[root@node1 network-scripts]# systemctl set-default graphical.target
	[root@node1 network-scripts]# yum install tigervnc-server
	[root@node1 ~]# vncserver 

输入密码即可


####2.4 安装kvm

	[root@node1 network-scripts]# yum -y install libcanberra-gtk2 qemu-kvm.x86_64 qemu-kvm-tools.x86_64    libvirt.x86_64 libvirt-cim.x86_64 libvirt-client.x86_64 libvirt-java.noarch  libvirt-python.x86_64 libiscsi-1.7.0-5.el6.x86_64  dbus-devel  virt-clone tunctl virt-manager libvirt libvirt-python python-virtinst libvirt-devel


####2.5 安装vsdm

	[root@node1 network-scripts]# yum install http://resources.ovirt.org/releases/ovirt-release/ovirt-release-master.rpm
	[root@node1 network-scripts]# yum install -y vdsm vdsm-cli
	[root@node1 ~]# vim /etc/vdsm/vdsm.conf 
		ssl = false
	[root@node1 ~]# vdsm-tool configure --force

重启




####2.6 注意

7在安装节点的时候vdsm会自动关闭firewall然后安装iptables


####2.7 附：

如果需要在节点机器上安装nfs则需要进行以下配置

#####2.7.1 安装服务

	[root@engine ~]# yum install nfs-utils -y
	[root@engine ~]# systemctl enable nfs.service 
	[root@engine ~]# systemctl enable rpcbind.service
	[root@engine ~]# systemctl start nfs.service
	[root@engine ~]# systemctl start rpcbind.service
	[root@engine ~]# mkdir /data/exports
	[root@engine ~]# vim /etc/exports	
			/data/exports *(rw)	
	[root@engine ~]# exportfs -arv
	[root@engine ~]# chown -R vdsm.kvm /exports/

#####2.7.2 调整nfs端口

	[root@engine ~]# mv /etc/sysconfig/nfs /etc/sysconfig/nfs.bak`date +%Y%m%d%H%M%S` \
	&& cat <<'_EOF' >>/etc/sysconfig/nfs
	RPCRQUOTADOPTS="-p 875"
	LOCKD_TCPPORT=32803
	LOCKD_UDPPORT=32769
	RPCNFSDCOUNT=8
	RPCMOUNTDOPTS="-p 892"
	STATDARG="-p 662 -o 2020"
	_EOF
	[root@engine ~]# systemctl restart nfs.service
	[root@engine ~]# systemctl restart rpcbind.service


#####2.7.3 最后调整防火墙的配置：

	[root@engine ~]# iptables-save >/tmp/rc.firewall.txt
	[root@engine ~]# sed -i '/-A INPUT -i lo -j ACCEPT/a\ \
 	\
 	\
	## NFS related \
	-A INPUT -p tcp -m state --state NEW -m tcp --dport 111 -j ACCEPT \
	-A INPUT -p udp -m state --state NEW -m udp --dport 111 -j ACCEPT \
	-A INPUT -p tcp -m state --state NEW -m tcp --dport 662 -j ACCEPT \
	-A INPUT -p udp -m state --state NEW -m udp --dport 662 -j ACCEPT \
	-A INPUT -p tcp -m state --state NEW -m tcp --dport 875 -j ACCEPT \
	-A INPUT -p udp -m state --state NEW -m udp --dport 875 -j ACCEPT \
	-A INPUT -p tcp -m state --state NEW -m tcp --dport 892 -j ACCEPT \
	-A INPUT -p udp -m state --state NEW -m udp --dport 892 -j ACCEPT \
	-A INPUT -p tcp -m state --state NEW -m tcp --dport 2049 -j ACCEPT \
	-A INPUT -p udp -m state --state NEW -m udp --dport 32769 -j ACCEPT \
	-A INPUT -p tcp -m state --state NEW -m tcp --dport 32803 -j ACCEPT \
	 \
	 \
	' /tmp/rc.firewall.txt
	[root@engine ~]# iptables-restore /tmp/rc.firewall.txt
	[root@engine ~]# service iptables save




###3 node2配置

####3.1 配置网卡
	
	[root@node2 ~]# yum install -y bridge-utils
	[root@node2 ~]# cd /etc/sysconfig/network-scripts/

em1

	[root@node2 network-scripts]# vi ifcfg-em1

修改为：

	DEVICE=em1
	ONBOOT=yes
	#BRIDGE=ovirtmgmt
   	MASTER=bond1
    SLAVE=yes

em2

	[root@node2 network-scripts]# vi ifcfg-em2

修改为：

	DEVICE=em2
	ONBOOT=yes
	#BRIDGE=ovirtmgmt
   	MASTER=bond1
    SLAVE=yes

em3

		[root@node2 network-scripts]# vi ifcfg-em3

修改为：

	DEVICE=em3
	ONBOOT=yes
	#BRIDGE=ovirtmgmt
   	MASTER=bond1
    SLAVE=yes

em4

		[root@node2 network-scripts]# vi ifcfg-em4

修改为：

	DEVICE=em4
	ONBOOT=yes
	#BRIDGE=ovirtmgmt
   	MASTER=bond1
    SLAVE=yes


p6p1

	[root@node2 network-scripts]# vi ifcfg-p6p1 

写入

	DEVICE=p6p1
	ONBOOT=yes
	#BRIDGE=ovirtmgmt
	MASTER=bond2
	SLAVE=yes


p6p2

	[root@node2 network-scripts]# vi ifcfg-p6p2

写入

	DEVICE=p6p2
	ONBOOT=yes
	#BRIDGE=ovirtmgmt
	MASTER=bond2
	SLAVE=yes


####3.2 增加Bonding

	[root@node2 network-scripts]# echo "alias netdev-bond0 bonding" > /etc/modprobe.d/bonding.conf
	[root@node2 network-scripts]# vi ifcfg-bond1


写入


	DEVICE=bond1
	TYPE=bond
	ONBOOT=yes
	BOOTPROTO=none
	BONDING_OPTS="miimon=80 mode=4"
	BRIDGE=ovirtmgmt


	[root@node2 network-scripts]# vi ifcfg-bond2


写入


	DEVICE=bond2
	TYPE=bond
	ONBOOT=yes
	BOOTPROTO=none
	BONDING_OPTS="miimon=80 mode=4"




	[root@node2 network-scripts]# vi ifcfg-ovirtmgmt

写入

	DEVICE=ovirtmgmt
	TYPE=Bridge
	ONBOOT=yes
	DELAY=0
	BOOTPROTO=static
	IPADDR=192.168.1.112
	NETMASK=255.255.255.0
	GATEWAY=192.168.1.1



	[root@node2 network-scripts]# systemctl disable NetworkManager.service
	[root@node2 network-scripts]# chkconfig network on

重启

####3.3 安装桌面
	[root@node2 network-scripts]# yum groupinstall "x window system"
	[root@node2 network-scripts]# yum install gnome-classic-session gnome-terminal dejavu-sans-mono-fonts nautilus-open-termina
	[root@node2 network-scripts]# yum install cjkuni-uming-fonts
	[root@node2 network-scripts]# systemctl set-default graphical.target
	[root@node2 network-scripts]# yum install tigervnc-server
	[root@node2 ~]# vncserver 

输入密码即可


####3.4 安装kvm

	[root@node2 network-scripts]# yum -y install libcanberra-gtk2 qemu-kvm.x86_64 qemu-kvm-tools.x86_64    libvirt.x86_64 libvirt-cim.x86_64 libvirt-client.x86_64 libvirt-java.noarch  libvirt-python.x86_64 libiscsi-1.7.0-5.el6.x86_64  dbus-devel  virt-clone tunctl virt-manager libvirt libvirt-python python-virtinst libvirt-devel


####3.5 安装vsdm

	[root@node2 network-scripts]# yum install http://resources.ovirt.org/releases/ovirt-release/ovirt-release-master.rpm
	[root@node2 network-scripts]# yum install -y vdsm vdsm-cli
	[root@node2 ~]# vim /etc/vdsm/vdsm.conf 
		ssl = false
	[root@node2 ~]# vdsm-tool configure --force

重启

####3.6 注：

至此node2也配置完成，当然如果你要配置node3,node4可以，如果你是原来就已经安装好kvm的，并不需要桌面的只需要执行安装vsdm就行。


###4. Engine管理vsdm

####4.1 登陆
浏览器输入：http://10.199.200.100/ovirt-engine/
![](http://images.allposs.com/20160518205844.png)

进入管理门户

![](http://images.allposs.com/20160518205910.png)


输入账号与密码


![](http://images.allposs.com/20160518210014.png)

####4.2 新建数据中心
Engine默认会有一个初始化数据中心，你也可以不使用他
![](http://images.allposs.com/20160518211137.png)
新建数据中心
![](http://images.allposs.com/20160518211137.png)
根据需要创建数据中心，配额为审计模式，配额为资源的配额如CPU,内存，硬盘和网络。
![](http://images.allposs.com/20160518212414.png)
点击确定后会下面界面，点击配置集群
![](http://images.allposs.com/20160518212507.png)

####4.3 配置集群
#####4.3.1 常规项目
根据自己需要配置
![](http://images.allposs.com/20160518212948.png)

#####4.3.2 优化项目
内存优化默认为无，CPU线程、内存 Balloon和KSM 控件默认都未启用，这里可以根据实际情况修改
![](http://images.allposs.com/20160518213203.png)

######4.3.3 Resilience 策略
默认为迁移虚拟机
![](http://images.allposs.com/20160518213426.png)

#####4.3.4 调度策略
默认不修改，可根据自己需要修改
![](http://images.allposs.com/20160518213616.png)

#####4.3.5 控制台
默认不修改，可根据自己需要修改
![](http://images.allposs.com/20160518213714.png)

#####4.3.6 隔离策略
默认不修改，可根据自己需要修改
![](http://images.allposs.com/20160518213809.png)

点击确定完成配置，会弹出下面界面
![](http://images.allposs.com/20160518214124.png)

####4.3 新建主机
新建主机可以由上一节完成后配置，也可以手动新建。

#####4.3.1 常规项目
根据自己需要配置
![](http://images.allposs.com/20160518214802.png)

#####4.3.2 电源管理项目
默认不起用，这里点启动，然后点加号
![](http://images.allposs.com/20160518214905.png)

下面配置根据自己服务器的情况设定然后确定
![](http://images.allposs.com/20160518215031.png)


#####4.3.3 SPM项目
根据自己需要配置，默认就行

![](http://images.allposs.com/20160518215118.png)
#####4.3.4 控制台项目
默认就行

![](http://images.allposs.com/20160518215218.png)
#####4.3.5 网络供应商项目
默认就行

![](http://images.allposs.com/20160518215254.png)

以上安装完成后还需要配置网络，设立主机网络！这个后面会继续说明！




## 结束##