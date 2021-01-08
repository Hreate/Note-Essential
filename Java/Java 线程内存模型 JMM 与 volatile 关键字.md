# Java 线程内存模型 JMM 与 volatile 关键字

### 1 CPU并发缓存架构

* Java 线程内存模型跟 cpu 缓存模型类似，是基于 cpu 缓存模型来建立的，Java 线程内存模型是标准化的，屏蔽掉了底层不同计算机的区别

  <img src="C:\Users\lenovo\Pictures\Screenshots\屏幕截图(32).png" style="zoom:33%;" />

* 总线加锁（性能太低）

  cpu从主内存读取数据到高速缓存，会在总线对这个数据加锁，这样其他cpu没法去读或写这个数据，直到这个cpu使用完数据释放锁之后其它cpu才能读取该数据

* MESI缓存一致性协议

  多个cpu从主内存读取同一个数据到各自的高速缓存，当其中某个cpu修改了缓存里的数据，该数据会马上同步回主内存，其它cpu通过总线嗅探机制可以感知到数据的变化从而将自己缓存里的数据失效

![](C:\Users\lenovo\Pictures\Screenshots\屏幕截图(36).png)



### 2 JMM数据原子操作

* read（读取）：从主内存读取数据
* load（载入）：将主内存读取到的数据写入工作内存
* use（使用）：从工作内存读取数据来计算
* assign（赋值）：将计算好的值重新赋值到工作内存中
* store（存储）：将工作内存数据写入主内存
* write（写入）：将 store 过去的变量值赋值给主内存中的变量
* lock（锁定）：将主内存变量加锁，标识为线程独占状态
* unlock（解锁）：将主内存变量解锁，解锁后其他线程可以锁定该变量



### 3 volatile 缓存可见性实现原理

* 底层实现主要是通过汇编 lock 前缀指令，它会锁定这块内存区域的缓存（缓存行锁定）并回写到主内存
* IA-32 架构软件开发者手册对 lock 指令的解释：
  1. 会将当前处理器缓存航的数据立即写回到系统内存
  2. 这个写回内存的操作会引起在其它CPU里缓存了该内存地址的数据无效（MESI 协议）



### 4 Java 程序汇编代码查看

```shell
-server -Xcomp -XX:+UnlockDiagnosticVMOptions -XX:+PrintAssembly -XX:CompileCommand=compileonly,*VolatileVisibilityTest.prepareData
```



### 5 并发编程三大特性

* 可见性
* 原子性
* 有序性
* `volatile` 保证可见性与有序性，但是不保证原子性，保证原子性需要借助 `synchronized` 这样的锁机制

