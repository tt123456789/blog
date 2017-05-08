Title: Redhat6.2 配置本地Yum源
Date: 2016/7/2 21:21:04   
Modified: 2016/7/2 21:21:07   
Category: Servers
Tags: Yum
Slug: 
Author: allposs
## 简介
&#160; &#160; &#160; &#160;Yum（全称为 Yellow dog Updater, Modified）是一个在Fedora和RedHat以及CentOS中的Shell前端软件包管理器。基于RPM包管理，能够从指定的服务器自动下载RPM包并且安装，可以自动处理依赖性关系，并且一次安装所有依赖的软件包，无须繁琐地一次次下载、安装。

## 环境

+ 操作系统：Redhat6.2 X84_64
+ Yum源：本地源
+ IP地址：10.199.255.15
+ DNS：10.199.255.15
+ 主机名：redhat6.2

## 软件包

###1. Yum源

###2. 源码包



## 正文
###1. 准备工作

	# mount –t iso9660 /dev/cdrom /mnt （这里是把光驱挂载到/mnt目录下，当然其它目录也可以）
	（# mount -o loop /mnt/hgfs/shared/rhel-server-6.0-i386-dvd.iso  /mnt 如果是镜像文件的话可以用这种方法挂载）
	# cp –Rf /mnt/* /data/dvd (把光盘文件拷贝到/data/dvd目录下，如果不想让yum长期生效可以不用这样操作)

###2. 删除原有repo文件

	# rm -rf /etc/yum.repos.d/redhat.repo
	# rm -rf /etc/yum.repos.d/rhel-source.repo

 
###3. 新建repo文件

	# vi /etc/yum.repos.d/dvd.repo

写入以下内容：


	[dvd]
	name=my_local_yumsource
	baseurl=file:///data/dvd/
	enable=1
	gpgcheck=0

保存并退出。
 
注：

[]：

中括号中的是yum源 id，id可以随意命名，不过要注意的是不能存在
相同的id，因为id是用来标识不同容器的；

name：

后接yum源的名称，用来说明容器，随意命名；

baseurl：

yum源的地址，如果是网络地址，就用http://，如果本地地址，就用
file://。我们这里用的就是本地地址。注意上面的之所以是三个“///”，
是因为第三个“/”表示根目录。

enable：

表示这个容器是否启用。启用就设置为1，不启用就设置为0。

gpgcheck：

表示是否检查rpm文件的数字签名。检查就设置为1，不检查就设置为0。

gpgkey：

就是数字签名的公钥文件所在位置。如果gpgcheck值为1，此处就需要指
定gpgkey文件的位置，如果gpgcheck值为0 ，就不需要此项了。上面的
gpgcheck设置为0，此处可以没有gpgkey。

	# yum clean all （清空所有yum缓存）

 
###4、测试

	# yum install httpd (安装apache,如果没有报错则本地yum源做好了)


## 结束