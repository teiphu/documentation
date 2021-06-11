**请注意，此页面仅适用于 JDK 7 及更早版本。对于 JDK 8，请参阅 Windows、Solaris 参考页面。**

本文档提供有关可能影响Java HotSpot虚拟机性能特征的典型命令行选项和环境变量的信息。除非另有说明，本文档中的所有信息均与Java HotSpot Client VM和Java HotSpot Server VM相关。

## Java HotSpot VM 选项的类别
Java HotSpot VM 识别的标准选项在适用于Windows和Solaris&Linux的Java应用程序启动器参考页面中进行了描述。本文档专门处理Java HotSpot VM识别的非标准选项:
* 以-X开头的选项是非标准的（不保证所有VM实现都支持），并且在JDK的后续版本中如有更改，恕不另行通知。

* 使用-XX指定的选项不稳定，如有更改，恕不另行通知。

希望移植到Java HotSpot VM早于1.3.0的JDK用户应该看[Java HotSpot Equivalents of Exact VM flags](https://www.oracle.com/java/technologies/javase/exactoptions-jsp.html)

## 一些有用的-XX选项
列出了带有-server的Java SE 6 for Solaris Sparc的默认值。某些选项可能因架构/操作系统/JVM版本而异。描述中列出了具有不同默认值的平台。

* Boolean选项用 \-XX:\+\<option\> 打开，用 \-XX:\-\<option\>.Disa 关闭
* Numeric选项设置为 \-XX:\<option\>=\<number\>。数字可以包括'm'或'M'表示兆字节(MB)，'k'或'K'表示千字节(KB)，'g'或'G'表示千兆字节(GB)（例如，32k等于32768）
* String选项设置为 \-XX:\<option\>=\<string\>，通常用于指定一个文件，一个路径或者命令列表。

标记为可管理的标志可通过JDK管理接口(com.sun.management.HotSpotDiagnosticMXBean API)和JConsole动态写入。在[Monitoring and Managing Java SE 6 Platform Applications](https://www.oracle.com/technical-resources/articles/javase/monitoring.html#Heap_Dump)中，图3显示了一个示例。也可以通过jinfo \-flag设置可管理标志。下面的选项被松散地分为几类。

* [Behavioral options](https://www.oracle.com/java/technologies/javase/vmoptions-jsp.html#BehavioralOptions)改变虚拟机的基本行为。
* [Garbage First (G1) Garbage Collection Options](https://www.oracle.com/java/technologies/javase/vmoptions-jsp.html#G1Options)
* [Performance tuning](https://www.oracle.com/java/technologies/javase/vmoptions-jsp.html#PerformanceTuning)选项是可用于调整VM性能的旋钮。
* [Debugging options](https://www.oracle.com/java/technologies/javase/vmoptions-jsp.html#DebuggingOptions)通常用于启用VM信息的跟踪、打印或输出。

#### Behavioral Options

|**Option and Default Value**|**Description**|
|:-------------------|:----------|
|-XX:-AllowUserSignalHandlers|如果应用程序安装了信号处理程序，不要抱怨。（仅与Solaris和Linux有关）|
|-XX:AltStackSize=16384|备用信号堆栈大小（单位为KB）。（仅与Solaris相关，从5.0中删除）|
|-XX:-DisableExplicitGC|默认情况下启用对System.gc()的调用(-XX:-DisableExplicitGC)。使用-XX:+DisableExplicitGC禁用对 System.gc()的调用。请注意，JVM仍会在必要时执行垃圾收集。|
|-XX:+FailOverToOldVerifier|当新的类型检查器失败时，故障转移到旧的验证器。（在6中引入）|
|-XX:+HandlePromotionFailure|最年轻代集合不需要保证所有存活对象都被完全提升。（在1.4.2 update 11中引入）[5.0及更早版本：false。]|
|-XX:+MaxFDLimit|将文件描述符的数量增加到最大值。（仅与Solaris相关）|
|-XX:PreBlockSpin=10|与-XX:+UseSpinning一起使用的旋转计数变量。控制进入操作系统线程同步代码之前允许的最大自旋迭代次数。（在1.4.2中引入）|
|-XX:-RelaxAccessControlCheck|放宽验证程序中的访问控制检查。（在6中引入）|
|-XX:+ScavengeBeforeFullGC|在full GC之前执行年轻代GC。（在1.4.1中引入）|
|-XX:+UseAltSigs|对于VM内部信号，使用备用信号而不是SIGUSR1和SIGUSR2。（在 1.3.1 update 9, 1.4.1中引入。仅与Solaris相关。）|
|-XX:+UseBoundThreads|将用户级线程绑定到内核线程。（仅与Solaris相关）|
|-XX:-UseConcMarkSweepGC|对老年代使用concurrent mark-sweep collection（CMS垃圾收集器）。（在1.4.1中引入）|
|-XX:+UseGCOverheadLimit|在抛出OutOfMemory错误之前，使用一个策略来限制VM花费在GC中的时间比例。（在6中引入）|
|-XX:+UseLWPSynchronization|使用基于LWP的同步而不是基于线程的同步。（在1.4.0中引入。仅与Solaris相关）|
|-XX:-UseParallelGC|使用parallel garbage collection进行清理。（在1.4.1中引入）|
|-XX:-UseParallelOldGC|对full收集使用parallel garbage collection。启用此选项会自动设置-XX:+UseParallelGC。（在5.0 update 6中引入）|
|-XX:-UseSerialGC|使用serial garbage collection。（在5.0中引入）|
|-XX:-UseSpinning|在进入操作系统线程同步代码之前，在Java监视器上启用naive spin。（仅与1.4.2和5.0相关）[1.4.2，多处理器Windows平台：true]|
|-XX:+UseTLAB|使用本地线程对象分配（在1.4.0中引入，在此之前称为UseTLE）[1.4.2及更早版本，x86或-client: false]|
|-XX:+UseSplitVerifier|使用具有StackMapTable属性的新类型检查器。（在5.0中引入）[5.0:false]|
|-XX:+UseThreadPriorities|使用本机线程优先级|
|-XX:+UseVMInterruptibleIO|用于 I/O 操作的 EINTR 之前或使用的线程中断导致 OS_INTRPT。（在6中引入。仅与Solaris相关）|

#### Garbage First (G1) Garbage Collection Options
|**Option and Default Value**|**Description**|
|:-------------------|:----------|
|-XX:+UseG1GC|使用G1收集器（Garbage First (G1) Collector）|
|-XX:MaxGCPauseMillis=n|设置最大GC暂停时间的目标。这是一个软目标，JVM将尽最大努力达成它。|
|-XX:InitiatingHeapOccupancyPercent=n|启动并发GC周期的（整个）堆占用百分比。GC使用它根据整个堆的占用率触发并发GC周期，而不仅仅是其中一代（例如G1）。值0表示“执行恒定的GC循环”。默认值为 45。|
|-XX:NewRatio=n|老年/新生代大小的比率。默认值为 2。|
|-XX:SurvivorRatio=n|eden/survivor空间大小的比率。默认值为 8。|
|-XX:MaxTenuringThreshold=n|任期阈值的最大值。默认值为 15。|
|-XX:ParallelGCThreads=n|设置垃圾收集器并行阶段使用的线程数。默认值因运行JVM的平台而异。|
|-XX:ConcGCThreads=n|并发垃圾收集器将使用的线程数。默认值因运行 JVM 的平台而异。|
|-XX:G1ReservePercent=n|设置保留为虚假上限的堆数量，以减少提升失败的可能性。默认值为 10。|
|-XX:G1HeapRegionSize=n|在G1中，Java堆被细分为大小均匀的区域。这将设置各个细分的大小。此参数的默认值是根据堆大小以符合人体工程学的方式确定的。最小值为1Mb，最大值为32Mb。|

#### Performance Options
|**Option and Default Value**|**Description**|
|:-------------------|:----------|
|-XX:+AggressiveOpts|打开预计在即将发布的版本中默认的点性能编译器优化。（在5.0 update 6中引入。）|
|-XX:CompileThreshold=10000|编译前方法调用/分支的数量 [-client: 1,500]|
|-XX:LargePageSizeInBytes=4m|设置用于Java堆的大页面大小。（在1.4.0 update 1中引入）[amd64：2m]|
|-XX:MaxHeapFreeRatio=70|GC后可用堆的最大百分比以避免收缩。|
|-XX:MaxNewSize=size|新生代的最大大小（以字节为单位）。从1.4开始，MaxNewSize 被计算为NewRatio的函数。[1.3.1 Sparc: 32m; 1.3.1 x86: 2.5m]|
|-XX:MaxPermSize=64m|永久代的大小。[5.0 and newer: 64 bit VMs are scaled 30% larger; 1.4 amd64: 96m; 1.3.1 -client: 32m]|
|-XX:MinHeapFreeRatio=40|GC后可用堆的最小百分比以避免扩展。|
|-XX:NewRatio=2|老年/新生代大小的比率。[Sparc -client: 8; x86 -server: 8; x86 -client: 12.]-client: 4 (1.3) 8 (1.3.1+), x86: 12]|
|-XX:NewSize=2m|新生代的默认大小（以字节为单位）[5.0 and newer: 64 bit VMs are scaled 30% larger; x86: 1m; x86, 5.0 and older: 640k]|
|-XX:ReservedCodeCacheSize=32m|保留代码缓存大小（以字节为单位）- 最大代码缓存大小。[Solaris 64-bit, amd64, and -server x86: 2048m; in 1.5.0_06 and earlier, Solaris 64-bit and amd64: 1024m]|
|-XX:SurvivorRatio=8|eden/survivor空间大小的比例[Solaris amd64: 6; Sparc in 1.3.1: 25; other Solaris platforms in 5.0 and earlier: 32]|
|-XX:TargetSurvivorRatio=50|清除后所需的survivor空间百分比。|
|-XX:ThreadStackSize=512|线程堆栈大小（以KB为单位）。（0 表示使用默认堆栈大小）[Sparc: 512; Solaris x86: 320 (was 256 prior in 5.0 and earlier); Sparc 64 bit: 1024; Linux amd64: 1024 (was 0 in 5.0 and earlier); all others 0.]|
|-XX:+UseBiasedLocking|启用偏置锁定。有关更多详细信息，请参阅此[调整示例](https://www.oracle.com/java/technologies/java-tuning.html#section4.2.5)。（在5.0 update 6中引入。）[5.0: false]|
|-XX:+UseFastAccessorMethods|使用 Get\<Primitive\>Field 的优化版本。|
|-XX:-UseISM|使用亲密共享内存。[不接受非Solaris平台。]|
|-XX:+UseLargePages|使用大页面内存。（在5.0 update 5中引入）有关详细信息，请参阅[Java Support for Large Memory Pages](https://www.oracle.com/java/technologies/javase/largememory-pages.html)。|
|-XX:+UseMPSS|对堆使用多页大小支持w/4mb页。不要与ISM一起使用，因为这取代了对ISM的需要。（在1.4.0 update 1中引入，与Solaris 9及更新版本相关。）[1.4.1 及更早版本：false]|
|-XX:+UseStringCache|启用缓存常用分配的字符串。|
|-XX:AllocatePrefetchLines=1|使用JIT编译代码中生成的预取指令在最后一次对象分配后加载的缓存行数。如果最后分配的对象是实例，则默认值为1，如果是数组，则默认值为3。|
|-XX:AllocatePrefetchStyle=1|预取指令的生成代码样式。0 - 不生成预取指令*d*，1 - 在每次分配后执行预取指令，2 - 在执行预取指令时使用TLAB分配水印指针来门控。|
|-XX:+UseCompressedStrings|对可以表示为纯ASCII的字符串使用 byte[]。（在Java 6 Update 21 Performance Release中引入）|
|-XX:+OptimizeStringConcat|尽可能优化字符串连接操作。（在Java 6 Update 20中引入）|

#### Debugging Options
|**Option and Default Value**|**Description**|
|:-------------------|:----------|
|-XX:-CITime|打印在JIT编译器中花费的时间。（在1.4.0中引入）|
|-XX:ErrorFile=./hs_err_pid\<pid\>.log|如果发生错误，请将错误数据保存到此文件。（在6中介绍）|
|-XX:-ExtendedDTraceProbes|启用影响性能[dtrace](http://docs.oracle.com/javase/6/docs/technotes/guides/vm/dtrace.html)探头。（在6中引入。仅与Solaris相关。)|
|-XX:HeapDumpPath=./java_pid\<pid\>.hprof|堆转储的目录或文件名路径。*可管理*。（在1.4.2 update 12, 5.0 update 7引入。）|
|-XX:-HeapDumpOnOutOfMemoryError|抛出java.lang.OutOfMemoryError时，将堆转储到文件。*可管理*。（在1.4.2 update 12, 5.0 update 7中引入。）|
|-XX:OnError="\<cmd args\>;\<cmd args\>"|在发生致命错误时运行用户定义的命令。（在1.4.2 update 9中引入。）|
|-XX:OnOutOfMemoryError="\<cmd args\>; \<cmd args\>"|在首次抛出OutOfMemoryError时运行用户定义的命令。（在1.4.2 update 12, 6中引入）|
|-XX:-PrintClassHistogram|在Ctrl-Break上打印类实例的直方图。*易于管理*。（在1.4.2中引入。）[jmap -histo](http://docs.oracle.com/javase/6/docs/technotes/tools/share/jmap.html)命令提供了等效的功能。|
|-XX:-PrintConcurrentLocks|在Ctrl-Break线程转储中打印java.util.concurrent锁。*易于管理*。（在6中引入。）[jstack -l](http://docs.oracle.com/javase/6/docs/technotes/tools/share/jstack.html)命令提供了等效的功能。|
|-XX:-PrintCommandLineFlags|打印出现在命令行上的标志。（在5.0中引入。）|
|-XX:-PrintCompilation|编译方法时打印消息。|
|-XX:-PrintGC|在垃圾收集时打印消息。*易于管理*。|
|-XX:-PrintGCDetails|在垃圾收集处打印更多详细信息。*易于管理*。（在1.4.0中引入。）|
|-XX:-PrintGCTimeStamps|在垃圾收集时打印时间戳。*可管理*（在1.4.0中引入。）|
|-XX:-PrintTenuringDistribution|打印任期年龄信息。|
|-XX:-PrintAdaptiveSizePolicy|启用有关自适应生成大小的信息的打印。|
|-XX:-TraceClassLoading|跟踪类的加载。|
|-XX:-TraceClassLoadingPreorder|跟踪按引用（未加载）顺序加载的所有类。（在1.4.2中引入。）|
|-XX:-TraceClassResolution|跟踪常量池分辨率。（在1.4.2中引入。）|
|-XX:-TraceClassUnloading|跟踪类的卸载。|
|-XX:-TraceLoaderConstraints|加载器约束的跟踪记录。（在6中引入。）|
|-XX:+PerfDataSaveToFile|退出时保存jvmstat二进制数据。|
|-XX:ParallelGCThreads=n|设置新旧并行垃圾收集器中垃圾收集线程的数量。默认值因运行JVM的平台而异。|
|-XX:+UseCompressedOops|允许使用压缩指针（对象引用表示为32位偏移而不是64位指针）以优化64位性能，Java堆大小小于 32GB。|
|-XX:+AlwaysPreTouch|在JVM初始化期间预先接触Java堆。因此，堆的每一页都在初始化期间被要求归零，而不是在应用程序执行期间递增。|
|-XX:AllocatePrefetchDistance=n|设置对象分配的预取距离。即将用新对象的值写入的内存在超出最后分配的对象地址的这个距离（以字节为单位）被预取到缓存中。每个Java线程都有自己的分配点。默认值因运行JVM的平台而异。|
|-XX:InlineSmallCode=n|仅当生成的本机代码大小小于此值时，才内联先前编译的方法。默认值因运行JVM的平台而异。|
|-XX:MaxInlineSize=35|要内联的方法的最大字节码大小。|
|-XX:FreqInlineSize=n|要内联的频繁执行的方法的最大字节码大小。默认值因运行JVM的平台而异。|
|-XX:LoopUnrollLimit=n|展开循环体，服务器编译器中间表示节点计数小于此值。服务器编译器使用的限制是此值的函数，而不是实际值。默认值因运行JVM的平台而异。|
|-XX:InitialTenuringThreshold=7|设置用于并行年轻收集器中自适应GC大小调整的初始任期阈值。年老阈值是对象在被提升到老一代或老一代之前在年轻集合中存活的次数。|
|-XX:MaxTenuringThreshold=n|设置用于自适应GC大小调整的最大任期阈值。当前最大值为15。并行收集器的默认值为15，CMS 的默认值为 4。|
|-Xloggc:\<filename\>|将GC详细输出记录到指定文件。详细输出由普通详细GC标志控制。|
|-XX:-UseGCLogFileRotation|启用GC日志轮换，需要-Xloggc|
|-XX:NumberOfGClogFiles=1|设置轮转日志时使用的文件数，必须>=1。轮转的日志文件将使用以下命名方案，\<filename\>.0、\<filename\>.1、 ..., \<filename\>.n-1。|
|-XX:GCLogFileSize=8K|日志将被轮换的日志文件的大小，必须>=8K。|



**备注：翻译自https://www.oracle.com/java/technologies/javase/vmoptions-jsp.html**