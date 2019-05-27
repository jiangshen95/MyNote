# DHCP 运作的原理
## DHCP 服务器的用途
连上 Internet 必要的参数：IP, netmask, network, broadcast, gateway, DNS IP。IP, netmask, network, broadcast 与 gateway 都可以在 /etc/sysconfig/network-scripts/ifcfg-eth[0-n] 这档案里面设定，DNS 服务器的地址则是在 /etc/resolv.conf 里头设定。  
DHCP (Dynamic Host Configuration Protocol) 服务器最主要的工作，就是自动的将网络参数正确的分配给网域中的每部计算机， 让客户端的计算机可以在开机的时候就立即自动的设定好网络的参数值，这些参数值可以包括了 IP、netmask、network、gateway 与 DNS 的地址等等。
## DHCP 协议的运作方式
DHCP 通常是用于局域网络内的一个通讯协议，他主要藉由客户端传送广播封包给整个物理网段内的所有主机， 若局域网络内有 DHCP 服务器时，才会响应客户端的 IP 参数要求。所以，DHCP 服务器与客户端是应该要在同一个物理网段内的。
1. 客户端：利用广播封包发送搜索 DHCP 服务器的封包：  
   若客户端网络设定使用 DHCP 协议取得 IP (在 Windows 内为『自动取得 IP』)，则当客户端开机或者是重新启动网络卡时， 客户端主机会发送出搜寻 DHCP 服务器的 UDP 封包给所有物理网段内的计算机。此封包的目标 IP 会是 255.255.255.255， 所以一般主机接收到这个封包后会直接予以丢弃，但若局域网络内有 DHCP 服务器时，则会开始进行后续行为。
2. 服务器端：提供客户端网络相关的租约以供选择：  
   DHCP 服务器在接收到这个客户端的要求后，会针对这个客户端的硬件地址 (MAC) 与本身的设定数据来进行下列工作：
     * 到服务器的登录文件中寻找该用户之前是否曾经用过某个 IP ，若有且该 IP 目前无人使用，则提供此 IP 给客户端；
     * 若配置文件针对该 MAC 提供额外的固定 IP (static IP) 时，则提供该固定 IP 给客户端；
     * 若不符合上述两个条件，则随机取用目前没有被使用的 IP 参数给客户端，并记录下来。
3. 客户端：决定选择的 DHCP 服务器提供的网络参数租约并回报服务器：  
   由于局域网络内可能并非仅有一部 DHCP 服务器，但客户端仅能接受一组网络参数的租约。 因此客户端必需要选择是否要认可该服务器提供的相关网络参数的租约。当决定好使用此服务器的网络参数租约后， 客户端便开始使用这组网络参数来设定自己的网络环境。此外，客户端也会发送一个广播封包给所有物理网段内的主机， 告知已经接受该服务器的租约。此时若有第二台以上的 DHCP 服务器，则这些没有被接受的服务器会收回该 IP 租约。至于被接受的 DHCP 服务器会继续进行底下的动作。
4. 服务器端：记录该次租约行为并回报客户端已确认的响应封包信息：  
   当服务器端收到客户端的确认选择后，服务器会回传确认的响应封包，并且告知客户端这个网络参数租约的期限， 并且开始租约计时。该次租约到期而被解约的时期：
     * 客户端脱机：不论是关闭网络接口 (ifdown)、重新启动 (reboot)、关机 (shutdown) 等行为，皆算是脱机状态，这个时候 Server 端就会将该 IP 回收，并放到 Server 自己的备用区中，等待未来的使用；
     * 客户端租约到期：前面提到 DHCP server 端发放的 IP 有使用的期限，客户端使用这个 IP 到达期限规定的时间，而且没有重新提出 DHCP 的申请时，就需要将 IP 缴回去！这个时候就会造成断线。但用户也可以再向 DHCP 服务器要求再次分配 IP 。
### DHCP 服务器给予客户端的 IP 参数为固定或动态：
可以设定 DHCP 服务器给予客户端的 IP 参数主要有两种：
* 固定 (Static) IP：  
  DHCP 可以根据 MAC 来给予固定的 IP 参数租约，所以该计算机每次都能以一个固定的 IP 连上 Internet 。这种情况比较适合当这部客户端计算机需要用来做为提供区域内的一些网络服务的主机之用 (所以 IP 要固定)。在 Linux 上获取网卡的 MAC ，使用 ifconfig 及 arp 进行：
  ```
  # 1. 观察自己的 MAC 可用 ifconfig：
  [root@www ~]# ifconfig | grep HW
  eth0      Link encap:Ethernet  HWaddr 08:00:27:71:85:BD
  eth1      Link encap:Ethernet  HWaddr 08:00:27:2A:30:14
  # 两张网卡，所以有两个硬件地址

  # 2. 观察别人的 MAC 可用 ping 配合 arp
  [root@www ~]# ping -c 3 192.168.1.254
  [root@www ~]# arp -n
  Address        HWtype  HWaddress           Flags Mask   Iface
  192.168.1.254  ether   00:0c:6e:85:d5:69   C            eth0
  ```
* 动态 (dynamic) IP：  
  Client 端每次连上 DHCP 服务器所取得的 IP 都不是固定的！都直接经由 DHCP 所随机由尚未被使用的 IP 中提供！

除非你的局域网络内的计算机有可能用来做为主机之用，所以必需要设定成为固定 IP ，否则使用动态 IP 的设定比较简单，而且使用上面具有较佳的弹性。因为你的使用者并非 24 小时都挂在在线的，所以你可以将这 150 个 IP 做良好的分配，让 200 个人来『轮流使用』这 150 个 IP 。
### 关于租约所造成的问题与租约期限：
设定期限最大的优点就是可以避免 IP 被某些使用者一直占用着，但该使用者却是 Idle (发呆) 的状态。因为目前的 DHCP 客户端程序大多会主动的依据租约时间去重新申请 IP (renew) ，也就是说在租约到期前你的 DHCP 客户端程序就已经又重新申请更新租约时间了。所以除非 DHCP 主机挂点， 否则你所取得的 IP 应该是可以一直使用下去的。
### 多部 DHCP 服务器在同一物理网段的情况
因为在网络上面，很多时候都是『先抢先赢』的， DHCP 的回应也是如此。先响应时，你使用的就是 Server1 所提供的网络参数内容，如果是 Server2 先响应，你就是使用 Server2 的参数来设定你的客户端 PC 。
## 何时需要架设 DHCP 服务器
### 使用 DHCP 的几个时机
* 具有相当多行动装置的场合：
* 区域内计算机数量相当的多时：
### 不建议使用 DHCP 主机的时机
* 在你网域内的计算机，有很多机器其实是做为主机的用途，很少用户需求，那么似乎就没有必要架设 DHCP ；
* 更极端的情况是，像一般家里，只有 3 ~ 4 部计算机，这个时候，架设 DHCP 只能拿来练练功力，事实上，并没有多大的效益；
* 当你管理的网域当中，大多网络卡都属于老旧的型号，并不支持 DHCP 的协议时；
* 很多用户的信息知识都很高，那么也没有需要架设 DHCP 

# DHCP 服务器端的设定
## 所需软件与档案结构
DHCP 的软件需求很简单，就是只要服务器端软件即可，在 CentOS 6.x 上面，这个软件的名称就是 dhcp 。使用『 rpm -ql dhcp 』来看看这个软件提供了哪些档案，基本上，比较重要的档案数据如下：
* /etc/dhcp/dhcpd.conf  
  这个就是 dhcp 服务器的主要配置文件。在某些 Linux 版本上头这个档案可能不存在，所以如果你确定有安装 dhcp 软件却找不到这个档案时，请手动自行建立它即可。
  > 其实 dhcp 软件在释出的时候都会附上一个范例档案，你可以使用『 rpm -ql dhcp 』来查询到 dhcpd.conf.sample 这个范例档案，然后将该档案复制成为 /etc/dhcp/dhcpd.conf 后，再手动去修改即可。
* /usr/sbin/dhcpd  
  启动整个 dhcp daemon 的执行档。『 man dhcpd 』来查阅
* /var/lib/dhcp/dhcpd.leases  
  DHCP 服务器端与客户端租约建立的启始与到期日就是记录在这个档案当中的
## 主要配置文件 /etc/dhcp/dhcpd.conf 的语法
DHCP 的设定很简单啊，只要将 dhcpd.conf 设定好就可以启动了。不过编辑这个档案时你必须要留意底下的规范：
* 『 # 』为批注符号；
* 除了右括号 ")" 后面之外，其他的每一行设定最后都要以『 ; 』做为结尾！重要！
* 设定项目语法主要为：『 <参数代号> <设定内容> 』，例如：
* default-lease-time 259200;
* 某些设定项目必须以 option 来设定，基本方式为『 option <参数代码> <设定内容> 』例如：  
  option domain-name "your.domain.name";

如果需要设定固定 IP 的话，那么就必须要知道要设定成固定 IP 的那部计算机的硬件地址 (MAC) 才行。dhcpd.conf 里头的设定主要分为两大项目，一个是服务器运作的整体设定 (Global) 一个是 IP 设定模式 (动态或固定)， 每个项目的设定值大概有底下这几项：
### 整体设定 (Global)
假设你的 dhcpd 只管理一个区段的区网，那么除了 IP 之外的许多网络参数就可以放在整体设定的区域中，这包括有租约期限、DNS 主机的 IP 地址、路由器的 IP 地址还有动态 DNS (DDNS) 更新的类型等等。当固定 IP 及动态 IP 内没有规范到某些设定时，则以整体设定值为准。这些参数的设定名称为：
* default-lease-time 时间：  
  用户的计算机也能够要求一段特定长度的租约时间，但若使用者没有特别要求租约时间的话， 那么就以此为预设的租约时间。后面的时间参数默认单位为秒；
* max-lease-time 时间：  
  与上面的预设租约时间类似，不过，这个设定值是在规范使用者所能要求的最大租约时间。也就是说， 使用者要求的租约时间若超过此设定值，则以此值为准；
* option domain-name "领域名"：  
  如果你在 /etc/resolv.conf 里面设定了一个『 search google.com 』的话，这表示当你要搜寻主机名时， DNS 系统会主动帮你加上这个领域名的意思。
* option domain-name-servers IP1, IP2：  
  这个设定参数可以修改客户端的 /etc/resolv.conf 档案！就是 nameserver 后面接的那个 DNS IP 。特别注意设定参数最末尾为『servers』 (有 s)；
* ddns-update-style 类型：  
  因为 DHCP 客户端所取得的 IP 通常是一直变动的，所以某部主机的主机名与 IP 的对应就很难处理。此时 DHCP 可以透过 ddns 来更新主机名与 IP 的对应。可以将他设定为 none 。
* ignore client-updates：  
  与上一个设定值较相关，客户端可以透过 dhcpd 服务器来更新 DNS 相关的信息。不过，这里我们也先不谈这个， 因此就将它设定为 ignore (忽略) 了。
* option routers 路由器的地址：  
  设定路由器的 IP 所在，记得那个『 routers 』要加 s 
### IP 设定模式 (动态或固定)
由于 dhcpd 主要是针对局域网络来给予 IP 参数的，因此在设定 IP 之前，我们得要指定一个区网才行。 指定区网的方式使用如下的参数：
```
subnet NETWORK_IP netmask NETMASK_IP { ... }
```
括号内的参数就是 IP 是固定还是动态的设定：
* range IP1 IP2：  
  在这个区网当中，给予一个连续的 IP 群用来发放成动态 IP 的设定，那个 IP1 IP2 指的是开放的 IP 范围。 举例来说，你想要开放 192.168.100.101 到 192.168.100.200 这 100 个 IP 用来作为动态分配，那就是： range 192.168.100.101 192.168.100.200;
* host 主机名 { ... };  
  这个 host 就是指定固定 IP 对应到固定 MAC 的设定值，那个主机名可以自己想想再给予即可。 不过在大括号内就得要指定 MAC 与固定的 IP ：
    * hardware ethernet 硬件地址：  
      利用网络卡上面的固定硬件地址来设定，亦即该设定仅针对这个硬件地址有效的意思；
    * fixed-address IP地址：
      给予一个固定的 IP 地址的意思。
## 一个局域网络的 DHCP 服务器设定案例
假设我的环境当中，Linux 主机除了 NAT 服务器之外还得要负责其他服务器，例如邮件服务器的支持。 而在后端局域网络中则想要提供 DHCP 的服务。网络参数设定为：
* Linux 主机对内的 eth1 的 IP 设定为 192.168.100.254 这个；
* 内部网段设定为 192.168.100.0/24 这一段，且内部计算机的 router 为 192.168.100.254 ，此外 DNS 主机的 IP 为中华电信的 168.95.1.1 及 Seednet 的 139.175.10.20 这两个；
* 我想要让每个使用者预设租约为 3 天，最长为 6 天；
* 只想要分配的 IP 只有 192.168.100.101 到 192.168.100.200 这几个，其他的 IP 则保留下来；
* 我还有一部主机，他的 MAC 是『 08:00:27:11:EB:C2 』，我要给他的主机名为 win7 ，且 IP 为 192.168.100.30 。
```
[root@www ~]# vim /etc/dhcp/dhcpd.conf
# 1. 整体的环境设定
ddns-update-style            none;            <==不要更新 DDNS 的设定
ignore client-updates;                        <==忽略客户端的 DNS 更新功能
default-lease-time           259200;          <==预设租约为 3 天
max-lease-time               518400;          <==最大租约为 6 天
option routers               192.168.100.254; <==这就是预设路由
option domain-name           "centos.vbird";  <==给予一个领域名
option domain-name-servers   168.95.1.1, 139.175.10.20;
# 上面是 DNS 的 IP 设定，这个设定值会修改客户端的 /etc/resolv.conf 档案内容

# 2. 关于动态分配的 IP
subnet 192.168.100.0 netmask 255.255.255.0 {
    range 192.168.100.101 192.168.100.200;  <==分配的 IP 范围

    # 3. 关于固定的 IP 
    host win7 {
        hardware ethernet    08:00:27:11:EB:C2; <==客户端网卡 MAC
        fixed-address        192.168.100.30;    <==给予固定的 IP
    }
}
# 相关的设定参数意义，请查询前一小节的介绍，或者 man dhcpd.conf
```
针对 dhcpd 这个执行文件设定他监听的接口， 而不是针对所有的接口都监听，在 CentOS (Red Hat 系统) 可以这样做：
```
[root@www ~]# vim /etc/sysconfig/dhcpd
DHCPDARGS="eth0"
```
不过这个动作在 CentOS 5.x 以后的版本上面已经不需要了，因为新版本的 dhcp 会主动的分析服务器的网段与实际的 dhcpd.conf 设定， 如果两者无法吻合，就会有错误提示。
## DHCP 服务器的启动与观察
启动前的注意事项：
* 你的 Linux 服务器网络环境已经设定好，例如 eth1 已经是 192.168.100.254；
* 你的防火墙规则已经处理好，例如：(1)放行内部区网的联机、(2)iptables.rule 的 NAT 服务已经设定妥当；

另外你要注意的是：dhcpd 使用的埠口是 port 67 ，并且启动的结果会记录在 /var/log/messages 档案内：
```
# 1. 启动后观察一下埠口的变化：
[root@www ~]# /etc/init.d/dhcpd start
[root@www ~]# chkconfig dhcpd on
[root@www ~]# netstat -tlunp | grep dhcp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address Foreign Address PID/Program name
udp        0      0 0.0.0.0:67    0.0.0.0:*       1581/dhcpd

# 2. 固定去看看登录文件的输出信息
[root@www ~]# tail -n 30 /var/log/messages
Jul 27 01:51:24 www dhcpd: Internet Systems Consortium DHCP Server 4.1.1-P1
Jul 27 01:51:24 www dhcpd: Copyright 2004-2010 Internet Systems Consortium.
Jul 27 01:51:24 www dhcpd: All rights reserved.
Jul 27 01:51:24 www dhcpd: For info, please visit https://www.isc.org/software/dhcp/
Jul 27 01:51:24 www dhcpd: WARNING: Host declarations are global.  They are not 
limited to the scope you declared them in.
Jul 27 01:51:24 www dhcpd: Not searching LDAP since ldap-server, ldap-port and 
ldap-base-dn were not specified in the config file
Jul 27 01:51:24 www dhcpd: Wrote 0 deleted host decls to leases file.
Jul 27 01:51:24 www dhcpd: Wrote 0 new dynamic host decls to leases file.
Jul 27 01:51:24 www dhcpd: Wrote 0 leases to leases file.
Jul 27 01:51:24 www dhcpd: Listening on LPF/eth1/08:00:27:2a:30:14/192.168.100.0/24
Jul 27 01:51:24 www dhcpd: Sending on   LPF/eth1/08:00:27:2a:30:14/192.168.100.0/24
....(以下省略)....
```
## 内部主机的 IP 对应
可以将所有可能的计算机 IP 都加进去 /etc/hosts 档案。不过， 更好的解决方案则是架设内部的 DNS 服务器，这样一来，内部的其他 Linux 服务器也不必更改 /etc/hosts 就能够取得每部主机的 IP 与主机名对应。

# DHCP 客户端的设定
## 客户端是 Linux
```
[root@clientlinux ~]# vim /etc/sysconfig/network-scripts/ifcfg-eth0
DEVICE=eth0
NM_CONTROLLED=no
ONBOOT=yes
BOOTPROTO=dhcp  <==指定这一个

[root@clientlinux ~]# /etc/init.d/network restart
```
同时记得要拿掉预设路由的设定。改完之后，就将我们的整个网络重新启动即可 (不要使用 ifdown 与 ifup ，因为还有预设路由要设定)。如果执行的结果有找到正确的 DHCP 主机，那么几个档案可能会被更动：
```
# 1. DNS 的 IP 会被更动，查阅 resolv.conf ：
[root@clientlinux ~]# cat /etc/resolv.conf
search centos.vbird      <==还记得设定过 domain-name 否？
domain centos.vbird      <==还记得设定过 domain-name 否？
nameserver 168.95.1.1    <==这就是我们在 dhcpd.conf内的设定值
nameserver 139.175.10.20

# 2. 观察一下路由
[root@clientlinux ~]# route -n
Kernel IP routing table
Destination    Gateway         Genmask        Flags Metric Ref  Use Iface
192.168.100.0  0.0.0.0         255.255.255.0  U     0      0      0 eth0
0.0.0.0        192.168.100.254 0.0.0.0        UG    0      0      0 eth0
# 路由也被正确的捉到了

# 3. 察看一下客户端的指令
[root@clientlinux ~]# netstat -tlunp | grep dhc
Proto Recv-Q Send-Q Local Address  Foreign Address State  PID/Program name
udp        0      0 0.0.0.0:68     0.0.0.0:*              1694/dhclient
# 确实是有个小程序在监测 DHCP 的联机状态

# 4. 客户端租约所记载的信息
[root@clientlinux ~]# cat /var/lib/dhclient/dhclient*
lease {
  interface "eth0";
  fixed-address 192.168.100.101; <==取得的 IP 
  option subnet-mask 255.255.255.0;
  option routers 192.168.100.254;
  option dhcp-lease-time 259200;
  option dhcp-message-type 5;
  option domain-name-servers 168.95.1.1,139.175.10.20;
  option dhcp-server-identifier 192.168.100.254;
  option domain-name "centos.vbird";
  renew 4 2011/07/28 05:01:24; <==下一次预计更新 (renew) 的时间点
  rebind 5 2011/07/29 09:06:36;
  expire 5 2011/07/29 18:06:36;
}
# 这个档案会记录该适配卡所曾经要求过的 DHCP 信息，与设定的 /etc/dhcp/dhcpd.conf 类似
```

客户端取得的数据都被记载在 /var/lib/dhclient/dhclient*-eth0.leases 里，如果有多张网卡，那么每张网卡自己的 DHCP 要求就会被写入到不同档名的档案当中去。
> 因为上头的 dhclient-eth0.leases 里面的 fixed-address 指定了想要固定 IP 的选项，所以这部客户端 clientlinux.centos.vbird 每次都能够取得相同的固定 IP 。如果 DHCP 服务器的该 IP 没有被用走，也在规定的 range 设定值内，那就会发放给你这个 IP 了。如果你想要不同的 IP ，就将你想要的 IP 取代上述的设定值。
## 客户端是 Windows
```
C:\Users\win7> ipconfig /all
....(前面省略)....
以太网络卡 区域联机:

   联机特定 DNS 后缀 . . . . . . . . : centos.vbird
   描述 . . . . . . . . . . . . . . .: Intel(R) PRO/1000 MT Desktop Adapter
   实体地址 . . . . . . . . . . . . .: 08-00-27-11-EB-C2
   DHCP 已启用 . . . . . . . . . . . : 是
   自动设定启用 . . . . . . . . . . .: 是
   链接-本机 IPv6 地址 . . . . . . . : fe80::ec92:b907:bc2a:a5fa%11(偏好选项)
   IPv4 地址 . . . . . . . . . . . . : 192.168.100.30(偏好选项) <==这是取得的IP
   子网掩码 . . . . . . . . . . . .: 255.255.255.0
   租用取得 . . . . . . . . . . . . .: 2011年7月27日 上午 11:59:18 <==这是租约
   租用到期 . . . . . . . . . . . . .: 2011年7月30日 上午 11:59:18
   预设网关 . . . . . . . . . . . . .: 192.168.100.254
   DHCP 服务器 . . . . . . . . . . . : 192.168.100.254  <==这一部 DHCP 服务器
   DNS 服务器 . . . . . . . . . . . .: 168.95.1.1       <==取得的 DNS
                                       139.175.10.20
   NetBIOS over Tcpip . . . . . . . .: 启用

C:\Users\win7> ipconfig /renew
# 这样可以立即要求更新 IP 信息
```

# DHCP 服务器端进阶观察与使用
## 检查租约档案
```
[root@www ~]# cat /var/lib/dhcpd/dhcpd.leases
lease 192.168.100.101 {
  starts 2 2011/07/26 18:06:36;  <==租约开始日期
  ends 5 2011/07/29 18:06:36;    <==租约结束日期
  tstp 5 2011/07/29 18:06:36;
  cltt 2 2011/07/26 18:06:36;
  binding state active;
  next binding state free;
  hardware ethernet 08:00:27:34:4e:44; <==客户端网卡
}
```
从这个档案里面我们就知道有多少客户端已经向我们申请了 DHCP 的 IP 使用了。
## 让大量的 PC 都具有固定 IP 的脚本
```
[root@www ~]# vim setup_dhcpd.conf
#!/bin/bash
read -p "Do you finished the IP's settings in every client (y/n)? " yn
read -p "How many PC's in this class (ex> 60)? " num
if [ "$yn" = "y" ]; then
        for site in $(seq 1 ${num})
        do
                siteip="192.168.100.${site}"
                allip="$allip $siteip"
                ping -c 1 -w 1 $siteip > /dev/null 2>&1
                if [ "$?" == "0" ]; then
                        okip="$okip $siteip"
                else
                        errorip="$errorip $siteip"
                        echo "$siteip is DOWN"
                fi
        done
        [ -f dhcpd.conf ] && rm dhcpd.conf
        for site in $allip
        do
                pcname=pc$(echo $site | cut -d '.' -f 4)
                mac=$(arp -n | grep "$site " | awk '{print $3}')
                echo "  host $pcname {"
                echo "          hardware ethernet ${mac};"
                echo "          fixed-address     ${site};"
                echo "  }"
                echo "  host $pcname {"                         >> dhcpd.conf
                echo "          hardware ethernet ${mac};"      >> dhcpd.conf
                echo "          fixed-address     ${site};"     >> dhcpd.conf
                echo "  }"                                      >> dhcpd.conf
        done
fi
echo "You can use dhcpd.conf (this directory) to modified your /etc/dhcp/dhcpd.conf"
echo "Finished."
```
## 使用 ether-wake 实行远程自动开机 (remote boot)
既然已经知道客户端的 MAC 地址了，如果客户端的主机符合一些电源标准， 并且该客户端主机所使用之网络卡暨主板支持网络唤醒的功能时，我们就可以透过网络来让客户端计算机开机了。 如果你有一部主机想要让他可以透过网络来启动时，你必须要在这部客户端计算机上进行：
1. 首先你得要在 BIOS 里面设定『网络唤醒』的功能，否则是没有用的
2. 再来你必须要让这部主机接上网络线，并且电源也是接通的。
3. 将这部主机的 MAC 抄下来，然后关机等待网络唤醒。

接下来请到永远开着的主机 DHCP 服务器上面 (其实只要任何一部 Linux 主机均可！) ，安装 net-tools 这个软件后， 就会取得 ether-wake 这个指令，这就是网络唤醒的主要功能。假设客户端主机的 MAC 为 11:22:33:44:55:66 并且与我的服务器 eth1 相连接：
```
[root@www ~]# ether-wake -i eth1 11:22:33:44:55:66

# 更多功能可以这样查阅
[root@www ~]# ether-wake -u
```
## DHCP 与 DNS 的关系
局域网络内如果很多 Linux 服务器时，你得要将 private IP 加入到每部主机的 /etc/hosts 里面， 这样在联机阶段的等待时间才不会有逾时或者是等待太久的问题。如果计算机数量太大，又有很多测试机时，此时在区网内架设一部 DNS 服务器负责主机名解析就很重要。  
理论上，我们应该在 DHCP 服务器主机上面在安装一个 DNS 服务器，提供内部计算机的名称解析为宜。 
### DHCP 响应速度与有网管 switch 的设定问题
