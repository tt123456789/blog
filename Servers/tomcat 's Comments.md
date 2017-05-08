Title: tomcat 详解
Date: 2016/5/9 11:00:28 
Modified: 2016/5/9 11:00:32 
Category: Servers
Tags: tomcat
Slug: 
Author: allposs


##简介##


##版本##

V1.0

##正文##

###Server.xml

Tomcat服务器是由一系列可配置的组件构成的，其中核心组件是Catalina Servlet，它是最顶层组件。

Tomcat的各组件是在server.xml（CATALINA_HOME\conf\server.xml）配置的

![](http://image.allposs.cn/20160507001.png)



###1 Server

&#160; &#160; &#160; &#160;Server即Catalina servlet组件，它是server.xml的最外层元素

常用属性：

	1. Address—Tomcat监听shutdown命令的地址，默认为：localhost 
	2. className—指定实现org.apache.catalina.Server接口的类，默认值为：org.apache.catalina.core.StandardServer
	3. port-Tomcat监听shutdown命令的端口。设置为-1，则禁止通过端口关闭Tomcat，同时shutdown.bat也不能使用
	4. shutdown－通过指定的地址（Address）、端口（port）关闭Tomcat所需的字符串。修改shutdown的值，对shutdown.bat无影响，通过端口关闭Tomcat，可以用telnet进行测试
	telnet localhost 8005
	然后输入SHUTDOWN即可关闭Tomcat
	
####1.1 Listener

&#160; &#160; &#160; &#160;Listener即监听器，负责监听特定的事件，当特定事件触发时，Listener会捕捉到该事件，并做出相应处理。Listener通常用在Tomcat的启动和关闭过程。Listener可嵌在Server、Engine、Host、Context内

常用属性：

	className－指定实现org.apache.catalina.LifecycleListener接口的类

####1.2 GlobalNamingResources

	GlobalNamingResources用于配置JNDI

####1.3 Service

&#160; &#160; &#160; &#160;Service包装Executor、Connector、Engine，以组成一个完整的服务

常用属性：

	1. className 	指定实现org.apache.catalina. Service接口的类，默认值为org.apache.catalina.core.StandardService
	2. name 	Service的名字
	3. Server可以包含多个Service组件

#####1.3.1 Executor

&#160; &#160; &#160; &#160;Executor即Service提供的线程池，供Service内各组件使用

常用属性：

	1. className 	指定实现org.apache.catalina. Executor接口的类，默认值为org.apache.catalina.core. StandardThreadExecutor
	2. name 	线程池的名字
	3. daemon 	是否为守护线程，默认值为true
	4. maxIdleTime 	总线程数高于minSpareThreads时，空闲线程的存活时间（单位：ms），默认值为60000，即1min
	5. maxQueueSize 	任务队列上限，默认值为Integer.MAX_VALUE(（2147483647），超过此值，将拒绝
	6. maxThreads 	线程池内线程数上限，默认值为200
	7. minSpareThreads 	线程池内线程数下限，默认值为25
	8. namePrefix 	线程名字的前缀。线程名字通常为namePrefix+ threadNumber
	9. prestartminSpareThreads 	是否在Executor启动时，就生成minSpareThreads个线程。默认为false
	10. threadPriority 	Executor内线程的优先级，默认值为5（Thread.NORM_PRIORITY）
	11. threadRenewalDelay 	重建线程的时间间隔。重建线程池内的线程时，为了避免线程同时重建，每隔threadRenewalDelay（单位：ms）重建一个线程。默认值为1000，设置为负则不重建

#####1.3.2 Connector

&#160; &#160; &#160; &#160;Connector是Tomcat接收请求的入口，每个Connector有自己专属的监听端口

&#160; &#160; &#160; &#160;Connector有两种：HTTP Connector和AJP Connector

常用属性：

	1. port 	Connector接收请求的端口
	2. protocol 	Connector使用的协议（HTTP/1.1或AJP/1.3）
	3. connectionTimeout 	每个请求的最长连接时间（单位：ms）
	4. redirectPort 	处理http请求时，收到一个SSL传输请求，该SSL传输请求将转移到此端口处理
	5. executor 	指定线程池,如果没设置executor，可在Connector标签内设置maxThreads（默认200）、minSpareThreads（默认10）
	6. acceptCount 	Connector请求队列的上限。当该Connector的请求队列超过acceptCount时，将拒绝接收请求,当同时连接的人数达到maxThreads时，还可以接收排队的连接数量，超过这个连接的则直接返回拒绝连接。（指定当任何能够使用的处理请求的线程数都 被使用时，能够放到处理队列中的请求数，超过这个数的请求将不予处理。默认值10。 ）
	7. maxThreads 	tomcat起动的最大线程数，即同时处理的任务个数,maxThreads="150"表示最多同时处理150个连接,Tomcat使用线程来处理接收的每个请求这个值表示Tomcat可创建的最大的线程数，默认值为200 
	8. minSpareThreads 	Tomcat初始化时创建的线程数,表示即使没有人使用也开这么多空线程等待 。默认值4。 
	9. maxSpareThreads 	maxSpareThreads="75"   表示如果最多可以空75个线程，例如某时刻有80人访问，之后没有人访问了，则tomcat不会保留80个空线程，而是关闭5个空的。  （一旦创建的线程超过这个值，Tomcat就会关闭不再需要的socket线程。默认值50。
	10. enableLookups 	是否反查域名，默认值为true。为了提高处理能力，应设置为false 
	11. maxKeepAliveRequests 	保持请求数量，默认值100。
	12. bufferSize 	输入流缓冲大小，默认值2048 bytes
	13. compression 	压缩传输，取值on/off/force，默认值off。
	14. keepAliveTimeout	长连接最大保持时间（毫秒）。
	15. maxKeepAliveRequests	最大长连接个数（1表示禁用，-1表示不限制个数，默认100个。一般设置在100~200之间）
	16. maxHttpHeaderSize	http请求头信息的最大程度，超过此长度的部分不予处理。一般8K。
	17. URIEncoding	指定Tomcat容器的URL编码格式。
	18. disableUploadTimeout	上传时是否使用超时机制
	19. enableLookups	是否反查域名，取值为：true或false。为了提高处理能力，应设置为false
	20. bufferSize defines the size (in bytes) of the buffer to be provided for input streams created by this connector. By default, buffers of 2048 bytes are provided.
	21. minProcessors	最小空闲连接线程数，用于提高系统处理性能，默认值为10。（用于Tomcat4中）
	22 .maxProcessors	最大连接线程数，即：并发处理的最大请求数，默认值为75。（用于Tomcat4中）
	注：
	Tomcat4：
	
		可以通过修改minProcessors和maxProcessors的值来控制线程数。
	
	Tomcat5+：
		maxThreads
		 Tomcat使用线程来处理接收的每个请求。这个值表示Tomcat可创建的最大的线程数。
		acceptCount 
		 指定当所有可以使用的处理请求的线程数都被使用时，可以放到处理队列中的请求数，超过这个数的请求将不予处理。
		connnectionTimeout 
		 网络连接超时，单位：毫秒。设置为0表示永不超时，这样设置有隐患的。通常可设置为30000毫秒。
		minSpareThreads 
		 Tomcat初始化时创建的线程数。
		maxSpareThreads 
		 一旦创建的线程超过这个值，Tomcat就会关闭不再需要的socket线程。 
	
#####1.3.3 Engine

&#160; &#160; &#160; &#160;Engine负责处理Service内的所有请求。它接收来自Connector的请求，并决定传给哪个Host来处理，Host处理完请求后，将结果返回给Engine，Engine再将结果返回给Connector

常用属性：

	1. name－Engine的名字
	2. defaultHost 		指定默认Host。Engine接收来自Connector的请求，然后将请求传递给defaultHost，defaultHost 负责处理请求
	3. className 	指定实现org.apache.catalina. Engine接口的类，默认值为org.apache.catalina.core. StandardEngine
	4. backgroundProcessorDelay 	Engine及其部分子组件（Host、Context）调用backgroundProcessor方法的时间间隔。backgroundProcessorDelay为负，将不调用backgroundProcessor。backgroundProcessorDelay的默认值为10

	注：
	Tomcat启动后，Engine、Host、Context会启动一个后台线程，定期调用backgroundProcessor方法。backgroundProcessor方法主要用于重新加载Web应用程序的类文件和资源、扫描Session过期

	5. jvmRoute－Tomcat集群节点的id。部署Tomcat集群时会用到该属性，Service内必须包含一个Engine组件
,Service包含一个或多个Connector组件，Service内的Connector共享一个Engine

######1.3.3.1 Host

&#160; &#160; &#160; &#160;Host负责管理一个或多个Web项目

常用属性：

	1. name－Host的名字
	2. appBase－存放Web项目的目录（绝对路径、相对路径均可）
	3. unpackWARs－当appBase下有WAR格式的项目时，是否将其解压（解成目录结构的Web项目）。设成false，则直接从WAR文件运行Web项目
	4. autoDeploy－是否开启自动部署。设为true，Tomcat检测到appBase有新添加的Web项目时，会自动将其部署
	5. startStopThreads－线程池内的线程数量。Tomcat启动时，Host提供一个线程池，用于部署Web项目
		startStopThreads为0，并行线程数=系统CPU核数
		startStopThreads为负数，并行线程数=系统CPU核数+startStopThreads，如果（系统CPU核数+startStopThreads）小于1，并行线程数设为1
		startStopThreads为正数，并行线程数= startStopThreads
		startStopThreads默认值为1
		startStopThreads为默认值时，Host只提供一个线程，用于部署Host下的所有Web项目。如果Host下的Web项目较多，由于只有一个线程负责部署这些项目，因此这些项目将依次部署，最终导致Tomcat的启动时间较长。此时，修改startStopThreads值，增加Host部署Web项目的并行线程数，可降低Tomcat的启动时间




#######1.3.3.1.1 Context

&#160; &#160; &#160; &#160;Context代表一个运行在Host上的Web项目。一个Host上可以有多个Context,将一个Web项目（/kaka）添加到Tomcat，在Host标签内，添加Context标签

	<Context path="/kaka" docBase="/kaka"  debug="0" reloadable="true" crossContext="true">

	</Context>

常用属性：

	1. path－该Web项目的URL入口。path设置为””，输入http://localhost:8080即可访问MyApp；path设置为”/test/MyApp”，输入http://localhost:8080/test/MyApp才能访问MyApp
	2. docBase－Web项目的路径，绝对路径、相对路径均可（相对路径是相对于CATALINA_HOME\webapps）
	3. reloadable－设置为true，Tomcat会自动监控Web项目的/WEB-INF/classes/和/WEB-INF/lib变化，当检测到变化时，会重新部署Web项目。reloadable默认值为false。通常项目开发过程中设为true，项目发布的则设为false
	4. crossContext－设置为true，该Web项目的Session信息可以共享给同一host下的其他Web项目。默认为false
		


注：
			
&#160; &#160; &#160; &#160;一个Host元素中嵌套任意多的Context元素。每个Context的路径必须是惟一的，由path属性定义。另外，你必须定义一个path=“”的context，这个Context称为该虚拟主机的缺省web应用，用来处理那些不能匹配任何Context的Context路径的请求。
&#160; &#160; &#160; &#160;在tomcat 5.5之后不推荐在server.xml中进行配置，而是在/conf/context.xml中进行独立的配置。因为 server.xml 是不可动态重加载的资源，服务器一旦启动了以后，要修改这个文件，就得重启服务器才能重新加载。而 context.xml 文件则不然， tomcat 服务器会定时去扫描这个文件。一旦发现文件被修改（时间戳改变了），就会自动重新加载这个文件，而不需要重启服务器 。
	Xml代码 
		<Context path="/kaka" docBase="kaka" debug="0" reloadbale="true" privileged="true">  
		<WatchedResource>WEB-INF/web.xml</WatchedResource>  
		<WatchedResource>WEB-INF/kaka.xml</WatchedResource> 监控资源文件，如果web.xml || kaka.xml改变了，则自动重新加载改应用。  
		<Resource name="jdbc/testSiteds" 表示指定的jndi名称  
		auth="Container" 表示认证方式，一般为Container  
		type="javax.sql.DataSource"  
		maxActive="100" 连接池支持的最大连接数  
		maxIdle="30" 连接池中最多可空闲maxIdle个连接  
		maxWait="10000" 连接池中连接用完时,新的请求等待时间,毫秒  
		username="root" 表示数据库用户名  
		password="root" 表示数据库用户的密码  
		driverClassName="com.mysql.jdbc.Driver" 表示JDBC DRIVER  
		url="jdbc:mysql://localhost:3306/testSite" /> 表示数据库URL地址  
		</Context>  
		
######1.3.3.2 Cluster

Tomcat集群配置。


######1.3.3.3 Realm

&#160; &#160; &#160; &#160;Realm可以理解为包含用户、密码、角色的”数据库”。Tomcat定义了多种Realm实现：JDBC Database Realm、DataSource Database Realm、JNDI Directory Realm、UserDatabase Realm等

######1.3.3.4 Valve

&#160; &#160; &#160; &#160;Valve可以理解为Tomcat的拦截器，而我们常用filter为项目内的拦截器。Valve可以用于Tomcat的日志、权限等。Valve可嵌在Engine、Host、Context内


###2. 目录结构



	|-- bin
	|   |-- bootstrap.jar		tomcat启动时所依赖的一个类，在启动tomcat时会发现Using CLASSPATH: 是加载的这个类
	|   |-- catalina-tasks.xml		定义tomcat载入的库文件，类文件
	|   |-- catalina.bat
	|   |-- catalina.sh		                 tomcat单个实例在Linux平台上的启动脚本
	|   |-- commons-daemon-native.tar.gz	           jsvc工具，可以使tomcat已守护进程方式运行，需单独编译安装
	|   |-- commons-daemon.jar		           jsvc工具所依赖的java类
	|   |-- configtest.bat
	|   |-- configtest.sh	        tomcat检查配置文件语法是否正确的Linux平台脚本
	|   |-- cpappend.bat
	|   |-- daemon.sh		tomcat已守护进程方式运行时的，启动，停止脚本
	|   |-- digest.bat
	|   |-- digest.sh
	|   |-- setclasspath.bat
	|   |-- setclasspath.sh
	|   |-- shutdown.bat
	|   |-- shutdown.sh		tomcat服务在Linux平台下关闭脚本
	|   |-- startup.bat
	|   |-- startup.sh		         tomcat服务在Linux平台下启动脚本
	|   |-- tomcat-juli.jar
	|   |-- tomcat-native.tar.gz	 使tomcat可以使用apache的apr运行库，以增强tomcat的性能需单独编译安装
	|   |-- tool-wrapper.bat
	|   |-- tool-wrapper.sh
	|   |-- version.bat
	|   |-- version.sh		查看tomcat以及JVM的版本信息
	|-- conf			顾名思义，配置文件目录
	|   |-- catalina.policy		配置tomcat对文件系统中目录或文件的读、写执行等权限，及对一些内存，session等的管理权限
	|   |-- catalina.properties		配置tomcat的classpath等
	|   |-- context.xml			tomcat的默认context容器
	|   |-- logging.properties		配置tomcat的日志输出方式
	|   |-- server.xml			       tomcat的主配置文件
	|   |-- tomcat-users.xml	       tomcat的角色(授权用户)配置文件
	|   |-- web.xml				tomcat的应用程序的部署描述符文件
	|-- lib
	|-- logs		日志文件默认存放目录
	|-- temp
	|   |-- safeToDelete.tmp
	|-- webapps		          tomcat默认存放应用程序的目录，好比apache的默认网页存放路径是/var/www/html一样
	|   |-- docs	tomcat文档
	|   |-- examples                     tomcat自带的一个独立的web应用程序例子
	|   |-- host-manager              tomcat的主机管理应用程序
	|	|   |-- META-INF	          整个应用程序的入口，用来描述jar文件的信息
	|	|   |   `-- context.xml     当前应用程序的context容器配置，它会覆盖tomcat/conf/context.xml中的配置
	|	|   |-- WEB-INF		 用于存放当前应用程序的私有资源
	|	|   |   |-- classes		 用于存放当前应用程序所需要的class文件
	|       |   |	|-- lib		         用于存放当前应用程序锁需要的jar文件
	|	|   |   `-- web.xml		当前应用程序的部署描述符文件，定义应用程序所要加载的serverlet类，以及该程序是如何部署的
	|   |-- manager                  tomcat的管理应用程序
	|   |-- ROOT	             指tomcat的应用程序的根，如果应用程序部署在ROOT中，则可直接通过http://ip:port 访问到
	|-- work	用于存放JSP应用程序在部署时编译后产生的class文件


###3. 术语

####线程
	
&#160; &#160; &#160; &#160;其实线程池的原理很简单，类似于操作系统中的缓冲区的概念，它的流程如下：先启动若干数量的线程，并让这些线程都处于睡眠 状态，当客户端有一个新请求时，就会唤醒线程池中的某一个睡眠线程，让它来处理客户端的这个请求，当处理完这个请求后，线程又处于睡眠状态。可能你也许会 问：为什么要搞得这么麻烦，如果每当客户端有新的请求时，我就创建一个新的线程不就完了？这也许是个不错的方法，因为它能使得你编写代码相对容易一些，但 你却忽略了一个重要的问题??性能！就拿我所在的单位来说，我的单位是一个省级数据大集中的银行网络中心，高峰期每秒的客户端请求并发数超过100，如果 为每个客户端请求创建一个新线程的话，那耗费的CPU时间和内存将是惊人的，如果采用一个拥有200个线程的线程池，那将会节约大量的的系统资源，使得更 多的CPU时间和内存用来处理实际的商业应用，而不是频繁的线程创建与销毁。

线程池一般有三个重要参数：
 
	1. 最大线程数。在程序运行的任何时候，线程数总数都不会超过这个数。如果请求数量超过最大数时，则会等待其他线程结束后再处理。 
	2. 最大共享线程数，即最大空闲线程数。如果当前的空闲线程数超过该值，则多余的线程会被杀掉。 
	3. 最小共享线程数，即最小空闲线程数。如果当前的空闲数小于该值，则一次性创建这个数量的空闲线程，所以它本身也是一个创建线程的步长。

线程池有两个概念： 

	1. Worker线程。工作线程主要是运行执行代码，有两种状态：空闲状态和运行状态。在空闲状态时，类似“休眠”，等待任务；处理运行状态时，表示正在运行任务（Runnable）。 
	2. 辅助线程。主要负责监控线程池的状态：空闲线程是否超过最大空闲线程数或者小于最小空闲线程数等。如果不满足要求，就调整之。

####栈

&#160; &#160; &#160; &#160;每当启用一个线程时，JVM就为他分配一个Java栈，栈是以帧为单位保存当前线程的运行状态。某个线程正在执行的方法称为当前方法，当前方法使用的栈帧称为当前帧，当前方法所属的类称为当前类，当前类的常量池称为当前常量池。当线程执行一个方法时，它会跟踪当前常量池。
&#160; &#160; &#160; &#160;每当线程调用一个Java方法时，JVM就会在该线程对应的栈中压入一个帧，这个帧自然就成了当前帧。当执行这个方法时，它使用这个帧来存储参数、局部变量、中间运算结果等等。
&#160; &#160; &#160; &#160;Java栈上的所有数据都是私有的。任何线程都不能访问另一个线程的栈数据。所以我们不用考虑多线程情况下栈数据访问同步的情况。
&#160; &#160; &#160; &#160;像方法区和堆一样，Java栈和帧在内存中也不必是连续的,帧可以分布在连续的栈里，也可以分布在堆里	
