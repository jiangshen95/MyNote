# NIS 的由来与功能
在一个大型的网域当中，如果有多部 Linux 主机，万一要每部主机都需要设定相同的账号与密码时，如果能够有一部账号主控服务器来管理网域中所有主机的账号， 当其他的主机有用户登入的需求时，才到这部主控服务器上面要求相关的账号、密码等用户信息， 如此一来，如果想要增加、修改、删除用户数据，只要到这部主控服务器上面处理即可， 这样就能够降低重复设定使用者账号的步骤了。  
这样的功能有很多的服务器软件可以达成，这里我们要介绍的则是 Network Information Services (NIS server) 这个服务器软件。
> NIS 主要提供的是用户的账号、密码、家目录文件名、UID等信息，但 NIS 并没有提供文件系统。同时， NIS 同样使用 RPC 服务器
## NIS 的主要功能：管理账户信息
通常我们都会建议，一部 Linux 主机的功能越单纯越好，也就是说，一部 Linux 就专门进行一项服务。这样有许多的好处，这包含功能单纯所以系统资源得以完整运用， 并且在发生入侵或者是系统产生状况的时候，也比较容易追查问题所在。因此，一个公司内部常常会有好几部 Linux 主机，有的专门负责 WWW 、有的专门负责 Mail 、有的专门负责 SAMBA 等等的服务。  
由于是同一个公司内的多部主机，所以事实上所有的 Linux 主机的账号与密码都是一样的。如果我设计了一部专门管理账号与密码的服务器，而其他的 Linux 主机当有客户端要登入的时候，就必须要到这部管理密码的服务器来查寻用户的账号与密码， 如此一来，我要管理所有的 Linux 主机的账号与密码，只要到那部主服务器上面去进行设定即可。包括新进人员的设定。这就是 Network Information Service, NIS 服务器的主要功能。  
事实上，Network Information Service 最早应该是称为 Sun Yellow Pages (简称 yp)，也就是 Sun 这家公司出的一个名为 Yellow Pages 的服务器软件。  
NIS 服务器主要有底下这些基本的数据提供给有登入需求的主机：
服务器端文件名|档案内容
:---:|:---:
/etc/passwd|提供用户账号、UID、GID、家目录所在、Shell 等等
/etc/group|提供群组数据以及 GID 的对应，还有该群组的加入人员
/etc/hosts|主机名与 IP 的对应，常用于 private IP 的主机名对应
/etc/services|每一种服务 (daemons) 所对应的埠口 (port number)
/etc/protocols|基础的 TCP/IP 封包协定，如 TCP, UDP, ICMP 等
/etc/rpc|每种 RPC 服务器所对应的程序号码
/var/yp/ypservers|NIS 服务器所提供的数据库
## NIS 的运作流程：透过 RPC 服务
由于 NIS 服务器主要是提供用户登入的信息给客户端主机来查询之用，所以， NIS 服务器所提供的数据就需要用到传输与读写比较快速的 "数据库" 文件系统， 而不是传统的纯文本数据。为了要达到这个目的，NIS 服务器就必须要将前一小节提到的那些档案制作成为数据库档案， 然后使用网络协议让客户端主机来查询。所使用的通讯协议与前一章的 NFS 相同，都使用远程过程调用 (RPC) 。  
此外，如果在一个很大型的网域里面，万一所有的 Linux 主机都向同一部 NIS 服务器要求用户数据时， 这部 NIS 服务器的负载 (loading) 可能会过大。 所以，在较为大型的企业环境当中， NIS 服务器可以使用 master/slave (主控/辅助服务器) 架构的。  
Master NIS 服务器提供系统管理者制作的数据库， slave 则取得来自 master 的数据，并藉以提供其他客户端的查询。 客户端可以向整个网域要求用户资料的响应，master 与 slave 皆可回答， 由于 slave 的数据来自于 master ，所以用户账号数据本身是同步的。如此一方面可以分散 NIS 服务器的负载，而且也可以避免因 NIS 服务器挂点而导致的无法登入的风险。
首先必须要有 NIS server 的存在，之后才会有 NIS Client 的存在。 那么当使用者有登入的需求时，整个 NIS 的运作程序是：
* 关于 NIS Server (master/slave) 的运作程序：
    1. NIS Master 先将本身的账号密码相关档案制作成为数据库档案；
    2. NIS Master 可以主动的告知 NIS slave server 来更新；
    3. NIS slave 亦可主动的前往 NIS master server 取得更新后的数据库档案；
    4. 若有账号密码的异动时，需要重新制作 database 与重新同步化 master/slave。
* 关于当 NIS Client 有任何登入查询的需求时：
    1. NIS client 若有登入需求时，会先查询其本机的 /etc/passwd, /etc/shadow 等档案；
    2. 若在 NIS Client 本机找不到相关的账号数据，才开始向整个 NIS 网域的主机广播查询；
    3. 每部 NIS server (不论 master/slave) 都可以响应，基本上是『先响应者优先』。

如果你的 NIS client 本身就有很多一般使用者的账号时，那跟 NIS server 所提供的账号就可能产生一定程度的差异。所以，一般来说，在这样的环境下，NIS client 或 NIS slave server 会主动拿掉自己本机的一般使用者账号，仅会保留系统所需要的 root 及系统账号而已。 如此一来，一般使用者才都会经由 NIS master server 所控管。  
NIS 环境大致上需要设定的基本组件就有：
* NIS Master server ：将档案建置成数据库，并提供 slave server 来更新；
* NIS Slave server ：以 Master server 的数据库作为本身的数据库来源；
* NIS client ：向 master/server 要求登入者的验证数据。

# NIS Server 端的设定
## 所需要的软件
由于 NIS 服务器需要使用 RPC 协议，且 NIS 服务器同时也可以当成客户端，因此它需要的软件就有底下这几个：
* yp-tools ：提供 NIS 相关的查寻指令功能
* ypbind   ：提供 NIS Client 端的设定软件
* ypserv   ：提供 NIS Server 端的设定软件
* rpcbind  ：就是 RPC 一定需要的数据
## NIS 服务器相关的配置文件
在 NIS 服务器上最重要的就是 ypserv 这个软件了，但是，由于 NIS 设定时还会使用到其他网络参数设定数据， 因此在配置文件方面需要有底下这些数据：
* /etc/ypserv.conf：这是最主要的 ypserv 软件所提供的配置文件，可以规范 NIS 客户端是否可登入的权限。
* /etc/hosts：由于 NIS server/client 会用到网络主机名与 IP 的对应，因此这个主机名对应档就显的相当重要！每一部主机名与 IP 都需要记录才行！
* /etc/sysconfig/network：可以在这个档案内指定 NIS 的网域 (nisdomainname)。
* /var/yp/Makefile：前面不是说账号数据要转成数据库文件吗？ 这就是与建立数据库有关的动作配置文件；

 NIS 服务器提供的主要服务方面有底下两个：
* /usr/sbin/ypserv：就是 NIS 服务器的主要提供服务；
* /usr/sbin/rpc.yppasswdd：提供额外的 NIS 客户端之用户密码修改服务， 透过这个服务， NIS 客户端可以直接修改在 NIS 服务器上的密码。相关的使用程序则是 yppasswd 指令；

与账号密码的数据库有关的指令方面有底下几个：
* /usr/lib64/yp/ypinit：建立数据库的指令，非常常用 (在 32 位的系统下，文件名则是 /usr/lib/yp/ypinit )
* /usr/bin/yppasswd：与 NIS 客户端有关，主要在让用户修改服务器上的密码。
## 一个实作案例
仅介绍 NIS master server 与 NIS client 两个组件而已，如果你有需要额外的 slave 的话， 再请查阅 NIS 官网的介绍。
* NIS 的域名为 vbirdnis
* 整个内部的信任网域为 192.168.100.0/24
* NIS master server 的 IP 为 192.168.100.254 ，主机名为 www.centos.vbird
* NIS client 的 IP 为 192.168.100.10，主机名为 clientlinux.centos.vbird
## NIS Server 的设定与启动
首先，你必须要在 NIS 服务器上面搞定你的账号与密码相关数据， 这包括 /etc/passwd, /etc/shadow, /etc/hosts, /etc/group .... 等等。
### 1. 先设定 NIS 的域名 (NIS domain name)
NIS 是会分领域名 (domain name) 来分辨不同的账号密码数据的，因此你必须要在服务器与客户端都指定相同的 NIS 领域名才行。设定这个 NIS 领域名的动作很简单，就直接编辑 /etc/sysconfig/network 即可
```
[root@www ~]# vim /etc/sysconfig/network
# 不要更改其他既有数据，只要加入底下这几行即可：
NISDOMAIN=vbirdnis      <==设定 NIS 领域名
YPSERV_ARGS="-p 1011"   <==设定 NIS 每次都启动在固定的埠口
```
也可以使用手动的方式暂时设定好你的 NIS 领域名，透过的方法就是 nisdomainname 这个指令。 (其实 nisdomainname 与 ypdomainname 及 domainname 都是一模一样的指令，只要记住一个指令名称即可。请自行 man domainname) 不过，这个指令现在大概只用来检查设定是否正确，因为启动 NIS 服务器时，服务器去捉取的数据就是从 network 这个档案里面捉取的。所以只要改这个配置文件即可。  
另外，由于未来想使用 iptables 直接管理 NIS 的使用，因此我们想要控制 NIS 启动在固定的埠口上。此时， 就使用『YPSERV_ARGS="-p 1011"』这个设定值来固定埠口在 1011 。
### 2. 主要配置文件 /etc/ypserv.conf
```
[root@www ~]# vim /etc/ypserv.conf
dns: no
# NIS 服务器大多使用于内部局域网络，只要有 /etc/hosts 即可，不用 DNS 

files: 30
# 预设会有 30 个数据库被读入内存当中，其实我们的账号档案并不多，30 个够用了。

xfr_check_port: yes
# 与 master/slave 有关，将同步更新的数据库比对所使用的端口，放置于 <1024 内。

# 底下则是设定限制客户端或 slave server 查询的权限，利用冒号隔成四部分：
# [主机名/IP] : [NIS域名] : [可用数据库名称] : [安全限制]
# [主机名/IP]   ：可以使用 network/netmask 如 192.168.100.0/255.255.255.0 
# [NIS域名]   ：例如本案例中的 vbirdnis
# [可用数据库名称]：就是由 NIS 制作出来的数据库名称；
# [安全限制]      ：包括没有限制 (none)、仅能使用 <1024 (port) 及拒绝 (deny)
# 一般来说，你可以依照我们的网域来设定成为底下的模样：
127.0.0.0/255.255.255.0     : * : * : none
192.168.100.0/255.255.255.0 : * : * : none
*                           : * : * : deny
# 星号 (*) 代表任何数据都接受的意思。上面三行的意思是，开放 lo 内部接口、
# 开放内部 LAN 网域，且杜绝所有其他来源的 NIS 要求的意思。

# 还有一个简单作法，你可以先将上面三行批注，然后加入底下这一行即可：
*                         : * : * : none
```
可以选择使用『 * : * : * : none 』那个设定值，然后透过 iptables 来管控可使用的来源。
### 3. 设定主机名与 IP 的对应 (/etc/hosts)
在 /etc/ypserv.conf 的设定当中我们谈到 NIS 大部分是给局域网络内的主机使用的，所以当然就不需要 DNS 的设定了。不过，由于 NIS 使用到很多的主机名，但是网络联机透过的是 IP ，所以你一定要设定好 /etc/hosts 里面的主机名与 IP 的对应，否则会无法成功联机 NIS 。
```
[root@www ~]# vim /etc/hosts
# 原本就有的 localhost 与 127.0.0.1 之类的设定都不要更动，只要新增数据：
192.168.100.254   www.centos.vbird
192.168.100.10    clientlinux.centos.vbird

[root@www ~]# hostname
www.centos.vbird
# 再做个确认，确定输出的主机名与本机 IP 确实有写入 /etc/hosts 
```
如果你的主机名 (hostname) 与 NIS 的主机名不一样，那么在这个档案当中还是需要将你的主机名给他设定进来，否则在后面数据库的设定时，肯定会发生问题。当然啦，你也可以直接在 /etc/sysconfig/network 当中直接重新设定主机名，然后重新启动，或者是利用 hostname 这个指令重新设定你的主机名也可以。
### 4. 启动与观察所有相关的服务
为了也让 yppasswdd 启动在固定的埠口，方便防火墙的管理， 因此，我们也建议你可以设定一下 /etc/sysconfig/yppasswdd 
```
[root@www ~]# vim /etc/sysconfig/yppasswdd
YPPASSWDD_ARGS="--port  1012"    <==找到这个设定值，修改一下内容成这样！

[root@www ~]# /etc/init.d/ypserv start
[root@www ~]# /etc/init.d/yppasswdd start
[root@www ~]# chkconfig ypserv on
[root@www ~]# chkconfig yppasswdd on
```
注意，主要的 NIS 服务是 ypserv ，不过，如果要提供 NIS 客户端的密码修改功能的话， 最好还是得要启动 yppasswdd 这个服务。在启动完毕后，我们可以利用 rpcinfo 来检查：
```
[root@www ~]# rpcinfo -p localhost
   program vers proto   port  service
    100000    4   tcp    111  portmapper
    100000    4   udp    111  portmapper
    100004    2   udp   1011  ypserv
    100004    1   udp   1011  ypserv
    100004    2   tcp   1011  ypserv
    100004    1   tcp   1011  ypserv
    100009    1   udp   1012  yppasswdd
# 看看埠口是否为我们规定的 1011, 1012 ，若不是的话，得要修改一下配置文件。

[root@www ~]# rpcinfo -u localhost ypserv
program 100004 version 1 ready and waiting
program 100004 version 2 ready and waiting
```
### 5. 处理账号并建立数据库
在完成了上面的所有步骤后，接下来我们得要开始将主机上面的账号档案转成数据库档案。不过，因为担心与 NIS 客户端的账号有冲突，加上之前我们已经建立过一些账号了。所以，这里我们建立三个新账号， 分别是 nisuser1, nisuser2, nisuser3 。不过账号主要是依据 UID 来判断，因此，我们使用大于 1000 的 UID 来建立这三个账号。
```
[root@www ~]# useradd -u 1001 nisuser1
[root@www ~]# useradd -u 1002 nisuser2
[root@www ~]# useradd -u 1003 nisuser3
[root@www ~]# echo password | passwd --stdin nisuser1
[root@www ~]# echo password | passwd --stdin nisuser2
[root@www ~]# echo password | passwd --stdin nisuser3
```
接下来，将建立的帐密数据转成数据库，转换的动作直接透过 /usr/lib64/yp/ypinit 这个指令来处理即可。整个步骤是这样做的：
```
[root@www ~]# /usr/lib64/yp/ypinit -m

At this point, we have to construct a list of the hosts which will run NIS
servers.  www.centos.vbird is in the list of NIS server hosts.  Please continue
to add the names for the other hosts, one per line.  When you are done with the
list, type a <control D>.
        next host to add:  www.centos.vbird  <==系统根据主机名自动捉取
        next host to add:                    <==这个地方按下 [crtl]-d
The current list of NIS servers looks like this:

www.centos.vbird

Is this correct?  [y/n: y]  y
We need a few minutes to build the databases...
Building /var/yp/vbirdnis/ypservers...
Running /var/yp/Makefile...
gmake[1]: Entering directory `/var/yp/vbirdnis'
Updating passwd.byname...
Updating passwd.byuid...
....(中间省略)....
gmake[1]: Leaving directory `/var/yp/vbirdnis'

www.centos.vbird has been set up as a NIS master server.

Now you can run ypinit -s www.centos.vbird on all slave server.
```
要注意出现的信息当中，在告知你可以直接输入 [ctrl]-d 以结束的那个地方， 你的主机名会主动的被捉出来，这个主机名务必需要在 /etc/hosts 可以被找到 IP 的对应， 否则会出现问题。另外，万一在执行 ypinit -m 时，出现如下的错误，那肯定就是有些数据没有被建立了。
```
gmake[1]: *** No rule to make target `/etc/aliases', needed by `mail.aliases'.  Stop.
gmake[1]: Leaving directory `/var/yp/vbirdnis'
make: *** [target] Error 2
Error running Makefile.
Please try it by hand.

[root@www ~]# touch /etc/aliases
# 解决方法很简单。缺少什么档案，就 touch 他

[root@www ~]# /usr/lib64/yp/ypinit -m
# 然后再重新执行一次即可
```
如果出现如下错误，可能因为：
* 你的 ypserv 服务没有顺利启动，请利用 rpcinfo 检查看看；
* 你的主机名与 IP 没有对应好，请检查 /etc/hosts
```
gmake[1]: Entering directory `/var/yp/vbirdnis'
Updating passwd.byname...
failed to send 'clear' to local ypserv: RPC: Program not registeredUpdating passwd.byuid...
failed to send 'clear' to local ypserv: RPC: Program not registeredUpdating group.byname...
....(底下省略)....
```
如果你的用户密码有变动过，那么你就得要重新制作数据库，重新启动 ypserv 及 yppasswdd 。
## 防火墙设置
NIS 与 NFS 都是使用 RPC Server 的，所以啰，除了上述谈到的固定埠口之外， 你还得要开放 port 111 才行。
```
[root@www ~]# vim /usr/local/virus/iptables/iptables.allow
iptables -A INPUT -i $EXTIF -p tcp -s 192.168.100.0/24 --dport 1011 -j ACCEPT
iptables -A INPUT -i $EXTIF -p udp -s 192.168.100.0/24 -m multiport \
         --dport 1011,1012 -j ACCEPT

[root@www ~]# /usr/local/virus/iptables/iptables.rule
```

# NIS Client 端的设定
在 NIS client 端有登入需求时，NIS client 基本上还是先搜寻自己的 /etc/passwd, /etc/group 等数据后才再去找 NIS server 的数据库。所以 NIS client 最好能够将本身的账号密码删除到仅剩下系统账号，亦即 UID, GID 均小于 500 以下的账号即可， 如此一来既可让系统执行无误，也能够让登入者的信息完全来自 NIS server ，比较单纯。
> 事实上， NIS 服务器写入的各项账号数据都在 NIS server 的 /var/yp/Makefile 那个档案设定的，可以进入该档案搜寻一下 UID 查看。
## NIS Client 所需软件与软件结构
NIS client 端所需要的软件仅有：
* ypbind
* yp-tools

yp-tools 是提供查询的软件，至于 ypbind 则是与 ypserv 互相沟通的客户端联机软件。另外，在 CentOS 当中我们还有很多配置文件是与认证有关的，包含 ypbind 的配置文件时， 在设定 NIS client 你可能需要动到底下的档案：
* /etc/sysconfig/network：就是 NIS 的领域名
* /etc/hosts：至少需要有各个 NIS 服务器的 IP 与主机名对应；
* /etc/yp.conf：这个则是 ypbind 的主要配置文件，里面主要设定 NIS 服务器所在
* /etc/sysconfig/authconfig：规范账号登入时的允许认证机制；
* /etc/pam.d/system-auth ：这个最容易忘记！因为账号通常由 PAM 模块所管理， 所以你必须要在 PAM 模块内加入 NIS 的支持才行！
* /etc/nsswitch.conf ：这个档案可以规范账号密码与相关信息的查询顺序，默认是先找 /etc/passwd 再找 NIS 数据库；

另外， NIS 还提供了几个有趣的程序给 NIS 客户端来进行账号相关参数的修改，例如密码、shell 等等， 主要有底下这几个指令：
* /usr/bin/yppasswd ：更改你在 NIS database (NIS Server 所制作的数据库) 的密码
* /usr/bin/ypchsh   ：同上，但是是更改 shell
* /usr/bin/ypchfn   ：同上，但是是更改一些用户的讯息
## NIS Client 的设定与启动
启动 NIS client 的设定，最主要是加入 NIS domain 当中，然后再启动 ypbind 即可。 虽然你可以手动去修改所有的配置文件，然而近期以来的 Linux distributions 账号处理机制越来越复杂，因此，这里建议你使用系统提供的工具来设定， 至于一些重要配置文件，最后有机会再去参考一下即可。  
CentOS 6.x 就利用 setup 这个指令即可。选择『Authentication configuration』的项目；选择 NIS 项目；最后再填写 NIS 网域 (Domain) 以及 NIS 服务器的 IP (Server)，按下确定即可。如果一直卡在如下画面
```
正在激活 rpcbind：                                         [  确定  ]
正在关闭 NIS 服务：                                        [  确定  ]
正在启动 NIS 服务：                                        [  确定  ]
正在绑定 NIS 服务：.......  <==这里一直卡住，没办法结束
```
代表你的 NIS client 没有办法连接上 NIS server，最常发生的就是服务器的防火墙忘记放行，或者是你客户端输入服务器 IP 时输入错误。 setup 修改的配置文件：
```
[root@clientlinux ~]# cat /etc/sysconfig/network
HOSTNAME=clientlinux.centos.vbird
NETWORKING=yes
GATEWAY=192.168.100.254
NISDOMAIN=vbirdnis    <==会主动的被建立起来

[root@clientlinux ~]# cat /etc/yp.conf
....(前面省略)....
domain vbirdnis server 192.168.100.254  <==主动建立

[root@clientlinux ~]# vim /etc/nsswitch.conf
passwd:     files nis
shadow:     files nis
group:      files nis
hosts:      files nis dns
# 上面几个项目是比较重要的，包括身份参数、密码、群组名、主机名与 IP 对应数据等。
# 你会看到，每个项目后面都会接着 nis ，所以 nis 有被支持
```
如果要手动处理的话，必须要手动的修改底下这些档案：
* /etc/sysconfig/network (加入 NISDOMAIN 项目)
* /etc/nsswitch.conf (修改许多主机验证功能的顺序)
* /etc/sysconfig/authconfig (CentOS 的认证机制)
* /etc/pam.d/system-auth (许多登入所需要的 PAM 认证过程)
* /etc/yp.conf (亦即是 ypbind 的配置文件)
## NIS Client 端的检验：yptest, ypwhich, ypcat
可以利用 id 这个指令直接检查 NIS server 有的，但是 NIS client 没有的账号，如果有出现该账号的相关 UID/GID 信息时，那表示数据传输也是正确的。 除此之外，我们还可以透过 NIS 提供的相关检验功能来检查。
### 利用 yptest 检验数据库之测试
```
[root@clientlinux ~]# yptest
Test 1: domainname
Configured domainname is "vbirdnis"

Test 2: ypbind
Used NIS server: www.centos.vbird

Test 3: yp_match
WARNING: No such key in map (Map passwd.byname, key nobody)
....(中间省略)....

Test 6: yp_master
www.centos.vbird

....(中间省略)....

Test 8: yp_maplist
passwd.byname
protocols.byname
hosts.byaddr
hosts.byname
....(中间省略)....

Test 9: yp_all
nisuser1 nisuser1:$1$U9Gccb60$K5lDQ.mGBw9x4oNEkM0Lz/:1001:1001::/home/nisuser1:/bin/bash
....(中间省略)....
1 tests failed
```
Test 3 出现的那个警告信息，只是说没有该数据库而已，是可以忽略的。重点在第 9 个步骤 yp_all 必须要有列出你 NIS server 上头的所有帐户信息，如果有出现账号相关数据的话，那么应该就算验证成功了。
### 利用 ypwhich 检验数据库数量
单纯使用 ypwhich 的时候显示的是『NIS Client 的 domain』名称，而当加入 -x 这个参数时， 则是显示『NIS Client 与 Server 之间沟通的数据库有哪些』
```
[root@clientlinux ~]# ypwhich -x
Use "hosts"     for map "hosts.byname"
Use "group"     for map "group.byname"
Use "passwd"    for map "passwd.byname"
....(以下省略)....
```
这些数据库档案则是放置在我的 NIS Server 的 /var/yp/vbirdnis/* 里面的。
### 利用 ypcat 读取数据库内容
```
[root@clientlinux ~]# ypcat [-h nisserver] [数据库名称]
选项与参数：
-h nisserver ：如果有设定的话，指向某一部特定的 NIS 服务器，
               如果没有指定的话，就以 ypbind 之设定为主；
数据库名称：亦即在 /var/yp/vbirdnis/ 内的档名。例如 passwd.byname

# 读出 passwd.byname 的数据库内容
[root@clientlinux ~]# ypcat passwd.byname
```
## 使用者参数修改：yppasswd, ypchfn, ypchsh
yppasswdd 可以接收 NIS client 端传来的密码修改，藉此而处理 NIS server 的 /etc/passwd, /etc/shadow ， 然后 yppasswdd 还能够重建密码数据库，让 NIS server 同步更新数据库。  
透过 yppasswd, ypchsh, ypchfn 来处理即可。这三个指令的对应是：
* yppasswd ：与 passwd 指令相同功能；
* ypchfn ：与 chfn 相同功能；
* ypchsh ：与 chsh 相同功能。
```
[root@clientlinux ~]# grep nisuser /etc/passwd  <==不会出现任何讯息，因为无此账号
[root@clientlinux ~]# su - nisuser1             <==直接切换身份看看！
su: warning: cannot change directory to /home/nisuser1: No such file or directory
-bash-4.1$ id
uid=1001(nisuser1) gid=1001(nisuser1) groups=1001(nisuser1)
# 因为我们 client.centos.vbird 仅有帐户信息，并没有用户家目录，
# 所以就会出现如上的警告，因此才需要用 id 验证，并且需要加挂 NFS 
# 仔细看，现在的身份确实是 nisuser1 ，确实有连上 NIS server

-bash-4.1$ yppasswd
Changing NIS account information for nisuser1 on www.centos.vbird.
Please enter old password:    <==这里输入旧密码
Changing NIS password for nisuser1 on www.centos.vbird.
Please enter new password:    <==这里输入新密码
Please retype new password:   <==再输入一遍

The NIS password has been changed on www.centos.vbird.

-bash-4.1$ exit
```
这样就更新了 NIS server 上头的 /etc/shadow 以及 /var/yp/vbirdnis/passwd.by* 的数据库。
```
[root@www ~]# ll /var/yp/vbirdnis/
-rw-------. 1 root root   13836 Jul 28 13:10 netid.byname
-rw-------. 1 root root   14562 Jul 28 13:29 passwd.byname
-rw-------. 1 root root   14490 Jul 28 13:29 passwd.byuid
-rw-------. 1 root root   28950 Jul 28 13:10 protocols.byname
# 密码档案被更动过，时间已经不一样了！再看看登录档

[root@www ~]# tail /var/log/messages
Jul 28 13:29:14 www rpc.yppasswdd[1707]: update nisuser1 (uid=1001) from host 
192.168.100.10 successful.
```
最终从登录档里面，我们也能够得到相关的记录。

# NIS 搭配 NFS 的设定在丛集计算机上的应用
### 什么是丛集计算机
超级计算机的结构中，主要是透过内部电路将好多颗 CPU 与内存连接在一块，因为是特殊设计，因此价格非常昂贵。 如果我们可以将较便宜的个人计算机串接在一块，然后将数值运算的任务分别丢给每一部串接在一块的个人计算机， 就是 PC cluster 最早的想法。  
因为每部计算机都需要运算相同的程序，而我们知道运算的数据都在内存当中， 而程序启动时需要给予一个身份，而程序读取的程序在每部计算机上面都需要是相同的！同时，每部计算机都需要支持平行化运算！ 所以，在 PC cluster 上面的所有计算机就得要有：
* 相同的用户帐户信息，包括账号、密码、家目录等等一大堆信息；
* 相同的文件系统，例如 /home, /var/spool/mail 以及数值程序放置的位置
* 可以搭配的平行化函式库，常见的有 MPICH, PVM...
### 另一个实例
* 账号：建立大于 2000 以上的账号，账号名称为 cluser1, cluser2, cluser3 (将 cluster user 缩写为 cluser，不是少写一个 t)，且这些账号的家目录预计放置于 /rhome 目录内，以与 NIS client 本地的用户分开；
* NIS 服务器：领域名为 vbirdcluster，服务器是 www.centos.vbird (192.168.100.254)，客户端是 clientlinux.centos.vbird (192.168.100.10)；
* NFS 服务器：服务器分享了 /rhome 给 192.168.100.0/24 这个网域，且预计将所有程序放置于 /cluster 目录中。 此外，假设所有客户端都是很干净的系统，因此不需要压缩客户端 root 的身份。
* NFS 客户端：将来自 server 的文件系统都挂载到相同目录名称底下
### NIS 实作阶段
```
# 1. 建立此次任务所需要的账号数据：
[root@www ~]# mkdir /rhome
[root@www ~]# useradd -u 2001 -d /rhome/cluser1 cluser1
[root@www ~]# useradd -u 2002 -d /rhome/cluser2 cluser2
[root@www ~]# useradd -u 2003 -d /rhome/cluser3 cluser3
[root@www ~]# echo password | passwd --stdin cluser1
[root@www ~]# echo password | passwd --stdin cluser2
[root@www ~]# echo password | passwd --stdin cluser3

# 2. 修改 NISDOMAIN 的名称
[root@www ~]# vim /etc/sysconfig/network
NISDOMAIN=vbirdcluster  <==重点在改这个项目

# 3. 制作数据库以及重新启动所需要的服务：
[root@www ~]# nisdomainname vbirdcluster
[root@www ~]# /etc/init.d/ypserv restart
[root@www ~]# /etc/init.d/yppasswdd restart
[root@www ~]# /usr/lib64/yp/ypinit -m
```
依序一个一个指令下达！上述的这四个指令稍微有相依性关系的！所以不要错乱了顺序，接下来换到客户端进行：
1. 以 setup 进行 NIS 的设定，在领域的部分请转为 vbirdcluster 才对！
2. 做完后再以 id cluser1 确认看看。
### NFS 服务器的设定
```
# 1. 设定 NFS 服务器开放的资源：
[root@www ~]# mkdir /cluster
[root@www ~]# vim /etc/exports
/rhome          192.168.100.0/24(rw,no_root_squash)
/cluster        192.168.100.0/24(rw,no_root_squash)

# 2. 重新启动 NFS ：
[root@www ~]# /etc/init.d/nfs restart
[root@www ~]# showmount -e localhost
Export list for localhost:
/rhome       192.168.100.0/24
/cluster     192.168.100.0/24

# 1. 设定 NIS Client 的 mount 数据！
[root@clientlinux ~]# mkdir /rhome /cluster
[root@clientlinux ~]# mount -t nfs 192.168.100.254:/rhome   /rhome
[root@clientlinux ~]# mount -t nfs 192.168.100.254:/cluster /cluster
# 如果上述两个指令没有问题，可以将他加入 /etc/rc.d/rc.local 当中

[root@clientlinux ~]# su - cluser1
[cluser1@clientlinux ~]$ 
```
最后你应该就能够在客户端以 cluser1 登入系统。就这么简单的将账号与文件系统同步做完了。