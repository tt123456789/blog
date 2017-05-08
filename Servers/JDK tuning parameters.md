Title: jdk调优参数
Date: 11:11 2016/10/21
Modified: 11:11 2016/10/21
Category: Servers
Tags: JAVA
Slug: 
Author: allposs

##简介##
&#160; &#160; &#160; &#160;个人收集的一些tomcat的jdk调优参数，没有什么文采，请见谅！


##版本##

V1.0


##正文##
####1.调优示例：
	JAVA_OPS="-Xms12480m -Xmx12480m -XX:PermSize=512m -XX:MaxPermSize=512m -Xmn512m -Xss256K  -XX：+UseConcMarkSweepGC  
	-XX:+CMSClassUnloadingEnabled  -XX:+UseParNewGC  -XX:+CMSParallelRemarkEnabled -XX:SoftRefLRUPolicyMSPerMB=0  
	-XX:+UseFastAccessorMethods -XX:+DisableExplicitGC -XX:ParallelGCThreads=10  -XX:CMSFullGCsBeforeCompaction=0
	-XX:+UseCMSCompactAtFullCollection -XX:SurvivorRatio=1  -XX:LargePageSizeInBytes=128m -XX:+UseCMSInitiatingOccupancyOnly -XX:CMSInitiatingOccupancyFraction=70 
	-XX:+UseCompressedOops -XX:+CMSScavengeBeforeRemark  -XX:MaxTenuringThreshold=15 -XX:TargetSurvivorRatio=90 -XX:+HeapDumpOnOutOfMemoryError -XX:+ShowMessageBoxOnError


####2.CMS收集器的过程
	CMS收集器的GC周期由6个阶段组成。其中4个阶段(名字以Concurrent开始的)与实际的应用程序是并发执行的，而其他2个阶段需要暂停应用程序线程。
	初始标记：为了收集应用程序的对象引用需要暂停应用程序线程，该阶段完成后，应用程序线程再次启动。
	并发标记：从第一阶段收集到的对象引用开始，遍历所有其他的对象引用。
	并发预清理：改变当运行第二阶段时，由应用程序线程产生的对象引用，以更新第二阶段的结果。
	重标记：由于第三阶段是并发的，对象引用可能会发生进一步改变。因此，应用程序线程会再一次被暂停以更新这些变化，并且在进行实际的清理之前确保一个正确的对象引用视图。这一阶段十分重要，因为必须避免收集到仍被引用的对象。
	并发清理：所有不再被应用的对象将从堆里清除掉。
	并发重置：收集器做一些收尾的工作，以便下一次GC周期能有一个干净的状态。
	一个常见的误解是,CMS收集器运行是完全与应用程序并发的。我们已经看到，事实并非如此，即使“stop-the-world”阶段相对于并发阶段的时间很短。
	应该指出，尽管CMS收集器为老年代垃圾回收提供了几乎完全并发的解决方案，然而年轻代仍然通过“stop-the-world”方法来进行收集。对于交互式应用，停顿也是可接受的，背后的原理是年轻带的垃圾回收时间通常是相当短的。

####2.CMS收集器的问题
	当我们在真实的应用中使用CMS收集器时，我们会面临两个主要的挑战，可能需要进行调优：
	堆碎片
	对象分配率高

	1.堆碎片是有可能的，不像吞吐量收集器，CMS收集器并没有任何碎片整理的机制。因此，应用程序有可能出现这样的情形，即使总的堆大小远没有耗尽，但却不能分配对象——仅仅是因为没有足够连续的空间完全容纳对象。
	当这种事发生后，并发算法不会帮上任何忙，因此，万不得已JVM会触发Full GC。回想一下，Full GC 将运行吞吐量收集器的算法，从而解决碎片问题——但却暂停了应用程序线程。因此尽管CMS收集器带来完全的并发性，
	但仍然有可能发生长时间的“stop-the-world”的风险。这是“设计”，而不能避免的——我们只能通过调优收集器来它的可能性。想要100%保证避免”stop-the-world”，对于交互式应用是有问题的。

	2.第二个挑战就是应用的对象分配率高。如果获取对象实例的频率高于收集器清除堆里死对象的频率，并发算法将再次失败。从某种程度上说，老年代将没有足够的可用空间来容纳一个从年轻代提升过来的对象。
	这种情况被称为“并发模式失败”，并且JVM会执行堆碎片整理：触发Full GC。当这些情形之一出现在实践中时(经常会出现在生产系统中)，经常被证实是老年代有大量不必要的对象。
	一个可行的办法就是增加年轻代的堆大小，以防止年轻代短生命的对象提前进入老年代。另一个办法就似乎利用分析器，快照运行系统的堆转储，并且分析过度的对象分配，找出这些对象，最终减少这些对象的申请。

####3.调优参数

	-XX:+UseConcMarkSweepGC 
		该标志首先是激活CMS收集器。默认HotSpot JVM使用的是并行收集器。
	-XX:+UseParNewGC 
		当使用CMS收集器时，该标志激活年轻代使用多线程并行执行垃圾回收。注意最新的JVM版本，当使用-XX：+UseConcMarkSweepGC时，-XX：UseParNewGC会自动开启。因此，如果年轻代的并行GC不想开启，可以通过设置-XX：-UseParNewGC来关掉。
	-XX：+CMSConcurrentMTEnabled 
		当该标志被启用时，并发的CMS阶段将以多线程执行(因此，多个GC线程会与所有的应用程序线程并行工作)。该标志已经默认开启，如果顺序执行更好，这取决于所使用的硬件，多线程执行可以通过-XX：-CMSConcurremntMTEnabled禁用。
	-XX：ConcGCThreads	
		-XX：ConcGCThreads=<value> (早期JVM版本也叫-XX:ParallelCMSThreads)定义并发CMS过程运行时的线程数。比如value=4意味着CMS周期的所有阶段都以4个线程来执行。
		尽管更多的线程会加快并发CMS过程，但其也会带来额外的同步开销。因此，对于特定的应用程序，应该通过测试来判断增加CMS线程数是否真的能够带来性能的提升。
		如果还标志未设置，JVM会根据并行收集器中的-XX：ParallelGCThreads参数的值来计算出默认的并行CMS线程数。该公式是ConcGCThreads = (ParallelGCThreads + 3)/4。
		因此，对于CMS收集器， -XX:ParallelGCThreads标志不仅影响“stop-the-world”垃圾收集阶段，还影响并发阶段。
		总之，有不少方法可以配置CMS收集器的多线程执行。正是由于这个原因,建议第一次运行CMS收集器时使用其默认设置, 然后如果需要调优再进行测试。
		只有在生产系统中测量(或类生产测试系统)发现应用程序的暂停时间的目标没有达到 , 就可以通过这些标志应该进行GC调优。
	-XX:CMSInitiatingOccupancyFraction	
		当堆满之后，并行收集器便开始进行垃圾收集，例如，当没有足够的空间来容纳新分配或提升的对象。
		对于CMS收集器，长时间等待是不可取的，因为在并发垃圾收集期间应用持续在运行(并且分配对象)。因此，为了在应用程序使用完内存之前完成垃圾收集周期，
		CMS收集器要比并行收集器更先启动。因为不同的应用会有不同对象分配模式，JVM会收集实际的对象分配(和释放)的运行时数据，并且分析这些数据，
		来决定什么时候启动一次CMS垃圾收集周期。为了引导这一过程， JVM会在一开始执行CMS周期前作一些线索查找。
		该线索由 -XX:CMSInitiatingOccupancyFraction=<value>来设置，该值代表老年代堆空间的使用率。比如，value=75意味着第一次CMS垃圾收集会在老年代被占用75%时被触发。
		通常CMSInitiatingOccupancyFraction的默认值为68(之前很长时间的经历来决定的)。

	-XX：+UseCMSInitiatingOccupancyOnly
		我们用-XX+UseCMSInitiatingOccupancyOnly标志来命令JVM不基于运行时收集的数据来启动CMS垃圾收集周期。而是，当该标志被开启时，JVM通过CMSInitiatingOccupancyFraction的值进行每一次CMS收集，而不仅仅是第一次。
		然而，请记住大多数情况下，JVM比我们自己能作出更好的垃圾收集决策。因此，只有当我们充足的理由(比如测试)并且对应用程序产生的对象的生命周期有深刻的认知时，才应该使用该标志。

	-XX:+CMSClassUnloadingEnabled
		相对于并行收集器，CMS收集器默认不会对永久代进行垃圾回收。如果希望对永久代进行垃圾回收，可用设置标志-XX:+CMSClassUnloadingEnabled。
		在早期JVM版本中，要求设置额外的标志-XX:+CMSPermGenSweepingEnabled。注意，即使没有设置这个标志，一旦永久代耗尽空间也会尝试进行垃圾回收，但是收集不会是并行的，而再一次进行Full GC。

	-XX:+CMSIncrementalMode
		该标志将开启CMS收集器的增量模式。增量模式经常暂停CMS过程，以便对应用程序线程作出完全的让步。因此，收集器将花更长的时间完成整个收集周期。
		因此，只有通过测试后发现正常CMS周期对应用程序线程干扰太大时，才应该使用增量模式。由于现代服务器有足够的处理器来适应并发的垃圾收集，所以这种情况发生得很少。

	-XX:+ExplicitGCInvokesConcurrent and -XX:+ExplicitGCInvokesConcurrentAndUnloadsClasses
		如今,被广泛接受的最佳实践是避免显式地调用GC(所谓的“系统GC”)，即在应用程序中调用system.gc()。然而，这个建议是不管使用的GC算法的，值得一提的是，当使用CMS收集器时，系统GC将是一件很不幸的事，因为它默认会触发一次Full GC。
		幸运的是，有一种方式可以改变默认设置。标志-XX:+ExplicitGCInvokesConcurrent命令JVM无论什么时候调用系统GC，都执行CMS GC，而不是Full GC。
		第二个标志-XX:+ExplicitGCInvokesConcurrentAndUnloadsClasses保证当有系统GC调用时，永久代也被包括进CMS垃圾回收的范围内。因此，通过使用这些标志，我们可以防止出现意料之外的”stop-the-world”的系统GC。

	-XX:+DisableExplicitGC
		然而在这个问题上…这是一个很好提到- XX:+ DisableExplicitGC标志的机会，该标志将告诉JVM完全忽略系统的GC调用(不管使用的收集器是什么类型)。
		对于我而言，该标志属于默认的标志集合中，可以安全地定义在每个JVM上运行，而不需要进一步思考。
	-XX:+CMSParallelRemarkEnabled
		CMS是不会整理堆碎片的，因此为了防止堆碎片引起full gc，通过会开启CMS阶段进行合并碎片选项：-XX:+UseCMSCompactAtFullCollection，开启这个选项一定程度上会影响性能，
		
	-XX:SoftRefLRUPolicyMSPerMB=0
		每兆堆空闲空间中SoftReference的存活时间

	-XX:+UseFastAccessorMethods	
		原始类型的快速优化
	-XX:+DisableExplicitGC
		关闭System.gc()
	-XX:ParallelGCThreads	
		并行收集器的线程数,此值最好配置与处理器数目相等,同样适用于CMS
		
	-XX:+UseCMSCompactAtFullCollection
		使用并发收集器时,开启对年老代的压缩.
	-XX:CMSFullGCsBeforeCompaction=0
		-XX:+UseCMSCompactAtFullCollection配置开启的情况下,这里设置多少次Full GC后,对年老代进行压缩

	-XX:SurvivorRatio	
		Eden区与Survivor区的大小比值,设置为8,则两个Survivor区与一个Eden区的比值为2:8,一个Survivor区占整个年轻代的1/10

	-XX:LargePageSizeInBytes	
		内存页的大小不可设置过大，会影响Perm的大小,最好128m
		
		
	-XX:MaxPermSize=512m 表示永久区为512m 
	-XX:+DisableExplicitGC 禁用显示的gc，程序程序中使用System.gc()中进行垃圾回收，使用这个参数后系统自动将 System.gc() 调用转换成一个空操作
	-XX:+UseConcMarkSweepGC 表示使用CMS 
	-XX:+CMSParallelRemarkEnabled 表示并行remark 
	-XX:+UseCMSCompactAtFullCollection 表示在FGC之后进行压缩，因为CMS默认不压缩空间的。 
	-XX:+UseCMSInitiatingOccupancyOnly 表示只在到达阀值的时候，才进行CMS GC
	-XX:CMSInitiatingOccupancyFraction=70 设置阀值为70%，默认为68%。 
	-XX:+UseCompressedOops 	JVM优化之压缩普通对象指针（CompressedOops），通常64位JVM消耗的内存会比32位的大1.5倍，这是因为对象指针在64位架构下，长度会翻倍（更宽的寻址）。对于那些将要从32位平台移植到64位的应用来说，平白无辜多了1/2的内存占用，这是开发者不愿意看到的。幸运的是，从JDK 1.6 update14开始，64 bit JVM正式支持了 -XX:+UseCompressedOops 这个可以压缩指针，起到节约内存占用的新参数.
	-XX:CMSFullGCsBeforeCompaction=1 	设置多少次full gc后进行内存压缩，由于并发收集器不对内存空间进行压缩,整理,所以运行一段时间以后会产生"碎片",使得运行效率降低.此值设置运行多少次GC以后对内存空间进行压缩,整理。
	-XX:ParallelGCThreads=20  配置并行收集器的线程数，即：同时多少个线程一起进行垃圾回收。此值最好配置与处理器数目相等。


	avaTM平台提供了垃圾回收算法的选择。对于每一种算法存在有许多个可调参数。通常来说，下面的前两个是大型服务器应用最常用的选择：
	-XX:+UseParallelGC 并行(吞吐)垃圾回收 此配置仅对年轻代有效。即上述配置下，年轻代使用并发收集，而年老代仍旧使用串行收集。
	-XX:+UseConcMarkSweepGC 并发(低暂停时间)垃圾回收
	-XX:+UseSerialGC 串行垃圾回收(对小的应用和系统)




	-Xdebug -Xnoagent -Xrunjdwp:transport=dt_socket,address=7777,server=y,suspend=n 开启debug模式

	jmap -dump:format=b,file=dumpFileName pid