# 网络文件系统还是网络驱动器
做为服务器的系统通常是需要储存设备的，而储存设备除了可以使用系统内建的磁盘之外，如果内建的磁盘容量不够大， 而且也没有额外的磁盘插槽 (SATA 或 IDE) 可用时，那么常见解决的方案就是增加 NAS (网络附加储存服务器) 或外接式储存设备。再高档一点的系统，可能就会用到 SAN (储存局域网络) 。  
不论是哪一种架构，基本上，它们的内部硬盘通常是以磁盘阵列 (RAID) 作为基础的。
## NAS 与 SAN
由于企业的数据量越来越大，而且重要性与保密性越来越高，尤其类似数据库的内容，常常容量单位是以 TB (1TB = 1024GB) 在进行计算的。磁盘阵列通常是 (1)主机内部有磁盘阵列控制卡，可以自行管理磁盘阵列。不过想要提供磁盘阵列的容量， 得要透过额外的网络服务才行； (2)外接式磁盘阵列设备，就是单纯的磁盘阵列设备，必须透过某些接口链接到主机上， 主机也要安装适当的驱动程序后，才能捉到这个设备所提供的磁盘容量。
### NAS (Network Attached Storage, 网络附加储存服务器)
基本上，NAS 其实就是一部客制化好的主机了，只要将 NAS 连接上网络，那么在网络上面的其他主机就能够存取 NAS 上头的资料了。简单的说，NAS 就是一部 file server 。过，NAS 由于也是接在网络上面，所以，如果网络上有某个用户大量存取 NAS 上头的数据时，是很容易造成网络停顿的问题。低阶的 NAS 通常会使用 Linux 系统搭配软件磁盘阵列来提供大容量文件系统。此外，NAS 也通常支持 TCP/IP ，并会提供 NFS, SAMBA, FTP 等常见的通讯协议来提供客户端取得文件系统。  
通常 NAS 还会包括很多组态的接口，通常是利用 Web 接口来控制磁盘阵列的设定状况、提供 IP 或其他相关网络设定， 以及是否提供某些特定的服务等等。因为具有较为亲和的操作与控制接口，对于非 IT 的人员来说，控管较为容易。 这也是 NAS 存在的目的。
### SAN (Storage Area Networks, 存储局域网络)
最简单的看法，就是将 SAN 视为一个外接式的储存设备。只是单纯的外接式储存设备仅能透过某些接口 (如 SCSI 或 eSATA) 提供单一部主机使用，而 SAN 却可以透过某些特殊的接口或信道来提供局域网络内的所有机器进行磁盘存取。SAN 是提供『磁盘 (block device)』给主机用，而不是像 NAS 提供的是『网络协议的文件系统 (NFS, SMB...)』。挂载使用 SAN 的主机会多出一个大磁盘，并可针对 SAN 提供的磁盘进行分割与格式化等动作。  
另外，既然 SAN 可以提供磁盘，而 NAS 则是提供相关的网络文件系统，那么 NAS 能不能透过网络去使用 SAN 所提供的磁盘呢？可以。因为 SAN 最大的目的就是在提供磁盘给服务器主机使用，NAS 也是一部完整的服务器， 所以 NAS 当然可以使用 SAN 。同时其他的网络服务器也能够使用这个 SAN 来进行数据存取。  
此外，既然 SAN 开发的目的是要提供大量的磁盘给用户，那么传输的速度当然是非常重要的。因此，早期的 SAN 大多配合光纤信道 (Fibre Channel) 来提供高速的数据传输。
## iSCSI 界面
早期的企业使用的服务器若有大容量磁盘的需求时，通常是透过 SCSI 来串接 SCSI 磁盘，因此服务器上面必须要加装 SCSI 适配卡，而且这个 SCSI 是专属于该服务器的。后来这个外接式的 SCSI 设备被上述提到的 SAN 的架构所取代， 在 SAN 的标准架构下，虽然有很多的服务器可以对同一个 SAN 进行存取的动作，不过为了速度需求，通常使用的是光纤信道。  
iSCSI 主要是透过 TCP/IP 的技术，将储存设备端透过 iSCSI target (iSCSI 目标) 功能，做成可以提供磁盘的服务器端，再透过 iSCSI initiator (iSCSI 初始化用户) 功能，做成能够挂载使用 iSCSI target 的客户端，如此便能透过 iSCSI 协议来进行磁盘的应用了。  
也就是说，iSCSI 这个架构主要将储存装置与使用的主机分为两个部分，分别是：
* iSCSI target：就是储存设备端，存放磁盘或 RAID 的设备，目前也能够将 Linux 主机仿真成 iSCSI target 了！目的在提供其他主机使用的『磁盘』；
* iSCSI initiator：就是能够使用 target 的客户端，通常是服务器。 也就是说，想要连接到 iSCSI target 的服务器，也必须要安装 iSCSI initiator 的相关功能后才能够使用 iSCSI target 提供的磁盘就是了。
## 各组件相关性
一部服务器如何取得磁盘或者是文件系统来利用：
* 直接存取 (direct-attached storage)：例如本机上面的磁盘，就是直接存取设备；
* 透过储存局域网络 (SAN)：来自区网内的其他储存设备提供的磁盘；
* 网络文件系统 (NAS)：来自 NAS 提供的文件系统，只能立即使用，不可进行格式化。

在一般的主机环境下，磁盘装置 (SATA, SAS, FC) 可以透过主机的接口 (DAS) 来直接进行文件系统的建立 (mkfs 进行格式化)，如果想要使用外部的磁盘，那可以透过 SAN (内含多个磁盘的设备)，然后透过 iSCSI 等接口来联机， 当然，还是得要进行格式化等动作 (假设这个 SAN 尚未被使用时)。最后，如果是 NAS 的条件下，那么 NAS 必须要先透过自己的操作系统将磁盘装置进行文件系统的建立后，再以 NFS/CIFS 等方式来提供其他主机挂载使用。  
NAS 可以使用自己的磁盘，也能够透过光纤或以太网络取得 SAN 所提供的磁盘来制作成为网络文件系统，提供其他人的使用。 Server 可以透过 NFS/CIFS 等方式取得 NAS 的文件系统，当然也能够直接存取 SAN 的磁盘。客户端主要则是透过网络文件系统， 并且直接使用 Server 提供的网络资源 (如 FTP, WWW, mail 等等)。

# iSCSI target 的设定
能够完成 iSCSI target/initiator 设定的项目：
* Linux SCSI target framework (tgt)：http://stgt.sourceforge.net/
* Linux-iSCSI Project：http://linux-iscsi.sourceforge.net/
* Open-iSCSI：http://www.open-iscsi.org/

CentOS 6.x 官方直接使用的是 tgt 这个软件
## 所需软件与软件结构
CentOS 将 tgt 的软件名称定义为 scsi-target-utils ，因此你得要使用 yum 去安装他才行。至于用来作为 initiator 的软件则是使用 linux-iscsi 的项目，该项目所提供的软件名称则为 iscsi-initiator-utils 。所以，总的来说，你需要的软件有：
* scsi-target-utils：用来将 Linux 系统仿真成为 iSCSI target 的功能；
* iscsi-initiator-utils：挂载来自 target 的磁盘到 Linux 本机上。

scsi-target-utils 主要提供的档案，基本上有底下几个比较重要需要注意的：
* /etc/tgt/targets.conf：主要配置文件，设定要分享的磁盘格式与哪几颗；
* /usr/sbin/tgt-admin：在线查询、删除 target 等功能的设定工具；
* /usr/sbin/tgt-setup-lun：建立 target 以及设定分享的磁盘与可使用的客户端等工具软件。
* /usr/sbin/tgtadm：手动直接管理的管理员工具 (可使用配置文件取代)；
* /usr/sbin/tgtd：主要提供 iSCSI target 服务的主程序；
* /usr/sbin/tgtimg：建置预计分享的映像文件装置的工具 (以映像文件仿真磁盘)；
## target 的实际设定
iSCSI 就是透过一个网络接口，将既有的磁盘给分享出去，可以分享的磁盘类型：
* 使用 dd 指令所建立的大型档案可供仿真为磁盘 (无须预先格式化)；
* 使用单一分割槽 (partition) 分享为磁盘；
* 使用单一完整的磁盘 (无须预先分割)；
* 使用磁盘阵列分享 (其实与单一磁盘相同方式)；
* 使用软件磁盘阵列 (software raid) 分享成单一磁盘；
* 使用 LVM 的 LV 装置分享为磁盘。

 (1)大型档案； (2)单一分割槽； (3)单一装置 (包括磁盘、数组、软件磁盘阵列、LVM 的 LV 装置文件名等等) 来进行分享。
### 建立所需要的磁盘装置
目前预计准备的磁盘为：
* 建立一个名为 /srv/iscsi/disk1.img 的 500MB 档案；
* 使用 /dev/sda10 提供 2GB 作为分享 (从第一章到目前为止的分割数)；
* 使用 /dev/server/iscsi01 的 2GB LV 作为分享 (再加入 5GB /dev/sda11 到 server VG 中)。
```
# 1. 建立大型档案：
[root@www ~]# mkdir /srv/iscsi
[root@www ~]# dd if=/dev/zero of=/srv/iscsi/disk1.img bs=1M count=500
[root@www ~]# chcon -Rv -t tgtd_var_lib_t /srv/iscsi/
[root@www ~]# ls -lh /srv/iscsi/disk1.img
-rw-r--r--. 1 root root 500M Aug  2 16:22 /srv/iscsi/disk1.img <==容量对的

# 2. 建立实际的 partition 分割：
[root@www ~]# fdisk /dev/sda  <==实际的分割方式自己处理
[root@www ~]# partprobe       <==某些情况下得 reboot 
[root@www ~]# fdisk -l
   Device Boot      Start         End      Blocks   Id  System
/dev/sda10           2202        2463     2104483+  83  Linux
/dev/sda11           2464        3117     5253223+  8e  Linux LVM
# 只有输出 /dev/sda{10,11} 信息，其他的都省略了。注意看容量，上述容量单位 KB

[root@www ~]# swapon -s; mount | grep 'sda1'
# 自己测试一下 /dev/sda{10,11} 不能够被使用。若有被使用，请 umount 或 swapoff

# 3. 建立 LV 装置 ：
[root@www ~]# pvcreate /dev/sda11
[root@www ~]# vgextend server /dev/sda11
[root@www ~]# lvcreate -L 2G -n iscsi01 server
[root@www ~]# lvscan
  ACTIVE            '/dev/server/myhome' [6.88 GiB] inherit
  ACTIVE            '/dev/server/iscsi01' [2.00 GB] inherit
```
### 规划分享的 iSCSI target 档名
iSCSI 有一套自己分享 target 档名的定义，基本上，藉由 iSCSI 分享出来的 target 檔名都是以 iqn 为开头，意思是：『iSCSI Qualified Name (iSCSI 合格名称)』的意思。
```
iqn.yyyy-mm.<reversed domain name>:identifier
iqn.年年-月.单位网域名的反转写法  :这个分享的target名称
```
例如机器是 www.centos.vbird ，反转网域写法为 vbird.centos， 然后，想要的 iSCSI target 名称是 vbirddisk ，那么就可以这样写：
* iqn.2011-08.vbird.centos:vbirddisk

另外，就如同一般外接式储存装置 (target 名称) 可以具有多个磁盘一样，我们的 target 也能够拥有数个磁盘装置的。 每个在同一个 target 上头的磁盘我们可以将它定义为逻辑单位编号 (Logical Unit Number, LUN)。我们的 iSCSI initiator 就是跟 target 协调后才取得 LUN 的存取权。
### 设定 tgt 的配置文件 /etc/tgt/targets.conf
接下来我们要开始来修改配置文件了。基本上，配置文件就是修改 /etc/tgt/targets.conf 。这个档案的内容可以改得很简单， 最重要的就是设定前一点规定的 iqn 名称，以及该名称所对应的装置，然后再给予一些可能会用到的参数而已。
```
[root@www ~]# vim /etc/tgt/targets.conf
# 此档案的语法如下：
<target iqn.相关装置的target名称>
    backing-store /你的/虚拟设备/完整檔名-1
    backing-store /你的/虚拟设备/完整檔名-2
</target>

<target iqn.2011-08.vbird.centos:vbirddisk>
    backing-store /srv/iscsi/disk1.img  <==LUN 1 (LUN 的编号通常照顺序)
    backing-store /dev/sda10            <==LUN 2
    backing-store /dev/server/iscsi01   <==LUN 3
</target>
```
除了 backing-store 之外，在这个配置文件当中还有一些比较特别的参数 (man tgt-admin)：
* backing-store (虚拟的装置), direct-store (实际的装置)： 设定装置时，如果你的整颗磁盘是全部被拿来当 iSCSI 分享之用，那么才能够使用 direct-store 。不过，根据网络上的其他文件， 似乎说明这个设定值有点危险的样子。所以，基本上还是建议单纯使用模拟的 backing-store 较佳。
* initiator-address (用户端地址)： 如果你想要限制能够使用这个 target 的客户端来源，才需要填写这个设定值。基本上，不用设定它 (代表所有人都能使用的意思)， 因为我们后来会使用 iptables 来规范可以联机的客户端
* incominguser (用户账号密码设定)： 如果除了来源 IP 的限制之外，你还想要让使用者输入账密才能使用你的 iSCSI target 的话，那么就加用这个设定项目。 此设定后面接两个参数，分别是账号与密码。
* write-cache [off|on] (是否使用快取)： 在预设的情况下，tgtd 会使用快取来增快速度。不过，这样可能会有遗失数据的风险。所以，如果你的数据比较重要的话， 或许不要使用快取，直接存取装置会比较妥当一些。

假设你的环境中，仅允许 192.168.100.0/24 这个网段可以存取 iSCSI target，而且存取时需要帐密分别为 vbirduser, vbirdpasswd ，此外，不要使用快取，那么原本的配置文件之外，还得要加上这样的参数
```
[root@www ~]# vim /etc/tgt/targets.conf
<target iqn.2011-04.vbird.centos:vbirddisk>
    backing-store /home/iscsi/disk1.img
    backing-store /dev/sda7
    backing-store /dev/server/iscsi01
    initiator-address 192.168.100.0/24
    incominguser vbirduser vbirdpasswd
    write-cache off
</target>
```
### 启动 iSCSI target 以及观察相关端口与磁盘信息
再来则是启动、开机启动，以及观察 iSCSI target 所启动的埠口
```
[root@www ~]# /etc/init.d/tgtd start
[root@www ~]# chkconfig tgtd on
[root@www ~]# netstat -tlunp | grep tgt
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address   Foreign Address   State   PID/Program name
tcp        0      0 0.0.0.0:3260    0.0.0.0:*         LISTEN  26944/tgtd
tcp        0      0 :::3260         :::*              LISTEN  26944/tgtd
# 重点就是那个 3260 TCP 封包，等一下的防火墙务必要开放这个埠口。

# 观察一下我们 target 相关信息，以及提供的 LUN 数据内容：
[root@www ~]# tgt-admin --show
Target 1: iqn.2011-08.vbird.centos:vbirddisk <==就是我们的 target
    System information:
        Driver: iscsi
        State: ready
    I_T nexus information:
    LUN information:
        LUN: 0
            Type: controller     <==这是个控制器，并非可以用的 LUN 
....(中间省略)....
        LUN: 1
            Type: disk       <==第一个 LUN，是磁盘 (disk) 
            SCSI ID: IET     00010001
            SCSI SN: beaf11
            Size: 2155 MB      <==容量有这么大
            Online: Yes
            Removable media: No
            Backing store type: rdwr
            Backing store path: /dev/sda10 <==磁盘所在的实际文件名
        LUN: 2
            Type: disk
            SCSI ID: IET     00010002
            SCSI SN: beaf12
            Size: 2147 MB
            Online: Yes
            Removable media: No
            Backing store type: rdwr
            Backing store path: /dev/server/iscsi01
        LUN: 3
            Type: disk
            SCSI ID: IET     00010003
            SCSI SN: beaf13
            Size: 524 MB
            Online: Yes
            Removable media: No
            Backing store type: rdwr
            Backing store path: /srv/iscsi/disk1.img
    Account information:
        vbirduser        <==额外的帐户信息
    ACL information:
        192.168.100.0/24 <==额外的来源 IP 限制
```
### 设定防火墙
不论你有没有使用 initiator-address 在 targets.conf 配置文件中，iSCSI target 就是使用 TCP/IP 传输数据的， 所以你还是得要在防火墙内设定可以联机的客户端。
```
[root@www ~]# vim /usr/local/virus/iptables/iptables.allow
iptables -A INPUT  -p tcp -s 192.168.100.0/24 --dport 3260 -j ACCEPT

[root@www ~]# /usr/local/virus/iptables/iptables.rule
[root@www ~]# iptables-save | grep 3260
-A INPUT -s 192.168.100.0/24 -p tcp -m tcp --dport 3260 -j ACCEPT
# 若有其他用户需要联机，自行复制 iptables.allow 内的语法，修改来源端即可。
```

# iSCSI initiator 的设定
## 所需软件与软件结构
要设定 iSCSI initiator 必须要安装 iscsi-initiator-utils 才行。
* /etc/iscsi/iscsid.conf：主要的配置文件，用来连结到 iSCSI target 的设定；
* /sbin/iscsid：启动 iSCSI initiator 的主要服务程序；
* /sbin/iscsiadm：用来管理 iSCSI initiator 的主要设定程序；
* /etc/init.d/iscsid：让本机模拟成为 iSCSI initiater 的主要服务；
* /etc/init.d/iscsi：在本机成为 iSCSI initiator 之后，启动此脚本，让我们可以登入 iSCSI target。所以 iscsid 先启动后，才能启动这个服务。为了防呆，所以 /etc/init.d/iscsi 已经写了一个启动指令， 启动 iscsi 前尚未启动 iscsid ，则会先呼叫 iscsid 才继续处理 iscsi 。

因为 /etc/init.d/iscsi 脚本已经包含了启动 /etc/init.d/iscsid 的步骤在里面，所以，理论上，只要启动 iscsi 就好了。此外，那个 iscsid.conf 里面大概只要设定好登入 target 时的帐密即可， 其他的 target 搜寻、设定、取得的方法都直接使用 iscsiadm 这个指令来完成。由于 iscsiadm 侦测到的结果会直接写入 /var/lib/iscsi/nodes/ 当中，因此只要启动 /etc/init.d/iscsi 就能够在下次开机时，自动的连结到正确的 target 。
## initiator 的实际设定
理论上，不论是 target 还是 initiator 都应该是要我们管理的机器才对。 而现在我们知道 target 其实有设定账号与密码的，所以底下我们就得要修改一下 iscsid.conf 的内容才行。
### 修改 /etc/iscsi/iscsid.conf 内容，并启动 iscsi
这个档案的修改很简单，因为里面的参数大多已经预设做的不错了，所以只要填写 target 登入时所需要的帐密即可。 修改的地方有两个，一个是侦测时 (discovery) 可能会用到的帐密，一个是联机时 (node) 会用到的帐密：
```
[root@clientlinux ~]# vim /etc/iscsi/iscsid.conf
node.session.auth.username = vbirduser   <==在 target 时设定的
node.session.auth.password = vbirdpasswd <==约在 53, 54 行
discovery.sendtargets.auth.username = vbirduser  <==约在 67, 68 行
discovery.sendtargets.auth.password = vbirdpasswd

[root@clientlinux ~]# chkconfig iscsid on
[root@clientlinux ~]# chkconfig iscsi on
```
由于我们尚未与 target 联机，所以 iscsi 并无法让我们顺利启动的，因此上面只要 chkconfig 即可，不需要启动他。 接下来侦测 target 与写入系统信息，使用 iscsiadm 这个指令就可以完成所有动作了。
### 侦测 192.168.100.254 这部 target 的相关数据
```
[root@clientlinux ~]# iscsiadm -m discovery -t sendtargets -p IP:port
选项与参数：
-m discovery   ：使用侦测的方式进行 iscsiadmin 指令功能；
-t sendtargets ：透过 iscsi 的协议，侦测后面的设备所拥有的 target 数据
-p IP:port     ：就是那部 iscsi 设备的 IP 与埠口，不写埠口预设是 3260 

范例：侦测 192.168.100.254 这部 iSCSI 设备的相关数据
[root@clientlinux ~]# iscsiadm -m discovery -t sendtargets -p 192.168.100.254
192.168.100.254:3260,1  iqn.2011-08.vbird.centos:vbirddisk
# 192.168.100.254:3260,1 ：在此 IP, 端口上面的 target 号码，本例中为 target1
# iqn.2011-08.vbird.centos:vbirddisk ：就是我们的 target 名称

[root@clientlinux ~]# ll -R /var/lib/iscsi/nodes/
/var/lib/iscsi/nodes/iqn.2011-08.vbird.centos:vbirddisk
/var/lib/iscsi/nodes/iqn.2011-08.vbird.centos:vbirddisk/192.168.100.254,3260,1
# 上面的特殊字体部分，就是我们利用 iscsiadm 侦测到的 target 结果
```
现在我们知道了 target 的名称，同时将所有侦测到的信息通通写入到上述 /var/lib/iscsi/nodes/iqn.2011-08.vbird.centos:vbirddisk/192.168.100.254,3260,1 目录内的 default 档案中， 若信息有修订过的话，那你可以到这个档案内修改，也可以透过 iscsiadm 的 update 功能处理相关参数。
### 开始进行联机 iSCSI target
因为 initiator 可能会连接多部的 target 设备，可以先列出目前系统上侦测到的 target ，然后再找到我们要的那部 target 来进行登入的作业。不过，如果你想要将所有侦测到的 target 全部都登入的话， 那么整个步骤可以再简化：
```
范例：根据前一个步骤侦测到的资料，启动全部的 target
[root@clientlinux ~]# /etc/init.d/iscsi restart
正在停止 iscsi：                                 [  确定  ]
正在激活 iscsi：                                 [  确定  ]
# 将系统里面全部的 target 通通以 /var/lib/iscs/nodes/ 内的设定登入

范例：显示出目前系统上面所有的 target 数据：
[root@clientlinux ~]# iscsiadm -m node
192.168.100.254:3260,1 iqn.2011-08.vbird.centos:vbirddisk
选项与参数：
-m node：找出目前本机上面所有侦测到的 target 信息，可能并未登入

范例：仅登入某部 target ，不要重新启动 iscsi 服务
[root@clientlinux ~]# iscsiadm -m node -T target名称 --login
选项与参数：
-T target名称：仅使用后面接的那部 target ，target 名称可用上个指令查到
--login      ：就是登入

[root@clientlinux ~]# iscsiadm -m node -T iqn.2011-08.vbird.centos:vbirddisk \
>  --login
# 这次进行会出现错误，是因为我们已经登入了，不可重复登入
```
```
[root@clientlinux ~]# fdisk -l
Disk /dev/sda: 8589 MB, 8589934592 bytes  <==这是原有的那颗磁盘，略过不看
....(中间省略)....

Disk /dev/sdc: 2147 MB, 2147483648 bytes
67 heads, 62 sectors/track, 1009 cylinders
Units = cylinders of 4154 * 512 = 2126848 bytes
Sector size (logical/physical): 512 bytes / 512 bytes

Disk /dev/sdb: 2154 MB, 2154991104 bytes
67 heads, 62 sectors/track, 1013 cylinders
Units = cylinders of 4154 * 512 = 2126848 bytes
Sector size (logical/physical): 512 bytes / 512 bytes

Disk /dev/sdd: 524 MB, 524288000 bytes
17 heads, 59 sectors/track, 1020 cylinders
Units = cylinders of 1003 * 512 = 513536 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
```
主机上面多出了三个新的磁盘，容量与刚刚在 192.168.100.254 那部 iSCSI target 上面分享的 LUN 一样大。iSCSI target 每次都要比 iSCSI initiator 这部主机还要早开机，否则我们的 initiator 会出问题。
### 更新/删除/新增 target 数据的方法
```
[root@clientlinux ~]# iscsiadm -m node -T targetname --logout
[root@clientlinux ~]# iscsiadm -m node -o [delete|new|update] -T targetname
选项与参数：
--logout ：就是注销 target，但是并没有删除 /var/lib/iscsi/nodes/ 内的数据
-o delete：删除后面接的那部 target 链接信息 (/var/lib/iscsi/nodes/*)
-o update：更新相关的信息
-o new   ：增加一个新的 target 信息。

范例：关闭 iSCSI target 的数据，并且移除链接
[root@clientlinux ~]# iscsiadm -m node   <==还是先显示出相关的 target iqn 名称
192.168.100.254:3260,1 iqn.2011-08.vbird.centos:vbirddisk
[root@clientlinux ~]# iscsiadm -m node -T iqn.2011-08.vbird.centos:vbirddisk \
>  --logout
Logging out of session [sid: 1, target: iqn.2011-08.vbird.centos:vbirddisk,
 portal: 192.168.100.254,3260]
Logout of [sid: 1, target: iqn.2011-08.vbird.centos:vbirddisk, portal:
 192.168.100.254,3260] successful.
# 这个时候的 target 连结还是存在的，虽然注销你还是看的到！

[root@clientlinux ~]# iscsiadm -m node -o delete \
>  -T iqn.2011-08.vbird.centos:vbirddisk
[root@clientlinux ~]# iscsiadm -m node
iscsiadm: no records found!

[root@clientlinux ~]# /etc/init.d/iscsi restart
# 会发现， target 的信息不见了
```
## 一个测试范例
假设：
1. 已经在 initiator 上面将 target 数据清除了；
2. 现在我们只知道 iSCSI target 的 IP 是 192.168.100.254 ，而需要的帐密是 vbirduser, vbirdpasswd；
3. 帐密信息你已经写入 /etc/iscsi/iscsid.conf 里面了；
4. 假设我们预计要将 target 的磁盘拿来当作 LVM 内的 PV 使用；
5. 并且将所有的磁盘容量都给一个名为 /dev/iscsi/disk 的 LV 使用；
6. 这个 LV 会被格式化为 ext4 ，且挂载在 /data/iscsi 内。
```
# 1. 启动 iscsi ，并且开始侦测及登入 192.168.100.254 上面的 target 名称
[root@clientlinux ~]# /etc/init.d/iscsi restart
[root@clientlinux ~]# chkconfig iscsi on
[root@clientlinux ~]# iscsiadm -m discovery -t sendtargets -p 192.168.100.254
[root@clientlinux ~]# /etc/init.d/iscsi restart
[root@clientlinux ~]# iscsiadm -m node
192.168.100.254:3260,1 iqn.2011-08.vbird.centos:vbirddisk

# 2. 开始处理 LVM 的流程，由 PV, VG, LV 依序处理
[root@clientlinux ~]# fdisk -l    <==出现的资料中你会发现 /dev/sd[b-d]
[root@clientlinux ~]# pvcreate /dev/sd{b,c,d}  <==建立 PV 
  Wiping swap signature on /dev/sdb
  Physical volume "/dev/sdb" successfully created
  Physical volume "/dev/sdc" successfully created
  Physical volume "/dev/sdd" successfully created

[root@clientlinux ~]# vgcreate iscsi /dev/sd{b,c,d}  <==建立 VG 去
  Volume group "iscsi" successfully created

[root@clientlinux ~]# vgdisplay  <==要找到可用的容量
  --- Volume group ---
  VG Name               iscsi
....(中间省略)....
  Act PV                3
  VG Size               4.48 GiB
  PE Size               4.00 MiB
  Total PE              1148  <==共 1148 个
  Alloc PE / Size       0 / 0
  Free  PE / Size       1148 / 4.48 GiB
....(底下省略)....

[root@clientlinux ~]# lvcreate -l 1148 -n disk iscsi
  Logical volume "disk" created

[root@clientlinux ~]# lvdisplay
  --- Logical volume ---
  LV Name                /dev/iscsi/disk
  VG Name                iscsi
  LV UUID                opR64B-Zeoe-C58n-ipN2-em3O-nUYs-wjEZDP
  LV Write Access        read/write
  LV Status              available
  # open                 0
  LV Size                4.48 GiB <==注意一下容量对不对
  Current LE             1148
  Segments               3
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:2

# 3. 开始格式化，并且进行开机自动挂载的动作
[root@clientlinux ~]# mkfs -t ext4 /dev/iscsi/disk
[root@clientlinux ~]# mkdir -p /data/iscsi
[root@clientlinux ~]# vim /etc/fstab
/dev/iscsi/disk   /data/iscsi   ext4   defaults,_netdev   1   2

[root@clientlinux ~]# mount -a
[root@clientlinux ~]# df -Th
文件系统      类型    Size  Used Avail Use% 挂载点
/dev/mapper/iscsi-disk
              ext4    4.5G  137M  4.1G   4% /data/iscsi
```
比较特殊的是 /etc/fstab 里面的第四个字段，加上 _netdev (最前面是底线) 指的是，因为这个 partition 位于网络上， 所以得要网络开机启动完成后才会挂载的意思。重新启动 iSCSI initiator ，和重新启动系统后，观察 /data/iscsi 是否存在。  
切回 iSCSI target 那部主机，观察有谁使用我们的 target：
```
[root@www ~]# tgt-admin --show
Target 1: iqn.2011-08.vbird.centos:vbirddisk
    System information:
        Driver: iscsi
        State: ready
    I_T nexus information:
        I_T nexus: 2
            Initiator: iqn.1994-05.com.redhat:71cf137f58f2 <==initiator 主机的名字
            Connection: 0
                IP Address: 192.168.100.10    <==就是这里联机进来
    LUN information:
....(后面省略)....
```
initiator 是 redhat 的名字，可以修改 initiator 那部主机的 /etc/iscsi/initiatorname.iscsi 这个档案的内容：
> 不过，这个动作最好在使用 target 的 LUN 之前就进行，否则，当你使用了 LUN 的磁盘后，再修改这个档案后， 你的磁盘文件名可能会改变。
```
# 1. 先在 iSCSI initiator 上面进行如下动作：
[root@clientlinux ~]# vim /etc/iscsi/initiatorname.iscsi
InitiatorName=iqn.2011-08.vbird.centos:initiator
[root@clientlinux ~]# /etc/init.d/iscsi restart

# 2. 在 iSCSI target 上面就可以发现如下的数据修订了：
[root@www ~]# tgt-admin --show
Target 1: iqn.2011-08.vbird.centos:vbirddisk
    System information:
        Driver: iscsi
        State: ready
    I_T nexus information:
        I_T nexus: 5
            Initiator: iqn.2011-08.vbird.centos:initiator
            Connection: 0
                IP Address: 192.168.100.10
....(后面省略)....
```