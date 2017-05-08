Title: CentOS6.5 配置YUM服务器
Date: 2016/5/3 11:28:40  
Modified: 2016/5/3 11:28:08 
Category: Servers
Tags: yum
Slug: 
Author: allposs

## 简介

&#160; &#160; &#160; &#160;yum（全称为 Yellow dog Updater, Modified）是一个在Fedora和RedHat以及CentOS中的Shell前端软件包管理器。基于RPM包管理，能够从指定的服务器自动下载RPM包并且安装，可以自动处理依赖性关系，并且一次安装所有依赖的软件包，无须繁琐地一次次下载、安装。


## 环境

+ 操作系统：CentOS6.5 X86_64
+ Yum源：本地Yum (详细见 CentOS6.2 配置本地Yum源)
+ IP地址：10.199.255.15
+ DNS：10.199.255.15
+ 主机名：NAS

## 软件包
######1. Yum安装######


	[root@NAS ~]# yum install createrepo


## 正文##

###YUM服务器###

######1. 安装createrepo工具######
安装createrepo时，可以使用本地yum源，或者rpm安装，当然编译也可以。  


	[root@NAS ~]# yum install createrepo

 
######2. 建立repo发布目录######
相关目录可以自定义


 
	[root@NAS ~]# mkdir -p /NAS/http/YUM/CentOS/6.5/{i386,x86_64}
	[root@NAS ~]# mkdir -p /NAS/http/YUM/redhat/6.2/{i386,x86_64}



######3. 下载软件包######
把相关软件包copy到新建目录下，可以从光盘copy.

	
	[root@NAS ~]# cp -Rf /mnt/Packages/ /NAS/http/YUM/CentOS/x86_64/6.5/


######4. 生成meta信息######
用createrepo生成meta信息

	
	[root@NAS 6.5]# createrepo -p -d -o /NAS/http/YUM/CentOS/6.5/x86_64 /NAS/http/YUM/CentOS/6.5/x86_64
	
![](http://image.allposs.cn/20150911002.png)


![](http://image.allposs.cn/20150911001.png)


######5. 安装apache######
可以使用本地yum源或rpm安装，当然编译也行。  


	[root@NAS ~]# yum install httpd


######6. 配置apache######

新建配置文件vrt.conf

	[root@NAS ~]# vim /etc/httpd/conf.d/vrt.conf
	<Virtualhost *:80>
		ServerName yum.allposs.com
		DocumentRoot  /NAS/http/YUM
		DirectoryIndex index.php index.html
		<Directory "/NAS/http/YUM">
 		#开启目录列表索引模式
 			Options Indexes FollowSymLinks
			IndexOptions NameWidth=25 Charset=UTF-8
 			AllowOverride None
			Order allow,deny
 			Allow from all
		</Directory>
	</VirtUalHost>

设置开机启动


	[root@NAS ~]# chkconfig httpd on


######7. 配置防火墙与Selinux######

	[root@NAS ~]# chcon -R -t httpd_sys_content_t /NAS/http/YUM


下图是防火墙需要添加的内容


![](http://image.allposs.cn/20150911003.png)

###客户端配置###

######1. 新建repo文件######

	
	[root@Test-Linux ~]# vim /etc/yum.repos.d/allposs.repo

输入下面内容

	[allposs]
	name=allposs.com
	baseurl=http://yum.allposs.com/RedHat/6.2/x86_64/ 
	#这里的域名可以改为IP地址，当然你也可以和我一样再搭一个DNS
	enabled=1
	gpgcheck=1

![](http://image.allposs.cn/20150911004.png)
######2. 测试######

清空缓存

	[root@Test-Linux ~]# yum clean all


安装软件测试

	[root@Test-Linux ~]# yum install httpd



![](http://image.allposs.cn/20150911005.png) 
yum服务器安装成功。


## 结束##