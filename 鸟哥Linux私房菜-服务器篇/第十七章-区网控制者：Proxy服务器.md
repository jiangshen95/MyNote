# 什么是代理服务器 (Proxy)
就是以类似代理人的身份去取得用户所需要的数据。由于它的『代理』能力，使得我们可以透过代理服务器来达成防火墙功能与用户浏览数据的分析。此外，也可以藉由代理服务器来达成节省带宽的目的，以及加快内部网络对因特网的 WWW 访问速度。
## 什么是代理服务器
当客户端有因特网的数据要求时，Proxy 会帮用户去向目的地取得用户所需要的数据。 所以，当客户端指定 WWW 的代理服务器之后，用户的所有 WWW 相关要求就会通过代理服务器去捉取。  
一般来说，代理服务器会架设在整个区网的单点对外防火墙上头，而在区网内部的计算机就都是透过 Proxy 来向因特网要求数据的，这就是所谓的『代理服务器』。这样的 Proxy server 还可以兼做高阶防火墙之用。  
客户端向外部要求的资料事实上都是 Proxy 帮用户取得的，因此因特网上面看到要求数据者，将会是 Proxy 服务器的 IP 而不是客户端的 IP。
## 代理服务器的运作流程
### 当 Proxy 的快取拥有用户所想要的数据时 (Step a ~ d):
* Client 端向 Server 端发送一个数据需求封包；
* Server 端接收之后，先比对这个封包的『来源』与预计要前往的『目标』网站是否为可接受？ 如果来源与目标都是合法的，或者说，来源与目标网站我们的 Proxy 都能帮忙取得资料时，那么 Server 端会开始替 Client 取得资料。这个步骤中比较重要的就是『比对政策』，有点像是认证的感觉;
* Server 首先会检查自己快取 (新的数据可能在内存中，较旧的数据则放置在硬盘上) 数据， 如果有 Client 所需的数据，那就将数据准备取出，而不经过向 Internet 要求数据的程序；
* 最后当然就是将数据回传给 Client 端；
### 当 Proxy 的快取没有用户所想要的数据时 (Step 1 ~ 5)：
1. Client 端向 Server 端发送一个数据需求封包；
2. Server 端接收之后，开始进行政策比对；
3. Server 发现快取并没有 Client 所需要的资料，准备前往因特网抓取数据；
4. Server 开始向 Internet 发送要求与取得相关资料；
5. 最后当然就是将数据回传给 Client 端。

当 Proxy 曾经帮某位用户取得过 A 数据后，当后来的用户想要重复取得 A 数据时，那么 Proxy 就会从自己的快取里面将 A 数据取出传送给用户，而不用跑到因特网去取得同样的这份资料。『假象网络加速』的功能。
## 上层代理服务器
上层代理服务器最大的好处其实是在于『分流』。  
由于当用户透过 Proxy 连到因特网时，网络看到的是 Proxy 在抓取数据而不是该客户端，因此，Proxy 有可能会被客户端过度的滥用，同时也有可能会被拿来为非作歹。所以，目前绝大部分的 Proxy 已经『停止对外开放』了，仅针对自己的网域内的用户提供本项服务而已。
## 代理服务器与 NAT 服务器的差异
* NAT 服务器的功能：  
  Linux 的 NAT 功能主要透过封包过滤的方式， 并使用 iptables 的 nat 表格进行 IP 伪装 (SNAT) ，让客户端自行前往因特网上的任何地方的一种方式。主要的运作行为是在 OSI 七层协定的二、三、四层。由于是透过封包过滤与伪装，因此客户端可以使用的端口号码 (第四层) 较弹性；
* Proxy 服务器的功能：  
  主要透过 Proxy 的服务程序 (daemon) 提供网络代理的任务，因此 Proxy 能不能进行某些工作，与该服务的程序功能有关。举例来说，如果你的 Proxy 并没有提供邮件或 FTP 代理，那么你的客户端就是无法透过 Proxy 去取得这些网络资源。 主要运作的行为在 OSI 七层协议的应用层部分(所谓的比较"高阶"之意)。

NAT 服务器是由较底层的网络去进行分析的工作，至于通过 NAT 的封包是干嘛用的， NAT 不去管他。至于 proxy 则主要是由一个 daemon 的功能达成的，所以必需要符合该 daemon 的需求，才能达到某些功能。
## 假设代理服务器的用途与优点
一般来说，代理服务器的功能主要有：  
* 作为 WWW 的网页资料取得代理人：这是最主要的功能
* 作为内部区网的单点对外防火墙系统：如果你的 Proxy 是放在内部区网的 Gateway 上头，那么这部代理服务器就能够作为内部计算机的防火墙了。

由于 Proxy 的这种特性，让他很常被使用于大型的企业内部，因为可以达到杜绝内部人员上班时使用非 WWW 以外的网络服务，而且还可以监测用户的资料要求流向与流量。优点：
* 节省单点对外的网络带宽，降低网络负载：当你的 Proxy 用户很多时，那么 Proxy 内部的快取数据将会累积较多。因此客户端想要取得网络上的数据时，很多将会从 Proxy 的快取中取得，而不用向因特网要求资料。 所以可以节省带宽
* 以较短的路径取得网络数据，有网络加速的感觉：例如你可以指定你的 ISP 提供的代理服务器连接到国外，由于 ISP 提供的 Proxy 通常具有较大的对外带宽，因此在对国外网站的数据取得上， 通常会比你自己的主机联机到国外要快的多。此外，从内部硬盘取得的路径总比对外的因特网要短的多。
* 透过上层代理服务器的辅助，达到自动数据分流的效果：
* 提供防火墙内部的计算机连上 Internet：

缺点：
* 容易被内部区网的人员滥用：
* 需要较高超的设定技巧与除错程序：
* 可能会取得旧的错误数据：

# Proxy 服务器的基础设定
## Proxy 所需的 squid 软件及其软件结构
目前代理服务器在 Unix Like 的环境下，大多就是使用 squid 。安装好 squid 之后，它主要的提供的配置文件有：
* /etc/squid/squid.conf  
  这个是主要的配置文件，所有 squid 所需要的设定都是放置在这个档案当中的
* /etc/squid/mime.conf  
  这个档案则是在设定 squid 所支持的 Internet 上面的文件格式，就是所谓的 mime 格式。一般来说，这个档案的预设内容已经能够符合我们的需求了，所以不需要更动他，除非你很清楚的知道你所需要额外支持的 mime 文件格式。

其他重要的目录与档案有：
* /usr/sbin/squid：提供 squid 的主程序
* /var/spool/squid：就是默认的 squid 快取放置的目录。
* /usr/lib64/squid/：提供 squid 额外的控制模块，尤其是影响认证密码方面的程序，都是放在这个目录下的；
## CentOS 预设的 squid 设定
在预设的情况下，CentOS 的 squid 具有底下几个特色：
* 仅有本机 (localhost, 127.0.0.1) 来源可以使用这个 squid 功能
* squid 所监听的 Proxy 服务埠口在 port 3128
* 快取目录所在的位置在 /var/spool/squid/ ，且仅有 100MB 的磁盘高速缓存量
* 除了 squid 程序所需要的基本内存之外，尚提供 8MB 的内存来给热门档案快取在内存中 (因为内存速度比硬盘还快)
* 默认启动 squid 程序的用户为 squid 这个账号 (与磁盘高速缓存目录权限有关)

其实， CentOS 预设的 squid 设定，是仅针对本机 (localhost) 开放的情况，而一大堆设定的默认值， 都是仅针对小型网络环境所指定的数值，同时，很多比较特殊的参数都没有启动。
```
[root@www ~]# vim /etc/squid/squid.conf
# 1. 信任用户与目标控制，透过 acl 定义出 localhost 等相关用户
acl manager proto cache_object              <==定义 manager 为管理功能
acl localhost src 127.0.0.1/32              <==定义 localhost 为本机来源
acl localhost src ::1/128
acl to_localhost dst 127.0.0.0/8 0.0.0.0/32 <==定义 to_localhost 可联机到本机
acl to_localhost dst ::1/128

# 2. 信任用户与目标控制，定义可能使用这部 proxy 的外部用户(内网)
acl localnet src 10.0.0.0/8      <==可发现底下都是 private IP 的设定
acl localnet src 172.16.0.0/12
acl localnet src 192.168.0.0/16
acl localnet src fc00::/7
acl localnet src fe80::/10
# 上述数据设定两个用户 (localhost, localnet) 与一个可取得目标 (to_localhost)

# 3. 定义可取得的数据端口所在！
acl SSL_ports port 443                  <==联机加密的埠口设定
acl Safe_ports port 80          # http  <==公认标准的协议使用埠口
acl Safe_ports port 21          # ftp
acl Safe_ports port 443         # https
# 定义出 SSL_ports 及标准的常用埠口 Safe_ports 两个名称

# 4. 定义这些名称是否可放行的标准依据(有顺序)
http_access allow manager localhost  <==放行管理本机的功能
http_access deny manager             <==其他管理来源都予以拒绝
http_access deny !Safe_ports         <==拒绝非正规的埠口联机要求
http_access deny CONNECT !SSL_ports  <==拒绝非正规的加密埠口联机要求
<==这个位置为你可以写入自己的规则的位置！不要写错了！有顺序之分的！
http_access allow localnet           <==放行内部网络的用户来源
http_access allow localhost          <==放行本机的使用
http_access deny all                 <==全部都予以拒绝

# 5. 网络相关参数，最重要的是那个定义 Proxy 协议埠口的 http_port
http_port 3128     <==Proxy 预设的监听客户端要求的埠口，是可以改的
# 其实，如果想让 proxy server/client 之间的联机加密，可以改用 https_port (923)

# 6. 快取与内存相关参数的设定值，尤其注意内存的计算方式
hierarchy_stoplist cgi-bin ? <==hierarchy_stoplist 后面的关键词 (此例为 cgi-bin)
# 若发现在客户端所需要的网址列，则不快取 (避免经常变动的数据库或程序讯息)
cache_mem 8 MB     <==给proxy额外的内存，用来处理最热门的快取数据(需自己加)

# 7. 磁盘高速缓存，亦即放置快取数据的目录所在与相关设定
cache_dir ufs /var/spool/squid 100 16 256 <==默认使用 100MB 的容量放置快取
coredump_dir /var/spool/squid
# 底下的四个参数得要自己加上来，旧版才有这样的默认值！
minimum_object_size 0 KB    <==小于多少 KB 的数据不要放快取，0 为不限制
maximum_object_size 4096 KB <==与上头相反，大于 4 MB 的数据就不快取到磁盘
cache_swap_low 90   <==与下一行有关，减低到剩下 90% 的磁盘高速缓存为止
cache_swap_high 95  <==当磁盘使用量超过 95% 就开始删除磁盘中的旧快取

# 8. 其他可能会用到的默认值！参考参考即可，并不会出现在配置文件中。
access_log /var/log/squid/access.log squid <==曾经使用过 squid 的用户记录
ftp_user Squid@  <==当以 Proxy 进行 FTP 代理匿名登录时，使用的账号名称
ftp_passive on   <==若有代理 FTP 服务，使用被动式联机
refresh_pattern ^ftp:           1440    20%     10080
refresh_pattern ^gopher:        1440    0%      1440
refresh_pattern -i (/cgi-bin/|\?) 0     0%      0
refresh_pattern .               0       20%     4320
# 上面这四行与快取的存在时间有关，底下内文会予以说明
cache_mgr root               <==预设的 proxy 管理员的 email
cache_effective_user squid   <==启动 squid PID 的拥有者
cache_effective_group squid  <==启动 squid PID 的群组
# visible_hostname <==有时由于 DNS 的问题，找不到主机名会出错，就得加上此设定
ipcache_size 1024  <==以下三个为指定 IP 进行快取的设定值
ipcache_low 90
ipcache_high 95
```
### 使用默认值来启动 squid 并观察相关信息
```
[root@www ~]# /etc/init.d/squid start
init_cache_dir /var/spool/squid... 正在激活 squid: .       [  确定  ]
# 第一次启动会初始化快取目录，因此会出现上述左边的数据，未来这个讯息不会再出现
[root@www ~]# netstat -tulnp | grep squid
Proto Recv-Q Send-Q Local Address   Foreign Address   State    PID/Program name
tcp        0      0 :::3128         :::*              LISTEN   2370/(squid)
udp        0      0 :::45470        :::*                       2370/(squid)
[root@www ~]# chkconfig squid on
```
如果你有设定 icp_port 时，squid 预设会启动 3128 及 3130 两个埠口，其中要注意的是， 实际帮用户进行监听与传送数据的是 port 3128 (TCP)，3130 (UDP) 仅是负责与邻近 Proxy 互相沟通彼此的快取数据库的功能，与实际的用户要求无关。因此，如果你的 proxy 是单纯的单一主机，或者是单纯的作为防火墙功能，那么这个 port 3130 是可以关闭的。将这个设定值批注不使用。
```
[root@www ~]# vim /etc/squid/squid.conf
#Default: VBird 2011/04/06 modified，将下列数据从 3130 改为 0 即可
icp_port 0

[root@www ~]# /etc/init.d/squid restart
```
事实上，如果你的客户端与 proxy 之间的沟通想要使用加密机制的 SSL 功能，以保障客户端的信息避免被窃取时， 那么还有个 https_port 可以取代 http_port 。不过，proxy 并非公开也仅是架设在内部区网， 因此还不需要使用到这个 https_port 。
### 观察与修改快取目录 (cache_dir)：权限与 SELinux
squid 是将数据分成一小块一小块，然后分别放置到个别的目录中。由于较多的目录可以节省在同一个目录内找好多档案的时间。因此，在默认的 /var/spool/squid/ 目录下， squid 又会将它分成两层子目录来存放相关的快取数据，所以观察该目录就会是：
```

[root@www ~]# ls /var/spool/squid
00  01  02  03  04  05  06  07  08  09  0A  0B  0C  0D  0E  0F  swap.state
# 算一下，你会发现共有 16 个子目录！那么我们来看看第一个子目录的内容：
[root@www ~]# ls /var/spool/squid/00
00  08  10  18  20  28 ... 98  A0  A8  B0  B8  C0  C8  D0  D8  E0  E8  F0  F8
01  09  11  19  21  29 ... 99  A1  A9  B1  B9  C1  C9  D1  D9  E1  E9  F1  F9
....(中间省略)....
06  0E  16  1E  26  2E ... 9E  A6  AE  B6  BE  C6  CE  D6  DE  E6  EE  F6  FE
07  0F  17  1F  27  2F ... 9F  A7  AF  B7  BF  C7  CF  D7  DF  E7  EF  F7  FF
# 总共有 256 个子目录出现
```
较多的目录是为了将数据分门别类放置，第一层 16 个与第二层 256 个由 cache_dir 这个重要参数的设定：
* cache_dir ufs /var/spool/squid 100 16 256

在 /var/spool/squid/ 后面的参数意义是：
* 第一个 100 代表的是磁盘使用量仅用掉该文件系统的 100MB
* 第二个 16 代表第一层次目录共有 16 个
* 第三个 256 代表每层次目录内部再分为 256 个次目录

根据 squid 的说法与其他文献的说明，这两层快取目录较佳的配置就是 16 256 以及 64 64 这两种配置， 所以我们也不需要修改相关的数据。需要注意这个目录的档案拥有着与 SELinux 类型。  
/var/squid 改为 500MB ，再加上 /srv/squid 给予 2 GB 容量
```
[root@www ~]# vim /etc/squid/squid.conf
#Default: VBird 2011/04/06 modified，底下的设定除了拿掉 # 之外还得修改！
cache_dir ufs /var/spool/squid 500 16 256
cache_dir ufs /srv/squid 2000 16 256

[root@www ~]# mkdir /srv/squid
[root@www ~]# chmod 750 /srv/squid
[root@www ~]# chown squid:squid /srv/squid
[root@www ~]# chcon --reference /var/spool/squid /srv/squid
[root@www ~]# ll -Zd /srv/squid
drwxr-x---. squid squid system_u:object_r:squid_cache_t:s0 /srv/squid/

[root@www ~]# /etc/init.d/squid restart
```
某些特定的目录 (例如 /home) 是不允许建立快取目录的  
因此 squid 有底下两个重要设定：
* cache_swap_low 90
* cache_swap_high 95

代表当磁盘使用量达 95% 时，比较旧的快取数据将会被删除，当删除到剩下磁盘使用量达 90% 时，就停止持续删除的动作。 通常这个设定值已经足够了，不需要变动他， 除了你的快取太大或太小时，才会调整这个设定值。
### squid 使用的内存计算方式
事实上，除了磁盘容量之外，内存可能是另一个相当重要的影响 proxy 效能的因子。因为 proxy 会将数据存一份在磁盘高速缓存中，但是同时也会将数据暂存在内存当中，以加快未来使用者存取同一份数据的速度。  
其实 cache_mem 是额外的指定一些内存来进行比较『热门』的数据存取， cache_mem 并不是指我要使用多少内存给 squid 使用，而是指 "我还要额外提供多少内存给 squid 使用" 的意思』。由于预设 1GB 的磁盘高速缓存会占用约 10M 的内存，而 squid 本身也会占用约 15MB 的内存， 因此，上个例题中 squid 使用掉的内存就有：
* 2.5 * 10 + 15 + "cache_mem 设定值 (8)"

squid 官方网站建议你的物理内存最好是上面数值的两倍，也就是说，上述的内存使用量已经是 48MB， 则我的物理内存最好至少要有 100 MB 以上，才会有比较好的效能。一般来说，如果你的 Proxy 很多人使用时，这个值越大越好，但是最好也要符合上面的需求。
## 管控信任来源 (如区网) 与目标 (如恶意网站)：acl 与 http_access 的使用
在上面的基础设定中，其实仅有 proxy 服务器本身可以向自己的 proxy 要求网页代理。要开放给区网来使用这个 proxy ，所以要修改信任用户的管控参数。这个 acl 的基本语法为：
```
acl <自定义的 acl 名称> <要控制的 acl 类型> <设定的内容>
```
由于 squid 并不会直接使用 IP 或网域来管控信任目标，而是透过 acl 名称来管理，这个 <acl 名称> 就必须要设定管理的是来源还是目标 (acl 类型) ，以及实际的 IP 或网域 (设定的内容) 。
### 管理是否能使用 proxy 的信任客户端方式：
由于因特网主要有使用 IP 或主机名来作为联机方式的，因此信任用户的来源至少就有底下几种：
* src ip-address/netmask：  
  主要控制『来源的 IP 地址』。比如，内网有两个，分别是 192.168.1.0/24 以及 192.168.100.0/24 ， 那么假设我想要制订一个 vbirdlan 的 acl 名称，那就可以在配置文件内写成：  
  `acl vbirdlan src 192.168.1.0/24 192.168.100.0/24`
* src addr1-addr2/netmask：  
  主要控制『一段范围来源的 IP 地址』。例如：  
  `acl vbirdlan2 src 192.168.1.100-192.168.1.200/24`
* srcdomain .domain.name：  
  如果来源用户的 IP 一直变，所以使用的是 DDNS 的方式来更新主机名与 IP 的对应，此时我们可以使用主机名来开放。例如来源是 .ksu.edu.tw 的来源用户就开放使用权，那就是：  
  `acl vbirdksu srcdomain .ksu.edu.tw`
### 管理是否让 proxy 帮忙代理到该目标去获取数据：
除了管理来源用户之外，我们还能够管理是否让 proxy 服务器到某些目标去获取数据。在预设的设定中， 我们的 proxy 仅管理可以向外取得 port 21, 80, 440... 等端口的目标网站，不是这些端口就无法帮忙代理取得。 至于 IP 或网域则没有管理。基本的管理有这些方式：
* dst ip-addr/netmask：  
  控制不能去的目标网站的 IP ，举例来说，我们不许 proxy 去捉取 120.114.150.21 这部主机的 IP 时，可以写成是：  
  `acl dropip dst 120.114.150.21/32`
* dstdomain .domain.name：  
  控制不能去的目标网站的主机名。举例来说，如果 .facebook.com 那就需要写成：  
  `acl dropfb dstdomain .facebook.com`
* url_regex [-i] ^http://url：  
  使用正规表示法来处理网址列的一种方式。这种方式的网址列必须要完整的输入正规表示法的开始到结尾才行。例如：  
  `acl ksuurl url_regex ^http://www.ksu.edu.tw/cht/.*`
* urlpath_regex [-i] \.gif$：  
  与上一个 acl 非常类似，只是上一个需要填写完整的网址数据，这里则是根据网址列的部分比对来处置。以上述的预设案例来说， 只要网址列结尾是 gif (图片文件) 就符合这个项目了。万一我要找出有问题的色情网站，有出现 /sexy 名称并以 jpg 结尾的， 就予以抵挡，那就是使用：  
  `acl sexurl urlpath_regex /sexy.*\.jpg$`

除了上述的功能之外，我们还能够使用外部的档案来提供相对应的 acl 内容设定值。假设我们想要抵挡的外部主机名常常会变动，那么我们可以使用 /etc/squid/dropdomain.txt 来设定主机名， 然后透过底下的方式来处理  
`acl dropdomain dstdomain "/etc/squid/dropdomain.txt"`  
然后在 dropdomain.txt 当中，一行一个待管理的主机名，这样也能够减少持续修改 squid.conf 的困扰。
### 以 http_access 调整管理信任来源与管控目标的『顺序』：
设定好 acl 之后，接下来就是要看看到底要不要放行，放行与否跟 http_access 这个项目有关。基本上， http_access 就是拒绝 (deny) 与允许 (allow) 两个控件目，然后再加上 acl 名称就能够达到这样的功能了。要特别注意的是：http_access 后面接的数据，是有顺序的。  
假设我要放行内部网络 192.168.1.0/24, 192.168.100.0/24 这两段网域，然后拒绝对外的色情相关图片， 以及 facebook.com 网站，那么就应该要这样做：
```
[root@www ~]# vim /etc/squid/squid.conf
# http_access 是有顺序的，因此建议你找到底下这个关键词行后，将你的资料加在后面
# INSERT YOUR OWN RULE(S) HERE TO ALLOW ACCESS FROM YOUR CLIENTS
acl vbirdlan src 192.168.1.0/24 192.168.100.0/24
acl dropdomain dstdomain .facebook.com
acl dropsex urlpath_regex /sexy.*jpg$
http_access deny dropdomain  <==这三行的『顺序』很重要
http_access deny dropsex
http_access allow vbirdlan

[root@www ~]# /etc/init.d/squid restart
```
如果先放行了 vbirdlan 才抵挡 dropdomain 时，你的设定可能会失败，因为内网已经先放行， 因此后面的规则不会比对，那么 facebook.com 就无法被抵挡。通常的作法是，先将要拒绝的写上去，然后才写要放行的数据。
## 其他额外的功能项目
### 不要进行某些网页的快取动作
在预设的情况下，squid 已经拒绝某些数据的快取了，那就是底下的几个设定值：
```
acl QUERY urlpath_regex cgi-bin \?
cache deny QUERY  <==重点就是这一行！可以拒绝，不要让后面的 URL 被快取！
```
```
# 只要网址列出现 .php 结尾的，就不予以快取
[root@www ~]# vim /etc/squid/squid.conf
acl denyphp urlpath_regex \.php$
cache deny denyphp
# 在此档案的最后新增这两行即可！

[root@www ~]# /etc/init.d/squid restart
```
### 磁盘中快取的存在时间
```
# refresh_pattern <regex>   <最小时间> <百分比> <最大时间>
refresh_pattern ^ftp:           1440    20%     10080
refresh_pattern ^gopher:        1440    0%      1440
refresh_pattern -i (/cgi-bin/|\?) 0     0%      0
refresh_pattern .               0       20%     4320
```
* regex：使用的是正规表示法来分析网址列的资料，如上面第一行设定为网址列开头是 ftp 的意思。
* 最小时间：单位是分钟，当取得这个数据的时间超过这个设定值，则该数据会被判定为旧资料。如上面第一行， 表示当取得的资料超过 1440 分钟时，该资料会被判定为旧数据，若有人尝试读取同样的网址列，那么 squid 会重新抓取该数据，不会使用快取内的旧数据。至于第三行，则表示除了上述的两个开头数据外，其他的数据都是被定义为新的， 因此 squid 只会从快取内抓数据给客户端。
* 百分比：这个项目与『最大时间』有关，当该资料被抓取到快取后，经过最大时间的多少百分比时，该数据就会被重抓。
* 最大时间：与上一个设定有关，就是这个数据存在快取内的最长时间。如上面第一行，最大时间为 10080 分钟，但是当超过此时间的 20% (2016分钟) 时，这个数据也会被判定为旧资料。
### 主机名与管理员的 email 指定
如果你的服务器主机名尚未决定，因此使用的主机名在因特网上面是找不到对应的 IP 的 (因为 DNS 未设定)， 那么在预设的 squid 设定中，恐怕会无法顺利的启动。此时你可以手动的加入一个主机名，就是透过 visible_hostname 来指定。 同时，如果客户端使用 squid 出现任何错误时，屏幕上都会出现管理员的 email 让用户可以回报。现在假设主机名为 www.centos.vbird 且管理员的 email 为 dmtsai@www.centos.vbird ，此时我们可以这样修改：
```
[root@www ~]# vim /etc/squid/squid.conf
cache_mgr dmtsai@www.centos.vbird  <==管理员的 email 
visible_hostname www.centos.vbird  <==直接设定主机名

[root@www ~]# /etc/init.d/squid restart
```
## 安全性设定：防火墙，SELinux 与黑名单档案
### 防火墙得要放行 tcp 的 port 3128
并不是开放防火墙就能使用 proxy server 的资源，还得要使用 acl 配合 http_access 才行
```
[root@www ~]# vim /usr/local/virus/iptables/iptables.allow
iptables -A INPUT -i $EXTIF -p tcp -s 192.168.1.0/24 --dport 3128 -j ACCEPT
# 因为内网 192.168.100.0/24 本来就是全部都接受放行的！

[root@www ~]# /usr/local/virus/iptables/iptables.rule
```
### SELinux 的注意事项
针对 proxy 来说，CentOS 6.x 倒是没有给予太多的规则限制，因此似乎不太需要修订规则。不过，SELinux 的安全本文在类型部分得注意。这包括配置文件 (/etc/squid/ 内的数据) 类型是 squid_conf_t 的样式， 而快取目录的类型则是 squid_cache_t 的类型，且上层类型 (/var/spool/) 应该是要成为 var_t 之类的才行。 修改的方法就是透过 chcon 来处理即可。
### 建立黑名单配置文件
例如，建立一个名为 /etc/squid/dropdomain.txt 的档案，内容为拒绝联机的目标网站。
```
[root@www ~]# vim /etc/squid/squid.conf
# 找到底下的数据，就是 dropdomain 那行，约在 629 行左右，并且修改一下
acl dropdomain dstdomain "/etc/squid/dropdomain.txt"
# 注意一下，如果是档名，请写绝对路径，且使用双引号或单引号圈起来！

[root@www ~]# vim /etc/squid/dropdomain.txt
.facebook.com
.yahoo.com
# 一行一个 domain 名称即可

[root@www ~]# /etc/init.d/squid reload
```
这个方法的好处是，你可以使用额外的控制方式去修改 /etc/squid/dropdomain.txt 这个档案的内容， 并且修改完毕后再使用 reload 去加载配置文件，不必要重新启动 (restart)，因为 reload 的速度比较快速。

# 客户端的使用与测试
在浏览器上设定。

# 服务器的其他应用设定
除了基本的 proxy 设定之外，如果你还有其他可供利用的上层代理服务器，说不定我们就能够设计一下如何进行分流的动作。以及认证功能。
## 上层 Proxy 与获取数据分流的设定
设定上层代理服务器与分流的参数主要有： cache_peer, cache_peer_domain, cache_peer_access 等
### cache_peer 的相关语法
```
cache_peer [上层proxy主机名] [proxy角色] [proxy port] [icp port] [额外参数]
```
这个设定值就是在规范上层代理服务器在哪里，以及我们想要对这部代理服务器如何查询的相关设定值
* 上层 proxy 主机名：
* proxy 角色：这部 proxy 是我们的上层 (parent) ？还是作为我们邻近 (sibling) 的协力运作的 proxy ？ 因为我们要利用上层去捉取数据，因此经常使用的是 parent 这个角色值；
* proxy port：通常就是 3128
* icp port：通常就是 3130 
* 额外参数：针对这部上层 proxy 我们想要对它进行的查询数据的行为设定。主要有：
    * proxy-only：向上层 proxy 要到的数据不会快取到本地的 proxy 服务器内，降低本地 proxy 负担；
    * wieght=n：权重的意思，因为我们可以指定多部上层 Proxy 主机，哪一部最重要？就可以利用这个 weight 来设定，n 越大表示这部 Proxy 越重要
    * no-query：如果向上层 Proxy 要求资料时，可以不需要发送 icp 封包，以降低主机的负担
    * no-digest：表示不向附近主机要求建立 digest 纪录表格
    * no-netdb-exchange：表示不向附近的 Proxy 主机送出 imcp 的封包要求
### cache_peer_domain 的相关语法
```
cache_peer_domain [上层proxy主机名] [要求的领域名]
```
要使用这部上层代理服务器向哪个领域名要求数据。
### cache_peer_access 的相关语法
```
cache_peer_access [上层proxy主机名] [allow|deny] [acl名称]
```
与 cache_peer_domain 相当类似，只是 cache_peer_domain 直接规范了主机名 (domain name)， 而如果你想要设计的并非领域名，而是某些特定的 IP 网段时，就得要先用 acl 设计一个名称后， 再以这个 cache_peer_access 去放行 (allow) 或拒绝 (deny) 读取了。  
根据上述的语法说明，那么我们想要达到 .cn 使用 hinet.centos.vbird 这部服务器的代理功能时， 应该要这样设计的：
```
[root@www ~]# vim /etc/squid/squid.conf
cache_peer hinet.centos.vbird parent 3128 3130 proxy-only no-query no-digest
cache_peer_domain hinet.centos.vbird .cn

[root@www ~]# /etc/init.d/squid reload
```
如果你还有其它的需求再利用 acl 规范了目标位置后，再以 cache_peer_access 去放行。
## Proxy 服务放在 NAT 服务器上：通透式代理 (Transparent Proxy)
强制使用者一定要使用 proxy 。 (1)在对外的防火墙服务器 (NAT) 上面安装 proxy； (2)在 proxy 上头启动 transparent 功能； (3) NAT 服务器加上一条 port 80 转 port 3128 的规则，如此一来，所有往 port 80 的封包就会被你的 NAT 转向 port 3128 ， 而你的 port 3128 就是 proxy ，那大家就得要用你的 proxy ，而且重点是，浏览器不需要进行任何设定。
```
# 1. 设定 proxy 成为通透式代理服务器的功能
[root@www ~]# vim /etc/squid/squid.conf
http_port 3128  transparent
# 找到 3128 这行后，在最后面加上 transparent 即可

[root@www ~]# /etc/init.d/squid reload
```
接下来，将来自 192.168.100.0/24 这个内网的来源，只要是要求 port 80 的，就将它重新导向 port 3128 的方式为：
```
[root@www ~]# vim /usr/local/virus/iptables/iptables.rule
iptables -t nat -A PREROUTING -i $INIF -s 192.168.100.0/24 -p tcp \
         --dport 80 -j REDIRECT --to-ports 3128
# 将上述这一行加在最底下 /etc/init.d/iptables save 的上面一行即可！

[root@www ~]# /usr/local/virus/iptables/iptables.rule
```
### 仅具有 proxy 无快取功能的代理
假设 transparent proxy 已经设定妥当，那么接下来就是让你的快取目录空空如也，且再也不写入任何资料。 此外，也不要有多余的内存来记录热门档案。
```
# 先关闭 squid ，然后删除快取目录，之后再重建快取目录，此时快取目录就空了
[root@www ~]# /etc/init.d/squid stop
[root@www ~]# rm -rf /var/spool/squid/*
[root@www ~]# vim /etc/squid/squid.conf
cache_dir ufs /var/spool/squid 100 16 256 read-only
#cache_dir ufs /srv/squid 2000 16 256
# 额外的那个 /srv/squid 批注掉，然后第一行多个 read-only 字样！
cache_mem 0 MB
# 本来规范有 32MB ，现在不要了！

[root@www ~]# /etc/init.d/squid start
```
## Proxy 的认证设定
可以透过认证来简单的输入账号密码， 若通过验证，就可以立刻使用我们的 proxy 了。其实 squid 提供很多认证功能， 我们需要的是最简单的功能即可。使用的是 squid 主动提供的 ncsa_auth 认证模块，这个模块会利用 apache (WWW 服务器) 提供的帐密建立指令 (htpasswd) 所制作的密码文件作为验证依据。所以，我们至少需要检查有没有这两样东西：
```
[root@www ~]# rpm -ql squid | grep ncsa
/usr/lib64/squid/ncsa_auth    <==就是这个验证模块档案！注意完整路径
/usr/share/man/man8/ncsa_auth.8.gz

[root@www ~]# yum install httpd   <==apache 软件安装
[root@www ~]# rpm -ql httpd | grep htpasswd
/usr/bin/htpasswd           <==就是需要这个帐密建立指令
/usr/share/man/man1/htpasswd.1.gz
```
案例设定：
* 内部网域 192.168.100.0/24 要使用 proxy 的，还是不需要透过验证；
* 外部主机想要使用 proxy (例如 192.168.1.0/24 这段) 才需要验证；
* 使用 NCSA 的基本身份验证方式，且密码文件建立在 /etc/squid/squid_user.txt
* 上述档案仅有一个用户 vbird ，他的密码为 1234
```
# 1. 先修改 squid.conf 档案内容
[root@www ~]# vim /etc/squid/squid.conf
# 1.1 先设定验证相关的参数
auth_param basic program /usr/lib64/squid/ncsa_auth /etc/squid/squid_user.txt
auth_param basic children 5
auth_param basic realm Welcome to VBird's proxy-only web server
# 非特殊字体为关键词不可更动，第一行为透过 ncsa_auth 读取 squid_user.txt 密码
# 第二行为启动 5 个程序 (squid 的子程序) 来管理验证的需求；
# 第三行为验证时，显示给用户看的欢迎讯息，这三行可写在最上面！

# 1.2 然后是针对验证功能放行与否的 acl 与 http_access 设定
acl vbirdlan src 192.168.100.0/24  <==修改一下，取消 192.168.1.0/24
acl dropdomain dstdomain "/etc/squid/dropdomain.txt"
acl dropsex urlpath_regex /sexy.*jpg$
acl squid_user proxy_auth REQUIRED <==建立一个需验证的 acl 名称
http_access deny dropdomain
http_access deny dropsex
http_access allow vbirdlan
http_access allow squid_user       <==请注意这样的规则顺序，验证在最后

# 2. 建立密码数据
[root@www ~]# htpasswd -c /etc/squid/squid_user.txt vbird
New password:
Re-type new password:
Adding password for user vbird
# 第一次建立才需要加上 -c 的参数，否则不需要加上 -c 

[root@www ~]# cat /etc/squid/squid_user.txt
vbird:vRC9ie/4E21c.  <==这就是用户与密码

[root@www ~]# /etc/init.d/squid restart
```
比较需要注意『acl squid_user proxy_auth REQUIRED』这一串设定，proxy_auth 是关键词，而 REQUIRED 则是指定任何在密码文件内的用户都能够使用验证的意思。如果一切顺利的话，那么你的内网依旧可以使用 transparent proxy ， 而外网则需要输入账密才能够使用 proxy server 提供的代理能力。
## 末端登陆档分析：sarg
事实上， squid 已经收集了众多的登录文件分析软件了，而且大多是免费的 (http://www.squid-cache.org/Scripts/) ，你可以依照自己的喜好来加以安装与分析你的 squid 登录档。  
Squid Analysis Report Generator (Squid 分析报告制作者)，他的官方网站在： http://sarg.sourceforge.net/sarg.php，他的原理相当的简单，就是将 logfile 拿出来，然后进行一下解析，依据不同的时间、网站、与热门网站等等来进行数据的输出。  
因为 SARG 功能太强大了，所以记录的『数据量』就实在是多了点，如果你的 Proxy 网站属于那种很大流量的网站时，那么就不要使用『日报表』。
```
[root@www ~]# yum install gd
[root@www ~]# rpm -ivh sarg-2.2.3.1-1.el5.rf.x86_64.rpm
[root@www ~]# vim /etc/sarg/sarg.conf
title "Squid 使用者存取报告"           <==第 49 行左右
font_size 12px                         <==第 69 行左右
charset UTF-8                          <==第 353 行左右

# 1. 一口气制作所有登录文件内的数据报表
[root@www ~]# sarg
SARG: Records in file: 2285, reading: 100.00%  <==列出分析信息

# 2. 制作 8 月 2 日的报表
[root@www ~]# sarg -d 02/08/2011
# 这两个范例，都会将数据丢到 /var/www/sarg/ONE-SHOT/ 底下去；

# 3. 制作昨天的报表
[root@www ~]# sh /etc/cron.daily/sarg
# 这个范例则是将每天的数据放置于 /var/www/sarg/daily/ 底下去！
```
如果想要查阅数据，只要在 proxy server 端输入 http://your.hostname/sarg 。