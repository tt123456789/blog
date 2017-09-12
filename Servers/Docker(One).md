Title: Docker（一）
Date: 9:14 2017/6/28
Modified: 9:14 2017/6/28
Category: Servers
Tags: Docker
Slug: 
Author: Temp



##简介

&#160; &#160; &#160; &#160; Docker 是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。容器是完全使用沙箱机制，相互之间不会有任何接口。
&#160; &#160; &#160; &#160; Docker 使用客户端-服务器 (C/S) 架构模式，使用远程API来管理和创建Docker容器。Docker 容器通过 Docker 镜像来创建。容器与镜像的关系类似于面向对象编程中的对象与类。

##说明

&#160; &#160; &#160; &#160; 这篇文章是我根据自己学习与实践产生问题所写，有些内容参考的是网上资源并实践正确后摘抄下来的。

​		

##环境


+ 主机IP：10.20.0.201
+ 主机名：DNode01.example.com
+ 主机配置：1CPU,2G内存,30G+100G硬盘
+ 系统版本：CentOS7 1611

##软件包

​	

##安装配置
###1. Docker安装
####1.1 CentOS Yum安装

	# yum install -y docker

####2.2 修改yum源安装

	# vim /etc/yum.repos.d/docker.repo

		[dockerrepo]
		name=Docker Repository
		baseurl=http://yum.dockerproject.org/repo/main/centos/7/
		enabled=1
		gpgcheck=0
	# yum clean all
	# yum update
	# yum install -y docker

###2. 启动docker

	# systemctl enabled docker.service
	# systemctl start docker.service

##基础操作

###1. 镜像操作

1.1 修改镜像默认源

	# vim /etc/sysconfig/docker
		增加：ADD_REGISTRY='--add-registry hub.c.163.com'

1.2 Pull镜像

	# docker pull public/centos
	默认镜像查找顺序：本地->默认镜像仓库
	参数pull子参数：
		-a,-all-tags=true|false			是否获取仓库中的所有镜像，默认为否。				# docker pull -a centos

1.3 查找镜像

	# docker search centos*
	参数serch子参数：
		--automated=true|false			仅显示自动创建的镜像，默认为否						# docker search --automated centos*
		--no-trunc=true|false			输出信息不截断显示，默认否							# docker search --no-trunc centos*
		-s,--stars=X					仅显示评价为指定星级以上的镜像，默认为0				# docker search -s 3 centos*

1.4 查看镜像信息
​	
	# docker images
	参数images子参数：
		-a,-all=true|false				 列出所有的镜像文件（包括临时文件）,默认为否		# docker images --all=true	
		--digests=true|false			 列出镜像的数字摘要值，默认否						# docker images --digests=true
		-f,--filter=[]					 过滤列出的镜像，默认为空							# docker images --filter "dangling=true"

1.5 添加镜像标签

	# docker tag hub.c.163.com/public/centos mycentos

1.6 查看镜像详细信息
​	
	# docker inspect mycentos

1.7 查看镜像历史

	# docker history mycentos

1.8 删除镜像

	使用标签删除镜像

		# docker rmi mycentos

		当多个镜像标签时,删除其中一个标签时，并不影响镜像本身，但是删除最好一个标签时会删除镜像本身。

	使用ID删除镜像

		# docker rmi 997f0ed97903

		当镜像存在标签时，使用ID删除镜像会先删除标签，然后再删除镜像本身，注意有容器依赖这个镜像时是无法删除的。

1.9	导出与导入镜像
​	
	# docker save -o blog.tar blog:0.1
	# docker load --input blog.tar

###2. 容器操作
2.1 创建容器

	# docker create -it blog:0.1
	使用create创建的容器默认是停止的

create命令与容器运行模式相关的选项
![](http://images.allposs.com/20170605133917.png)

create命令与容器运行环境和配置相关的选项
![](http://images.allposs.com/20170605133918.png)

create命令与容器资源限制和安全保护相关的选项：
![](http://images.allposs.com/20170605133919.png)
![](http://images.allposs.com/20170605133920.png)

	其他比较重要的选项：
		-l,--label=[]					以键值对方式指定容器的标签信息
		--label-file=[]					从文件中读取标签信息

2.2 查看容器

	# docker ps -a

2.3 启动容器

	# docker start 3b12dcc7adab

2.4 新建容器并启动

	# docker run -it mycentos /bin/echo "blog.allposs.com"
	使用docker run来创建启动容器时，docker后台执行的操作：
		检查本地是否存在镜像，不存在就从默认的仓库下载
		利用镜像创建一个容器create，并启动，所以create参数同样适用于run
		分配一个文件系统给容器，并在只读的镜像层外挂一层可读层
		从宿主配置的网桥接口桥接一个接口到容器
		从网桥地址池配置一个IP给容器
		执行用户指定的应用程序
		执行完成后容器自动终止
		
	参数run子参数：
		-d, --detach=false         		指定容器运行于前台还是后台，默认为false     
		-i, --interactive=false    		打开STDIN，用于控制台交互    
		-t, --tty=false            		分配tty设备，该可以支持终端登录，默认为false    
		-u, --user=""              		指定容器的用户    
		-a, --attach=[]            		登录容器（必须是以docker run -d启动的容器）  
		-w, --workdir=""           		指定容器的工作目录   
		-c, --cpu-shares=0         		设置容器CPU权重，在CPU共享场景使用    
		-e, --env=[]               		指定环境变量，容器中可以使用该环境变量    
		-m, --memory=""            		指定容器的内存上限    
		-P, --publish-all=false    		指定容器暴露的端口    
		-p, --publish=[]           		指定容器暴露的端口   
		-h, --hostname=""          		指定容器的主机名    
		-v, --volume=[]            		给容器挂载存储卷，挂载到容器的某个目录    
		--volumes-from=[]          		给容器挂载其他容器上的卷，挂载到容器的某个目录  
		--cap-add=[]               		添加权限，权限清单详见：http://linux.die.net/man/7/capabilities    
		--cap-drop=[]              		删除权限，权限清单详见：http://linux.die.net/man/7/capabilities    
		--cidfile=""               		运行容器后，在指定文件中写入容器PID值，一种典型的监控系统用法    
		--cpuset=""                		设置容器可以使用哪些CPU，此参数可以用来容器独占CPU    
		--device=[]                		添加主机设备给容器，相当于设备直通    
		--dns=[]                   		指定容器的dns服务器    
		--dns-search=[]            		指定容器的dns搜索域名，写入到容器的/etc/resolv.conf文件    
		--entrypoint=""            		覆盖image的入口点    
		--env-file=[]              		指定环境变量文件，文件格式为每行一个环境变量    
		--expose=[]                		指定容器暴露的端口，即修改镜像的暴露端口    
		--link=[]                  		指定容器间的关联，使用其他容器的IP、env等信息    
		--lxc-conf=[]              		指定容器的配置文件，只有在指定--exec-driver=lxc时使用    
		--name=""                  		指定容器名字，后续可以通过名字进行容器管理，links特性需要使用名字    
		--net="bridge"             		容器网络设置:bridge 使用docker daemon指定的网桥，host容器使用主机的网络，container:NAME_or_ID  >//使用其他容器的网路，共享IP和PORT等网络资源，none 容器使用自己的网络（类似--net=bridge），但是不进行配置   
		--privileged=false         		指定容器是否为特权容器，特权容器拥有所有的capabilities    
		--restart="no"             		指定容器停止后的重启策略:  no：容器退出时不重启，on-failure：容器故障退出（返回值非零）时重启，always：容器退出时总是重启    
		--rm=false                 		指定容器停止后自动删除容器(不支持以docker run -d启动的容器)    
		--sig-proxy=true           		设置由代理接受并处理信号，但是SIGCHLD、SIGSTOP和SIGKILL不能被代理   	


​		
2.5 获取容器输出信息

	# docker logs 918b5e6f768a

2.6 终止容器

	# docker stop 918b5e6f768a

2.7 进入容器
​	

	# docker attach happy_heyrovsky
	参数attach子参数：
		--detach-keys[=[]]				指定退出attach模式的快捷键序列，默认是CTRL-p CTTL-q
		--no-stdin=true|false			是否关闭标准输入，默认保持打开
		--sig-proxy=true|false			是否代理收到的系统信号给应用进程，默认为true
		
	# docker exec -it 3b12dcc7adab /bin/bash
	参数exec子参数：
		-i,--interactive=true|false		打开标准输入接受用户输入命令，默认为false
		--privileged=true|false			是否给执行命令以最高权限，默认为false
		-t,--tty=true|false				分配为终端，默认为false
		-u,--user=""					执行命令的用户名和ID

2.8 删除容器
​	
	# docker stop 3b12dcc7ad
	# docker rm 3b12dcc7adab
	参数rm子参数：
		-f,--force=false				是否强制终止并删除一个运行中的容器
		-l,--link=false					删除容器的连接，但保留容器
		-v,--volumes=false				删除容器挂载的数据卷

2.9 导出与导入容器

	# docker export -o test_centos.tar 918b5e6f768a
	参数export子参数：
		-o								将输入内容写到文件。
	# docker stop 918b5e6f768a
	# docker rm 918b5e6f768a
	# docker import test_centos.tar  test/centos
	参数import子参数：
		-c 								应用docker 指令创建镜像
		-m 								提交时的说明文字


###3. docker数据管理

3.1 容器创建数据卷

	# docker run -d -p 5000:5000 --name registry --privileged=true -v /opt/registry:/tmp/registry registry
	# docker run -d -P  --name images --privileged=true -v /tmp/registry registry

3.2 数据卷容器
​	
	# docker run -it -v /data --name data1 public/centos
	# docker run -it --volumes-from data1 --name db1 public/centos
	# docker run -it --volumes-from db1 --name db2 public/centos
	# docker rm -v  db2 
	# docker run -it -v /tmp/data:/data --name data2 public/centos
	# docker run -it --volumes-from data2 --name db2 public/centos
	利用数据卷容器迁移数据
	备份
	# docker run --volumes-from data1 -v $(pwd):/backup --name worker ubuntu tar czf /backup/backup.tar /data
	恢复
	# docker run -v /dbdata --name data3 public/centos
	# docker run --volumes-from data3 -v $(pwd):/backup busybox tar xvf /backup/backup.tar

###4. 网络操作

4.1 外部访问

	# docker run -id -P  --name images01 --privileged=true -v /tmp/registry registry /bin/bash
	# docker run -id -p 5000:5000 --name images02 --privileged=true -v /tmp/registry registry /bin/bash
	# docker run -id -p 10.20.0.201:5000:5000  --name images03 --privileged=true -v /tmp/registry registry /bin/bash
	# docker run -id -p 10.20.0.201::5000  --name images04 --privileged=true -v /tmp/registry registry /bin/bash
	# docker run -id -p 10.20.0.201::5000/udp  --name images04 --privileged=true -v /tmp/registry registry /bin/bash

4.2 容器互联
	# docker run -id --name db public/centos /bin/bash
	# docker run -id -P --name web --link db:db public/centos /bin/bash

##进阶操作

###1. 创建镜像
1.1 基于已有容器创建

	# docker commit -m "add a new file blog" -a "blog" 979aa745589e blog:0.1
	参数commit子参数：
		-a,--author=""			 		 作者信息											# docker commit -m "add a new file blog" -a "allposs" 979aa745589e blog:0.1
		-c,--change=[]					 提交的时候执行Dockerfile指令(CMD|
										 ENTRYPOINT|ENV|EXPOSE|LABEL|ONBUILD|
										 USER|VOLUME|WORKDIR)
		-m,--message=""				 	 提交消息											# docker commit -m "add a new file blog" -a "allposs" 979aa745589e blog:0.1
		-p,--pause=true					 提交时暂停容器运行									 

1.2 使用dockerfile创建


1.2.1 Dockerfile的书写规则及指令使用方法

&#160; &#160; &#160; &#160;Dockerfile的指令是忽略大小写的，建议使用大写，使用 # 作为注释，每一行只支持一条指令，每条指令可以携带多个参数。
&#160; &#160; &#160; &#160;Dockerfile的指令根据作用可以分为两种，构建指令和设置指令。构建指令用于构建image，其指定的操作不会在运行image的容器上执行；设置指令用于设置image的属性，其指定的操作将在运行image的容器中执行。

	（1）FROM（指定基础image）
	构建指令，必须指定且需要在Dockerfile其他指令的前面。后续的指令都依赖于该指令指定的image。FROM指令指定的基础image可以是官方远程仓库中的，也可以位于本地仓库。
	格式：
		[plain] view plaincopy 
		FROM <image>  
	
	指定基础image为该image的最后修改的版本。或者：
		[plain] view plaincopy 
		FROM <image>:<tag>  
	指定基础image为该image的一个tag版本。
	
	（2）MAINTAINER（用来指定镜像创建者信息）
	构建指令，用于将image的制作者相关的信息写入到image中。当我们对该image执行docker inspect命令时，输出中有相应的字段记录该信息。
	格式：
		[plain] view plaincopy 
		MAINTAINER <name>  
	
	（3）RUN（安装软件用）
	构建指令，RUN可以运行任何被基础image支持的命令。如基础image选择了ubuntu，那么软件管理部分只能使用ubuntu的命令。
	格式：
		[plain] view plaincopy 
		RUN <command> (the command is run in a shell - `/bin/sh -c`)  
		RUN ["executable", "param1", "param2" ... ]  (exec form)  
	
	（4）CMD（设置container启动时执行的操作）
	设置指令，用于container启动时指定的操作。该操作可以是执行自定义脚本，也可以是执行系统命令。该指令只能在文件中存在一次，如果有多个，则只执行最后一条。
	格式：
		[plain] view plaincopy 
		CMD ["executable","param1","param2"] (like an exec, this is the preferred form)  
		CMD command param1 param2 (as a shell)  
	当Dockerfile指定了ENTRYPOINT，那么使用下面的格式：
		[plain] view plaincopy 
		CMD ["param1","param2"] (as default parameters to ENTRYPOINT)  
	ENTRYPOINT指定的是一个可执行的脚本或者程序的路径，该指定的脚本或者程序将会以param1和param2作为参数执行。所以如果CMD指令使用上面的形式，那么Dockerfile中必须要有配套的ENTRYPOINT。
	
	（5）ENTRYPOINT（设置container启动时执行的操作）
	设置指令，指定容器启动时执行的命令，可以多次设置，但是只有最后一个有效。
	格式:
		[plain] view plaincopy 
		ENTRYPOINT ["executable", "param1", "param2"] (like an exec, the preferred form)  
		ENTRYPOINT command param1 param2 (as a shell)  
	该指令的使用分为两种情况，一种是独自使用，另一种和CMD指令配合使用。
	当独自使用时，如果你还使用了CMD命令且CMD是一个完整的可执行的命令，那么CMD指令和ENTRYPOINT会互相覆盖只有最后一个CMD或者ENTRYPOINT有效。
	[plain] view plaincopy 
	# CMD指令将不会被执行，只有ENTRYPOINT指令被执行  
	CMD echo “Hello, World!”  
	ENTRYPOINT ls -l  
	另一种用法和CMD指令配合使用来指定ENTRYPOINT的默认参数，这时CMD指令不是一个完整的可执行命令，仅仅是参数部分；ENTRYPOINT指令只能使用JSON方式指定执行命令，而不能指定参数。
		[plain] view plaincopy 
		FROM ubuntu  
		CMD ["-l"]  
		ENTRYPOINT ["/usr/bin/ls"]  
	
	（6）USER（设置container容器的用户）
	设置指令，设置启动容器的用户，默认是root用户。
		[plain] view plaincopy 
		# 指定memcached的运行用户  
		ENTRYPOINT ["memcached"]  
		USER daemon  
	或  
		ENTRYPOINT ["memcached", "-u", "daemon"]  
	
	（7）EXPOSE（指定容器需要映射到宿主机器的端口）
	设置指令，该指令会将容器中的端口映射成宿主机器中的某个端口。当你需要访问容器的时候，可以不是用容器的IP地址而是使用宿主机器的IP地址和映射后的端口。要完成整个操作需要两个步骤，首先在Dockerfile使用EXPOSE设置需要映射的容器端口，然后在运行容器的时候指定-p选项加上EXPOSE设置的端口，这样EXPOSE设置的端口号会被随机映射成宿主机器中的一个端口号。也可以指定需要映射到宿主机器的那个端口，这时要确保宿主机器上的端口号没有被使用。EXPOSE指令可以一次设置多个端口号，相应的运行容器的时候，可以配套的多次使用-p选项。
	格式:
		[plain] view plaincopy 
		EXPOSE <port> [<port>...]  
	
		[plain] view plaincopy 
		# 映射一个端口  
		EXPOSE port1  
	
	# 相应的运行容器使用的命令  
	docker run -p port1 image  
	
	# 映射多个端口  
	EXPOSE port1 port2 port3  
	# 相应的运行容器使用的命令  
	docker run -p port1 -p port2 -p port3 image  
	# 还可以指定需要映射到宿主机器上的某个端口号  
	docker run -p host_port1:port1 -p host_port2:port2 -p host_port3:port3 image  
	端口映射是docker比较重要的一个功能，原因在于我们每次运行容器的时候容器的IP地址不能指定而是在桥接网卡的地址范围内随机生成的。宿主机器的IP地址是固定的，我们可以将容器的端口的映射到宿主机器上的一个端口，免去每次访问容器中的某个服务时都要查看容器的IP的地址。对于一个运行的容器，可以使用docker port加上容器中需要映射的端口和容器的ID来查看该端口号在宿主机器上的映射端口。
	
	（8）ENV（用于设置环境变量）
	构建指令，在image中设置一个环境变量。
	格式:
		[plain] view plaincopy 
		ENV <key> <value>  
	
	设置了后，后续的RUN命令都可以使用，container启动后，可以通过docker inspect查看这个环境变量，也可以通过在docker run --env key=value时设置或修改环境变量。
	假如你安装了JAVA程序，需要设置JAVA_HOME，那么可以在Dockerfile中这样写：
		ENV JAVA_HOME /path/to/java/dirent
	
	（9）ADD（从src复制文件到container的dest路径）
	构建指令，所有拷贝到container中的文件和文件夹权限为0755，uid和gid为0；如果是一个目录，那么会将该目录下的所有文件添加到container中，不包括目录；如果文件是可识别的压缩格式，则docker会帮忙解压缩（注意压缩格式）；如果<src>是文件且<dest>中不使用斜杠结束，则会将<dest>视为文件，<src>的内容会写入<dest>；如果<src>是文件且<dest>中使用斜杠结束，则会<src>文件拷贝到<dest>目录下。
	格式:
		[plain] view plaincopy 
		ADD <src> <dest>  
	
	<src> 是相对被构建的源目录的相对路径，可以是文件或目录的路径，也可以是一个远程的文件url;
	<dest> 是container中的绝对路径
	
	（10）VOLUME（指定挂载点)）
	设置指令，使容器中的一个目录具有持久化存储数据的功能，该目录可以被容器本身使用，也可以共享给其他容器使用。我们知道容器使用的是AUFS，这种文件系统不能持久化数据，当容器关闭后，所有的更改都会丢失。当容器中的应用有持久化数据的需求时可以在Dockerfile中使用该指令。
	格式:
		[plain] view plaincopy 
		VOLUME ["<mountpoint>"]  
	
		[plain] view plaincopy 
		FROM base  
		VOLUME ["/tmp/data"]  
	运行通过该Dockerfile生成image的容器，/tmp/data目录中的数据在容器关闭后，里面的数据还存在。例如另一个容器也有持久化数据的需求，且想使用上面容器共享的/tmp/data目录，那么可以运行下面的命令启动一个容器：
		[plain] view plaincopy 
		docker run -t -i -rm -volumes-from container1 image2 bash  
	container1为第一个容器的ID，image2为第二个容器运行image的名字。
	
	（11）WORKDIR（切换目录）
	设置指令，可以多次切换(相当于cd命令)，对RUN,CMD,ENTRYPOINT生效。
	格式:
		[plain] view plaincopy 
		WORKDIR /path/to/workdir  
	
		[plain] view plaincopy 
		# 在 /p1/p2 下执行 vim a.txt  
		WORKDIR /p1 WORKDIR p2 RUN vim a.txt  
	
	（12）ONBUILD（在子镜像中执行）
		[plain] view plaincopy 
		ONBUILD <Dockerfile关键字>  
	ONBUILD 指定的命令在构建镜像时并不执行，而是在它的子镜像中执行。

1.2.2 Nginx软件下载服务器


	# vim Dockerfile
		# Pull base image  
		FROM nginx
		
		MAINTAINER allposs "tanzj520@gmail.com"
		
		#volume soft
		VOLUME ["/www/soft"]
		# update source  
		RUN rm -rf /etc/nginx/conf.d/default.conf
		
		#config nginx default
		ADD default.conf /etc/nginx/conf.d/


		# Expose ports.  
		EXPOSE 80
		
		# Define default command.  
		ENTRYPOINT nginx -g "daemon off;"
		
	# vim default.conf	
		
		server {
			listen       80;
			server_name  localhost;
			access_log  /var/log/nginx/log/host.access.log  main;
			location / {
				root   /www/soft/;
				autoindex on;
				autoindex_exact_size off;
				autoindex_localtime on;
			}
			error_page   500 502 503 504  /50x.html;
			location = /50x.html {
				root   /usr/share/nginx/html;
			}
		}
	# docker build /root/soft/ -t nginxsoft:1.1


​	


####2. 创建本地私有仓库

2.1 使用容器创建本地私有仓库

	# docker run -d -p 5000:5000 --privileged=true -v /opt/registry:/tmp/registry registry
	参数说明： 
		-v /opt/registry:/tmp/registry 	默认情况下，会将仓库存放于容器内的/tmp/registry目录下，指定本地目录挂载到容器 
		–privileged=true 				CentOS7中的安全模块selinux把权限禁掉了，参数给容器加特权，不加上传镜像会报权限错误(OSError: [Errno 13] Permission denied: ‘/tmp/registry/repositories/liibrary’)或者（Received unexpected HTTP status: 500 Internal Server Error）错误

2.2 客户端上传镜像

	# vim /etc/sysconfig/docker
		增加：
		OPTIONS='--insecure-registry 10.20.0.201:5000'    #CentOS 7系统
		other_args='--insecure-registry 10.20.0.201:5000' #CentOS 6系统
	
	# vim /etc/sysconfig/docker
		增加：ADD_REGISTRY='--add-registry 10.20.0.201:5000'
		
	# docker pull public/centos
	# docker tag hub.c.163.com/public/centos 10.20.0.201:5000/centos
	# docker push 10.20.0.201:5000/centos:latest
	# curl -u myuser https://10.20.0.201:5000/v1/search
	# curl 10.20.0.201:5000/v1/search
	# docker search local/

##优化

###1. 资源控制
1.1 内存
	内存限制，只会限制容器内使用的内存大小，超出限制后，应用假死，容器存活
	# docker run -id -m 1024m --name db public/centos /bin/bash		
	内存锁死,只要程序超过限制大小就被kill掉
	# docker run -id -m 1024m -memory-swap=1024m --name db public/centos /bin/bash

1.2 CPU
	# run -it --rm --cpuset=0-4 agileek/cpuset-test /cpus 5
	# docker run --name cpuuse -d --cpuset-cpus=0,3   public/centos /bin/bash
	
	–cpu-period、–cpu-quota两个参数控制容器可以分配到的CPU时钟周期。–cpu-period是用来指定容器对CPU的使用要在多长时间内做一次重新分配，而–cpu-quota是用来指定在这个周期内，最多可以有多少时间用来跑这个容器。跟–cpu-shares不同的是这种配置是指定一个绝对值，而且没有弹性在里面，容器对CPU资源的使用绝对不会超过配置的值。
	
	cpu-period和cpu-quota的单位为微秒（μs）。cpu-period的最小值为1000微秒，最大值为1秒（10^6 μs），默认值为0.1秒（100000 μs）。cpu-quota的值默认为-1，表示不做控制。
	
	举个例子，如果容器进程需要每1秒使用单个CPU的0.2秒时间，可以将cpu-period设置为1000000（即1秒），cpu-quota设置为200000（0.2秒）。当然，在多核情况下，如果允许容器进程需要完全占用两个CPU，则可以将cpu-period设置为100000（即0.1秒），cpu-quota设置为200000（0.2秒）。
	
	参数
		-m, --memory=""					内存限制（格式：<number> [<unit>]）。数字是一个正整数。单位可以是b，k，m或g之一。最低为4M。
		--memory-swap=""				总内存限制（内存+交换格式：<number> [<unit>]）。数字是一个正整数。单位可以是b，k，m或g之一。
		--memory-reservation=""			内存软限制（格式：<number> [<unit>]）。数字是一个正整数。单位可以是b，k，m或g之一。
		--kernel-memory=""				内核内存限制（格式：<number> [<unit>]）。数字是一个正整数。单位可以是b，k，m或g之一。最低为4M。
		-c, --cpu-shares=0				CPU份额（相对权重）
		--cpus=0.000					CPU数量数字是小数。 0.000表示无限制。
		--cpu-period=0					限制CPU CFS（完全公平的调度程序）周期
		--cpuset-cpus=""				允许使用的CPU（0-3,0），绑定 CPU
		--cpuset-mems=""				允许执行的存储器节点（MEM）（0-3,0.1）。仅在NUMA系统上有效。
		--cpu-quota=0					限制CPU CFS（完全公平的调度程序）配额
		--cpu-rt-period=0				限制CPU实时时间。微秒。需要设置父cgroup并且不能高于父。还要检查rtprio ulimits。
		--cpu-rt-runtime=0				限制CPU实时运行时间。微秒。需要设置父cgroup并且不能高于父。还要检查rtprio ulimits。
		--blkio-weight=0				块IO权重比（相对权重）接受10到1000之间的权重比值。
		--blkio-weight-device=""		针对特定设备的权重比（相对设备权重比，格式：DEVICE_NAME：WEIGHT）
		--device-read-bps=""			从设备限制读取速率（格式：<device-path>：<number> [<unit>]）。数字是一个正整数。单位可以是kb，mb或gb之一。
		--device-write-bps=""			限制对设备的写入速率（格式：<device-path>：<number> [<unit>]）。数字是一个正整数。单位可以是kb，mb或gb之一。
		--device-read-iops=""			从设备限制读取速率（每秒IO）（格式：<device-path>：<number>）。数字是一个正整数。
		--device-write-iops=""			限制写入速率（IO每秒）到设备（格式：<device-path>：<number>）。数字是一个正整数。
		--oom-kill-disable=false		是否禁用OOM杀手的容器？
		--oom-score-adj=0				调整容器的OOM偏好（-1000到1000）
		--memory-swappiness=""			调整容器的内存变化行为。接受0到100之间的整数。
		--shm-size=""					/ dev / shm的大小。格式为<number> <unit>。数字必须大于0.单位是可选的，可以是b（字节），k（千字节），m（兆字节）或g（千兆字节））。如果省略单位，系统将使用字节。如果您完全省略了大小，系统将使用64m。

##结束

详细资料可参考https://docs.docker.com/engine/
参考书籍《Docker入门与实践》

