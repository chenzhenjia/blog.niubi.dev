---
title: "Linux 各个目录的说明"
date: 2018-08-17T11:52:00+08:00
aliases: [linux-ge-ge-mu-lu-de-shuo-ming]
---

## 根文件系统
### /bin
&nbsp;&nbsp;&nbsp;&nbsp;这一目录中存放了供所有用户使用的完成基本维护任务的命令。其中bin是binary的缩写，表示二进制文件，通常为可执行文件。
&nbsp;&nbsp;&nbsp;&nbsp;一些常用的系统命令，如`cp`、`ls`等保存在该目录中。
### /boot
&nbsp;&nbsp;&nbsp;&nbsp;这里存放的是启动Linux时使用的一些核心文件。如操作系统内核、引导程序*Grub*等。
### /dev
&nbsp;&nbsp;&nbsp;&nbsp;在此目录中包含所有的系统设备文件。从此目录可以访问各种系统设备。如CD-ROM，磁盘驱动器，调制解调器和内存等。
&nbsp;&nbsp;&nbsp;&nbsp;在该目录中还包含有各种实用功能，如用于创建设备文件的MAKEDEV。
### /etc
&nbsp;&nbsp;&nbsp;&nbsp;该目录中包含系统和应用软件的配置文件。
### /etc/passwd
&nbsp;&nbsp;&nbsp;&nbsp;该目录中包含了系统中的用户描述信息，每行记录一个用户的信息。
### /home
&nbsp;&nbsp;&nbsp;&nbsp;存储普通用户的个人文件。每个用户的主目录均在/home下以自己的用户名命名。
### /lib
&nbsp;&nbsp;&nbsp;&nbsp;这个目录里存放着系统最基本的共享链接库和内核模块。共享链接库在功能上类似于Windows里的.dll文件。
### /lib64
&nbsp;&nbsp;&nbsp;&nbsp;**64位系统**有这个文件夹，64位程序的库。
### /lost+found
&nbsp;&nbsp;&nbsp;&nbsp;这并不是Linux目录结构的组成部分，而是ext3文件系统用于保存丢失文件的地方。
&nbsp;&nbsp;&nbsp;&nbsp;不恰当的关机操作和磁盘错误均会导致文件丢失，这意味着这些被标注为“在使用”，但却并未列于磁盘上的数据结构上。
&nbsp;&nbsp;&nbsp;&nbsp;正常情况下，引导进程会运行fsck程序，该程序能发现这些文件。除了“/”分区上的这个目录外，在每个分区上均有一个lost+found目录。
### /media
&nbsp;&nbsp;&nbsp;&nbsp;可移动设备的挂载点，当前的操作系统通常会把U盘等设备自动挂载到该文件夹下。
### /mnt
&nbsp;&nbsp;&nbsp;&nbsp;临时用于挂载文件系统的地方。一般情况下这个目录是空的，而在我们将要挂载分区时在这个目录下建立目录，再将我们将要访问的设备挂载在这个目录上，这样我们就可访问文件了。（注意在GNOME中，只有挂载到/media的文件夹才会显示在“计算机”中，挂载到/mnt不会做为特殊设备显示，详见自动挂载分区）
### /opt
&nbsp;&nbsp;&nbsp;&nbsp;多数第三方软件默认安装到此位置，如Adobe Reader、google-earth等。并不是每个系统都会创建这个目录。
### /proc
&nbsp;&nbsp;&nbsp;&nbsp;它是存在于内存中的虚拟文件系统。里面保存了内核和进程的状态信息。多为文本文件，可以直接查看。如/proc/cpuinfo保存了有关CPU的信息。
### /root
&nbsp;&nbsp;&nbsp;&nbsp;这是根用户的主目录。与保留给个人用户的/home下的目录很相似，该目录中还包含仅与根用户有关的条目。
### /sbin
&nbsp;&nbsp;&nbsp;&nbsp;供超级用户使用的可执行文件，里面多是系统管理命令，如`fsck`, `reboot`, `shutdown`, `ifconfig`等。
### /tmp
&nbsp;&nbsp;&nbsp;&nbsp;该目录用以保存临时文件。该目录具有Sticky特殊权限，所有用户都可以在这个目录中创建、编辑文件。但只有文件拥有者才能删除文件。为了加快临时文件的访问速度，有的实现把/tmp放在内存中。
### /usr
&nbsp;&nbsp;&nbsp;&nbsp;静态的用户级应用程序等，见下。
### /var
&nbsp;&nbsp;&nbsp;&nbsp;动态的程序数据等，见下文。
## /usr目录结构

-------
&nbsp;&nbsp;&nbsp;&nbsp;/usr通常是一个庞大的文件夹，其下的目录结构与根目录相似，但根目录中的文件多是系统级的文件，而/usr中是用户级的文件，一般与具体的系统无关。
&nbsp;&nbsp;&nbsp;&nbsp;应注意，程序的配置文件、动态的数据文件等都不会存放到/usr，所以除了安装、卸载软件外，一般无需修改/usr中的内容。说在系统正常运行时，/usr甚至可以被只读挂载。由于这一特性，/usr常被划分在单独的分区，甚至有时多台计算机可以共享一个/usr。
> 提示：
> &nbsp;&nbsp;&nbsp;&nbsp;usr最早是user的缩写，/usr的作用与现在的/home相同。而目前其通常被认为是 User System Resources 的缩写，其中通常是用户级的软件等，与存放系统级文件的根目录形成对比。
### /usr/bin
&nbsp;&nbsp;&nbsp;&nbsp;多数日常应用程序存放的位置。如果/usr被放在单独的分区中，Linux的单用户模式不能访问/usr/bin，所以对系统至关重要的程序不应放在此文件夹中。
### /usr/include
&nbsp;&nbsp;&nbsp;&nbsp;存放C/C++头文件的目录
### /usr/lib
&nbsp;&nbsp;&nbsp;&nbsp;系统的库文件
### /usr/local
&nbsp;&nbsp;&nbsp;&nbsp;新装的系统中这个文件夹是空的，可以用于存放个人安装的软件。安装了本地软件的/usr/local里的目录结构与/usr相似
### /usr/sbin
&nbsp;&nbsp;&nbsp;&nbsp;在单用户模式中不用的系统管理程序，如apache2等。
### /usr/share
&nbsp;&nbsp;&nbsp;&nbsp;与架构无关的数据。多数软件安装在此。
### /usr/X11R6
&nbsp;&nbsp;&nbsp;&nbsp;该目录用于保存运行X-Window所需的所有文件。该目录中还包含用于运行GUI要的配置文件和二进制文件。
###/usr/src
&nbsp;&nbsp;&nbsp;&nbsp;源代码
## /var目录结构
-------
&nbsp;&nbsp;&nbsp;&nbsp;/var中包括了一些数据文件，如系统日志等。/var的存放使得/usr被只读挂载成为可能。
### /var/cache
&nbsp;&nbsp;&nbsp;&nbsp;应用程序的缓存文件
### /var/lib
&nbsp;&nbsp;&nbsp;&nbsp;应用程序的信息、数据。如数据库的数据等都存放在此文件夹。
### /var/local
&nbsp;&nbsp;&nbsp;&nbsp;/usr/local中程序的信息、数据
### /var/lock
&nbsp;&nbsp;&nbsp;&nbsp;锁文件
### /var/log
&nbsp;&nbsp;&nbsp;&nbsp;日志文件
### /var/opt
&nbsp;&nbsp;&nbsp;&nbsp;/opt中程序的信息、数据
### /var/run
&nbsp;&nbsp;&nbsp;&nbsp;正在执行着的程序的信息，如PID文件应存放于此
### /var/spool
&nbsp;&nbsp;&nbsp;&nbsp;存放程序的假脱机数据（即spool data）
### /var/tmp
&nbsp;&nbsp;&nbsp;&nbsp;临时文件