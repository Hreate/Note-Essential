# IO 流

## 1 流的概念和作用

流：代表任何有能力产生数据的数据源对象或者是有能力接受数据的接收端对象

流的本质：数据传输，根据数据传输特性将流抽象为各种类，方便更直观的进行数据操作

作用：为数据源和目的地建立一个输送通道



### 1.1 Java IO 所采用的模型

Java 的 IO 模型设计使用的 Decorator（装饰者）模式，按功能划分 Stream，可以动态装配这些 Stream，以便获得需要的功能



### 1.2 IO 流的分类

#### 1.2.1 按数据流的方向分为输入流、输出流

此输入、输出是相对于代码程序而言



#### 1.2.2 按处理数据单位不同分为字节流、字符流

1字符 = 2字节、1字节(byte) = 8位(bit)、一个汉字占两个字节长度

字节流：每次读取（写出）一个字节，当传输的资源文件有中文时，就会出现乱码

字符流：每次读取（写出）两个字节，有中文时，使用该流就可以正确传输显示中文



#### 1.2.3 按功能不同分为节点流、处理流

节点流：以从或向一个特定的地方（节点）读写数据。如 FileInputStream

处理流：是对一个已存在的流的连接和封装，通过所封装的流的功能调用实现数据读写。如 BufferedReader。处理流的构造方法总是要带一个其他的流对象做参数。一个流对象经过其他流的多次包装



#### 1.2.4 4个基本的抽象流类型，所有的流都继承这四个

|        | 输入流      | 输出流       |
| ------ | ----------- | ------------ |
| 字节流 | InputStream | OutputStream |
| 字符流 | Reader      | Writer       |



### 1.3 IO 流常用到的五类一接口

在整个 java.io 包中最重要的就是五个类和一个接口。五个类指的是 File、OutputStream、InputStream、Writer、Reader；一个接口指的是 Serializeble



> 主要的类如下

* File（文件特征与管理）：File 类是对文件系统中文件以及文件夹进行封装的对象，可以通过对象的思想来操作文件和文件夹。File 类保存文件或目录的各种元数据信息，包括文件名、文件长度、最后修改时间、是否可读、获取当前文件的路径名，判断指定文件是否存在、获得当前目录中的文件列表，创建、删除文件和目录等方法
* InputStream（二进制格式操作）：抽象类，基于字节的输入操作，是所有输入流的父类。定义了所有输入流都具有的共同特征
* OutputStream（二进制格式操作）：抽象类，基于字节的输出操作，是所有输出流的父类。定义了所有输出流都具有的共同特征
* Reader（文件格式操作）：抽象类，基于字符的输入操作
* Writer（文件格式操作）：抽象类，基于字符的输出操作
* RandomAccesssFile（随机文件操作）：一个独立的类，直接继承自 Object。它的功能丰富，可以从文件的任意位置进行存取（输入输出）操作



### 1.4 IO 流对象

#### 1.4.1 输入字节流 InputStream

![](C:\Users\Cc_GY\Desktop\temp\images\874710-20170530112251336-478728150.png)

> ByteArrayInputStream

字节数组输入流，该类的功能就是从字节数组（byte[]）中进行以字节为单位的读取，也就是将资源文件都以字节的形式存入到该类中的字节数组中去，读取时也是从这个字节数组中读取



> PipedInputStream

管道字节输入流，它和 PipedOutpuStream 一起使用，能实现多线程间的管道通信



> FilterInputStream

装饰着模式中处于装饰者，具体的装饰者都要继承它，所以在该类的子类下都是用来装饰其他流的，也就是处理类



> BufferedInputStream

缓冲流，对处理流进行装饰，内部有一个缓冲区，用来存放字节，每次都是将缓冲区存满然后发送，而不是一个字节或两个字节这样发送。效率更高



> DataInputStream

数据输入流，用来装饰其他流，它“允许应用程序以与机器无关方式从底层输入流中读取基本 Java 数据类型”



> FileInputStream

文件输入流。它通常用于对文件进行读取操作



> File

对指定目录的文件进行操作



> ObjectInputStream

对象输入流，用来提供对“基本数据或对象”的持久存储。通俗点讲，也就是能直接传输对象（反序列化中使用）



#### 1.4.2 输出字节流 OutputStream

![](C:\Users\Cc_GY\Desktop\temp\images\874710-20170530121318274-160516931.png)

* ByteArrayOutputStream、FileOutputStream 是两种基本的介质流，它们分别向 Byte 数组、和本地文件中写入数据。
* PipedOutputStream 是向与其他线程公用的管道中写入数据
* ObjectOutputStream 和所有 FilterOutputStream 的子类都是装饰流（序列化中使用）