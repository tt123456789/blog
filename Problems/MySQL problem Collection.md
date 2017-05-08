Title: MySql问题合集
Date: 2016/5/14 9:29:33 
Modified: 2016/5/14 9:29:36 
Category: Problems
Tags: 问题合集
Slug: 
Author: allposs


##1. 问题一
###环境
+ 操作系统：无
+ Yum源：无
+ IP地址： 无 
+ DNS：  无
+ 主机名：无
+ URL：无
###现象


	CMake Error at cmake/boost.cmake:76 (MESSAGE):
  		You can download it with -DDOWNLOAD_BOOST=1 -DWITH_BOOST=<directory>
 
  		This CMake script will look for boost in <directory>.  If it is not there,
  		it will download and unpack it (in that directory) for you.
 
  		If you are inside a firewall, you may need to use an http proxy:
 
  		export http_proxy=http://example.com:80
 
	Call Stack (most recent call first):
  		cmake/boost.cmake:228 (COULD_NOT_FIND_BOOST)
  		CMakeLists.txt:452 (INCLUDE)
	-- Configuring incomplete, errors occurred!


###错误日志

	CMake Error at cmake/boost.cmake:76 (MESSAGE):

###分析

没有boost包

###解决

####第一种

在预编译时添加相应的选项：

	[root@kongxl mysql-5.7.6-m16]# cmake . -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DMYSQL_DATADIR=/data/mysql -DWITH_INNOBASE_STORAGE_ENGINE=1 -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci -DDOWNLOAD_BOOST=1 -DWITH_BOOST=/usr/local/boost

####第二种
	或者下载一个boost包，放到/usr/local/boost目录下，然后在cmake后面加选项-DWITH_BOOST=/usr/local/boost

	[root@kongxl mysql-5.7.6-m16]# cmake . -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DMYSQL_DATADIR=/data/mysql -DWITH_INNOBASE_STORAGE_ENGINE=1 -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci -DWITH_BOOST=/usr/local/boost
