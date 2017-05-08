Title: java数据库连接池(一)
Date: 9:30 2016/11/24
Modified: 9:30 2016/11/24 
Category: Servers
Tags: 开源数据库连接池
Slug: 
Author: allposs

##简介##
&#160; &#160; &#160; &#160;Apache是世界使用排名第一的Web服务器软件。它可以运行在几乎所有广泛使用的计算机平台上，由于其跨平台和安全性被广泛使用，是最流行的Web服务器端软件之一。它快速、可靠并且可通过简单的API扩充，将Perl/Python等解释器编译到服务器中。

##版本##

apache 2.2.31

##正文##
&#160; &#160; &#160; &#160;池(Pool)技术在一定程度上可以明显优化服务器应用程序的性能，提高程序执行效率和降低系统资源开销。这里所说的池是一种广义上的池，比如数据库连接池、线程池、内存池、对象池等。其中，对象池可以看成保存对象的容器，在进程初始化时创建一定数量的对象。需要时直接从池中取出一个空闲对象，用完后并不直接释放掉对象，而是再放到对象池中以方便下一次对象请求可以直接复用。其他几种池的设计思想也是如此，池技术的优势是，可以消除对象创建所带来的延迟，从而提高系统的性能。

&#160; &#160; &#160; &#160;池要了解Java连接池我们先要了解数据库连接池（connection pool）的原理，Java连接池正是数据库连接池在Java上的应用。——我们知道，对于共享资源，有一个很著名的设计模式：资源池（Resource Pool）。

&#160; &#160; &#160; &#160;池该模式正是为了解决资源的频繁分配﹑释放所造成的问题。为解决上述问题，可以采用数据库连接池技术。数据库连接池的基本思想就是为数据库连接建立一个“缓冲池”。预先在缓冲池中放入一定数量的连接，当需要建立数据库连接时，只需从“缓冲池”中取出一个，使用完毕之后再放回去。我们可以通过设定连接池最大连接数来防止系统无尽的与数据库连接。更为重要的是我们可以通过连接池的管理机制监视数据库的连接的数量﹑使用情况，为系统开发﹑测试及性能调整提供依据。

&#160; &#160; &#160; &#160;最原始的数据库使用就是打开一个连接并进行使用，使用过后一定要关闭连接释放资源。由于频繁的打开和关闭连接对jvm包括数据库
&#160; &#160; &#160; &#160;都有一定的资源负荷，尤其应用压力较大时资源占用比较多容易产生性能问题。由此使用连接池的作用就显现出来，他的原理其实不复杂：
&#160; &#160; &#160; &#160;先打开一定数量的数据库连接，当使用的时候分配给调用者，调用完毕后返回给连接池，注意返回给连接池后这些连接并不会关闭，而是
&#160; &#160; &#160; &#160;准备给下一个调用者进行分配。由此可以看出连接池节省了大量的数据库连接打开和关闭的动作，对系统性能提升的益处不言而喻。
&#160; &#160; &#160; &#160;几个概念：

	1.最小连接--应用启动后随即打开的连接数以及后续最小维持的连接数。
	2.最大连接数--应用能够使用的最多的连接数
	3.连接增长数--应用每次新打开的连接个数

&#160; &#160; &#160; &#160;举个例子说明连接池的运作：

&#160; &#160; &#160; &#160;假设设置了最小和最大的连接为10，20，那么应用一旦启动则首先打开10个数据库连接，但注意此时数据库连接池的正在使用数字为0--因为你并没有使用这些连接，而空闲的数量则是10。然后你开始登录，假设登录代码使用了一个连接进行查询，那么此时数据库连接池的正在使用数字为1、空闲数为9，这并不需要从数据库打开连接--因为连接池已经准备好了10个给你留着呢。登录结束了，当前连接池的连接数量是多少？当然是0，因为那个连接随着事务的结束已经返还给连接池了。然后同时有11个人在同一秒进行登录，会发生什么：连接池从数据库新申请（打开）了一个连接，连同另外的10个一并送出，这个瞬间连接池的使用数是11个，不过没关系正常情况下过一会儿又会变成0。如果同时有21个人登录呢？那第21个人就只能等前面的某个人登录完毕后释放连接给他。这时连接池开启了20个数据库连接--虽然很可能正在使用数量的已经降为0，那么20个连接会一直保持吗？当然不，连接池会在一定时间内关闭一定量的连接还给数据库，在这个例子里数字是20-10=10，因为只需要保持最小连接数就好了，而这个时间周期也是连接池里配置的。

&#160; &#160; &#160; &#160;Java中常用的数据库连接池有：DBCP 、C3P0、BoneCP、Proxool、DDConnectionBroker、DBPool、XAPool、Primrose、SmartPool、MiniConnectionPoolManager及Druid等。

&#160; &#160; &#160; &#160;C3P0是一个开放源代码的JDBC连接池，它在lib目录中与Hibernate一起发布,包括了实现jdbc3和jdbc2扩展规范说明的Connection 和Statement 池的DataSources 对象。（主页：http://sourceforge.net/projects/c3p0/）
 
&#160; &#160; &#160; &#160;BoneCP 是一个开源的快速的 JDBC 连接池。BoneCP很小，只有四十几K（运行时需要log4j和Google Collections的支持，这二者加起来就不小了），而相比之下 C3P0 要六百多K。另外个人觉得 BoneCP 有个缺点是，JDBC驱动的加载是在连接池之外的，这样在一些应用服务器的配置上就不够灵活。当然，体积小并不是 BoneCP 优秀的原因，BoneCP 到底有什么突出的地方呢，请看看性能测试报告。（主页：http://jolbox.com/）
 
&#160; &#160; &#160; &#160;DBCP （Database Connection Pool）是一个依赖Jakarta commons-pool对象池机制的数据库连接池，Tomcat的数据源使用的就是DBCP。目前 DBCP 有两个版本分别是 1.3 和 1.4。1.3 版本对应的是 JDK 1.4-1.5 和 JDBC 3，而1.4 版本对应 JDK 1.6 和 JDBC 4。因此在选择版本的时候要看看你用的是什么 JDK 版本了，功能上倒是没有什么区别。（主页：http://commons.apache.org/dbcp/）
 
&#160; &#160; &#160; &#160;Proxool是一个Java SQL Driver驱动程序，提供了对你选择的其它类型的驱动程序的连接池封装。可以非常简单的移植到现存的代码中。完全可配置。快速，成熟，健壮。可以透明地为你现存的JDBC驱动程序增加连接池功能。

&#160; &#160; &#160; &#160;数据库连接是一种关键的有限的昂贵的资源，这一点在多用户的网页应用程序中体现得尤为突出。对数据库连接的管理能显著影响到整个应用程序的伸缩性和健壮性，影响到程序的性能指标。数据库连接池正是针对这个问题提出来的。数据库连接池负责分配、管理和释放数据库连接，它允许应用程序重复使用一个现有的数据库连接，而再不是重新建立一个；释放空闲时间超过最大空闲时间的数据库连接来避免因为没有释放数据库连接而引起的数据库连接遗漏。这项技术能明显提高对数据库操作的性能。


###1 dbcp
&#160; &#160; &#160; &#160;dbcp可能是使用最多的开源连接池，原因大概是因为配置方便，而且很多开源和tomcat应用例子都是使用的这个连接池吧。DBCP(DataBase connection pool),数据库连接池。是 apache 上的一个 java 连接池项目，也是 tomcat 使用的连接池组件。单独使用dbcp需要2个包：commons-dbcp.jar,commons-pool.jar由于建立数据库连接是一个非常耗时耗资源的行为，所以通过连接池预先同数据库建立一些连接，放在内存中，应用程序需要建立数据库连接时直接到连接池中申请一个就行，用完后再放回去。Hibernate官方宣布由于Bug太多不再支持DBCP

&#160; &#160; &#160; &#160;使用评价：在具体项目应用中，发现此连接池的持续运行的稳定性还是可以，不过速度稍慢，在大并发量的压力下稳定性有所下降，此外不提供连接池监控

####配置详解：
		配置示例：
		dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">   
		"driverClassName" value="com.mysql.jdbc.Driver" />  
		"url" value="xxxx" />  
		"username">xxxx  
		"password">xxxxx  
		"maxActive">20  
		"initialSize">1  
		"maxWait">60000  
		"maxIdle">20  
		"minIdle">3  
		"removeAbandoned">true  
		"removeAbandonedTimeout">180  
		"connectionProperties">clientEncoding=GBK  
		
		官方文档：http://commons.apache.org/proper/commons-dbcp/configuration.html
		1.initialSize ：连接池启动时创建的初始化连接数量（默认值为0）
		2.maxActive ：连接池中可同时连接的最大的连接数（默认值为8，调整为20，高峰单机器在20并发左右，自己根据应用场景定）
		3.maxIdle：连接池中最大的空闲的连接数，超过的空闲连接将被释放，如果设置为负数表示不限制（默认为8个，maxIdle不能设置太小，因为假如在高负载的情况下，连接的打开时间比关闭的时间快，会引起连接池中idle的个数 上升超过maxIdle，而造成频繁的连接销毁和创建，类似于jvm参数中的Xmx设置)
		4.minIdle：连接池中最小的空闲的连接数，低于这个数量会被创建新的连接（默认为0，调整为5，该参数越接近maxIdle，性能越好，因为连接的创建和销毁，都是需要消耗资源的；但是不能太大，因为在机器很空闲的时候，也会创建低于minidle个数的连接，类似于jvm参数中的Xmn设置）
		5.maxWait  ：最大等待时间，当没有可用连接时，连接池等待连接释放的最大时间，超过该时间限制会抛出异常，如果设置-1表示无限等待（默认为无限，调整为60000ms，避免因线程池不够用，而导致请求被无限制挂起）
		6.poolPreparedStatements：开启池的prepared（默认是false，未调整，经过测试，开启后的性能没有关闭的好。）
		7.maxOpenPreparedStatements：开启池的prepared 后的同时最大连接数（默认无限制，同上，未配置）
		8.minEvictableIdleTimeMillis  ：连接池中连接，在时间段内一直空闲， 被逐出连接池的时间（默认为30分钟，可以适当做调整，需要和后端服务端的策略配置相关）
		9.removeAbandonedTimeout  ：超过时间限制，回收没有用(废弃)的连接（默认为 300秒，调整为180）
		10.removeAbandoned  ：超过removeAbandonedTimeout时间后，是否进 行没用连接（废弃）的回收（默认为false，调整为true)

###2 c3p0
&#160; &#160; &#160; &#160;C3P0是一个开源的JDBC连接池，它实现了数据源和JNDI绑定，支持JDBC3规范和JDBC2的标准扩展。这个连接池可以设置最大和最小连接，连接等待时间等，基本功能都有。

&#160; &#160; &#160; &#160;使用评价：在具体项目应用中，发现此连接池的持续运行的稳定性相当不错，在大并发量的压力下稳定性也有一定保证，此外不提供连接池监控。

####配置详解：

官方文档：http://www.mchange.com/projects/c3p0/index.html

c3p0的配置方式分为三种，分别是

#####2.1.setters一个个地设置各个配置项

&#160; &#160; &#160; &#160;这种方式最繁琐，形式一般是这样：
			Properties props = new Properties();
			InputStream in = ConnectionManager.class.getResourceAsStream("/c3p0.properties");
			props.load(in);
			in.close();	 
			ComboPooledDataSource cpds = new ComboPooledDataSource();
			cpds.setDriverClass(props.getProperty("driverClass"));
			cpds.setJdbcUrl(props.getProperty("jdbcUrl"));
			cpds.setUser(props.getProperty("user"));
			cpds.setPassword(props.getProperty("password"));
			
因为繁琐，所以很不适合采用，于是文档提供了另外另种方式。

#####2.2.类路径下提供一个c3p0.properties文件
		配置示例：
		文件的命名必须是c3p0.properties，里面配置项的格式为：

			c3p0.driverClass=com.mysql.jdbc.Driver
			c3p0.jdbcUrl=jdbc:mysql://localhost:3306/jdbc
			c3p0.user=root
			c3p0.password=java
			
		上面只提供了最基本的配置项，其他配置项参照官方文档配置，记得是c3p0.后面加属性名就是了，最后初始化数据源的方式就是这样简单： 

			private static ComboPooledDataSource ds = new ComboPooledDataSource();
			public static Connection getConnection() {
				try {
					return ds.getConnection();
				} catch (SQLException e) {
					throw new RuntimeException(e);
				}
			}
#####2.3.类路径下提供一个c3p0-config.xml文件
		配置示例：
        <c3p0-config>
		<default-config>
		<!--当连接池中的连接耗尽的时候c3p0一次同时获取的连接数。Default: 3 -->
		<property name="acquireIncrement">3</property>
		<!--定义在从数据库获取新连接失败后重复尝试的次数。Default: 30 -->
		<property name="acquireRetryAttempts">30</property>
		<!--两次连接中间隔时间，单位毫秒。Default: 1000 -->
		<property name="acquireRetryDelay">1000</property>
		<!--连接关闭时默认将所有未提交的操作回滚。Default: false -->
		<property name="autoCommitOnClose">false</property>
		<!--c3p0将建一张名为Test的空表，并使用其自带的查询语句进行测试。如果定义了这个参数那么 属性preferredTestQuery将被忽略。你不能在这张Test表上进行任何操作，它将只供c3p0测试 使用。Default: null-->
		<property name="automaticTestTable">Test</property>
		<!--获取连接失败将会引起所有等待连接池来获取连接的线程抛出异常。但是数据源仍有效 保留，并在下次调用getConnection()的时候继续尝试获取连接。如果设为true，那么在尝试 获取连接失败后该数据源将申明已断开并永久关闭。Default: false-->
		<property name="breakAfterAcquireFailure">false</property>
		<!--当连接池用完时客户端调用getConnection()后等待获取新连接的时间，超时后将抛出 SQLException,如设为0则无限期等待。单位毫秒。Default: 0 -->
		<property name="checkoutTimeout">100</property>
		<!--通过实现ConnectionTester或QueryConnectionTester的类来测试连接。类名需制定全路径。 Default: com.mchange.v2.c3p0.impl.DefaultConnectionTester-->
		<property name="connectionTesterClassName"></property>
		<!--指定c3p0 libraries的路径，如果（通常都是这样）在本地即可获得那么无需设置，默认null即可 Default: null-->
		<property name="factoryClassLocation">null</property>
		<!--Strongly disrecommended. Setting this to true may lead to subtle and bizarre bugs. （文档原文）作者强烈建议不使用的一个属性-->
		<property name="forceIgnoreUnresolvedTransactions">false</property>
		<!--每60秒检查所有连接池中的空闲连接。Default: 0 -->
		<property name="idleConnectionTestPeriod">60</property>
		<!--初始化时获取三个连接，取值应在minPoolSize与maxPoolSize之间。Default: 3 -->
		<property name="initialPoolSize">3</property>
		<!--最大空闲时间,60秒内未使用则连接被丢弃。若为0则永不丢弃。Default: 0 -->
		<property name="maxIdleTime">60</property>
		<!--连接池中保留的最大连接数。Default: 15 -->
		<property name="maxPoolSize">15</property>
		<!--JDBC的标准参数，用以控制数据源内加载的PreparedStatements数量。但由于预缓存的statements 属于单个connection而不是整个连接池。所以设置这个参数需要考虑到多方面的因素。 如果maxStatements与maxStatementsPerConnection均为0，则缓存被关闭。Default: 0-->
		<property name="maxStatements">100</property>
		<!--maxStatementsPerConnection定义了连接池内单个连接所拥有的最大缓存statements数。Default: 0 -->
		<property name="maxStatementsPerConnection"></property>
		<!--c3p0是异步操作的，缓慢的JDBC操作通过帮助进程完成。扩展这些操作可以有效的提升性能 通过多线程实现多个操作同时被执行。Default: 3--> <property name="numHelperThreads">3</property>
		<!--当用户调用getConnection()时使root用户成为去获取连接的用户。主要用于连接池连接非c3p0 的数据源时。Default: null--> <property name="overrideDefaultUser">root</property>
		<!--与overrideDefaultUser参数对应使用的一个参数。Default: null-->
		<property name="overrideDefaultPassword">password</property>
		<!--密码。Default: null-->
		<property name="password"></property>
		<!--定义所有连接测试都执行的测试语句。在使用连接测试的情况下这个一显著提高测试速度。注意： 测试的表必须在初始数据源的时候就存在。Default: null--> <property name="preferredTestQuery">select id from test where id=1</property>
		<!--用户修改系统配置参数执行前最多等待300秒。Default: 300 -->
		<property name="propertyCycle">300</property>
		<!--因性能消耗大请只在需要的时候使用它。如果设为true那么在每个connection提交的 时候都将校验其有效性。建议使用idleConnectionTestPeriod或automaticTestTable 等方法来提升连接测试的性能。Default: false -->
		<property name="testConnectionOnCheckout">false</property>
		<!--如果设为true那么在取得连接的同时将校验连接的有效性。Default: false -->
		<property name="testConnectionOnCheckin">true</property>
		<!--用户名。Default: null-->
		<property name="user">root</property>
		<!--早期的c3p0版本对JDBC接口采用动态反射代理。在早期版本用途广泛的情况下这个参数 允许用户恢复到动态反射代理以解决不稳定的故障。最新的非反射代理更快并且已经开始 广泛的被使用，所以这个参数未必有用。现在原先的动态反射与新的非反射代理同时受到 支持，但今后可能的版本可能不支持动态反射代理。Default: false--> <property name="usesTraditionalReflectiveProxies">false</property>
		<property name="automaticTestTable">con_test</property>
		<property name="checkoutTimeout">30000</property>
		<property name="idleConnectionTestPeriod">30</property>
		<property name="initialPoolSize">10</property>
		<property name="maxIdleTime">30</property>
		<property name="maxPoolSize">25</property>
		<property name="minPoolSize">10</property>
		<property name="maxStatements">0</property>
		<user-overrides user="swaldman">
		</user-overrides> </default-config>
		<named-config name="dumbTestConfig">
		<property name="maxStatements">200</property>
		<user-overrides user="poop">
		<property name="maxStatements">300</property>
		</user-overrides> </named-config> </c3p0-config>

###3 proxool

&#160; &#160; &#160; &#160;proxool这个连接池可能用到的人比较少，但也有一定知名度，这个连接池可以设置最大和最小连接，连接等待时间等，基本功能都有,sourceforge下的一个开源项目,这个项目提供一个健壮、易用的连接池，最为关键的是这个连接池提供监控的功能，方便易用，便于发现连接泄漏的情况。

&#160; &#160; &#160; &#160;使用评价：在具体项目应用中，发现此连接池的持续运行的稳定性有一定问题，有一个需要长时间跑批的任务场景任务，同样的代码在另外2个开源连接池中成功结束，但在proxool中出现异常退出。但是proxool有一个优势--连接池监控，这是个很诱人的东西，大概的配置方式就是在web.xml中添加如下定义：

    <servlet>
        <servlet-name>admin</servlet-name>
        <servlet-class>org.logicalcobwebs.proxool.admin.servlet.AdminServlet</servlet-class>     
   </servlet>
   <servlet-mapping>
      <servlet-name>admin</servlet-name>
      <url-pattern>/admin</url-pattern>
   </servlet-mapping>  
   
并在应用启动后访问：http://localhost:8080/myapp/admin这个url即可监控,不过proxool本身的包在监测使用中会有编码问题

####配置详解：

#####(1) 建立proxool.xml文件,路径为根目录src下即与hibernate.cfg.xml同目录,内容如下:
	　　<?xml version="1.0" encoding="UTF-8"?>
	　　<!--the proxool configuration can be embedded within your own application's.Anything outside the "proxool" tag is ignored.-->
	　　<something-else-entirely>
	　　	<proxool>
	　　		<alias>mssqlProxool</alias>
	　　		<driver-url>jdbc:sqlserver://XXX.XXX.XXX.XX:1433;databaseName=XXX</driver-url>
	　　		<driver-class>com.microsoft.sqlserver.jdbc.SQLServerDriver</driver-class>
	　　		<driver-properties>
	　　			<property name="user" value="sa" />
	　　			<property name="password" value="XXX" />
	　　		</driver-properties>
	　　		<house-keeping-test-sql>select CURRENT_DATE</house-keeping-test-sql>
	　　		<house-keeping-sleep-time>90000</house-keeping-sleep-time>
	　　		<simultaneous-build-throttle>20</simultaneous-build-throttle>
	　　		<maximum-connection-count>100</maximum-connection-count>
	　　		<minimum-connection-count>10</minimum-connection-count>
	　　		<maximum-connection-lifetime>3600000</maximum-connection-lifetime>
	　　	</proxool>
	　　</something-else-entirely>
	　　属性列表说明:
	　　fatal-sql-exception: 它是一个逗号分割的信息片段.当一个SQL异常发生时,他的异常信息将与这个信息片段进行比较.如果在片段中存在,那么这个异常将被认为是个致命错误 (Fatal SQL Exception ).
			这种情况下,数据库连接将要被放弃.无论发生什么,这个异常将会被重掷以提供给消费者.用户最好自己配置一个不同的异常来抛出.
			
	　　fatal-sql-exception-wrapper-class:正如上面所说,你最好配置一个不同的异常来重掷.利用这个属性,用户可以包装SQLException,使他变成另外一个异常.这个异常或者继承SQLException或者继承字RuntimeException.proxool,
			自带了2个实现:'org.logicalcobwebs.proxool.FatalSQLException' 和'org.logicalcobwebs.proxool.FatalRuntimeException'.后者更合适.
			
	　　house-keeping-sleep-time: house keeper 保留线程处于睡眠状态的最长时间,house keeper 的职责就是检查各个连接的状态,并判断是否需要销毁或者创建.
	　　house-keeping-test-sql: 如果发现了空闲的数据库连接.house keeper 将会用这个语句来测试.这个语句最好非常快的被执行.如果没有定义,测试过程将会被忽略。
	　　injectable-connection-interface: 允许proxool实现被代理的connection对象的方法.
	　　injectable-statement-interface: 允许proxool实现被代理的Statement 对象方法.
	　　injectable-prepared-statement-interface: 允许proxool实现被代理的PreparedStatement 对象方法.
	　　injectable-callable-statement-interface: 允许proxool实现被代理的CallableStatement 对象方法.
	　　jmx: 如果属性为true，就会注册一个消息Bean到jms服务，消息Bean对象名: "Proxool:type=Pool, name=<alias>". 默认值为false.
	　　jmx-agent-id: 一个逗号分隔的JMX代理列表(如使用MBeanServerFactory.findMBeanServer(String agentId)注册的连接池。）这个属性是仅当"jmx"属性设置为"true"才有效。所有注册jmx服务器使用这个属性是不确定的
	　　jndi-name: 数据源的名称
	　　maximum-active-time: 如果housekeeper 检测到某个线程的活动时间大于这个数值.它将会杀掉这个线程.所以确认一下你的服务器的带宽.然后定一个合适的值.默认是5分钟.
	　　maximum-connection-count: 最大的数据库连接数.
	　　maximum-connection-lifetime: 一个线程的最大寿命.
	　　minimum-connection-count: 最小的数据库连接数
	　　overload-without-refusal-lifetime: 这可以帮助我们确定连接池的状态。如果我们已经拒绝了一个连接在这个设定值(毫秒),然后被认为是超载。默认为60秒。
	　　prototype-count: 连接池中可用的连接数量.如果当前的连接池中的连接少于这个数值.新的连接将被建立(假设没有超过最大可用数).例如.我们有3个活动连接2个可用连接,而我们的prototype-count是4,那么数
			据库连接池将试图建立另外2个连接.这和 minimum-connection-count不同. minimum-connection-count把活动的连接也计算在内.prototype-count 是spare connections 的数量.
			
	　　recently-started-threshold: 这可以帮助我们确定连接池的状态,连接数少还是多或超载。只要至少有一个连接已开始在此值(毫秒)内,或者有一些多余的可用连接,那么我们假设连接池是开启的。默认为60秒

	　　simultaneous-build-throttle: 这是我们可一次建立的最大连接数。那就是新增的连接请求,但还没有可供使用的连接。由于连接可以使用多线程,在有限的时间之间建立联系从而带来可用连接,
			但是我们需要通过一些方式确认一些线程并不是立即响应连接请求的，默认是10。
		
	　　statistics: 连接池使用状况统计。参数“10s,1m,1d”
	　　statistics-log-level: 日志统计跟踪类型。参数“ERROR”或 “INFO”
	　　test-before-use: 如果为true，在每个连接被测试前都会服务这个连接，如果一个连接失败，那么将被丢弃，另一个连接将会被处理，如果所有连接都失败，一个新的连接将会被建立。否则将会抛出一个SQLException异常。
	　　test-after-use: 如果为true，在每个连接被测试后都会服务这个连接，使其回到连接池中，如果连接失败，那么将被废弃。
	　　trace: 如果为true,那么每个被执行的SQL语句将会在执行期被log记录(DEBUG LEVEL).你也可以注册一个ConnectionListener (参看ProxoolFacade)得到这些信息.
	
#####(2)修改hibernate.cfg.xml文件,内容如下:

	　　<?xml version='1.0' encoding='UTF-8'?>
	　　<!DOCTYPE hibernate-configuration PUBLIC"-//Hibernate/Hibernate Configuration DTD 3.0//EN" "http://hibernate.sourceforge.net/hibernate-configuration-3.0.dtd">
	　　<!-- Generated by MyEclipse Hibernate Tools. -->
	　　<hibernate-configuration>
	　　	<session-factory>
	　　		<property name="hibernate.proxool.pool_alias">mssqlProxool</property>
	　　		<property name="hibernate.proxool.xml">proxool.xml</property>
				<property name="hibernate.connection.provider_class">org.hibernate.connection.ProxoolConnectionProvider</property>
	　　		<property name="hibernate.proxool.existing_pool">true</property>
	　　		<property name="dialect">org.hibernate.dialect.SQLServerDialect</property>
	　　		<property name="hibernate.cache.use_query_cache">true</property>
	　　	</session-factory>
	　　</hibernate-configuration>

	　　这里需注意三点:
	　　1.别名与proxool中的别名保持一致.
	　　2.路径确保正确
	　　3一般网上只说明了以上两点,hibernate.proxool.existing_pool 这个参数很重要hibernate.proxool.existing_pool：此值设为 false，当 hibernate 开始被调用时，就会初始化 proxool，进行数据库连接等操作.

#####(3)要让程序直接使用proxool连接池,可以在web.xml中配置初始化servlet,在web容器加载的时候自动加载配置文件

	　　<servlet>
	　　	<servlet-name>ServletConfigurator </servlet-name>
	　　	<servlet-class>org.logicalcobwebs.proxool.configuration.ServletConfigurator</servlet-class>
	　　	<init-param>
	　　		<param-name>xmlFile</param-name>
	　　		<param-value>/WEB-INF/classes/proxool.xml</param-value>
	　　	</init-param>
	　　	<load-on-startup>1</load-on-startup>
	　　</servlet>

	　　注意:如果同时配置了web.xml和hibernate.cfg.xml会产生错误:org.logicalcobwebs.proxool.ProxoolException: Parsing failed.因为同名的proxool连接池已经启动，而hibernate开始运行时会自己启动关联的proxool连接池.
			所以此时应改变hibernate.cfg.xml配置为:<property name="hibernate.proxool.existing_pool">true</property>

#####(4)这一步是可选的 在应用中实时监控连接池

	　　<servlet>
	　　	<servlet-name>adminProxool</servlet-name>
	　　	<servlet-class>org.logicalcobwebs.proxool.admin.servlet.AdminServlet</servlet-class>
	　　</servlet>
	　　<servlet-mapping>
	　　	<servlet-name>adminProxool</servlet-name>
	　　	<url-pattern>/admin/proxool</url-pattern>
	　　</servlet-mapping>

　　	访问http://localhost:8080/项目名称/admin/proxool即可看到页面

###4.druid
&#160; &#160; &#160; &#160;Druid是一个用于大数据实时查询和分析的高容错、高性能开源分布式系统，旨在快速处理大规模的数据，并能够实现快速查询和分析。尤其是当发生代码部署、机器故障以及其他产品系统遇到宕机等情况时，Druid仍能够保持100%正常运行。创建Druid的最初意图主要是为了解决查询延迟问题，当时试图使用Hadoop来实现交互式查询分析，但是很难满足实时分析的需要。而Druid提供了以交互方式访问数据的能力，并权衡了查询的灵活性和性能而采取了特殊的存储格式。

&#160; &#160; &#160; &#160;使用评价：在负载较小的情况下Druid的优势比较明显，在大规模查询情况下也表现出来比较好的性能，至少不比c3p0差。

####druid配置详解：

#####4.1 Druid依赖代码,jar包依赖:

		<dependency>  
            <groupId>com.alibaba</groupId>  
            <artifactId>druid</artifactId>  
            <version>0.2.15</version>  
        </dependency>  
		下载地址：http://maven.outofmemory.cn/com.alibaba/druid/
		
#####4.2 applicationContext-resources.xml 配置数据库连接池，以mysql数据库为例

			<!-- JNDI DataSource for J2EE environments -->  
			<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close">  
				<property name="url" value="jdbc:mysql://127.0.0.1:3306/XXX" />  
				<property name="username" value="root" />  
				<property name="password" value="root" />  
				<property name="maxActive" value="20" />  
				<property name="initialSize" value="1" />  
				<property name="maxWait" value="60000" />  
				<property name="minIdle" value="1" />  
				<property name="timeBetweenEvictionRunsMillis" value="3000" />  
				<property name="minEvictableIdleTimeMillis" value="300000" />  
				<property name="validationQuery" value="SELECT 'x' FROM DUAL" />  
				<property name="testWhileIdle" value="true" />  
				<property name="testOnBorrow" value="false" />  
				<property name="testOnReturn" value="false" />  
				<!-- mysql 不支持 poolPreparedStatements-->  
				<!--<property name="poolPreparedStatements" value="true" />-->  
				<!--<property name="maxPoolPreparedStatementPerConnectionSize" value="20" />-->  
				<!-- 开启Druid的监控统计功能 -->  
				<property name="filters" value="stat" />   
			</bean>
		另oracle:
			<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" destroy-method="close">
				<!-- 数据库基本信息配置 -->
				<property name="url" value="${kjne.druid.master.url}" />
				<property name="username" value="${kjne.druid.master.username}" />
				<property name="password" value="${kjne.druid.master.password}" />
				<property name="driverClassName" value="${kjne.druid.master.driverClassName}" />
				<property name="filters" value="${kjne.druid.master.filters}" />
				<!-- 最大并发连接数 -->
				<property name="maxActive" value="${kjne.druid.master.maxActive}" />
				<!-- 初始化连接数量 -->
				<property name="initialSize" value="${kjne.druid.master.initialSize}" />
				<!-- 配置获取连接等待超时的时间 -->
				<property name="maxWait" value="${kjne.druid.master.maxWait}" />
				<!-- 最小空闲连接数 -->
				<property name="minIdle" value="${kjne.druid.master.minIdle}" />
				<!-- 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒 -->
				<property name="timeBetweenEvictionRunsMillis" value="${kjne.druid.master.timeBetweenEvictionRunsMillis}" />
				<!-- 配置一个连接在池中最小生存的时间，单位是毫秒 -->
				<property name="minEvictableIdleTimeMillis" value="${kjne.druid.master.minEvictableIdleTimeMillis}" />
				<property name="validationQuery" value="${kjne.druid.master.validationQuery}" />
				<property name="testWhileIdle" value="${kjne.druid.master.testWhileIdle}" />
				<property name="testOnBorrow" value="${kjne.druid.master.testOnBorrow}" />
				<property name="testOnReturn" value="${kjne.druid.master.testOnReturn}" />
				<property name="maxOpenPreparedStatements" value="${kjne.druid.master.maxOpenPreparedStatements}" />
				<!-- 打开removeAbandoned功能 -->
				<property name="removeAbandoned" value="${kjne.druid.master.removeAbandoned}" />
				<!-- 1800秒，也就是30分钟 -->
				<property name="removeAbandonedTimeout" value="${kjne.druid.master.removeAbandonedTimeout}" />
				<!-- 关闭abanded连接时输出错误日志 -->
				<property name="logAbandoned" value="${kjne.druid.master.logAbandoned}" />
			</bean>
			
#####4.3.内置监控界面使用配置修改web.xml，加入如下内容：

			</servlet-mapping>
				<servlet>
				<servlet-name>DruidStatView</servlet-name>
				<servlet-class>com.alibaba.druid.support.http.StatViewServlet</servlet-class>
				</servlet>
				<servlet-mapping>
				<servlet-name>DruidStatView</servlet-name>
				<url-pattern>/druid/*</url-pattern>
			</servlet-mapping>
			
#####4.4.jdbc.properties配置：	
			和其它连接池一样DRUID的DataSource类为：com.alibaba.druid.pool.DruidDataSource，基本配置参数如下：
			name:配置这个属性的意义在于，如果存在多个数据源，监控的时候可以通过名字来区分开来。如果没有配置，将会生成一个名字，格式是："DataSource-" + System.identityHashCode(this)
			jdbcUrl:连接数据库的url，不同数据库不一样。例如:mysql:jdbc:mysql://10.20.153.104:3306/druid2 	oracle : jdbc:oracle:thin:@10.20.149.85:1521:ocnauto
			username:连接数据库的用户名
			password:连接数据库的密码。如果你不希望密码直接写在配置文件中，可以使用ConfigFilter。详细看这里：https://github.com/alibaba/druid/wiki/%E4%BD%BF%E7%94%A8ConfigFilter
			driverClassName:根据url自动识别	这一项可配可不配，如果不配置druid会根据url自动识别dbType，然后选择相应的driverClassName(建议配置下)
			initialSize:缺省值为0,初始化时建立物理连接的个数。初始化发生在显示调用init方法，或者第一次getConnection时
			maxActive:缺省值为8,最大连接池数量
			maxIdle:缺省值为8,已经不再使用，配置了也没效果
			minIdle:最小连接池数量
			maxWait:获取连接时最大等待时间，单位毫秒。配置了maxWait之后，缺省启用公平锁，并发效率会有所下降，如果需要可以通过配置useUnfairLock属性为true使用非公平锁。
			poolPreparedStatements:缺省值为false,是否缓存preparedStatement，也就是PSCache。PSCache对支持游标的数据库性能提升巨大，比如说oracle。在mysql下建议关闭。
			maxOpenPreparedStatements:缺省值为-1,要启用PSCache，必须配置大于0，当大于0时，poolPreparedStatements自动触发修改为true。在Druid中，不会存在Oracle下PSCache占用内存过多的问题，可以把这个数值配置大一些，比如说100
			validationQuery:用来检测连接是否有效的sql，要求是一个查询语句。如果validationQuery为null，testOnBorrow、testOnReturn、testWhileIdle都不会其作用。
			testOnBorrow:缺省值为true,申请连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能。
			testOnReturn:缺省值为false,归还连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能
			testWhileIdle:缺省值为false,建议配置为true，不影响性能，并且保证安全性。申请连接的时候检测，如果空闲时间大于timeBetweenEvictionRunsMillis，执行validationQuery检测连接是否有效。
			timeBetweenEvictionRunsMillis:有两个含义： 1) Destroy线程会检测连接的间隔时间2) testWhileIdle的判断依据，详细看testWhileIdle属性的说明
			numTestsPerEvictionRun:不再使用，一个DruidDataSource只支持一个EvictionRun
			connectionInitSqls:物理连接初始化的时候执行的sql
			exceptionSorter:缺省值根据dbType自动识别，当数据库抛出一些不可恢复的异常时，抛弃连接
			filters:属性类型是字符串，通过别名的方式配置扩展插件，常用的插件有： 监控统计用的filter:stat日志用的filter:log4j防御sql注入的filter:wall
			proxyFilters:类型是List<com.alibaba.druid.filter.Filter>，如果同时配置了filters和proxyFilters，是组合关系，并非替换关系
			示例：
			kjne.druid.master.url=jdbc:oracle:thin:@192.168.0.2:1521:csdb
			kjne.druid.master.driverClassName=oracle.jdbc.OracleDriver
			kjne.druid.master.username=kjne
			kjne.druid.master.password=kjne
			kjne.druid.master.filters=stat
			kjne.druid.master.maxActive=200
			kjne.druid.master.initialSize=1
			kjne.druid.master.maxWait=60000
			kjne.druid.master.minIdle=10
			kjne.druid.master.maxIdle=15
			kjne.druid.master.timeBetweenEvictionRunsMillis=60000
			kjne.druid.master.minEvictableIdleTimeMillis=300000
			kjne.druid.master.validationQuery=SELECT 'x' FROM DUAL
			kjne.druid.master.testWhileIdle=true
			kjne.druid.master.testOnBorrow=false
			kjne.druid.master.testOnReturn=false
			kjne.druid.master.maxOpenPreparedStatements=20
			kjne.druid.master.removeAbandoned=true
			kjne.druid.master.removeAbandonedTimeout=1800
			kjne.druid.master.logAbandoned=true
		