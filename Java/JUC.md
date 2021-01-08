# JUC

## 什么是 JUC

* java.util.concurrent

  java.util.concurrent.atomic

  java.util.concurrent.locks



## 进程和线程

* 进程：是计算机中的程序关于某数据集合上的一次运行活动，是系统进行资源分配和调度的基本单位，是操作系统结构的基础
* 线程：是操作系统能够进行运算调度的最小单位。它被包含在进程之中，是进程中的实际运作单位。一条线程指的是进程中一个单一顺序的控制流，一个进程中可以并发多个线程，每条线程并行执行不同的任务
* Java 程序中默认有两个线程：main、gc
* Java 调用 native 本地方法开启的线程



> 并发

在同一个时间段内，两个或多个程序执行，有时间上的重叠（宏观上是同时，微观上仍是顺序执行）



> 并行

在操作系统中是指，一组程序按独立异步的速度执行，无论从微观还是宏观，程序都是一起执行的



```java
package com.example.demo01;

public class Test1 {
    public static void main(String[] args) {
        // 获取cpu线程数
        // CPU密集型，IO密集型
        System.out.println(Runtime.getRuntime().availableProcessors());
    }
}
```

> 并发编程的本质

充分利用 CPU 的资源



> 线程状态

```java
public enum State {
        // 新生
        NEW,

        // 就绪
        RUNNABLE,

        // 阻塞
        BLOCKED,

        // 等待
        WAITING,

        // 计时等待
        TIMED_WAITING,

        // 终止
        TERMINATED;
    }
```



> wait、sleep区别

|                      | await/wait       | Sleep      | Yield          |
| -------------------- | ---------------- | ---------- | -------------- |
| **是否释放持有的锁** | 释放             | 不释放     | 不释放         |
| **调用后何时恢复**   | 唤醒后进入就绪态 | 指定时间后 | 立刻进入就绪态 |
| **谁的方法**         | Condition/Object | Thread     | Thread         |
| **执行环境**         | 同步代码块       | 任意位置   | 任意位置       |



## Lock 锁

> sychronized

同步方法和同步代码块



> Lock

实现类：ReentrantLock、ReentrantReadWriteLock.ReadLock、ReentrantReadWriteLock.WriteLock



> 公平锁

加锁前先查看是否有排队等待的线程，有的话优先处理排在前面的线程，先来先得



> 非公平锁

线程加锁时直接尝试获取锁，获取不到就自动到队尾等待



> synchronized 和 Lock 区别

* synchronized 是 java 关键字，Lock 是一个 java 类
* synchronized 无法判断锁的状态，Lock 可以判断锁的状态
* synchronized 自动释放锁，Lock 必须手动释放锁
* synchronized 可重入锁，不可以中断，非公平；Lock 可重入锁，自定义是否公平
* synchronized 适合少量的代码同步，Lock 可以使用大量的代码同步



## 虚假唤醒

* 防止虚假唤醒：线程等待语句使用 while 进行条件判断，即等待总是应该出现在循环中



## Condition

> 精准通知和唤醒线程

官方示例

```java
class BoundedBuffer<E> {
   final Lock lock = new ReentrantLock();
   final Condition notFull  = lock.newCondition(); 
   final Condition notEmpty = lock.newCondition(); 

   final Object[] items = new Object[100];
   int putptr, takeptr, count;

   public void put(E x) throws InterruptedException {
     lock.lock();
     try {
       while (count == items.length)
         notFull.await();
       items[putptr] = x;
       if (++putptr == items.length) putptr = 0;
       ++count;
       notEmpty.signal();
     } finally {
       lock.unlock();
     }
   }

   public E take() throws InterruptedException {
     lock.lock();
     try {
       while (count == 0)
         notEmpty.await();
       E x = (E) items[takeptr];
       if (++takeptr == items.length) takeptr = 0;
       --count;
       notFull.signal();
       return x;
     } finally {
       lock.unlock();
     }
   }
 }
```



## 锁的对象

> 同步方法

锁的对象是this——方法的调用者



> 静态同步方法

锁的对象是class（类）



## CopyOnWriteArrayList

* 写入时复制；COW，计算机程序设计领域的一种优化策略
* 读写分离，大大提高了读操作的性能，因此很适合读多写少的应用场景
* 缺陷：
  * 内存占用：在写操作时需要复制一个新的数组，使得内存占用为原来的两倍左右
  * 数据不一致：读操作不能读取实时性的数据，因为部分写操作的数据还未同步到读数组中
  * 所以不适合内存敏感以及对实时性要求很高的场景



## CopyOnWriteArraySet

同上



> HashSet 底层是 HashMap

```java
public HashSet {
    map = new HashMap<>();
}

// 本质就是 map 的 key 不能重复
public boolean add(E e) {
    return map.put(e, PRESENT) == null;
}

private static final Object PRESENT = new Object();
```



## ConcurrentHashMap



## Callable

> 特性

* 可以有返回值
* 可以抛出异常
* 方法为 call()

> 使用方式一

* `public FutureTask(Callable<V> callable)`：使用 Runnable 的实现类 FutureTask （异步任务）构建
* 再将 FutureTask 传入 Thread 调用 start.()
* `futureTask.get()` 获取返回值；会阻塞当前线程，直至异步任务执行结束
* FutureTask 只能被执行一次



## 常用的辅助类

> CountDownLatch

* 减法计数器
* 作用相当于 Golang 的等待组
* `countDownLatch.countDown();` 计数器-1
* `countDownLatch.await();` 阻塞当前线程，等待计数器归零




>CyclicBarrier

* 加法计数器

  ```java
  package com.example.cyclicbarrier;
  
  import java.util.concurrent.BrokenBarrierException;
  import java.util.concurrent.CyclicBarrier;
  
  public class TestCyclicBarrier {
      public static void main(String[] args) {
          CyclicBarrier cyclicBarrier = new CyclicBarrier(7, () -> {
              System.out.println("神龙出现");
          });
          for (int i = 1; i <= 7; i++) {
              int temp = i;
              new Thread(() -> {
                  System.out.println(Thread.currentThread().getName() + "收集到第" + temp + "个龙珠");
                  try {
                      cyclicBarrier.await();
                  } catch (InterruptedException | BrokenBarrierException e) {
                      e.printStackTrace();
                  }
              }).start();
          }
      }
  }
  ```

  

> Semaphore

* 信号量
* `semaphore.acquire();` 获得，阻塞直到成功获得
* `semaphore.release();` 释放，会将当前的信号量释放+1，然后唤醒等待的线程
* 多个共享资源互斥使用
* 并发限流，控制最大的线程数



## 读写锁 ReadWriteLock

```java
private final ReentrantReadWriteLock reentrantReadWriteLock = new ReentrantReadWriteLock();
```

* 写锁（独占锁）：`reentrantReadWriteLock.writeLock()`

* 读锁（共享锁）：`reentrantReadWriteLock.readLock()`




## 阻塞队列 BlockingQueue

> 四组 API

| 操作 | 抛出异常 | 有返回值，不抛出异常 | 阻塞等待 | 超时等待                                |
| ---- | -------- | -------------------- | -------- | --------------------------------------- |
| 添加 | add      | offer()              | put      | offer(E e, long timeout, TimeUnit unit) |
| 移除 | remove   | poll()               | take     | poll(long timeout, TimeUnit unit        |
| 判断 | element  | peek                 |          |                                         |



> 抛出异常

```java
// 抛出异常
public static void test1() {
    ArrayBlockingQueue<String> arrayBlockingQueue = new ArrayBlockingQueue<>(3);

    // 查看队首元素，队首没有元素时抛出异常：Exception in thread "main" java.util.NoSuchElementException
    // System.out.println(arrayBlockingQueue.element());
    
    System.out.println(arrayBlockingQueue.add("a"));
    System.out.println(arrayBlockingQueue.add("b"));
    System.out.println(arrayBlockingQueue.add("c"));
    // 超出容量会抛出异常：Exception in thread "main" java.lang.IllegalStateException: Queue full
    // System.out.println(arrayBlockingQueue.add("d"));
    
    // 查看队首元素
    System.out.println(arrayBlockingQueue.element());
    
    System.out.println(arrayBlockingQueue.remove());
    System.out.println(arrayBlockingQueue.remove());
    System.out.println(arrayBlockingQueue.remove());
    // 空队列继续取出会抛出异常：Exception in thread "main" java.util.NoSuchElementException
    // System.out.println(arrayBlockingQueue.remove());
}
```



> 有返回值，不抛出异常

```java
// 有返回值，不抛出异常
public static void test2() {
    ArrayBlockingQueue<String> arrayBlockingQueue = new ArrayBlockingQueue<>(3);
    
    // 查看队首元素，队首没有元素时，不抛出异常，返回 null
    System.out.println(arrayBlockingQueue.peek());
    
    System.out.println(arrayBlockingQueue.offer("a"));
    System.out.println(arrayBlockingQueue.offer("b"));
    System.out.println(arrayBlockingQueue.offer("c"));
    // 不抛出异常，返回 false
    // System.out.println(arrayBlockingQueue.offer("d"));
    
    // 查看队首元素
    System.out.println(arrayBlockingQueue.peek());
    
    System.out.println(arrayBlockingQueue.poll());
    System.out.println(arrayBlockingQueue.poll());
    System.out.println(arrayBlockingQueue.poll());
    // 不抛出异常，返回 null
    // System.out.println(arrayBlockingQueue.poll());
}
```



> 阻塞

```java
// 阻塞
public static void test3() throws InterruptedException {
    ArrayBlockingQueue<String> arrayBlockingQueue = new ArrayBlockingQueue<>(3);
    arrayBlockingQueue.put("a");
    arrayBlockingQueue.put("b");
    arrayBlockingQueue.put("c");
    // 队列满了，阻塞
    // arrayBlockingQueue.put("d");
    System.out.println(arrayBlockingQueue.take());
    System.out.println(arrayBlockingQueue.take());
    System.out.println(arrayBlockingQueue.take());
    // 队列空了，阻塞
    // System.out.println(arrayBlockingQueue.take());
}
```



> 超时阻塞

```java
// 超时阻塞
public static void test4() throws InterruptedException {
    ArrayBlockingQueue<String> arrayBlockingQueue = new ArrayBlockingQueue<>(3);
    System.out.println(arrayBlockingQueue.offer("a"));
    System.out.println(arrayBlockingQueue.offer("b"));
    System.out.println(arrayBlockingQueue.offer("c"));
    // 阻塞等待指定时间
    System.out.println(arrayBlockingQueue.offer("d", 2, TimeUnit.SECONDS));
    System.out.println(arrayBlockingQueue.poll());
    System.out.println(arrayBlockingQueue.poll());
    System.out.println(arrayBlockingQueue.poll());
    // 阻塞等待指定时间
    System.out.println(arrayBlockingQueue.poll(2, TimeUnit.SECONDS));
}
```



> SychronizedQueue 同步队列

容量为1，放入元素后，必须等取出后才能再次放入



## 线程池（重点）：三大方法、七大参数、四种拒绝策略

> 好处

* 降低资源消耗
* 提高响应速度
* 方便管理



> 作用

* 线程复用
* 可以控制最大并发数
* 管理线程



> 三大方法

```java
package com.example.threadpool;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;

// Executors 工具类，含三大方法
public class Test1 {
    public static void main(String[] args) {
//        ExecutorService executorService = Executors.newSingleThreadExecutor();// 单个线程的线程池
//        ExecutorService executorService = Executors.newFixedThreadPool(5);// 固定大小的线程池
        ExecutorService executorService = Executors.newCachedThreadPool();// 可伸缩的线程池

        try {
            for (int i = 0; i < 10; i++) {
                executorService.execute(() -> {
                    System.out.println(Thread.currentThread().getName());
                    try {
                        TimeUnit.SECONDS.sleep(1);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                });
            }
        } finally {
            executorService.shutdown();
        }
    }
}
```



> 七大参数

* 源码分析

  ```java
  public static ExecutorService newSingleThreadExecutor() {
      return new FinalizableDelegatedExecutorService
          (new ThreadPoolExecutor(1, 1,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>()));
  }
  
  public static ExecutorService newFixedThreadPool(int nThreads) {
      return new ThreadPoolExecutor(nThreads, nThreads,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>());
  }
  
  public static ExecutorService newCachedThreadPool() {
      return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                    60L, TimeUnit.SECONDS,
                                    new SynchronousQueue<Runnable>());
  }
  ```

  ```java
  public ThreadPoolExecutor(int corePoolSize, // 线程池的基本大小
                            int maximumPoolSize, // 线程池中允许的最大线程数
                            long keepAliveTime, // 存活的时间
                            TimeUnit unit, // 时间单位
                            BlockingQueue<Runnable> workQueue, // 工作队列
                            ThreadFactory threadFactory, // 线程工厂
                            // 拒绝策略
                            RejectedExecutionHandler handler) {
      if (corePoolSize < 0 ||
          maximumPoolSize <= 0 ||
          maximumPoolSize < corePoolSize ||
          keepAliveTime < 0)
          throw new IllegalArgumentException();
      if (workQueue == null || threadFactory == null || handler == null)
          throw new NullPointerException();
      this.corePoolSize = corePoolSize;
      this.maximumPoolSize = maximumPoolSize;
      this.workQueue = workQueue;
      this.keepAliveTime = unit.toNanos(keepAliveTime);
      this.threadFactory = threadFactory;
      this.handler = handler;
  }
  ```



> 四种拒绝策略

* AbortPolicy：当新的任务被拒绝时，会抛出一个异常
* DiscardPolicy：会悄悄地忽略掉被拒绝的任务，不会抛出异常
* CallerRunsPolicy：由提交者自己执行该任务
* DicardOldestPolicy：会把最先进入工作队列的任务出队，为新的任务腾出位置



> 自定义线程池

```java
// 自定义线程池
ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(
    2,
    // CPU 可同时执行线程数
    Runtime.getRuntime().availableProcessors(),
    3,
    TimeUnit.SECONDS,
    new LinkedBlockingQueue<Runnable>(3),
    Executors.defaultThreadFactory(),
    new ThreadPoolExecutor.DiscardPolicy()
);
```



### 如何确定线程池允许的最大线程数

> CPU 密集型

根据 CPU 可同时执行线程数确定

```java
Runtime.getRuntime.availableProcessors();
```



> IO 密集型

根据程序中的网络、磁盘 IO 数确定



## 四大函数式接口（必须掌握）

函数式接口：一个有且仅有一个抽象方法，但是可以有多个非抽象方法的接口



> Supplier

```java
package java.util.function;

@FunctionalInterface
public interface Supplier<T> {

    /**
     * Gets a result.
     *
     * @return a result
     */
    T get();
}
```





> Consumer

```java
package java.util.function;

import java.util.Objects;
 
@FunctionalInterface
public interface Consumer<T> {

    /**
     * Performs this operation on the given argument.
     *
     * @param t the input argument
     */
    void accept(T t);

    default Consumer<T> andThen(Consumer<? super T> after) {
        Objects.requireNonNull(after);
        return (T t) -> { accept(t); after.accept(t); };
    }
}
```



> Function

```java
package java.util.function;

import java.util.Objects;

@FunctionalInterface
public interface Function<T, R> {

    /**
     * Applies this function to the given argument.
     *
     * @param t the function argument
     * @return the function result
     */
    R apply(T t);

    default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
        return (V v) -> apply(before.apply(v));
    }

    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t) -> after.apply(apply(t));
    }

    static <T> Function<T, T> identity() {
        return t -> t;
    }
}

```



> Predicate

```java
package java.util.function;

import java.util.Objects;

@FunctionalInterface
public interface Predicate<T> {

    /**
     * Evaluates this predicate on the given argument.
     *
     * @param t the input argument
     * @return {@code true} if the input argument matches the predicate,
     * otherwise {@code false}
     */
    boolean test(T t);

    default Predicate<T> and(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) && other.test(t);
    }

    default Predicate<T> negate() {
        return (t) -> !test(t);
    }

    default Predicate<T> or(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) || other.test(t);
    }

    static <T> Predicate<T> isEqual(Object targetRef) {
        return (null == targetRef)
                ? Objects::isNull
                : object -> targetRef.equals(object);
    }

    @SuppressWarnings("unchecked")
    static <T> Predicate<T> not(Predicate<? super T> target) {
        Objects.requireNonNull(target);
        return (Predicate<T>)target.negate();
    }
}
```



## Stream 流式计算

> 示例

```java
package com.example.streamdemo;

import com.example.streamdemo.pojo.User;
import org.springframework.util.StringUtils;

import java.util.Arrays;
import java.util.Comparator;
import java.util.List;
import java.util.Optional;

/**
 * 题目要求：
 * 现在有5个用户
 * 筛选：
 * 1. ID 必须是偶数
 * 2. 年龄必须大于23岁
 * 3. 用户名转为大写
 * 4. 用户名字母倒序排序
 * 5. 只输出一个用户
 */
public class Test1 {
    public static void main(String[] args) {
        User user1 = new User(1, "a", 21);
        User user2 = new User(2, "b", 22);
        User user3 = new User(3, "c", 23);
        User user4 = new User(4, "d", 24);
        User user5 = new User(6, "e", 25);
        List<User> users = Arrays.asList(user1, user2, user3, user4, user5);
        users.stream()
                .filter(user -> user.getId() % 2 == 0)
                .filter(user -> user.getAge() > 23)
                .peek(user -> user.setName(user.getName().toUpperCase()))
                .sorted(Comparator.comparing(User::getName).reversed())
                .limit(1)
                .forEach(System.out::println);
    }
}
```



## 分支合并 ForkJoin

具体参考：https://www.cnblogs.com/hayasi/p/10639994.html

​				   https://www.jianshu.com/p/ef2eb256840d





## 异步回调

> Future 接口的实现类 CompletableFuture

```java
package com.example.future;

import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.TimeUnit;

public class TestFuture {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        // 没有返回值的 runAsync 异步回调
//        CompletableFuture<Void> completableFuture = CompletableFuture.runAsync(() -> {
//            try {
//                TimeUnit.SECONDS.sleep(2);
//            } catch (InterruptedException e) {
//                e.printStackTrace();
//            }
//            System.out.println(Thread.currentThread().getName() + "异步回调");
//        });
//        System.out.println("1111");
//        completableFuture.get(); // 阻塞获取执行结果
        CompletableFuture<Integer> completableFuture = CompletableFuture.supplyAsync(() -> {
            System.out.println(Thread.currentThread().getName() + "有返回值异步回调");
            return 1024;
        });
        System.out.println(completableFuture.whenComplete((t, u) -> {
            System.out.println("t：" + t); // 正常的返回值
            System.out.println("u：" + u); // 异常信息
        }).exceptionally((e) -> {
            e.printStackTrace();
            return 233;
        }).get());
    }
}
```



## JMM





## volatile

* 保证可见性
* 保证顺序性，禁止指令重排
* 不保证原子性



## 单例模式

具体参考：https://www.cnblogs.com/happy4java/p/11206105.html



## 深入理解CAS

> 什么是 CAS

* compare and swap 比较并交换
* 解决多线程并行情况下使用锁造成性能损耗的一种机制，CAS操作包含三个操作数——内存位置（V）、预期原值（A）和新值（B）。如果内存位置的值与预期原值相匹配，那么处理器会自动将该位置值更新为新值。否则，处理器不足怕任何操作。

> Unsafe 类



> ABA 问题



## 原子引用



## 可重入锁



## 自旋锁

```java
package com.example.spinlock;

import java.util.concurrent.atomic.AtomicReference;

public class SpinLock {
    private AtomicReference<Thread> atomicReference = new AtomicReference<>();
    
    public void lock() {
        Thread thread = Thread.currentThread();
        while (!atomicReference.compareAndSet(null, thread)) {
            
        }
    }
    
    public void unlock() {
        Thread thread = Thread.currentThread();
        atomicReference.compareAndSet(thread, null);
    }
}
```



## 死锁排查

1. 使用 `jps -l` 定位 java 程序进程号
2. 使用 `jstack 进程号` 查看进程信息