title: Netkit Filesystem 定制和裁剪
tags:
  - Network
id: 338
categories:
  - Network
date: 2013-11-16 18:42:51
---

Netkit, “The poor man’s system for experimenting computer networking”，是基于Linux的开源网络实验环境，该环境通过启动多台Linux虚拟机，模拟路由器，交换机和PC的网络通信功能，Netkit中预装了路由协议软件包Quagga,可以提供多种路由协议功能，如RIP， OSPF， BGP，IS-IS等，Netkit主要用于网络功能的模拟，而不做网络性能的模拟。

最近花了一周的时间实验Netkit，是网络的大作业，如果不是作业的机会，也不会有空去看这么古老的Netkit，他是北欧开发的一个开源网络实验环境，相比Packet Tracer美观的图形界面，对于没有图形界面的Netkit来说，Netkit的优势在于提供了用户模式的Linux内核，每一个虚拟机就是一个Linux系统，可以更真实地模拟路由等网络功能，用户通过命令行操作虚拟机，能更好地学习和实验网络，而不是学习一个Cisco设备。如果有朋友以后用到Netkit，希望这个Blog会helpful。

Netkit主要由三部分构成：

*   netkit-core: 包含启动虚拟机和网络实验的脚本文件，如vstart，lstart等。
*   netkit-uml-filesystem: 这是一个基于Debian的文件系统，Netkit在该文件系统中安装了软件包，并配置了quagga，ebtables, iptables等服务。Netkit的文件系统是“copy-on-write”的模式，每个虚拟机启动时会生成一个*.disk文件，该文件就是该文件系统的拷贝，在虚拟机中作的任何更改都不会影响Netkit的Filesystem，修改写入*.disk文件.
*   netkit-uml-kernel: 这是一个linux kernel， Netkit通过patch文件对kernel进行了相关配置，Netkit的kernel是用户模式的linux进程，也叫做虚拟机,我们可以在Linux系统如Ubuntu中启动netkit，kernel进程的运行依赖于文件系统。
我们发现Netkit的文件系统解压后超过10GB，每次启动一个虚拟机就会产生一个10GB的*.disk文件，占用空间较大，我们主要使用Netkit的Quagga进行路由协议的模拟，而Netkit很强大，提供了apache, FTP, openssl,openvpn, DNS, email等功能，我们不需要这些冗余功能，因此我们着手重新制作一个文件系统，并尽量让文件系统容量小，提高Netkit网络实验的性能。

Netkit官方提供了Makefile文件让开发者自己Build文件系统，然而实验后发现官方Makefile无法运行通过，并且Netkit官方指南中并不建议自己Build一个文件系统，在Google后也没有发现更多有用的文档，因此希望通过本Blog详细地描述如何制作一个小巧可用的Netkit文件系统以及遇到的问题。 [more...]

## 实验成果

最终定制和裁剪的文件系统只有335M，该文件系统基于Debian Squeeze版本定制，没有采用当前的最新版本Wheezy版，因为并不修改Netkit-Kernel，最新版的Wheezy文件系统和现有的Netkit Kernel不兼容。

该文件系统的制作简单来说，分为三个阶段：制作基础的linux filesystem， 配置为可用的Netkit文件系统，安装quagga和必要的软件包。这些步骤都是通过shell命令来完成的。

你可能会问，为什么是335M？我们最早制作的文件系统是2GB，在运行稳定后，我们发现2GB是稀疏文件，在虚拟机中通过df命令我们看到磁盘空间只占用了310M，在基础的Linux文件系统安装好后，磁盘占用空间在250M左右，而我们安装了less,vim,chkconfig,quagga,telnet, telnetd, tcptraceroute这7个必要的包后，磁盘占用空间为310M+，因此我们取文件系统320M来制作，在制作完成后，大小为335M。

最终制作和裁剪的文件系统通过了静态路由，RIP， OSPF Single Area, OSPF Multi Area, BGP的测试。

定制文件系统的源代码在：[NetkitNewFS](https://github.com/lgrcyanny/NetkitNewFS)

测试的实验拓扑文件在：[NetkitLabs](https://github.com/lgrcyanny/NetkitLabs)

## 准备工作

1\. Ubuntu 32位环境或虚拟机

Netkit是基于32bit的Linux系统编译的，建议在32bit环境下运行，64Bit的机器安装会缺一些lib包，让工作更加麻烦.

2\. 安装Netkit

其次，安装Netkit，参见[官方网站](http://wiki.netkit.org/index.php/Main_Page)，安装很简单，解压+配置环境变量。

也可以看[NetkitNewFS](https://github.com/lgrcyanny/NetkitNewFS)源码下的README，有安装方法。

## 定制流程

整个定制过程大体可以分为三歩：制作基础的Linux内核文件系统，修改文件系统为可用的Netkit文件系统，为文件系统安装Quagga。下面让我们按照这个顺序来看看整个定制流程。

### step1.制作基础的Linux内核文件系统：

我们直接写一个bash脚本来完成第一步制作基础的Linux内核的文件系统的工作，完整的脚本内容如下：

脚本不长([netkit-fs-build.sh](https://github.com/lgrcyanny/NetkitNewFS/blob/master/netkit-fs-build.sh))
[bash]
#!/bin/bash
echo &quot;====================Installing Base Linux System======================&quot;
FS_NAME=netkit-fs-light
MOUNT_DIR=/mnt/nkfs2
FILESYSTEM_SIZE=320
# each cylinder has 63 sectors, each of which is 512 bytes
let CYL_COUNT=$FILESYSTEM_SIZE*1048576/32256

# Create empty netkit-fs file
dd if=/dev/zero of=$FS_NAME bs=1M count=0 seek=$FILESYSTEM_SIZE

# Create image partition
echo &quot;,,L,*&quot; | sfdisk -q -H 1 -S 63 -C $CYL_COUNT $FS_NAME

# Binding lookback device to netkit-fs
device_name=$(losetup -f)
# The offset is the size of one track
losetup --offset 512 $device_name $FS_NAME

# Create ext2 filesystem
mkfs.ext2 $device_name
losetup -d $device_name

# Mount netkit-fs
rm -rf $MOUNT_DIR
mkdir $MOUNT_DIR
mount -o loop,offset=512 -t ext2 $FS_NAME $MOUNT_DIR

# Install the base filesystem  from Internet
debootstrap --arch i386 squeeze $MOUNT_DIR http://ftp.cn.debian.org/debian
echo &quot;=================Base Linux System Installed Success==================&quot;

umount $MOUNT_DIR
echo &quot;done!&quot;
[/bash]
脚本的功能是创建一个320M的文件，采用sfdisk创建Linux分区表，绑定到环回设备上创建ext2文件系统，在mount到主机的/mnt/nkfs2下，通过debootstrap命令联网联网下载并安装Linux Squeeze 文件系统。具体每一歩，在脚本注释中。
值得注意的是，脚本的最后部分，即命令：
[bash]
debootstrap --arch i386 squeeze $MOUNT_DIR http://ftp.cn.debian.org/debian
[/bash]
需要通过联网安装debian squeeze内核，该文件系统下载解压后只有200多兆，选择squeeze主要是为了和netkit-kernel兼容。
保存脚本文件，比如存为netkit-fs-build.sh，打开终端，cd到脚本文件所在目录下，使用命令：
[bash]
sudo sh netkit-fs-build.sh
[/bash]
运行脚本（注意使用root权限），即可得到基础的Linux文件系统了。

### step2.修改文件系统为可用的Netkit文件系统：

#### 1\. 从Netkit原始文件系统中拷贝必要的文件，并修改netkit-phase1和netkit-phase2

上一步得到的文件系统，并不可以直接用于Netkit，启动虚拟机会失败，需要用到Netkit原始的文件系统中的部分文件进行补充，将其配置成可用的Netkit FileSystem。
所有需要添加到自制文件系统中的Netkit原始文件系统中的文件我们把他们放在了[netkit-tweaks](https://github.com/lgrcyanny/NetkitNewFS/tree/master/netkit-tweaks)文件夹下，其目录结构如下，各自作用见注释：
[text]
netkit-tweaks
---- etc
---------init.d #contains netkit-phase1 and netkit-phase2, the two important file for netkit bootstrap
---------network #interfaces configure
---------inittab # inittab file for bootstrap runlevel
---------resolv.conf # DNS parse
---------sysctl.conf # net.ipv4.ip_forward=1 for ip forwarding, important configure, enable ip packet forwarding
-----sbin
---------mingetty # mingetty is a minimal getty program for watching virtual termianl
[/text]

该文件夹下所有的文件都可以在Netkit原始文件系统下找到，可以通过挂载原始文件系统把他们从挂载地址拷贝出来，其中/mnt/nkfs1为自定义的挂载地址：
[bash]
mount -o loop,offset=32768 netkit-fs-i386-F5.2 /mnt/nkfs1
[/bash]
文件netkit-phase1和netkit-phase2需要稍作修改， 修改netkit-phase1中开头部分为：
[bash]
### BEGIN INIT INFO
# Provides:
# Required-Start:
# Required-Stop: $all
# Default-Start: 2 3 4 5
# Default-Stop: 0 1 6
# Short-Description:
# Description:
### END INIT INFO
[/bash]

修改netkit-phase2中开头部分为：
[bash]
### BEGIN INIT INFO
# Provides:
# Required-Start: $all
# Required-Stop: 
# Default-Start: 2 3 4 5
# Default-Stop: 0 1 6
# Short-Description:
# Description:
### END INIT INFO
[/bash]
$all的意思是最后，在netkit-phase1中表示最后停止，在netkit-phase2中表示最后开始，这是启动优先级的配置。
同时在netkit-phase2中，并将文件中/bin/sh -c 'source /hostlab/$HOSTNAME.startup'这种类似的语句修改为"source /hostlab/$HOSTNAME.startup"。
因为用/bin/sh启动 startup脚本会会报”/bin/sh: source not found”的错误，默认用bash启动startup脚本就不会报错。
至此，所有需要添加进自制文件系统的文件已经全部在netkit-tweaks文件夹中了。

#### 2\. 拷贝必要的文件到文件系统中

首先挂载刚刚创建的文件系统netkit-fs-light：
[bash]
FS_NAME=netkit-fs-light
NETKIT_TWEAKS_DIR=netkit-tweaks
MOUNT_DIR=/mnt/nkfs2
mount -o loop,offset=512 -t ext2 $FS_NAME $MOUNT_DIR
[/bash]
下面需要把一些必要的文件拷贝到自制的文件系统中，其中**/etc/sysctl.conf**文件很重要，作用是将ipv4.ip_forward设为1，否则虚拟机无法ping通：
[bash]
cp -Rvf $NETKIT_TWEAKS_DIR/etc $MOUNT_DIR/
cp -Rvf $NETKIT_TWEAKS_DIR/sbin $MOUNT_DIR/
[/bash]

#### 3\. 改变root根目录位置，准备安装必要的工具和软件

改变root的执行程序参考的根目录位置至挂载文件系统的位置，方便我们直接配置文件系统和使用文件系统中的命令：
[bash]
chroot $MOUNT_DIR
[/bash]
挂载proc文件系统，proc文件系统是提供Linux内核数据结构接口的伪文件系统，文件系统并没有挂载proc文件系统，proc文件系统一般挂载在/proc目录下，如果没有挂载，虚拟机启动时会出现mtab无法初始化错误，也就是mtab文件会被损坏，因此我们为了让文件系统更完善，增加挂载proc文件系统的命令。
[bash]
mount -t proc none /proc
[/bash]

#### 4\. 安装必要的软件包

紧接着为文件系统更新和安装一些必要的工具和软件：
[bash]
apt-get update
apt-get install less
apt-get install vim
apt-get install chkconfig #为设置netkit-phase文件的启动优先级
[/bash]

#### 5\. 设置netkit-phase1和netkit-phase2脚本的启动优先级

在Debian 6.0之后，不能采用update-rc.d命令设置init.d目录下启动文件的优先级，采用insserv命令可以简单设置。我们尝试过将rcX.d文件从原始netkit文件系统中拷贝到新的文件系统中，发现不能使用，rcX.d这些链接文件是需要通过命令来生成的。在原始的netkit Makefile中采用update-rc.d设置是不成功的，采用insserv是我们做出的改进。
将netkit-phase1和netkit-phase2链接到rcX.d并设置优先级：
[bash]
insserv netkit-phase1 
chkconfig netkit-phase1 on
insserv netkit-phase2 
chkconfig netkit-phase2 on 
[/bash]

#### 6\. 禁止不必要的启动项，取消挂载

取消cron job的启动项，加快启动
[bash]
update-rc.d cron remove
[/bash]
最后退出并取消挂载：
[bash]
exit # 退出新文件系统的根目录环境，回到ubunu根目录环境下
umount $MOUNT_DIR/proc
umount $MOUNT_DIR
[/bash]
如果因为设备正忙而无法取消挂载，可以察看相应进程，先杀死进程再取消挂载：
[bash]
fuser -m $MOUNT_DIR
# if out put is /dev/sdc1: 538 
ps -aux | grep 538
# then kill the process by proccess id
[/bash]

### step3.为文件系统安装quagga：

以上两步都是在Ubuntu主机下运行的，现在我们自制的Netkit FileSystem应该可以正常启动虚拟机了，最后一步需要启动虚拟机并在虚拟机内安装quagga。
首先启动Netkit虚拟机：
[bash]
FS_NAME=netkit-fs-light
vstart pc1 -m $FS_NAME -M 512 --eth0=tap,10.0.0.1,10.0.0.2 --append=rout:98:1 –W
[/bash]
该命令中参数解释如下：
–m $FS_NAME 表示以指定的文件系统作为虚拟机的文件系统
-M 512 指定内存512M，以为要联网安装，所以不能用默认的32M内存，太小
--eth0=tap,10.0.0.1,10.0.0.2 绑定eth0网卡到tap碰撞域，tap碰撞域是netkit保留的碰撞域，通过tap可以连接主机上外部因特网，10.0.0.1是TAP-ADDRESS是在Tap碰撞域中分配给主机Ubuntu的地址，10.0.0.2是GUEST-ADDRESS是分配给虚拟机的地址，此时可以ping任意的因特网，如baidu.com是可以成功的。
--append=rout:98:1 为netkit-kernel提供的参数
-W 表示对虚拟机的修改就是对Netkit文件系统的修改，不产生*.disk文件
启动后的虚拟机如图：
[![netkit](http://cyanny/myblog/wp-content/uploads/2013/11/netkit-580x393.png)](http://cyanny/myblog/wp-content/uploads/2013/11/netkit.png)
接下来在虚拟机中安装必要软件和工具：
[bash]
apt-get install telnet
apt-get install telnetd
apt-get install tcptraceroute
[/bash]
接着安装quagga，注意安装的quagga 版本为0.99.20.1，太高的版本可能会因为与内核版本不兼容：
[bash]
apt-cache policy quagga # 查看quagga可安装的版本，默认就有 0.99.20.1版本
apt-get install quagga -v 0.99.20.1
[/bash]
察看占用空间，最后退出虚拟机:
[bash]
dh -lf  # 320M is the smallest solution, since up to now, 310M has been used
halt
[/bash]
至此，我们的文件系统就制作成功了，以上step2和step3的脚本，请参考[configure-netkit.sh](https://github.com/lgrcyanny/NetkitNewFS/blob/master/configure-netkit.sh)，我采用的是手工配置，一方面命令不多，另一方面手工配置可以保证每一步的正确性。通过查看文件系统的信息我们看到文件系统为335M左右。
使用这一自制文件系统，我们通过了rip, ospf single area, ospf multi area和simple bgp的网络实验测试。Netkit lab实验在官方网站中有详细的教程，在此就不详细描述实验了，能做出这个文件系统，我表示很欣慰，同时也学习到文件系统的制作是通过shell命令完成，开始以为需要C方面的知识。

## 遇到的问题

在实验中，遇到了很多问题，这些问题对定制该文件系统有很大价值，在此做一个记录和说明，如果遇到相似的问题，可以参考。

### 1\. 虚拟机启动闪退

我们在做netkit文件系统时，有三个组员出现了启动虚拟机闪退的问题，我们为此花费了很多时间，的解决方法如下：
1.检查基础Linux文件系统是否安装成功，之前是因为文件系统的分区就没有创建成功，造成debootstrap也没有成功，文件没有安装好，启动闪退。
2.基础文件系统安装好了，拷贝了netkit-phaseX等文件后，启动虚拟机闪退，这是因为netkit-phaseX文件没有设置好优先级，需要在Ubuntu主机中通过insserv和chkconfig设置优先级。方可启动不闪退。

### 2\. Quagga安装失败

我尝试过在Ubuntu中，通过chroot后，在文件系统中安装quagga，此时安装的quagga的版本不是0.99.20.1版，安装失败，之后又在虚拟机种通过手工编译安装quagga，依然失败。
最后的解决方法是，在虚拟机中，采用apt-get install quagga –v 0.99.20.1安装，-v指定版本，quagga的安装需要依赖netkit提供的内核，在ubuntu安装是没有netkit内核的，造成不能使用。

### 3\. IP Forwarding问题

Quagga安装成功后，lab可以启动了，路由可以学习到路由，但是不能ping通，纠结一阵之后，我们发现是ip forward在Linux中默认关闭了，打开就可以。因此从原netkit文件系统中拷贝sysctl.conf文件，该文件可以永久设定net.ipv4.ip_forward=1，这样ip forward在启动时就已经打开，虚拟机之间就可以ping通了。

### 4\. Fedora- x64下配置虚拟机和配更改文件系统的问题(from Aqua)

在Ubuntu系统中配置完成实验之后，我们尝试了在64位Fedora系统中配置Netkit虚拟机。由于Netkit是基于32位系统编译，所以要首先解决64位系统中的不同函数库的依赖问题。在Debian系统中，解决32位函数库依赖只需要apt-get ia32-libs即可。但是在Fedora的yum系统中，并不存在这个名称的包。经验证，如下命令解决问题。
[bash]
su -c 'yum -y install --skip-broken glibc.i686 arts.i686 audiofile.i686 bzip2-libs.i686 cairo.i686 cyrus-sasl-lib.i686 dbus-libs.i686 directfb.i686 esound-libs.i686 fltk.i686 freeglut.i686 gtk2.i686 hal-libs.i686 imlib.i686 lcms-libs.i686 lesstif.i686 libacl.i686 libao.i686 libattr.i686 libcap.i686 libdrm.i686 libexif.i686 libgnomecanvas.i686 libICE.i686 libieee1284.i686 libsigc++20.i686 libSM.i686 libtool-ltdl.i686 libusb.i686 libwmf.i686 libwmf-lite.i686 libX11.i686 libXau.i686 libXaw.i686 libXcomposite.i686 libXdamage.i686 libXdmcp.i686 libXext.i686 libXfixes.i686 libxkbfile.i686 libxml2.i686 libXmu.i686 libXp.i686 libXpm.i686 libXScrnSaver.i686 libxslt.i686 libXt.i686 libXtst.i686 libXv.i686 libXxf86vm.i686 lzo.i686 mesa-libGL.i686 mesa-libGLU.i686 nas-libs.i686 nss_ldap.i686 cdk.i686 openldap.i686 pam.i686 popt.i686 pulseaudio-libs.i686 sane-backends-libs-gphoto2.i686 sane-backends-libs.i686 SDL.i686 svgalib.i686 unixODBC.i686 zlib.i686 compat-expat1.i686 compat-libstdc++-33.i686 openal-soft.i686 alsa-oss-libs.i686 redhat-lsb.i686 alsa-plugins-pulseaudio.i686 alsa-plugins-oss.i686 alsa-lib.i686 nspluginwrapper.i686 libXv.i686 libXScrnSaver.i686 qt.i686 qt-x11.i686 pulseaudio-libs.i686 pulseaudio-libs-glib2.i686 alsa-plugins-pulseaudio.i686'
[/bash]
这个长命令涵盖了ia32-libs中包含的所有依赖文件。
但是在生成文件系统过程中，由于Debian的Debootstrap并不能很好的兼容Fedora系统，导致生成文件系统失败。尚未找到更好的解决方案。
在Fedora下的尝试不成功，这也就是我们选取Ubuntu的原因，Netkit是32bitDebian系统，在32bitDebian系统下编译会更好。

### 参考资源

[1] [Netkit官方网站](http://wiki.netkit.org/index.php/Main_Page)
[2] [Netkit官方FAQ](http://wiki.netkit.org/index.php/FAQ#How_can_I_build_a_custom_kernel.2Ffilesystem_from_scratch_for_use_with_Netkit.3F)
[3] [Netkit 官方实验和介绍](http://wiki.netkit.org/index.php/Labs_Official)
[4] [Netkit用户问答](http://list.dia.uniroma3.it/pipermail/netkit.users/2007-October/000271.html)
[5] [Debian内核release介绍]( http://www.debian.org/releases/)
[6] [An introduction to services, runlevels, and rc.d scripts](http://www.linux.com/news/enterprise/systems-management/8116-an-introduction-to-services-runlevels-and-rcd-scripts)
[7] [Install quagga as linux router](http://opensourcecentre.wordpress.com/article/install-quagga-as-linux-router/)
[8] [Quagga Tutorial for ip forwarding](http://openmaniak.com/quagga_tutorial.php#ip_forwarding  )
[9] [How to umount when device is busy]( http://ocaoimh.ie/2008/02/13/how-to-umount-when-the-device-is-busy/)