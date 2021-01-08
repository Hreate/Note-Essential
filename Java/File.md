# File

## 1 File类概述和构造方法

File：它是文件和目录路径名的抽象表示

* 文件和目录是可以通过 File 封装成对象的
* 对于 File 而言，其封装的并不是一个真正存在的文件，仅仅是一个路径名而已。它可以是存在的，也可以是不存在的。将来是要通过具体的操作把这个路径的内容转换为具体存在的



> 构造方法

| Constructor                       | 描述                                                       |
| --------------------------------- | ---------------------------------------------------------- |
| File(File parent, String child)   | 从父抽象路径和子路径名字字符串创建新的 File 实例           |
| File(String pathname)             | 通过将给定的路径字符串转换为抽象路径名来创建新的 File 实例 |
| File(String parent, String child) | 从父路径名字符串和子路径名字符串创建新的 File 实例         |
| File(URI uri)                     | 通过将给定的 file:URI 转换为抽象路径名来创建新的 File 实例 |



## 2 File 类创建功能

| 方法名                       | 说明                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| public boolean createNewFile | 当具有该名称的文件不存在时，创建一个由该抽象路径名命名的新空文件 |
| public boolean mkdir()       | 创建由此抽象路径名命名的目录                                 |
| public boolean mkdirs()      | 创建由此抽象路径名命名的目录，包括任何必需但不存在的父目录   |



## 3 File 类判断和获取功能

| 方法名                          | 说明                                                     |
| ------------------------------- | -------------------------------------------------------- |
| public boolean isDirectory()    | 测试此抽象路径名表示的 File 是否为目录                   |
| public boolean isFile()         | 测试此抽象路径名表示的 File 是否为文件                   |
| public boolean exists()         | 测试此抽象路径名表示的 File 是否存在                     |
| public String getAbsolutePath() | 返回此抽象路径名的绝对路径名字符串                       |
| public String getPath()         | 将此抽象路径名转换为路径名字字符串                       |
| public String getName()         | 返回由此抽象路径名表示的文件或目录的名称                 |
| public String[] list()          | 返回此抽象路径名表示的目录中的文件和目录的名称字符串数组 |
| public File[] listFiles()       | 返回此抽象路径名表示的目录中的文件和目录的 File 对象数组 |



## 4 File 类删除功能

| 方法名                  | 说明                               |
| ----------------------- | ---------------------------------- |
| public boolean delete() | 删除由此抽象路径名表示的文件或目录 |

