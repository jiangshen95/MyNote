# Linux 连上 Internet 前的注意事项
## Linux 的网络卡
* 网络卡的装置代号  
  在 Linux 里面的各项装置几乎都是以文件名来取代的，例如 /dev/hda 代表 IDE1 接口的第一个 master 硬盘等等。 不过，网络卡的代号 (Network Interface Card, NIC) 却是以模块对应装置名称来代替的， 而默认的网络卡代号为 eth0 ，第二张网络卡则为 eth1 ，以此类推。
* 网络卡的模块 (驱动程序)  
  需要核心支持才能驱动。一般来说，目前新版的 Linux distributions 默认可以支持的网络卡芯片组数量已经很完备了。
* 观察核心所捉到的网卡信息  
  利用 dmesg 查阅
  ```
  [root@www ~]# dmesg | grep -in eth
  377:e1000: eth0: e1000_probe: Intel(R) PRO/1000 Network Connection
  383:e1000: eth1: e1000_probe: Intel(R) PRO/1000 Network Connection
  418:e1000: eth0 NIC Link is Up 1000 Mbps Full Duplex, Flow Control: RX
  419:eth0: no IPv6 routers present
  ```
  也可以透过 lspci 来查询相关的设备芯片数据
  ```
  [root@www ~]# lspci | grep -i ethernet
  00:03.0 Ethernet controller: Intel Corporation 82540EM Gigabit Ethernet 
  Controller (rev 02)
  ```
* 观察网络卡的模块  
  ```
  [root@www ~]# lsmod | grep 1000
  e1000                 119381  0  <==确实有载入到核心中！

  [root@www ~]# modinfo e1000
  filename:    /lib/modules/2.6.32-71.29.1.el6.x86_64/kernel/drivers/net/e1000/e1000.ko
  version:     7.3.21-k6-NAPI
  license:     GPL
  description: Intel(R) PRO/1000 Network Driver
  .....(以下省略).....
  ```
## 编译网卡驱动程序(Option)
1. 取得官方网站的驱动程序：
2. 解压缩与编译：
3. 模块之测试与处理
4. 设定开机自动启动网络卡模块 (Option)
5. 尝试设定 IP 
## Linux 网络相关配置文件
所需网络参数|主要配置文件档名|重要参数
:---:|:---:|:---:
IP<br>Netmask<br>DHCP 与否<br>Gateway 等|/etc/sysconfig/network-scripts/ifcfg-eth0|DEVICE=网卡的代号<br>BOOTPROTO=是否使用 dhcp<br>HWADDR=是否加入网卡卡号(MAC)<br>IPADDR=就是IP地址<br>NETMASK=只网络屏蔽<br>ONBOOT=要不要默认启动此接口<br>GATEWAY=就是通讯闸<br>NM_CONTROLLED=额外的网管软件，建议取消这个项目
主机名|/etc/sysconfig/network|NETWORKING=要不要有网络<br>NETWORKING_IPV6=支持IPv6否？<br>HOSTNAME=你的主机名
DNS IP|/etc/resolv.conf|nameserver DNS的IP
私有 IP 对应的主机名|/etc/hosts|私有IP 主机名 别名
* /etc/services  
  这个档案则是记录架构在 TCP/IP 上面的总总协议，包括 http, ftp, ssh, telnet 等等服务所定义的 port number ，都是这个档案所规划出来的。如果你想要自定义一个新的协议与 port 的对应，就得要改这个档案了；
* /etc/protocols  
  这个档案则是在定义出 IP 封包协议的相关数据，包括 ICMP/TCP/UDP 这方面的封包协议的定义等。

网络方便的启动指令：
* /etc/init.d/network restart  
  可以重新启动整个网络的参数。主动的去读取所有的网络配置文件，所以可以很快的恢复系统默认的参数值。
* ifup eth0 (ifdown eth0)  
  启动或者是关闭某张网络接口。可以透过这个简单的 script 来处理。 这两个 script 会主动到 /etc/sysconfig/network-scripts/ 目录下， 读取适当的配置文件来处理。

# 连上 Internet 的设定方法
## 手动设定固定 IP 参数 (设用学术网络、ADSL 固定制) + 五大检查步骤
一般来说，固定 IP 来自：
* 学术网络：由学校单位直接给予的一组 IP 网络参数；
* 固定制 ADSL：向 ISP 申请的一组固定 IP 的网络参数；
* 企业内部或 IP 分享器内部的局域网络：例如企业内使用私有 IP 作为局域网络的联机之用时，那么我们的 Linux 当然也就需要向企业的网管人员申请一组固定的 IP 网络参数。

设定信息为：
```
IP:       192.168.1.100
Netmask:  255.255.255.0
Gateway:  192.168.1.254
DNS IP:   168.95.1.1
Hostname: www.centos.vbird
```
修改的参数|配置文件与重要启动脚本|观察结果的指令
:---:|:---:|:---:
IP相关参数|/etc/sysconfig/network-scripts/ifcfg-eth0<br>/etc/init.d/network restart|ifconfig (IP/Netmask)<br>route -n (gateway)
DNS|/etc/resolv.conf|dig www.google.com
主机名|/etc/sysconfig/network<br>/etc/hosts|hostname (主机名)<br>ping $(hostname)<br>reboot
1. IP/Netmask/Gateway 的设定、启动与观察  
   ```
   [root@www ~]# vim /etc/sysconfig/network-scripts/ifcfg-eth0
   DEVICE="eth0"               <==网络卡代号，必须要 ifcfg-eth0 相对应
   HWADDR="08:00:27:71:85:BD"  <==就是网络卡地址，若只有一张网卡，可省略此项目
   NM_CONTROLLED="no"          <==不要受到其他软件的网络管理！
   ONBOOT="yes"                <==是否默认启动此接口的意思
   BOOTPROTO=none              <==取得IP的方式，其实关键词只有dhcp，手动可输入none
   IPADDR=192.168.1.100        <== IP 
   NETMASK=255.255.255.0       <==就是子网掩码
   GATEWAY=192.168.1.254       <==就是预设路由
   # 重点是上面这几个设定项目，底下的则可以省略的
   NETWORK=192.168.1.0         <==就是该网段的第一个 IP，可省略
   BROADCAST=192.168.1.255     <==就是广播地址啰，可省略
   MTU=1500                    <==就是最大传输单元的设定值，若不更改则可省略
   ```
   请注意每个变量(左边的英文)都应该要大写，否则我们的 script 会误判。
     *  DEVICE：这个设定值后面接的装置代号需要与文件名 (ifcfg-eth0) 那个装置代号相同才行！否则可能会造成一些装置名称找不到的困扰。
     * BOOTPROTO：启动该网络接口时，使用何种协议？ 如果是手动给予 IP 的环境，请输入 static 或 none ，如果是自动取得 IP 的时候， 请输入 dhcp 。
     * GATEWAY：代表的是『整个主机系统的 default gateway』。设定这个项目时，不要有重复设定的情况发生，也就是当你有 ifcfg-eth0, ifcfg-eth1.... 等多个档案，只要在其中一个档案设定 GATEWAY 即可。
     * GATEWAYDEV：如果你不是使用固定的 IP 作为 Gateway ， 而是使用网络装置作为 Gateway (通常 Router 最常有这样的设定)，那也可以使用 GATEWAYDEV 来设定通讯闸装置。不过这个设定项目很少使用。
     * HWADDR：就是网络卡的卡号。如果你的主机上面有两张一模一样的网卡，使用的模块是相同的。 此时，你的 Linux 很可能会将 eth0, eth1 搞混，而造成你网络设定的困扰。由于 MAC 是直接写在网卡上的，因此指定 HWADDR 到这个配置文件中，就可以解决网卡对应代号的问题了。
   ```
   [root@www ~]# /etc/init.d/network restart
   Shutting down interface eth0:         [ OK ]  <== 先关闭界面
   Shutting down loopback interface:     [ OK ]
   Bringing up loopback interface:       [ OK ]  <== 再开启界面
   Bringing up interface eth0:           [ OK ]
   # 针对这部主机的所有网络接口 (包含 lo) 与通讯闸进行重新启动，所以网络会停顿再开
   ```
   ```
   # 检查一：先察看 IP 参数，重点是 IP 与 Netmask 
   [root@www ~]# ifconfig eth0
   eth0      Link encap:Ethernet  HWaddr 08:00:27:71:85:BD
             inet addr:192.168.1.100  Bcast:192.168.1.255  Mask:255.255.255.0
             inet6 addr: fe80::a00:27ff:fe71:85bd/64 Scope:Link
             UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
             RX packets:655 errors:0 dropped:0 overruns:0 frame:0
             TX packets:468 errors:0 dropped:0 overruns:0 carrier:0
             collisions:0 txqueuelen:1000
             RX bytes:61350 (59.9 KiB)  TX bytes:68722 (67.1 KiB)
   # 有出现上头那个 IP 的数据才是正确的启动；特别注意 inet addr 与 Mask 项目
   # 这里如果没有成功，得回去看看配置文件有没有错误，然后再重新 network restart ！

   # 检查二：检查一下你的路由设定是否正确
   [root@www ~]# route -n
   Kernel IP routing table
   Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
   192.168.1.0     0.0.0.0         255.255.255.0   U     0      0        0 eth0
   169.254.0.0     0.0.0.0         255.255.0.0     U     1002   0        0 eth0
   0.0.0.0         192.168.1.254   0.0.0.0         UG    0      0        0 eth0
   # 前面的 0.0.0.0 代表预设路由的设定值！

   # 检查三：测试看看与路由器之间是否能够联机成功
   [root@www ~]# ping -c 3 192.168.1.254
   PING 192.168.1.254 (192.168.1.254) 56(84) bytes of data.
   64 bytes from 192.168.1.254: icmp_seq=1 ttl=64 time=2.08 ms
   64 bytes from 192.168.1.254: icmp_seq=2 ttl=64 time=0.309 ms
   64 bytes from 192.168.1.254: icmp_seq=3 ttl=64 time=0.216 ms

   --- 192.168.1.254 ping statistics ---
   3 packets transmitted, 3 received, 0% packet loss, time 2004ms
   rtt min/avg/max/mdev = 0.216/0.871/2.088/0.861 ms
   # 有出现 ttl 才是正确的响应，如果出现『 Destination Host Unreachable 』，表示没有成功的联机到你的 GATEWAY。请检查有无设定错误。
   ```
   要注意，第三个检查如果失败，可能要看你的路由器是否已经关闭？或者是你的 switch/hub 是否有问题，或者是你的网络线是否错误，还是说你的或路由器的防火墙设定错误了？
2. DNS 服务器的 IP 设定与观察  
   ```
   [root@www ~]# vim /etc/resolv.conf
   nameserver 168.95.1.1
   nameserver 139.175.10.20
   ```
   ```
   # 检查四： DNS 是否顺利运作
   [root@www ~]# dig www.google.com
   ....(前面省略)....
   ;; QUESTION SECTION:
   ;www.google.com.                        IN      A

   ;; ANSWER SECTION:
   www.google.com.         428539  IN      CNAME   www.l.google.com.
   www.l.google.com.       122     IN      A       74.125.71.106
   ....(中间省略)....

   ;; Query time: 30 msec
   ;; SERVER: 168.95.1.1#53(168.95.1.1)  <==这里的项目也很重要！
   ;; WHEN: Mon Jul 18 01:26:50 2011
   ;; MSG SIZE  rcvd: 284
   ```
3. 主机名的修改、启动与观察   
   ```
   [root@www ~]# vim /etc/sysconfig/network
   NETWORKING=yes
   HOSTNAME=www.centos.vbird

   [root@www ~]# vim /etc/hosts
   192.168.1.100    www.centos.vbird
   # 特别注意，这个档案的原本内容不要删除！只要新增额外的数据即可！
   ```
   修改完毕之后要顺利启动的话，得要重新启动才可以。因为系统已经有非常多的服务启动了， 这些服务如果需要主机名，都是到这个档案去读取的。而我们知道配置文件更新过后，服务都得要重新启动才行。 
   ```
   [root@www ~]# hostname
   localhost.localdomain
   # 还是默认值，尚未更新成功！我们还得要进行底下的动作！

   # 检查五：看看你的主机名有没有对应的 IP 呢？没有的话，开机流程会很慢！
   [root@www ~]# ping -c 2 www.centos.vbird
   PING www.centos.vbird (192.168.1.100) 56(84) bytes of data.
   64 bytes from www.centos.vbird (192.168.1.100): icmp_seq=1 ttl=64 time=0.015 ms
   64 bytes from www.centos.vbird (192.168.1.100): icmp_seq=2 ttl=64 time=0.028 ms

   --- www.centos.vbird ping statistics ---
   2 packets transmitted, 2 received, 0% packet loss, time 1000ms
   rtt min/avg/max/mdev = 0.015/0.021/0.028/0.008 ms
   # 因为我们有设定 /etc/hosts 规定 www.centos.vbird 的 IP ，
   # 所以才找的到主机主机名对应的正确 IP！这时才能够 reboot 。
   ```
## 自动取得 IP 参数 (DHCP 方法，适用 Cable modem、IP 分享器的环境)
DHCP (Dynamic Host Configuration Protocol) 『动态的调整主机的网络参数』
* Cable Modem：就是使用电视缆线进行网络回路联机的方式
* ADSL 多 IP 的 DHCP 制：
* IP 分享器或 NAT 有架设 DHCP 服务时：当你的主机位于 IP 分享器的后端，或者是你的 LAN 当中有 NAT 主机且 NAT 主机有架设 DHCP 服务时， 那取得 IP 的方法也是这样

依旧需要手动设定 IP 的主机名设定，IP 参数与 DNS 则不需要额外设定，仅需要修改 ifcfg-eth0 即可。
```
[root@www ~]# vim /etc/sysconfig/network-scripts/ifcfg-eth0
DEVICE=eth0
HWADDR="08:00:27:71:85:BD"
NM_CONTROLLED="no"
ONBOOT=yes
BOOTPROTO=dhcp
```
然后，重新启动网络。基本上，/etc/resolv.conf 预设会被 DHCP 所修改过，因此你不需要修改 /etc/resolv.conf。甚至连主机名都会被 DHCP 所修订。不过，如果你有特殊需求，那么 /etc/sysconfig/network 以及 /etc/hosts 请自行修改正确
## ADSL 拨接上网
可以使用 rp-pppoe 这套软件。rp-pppoe 使用的是 Point to Point (ppp) over Ethernet 的点对点协议所产生的网络接口，因此当你顺利的拨接成功之后， 会多产生一个实体网络接口『 ppp0 』。  
而由于 ppp0 是架构在以太网络卡上的，你必须要有以太网卡，同时，即使拨接成功后，你也不能将没有用到的 eth0 关闭。因此，拨接成功后就会有：
* 内部循环测试用的 lo 接口；
* 网络卡 eth0 这个接口；
* 拨接之后产生的经由 ISP 对外连接的 ppp0 接口。

虽然 ppp0 是架构在以太网卡上面的，但上头这三个接口在使用上是完全独立的，互不相干。
* 这张网络卡 (假设是 eth0) 有接内部网络(LAN)：  
  ppp0 可以连上 Internet ，但是内网则使用 eth0 来跟其他内部主机联机时， 那么你的 IP 设定参数： /etc/sysconfig/network-scripts/ifcfg-eth0 应该要给予一个私有 IP 以使内部的 LAN 也可以透过 eth0 来进行联机
  ```
  [root@www ~]# vim /etc/sysconfig/network-scripts/ifcfg-eth0
  DEVICE=eth0
  BOOTPROTO=none
  NM_CONTROLLED=no
  IPADDR=192.168.1.100
  NETMASK=255.255.255.0
  ONBOOT=yes
  ```
  『千万不要有 GATEWAY 的设定！』， 因为 ppp0 拨接成功后， ISP 会主动的给予 ppp0 接口一个可以连上 Internet 的 default gateway ， 如果你又设定另一个 default gateway ，两个网关可能会造成你的网络不通。
* 这部主机仅有连接 ADSL 调制解调器，并没有内部网域：  
  你的 eth0 有没有 IP 都没有关系时，那么上面的设定当中的那个『 ONBOOT=yes 』直接改成『 ONBOOT=no 』，拨接启动 ppp0 时，系统会主动的唤醒 eth0 ，只是 eth0 不会有 IP 信息。

使用 rp-pppoe 拨接
1. 设定连接到 ADSL 调制解调器那张网卡 (暂订为 eth0)
2. 设定拨接的账号与密码  
   这个动作只要在第一次建立账号/密码时处理即可，未来除非账号密码改变了， 否则这个动作都不需要重新处理。
   ```
    [root@www ~]# pppoe-setup
    Welcome to the PPPoE client setup.  First, I will run some checks on
    your system to make sure the PPPoE client is installed properly...

    LOGIN NAME  (从 ISP 处取得的账号填入处)
    Enter your Login Name (default root): T1234567
    # 注这个账号名称是 ISP 给的，其中如果是 SeedNet ，输入如上，
    # 如果是 Hinet 的话，就得要输入 username@hinet.net，后面的主机名也要写。

    INTERFACE  (ADSL 调制解调器所接的网卡代号)
    Enter the Ethernet interface connected to the PPPoE modem
    For Solaris, this is likely to be something like /dev/hme0.
    For Linux, it will be ethX, where 'X' is a number.
    (default eth0): eth0

    Enter the demand value (default no): no

    DNS  (就填入 ISP 处取得的 DNS 号码)
    Enter the DNS information here: 168.95.1.1
    Enter the secondary DNS server address here: <==若无第二部就按 enter

    PASSWORD  (从 ISP 处取得的密码)
    Please enter your Password: <==输入密码两次，屏幕不会有星号 * 
    Please re-enter your Password:

    USERCTRL  (要不要让一般用户启动与关闭？最好是不要！)
    Please enter 'yes' (three letters, lower-case.) if you want to allow
    normal user to start or stop DSL connection (default yes): no

    FIREWALLING  (防火墙方面，先取消，用自己未来设定的)
    The firewall choices are:
    0 - NONE: This script will not set any firewall rules.  You are responsible
              for ensuring the security of your machine.  You are STRONGLY
              recommended to use some kind of firewall rules.
    1 - STANDALONE: Appropriate for a basic stand-alone web-surfing workstation
    2 - MASQUERADE: Appropriate for a machine acting as an Internet gateway
                    for a LAN
    Choose a type of firewall (0-2): 0

    Start this connection at boot time (要不要开机立即启动拨接程序？)
    Do you want to start this connection at boot time?
    Please enter no or yes (default no):yes

    ** Summary of what you entered **
    Ethernet Interface: eth0
    User name:          T1234567
    Activate-on-demand: No
    Primary DNS:        168.95.1.1
    Firewalling:        NONE
    User Control:       no
    Accept these settings and adjust configuration files (y/n)? y
    Adjusting /etc/sysconfig/network-scripts/ifcfg-ppp0
    Adjusting /etc/resolv.conf
      (But first backing it up to /etc/resolv.conf.bak)
    Adjusting /etc/ppp/chap-secrets and /etc/ppp/pap-secrets
      (But first backing it up to /etc/ppp/chap-secrets.bak)
      (But first backing it up to /etc/ppp/pap-secrets.bak)
    # 上面具有特殊字体的档案主要功能是：
    # ifcfg-ppp0  ：亦即是 ppp0 这个网络接口的配置文件案；
    # resolv.conf ：这个档案会被备份后，然后以刚刚我们上面输入的 DNS 数据取代；
    # pap-secrets, chap-secrets：我们输入的密码就放在这里！
   ```
3. 透过 adsl-start, pppoe-start 或 network restart 开始拨接上网
4. 开始检查的步骤：  
   ```
    [root@www ~]# ifconfig
    [root@www ~]# route -n
    [root@www ~]# ping GW的IP
    [root@www ~]# dig www.google.com
    [root@www ~]# hostname
   ```
   比较特殊的是，因为 ADSL 拨接是透过点对点 (ppp) 协议，所谓的点对点，就是你的 ppp0 直接连接到 ISP 的某个点 (IP) ， 所以，理论上，ppp0 是个独立的 IP ，并没有子网！因此，当你察看 ppp0 的网络参数时，他会变成这样：
   ```
    [root@www ~]# ifconfig ppp0
    ppp0      Link encap:Point-to-Point Protocol
              inet addr:111.255.69.90  P-t-P:168.95.98.254  Mask:255.255.255.255
              UP POINTOPOINT RUNNING NOARP MULTICAST  MTU:1492  Metric:1
              RX packets:59 errors:0 dropped:0 overruns:0 frame:0
              TX packets:59 errors:0 dropped:0 overruns:0 carrier:0
              collisions:0 txqueuelen:3
              RX bytes:7155 (6.9 KiB)  TX bytes:8630 (8.4 KiB)
   ```
   Mask 是 255.255.255.255 ，没有子网。
5. 取消拨接功能 (Option)  
   将 /etc/sysconfig/network-scripts/ifcfg-ppp0 内的 ONBOOT 改成 no， 然后进行：
   ```
    [root@www ~]# vim /etc/sysconfig/network-scripts/ifcfg-ppp0
    DEVICE=ppp0
    ONBOOT=no
    ....(其他省略)....

    [root@www ~]# chkconfig pppoe-server off
   ```

# 无线网络--以笔记本电脑为例
## 无线网络所需要的硬件：AP、无线网卡
无线基地台 (Wireless Access Point, 简称 AP)。无线基地台本身就是个 IP 分享器了，他本身会有两个接口，一个可以与外部的 IP 做沟通，另外一个则是作为 LAN 内部其他主机的 GATEWAY 。其他主机上面只要安装了无线网卡，并且顺利的连上 AP 后，自然就可以透过 AP 来连上 Internet 。  
## 关于 AP 的设定：网络安全方面
『如果 AP 不设定任何联机限制，那任何拥有无线网卡的主机都可以透过这个 AP 连接上你的 LAN 』。通常我们都会认为 LAN 是信任网域，所以内部是没有防火墙的，亦即是不设防的状态。
* 在 AP 上面使用网卡卡号 (MAC) 来作为是否可以存取 AP 的限制：
* 设定你的 AP 联机加密机制与密钥：
### 关于 ESSID/SSID
每部 AP 都会有一个联机的名字，那就是 SSID 或 ESSID，这个 SSID 可以提供给 client 端， 当 client 端需要进行无线联机时，他必须要说明他要利用哪一部 AP ，那个 ESSID 就是那时需要输入的数据。
## 利用无线网卡开始联机
1. 检查无线网卡的硬件装置：
2. 察看模块与相对应的网卡代号：(modinfo 与 iwconfig)  
   ```
    [root@www ~]# iwconfig
    lo        no wireless extensions.
    eth0      no wireless extensions.
    # 要出现名为 wlan0 之类的网卡才是有捉到
   ```
   先安装 RPM 驱动程序。先将 USB 拔出来，然后再安装 RPM 档案。  
   这个 iwconfig 是用在作为无线网络设定之用的一个指令，与 ifconfig 类似
3. 利用 iwlist 侦测 AP ：  
   先启动无线网卡
   ```
   [root@www ~]# ifconfig ra0 up
   ```
   使用 iwlist 来使用这个无线网卡搜寻整个区域内的无线基地台。
   ```
    [root@www ~]# iwlist ra0 scan
    ra0       Scan completed :
              Cell 01 - Address: 74:EA:3A:C9:EE:1A
                        Protocol:802.11b/g/n
                        ESSID:"vbird_tsai"
                        Mode:Managed
                        Frequency:2.437 GHz (Channel 6)
                        Quality=100/100  Signal level=-45 dBm  Noise level=-92 dBm
                        Encryption key:on
                        Bit Rates:54 Mb/s
                        IE: WPA Version 1
                            Group Cipher : CCMP
                            Pairwise Ciphers (1) : CCMP
                            Authentication Suites (1) : PSK
                        IE: IEEE 802.11i/WPA2 Version 1
                            Group Cipher : CCMP
                            Pairwise Ciphers (1) : CCMP
                            Authentication Suites (1) : PSK
    ....(底下省略)....
   ```
   接下来，修改配置文件：
   ```
    [root@www ~]# ifconfig ra0 down && rmmod rt3070sta
    [root@www ~]# vim /etc/Wireless/RT2870STA/RT2870STA.dat
    Default
    CountryRegion=5
    CountryRegionABand=7
    CountryCode=TW 
    ChannelGeography=1
    SSID=vbird_tsai        <== AP 的 ESSID 
    NetworkType=Infra
    WirelessMode=9         <==与无线 AP 支持的协议有关
    Channel=6              <==与 CountryRegion 及侦测到的频道有关的设定！
    ....(中间省略)....
    AuthMode=WPAPSK        <==我们的 AP 提供的认证模式
    EncrypType=AES         <==传送认证码的加密机制
    WPAPSK="123456780aaa"  <==密钥密码！最好用双引号括起来较佳！
    ....(底下省略)....
    # 其余的地方都保留默认值即可。

    [root@www ~]# modprobe rt3070sta && ifconfig ra0 up
    [root@www ~]# iwconfig ra0
    ra0       Ralink STA  ESSID:"vbird_tsai"  Nickname:"RT2870STA"
              Mode:Auto  Frequency=2.437 GHz  Access Point: 74:EA:3A:C9:EE:1A
              Bit Rate=1 Mb/s
              RTS thr:off   Fragment thr:off
              Encryption key:off
              Link Quality=100/100  Signal level:-37 dBm  Noise level:-37 dBm
              Rx invalid nwid:0  Rx invalid crypt:0  Rx invalid frag:0
              Tx excessive retries:0  Invalid misc:0   Missed beacon:0
   ```
4. 设定网络卡配置文件 (ifcfg-ethn)  
   详细的各项变量设定参考：/etc/sysconfig/network-scripts/ifup-wireless
   ```
    [root@www ~]# cd /etc/sysconfig/network-scripts
    [root@www network-scripts]# vim ifcfg-ra0
    DEVICE=ra0
    BOOTPROTO=dhcp
    ONBOOT=no   <== 若需要每次都自动启动，改成 yes 即可！
    ESSID=vbird_tsai
    RATE=54M    <== 可以严格指定传输的速率，要与上面 iwconfig 相同，单位 b/s
   ```
5. 启动与观察无线网卡

# 常见问题说明
## 内部网域使用某些联机服务 (如 FTP, POP3) 所遇到的联机延迟问题
如果两部主机之间无法查询到正确的主机名与 IP 的对应， 那么将『可能』发生持续查询主机名对应的动作，这个动作一般需要持续 30-60 秒，因此，你的该次联机将会持续检查主机名 30 秒钟，也就会造成奇怪的 delay 的情况。尤其是在 FTP 及 POP3 等网络联机软件上最常见。  
避免这个情况，最简单的方法就是『给予内部的主机每部主机一个名称与 IP 的对应』即可。当我们需要主机名与 IP 的对应时，系统就会先到 /etc/hosts 找寻对应的设定值， 如果找不到，才会使用 /etc/resolv.conf 的设定去因特网找。只要修改了 /etc/hosts，加入每部主机与 IP 的对应， 就能够加快主机名的检查。  
```
[root@www ~]# cat /etc/hosts
# Do not remove the following line, or various programs
# that require network functionality will fail.
127.0.0.1               localhost.localdomain   localhost
# 主机的 IP             主机的名称              主机的别名
```
将局域网内的所有计算机 IP 都写进去，并且每一部都对应一个名字，即使与 client 的计算机名称设定不同也没关系。
## 网址列无法解析问题
DNS 解析的问题。修改 /etc/resolv.conf 这个档案。
## 预设路由的问题
预设路由通常仅有一个，用来做为同一网域的其他主机传递非本网域的封包网关。 你的 default gateway 应该只能有一个， 如果是拨接，请不要在 ifcfg-eth0 当中指定 GATEWAY或 GATEWAYDEV 等变量