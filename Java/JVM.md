# JVM

## 1 Garbage Collection

### 1.1 识别垃圾

* #### 引用计数（reference count）

  ##### 问题：

  * 不能解决循环引用

    

* #### JVM中使用

  ##### 根可达算法（Root Searching）

  * 这个算法的基本思想就是通过一系列的称为 “GC Roots” 的对象作为起点，从这些结点开始向下搜索，结点所走过的路径称为引用链，当一个对象到 GC Roots 没有任何引用链相连的话，则证明此对象是不可用的。
* GC Roots 根结点：类加载器、Thread、虚拟机栈的本地变量表、static成员、常量引用、本地方法栈的变量等等
  
  * 解释：
  
    从 java 程序主入口（main 方法）中声明的各种实例开始向下查找引用的其他实例，能找到的就是有用的实例，找不到的就是垃圾。

### 1.2清除回收算法

* #### Mark-Sweep（标记清除）

  * 原理：找到垃圾后，直接清除并修改为可用状态
  * 优点：算法简单，速度较快
  * 缺点：导致内存碎片化

* #### Copying（拷贝）

  * 原理：把内存一分为二，永远只用一半。回收时，将有用的内存数据拷贝到另一半内存上，连续放置
  * 优点：速度极快，算法简单
  * 缺点：浪费内存空间

* #### Mark-Compact（标记压缩）

  * 原理：以内存空间连续为目的，将要回收的内存空间以有用的内存数据进行填充，闲置的内存空间也进行填充
  * 优点：保证了内存空间的连续
  * 缺点：效率较低



## 2 Garbage Collectors（10种）

![](C:\Users\lenovo\Pictures\Screenshots\屏幕截图(9).png)

* 分代模型中，上边三种垃圾收集器在新生代中使用，下边三种在老年代中使用，虚线连接的是可以相互搭配的垃圾收集器

* 垃圾收集器的发展路线，是随着内存越来越大的过程而演进

  从分代算法演化到不分代算法

  Serial 算法 几十M

  Parallel 算法 几个G

  CMS 几十个G 开始并发回收

* JDK 诞生 Serial 追随提高效率，诞生了 PS，为了配合 CMS，诞生了 PN。CMS 是1.4版本后期引入，CMS 是里程碑式的 GC，它开启了并发回收的过程，但是 CMS 毛病较多，因此目前没有任何一个 JDK 版本默认是 CMS 并发垃圾回收是因为无法忍受 STW

* Serial 新生代 串行回收

  <img src="C:\Users\lenovo\Pictures\Screenshots\屏幕截图(10).png" style="zoom: 33%;" />

  * 垃圾收集器以单线程启动，进行GC时，会停止所有正在工作的线程（stop-the-world 简称 STW）

* PS 新生代 并行回收（多线程启动，会 STW）

  <img src="C:\Users\lenovo\Pictures\Screenshots\屏幕截图(12).png" style="zoom:33%;" />

* ParNew 新生代 配合 CMS 的并行回收

  <img src="C:\Users\lenovo\Pictures\Screenshots\屏幕截图(14).png" style="zoom:33%;" />

* Serial Old（单线程启动，会 STW）

  <img src="C:\Users\lenovo\Pictures\Screenshots\屏幕截图(11).png" style="zoom:33%;" />

* Parallel Old（多线程启动，会 STW）

  <img src="C:\Users\lenovo\Pictures\Screenshots\屏幕截图(13).png" style="zoom:33%;" />

* CMS（ConcurrentMarkSweep） 老年代 并发的，垃圾回收和应用程序同时运行，降低 STW 的时间（200ms）

  CMS 问题比较多，所以现在没有一个版本默认是 CMS，只能手工指定

  CMS 既然是 MarkSweep 就一定会有碎片化的问题，碎片到达一定程度，CMS 的老年代分配对象分配不下的时候，使用 SerialOld 进行老年代回收

  想象一下：

  PS + PO -> 加内存 换垃圾收集器 -> PN + CMS + Serial Old （几个小时 - 几天的 STW）

  几十个 G 的内存，单线程回收 -> G1 + FGC 几十个G -> 上 T 内存的服务器 ZGC

  算法：三色标记 + Incremental Update

  <img src="C:\Users\lenovo\Pictures\Screenshots\屏幕截图(15).png" style="zoom:33%;" />

  <img src="C:\Users\lenovo\Pictures\Screenshots\屏幕截图(16).png" style="zoom:33%;" />

  * 四个主要阶段
    * 初始标记
    
    * 并发标记
    
      * 三色标记法
    
        ![](C:\Users\lenovo\Pictures\Screenshots\屏幕截图(17).png)
    
        ![](C:\Users\lenovo\Pictures\Screenshots\屏幕截图(18).png)
    
        ![](C:\Users\lenovo\Pictures\Screenshots\屏幕截图(23).png)
    
      * 产生漏标：
    
        1. 标记进行时增加了一个黑到白的引用，如果不重新对黑色进行处理，则会漏标
        2. 标记进行时删除了灰对象到白对象的引用，那么这个白对象有可能被loubiao
    
    * 重新标记（最终标记）
    
      所以，CMS 的 remark 阶段必须从头扫描一遍
    
    * 并发清理

* G1 逻辑分代，物理不分

  算法：三色标记 + SATB

  <img src="C:\Users\lenovo\Pictures\Screenshots\屏幕截图(20).png" style="zoom:33%;" />

  <img src="C:\Users\lenovo\Pictures\Screenshots\屏幕截图(21).png" style="zoom:33%;" />

  <img src="C:\Users\lenovo\Pictures\Screenshots\屏幕截图(25).png" style="zoom:33%;" />

* ZGC

  算法：ColoredPointers + LoadBarrier

  <img src="C:\Users\lenovo\Pictures\Screenshots\屏幕截图(26).png" style="zoom:33%;" />

* Shenandoah

  算法：ColoredPointers + WriteBarrier

* Eplison

* 垃圾收集器跟内存大小的关系

  * Serial 几十M
  * PS 上百M - 几个G
  * CMS 20G
  * G1 上百G
  * ZGC 4T - 16T

* JDK 1.8 默认 Parallel Scavenge 和 Parallel Old

* JDK 1.8 推荐使用 G1



### 2.1 分代模型

![](C:\Users\lenovo\Pictures\Screenshots\屏幕截图(5).png)

* 原理：
  * 将一块内存空间分为两部分—— new / young（新生代）和 old（老年代），默认占用空间比为 `1:2` ，也可通过参数进行自定义
  
  * 新生代与老年代采用了不同的垃圾回收算法
  
  * 因为新 `new` 出来的对象很大概率会很快变成垃圾，所以新生代采用了 `Copying` 算法。将新生代也分为两部分共三个区域—— 1个 eden 和 2个 survivor，比例默认为 `8:1:1`。新 `new` 出的对象全存储在 eden 区，其中大部分会很快死去，有用的数据就拷贝到 第一个 survivor 区；第二次垃圾回收，eden 区和第一个 survivor区有用的数据会拷贝到第二个 survivor 区，然后将 eden 区和 第一个 survivor 区进行回收；再下一次，重复之前的过程，eden 区和第二个 survivor 区有用的数据拷贝到第一个 survivor 区后进行回收
  
  * 新生代中存储的是新 `new` 出来的对象。回收算法进行回收时，有用的内存不会被回收，但是年龄会加1，年龄超过设置的值时就会被移入老年代
  
  * 总结：
  
    ![](C:\Users\lenovo\Pictures\Screenshots\屏幕截图(8).png)



### 

### 2.2 GC Tuning 实战

#### 2.2.1 常见垃圾收集器组合参数设定（1.8）

* `-XX:+UseSerialGC` = Serial New(DefNew) + Serial Old
  * 小型程序。默认情况下不会是这种选项，HotSpot 会根据计算及配置和 JDK 版本自动选择垃圾收集器
* `-XX:+UseParNewGC` = ParNew + Serial Old
  * 这个组合已经很少用（在某些版本中已经废弃）
* `-XX:+UseConcMarkSweepGC` = ParNew + CMS + Serial Old
* `-XX:+UseParallelGC` = Parallel Scavenge + Parallel Old（1.8默认）【PS + Serial Old】
* `-XX:+UseParalleOldGC` = Parallel Scavenge + Parallel Old
* `-XX:+UseG1GC` = G1
* 默认 GC 的查看
  * `java +XX:+PrintCommandLineFlags -version`
  * 通过 GC 的日志来分辨



#### 2.2.2 GC常用参数

* `-Xmn` `-Xms` `-Xmx` `-Xss`

  新生代 最小堆 最大堆 栈空间

* `-XX:+UseTLAB`

  使用 TLAB，默认打开

* `-XX:+PrintTLAB`

  打印 TLAB 的使用情况

* `-XX:+TLABSize`

  设置 TLAB 大小

* `-XX:+DisableExplictGC`

  System.gc()不管用，FGC

* `-XX:+PrintGC`

* `-XX:+PrintGCDetails`

* `-XX:+PrintHeapAtGC`

* `-XX:+PrintGCTimeStamps`

* `-XX:+PrintGCApplicationConcurrentTime`（低）

  打印应用程序时间

* `-XX:+PrintGCApplicationStoppedTime`（低）

  打印暂停时长

* `-XX:+PrintReferenceGC`（重要性低）

  记录回收了多少种不同引用类型的引用
  
* `-verbose:class`

  类加载详细过程

* `-XX:+PrintVMOptions`

* `-XX:+PrintFlagsFinal` `-XX:+PrintFlagsInitial`（必须会用）



#### 2.2.3 Parallel 常用参数

* `-XX:SurvivorRatio`

* `-XX:PretenureSizeThreshold`

  大对象到底多大

* `-XX:MaxTenuringThreshold`

* `-XX:+ParallelGCThreads`

  并行收集器的线程数，同样适用于 CMS，一般设为和 CPU 核数相同

* `-XX:+UseAdaptiveSizePolicy`

  自动选择各区大小比例



#### 2.2.4 CMS 常用参数

* `-XX:+UseConcMarkSweepGC`

* `-XX:ParallelCMSThreads`

  CMS 线程数量

* `-XX:CMSInitiatingOccupancyFraction`

  使用多少比例的老年代后开始 CMS 回收，默认是68%（近似值），如果频繁发生 Serial Old 卡顿，应该调小（频繁 CMS 回收）

* `-XX:UseCMSCompactAtFullCollection`

  在 FGC 时进行压缩
  
* `-XX:CMSFullGCsBeforeCompaction`

  多少次 FGC 之后进行压缩

* `-XX:+CMSClassUnloadingEnabled`

* `-XX:CMSInitiationgPermOccupancyFraction`

  达到什么比例时进行 Perm 回收

* `GCTimeRatio`

  设置GC时间占用程序运行时间的百分比

* `-XX:MacGCPauseMillis`

  停顿时间，是一个建议时间，GC会尝试用各种手段达到这个时间，比如减小新生代



#### 2.2.5 G1 常用参数

* `-XX:+UseG1GC`

* `-XX:MaxGCPauseMillis`

  建议值，G1会尝试调整 Young 区的块数来达到这个值

* `-XX:GCPauseIntervalMillis`

  GC的间隔时间

* `-XX:+G1HeapRegionSize`

  分区大小，建议逐渐增大该值，1 2 4 8 16 32。

  随着 size 增大，垃圾的存活时间更长，GC间隔更长，但每次GC的时间也会更长

  ZGC做了改进（动态区块大小）

* `G1NewSizePercent`

  新生代最小比例，默认为5%

* `G1MaxNewSizePercent`

  新生代最大比例，默认为60%

* `GCTimeRatio`

  GC时间建议比例，G1会根据这个值调整堆空间

* `ConcGCThreads`

  线程数量

* `InitiatingHeapOccupancyPercent`

  启动G1的堆空间占用比例



#### 2.2.6 调优前的基础概念

1. 吞吐量：用户代码时间/(用户代码执行时间 + 垃圾回收时间)
2. 响应时间：STW越短，响应时间越好

所谓调优，首先确定，追求啥？吞吐量优先，还是响应时间优先？还是在满足一定的响应时间的情况下，要求达到多大的吞吐量

问题：

科学计算，吞吐量，数据挖掘，thrput。吞吐量优先的一般：PS + PO

响应时间：网站 GUI API （1.8 G1）



#### 2.2.7 什么是调优

1. 根据需求进行 JVM 规划和预调优
2. 优化运行 JVM 运行环境（慢、卡顿）
3. 解决 JVM 运行过程中出现的各种问题（OOM - Out Of Memory）



#### 2.2.8 调优，从规划开始

* 调优，从业务场景开始，没有业务场景的调优都是耍流氓

* 无监控（压力测试，能看到结果），不调优

* 步骤：

  1. 熟悉业务场景（没有最好的垃圾收集器，只有最合适的垃圾收集器）
     1. 响应时间、停顿时间 [CMS G1 ZGC]（需要给用户作响应）
     2. 吞吐量 = 用户时间/(用户时间 + GC时间) [PS]
  2. 选择收集器组合
  3. 计算内存需求（经验值 1.5G 16G）
  4. 选定 CPU （越高越好）
  5. 设定年代大小、升级年龄
  6. 设定日志参数
     1. `-Xloggc:/opt/xxx/log/xxx-xxx-gc-%t.log -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=5 -XX:GCLogFileSize=20M -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCCause`
     2. 或者每天产生一个日志
  7. 观察日志情况

* 案例1：垂直电商，最高每日百万订单，处理订单系统需要什么样的服务器配置

  > 这个问题比较业余，因为很多不同的服务器配置都能支撑（1.5G 16G）
  >
  > 1小时360000集中时间段，100个订单/秒，（找一小时内的高峰期，1000订单/秒）
  >
  > 经验值，
  >
  > 非要计算：一个订单产生需要多少内存？512K * 1000 500M内存
  >
  > 专业点儿问法：要求响应时间100ms
  >
  > 压测！

* 案例2：12306遭遇春节大规模抢票应该如何支撑？

  > 12306应该是中国并发量最大的秒杀网站：
  >
  > 号称并发量100w最高
  >
  > CDN -> LVS -> NGINX -> 业务系统 -> 每台机器1W并发（10K问题）100台机器
  >
  > 普通电商订单 -> 下单 -> 订单系统（IO）减库存 -> 等待用户付款
  >
  > 12306的一种可能的模型：下单 -> 减库存和订单（redis kafka）同时异步进行 -> 等付款
  >
  > 减库存最后还会把压力压倒一台服务器
  >
  > 可以做分布式本地库存 + 单独服务器做库存均衡
  >
  > 大流量的处理方法：分而治之

* 怎么得到一个事务会消耗多少内存？

  > 1.弄台机器，看能承受多少 TPS？是不是达到目标？扩容或调优，让它达到
  >
  > 2.用压测来确定



#### 2.2.9 优化环境

1. 有一个50万PV的资料类网站（从磁盘提取文档到内存）原服务器32位，1.5G的堆，用户反馈网站比较缓慢，因此公司决定升级，新的服务器位64位，16G的堆内存，结果用户反馈卡顿十分严重，反而比以前效率更低了

   1. 为什么原网站慢？

      很多用户浏览数据，很多数据 load 到内存，内存不足，频繁GC，STW 长，响应时间变长

   2. 为什么会更卡顿？

      内存越大，FGC时间越长

   3. 咋办？

      PS -> PN + CMS 或者 G1

2. 系统 CPU 经常100%，如何调优？（面试高频）

   CPU 100%那么一定有线程在占用系统资源

   1. 找出哪个进程 cpu 高（`top`）
   2. 该进程中的哪个线程 cpu 高（`top -Hp`）
   3. 导出该线程的堆栈（`jstack`）
   4. 查找哪个方法（栈帧）消耗时间（`jstack`）
   5. 工作线程占比高|垃圾回收线程占比高

3. 系统内存飙高，如何查找问题？（面试高频）

   1. 导出堆内存（`jmap`）
   2. 分析（jhat jvisualvm mat jprofiler ...）

4. 如何监控 JVM

   1. jstat jvisualvm jprofiler arthas top ...



#### 2.9.10 解决 JVM 运行中的问题

* 一个案例理解常用工具

  1. 测试代码

     ```java
     package com.example.demo01.jvm;
     
     import java.math.BigDecimal;
     import java.util.ArrayList;
     import java.util.Date;
     import java.util.List;
     import java.util.concurrent.ScheduledThreadPoolExecutor;
     import java.util.concurrent.ThreadPoolExecutor;
     import java.util.concurrent.TimeUnit;
     
     /**
      * 从数据库中读取信用数据，套用模型，并把结果进行记录和传输
      */
     public class T15_FullGC_Problem01 {
         private static class CardInfo {
             BigDecimal price = new BigDecimal("0.0");
             String name = "张三";
             int age = 5;
             Date birthdate = new Date();
     
             public void m() {}
         }
     
         private static ScheduledThreadPoolExecutor executor = new ScheduledThreadPoolExecutor(50, new ThreadPoolExecutor.DiscardOldestPolicy());
     
         public static void main(String[] args) throws InterruptedException {
             executor.setMaximumPoolSize(50);
     
             while (true){
                 modelFit();
                 Thread.sleep(100);
             }
         }
     
         private static void modelFit() {
             List<CardInfo> taskList = getAllCardInfo();
             taskList.forEach(info -> {
                 // do something
                 // do sth with info
                 executor.scheduleWithFixedDelay(info::m, 2, 3, TimeUnit.SECONDS);
             });
         }
     
         private static List<CardInfo> getAllCardInfo() {
             List<CardInfo> taskList = new ArrayList<>();
     
             for (int i = 0; i < 100; i++) {
                 CardInfo ci = new CardInfo();
                 taskList.add(ci);
             }
             return taskList;
         }
     }
     ```

  2. `java -Xms200M -Xmx200M -XX:+PrintGC com.example.demo01.jvm.T15_FullGC_Problem01`

  3. 一般是运维团队首先收到报警信息（CPU Memory）

  4. `top` 命令观察到问题：内存不断增长，CPU 占用率居高不下

  5. `top -Hp` 观察进程中的线程，哪个线程 CPU 和内存占比高

  6. `jps` 定位具体 java 进程

     `jstack` 定位线程状况，重点关注：WAITING BLOCKED

     eg.

     `waiting on <0x0000000088ca3310>`（a java.lang.Object）

     假如一个进程中有100个线程，很多线程都在 waiting on <xx>，一定要找到是哪个线程持有这把锁

     怎么找？搜索 jstack dump 的信息，找 <xx>，看哪个线程持有这把锁 RUNNABLe

     作业：1、写一个死锁程序，用 jstack 观察 2、写一个程序，一个线程持有锁不释放，其他线程等待

  7. 为什么阿里规范里规定，线程的名称（尤其是线程池）都要写有意义的名称

     怎么样自定义线程池里的线程名称？（自定义 ThreadFactory）

  8. jinfo pid

  9. `jstat -gc` 动态观察GC情况 / 阅读GC日志发现频繁GC / arthas 观察 / jconsole / jvisualVM / Jprofiler（最好用）

     `jstat -gc 4655 500` ：每个500个毫秒打印GC的情况

     如果面试官问你是怎么定位 OOM 问题的？如果你回答用图形界面（错误）
     
     1. 已经上线的系统不用图形界面用什么？（cmdline arthas）
     2. 图形界面到底用在什么地方？测试！测试的时候进行监控！（压测观察）
     
  10. `jmap -histo 4655 | head -20`，查找有多少对象产生
  
  11. `jamp -dump:format=b,file=xxx pid`
  
      线上系统，内存特别大，`jmap` 执行期间会对进程产生很大影响，甚至卡顿（电商不适合）
  
      1. 设定了参数 HeapDump，OOM 的时候会自动产生堆转储文件（不是很专业，因为都有监控，内存增长就会报警）
      2. 很多服务器备份（高可用），停掉这台服务器对其他服务器不影响
      3. 在线定位（一般小点儿公司用不到）
      4. 在测试环境中压测（产生类似内存增长问题，在堆还不是很大的时候进行转储）
  
  12. `java -Xms20M -Xmx20M -XX:+UseParallelGC -XX:+HeapDumpOnOutOfMemoryError com.mashibing.jvm.gc.T15_FullGC_Problem01`
  
  13. 使用 `MAT`/`jhat`/`jvisualvm` 进行 dump 文件分析
  
      https://www.cnblogs.com/baihuitestsoftware/articles/6406271.html
  
      `jhat -J-mx512M xxx.dump`
  
      可以使用 OQL 查找特定问题对象
  
  14. `top` 命令是 Linux 下常用的性能分析工具，能够实时显示系统中各个进程的资源占用状况，类似于 Windows 的任务管理器
  
  15. `jps`（Java Virtual Machine Process Status Tool），是 java 提供的一个显示当前所有 java 进程 pid 的命令
  
  16. `jinfo` 是 JDK 自带的命令，可以用来查看正在运行的 java 应用程序的扩展参数，包括 Java System 属性和 JVM 命令行参数；也可以动态地修改正在运行的 JVM 一些参数。
  
  17. `jstat` 命令可以查看堆内存各部分的使用量，以及加载类的数量
  
  18. `jstack` 可以用来查看某个 java 进程内的线程堆栈信息
  
  19. `jmap` 命令是一个可以输出所有内存中对象的工具，甚至可以将 VM 中的 heap，以二进制输出成文本。打印出某个 java 进程（使用pid）内存内的，所有“对象”的情况（如：产生哪些对象，及其数量）

#### 2.9.11 arthas 在线排查工具

* 为什么需要在线排查？

  在生产上我们经常会碰到一些不好排查的问题，例如线程安全问题，用最简单的 threaddump 或者 headdump 不好查到问题原因。为了排查这些问题，有时我们会临时加一些日志，比如在一些关键的函数里打印出参数，然后重新打包发布，如果打了日志还是没找到问题，继续加日志，重新打包发布。对于上线流程复杂而且审核比较严的公司，从改代码到上线需要层层流转，会大大影响问题排查的进度。

* `jvm` 观察 jvm 信息

* `thread` 定位线程问题

* `dashboard` 观察系统情况

* `jad` 反编译

  动态代理生成类的问题定位

  第三方的类（观察代码）

  版本问题（确定自己最新提交的版本是不是被使用）

* `redefine` 热替换

  目前有些限制条件：只能改方法实现（方法已经运行完成），不能改方法名，不能改属性

* `sc` - search class

* `watch` - watch method

* 没有包含的功能：`jmap`



### 3 JVM 结构

<img src="C:\Users\lenovo\Pictures\Screenshots\屏幕截图(28).png"  />



### 4 实战调优

#### 4.1 JVM 调优主要就是调整下面两个指标

* 停顿时间：垃圾收集器做垃圾回收中断应用执行的时间。`-XX:MaxGCPauseMilles`
* 吞吐量：垃圾回收的时间和总时间的占比：1/(1+n)，吞吐量为 1-1/(1+n)。`-XX:GCTimeRatio=n`



#### 4.2 GC 调优步骤

* 打印 GC 日志

  `-XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps -Xloggc:./gc.log`

  Tomcat 则直接加在 `JAVA_OPTS` 变量里

* 分析日志得到关键性指标

* 分析 GC 原因，调优 JVM 参数



#### 4.2.1 Parallel Scavenge 收集器（默认）

* 分析 parallel-gc.log

* 调优

  第一次调优，设置 Metaspace 大小：增大元空间大小 `-XX:MetaspaceSize=64M -XX:MaxMetaspaceSize=64M`

  第二次调优，增大新生代动态扩展增量，默认是20(%)，可以减少 young gc：`-XX:YoungGenerationSizeincrement=30`

* 配置 CMS 收集器

  `-XX:+UseConcMarkSweepGC`

  分析 cms-gc.log

  

* 配置 G1 收集器

  `-XX:+UseG1GC`

  分析 g1-gc.log

  young GC：`[GC pause (G1 Evacuation Pause) (young)]`

  initial-mark：`[GC pause (Metadata GC Threshold) (young) (initial-mark) (参数：InitiatingHeapOccupancyPercent)]`

  mixed GC：`[GC pause (G1 Evacution Pause) (mixed)]`

  full GC：`[Full GC (Allocation Failure)]`


  （G1内部，前面提到的混合 GC 是非常重要的释放内存机制，它避免了 G1 出现 Region 没有可用的情况，否则就会触发 Full GC 事件。CMS、Parallel、Serial GC 都需要通过 Full GC去压缩老年代并在这个过程中扫描整个老年代。G1 的 Full GC 算法和 Serial GC 收集器完全一致。当一个 Full GC 发生时，整个 Java 堆执行一个完整的压缩，这样确保了最大的空余内存可用。G1 的 Full GC 是一个单线程，它可能引起一个长时间的停顿时间，G1 的设计目标是减少 Full GC，满足应用性能目标）


  查看发生 MixedGC 的阈值：`jinfo -flag InitiatingHeapOccupancyPercent 进程id`

* 调优

  第一次调优，设置 Metaspace 大小：增大元空间大小 `-XX:MetaspaceSize=64M -XX:MaxMetaspaceSize=64M`

  第二次调优，添加吞吐量和停顿时间参数：`-XX:GCTimeRatio=80 -XX:MaxGCPauMillis=100`


  分析工具：gceasy、GCViewer



#### 4.2.2 G1调优相关

1. 常用参数
   * `-XX:+UseG1GC` 开启 G1
   * `-XX:G1HeapRegionSize=n` region 的大小，1 - 32M，2048个
   * `-XX:MaxGCPauseMillis=200` 最大停顿时间
   * `-XX:G1NewSizePercent -XX:G1MaxNewSizePercent`
   * `-XX:ParallelGCThreads=n` SWT 线程数（停止应用程序）
   * `-XX:ConcGCThreads=n` 并发线程数=1/4*并行
2. 最佳实践
   * 新生代大小，避免使用 `-Xmn`、`-XX:NewRatio` 等显示设置 Young 区大小，会覆盖暂停时间目标（常用参数3）
   * 暂停时间目标：暂停时间不要太严苛，其吞吐量目标是90%的应用程序时间和10%的垃圾回收时间，太严苛会直接影响到吞吐量
3. 是否需要切换到 G1
   * 50% 以上的堆被存活对象占用
   * 对象分配和晋升的速度变化非常大
   * 垃圾回收时间特别长，超过1秒
4. G1 调优目标
   * 6GB以上内存
   * 停顿时间是500ms以内
   * 吞吐量是90%以上



### 4.3 GC常用参数

#### 4.3.1 堆栈设置

* `-Xss`：每个线程的栈的大小
* `-Xms`：初始堆大小，默认物理内存的1/64
* `-Xmx`：最大堆大小，默认物理内存的1/4
* `-Xmn`：新生代大小
* `-XX:NewSize`：设置新生代初始大小
* `-XX:NewRatio`：默认2标识新生代占老年代的1/2，占整个堆内存的1/3
* `-XX:SurvivorRatio`：默认8标识一个 survivor 区占用1/8的 Eden 内存，即1/10的新生代内存
* `-XX:Metaspace`：设置元空间大小
* `-XX:MaxMetaspaceSize`：设置元空间最大允许大小，默认不受限制，JVM Metaspace 会进行动态扩展



#### 4.3.2 垃圾回收统计信息

* `-XX:+PrintGC`
* `-XX:+PrintGCDetails`
* `-XX:+PrintGCTimeStamps`
* `-Xloggc:filename`



### 4.4 收集器设置

* `-XX:+UseSerialGC`：设置串行收集器
* `-XX:+useParallelGC`：设置并行收集器
* `-XX:+UseParallelOldGC`：老年代使用并行回收收集器
* `-XX:+UserParNewGC`：在新生代使用并行收集器
* `-XX:+UseConcMarkSweepGC`：设置 CMS 并发收集器
* `-XX:+UseG1GC`：设置 G1 收集器
* `-XX:ParallelGCThreads`：设置用于垃圾回收的线程数



### 4.5 并行收集器设置

* `-XX:ParallelGCThreads`：设置并行收集器的线程数
* `-XX:MaxGCPauseMillis`：设置并行手机最大暂停时间
* `-XX:GCTimeRatio`：设置垃圾回收时间占程序运行时间的百分比，公式为 1/(1+n)
* `-XX:YoungGenerationSizeincrement`：新生代gc后扩容比例，默认是20(%)



### 4.6 CMS 收集器设置

* `-XX:+UseConcMarkSweepGC`：设置 CMS 并发收集器
* `-XX:+CMSincrementalMode`：设置为增量模式。适用于单CPU情况。
* `-XX:ParallelGCThreads`：设置并行收集器的线程数
* `-XX:CMSFullGCsBeforeCompaction`：设定进行多少次 CMS 垃圾回收后，进行一次内存压缩
* `-XX:+CMSClassUnloadingEnabled`：允许对类元数据进行回收
* `-XX:UseCMSInitiatingOccupancyOnly`：表示只在达到阈值的时候，才进行CMS回收
* `-XX:CMSInitiatingOccupancyFraction`：设置 CMS 收集器在老年代空间被使用多少后触发
* `-XX:+UseCMSCompactAtFullColleciton`：设置 CMS 收集器在完成垃圾收集后是否要进行一次内存碎片整理



### 4.7 G1 收集器设置

* `-XX:+UseG1GC`：使用 G1 收集器
* `-XX:ParallelGCThreads`：指定GC工作的线程数量
* `-XX:G1HeapRegionSize`：指定分区大小（1MB~32MB，且必须是2的幂），默认将整堆划分为2048个分区
* `-XX:GCTimeRatio`：吞吐量大小，0-100的整数（默认9），值为n则系统将花费不超过 1/(1+n) 的时间用于垃圾收集
* `-XX:MaxGCPauseMillis`：目标暂停时间（默认200ms）
* `-XX:G1NewSizePercent`：新生代内存初始空间（默认整堆5%）
* `-XX:G1MaxNewSizePercent`：新生代内存最大空间
* `-XX:TargetSurvivorRatio`：Survivor 填充容量（默认50%）
* `-XX:MaxTenuringThreashold`：最大任期阈值（默认15）
* `-XX:InitiatingHeapOccupancyPercen`：老年代占用空间超过整堆比IHOP阈值（默认45%），超过则执行混合收集
* `-XX:G1HeapWastePercent`：堆废物百分比（默认5%）