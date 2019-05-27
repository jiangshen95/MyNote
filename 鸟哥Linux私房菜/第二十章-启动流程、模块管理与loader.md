# Linux 的启动流程分析
## 启动流程一览
目前各大 Linux distributions 的主流为 grub，但早期 Linux 默认是使用 LILO 。  
当你按下电源按键后计算机硬件会主动的读取 BIOS 来加载硬件资讯及进行硬件系统的自我测试， 之后系统会主动的去读取第一个可启动的装置 (由 BIOS 配置的) ，此时就可以读入启动管理程序了。  
启动管理程序可以指定使用哪个核心文件来启动，并实际加载核心到内存当中解压缩与运行， 此时核心就能够开始在内存内活动，并侦测所有硬件资讯与加载适当的驱动程序来使整部主机开始运行， 等到核心侦测硬件与加载驱动程序完毕后，一个最简单的操作系统就开始在 PC 上运行了。  
主机系统开始运行后，此时 Linux 才会呼叫外部程序开始准备软件运行的环境， 并且实际的加载所有系统运行所需要的软件程序。最后系统会开始等待登陆与操作。
1. 加载 BIOS 的硬件资讯与进行自我测试，并依据配置取得第一个可启动的装置；
2. 读取并运行第一个启动装置内 MBR 的 boot Loader (亦即是 grub, spfdisk 等程序)；
3. 依据 boot loader 的配置加载 Kernel ，Kernel 会开始侦测硬件与加载驱动程序；
4. 在硬件驱动成功后，Kernel 会主动呼叫 init 程序，而 init 会取得 run-level 资讯；
5. init 运行 /etc/rc.d/rc.sysinit 文件来准备软件运行的作业环境 (如网络、时区等)；
6. init 运行 run-level 的各个服务之启动 (script 方式)；
7. init 运行 /etc/rc.d/rc.local 文件；
8. init 运行终端机模拟程序 mingetty 来启动 login 程序，最后就等待使用者登陆。
## BIOS, boot loader 与 kernel 加载
### BIOS，启动自我测试与 MBR
在个人计算机架构下，你想要启动整部系统首先就得要让系统去加载 BIOS (Basic Input Output System)，并透过 BIOS 程序去加载 CMOS 的资讯，并且藉由 CMOS 内的配置值取得主机的各项硬件配置， 例如 CPU 与周边设备的沟通时脉、启动装置的搜寻顺序、硬盘的大小与类型、 系统时间、各周边汇流排的是否启动 Plug and Play (PnP, 随插即用装置) 、 各周边设备的 I/O 位址、以及与 CPU 沟通的 IRQ 岔断等等的资讯。  
在取得这些资讯后，BIOS 还会进行启动自我测试 (Power-on Self Test, POST)。 然后开始运行硬件侦测的初始化，并配置 PnP 装置，之后再定义出可启动的装置顺序， 接下来就会开始进行启动装置的数据读取了 (MBR 相关的任务开始)。  
BIOS 会指定启动的装置好让我们可以读取磁碟中的操作系统核心文件。 但由于不同的操作系统他的文件系统格式不相同，因此我们必须要以一个启动管理程序来处理核心文件加载 (load) 的问题， 这个启动管理程序就被称为 Boot Loader。 Boot Loader 程序就在启动装置的第一个磁区 (sector) 内，也就是 MBR (Master Boot Record, 主要启动记录区)。  
BIOS 是透过硬件的 INT 13 中断功能来读取 MBR 的，只要 BIOS 能够侦测的到你的磁碟 (不论该磁碟是 SATA 还是 IDE 介面)，就可以透过 INT 13 这条通道来读取该磁碟的第一个磁区内的 MBR ， boot loader 就能够被运行。
### Boot Loader 的功能
每个文件系统 (filesystem, 或者是 partition) 都会保留一块启动磁区 (boot sector) 提供操作系统安装 boot loader ， 而通常操作系统默认都会安装一份 loader 到他根目录所在的文件系统的 boot sector 上。  
在 Linux 系统安装时，你可以选择将 boot loader 安装到 MBR 去，也可以选择不安装。 如果选择安装到 MBR 的话，那理论上你在 MBR 与 boot sector 都会保有一份 boot loader 程序的。 而 Windows 安装时，他默认会主动的将 MBR 与 boot sector 都装上一份 boot loader。  
boot loader 的主要功能如下：
* 提供菜单：使用者可以选择不同的启动项目，这也是多重启动的重要功能！
* 加载核心文件：直接指向可启动的程序区段来开始操作系统；
* 转交其他 loader：将启动管理功能转交给其他 loader 负责。

Windows 的 loader 默认不具有控制权转交的功能，因此不能使用 Windows 的 loader 来加载 Linux 的 loader 。所以多重启动时，要先安装 Windows 再装 Linux。  
最终 boot loader 的功能就是『加载 kernel 文件』。
### 加载核心侦测硬件与 initrd 的功能
当我们藉由 boot loader 的管理而开始读取核心文件后，接下来， Linux 就会将核心解压缩到主内存当中， 并且利用核心的功能，开始测试与驱动各个周边装置，包括储存装置、CPU、网络卡、声卡等等。 此时 Linux 核心会以自己的功能重新侦测一次硬件，而不一定会使用 BIOS 侦测到的硬件资讯。也就是说，核心此时才开始接管 BIOS 后的工作了。一般，核心文件放置到 /boot 里面，并且取名为 /boot/vmlinuz。
```
[root@www ~]# ls --format=single-column -F /boot
config-2.6.18-92.el5      <==此版本核心被编译时选择的功能与模块配置档
grub/                     <==就是启动管理程序 grub 相关数据目录
initrd-2.6.18-92.el5.img  <==虚拟文件系统档！
System.map-2.6.18-92.el5  <==核心功能放置到内存位址的对应表
vmlinuz-2.6.18-92.el5     <==就是核心文件
```
为了硬件开发商与其他核心功能开发者的便利， 因此 Linux 核心是可以透过动态加载核心模块的 (就请想成驱动程序即可)，这些核心模块就放置在 /lib/modules/ 目录内。由于模块放置到磁碟根目录内 (要记得 /lib 不可以与 / 分别放在不同的 partition ！)， 因此在启动的过程中核心必须要挂载根目录，这样才能够读取核心模块提供加载驱动程序的功能。而且为了担心影响到磁碟内的文件系统，因此启动过程中根目录是以只读的方式来挂载。  
一般来说，非必要的功能且可以编译成为模块的核心功能，目前的 Linux distributions 都会将他编译成为模块。 因此 U盘, SATA, SCSI... 等磁碟装置的驱动程序通常都是以模块的方式来存在的。   
虚拟文件系统 (Initial RAM Disk) 一般使用的档名为 /boot/initrd ，这个文件的特色是，他也能够透过 boot loader 来加载到内存中， 然后这个文件会被解压缩并且在内存当中模拟成一个根目录， 且此模拟在内存当中的文件系统能够提供一支可运行的程序，透过该程序来加载启动过程中所最需要的核心模块， 通常这些模块就是 U盘, RAID, LVM, SCSI 等文件系统与磁碟介面的驱动程序。等加载完成后， 会帮助核心重新呼叫 /sbin/init 来开始后续的正常启动流程。  
boot loader 可以加载 kernel 与 initrd ，然后在内存中让 initrd 解压缩成为根目录， kernel 就能够藉此加载适当的驱动程序，最终释放虚拟文件系统，并挂载实际的根目录文件系统， 就能够开始后续的正常启动流程。CentOS 5.x的 initrd 文件内容：
```
# 1. 先将 /boot/initrd 复制到 /tmp/initrd 目录中，等待解压缩：
[root@www ~]# mkdir /tmp/initrd
[root@www ~]# cp /boot/initrd-2.6.18-92.el5.img /tmp/initrd/
[root@www ~]# cd /tmp/initrd
[root@www initrd]# file initrd-2.6.18-92.el5.img
initrd-2.6.18-92.el5.img: gzip compressed data, ...
# 原来是 gzip 的压缩档，将扩展名给改成 .gz 

# 2. 将上述的文件解压缩：
[root@www initrd]# mv initrd-2.6.18-92.el5.img initrd-2.6.18-92.el5.gz
[root@www initrd]# gzip -d initrd-2.6.18-92.el5.gz
[root@www initrd]# file initrd-2.6.18-92.el5
initrd-2.6.18-92.el5: ASCII cpio archive (SVR4 with no CRC)
# 是 cpio 的命令压缩成的文件

# 3. 用 cpio 解压缩
[root@www initrd]# cpio -ivcdu < initrd-2.6.18-92.el5
[root@www initrd]# ll
drwx------ 2 root root    4096 Apr 10 02:05 bin
drwx------ 3 root root    4096 Apr 10 02:05 dev
drwx------ 2 root root    4096 Apr 10 02:05 etc
-rwx------ 1 root root    1888 Apr 10 02:05 init
-rw------- 1 root root 5408768 Apr 10 02:00 initrd-2.6.18-92.el5
drwx------ 3 root root    4096 Apr 10 02:05 lib
drwx------ 2 root root    4096 Apr 10 02:05 proc
lrwxrwxrwx 1 root root       3 Apr 10 02:05 sbin -> bin
drwx------ 2 root root    4096 Apr 10 02:05 sys
drwx------ 2 root root    4096 Apr 10 02:05 sysroot
# 跟根目录很相似，也有 init 这个运行档

# 4. 观察 init 文件内较重要的运行项目
[root@www initrd]# cat init
#!/bin/nash                  <==使用类似 bash 的 shell 来运行
mount -t proc /proc /proc    <==挂载内存的虚拟文件系统
....(中间省略)....
echo Creating initial device nodes
mknod /dev/null c 1 3        <==创建系统所需要的各项装置！
....(中间省略)....
echo "Loading ehci-hcd.ko module"
insmod /lib/ehci-hcd.ko      <==加载各项核心模块，就是驱动程序！
....(中间省略)....
echo Creating root device.
mkrootdev -t ext3 -o defaults,ro hdc2 <==尝试挂载根目录
....(底下省略)....
```
需要 initrd 最重要的原因是，当启动时无法挂载根目录的情况下， 此时就一定需要 initrd ，例如你的根目录在特殊的磁碟介面 (U盘, SATA, SCSI) ， 或者是你的文件系统较为特殊 (LVM, RAID) 等等。如果你的 Linux 是安装在 IDE 介面的磁碟上，并且使用默认的 ext2/ext3 文件系统， 那么不需要 initrd 也能够顺利的启动进入 Linux 。  
接下来开始运行系统的第一支程序：/sbin/init
## 第一支程序 init 及配置档 /etc/inittab 与 runlevel
在核心加载完毕、进行完硬件侦测与驱动程序加载后，此时你的主机硬件应该已经准备就绪了 (ready) ， 此时核心会主动的呼叫第一支程序，那就是 /sbin/init ，PID 是 1 。/sbin/init 最主要的功能就是准备软件运行的环境，包括系统的主机名称、网络配置、语系处理、文件系统格式及其他服务的启动等。 而所有的动作都会透过 init 的配置档，亦即是 /etc/inittab 来规划，而 inittab 内还有一个很重要的配置项目，那就是默认的 runlevel (启动运行等级)。
### Run level：运行等级
Linux 就是藉由配置 run level 来规定系统使用不同的服务来启动，让 Linux 的使用环境不同。基本上，依据有无网络与有无 X Window 而将 run level 分为 7 个等级，分别是：
* 0 - halt (系统直接关机)
* 1 - single user mode (单人维护模式，用在系统出问题时的维护)
* 2 - Multi-user, without NFS (类似底下的 runlevel 3，但无 NFS 服务)
* 3 - Full multi-user mode (完整含有网络功能的纯文字模式)
* 4 - unused (系统保留功能)
* 5 - X11 (与 runlevel 3 类似，但加载使用 X Window)
* 6 - reboot (重新启动)

『 不能将默认的 run level 配置为0，4，6 』。通过 /etc/inittab 取得 run level。
### /etc/inittab 的语法与内容
```
[root@www ~]# vim /etc/inittab
id:5:initdefault:                 <==默认的 runlevel 配置, 此 runlevel 为 5 

si::sysinit:/etc/rc.d/rc.sysinit  <==准备系统软件运行的环境的脚本运行档

# 7 个不同 run level 的，需要启动的服务的 scripts 放置路径：
l0:0:wait:/etc/rc.d/rc 0    <==runlevel 0 在 /etc/rc.d/rc0.d/
l1:1:wait:/etc/rc.d/rc 1    <==runlevel 1 在 /etc/rc.d/rc1.d/
l2:2:wait:/etc/rc.d/rc 2    <==runlevel 2 在 /etc/rc.d/rc2.d/
l3:3:wait:/etc/rc.d/rc 3    <==runlevel 3 在 /etc/rc.d/rc3.d/
l4:4:wait:/etc/rc.d/rc 4    <==runlevel 4 在 /etc/rc.d/rc4.d/
l5:5:wait:/etc/rc.d/rc 5    <==runlevel 5 在 /etc/rc.d/rc5.d/
l6:6:wait:/etc/rc.d/rc 6    <==runlevel 6 在 /etc/rc.d/rc6.d/

# 是否允许按下 [ctrl]+[alt]+[del] 就重新启动的配置项目：
ca::ctrlaltdel:/sbin/shutdown -t3 -r now

# 底下两个配置则是关于不断电系统的 (UPS)，一个是没电力时的关机，一个是复电的处理
pf::powerfail:/sbin/shutdown -f -h +2 "Power Failure; System Shutting Down"
pr:12345:powerokwait:/sbin/shutdown -c "Power Restored; Shutdown Cancelled"

1:2345:respawn:/sbin/mingetty tty1  <==其实 tty1~tty6 是由底下这六行决定的。
2:2345:respawn:/sbin/mingetty tty2
3:2345:respawn:/sbin/mingetty tty3
4:2345:respawn:/sbin/mingetty tty4
5:2345:respawn:/sbin/mingetty tty5
6:2345:respawn:/sbin/mingetty tty6

x:5:respawn:/etc/X11/prefdm -nodaemon <==X window 则是这行决定的！
```
```
[配置项目]:[run level]:[init 的动作行为]:[命令项目]
```
1. 配置项目：最多四个字节，代表 init 的主要工作项目，只是一个简单的代表说明
2. run level：该项目在哪些 run level 底下进行的意思。如果是 35 则代表 runlevel 3 与 5 都会运行。
3. init 的动作项目：主要可以进行的动作项目意义有：  
   inittab 配置值|意义说明
   :---:|:---:
   initdefault|代表默认的 run level 配置值
   sysinit|代表系统初始化的动作项目
   ctrlaltdel|代表 [ctrl]+[alt]+[del] 三个按键是否可以重新启动的配置
   wait|代表后面栏位配置的命令项目必须要运行完毕才能继续底下其他的动作
   respawn|代表后面栏位的命令可以无限制的再生 (重新启动)。举例来说， tty1 的 mingetty 产生的可登陆画面， 在你注销而结束后，系统会再开一个新的可登陆画面等待下一个登陆。
   更多配置项目参考 man inittab 的说明
4. 命令项目：亦即应该可以进行的命令，通常是一些 script 。
### init 的处理流程
该文件内容的配置也是一行一行的从上往下处理的，因此可以知道 CentOS 的 init 依据 inittab 配置的处理流程会是：
1. 先取得 runlevel 亦即默认运行等级的相关等级 (以测试机为例，为 5 号)；
2. 使用 /etc/rc.d/rc.sysinit 进行系统初始化
3. 由于 runlevel 是 5 ，因此只进行『l5:5:wait:/etc/rc.d/rc 5』，其他行则略过
4. 配置好 [ctrl]+[alt]+[del] 这组的组合键功能
5. 配置不断电系统的 pf, pr 两种机制；
6. 启动 mingetty 的六个终端机 (tty1 ~ tty6)
7. 最终以 /etc/X11/perfdm -nodaemon 启动图形介面

由于整个配置都是依据 /etc/inittab 来决定的，因此如果你想要修改任何细节的话， 可以这样做：
* 如果不想让使用者利用 [crtl]+[alt]+[del] 来重新启动系统，可以将『 ca::ctrlaltdel:/sbin/shutdown -t3 -r now 』加上注解 (#) 来取消该配置
* 规定启动的默认 run level 是纯文字的 3 号或者是具有图形介面的 5 号 ，可经由 『 id:5:initdefault: 』那个数字来决定。
* 如果不想要启动六个终端机 (tty1~tty6)，那么可以将『 6:2345:respawn:/sbin/mingetty tty6』关闭数个。但务必至少启动一个。
## init 处理系统初始化流程 (/etc/rc.d/rc.sysinit)
/etc/inittab 里的 『 si::sysinit:/etc/rc.d/rc.sysinit 』表示，『开始加载各项系统服务之前，得先做好整个系统环境，主要利用 /etc/rc.d/rc.sysinit 这个 shell script 来配置好系统环境。』  
/etc/rc.d/rc.sysinit 的工作：
1. 取得网络环境与主机类型：  
   读取网络配置档 /etc/sysconfig/network ，取得主机名称与默认通讯闸 (gateway) 等网络环境。
2. 测试与挂载内存装置 /proc 及 U盘 装置 /sys：  
   除挂载内存装置 /proc 之外，还会主动侦测系统上是否具有 usb 的装置， 若有则会主动加载 usb 的驱动程序，并且尝试挂载 usb 的文件系统。
3. 决定是否启动 SELinux ：  
   我们在第十七章谈到的 SELinux 在此时进行一些检测， 并且检测是否需要帮所有的文件重新编写标准的 SELinux 类型 (auto relabel)。
4. 启动系统的乱数产生器  
   乱数产生器可以帮助系统进行一些口令加密演算的功能，在此需要启动两次乱数产生器。
5. 配置终端机 (console) 字形：  
6. 配置显示于启动过程中的欢迎画面 (text banner)；
7. 配置系统时间 (clock) 与时区配置：需读入 /etc/sysconfig/clock 配置值  
8. 周边设备的侦测与 Plug and Play (PnP) 参数的测试：  
   根据核心在启动时侦测的结果 (/proc/sys/kernel/modprobe ) 开始进行 ide / scsi / 网络 / 音效 等周边设备的侦测，以及利用以加载的核心模块进行 PnP 装置的参数测试。
9. 使用者自订模块的加载  
   使用者可以在 /etc/sysconfig/modules/*.modules 加入自订的模块，则此时会被加载到系统当中
10. 加载核心的相关配置：  
    系统会主动去读取 /etc/sysctl.conf 这个文件的配置值，使核心功能成为我们想要的样子。
11. 配置主机名称与初始化电源管理模块 (ACPI)
12. 初始化软件磁盘阵列：主要是透过 /etc/mdadm.conf 来配置好的。
13. 初始化 LVM 的文件系统功能
14. 以 fsck 检验磁碟文件系统：会进行 filesystem check
15. 进行磁碟配额 quota 的转换 (非必要)：
16. 重新以可读写模式挂载系统磁碟：
17. 启动 quota 功能：所以我们不需要自订 quotaon 的动作
18. 启动系统虚拟乱数产生器 (pseudo-random)：
19. 清除启动过程当中的缓存文件：
20. 将启动相关资讯加载 /var/log/dmesg 文件中。

『 dmesg 』可以显示记录的启动过程中的信息。在 /etc/rc.d/rc.sysinit 中进行的很多工作的默认配置档，都在 /etc/sysconfig/ 中。  
在 CentOS 当中，如果我们想要加载核心模块的话， 可以将整个模块写入到 /etc/sysconfig/modules/*.modules 当中，在该目录下， 只要记得档名最后是以 .modules 结尾即可。
## 启动系统服务与相关启动配置档 (/etc/rc.d/rc N & /etc/sysconfig)
依据我们在 /etc/inittab 里面提到的 run level 配置值，就可以来决定启动的服务项目。
```
l0:0:wait:/etc/rc.d/rc 0
l1:1:wait:/etc/rc.d/rc 1
l2:2:wait:/etc/rc.d/rc 2
l3:3:wait:/etc/rc.d/rc 3
l4:4:wait:/etc/rc.d/rc 4
l5:5:wait:/etc/rc.d/rc 5  <==本例中，以此项目来解释
l6:6:wait:/etc/rc.d/rc 6
```
主要是透过 /etc/rc.d/rc 这个命令来处理相关任务：
* 透过外部第一号参数 ($1) 来取得想要运行的脚本目录。亦即由 /etc/rc.d/rc 5 可以取得 /etc/rc5.d/ 这个目录来准备处理相关的脚本程序；
* 找到 /etc/rc5.d/K??* 开头的文件，并进行『 /etc/rc5.d/K??* stop 』的动作；
* 找到 /etc/rc5.d/S??* 开头的文件，并进行『 /etc/rc5.d/S??* start 』的动作；
```
[root@www ~]# ll /etc/rc5.d/
lrwxrwxrwx 1 root root 16 Sep  4  2008 K02dhcdbd -> ../init.d/dhcdbd
....(中间省略)....
lrwxrwxrwx 1 root root 14 Sep  4  2008 K91capi -> ../init.d/capi
lrwxrwxrwx 1 root root 23 Sep  4  2008 S00microcode_ctl -> ../init.d/microcode_ctl
lrwxrwxrwx 1 root root 22 Sep  4  2008 S02lvm2-monitor -> ../init.d/lvm2-monitor
....(中间省略)....
lrwxrwxrwx 1 root root 17 Sep  4  2008 S10network -> ../init.d/network
....(中间省略)....
lrwxrwxrwx 1 root root 11 Sep  4  2008 S99local -> ../rc.local
lrwxrwxrwx 1 root root 16 Sep  4  2008 S99smartd -> ../init.d/smartd
....(底下省略)....
```
* 档名全部以 Sxx 或 Kxx ，其中 xx 为数字，且这些数字在文件之间是有相关性的！
* 全部是连结档，连结到 stand alone 服务启动的目录 /etc/init.d/ 去
  ```
  /etc/rc5.d/K91capi stop --> /etc/init.d/capi stop
  /etc/rc5.d/S10network start --> /etc/init.d/network start
  ```
所以说，你有想要启动该 runlevel 时就运行的服务，那么利用 Sxx 并指向 /etc/init.d/ 的特定服务启动脚本后， 该服务就能够在启动时启动。且不需要自行处理 K, S 开头的连结档，使用第十八章提到的 chkconfig 就是负责处理这个连结档。  
S 或 K 后面接的数字，就是运行的顺序。
## 使用者自订启动启动程序 (/etc/rc.d/rc.local)
在完成默认 runlevel 指定的各项服务的启动后，如果我还有其他的动作想要完成时，/etc/rc.d/rc.local 用来运行自己想要运行的系统命令。
## 根据 /etc/inittab 之配置，加载终端机或 X-Window 界面
```
1:2345:respawn:/sbin/mingetty tty1
2:2345:respawn:/sbin/mingetty tty2
3:2345:respawn:/sbin/mingetty tty3
4:2345:respawn:/sbin/mingetty tty4
5:2345:respawn:/sbin/mingetty tty5
6:2345:respawn:/sbin/mingetty tty6
x:5:respawn:/etc/X11/prefdm -nodaemon
```
这一段代表，在 run level 2, 3, 4, 5 时，都会运行 /sbin/mingetty，而且运行六个，这也是为何我们 Linux 会提供『六个纯文字终端机』的配置。mingetty 就是在启动终端机的命令。  
要注意的是那个 respawn 的 init 动作项目，他代表『当后面的命令被终止 (terminal) 时， init 会主动的重新启动该项目。』这也是为何我们登陆 tty1 终端机介面后，以 exit 离开后， 系统还是会重新显示等待使用者输入的画面的原因。  
可以通过注释掉该行，将此终端界面取消。
## 启动过程会用到的主要配置档
启动过程会用到的配置档则大多放置在 /etc/sysconfig/ 目录下。同时，由于核心还是需要加载一些驱动程序 (核心模块)，此时系统自订的装置与模块对应档 (/etc/modprobe.conf) 就很重要了。
### 关于模块：/etc/modprobe.conf
加载使用者自订模块，是在 /etc/sysconfig/modules/ 目录下。虽然核心提供的默认模块已经很足够我们使用了，但是，某些条件下我们还是得对模块进行一些参数的规划， 此时就得要使用到 /etc/modprobe.conf 。
```
[root@www ~]# cat /etc/modprobe.conf
alias eth0 8139too               <==让 eth0 使用 8139too 的模块
alias scsi_hostadapter pata_sis
alias snd-card-0 snd-trident
options snd-card-0 index=0       <==额外指定 snd-card-0 的参数功能
options snd-trident index=0
```
这个文件大多在指定系统内的硬件所使用的模块，这个文件通常系统是可以自行产生，不必手动订正。不过，如果系统捉到错误的驱动程序，或者是你想要使用升级的驱动程序来对应相关的硬件配备时， 你就得要自行手动的处理一下这个文件了。更多说明参考 man modprobe.conf 。
### /etc/sysconfig/*
* authconfig:  
  这个文件主要在规范使用者的身份认证的机制，包括是否使用本机的 /etc/passwd, /etc/shadow 等， 以及 /etc/shadow 口令记录使用何种加密演算法，还有是否使用外部口令服务器提供的帐号验证 (NIS, LDAP) 等。 系统默认使用 MD5 加密演算法，并且不使用外部的身份验证机制；
* clock:  
  此文件在配置 Linux 主机的时区，可以使用格林威治时间(GMT)，也可以使用本地时间 (local)。基本上，在 clock 文件内的配置项目『 ZONE 』所参考的时区位于 /usr/share/zoneinfo 目录下的相对路径中。而且要修改时区的话，还得将 /usr/share/zoneinfo/Asia/Taipei 这个文件复制成为 /etc/localtime 才行。
* il8n:  
  i18n 在配置一些语系的使用方面，例如最麻烦的文字介面下的日期显示问题！ 如果你是以中文安装的，那么默认语系会被选择 zh_TW.UTF8 ，所以在纯文字介面之下， 你的文件日期显示可能就会呈现乱码！这个时候就需要更改一下这里。更动这个 i18n 的文件，将里面的 LC_TIME 改成 en 即可
* keyboard & mouse:
* netword:  
  network 可以配置是否要启动网络，以及配置主机名称还有通讯闸 (GATEWAY) 这两个重要资讯
* network-scripts/:  
  network-scripts 里面的文件，则是主要用在配置网络卡
## Run level 的切换
事实上，与 run level 有关的启动其实是在 /etc/rc.d/rc.sysinit 运行完毕之后。也就是说，其实 run level 的不同仅是 /etc/rc[0-6].d 里面启动的服务不同而已。不过，依据启动是否自动进入不同 run level 的配置，我们可以说：
1. 要每次启动都运行某个默认的 run level ，则需要修改 /etc/inittab 内的配置项目， 亦即是『 id:5:initdefault: 』里头的数字
2. 如果仅只是暂时变更系统的 run level 时，则使用 init [0-6] 来进行 run level 的变更。 但下次重新启动时，依旧会是以 /etc/inittab 的配置为准。

当运行 init 3 时，系统会：
* 先比对 /etc/rc3.d/ 及 /etc/rc5.d 内的 K 与 S 开头的文件；
* 在新的 runlevel 亦即是 /etc/rc3.d/ 内有多的 K 开头文件，则予以关闭；
* 在新的 runlevel 亦即是 /etc/rc3.d/ 内有多的 S 开头文件，则予以启动；

两个 Run level 都存在的服务不会被关闭。如此一来，就很容易切换 run level 了， 而且还不需要重新启动。通过如下方式，获取目前 run level.
```
[root@www ~]# runlevel
N 5
# 左边代表前一个 runlevel ，右边代表目前的 runlevel。
# 由于之前并没有切换过 runlevel ，因此前一个 runlevel 不存在 (N)
```
利用『 init 0 』就能够关机， 而『 init 6 』就能够重新启动。

# 核心与核心模块
目前的核心都是具有『可读取模块化驱动程序』的功能， 亦即是所谓的『 modules (模块化)』功能。所谓的模块化可以将他想成是一个『外挂程序』， 该外挂程序可能由硬件开发厂商提供，也有可能我们的核心本来就支持，较新的硬件，通常都需要硬件开发商提供驱动程序模块。
* 核心： /boot/vmlinuz 或 /boot/vmlinuz-version；
* 核心解压缩所需 RAM Disk： /boot/initrd (/boot/initrd-version)；
* 核心模块： /lib/modules/version/kernel 或 /lib/modules/$(uname -r)/kernel；
* 核心原始码： /usr/src/linux 或 /usr/src/kernels/ (要安装才会有，默认不安装)

如果核心被顺利的加载到系统中，会有以下信息记录下来：
* 核心版本：/proc/version
* 系统核心功能：/proc/sys/kernel

加入新的硬件：
* 重新编译核心，并加入最新的硬件驱动程序原始码；
* 将该硬件的驱动程序编译成为模块，在启动时加载该模块
## 核心模块与相依性
基本上，核心模块的放置处是在 /lib/modules/$(uname -r)/kernel 当中，里面主要还分成几个目录：
```
arch	：与硬件平台有关的项目，例如 CPU 的等级等等；
crypto	：核心所支持的加密的技术，例如 md5 或者是 des 等等；
drivers	：一些硬件的驱动程序，例如显卡、网络卡、PCI 相关硬件等等；
fs	：核心所支持的 filesystems ，例如 vfat, reiserfs, nfs 等等；
lib	：一些函式库；
net	：与网络有关的各项协议数据，还有防火墙模块 (net/ipv4/netfilter/*) 等等；
sound	：与音效有关的各项模块；
```
Linux 当然会提供一些模块相依性的解决方案，就是检查 /lib/modules/$(uname -r)/modules.dep 这个文件，其记录了在核心支持的模块的各项相依性。  
利用 depmod 命令可以创建该文件：
```
[root@www ~]# depmod [-Ane]
选项与参数：
-A  ：不加任何参数时， depmod 会主动的去分析目前核心的模块，并且重新写入
      /lib/modules/$(uname -r)/modules.dep 当中。若加入 -A 参数时，则 depmod
      会去搜寻比 modules.dep 内还要新的模块，如果真找到新模块，才会升级。
-n  ：不写入 modules.dep ，而是将结果输出到屏幕上(standard out)；
-e  ：显示出目前已加载的不可运行的模块名称

[root@www ~]# cp a.ko /lib/modules/$(uname -r)/kernel/drivers/net
[root@www ~]# depmod
```
Linux kernel 2.6.x 版本的核心模块扩展名一定是 .ko 结尾的， 当使用 depmod 之后，该程序会移动到模块标准放置目录 /lib/modules/$(uname -r)/kernel ， 并依据相关目录的定义将全部的模块捉出来分析，最终才将分析的结果写入 modules.dep 文件中。这个文件很重要，并且会影响到本章稍后会介绍的 modprobe 命令的应用。
## 核心模块的观察
```
[root@www ~]# lsmod
Module                  Size  Used by
autofs4                24517  2
hidp                   23105  2
....(中间省略)....
8139too                28737  0
8139cp                 26305  0
mii                     9409  2 8139too,8139cp <==mii 还被 8139cp, 8139too 使用
....(中间省略)....
uhci_hcd               25421  0  <==底下三个是 U盘 相关的模块！
ohci_hcd               23261  0
ehci_hcd               33357  0
```
显示内容包括：
* 模块名称(Module)；
* 模块的大小(size)；
* 此模块是否被其他模块所使用 (Used by)。

也就是说，模块具有相依性。例如，mii 这个模块会被 8139too 所使用，『当要加载 8139too 时，需要先加载 mii 这个模块才可以顺利的加载 8139too』。使用 modinfo 命令，观察每个模块的信息：
```
[root@www ~]# modinfo [-adln] [module_name|filename]
选项与参数：
-a  ：仅列出作者名称；
-d  ：仅列出该 modules 的说明 (description)；
-l  ：仅列出授权 (license)；
-n  ：仅列出该模块的详细路径。
```
modinfo 除了可以『查阅在核心内的模块』之外，还可以检查『某个模块文件』， 因此，如果想要知道某个文件代表的意义为何，可以利用 modinfo 加上完整档名观察。
## 核心模块的加载与移除
手动加载模块，最简单的方式是使用 modprobe 命令加载模块，modprobe 会主动的去搜寻 modules.dep 的内容，先克服了模块的相依性后， 才决定需要加载的模块有哪些。而 insmod 则完全由使用者自行加载一个完整档名的模块，不会主动分析模块相依性。
```
[root@www ~]# insmod [/full/path/module_name] [parameters]

[root@www ~]# insmod /lib/modules/$(uname -r)/kernel/fs/cifs/cifs.ko
[root@www ~]# lsmod | grep cifs
cifs                  212789  0
```
insmod 后面接的模块必须是完整的【档名】。若要移除这个模块：
```
[root@www ~]# rmmod [-fw] module_name
选项与参数：
-f  ：强制将该模块移除掉，不论是否正被使用；
-w  ：若该模块正被使用，则 rmmod 会等待该模块被使用完毕后，才移除他！
```
使用 insmod 与 rmmod 的问题就是，你必须要自行找到模块的完整档名才行，万一模块有相依属性的问题时，你将无法直接加载或移除该模块。更建议使用 modeprobe 来处理加载模块的问题。用法如下：
```
[root@www ~]# modprobe [-lcfr] module_name
选项与参数：
-c  ：列出目前系统所有的模块！(更详细的代号对应表)
-l  ：列出目前在 /lib/modules/`uname -r`/kernel 当中的所有模块完整档名；
-f  ：强制加载该模块；
-r  ：类似 rmmod ，就是移除某个模块

[root@www ~]# modprobe cifs
# 不需要知道完整的模块档名，这是因为该完整档名已经记录到 /lib/modules/`uname -r`/modules.dep 当中
[root@www ~]# modprobe -r cifs
```
## 核心模块的额外参数配置：/etc/modprobe.conf
如果想要修改某些模块的额外参数配置， 就在这个文件内配置。假设我的网络卡 eth0 是使用 ne ， 但是 eth1 同样也使用 ne ，为了避免同一个模块会导致网络卡的错乱， 因此，我可以先找到 eth0 与 eth1 的 I/O 与 IRQ ，假设：
* eth0: I/O (0x300) 且 IRQ=5
* eth1: I/O (0x320) 且 IRQ=7
```
[root@www ~]# vi /etc/modprobe.conf
alias eth0 ne
alias eth1 ne
options eth0 io=0x300 irq=5
options eth1 io=0x320 irq=7
```

# Boot Loader：Grub
## boot loader 的两个 stage
第一个启动装置的 MBR 的 boot loader 可以具有菜单功能、直接加载核心文件以及控制权移交的功能等。  
Linux 将 boot loader 的程序码运行与配置值加载分成两个阶段 (stage) 来运行：
* Stage 1：运行 boot loader 主程序：  
  第一阶段为运行 boot loader 的主程序，这个主程序必须要被安装在启动区，亦即是 MBR 或者是 boot sector 。但如前所述，因为 MBR 实在太小了，所以，MBR 或 boot sector 通常仅安装 boot loader 的最小主程序， 并没有安装 loader 的相关配置档；
* Stage 2：主程序加载配置档：  
  第二阶段为透过 boot loader 加载所有配置档与相关的环境参数文件 (包括文件系统定义与主要配置档 menu.lst)， 一般来说，配置档都在 /boot 底下。

与 grub 有关的文件都放置在 /boot/grub 中：
```
[root@www ~]# ls -l /boot/grub
-rw-r--r--  device.map              <==grub 的装置对应档(底下会谈到)
-rw-r--r--  e2fs_stage1_5           <==ext2/ext3 文件系统之定义档
-rw-r--r--  fat_stage1_5            <==FAT 文件系统之定义档
-rw-r--r--  ffs_stage1_5            <==FFS 文件系统之定义档
-rw-------  grub.conf               <==grub 在 Red Hat 的配置档
-rw-r--r--  iso9660_stage1_5        <==光驱文件系统定义档
-rw-r--r--  jfs_stage1_5            <==jfs 文件系统定义档
lrwxrwxrwx  menu.lst -> ./grub.conf <==其实 menu.lst 才是配置档！
-rw-r--r--  minix_stage1_5          <==minix 文件系统定义档
-rw-r--r--  reiserfs_stage1_5       <==reiserfs 文件系统定义档
-rw-r--r--  splash.xpm.gz           <==启动时在 grub 底下的背景图示
-rw-r--r--  stage1                  <==stage 1 的相关说明
-rw-r--r--  stage2                  <==stage 2 的相关说明
-rw-r--r--  ufs2_stage1_5           <==UFS 的文件系统定义档
-rw-r--r--  vstafs_stage1_5         <==vstafs 文件系统定义档
-rw-r--r--  xfs_stage1_5            <==xfs 文件系统定义档
```
/boot/grub/ 目录下最重要的就是配置档 (menu.lst) 以及各种文件系统的定义，loader 读取了这种文件系统定义数据后，就能够认识文件系统并读取在该文件系统内的核心文件。grub 的配置档档名，其实应该是 menu.lst ，只是在 Red Hat 里面被定义成为 /boot/grub.conf 而已。
## grub 的配置档 /boot/grub/menu.list 与菜单类型
grub 是目前使用最广泛的 Linux 启动管理程序。
* 认识与支持较多的文件系统，并且可以使用 grub 的主程序直接在文件系统中搜寻核心档名；
* 启动的时候，可以『自行编辑与修改启动配置项目』，类似 bash 的命令模式；
* 可以动态搜寻配置档，而不需要在修改配置档后重新安装 grub 。亦即是我们只要修改完 /boot/grub/menu.lst 里头的配置后，下次启动就生效了。因为 Stage 1, Stage 2 分别安装在 MBR (主程序) 与文件系统当中 (配置档与定义档) 的原因。
### 硬盘与分割槽在 grub 中的代号
安装在 MBR 的 grub 主程序，最重要的任务之一就是从磁碟当中加载核心文件， 以让核心能够顺利的驱动整个系统的硬件。因此 grub 必须能识别硬盘。grub 对硬盘的代号配置与传统的 Linux 磁碟代号完全不同。grub 对硬盘的识别使用的是如下的代号： `(hd0,0)`。
* 硬盘代号以小括号 ( ) 包起来；
* 硬盘以 hd 表示，后面会接一组数字；
* 以『搜寻顺序』做为硬盘的编号，而不是依照硬盘排线的排序！(这个重要！)
* 第一个搜寻到的硬盘为 0 号，第二个为 1 号，以此类推；
* 每颗硬盘的第一个 partition 代号为 0 ，依序类推。

所以说，第一颗『搜寻到的硬盘』代号为：『(hd0)』，而该颗硬盘的第一号分割槽为『(hd0,0)』。所以，整个硬盘代号为：
硬盘搜寻顺序|在 Grub 当中的代号
:---:|:---:
第一颗|(hd0) (hd0,0) (hd0,1) (hd0,4)....
第二颗|(hd1) (hd1,0) (hd1,1) (hd1,4)....
第三颗|(hd2) (hd2,0) (hd2,1) (hd2,4)....
第一颗硬盘的 MBR 安装处的硬盘代号就是『(hd0)』， 而第一颗硬盘的第一个分割槽的 boot sector 代号就是『(hd0,0)』，第一颗硬盘的第一个逻辑分割槽的 boot sector 代号为『(hd0,4)』。
### /boot/grub/menu.lst 的配置档：
```
[root@www ~]# vim /boot/grub/menu.lst
default=0     <==默认启动选项，使用第 1 个启动菜单 (title)
timeout=5     <==若 5 秒内未动键盘，使用默认菜单启动
splashimage=(hd0,0)/grub/splash.xpm.gz <==背景图示所在的文件
hiddenmenu    <==读秒期间是否显示出完整的菜单画面(默认隐藏)
title CentOS (2.6.18-92.el5)    <==第一个菜单的内容
        root (hd0,0)
        kernel /vmlinuz-2.6.18-92.el5 ro root=LABEL=/1 rhgb quiet
        initrd /initrd-2.6.18-92.el5.img
```
在 title 以前的四行，都是属于 grub 的整体配置，包括默认的等待时间与默认的启动项目， 还有显示的画面特性等等。至于 title 后面才是指定启动的核心文件或者是 boot loader 控制权。 在整体配置方面的项目主要常见的有：
* default=0  
  这个必须要与 title 作为对照，在配置档里面有几个 title ，启动的时候就会有几个菜单可以选择。 由于 grub 启始号码为 0 号，因此 default=0 代表使用『第一个 title 项目』来启动的意思。 default 的意思是，如果在读秒时间结束前都没有动到键盘， grub 默认使用此 title 项目 (在此为 0 号) 来启动。
* timeout=5  
  启动时会进行读秒，如果在 5 秒钟内没有按下任何按键，就会使用上面提到的 default 后面接的那个 title 项目来启动的意思。如果你觉得 5 秒太短，那可以将这个数值调大 (例如 30 秒) 即可。此外，如果 timeout=0 代表直接使用 default 值进行启动而不读秒，timeout=-1 则代表直接进入菜单不读秒了！
* splashimage=(hd0,0)/grub/splash.xpm.gz  
  CentOS 在启动的时候背景不是黑白而是有色彩变化的，就是这个文件提供的背景图示。不过这个文件的实际路径写法的意思是：在 (hd0,0)	这个分割槽内的最顶层目录中，底下的 grub/splash.xpm.gz 那个文件的意思。 由于示例将 /boot 这个目录独立成为 /dev/hda1 ，因此这边就会写成『在 /dev/hda1 里面的 grub/splash.xpm.gz 』的意思。
* hiddenmenu  
  这个说的是，启动时是否要显示菜单？目前 CentOS 默认是不要显示菜单， 如果您想要显示菜单，那就将这个配置值注解掉！

底下那个 title 则是显示启动的配置项目。启动时可以选择 (1)直接指定核心文件启动或 (2)将 boot loader 控制权转移到下个 loader (此过程称为 chain-loader)。每个 title 后面接的是『该启动项目名称的显示』，亦即是在菜单出现时，菜单上面的名称而已。
1. 直接指定核心启动  
   要找到核心文件。此外，有可能还需要用到 initrd 的 RAM Disk 配置档。但是如前说的， 尚未启动完成，所以我们必须要以 grub 的硬盘识别方式找出完整的 kernel 与 initrd 档名才行。因此，我们可能需要有底下的方式来配置：
   ```
   1. 先指定核心文件放置的 partition，再读取文件 (目录树)，最后才加入文件的实际档名与路径 (kernel 与 initrd)；
   root    (hd0,0)          <==代表核心文件放在那个 partition 当中
   kernel  /vmlinuz-2.6.18-92.el5 ro root=LABEL=/1 rhgb quiet
   initrd  /initrd-2.6.18-92.el5.img
   ```
   上面的 root, kernel, initrd 后面接的参数的意义说明如下：  
     * root ：代表的是『核心文件放置的那个 partition 而不是根目录』 以案例来说，我的根目录为 /dev/hda2 而 /boot 独立为 /dev/hda1 ，因为与 /boot 有关， 所以磁碟代号就会成为 (hd0,0)。     
     * kernel ：至于 kernel 后面接的则是核心的档名，而在档名后面接的则是核心的参数。 由于启动过程中需要挂载根目录，因此 kernel 后面接的那个 root=LABEL=/1 指的是『Linux 的根目录在哪个 partition 』的意思。这里使用 LABEL 来挂载根目录。至于 rhgb 为色彩显示而 quiet 则是安静模式 (屏幕不会输出核心侦测的资讯)。 
     * initrd ：就是前面提到的 initrd 制作出 RAM Disk 的文件档名
   ```
   2. 直接指定 partition 与档名，不需要额外指定核心文件所在装置代号
   kernel  (hd0,0)/vmlinuz-2.6.18-92.el5 ro root=LABEL=/1 rhgb quiet
   initrd  (hd0,0)/initrd-2.6.18-92.el5.img
   ```
2. 利用 chain loader 的方式转交控制权  
   所谓的 chain loader (启动管理程序的链结) 仅是在将控制权交给下一个 boot loader 而已， 所以 grub 并不需要认识与找出 kernel 的档名 ，『 他只是将 boot 的控制权交给下一个 boot sector 或 MBR 内的 boot loader 而已 』 所以通常他也不需要去查验下一个 boot loader 的文件系统。  
   一般来说， chain loader 的配置只要两个就够了，一个是预计要前往的 boot sector 所在的分割槽代号， 另一个则是配置 chainloader 在那个分割槽的 boot sector (第一个磁区) 上。假设我的 Windows 分割槽在 /dev/hda1 ，且我又只有一颗硬盘，那么要 grub 将控制权交给 windows 的 loader 只要这样就够了：
   ```
   [root@www ~]# vi /boot/grub/menu.lst
   ....前略....
   title Windows partition
	   root (hd0,0)    <==配置使用此分割槽
	   chainloader +1  <== +1 可以想成第一个磁区，亦即是 boot sector
   ```
   不过，由于 Windows 的启动碟需要配置为活化 (active) 状态，且我们的 grub 默认会去检验该分割槽的文件系统。 因此我们可以重新将上面的范例改写成这样：
   ```
   [root@www ~]# vi /boot/grub/menu.lst
   ....前略....
   title Windows partition
	   rootnoverify (hd0,0)   <==不检验此分割槽
	   chainloader +1
	   makeactive             <==配置此分割槽为启动碟(active)
   ```
   grub 的功能还不止此，他还能够隐藏某些分割槽。举例来说，我的 /dev/hda5 是安装 Linux 的分割槽， 我不想让 Windows 能够认识这个分割槽时，你可以这样做：
   ```
   [root@www ~]# vi /boot/grub/menu.lst
   ....前略....
   title Windows partition
	   hide (hd0,4)           <==隐藏 (hd0,4) 这个分割槽
	   rootnoverify (hd0,0)
	   chainloader +1
	   makeactive
   ```
## initrd 的重要性与创建新 initrd 文件
initrd 的目的在于提供启动过程中所需要的最重要核心模块，以让系统启动过程可以顺利完成。会需要 initrd 的原因，是因为核心模块放置于 /lib/modules/$(uname -r)/kernel/ 当中， 这些模块必须要根目录 (/) 被挂载时才能够被读取。但是如果核心本身不具备磁碟的驱动程序时， 当然无法挂载根目录，也就没有办法取得驱动程序。  
initrd 可以将 /lib/modules/.... 内的『启动过程当中一定需要的模块』包成一个文件 (档名就是 initrd)， 然后在启动时透过主机的 INT 13 硬件功能将该文件读出来解压缩，并且 initrd 在内存内会模拟成为根目录， 由于此虚拟文件系统 (Initial RAM Disk) 主要包含磁碟与文件系统的模块，因此我们的核心最后就能够认识实际的磁碟， 就能够进行实际根目录的挂载。所以说：『initrd 内所包含的模块大多是与启动过程有关，而主要以文件系统及硬盘模块 (如 usb, SCSI 等) 为主』。  
一般来说，需要 initrd 的时刻为：
* 根目录所在磁碟为 SATA、U盘 或 SCSI 等连接介面；
* 根目录所在文件系统为 LVM, RAID 等特殊格式；
* 根目录所在文件系统为非传统 Linux 认识的文件系统时；
* 其他必须要在核心加载时提供的模块。

一般来说，各 distribution 提供的核心都会附上 initrd 文件，但如果你有特殊需要所以想重制 initrd 文件的话， 可以使用 mkinitrd 来处理的。
```
[root@www ~]# mkinitrd [-v] [--with=模块名称] initrd档名 核心版本
选项与参数：
-v  ：显示 mkinitrd 的运行过程
--with=模块名称：模块名称指的是模块的名字而已，不需要填写档名。举例来说，
       目前核心版本的 ext3 文件系统模块为底下的档名：
       /lib/modules/$(uname -r)/kernel/fs/ext3/ext3.ko
       那你应该要写成： --with=ext3 就好了 (省略 .ko)
initrd档名：你所要创建的 initrd 档名，尽量取有意义又好记的名字。
核心版本  ：某一个核心的版本，如果是目前的核心则是『 $(uname -r) 』

以 mkinitrd 的默认功能创建一个 initrd 虚拟磁碟文件
[root@www ~]# mkinitrd -v initrd_$(uname -r) $(uname -r)
Creating initramfs
Looking for deps of module ehci-hcd
Looking for deps of module ohci-hcd
....(中间省略)....
Adding module ehci-hcd  <==最终加入 initrd 的就是底下的模块
Adding module ohci-hcd
Adding module uhci-hcd
Adding module jbd
Adding module ext3
Adding module scsi_mod
Adding module sd_mod
Adding module libata
Adding module pata_sis

添加 8139too 这个模块的 initrd 文件
[root@www ~]# mkinitrd -v --with=8139too initrd_vbirdtest $(uname -r)
```
## 测试与安装 grub
安装 grub ，首先，你必须要使用 grub-install 将一些必要的文件复制到 /boot/grub 里面去。
```
[root@www ~]# grub-install [--root-directory=DIR] INSTALL_DEVICE
选项与参数：
--root-directory=DIR 那个 DIR 为实际的目录，使用 grub-install 默认会将
  grub 所有的文件都复制到 /boot/grub/* ，如果想要复制到其他目录与装置去，
  就得要用这个参数。
INSTALL_DEVICE 安装的装置代号

将 grub 安装在目前系统的 MBR 底下，当前系统为 /dev/hda：
[root@www ~]# grub-install /dev/hda

我的 /home 为独立的 /dev/hda3 ，如何安装 grub 到 /dev/hda3 (boot sector)
[root@www ~]# grub-install --root-directory=/home /dev/hda3
Probing devices to guess BIOS drives. This may take a long time.
Installation finished. No error reported.
This is the contents of the device map /home/boot/grub/device.map.
Check if this is correct or not. If any of the lines is incorrect,
fix it and re-run the script `grub-install'.

(fd0)   /dev/fd0
(hd0)   /dev/hda   <==会给予装置代号的对应表！

# 安装完毕之后，并没有生成配置档，需要自己创建。
```
grub-install 是安装 grub 相关的文件 (例如文件系统定义档) 到你的装置上面去等待在启动时被读取，但还需要配置好配置档 (menu.lst) 后，再以 grub shell 来安装 grub 主程序到 MBR 或者是 boot sector 上面。  
grub 配置档的一个示例：
```
[root@www ~]# vim /boot/grub/menu.lst
default=0
timeout=30  # 修改等待时间为 30 s
splashimage=(hd0,0)/grub/splash.xpm.gz
#hiddenmenu  # 不再隐藏菜单
title CentOS (2.6.18-92.el5)
        root (hd0,0)
        kernel /vmlinuz-2.6.18-92.el5 ro root=LABEL=/1 rhgb quiet
        initrd /initrd-2.6.18-92.el5.img
title /dev/hda1 boot sector  <==新增一个转交控制权方式的 loader ，再 /dev/hda1 内
        root (hd0,0)
        chainloader +1
title MBR loader             <==新增的第二个菜单，重新读取 MBR 内的 loader
        root (hd0)           <==MBR 为整颗磁碟的第一个磁区，所以用整颗磁碟的代号
        chainloader +1
title single user mode       <==新增的第三个菜单(其实由原本的title复制来的)
        root (hd0,0)            利用原本的系统核心文件，创建一个可强制进入单人维护模式的菜单。
        kernel /vmlinuz-2.6.18-92.el5 ro root=LABEL=/1 rhgb quiet single
        initrd /initrd-2.6.18-92.el5.img
```
使用 grub shell 将 grub 主程序再次的安装到 /dev/hda1 的 boot sector ，并且重新安装 grub 到 MBR 上面去：具体命令可使用 info grub 查阅
* 用『 root (hdx,x) 』选择含有 grub 目录的那个 partition 代号；
* 用『 find /boot/grub/stage1 』看看能否找到安装资讯文件；
* 用『 find /boot/vmlinuz 』看看能否找到 kernel file (不一定要成功！)；
* 用『 setup (hdx,x) 』或『 setup (hdx) 』将 grub 安装在 boot sector 或 MBR；
* 用『 quit 』来离开 grub shell ！

最需要安装的是 stage1 即 Grub 的主程序，而且配置档通常与主程序放在同一目录下。所以需要用 root (hd0,0) 去找到 /boot/grub/stage1 。使用 grub 进入 grub shell ，进入 grub 后，会出现一个『 grub> 』的提示字节。
```
[root@www ~]# grub

# 1. 先配置一下含有 grub 目录的那个 partition
grub> root (hd0,0)
 Filesystem type is ext2fs, partition type 0x83
# grub 也能够分辨出该分割槽的文件系统 (ext2)。

# 2. 搜寻一下，是否存在 stage1 这个资讯文件？
grub> find /boot/grub/stage1
 (hd0,2)
# 只找到一个，但是我们有 /boot/grub 与 /home/boot/grub 两个 grub 安装
# 因为 /boot 是独立的，因此要找到该档名就得要用如下的方式：

grub> find /grub/stage1
 (hd0,0)
# 这样就能够找到，要特别注意 grub 找到不是目录树，而是装置内的文件。

# 3. 搜寻一下是否可以找到核心？ /boot/vmlinuz-2.6.18-92.el5 ？
grub> find /boot/vmlinuz-2.6.18-92.el5
Error 15: File not found
grub> find /vmlinuz-2.6.18-92.el5
 (hd0,0)
# 再次强调，因为 /boot/ 是独立的

# 4. 将主程序安装上去！安装到 MBR 看看！
grub> setup (hd0)
 Checking if "/boot/grub/stage1" exists... no <==因为 /boot 是独立的
 Checking if "/grub/stage1" exists... yes     <==所以这个档名才是对的！
 Checking if "/grub/stage2" exists... yes
 Checking if "/grub/e2fs_stage1_5" exists... yes
 Running "embed /grub/e2fs_stage1_5 (hd0)"...  15 sectors are embedded.
succeeded
 Running "install /grub/stage1 (hd0) (hd0)1+15 p (hd0,0)/grub/stage2 
/grub/grub.conf"... succeeded  <==将 stage1 程序安装妥当
Done.
# 这样 grub 就在 MBR 当中了

# 5. 那么重复安装到我的 /dev/hda1 呢？亦即是 boot sector 当中？
grub> setup (hd0,0)
 Checking if "/boot/grub/stage1" exists... no
 Checking if "/grub/stage1" exists... yes
 Checking if "/grub/stage2" exists... yes
 Checking if "/grub/e2fs_stage1_5" exists... yes
 Running "embed /grub/e2fs_stage1_5 (hd0,0)"... failed (this is not fatal)
 Running "embed /grub/e2fs_stage1_5 (hd0,0)"... failed (this is not fatal)
 Running "install /grub/stage1 (hd0,0) /grub/stage2 p /grub/grub.conf "... 
succeeded
Done.
# 虽然无法将 stage1_5 安装到 boot sector 去，不过，还不会有问题，
# 重点是最后面那个 stage1 要安装后，显示 succeeded 字样就可以了！

grub> quit
```
读取的会是 (hd0,0) 里面的 /grub/menu.lst 那个文件。
* 如果是从其他 boot loader 转成 grub 时，得先使用 grub-install 安装 grub 配置档；
* 开始编辑 menu.lst 这个重要的配置档；
* 透过 grub 来将主程序安装到系统中，如 MBR 的 (hd0) 或 boot sector 的 (hd0,0) 等等。
## 启动前的额外功能修改
可以查阅相应 title 菜单的内容并加以修改，将光标移动到该菜单，按 `e` 会显示 menu.lst 里配置的该 title 的内容，此时可以进一步修改。
* e：进入 grub shell 的编辑画面；
* o：在光标所在行底下再新增一行；
* d：将光标所在行删除。

可以在相应界面下使用 o, d, e 三个按键编修，正确的核心文件位置，按下 [Enter] 键后，输入 b 来 boot，就可以启动了。可以应对 /boot/grub/menu.lst 配置错误，或者是因为安装的缘故，或者是因为核心文件的缘故，导致无法顺利启动。  
如果 grub 发生错误，无法启动，可以利用具有 grub 启动的 CD 来启动，然后再以 CD 的 grub 的线上编修。
## 关于核心功能当中的 vga 配置
事实上， tty1~tty6 除了 80x24 的解析度外，还能够有其他解析度的支持。但前提是核心必须支持 FRAMEBUFFER_CONSOLE 这个核心功能选项。查阅 /boot/config-2.6.18-92.e15 这个文件，然后搜寻：
```
[root@www ~]# grep 'FRAMEBUFFER_CONSOLE' /boot/config-2.6.18-92.el5
CONFIG_FRAMEBUFFER_CONSOLE=y
# 这个项目如果出现 y 那就是有支持，如果被注解或是 n ，那就是没支持。
```
调整 tty1 ~ tty6 终端机的解析度
彩度\解析度|640x480|800x600|1024x768|1280x1024|bit
:---:|:---:|:---:|:---:|:---:|:---:
256|769|771|773|775|8bit
32768|784|787|790|793|15 bit
65536|785|788|791|794|16 bit
16.8M|786|789|792|795|32 bit
假设你想要将你的终端机萤幕解析度调整到 1024x768 ，且色彩深度为 15bit 色的时候，就得要指定 vga=790 那个数字。
```
[root@www ~]# vim /boot/grub/menu.lst
....(前面省略)....
title CentOS (2.6.18-92.el5)
        root (hd0,0)
        kernel /vmlinuz-2.6.18-92.el5 ro root=LABEL=/1 rhgb quiet vga=790
        initrd /initrd-2.6.18-92.el5.img
....(后面省略)....
```
## BIOS 无法读取大硬盘的问题
现今的硬盘容量越来越大，如果你使用旧的主板来安插大容量硬盘时，可能由于系统 BIOS 或者是其他问题， 导致 BIOS 无法判断该硬盘的容量，此时你的系统读取可能会有问题。  
当进入 Linux 核心功能后，他会主动的再去侦测一下整个系统， 因此 BIOS 捉不到的硬件在 Linux 核心反而可能会可以捉到而正常使用。举例来说，过去很多朋友常常会发现， 『我的系统使用 DVD 启动安装时，可以顺利的安装好 Linux ，但是第一次启动时， 萤幕只出现黑压压的一片，且出现 grub> 的字样，而无法进入 Linux 系统中』，这是由于：
* 在安装的过程中，由于是使用 DVD 或 CD 启动，因此加载 Linux 核心不成问题，而核心会去侦测系统硬件，因此可以捉到 BIOS 捉不到的硬盘，此时你确实可以安装 Linux 在大容量的硬盘上，且不会出现任何问题。
* 但是在进入硬盘启动时，由于 kernel 与 initrd 文件都是透过 BIOS 的 INT 13 通道读取的， 因此你的 kernel 与 initrd 如果放置在 BIOS 无法判断的磁区中，当然就无法被系统加载，而仅会出现 grub shell (grub>) 等待你的处理而已。

解决办法，就是让 kernel 与 initrd 文件放置在大硬盘的最前头，由于 BIOS 至少可以读到大磁碟的 1024 磁柱内的数据，因此就能够读取核心与虚拟文件系统的文件。要让 kernel 与 initrd 放置到整颗硬盘的最前面，就创建 /boot 独立分割槽，并将 /boot 放置到最前面即可。  
万一已经安装了 Linux 且发生了上述问题，可以这样做：
* 直接重装，并且制作出 /boot 挂载的 partition，同时确认该 partition 是在 1024 cylinder 之前才行。
* 利用 grub 的功能，额外创建一个可启动软盘， 或者是直接以光驱启动，然后以 grub 的编写能力进入 Linux 。
* 另外的办法其实是骗过 BIOS ，直接将硬盘的 cylinder, head, sector 等等资讯直接写到 BIOS 当中去，如此一来你的 BIOS 可能就可以读得到与支持的到你的大硬盘了。

建议重新安装，并制作出 /boot 这个 partition。
## 为个别菜单加上口令
通过 grub 提供的 md5 编码，来创建加密的口令：
```
[root@www ~]# grub-md5-crypt
Password: <==输入口令
Retype password: <==再输入一次
$1$kvlI0/$byrbNgkt/.REKPQdfg287. <==这就是产生的 md5 口令！
```
将这个口令复制下来，假设要将第一个选项加入这个口令，而将第四个选项加入另外的口令:
```
[root@www ~]# vim /boot/grub/menu.lst
....(前面省略)....
title CentOS (2.6.18-92.el5)
        password --md5 $1$kvlI0/$byrbNgkt/.REKPQdfg287.
        root (hd0,0)
        kernel /vmlinuz-2.6.18-92.el5 ro root=LABEL=/1 rhgb quiet vga=790
        initrd /initrd-2.6.18-92.el5.img
....(中间省略)....
title single user mode
        password --md5 $1$GFnI0/$UuiZc/7snugLtVN4J/WyM/
        root (hd0,0)
        kernel /vmlinuz-2.6.18-92.el5 ro root=LABEL=/1 rhgb quiet single
        initrd /initrd-2.6.18-92.el5.img
```
一定要输入口令才能进入启动流程，如果想要远程使用 reboot 重新启动，并且主机前面没有任何人，主机不会主动进入启动程序。  
password 这个项目一定要在 title 底下的第一行。 不过，此项功能还是可能被破解的，因为使用者可以透过编辑模式 (e) 进入菜单，并删除口令栏位并按下 b 就能够进行启动流程了。可以通过整体的 password (放在所有的 title 之前) ， 然后在 title 底下的第一行配置 lock ，那使用者想要编辑时，也得要输入口令才行。
```
[root@www ~]# vim /boot/grub/menu.lst
default=0
timeout=30
password --md5 $1$kvlI0/$byrbNgkt/.REKPQdfg287.  <==放在整体配置处
splashimage=(hd0,0)/grub/splash.xpm.gz
#hiddenmenu
title CentOS (2.6.18-92.el5)
        lock  <==多了锁死的功能
        root (hd0,0)
        kernel /vmlinuz-2.6.18-92.el5 ro root=LABEL=/1 rhgb quiet vga=790
        initrd /initrd-2.6.18-92.el5.img
```
由于 2, 3, 4 菜单并没有使用 lock ，因此这三个菜单使用者还是可以运行启动程序， 但是第一个菜单由于有 lock 项目，因此除非你输入正确的口令，否则第一个菜单是无法被加载运行的。 另外，这个项目也能够避免你的 menu.lst 在启动的过程中被乱改，是具有保密 menu.lst 的功能。

# 启动过程的问题解决
很多时候，我们可能因为做了某些配置，或者是因为不正常关机 (例如未经通知的停电等等) 而导致系统的 filesystem 错乱，此时，Linux 可能无法顺利启动成功，可以进入 run level 1 (单人维护模式) 去处理。
## 忘记 root 口令的解决方法
启动流程中，若强制核心进入 runlevel 1 时， 默认是不需要口令即可取得一个 root 的 shell 来救援的。操作如下：
1. 重新启动！一定要重新启动！怎么重开都没关系；
2. 在启动进入 grub 菜单后， (1)在你要进入的菜单上面点 'e' 进入详细配置； (2)将光棒移动到 kernel 上方并点 'e' 进入编辑画面； (3)然后出现如下画面来处理：
   ```
   grub edit> kernel /vmlinuz-2.6.18-92.el5 ro root=LABEL=/ rhgb quiet single
   ```
   按下 [enter] 再按下 b 就能够启动进入单人维护模式了。
3. 进入单人维护模式后，系统会以 root 的权限直接给你一个 shell ，此时你就能够运行『 passwd 』这个命令来重建 root 的口令，然后直接『 init 5 』就可以切换成为 X 窗口介面。
## init 配置档错误
前一个 root 口令挽救的方法其实可以用在很多地方，唯一一个无法挽救的情况，那就是 /etc/inittab 这个文件配置错误导致的无法启动。runlevel 0~6 都会读取 /etc/inittab 配置档， 因此你使用 single mode (runlevel 1) 当然也是要读取 /etc/inittab 来进行启动的。既然默认的 init 无法运行，那我们就告诉核心不要运行 init ，改呼叫 bash 。同样在启动进入 grub 后，同样在 grub edit 的情况下这样做
```
grub edit> kernel /vmlinuz-2.6.18-92.el5 ro root=LABEL=/ rhgb quiet init=/bin/bash
```
因为我们指定了核心呼叫的第一支程序 (init) 变成 /bin/bash，因此 /sbin/init 就不会被运行。 又根据启动流程的说明，我们知道此时虽然可以利用 root 取得 bash 来工作，但此时 (1)除了根目录外，其他的目录都没有被挂载； (2)根目录被挂载成为唯读状态。因此我们还需要进行一些动作才行。  
『 mount -o remount,rw / 』将根目录重新挂载成为可读写，『 mount -a 』则是参考 /etc/fstab 的内容重新挂载文件系统。此时就可以启动进行救援工作了，救援完毕后要使用『 reboot 』重新启动一次。
## BIOS 磁碟对应的问题 (device.map)
由于 grub 对磁碟的装置代号使用的是侦测到的顺序，如果调整了 BIOS 磁碟启动顺序， menu.lst 内的装置代号就可能会对应到错误的磁碟上。  
可以通过 /boot/grub/device.map 这个文件来写死每个装置对 grub 磁碟代号的对应。
```
[root@www ~]# cat /boot/grub/device.map
(fd0)   /dev/fd0
(hd0)   /dev/hda
```
也可以利用 grub-install 的功能。
```
[root@www ~]# grub-install --recheck /dev/hda1

```
这样 device.map 会主动的被升级。
## 因文件系统错误而无法启动
最容易出错的配置而导致无法顺利启动的步骤，通常就是 /etc/fstab 这个文件，尤其是使用者在实作 Quota 时，最容易写错参数， 又没有经过 mount -a 来测试挂载，就立刻直接重新启动的情况。  
请输入 root 的口令来取得 bash 并以 mount -o remount,rw / 将根目录挂载成可读写后，继续处理。造成这种错误的原因除了 /etc/fstab 编辑错误之外，如果你曾经不正常关机后，也可能导致文件系统不一致 (Inconsistent) 的情况， 也有可能会出现相同的问题。如果是磁区错乱的情况 fsck 告知其实是 /dev/md0 出错， 此时你就应该要利用 fsck 去检测 /dev/md0 ，等系统发现错误，并且出现『clear [Y/N]』时，输入『 y 』。  
这个 fsck 的过程可能会很长，而且如果你的 partition 上面的 filesystem 有过多的数据损毁时， 即使 fsck 完成后，可能因为伤到系统槽，导致某些关键系统文件数据的损毁，那么依旧是无法进入 Linux 的。此时，就好就是将系统当中的重要数据复制出来，然后重新安装，并且检验一下， 是否实体硬盘有损伤的现象。不过一般来说，不太可能会这样。常都是 fsck 处理完毕后，就能够顺利再次进入 Linux 了。
## 利用 chroot 切换到另一颗硬盘工作
chroot 命令，『 change root directory 』的意思。可以暂时将根目录移动到某个目录下， 然后去处理某个问题，最后再离开该 root 而回到原本的系统当中。  
可以将你的 Linux 硬盘拔到另一个 Linux 主机上面去，然后用这个 chroot 来切换， 以处理你的硬盘问题。
1. 用尽任何方法，进入一个完整的 Linux 系统 ( run level 3 或 5 )；
2. 假设有问题的 Linux 磁碟在 /dev/hdb1 上面，且他整个系统的排列是：
   挂载点|装置档名
   :---:|:---:
   /|/dev/hdb1
   /var|/dev/hdb2
   /home|/dev/hdb3
   /usr|/dev/hdb5
   若如此的话，再目前这个 Linux 底下，可以创建一个目录，然后可以这样做：
    挂载点|装置档名
   :---:|:---:
   /chroot/|/dev/hdb1
   /chroot/var/|/dev/hdb2
   /chroot/home/|/dev/hdb3
   /chroot/usr/|/dev/hdb5
3. 全部挂载完毕后，再输入『 chroot /chroot 』，根目录 (/) 就变成 /dev/hdb1 的环境了。