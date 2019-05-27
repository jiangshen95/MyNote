# 认识防火墙
防火墙就是透过订定一些有顺序的规则，并管制进入到我们网域内的主机 (或者可以说是网域) 数据封包的一种机制。更广义的来说，只要能够分析与过滤进出我们管理之网域的封包数据， 就可以称为防火墙。  
防火墙又可以分为硬件防火墙与本机的软件防火墙。硬件防火墙是由厂商设计好的主机硬件， 这部硬件防火墙内的操作系统主要以提供封包数据的过滤机制为主，并将其他不必要的功能拿掉。因为单纯作为防火墙功能而已， 因此封包过滤的效率较佳。软件防火墙本身就是在保护系统网络安全的一套软件(或称为机制)，例如 Netfilter 与 TCP Wrappers 都可以称为软件防火墙。  
本章主要介绍 Linux 系统本身提供的软件防火墙的功能 Netfilter 。
## 开始之前的提醒事项
虽然 Netfilter 机制可以透过 iptables 指令的方式来进行规则的排序与修改，不过建议利用 shell script 来撰写属于你自己的防火墙机制比较好，因为对于规则的排序与汇整有比较好的观察性， 可以让你的防火墙规则比较清晰一点。
## 为何需要防火墙
封包进入本机时，会通过防火墙、服务器软件程序、SELinux与文件系统等。所以基本上，如果你的系统 (1)已经关闭不需要而且危险的服务； (2)已经将整个系统的所有软件都保持在最新的状态； (3)权限设定妥当且定时进行备份工作； (4)已经教育用户具有良好的网络、系统操作习惯。 那么你的系统实际上已经颇为安全了。  
防火墙最大的功能就是帮助你『限制某些服务的存取来源』。例如，(1)你可以限制文件传输服务 (FTP) 只在子域内的主机才能够使用，而不对整个 Internet 开放； (2)你可以限制整部 Linux 主机仅可以接受客户端的 WWW 要求，其他的服务都关闭； (3)你还可以限制整部主机仅能主动对外联机。反过来说，若有客户端对我们主机发送主动联机的封包状态 (TCP 封包的 SYN flag) 就予以抵挡等等。这些就是最主要的防火墙功能。  
防火墙最重要的任务就是规划出：
* 切割被信任(如子域)与不被信任(如 Internet)的网段；
* 划分出可提供 Internet 的服务与必须受保护的服务；
* 分析出可接受与不可接受的封包状态；

Linux 的 iptables 防火墙软件还可以进行更细部深入的 NAT (Network Address Translation) 的设定，并进行更弹性的 IP 封包伪装功能，不过，对于单一主机的防火墙来说， 最简单的任务还是上面那三项。必须要知道『你的系统哪些数据与服务需要保护』，针对需要受保护的服务来设定防火墙的规则。
## Linux 系统上防火墙的主要类别
依据防火墙管理的范围，我们可以将防火墙区分为网域型与单一主机型的控管。在单一主机型的控管方面， 主要的防火墙有封包过滤型的 Netfilter 与依据服务软件程序作为分析的 TCP Wrappers 两种。若以区域型的防火墙而言， 由于此类防火墙都是当作路由器角色，因此防火墙类型主要则有封包过滤的 Netfilter 与利用代理服务器 (proxy server) 进行存取代理的方式了。
### Netfilter (封包过滤机制)
所谓的封包过滤，亦即是分析进入主机的网络封包，将封包的表头数据捉出来进行分析，以决定该联机为放行或抵挡的机制。 由于这种方式可以直接分析封包表头数据，所以包括硬件地址(MAC), 软件地址 (IP), TCP, UDP, ICMP 等封包的信息都可以进行过滤分析的功能，因此用途非常的广泛。(主要分析的是 OSI 七层协议的 2，3，4 层)  
在 Linux 上面我们使用核心内建的 Netfilter 这个机制，而 Netfilter 提供了 iptables 这个软件来作为防火墙封包过滤的指令。由于 Netfilter 是核心内建的功能，因此他的效率非常的高，适合一般小型环境的设定。
### TCP Wrapper (程序管控)
透过服务器程序的外挂 (tcpd) 来处置的。与封包过滤不同的是， 这种机制主要是分析谁对某程序进行存取，然后透过规则去分析该服务器程序谁能够联机、谁不能联机。 由于主要是透过分析服务器程序来控管，因此与启动的埠口无关，只与程序的名称有关。举例来说，我们知道 FTP 可以启动在非正规的 port 21 进行监听，当你透过 Linux 内建的 TCP wrappers 限制 FTP 时， 那么你只要知道 FTP 的软件名称 (vsftpd) ，然后对他作限制，则不管 FTP 启动在哪个埠口，都会被该规则管理的。
### Proxy (代理服务器)
其实代理服务器是一种网络服务，它可以『代理』用户的需求，而代为前往服务器取得相关的资料。  
当有人想要攻击 client 端的主机时， 除非他能够攻破 Proxy server ，否则是无法与 client 联机的。  
一般 proxy 主机通常仅开放 port 80, 21, 20 等 WWW 与 FTP 的埠口而已，而且通常 Proxy 就架设在路由器上面，因此可以完整的掌控局域网络内的对外联机。
## 防火墙的一般网络布线示意
防火墙除了可以『保护防火墙机制本身所在的那部主机』之外，还可以『保护防火墙后面的主机』。也就是说，防火墙除了可以防备本机被入侵之外， 他还可以架设在路由器上面藉以控管进出本地端网域的网络封包。这种规划对于内部私有网域的安全也有一定程度的保护作用。
### 单一网域，仅有一个路由器：
防火墙除了可以作为 Linux 本机的基本防护之外，他还可以架设在路由器上面以管控整个局域网络的封包进出。 因此，在这类的防火墙上头通常至少需要有两个接口，将可信任的内部与不可信任的 Internet 分开， 所以可以分别设定两块网络接口的防火墙规则。  
想要将局域网络控管的更严格的话，那你甚至可以在这部 Linux 防火墙上面架设更严格的代理服务器， 让客户端仅能连上你所开放的 WWW 服务器而已，而且还可以透过代理服务器的登录文件分析功能， 明确的查出来那个使用者在某个时间点曾经连上哪些 WWW 服务器。如果在这个防火墙上面再加装类似 MRTG 的流量监控软件，还能针对整个网域的流量进行监测。这样配置的优点是：
* 因为内外网域已经分开，所以安全维护在内部可以开放的权限较大
* 安全机制的设定可以针对 Linux 防火墙主机来维护即可
* 对外只看的到 Linux 防火墙主机，所以对于内部可以达到有效的安全防护
### 内部网络包含安全性更高的子网，需内部防火墙切开子网：
将 LAN 里面再加设一个防火墙，将安全等级分类，那么将会让你的重要数据获得更佳的保护。
### 在防火墙的后面架设网络服务器主机
将提供网络服务的服务器放在防火墙后面，Web, Mail 与 FTP 都是透过防火墙连到 Internet 上面去，所以， 底下这四部主机在 Internet 上面的 Public IP 都是一样的。只是透过防火墙的封包分析后，将 WWW 的要求封包转送到 Web 主机，将 Mail 送给 Mail Server 去处理而已(透过 port 的不同来转递)。  
当有攻击者想要入侵你的 FTP 主机好了，他使用各种分析方法去进攻的主机，其实是『防火墙』那一部， 攻击者想要攻击你内部的主机，除非他能够成功的搞定你的防火墙，否则就很难入侵你的内部主机。  
而且，由于主机放置在两部防火墙中间，内部网络如果发生状况时 (例如某些使用者不良操作导致中毒啊、 被社交工程攻陷导致内部主机被绑架啊等等的) ，是不会影响到网络服务器的正常运作的。 这种方式适用在比较大型的企业当中，因为对这些企业来说，网络主机能否提供正常稳定的服务是很重要的。  
将网络服务器独立放置在两个防火墙中间的网络，我们称之为非军事区域 (DMZ)。 DMZ 的目的就如同前面提到的，重点在保护服务器本身，所以将 Internet 与 LAN 都隔离开来，如此一来不论是服务器本身，或者是 LAN 被攻陷时，另一个区块还是完好无缺的。
## 防火墙的使用限制
可以进行的分析工作主要有：
* 拒绝让 Internet 的封包进入主机的某些端口
* 拒绝让某些来源 IP 的封包进入
* 拒绝让带有某些特殊旗标 (flag) 的封包进入  
  最常拒绝的就是带有 SYN 的主动联机的旗标了
* 分析硬件地址 (MAC) 来决定联机与否

局限性：
* 防火墙并不能很有效的抵挡病毒或木马程序  
  假设你已经开放了 WWW 的服务，那么你的 WWW 主机上面，防火墙一定得要将 WWW 服务的 port 开放给 Client 端登入，也就是说，只要进入你的主机的封包是要求 WWW 数据的，就可以通过你的防火墙。那好了，『万一你的 WWW 服务器软件有漏洞，或者本身向你要求 WWW 服务的该封包就是病毒在侦测你的系统』时，防火墙不起作用。
* 防火墙对于来自内部 LAN 的攻击较无承受力

# TCP Wrappers
TCP wrappers 是透过客户端想要链接的程序文件名，然后分析客户端的 IP ，看看是否需要放行。
## 支持 TCP Wrappers 的服务
TCP wrappers 就是透过 /etc/hosts.allow, /etc/hosts.deny 管理的一个类似防火墙的机制， 但并非所有的软件都可以透过这两个档案来控管，只有底下的软件才能够透过这两个档案来管理防火墙规则，分别是：
* 由 super daemon (xinetd) 所管理的服务；
* 有支援 libwrap.so 模块的服务。
## /etc/hosts.{allow|deny} 的设定方式
```
<service(program_name)> : <IP, domain, hostname> 
<服务   (亦即程序名称)> : <IP 或领域 或主机名>
# 上头的 > < 是不存在于配置文件中的
```
* 先以 /etc/hosts.allow 为优先比对，该规则符合就予以放行；
* 再以 /etc/hosts.deny 比对，规则符合就予以抵挡；
* 若不在这两个档案内，亦即规则都不符合，最终则予以放行。

# Linux 的封包过滤软件：iptables
## 不同 Linux 核心版本的防火墙软件
## 封包进入流程：规则顺序的重要性
因为 iptables 是利用封包过滤的机制， 所以他会分析封包的表头数据。根据表头数据与定义的『规则』来决定该封包是否可以进入主机或者是被丢弃。 意思就是说：『根据封包的分析资料 "比对" 你预先定义的规则内容， 若封包数据与规则内容相同则进行动作，否则就继续下一条规则的比对！』 重点在那个『比对与分析顺序』上。  
当一个网络封包要进入到主机之前，会先经由 NetFilter 进行检查，那就是 iptables 的规则了。 检查通过则接受 (ACCEPT) 进入本机取得资源，如果检查不通过，则可能予以丢弃 (DROP) 。『规则是有顺序的』，例如当网络封包进入 Rule 1 的比对时， 如果比对结果符合 Rule 1 ，此时这个网络封包就会进行 Action 1 的动作，而不会理会后续的 Rule 2, Rule 3.... 等规则的分析了。  
而如果这个封包并不符合 Rule 1 的比对，那就会进入 Rule 2 的比对了！如此一个一个规则去进行比对就是了。 那如果所有的规则都不符合，此时就会透过预设动作 (封包政策, Policy) 来决定这个封包的去向。  
假设你的 Linux 主机提供了 WWW 的服务，那么自然就要针对 port 80 来启用通过的封包规则，但是你发现 IP 来源为 192.168.100.100 老是恶意的尝试入侵你的系统，所以你想要将该 IP 拒绝往来，最后，所有的非 WWW 的封包都给他丢弃，就这三个规则来说：
1. Rule 1 先抵挡 192.168.100.100 ；
2. Rule 2 再让要求 WWW 服务的封包通过；
3. Rule 3 将所有的封包丢弃。

万一你的顺序排错了，变成：
1. Rule 1 先让要求 WWW 服务的封包通过；
2. Rule 2 再抵挡 192.168.100.100 ；
3. Rule 3 将所有的封包丢弃。

192.168.100.100 只要对你的主机送出 WWW 要求封包，就可以使用你的 WWW 功能了，因为你的规则顺序定义第一条就会让他通过，而不去考虑第二条规则。
## iptables 的表格 (table) 与链 (chain)
防火墙软件里面有多个表格 (table) ，每个表格都定义出自己的默认政策与规则， 且每个表格的用途都不相同。预设的情况下，Linux 的 iptables 至少就有三个表格，包括管理本机进出的 filter 、管理后端主机 (防火墙内部的其他计算机) 的 nat 、管理特殊旗标使用的 mangle (较少使用) 。更有甚者，我们还可以自定义额外的链。每个表格与其中链的用途分别是这样的：
* filter (过滤器)：主要跟进入 Linux 本机的封包有关，这个是预设的 table
    * INPUT：主要与想要进入我们 Linux 本机的封包有关；
    * OUTPUT：主要与我们 Linux 本机所要送出的封包有关；
    * FORWARD：与 Linux 本机比较没有关系， 可以『转递封包』到后端的计算机中，与下列 nat table 相关性较高。
* nat (地址转换)：是 Network Address Translation 的缩写， 这个表格主要在进行来源与目的之 IP 或 port 的转换，与 Linux 本机较无关，主要与 Linux 主机后的局域网络内计算机较有相关。
    * PREROUTING：在进行路由判断之前所要进行的规则(DNAT/REDIRECT)
    * POSTROUTING：在进行路由判断之后所要进行的规则(SNAT/MASQUERADE)
    * OUTPUT：与发送出去的封包有关
* mangle (破坏者)：这个表格主要是与特殊的封包的路由旗标有关， 早期仅有 PREROUTING 及 OUTPUT 链，不过从 kernel 2.4.18 之后加入了 INPUT 及 FORWARD 链。 由于这个表格与特殊旗标相关性较高，所以像咱们这种单纯的环境当中，较少使用 mangle 这个表格。

如果你的 Linux 是作为 www 服务，那么要开放客户端对你的 www 要求有响应，就得要处理 filter 的 INPUT 链； 而如果你的 Linux 是作为局域网络的路由器，那么就得要分析 nat 的各个链以及 filter 的 FORWARD 链才行。  
iptables 可以控制三种封包的流向：
* 封包进入 Linux 主机使用资源： 在路由判断后确定是向 Linux 主机要求数据的封包，主要就会透过 filter 的 INPUT 链来进行控管；
* 封包经由 Linux 主机的转递，没有使用主机资源，而是向后端主机流动： 在路由判断之前进行封包表头的修订作业后，发现到封包主要是要透过防火墙而去后端。 也就是说，该封包的目标并非我们的 Linux 本机。主要经过的链是 filter 的 FORWARD 以及 nat 的 POSTROUTING, PREROUTING。
* 封包由 Linux 本机发送出去： 例如响应客户端的要求，或者是 Linux 本机主动送出的封包。先是透过路由判断， 决定了输出的路径后，再透过 filter 的 OUTPUT 链来传送的。当然，最终还是会经过 nat 的 POSTROUTING 链。

事实上与本机最有关的其实是 filter 这个表格内的 INPUT 与 OUTPUT 这两条链，如果你的 iptables 只是用来保护 Linux 主机本身的话，那 nat 的规则就不是很需要，直接设定为开放即可。  
不过，如果你的防火墙事实上是用来管制 LAN 内的其他主机的话，那么你就必须要再针对 filter 的 FORWARD 这条链，还有 nat 的 PREROUTING, POSTROUTING 以及 OUTPUT 进行额外的规则订定才行。 nat 表格的使用需要很清晰的路由概念才能够设定的好。
## 本机的 iptables 语法
iptables 至少有三个预设的 table (filter, nat, mangle)，较常用的是本机的 filter 表格， 这也是默认表格。由于不同的 table 他们的链不一样，导致使用的指令语法或多或少都有点差异。
### 规则的观察与清除
如果你在安装的时候选择没有防火墙的话，那么 iptables 在一开始的时候应该是没有规则的，不过， 可能因为你在安装的时候就有选择系统自动帮你建立防火墙机制，那系统就会有默认的防火墙规则。
```
[root@www ~]# iptables [-t tables] [-L] [-nv]
选项与参数：
-t ：后面接 table ，例如 nat 或 filter ，若省略此项目，则使用默认的 filter
-L ：列出目前的 table 的规则
-n ：不进行 IP 与 HOSTNAME 的反查，显示讯息的速度会快很多！
-v ：列出更多的信息，包括通过该规则的封包总位数、相关的网络接口等

范例：列出 filter table 三条链的规则
[root@www ~]# iptables -L -n
Chain INPUT (policy ACCEPT)   <==针对 INPUT 链，且预设政策为可接受
target  prot opt source     destination <==说明栏
ACCEPT  all  --  0.0.0.0/0  0.0.0.0/0   state RELATED,ESTABLISHED <==第 1 条规则
ACCEPT  icmp --  0.0.0.0/0  0.0.0.0/0                             <==第 2 条规则
ACCEPT  all  --  0.0.0.0/0  0.0.0.0/0                             <==第 3 条规则
ACCEPT  tcp  --  0.0.0.0/0  0.0.0.0/0   state NEW tcp dpt:22      <==以下类推
REJECT  all  --  0.0.0.0/0  0.0.0.0/0   reject-with icmp-host-prohibited

Chain FORWARD (policy ACCEPT)  <==针对 FORWARD 链，且预设政策为可接受
target  prot opt source     destination
REJECT  all  --  0.0.0.0/0  0.0.0.0/0   reject-with icmp-host-prohibited

Chain OUTPUT (policy ACCEPT)  <==针对 OUTPUT 链，且预设政策为可接受
target  prot opt source     destination

范例：列出 nat table 三条链的规则
[root@www ~]# iptables -t nat -L -n
Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination

Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```
Chain 那一行里面括号的 policy 就是预设的政策，底下的代表：
* target：代表进行的动作， ACCEPT 是放行，而 REJECT 则是拒绝，此外，尚有 DROP (丢弃) 的项目！
* prot：代表使用的封包协议，主要有 tcp, udp 及 icmp 三种封包格式；
* opt：额外的选项说明
* source ：代表此规则是针对哪个『来源 IP』进行限制？
* destination ：代表此规则是针对哪个『目标 IP』进行限制？

iptables-save 会列出完整的防火墙规则，不过没有格式化输出：
```
[root@www ~]# iptables-save [-t table]
选项与参数：
-t ：可以仅针对某些表格来输出，例如仅针对 nat 或 filter 等等

[root@www ~]# iptables-save
# Generated by iptables-save v1.4.7 on Fri Jul 22 15:51:52 2011
*filter                      <==星号开头的指的是表格，这里为 filter
:INPUT ACCEPT [0:0]          <==冒号开头的指的是链，三条内建的链
:FORWARD ACCEPT [0:0]        <==三条内建链的政策都是 ACCEPT 
:OUTPUT ACCEPT [680:100461]
-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT <==针对 INPUT 的规则
-A INPUT -p icmp -j ACCEPT
-A INPUT -i lo -j ACCEPT  <==这条很重要！针对本机内部接口开放！
-A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
-A INPUT -j REJECT --reject-with icmp-host-prohibited
-A FORWARD -j REJECT --reject-with icmp-host-prohibited <==针对 FORWARD 的规则
COMMIT
# Completed on Fri Jul 22 15:51:52 2011
```
清除规则：
```
[root@www ~]# iptables [-t tables] [-FXZ]
选项与参数：
-F ：清除所有的已订定的规则；
-X ：杀掉所有使用者 "自定义" 的 chain (应该说的是 tables ）
-Z ：将所有的 chain 的计数与流量统计都归零

范例：清除本机防火墙 (filter) 的所有规则
[root@www ~]# iptables -F
[root@www ~]# iptables -X
[root@www ~]# iptables -Z
```
由于这三个指令会将本机防火墙的所有规则都清除，但却不会改变预设政策 (policy) ，如果不是在本机下达这三行指令时，可能会被挡在门外 (若 INPUT 设定为 DROP 时)。
### 定义预设政策 (policy)
『 当你的封包不在你设定的规则之内时，则该封包的通过与否，是以 Policy 的设定为准』，在本机方面的预设政策中，假设你对于内部的使用者有信心的话， 那么 filter 内的 INPUT 链方面可以定义的比较严格一点，而 FORWARD 与 OUTPUT 则可以订定的松一些。通常可以将 INPUT 的 policy 定义为 DROP ，其他两个则定义为 ACCEPT。
```
[root@www ~]# iptables [-t nat] -P [INPUT,OUTPUT,FORWARD] [ACCEPT,DROP]
选项与参数：
-P ：定义政策( Policy )。注意，这个 P 为大写
ACCEPT ：该封包可接受
DROP   ：该封包直接丢弃，不会让 client 端知道为何被丢弃。

范例：将本机的 INPUT 设定为 DROP ，其他设定为 ACCEPT
[root@www ~]# iptables -P INPUT   DROP
[root@www ~]# iptables -P OUTPUT  ACCEPT
[root@www ~]# iptables -P FORWARD ACCEPT
[root@www ~]# iptables-save
# Generated by iptables-save v1.4.7 on Fri Jul 22 15:56:34 2011
*filter
:INPUT DROP [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
COMMIT
# Completed on Fri Jul 22 15:56:34 2011
# 由于 INPUT 设定为 DROP 而又尚未有任何规则，所以上面的输出结果显示：
# 所有的封包都无法进入你的主机！是不通的防火墙设定！(网络联机是双向的)
```
### 封包的基础比对：IP, 网域及接口装置
```
[root@www ~]# iptables [-AI 链名] [-io 网络接口] [-p 协议] \
> [-s 来源IP/网域] [-d 目标IP/网域] -j [ACCEPT|DROP|REJECT|LOG]
选项与参数：
-AI 链名：针对某的链进行规则的 "插入" 或 "累加"
    -A ：新增加一条规则，该规则增加在原本规则的最后面。例如原本已经有四条规则，
         使用 -A 就可以加上第五条规则！
    -I ：插入一条规则。如果没有指定此规则的顺序，默认是插入变成第一条规则。
         例如原本有四条规则，使用 -I 则该规则变成第一条，而原本四条变成 2~5 号
    链 ：有 INPUT, OUTPUT, FORWARD 等，此链名称又与 -io 有关，请看底下。

-io 网络接口：设定封包进出的接口规范
    -i ：封包所进入的那个网络接口，例如 eth0, lo 等接口。需与 INPUT 链配合；
    -o ：封包所传出的那个网络接口，需与 OUTPUT 链配合；

-p 协定：设定此规则适用于哪种封包格式
   主要的封包格式有： tcp, udp, icmp 及 all 。

-s 来源 IP/网域：设定此规则之封包的来源项目，可指定单纯的 IP 或包括网域，例如：
   IP  ：192.168.0.100
   网域：192.168.0.0/24, 192.168.0.0/255.255.255.0 均可。
   若规范为『不许』时，则加上 ! 即可，例如：
   -s ! 192.168.100.0/24 表示不许 192.168.100.0/24 之封包来源；

-d 目标 IP/网域：同 -s ，只不过这里指的是目标的 IP 或网域。

-j ：后面接动作，主要的动作有接受(ACCEPT)、丢弃(DROP)、拒绝(REJECT)及记录(LOG)
```
开放 lo 这个本机的接口以及某个 IP 来源：
```
范例：设定 lo 成为受信任的装置，亦即进出 lo 的封包都予以接受
[root@www ~]# iptables -A INPUT -i lo -j ACCEPT
```
上面并没有列出 -s, -d 等等的规则，这表示：不论封包来自何处或去到哪里，只要是来自 lo 这个界面，就予以接受。『没有指定的项目，则表示该项目完全接受』。  
这就是所谓的信任装置，假如你的主机有两张以太网络卡，其中一张是对内部的网域，假设该网卡的代号为 eth1 。如果内部网域是可信任的，那么该网卡的进出封包就通通会被接受，那你就能够用：『iptables -A INPUT -i eth1 -j ACCEPT』 来将该装置设定为信任装置。不过，下达这个指令前要特别注意，因为这样等于该网卡没有任何防备了。
```
范例：只要是来自内网的 (192.168.100.0/24) 的封包通通接受
[root@www ~]# iptables -A INPUT -i eth1 -s 192.168.100.0/24 -j ACCEPT
# 由于是内网就接受，因此也可以称之为『信任网域』

范例：只要是来自 192.168.100.10 就接受，但 192.168.100.230 这个恶意来源就丢弃
[root@www ~]# iptables -A INPUT -i eth1 -s 192.168.100.10 -j ACCEPT
[root@www ~]# iptables -A INPUT -i eth1 -s 192.168.100.230 -j DROP
# 针对单一 IP 来源，可视为信任主机或者是不信任的恶意来源

[root@www ~]# iptables-save
# Generated by iptables-save v1.4.7 on Fri Jul 22 16:00:43 2011
*filter
:INPUT DROP [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [17:1724]
-A INPUT -i lo -j ACCEPT
-A INPUT -s 192.168.100.0/24 -i eth1 -j ACCEPT
-A INPUT -s 192.168.100.10/32 -i eth1 -j ACCEPT
-A INPUT -s 192.168.100.230/32 -i eth1 -j DROP
COMMIT
# Completed on Fri Jul 22 16:00:43 2011
```
如果你想要记录某个规则的纪录：
```
[root@www ~]# iptables -A INPUT -s 192.168.2.200 -j LOG
[root@www ~]# iptables -L -n
target prot opt source         destination
LOG    all  --  192.168.2.200  0.0.0.0/0   LOG flags 0 level 4
```
只要有封包来自 192.168.2.200 这个 IP 时， 那么该封包的相关信息就会被写入到核心讯息，亦即是 /var/log/messages 这个档案当中。 然后该封包会继续进行后续的规则比对。所以说， LOG 这个动作仅在进行记录而已，并不会影响到这个封包的其他规则比对的。
### TCP, UDP 的规则比对：针对埠口设定
```
[root@www ~]# iptables [-AI 链] [-io 网络接口] [-p tcp,udp] \
> [-s 来源IP/网域] [--sport 埠口范围] \
> [-d 目标IP/网域] [--dport 埠口范围] -j [ACCEPT|DROP|REJECT]
选项与参数：
--sport 埠口范围：限制来源的端口号码，端口号码可以是连续的，例如 1024:65535
--dport 埠口范围：限制目标的端口号码。
```
因为仅有 tcp 与 udp 封包具有埠口，因此你想要使用 --dport, --sport 时，得要加上 -p tcp 或 -p udp 的参数才会成功。
```
范例：想要联机进入本机 port 21 的封包都抵挡掉：
[root@www ~]# iptables -A INPUT -i eth0 -p tcp --dport 21 -j DROP

范例：想连到我这部主机的网芳 (upd port 137,138 tcp port 139,445) 就放行
[root@www ~]# iptables -A INPUT -i eth0 -p udp --dport 137:138 -j ACCEPT
[root@www ~]# iptables -A INPUT -i eth0 -p tcp --dport 139 -j ACCEPT
[root@www ~]# iptables -A INPUT -i eth0 -p tcp --dport 445 -j ACCEPT
```
可以利用 UDP 与 TCP 协议所拥有的端口号码来进行某些服务的开放或关闭。还可以综合处理，例如，只要来自 192.168.1.0/24 的 1024:65535 埠口的封包，且想要联机到本机的 ssh port 就予以抵挡，可以这样做：
```
[root@www ~]# iptables -A INPUT -i eth0 -p tcp -s 192.168.1.0/24 \
> --sport 1024:65534 --dport ssh -j DROP
```
除了埠口之外，在 TCP 还有特殊的旗标，最常见的就是那个主动联机的 SYN 旗标。在 iptables 里面还支持『 --syn 』的处理方式：
```
范例：将来自任何地方来源 port 1:1023 的主动联机到本机端的 1:1023 联机丢弃
[root@www ~]# iptables -A INPUT -i eth0 -p tcp --sport 1:1023 \
> --dport 1:1023 --syn -j DROP
```
### iptables 外挂模块：mac 与 state
iptables 可以透过一个状态模块来分析 『这个想要进入的封包是否为刚刚我发出去的响应？』 如果是刚刚我发出去的响应，那么就可以予以接受放行。
```
[root@www ~]# iptables -A INPUT [-m state] [--state 状态]
选项与参数：
-m ：一些 iptables 的外挂模块，主要常见的有：
     state ：状态模块
     mac   ：网络卡硬件地址 (hardware address)
--state ：一些封包的状态，主要有：
     INVALID    ：无效的封包，例如数据破损的封包状态
     ESTABLISHED：已经联机成功的联机状态；
     NEW        ：想要新建立联机的封包状态；
     RELATED    ：这个最常用！表示这个封包是与我们主机发送出去的封包有关

范例：只要已建立或相关封包就予以通过，只要是不合法封包就丢弃
[root@www ~]# iptables -A INPUT -m state \
> --state RELATED,ESTABLISHED -j ACCEPT
[root@www ~]# iptables -A INPUT -m state --state INVALID -j DROP
```
另一个外挂，针对网卡来进行放行与防御：
```
范例：针对局域网络内的 aa:bb:cc:dd:ee:ff 主机开放其联机
[root@www ~]# iptables -A INPUT -m mac --mac-source aa:bb:cc:dd:ee:ff \
> -j ACCEPT
选项与参数：
--mac-source ：就是来源主机的 MAC
```
### ICMP 封包规则的比对：针对是否响应 ping 来设计
ICMP 的类型相当的多，而且很多 ICMP 封包的类型都是为了要用来进行网络检测用的。所以最好不要将所有的 ICMP 封包都丢弃，如果不是做为路由器的主机时，通常我们会把 ICMP type 8 (echo request) 拿掉而已，让远程主机不知道我们是否存在，也不会接受 ping 的响应就是了。ICMP 封包格式的处理是这样的：
```
[root@www ~]# iptables -A INPUT [-p icmp] [--icmp-type 类型] -j ACCEPT
选项与参数：
--icmp-type ：后面必须要接 ICMP 的封包类型，也可以使用代号，
              例如 8  代表 echo request 的意思。

范例：让 0,3,4,11,12,14,16,18 的 ICMP type 可以进入本机：
[root@www ~]# vi somefile
#!/bin/bash
icmp_type="0 3 4 11 12 14 16 18"
for typeicmp in $icmp_type
do
   iptables -A INPUT -i eth0 -p icmp --icmp-type $typeicmp -j ACCEPT
done

[root@www ~]# sh  somefile
```
如果你的主机是作为区网的路由器， 那么建议 icmp 封包还是要通通放行才好，这是因为客户端检测网络时，常常会使用 ping 来测试到路由器的线路是否畅通。
### 简单的客户端防火墙设计与防火墙规则储存
理论上， 应该要有的规则如下：
* 规则归零：清除所有已经存在的规则 (iptables -F...)
* 预设政策：除了 INPUT 这个自定义链设为 DROP 外，其他为预设 ACCEPT；
* 信任本机：由于 lo 对本机来说是相当重要的，因此 lo 必须设定为信任装置；
* 回应封包：让本机主动向外要求而响应的封包可以进入本机 (ESTABLISHED,RELATED)
* 信任用户：这是非必要的，如果你想要让区网的来源可用你的主机资源时
```
[root@www ~]# vim bin/firewall.sh
#!/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin; export PATH

# 1. 清除规则
iptables -F
iptables -X
iptables -Z

# 2. 设定政策
iptables -P   INPUT DROP
iptables -P  OUTPUT ACCEPT
iptables -P FORWARD ACCEPT

# 3~5. 制订各项规则
iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -i eth0 -m state --state RELATED,ESTABLISHED -j ACCEPT
#iptables -A INPUT -i eth0 -s 192.168.1.0/24 -j ACCEPT

# 6. 写入防火墙规则配置文件
/etc/init.d/iptables save

[root@www ~]# sh bin/firewall.sh
iptables: Saving firewall rules to /etc/sysconfig/iptables:[  OK  ]
```
其实防火墙也是一个服务，你可以透过『chkconfig --list iptables』去察看。因此，你这次修改的各种设定想要在下次开机还保存，那就得要进行『 /etc/init.d/iptables save 』这个指令加参数。  
测试：
1. 先由主机向外面主动联机试看看；
2. 再由私有网域内的 PC 向外面主动联机试看看；
3. 最后，由 Internet 上面的主机，主动联机到你的 Linux 主机试看看；
## IPv4 的核心管理功能：/proc/sys/net/ipv4/*
除了 iptables 这个防火墙软件之外，其实 Linux kernel 2.6 提供很多核心预设的攻击抵挡机制。由于是核心的网络功能，所以相关的设定数据都是放置在 /proc/sys/net/ipv4/ 这个目录当中。 至于该目录下各个档案的详细资料，可以参考核心的说明文件 (要先安装 kernel-doc 软件)：
* /usr/share/doc/kernel-doc-2.6.32/Documentation/networking/ip-sysctl.txt
### /proc/sys/net/ipv4/tcp_syncookies
阻断式服务 (DoS) 攻击法当中的一种方式，就是利用 TCP 封包的 SYN 三向交握原理所达成的， 这种方式称为 SYN Flooding 。可以启用核心的 SYN Cookie 模块，预防这种攻击。这个 SYN Cookie 模块可以在系统用来启动随机联机的埠口 (1024:65535) 即将用完时自动启动。  
当启动 SYN Cookie 时，主机在发送 SYN/ACK 确认封包前，会要求 Client 端在短时间内回复一个序号，这个序号包含许多原本 SYN 封包内的信息，包括 IP、port 等。若 Client 端可以回复正确的序号，那么主机就确定该封包为可信的，因此会发送 SYN/ACK 封包，否则就不理会此一封包。  
透过此一机制可以大大的降低无效的 SYN 等待埠口，而避免 SYN Flooding 的 DoS 攻击。启动这个模块：
```
[root@www ~]# echo "1" > /proc/sys/net/ipv4/tcp_syncookies
```
但是这个设定值由于违反 TCP 的三向交握 (因为主机在发送 SYN/ACK 之前需要先等待 client 的序号响应)， 所以可能会造成某些服务的延迟现象，例如 SMTP (mail server)。不适合用在负载已经很高的服务器内，因为负载太高的主机有时会让核心误判遭受 SYN Flooding 的攻击。  
如果是为了系统的 TCP 封包联机优化，则可以参考 tcp_max_syn_backlog, tcp_synack_retries, tcp_abort_on_overflow 这几个设定值的意义。
### /proc/sys/net/ipv4/icmp_echo_ignore_broadcasts
ping flooding (不断发 ping) 及 ping of death (发送大的 ping 封包)。  
可以通过取消 ICMP 类型 8 的 ICMP 封包回应的防火墙设定，抵挡这种攻击。也可以让核心自动取消 ping 的响应。不过，某些局域网络内常见的服务 (例如动态 IP 分配 DHCP 协议) 会使用 ping 的方式来侦测是否有重复的 IP ，所以你最好不要取消所有的 ping 响应比较好。  
核心取消 ping 回应的设定值有两个，分别是：/proc/sys/net/ipv4 内的 icmp_echo_ignore_broadcasts (仅有 ping broadcast 地址时才取消 ping 的回应) 及 icmp_echo_ignore_all (全部的 ping 都不回应)。建议设定 icmp_echo_ignore_broadcasts 就好了：
```
[root@www ~]# echo "1" > /proc/sys/net/ipv4/icmp_echo_ignore_broadcasts
```
### /proc/sys/net/ipv4/conf/网络接口/*
咱们的核心还可以针对不同的网络接口进行不一样的参数设定。网络接口的相关设定放置在 /proc/sys/net/ipv4/conf/ 当中，每个接口都以接口代号做为其代表，例如 eth0 接口的相关设定数据在 /proc/sys/net/ipv4/conf/eth0/ 内。网络接口的设定数据需要注意的有：
* rp_filter：称为逆向路径过滤 (Reverse Path Filtering)， 可以藉由分析网络接口的路由信息配合封包的来源地址，来分析该封包是否为合理。举例来说，你有两张网卡，eth0 为 192.168.1.10/24 ，eth1 为 public IP 。那么当有一个封包自称来自 eth1 ，但是其 IP 来源为 192.168.1.200 ， 那这个封包就不合理，应予以丢弃。这个设定值建议可以启动的。
* log_martians：这个设定数据可以用来启动记录不合法的 IP 来源， 举例来说，包括来源为 0.0.0.0、127.x.x.x、及 Class E 的 IP 来源，因为这些来源的 IP 不应该应用于 Internet 。 记录的数据默认放置到核心放置的登录档 /var/log/messages。
* accept_source_route：或许某些路由器会启动这个设定值， 不过目前的设备很少使用到这种来源路由，你可以取消这个设定值。
* accept_redirects：当你在同一个实体网域内架设一部路由器， 但这个实体网域有两个 IP 网域，例如 192.168.0.0/24, 192.168.1.0/24。此时你的 192.168.0.100 想要向 192.168.1.100 传送讯息时，路由器可能会传送一个 ICMP redirect 封包告知 192.168.0.100 直接传送数据给 192.168.1.100 即可，而不需透过路由器。因为 192.168.0.100 与 192.168.1.100确实是在同一个实体线路上 (两者可以直接互通)，所以路由器会告知来源 IP 使用最短路径去传递数据。但那两部主机在不同的 IP 段，却是无法实际传递讯息的。这个设定也可能会产生一些轻微的安全风险，所以建议关闭他。
* send_redirects：与上一个类似，只是此值为发送一个 ICMP redirect 封包。 同样建议关闭。

可以使用『 echo "1" > /proc/sys/net/ipv4/conf/???/rp_filter 』之类的方法来启动这个项目，不过，哥比较建议修改系统设定值，那就是 /etc/sysctl.conf 这个档案。假设我们仅有 eth0 这个以太接口，而且上述的功能要启动， 可以这样做：
```
[root@www ~]# vim /etc/sysctl.conf
# Adding by VBird 2011/01/28
net.ipv4.tcp_syncookies = 1
net.ipv4.icmp_echo_ignore_broadcasts = 1
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1
net.ipv4.conf.eth0.rp_filter = 1
net.ipv4.conf.lo.rp_filter = 1
....(以下省略)....

[root@www ~]# sysctl -p
```

# 单机防火墙的一个实例
建议通过脚本来撰写防火墙， 然后透过最终的 /etc/init.d/iptables save 来将结果储存到 /etc/sysconfig/iptables 去。还可以用在呼叫其他的 scripts ，可以让防火墙规则具有较为灵活的使用方式。
## 规则草拟
假设网络接口有：
* 外部网络使用 eth0 (如果是拨接，有可能是 ppp0，请针对你的环境来设定)；
* 内部网络使用 eth1 ，且内部使用 192.168.100.0/24 这个 Class ；
* 主机默认开放的服务有 WWW, SSH, https 等等；

由于希望将信任网域 (LAN) 与不信任网域 (Internet) 整个分开的完整一点， 所以希望你可以在 Linux 上面安装两块以上的实体网卡，将两块网卡接在不同的网域，这样可以避免很多问题。 最重要的防火墙政策是：『关闭所有的联机，仅开放特定的服务』模式。 而且假设内部使用者已经受过良好的训练，因此在 filter table 的三条链个预设政策是：
* INPUT 为 DROP
* OUTPUT 及 FORWARD 为 ACCEPT

原则上，内部 LAN 主机与主机本身的开放度很高，因为 Output 与 Forward 是完全开放不理的，对于小家庭的主机是可以接受的，因为我们内部的计算机数量不多，而且人员都是熟悉的， 所以不需要特别加以控管。但是：『在大企业的内部，这样的规划是很不合格的， 因为你不能保证内部所有的人都可以按照你的规定来使用 Network 』，那样的环境连 Output 与 Forward 都需要特别加以管理才行。
## 实际设定
将整个 script 拆成三部分，分别是：
* iptables.rule：设定最基本的规则，包括清除防火墙规则、加载模块、设定服务可接受等；
* iptables.deny：设定抵挡某些恶意主机的进入；
* iptables.allow：设定允许某些自定义的后门来源主机
```
[root@www ~]# mkdir -p /usr/local/virus/iptables
[root@www ~]# cd /usr/local/virus/iptables
[root@www iptables]# vim iptables.rule
#!/bin/bash

# 请先输入您的相关参数，不要输入错误了！
  EXTIF="eth0"             # 这个是可以连上 Public IP 的网络接口
  INIF="eth1"              # 内部 LAN 的连接接口；若无则写成 INIF=""
  INNET="192.168.100.0/24" # 若无内部网域接口，请填写成 INNET=""
  export EXTIF INIF INNET

# 第一部份，针对本机的防火墙设定！##########################################
# 1. 先设定好核心的网络功能：
  echo "1" > /proc/sys/net/ipv4/tcp_syncookies
  echo "1" > /proc/sys/net/ipv4/icmp_echo_ignore_broadcasts
  for i in /proc/sys/net/ipv4/conf/*/{rp_filter,log_martians}; do
        echo "1" > $i
  done
  for i in /proc/sys/net/ipv4/conf/*/{accept_source_route,accept_redirects,\
send_redirects}; do
        echo "0" > $i
  done

# 2. 清除规则、设定默认政策及开放 lo 与相关的设定值
  PATH=/sbin:/usr/sbin:/bin:/usr/bin:/usr/local/sbin:/usr/local/bin; export PATH
  iptables -F
  iptables -X
  iptables -Z
  iptables -P INPUT   DROP
  iptables -P OUTPUT  ACCEPT
  iptables -P FORWARD ACCEPT
  iptables -A INPUT -i lo -j ACCEPT
  iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT

# 3. 启动额外的防火墙 script 模块
  if [ -f /usr/local/virus/iptables/iptables.deny ]; then
        sh /usr/local/virus/iptables/iptables.deny
  fi
  if [ -f /usr/local/virus/iptables/iptables.allow ]; then
        sh /usr/local/virus/iptables/iptables.allow
  fi
  if [ -f /usr/local/virus/httpd-err/iptables.http ]; then
        sh /usr/local/virus/httpd-err/iptables.http
  fi

# 4. 允许某些类型的 ICMP 封包进入
  AICMP="0 3 3/4 4 11 12 14 16 18"
  for tyicmp in $AICMP
  do
    iptables -A INPUT -i $EXTIF -p icmp --icmp-type $tyicmp -j ACCEPT
  done

# 5. 允许某些服务的进入，请依照你自己的环境开启
# iptables -A INPUT -p TCP -i $EXTIF --dport  21 --sport 1024:65534 -j ACCEPT # FTP
# iptables -A INPUT -p TCP -i $EXTIF --dport  22 --sport 1024:65534 -j ACCEPT # SSH
# iptables -A INPUT -p TCP -i $EXTIF --dport  25 --sport 1024:65534 -j ACCEPT # SMTP
# iptables -A INPUT -p UDP -i $EXTIF --dport  53 --sport 1024:65534 -j ACCEPT # DNS
# iptables -A INPUT -p TCP -i $EXTIF --dport  53 --sport 1024:65534 -j ACCEPT # DNS
# iptables -A INPUT -p TCP -i $EXTIF --dport  80 --sport 1024:65534 -j ACCEPT # WWW
# iptables -A INPUT -p TCP -i $EXTIF --dport 110 --sport 1024:65534 -j ACCEPT # POP3
# iptables -A INPUT -p TCP -i $EXTIF --dport 443 --sport 1024:65534 -j ACCEPT # HTTPS


# 第二部份，针对后端主机的防火墙设定！###############################
# 1. 先加载一些有用的模块
  modules="ip_tables iptable_nat ip_nat_ftp ip_nat_irc ip_conntrack 
ip_conntrack_ftp ip_conntrack_irc"
  for mod in $modules
  do
      testmod=`lsmod | grep "^${mod} " | awk '{print $1}'`
      if [ "$testmod" == "" ]; then
            modprobe $mod
      fi
  done

# 2. 清除 NAT table 的规则
  iptables -F -t nat
  iptables -X -t nat
  iptables -Z -t nat
  iptables -t nat -P PREROUTING  ACCEPT
  iptables -t nat -P POSTROUTING ACCEPT
  iptables -t nat -P OUTPUT      ACCEPT

# 3. 若有内部接口的存在 (双网卡) 开放成为路由器，且为 IP 分享器！
  if [ "$INIF" != "" ]; then
    iptables -A INPUT -i $INIF -j ACCEPT
    echo "1" > /proc/sys/net/ipv4/ip_forward
    if [ "$INNET" != "" ]; then
        for innet in $INNET
        do
            iptables -t nat -A POSTROUTING -s $innet -o $EXTIF -j MASQUERADE
        done
    fi
  fi
  # 如果你的 MSN 一直无法联机，或者是某些网站 OK 某些网站不 OK，
  # 可能是 MTU 的问题，那你可以将底下这一行给他取消批注来启动 MTU 限制范围
  # iptables -A FORWARD -p tcp -m tcp --tcp-flags SYN,RST SYN -m tcpmss \
  #          --mss 1400:1536 -j TCPMSS --clamp-mss-to-pmtu

# 4. NAT 服务器后端的 LAN 内对外之服务器设定
# iptables -t nat -A PREROUTING -p tcp -i $EXTIF --dport 80 \
#          -j DNAT --to-destination 192.168.1.210:80 # WWW

# 5. 特殊的功能，包括 Windows 远程桌面所产生的规则，假设桌面主机为 1.2.3.4
# iptables -t nat -A PREROUTING -p tcp -s 1.2.3.4  --dport 6000 \
#          -j DNAT --to-destination 192.168.100.10
# iptables -t nat -A PREROUTING -p tcp -s 1.2.3.4  --sport 3389 \
#          -j DNAT --to-destination 192.168.100.20

# 6. 最终将这些功能储存下来
  /etc/init.d/iptables save
```
再来看一下关于 iptables.allow 的内容，假如我要让一个 140.116.44.0/24 这个网域的所有主机来源可以进入我的主机的话，那么这个档案的内容可以写成这样：
```
[root@www iptables]# vim iptables.allow
#!/bin/bash
# 底下则填写你允许进入本机的其他网域或主机
  iptables -A INPUT -i $EXTIF -s 140.116.44.0/24 -j ACCEPT

# 底下则是关于抵挡的档案设定法！
[root@www iptables]# vim iptables.deny
#!/bin/bash

  iptables -A INPUT -i $EXTIF -s 140.116.44.254 -j DROP

[root@www iptables]# chmod 700 iptables.*
```
将这三个档案的权限设定为 700 且只属于 root 的权限后，就能够直接执行 iptables.rule 。  
如果你希望一开机就自动执行这个 script 的话，请将这个档案的完整档名写入 /etc/rc.d/rc.local 当中：
```
[root@www ~]# vim /etc/rc.d/rc.local
....(其他省略)....
# 1. Firewall
/usr/local/virus/iptables/iptables.rule
```
事实上，这个脚本的最底下已经加入写入防火墙默认规则文件的功能，所以你只要执行一次，就拥有最正确的规则了。同时，这个防火墙还可以具有最简单的 IP 分享器的功能。

# NAT 服务器的设定
NAT 的全名是 Network Address Translation，字面上的意思是『网络地址的转换』。iptables 指令能够修改 IP 封包的表头数据。
## 什么是 NAT, SNAT, DNAT
内部 LAN 有任何一部主机想要传送封包出去时， 那么这个封包要透过 Linux 主机而传送出去：
* 先经过 NAT table 的 PREROUTING 链；
* 经由路由判断确定这个封包是要进入本机与否，若不进入本机，则下一步；
* 再经过 Filter table 的 FORWARD 链；
* 通过 NAT table 的 POSTROUTING 链，最后传送出去。

NAT 服务器的重点就在于上面流程的第 1,4 步骤，也就是 NAT table 的两条重要的链：PREROUTING 与 POSTROUTING。 重点在于修改 IP 。但是这两条链修改的 IP 是不一样的。POSTROUTING 在修改来源 IP ，PREROUTING 则在修改目标 IP 。 由于修改的 IP 不一样，所以就称为来源 NAT (Source NAT, SNAT) 及目标 NAT (Destination NAT, DNAT)。
### 来源 NAT, SNAT：修改封包表头的『来源』项目
IP 分享的功能就是透过 NAT 表格的 POSTROUTING 来处理的。
1. 客户端所发出的封包表头中，来源会是 192.168.1.100 ，然后传送到 NAT 这部主机；
2. NAT 这部主机的内部接口 (192.168.1.2) 接收到这个封包后，会主动分析表头数据， 因为表头数据显示目的并非 Linux 本机，所以开始经过路由， 将此封包转到可以连接到 Internet 的 Public IP 处；
3. 由于 private IP 与 public IP 不能互通，所以 Linux 主机透过 iptables 的 NAT table 内的 Postrouting 链将封包表头的来源伪装成为 Linux 的 Public IP ，并且将两个不同来源 (192.168.1.100 及 public IP) 的封包对应写入暂存内存当中， 然后将此封包传送出去了；

此时 Internet 上面看到这个封包时，都只会知道这个封包来自那个 Public IP 而不知道其实是来自内部。如果 Internet 回传封包：
4. 在 Internet 上面的主机接到这个封包时，会将响应数据传送给那个 Public IP 的主机；
5. 当 Linux NAT 服务器收到来自 Internet 的回应封包后，会分析该封包的序号，并比对刚刚记录到内存当中的数据， 由于发现该封包为后端主机之前传送出去的，因此在 NAT Prerouting 链中，会将目标 IP 修改成为后端主机，亦即那部 192.168.1.100，然后发现目标已经不是本机 (public IP)， 所以开始透过路由分析封包流向；
6. 封包会传送到 192.168.1.2 这个内部接口，然后再传送到最终目标 192.168.1.100 机器上去！

所有内部 LAN 的主机都可以透过这部 NAT 服务器联机出去， 而大家在 Internet 上面看到的都是同一个 IP (就是 NAT 那部主机的 public IP)， 所以，如果内部 LAN 主机没有连上不明网站的话，那么内部主机其实是具有一定程度的安全性的，因为 Internet 上的其他主机没有办法主动攻击你的 LAN 内的 PC。所以我们才会说， NAT 最简单的功能就是类似 IP 分享器。那也是 SNAT 的一种。
> 基本上，NAT 服务器一定是路由器，不过， NAT 服务器由于会修改 IP 表头数据， 因此与单纯转递封包的路由器不同。最常见的 IP 分享器就是一个路由器，但是这个 IP 分享器一定会有一个 Public IP 与一个 Private IP，让 LAN 内的 Private IP 可以透过 IP 分享器的 Public IP 传送出去。
### 目标 NAT, DNAT：修改封包表头的『目标』项目
SNAT 主要是应付内部 LAN 连接到 Internet 的使用方式，至于 DNAT 则主要用在内部主机想要架设可以让 Internet 存取的服务器。  
假设我的内部主机 192.168.1.210 启动了 WWW 服务，这个服务的 port 开启在 port 80 ， 那么 Internet 上面的主机 (61.xx.xx.xx) 要连接到我的内部服务器，还是得要透过 Linux NAT 服务器。所以这部 Internet 上面的机器必须要连接到我们的 NAT 的 public IP 才行。
1. 外部主机想要连接到目的端的 WWW 服务，则必须要连接到我们的 NAT 服务器上头；
2. 我们的 NAT 服务器已经设定好要分析出 port 80 的封包，所以当 NAT 服务器接到这个封包后， 会将目标 IP 由 public IP 改成 192.168.1.210 ，且将该封包相关信息记录下来，等待内部服务器的响应；
3. 上述的封包在经过路由后，来到 private 接口处，然后透过内部的 LAN 传送到 192.168.1.210 上头！
4. 192.186.1.210 会响应数据给 61.xx.xx.xx ，这个回应当然会传送到 192.168.1.2 上头去；
5. 经过路由判断后，来到 NAT Postrouting 的链，然后透过刚刚第二步骤的记录，将来源 IP 由 192.168.1.210 改为 public IP 后，就可以传送出去了！
## 最简单的 NAT 服务器：IP 分享功能
IP 分享器的功能其实就是 SNAT 。作用就只是在 iptables 内的 NAT 表格当中，那个路由后的 POSTROUTING 链进行 IP 的伪装。NAT 服务器必须要有一个 public IP 接口，以及一个内部 LAN 连接的 private IP 界面才行。假设：
* 外部接口使用 eth0 ，这个接口具有 public IP；
* 内部接口使用 eth1 ，假设这个 IP 为 192.168.100.254 ；

关于 NAT 服务器部分的脚本：
```
iptables -A INPUT -i $INIF -j ACCEPT
# 这一行为非必要的，主要的目的是让内网 LAN 能够完全的使用 NAT 服务器资源。
# 其中 $INIF 在本例中为 eth1 接口

echo "1" > /proc/sys/net/ipv4/ip_forward
# 上头这一行则是在让你的 Linux 具有 router 的能力

iptables -t nat -A POSTROUTING -s $innet -o $EXTIF -j MASQUERADE
# 这一行最关键！就是加入 nat table 封包伪装！本例中 $innet 是 192.168.100.0/24
# 而 $EXTIF 则是对外界面，本例中为 eth0 
```
『 MASQUERADE 』这个设定值就是『 IP 伪装成为封包出去 (-o) 的那块装置上的 IP 』。所以封包来源只要来自 $innet (也就是内部 LAN 的其他主机) ，只要该封包可透过 eth0 传送出去， 那就会自动的修改 IP 的来源表头成为 eth0 的 public IP 。LAN 内的其他 PC 将 NAT 服务器作为 PC 的 GATEWAY 即可。  
除了 IP 伪装 (MASQUERADE) 之外，我们还可以直接指定修改 IP 封包表头的来源 IP ：
```
iptables -t nat -A POSTROUTING -o eth0 -j SNAT \
         --to-source 192.168.1.100
# 对外的 IP 固定为 192.168.1.100 

iptables -t nat -A POSTROUTING -o eth0 -j SNAT \
         --to-source 192.168.1.210-192.168.1.220
# 轮流使用不同的对外 IP ，192.168.1.210-192.168.1.220
```
## iptables 的额外核心模块功能
iptables 提供很多好用的模块， 这些模块可以辅助封包的过滤用途，让我们可以节省很多 iptables 的规则拟定。
## 在防火墙后端之网络服务器 DNAT 设定
不同的服务器封包传输的方式可能有点差异，可能容易导致某些服务无法顺利对 Internet 提供的问题。DNAT 用到的是 nat table 的 Prerouting 链。
```
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 \
     -j DNAT --to-destination 192.168.100.10:80 

-j REDIRECT --to-ports <port number>
# 这个也挺常见的，基本上，就是进行本机上面 port 的转换就是了！
# 不过，特别留意的是，这个动作仅能够在 nat table 的 PREROUTING 以及 OUTPUT 链上面实行而已

范例：将要求与 80 联机的封包转递到 8080 这个 port
[root@www ~]# iptables -t nat -A PREROUTING -p tcp  --dport 80 \
> -j REDIRECT --to-ports 8080
# 在使用了非正规的 port 来进行某些 well known 的协议，例如使用 8080 这个 port 来启动 WWW ，但是别人都以 port 80 来联机，所以，你就可以使用上面的方式来将对方对你主机的联机传递到 8080 
```