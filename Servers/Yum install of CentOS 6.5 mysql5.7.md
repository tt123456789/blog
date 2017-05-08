Title: centos 6.5 mysql5.7 yum安装
Date: 2016/7/2 21:17:31 
Modified: 2016/7/2 21:17:34 
Category: Servers
Tags: mysql
Slug: 
Author: allposs


## 简介##
&#160; &#160; &#160; &#160;mysql有好多安装方式，这里只yum安装，基本上操作如下面，后面会补充编译安装，生产环境建议使用编译安装！

## 环境##

+ 操作系统：CentOS6.5 X86_64
+ Yum源：163源，mysql源
+ IP地址：
+ DNS：
+ 主机名：

## 软件包##

###1. Yum源###

###2. 源码包###


##拓扑图##


## 正文##
检测系统是否自带安装mysql

	# yum list installed | grep mysql

删除系统自带的mysql及其依赖

	# yum -y remove mysql-libs.x86_64

配置yum源，并使用最新的yum源

	# wget dev.mysql.com/get/mysql-community-release-el6-5.noarch.rpm
	# yum localinstall mysql-community-release-el6-5.noarch.rpm
	# yum -y install yum-utils
	# yum repolist all | grep mysql
	# yum-config-manager --disable mysql55-community
	# yum-config-manager --disable mysql56-community
	# yum-config-manager --enable mysql57-community-dmr
	# yum repolist enabled | grep mysql
安装mysql-server

	# yum install mysql-community-server
启动mysql
	
	# service mysqld start
在日志文件中找到mysql初始密码，我的是：!j./&&QRi9.r

	# sevice mysqld stop

修改密码

	# mysqld_safe --skip-grant-tables &
	或
	# mysqld_safe --skip-grant-tables --skip-networking &
	这样就跳过远程连接和安装认证
	# mysql -u root -p 
	mysql> update mysql.user set authentication_string=password('ane56!') where user='root' and Host = 'localhost';
	mysql> update mysql.user set authentication_string=password('123qwe') where user='root' and Host = '%';
	mysql> flush privileges;
 	mysql> exit
	# killall -TERM mysqld
	# service mysqld start
密码修改完成

修改权限

	mysql> set global validate_password_policy=0;
	修改validate_password_policy参数的值，取消密码复杂度检查
这样，判断密码的标准就基于密码的长度了。这个由validate_password_length参数来决定。	

validate_password_length参数默认为8，它有最小值的限制，最小值为：
	validate_password_number_count
	validate_password_special_char_count
	(2* validate_password_mixed_case_count)
其中，validate_password_number_count指定了密码中数据的长度，validate_password_special_char_count指定了密码中特殊字符的长度，validate_password_mixed_case_count指定了密码中大小字母的长度。

这些参数，默认值均为1，所以validate_password_length最小值为4，如果你显性指定validate_password_length的值小于4，尽管不会报错，但validate_password_length的值将设为4。

如果修改了validate_password_number_count，validate_password_special_char_count，validate_password_mixed_case_count中任何一个值，则validate_password_length将进行动态修改。
	
	mysql> GRANT ALL PRIVILEGES ON *.* TO root@'%' IDENTIFIED BY 'ane56.123';
	# chkconfig --add mysqld
	# chkconfig --level 345 mysqld on
登陆测试一下
	
## 结束##