# 什么是 SAMBA
## SAMBA 的发展历史与名称的由来
### 让档案在两部主机之间直接修改：NFS 与 CIFS
Windows 系统的计算机可以透过桌面上『网上邻居』来分享别人所提供的档案数据。不过，NFS 仅能让 Unix 机器沟通， CIFS 只能让 Windows 机器沟通。有没有让 Windows 与 Unix-Like 这两个不同的平台相互分享档案数据的文件系统。
### 利用封包侦测逆向工程发展的 SMB Server
在 1991 年一个名叫 Andrew Tridgell 博士班研究生，自行写了个 program 去侦测当 DOS 与 DEC 的 Unix 系统在进行数据分享传送时所使用到的通讯协议信息，然后将这些重要的信息撷取下来， 并且基于上述所找到的通讯协议而开发出 Server Message Block (SMB) 这个文件系统，而就是这套 SMB 软件就能够让 Unix 与 DOS 互相的分享数据。
## SAMBA 常见的应用
SAMBA 最初发展的主要目就是要用来沟通 Windows 与 Unix Like 这两个不同的作业平台
* 分享档案与打印机服务；
* 可以提供用户登入 SAMBA 主机时的身份认证，以提供不同身份者的个别数据；
* 可以进行 Windows 网络上的主机名解析 (NetBIOS name)
* 可以进行装置的分享 (例如 Zip, CDROM...)
### 利用软件直接编修 WWW 主机上面的网页数据
相信很多人都是利用个人计算机将网页制作完毕之后，再以类似 FTP 之类的服务将网页上传到 WWW 主机的， 但这样有个困扰，那就是同时在客户端与 WWW 主机上头都有一份网页数据，常常会忘记哪一份是最新的， 最麻烦的是，有时候下载下来的档案已经经过好多修改了，却在下次的 FTP 作业，不小心又下载一次旧数据， 结果将已经修改过的数据覆盖过去。
### 做成可直接联机的文件服务器
### 打印机服务器
## SAMBA 使用的 NetBIOS 通讯协议
事实上，就像 NFS 是架构在 RPC Server 上面一样， SAMBA 这个文件系统是架构在 NetBIOS (Network Basic Input/Output System, NetBIOS) 这个通讯协议上面所开发出来的。  
最早 IBM 发展出 NetBIOS 的目的仅是要让局域网络内少数计算机进行网络链接的一个通讯协议而已， 所以考虑的角度并不是针对大型网络，因此，这个 NetBIOS 是无法跨路由的 (Router / Gateway)。微软的网络架构就使用了这个服务进行沟通的。SAMBA 最早发展的时候，其实是想要让 Linux 系统可以加入 Windows 的系统当中来分享使用彼此的档案数据的，所以 SAMBA 就架构在 NetBIOS 发展出来的。  
不过 NetBIOS 是无法跨路由的，因此使用 NetBIOS 发展起来的服务器理论上也是无法跨越路由的，好在，我们还有所谓的 NetBIOS over TCP/IP 的技术。  
## SAMBA 使用的 daemons
NetBIOS 当初发展时就着眼在局域网络内的快速数据交流，而因为是定义在局域网络内，因此他并没有使用类似 TCP/IP 之类的传输协议，也就不需要 IP 的设定。其实主机在 NetBIOS 协议当中的定义为使用『NetBIOS Name』，每一部主机必须要有不同的 NetBIOS Name 才行， 而档案数据就是在不同的 NetBIOS name 之间沟通。
1. 取得对方主机的 NetBIOS name 定位该主机所在：  
   当我们想要登入某部 Windows 主机使用他所提供的档案数据时，必需要加入该 Windows 主机的群组 (Workgroup)，并且我们的机器也必需要设定一个主机名，这个主机名跟 Hostname 是不一样的，因为这个主机名是架构在 NetBIOS 协议上的，我们可以简单的称呼他为 NetBIOS Name。在同一个群组当中，NetBIOS Name 必需要是独一无二的。
2. 利用对方给予权限存取可用资源：  
   在我们找到该主机名后，是否能登入该对方主机或者是取用对方主机所提供的资源， 还要看对方 Windows 主机有没有提供我们使用的权限。如果对方主机允许你登入， 但是却没有开放任何资源让你取用，登入主机也无法查看对方的硬盘里面的数据。

SAMBA 则是透过两支服务来控制这两个步骤，分别是：
* nmbd ：这个 daemon 是用来管理工作组、NetBIOS name 等等的解析。主要利用 UDP 协议开启 port 137, 138 来负责名称解析的任务；
* smbd ：这个 daemon 的主要功能就是用来管理 SAMBA 主机分享的目录、档案与打印机等等。 主要利用可靠的 TCP 协议来传输数据，开放的端口为 139 及 445(不一定存在) 。

所以，SAMBA 每次启动至少都需要有这两个 daemons 。而当我们启动了 SAMBA 之后，主机系统就会启动 137, 138 这两个 UDP 及 139 这一个 TCP 埠口。
## 联机模式的介绍 (peer/peer, domain model)
SAMBA 服务器的应用相当的广泛，而且可以依照不同的网域联机方式，与不同的用户账号密码的控管方式来进行分类。 例如最常见的 Workgroup 及 Domain 两种方式的联机模式。两种最常见的局域网络的联机模式： peer/peer (对等模式) 及 domain model (主控模式)。
### peer/peer (Workgroup model, 对等模式)：
假如在局域网络里面的所有 PC 均可以在自己的计算机上面管理自己的账号与密码， 同时每一部计算机也都具有独力执行各项软件的能力，只是藉由网络将各个 PC 链接在一起而已的一个架构， 所以，每一部机器都是可以独立运作的。  
这样的架构在目前小型办公室里面是最常见的。例如办公室里面有十个人，每个人桌上可能都安装有一套 Windows 操作系统的个人计算机，而这十部计算机都可以独立进行办公室软件的执行、独立上网、独立玩游戏等等的， 因为这十部计算机都可以独立运作，所以不会有一部计算机关掉，其他的计算机就无法工作的情况发生，这就是 peer/peer 的典型架构。  
使用 peer/peer 的架构的好处是每部计算机均可以独立运作，而不受他人的影响。不过， 缺点就是当整个网域内的所有人员都要进行数据分享时，需要知道所有计算机里面的账号与密码。所以， Peer/Peer 的架构是比较适合 (1)小型的网域，或者是 (2)没有需要常常进行档案数据分享的网络环境，或者是 (3)每个使用者都独自拥有该计算机的拥有权(就是说，该计算机是用户的，而不是公用的) 而，如果该单位的所有 PC 均是公有的 (例如学校的计算机教室环境)，而且你需要统一控管整个网域里面的账号与密码的话， 那就得使用底下的 domain models 了。
### domain model (主控模式)
既然使用计算机资源需要账号与密码， 那么我将所有的账号与密码都放置在一部主控计算机 (Primary Domain Controller, PDC) 上面，在我的网域里面，任何人想要使用任何计算机时，都需要在屏幕前方输入账号与密码，然后通通藉由 PDC 服务器的辨识后，才给予适当的权限。也就是说，不同的身份还具有不一样的计算机资源权限。  
PDC 服务器控管整个网域里面的各个机器 (PC A ~ PC D) 的账号与密码的信息，假如今天有个使用者账号名称为 vbird ，且密码为 12345 时，他不论使用哪一部计算机 (PC A ~ PC D) 只要在屏幕前方输入 vbird 与他的密码，则该机器会先到 PDC 上面查验是否有 vbird 以及 vbird 的密码，并且 PDC 主机会给予 vbird 这个用户相关的计算机资源权限。当 vbird 在任何一部主机上面登入成功后，他就可以使用相关的计算机资源了。  
这样的架构比较适合人来人往的企业架构，当系统管理员要控管新进人员的计算机资源使用权时，可以直接针对 PDC 来修改就好了，不需要每一部主机都去修修改改的，对于系统管理员来说，这样的架构在控管账号资源上，当然是比较简单的。

# SAMBA 服务器的基础设定
SAMBA 这个软件几乎在所有的 Linux distributions 上面都有提供。
## Samba 所需软件及其软件结构
以 3.x 版为例：
* samba： 这个软件主要提供了 SMB 服务器所需的各项服务程序 (smbd 及 nmbd)、 的文件档、以及其他与 SAMBA 相关的 logrotate 配置文件及开机默认选项档案等；
* samba-client： 这个软件则提供了当 Linux 做为 SAMBA Client 端时，所需要的工具指令，例如挂载 SAMBA 文件格式的 mount.cifs、 取得类似网芳相关树形图的 smbtree 等等；
* samba-common： 这个软件提供的则是服务器与客户端都会使用到的数据，包括 SAMBA 的主要配置文件 (smb.conf)、语法检验指令 (testparm) 等等；

与它相关的配置文件基本上有这些：
* /etc/samba/smb.conf： 这是 Samba 的主要配置文件，基本上，Samba 就仅有这个配置文件而已，且这个配置文件本身就是很详细的说明文件了，请用 vim 去查阅它。主要的设定项目分为服务器的相关设定 (global)，如工作组、NetBIOS 名称与密码等级等， 以及分享的目录等相关设定，如实际目录、分享资源名称与权限等等两大部分。
* /etc/samba/lmhosts： 早期的 NetBIOS name 需额外设定，因此需要这个 lmhosts 的 NetBIOS name 对应的 IP 档。 事实上它有点像是 /etc/hosts 的功能。只不过这个 lmhosts 对应的主机名是 NetBIOS name 。不要跟 /etc/hosts 搞混了。目前 Samba 预设会去使用你的本机名称 (hostname) 作为你的 NetBIOS name，因此这个档案不设定也无所谓。
* /etc/sysconfig/samba： 提供启动 smbd, nmbd 时，你还想要加入的相关服务参数。
* /etc/samba/smbusers： 由于 Windows 与 Linux 在管理员与访客的账号名称不一致，例如： administrator (windows) 及 root(linux)， 为了对应这两者之间的账号关系，可使用这个档案来设定
* /var/lib/samba/private/{passdb.tdb,secrets.tdb}： 管理 Samba 的用户账号/密码时，会用到的数据库档案；
* /usr/share/doc/samba-<版本>： 这个目录包含了 SAMBA 的所有相关的技术手册。也就是说，当你安装好了 SAMBA 之后，你的系统里面就已经含有相当丰富而完整的 SAMBA 使用手册了。

至于常用的脚本文件案方面，若分为服务器与客户端功能，则主要有底下这几个数据：
* /usr/sbin/{smbd,nmbd}：服务器功能，就是最重要的权限管理 (smbd) 以及 NetBIOS name 查询 (nmbd) 两个重要的服务程序；
* /usr/bin/{tdbdump,tdbtool}：服务器功能，在 Samba 3.0 以后的版本中，用户的账号与密码参数已经转为使用数据库了。Samba 使用的数据库名称为 TDB (Trivial DataBase)。 既然是使用数据库，当然要使用数据库的控制指令来处理。tdbdump 可以察看数据库的内容，tdbtool 则可以进入数据库操作接口直接手动修改帐密参数。不过，你得要安装 tdb-tools 这个软件才行；
* /usr/bin/smbstatus：服务器功能，可以列出目前 Samba 的联机状况， 包括每一条 Samba 联机的 PID, 分享的资源，使用的用户来源等等，让你轻松管理 Samba 。
* /usr/bin/{smbpasswd,pdbedit}：服务器功能，在管理 Samba 的用户账号密码时， 早期是使用 smbpasswd 这个指令，不过因为后来使用 TDB 数据库了，因此建议使用新的 pdbedit 指令来管理用户数据；
* /usr/bin/testparm：服务器功能，这个指令主要在检验配置文件 smb.conf 的语法正确与否，当你编辑过 smb.conf 时，请务必使用这个指令来检查一次，避免因为打字错误引起的困扰。
* /sbin/mount.cifs：客户端功能，在 Windows 上面我们可以设定『网络驱动器机』来连接到自己的主机上面。在 Linux 上面，我们则是透过 mount (mount.cifs) 来将远程主机分享的档案与目录挂载到自己的 Linux 主机上面。
* /usr/bin/smbclient：客户端功能，当你的 Linux 主机想要藉由『网络上的芳邻』的功能来查看别台计算机所分享出来的目录与装置时，就可以使用 smbclient 来查看。这个指令也可以使用在自己的 SAMBA 主机上面，用来查看是否设定成功。
* /usr/bin/nmblookup：客户端功能，有点类似 nslookup 。重点在查出 NetBIOS name 就是了。
* /usr/bin/smbtree：客户端功能，这玩意就有点像 Windows 系统的网络上的芳邻显示的结果，可以显示类似『靠近我的计算机』之类的数据， 能够查到工作组与计算机名称的树状目录分布图。
## 基础的网芳分享流程与 smb.conf 的常用设定项目
既然 Samba 是要加入 Windows 的网芳服务当中，所以它的设定方式应该是要与网芳差不多才是。后来在 Windows XP 的 SP2 (服务包第二版) 之后加入了很多的预设防火墙机制， 因此使用网芳的预设限制常常会是这样的：
* 服务器与客户端之间必须要在同一个网域当中 (否则需要修改 Windows 预设防火墙)；
* 最好设定为同一工作组；
* 主机的名称不可相同 (NetBIOS name)；
* 专业版 Windows XP 最多仅能提供同时 10 个用户联机到同一台网芳服务器上。

工作组与主机名的设定，你可以在『我的计算机』右键单击，选择内容后去修订相关的设定值。当你的 Windows 主机群符合上述的条件后，就很容易处理网芳分享的工作。分享的步骤一般是这样的：
1. 叫出档案总管，然后在要分享的目录、磁盘或装置 (如打印机) 上面按下右键，选择『共享』，然后就能够设定好分享的数据了；
2. 最好建立一组给用户使用的账号与密码，让其他主机的用户可以透过该账号密码联机进入使用网芳分享的资源；

Samba 的设定：
1. 服务器整体设定方面：在 smb.conf 当中设定好工作组、NetBIOS 主机名、密码使用状态 (无密码分享或本机密码) 等等；
2. 规划准备分享的目录参数：在 smb.conf 内设定好预计要分享的目录或装置以及可供使用的账号数据；
3. 建立所需要的文件系统：根据步骤 2 的设定，在 Linux 文件系统当中建立好分享出去的档案或装置，以及相关的权限参数；
4. 建立可用 Samba 的账号：根据步骤 2 的设定，建立所需的 Linux 实体账号，再以 pdbedit 建立使用 Samba 的密码；
5. 启动服务：启动 Samba 的 smbd, nmbd 服务，开始运转

根据上面的流程，其实我们最需要知道的就是 smb.conf 这个配置文件的信息。这个档案其实可以分为两部份来看， 一个是主机信息部分，在 smb.conf 当中以 [global] (全领域) 作为设定的依据；另一个则是分享的信息， 以个别的目录名称为依据。另外，由于 Samba 主要是想加入网芳功能，因此在 smb.conf 内的很多设定都与 Windows 类似：
* 在 smb.conf 当中，井字号与分号 (# 跟 ;) 都是批注符号；
* 在这个配置文件中，大小写是没关系的！因为 Windows 没分大小写！
## smb.conf 的服务器整体参数：[global] 项目
在 smb.conf 这个配置文件当中的设定项目有点像底下这样：
```
# 会有很多加上 # 或 ; 的批注说明，你也可以自行加上来提醒自己相关设定
[global]
   参数项目 = 设定内容
   ....

[分享资源名称]
   参数项目 = 设定内容
   ....
```
在 [global] 当中的就是一些服务器的整体参数了，包括工作组、主机的 NetBIOS 名称、字符编码的显示、登录文件的设定、 是否使用密码以及使用密码验证的机制等等，都是在这个 [global] 项目中设定的。至于 [分享资源名称] 则是针对你开放的目录来进权限方面的设定，包括谁可以浏览该目录、是否可以读写等等参数。 在 [global] 部分关于主机名信息方面的参数主要有：
* workgroup = 工作组的名称：注意，主机群要相同；
* netbios name = 主机的 NetBIOS 名称，每部主机均不同；
* server string = 主机的简易说明，这个随便写即可。

SAMBA 服务器上面的数据 (例如 mount 磁盘分区槽的参数以及原本的数据编码), SAMBA 服务器显示的语系, Windows 客户端显示的语系, Windows 客户端连上 SAMBA 的软件 都需要符合设定值才行。
* display charset = 自己服务器上面的显示编码， 例如你在终端机时所查阅的编码信息。一般来说，与底下的 unix charset 会相同。
* unix charset = 在 Linux 服务器上面所使用的编码，一般来说就是 i18n 的编码。所以你必须要参考 /etc/sysconfig/i18n 内的『默认』编码。
* dos charset = 就是 Windows 客户端的编码了！ 一般来说我们的繁体中文 Windows 使用的是 big5 编码，这个编码在 Samba 内的格式被称为『 cp950 』。

除此之外，还有登录文件方面的信息，包括这些参数：
* log file = 登录档放置的档案，文件名可能会使用变量处理；
* max log size = 登录档最大仅能到多少 Kbytes ，若大于该数字，则会被 rotate 掉。

还有网芳开放分享时，安全性程度有关的密码参数，包括这几个：
* security = share, user, domain：三选一，这三个设定值分别代表：
    * share：分享的数据不需要密码，大家均可使用 (没有安全性)；
    * user ：使用 SAMBA 服务器本身的密码数据库，密码数据库与底下的 passdb backend 有关；
    * domain：使用外部服务器的密码，亦即 SAMBA 是客户端之意，如果设定这个项目， 你还得要提供『password server = IP』的设定值才行；
* encrypt passwords = Yes 代表密码要加密，注意那个 passwords 要有 s
* passdb backend = 数据库格式，如前所述，为了加快速度， 目前密码文件已经转为使用数据库了。默认的数据库格式微 tdbsam ，而预设的档案则放置到 /var/lib/samba/private/passwd.tdb。
### 分享资源的相关参数设定 [分享的名称]
这个分享名称内常见的参数有：
* [分享名称] ：这个分享名称很重要，它是一个『代号』而已。记得回去看看 16.2.2 里面提到的那个范例；
* comment ：只是这个目录的说明而已！
* path ：这个分享名称实际会进入的 Linux 文件系统 (目录)。 也就是说，在网芳当中看到的是 [分享] 的名称，而实际操作的文件系统则是在 path 里头所设定的。
* browseable ：是否让所有的用户看到这个项目？
* writable ：是否可以写入？这里需要注意，那个 read only 与 writable 不是两个蛮相似的设定值吗？如果 writable 在这里设定为 yes ，亦即可以写入，如果 read only 同时设定为 yes ， 那不就互相抵触了！最后出现的那个设定值为主要的设定。
* create mode 与 directory mode 都与权限有关
* writelist = 使用者, @群组，这个项目可以指定能够进入到此资源的特定使用者。 如果是 @group 的格式，则加入该群组的使用者均可取得使用的权限，设定上会比较简单
### smb.conf 内的可用变量功能
为了简化设定值，Samba 提供很多不同的变量给我们来使用，主要有底下这几个变量：
* %S：取代目前的设定项目值，所谓的『设定项目值』就是在 [分享] 里面的内容！举例来说，例如底下的设定范例：
  ```
  [homes]
     valid users = %S
     ....
  ```
  因为 valid users 是允许的登入者，设定为 %S 表示任何可登入的使用者都能够登入的意思。如果 dmtsai 这个使用者登入之后，那个 [homes] 就会自动的变成了 [dmtsai] 了， %S 的用意就是在替换掉目前 [ ] 里面的内容
* %m：代表 Client 端的 NetBIOS 主机名
* %M：代表 Client 端的 Internet 主机名，就是 HOSTNAME。
* %L：代表 SAMBA 主机的 NetBIOS 主机名。
* %H：代表用户的家目录。
* %U：代表目前登入的使用者的使用者名称
* %g：代表登入的使用者的组名。
* %h：代表目前这部 SAMBA 主机的 HOSTNAME ，注意是 hostname 不是 NetBIOS name
* %I：代表 Client 的 IP 
* %T：代表目前的日期与时间

若有其他额外的参数须知，务必自行 man smb.conf 。
## 不需要密码的分享 (security = share, 纯测试)
### 0. 假设条件
服务器 (192.168.100.254) 预计设定的参数状况为：
* 在 LAN 内所有的网芳主机工作组 (workgroup) 为： vbirdhouse
* 这部 Samba 服务器的 NetBIOS 名称 (netbios name) 为： vbirdserver
* 使用者认证层级设定 (security) 为： share
* 取消原本有放行的 [homes] 目录；
* 仅分享 /tmp 这个目录而已，且取名为： temp
* Linux 服务器的编码格式假设为万国码 (Unicode, 亦即 utf8)
* 客户端为中文 Windows ，在客户端的软件也使用 big5 的编码
### 1. 设定 smb.conf 配置文件
由于我们有设定语系相关的数据，因此得要先查查看，到底我们 Linux 服务器的语系是否为 utf8 ：
```
[root@www ~]# cat /etc/sysconfig/i18n
LANG="zh_TW.UTF-8"  <==确实是出现了 utf8
```
在这个例子当中我们仅分享 /tmp 这个目录
```
[root@www ~]# cd /etc/samba
[root@www samba]# cp smb.conf smb.conf.raw  <==先备份再说
[root@www samba]# vim smb.conf
# 1. 先设定好服务器整体环境方面的参数
[global]
        # 与主机名有关的设定信息
        workgroup     = vbirdhouse
        netbios name  = vbirdserver
        server string = This is vbird's samba server

        # 与语系方面有关的设定项目，为何如此设定请参考前面的说明
        unix charset    = utf8
        display charset = utf8
        dos charset     = cp950

        # 与登录文件有关的设定项目，注意变量 (%m)
        log file = /var/log/samba/log.%m
        max log size = 50

        # 这里才是与密码有关的设定项目
        security = share

        # 修改一下打印机的加载方式，不要加载
        load printers	= no

# 2. 分享的资源设定方面：主要得将旧的批注，新的加入！
#    先取消 [homes], [printers] 的项目，然后针对 /tmp 的设定，可浏览且可写入
[temp]                                     <==分享资源名称
        comment    = Temporary file space  <==简单的解释此资源
        path       = /tmp                  <==实际 Linux 分享的目录
        writable   = yes                   <==是否可写入？在此例为是的
        browseable = yes                   <==能不能被浏览到资源名称
        guest ok   = yes                   <==单纯分享时，让用户随意登入的设定值
```
在原本的 smb.conf 上面就已经有很多默认值了，这些默认值如果你不知道他的用途， 尽量保留默认值，也可以使用 man smb.conf 去查询该默认值的意义。上述的设定是完全控制使用者的认证层级的。
### 2. 用 testparm 查阅 smb.conf 的语法设定正确性
在启动 samba 之前，我们务必要了解到 smb.conf 里面语法是否正确，检验的方式使用 testparm 这个指令即可。 测试方式如下：
```
[root@www ~]# testparm
选项与参数：
-v ：查阅完整的参数设定，连同默认值也会显示出来

[root@www ~]# testparm
Load smb config files from /etc/samba/smb.conf
Processing section "[temp]"   <==看有几个中括号，若中刮号前出现讯息，则有错误
Loaded services file OK.
Server role: ROLE_STANDALONE
Press enter to see a dump of your service definitions <==按 Enter 继续

[global]   <==底下就是刚刚在 smb.conf 里头设定的数据
        dos charset = cp950
        unix charset = utf8
        display charset = utf8
        workgroup = VBIRDHOUSE
        netbios name = VBIRDSERVER
        server string = This is vbird's samba server
        security = SHARE
        log file = /var/log/samba/log.%m
        max log size = 50
        load printers = No

[temp]
        comment = Temporary file space
        path = /tmp
        read only = No
        guest ok = Yes
```
上头是语法验证与各个项目的列出，如果你下达 testparm 却出现如下画面那就是有问题：
```
[root@www ~]# testparm
Load smb config files from /etc/samba/smb.conf
Unknown parameter encountered: "linux charset" <==中括号前为错误讯息
Ignoring unknown parameter "linux charset"
Processing section "[temp]"
Loaded services file OK.
Server role: ROLE_STANDALONE
Press enter to see a dump of your service definitions
```
如果你想要了解 samba 的所有设定 (包括没有在 smb.conf 里头设定的默认值)，可以使用 testparm -v 来作详细的输出， 资料相当的丰富，透过这个你也可以知道你的主机环境设定为何。
### 3. 服务器端的服务启动与埠口观察
利用预设的 CentOS 启动方式来处理即可：
```
[root@www ~]# /etc/init.d/smb start  <==这一版开始要启动两个daemon
[root@www ~]# /etc/init.d/nmb start
[root@www ~]# chkconfig smb on
[root@www ~]# chkconfig nmb on
[root@www ~]# netstat -tlunp | grep mbd
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address        Foreign Address State   PID/Program name
tcp        0      0 :::139               :::*            LISTEN  1772/smbd
tcp        0      0 :::445               :::*            LISTEN  1772/smbd
udp        0      0 192.168.1.100:137    0.0.0.0:*               1780/nmbd
udp        0      0 192.168.100.254:137  0.0.0.0:*               1780/nmbd
udp        0      0 0.0.0.0:137          0.0.0.0:*               1780/nmbd
udp        0      0 192.168.1.100:138    0.0.0.0:*               1780/nmbd
udp        0      0 192.168.100.254:138  0.0.0.0:*               1780/nmbd
udp        0      0 0.0.0.0:138          0.0.0.0:*               1780/nmbd
```
在 Samba 当中预设会启动多个端口，这包括数据传输的 TCP 端口 (139, 445)，以及进行 NetBIOS 名称解析之类工作的 UDP 埠口 (137, 138)，所以你才会看到很多数据的。透过 testparm -v 的观察，可以发现『 smb ports = 445 139 』这个设定值指定两个埠口的，因此你可以在 smb.conf 增加这个设定值，并改为 smb ports = 139 即可。不过，建议先保留默认值。
### 4. 假设自我为客户端的检验 (默认用 lo 接口)
我们可以在本机上透过 smbclient 这支程序来处理，它的基本查询语法是这样的：
```
[root@www ~]# smbclient -L [//主机或IP] [-U 使用者账号]
选项与参数：
-L ：仅查阅后面接的主机所提供分享的目录资源；
-U ：以后面接的这个账号来尝试取得该主机的可使用资源
```
由于在这个范例当中我们并没有规范用户的安全等级 (share)，所以不必使用 -U 这个选项，因此你可以这样看看：
```
[root@www ~]# smbclient -L //127.0.0.1 
Enter root's password: <==因为不需要密码，因此这里单击 [Enter] 
Domain=[VBIRDHOUSE] OS=[Unix] Server=[Samba 3.5.4-68.el6_0.2]

        Sharename       Type      Comment
        ---------       ----      -------
        temp            Disk      Temporary file space
        IPC$            IPC       IPC Service (This is vbird's samba server)
Domain=[VBIRDHOUSE] OS=[Unix] Server=[Samba 3.5.4-68.el6_0.2]

        Server               Comment
        ---------            -------
        VBIRDSERVER          This is vbird's samba server

        Workgroup            Master
        ---------            -------
        VBIRDHOUSE           VBIRDSERVER
```
上表输出的信息当中，分享的目录资源 (Sharename) 就是在 smb.conf 当中设定的 [temp] 名称。因此在这里的意思是：任何人都可以进入 //127.0.0.1/temp 这个目录当中， 而这个目录在 Linux 系统其实是 /tmp 目录。至于那个 IPC$ 则是为了要应付 Windows 环境所必须要存在的项目就是了。可以利用 mount 这个指令来测试看看：
```
[root@www ~]# mount -t cifs //127.0.0.1/temp /mnt
Password: <==因为没有密码，所以这里还是按 Enter 即可
[root@www ~]# df
Filesystem           1K-blocks      Used Available Use% Mounted on
....(前面省略)....
//127.0.0.1/temp/      1007896     53688    903008   6% /mnt

[root@www ~]# cd /mnt
[root@www mnt]# ll  <==以上这两个动作要进行！才会知道有没有权限的问题！
[root@www mnt]# touch zzz
[root@www mnt]# ll zzz /tmp/zzz
-rw-r--r--. 1 nobody nobody 0 Jul 29 13:08 /tmp/zzz
-rw-r--r--. 1 nobody nobody 0 Jul 29 13:08 zzz
# 注意，你进入 /mnt 身份会被压缩成为 nobody ，不再是 root 

[root@www mnt]# cd ; umount /mnt
```
## 需要账号密码才可登入的分享 (security = user)
可以透过 Samba 服务器提供的认证方式来进行用户权力的给予， 也就是说，你在客户端联机到服务器时，必须要输入正确的账号与密码后，才能够登入 Samba 查阅到你自己的数据。Samba 本身就提供一个小程序来帮助我们处理密码的建立了，整个流程还不太难。  
比较重要的是 Samba 使用者账号必须要存在于 Linux 系统当中 (/etc/passwd)， 但是 Samba 的密码与 Unix 的密码档案并不相同 (这是因为 Linux 与网芳的密码验证方式及编码格式不同所致)。
### 0. 假设条件
由于使用者层级会改变成 user 的阶段，因此 [temp] 已经没有必要存在！请将该设定删除或批注。 而服务器方面的整体数据则请保留，包括工作组等等的数据，并新增底下的资料：
* 使用者认证层级设定 (security) 为： user
* 用户密码档案使用 TDB 数据库格式，默认档案在 /var/lib/samba/private/ 内；
* 密码必须要加密；
* 每个可使用 samba 的使用者均拥有自己的家目录；
* 设定三个用户，名称为 smb1, smb2, smb3 ，且均加入 users 为次要群组。此三个用户 Linux 密码为 1234， Samba 密码则为 4321；
* 分享 /home/project 这个目录，且资源名称取名为： project；
* 加入 users 这个群组的使用者可以使用 //IP/project 资源，且在该目录下 users 这个群组的使用者具有写入的权限。
### 1. 设定 smb.conf 配置文件与目录权限相关之设定
在这个范例的配置文件当中，我们会新增几个参数：
```
# 1. 开始设定重要的 smb.conf 档案
[root@www ~]# vim /etc/samba/smb.conf
[global]
        workgroup       = vbirdhouse
        netbios name    = vbirdserver
        server string   = This is vbird's samba server
        unix charset    = utf8
        display charset = utf8
        dos charset     = cp950
        log file        = /var/log/samba/log.%m
        max log size    = 50
        load printers	= no

        # 与密码有关的设定项目，包括密码档案所在格式
        security = user          <==这行就是重点，改成 user 层级
        passdb backend = tdbsam  <==使用的是 TDB 数据库格式

# 2. 分享的资源设定方面：删除 temp  加入 homes 与 project
[homes]                                   <==分享的资源名称
        comment        = Home Directories
        browseable     = no               <==除了使用者自己外，不可被其他人浏览
        writable       = yes              <==挂载后可擦写此分享
        create mode    = 0664             <==建立档案的权限为 664
        directory mode = 0775             <==建立目录的权限为 775

[project]                                 <==就是那三位使用者的共享资源
        comment    = smbuser's project
        path       = /home/project        <==实际的 Linux 上面的目录位置
        browseable = yes                  <==可被其他人所浏览到资源名称(非内容)
        writable   = yes                  <==可以被写入
        write list = @users               <==写入者有哪些人的意思

# 2. 每次改完 smb.conf 你都需要重新检查一下语法正确否
[root@www ~]# testparm  <==详细的 debug 请自行处理
```
* [global] 修改与新增的部分：security 设定为 user 层级，且使用『passdb backend = tdbsam』这个数据库格式，因此密码文件会放置于 /var/lib/samba/private/ 内。 此外，默认密码就是加密的，因此不需要额外使用其他的设定参数来规范；
* [homes] 这个使用者资源共享部分： homes 是最特殊的资源共享名称，因为 Linux 上面的每位用户均有家目录，例如 smb1 的家目录位于 /home/smb1/ ，那当 smb1 用户使用 samba 时，她就会发现多了个 //127.0.0.1/smb1/ 的资源可用，而 smb2 就在 //127.0.0.1/smb2/ 这个资源。由于不可浏览 (browseable)，所以除了使用者可以看到自己的家目录资源外，其他人是无法浏览的。此外，为了规范权限，而多了 create mode 与 directory mode 两个设定值 (此值可设定也可不理会)；
* [project] 这个使用者资源共享部分：当我们新增一个共享资源时， 最重要的就是规范资源名称。在此例中我们使用 project 这个资源名称来指向 /home/project ，也就是说， //127.0.0.1/project 代表的是 /home/project 的意思。此外，能够使用这个资源的账号，为加入 users 这个群组的用户。 透过 write list 这个项目比较单纯，如果是早期的设定，可能会使用 valid users ，但近来鸟哥比较偏好 write list 设定项目。 不过能否顺利的存取档案还与 Linux 最底层的档案权限有关。
### 2. 设定可使用 Samba 的用户账号与密码
设定使用者账号是很重要的一环，因为设定错误的话，当然也就任何人都没有办法登入的，Linux 的文件系统与 SAMBA 设定的使用者登入权限的相关性：
* 在 Linux 这个系统下，任何程序都需要取得 UID 与 GID (User ID 与 Group ID) 的身份之后，才能够拥有该身份的权限，也才能够适当的进行存取档案等动作
* 关于 Linux 这个系统的 UID 与 GID 与账号的相对关系，一般记录在 /etc/passwd 当中，当然也能透过 NIS, ldap 等方式来取对应；
* SAMBA 仅只是 Linux 底下的一套软件，使用 SAMBA 来进行 Linux 文件系统时，还是需要以 Linux 系统下的 UID 与 GID 为准则

我们需要透过 SAMBA 所提供的功能来进行 Linux 的存取，而 Linux 的存取是需要取得 Linux 系统上面的 UID 与 GID 的，因此，我们登入 SAMBA 服务器时，所利用 SAMBA 取得的其实是 Linux 系统里面的相关账号。这也就是说，在 SAMBA 上面的使用者账号，必须要是 Linux 账号中的一个。  
所以说，在不考虑 NIS 或 LDAP 等其他账号的验证方式，单纯以 Linux 本机账号 (/etc/passwd) 作为身份验证时， 在 Samba 服务器所提供可登入的账号名称，必须要存在于 /etc/passwd 当中，这是一个很重要的概念。  
现在我们知道需要新增 smb1, smb2, smb3 三个用户，且这三个用户需要加入 users 群组。此外，我们之前还建立过 student 这个用户，假设这四个人都需要能用 Samba 服务，那么除了新增用户之外，我们还需要利用 pdbedit 这个指令来处理 Samba 用户功能：
```
# 1. 先来建立所需要的各个账号，但假设 student 已经存在了
[root@www ~]# useradd -G users smb1
[root@www ~]# useradd -G users smb2
[root@www ~]# useradd -G users smb3
[root@www ~]# echo 1234 | passwd --stdin smb1
[root@www ~]# echo 1234 | passwd --stdin smb2
[root@www ~]# echo 1234 | passwd --stdin smb3

# 2. 使用 pdbedit 指令功能
[root@www ~]# pdbedit -L [-vw]            <==单纯的察看帐户信息
[root@www ~]# pdbedit -a|-r|-x -u 账号    <==新增/修改/删除账号
[root@www ~]# pdbedit -a -m -u 机器账号   <==与 PDC 有关的机器码
选项与参数：
-L ：列出目前在数据库当中的账号与 UID 等相关信息；
-v ：需要搭配 -L 来执行，可列出更多的讯息，包括家目录等数据；
-w ：需要搭配 -L 来执行，使用旧版的 smbpasswd 格式来显示数据；
-a ：新增一个可使用 Samba 的账号，后面的账号需要在 /etc/passwd 内存在者；
-r ：修改一个账号的相关信息，需搭配很多特殊参数，请 man pdbedit；
-x ：删除一个可使用 Samba 的账号，可先用 -L 找到账号后再删除；
-m ：后面接的是机器的代码 (machine account)，与 domain model 有关！

# 2.1 开始新增使用者
[root@www ~]# pdbedit -a -u smb1
new password: <==
retype new password: <==再输入一次
Unix username:        smb1   <==底下为输入正确后的显示结果
NT username:
Account Flags:        [U          ]
User SID:             S-1-5-21-4073076488-3046109240-798551845-1000
Primary Group SID:    S-1-5-21-4073076488-3046109240-798551845-513
Full Name:
Home Directory:       \\vbirdserver\smb1
HomeDir Drive:
Logon Script:
Profile Path:         \\vbirdserver\smb1\profile
Domain:               VBIRDSERVER
Account desc:
Workstations:
Munged dial:
Logon time:           0
Logoff time:          9223372036854775807 seconds since the Epoch
Kickoff time:         9223372036854775807 seconds since the Epoch
Password last set:    Fri, 29 Jul 2011 13:19:56 CST
Password can change:  Fri, 29 Jul 2011 13:19:56 CST
Password must change: never
Last bad password   : 0
Bad password count  : 0
Logon hours         : FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF
# 你可以发现其实讯息非常的多！若需修改细部设定，请 man pdbedit 
[root@www ~]# pdbedit -a -u smb2
[root@www ~]# pdbedit -a -u smb3
[root@www ~]# pdbedit -a -u student

# 2.2 查询目前已经存在的 Samba 账号
[root@www ~]# pdbedit -L
smb1:2004:
smb3:2006:
smb2:2005:
student:505:
# 仅会列出账号与 UID 而已

# 2.3 尝试修改与删除 smb3 这个账号看看
[root@www ~]# smbpasswd smb3
New SMB password: 
Retype new SMB password:
# 修改密码比较特殊，管理密码参数是使用 pdbedit，修改密码得要用 smbpasswd 

[root@www ~]# pdbedit -x -u smb3
[root@www ~]# pdbedit -Lw
# 此时你就看不到 smb3 这个用户了，所以测试完请立即将它加回来！
```
以后如果有需要新增额外的使用者账号，若该账号原本不存在，则使用 useradd 再以 pdbedit -a 去新增。 若已经存在于 Linux 的实体账号，直接用 pdbedit -a 新增即可。同时要注意，管理 TDB 数据库格式建议使用 pdbedit ，smbpasswd 仅剩下修改密码的功能。
### 3. 重新启动 Samba 并进行自我测试
```
[root@www ~]# /etc/init.d/smb restart
[root@www ~]# /etc/init.d/nmb restart

# 1. 先用匿名登录试看看！
[root@www ~]# smbclient -L //127.0.0.1
Enter root's password:      <==直接按下 [Enter] 即可。
Anonymous login successful  <==有看到匿名的字样了！
Domain=[VBIRDHOUSE] OS=[Unix] Server=[Samba 3.5.4-68.el6_0.2]

        Sharename       Type      Comment
        ---------       ----      -------
        project         Disk      smbuser's project
        IPC$            IPC       IPC Service (This is vbird's samba server)
....(底下省略)....

# 2. 再使用 smb1 这个账号登入试看看！
[root@www ~]# smbclient -L //127.0.0.1 -U smb1
Enter smb1's password:  <==输入 smb1 在 pdbedit 所建立的密码！
Domain=[VBIRDHOUSE] OS=[Unix] Server=[Samba 3.5.4-68.el6_0.2]

        Sharename       Type      Comment
        ---------       ----      -------
        project         Disk      smbuser's project
        IPC$            IPC       IPC Service (This is vbird's samba server)
        smb1            Disk      Home Directories <==多了这玩意儿！
....(底下省略)....
```
由不同的身份登入可以取得不一样的浏览数据， 所以在使用上面需要特别留意，接下来，让我们开始来自我挂载测试：
```
[root@www ~]# mount -t cifs //127.0.0.1/smb1 /mnt -o username=smb1
Password: <==确定是输入正确的密码
# 此时 /home/smb1/ 与 /mnt 应该拥有相同的档名才对！因为挂载

[root@www ~]# ll /home/smb1/.bashrc
-rw-r--r--. 1 smb1 smb1 124 May 30 23:46 /home/smb1/.bashrc <==确定有档案
[root@www ~]# ls -a /mnt
# 却看不到任何东西！应该是 SELinux 的问题吧！根据 /var/log/messages 的讯息，
# 进行如下的动作就能够处理好这个程序！

[root@www ~]# setsebool -P samba_enable_home_dirs=1
[root@www ~]# ls -a /mnt
.  ..  .bash_logout  .bash_profile  .bashrc  .gnome2  .mozilla
# 这个使用者挂载处理完毕

[root@www ~]# umount /mnt
```
自我测试是非常重要的，因为 Samba 是会对外提供服务的，因此 SELinux 影响这个服务。包括默认用户家目录不会有开放的权限、预设的 SELinux type 不对就无法使用。
### 4. 关于权限的再说明与累加其他分享资源的方式：
需要注意 Linux 自身文件系统的权限。以及 SELinux ，这部份得要用 setsebool 与 chcon 或 restorecon 等指令来克服。另外就是，我们在 smb.conf 当中设定 [project] 为可写入，亦即 /home/project 是可写入的。假设 smb1 属于 users 这个群组，因此以 smb1 登入 SAMBA 服务器后，对于 /home/project 应该是具有可以读写的能力的。但是，如果你以 root 的身份建立 /home/project 却又忘记修改权限的话， 此时 /home/project 是无法让 users 这个群组写入的，因此 smb1 这个使用者当然不具有写入的能力。  
那如果你还要扩充分享的目录与能够登入的使用者时，可以这样做：
* 利用编辑 smb.conf 来开放其他的目录资源，并且特别注意 Linux 在该目录下的权限，请使用 chown 与 chmod 
* 利用 pdbedit 来新增其他可用 Samba 的账号，如果该账号并没有出现在 /etc/passwd 里面，请先以 useradd 新增该账号；
* 不论进行完任何的设定，请先以 testparm 进行确认，之后以 /etc/init.d/{smb,nmb} restart 来重新启动
## 设定成为打印机服务器 (CUPS 系统)
### 0. 假设条件
* CUPS 连接到 USB 打印机，并且开放非本机的 IP 来源使用此打印机；
* 使用 CUPS 内建的打印机驱动程序；
* 前往 HP 打印机官网取得 Windows 操作系统的驱动程序；
### 1. 安装打印机与确定打印机的联机正常
如果你的打印机端口为使用 USB 或者是平行串行端口的话，那么当你连接上打印机后， 可以利用底下的方式测试看看是否成功的连接上了：
```
[root@www ~]# lsusb
Bus 001 Device 002: ID 03f0:3817 Hewlett-Packard LaserJet P2015 series
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
[root@www ~]# ll /dev/usb/lp0
crw-rw----. 1 root lp 180, 0 Jul 29 13:55 /dev/usb/lp0
# 看得出来，已经有个 lp0 的打印机。测试打印一下。

[root@www ~]# echo "Hello printer" > /dev/usb/lp0
```
### 2. 设定 CUPS 与打印机的联机
CUPS 的设定原则是这样的：
* 我需要让 192.168.100.0/24 这个网域可以使用打印机
* 我需要让 192.168.100.0/24 及 127.0.0.0/8 可以管理 CUPS 系统
```
[root@www ~]# yum groupinstall "Print Server"
[root@www ~]# vim /etc/cups/cupsd.conf
# 1. 让监听的接口开放在所有接口！
# Listen localhost:631  <==约在第 18 行左右，改成如下：
Listen 0.0.0.0:631

# 2. 让内部网域能够进行 CUPS 的浏览与管控
<Location />        <==约在 32 行左右，新增能够让内网其他 IP 浏览者
  Order allow,deny
  Allow From 127.0.0.0/8
  Allow From 192.168.100.0/24
</Location>

<Location /admin>       <==约在 39 行左右，新增能够管理 CUPS 者
  Encryption Required   <==因为这里的关系，所以可能会用 https://IP 
  Order allow,deny
  Allow From 127.0.0.0/8
  Allow From 192.168.100.0/24
</Location>

# 设置完毕，开始启动 cups 系统
[root@www ~]# /etc/init.d/cups start
[root@www ~]# chkconfig cups on
[root@www ~]# netstat -tunlp | grep 'cups'
tcp     0  0  0.0.0.0:631         0.0.0.0:*      LISTEN      1851/cupsd
udp     0  0  0.0.0.0:631         0.0.0.0:*                  1851/cupsd
```
631 的埠口就是 CUPS 所启动的，要注意的是，开放界面得要给 0.0.0.0 才对。CUPS 支持很多不同的打印机端口，每种端口都不一样，常见的有：
* USB 端口： usb:/dev/usb/lp0
* 网络打印机： ipp://ip/打印机型号
* 网络芳邻打印机： smb://user:password@host/printer

这部打印机在网络的网址为：
* http://服务器IP:631/printers/打印机队列名称
* http://192.168.100.254:631/printers/HP_LaserJet_P2015_Series
### 3. 在 smb.conf 当中加入打印机的支持 (Optional)
```
[root@www ~]# vim /etc/samba/smb.conf
[global]
        # 得要修改 load printers 的设定，然后新增几个数据
        load printers = yes
        cups options  = raw       <==可支持来自 Windows 用户的打印作业
        printcap name = cups
        printing      = cups      <==与上面这两个在告知使用 CUPS 打印系统

[printers]                        <==打印机一定要写 printers
        comment = All Printers
        path    = /var/spool/samba<==预设把来自 samba 的打印作业暂时放置的队列
        browseable = no           <==不被外人所浏览，有权限才可浏览
        guest ok   = no           <==与底下两个都不许访客来源与写入(非文件系统)
        writable   = no
        printable  = yes          <==允许打印很重要的一项工作

[root@www ~]# testparm  <==若有错误，请自行处理一下
[root@www ~]# /etc/init.d/smb restart
[root@www ~]# /etc/init.d/nmb restart
```
基本上透过这样的设定你的 Samba 就能够顺利的提供打印机的服务了，不过，Windows 客户端依旧得要安装打印机的驱动程序才能够使用 Samba 所提供的打印机，而 Samba 3.x 可以让 Samba 主动的提供驱动程序给使用者，这样一来客户端就不需要额外去找驱动程序了。
### 4. 让 Samba 主动提供驱动程序给 Windows 用户使用
CUPS 主要是透过利用 Postscript 的打印语言与打印机沟通的，因此客户端只要取得 postscript 的驱动程序就可以使用 Samba 服务器所提供的打印机了。CUPS 官网本身就有提供 CUPS 的 Postscript 驱动程序。
```
[root@www ~]# ll /usr/share/cups/drivers
-rw-r--r-- 1 root root    803  4月 20  2006 cups6.inf
-rw-r--r-- 1 root root     72  4月 20  2006 cups6.ini
-rw-r--r-- 1 root root  12568  4月 20  2006 cupsps6.dll
-rw-r--r-- 1 root root  13672  4月 20  2006 cupsui6.dll  <==上面为 cups 提供
-rw-r--r-- 1 root root 129024  3月 24 13:29 ps5ui.dll    <==底下为 Win XP 提供
-rw-r--r-- 1 root root 455168  3月 24 13:29 pscript5.dll
-rw-r--r-- 1 root root  27568  3月 24 13:29 pscript.hlp
-rw-r--r-- 1 root root 792644  3月 24 13:29 pscript.ntf
```
接下来我们必须要在 smb.conf 里面增加一笔新的分享数据，这个分享数据必须是 [print$] 名称才行：
```
[root@www ~]# vim /etc/samba/smb.conf
[global]
....(设定保留原本数据)....
[homes]
....(设定保留原本数据)....
[printers]
....(设定保留原本数据)....
[print$]
        comment    = Printer drivers
        path       = /etc/samba/drivers  <==存放打印机驱动程序的目录
        browseable = yes
        guest ok   = no
        read only  = yes
        write list = root                <==这个驱动程序的管理员
[project]
....(设定保留原本数据)....

[root@www ~]# mkdir /etc/samba/drivers
[root@www ~]# chcon -t samba_share_t /etc/samba/drivers
# 由于预设的 CUPS 仅有 root 能管理，因此我们以 root 作为打印机管理员；
# 同时 SELinux 的类型也要修订如上的方式！那 root 就得要加入 samba 的支持才行：
[root@www ~]# pdbedit -a -u root

[root@www ~]# testparm                 <==测试语法
[root@www ~]# /etc/init.d/smb restart  <==重新启动

[root@www ~]# smbclient -L //127.0.0.1 -U root
Enter root's password:  <==输入 root 在 samba 的密码先
Domain=[VBIRDHOUSE] OS=[Unix] Server=[Samba 3.5.4-68.el6_0.2]

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer drivers
        project         Disk      smbuser's project
        HP_LaserJet_P2015_Series Printer   HP LaserJet P2015 Series
        IPC$            IPC       IPC Service (This is vbird's samba server)
        root            Disk      Home Directories
# 看到一部打印机以及驱动程序所在的分享数据
```
现在要告知 Samba ， CUPS 可提供 Windows 客户端的驱动程序，所以用户不需要自行设定他们的驱动程序。要由 cups 告知 Samba 是由 cupsaddsmb 这个指令：
```
[root@www ~]# cupsaddsmb [-H SAMBA 服务器名] [-h CUPS 服务器名] \
>   -a -v [-U 使用者账号]
选项与参数：
-H ：后续接的是 Samba 服务器名，本机的话可以直接用 localhost 即可；
-h ：后续接的为 CUPS 的服务器名，同样的可使用 localhost 即可；
-a ：自动搜寻出所有可用的 CUPS 打印机；
-v ：列出更多的信息；
-U ：打印机管理员

# 利用前面的说明将打印机驱动程序挂上 SAMBA (注意 CUPS 管理员预设是 root)
[root@www ~]# cupsaddsmb -H localhost -U root -a -v
Password for root required to access localhost via SAMBA: <==root 在 SAMBA 密码
# 这里会闪过很多的讯息，说明已经安装了某些信息，底下仅列出简单的讯息而已。
Running command: smbclient //localhost/print$ -N -A /tmp/cupsbrdBaE -c 'mkdir 
W32X86;put /tmp/cupsu13OSU W32X86/HP_LaserJet_P2015_Series.ppd;...

[root@www ~]# ll /etc/samba/drivers
drwxr-xr-x. 3 root root 4096 Jul 29 15:15 W32X86  <==这就是驱动程序目录
```
最后在驱动程序的存放目录会多出一个 W32X86 的目录，就是预计要给客户端使用的驱动程序。为了将所有的数据通通驱动， 建议你将 CUPS 及 SAMBA 重新启动：
```
[root@www ~]# /etc/init.d/cups restart
[root@www ~]# /etc/init.d/smb restart
[root@www ~]# /etc/init.d/nmb restart
```
### 5. 一些问题的克服
```
# 1. 列出所有可用的打印机状态
[root@www ~]# lpstat -a
HP_LaserJet_P2015_Series accepting requests since Fri 29 Jul 2011 02:55:28 PM CST

# 2. 查询目前默认打印机的的工作情况
[root@www ~]# lpq
hpljp2015dn 已就绪
没有项目
# 列出打印机的工作，若有打印作业存在时 (例如关掉打印机再印测试页)，会如下所示：
hpljp2015dn 已就绪并正在打印
等级    拥有人  工作    档案                            总计  大小
active  root    2       Test Page                       17408 byte

# 3. 删除所有的工作项目
[root@www ~]# lprm -
# 加上那个减号 (-) 代表移除所有等待中的打印作业
```
## 安全性的议题与管理
使用 SAMBA 其实是有一定程度的危险性的，这是因为很多网络攻击的蠕虫、病毒、木马就是透过网芳来攻击的。为了抵挡不必要的联机，所以 CentOS 5.x 预设的 SELinux 已经关闭了很多 Samba 联机的功能， 因此预设情况下，很多客户端的挂载可能会有问题。此外，仅开放有权限的网域来源，以及透过 smb.conf 来管理特定的权限，也是很重要的。同时，Linux 文件系统的 r, w, x 权限也是需要注意的。
### SELinux 的相关议题：
基本的 Samba 规则主要有：
```
[root@www ~]# getsebool -a | grep samba
samba_domain_controller --> off  <==PDC 时可能会用到
samba_enable_home_dirs --> off   <==开放用户使用家目录
samba_export_all_ro --> off      <==允许只读文件系统的功能
samba_export_all_rw --> off      <==允许读写文件系统的功能
samba_share_fusefs --> off
samba_share_nfs --> off
use_samba_home_dirs --> off      <==类似用户家目录的开放！
virt_use_samba --> off
```
几乎所有的规则默认都是关闭的，目前我们仅会用到用户的家目录以及分享成为可擦写， 不过似乎仅要 samba_enable_home_dirs 那个项目设定妥当即可。可以这样做：
```
[root@www ~]# setsebool -P samba_enable_home_dirs=1
[root@www ~]# getsebool -a | grep samba_enable_home
samba_enable_home_dirs --> on
```
这样用户挂载他们的家目录时 (例如 smb1 使用 //127.0.0.1/smb1/) 就不会出现无法挂载的问题了。此外， 由于分享成为 Samba 的目录还需要有 samba_share_t 的类型。我们还有分享 /home/project 也需要修订。这样做：
```
[root@www ~]# ll -Zd /home/project
drwxrws---. root users unconfined_u:object_r:home_root_t:s0 /home/project

[root@www ~]# chcon -t samba_share_t /home/project
[root@www ~]# ll -Zd /home/project
drwxrws---. root users unconfined_u:object_r:samba_share_t:s0 /home/project
```
如果你分享的目录不只是 Samba ，还包括 FTP 或者是其他的服务时，那可能就得要使用 public_content_t 这个大家都能够读取的类型才行。若你还有发现任何 SELinux 的问题，请依照 /var/log/messages 里面的信息去修订。
### 防火墙议题：利用 iptables 来管理
在 iptables.allow 规则中要加入：
```
[root@www ~]# vim /usr/local/virus/iptables/iptables.allow
# 加入底下这几行！
iptables -A INPUT -i $EXTIF -p tcp -s 192.168.100.0/24 -m multiport \
         --dport 139,445 -j ACCEPT
iptables -A INPUT -i $EXTIF -p tcp -s 192.168.1.0/24 -m multiport \
         --dport 139,445 -j ACCEPT
iptables -A INPUT -i $EXTIF -p udp -s 192.168.100.0/24 -m multiport \
         --dport 137,138 -j ACCEPT
iptables -A INPUT -i $EXTIF -p udp -s 192.168.1.0/24 -m multiport \
         --dport 137,138 -j ACCEPT
[root@www ~]# /usr/local/virus/iptables/iptables.rule
```
由于 smbd 及 nmbd 并不支持 TCP Wrappers ，只能透过 iptables 来控制。
### 防火墙议题：透过内建的 Samba 设定 (smb.conf)
事实上 Samba 已经有许多防火墙机制，就是在 smb.conf 内的 hosts allow 及 hosts deny 这两个参数。通常我们只要使用 hosts allow 即可，那么没有写入这个设定项目的其他来源就会被拒绝联机。这是比较严格的设定。举例来说，如果你只想要让本机、192.168.100.254, 192.168.100.10, 192.168.1.0/24 使用 SAMBA 而已，那么可以这样写：
```
[root@www ~]# vim /etc/samba/smb.conf
[global]
        # 跟防火墙的议题有关的设定
        hosts allow = 127. 192.168.100.254 192.168.100.10 192.168.1.
[homes]
....保留原始设定....
[root@www ~]# testparm
[root@www ~]# /etc/init.d/smb restart
```
这个设定值的内容支持部分比对，因此 192.168.1.0/24 只要写出前面三个 IP 段即可 (192.168.1.)。 如此一来不但只有数部主机可以登入我们的 SAMBA 服务器，而且设定值又简单。建议在防火墙议题方面，只要使用 iptables 或 hosts allow 其中一项即可。
### 文件系统议题：利用 Quota 限制用户磁盘使用
```
[root@www ~]# edquota -u smb1
Disk quotas for user smb1 (uid 2004):
  Filesystem                blocks    soft    hard inodes  soft  hard
  /dev/mapper/server-myhome       0 300000  400000      0     0     0

[root@www ~]# edquota -p smb1 smb2
[root@www ~]# edquota -p smb1 smb3
[root@www ~]# repquota -ua
*** Report for user quotas on device /dev/mapper/server-myhome
Block grace time: 7days; Inode grace time: 7days
                        Block limits                File limits
User            used    soft    hard  grace    used  soft  hard  grace
----------------------------------------------------------------------
smb1      --      32  300000  400000              9     0     0
smb2      --      32  300000  400000              8     0     0
smb3      --      32  300000  400000              8     0     0
```
## 主机安装时的规划与中文扇区挂载
* 在安装 Linux 的时候，建议不需要安装 X Window ；
* 在规划 Linux 时，/home 最好独立出一个 partition ，而且硬盘空间最好能够大一些；
* /home 独立出来的 partition 可以单独进行 quota 的作业，以规范用户的最大硬盘用量；
* 无网卡的打印机 (USB) 可直接链接到 Linux 主机再透过 Samba 分享；
* 由于 SAMBA 一般来说都仅针对内部 (LAN) 主机进行开放，所以，可能的话 SAMBA 主机直接使用 private IP 来设定即可；
* 如果你的 SAMBA 主机使用 Public IP 时，请特别留意规范好防火墙的设定，尽量仅让 LAN 内的计算机可以联机进来即可，不要对 Internet 开放。

另外，如果你的 Samba 服务器需要挂载含有中文的 partition 时，譬如说你将原本 Windows XP 的 FAT32 文件系统挪到 Linux 系统下，此时如果用一般模式来挂载该分割槽时，一些中文档名可能会无法被顺利的显示出来。 这个时候你就得需要这样做了：
```
mount -t vfat -o iocharset=big5,codepage=950 /dev/sd[a-p][1-15] /mount/point
```
其中 iocharset 指的是本机的语系编码方式， codepage 则与远程软件有关。因为我们是在本机进行挂载， 所以实际上使用 iocharset 这个参数即可。

# Samba 客户端软件功能
假设局域网络内有 Windows/Linux 系统，这两种系统都是透过 NetBIOS over TCP/IP 来连上 Samba 服务器的， 在设定之前你必须要知道的有几件事：
* 在区网内的主机最好具有相同的工作组，且具有不同的主机名；
* Windows XP pro. 最多仅能允许十个用户同时连接到自己的网芳；
* 你可以在网芳当中看到的通常是相同群组的主机；
* 可以使用『搜寻』-->『计算机』-->『输入 IP』来查到 Samba 主机；
* Windows 的网芳预设仅有同一 IP 网段的主机才能登入 (Windows 防火墙设定)！
## Windows 系统的使用
### 让 Windows 系统的网芳支持不同网域的 IP 联机
调整防火墙设置
### 透过 port 445 的特殊登入方式
## Linux 系统的使用
### smbclient：查询网芳分享的资源，以及使用类似 FTP 的方式上传/下载网芳
主要是透过 smbclient 来观察，再以 mount 来挂载文件系统。
```
# 1. 关于查询的功能，例如查出 192.168.100.254 的网芳数据
[root@clientlinux ~]# smbclient -L //[IP|hostname] [-U username]
[root@clientlinux ~]# smbclient -L //192.168.100.254 -U smb1
Enter smb1's password:
Domain=[VBIRDHOUSE] OS=[Unix] Server=[Samba 3.5.4-68.el6_0.2]

        Sharename       Type      Comment
        ---------       ----      -------
        project         Disk      smbuser's project
        print$          Disk      Printer drivers
        IPC$            IPC       IPC Service (This is vbird's samba server)
        HP_LaserJet_P2015_Series Printer   HP LaserJet P2015 Series
        smb1            Disk      Home Directories <==等一下用这个当范例
Domain=[VBIRDHOUSE] OS=[Unix] Server=[Samba 3.5.4-68.el6_0.2]

        Server               Comment
        ---------            -------
        VBIRDSERVER          This is vbird's samba server

        Workgroup            Master
        ---------            -------
        VBIRDHOUSE           VBIRDSERVER
# 从这里可以知道在目前网域当中有多少个工作组与主要的名称解析主机
```
```
# 2. 利用类似 FTP 的方式登入远程主机
[root@clientlinux ~]# smbclient '//[IP|hostname]/资源名称' [-U username]
# 意思是使用某个账号来直接登入某部主机的某个分享资源，举例如下：
[root@clientlinux ~]# smbclient '//192.168.100.254/smb1' -U smb1
Enter smb1's password:
Domain=[VBIRDHOUSE] OS=[Unix] Server=[Samba 3.5.4-68.el6_0.2]
smb: \> dir
# 在 smb: \> 底下其实就是在 //192.168.100.254/dmtsai 这个目录底下。所以，
# 我们可以使用 dir, get, put 等常用的 ftp 指令来进行数据传输
?   :列出所有可以用的指令，常用！
cd  :变换到远程主机的目录
del :杀掉某个档案
lcd :变换本机端的目录
ls  :察看目前所在目录的档案
dir :与 ls 相同
get :下载单一档案
mget:下载大量档案
mput:上传大量档案
put :上传单一档案
rm  :删除档案
exit:离开 smbclient 的软件功能
# 其他的指令用法请参考 man smbclient 
```
### mount.cifs：直接挂载网芳成为网络驱动器机
早期的 Samba 主要是提供 smbmount 或 mount.smbfs 这个指令来挂载 (smbfs 是 SMB filesystem 的缩写)， 不过这个指令已经被可以进行比较好的编码判断的 mount.cifs 所取代。mount.cifs 可以将远程服务器分享出来的目录整个给他挂载到本机的挂载点，如此一来， 远程服务器的目录就好像在我们本机的一个分割槽一样。可以直接执行复制、编辑等动作。
```
[root@clientlinux ~]# mount -t cifs //IP/分享资源 /挂载点 [-o options]
选项与参数：
-o 后面接的参数 (options) 常用的有底下这些：
   username=你的登入账号：例如 username=smb1
   password=你的登入密码：需要与上面 username 相对应
   iocharset=本机的语系编码方式，如 big5 或 utf8 等等；
   codepage=远程主机的语系编码方式，例如繁体中文为cp950

# 范例一：以 smb1 的身份将其家目录挂载至 /mnt/samba 中
[root@clientlinux ~]# mkdir /mnt/samba
[root@clientlinux ~]# mount -t cifs //192.168.100.254/smb1 /mnt/samba \
> -o username=smb1,password=4321,codepage=cp950
[root@clientlinux ~]# df
文件系统               1K-区段      已用     可用 已用% 挂载点
//192.168.100.254/smb1/
                       7104632    143368   6606784   3% /mnt/samba
```
更详细的 mount 用法，请 man mount
### nmblookup：查询 NetBIOS name 与 IP 及其他相关信息：
现在我们可以透过一些 NetBIOS 相关的功能来取得 NetBIOS name ，不过，如果你还想要知道这个 NetBIOS name 的其他信息时， 例如 IP、分享的资源等等，那可以使用 nmblookup 这个指令：
```
[root@clientlinux ~]# nmblookup [-S] [-U wins IP] [-A IP] name
选项与参数：
-S ：除了查询 name 的 IP 之外，亦会找出该主机的分享资源与 MAC 等；
-U ：后面一般可接 Windows 的主要名称管理服务器的 IP ，可与 -R 互用；
-R ：与 -U 互用，以 Wins 服务器来查询某个 Netbios name；
-A ：相对于其他的参数， -A 后面可接 IP ，藉 IP 来找出相对的 NetBIOS 数据；

# 范例一：藉由 192.168.100.254 找出 vbirdserver 这部主机的 IP 地址
[root@clientlinux ~]# nmblookup -U 192.168.100.254 vbirdserver
querying vbirdserver on 192.168.100.254
192.168.100.254 vbirdserver<00>
192.168.1.100 vbirdserver<00>   

# 范例二：找出 vbirdserver 的 MAC 与 IP 等信息：
[root@clientlinux ~]# nmblookup -S vbirdserver
querying vbirdserver on 192.168.100.255  <==在区网内广播开始找！
192.168.100.254 vbirdserver<00>          <==找到 IP 
Looking up status of 192.168.100.254
        VBIRDSERVER     <00> -         B <ACTIVE>
        ..__MSBROWSE__. <01> - <GROUP> B <ACTIVE>
        VBIRDHOUSE      <00> - <GROUP> B <ACTIVE>
```
### smbtree：网芳浏览器显示模式
如果你想要使用类似 Windows 上面，可以一看就明了各个网芳所分享的资源时，你能使用 smbtree 来直接查询
```
[root@clientlinux ~]# smbtree [-bDS]
选项与参数：
-b ：以广播的方式取代主要浏览器的查询
-D ：仅列出工作组，不包括分享的资源
-S ：列出工作组与该工作组下的计算机名称 (NetBIOS) 不包括各项资源目录

# 范例一：列出目前的网芳树状相关图
[root@clientlinux ~]# smbtree
Enter root's password:  <==直接按 [Enter] 即可！
WORKGROUP
        \\WIN7-PC
VBIRDHOUSE
        \\WINXP
cli_start_connection: failed to connect to WINXP<20> (0.0.0.0). 
        \\VBIRDSERVER                   This is vbird's samba server
             \\VBIRDSERVER\HP_LaserJet_P2015_Series  HP LaserJet P2015 Series
             \\VBIRDSERVER\IPC$     IPC Service (This is vbird's samba server)
             \\VBIRDSERVER\print$   Printer drivers
             \\VBIRDSERVER\project  smbuser's project

[root@clientlinux ~]# smbtree -S
Enter root's password:
WORKGROUP
        \\WIN7-PC
VBIRDHOUSE
        \\WINXP
        \\VBIRDSERVER                   This is vbird's samba server
# 此时仅有工作组与计算机名称而已
```
### smbstatus：观察 SAMBA 的状态
其实这个指令算是服务器的相关功能。因为它主要的目的是查阅目前 SAMBA 有多少人来联机， 且哪些资源共享已经被使用等等的信息。所以如果你想要使用这个软件，请先安装 samba ：
```
[root@www ~]# smbstatus [-pS] [-u username]
选项与参数：
-p ：列出已经使用 SAMBA 联机的程序 PID ；
-S ：列出已经被使用的资源共享状态；
-u ：只列出某个用户相关的分享数据

# 范例一：列出目前主机完整的 Samba 状态
[root@www ~]# smbstatus
Samba version 3.5.4-68.el6_0.2
PID     Username      Group         Machine
-------------------------------------------------------------------
5993      smb1          smb1          __ffff_192.168.100.10 (::ffff:192.168.100.10)
5930      smb1          smb1          win7-pc      (::ffff:192.168.100.30)
# 上半部主要在列出目前联机的状态中，主要来自那个客户端机器与登入的用户名

Service      pid     machine       Connected at
-------------------------------------------------------
IPC$         5930   win7-pc       Fri Jul 29 15:56:03 2011
project      5930   win7-pc       Fri Jul 29 15:59:25 2011
smb1         5993   __ffff_192.168.100.10  Fri Jul 29 16:32:45 2011
# 这部分则显示出，目前有几个目录被使用了？那个 smb1 代表 //IP/smb1/
```

# 以 PDC 服务器提供账号管理
PDC 可以让用户在计算机教室的任何一个地方，都用同一组账号密码登入，并可取得相同的家目录等数据。与在 Linux 底下使用 NIS 搭配 NFS 是很类似的作法，只是它是用在 Windows 上。
## 让 Samba 管理网域使用者的一个实作案例
前面介绍的内容都是属于 Peer/Peer 的联机状况，也就是 Samba 服务器与 Windows 客户端其实是平等地位的。所以 Windows 客户端需要知道 Samba 服务器内的账号密码数据后，才能够顺利的使用 Samba 的资源。 不过，这样的方式在较大型一些的局域网络环境可能就会有点困扰，例如学校的环境。  
其实 Samba PDC 的作用很简单，就是让 Samba PDC 成为整个局域网络的领域管理员 (domain controller)， 然后让 Windows 主机加入这个领域，未来使用者利用 Windows 登入时，(1)Windows 会前往 PDC 服务器取得用户的账号密码， 同时 (2)PDC 还会传送用户的重要数据到那部 Windows 个人计算机上，而 Windows 计算机上的用户注销时， (3)该用户修改过的数据也会回传给 PDC 。如此一来不管这个使用者在哪一部个人计算机上面登入， 他都能够取得正确的个人资料。  
PDC 是个很复杂的环境，他可以达到的功能相当的多，而且密码的验证也不必在同一部 PDC 主机上面。假设底下的这部 PDC 使用 Linux 自己的密码来进行验证， 并且也只管理自己所分享出去的资源。  
整个基本的设定流程应该是这样的：
* 区网计算机环境设定：整体网域设定好，尤其 Windows 的工作组与计算机名称及 IP 等参数；
* PDC 设定：因为 PDC 管理自己的密码，所以 security = user；
* PDC 最好拥有整个网域的名称解析权力，亦即成为主要的名称解析器；
* 需有 netlogon 资源共享，提供 windows 2000/XP pro. 客户端的登入之用；
* 由于 Windows 需读入个人配置文件，默认目录为 profile，Linux 系统需预先设定此目录；
* 增加 PDC 上的使用者账号以及机器代码 (machine account) 等等
* 在 Windows 2000/XP pro. 个人计算机上设定成为 PDC 的客户端。
## PDC 服务器的建置
由于建置 PDC 的环境主要在管理整个区网内的 Windows 计算机，因此每部 Windows 计算机的主机名与相关参数要先确定下来。
### 1. 建置 NetBIOS 与 IP 对应的数据：设定 lmhosts 与 /etc/hosts
由于我们的 Samba 即将成为整个网域的名称解析者，因此你最好将整个网域的 NetBIOS name 与 IP 的对应写入 lmhosts 档案当中。如果你的区网是以 DHCP 发放 IP 的，那么你最好搭配 DNS 系统去建置你的主机名对应信息， 否则主机名对应不起来。在此案例中，由于使用的 NetBIOS name (如 vbirdserver) 与主机名 (如 www.centos.vbird) 并不相同，因此这里建议需要修改 lmhosts 
```
[root@www ~]# vim /etc/samba/lmhosts
127.0.0.1       localhost      <==这行是预设存在的，不要动他，底下的请自行新增
192.168.100.254  vbirdserver
192.168.100.10   vbirdlinux
192.168.100.20   vbirdwinxp
192.168.100.30   vbirdwin7

[root@www ~]# vim /etc/hosts
192.168.100.254 www.centos.vbird        vbirdserver
192.168.100.10  clientlinux.centos.vbird     vbirdlinux
192.168.100.20  vbirdwinxp
192.168.100.30  vbirdwin7
```
由于 Linux 上的 Samba 很多数据还是与 TCP/IP 的主机名有关，所以除了 lmhosts 之外，建议还是处理一下 /etc/hosts 。
### 2. 建置 PDC 主设定：处理 smb.conf
假设我们要让 PDC 客户端登入时可以取得他自己的家目录，那么需要这样处理：
```
[root@www ~]# vim /etc/samba/smb.conf
[global]
        workgroup       = vbirdhouse   <==请务必确认一下工作组与主机名
        netbios name    = vbirdserver
        server string   = This is vbird's samba server
        unix charset    = utf8
        display charset = utf8
        dos charset     = cp950
        log file        = /var/log/samba/log.%m
        max log size    = 50
        security        = user
        passdb backend  = tdbsam
        load printers   = yes
        cups options    = raw
        printcap name   = cups
        printing        = cups

        # 与 PDC 有关的一些设定值：
        # 底下几个设定值处理成为本局域网络内的主要名称解析器
        preferred master = yes
        domain master    = yes
        local master     = yes
        wins support     = yes
        # 操作系统 (OS) 等级越高才能成为主网域的控制者，一般 NT 为 32,
        # Windows 2000 为 64 ，所以这里我们设定高一点，但不可超过 255
        os level      = 100
        # 底下则是设定能否利用 PDC 登入，且登入需要进行哪些动作：
        domain logons = yes
        logon drive   = K:              <==登入后家目录挂载成 Windows 哪一槽
        logon script  = startup.bat     <==每个使用者登入后会自动执行的程序
        time server   = yes             <==自动调整 Windows 时间与 Samba 同步
        admin users   = root            <==预设的管理员账号！预设为 root 
        logon path    = \\%N\%U\profile <==使用者的个人化设定
        logon home    = \\%N\%U         <==用户的家目录位置！

# 这个在指定登入者能够进行的工作，里面主要是具有许多执行程序：
[netlogon]  <==与前面的 logon script 有关，该程序放置在这里
   comment         = Network Logon Service
   path            = /winhome/netlogon  <==重要的目录，要自己建立才行！
   writable        = no
   write list      = root
   follow symlinks = yes
   guest ok        = yes

[homes]
....(底下保留原本设定)....

[root@www ~]# testparm
[root@www ~]# /etc/init.d/smb restart
[root@www ~]# /etc/init.d/nmb restart
```
* time server：要使 Samba 与 Windows 主机的时间同步，使用这个项目；
* logon script：当使用者以 Windows 客户端登入后，Samba 可以提供一支批处理文件，让使用者去设定好他们自己的目录配置。整个配置的内容记录在 startup.bat 当中。 你要注意的是，这个 startup.bat 档名可以随意更改，不过他必须要放置到 [netlogon] 所指定的目录内；
* logon drive：那么这个家目录要挂载到那个分割槽？ 在 Windows 底下大多以 C, D, E... 做为磁盘的代号，你这里可以指定一下家目录要放置成为那个磁盘代号；
* admin users：指定这个 Samba PDC 的管理员身份。
* [netlogon]：指定利用网络登录时首先去查询的目录资源。
* logon path：用户登入后，会取得的环境设定数据在哪？ 我们知道用户会有一堆环境数据，例如桌面等，这些东西都放置到这里来。使用的变量中， %N 代表 PDC 服务器的位置，	%U 则代表用户的 Linux 家目录。因此最终你得要有 ~someone/profile 的目录才可以。
* logon home：用户的家目录，默认与 Linux 的家目录相同位置。
### 3. 建立 Windows 客户端登入时所需的设定数据 netlogon 目录
先来建立 [netlogon] 内所需要的数据，就是一个目录。预计将所有的 PDC 数据通通放置到 /winhome 当中，包括用户家目录，因此很多东西都需要修订:
```
[root@www ~]# mkdir -p /winhome/netlogon
```
接下来我们还得要建立允许使用者执行的档案，就是那个 startup.bat 。
```
[root@www ~]# vim /winhome/netlogon/startup.bat
net time \\vbirdserver /set /yes
net use K: /home
# 这个档案的格式为：net use [device:] [directory]

# 再将该档案转成 DOS 的断行格式才行！因为是提供给 Windows  系统

[root@www ~]# yum install unix2dos
[root@www ~]# unix2dos /winhome/netlogon/startup.bat
[root@www ~]# cat -A /winhome/netlogon/startup.bat
net time \\vbirdserver /set /yes^M$
net use K: /home^M$
# 会多出个 ^M 符号，那就是 Windows 断行字符。
```
### 4. 建立 WIndows 专用的使用者
预计将使用者全部挪到 /winhome 底下，而且每个用户家目录应该还要有 profile 目录存在才行， 为了避免麻烦，所以我们先到 /etc/skel 去处理一下，然后才建立账号，最后才产生 samba 用户。产生 samba 用户可以使用 pdbedit 也能够直接使用 smbpasswd -a ，因为没有要用特殊的参数， 所以，Samba 用户就用旧的 smbpasswd 来处理即可。
```
[root@www ~]# mkdir /etc/skel/profile
[root@www ~]# useradd -d /winhome/dmtsai dmtsai
[root@www ~]# useradd -d /winhome/nikky  nikky
[root@www ~]# smbpasswd -a root
[root@www ~]# smbpasswd -a dmtsai
[root@www ~]# smbpasswd -a nikky
[root@www ~]# pdbedit -L
smb1:2004:
smb3:2006:
smb2:2005:
student:505:
root:0:root
dmtsai:2007:
nikky:2008:
# 重点是需要有画底线的那几个人物出现

[root@www ~]# ll /winhome
drwx------. 5 dmtsai dmtsai 4096 Jul 29 16:49 dmtsai
drwxr-xr-x. 2 root   root   4096 Jul 29 16:48 netlogon
drwx------. 5 nikky  nikky  4096 Jul 29 16:49 nikky
# 用户的家目录不是在 /home 而是在 /winhome 里头才是对的
```
那以后新增的使用者都有可以存放来自 Windows 的特殊配置文件目录，比较好管理。使用 useradd 新增使用者后，记得也要使用 smbpasswd -a username 来让该使用者可以使用 Samba 。
### 5. 建立机器码账号
由于 PDC 会针对 Windows 客户端的主机名 (NetBIOS name) 进行主机账号检查， 所以我们也要为客户端的主机名进行账号的设定。一般用户账号是英文或数字，主机账号则在该账号最后面加上一个钱字号『$』即可。举例来说， vbirdwinxp 这部主机可设定的账号名称为 vbirdwinxp$。  
要使用 smbpasswd 增加的使用者必须要在 /etc/passwd 当中，因此要建立这个账号你就得要这样做：
```
[root@www ~]# useradd -M -s /sbin/nologin -d /dev/null vbirdwinxp$
[root@www ~]# useradd -M -s /sbin/nologin -d /dev/null vbirdwin7$
```
会增加 -M -s -d 等参数的原因是因为不想要让这个账号具有可以登入的权限，接下来让 Samba 知道这个账号是主机账号，所以你应该要这样做：
```
[root@www ~]# smbpasswd -a -m vbirdwinxp$
[root@www ~]# smbpasswd -a -m vbirdwin7$
```
Samba PDC 也就可以透过『主机账号』来判断 Windows 客户端能否连上来， 若连接上 PDC 与 Windows 客户端后，接下来一般使用者账号就可以在 windows 客户端登入了。
### 6. 修改安全性相关数据
由于我们建立的账号目录在 /winhome 底下，并非正规的 CentOS 目录，所以最重要的 SELinux 可能会跑掉，所以，还要修改 SELinux 。将 SELinux type 转为 samba_share_t 即可：
```
[root@www ~]# chcon -R -t samba_share_t /winhome
```
由于 SELinux 的数据是会继承上层目录的，因此未来新增的用户，理论上，就不需要重新修订 SELinux 的文件类型了。 但是，如果你老是发现登入 PDC 的账号却无法取得家目录，那么就观察 /var/log/messages 内的资料来修订。
## Windows XP Pro 的客户端
### 1. 确认 windows 客户端的网域与主机名
### 2. 设定主机名与域名
### 3. 重新启动并以新的域名登入
### 4. 观察用户的家目录与配置文件
当你注销之后，你在 Windows 桌面上头所进行的各项个人化设定通通会被移动到 /winhome/dmtsai/profile 当中
## Windows 7 的客户端
## PDC 之问题克服
如果老是发生错误讯息为『使用的帐户是计算机帐户。请使用你的通用用户帐户或本机用户帐户来存取这台服务器』时，你可以这样做：
* 先察看一下 /var/log/samba 里面的登录文件信息，尤其是 log.vbirdwinxp 关于这部主机的信息
* 如果还是无法解决，可以在 lmhosts 里面增加 vbirdwinxp 的 IP 与主机名的对应，然后将 samba 整个关掉『/etc/init.d/smb stop』，等待一段时间让 NetBIOS 的名称解析时间逾时，再重新启动 samba 『/etc/init.d/smb start』，然后再重新做一次输入 root 的密码那个动作
### 一些 Windows 账号在 Windows 系统上面的使用技巧
虽然 PDC 很好用，不过你要注意的是，每次你使用 PDC 上头的账号登入 Windows 客户端主机时， Windows 主机会由 /winhome/username/profile/ 当中加载所需要的数据， 并暂时启动一个文件夹在 Windows 系统的 C:\Documents and Settings\username 当中，如果你的家目录下的 profile 数据太多时， 光是传输就会花去很多时间。  
所以，你应该将一些档案数据放置到你的家目录下，亦即 K 槽当中，尽量不要使用 Windows 预设的『我的文档夹』， 因为『我的文档夹』会将数据移动到『 /winhome/username/profile/My Documents/ 』目录下，同样的， 储存到桌面的数据会被放置到『 /winhome/username/profile/桌面/ 』目录中，那样在登入与注销时会花去很多时间。  

# 服务器简单维护与管理
## 服务器相关问题克服
通常我们在设定 SAMBA 的时候，如果是以单一主机的工作组 (Workgroup) 的方式来进行 smb.conf 的设定时，几乎很容易就可以设定成功了，并没有什么很困难的步骤。不过，万一还是无法成功的设定起来， 请务必察看登录档，也就是在 /var/log/samba/ 里面的数据。在这里面的资料当中，你会发现有很多档案，因为我们在 smb.conf 里面设定了：
* log file = /var/log/samba/log.%m

那个 %m 是指客户端计算机的 NetBIOS Name 的意思，所以，当有个 vbirdwinxp 的主机来登入我们的 vbirdserver 主机时，那么登入的信息就会被纪录在 /var/log/samba/log.vbirdwinxp 档案。而如果万一来源 IP 并没有 Netbios name 的时候，那么很可能是一些错误讯息，这些错误讯息就会被纪录到 log.smbd, log.nmbd 里面去。  
另外，如果你的 SAMBA 明明已经启动完成了，却偏偏老是无法成功，又无法查出问题时，建议先关闭 Samba 一阵子，再重新启动：
* /etc/init.d/smb stop

还有，万一你在进行写入的动作时，老是发现『你没有相关写入的权限』，几乎可以确定是 Permission 的问题，也就是 Linux 的权限与 SAMBA 开放的权限并不相符合，或者是 SELinux 的问题。能不能写入 Linux 磁盘，看的是 PID 的权限与 Linxu 文件系统是否吻合，而那个 smb.conf 里面设定的相关权限只是在 SAMBA 运作过程当中『预计』要给使用者的权限而已，并不能取代真正的 Linux 权限。  
另外，通常造成明明已经查到分享 (smbclient -L 的结果)，却老是无法顺利挂载的情况，主要有底下几个可能的原因：
* 虽然 smb.conf 设定正确，但是设定值『 path 』所指定的目录却忘记建立了 (最常见)；
* 虽然 smb.conf 设定为可擦写，但是目录针对该用户的权限却是只读或者是无权限；
* 虽然权限全部都正确，但是 SELinux 的类型却错误了！
* 虽然全部的数据都是正确的，但是 SELinux 的规则 (getsebool -a) 却没有顺利启动。
## 让使用者修改 samba 密码同时同步更新 /etc/shadow 密码
使用者可以透过 passwd 修改 /etc/shadow 内的密码，而且用户也能够自行以 smbpasswd 修改 Samba 的密码。如果用户是类似 PDC 的用户，那么这些用户理论上就很少使用 Linux 。能否让用户在修改 Windows 密码 (就是 Samba) 时，同步更新 Linux 上面的 /etc/shadow 密码？smb.conf 里头提供了相对应的参数设定值。
```
[root@www ~]# vim /etc/samba/smb.conf
[global]
# 保留前面的各项设定值，并新增底下三行即可：
        unix password sync  = yes                <==让 Samba 与 Linux 密码同步
        passwd program      = /usr/bin/passwd %u <==以 root 呼叫修改密码的指令
        pam password change = yes                <==并且支持 pam 模块！

[root@www ~]# testparm
[root@www ~]# /etc/init.d/smb restart
```
接下来，当你以一般用户 (例如 dmtsai) 修改 samba 的密码时，就会像这样：
```
[dmtsai@www ~]$ smbpasswd
Old SMB password:  <==得先输入旧密码，才能输入新密码
New SMB password:
Retype new SMB password:
Password changed for user dmtsai <==这就是成功的字样！

# 若出现底下的字样，应该就是你的密码输入被限制了！例如输入的密码字符少于 6 个！
machine 127.0.0.1 rejected the password change: Error was : Password restriction.
Failed to change password for dmtsai
```
## 利用 ACL 配合单一使用者时的控管
