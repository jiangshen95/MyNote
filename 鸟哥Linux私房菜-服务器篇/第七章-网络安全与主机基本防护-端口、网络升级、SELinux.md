# 网络封包联机进入主机的流程
当来自一个网络上的联机要求想进入我们的主机时， 这个网络封包在进入主机实际取得数据的整个流程。
## 封包进入主机的流程
1. 经过防火墙的分析：  
   Linux 系统有内建的防火墙机制，预设的 Linux 防火墙就有两个机制，这两个机制都是独立存在的，因此我们预设就有两层防火墙喔。第一层是封包过滤式的 netfilter 防火墙， 另一个则是透过软件控管的 TCP Wrappers 防火墙。
     * 封包过滤防火墙：IP Filtering 或 Net Filter  
       要进入 Linux 本机的封包都会先通过 Linux 核心的预设防火墙，就是称为 netfilter 的一层防火墙。简单的说，就是 iptables 这个软件所提供的防火墙功能。主要是分析 TCP/IP 的封包表头来进行过滤的机制，主要分析的是 OSI 的第二、三、四层，主要控制的就是 MAC, IP, ICMP, TCP 与 UDP 的埠口与状态 (SYN, ACK...) 等。
     * 第二层防火墙：TCP Wrappers  
       通过 netfilter 之后，网络封包会开始接受 Super daemons 及 TCP_Wrappers 的检验，就是 /etc/hosts.allow 与 /etc/hosts.deny 的配置文件功能。这个功能也是针对 TCP 的 Header 进行再次的分析，同样你可以设定一些机制来抵制某些 IP 或 Port ，好让来源端的封包被丢弃或通过检验；
2. 服务 (daemon) 的基本功能：  
   预设的防火墙是 Linux 的内建功能，但防火墙主要管理的是 MAC, IP, Port 等封包表头方面的信息，如果想要控管某些目录可以进入， 某些目录则无法使用的功能，那就得要透过权限以及服务器软件提供的相关功能了。举例来说，你可以在 httpd.conf 这个配置文件之内规范某些 IP 来源不能使用 httpd 这个服务来取得主机的数据， 那么即使该 IP 通过前面两层的过滤，他依旧无法取得主机的资源。但要注意的是， 如果 httpd 这支程序本来就有问题的话，那么 client 端将可直接利用 httpd 软件的漏洞来入侵主机，而不需要取得主机内 root 的密码，因此，要小心这些启动在因特网上面的软件。
3. SELinux 对网络服务的细部权限控制：  
   为了避免前面一个步骤的权限误用，或者是程序有问题所造成的资安状况，因此 Security Enhanced Linux (安全强化 Linux) 就来发挥它的功能。简单的说，SELinux 可以针对网络服务的权限来设定一些规则 (policy) ，让程序能够进行的功能有限， 因此即使使用者的档案权限设定错误，以及程序有问题时，该程序能够进行的动作还是被限制的，即使该程序使用的是 root 的权限也一样。
4. 使用主机的文件系统资源：  
   最终网络封包其实是要向主机要求文件系统的数据。假设你要使用 httpd 这支程序来取得系统的档案数据，但 httpd 默认是由一个系统账号名称为 httpd 来启动的，所以，网页数据的权限要让 httpd 这支程序可以读取才行。如果你前面三关的设定都 OK ，最终权限设定错误，使用者依旧无法浏览你的网页数据。

在这些步骤之外，我们的 Linux 以及相关的软件都可能还会支持登录文件记录的功能，为了记录历史历程， 以方便管理者在未来的错误查询与入侵检测，良好的分析登录档的习惯是一定要建立的，尤其是 /var/log/messages 与 /var/log/secure 这些个档案。
## 常见的攻击手法与相关保护
### 取得账户信息后猜测密码
* 减少信息的曝光机会
* 建立较严格的密码设定规则：包括 /etc/shadow, /etc/login.defs 等档案的设定， 建议你可以参考基础篇内的账号管理那一章来规范你的用户密码变更时间等等， 如果主机够稳定且不会持续加入某些账号时，也可以考虑使用 chattr 来限制账号 (/etc/passwd, /etc/shadow) 的更改；
* 完善的权限设定：由于这类的攻击方式会取得你的某个使用者账号的登入权限， 所以如果你的系统权限设定得宜的话，那么攻击者也仅能取得一般使用者的权限而已，对于主机的伤害比较有限。
### 利用系统的程序漏洞『主动』攻击
* 关闭不需要的网络服务
* 随时保持更新
* 关闭不需要的软件功能
### 利用社交工程作诈骗
不要随意透露账号/密码等信息
### 利用程序功能的『被动』攻击
* 随时更新主机上的所有软件：如果你的浏览器是没有问题的， 那对方传递恶意代码时，你的浏览器就不会执行
* 较小化软件的功能
* 不要连接到不明的主机
### 蠕虫或木马的 rootkit
rootkit 意思是说可以取得 root 权限的一群工具组 (kit)，就如同前面主动攻击程序漏洞的方法一样， rootkit 主要也是透过主机的程序漏洞。不过， rootkit 也会透过社交工程让用户下载、安装 rootkit 软件， 结果让 cracker 得以简单的绑架对方主机。  
rootkit 除了可以透过上述的方法来进行入侵之外，rootkit 还会伪装或者是进行自我复制， 举例来说，很多的 rootkit 本身就是蠕虫或者是木马间谍程序。蠕虫会让你的主机一直发送封包向外攻击， 结果会让你的网络带宽被占光；至于木马程序 (Trojan Horse) 则会对你的主机进行开启后门 (开一个 port 来让 cracker 主动的入侵)。  
很多时候，rootkit 会主动的去修改系统观察的指令， 包括 ls, top, netstat, ps, who, w, last, find 等等，让你看不到某些有问题的程序， 如此一来，你的 Linux 主机就很容易被当成是跳板。
* 不要随意安装不明来源的档案或者是不明网站的档案数据；
* 不要让系统有太多危险的指令：例如 SUID/SGID 的程序， 这些程序很可能会造成用户不当的使用，而使得木马程序有机可趁
* 可以定时以 rkhunter 之类的软件来追查：有个网站提供 rootkit 程序的检查，你可以前往下载与分析你的主机： http://www.rootkit.nl/projects/rootkit_hunter.html
### DDoS 攻击法 (Distributed Denial of Service)
『分布式阻断服务攻击』，从字面上的意义来看，它就是透过分散在各地的僵尸计算机进行攻击， 让你的系统所提供的服务被阻断而无法顺利的提供服务给其他用户的方式。方法有很多，最常见的就属 SYN Flood 攻击法，由于当主机接收了一个带有 SYN 的 TCP 封包之后，就会启用对方要求的 port 来等待联机，并且发送出回应封包 (带有 SYN/ACK 旗目标 TCP 封包)，并等待 Client 端的再次回应。  
在这个步骤当中，如果 cient 端在发送出 SYN 的封包后，却将来自 Server 端的确认封包丢弃，那么你的 Server 端就会一直空等，而且 Client 端可以透过软件功能，在短短的时间内持续发送出这样的 SYN 封包，那么你的 Server 就会持续不断的发送确认封包，并且开启大量的 port 在空等。等到全部主机的 port 都启用完毕，系统就会挂掉。  
更可怕的是，通常攻击主机的一方不会只有一部！他会透过 Internet 上面的僵尸网络 (已经成为跳板，但网站主却没有发现的主机) 发动全体攻击，让你的主机在短时间内就立刻挂点。最常被用来作为阻断式服务的网络服务就是 WWW 了，因为 WWW 通常得对整个 Internet 开放服务。
### 其他
例如 IP 欺骗。他可以欺骗你主机告知该封包来源是来自信任网域，而且透过封包传送的机制， 由攻击的一方持续的主动发送出确认封包与工作指令。如此一来，你的主机可能就会误判该封包确实有响应， 而且是来自内部的主机。
* 设定规则完善的防火墙：利用 Linux 内建的防火墙软件 iptables 建立较为完善的防火墙，可以防范部分的攻击行为；
* 核心功能：这部份比较复杂，你必须要对系统核心有很深入的了解， 才有办法设定好你的核心网络功能。
* 登录文件与系统监控：你可以透过分析登录文件来了解系统的状况， 另外也可以透过类似 MRTG 之类的监控软件 来实时了解到系统是否有异常，这些工作都是很好的努力方向！
### 小结语
* 建立完善的登入密码规则限制；
* 完善的主机权限设定；
* 设定自动升级与修补软件漏洞、及移除危险软件；
* 在每项系统服务的设定当中，强化安全设定的项目；
* 利用 iptables, TCP_Wrappers 强化网络防火墙；
* 利用主机监控软件如 MRTG 与 logwatch 来分析主机状况与登录文件；
## 主机能作的保护：软件更新、减少网络服务、启动 SELinux
### 软件更新的重要性
假设要开放 WWW 服务，那么提供 WWW 服务的 httpd 这只程序就得要执行，并且，你的防火墙得要打开 port 80 。如果 httpd 这支程序有安全方面的问题时，防火墙就没有效用。  
### 认识系统服务的重要性
如果能够减少服务器上面的监听埠口， 此时因为服务器端没有可供联机的埠口，客户端当然也就无法联机到服务器端，关闭埠口的方式是透过关闭网络服务。能够减少网络服务就减少，可以避免很多不必要的麻烦。 
### 权限与 SELinux 的辅助
可以通过 ALC ，针对单一账号或单一群组进行特定的权限设定。  
可以使用 SELinux 控制，避免用户乱用系统，乱设定权限。SELinux 可以在程序与档案之间再加入一道细部的权限控制，因此，即使程序与档案的权限符合了操作动作，但如果程序与档案的 SELinux 类型 (type) 不吻合时，那么该程序就无法读取该档案。此外，CentOS 也针对了某些常用的网络服务制订了许多的档案使用规则 (rule)，如果这些规则没有启用， 那么即使权限、SELinux 类型都对了，该网络服务的功能还是无法顺利的运作。

# 网络自动升级软件
## 如何进行软件升级
* RPM：  
  这是目前最常见于 Linux distribution 当中的软件管理方式，包括 CentOS / Fedora / SuSE / Red Hat / Mandriva 等等，都是使用这个方式来管理的；
* Tarball：  
  利用软件的官方网站所释出的原始码在您的系统上面编译与安装， 一般来说，由于软件是直接在自己的机器上面编译的，所以效能会比较好一些。 不过，升级的时候就比较麻烦，因为又得要下载新的原始码并且重新编译一次。 这种安装模式常见于某些特殊软件 (没有包含在 distribution 当中)，或者是 Gentoo 这个强调效能的 distribution；
* dpkg：  
  是 debian 这个 distribution 所使用的软件管理方式，与 RPM 很类似，都是透过预先编译的处理，可以让 end user 直接使用来升级与安装。  
  RPM 与 dpkg 软件档案都有一些软件的基本信息， 并同时记录了软件的相依属性 (使用 rpm -q 的查询)，所以当分析这些基本信息并使用一些机制将这些相依信息记录下来后， 再透过一些额外的网络功能，就能够自动的分析你的系统与修补软件之间的差异， 并可进一步帮你分析所需要升级与相依属性的软件，就可达成自动升级。  
  
不同的在线升级机制：
* yum：  
  CentOS 与 Fedora 所常用的自动升级机制，透过 FTP 或 WWW 来进行在线升级以及在线直接安装软件；
* apt：  
  最早由 debian 这个 distribution 所发展，现在 B2D 也是使用 apt ，同时由于 apt 的可移植性， 所以只要你的 RPM 可以使用 apt 来管理的话，就可以自行建立 apt 服务器来提供其他使用者进行在线安装与升级。 
* you:  
  所谓的 Yast Online Update (YOU) 是由 SuSE 所自行开发出来的在线安装升级方式， 经过注册取得一组账号密码后，就能够使用 you 的机制来进行在线升级。不过如果是免费的版本， 则仅有 60 天的试用期
* urpmi：  
  这个则是 Mandriva 所提供的在线升级机制
## CentOS 的 yum 软件更新、映像站使用的原理
CentOS 会在 yum 服务器上，下载官方网站释出的 RPM 表头列表数据，该数据除了记载每个 RPM 软件的相依性之外，也说明了 RPM 档案所放置的容器 (repository) 所在。因此透过分析这些数据，我们的 CentOS 就能够直接使用 yum 去下载与安装所需要的软件了。
1. 先由配置文件判断 yum server 所在 IP 地址；
2. 连接到 yum server 后，先下载新的 RPM 档案的表头数据；
3. 分析比较使用者所欲安装/升级的档案，并提供使用者确认；
4. 下载用户选择的档案到系统中的 /var/cache/yum ，并进行实际安装；

由于你所下载的清单当中已经含有所有官方网站所释出的 RPM 档案的表头相依属性的关系， 所以如果你想要安装的软件包含某些尚未安装的相依软件时，我们的 yum 会顺便帮你下载所需要的其他软件，预安装后， 再安装你所实际需要的软件。  
CentOS 在世界各地都有映射站，这些映射站会将官网的 yum 服务器的数据复制一份，同时在映射站上面也提供同样的 yum 功能，因此，你可以在任何一部 yum 服务器的映像站上面下载与安装软件。  
yum 会自动的去分析离你的主机最近的那部映射站，然后直接使用该部映像主机作为你的 yum 来源， 因此，『理论上』你不需要更动任何设定。
### yum 的使用：安装，软件群组，全系统更新
yum 可不止能够在线自动升级而已，他还可以作查询、软件群组的安装、整体版本的升级等等。
```
[root@www ~]# yum [option] [查询的工作项目] [相关参数]
选项与参数：
option：主要的参数，包括有：
   -y ：当 yum 询问使用者的意见时，主动回答 yes 而不需要由键盘输入；

[查询的工作项目]：由于不同的使用条件，而有一些选择的项目，包括：
   install ：指定安装的软件名称，所以后面需接『 软件名称 』
   update  ：进行整体升级的行为；当然也可以接某个软件，仅升级一个软件；
   remove  ：移除某个软件，后面需接软件名称；
   search  ：搜寻某个软件或者是重要关键字；
   list    ：列出目前 yum 所管理的所有的软件名称与版本，有点类似 rpm -qa；
   info    ：同上，不过有点类似 rpm -qai 的执行结果；
   clean   ：下载的档案被放到 /var/cache/yum ，可使用 clean 将他移除，
             可清除的项目：packages | headers | metadata | cache 等；

在[查询的工作项目]部分还可以具有整个群组软件的安装方式，如下所示：
   grouplist   ：列出所有可使用的『软件群组』，例如 Development Tools 之类的；
   groupinfo   ：后面接 group_name，则可了解该 group 内含的所有软件名；
   groupinstall：可以安装一整组的软件群组
                 更常与 --installroot=/some/path 共享来安装新系统
   groupremove ：移除某个软件群组；

# 范例一：搜寻 CentOS 官网提供的软件名称是否有与 RAID 有关的？
[root@www ~]# yum search raid
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile  <==这里就是在测试最快的映射站
 * base: ftp.isu.edu.tw                     <==共有四个容器内容
 * extras: ftp.isu.edu.tw                   <==每个容器都在 ftp.isu.edu.tw 上
 * updates: ftp.isu.edu.tw
base                           | 3.7 kB     00:00  <==下载软件的表头列表中
extras                         |  951 B     00:00
updates                        | 3.5 kB     00:00
=================== Matched: raid =================<==找到的结果如下
dmraid.i686 : dmraid (Device-mapper RAID tool and library)
....(中间省略)....
mdadm.x86_64 : The mdadm program controls Linux md devices (software RAID
....(底下省略)....

# 范例二：上述输出结果中， mdadm 的功能为何？
[root@www ~]# yum info mdadm
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: ftp.twaren.net
 * extras: ftp.twaren.net
 * updates: ftp.twaren.net
Installed Packages  <==这里说明这是已经安装的软件！
Name       : mdadm
Arch       : x86_64
Version    : 3.1.3
Release    : 1.el6
Size       : 667 k
Repo       : installed
From repo  : anaconda-CentOS-201106060106.x86_64
Summary    : The mdadm program controls Linux md devices (software RAID
URL        : http://www.kernel.org/pub/linux/utils/raid/mdadm/
License    : GPLv2+
Description: The mdadm program is used to create, manage, and monitor
....(底下省略)....
# 由上述的 Summary 关键词，知道这软件在达成软件磁盘阵列功能。
```
yum 可以直接查询是否有某些特殊的软件名称：
* yum search "一些关键词"
* yum list (可列出所有的软件文件名)

然后再以正规表示法取得关键词，或者是『 yum info "软件名称" 』就能够知道该软件的用途，最后再决定要不要安装。
### 利用 yum 进行安装
```
[root@www ~]# yum install mdadm
....(前面省略)....
Setting up Install Process
Package mdadm-3.1.3-1.el6.x86_64 already installed and latest version
Nothing to do

[root@www ~]# yum install mdadma
Setting up Install Process
No package mdadma available.
Nothing to do

[root@www ~]# yum list javacc*
Available Packages
javacc.x86_64            4.1-0.5.el6      base
javacc-demo.x86_64       4.1-0.5.el6      base
javacc-manual.x86_64     4.1-0.5.el6      base
# 共有三套软件，分别是 javacc, javacc-demo, javacc-manual ，版本为 4.1-0.5.el6，
# 软件是放置到名称为 base 的容器当中存放的。

[root@www ~]# yum install javacc
....(前面省略)....
Setting up Install Process
Resolving Dependencies
--> Running transaction check  <==开始检查有没有相依属性的软件问题
---> Package javacc.x86_64 0:4.1-0.5.el6 set to be updated
....(中间省略)....

=========================================================================
 Package                     Arch     Version           Repository  Size
=========================================================================
Installing:
 javacc                      x86_64   4.1-0.5.el6       base       895 k
Installing for dependencies:
 java-1.5.0-gcj              x86_64   1.5.0.0-29.1.el6  base       139 k
 java_cup                    x86_64   1:0.10k-5.el6     base       197 k
 sinjdoc                     x86_64   0.5-9.1.el6       base       705 k

Transaction Summary
=========================================================================
Install       4 Package(s)  <==安装软件汇整，共安装 4 个，升级 0 个软件
Upgrade       0 Package(s)

Total download size: 1.9 M
Installed size: 5.6 M
Is this ok [y/N]: y  <==让你确认要下载否！
Downloading Packages:
(1/4): java-1.5.0-gcj-1.5.0.0-29.1.el6.x86_64.rpm      | 139 kB     00:00
(2/4): java_cup-0.10k-5.el6.x86_64.rpm                 | 197 kB     00:00
(3/4): javacc-4.1-0.5.el6.x86_64.rpm                   | 895 kB     00:00
(4/4): sinjdoc-0.5-9.1.el6.x86_64.rpm                  | 705 kB     00:00
-------------------------------------------------------------------------
Total                                         3.1 MB/s | 1.9 MB     00:00
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
  Installing     : java-1.5.0-gcj-1.5.0.0-29.1.el6.x86_64             1/4
  Installing     : 1:java_cup-0.10k-5.el6.x86_64                      2/4
  Installing     : sinjdoc-0.5-9.1.el6.x86_64                         3/4
  Installing     : javacc-4.1-0.5.el6.x86_64                          4/4

Installed:            <==主要需要安装的
  javacc.x86_64 0:4.1-0.5.el6

Dependency Installed: <==为解决相依性额外装的
  java-1.5.0-gcj.x86_64 0:1.5.0.0-29.1.el6   java_cup.x86_64 1:0.10k-5.el6
  sinjdoc.x86_64 0:0.5-9.1.el6

Complete!
```
yum 下载的数据除了每个容器的表头清单档案之外，所有下载的 RPM 档案都会在安装完毕之后予以删除，如果想要下载的 RPM 档案继续保留在 /var/cache/yum 当中，就得要修改 /etc/yum.conf 配置文件了。
```
[root@www ~]# vim /etc/yum.conf 
[main]
cachedir=/var/cache/yum/$basearch/$releasever
keepcache=1    # 0 改为 1 就能让 RPM 档案保存下来。
debuglevel=2
logfile=/var/log/yum.log
exactarch=1
obsoletes=1
....(底下省略)....
```
除非有多部主机要更新，可以利用一台先 yum 升级且下载，然后将所有的 RPM 档案收集起来给内网的机器升级 (rpm -Fvh *.rpm) 之外，不建议改动。
### yum 安装软件群组
由于 RPM 软件将一个大项目分成好几个小计划来执行，每个小计划都可以独立安装， 这样的好处是可以让使用者与软件开发者安装不同的环境。
```
# 范例四：查询系统有的软件群组有多少个？
[root@www ~]# LANG=C yum grouplist
Installed Groups:             <==这个是已安装的软件群组
   Additional Development
   Arabic Support
   Armenian Support
   Base
....(中间省略)....
Available Groups:             <==这个是尚可安装的软件群组
   Afrikaans Support
   Albanian Support
   Amazigh Support
....(中间省略)....
   Desktop Platform
   Desktop Platform Development
....(后面省略)....

# 范例五：那么 Desktop Platform 内含多少个 RPM 软件
[root@www ~]# yum groupinfo "Desktop Platform"
Group: 桌面环境平台
 Description: 受支援的 CentOS Linux 桌面平台函式库。
 Mandatory Packages: <==主要的会被安装的软件有这些
   atk
....(中间省略)....
 Optional Packages:  <==额外可选择的软件是这些
   qt-mysql
....(底下省略)....
# 如果你确定要安装这个软件群组的话，那就这样做：

[root@www ~]# yum groupinstall "Desktop Platform"
```
### 全系统更新
不同版本 (ex> 5.x --> 6.x) 间的升级最好还是不要尝试，重新安装可能是最好的状况。
## 挑选特定的映射站：修改 yum 配置文件与清除 yum 快取
```
[root@www ~]# vim /etc/yum.repos.d/CentOS-Base.repo
[base]
name=CentOS-$releasever - Base
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os
#baseurl=http://mirror.centos.org/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6
```
* [base]:  
  代表容器的名字，中括号一定要存在，里面的名称则可以随意取。但是不能有两个相同的容器名称，否则 yum 会不晓得该到哪里去找容器相关软件列表档案。
* name:  
  只是说明一下这个容器的意义
* mirrorlist= :  
  列出这个容器可以使用的映射站台，如果不想使用，可以批注到这行。由于等一下我们是直接设定映像站， 因此这行待会儿确实是需要批注掉
* baseurl= :  
  这个最重要，因为后面接的就是容器的实际网址 。mirrorlist 是由 yum 程序自行去捉映像站台， baseurl 则是指定固定的一个容器网址！我们刚刚找到的网址放到这里来
* enable=1 :  
  就是让这个容器被启动。如果不想启动可以使用 enable=0 
* gpgcheck=1 :  
  指定是否需要查阅 RPM 档案内的数字签名
* gpgkey=：  
  就是数字签名的公钥文件所在位置！使用默认值即可

按如下方式修改：
```
[root@www ~]# vim /etc/yum.repos.d/CentOS-Base.repo
[base]
name=CentOS-$releasever - Base
baseurl=http://ftp.twaren.net/Linux/CentOS/6/os/x86_64/ 
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6

[root@www ~]# yum clean all  <==改过配置文件，最好清除既有清单
```
测试：
```
# 范例：列出目前 yum server 所使用的容器有哪些？
[root@www ~]# yum repolist all
repo id        repo name                status
base           CentOS-6 - Base          enabled: 6,019
c6-media       CentOS-6 - Media         disabled
centosplus     CentOS-6 - Plus          disabled
contrib        CentOS-6 - Contrib       disabled
debug          CentOS-6 - Debuginfo     disabled
extras         CentOS-6 - Extras        enabled:     0
updates        CentOS-6 - Updates       enabled: 1,042
repolist: 7,061
# 在 status 上写 enabled 才是有启动的！由于 /etc/yum.repos.d/
# 有多个配置文件，所以你会发现还有其他的容器存在。
```
### 修改容器产生的问题与解决之道
由于我们是修改系统默认的配置文件，事实上，我们应该要在 /etc/yum.repos.d/ 底下新建一个档案， 该扩展名必须是 .repo 才行。但因为我们使用的是指定特定的映射站台，而不是其他软件开发生提供的容器， 因此才修改系统默认配置文件。但是可能由于使用的容器版本有新旧之分，yum 会先下载容器的清单到本机的 /var/cache/yum 里面去。如果我们修改了网址却没有修改容器名称 (中刮号内的文字)， 可能就会造成本机的列表与 yum 服务器的列表不同步，此时就会出现无法更新的问题。  
解决办法，清除掉本机上面的旧数据即可。通过 yum 的 clean 项目来处理即可。
```
[root@www ~]# yum clean [packages|headers|all] 
选项与参数：
 packages：将已下载的软件档案删除
 headers ：将下载的软件文件头删除
 all     ：将所有容器数据都删除！

# 范例：删除已下载过的所有容器的相关数据 (含软件本身与列表)
[root@www ~]# yum clean all
```

# 限制联机埠口 (port)
## 什么是 port
启动一个网络服务，这个服务会依据 TCP/IP 的相关通讯协议启动一个埠口在进行监听， 那就是 TCP/UDP 封包的 port (埠口) 了。网络联机是双向的，服务器端得要启动一个监听的埠口， 客户端得要随机启动一个埠口来接收响应的数据才行。
* 服务器端启动的监听埠口所对应的服务是固定的：  
  例如 WWW 服务开启在 port 80 ，FTP 服务开启在 port 21，email 传送开启在 port 25 等等，都是通讯协议上面的规范
* 客户端启动程序时，随机启动一个大于 1024以上的埠口：  
  客户端启动的 port 是随机产生的，主要是开启在大于 1024 以上的埠口。这个 port 也是由某些软件所产生的， 例如浏览器、Filezilla 这个 FTP 客户端程序等等。
* 一部服务器可以同时提供多种服务：  
  所谓的『监听』是某个服务程序会一直常驻在内存当中，所以该程序启动的 port 就会一直存在。 只要服务器软件激活的埠口不同，那就不会造成冲突。当客户端连接到此服务器时，透过不同的埠口，就可以取得不同的服务数据。所以，一部主机上面当然可以同时启动很多不同的服务。
* 共 65536 个 port：  
  port 占用 16 个位，因此一般主机会有 65536 个 port，而这些 port 又分成两个部分，以 port 1024 作区隔：
    * 只有 root 才能启动的保留的 port：  
      在小于 1024 的埠口，都是需要以 root 的身份才能启动的，这些 port 主要是用于一些常见的通讯服务，在 Linux 系统下，常见的协议与 port 的对应是记录在 /etc/services 里面的。
    * 大于 1024 用于 client 端的 port：  
      在大于 1024 以上的 port 主要是作为 client 端的软件激活的 port 。
* 是否需要三向交握：  
  建立可靠的联机服务需要使用到 TCP 协议，也就需要所谓的三向交握了，如果是非面向连接的服务，例如 DNS 与视讯系统， 那只要使用 UDP 协议即可。
* 通讯协议可以启用在非正规的 port：  
  可以透过 WWW 软件的设定功能将该软件使用的 port 启动在非正规的埠口， 只是如此一来，您的客户端要连接到你的主机时，就得要在浏览器的地方额外指定你所启用的非正规的埠口才行。 这个启动在非正规的端口功能，常常被用在一些所谓的地下网站。另外， 某些软件默认就启动在大于 1024 以上的端口，如 MySQL 数据库软件就启动在 3306。
* 所谓的 port 的安全性：  
  因为『Port 的启用是由服务软件所造成的』， 也就是说，真正影响网络安全的并不是 port ，而是启动 port 的那个软件 (程序)。
## 埠口的观察：netstat, nmap
由于 port 的启动与服务有关，那么『服务』跟『 port 』对应的档案是『 /etc/services 』。常用来观察 port 的有底下两个程序：
* netstat：在本机上面以自己的程序监测自己的 port；
* nmap：透过网络的侦测软件辅助，可侦测非本机上的其他网络主机，但有违法之虞。
### netstat
在做为服务器的 Linux 系统中，开启的网络服务越少越好！ 因为较少的服务可以较容易除错 (debug) 与了解安全漏洞，并可避免不必要的入侵管道。要了解自己的系统当中的服务项目，最简便的方法就是使用 netstat 了。
* 列出在监听的网络服务：  
  ```
  [root@www ~]# netstat -tunl
  ctive Internet connections (only servers)
  Proto Recv-Q Send-Q Local Address    Foreign Address    State
  tcp        0      0 0.0.0.0:111      0.0.0.0:*          LISTEN
  tcp        0      0 0.0.0.0:22       0.0.0.0:*          LISTEN
  tcp        0      0 127.0.0.1:25     0.0.0.0:*          LISTEN
  ....(底下省略)....
  ```
  上面说明了我的主机至少有启动 port 111, 22, 25 等，而且观察各联机接口，可发现 25 为 TCP 埠口，但只针对 lo 内部循环测试网络提供服务，因特网是连不到该埠口的。至于 port 22 则有提供因特网的联机功能。
* 列出已联机的网络联机状态：  
  ```
  [root@www ~]# netstat -tun
  Active Internet connections (w/o servers)
  Proto Recv-Q Send-Q Local Address       Foreign Address     State
  tcp        0     52 192.168.1.100:22    192.168.1.101:2162  ESTABLISHED
  ```
  从上面的数据来看，我的本地端服务器 (Local Address, 192.168.1.100) 目前仅有一条已建立的联机，那就是与 192.168.1.101 那部主机连接的联机，并且联机方线是由对方连接到我主机的 port 22 来取用我服务器的服务。
* 删除已建立或在监听当中的联机：  
  如果想要将已经建立，或者是正在监听当中的网络服务关闭的话，最简单的方法当然就是找出该联机的 PID， 然后将他 kill 掉即可：
  ```
  [root@www ~]# netstat -tunp
  Active Internet connections (w/o servers)
  Proto Recv-Q Send-Q Local Address    Foreign Address     State       PID/P name
  tcp        0     52 192.168.1.100:22 192.168.1.101:2162  ESTABLISHED 1342/0
  ```
  ···
  [root@www ~]# kill -9 1342
  ···
### nmap
如果你要侦测的设备并没有可让你登入的操作系统时，可以使用 nmap 。  
nmap 的软件说明之名称为：『Network exploration tool and security / port scanner』，顾名思义， 他是被系统管理员用来管理系统安全性查核的工具。他的具体描述当中也提到了， nmap 可以经由程序内部自行定义的几个 port 对应的指纹数据，来查出该 port 的服务为何，所以我们也可以藉此了解我们主机的 port 的作用。在 CentOS 里头是有提供 nmap 的，如果没有安装，可以使用 yum 安装。
```
[root@www ~]# nmap [扫瞄类型] [扫瞄参数] [hosts 地址与范围]
选项与参数：
[扫瞄类型]：主要的扫瞄类型有底下几种：
    -sT：扫瞄 TCP 封包已建立的联机 connect() ！
    -sS：扫瞄 TCP 封包带有 SYN 卷标的数据
    -sP：以 ping 的方式进行扫瞄
    -sU：以 UDP 的封包格式进行扫瞄
    -sO：以 IP 的协议 (protocol) 进行主机的扫瞄
[扫瞄参数]：主要的扫瞄参数有几种：
    -PT：使用 TCP 里头的 ping 的方式来进行扫瞄，可以获知目前有几部计算机存活(较常用)
    -PI：使用实际的 ping (带有 ICMP 封包的) 来进行扫瞄
    -p ：这个是 port range ，例如 1024-, 80-1023, 30000-60000 等等的使用方式
[Hosts 地址与范围]：这个有趣多了，有几种类似的类型
    192.168.1.100  ：直接写入 HOST IP 而已，仅检查一部；
    192.168.1.0/24 ：为 C Class 的型态，
    192.168.*.*　　：则变为 B Class 的型态了！扫瞄的范围变广了！
    192.168.1.0-50,60-100,103,200 ：这种是变形的主机范围

# 范例一：使用预设参数扫瞄本机所启用的 port (只会扫瞄 TCP)
[root@www ~]# yum install nmap
[root@www ~]# nmap localhost
PORT    STATE SERVICE
22/tcp  open  ssh
25/tcp  open  smtp
111/tcp open  rpcbind
# 在预设的情况下，nmap 仅会扫瞄 TCP 的协议
```
```
# 范例二：同时扫瞄本机的 TCP/UDP 埠口
[root@www ~]# nmap -sTU localhost
PORT    STATE SERVICE
22/tcp  open  ssh
25/tcp  open  smtp
111/tcp open  rpcbind
111/udp open  rpcbind  <==会多出 UDP 的通讯协议埠口！
```
```
# 范例三：透过 ICMP 封包的检测，分析区网内有几部主机是启动的
[root@www ~]# nmap -sP 192.168.1.0/24
Starting Nmap 5.21 ( http://nmap.org ) at 2011-07-20 17:05 CST
Nmap scan report for www.centos.vbird (192.168.1.100)
Host is up.
Nmap scan report for 192.168.1.101 <==这三行讲的是 192.168.101 的范例！
Host is up (0.00024s latency).
MAC Address: 00:1B:FC:58:9A:BB (Asustek Computer) 
Nmap scan report for 192.168.1.254
Host is up (0.00026s latency).
MAC Address: 00:0C:6E:85:D5:69 (Asustek Computer)
Nmap done: 256 IP addresses (3 hosts up) scanned in 3.81 seconds
```
如果你还想要将各个主机的启动的 port 作一番侦测的话，那就得要使用：
```
[root@www ~]# nmap 192.168.1.0/24
```
## 埠口与服务的启动/关闭及开机时状态设定
关闭 port 的方法，直接使用 kill ，不过因为 kill 这个指令通常具有强制关闭某些程序的功能，要正常的关闭该程序，就利用系统给我们的 script 来关闭。
### stand alone 与 super daemon
在一般正常的 Linux 系统环境下，服务的启动与管理主要有两种方式：
* Stand alone  
  stand alone 就是直接执行该服务的执行档，让该执行文件直接加载到内存当中运作， 用这种方式来启动可以让该服务具有较快速响应的优点。一般来说，这种服务的启动 script 都会放置到 /etc/init.d/ 这个目录底下，所以你通常可以使用：『 /etc/init.d/sshd restart 』之类的方式来重新启动这种服务；
* Super daemon  
  用一个超级服务作为总管，来统一管理某些特殊的服务。在 CentOS 6.x 里面使用的则是 xinetd 这个 super daemon 。这种方式启动的网络服务虽然在响应上速度会比较慢， 不过，可以透过 super daemon 额外提供一些控管，例如控制何时启动、何时可以进行联机、 那个 IP 可以连进来、是否允许同时联机等等。通常个别服务的配置文件放置在 /etc/xinetd.d/ 当中，但设定完毕后需要重新以『 /etc/init.d/xinetd restart 』重新来启动才行。

```
[root@www ~]# netstat -tnlp | grep 111
tcp        0      0 0.0.0.0:111    0.0.0.0:*       LISTEN  990/rpcbind
tcp        0      0 :::111         :::*            LISTEN  990/rpcbind

[root@www ~]# which rpcbind
/sbin/rpcbind
# 找到档案后，再以 rpm 处理处理

[root@www ~]# rpm -qf /sbin/rpcbind
rpcbind-0.2.0-8.el6.x86_64

[root@www ~]# rpm -qc rpcbind | grep init
/etc/rc.d/init.d/rpcbind
[root@www ~]# /etc/init.d/rpcbind stop
```
透过上面的这个分析的流程，你可以利用系统提供的很多方便的工具来达成某个服务的关闭。
### 预设启动该的服务
在 Unix like 的系统当中我们都是透过 run level 来设定某些执行等级需要启动的服务，以 Red Hat 系统来说，这些 run level 启动的数据都是放置在 /etc/rc.d/rc[0-6].d/ 里面的。必须要熟悉 chkconfig 或 Red Hat 系统的 ntsysv 这几个指令。  
几个常见的必须要存在的系统服务，这些服务请不要关闭：
服务名称|服务内容
:---:|:---:
acpid|新版的电源管理模块，通常建议开启，不过，某些笔记本电脑可能不支持此项服务，那就得关闭
atd|在管理单一预约命令执行的服务，应该要启动的
crond|在管理工作排程的重要服务，请务必要启动
haldaemon|作系统硬件变更侦测的服务，与 USB 设备关系很大
iptables|Linux 内建的防火墙软件，这个也可以启动
network|要网络就要有他
postfix|系统内部邮件传递服务，不要随便关闭他！
rsyslog|系统的登录文件记录，很重要的，务必启动
sshd|这是系统默认会启动的，可以让你在远程以文字型态的终端机登入
xinetd|就是那个 super daemon ，所以也要启动
## 安全性考虑-关闭网络服务端口

# SELinux 管理原则
SELinux 使用所谓的委任式访问控制 (Mandatory Access Control, MAC) ，他可以针对特定的程序与特定的档案资源来进行权限的控管。 也就是说，即使你是 root ，那么在使用不同的程序时，你所能取得的权限并不一定是 root ，而得要看当时该程序的设定而定。 如此一来，我们针对控制的『主体』变成了『程序』而不是『使用者』。因此，这个权限的管理模式就特别适合网络服务的『程序』了。因为，即使你的程序使用 root 的身份去启动，如果这个程序被攻击而被取得操作权，那该程序能作的事情还是有限的， 因为被 SELinux 限制住了能进行的工作了。  
举例来说， WWW 服务器软件的达成程序为 httpd 这支程序， 而默认情况下， httpd 仅能在 /var/www/ 这个目录底下存取档案，如果 httpd 这个程序想要到其他目录去存取数据时，除了规则设定要开放外，目标目录也得要设定成 httpd 可读取的模式 (type) 才行。即使不小心 httpd 被 cracker 取得了控制权，他也无权浏览 /etc/shadow 等重要的配置文件。
## SELinux 的运作模式
SELinux 是透过 MAC 的方式来控管程序，他控制的主体是程序， 而目标则是该程序能否读取的『档案资源』。
* 主体 (Subject)：  
  SELinux 主要想要管理的就是程序，因此你可以将『主体』跟本章谈到的 process 划上等号；
* 目标 (Object)：  
  主体程序能否存取的『目标资源』一般就是文件系统。因此这个目标项目可以等文件系统划上等号；
* 政策 (Policy)：  
  由于程序与档案数量庞大，因此 SELinux 会依据某些服务来制订基本的存取安全性政策。这些政策内还会有详细的规则 (rule) 来指定不同的服务开放某些资源的存取与否。在目前的 CentOS 6.x 里面仅有提供两个主要的政策如下，一般来说，使用预设的 target 政策即可。
    * targeted：针对网络服务限制较多，针对本机限制较少，是预设的政策；
    * mls：完整的 SELinux 限制，限制方面较为严格。
* 安全性本文 (security context)：  
  主体能不能存取目标除了要符合政策指定之外，主体与目标的安全性本文必须一致才能够顺利存取。 这个安全性本文 (security context) 有点类似文件系统的 rwx 。安全性本文的内容与设定是非常重要，如果设定错误，你的某些服务(主体程序)就无法存取文件系统(目标资源)，当然就会一直出现『权限不符』的错误讯息了。  
  重点在『主体』如何取得『目标』的资源访问权限。(1)主体程序必须要通过 SELinux 政策内的规则放行后，就可以与目标资源进行安全性本文的比对， (2)若比对失败则无法存取目标，若比对成功则可以开始存取目标。问题是，最终能否存取目标还是与文件系统的 rwx 权限设定有关。如此一来，加入了 SELinux 之后，出现权限不符的情况时，你就得要一步一步的分析可能的问题了！ 
### 安全性本文 (Security Context)
CentOS 6.x 的 target 政策已经帮我们制订好非常多的规则了，因此你只要知道如何开启/关闭某项规则的放行与否即可。安全性本文比较麻烦，可能需要自行配置文件案的安全性本文。  
安全性本文存在于主体程序中与目标档案资源中。程序在内存内，所以安全性本文可以存入是没问题。档案的安全性本文放置到其 inode 内，因此主体程序想要读取目标档案资源时，同样需要读取 inode ， 这 inode 内就可以比对安全性本文以及 rwx 等权限值是否正确，而给予适当的读取权限依据。  
观察安全性本文可使用『 ls -Z 』去观察如下：
```
[root@www ~]# ls -Z
-rw-------. root  root  system_u:object_r:admin_home_t:s0     anaconda-ks.cfg
drwxr-xr-x. root  root  unconfined_u:object_r:admin_home_t:s0 bin
-rw-r--r--. root  root  system_u:object_r:admin_home_t:s0     install.log
-rw-r--r--. root  root  system_u:object_r:admin_home_t:s0     install.log.syslog
```
安全性本文主要用冒号分为三个字段 (最后一个字段先略过不看)，这三个字段的意义为：
```
Identify:role:type
身份识别:角色:类型
```
* 身份识别 (Identify)： 相当于账号方面的身份识别！主要的身份识别则有底下三种常见的类型：
    * root：表示 root 的账号身份
    * system_u：表示系统程序方面的识别，通常就是程序
    * user_u：代表的是一般使用者账号相关的身份。
* 角色 (Role)： 透过角色字段，我们可以知道这个数据是属于程序、档案资源还是代表使用者。一般的角色有：
    * object_r：代表的是档案或目录等档案资源，这应该是最常见的
    * system_r：代表的就是程序，不过，一般使用者也会被指定成为 system_r 。
* 类型 (Type)： 在预设的 targeted 政策中， Identify 与 Role 字段基本上是不重要的！重要的在于这个类型 (type) 字段！ 基本上，一个主体程序能不能读取到这个档案资源，与类型字段有关！而类型字段在档案与程序的定义不太相同，分别是：
    * type：在档案资源 (Object) 上面称为类型 (Type)；
    * domain：在主体程序 (Subject) 则称为领域 (domain) 

  domain 需要与 type 搭配，则该程序才能够顺利的读取档案资源
### 程序与档案 SELinux type 字段的相关性
透过身份识别与角色字段的定义， 我们可以约略知道某个程序所代表的意义，基本上，这些对应资料在 targeted 政策下的对应如下：
身份识别|角色|该对应在 targeted 的意义
:---:|:---:|:---:
root|system_r|代表供 root 账号登入时所取得的权限
system_u|system_r|由于为系统账号，因此是非交谈式的系统运作程序
user_u|system_r|一般可登入用户的程序
最重要的字段是类型字段，主体与目标之间是否具有可以读写的权限，与程序的 domain 及档案的 type 有关。这两者的关系我们可以使用达成 WWW 服务器功能的 httpd 这支程序与 /var/www/html 这个网页放置的目录来说明。 
```
[root@www ~]# yum install httpd
[root@www ~]# ll -Zd /usr/sbin/httpd /var/www/html
-rwxr-xr-x. root root system_u:object_r:httpd_exec_t:s0 /usr/sbin/httpd
drwxr-xr-x. root root system_u:object_r:httpd_sys_content_t:s0 /var/www/html
# 两者的角色字段都是 object_r ，代表都是档案！而 httpd 属于 httpd_exec_t 类型，
# /var/www/html 则属于 httpd_sys_content_t 这个类型！
```
httpd 属于 httpd_exec_t 这个可以执行的类型，而 /var/www/html 则属于 httpd_sys_content_t 这个可以让 httpd 领域 (domain) 读取的类型。
1. 首先，我们触发一个可执行的目标档案，那就是具有 httpd_exec_t 这个类型的 /usr/sbin/httpd
2. 该档案的类型会让这个档案所造成的主体程序 (Subject) 具有 httpd 这个领域 (domain)， 我们的政策针对这个领域已经制定了许多规则，其中包括这个领域可以读取的目标资源类型；
3. 由于 httpd domain 被设定为可以读取 httpd_sys_content_t 这个类型的目标档案 (Object)， 因此你的网页放置到 /var/www/html/ 目录下，就能够被 httpd 那支程序所读取了；
4. 但最终能不能读到正确的资料，还得要看 rwx 是否符合 Linux 权限的规范

几个重点，第一个是政策内需要制订详细的 domain/type 相关性；第二个是若档案的 type 设定错误， 那么即使权限设定为 rwx 全开的 777 ，该主体程序也无法读取目标档案资源。
## SELinux 的启动、关闭与观察
目前 SELinux 支持三种模式，分别如下：
* enforcing：强制模式，代表 SELinux 运作中，且已经正确的开始限制 domain/type 了；
* permissive：宽容模式：代表 SELinux 运作中，不过仅会有警告讯息并不会实际限制 domain/type 的存取。这种模式可以运来作为 SELinux 的 debug 之用；
* disabled：关闭，SELinux 并没有实际运作。

通过 getenforce 获知 SELinux 的模式
```
[root@www ~]# getenforce
Enforcing 
```
查看配置文件获取 SELinux 的政策 (Policy)
```
[root@www ~]# vim /etc/selinux/config
SELINUX=enforcing     <==调整 enforcing|disabled|permissive
SELINUXTYPE=targeted  <==目前仅有 targeted 与 mls
```
### SELinux 的启动与关闭
如果改变了政策则需要重新启动；如果由 enforcing 或 permissive 改成 disabled ，或由 disabled 改成其他两个，那也必须要重新启动。这是因为 SELinux 是整合到核心里面去的， 你只可以在 SELinux 运作下切换成为强制 (enforcing) 或宽容 (permissive) 模式，不能够直接关闭 SELinux 。  
如果从 disable 转到启动 SELinux 的模式时， 由于系统必须要针对档案写入安全性本文的信息，因此开机过程会花费不少时间在等待重新写入 SELinux 安全性本文 (有时也称为 SELinux Label) ，而且在写完之后还得要再次的重新启动一次。  
让 SELinux 模式在 enforcing 与 permissive 之间切换的方法为：
```
[root@www ~]# setenforce [0|1]
选项与参数：
0 ：转成 permissive 宽容模式；
1 ：转成 Enforcing 强制模式
```
setenforce 无法在 Disabled 的模式底下进行模式的切换。
> 从 Disabled 切换成 Enforcing 之后，可能会有一些服务无法启动，都会跟你说在 /lib/xxx 里面的数据没有权限读取，所以启动失败。这大多是由于在重新写入 SELinux type (Relable) 出错之故，使用 Permissive 就没有这个错误。最简单的方法就是在 Permissive 的状态下，使用『 restorecon -Rv / 』重新还原所有 SELinux 的类型，就能够处理这个错误。
## SELinux type 的修改
```
# 范例：将 /etc/hosts 复制到 root 家目录，并观察相关的 SELinux 类型变化
[root@www ~]# cp /etc/hosts /root
[root@www ~]# ls -dZ /etc/hosts /root/hosts /root
-rw-r--r--. root root system_u:object_r:net_conf_t:s0  /etc/hosts
dr-xr-x---. root root system_u:object_r:admin_home_t:s0 /root
-rw-r--r--. root root unconfined_u:object_r:admin_home_t:s0 /root/hosts

# 范例：将 /root/hosts 移动到 /tmp 下，并观察相关的 SELinux 类型变化
[root@www ~]# mv /root/hosts /tmp
[root@www ~]# ls -dZ /tmp /tmp/hosts
drwxrwxrwt. root root system_u:object_r:tmp_t:s0       /tmp
-rw-r--r--. root root unconfined_u:object_r:admin_home_t:s0 /tmp/hosts
```
当你单纯的复制时，SELinux 的 type 字段是会继承目标目录的。如果是移动，那么连同 SELinux 的类型也会被移动过去。
### chcon
```
[root@www ~]# chcon [-R] [-t type] [-u user] [-r role] 档案
[root@www ~]# chcon [-R] --reference=范例文件 档案
选项与参数：
-R  ：连同该目录下的次目录也同时修改；
-t  ：后面接安全性本文的类型字段！例如 httpd_sys_content_t ；
-u  ：后面接身份识别，例如 system_u；
-r  ：后面街角色，例如 system_r；
--reference=范例文件：拿某个档案当范例来修改后续接的档案的类型！

# 范例：将刚刚的 /tmp/hosts 类型改为 etc_t 的类型
[root@www ~]# chcon -t net_conf_t /tmp/hosts
[root@www ~]# ll -Z /tmp/hosts
-rw-r--r--. root root unconfined_u:object_r:net_conf_t:s0 /tmp/hosts

# 范例：以 /var/spool/mail/ 为依据，将 /tmp/hosts 修改成该类型
[root@www ~]# ll -dZ /var/spool/mail
drwxrwxr-x. root mail system_u:object_r:mail_spool_t:s0 /var/spool/mail
[root@www ~]# chcon --reference=/var/spool/mail /tmp/hosts
[root@www ~]# ll -Z /tmp/hosts
-rw-r--r--. root root system_u:object_r:mail_spool_t:s0 /tmp/hosts
```
chcon 的修改方式中，我们必须要知道最终我们的 SELinux type 是何种类型后，才能够变更成功。 如果你想要作的是『复原成原有的 SELinux type』，采用下面的指令。
### restorencon
```
[root@www ~]# restorecon [-Rv] 档案或目录
选项与参数：
-R  ：连同次目录一起修改；
-v  ：将过程显示到屏幕上

# 范例：将刚刚 /tmp/hosts 移动至 /root 并以预设的安全性本文改正过来
[root@www ~]# mv /tmp/hosts /root
[root@www ~]# ll -Z /root/hosts
-rw-r--r--. root root system_u:object_r:mail_spool_t:s0 /root/hosts
[root@www ~]# restorecon -Rv /root
restorecon reset /root/hosts context system_u:object_r:mail_spool_t:s0->
system_u:object_r:admin_home_t:s0
# 上面这两行其实是同一行，表示将 hosts 由 mail_spool_t 改为 admin_home_t
```
### 默认目录的安全性本文查询与修改
默认 SELinux type 类型记录在 /etc/selinux/targeted/contexts，但是该目录内有很多不同的数据，此时，我们可以透过 semanage 这个指令的功能来查询与修改
```
[root@www ~]# semanage {login|user|port|interface|fcontext|translation} -l
[root@www ~]# semanage fcontext -{a|d|m} [-frst] file_spec
选项与参数：
fcontext ：主要用在安全性本文方面的用途， -l 为查询的意思；
-a ：增加的意思，你可以增加一些目录的默认安全性本文类型设定；
-m ：修改的意思；
-d ：删除的意思。

# 范例：查询一下 /var/www/ 的预设安全性本文设定为何！
[root@www ~]# yum install policycoreutils-python
[root@www ~]# semanage fcontext -l | grep '/var/www'
SELinux fcontext           类型            Context
/var/www(/.*)?             all files     system_u:object_r:httpd_sys_content_t:s0
/var/www(/.*)?/logs(/.*)?  all files     system_u:object_r:httpd_log_t:s0
....(后面省略)....
```
增加某些自定义的目录的安全性本文
```
# 范例：利用 semanage 设定 /srv/vbird 目录的默认安全性本文为 public_content_t
[root@www ~]# mkdir /srv/vbird
[root@www ~]# ll -Zd /srv/vbird
drwxr-xr-x. root root unconfined_u:object_r:var_t:s0   /srv/vbird
# 如上所示，预设的情况应该是 var_t

[root@www ~]# semanage fcontext -l | grep '/srv'
/srv                  directory    system_u:object_r:var_t:s0 <==看这里
/srv/.*               all files    system_u:object_r:var_t:s0
....(底下省略)....
# 上面则是预设的 /srv 底下的安全性本文数据，不过，并没有指定到 /srv/vbird 

[root@www ~]# semanage fcontext -a -t public_content_t "/srv/vbird(/.*)?"
[root@www ~]# semanage fcontext -l | grep '/srv/vbird'
/srv/vbird(/.*)?          all files  system_u:object_r:public_content_t:s0

[root@www ~]# cat /etc/selinux/targeted/contexts/files/file_contexts.local
# This file is auto-generated by libsemanage
# Please use the semanage command to make changes
/srv/vbird(/.*)?    system_u:object_r:public_content_t:s0
# 其实就是写入这个档案

[root@www ~]# restorecon -Rv /srv/vbird* <==尝试恢复默认值
[root@www ~]# ll -Zd /srv/vbird
drwxr-xr-x. root root system_u:object_r:public_content_t:s0 /srv/vbird
# 有默认值，以后用 restorecon 来修改比较简单！
```
## SELinux 政策内的规则布尔值修订
通过 SELinux 的验证之后才能开始档案权限 rwx 的判断，而 SELinux 的判断主要是 (1)政策内的规则比对与 (2)程序与档案的 SELinux type 要符合才能够放行。
### 政策查询
CentOS 6.x 预设使使用 targeted 政策，可以透过 seinfo 来查询：
```
[root@www ~]# yum install setools-console
[root@www ~]# seinfo [-Atrub]
选项与参数：
-A  ：列出 SELinux 的状态、规则布尔值、身份识别、角色、类别等所有信息
-t  ：列出 SELinux 的所有类别 (type) 种类
-r  ：列出 SELinux 的所有角色 (role) 种类
-u  ：列出 SELinux 的所有身份识别 (user) 种类
-b  ：列出所有规则的种类 (布尔值)

# 范例一：列出 SELinux 在此政策下的统计状态
[root@www ~]# seinfo
tatistics for policy file: /etc/selinux/targeted/policy/policy.24
Policy Version & Type: v.24 (binary, mls)  <==列出政策所在档与版本

   Classes:            77    Permissions:       229
   Sensitivities:       1    Categories:       1024
   Types:            3076    Attributes:        251
   Users:               9    Roles:              13
   Booleans:          173    Cond. Expr.:       208
   Allow:          271307    Neverallow:          0
   Auditallow:         44    Dontaudit:      163738
   Type_trans:      10941    Type_change:        38
   Type_member:        44    Role allow:         20
   Role_trans:        241    Range_trans:      2590
....(底下省略)....
# 从上面我们可以看到这个政策是 targeted ，此政策的 SELinux type 有 3076 个；
# 而针对网络服务的规则 (Booleans) 共制订了 173 条规则！

# 范例二：列出与 httpd 有关的规则 (booleans) 有哪些？
[root@www ~]# seinfo -b | grep httpd
Conditional Booleans: 173
   allow_httpd_mod_auth_pam
   httpd_setrlimit
   httpd_enable_ftp_server
....(底下省略)....
# 你可以看到，有非常多的与 httpd 有关的规则订定
```
同样的，如果你想要找到有 httpd 字样的安全性本文类别时， 就可以使用『 seinfo -t | grep httpd 』来查询。如果查询到相关的类别或者是布尔值后，想要知道详细的规则时， 就得要使用 sesearch 这个指令。
```
[root@www ~]# sesearch [--all] [-s 主体类别] [-t 目标类别] [-b 布尔值]
选项与参数：
--all  ：列出该类别或布尔值的所有相关信息
-t  ：后面还要接类别，例如 -t httpd_t
-b  ：后面还要接布尔值的规则，例如 -b httpd_enable_ftp_server

# 范例一：找出目标档案资源类别为 httpd_sys_content_t 的有关信息
[root@www ~]# sesearch --all -t httpd_sys_content_t
Found 683 semantic av rules:
   allow avahi_t file_type : filesystem getattr ;
   allow corosync_t file_type : filesystem getattr ;
   allow munin_system_plugin_t file_type : filesystem getattr ;
....(底下省略)....
# 『 allow  主体程序安全性本文类别  目标档案安全性本文类别 』
# 如上，说明这个类别可以被那个主题程序的类别所读取，以及目标档案资源的格式。
```
可以很轻易的查询到某个主体程序 (subject) 可以读取的目标档案资源 (Object)。 布尔值里面规范的内容是：
```
# 范例三：我知道有个布尔值为 httpd_enable_homedirs ，请问该布尔值规范多少规则？
[root@www ~]# sesearch -b httpd_enable_homedirs --all
Found 43 semantic av rules:
   allow httpd_user_script_t user_home_dir_t : dir { getattr search open } ;
   allow httpd_sys_script_t user_home_dir_t : dir { ioctl read getattr  } ;
....(后面省略)....
```
规范了非常多的主体程序与目标档案资源的放行与否。实际规范这些规则的，就是布尔值的项目。主体程序能否对某些目标档案进行存取，与这个布尔值非常有关系，因为布尔值可以将规则设定为启动 (1) 或者是关闭 (0)。
### 布尔值的查询与修改
```
[root@www ~]# getsebool [-a] [布尔值条款]
选项与参数：
-a  ：列出目前系统上面的所有布尔值条款设定为开启或关闭值

# 范例一：查询本系统内所有的布尔值设定状况
[root@www ~]# getsebool -a
abrt_anon_write --> off
allow_console_login --> on
allow_cvs_read_shadow --> off
....(底下省略)....
```
如果查询到某个布尔值，并且以 sesearch 知道该布尔值的用途后，想要关闭或启动他：
```
[root@www ~]# setsebool [-P] 布尔值=[0|1]
选项与参数：
-P  ：直接将设定值写入配置文件，该设定数据未来会生效的！

# 范例一：查询 httpd_enable_homedirs 是否为 on，若不为 on 请启动他！
[root@www ~]# getsebool httpd_enable_homedirs
httpd_enable_homedirs --> off  <==结果是 off ，依题意给他启动！

[root@www ~]# setsebool -P httpd_enable_homedirs=1
[root@www ~]# getsebool httpd_enable_homedirs
httpd_enable_homedirs --> on
```
setsebool 最好记得一定要加上 -P 的选项，这样才能将此设定写入配置文件。
## SELinux 登陆文件记录所需服务
### setroubleshoot --> 错误讯息写入 /var/log/messages
几乎所有 SELinux 相关的程序都会以 se 为开头，这个服务也是以 se 为开头，troubleshoot 代表错误克服。这个服务会将关于 SELinux 的错误讯息与克服方法记录到 /var/log/messages 与 /var/log/setroubleshoot/* 里头。安装这个服务，总共需要两个软件，分别是 setroublshoot 与 setroubleshoot-server。  
此外，原本的 SELinux 信息本来是以两个服务来记录的，分别是 auditd 与 setroubleshootd。既然是同样的信息， 因此 CentOS 6.x 将两者整合在 auditd 当中了。所以，并没有 setroubleshootd 的服务存在了。因此， 当你安装好了 setroubleshoot-server 之后，请记得要重新启动 auditd，否则 setroubleshootd 的功能不会被启动的。
```
[root@www ~]# yum install setroubleshoot setroubleshoot-server
[root@www ~]# /etc/init.d/auditd restart <==整合到 auditd 当中了！
```
> 事实上，CentOS 6.x 对 setroubleshootd 的运作方式是： (1)先由 auditd 去呼叫 audispd 服务， (2)然后 audispd 服务去启动 sedispatch 程序， (3)sedispatch 再将原本的 auditd 讯息转成 setroubleshootd 的讯息，进一步储存下来

使用 httpd 这支程序产生的错误来说明:
```
[root@www ~]# /etc/init.d/httpd start
[root@www ~]# netstat -tlnp | grep http
tcp     0   0 :::80   :::*              LISTEN      2218/httpd
```
```

[root@www ~]# echo "My first selinux check" > index.html
[root@www ~]# ll index.html
-rw-r--r--. 1 root root 23 2011-07-20 18:16 index.html  <==权限没问题
[root@www ~]# mv index.html /var/www/html
```
```
[root@www ~]# links http://localhost/index.html -dump
                                   Forbidden

   You don't have permission to access /index.html on this server.

   --------------------------------------------------------------------------

    Apache/2.2.15 (CentOS) Server at localhost Port 80

```
没有权限可以存取 index.html，透过 setroubleshoot 的功能去检查。此时请分析一下 /var/log/messages 的内容：
```
[root@www ~]# cat /var/log/messages | grep setroubleshoot
Jul 21 14:53:20 www setroubleshoot: SELinux is preventing /usr/sbin/httpd 
"getattr" access to /var/www/html/index.html. For complete SELinux messages. 
run sealert -l 6c927892-2469-4fcc-8568-949da0b4cf8d
```
『SElinux 被用来避免 httpd 读取到错误的安全性本文， 想要查阅完整的数据，请执行 sealert -l ...』上面提供的信息并不完整，想要更完整的说明得要靠 sealert 配合侦测到的错误代码来处理。 实际处理后会像这样：
```
[root@www ~]# sealert -l 6c927892-2469-4fcc-8568-949da0b4cf8d
Summary:

SELinux is preventing /usr/sbin/httpd "getattr" access to
/var/www/html/index.html.   <==刚刚在 messages 里面看到的信息！

Detailed Description:       <==接下来是详细的状况解析

SELinux denied access requested by httpd. /var/www/html/index.html may 
be a mislabeled. /var/www/html/index.html default SELinux type is 
httpd_sys_content_t, but its current type is admin_home_t. Changing 
this file back to the default type, may fix your problem.
....(中间省略)....

Allowing Access:  <==重要的项目

You can restore the default system context to this file by executing the
restorecon command. restorecon '/var/www/html/index.html', if this file 
is a directory, you can recursively restore using restorecon -R
'/var/www/html/index.html'.

Fix Command:

/sbin/restorecon '/var/www/html/index.html'  <==如何解决

Additional Information:  <==额外的信息
....(底下省略)....

[root@www ~]# restorecon -Rv '/var/www/html/index.html'
restorecon reset /var/www/html/index.html context unconfined_u:object_r:
admin_home_t:s0->system_u:object_r:httpd_sys_content_t:s0
```
只要照着『Allowing Access』里面的提示去进行处理， 就能够完成你的 SELinux 类型设定了.
### 用 email 或在指令列上面直接提供 setroubleshoot 错误讯息
可以让 setroubleshoot 主动的发送产生的信息到我们指定的 email，这样可以方便我们实时的分析。修改 setroubleshoot 的配置文件即可。/etc/setroubleshoot/setroubleshoot.cfg 这个档案的内容：
```
[root@www ~]# vim /etc/setroubleshoot/setroubleshoot.cfg
[email]
# 大约在 81 行左右，这行要存在才行！
recipients_filepath = /var/lib/setroubleshoot/email_alert_recipients

# 大约在 147 行左右，将原本的 False 修改成 True 先！
console = True

[root@www ~]# vim /var/lib/setroubleshoot/email_alert_recipients
root@localhost
your@email.address

[root@www ~]# /etc/init.d/auditd restart
```
### SELinux 错误克服的总结
(1)需要通过政策的各项规则比对后 (2)才能够进行 SELinux type 安全性本文的比对，这两项工作都得要正确才行。而后续的 SELinux 修改主要是透过 chcon, restorecon, setsebool 等指令来处理的。可以透过分析 /var/log/messages 内提供的 setroubleshoot 的信息来处置。  
如果因为某些原因，举例来说 CentOS 没有规范到的 setroubleshoot 信息时，可能你还是无法了解到事情到底是哪里出错。 那此时我们会这样建议：
1. 在服务与 rwx 权限都没有问题，却无法成功的使用网络服务时；
2. 先使用 setenforce 0 设定为宽容模式；
3. 再次使用该网络服务，如果这样就能用，表示 SELinux 出问题，请往下继续处理。如果这样还不能用，那问题就不是在 SELinux 上面！请再找其他解决方法，底下的动作不适合你；
4. 分析 /var/log/messages 内的信息，找到 sealert -l 相关的信息并且执行；
5. 找到 Allow Access 的关键词，照里面的动作来进行 SELinux 的错误克服；
6. 处理完毕重新 setenforce 1 ，再次测试网络服务

# 被攻击后的主机修复工作
## 网管人员应具备的技能
* 了解什么是需要保护的内容：
* 预防黑客 (Black hats) 的入侵：
* 主机环境安全化：
* 防火墙规则的订定：
* 实时维护你的主机：
* 良好的教育训练课程：
* 完善的备份计划：
## 主机受攻击后复原工作流程
1. 立即拔除网络线：  
   拿掉网络线最主要的功能除了保护自己之外，还可以保护同网域的其他主机。
2. 分析登录文件信息，搜寻可能的入侵途径：  
   找出入侵的途径：
     * 分析登录档：低级的 cracker 通常仅是利用工具软件来入侵你的系统，所以我们可以藉由分析一些主要的登录档来找出对方的 IP 以及可能有问题的漏洞。可以分析 /var/log/messages, /var/log/secure 还有利用 last 指令来找出上次登入者的信息。
     * 检查主机开放的服务：
     * 查询 Internet 上面的安全通报：透过安全通报来了解一下最新的漏洞信息
3. 重要数据备份：  
   检查完了入侵途径，再来就是要备份重要的数据。基本上，重要的数据应该是『非 Linux 系统上面原有的数据』，例如 /etc/passwd, /etc/shadow, WWW 网页的数据, /home 里面的使用者重要档案等等，至于 /etc/*, /usr/, /var 等目录下的数据，就不见得需要备份了。 注意：不要备份一些 binary 执行文件，因为 Linux 系统安装完毕后本来就有这些档案，此外， 这些档案也很有可能『已经被窜改过了』，那备份这些数据，反而造成下次系统还是不干净。
4. 重新全新安装：  
   最好选择适合你自己的安装软件即可，不要全部软件都给他安装上去
5. 软件的漏洞修补：  
   重新安装完毕之后，请立即更新你的系统软件
6. 关闭或移除不需要的服务：
7. 数据回复与恢复服务设定：
8. 连上 Internet：