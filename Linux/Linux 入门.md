# Linux 入门

## 1 目录结构

### 1.1 基本介绍

* Linux 的文件系统是采用层级式的树状目录结构，在此结构中的最上层是根目录 `/` ，然后在此目录下再创建其他的目录。
* 在 Linux 世界里，一切皆文件



### 1.2 具体的目录结构

* `/bin` [常用] (/usr/bin、/usr/local/bin)

  是 Binary 的缩写，这个目录存放着最经常使用的命令

* `/sbin` (/usr/sbin、/usr/local/sbin)

  s 就是 Super User 的意思，这里存放的是系统管理员使用的系统管理程序

* `/home` [常用]

  存放普通用户的主目录，在 Linux 中每个用户都有一个自己的目录，一般该目录名是以用户的账号命名

* `/root` [常用]

  该目录为系统管理员，也称作超级权限者的用户主目录

* `/lib`

  系统开机所需要最基本的动态连接共享库，其作用类似于 Windows 里的 DLL 文件。几乎所有的应用程序都需要用到这些共享库

* `/lost+found`

  这个目录一般情况下是空的，当系统非法关机后 ，这里就存放了一些文件

* `/etc` [常用]

  所有的系统管理所需要的配置文件和子目录，比如安装 mysql 数据库 my.conf

* `/usr` [常用]

  这是一个非常重要的目录，用户的很多应用程序和文件都放在这个目录下，类似于 Windows 下的 program files 目录

* `/boot` [常用]

  存放的是启动 Linux 时使用的一些核心文件，包括一些连接文件以及镜像文件

* `/proc` [不能动]

  这个目录是一个虚拟的目录，它是系统内存的映射，访问这个目录来获取系统信息

* `/srv` [不能动]

  service 缩写，该目录存放一些服务启动之后需要提取的数据

* `/sys` [不能动]

  这是 Linux 2.6 内核的一个很大的变化。该目录下安装了2.6内核中新出现的一个文件系统 sysfs

* `/tmp`

  这个目录是用来存放一些临时文件的

* `/dev`

  类似于 Windows 的设备管理器，把所有的硬件用文件的形式存储

* `/media` [常用]

  Linux 系统会自动识别一些设备，如闪存、光驱等，当识别后，Linux 会把识别的设备挂在到这个目录下

* `/mnt` [常用]

  系统提供该目录是为了让用户临时挂载别的文件系统的，我们可以将外部的存储挂载在 `/mnt` 上，然后进入该目录就可以查看内容了

* `/opt`

  这是给主机额外安装软件所存放的目录。如安装 Oracle 数据库就可以放在该目录下

* `/usr/local` [常用]

  这是另一个给主机额外安装软件所安装的目录。一般是通过编译源码方式安装的程序

* `/var` [常用]

  这个目录中存放着不断扩充着的东西，习惯将经常被修改的目录放在这个目录下。包括各种日志文件

* /selinux

  security-enhanced linux SELinux 是一种安全子系统，它能控制程序只能访问特定文件，有三种工作模式，可以自从设置

