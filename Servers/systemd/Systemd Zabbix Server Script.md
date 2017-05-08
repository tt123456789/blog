Title: Systemd Zabbix服务端启动脚本
Date: 2016/5/3 11:28:18 
Modified: 2016/5/3 11:28:24 
Category: Servers
Tags: zabbix
Slug: 

Author: allposs
##适用环境##
Redhat7.*+Zabbix*

##版本号##
V1.0

##作用##
Systemd的zabbix服务端启动脚本



##代码##

	[Unit]
	Description=Zabbix Monitor server
	After=network.target

	[Service]
	Type=forking
	PIDFile=/tmp/zabbix_server.pid
	Environment="CONFIG=/etc/zabbix/zabbix_server.conf"
	ExecStart=/usr/local/zabbix/sbin/zabbix_server  -c ${CONFIG}
	ExecStop=/bin/kill `cat $PIDFile`
	RemainAfterExit=yes
	Restart=always

	[Install]
	WantedBy=multi-user.target
