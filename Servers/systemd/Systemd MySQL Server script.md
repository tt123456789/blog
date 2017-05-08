Title: Systemd MySQL服务脚本
Date: 2016/5/4 11:28:50 
Modified: 2016/5/4 11:28:53
Category: Servers
Tags: Mysql
Slug: 
Author: allposs

##适用环境##
RedHat7.*+MySQL*

##版本号##
V1.0
##作用##
systemd启动Mysql服务脚本



##代码##


	[Unit]
	Description=mysql rdbms

	[Service]
	User=mysql
	Group=mysql
	Type=simple
	GuessMainPID=yes
	ExecStart=/usr/local/mysql/bin/mysqld --defaults-file=/etc/my.cnf
	ExecStop=kill /data/mysql/mysql.pid
	Restart=systemctl stop mysql && systemctl start mysql

	[Install]
	WantedBy=multi-user.target
