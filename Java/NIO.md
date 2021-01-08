# NIO

## 1 Java NIO 简介

* Java NIO（New IO）是从 Java 1.4版本开始引入的一个新的 IO API，可以替代标准的 Java IO API
* NIO 与原来的 IO 有同样的作用和目的，但是使用的方式完全不同，NIO 支持面向缓冲区的、基于通道的 IO 操作
* NIO 将以更加高效的方式进行文件的读写操作



## 2 Java NIO 与 IO 的主要区别

| IO                        | NIO                           |
| ------------------------- | ----------------------------- |
| 面向流（Stream Oriented） | 面向缓冲区（Buffer Oriented） |
| 阻塞IO（Blocking IO）     | 非阻塞 IO（Non Blocking IO）  |
| （无）                    | 选择器（Selectors）           |



## 3 缓冲区（Buffer）和通道（Channel）

* Java NIO 的核心在于：通道（Channel）和缓冲区（Buffer）。
* 通道表示打开到 IO 设备的连接。若需要使用 NIO，需要获取用于连接 IO 设备的通道以及用于容纳数据你的缓冲区。然后操作缓冲区，对数据进行处理

> 缓冲区（Buffer）

* 一个用于特定基本数据类型的容器。由 java.nio 包定义的，所有缓冲区都是 Buffer 抽象类的子类

* Java NIO 中的 Buffer 主要用于与 NIO 通道进行交互，数据是从通道读入缓冲区，从缓冲区写入通道中的

* 缓冲区就是数组。用于存储不同数据类型的数据，根据数据类型不同（boolean 除外），提供了相应类型的缓冲区，通过 allocate() 获取缓冲区

   * ByteBuffer
   * CharBuffer
   * ShortBuffer
   * IntBuffer
   * LongBuffer
   * FloatBuffer
   * DoubleBuffer

* 缓冲区存取数据的两个核心方法：
   * put()：存入数据到缓冲区中
   * get()：获取缓冲区中的数据

  ```java
  @Test
  void contextLoads() {
      String str = "abcde";
  
      // 1. 分配一个指定大小的缓冲区
      ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
      System.out.println(byteBuffer.position());
      System.out.println(byteBuffer.limit());
      System.out.println(byteBuffer.capacity());
  
      // 2. 利用 put() 存入数据到缓冲区中
      byteBuffer.put(str.getBytes(StandardCharsets.UTF_8));
      System.out.println(byteBuffer.position());
      System.out.println(byteBuffer.limit());
      System.out.println(byteBuffer.capacity());
  
      // 3. 切换读取数据模式
      byteBuffer.flip();
      System.out.println(byteBuffer.position());
      System.out.println(byteBuffer.limit());
      System.out.println(byteBuffer.capacity());
  
      // 4. 利用 get() 读取缓冲区中的数据
      byte[] dst = new byte[byteBuffer.remaining()];
      byteBuffer.get(dst);
      System.out.println(new String(dst,0, dst.length, StandardCharsets.UTF_8));
      System.out.println(byteBuffer.position());
      System.out.println(byteBuffer.limit());
      System.out.println(byteBuffer.capacity());
  
      // 5. rewind()，重置读取状态的 position 和 mark
      byteBuffer.rewind();
      System.out.println(byteBuffer.position());
      System.out.println(byteBuffer.limit());
      System.out.println(byteBuffer.capacity());
  
      // 6. clear()，清空缓冲区，但是缓冲区中的数据依然存在，只是处于“被遗忘”状态
      byteBuffer.clear();
      System.out.println(byteBuffer.position());
      System.out.println(byteBuffer.limit());
      System.out.println(byteBuffer.capacity());
  }
  ```

  

* 缓冲区中的四个核心属性
   * capacity：容量，表示缓冲区中最大存储数据的容量，一旦声明不能改变
   * limit：界限，表示缓冲区中可以操作数据的大小（从 limit 标记的数据开始，不能进行读写）
   * position：位置，表示缓冲区中正在操作数据的位置
   * mark：标记，记录当前 position 的位置。可以通过 reset() 将 position 恢复到 mark 的位置
   * mark <= position <= limit <= capacity

  ```java
  @Test
  void test1() {
      String str = "abcde";
      ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
      byteBuffer.put(str.getBytes(StandardCharsets.UTF_8));
      byteBuffer.flip();
      byte[] dst = new byte[byteBuffer.remaining()];
      byteBuffer.get(dst, 0, 2);
      System.out.println(byteBuffer.position());
      System.out.println(new String(dst, 0, dst.length, StandardCharsets.UTF_8));
  
      // 标记
      byteBuffer.mark();
      byteBuffer.get(dst, 0, 2);
      System.out.println(byteBuffer.position());
      System.out.println(new String(dst, 0, dst.length, StandardCharsets.UTF_8));
  
      // 将 position 恢复到 mark 的位置
      byteBuffer.reset();
      System.out.println(byteBuffer.position());
  
      // 判断缓冲区中是否还有未读取的数据
      if (byteBuffer.hasRemaining()) {
          // 获取缓冲区中可以操作的数据的大小
          System.out.println(byteBuffer.remaining());
      }
  }
  ```

  

> 直接缓冲区与非直接缓冲区

* 字节缓冲区要么是直接的，要么是非直接的。如果为直接字节缓冲区，则 Java 虚拟机会尽最大努力直接在此缓冲区上执行本机 I/O 操作。也就是说，在每次调用基础操作系统的一个本机 I/O 操作之前（或之后），虚拟机都会尽量避免将缓冲区的内容复制到中间缓冲区中（或从中间缓冲区中复制内容）
* 直接字节缓冲区可以通过调用此类的 `allacateDirect()` 工厂方法来创建。此方法返回的缓冲区进行分配和取消分配所需成本通常高于非直接缓冲区。直接缓冲区的内容可以驻留在常规的垃圾回收堆之外，因此，它们对应用程序的内存需求量造成的影响可能并不明显。所以，建议将直接缓冲区主要分配给那些易受基础系统的本机 I/O 操作影响的大型、持久的缓冲区。一般情况下，最好仅在直接缓冲区能在程序性能方面带来明显好处时分配它们
* 直接字节缓冲区还可以通过 FileChannel 的 `map()` 方法将文件区域直接映射到内存中来创建。该方法返回 MapperByteBuffer。Java 平台的实现有助于通过 JNI 从本机代码创建直接字节缓冲区。如果以上这些缓冲区中的某个缓冲区实例指的是不可访问的内存区域，则试图访问该区域不会更改该缓冲区的内容，并且将会在访问期间或稍后的某个时间导致抛出不确定的异常
* 字节缓冲区是直接缓冲区还是非直接缓冲区可通过调用其 `isDirect()` 方法来确定。提供此方法是为了能够在性能关键型代码中执行显式缓冲区管理

```java
@Test
void test3() {
    // 分配直接缓冲区
    ByteBuffer byteBuffer = ByteBuffer.allocateDirect(1024);
    System.out.println(byteBuffer.isDirect());
}
```



> 通道

* 由 java.nio.channels 包定义的。Channel 表示 IO 源与目标打开的连接。Channel 类似于传统的“流”。只不过 Channel 本身不能直接访问数据，Channel 只能与 Buffer 进行交互
* 通道（java.nio.channels.Channel）的主要实现类
  * FileChannel
  * SocketChannel
  * ServerSocketChannel
  * DatagramChannel
* 获取通道
  1. Java 针对支持通道的类提供了 getChannel() 方法
     * 本地 IO
       * FileInputStream / FileOutputStream
       * RandomAccessFile
     * 网络 IO
       * Socket
       * ServerSocket
       * DatagramSocket
  2. 在 JDK 1.7 中的 NIO2 针对各个通道提供了静态方法 open()
  3. 在 JDK 1.7 中的 NIO2 的 Files 工具类 newByteChannel()



## 4 文件通道（FileChannel）



## 5 NIO 的非阻塞式网络通信

### 5.1 选择器（Selector）

### 5.2 SocketChannel、ServerSocketChannel、DatagramChannel



## 6 管道（Pipe）





## 7 Java NIO2（Path、Paths 与 Files）