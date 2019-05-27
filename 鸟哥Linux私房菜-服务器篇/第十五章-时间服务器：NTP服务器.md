# 关于时区与网络校正时的通讯协议
## 什么是时区？全球有多少时区？GMT 在哪个时区？
因为绕地球一圈是 360 度角，这 360 度角共分为 24 个时区，一个时区 15 度角。又由于是以格林威治时间为标准时间 (Greenwich Mean Time, GMT 时间)，加上地球自转的关系，因此，在格林威治以东的区域时间是比较快的(+小时)，而以西的地方当然就是较慢。  
底下约略列出各个时区的名称与所在经度，以及与 GMT 时间的时差：
标准时区|经度|时差
:---:|:---|:---:
GMT , Greenwich Mean Time|0|  W/E	标准时间
CET , Central European|15  E|+1  东一区
EET , Eastern European|30  E|+2  东二区
BT  , Baghdad|45 E|+3  东三区
USSR, Zone 3|60  E|+4  东四区
USSR, Zone 4|75  E|+5  东五区
Indian, First|82.3E|+5.5东五半区
USSR, Zone 5|90  E|+6  东六区
SST , South Sumatra|105 E|+7  东七区
JT  , Java|112 E|+7.5东七半区
CCT , China Coast|120 E|+8  东八区
JST , Japan|135 E|+9  东九区
SAST, South Australia|142 E|+9.5东九半区
GST , Guam|150 E|+10 东十区
NZT , New Zealand|180 E|+12 东十二区
Int'l Date Line|180 E/W|国际日期变更线
BST , Bering|165 W|-11 西十一区
SHST, Alaska/Hawaiian|150 W|-10 西十区
YST , Yukon|135 W|-9  西九区
PST , Pacific|120 W|-8  西八区
MST , Mountain|105 W|-7  西七区
CST , Central|90  W	|6  西六区
EST , Eastern|75  W|-5  西五区
AST , Atlantic|60  W|-4  西四区
Brazil, Zone 2|45  W|-3  西三区
AT  , Azores|30  W|-2  西二区
WAT , West Africa|15  W|-1  西一区
在安装 Linux 的时候， 总是会发现目前的时间慢或者快了 8 小时，就与时区有关。
## 什么是夏季节约时间 (daylight savings)
因为地球在运行的时候是呈现一个倾斜角在绕太阳运转的，所以才有春夏秋冬，在夏天的时候，白天的时间会比较长，所以为了节约用电， 因此在夏天的时候某些地区会将他们的时间定早一小时。
## Coordinated Universal Time (UTC) 与系统时间的误差
在计算时间的时候，最准确的计算应该是使用『原子震荡周期』所计算的物理时钟了 (Atomic Clock, 也被称为原子钟)，这也被定义为标准时间 (International Atomic Time)。而我们常常看见的 UTC 也就是 Coordinated Universal Time (协和标准时间)就是利用这种 Atomic Clock 为基准所定义出来的正确时间。例如 1999 年在美国启用的原子钟 NIST F-1， 他所产生的时间误差每两千年才差一秒钟。这个 UTC 标准时间虽然与 GMT 时间放在同一个时区为基准， 不过由于计时的方式不同，UTC 时间与 GMT 时间有差不多 16 分钟的误差。  
事实上，在我们的身边就有很多的原子钟，例如石英表，还有计算机主机上面的 BIOS 内部就含有一个原子钟在纪录与计算时间的进行。不过由于原子钟主要是利用计算芯片 (crystal) 的原子震荡周期去计时的，这是因为每种芯片都有自己的独特的震荡周期之故。 然而因为这种芯片的震荡周期在不同的芯片之间多多少少都会有点差异性，甚至同一批芯片也可能会或多或少有些许的差异 (就连温度也可能造成这样的误差)，因此也就造成了 BIOS 的时间会经常的给他快了几秒或者慢了几秒。  
## NTP 通讯协议
Linux 操作系统的计时方式主要是由 1970/01/01 开始计算总秒数。 date 指令，有个 +%s 的参数，可以取得总秒数，这个就是软件时钟。而计算机硬件主要是以 BIOS 内部的时间为主要的时间依据 (硬件时钟)，而偏偏这个时间可能因为 BIOS 内部芯片本身的问题，而导致 BIOS 时间与标准时间 (UTC) 有一点点的差异存在。所以为了避免主机时间因为长期运作下所导致的时间偏差，进行时间同步 (synchronize) 的工作就显的很重要了。
* 软件时钟：由 Linux 操作系统根据 1970/01/01 开始计算的总秒数；
* 硬件时钟：主机硬件系统上面的时钟，例如 BIOS 记录的时间；

Network Time Protocol 另外还有 Digital Time Synchronization Protocol (DTSS) 可以达到同步时间的功能：
1. 首先，主机当然需要启动这个 daemon ，之后，
2. Client 会向 NTP Server 发送出调校时间的 message ，
3. 然后 NTP Server 会送出目前的标准时间给 Client ，
4. Client 接收了来自 Server 的时间后，会据以调整自己的时间，就达成了网络校时

为了解决延迟的问题，有一些 program 已经开发了自动计算时间传送过程的误差，以更准确的校准自己的时间。在 daemon 的部分，也同时以 server/client 及 master/slave 的架构来提供用户进行网络校时的动作。所谓的 master/slave 就有点类似 DNS 的系统。  
ntp 这个 daemon 是以 port 123 为连结的埠口 (使用 UDP 封包)，所以我们要利用 Time server 来进行时间的同步更新时，就得要使用 NTP 软件提供的 ntpdate 来进行 port 123 的联机。
## NTP 服务器的阶层概念
由于 NTP 时间服务器采用类似阶层架构 (stratum) 来处理时间的同步化， 所以他使用的是类似一般 server/client 的主从架构。网络社会上面有提供一些主要与次要的时间服务器， 这些均属于第一阶及第二阶的时间服务器 (stratum-1, stratum-2) ，如下所示：
* 主要时间服务器： http://support.ntp.org/bin/view/Servers/StratumOneTimeServers
* 次要时间服务器： http://support.ntp.org/bin/view/Servers/StratumTwoTimeServers

# NTP 服务器的安装与设定
## 所需软件与软件结构
还需要时区相关的数据文件，所以你需要的软件有：
* ntp： 就是 NTP 服务器的主要软件，包括配置文件以及执行档等等。
* tzdata： 软件名称为『 Time Zone data 』的缩写，提供各时区对应的显示格式。

与时间及 NTP 服务器设定相关的配置文件与重要数据文件有底下几个：
* /etc/ntp.conf： 就是 NTP 服务器的主要配置文件，也是唯一的一个；
* /usr/share/zoneinfo/： 由 tzdata 所提供，为各时区的时间格式对应档。 例如台湾地区的时区格式对应档案在 /usr/share/zoneinfo/Asia/Taipei ，这个目录里面的档案与底下要谈的两个档案 (clock 与 localtime) 是有关系的
* /etc/sysconfig/clock： 设定时区与是否使用 UTC 时间钟的配置文件。 每次开机后 Linux 会自动的读取这个档案来设定自己系统所默认要显示的时间。举个例子来说， 在我们台湾地区的本地时间设定中，这个档案内应该会出现一行『ZONE="Asia/Taipei"』的字样， 这表示我们的时间配置文件案『要取用 /usr/share/zoneinfo/Asia/Taipei 那个档案』的意思。
* /etc/localtime： 这个档案就是『本地端的时间配置文件』。刚刚那个 clock 档案里面规定了使用的时间配置文件 (ZONE) 为 /usr/share/zoneinfo/Asia/Taipei ，所以说这就是本地端的时间了，此时 Linux 系统就会将 Taipei 那个档案复制一份成为 /etc/localtime ，所以未来我们的时间显示就会以 Taipei 那个时间配置文件案为准。

常用于时间服务器与修改时间的指令方面，主要有底下这几个：
* /bin/date： 用于 Linux 时间 (软件时钟) 的修改与显示的指令；
* /sbin/hwclock： 用于 BIOS 时钟 (硬件时钟) 的修改与显示的指令。 这是一个 root 才能执行的指令，因为 Linux 系统上面 BIOS 时间与 Linux 系统时间是分开的，所以使用 date 这个指令调整了时间之后，还需要使用 hwclock 才能将修改过后的时间写入 BIOS 当中！
* /usr/sbin/ntpd： 主要提供 NTP 服务的程序，配置文件为 /etc/ntp.conf
* /usr/sbin/ntpdate： 用于客户端的时间校正，如果你没有要启用 NTP 而仅想要使用 NTP Client 功能的话，那么只会用到这个指令而已
## 主要配置文件 ntp.conf 的处理
假设 NTP 服务器所需要设定的架构如下：
* 我的上层 NTP 服务器共有 tock.stdtime.gov.tw, tick.stdtime.gov.tw, time.stdtime.gov.tw 三部，其中以 tock.stdtime.gov.tw 最优先使用 (prefer)；
* 不对 Internet 提供服务，仅允许来自内部网域 192.168.100.0/24 的查询而已；
* 侦测一些 BIOS 时钟与 Linux 系统时间的差异并写入 /var/lib/ntp/drift 档案当中。
### 利用 restrict 来管理权限控制
在 ntp.conf 档案内可以利用『 restrict 』来控管权限，这个参数的设定方式为：
```
restrict [你的IP] mask [netmask_IP] [parameter]
```
其中 parameter 的参数主要有底下这些：
* ignore： 拒绝所有类型的 NTP 联机；
* nomodify： 客户端不能使用 ntpc 与 ntpq 这两支程序来修改服务器的时间参数， 但客户端仍可透过这部主机来进行网络校时的；
* noquery： 客户端不能够使用 ntpq, ntpc 等指令来查询时间服务器，等于不提供 NTP 的网络校时；
* notrap： 不提供 trap 这个远程事件登录 (remote event logging) 的功能。
* notrust： 拒绝没有认证的客户端。

那如果你没有在 parameter 的地方加上任何参数的话，这表示『该 IP 或网段不受任何限制』的意思。一般来说，我们可以先关闭 NTP 的权限，然后再一个一个的启用允许登入的网段。
### 利用 server 设定上层 NTP 服务器
上层 NTP 服务器的设定方式为：
```
server [IP or hostname] [prefer]
```
 perfer 表示『优先使用』的服务器
### 以 driftfile 记录时间差异
```
driftfile [可以被 ntpd 写入的目录与档案]
```
因为预设的 NTP Server 本身的时间计算是依据 BIOS 的芯片震荡周期频率来计算的，但是这个数值与上层 Time Server 不见得会一致。所以 NTP 这个 daemon (ntpd) 会自动的去计算我们自己主机的频率与上层 Time server 的频率，并且将两个频率的误差记录下来，记录下来的档案就是在 driftfile 后面接的完整档名当中了。关于档名你必须要知道：
* driftfile 后面接的档案需要使用完整路径文件名；
* 该档案不能是连结档；
* 该档案需要设定成 ntpd 这个 daemon 可以写入的权限。
* 该档案所记录的数值单位为：百万分之一秒 (ppm)。

driftfile 后面接的档案会被 ntpd 自动更新，所以他的权限一定要能够让 ntpd 写入才行。在 CentOS 6.x 预设的 NTP 服务器中，使用的 ntpd 的 owner 是 ntp ，这部份可以查阅 /etc/sysconfig/ntpd 就可以知道。
### keys [key_file]
除了以 restrict 来限制客户端的联机之外，我们也可以透过密钥系统来给客户端认证， 如此一来可以让主机端更放心了。不过在这个章节里面我们暂不讨论这个部分，有兴趣的朋友可以参考 ntp-keygen 这个指令的相关说明。
根据上面的说明，我们最终可以取得这样的配置文件案内容：
```
[root@www ~]# vim /etc/ntp.conf
# 1. 先处理权限方面的问题，包括放行上层服务器以及开放区网用户来源：
restrict default kod nomodify notrap nopeer noquery     <==拒绝 IPv4 的用户
restrict -6 default kod nomodify notrap nopeer noquery  <==拒绝 IPv6 的用户
restrict 220.130.158.71   <==放行 tock.stdtime.gov.tw 进入本 NTP 服务器
restrict 59.124.196.83    <==放行 tick.stdtime.gov.tw 进入本 NTP 服务器
restrict 59.124.196.84    <==放行 time.stdtime.gov.tw 进入本 NTP 服务器
restrict 127.0.0.1        <==底下两个是默认值，放行本机来源
restrict -6 ::1
restrict 192.168.100.0 mask 255.255.255.0 nomodify <==放行区网来源

# 2. 设定主机来源，请先将原本的 [0|1|2].centos.pool.ntp.org 的设定批注掉：
server 220.130.158.71 prefer  <==以这部主机为最优先
server 59.124.196.83
server 59.124.196.84

# 3.预设时间差异分析档案与暂不用到的 keys 等，不需要更动它：
driftfile /var/lib/ntp/drift
keys      /etc/ntp/keys
```
## NTP 的启动与观察
设定完 ntp.conf 之后就可以启动 ntp 服务器了。启动与观察的方式如下：
```
# 1. 启动 NTP 
[root@www ~]# /etc/init.d/ntpd start
[root@www ~]# chkconfig ntpd on
[root@www ~]# tail /var/log/messages  <==自行检查看看有无错误

# 2. 观察启动的埠口看看：
[root@www ~]# netstat -tlunp | grep ntp
Proto Recv-Q Send-Q Local Address       Foreign Address  PID/Program name
udp        0      0 192.168.100.254:123 0.0.0.0:*        3492/ntpd
udp        0      0 192.168.1.100:123   0.0.0.0:*        3492/ntpd
udp        0      0 127.0.0.1:123       0.0.0.0:*        3492/ntpd
udp        0      0 0.0.0.0:123         0.0.0.0:*        3492/ntpd
udp        0      0 ::1:123             :::*             3492/ntpd
udp        0      0 :::123              :::*             3492/ntpd
# 主要是 UDP 封包，且在 port 123 这个埠口
```
这样就表示我们的 NTP 服务器已经启动了，不过要与上层 NTP 服务器联机则还需要一些时间， 通常启动 NTP 后约在 15 分钟内才会和上层 NTP 服务器顺利连接上。使用底下几个指令确认 NTP 服务器有顺利的更新自己的时间 (等待数分钟后)：
```
[root@www ~]# ntpstat
synchronised to NTP server (220.130.158.71) at stratum 3
   time correct to within 538 ms
   polling server every 128 s
```
这个指令可以列出我们的 NTP 服务器有跟上层联机否。由上述的输出结果可以知道，时间有校正约 538 * 10^(-3) 秒，且每隔 64 秒会主动去更新时间。
```
[root@www ~]# ntpq -p
     remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
*tock.stdtime.go 59.124.196.87    2 u   19  128  377   12.092   -0.953   0.942
+59-124-196-83.H 59.124.196.86    2 u    8  128  377   14.154    7.616   1.533
+59-124-196-84.H 59.124.196.86    2 u    2  128  377   14.524    4.354   1.079
```
这个 ntpq -p 可以列出目前我们的 NTP 与相关的上层 NTP 的状态，上头的几个字段的意义为：
* remote：亦即是 NTP 主机的 IP 或主机名，注意最左边的符号
    * 如果有『 * 』代表目前正在作用当中的上层 NTP
    * 如果是『 + 』代表也有连上线，而且可作为下一个提供时间更新的候选者。
* refid：参考的上一层 NTP 主机的地址
* st：就是 stratum 阶层
* when：几秒钟前曾经做过时间同步化更新的动作；
* poll：下一次更新在几秒钟之后；
* reach：已经向上层 NTP 服务器要求更新的次数
* delay：网络传输过程当中延迟的时间，单位为 10^(-6) 秒
* offset：时间补偿的结果，单位与 10^(-3) 秒
* jitter：Linux 系统时间与 BIOS 硬件时间的差异时间， 单位为 10^(-6) 秒。

另外，你也可以检查一下你的 BIOS 时间与 Linux 系统时间的差异， 就是 /var/lib/ntp/drift 这个档案的内容，单位为 10^(-6) 秒。  
要让你的 NTP Server/Client 真的能运作，在上述的动作中得注意：
* 上述的 ntpstat 以及 ntpq -p 的输出结果中，你的 NTP 服务器真的要能够连结上层 NTP ，否则你的客户端将无法对你的 NTP 服务器进行同步更新的
* 你的 NTP 服务器时间不可与上层差异太多。
* 服务器防火墙在 UDP port 123 有没有开
* 等待的时间够不够长
## 安全性设定
在 /etc/ntp.conf 里面的 restrict 参数中就已经设定了 NTP 这个 daemon 的服务限制范围了。不过，在防火墙 iptables 的部分，还是需要开启联机监听的。所以，在你的 iptables 规则的 scripts 当中，需要加入这一段：
```
[root@www ~]# vim /usr/local/virus/iptables/iptables.allow
iptables -A INPUT -i $EXTIF -p udp -s 192.168.100.0/24 --dport 123 -j ACCEPT

[root@www ~]# /usr/local/virus/iptables/iptables.rule
```

# 客户端的时间更新方式
## Linux 手动校时工作：date, hwclock
Linux 操作系统当中其实有两个时间，分别是：
* 软件时钟： Linux 自己的系统时间，由 1970/01/01 开始记录的时间参数
* 硬件时钟： 计算机系统在 BIOS 记录的实际时间，这也是硬件所记录的

在软件时钟方面，我们可以透过 date 这个指令来进行手动修订，但如果要修改 BIOS 记录的时间，就得要使用 hwclock 这个指令来写入才行。相关的用法如下：
```
[root@clientlinux ~]# date MMDDhhmmYYYY
选项与参数：
MM：月份
DD：日期
hh：小时
mm：分钟
YYYY：公元年

# 1. 修改时间成为 1 小时后的时间该如何是好？
[root@clientlinux ~]# date
Thu Jul 28 15:33:38 CST 2011

[root@clientlinux ~]# date 072816332011
Thu Jul 28 16:33:00 CST 2011
```
```
[root@clientlinux ~]# hwclock [-rw]
选项与参数：
-r ：亦即 read ，读出目前 BIOS 内的时间参数；
-w ：亦即 write ，将目前的 Linux 系统时间写入 BIOS 当中

# 2. 查阅 BIOS 时间，并且写入更改过的时间
[root@clientlinux ~]# date; hwclock -r
Thu Jul 28 16:34:00 CST 2011
Thu 28 Jul 2011 03:34:57 PM CST  -0.317679 seconds
# 是否刚好差异约一个小时，这就是 BIOS 时间

[root@clientlinux ~]# hwclock -w; hwclock -r; date
Thu 28 Jul 2011 04:35:12 PM CST  -0.265656 seconds
Thu Jul 28 16:35:11 CST 2011
# 这样就写入了，所以软件时钟与硬件时钟就同步了
```
当我们进行完 Linux 时间的校时后，还需要以 hwclock 来更新 BIOS 的时间，因为每次重新启动的时候，系统会重新由 BIOS 将时间读出来，所以， BIOS 才是重要的时间依据。
## Linux 的网络校时
在 Linux 的环境当中可利用 NTP 的客户端程序，亦即是 ntpdate 这支程序就能够进行时间的同步化。不过，因为 NTP 服务器本来就会与上层时间服务器进行时间的同步化， 所以在预设的情况下，NTP 服务器不可以使用 ntpdate 。也就是说 ntpdate 与 ntpd 不能同时启用的。所以不要再 NTP server 上执行这个指令。
```
[root@clientlinux ~]# ntpdate [-dv] [NTP IP/hostname]
选项与参数：
-d ：进入除错模式 (debug) ，可以显示出更多的有效信息。
-v ：有较多讯息的显示。

[root@clientlinux ~]# ntpdate 192.168.100.254
28 Jul 17:19:33 ntpdate[3432]: step time server 192.168.100.254 offset -2428.396146 sec
# 最后面会显示微调的时间有多少 (offset)

[root@clientlinux ~]# date; hwclock -r
四  7月 28 17:20:27 CST 2011
公元2011年07月28日 (周四) 18时19分26秒  -0.752303 seconds
# 还得 hwclock -w 写入 BIOS 时间才行

[root@clientlinux ~]# vim /etc/crontab
# 加入这一行
10 5 * * * root (/usr/sbin/ntpdate tock.stdtime.gov.tw && /sbin/hwclock -w) &> /dev/null
```
使用 crontab 之后，每天 5:10 Linux 系统就会自动的进行网络校时。不过，这个方式仅适合不要启动 NTP 的情况。要透过 NTP 去主动的更新时间，修改 /etc/ntp.conf 即可达成：
```
[root@clientlinux ~]# ntpdate 192.168.100.254
# 由于 ntpd 的 server/client 之间的时间误差不允许超过 1000 秒，
# 因此你得先手动进行时间同步，然后再设定与启动时间服务器

[root@clientlinux ~]# vim /etc/ntp.conf
#server 0.centos.pool.ntp.org
#server 1.centos.pool.ntp.org
#server 2.centos.pool.ntp.org
restrict 192.168.100.254  <==放行服务器来源！
server 192.168.100.254    <==这就是服务器！
# 很简单，就是将原本的 server 项目批注，加入我们要的服务器即可

[root@clientlinux ~]# /etc/init.d/ntpd start
[root@clientlinux ~]# chkconfig ntpd on
```
然后取消掉 crontab 的更新程序，这样你的 client 计算机就会主动的到 NTP 服务器去更新。
## Windows 的网络校时