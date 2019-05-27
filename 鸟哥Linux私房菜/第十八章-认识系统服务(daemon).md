# deamon 与服务 (Service)
『常驻在记体体中的程序，且可以提供一些系统或网络功能，那就是服务』『 service 』。  
系统为了某些功能必须要提供一些服务 (不论是系统本身还是网络方面)，这个服务就称为 service 。而达成这个 service 的程序就称为 daemon。例如，达成循环型例行性工作排程服务 (service) 的程序为 crond 这个 daemon。  
一般来说，当我们以文本模式或图形模式 (非单人维护模式) 完整启动进入 Linux 主机后， 系统已经提供我们很多的服务了！包括打印服务、工作排程服务、邮件管理服务等等
## daemon 的主要分类
依据 daemon 的启动与管理方式来区分，基本上，可以将 daemon 分为可独立启动的 stand alone，与透过一支 super daemon 来统一管理的服务这两大类。
### stand_alone：此 daemon 可以自行单独启动运行
stand alone 就是『独立的启动』的意思。这种类型的 daemon 可以自行启动而不必透过其他机制的管理； daemon 启动并加载到内存后就一直占用内存与系统资源。最大的优点就是：因为是一直存在内存内持续的提供服务， 因此对于发生客户端的要求时，stand alone 的 daemon 响应速度较快。常见的 stand alone daemon 有 WWW 的 daemon (httpd)、FTP 的 daemon (vsftpd) 等等。
### super daemon：一支特殊的 daemon 来统一管理
这个特殊的 daemon 就被称为 super daemon 。早期的 super daemon 是 inetd 这一个，后来则被 xinetd 所取代了。这种机制比较有趣的地方在于， 当没有客户端的要求时，各项服务都是未启动的情况，等到有来自客户端的要求时， super daemon 才唤醒相对应的服务。当客户端的要求结束后，被唤醒的这个服务也会关闭并释放系统资源。  
这种机制的好处是： (1)由于 super daemon 负责唤醒各项服务，因此 super daemon 可以具有安全控管的机制，就是类似网络防火墙的功能； (2)由于服务在客户端的联机结束后就关闭，因此不会一直占用系统资源。缺点是，因为有客户端的联机才会唤醒该服务，因此服务的反应时间会比较慢。常见的 super daemon 所管理的服务例如 telnet 。Super daemon 常驻在内存中。
### 窗口类型的解说
* 个别窗口负责单一服务的 stand alone：
* 统一窗口负责各种业务的 super daemon：  
  基本上，针对 super daemon 的处理模式有两种：分别是：
    * multi-threaded (多重线程)：  
      全部的客户端请求，一次性接收，一个服务同时会负责几个程序
    * single-threaded (单个线程)：  
      一个一个请求处理

  为多重运行的方式， daemon 会一直被触发多支程序来提供不同 client 的服务，所以不论你是第几个登陆者， 都可以享用 daemon 的服务。至于右侧则是单一运行的方式，仅会有一支 daemon 被唤醒，第一个用户达成联机后， 后续想要联机的用户就得要等待，他们的联机不会成功的。
### daemon 工作形态的类型
以 daemon 提供服务的工作状态来区分，又可以将 daemon 分为两大类，分别是：
* signal-control  
  这种 daemon 是透过讯号来管理的，只要有任何客户端的需求进来，他就会立即启动去处理！例如打印机的服务 (cupsd)。 
* interval-control  
  这种 daemon 则主要是『每隔一段时间就主动的去运行某项工作』，所以，你要作的是在配置文件指定服务要进行的时间与工作， 该服务在指定的时间才会去完成工作。我们在第十六章提到的 atd 与 crond 就属于这种类型的 daemon (每分钟侦测一次配置文件)。
### daemon 的命名守则
这些服务的名称被创建之后，被挂上 Linux 使用时，通常在服务的名称之后会加上一个 d ，例如例行性命令的创建的	at, 与 cron 这两个服务， 他的程序文件名会被取为 atd 与 crond，这个 d 代表的就是 daemon 的意思。
## 服务与端口的对应
port 与 daemon 的对应，客户端连接协议不同，服务导向端口号亦不同。为了统一整个因特网的端口号对应服务的功能，让所有的主机都能够使用相同的机制来提供服务与要求服务， 就有了『通讯协议』，有些约定俗成的服务都放置在同一个埠号上面。例如， 网址列上面的 http 会让浏览器向 WWW 服务器的 80 埠号进行联机的要求，而 WWW 服务器也会将 httpd 这个软件激活在 port 80， 这样两者才能够达成联机。  
系统通过 /etc/services 配置服务于埠号的对应
```
[root@www ~]# cat /etc/services
....(前面省略)....
ftp             21/tcp
ftp             21/udp          fsp fspd
ssh             22/tcp                          # SSH Remote Login Protocol
ssh             22/udp                          # SSH Remote Login Protocol
....(中间省略)....
http            80/tcp          www www-http    # WorldWideWeb HTTP
http            80/udp          www www-http    # HyperText Transfer Protocol
....(底下省略)....
# 这个文件的内容是以底下的方式来编排的：
# <daemon name>   <port/封包协议>   <该服务的说明>
```
第一栏为 daemon 的名称、第二栏为该 daemon 所使用的端口号与网络数据封包协议， 封包协议主要为可靠联机的 TCP 封包以及较快速但为非面向连接的 UDP 封包。举个例子说，那个远程联机机制使用的是 ssh 这个服务，而这个服务的使用的埠号为 22 。
> 不建议修改 /etc/services

## daemon 的启动脚本于启动方式
提供某个服务的 daemon 虽然只是一支程序而已，但是这支 daemon 的启动还是需要运行档、配置文件、运行环境等等，例如， httpd 这个程序 (man httpd) ，有不少的选项与参数。此外，为了管理上面的方便， 所以通常 distribution 都会记录每一支 daemon 启动后所取得程序的 PID 在 /var/run/ 这个目录下。另外，在启动这些服务之前，可能需要自行处理一下 daemon 能够顺利运行的环境是否正确等等。  
通常 distribution 会给我们一个简单的 shell script 来进行启动的功能。 该 script 可以进行环境的侦测、配置文件的分析、PID 文件的放置，以及相关重要交换文件案的锁住 (lock) 动作， 你只要运行该 script ，上述的动作就自动的进行，最终就能够顺利且简单的启动这个 daemon。  
这些启动脚本 (shell script) 及重要的配置文件，通常在：
* /etc/init.d/* ：启动脚本放置处  
  系统上几乎所有的服务启动脚本都放置在这里！事实上这是公认的目录，也有配置连结档到 /etc/init.d/。
* /etc/sysconfig/* ：各服务的初始化环境配置文件  
  几乎所有的服务都会将初始化的一些选项配置写入到这个目录下
* /etc/xinetd.conf, /etc/xinetd.d/*：super daemon 配置文件  
  super daemon 的主要配置文件 (其实是默认值) 为 /etc/xinetd.conf ，不过我们上面就谈到了， super daemon 只是一个统一管理的机制，他所管理的其他 daemon 的配置则写在 /etc/xinetd.d/* 里头。
* /etc/* ：各服务各自的配置文件
* /var/lib/*：各服务产生的数据库
* /var/run* ：各服务的程序之 PID 记录处  
  可以使用讯号 (signal) 来管理程序， 既然 daemon 是程序，所以当然也可以利用 kill 或 killall 来管理。不过为了担心管理时影响到其他的程序， 因此 daemon 通常会将自己的 PID 记录一份到 /var/run/ 当中。例如登录文件的 PID 就记录在 /var/run/syslogd.pid 这个文件中。如此一来， /etc/init.d/syslog 就能够简单的管理自己的程序。
### Stand alone 的 /etc/init.d/* 启动
几乎系统上面所有服务的启动脚本都在 /etc/init.d/ 底下，这里面的脚本会去侦测环境、搜寻配置文件、 加载 distribution 提供的函数功能、判断环境是否可以运行此 daemon 等等，等到一切都侦测完毕且确定可以运行后， 再以 shell script 的 case....esac 语法来启动、关闭、 观察此 daemon。简单的以 /etc/init.d/syslog 这个登录档启动脚本来进行说明：
```
[root@www ~]# /etc/init.d/syslog
用法: /etc/init.d/syslog {start|stop|status|restart|condrestart}
# 什么参数都不加的时候，系统会告诉你可以用的参数有哪些，如上所示。

观察 syslog 这个 daemon 目前的状态
[root@www ~]# /etc/init.d/syslog status
syslogd (pid 4264) 正在运行...
klogd (pid 4267) 正在运行...
# 代表 syslog 管理两个 daemon ，这两个 daemon 正在运行中

重新让 syslog 读取一次配置文件
[root@www ~]# /etc/init.d/syslog restart
正在关闭核心记录器:          [  确定  ]
正在关闭系统记录器:          [  确定  ]
正在启动系统记录器:          [  确定  ]
正在启动核心记录器:          [  确定  ]
[root@www ~]# /etc/init.d/syslog status
syslogd (pid 4793) 正在运行...
klogd (pid 4796) 正在运行...
# 因为重新启动过，所以 PID 与第一次观察的值就不一样了
```
CentOS 还提供另外一支可以启动 stand alone 服务的脚本，就是 service 这个程序。 其实 service 仅是一支 script。可以分析你下达的 service 后面的参数，然后根据你的参数再到 /etc/init.d/ 去取得正确的服务来 start 或 stop 。
```
[root@www ~]# service [service name] (start|stop|restart|...)
[root@www ~]# service --status-all
选项与参数：
service name：亦即是需要启动的服务名称，需与 /etc/init.d/ 对应；
start|...   ：亦即是该服务要进行的工作。
--status-all：将系统所有的 stand alone 的服务状态通通列出来
```
> 事实上，在 Linux 系统中，要『开或关某个 port 』，就是需要『 启动或关闭某个服务』，因此，可以找出某个 port 对应的服务，进而启动或关闭。

### Super daemon 的启动方式
其实 Super daemon 本身也是一支 stand alone 的服务。所以 Super daemon 自己启动的方式与 stand alone 是相同的。但是他管理的其他 daemon 必须要在配置文件中配置为启动该 daemon 才行。配置文件就是 /etc/xinetd.d/* 的所有文件。查找 super daemon 所管理的服务是否启动：
```
[root@www ~]# grep -i 'disable' /etc/xinetd.d/*
....(前面省略)....
/etc/xinetd.d/rsync:          disable = yes
/etc/xinetd.d/tcpmux-server:  disable = yes
/etc/xinetd.d/time-dgram:     disable = yes
/etc/xinetd.d/time-stream:    disable = yes
```
如果『 disable = yes 』则代表取消此项服务的启动，如果是『 disable = no 』 才是有启动该服务。假如要启动 rsync 服务：
```
# 1. 先修改配置文件成为启动的模样：
[root@www ~]# vim /etc/xinetd.d/rsync
# 请将 disable 那一行改成如下的模样 (原本是 yes 改成 no 就对了)
service rsync
{
        disable = no

# 2. 重新启动 xinetd 这个服务
[root@www ~]# /etc/init.d/xinetd restart
正在停止 xinetd:             [  确定  ]
正在激活 xinetd:             [  确定  ]

# 3. 观察启动的端口
[root@www ~]# grep 'rsync' /etc/services  <==先看看端口是哪一号
rsync           873/tcp               # rsync
rsync           873/udp               # rsync
[root@www ~]# netstat -tnlp | grep 873
tcp    0 0 0.0.0.0:873      0.0.0.0:*     LISTEN      4925/xinetd
# 启动的服务并非 rsync 而是 xinetd ，因为他要控管 rsync 。
```
也就是说，先修改 /etc/xinetd.d/ 底下的配置文件，再重新启动 xinetd 。而 xinetd 是一个 stand alone 启动的服务。

# 解析 super daemon 的配置文件
xinetd 可以进行安全性或者是其他管理机制的控管，也能够控制联机的行为。这些控制的手段都可以让我们的某些服务更为安全， 资源管理更为合理。由于 super daemon 可以作这样的管理，因此一些对客户端开放较多权限的服务 (例如 telnet)， 或者本身不具有管理机制或防火墙机制的服务，就可以透过 xinetd 来管理。
## 默认值配置文件：xinetd.conf
```
[root@www ~]# vim /etc/xinetd.conf
defaults
{
# 服务启动成功或失败，以及相关登陆行为的记录文件
        log_type        = SYSLOG daemon info  <==登录文件的记录服务类型
        log_on_failure  = HOST   <==发生错误时需要记录的信息为主机 (HOST)
        log_on_success  = PID HOST DURATION EXIT <==成功启动或登陆时的记录信息
# 允许或限制联机的默认值
        cps         = 50 10 <==同一秒内的最大联机数为 50 个，若超过则暂停 10 秒
        instances   = 50    <==同一服务的最大同时联机数
        per_source  = 10    <==同一来源的客户端的最大联机数
# 网络 (network) 相关的默认值
        v6only          = no <==是否仅允许 IPv6 ？可以先暂时不启动 IPv6 支持！
# 环境参数的配置
        groups          = yes
        umask           = 002
}

includedir /etc/xinetd.d <==更多的配置值在 /etc/xinetd.d 那个目录内
```
启动某个 super daemon 管理的服务，但是该服务的配置值并没有指定上述的那些项目，那么该服务的配置值就以上述的默认值为主。至于上述的默认值会将 super daemon 管理的服务配置为：『一个服务最多可以有 50 个同时联机， 但每秒钟发起的「新」联机最多仅能有 50 条，若超过 50 条则该服务会暂停 10 秒钟。同一个来源的用户最多仅能达成 10 条联机。 而登陆的成功与失败所记录的信息并不相同。』  
所有的服务参数档都在	/etc/xinetd.d 里面，如最后一行所示。每个参数文件的内容，一般来说，如下
```
service  <service_name>
{
       <attribute>   <assign_op>   <value>   <value> ...
       .............
}
```
第一行一定都有个 service ，至于那个 \<service_name\> 里面的内容，则与 /etc/services 有关，因为他可以对照着 /etc/services 内的服务名称与埠号来决定所要激活的 port 是哪个，然后相关的参数就在两个大刮号中间。attribute 是一些 xinetd 的管理参数， assign_op 则是参数的配置方法。 assign_op 的主要配置形式为：
*  = ： 表示后面的配置参数就是这样
* += ： 表示后面的配置为『在原来的配置里头加入新的参数』
* -= ： 表示后面的配置为『在原来的参数舍弃这里输入的参数！』

<table>
<tbody><tr align="center">
	<th>attribute (功能)</th><th>说明与范例</th></tr>
<tr><td colspan="2"><b>一般配置项目：服务的识别、启动与程序</b></td></tr>
<tr>
	<td align="center">disable<br>(启动与否)</td>
	<td>
		* 配置值：[yes|no]，默认 disable = yes<br>
		disable 为取消的意思，此值可配置该服务是否要启动。默认所有的 super daemon 管理的服务都不启动的。
		若要启动就得要配置为『 disable = no 』</td></tr>
<tr>
	<td align="center">id<br>(服务识别)</td>
	<td>
		* 配置值：[服务的名称]<br>
		虽然服务在配置文件开头『 service 服务名称』已经指定了，不过有时后会有重复的配置值，此时可以用 id 来取代服务名称。你可以参考一下 /etc/xinetd.d/time-stream 来思考一下原理。</td></tr>
<tr>
	<td align="center">server<br>(程序文件名)</td>
	<td>
		* 配置值：[program 的完整档名]
		这个就是指出这个服务的启动程序！例如 /usr/bin/rsync 为启动 rsync 服务的命令，所以这个配置值就会成为：『 server = /usr/bin/rsync 』</td></tr>
<tr>
	<td align="center">server_args<br>(程序参数)</td>
	<td>
		* 配置值：[程序相关的参数]<br>
		这里应该输入的就是 server 那里需要输入的一些参数，例如 rsync 需要加入 --daemon ，
		所以这里就配置：『 server_args = --daemon 』。与上面 server 搭配，最终启动服务的方式『/usr/bin/rsync --daemon』</td></tr>
<tr>
	<td align="center">user<br>(服务所属UID)</td>
	<td>
		* 配置值：[使用者账号]<br>
		如果 xinetd 是以 root 的身份启动来管理的，那么这个项目可以配置为其他用户。此时这个 daemon 
		将会以此配置值指定的身份来启动该服务的程序。举例来说，你启动 rsync 时会以这个配置值作为该程序的 UID。</td></tr>
<tr>
	<td align="center">group</td>
	<td>跟 user 的意思相同，此项目填入组名即可。</td></tr>

<tr><td colspan="2"><b>一般配置项目：联机方式与联机封包协议</b></td></tr>
<tr>
	<td align="center">socket_type<br>(封包类型)</td>
	<td>
		* 配置值：[stream|dgram|raw]，与封包有关<br>
		stream 为联机机制较为可靠的 TCP 封包，若为 UDP 封包则使用 dgram 机制。raw 代表 server 需要与 IP 直接对谈。举例来说 rsync 使用 TCP ，故配置为『socket_type = stream 』</td></tr>
<tr>
	<td align="center">protocol<br>(封包类型)</td>
	<td>
		* 配置值：[tcp|udp]，通常使用 socket_type 取代此配置<br>
		使用的网络协议，需参考 /etc/protocols 内的通讯协议，一般使用 tcp 或 udp。由于与 socket_type 重复，因此这个项目可以不指定。</td></tr>
<tr>
	<td align="center">wait<br>(联机机制)</td>
	<td>
		* 配置值：[yes(single)|no(multi)]，默认 wait = no<br>
		这就是我们刚刚提到的 <b>Multi-threaded</b> 与 <b>single-threaded</b> 
		！一般来说，我们希望大家的要求都可以同时被激活，所以可以配置『 wait = no 』
		此外，一般 udp 配置为 yes 而 tcp 配置为 no。</td></tr>
<tr>
	<td align="center">instances<br>(最大联机数)</td>
	<td>
		* 配置值：[数字或 UNLIMITED]<br>
		这个服务可接受的最大联机数量。如果你只想要开放 30 个人联机 rsync 时，可在配置文件内加入：『
		instances = 30 』</td></tr>
<tr>
	<td align="center">per_source<br>(单一用户来源)</td>
	<td>
		* 配置值：[一个数字或 UNLIMITED]<br>
		如果想要控制每个来源 IP 仅能有一个最大的同时联机数，就指定这个项目，例如同一个 IP 最多只能连 10 条联机『per_source = 10 』</td></tr>
<tr>
	<td align="center">cps<br>(新联机限制)</td>
	<td>
		* 配置值：[两个数字]<br>
		为了避免短时间内大量的联机要求导致系统出现忙碌的状态而有这个 cps 的配置值。第一个数字为一秒内能够接受的最多新联机要求，第二个数字则为，若超过第一个数字那暂时关闭该服务的秒数。</td></tr>

<tr><td colspan="2"><b>一般配置项目：登录文件的记录</b></td></tr>
<tr>
	<td align="center">log_type<br>(登录档类型)</td>
	<td>
		* 配置值：[登录项目  等级]<br>
		当数据记录时，以什么登录项目记载？且需要记载的等级为何(默认为 info 等级)。在下一章说明。</td></tr>
<tr>
	<td align="center">log_on_success<br>log_on_failure<br>(登录状态)</td>
	<td>
		* 配置值：[PID,HOST,USERID,EXIT,DURATION]<br>
		在『成功登陆』或『失败登陆』之后，需要记录的项目：PID 为纪录该 server 启动时候的 process ID ，
		HOST 为远程主机的 IP、USERID 为登陆者的账号、EXIT 为离开的时候记录的项目、DURATION 为该用户使用此服务多久？</td></tr>

<tr><td colspan="2"><b>进阶配置项目：环境、网络端口口与联机机制等</b></td></tr>
<tr>
	<td align="center">env<br>(额外变量配置)</td>
	<td>
		* 配置值：[变量名称=变量内容]<br>
		这一个项目可以让你配置环境变量，环境变量的配置守则可以参考第十一章</td></tr>
<tr>
	<td align="center">port<br>(非正规埠号)</td>
	<td>
		* 配置值：[一组数字(小于 65534)]<br>
		这里可以配置不同的服务与对应的 port ，但是请记住你的 port 与服务名称必须与 /etc/services 
		内记载的相同才行！不过，若服务名称是你自定义的，那么这个 port 就可以随你指定</td></tr>
<tr>
	<td align="center">redirect<br>(服务转址)</td>
	<td>
		* 配置值：[IP port]<br>
		将 client 端对我们 server 的要求，转到另一部主机上去。例如当有人要使用你的 ftp 时，你可以将他转到另一部机器上面去！那个 IP_Address 就代表另一部远程主机的 IP </td></tr>
<tr>
	<td align="center">includedir<br>(呼叫外部配置)</td>
	<td>
		<li>配置值：[目录名称]</li></ul>
		表示将某个目录底下的所有文件都给他塞进来 xinetd.conf 这个配置里，如此一来我们可以一个一个配置不同的项目！而不需要将所有的服务都写在 xinetd.conf 当中！你可以在 /etc/xinetd.conf 发现这个配置</td></tr>

<tr><td colspan="2"><b>安全控管项目：</b></td></tr>
<tr>
	<td align="center">bind<br>(服务接口锁定)</td>
	<td>
		<li>配置值：[IP]</li></ul>
		这个是配置『允许使用此一服务的适配卡』的意思！举个例子来说，你的 Linux 主机上面有两个 IP ，而你只想要让 IP1 可以使用此一服务，但 IP2 不能使用此服务，这里就可以将 IP1 写入即可！那么 IP2 就不可以使用此一 server </td></tr>
<tr>
	<td align="center">interface</td>
	<td>
		<li>配置值：[IP]</li></ul>
		与 bind 相同</td></tr>
<tr>
	<td align="center">only_from<br>(防火墙机制)</td>
	<td>
		<li>配置值：[0.0.0.0, 192.168.1.0/24, hostname, domainname]</li></ul>
		用在安全机制上面，也就是管制『只有这里面规定的 IP 或者是主机名可以登陆！』如果是 0.0.0.0 表示所有的 PC 皆可登陆，如果是 192.168.1.0/24 则表示为 C class 的网域！亦即由 192.168.1.1 ~ 192.168.1.255 皆可登陆！另外，也可以选择 domain name ，例如 .dic.ksu.edu.tw 就可以允许昆山资传系网域的 IP 登陆你的主机使用该 server ！</td></tr>
<tr>
	<td align="center">no_access<br>(防火墙机制)</td>
	<td>
		<li>配置值：[0.0.0.0, 192.168.1.0/24, hostname, domainname]</li></ul>
		跟 only_from 相似，就是用来管理可否进入你的 Linux 主机激活你的 server 服务的管理项目！ no_access 表示『不可登陆』的 PC</td></tr>
<tr>
	<td align="center">access_times<br>(时间控管)</td>
	<td>
		<li>配置值：[00:00-12:00, HH:MM-HH:MM]</li></ul>
		这个项目在配置『该服务 server 启动的时间』，使用的是 24 小时的配置！例如你的 ftp 要在 8 点到 16 点开放的话，就是： 08:00-16:00。</td></tr>
<tr>
	<td align="center">umask</td>
	<td>
		<li>配置值：[000, 777, 022]</li></ul>
		可以配置用户创建目录或者是文件时候的属性！系统建议值是 022 。</td></tr>
</tbody></table>

## 一个简单的 rsync 范例配置
透过 super daemon 控管的服务可以多一层管理的手续来达成类似防火墙的机制，使用 rsync 这个可以进行远程镜射 (mirrow) 的服务来说明。 rsync 可以让两部主机上面的某个目录一模一样，在远程异地备援系统上面是挺好用的一个机制。
```
[root@www ~]# vim /etc/xinetd.d/rsync
service rsync  <==服务名称为 rsync
{
        disable = no                     <==默认是关闭的！刚刚被我们打开了
        socket_type     = stream         <==使用 TCP 的联机机制之故
        wait            = no             <==可以同时进行大量联机功能
        user            = root           <==启动服务为 root 这个身份
        server          = /usr/bin/rsync <==就是这支程序启动 rsync 的服务
        server_args     = --daemon       <==这是必要的选项
        log_on_failure  += USERID        <==登陆错误时，额外记录用户 ID
}
```
由于在 /etc/services 当中规定 rsync 使用的端口口号码为 873 ，这个端口小于 1024 ，所以理论上启动这个端口的身份一定要是 root 才行。目前有两个接口，一个是 192.168.1.100 ，一个则是 127.0.0.1， 假设我将 192.168.1.100 设计为对外网域， 127.0.0.1 为内部网域，且内外网域的分别权限配置为：
* 对内部 127.0.0.1 网域开放较多权限的部分：
    * 这里的配置值需绑在 127.0.0.1 这个接口上；
    * 对 127.0.0.0/8 开放登陆权限；
    * 不进行任何联机的限制，包括总联机数量与时间；
    * 但是 127.0.0.100 及 127.0.0.200 不允许登陆 rsync 服务。
* 对外部 192.168.1.100 网域较多限制的配置：
    * 对外配置绑住 192.168.1.100 这个接口；
    * 这个接口仅开放 140.116.0.0/16 这个 B 等级的网域及 .edu.tw 网域可以登陆；
    * 开放的时间为早上 1-9 点以及晚上 20-24 点两个时段；
    * 最多允许 10 条同时联机的限制。
> 127.0.0.1 是内部循环测试用的 IP ，用他来设计网络是没有意义的。 
```
[root@www ~]# vim /etc/xinetd.d/rsync
# 先针对对内的较为松散的限制来配置：
service rsync
{
        disable = no                        <==要启动才行
        bind            = 127.0.0.1         <==服务绑在这个接口上！
        only_from       = 127.0.0.0/8       <==只开放这个网域的来源登陆
        no_access       = 127.0.0.{100,200} <==限制这两个不可登陆
        instances       = UNLIMITED         <==取代 /etc/xinetd.conf 的配置值
        socket_type     = stream            <==底下的配置则保留
        wait            = no
        user            = root
        server          = /usr/bin/rsync
        server_args     = --daemon
        log_on_failure  += USERID
}

# 再针对外部的联机来进行限制
service rsync
{
        disable = no
        bind            = 192.168.1.100
        only_from       = 140.116.0.0/16
        only_from      += .edu.tw           <==因为累加，所以利用 += 配置
        access_times    = 01:00-9:00 20:00-23:59 <==时间有两时段，有空格隔开
        instances       = 10                <==只有 10 条联机
        socket_type     = stream
        wait            = no
        user            = root
        server          = /usr/bin/rsync
        server_args     = --daemon
        log_on_failure  += USERID
}

IP/数字 表示子网掩码，即掩码有多少位 1 ，/8 /16 /24
```
设计完毕，重新启动服务：
```
# 0. 先看看原本的 873 状态为何！
[root@www ~]# netstat -tnlp | grep 873
tcp    0 0 0.0.0.0:873      0.0.0.0:*     LISTEN      4925/xinetd
# 仅针对 0.0.0.0 这个全局网域监听

# 1. 重新启动 xinetd，不是 rsync
[root@www ~]# /etc/init.d/xinetd restart
[root@www ~]# netstat -tnlp | grep 873
tcp    0 0 192.168.1.100:873     0.0.0.0:*       LISTEN    7227/xinetd
tcp    0 0 127.0.0.1:873         0.0.0.0:*       LISTEN    7227/xinetd
# 两个接口，PID 是同一个
```

# 服务的防火墙管理 xinetd, TCP Wrappers
一般来说，系统的防火墙分析主要可以透过封包过滤或者是透过软件分析，Linux 默认提供一个软件分析的工具，/etc/hosts.deny, /etc/hosts.allow。如果安装了 tcp wrappers 套件时，还可以额外加上一些追踪功能。
## /etc/hosts.allow, /etc/hosts.deny 管理
任何以 xinetd 管理的服务，都可以透过 /etc/hosts.allow, /etc/hosts.deny 来配置防火墙。针对来源 IP 或网域进行允许或拒绝的配置， 以决定该联机是否能够成功达成连接的一种方式。/etc/xinetd.d/rsync 里的 no_access，only_from 也可以进行这方面的防火墙配置。不过，是使用 /etc/hosts.allow, /etc/hosts.deny 更容易集中管控，在配置与查询方面也较方便。  
其实 /etc/hosts.allow 与 /etc/hosts.deny 也是 /usr/sbin/tcpd 的配置文件，而这个 /usr/sbin/tcpd 则是用来分析进入系统的 TCP 网络封包的一个软件，TCP 是一种面向连接的网络联机封包，包括 www, email, ftp 等等都是使用 TCP 封包来达成联机的。这个套件本身的功能就是在分析 TCP 网络数据封包，而 TCP 封包的文件头主要记录了来源与目主机的 IP 与 port ，因此藉由分析 TCP 封包并搭配 /etc/hosts.{allow,deny} 的守则对比，就可以决定该联机是否能够进入我们的主机。所以，使用 TCP Wrappers 来管控的就是：
* 来源 IP 或/与 整个网域的 IP 网段；
* port (就是服务，前面有谈到启动某个端口是 daemon 的责任)

基本上只要一个服务受到 xinetd 管理，或者是该服务的程序支持 TCP Wrappers 函式的功能时，那么该服务的防火墙方面的配置就能够以 /etc/hosts.{allow,deny} 来处理。换个方式来说，只要不支持 TCP Wrappers 函式功能的软件程序就无法使用 /etc/hosts.{allow,deny} 的配置值。要得知一个服务的程序支不支持 TCP Wrappers
```
[root@www ~]# ldd $(which sshd httpd)
/usr/sbin/sshd:
        libwrap.so.0 => /usr/lib64/libwrap.so.0 (0x00002abcbfaed000)
        libpam.so.0 => /lib64/libpam.so.0 (0x00002abcbfcf6000)
....(中间省略)....
/usr/sbin/httpd:
        libm.so.6 => /lib64/libm.so.6 (0x00002ad395843000)
        libpcre.so.0 => /lib64/libpcre.so.0 (0x00002ad395ac6000)
....(底下省略)....
# 重点在于软件有没有支持 libwrap.so 那个函式库
```
ldd (library dependency discovery) 这个命令可以查询某个程序的动态函式库支持状态，因此透过这个 ldd 我们可以轻松的就查询到 sshd, httpd 有无支持 tcp wrappers 所提供的 libwrap.so 这个函式库文件。通过上面发现，shd 有支持但是 httpd 则没有支持。因此我们知道 sshd 可以使用 /etc/hosts.{allow,deny} 进行类似防火墙的抵挡机制，但是 httpd 则没有此项功能。
### 配置文件语法
```
<service(program_name)> : <IP, domain, hostname> : <action>
<服务   (亦即程序名称)> : <IP 或领域 或主机名> : < 动作 >
# 以上的 < > 是不存在于配置文件中的
```
重点有两个，一是找出想要管理的程序的文件名即档名，二是写下来想要放行或抵挡的 IP 或网域。
```
[root@www ~]# vim /etc/hosts.deny
rsync : 127.0.0.100 127.0.0.200 : deny

[root@www ~]# vim /etc/hosts.deny
rsync : 127.0.0.100       : deny
rsync : 127.0.0.200       : deny
```
/etc/hosts.allow 及 /etc/hosts.deny 有一个文件存在就够了，不过为了配置方便，存在两个文件，需要注意的是：
* 写在 hosts.allow 当中的 IP 与网段，为默认『可通行』的意思，亦即最后一个字段 allow 可以不用写；
* 而写在 hosts.deny 当中的 IP 与网段则默认为 deny ，第三栏的 deny 亦可省略；
* 这两个文件的判断依据是： (1) 以 /etc/hosts.allow 为优先，而 (2) 若分析到的 IP 或网段并没有记录在 /etc/hosts.allow ，则以 /etc/hosts.deny 来判断。

也就是说， /etc/hosts.allow 的配置优先于 /etc/hosts.deny。基本上，只要 hosts.allow 也就够了，因为我们可以将 allow 与 deny 都写在同一个文件内，只是这样一来似乎显得有点杂乱无章，因此，通常我们都是：
1. 允许进入的写在 /etc/hosts.allow 当中；
2. 不许进入的则写在 /etc/hosts.deny 当中。

此外，还可以使用一些特殊参数在第一及第二字段。内容有：
* ALL：代表全部的 program_name 或者是 IP 都接受的意思，例如 ALL: ALL: deny
* LOCAL：代表来自本机的意思，例如： ALL: LOCAL: allow
* UNKNOWN：代表不知道的 IP 或者是 domain 或者是服务时；
* KNOWN：代表为可解析的 IP, domain 等等信息时；

## TCP Wrappers 特殊功能
安装 TCP Wrappers 软件之后，将额外的动作添加到第三栏。查询是否安装 TCP Wrappers 可以使用『 rpm -q tcp_wrappers 』。更加细部的主要动作有：
* spawn (action)  
  可以利用后续接的 shell 来进行额外的工作，且具有变量功能，主要的变量内容为： %h (hostname), %a (address), %d (daemon)等等；
* twist (action)  
  立刻以后续的命令进行，且运行完后终止该次联机的要求 (DENY)

为了达成追踪来源目标的相关信息的目的，此时我们需要 safe_finger 这个命令的辅助才行。而且我们还希望客户端的这个恶意者能够被警告。 整个流程可以是这样的：
1. 利用 safe_finger 去追踪出对方主机的信息 (包括主机名、用户相关信息等)；
2. 将该追踪到的结果以 email 的方式寄给我们本机的 root ；
3. 在对方屏幕上面显示不可登陆且警告他已经被记录的信息

由于是抵挡的机制， spawn 与 twist 的动作大多是写在 /etc/hosts.deny 文件中的。我们将上述的动作写成如下的形式：
```
[root@www ~]# vim /etc/hosts.deny
rsync : ALL: spawn (echo "security notice from host $(/bin/hostname)" ;\
	echo; /usr/sbin/safe_finger @%h ) | \
	/bin/mail -s "%d-%h security" root & \
	: twist ( /bin/echo -e "\n\nWARNING connection not allowed.\n\n" )
```
1. rsync： 指的就是 rsync 这个服务的程序
2. ALL： 指的是来源，这个范围指的当然是全部的所有来源
3. spawn (echo "security notice from host $(/bin/hostname)" ; echo ; /usr/sbin/safe_finger @%h ) | /bin/mail -s "%d-%h security" root &： 由于要将一些侦测的数据送给 root 的邮件信箱，因此需要使用数据流汇整的括号( )，括号内的重点在于 safe_finger 的项目，他会侦测到客户端主机的相关信息，然后使用管线命令将这些数据送给 mail 处理， mail 会将该信息以标头为 security 的字样寄给 root ，由于 spawn 只是中间的过程，所以还能够有后续的动作
3. twist ( /bin/echo -e "\n\nWARNING connection not allowed.\n\n" )： 这个动作会将 Warning 的字样传送到客户端主机的屏幕上！ 然后将该联机中断。

# 系统开启的服务
## 观察系统启动的服务
最常使用 netstat 观察。基本上，以 ps 来观察整个系统上面的服务是比较妥当的，因为他可以将全部的 process 都找出来。不过，要观察启动网络监听的服务，使用 netstat 较方便。
```
找出目前系统开启的『网络服务』
[root@www ~]# netstat -tulp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address       Foreign Address State  PID/Program name
tcp        0      0 www.vbird.tsai:2208 *:*             LISTEN 4575/hpiod
tcp        0      0 *:737               *:*             LISTEN 4371/rpc.statd
tcp        0      0 *:sunrpc            *:*             LISTEN 4336/portmap
tcp        0      0 www.vbird.tsai:ipp  *:*             LISTEN 4606/cupsd
tcp        0      0 www.vbird.tsai:smtp *:*             LISTEN 4638/sendmail: acce
tcp        0      0 *:ssh               *:*             LISTEN 4595/sshd
udp        0      0 *:filenet-tms       *:*                    4755/avahi-daemon:
....(底下省略)....
# 看一下上头， Local Address 的地方会出现主机名与服务名称的，要记得的是，
# 可以加上 -n 来显示 port number ，而服务名称与 port 对应则在 /etc/services

找出所有的有监听网络的服务 (包含 socket 状态)：
[root@www ~]# netstat -lnp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address  Foreign Address  State   PID/Program name
tcp        0      0 127.0.0.1:2208 0.0.0.0:*        LISTEN  4575/hpiod
....(中间省略)....
Active UNIX domain sockets (only servers)
Proto RefCnt Flags   Type   State     I-Node PID/Program name Path
....(中间省略)....
unix  2      [ ACC ] STREAM LISTENING 10624  4701/xfs         /tmp/.font-unix/fs7100
unix  2      [ ACC ] STREAM LISTENING 12824  5015/Xorg        /tmp/.X11-unix/X0
unix  2      [ ACC ] STREAM LISTENING 12770  4932/gdm-binary  /tmp/.gdm_socket
....(以下省略)....
# 除了原有的网络监听 port 之外，还会有 socket 显示在上面

观察所有的服务状态
[root@www ~]# service --status-all
```
## 配置启动后立即启动服务的方法
Linux 主机启动：
1. 打开计算机电源，开始读取 BIOS 并进行主机的自我测试；
2. 透过 BIOS 取得第一个可启动装置，读取主要启动区 (MBR) 取得启动管理程序；
3. 透过启动管理程序的配置，取得 kernel 并加载内存且侦测系统硬件；
4. 核心主动呼叫 init 程序；
5. init 程序开始运行系统初始化 (/etc/rc.d/rc.sysinit)
6. 依据 init 的配置进行 daemon start (/etc/rc.d/rc[0-6].d/*)
7. 加载本机配置 (/etc/rc.d/rc.local)

第六个步骤就是以不同的运行等级呼叫不同的服务。  
启动 Linux 系统时，可以进入不同的模式，称为运行等级 (run level)。不同的运行等级有不同的功能与服务， 目前我们先知道正常的运行等级有两个，一个是具有 X 窗口接口的 run level 5 ，另一个则是纯文本界面的 run level 3。
### chkconfig：管理系统服务默认启动与否
```
[root@www ~]# chkconfig --list [服务名称]
[root@www ~]# chkconfig [--level [0123456]] [服务名称] [on|off]
选项与参数：
--list ：仅将目前的各项服务状态栏出来
--level：配置某个服务在该 level 下启动 (on) 或关闭 (off)

[root@www ~]# chkconfig --list |more
NetworkManager  0:off   1:off   2:off   3:off   4:off   5:off   6:off
acpid           0:off   1:off   2:off   3:on    4:on    5:on    6:off
....(中间省略)....
yum-updatesd    0:off   1:off   2:on    3:on    4:on    5:on    6:off

xinetd based services:  <==底下为 super daemon 所管理的服务
        chargen-dgram:  off
        chargen-stream: off
# 分为两个区块，一个具有 1, 2, 3 等数字，一个则被 xinetd 管理。

显示出目前在 run level 3 为启动的服务
[root@www ~]# chkconfig --list | grep '3:on'

让 atd 这个服务在 run level 为 3, 4, 5 时启动：
[root@www ~]# chkconfig --level 345 atd on
```
chkconfig 仅配置启动时默认会启动的服务，该服务目前的状态并不能得知。  
但是，chkconfig 可以通过重启 xinetd 服务，管理 super daemon 的启动或关闭。
```
[root@www ~]# /etc/init.d/rsync status
-bash: /etc/init.d/rsync: No such file or directory
# rsync 是 super daemon 管理的，所以当然不可以使用 stand alone 的启动方式来观察

[root@www ~]# netstat -tlup | grep rsync
tcp  0 0 192.168.201.110:rsync  *:*     LISTEN     4618/xinetd
tcp  0 0 www.vbird.tsai:rsync   *:*     LISTEN     4618/xinetd

[root@www ~]# chkconfig --list rsync
rsync           on   <==默认启动呢！将它处理成默认不启动

[root@www ~]# chkconfig rsync off; chkconfig --list rsync
rsync           off 

[root@www ~]# /etc/init.d/xinetd restart; netstat -tlup | grep rsync
# rsync 服务已关闭
```
### ntsysv：类图形接口管理模式
ntsyssv 是 Red Hat 系统特有的。
```
[root@www ~]# ntsysv [--level <levels>]
选项与参数：
--level ：后面可以接不同的 run level ，例如 ntsysv --level 35
```
### chkconfig：配置自己的系统服务
```
[root@www ~]# chkconfig [--add|--del] [服务名称]
选项与参数：
--add ：添加一个服务名称给 chkconfig 来管理，该服务名称必须在 /etc/init.d/ 内
--del ：删除一个给 chkconfig 管理的服务
```
/etc/init.d/ 里面启动服务的 script 文件范例
```
[root@www ~]# vim /etc/init.d/myvbird
#!/bin/bash
# chkconfig: 35 80 70
# description: 
echo "Nothing"
```
比较重要的是第二行，他的语法是： 『 chkconfig: [runlevels] [启动顺位] [停止顺位] 』其中， runlevels 为不同的 run level 状态，启动顺位 (start number) 与 结束顺位 (stop number) 则是在 /etc/rc.d/rc[35].d 内创建以 S80myvbird 及 K70myvbird 为档名的配置方式。
## CentOS 5.x 默认启动的服务简易说明
服务名称|功能简介
:---:|:---:
acpid|(系统)高级电源管理的接口，这是一个新的电源管理模块， 可以监听来自核心层的电源相关事件而予以回应。 CentOS 的配置文件在 /etc/acpi/events/power.conf 中，默认仅有当你按下 power 按钮时，系统会自动关机
anacron<br>(可关闭)|(系统)与循环型的工作排程 cron 有关，可在排程过期后还可以唤醒来继续运行， 配置文件在 /etc/anacrontab。详情请参考第十六章的说明。
apmd<br>(可关闭)|(系统)配置文件在 /etc/sysconfig/apmd ，也是电源管理模块！ 可侦测电池电量，当电池电力不足时，可以自动关机以保护计算机主机。
atd|(系统)单一的例行性工作排程，详细说明请参考第十六章。 抵挡机制的配置文件在 /etc/at.{allow,deny}
auditd|(系统)SELinux 所需服务中的一项，可以让系统需 SELinux 稽核的信息写入 /var/log/audit/audit.log 中。若此服务没有启动，则信息会传给 syslog 管理。
autofs<br>(可关闭)|(系统)可用来自动挂载来自网络上的其他服务器所提供的网络驱动器机 (一般是 NFS)。 不过我们是单机系统，所以目前还没必要这个服务。
avahi-daemon<br>(可关闭)|(系统)也是一个客户端的服务，可以透过 Zeroconf 自动的分析与管理网络。 Zeroconf 较常用在笔记型计算机与行动装置上，所以可以先关闭他
bluetooth<br>(可关闭)|(系统)用在蓝芽装置的搜寻上，如果 Linux 是当作服务器使用时， 这个服务可以暂时关闭
cpuspeed|(系统)可以用来管理 CPU 的频率功能。若系统闲置时，此项功能可以自动的降低 CPU 频率来节省电量与降低 CPU 温度
crond|(系统)系统配置文件为 /etc/crontab，详细数据可参考第十六章的说明。
cups<br>(可关闭)|(网络)用来管理打印机的服务，可以提供网络联机的功能，有点类似打印服务器的功能，可以在 Linux 本机上面以浏览器的 http://localhost:631 来管理打印机
firstboot<br>(可关闭)|(系统)系统第一次进入图形接口还需要进行一些额外的配置，就是使用这个服务，既然已经安装妥当，可以将这个服务关闭
gpm|(系统)在 tty1~tty6 的环境下可以使用鼠标功能来复制贴上，就是这个 gpm 提供的能力
haldaemon<br>(可关闭)|(系统)通常用在壁纸计算机的环境中，可侦测类似 usb 的装置！ 不过，如果是服务器环境，这个服务倒是可以关闭，如果是壁纸计算机，那最好可以启动
hidd<br>(可关闭)|(系统)也是蓝牙服务的功能，可以提供键盘、鼠标等蓝牙装置的侦测，须搭配 bluetooth。服务器环境不需要此项服务
hplip<br>(可关闭)|(系统)主要是针对 HP 的打印机功能所开发的脚本服务，如果你的环境中并没有 HP 相关设备，这个服务可关闭
ip6tables<br>(可关闭)|(网络)是针对本机的防火墙功能！这个防火墙主要是针对 IPv6 的版本， 如果你的网络环境并没有 IPv6 的设备，那么这个服务是可以关闭的。
iptables|(网络)本机防火墙功能，是核心支持的，所以功能与效能都非常棒！不能够取消，在服务器篇介绍网络相关信息。
irqbalance|(系统)如果你的系统是多核心的硬件，那么这个服务要启动， 因为它可以自动的分配系统中断 (IRQ) 之类的硬件资源。
isdn<br>(可关闭)|(网络)ISDN 是一种宽带设备 (调制解调器的一种) 
kudzu<br>(可关闭)|(系统)有添加新的硬件时，这个服务可以在启动时自动的侦测硬件， 并且会自动的呼叫相关的配置软件，方便在启动时就处理好硬件
lm_sensors<br>(可关闭)|(系统)这个服务可以帮你侦测主板的相关侦测芯片，举例来说， 某些主板会主动的侦测 CPU 温度、频率、电压等，这个 lm_sensors 能够将这些温度、频率等数据显示出来，我们会在第二十一章介绍。
lvm2-monitor|(系统)我们已经谈过 LVM ，所以我们当然要启动这个服务比较妥当。
mcstrans|(系统)与 SELinux 有关的服务，最好也启动啊！
mdmonitor<br>(可关闭)|(系统)可以侦测所有软件的状态，暂时不需要启动这个服务
messagebus<br>(可关闭)|(系统)可用来沟通各个软件之间的信息，有点类似剪贴簿的感觉。 不过在服务器环境则没有强烈需求就是了。
microcode_ctl<br>(可关闭)|(系统)Intel 的 CPU 会提供一个外挂的微命令集提供系统运行， 不过，如果你没有下载 Intel 相关的命令集文件，那么这个服务不需要启动的，也不会影响系统运行。
netfs<br>(可关闭)|(网络)可以进行网络驱动器机 (NFS, SMB/CIFS) 的挂载与卸除功能。 目前我们尚未使用网络，因此这个服务可以先关闭。
network|(网络)提供网络配置的功能，所以一定要启动
nfslock<br>(可关闭)|(网络)NFS 为一种 Unix like 的网络驱动器机，但在进行文件的分享时， 为了担心同一文件多重编辑的问题，所以会有这个锁住 (lock) 的服务！可以避免同一个文件被两个不同的人编辑时所造成的文件错误问题。
pcscd<br>(可关闭)|(系统)智能卡侦测的服务，可以关闭
portmap|(网络)用在远程过程调用的服务，很多服务都使用其辅助联机， 因此建议不要取消他，除非你确定你的系统没有使用到任何的 RPC 服务
readahead_early<br>readahead_later<br>(可关闭)|(系统)在系统启动的时候可以先将某些程序加载到内存中，以方便快速的加载， 可加快一些启动的速度。
restorecond|(系统)利用 /etc/selinux/restorecond.conf 的配置来判断当新建文件时，该文件的 SELinux 类型应该如何还原。需要注意的是，如果你的系统有很多非正规的 SELinux 文件类型配置时，这个 daemon 最好关闭，否则他会将你配置的 type 修改回默认值。
rpcgssd<br>rpcidmapd<br>(可关闭)|(网络)与 NFS 有关的客户端功能
sendmail|(网络)这就是电子邮件的软件，我们想要拥有可寄信的功能时， 这个服务可不能关闭。不过，默认这个服务仅能支持本机的功能，无法收受来自因特网的邮件
setroubleshoot|(系统)一定要启动，将 SELinux 相关信息记录在 /var/log/messages 里面
smartd|(系统)这个服务可以自动的侦测硬盘状态，如果硬盘发生问题的话， 还能够自动的回报给系统管理员，是个非常有帮助的服务！不可关闭他
sshd|(网络)这个是远程联机服务器的软件功能， 这个通讯协议比 telnet 好的地方在于 sshd 在传送数据时可以进行加密！这个服务不要关闭
syslog|(系统)这个服务可以记录系统所产生的各项信息， 包括 /var/log/messages 内的几个重要的登录档。
xfs<br>(可关闭)|(系统)这个是 X Font Server，主要提供图形接口的字型的一个服务， 如果你不启动 X 窗口的话，那么这个服务可以不启动。但是如果你有需要用到 X 时，一定要启动，否则图形接口无法启动
xinetd|(系统)就是 super daemon
yum-updatesd|(系统)可以透过 yum 的功能进行软件的在线升级机制， 若有升级的软件释出时，就能够以邮件或者是 syslog 来通知系统管理原来手动升级
### 其他服务的简易说明
服务名称|功能简介
:---:|:---:
dovecot|(网络)可以配置 POP3/IMAP 等收受信件的服务，如果你的 Linux 主机是 email server 才需要这个服务，否则不需要启动
httpd|(网络)这个服务可以让你的 Linux 服务器成为 www server
named|(网络)这是域名服务器 (Domain Name System) 的服务， 这个服务非常重要，但是配置非常困难
nfs|(网络)这就是 Network Filesystem，是 Unix-Like 之间互相作为网络驱动器机的一个功能。
ntpd|(网络)服务的全名是 Network Time Protocol ，这个服务可以用来进行网络校时， 让你系统的时间永远都是正确的
smb|(网络)这个服务可以让 Linux 仿真成为 Windows 上面的网络上的芳邻。 如果你的 Linux 主机想要做为 Windows 客户端的网络驱动器机服务器，就很重要
squid|(网络)作为代理服务器的一个服务，可作为一个局域网络的防火墙之用。
vsftpd|(网络)作为文件传输服务器 (FTP) 的服务。