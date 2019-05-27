# 软件管理员简介
在 Linux 上面至少就有两种常见的软件管理员，分别是 RPM 与 Debian 的 dpkg 。
## Linux 界的两大主流：RPM 与 DPKG
Linux 开发商先在固定的硬件平台与操作系统平台上面将需要安装或升级的软件编译好， 然后将这个软件的所有相关文件打包成为一个特殊格式的文件，在这个软件文件内还包含了预先侦测系统与相依软件的脚本， 并提供记载该软件提供的所有文件资讯等。最终将这个软件文件释出。用户端取得这个文件后，只要透过特定的命令来安装， 那么该软件文件就会依照内部的脚本来侦测相依的前驱软件是否存在，若安装的环境符合需求，那就会开始安装， 安装完成后还会将该软件的资讯写入软件管理机制中，以达成未来可以进行升级、移除等动作。  
目前在 Linux 界软件安装方式最常见的有两种，分别是：
* dpkg：  
  这个机制最早是由 Debian Linux 社群所开发出来的，透过 dpkg 的机制， Debian 提供的软件就能够简单的安装起来，同时还能提供安装后的软件资讯。 只要是衍生于 Debian 的其他 Linux distributions 大多使用 dpkg 这个机制来管理软件的， 包括 B2D, Ubuntu 等等。
* RPM：  
  这个机制最早是由 Red Hat 这家公司开发出来的，后来很多 distributions 使用这个机制来作为软件安装的管理方式。包括 Fedora, CentOS, SuSE 等等。

对于软件属性相依的问题，将相依属性的数据做成列表， 等到实际软件安装时，若发生有相依属性的软件状况时，例如安装 A 需要先安装 B 与 C ，而安装 B 则需要安装 D 与 E 时，那么当你要安装 A ，透过相依属性列表，管理机制自动去取得 B, C, D, E 来同时安装，就解决了属性相依的问题。  
在 dpkg 管理机制上就开发出 APT 的线上升级机制，RPM 则依开发商的不同，有 Red Hat 系统的 yum ， SuSE 系统的 Yast Online Update (YOU)， Mandriva 的 urpmi 软件等。
distribution 代表|软件管理机制|使用命令|线上升级机制(命令)
:---:|:---:|:---:|:---:
Red Hat/Fedora|RPM|rpm, rpmbuild|YUM (yum)
Debian/Ubuntu|DPKG|dpkg|APT (apt-get)
CentOS 使用的软件管理机制为 RPM 机制，而用来作为线上升级的方式则为 yum 。
## RPM 与 SRPM
RPM 全名是『 RedHat Package Manager 』。当初这个软件管理的机制是由 Red Hat 这家公司发展出来的。 RPM 是以一种数据库记录的方式来将你所需要的软件安装到你的 Linux 系统的一套管理机制。  
他最大的特点就是将你要安装的软件先编译过， 并且打包成为 RPM 机制的包装文件，透过包装好的软件里头默认的数据库记录， 记录这个软件要安装的时候必须具备的相依属性软件，当安装在你的 Linux 主机时， RPM 会先依照软件里头的数据查询 Linux 主机的相依属性软件是否满足， 若满足则予以安装，若不满足则不予安装。那么安装的时候就将该软件的资讯整个写入 RPM 的数据库中，以便未来的查询、验证与反安装。优点是：
1. 由于已经编译完成并且打包完毕，所以软件传输与安装上很方便 (不需要再重新编译)；
2. 由于软件的资讯都已经记录在 Linux 主机的数据库上，很方便查询、升级与反安装

由于 RPM 文件是已经包装好的数据，也就是说， 里面的数据已经都『编译完成』了！所以，该软件文件几乎只能安装在原本默认的硬件与操作系统版本中。也就是说，你的主机系统环境必须要与当初创建这个软件文件的主机环境相同才行。  
所以，通常不同的 distribution 所释出的 RPM 文件，并不能用在其他的 distributions 上。更有甚者，相同 distribution 的不同版本之间也无法互通。这些软件管理机制的问题是：
1. 软件文件安装的环境必须与打包时的环境需求一致或相当；
2. 需要满足软件的相依属性需求；
3. 反安装时需要特别小心，最底层的软件不可先移除，否则可能造成整个系统的问题

想安装其他 distributions 提供的 RPM 软件文件时，要使用 SRPM 。Source RPM 的意思，也就是这个 RPM 文件里面含有原始码。特别注意的是，这个 SRPM 所提供的软件内容『并没有经过编译』， 他提供的是原始码。  
通常 SRPM 的扩展名是以 ***.src.rpm 这种格式来命名的。因为 SRPM 虽然内容是原始码， 但是他仍然含有该软件所需要的相依性软件说明、以及所有 RPM 文件所提供的数据。同时，他与 RPM 不同的是，他也提供了参数配置档 (就是 configure 与 makefile)。所以，如果我们下载的是 SRPM ，那么要安装该软件时，你就必须要：
* 先将该软件以 RPM 管理的方式编译，此时 SRPM 会被编译成为 RPM 文件；
* 然后将编译完成的 RPM 文件安装到 Linux 系统当中

通常一个软件在释出的时候，都会同时释出该软件的 RPM 与 SRPM 。RPM 文件必须要在相同的 Linux 环境下才能够安装，而 SRPM 既然是原始码的格式，自然我们就可以透过修改 SRPM 内的参数配置档，然后重新编译产生能适合我们 Linux 环境的 RPM 文件。
文件格式|档名格式|直接安装与否|内含程序类型|可否修改参数并编译
:---:|:---:|:---:|:---:|:---:
RPM|xxx.rpm|可|已编译|不可
SRPM|xxx.src.rpm|不可|未编译之原始码|可
> CentOS 是 『社群维护的企业版』。Red Hat 公司的 RHEL 释出后，连带会将 SRPM 释出。 社群的朋友就将这些 SRPM 收集起来并重新编译成为所需要的软件，再重复释出成为 CentOS，所以才能号称与 Red Hat 的 RHEL 企业版同步。
## i386, i586, i686, noarch, x86_64
RPM 与 SRPM 的格式分别为：
```
xxxxxxxxx.rpm   <==RPM 的格式，已经经过编译且包装完成的 rpm 文件；
xxxxx.src.rpm   <==SRPM的格式，包含未编译的原始码资讯。
```
通过档名就可以知道这个软件的版本、适用的平台、编译释出的次数等信息。
```
rp-pppoe -        3.1    -     5        .i386        .rpm
软件名称   软件的版本资讯 释出的次数 适合的硬件平台 扩展名
```
除了后面适合的硬件平台与扩展名外，主要是以『-』来隔开各个部分。
* 软件名称：  
  就是每一个软件的名称了，范例就是 rp-pppoe 。
* 版本资讯：  
  每一次升级版本就需要有一个版本的资讯，否则如何知道这一版是新是旧？这里通常又分为主版本跟次版本。以上面为例，主版本为 3 ，在主版本的架构下更动部分原始码内容，而释出一个新的版本，就是次版本
* 释出版本次数：  
  通常就是编译的次数。由于同一版的软件中，可能由于有某些 bug 或者是安全上的顾虑，所以必须要进行小幅度的 patch 或重设一些编译参数。 配置完成之后重新编译并打包成 RPM 文件。因此就有不同的打包数出现了。
* 操作硬件平台：  
  由于 RPM 可以适用在不同的操作平台上，但是不同的平台配置的参数还是有所差异性。并且，我们可以针对比较高阶的 CPU 来进行最佳化参数的配置，这样才能够使用高阶 CPU 所带来的硬件加速功能。 所以就有所谓的 i386, i586, i686, x86_64 与 noarch 等的文件名称出现了。
平台名称|适合平台说明
:---:|:---:
i386|几乎适用于所有的 x86 平台，不论是旧的 pentum 或者是新的 Intel Core 2 与 K8 系列的 CPU 等等，都可以正常的工作。那个 i 指的是 Intel 兼容的 CPU 的意思，至于 386 不用说，就是 CPU 的等级。
i586|就是针对 586 等级的计算机进行最佳化编译。包括 pentum 第一代 MMX CPU， AMD 的 K5, K6 系列 CPU (socket 7 插脚) 等等的 CPU 都算是这个等级；
i686|在 pentun II 以后的 Intel 系列 CPU ，及 K7 以后等级的 CPU 都属于这个 686 等级！ 由于目前市面上几乎仅剩 P-II 以后等级的硬件平台，因此很多 distributions 都直接释出这种等级的 RPM 文件。
x86_64|针对 64 位的 CPU 进行最佳化编译配置，包括 Intel 的 Core 2 以上等级 CPU ，以及 AMD 的 Athlon64 以后等级的 CPU ，都属于这一类型的硬件平台。
noarch|就是没有任何硬件等级上的限制。一般来说，这种类型的 RPM 文件，里面应该没有 binary program 存在， 较常出现的就是属于 shell script 方面的软件。
受惠于目前 x86 系统的支持方面，新的 CPU 都能够运行旧型 CPU 所支持的软件，也就是说硬件方面都可以向下兼容的， 因此最低等级的 i386 软件可以安装在所有的 x86 硬件平台上面，不论是 32 位还是 64 位。但是反过来说就不行了。  
根据上面的说明，其实只要选择 i386 版本来安装在 x86 硬件上面就肯定没问题。但是如果强调效能的话， 还是选择搭配硬件的 RPM 文件，该软件针对 CPU 硬件平台进行过参数最佳化的编译。
## RPM 的优点
由于 RPM 是透过预先编译并打包成为 RPM 文件格式后，再加以安装的一种方式，并且还能够进行数据库的记载。 所以 RPM 有以下的优点：
* RPM 内含已经编译过的程序与配置档等数据，可以让使用者免除重新编译的困扰；
* RPM 在被安装之前，会先检查系统的硬盘容量、操作系统版本等，可避免文件被错误安装；
* RPM 文件本身提供软件版本资讯、相依属性软件名称、软件用途说明、软件所含文件等资讯，便于了解软件；
* RPM 管理的方式使用数据库记录 RPM 文件的相关参数，便于升级、移除、查询与验证。

为了解决这种具有相关性的软件之间的问题 (就是所谓的软件相依属性)，RPM 就在提供打包的软件时，同时加入一些信息登录的功能，这些信息包括软件的版本、 打包软件者、相依属性的其他软件、本软件的功能说明、本软件的所有文件记录等等，然后在 Linux 系统上面亦创建一个 RPM 软件数据库，如此一来，当你要安装某个以 RPM 型态提供的软件时，在安装的过程中， RPM 会去检验一下数据库里面是否已经存在相关的软件了， 如果数据库显示不存在，那么这个 RPM 文件『默认』就不能安装。
## RPM 属性相依的克服方式：YUM 线上升级
为了重复利用既有的软件功能，因此很多软件都会以函式库的方式释出部分功能，以方便其他软件的呼叫应用， 例如 PAM 模块的验证功能。此外，为了节省使用者的数据量，目前的 distributions 在释出软件时， 都会将软件的内容分为一般使用与开发使用 (development) 两大类。类似 pam-x.x.rpm 与 pam-devel-x.x.rpm 之类的档名。默认情况下，大部分的 software-devel-x.x.rpm 都不会安装，因为终端用户大部分不会去开发软件。  
CentOS 先将释出的软件放置到 YUM 服务器内，然后分析这些软件的相依属性问题，将软件内的记录资讯写下来 (header)。 然后再将这些资讯分析后记录成软件相关性的清单列表。这些列表数据与软件所在的位置可以称呼为容器 (repository)。当用户端有软件安装的需求时，用户端主机会主动的向网络上面的 yum 服务器的容器网址下载清单列表， 然后透过清单列表的数据与本机 RPM 数据库已存在的软件数据相比较，就能够一口气安装所有需要的具有相依属性的软件了。  
当用户端有升级、安装的需求时， yum 会向容器要求清单的升级，等到清单升级到本机的 /var/cache/yum 里面后， 等一下升级时就会用这个本机清单与本机的 RPM 数据库进行比较，这样就知道该下载什么软件。接下来 yum 会跑到容器服务器 (yum server) 下载所需要的软件，然后再透过 RPM 的机制开始安装软件。
> 由于 yum 服务器提供的 RPM 文件内容可能有所差异，举例来说，原厂释出的数据有 (1)原版数据； (2)升级数据 (update)； (3)特殊数据 (例如第三方协力软件，或某些特殊功能的软件)。 这些软件文件基本上不会放置到一起，就用『容器』的概念来区分这些软件功能，不同的『容器』网址，可以放置不同的软件功能之意。

# RPM 软件管理程序：rpm
## RPM 默认安装的路径
一般来说，RPM 类型的文件在安装的时候，会先去读取文件内记载的配置参数内容，然后将该数据用来比对 Linux 系统的环境，以找出是否有属性相依的软件尚未安装的问题。  
若环境检查合格了，那么 RPM 文件就开始被安装到你的 Linux 系统上。安装完毕后，该软件相关的资讯就会被写入 /var/lib/rpm/ 目录下的数据库文件中了。未来如果我们有任何软件升级的需求，版本之间的比较就是来自于这个数据库，而如果你想要查询系统已经安装的软件，也是从这里查询。同时，目前的 RPM 也提供数码签章资讯， 这些数码签章也是在这个目录内记录的。
|||
:---:|:---:
/etc|一些配置档放置的目录，例如 /etc/crontab
/usr/bin|一些可运行文件
/usr/lib|一些程序使用的动态函式库
/usr/share/doc|一些基本的软件使用手册与说明档
/usr/share/man|一些 man page 文件
## RPM 安装 (install)
root 的身份才能够操作 rpm 命令。
```
[root@www ~]# rpm -ivh package_name
选项与参数：
-i ：install 的意思
-v ：察看更细部的安装资讯画面
-h ：以安装资讯列显示安装进度

# 后面直接接上许多的软件文件！

直接由网络上面的某个文件安装，以网址来安装：
[root@www ~]# rpm -ivh http://website.name/path/pkgname.rpm
```
如果安装过程中发现问题，仍要安装，可以使用如下的参数『强制』安装上去：
可下达的选项|代表意义
:---:|:---:
--nodeps|使用时机：当发生软件属性相依问题而无法安装，但你执意安装时<br>危险性： 软件会有相依性的原因是因为彼此会使用到对方的机制或功能，如果强制安装而不考虑软件的属性相依， 则可能会造成该软件的无法正常使用！
--replacefiles|使用时机： 如果在安装的过程当中出现了『某个文件已经被安装在你的系统上面』的资讯，又或许出现版本不合的信息 (confilcting files) 时，可以使用这个参数来直接覆盖文件。<br>危险性： 覆盖的动作是无法复原的！所以，你必须要很清楚的知道被覆盖的文件是真的可以被覆盖。
--replacepkgs|使用时机： 重新安装某个已经安装过的软件。如果你要安装一堆 RPM 软件文件时，可以使用 rpm -ivh *.rpm ，但若某些软件已经安装过了， 此时系统会出现『某软件已安装』的资讯，导致无法继续安装。此时可使用这个选项来重复安装。
--force|使用时机：这个参数其实就是 --replacefiles 与 --replacepkgs 的综合体！
--test|使用时机： 想要测试一下该软件是否可以被安装到使用者的 Linux 环境当中，可找出是否有属性相依的问题。范例为：<br>rpm -ivh pkgname.i386.rpm --test
--justdb|使用时机： 由于 RPM 数据库破损或者是某些缘故产生错误时，可使用这个选项来升级软件在数据库内的相关资讯。
--nosignature|使用时机： 想要略过数码签章的检查时，可以使用这个选项。
--prefix|新路径	使用时机： 要将软件安装到其他非正规目录时。举例来说，你想要将某软件安装到 /usr/local 而非正规的 /bin, /etc 等目录， 就可以使用『 --prefix /usr/local 』来处理了。
--noscripts|使用时机：不想让该软件在安装过程中自行运行某些系统命令。<br>说明： RPM 的优点除了可以将文件放置到定位之外，还可以自动运行一些前置作业的命令，例如数据库的初始化。 如果你不想要让 RPM 帮你自动运行这一类型的命令，就加上他。
一般来说，rpm 的安装选项与参数大约就是这些了。建议直接使用 -ivh 就好了， 如果安装的过程中发现问题，一个一个去将问题找出来，尽量不要使用『 暴力安装法 』，就是透过 --force 去强制安装。
## RPM 升级与升级 (upgrade/freshen)
以 -Uvh 或 -Fvh 来升级即可，而 -Uvh 与 -Fvh 可以用的选项与参数，跟 install 是一样的。不过， -U 与 -F 的意义还是不太一样的，基本的差别是这样的：
|||
:---:|:---:
-Uvh|后面接的软件即使没有安装过，则系统将予以直接安装； 若后面接的软件有安装过旧版，则系统自动升级至新版；
-Fvh|如果后面接的软件并未安装到你的 Linux 系统上，则该软件不会被安装；亦即只有已安装至你 Linux 系统内的软件会被『升级』。
想要大量的升级系统旧版本的软件时，使用 -Fvh 则是比较好的作法，因为没有安装的软件不会被安装进系统中。升级也是可以利用 --nodeps/--force 等等的参数。
## RPM 查询 (query)
RPM 在查询的时候，其实查询的地方是在 /var/lib/rpm/ 这个目录下的数据库文件。另外， RPM 也可以查询未安装的 RPM 文件内的资讯。
```
[root@www ~]# rpm -qa                              <==已安装软件
[root@www ~]# rpm -q[licdR] 已安装的软件名称       <==已安装软件
[root@www ~]# rpm -qf 存在于系统上面的某个档名     <==已安装软件
[root@www ~]# rpm -qp[licdR] 未安装的某个文件名称  <==查阅RPM文件
选项与参数：
查询已安装软件的资讯：
-q  ：仅查询，后面接的软件名称是否有安装；
-qa ：列出所有的，已经安装在本机 Linux 系统上面的所有软件名称；
-qi ：列出该软件的详细资讯 (information)，包含开发商、版本与说明等；
-ql ：列出该软件所有的文件与目录所在完整档名 (list)；
-qc ：列出该软件的所有配置档 (找出在 /etc/ 底下的档名而已)
-qd ：列出该软件的所有说明档 (找出与 man 有关的文件而已)
-qR ：列出与该软件有关的相依软件所含的文件 (Required 的意思)
-qf ：由后面接的文件名称，找出该文件属于哪一个已安装的软件；
查询某个 RPM 文件内含有的资讯：
-qp[icdlR]：注意 -qp 后面接的所有参数以上面的说明一致。但用途仅在于找出某个 RPM 文件内的资讯，而非已安装的软件资讯。
```
在查询的部分，所有的参数之前都需要加上 -q 才是所谓的查询！查询主要分为两部分， 一个是查已安装到系统上面的的软件资讯，这部份的资讯都是由 /var/lib/rpm/ 所提供。另一个则是查某个 rpm 文件内容， 等于是由 RPM 文件内找出一些要写入数据库内的资讯就是了，这部份就得要使用 -qp (p 是 package 的意思)。  
在查询本机上面的 RPM 软件相关资讯时， 不需要加上版本的名称，只要加上软件名称即可。但是查询某个 RPM 文件就不同了，我们必须要列出整个文件的完整档名才行。  
即使指定的文件被误删，因为 RPM 有记录在 /var/lib/rpm 当中的数据库，仍可以使用 rpm -qf 查询。
## RPM 验证与数码签章 (Verify/signature)
验证 (Verify) 的功能主要在于提供系统管理员一个有用的管理机制。作用的方式是『使用 /var/lib/rpm 底下的数据库内容来比对目前 Linux 系统的环境下的所有软件文件 』也就是说，当你有数据不小心遗失， 或者是因为你误杀了某个软件的文件，或者是不小心不知道修改到某一个软件的文件内容， 就用这个简单的方法来验证一下原本的文件系统。
```
[root@www ~]# rpm -Va
[root@www ~]# rpm -V  已安装的软件名称
[root@www ~]# rpm -Vp 某个 RPM 文件的档名
[root@www ~]# rpm -Vf 在系统上面的某个文件
选项与参数：
-V  ：后面加的是软件名称，若该软件所含的文件被更动过，才会列出来；
-Va ：列出目前系统上面所有可能被更动过的文件；
-Vp ：后面加的是文件名称，列出该软件内可能被更动过的文件；
-Vf ：列出某个文件是否被更动过
```
```
[root@www ~]# rpm -V logrotate
..5....T  c /etc/logrotate.conf
```
c 代表的是 configuration ， 就是配置档的意思。至于最前面的八个资讯是：
* S ：(file Size differs) 文件的容量大小是否被改变
* M ：(Mode differs) 文件的类型或文件的属性 (rwx) 是否被改变？如是否可运行等参数已被改变
* 5 ：(MD5 sum differs) MD5 这一种指纹码的内容已经不同
* D ：(Device major/minor number mis-match) 装置的主/次代码已经改变
* L ：(readLink(2) path mis-match) Link 路径已被改变
* U ：(User ownership differs) 文件的所属人已被改变
* G ：(Group ownership differs) 文件的所属群组已被改变
* T ：(mTime differs) 文件的创建时间已被改变

所以，如果当一个配置档所有的资讯都被更动过，那么他的显示就会是：
```
SM5DLUGT c filename
```
c 代表的是『 Config file 』的意思，也就是文件的类型，文件类型有底下这几类：
* c ：配置档 (config file)
* d ：文件数据档 (documentation)
* g ：鬼文件～通常是该文件不被某个软件所包含，较少发生！(ghost file)
* l ：授权文件 (license file)
* r ：读我文件 (read me)

一般来说，配置档 (configure) 被更动过是很正常的， binary program 被更动过就有异常了。
### 数码签章 (digital signature)
验证只能验证软件内的资讯与 /var/lib/rpm/ 里面的数据库资讯而已，如果该软件文件所提供的数据本身就有问题，使用验证的手段也无法确定该软件的正确性。在 Tarball 与文件的验证方面，我们可以使用前一章谈到的 md5 指纹码来检查， 指纹码也可能会被窜改。我们可以透过数码签章来检验软件的来源的。  
厂商可以数码签章系统产生一个专属于该软件的签章，并将该签章的公钥 (public key) 释出。 当你要安装一个 RPM 文件时：
1. 首先你必须要先安装原厂释出的公钥文件；
2. 实际安装原厂的 RPM 软件时， rpm 命令会去读取 RPM 文件的签章资讯，与本机系统内的签章资讯比对，
3. 若签章相同则予以安装，若找不到相关的签章资讯时，则给予警告并且停止安装。

CentOS 使用的数码签章系统为 GNU 计画的 GnuPG (GNU Privacy Guard, GPG)。 GPG 可以透过杂凑运算，算出独一无二的专属金钥系统或者是数码签章系统。CentOS 的数码签章位于：
```
[root@www ~]# ll /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-5
-rw-r--r-- 1 root root 1504  6月 19  2008 /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-5
[root@www ~]# cat /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-5
-----BEGIN PGP PUBLIC KEY BLOCK-----
Version: GnuPG v1.2.6 (GNU/Linux)

mQGiBEWfB6MRBACrnYW6yKMT+MwJlCIhoyTxGf3mAxmnAiDEy6HcYN8rivssVTJk
....(中间省略)....
-----END PGP PUBLIC KEY BLOCK-----
```
该数码签章码其实仅是一个乱数而已，这个乱数对于数码签章有意义。安装这个文件:
```
[root@www ~]# rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-5
```
由于不同版本 GPG 金钥文件放置的位置可能不同，不过档名大多是以 GPG-KEY 来说明的， 因此你可以简单的使用 locate 或 find 来找寻，如以下的方式来搜寻即可：
```
[root@www ~]# locate GPG-KEY
[root@www ~]# find /etc -name '*GPG-KEY*'
```
安装完成之后，这个金钥的内容会以 pubkey 作为软件的名称。
```
[root@www ~]# rpm -qa | grep pubkey
gpg-pubkey-e8562897-459f07a4
[root@www ~]# rpm -qi gpg-pubkey-e8562897-459f07a4
Name        : gpg-pubkey        Relocations: (not relocatable)
Version     : e8562897               Vendor: (none)
Release     : 459f07a4           Build Date: Wed 27 May 2009 10:07:26 PM CST
Install Date: Wed 27 May 2009 10:07:26 PM CST   Build Host: localhost
Group       : Public Keys        Source RPM: (none)
Size        : 0                     License: pubkey
Signature   : (none)
Summary     : gpg(CentOS-5 Key <centos-5-key@centos.org>)
Description :
-----BEGIN PGP PUBLIC KEY BLOCK-----
Version: rpm-4.4.2 (beecrypt-4.1.2)
....(底下省略)....
```
如果忘记加上数码签章，很可能很多原版软件就无法安装，除非你利用 rpm 时选择略过数码签章的选项。
## RPM 反安装与重建数据库 (erase/rebuilddb)
卸载。『解安装的过程一定要由最上一级往下解除』。  
通过 -e 选项移除。不过，常发生软件属性相依导致无法移除某些软件的问题。例如， pam 所提供的函式库是让非常多其他软件使用的，因此你不能移除 pam ，除非将其他相依软件一口气也全部移除。当然也能加 --nodeps 来强制移除， 不过，如此一来所有会用到 pam 函式库的软件，都将成为无法运行的程序。  
由于 RPM 文件常常会安装/移除/升级等，某些动作或许可能会导致 RPM 数据库 /var/lib/rpm/ 内的文件破损。使用 --rebuilddb 这个选项来重建一下数据库：
```
[root@www ~]# rpm --rebuilddb   <==重建数据库
```

# SRPM 的使用：rpmbuild
新版的 rpm 已经将 RPM 与 SRPM 的命令分开了，SRPM 使用的是 rpmbuild 这个命令，而不是 rpm 。
## 利用默认值安装 SRPM 文件 (--rebuild/--recompile)
rpmbuild 配合选项主要有以下两个：
|||
:---:|:---:
--rebuild|这个选项会将后面的 SRPM 进行『编译』与『打包』的动作，最后会产生 RPM 的文件，但是产生的 RPM 文件并没有安装到系统上。当你使用 --rebuild 的时候，最后通常会发现一行字体：<br>Wrote: /usr/src/redhat/RPMS/i386/pkgname.i386.rpm<br>这个就是编译完成的 RPM 文件，这个文件就可以用来安装。安装的时候请加绝对路径来安装即可。
--recompile|这个动作会直接的『编译』『打包』并且『安装』。
这两个选项都没有修改过 SRPM 内的配置值，仅是透过再次编译来产生 RPM 可安装软件文件而已。 一般来说，如果编译的动作顺利的话，那么编译过程所产生的中间缓存档都会被自动删除，如果发生任何错误， 则该中间文件会被保留在系统上，等待使用者的除错动作。自行出错，或者要修改 SRPM 内的配置值时，要知道利用 SRPM 的时候，系统会动用到哪些重要的目录。
## SRPM 使用的路径与需要的软件
SRPM 既然含有 source code ，那么其中必定有配置档。以 CentOS 5.x 以 /usr/src/redhat/ 为工作目录， Openlinux 则是以 /usr/src/openlinux 为工作目录。
|||
:---:|:---:
/usr/src/redhat/SPECS|这个目录当中放置的是该软件的配置档，例如这个软件的资讯参数、配置项目等等都放置在这里；
/usr/src/redhat/SOURCES|这个目录当中放置的是该软件的原始档 (*.tar.gz 的文件) 以及 config 这个配置档；
/usr/src/redhat/BUILD|在编译的过程中，有些缓存的数据都会放置在这个目录当中；
/usr/src/redhat/RPMS|经过编译之后，并且顺利的编译成功之后，将打包完成的文件放置在这个目录当中。里头有包含了 i386, i586, i686, noarch.... 等等的次目录。
/usr/src/redhat/SRPMS|与 RPMS 内相似的，这里放置的就是 SRPM 封装的文件。有时候你想要将你的软件用 SRPM 的方式释出时， 你的 SRPM 文件就会放置在这个目录中了。
此外，在编译的过程当中，可能会发生不明的错误，或者是配置的错误，这个时候就会在 /tmp 底下产生一个相对应的错误档，你可以根据该错误档进行除错的工作。等到所有的问题都解决之后，也编译成功了，那么刚刚解压缩之后的文件，就是在 /usr/src/redhat/SPECS, SOURCES, BUILD 等等的文件都会被杀掉，而只剩下放置在 /usr/src/redhat/RPMS 底下的文件了。  
由于 SRPM 需要重新编译，而编译的过程当中，我们至少需要有 make 与其相关的程序，及 gcc, c, c++ 等其他的编译用的程序语言来进行编译。
```
[root@www ~]# rpmbuild --rebuild rp-pppoe-3.5-32.1.src.rpm
正在安装 rp-pppoe-3.5-32.1.src.rpm
警告：使用者 mockbuild 不存在 - 现使用 root 代替
....(中间省略)....
已写入：/usr/src/redhat/RPMS/i386/rp-pppoe-3.5-32.1.i386.rpm
已写入：/usr/src/redhat/RPMS/i386/rp-pppoe-debuginfo-3.5-32.1.i386.rpm
正在运行 (%clean)：/bin/sh -e /var/tmp/rpm-tmp.69789
+ umask 022
+ cd /usr/src/redhat/BUILD
+ cd rp-pppoe-3.5
+ rm -rf /var/tmp/rp-pppoe-3.5-32.1-root
+ exit 0
正在运行 (--clean)：/bin/sh -e /var/tmp/rpm-tmp.69789
+ umask 022
+ cd /usr/src/redhat/BUILD
+ rm -rf rp-pppoe-3.5
+ exit 0

# 若一切正常，则会看到 exit 0 的字样，且会主动的删除 (rm) 很多中间缓存档。
```
## 配置档的主要内容 (*.spec)
除了使用 SRPM 内默认的参数来进行编译之外，我们还可以修改这些参数后再重新编译。首先我们必须要将 SRPM 内的文件安置到 /usr/src/redhat/ 内的相关目录，然后再去修改配置档即可。
```
[root@www ~]# rpm -i rp-pppoe-3.5-32.1.src.rpm
# 过程不会显示任何东西，他只会将 SRPM 的文件解开后，放置到 /usr/src/redhat/

[root@www ~]# find /usr/src/redhat/ -type f
/usr/src/redhat/SOURCES/rp-pppoe-3.5-firewall.patch  <==补丁档
/usr/src/redhat/SOURCES/adsl-stop                    <==CentOS 提供的脚本
/usr/src/redhat/SOURCES/rp-pppoe-3.5.tar.gz          <==原始码
/usr/src/redhat/SOURCES/rp-pppoe-3.5-buildroot.patch <==补丁档
/usr/src/redhat/SOURCES/adsl-start                   <==CentOS 提供的脚本
/usr/src/redhat/SOURCES/adsl-connect
/usr/src/redhat/SOURCES/adsl-setup
/usr/src/redhat/SOURCES/adsl-status
/usr/src/redhat/SOURCES/rp-pppoe-3.4-redhat.patch    <==补丁档
/usr/src/redhat/SPECS/rp-pppoe.spec                  <==重要配置档！
# 主要含有原始码与一个重要的配置档 rp-pppoe.spec
```
```
[root@www ~]# cd /usr/src/redhat/SPECS
[root@www SPECS]# vi rp-pppoe.spec
# 1. 首先，这个部分在介绍整个软件的基本相关资讯！不论是版本还是释出次数等。
Summary: A PPP over Ethernet client (for xDSL support).
Name: rp-pppoe
Version: 3.5
Release: 32.1
License: GPL
Group: System Environment/Daemons
Url: http://www.roaringpenguin.com/pppoe/
Source: http://www.roaringpenguin.com/rp-pppoe-%{version}.tar.gz
Source1: adsl-connect
Source2: adsl-setup
....(中间省略)....

# 2. 这部分则是在配置相依属性需求的地方！
BuildRoot: %{_tmppath}/%{name}-%{version}-%{release}-root

Prereq: /sbin/chkconfig  <==需要的前驱程序有哪些！
Prereq: /sbin/service
Prereq: fileutils

Requires: ppp >= 2.4.2         <==需要的软件又有哪些！
Requires: initscripts >= 5.92
Requires: iproute >= 2.6

BuildRequires: libtool   <==还需要哪些工具软件？
BuildRequires: autoconf
BuildRequires: automake

%description             <==此软件的描述
PPPoE (Point-to-Point Protocol over Ethernet) is a protocol used by
many ADSL Internet Service Providers. This package contains the
Roaring Penguin PPPoE client, a user-mode program that does not
require any kernel modifications. It is fully compliant with RFC 2516,
the official PPPoE specification.


# 3. 编译前的预处理，以及编译过程当中所需要进行的命令，都写在这里
#    尤其 %build 底下的数据，几乎就是 makefile 里面的资讯
%prep  <==这部份在预先 (pre) 进行处理，大致就是 patch 软件
%setup -q
%patch0 -p1 -b .config
%patch1 -p1 -b .buildroot
%patch2 -p1 -b .ipchains

%build <==这部分就是在实际编译
cd src
autoconf
CFLAGS="-D_GNU_SOURCE" %configure
make

install -m 0755 %{SOURCE1} scripts
install -m 0755 %{SOURCE2} scripts
install -m 0755 %{SOURCE3} scripts
install -m 0755 %{SOURCE4} scripts
install -m 0755 %{SOURCE5} scripts

%install  <==这就是安装过程
rm -rf %{buildroot}

mkdir -p %{buildroot}/sbin
make -C src install RPM_INSTALL_ROOT=%{buildroot}
....(中间省略)....

# 4. 这里列出，这个软件释出的文件有哪些的意思！
%files   <==这个软件提供的文件有哪些？需要记录在数据库内！
%defattr(-,root,root)
%doc doc/LICENSE scripts/adsl-connect scripts/adsl-setup scripts/adsl-init
%doc scripts/adsl-start scripts/adsl-status scripts/adsl-stop
%doc configs
%config(noreplace) %{_sysconfdir}/ppp/pppoe-server-options
%config(noreplace) %{_sysconfdir}/ppp/firewall*
/sbin/*
%{_sbindir}/*
%{_mandir}/man?/*

# 5. 列出这个软件的更改历史纪录档！
%changelog
* Wed Jul 12 2006 Jesse Keating <jkeating@redhat.com> - 3.5-32.1
- rebuild
....(中间省略)....
* Wed May 31 2000 Than Ngo <than@redhat.de>
- adopted for Winston.
```
.spec 文件的基本守则：
1. 整个文件的开头以Summary为开始，这部份的配置都是最基础的说明内容；
2. 然后每个不同的段落之间，都以%来做为开头，例如%prep与%install等；
### 系统整体资讯方面
<table>
<tbody><tr><th>参数</th><th>参数意义</th></tr>
<tr><td>Summary</td>
	<td>本软件的主要说明，例如上表中说明了本软件是针对 xDSL 的拨接用途</td></tr>
<tr><td>Name</td>
	<td>本软件的软件名称 (最终会是 RPM 文件的档名构成之一)</td></tr>
<tr><td>Version</td>
	<td>本软件的版本 (也会是 RPM 档名的构成之一)</td></tr>
<tr><td>Release</td>
	<td>这个是该版本打包的次数说明 (也会是 RPM 档名的构成之一)。</td></tr>
<tr><td>License</td>
	<td>这个软件的授权模式，我们是使用 GPL </td></tr>
<tr><td>Group</td>
	<td>这个软件的发展团体名称；</td></tr>
<tr><td>Url</td>
	<td>这个原始码的主要官方网站；</td></tr>
<tr><td>Source</td>
	<td>这个软件的来源，如果是网络上下载的软件，通常一定会有这个资讯来告诉大家这个原始档的来源。此外，还有来自开发商自己提供的原始档数据。例如上面的 adsl-start 等程序。</td></tr>
<tr><td>Patch</td>
	<td>就是作为补丁的 patch file </td></tr>
<tr><td>BuildRoot</td>
	<td>配置作为编译时，该使用哪个目录来缓存中间文件 (如编译过程的目标文件/连结文件等档)。</td></tr>
<tr><td>ExclusiveArch</td>
	<td>这个是说明这个软件的适合安装的硬件，通常默认为 i386，也可以调整为 i586 等等。由于我们的系统是新的 CPU 架构，这里我们修改内容成为『ExclusiveArch: i686』。</td></tr>
<tr><td colspan="2" align="center"><b>上述为必须要存在的项目，底下为可使用的额外配置值</b></td></tr>
<tr><td>Requires</td>
	<td>如果你这个软件还需要其他的软件的支持，那么这里就必需写上来，则当你制作成 RPM 之后，系统就会自动的去检查。这就是『相依属性』的主要来源</td></tr>
<tr><td>Prereq</td>
	<td>这个软件需要的前驱程序为何！这里指的是『程序』而 Requires 指的是『软件』！</td></tr>
<tr><td>BuildRequires</td>
	<td>编译过程中所需要的软件。Requires 指的是『安装时需要检查』的，因为与实际运行有关，这个 BuildRequires 指的是『编译时』所需要的软件，只有在 SRPM 编译成为 RPM 时才会检查的项目。</td></tr>
<tr><td>Packager</td>
	<td>这个软件是经由谁来打包的</td></tr>
<tr><td>Vender</td>
	<td>发展的厂商</td></tr>
</tbody></table>

根据上面的配置，最终的档名就会是『{Name}-{Version}-{Release}.{ExclusiveArch}.rpm』的样式。
### %description:
将软件做一个简短的说明。这个也是必需要的。使用『 rpm -qi 软件名称 』出现的一些基础的说明就是这些东西包括 Description 就是在显示这些重要资讯。这里记得要详加解释。
### %prep:
『尚未进行配置或安装之前，你要编译完成的 RPM	帮你事先做的事情』，prepare 的简写。工作事项主要有：
1. 进行软件的补丁 (patch) 等相关工作；
2. 寻找软件所需要的目录是否已经存在？确认用的！
3. 事先创建你的软件所需要的目录，或者事先需要进行的任务；
4. 如果待安装的 Linux 系统内已经有安装的时候可能会被覆盖掉的文件时，那么就必需要进行备份(backup)的工作了
### %setup:
这个项目就是在进行类似解压缩之类的工作，一定要写，不然 tarball 原始码是无法被解压缩。
### %build:
如何 make 编译成为可运行的程序。此部分的程序码方面，就是 ./configure, make 等项目
### %install:
编译完成 (build) 之后，要进行安装。安装就是写在这里，也就是类似 Tarball 里面的 make install 的意思。
### %clean:
编译与安装完毕后，必须要将一些缓存在 BuildRoot 内的数据删除。make clean 相似
### %files:
这个软件安装的文件都需要写到这里来，当然包括了『目录』。以备检查。此外，你也可以指定每个文件的类型，包括文件档 (%doc 后面接的) 与配置档 (%config 后面接的) 等等。
### %changelog:
这个项目主要则是在记录这个软件曾经的升级纪录。星号 (*) 后面应该要以时间，修改者， email 与软件版本来作为说明， 减号 (-) 后面则是要作的详细说明。
```
%changelog
* Wed Jul 01 2009 VBird Tsai <vbird@mail.vbird.idv.tw> - 3.5-32.2.vbird
- only rebuild this SRPM to RPM

* Wed Jul 12 2006 Jesse Keating <jkeating@redhat.com> - 3.5-32.1
....(底下省略)....
```
## SRPM 的编译命令 (-ba/-bb)
要将在 /usr/src/redhat 底下的数据编译或者是单纯的打包成为 RPM 或 SRPM 时，就需要 rpmbuild 命令与相关选项。
```
[root@www ~]# rpmbuild -ba rp-pppoe.spec  <==编译并同时产生 RPM 与 SRPM 文件
[root@www ~]# rpmbuild -bb rp-pppoe.spec  <==仅编译成 RPM 文件
```
这个时候系统就会这样做：
1. 先进入到 BUILD 这个目录中，亦即是： /usr/src/redhat/BUILD 这个目录；
2. 依照 *.spec 文件内的 Name 与 Version 定义出工作的目录名称，以我们上面的例子为例，那么系统就会在 BUILD 目录中先删除 rp-pppoe-3.5 的目录，再重新创建一个 rp-pppoe-3.5 的目录，并进入该目录；
3. 在新建的目录里面，针对 SOURCES 目录下的来源文件，也就是 *.spec 里面的 Source 配置的那个文件，以 tar 进行解压缩，以我们这个例子来说，则会在 /usr/src/redhat/BUILD/rp-pppoe-3.5 当中，将 /usr/src/redhat/SOURCES/rp-pppoe-3.5.tar.gz 进行解压缩
4. 再来开始 %build 及 %install 的配置与编译！
5. 最后将完成打包的文件给他放置到该放置的地方去，如果你的规定的硬件是在 i386 的系统，那么最后编译成功的 *.i386.rpm文件就会被放置在 /usr/src/redhat/RPMS/i386 里面。如果是 i686 就是 /usr/src/redhat/RPMS/i686 目录下。
## 一个打包自己软件的范例
### 制作原始码文件 tarball 产生：
```
[root@www ~]# mkdir /usr/local/src/main-0.1
[root@www ~]# tar -zxvf main.tgz -C /usr/local/src/main-0.1
[root@www ~]# cd /usr/local/src/main-0.1
[root@www main-0.1]# vim Makefile  <==创建原始码所需 make 守则
LIBS = -lm
OBJS = main.o haha.o sin_value.o cos_value.o
main: ${OBJS}
	gcc -o main ${OBJS} ${LIBS}
clean:
	rm -f main ${OBJS}
install:
	install -m 755 main $(RPM_INSTALL_ROOT)/usr/local/bin/main
# 记得 gcc 与 rm 之前是使用 <tab> 按键作出来的空白

[root@www main-0.1]# cd ..
[root@www src]# tar -zcvf main-0.1.tar.gz main-0.1
# 此时会产生 main-0.1.tar.gz ，将他挪到 /usr/src/redhat/SOURCES 底下：
[root@www src]# cp main-0.1.tar.gz /usr/src/redhat/SOURCES
```
### 创建 *.spec 的配置档
```
[root@www ~]# cd /usr/src/redhat/SPECS
[root@www SPECS]# vim main.spec
Summary:   calculate sin and cos value.
Name:      main
Version:   0.1
Release:   1
License:   GPL
Group:     VBird's Home
Source:    main-0.1.tar.gz   <==记得要写正确的 Tarball 档名
Url:       http://linux.vbird.org
Packager:  VBird
BuildRoot: %{_tmppath}/%{name}-%{version}-%{release}-root

%description
This package will let you input your name and calculate sin cos value.

%prep
%setup -q

%build
make

%install
rm -rf %{buildroot}
mkdir -p %{buildroot}/usr/local/bin
make install RPM_INSTALL_ROOT=%{buildroot}   <==这项目也很重要！

%files
/usr/local/bin/main

%changelog
* Wed Jul 01 2009 VBird Tsai <vbird@mail.vbird.idv.tw> 0.1
- build the program
```
### 编译成为 RPM 与 SRPM
```
[root@www SPECS]# rpmbuild -ba main.spec
....(前面省略)....
已写入：/usr/src/redhat/SRPMS/main-0.1-1.src.rpm
已写入：/usr/src/redhat/RPMS/i386/main-0.1-1.i386.rpm
已写入：/usr/src/redhat/RPMS/i386/main-debuginfo-0.1-1.i386.rpm
```
### 安装/测试/实际查询
```
[root@www ~]# rpm -ivh /usr/src/redhat/RPMS/i386/main-0.1-1.i386.rpm
正在准备…             ########################################### [100%]
   1:main                   ########################################### [100%]

[root@www ~]# rpm -ql main
/usr/local/bin/main   <==自己尝试运行 main 看看！

[root@www ~]# rpm -qi main
Name        : main              Relocations: (not relocatable)
Version     : 0.1                    Vendor: (none)
Release     : 1                  Build Date: 西元2009年07月02日 (周四)
Install Date: 西元2009年07月02日 Build Host: www.vbird.tsai
Group       : VBird's Home       Source RPM: main-0.1-1.src.rpm
Size        : 3360                  License: GPL
Signature   : (none)
Packager    : VBird
URL         : http://linux.vbird.org
Summary     : calculate sin and cos value.
Description :
This package will let you input your name and calculate sin cos value.
```

# YUM 线上升级机制
yum 是透过分析 RPM 的标头数据后， 根据各软件的相关性制作出属性相依时的解决方案，然后可以自动处理软件的相依属性问题，以解决软件安装或移除与升级的问题。  
由于 distribution 必须要先释出软件，然后将软件放置于 yum 服务器上面，以提供用户端来要求安装与升级之用的。 因此我们想要使用 yum 的功能时，必须要先找到适合的 yum server 才行。每个 yum server 可能都会提供许多不同的软件功能，那就是我们之前谈到的『容器』。因此，你必须要前往 yum server 查询到相关的容器网址后，再继续处理后续的配置事宜。  
事实上 CentOS 在释出软件时已经制作出多部映射站台 (mirror site) 提供全世界的软件升级之用。 所以，理论上我们不需要处理任何配置值，只要能够连上 Internet ，就可以使用 yum 。
## 利用 yum 进行查询、安装、升级与移除功能
### 查询功能：yum [list|info|search|provides|whatprovides] 参数
```
[root@www ~]# yum [option] [查询工作项目] [相关参数]
选项与参数：
[option]：主要的选项，包括有：
  -y ：当 yum 要等待使用者输入时，这个选项可以自动提供 yes 的回应；
  --installroot=/some/path ：将该软件安装在 /some/path 而不使用默认路径
[查询工作项目] [相关参数]：这方面的参数有：
  search  ：搜寻某个软件名称或者是描述 (description) 的重要关键字；
  list    ：列出目前 yum 所管理的所有的软件名称与版本，有点类似 rpm -qa；
  info    ：同上，不过有点类似 rpm -qai 的运行结果；
  provides：从文件去搜寻软件！类似 rpm -qf 的功能！

[root@www ~]# yum search raid
....(前面省略)....
mdadm.i386 : mdadm controls Linux md devices (software RAID arrays)
lvm2.i386 : Userland logical volume management tools
....(后面省略)....
# 在冒号 (:)  左边的是软件名称，右边的则是在 RPM 内的 name 配置 (软件名)

[root@www ~]# yum info mdadm
Installed Packages      <==这说明该软件是已经安装的了
Name   : mdadm          <==这个软件的名称
Arch   : i386           <==这个软件的编译架构
Version: 2.6.4          <==此软件的版本
Release: 1.el5          <==释出的版本
Size   : 1.7 M          <==此软件的文件总容量
Repo   : installed      <==容器回报说已安装的
Summary: mdadm controls Linux md devices (software RAID arrays)
Description:            <==与 rpm -qi 相同
mdadm is used to create, manage, and monitor Linux MD (software RAID)
devices.  As such, it provides similar functionality to the raidtools
package.  However, mdadm is a single program, and it can perform
almost all functions without a configuration file, though a configuration
file can be used to help with some common tasks.

列出 yum 服务器上面提供的所有软件名称
[root@www ~]# yum list
Installed Packages <==已安装软件
Deployment_Guide-en-US.noarch            5.2-9.el5.centos       installed
Deployment_Guide-zh-CN.noarch            5.2-9.el5.centos       installed
Deployment_Guide-zh-TW.noarch            5.2-9.el5.centos       installed
....(中间省略)....
Available Packages <==还可以安装的其他软件
Cluster_Administration-as-IN.noarch      5.2-1.el5.centos       base
Cluster_Administration-bn-IN.noarch      5.2-1.el5.centos       base
# 上面提供的意义为：『 软件名称   版本   在那个容器内 』

列出目前服务器上可供本机进行升级的软件有哪些？
[root@www ~]# yum list updates  <==一定要是 updates
Updated Packages
Deployment_Guide-en-US.noarch            5.2-11.el5.centos      base
Deployment_Guide-zh-CN.noarch            5.2-11.el5.centos      base
Deployment_Guide-zh-TW.noarch            5.2-11.el5.centos      base
....(底下省略)....
# 上面就列出在那个容器内可以提供升级的软件与版本！

[root@www ~]# yum list pam*

列出提供 passwd 这个文件的软件有哪些
[root@www ~]# yum provides passwd
passwd.i386 : The passwd utility for setting/changing passwords using PAM
passwd.i386 : The passwd utility for setting/changing passwords using PAM
```
### 安装/升级功能：yum [install|update] 软件
```
[root@www ~]# yum [option] [查询工作项目] [相关参数]
选项与参数：
  install ：后面接要安装的软件！
  update  ：后面接要升级的软件，若要整个系统都升级，就直接 update 即可
```
### 移除功能：yum [remove] 软件
## yum 的配置档
当你要找容器所在网址时， 最重要的就是该网址底下一定要有个名为 repodata 的目录存在，那就是容器的网址了，因为该目录就是分析 RPM 软件后所产生的软件属性相依数据放置处。
```
[root@www ~]# vi /etc/yum.repos.d/CentOS-Base.repo
[base]
name=CentOS-$releasever - Base
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os
#baseurl=http://mirror.centos.org/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-5
```
* [base]：代表容器的名字。中括号一定要存在，里面的名称则可以随意取。但是不能有两个相同的容器名称， 否则 yum 会无法分辨该到哪里去找容器相关软件清单文件。
* name：只是说明一下这个容器的意义而已，重要性不高！
* mirrorlist=：列出这个容器可以使用的映射站台，如果不想使用，可以注解到这行；
* baseurl=：这个最重要，因为后面接的就是容器的实际网址！ mirrorlist 是由 yum 程序自行去捉映射站台， baseurl 则是指定固定的一个容器网址！我们刚刚找到的网址放到这里来。
* enable=1：就是让这个容器被启动。如果不想启动可以使用 enable=0 
* gpgcheck=1：RPM 的数码签章。这就是指定是否需要查阅 RPM 文件内的数码签章。
* gpgkey=：就是数码签章的公钥档所在位置！使用默认值即可
```
列出目前 yum server 所使用的容器有哪些？
[root@www ~]# yum repolist all
repo id         repo name                  status
addons          CentOS-5 - Addons          enabled
base            CentOS-5 - Base            enabled
c5-media        CentOS-5 - Media           disabled
centosplus      CentOS-5 - Plus            disabled
extras          CentOS-5 - Extras          enabled
updates         CentOS-5 - Updates         enabled
# 上面最右边有写 enabled 才是有启动的！由于 /etc/yum.repos.d/
# 有多个配置档，所以你会发现还有其他的容器存在。
```
### 修改容器产生的问题与解决办法
由于我们是修改系统默认的配置档，事实上，我们应该要在 /etc/yum.repos.d/ 底下新建一个文件， 该扩展名必须是 .repo 才行。但因为我们使用的是指定特定的映射站台，而不是其他软件开发生提供的容器， 因此才修改系统默认配置档。但是可能由于使用的容器版本有新旧之分， yum 会先下载容器的清单到本机的 /var/cache/yum ，修改了网址却没有修改容器名称 (中刮号内的文字)， 可能就会造成本机的清单与 yum 服务器的清单不同步，此时就会出现无法升级的问题。  
通过 yum 的 clean 项目，清除本机上的旧数据即可：
```
[root@www ~]# yum clean [packages|headers|all] 
选项与参数：
 packages：将已下载的软件文件删除
 headers ：将下载的软件档头删除
 all     ：将所有容器数据都删除！
```
## yum 的软件群组功能
安装大型专案。
```
[root@www ~]# yum [群组功能] [软件群组]
选项与参数：
   grouplist   ：列出所有可使用的『套件组』，例如 Development Tools 之类的；
   groupinfo   ：后面接 group_name，则可了解该 group 内含的所有套件名；
   groupinstall：可以安装一整组的套件群组
   groupremove ：移除某个套件群组；

查阅目前容器与本机上面的可用与安装过的软件群组有哪些？
[root@www ~]# yum grouplist
Installed Groups:
   Office/Productivity
   Editors
   System Tools
....(中间省略)....
Available Groups:
   Tomboy
   Cluster Storage
   Engineering and Scientific
....(以下省略)....
```
系统上面的软件大多是群组的方式一口气来提供安装的。全新安装 CentOS 时，提供的可选择需要的软件，就是软件群组。
```
[root@www ~]# yum groupinfo XFCE-4.4
Setting up Group Process

Group: XFCE-4.4
 Description: This group contains the XFCE desktop environment.
 Mandatory Packages:
   xfce4-session
....(中间省略)....
 Default Packages:
   xfce4-websearch-plugin
....(中间省略)....
 Optional Packages:
   xfce-mcs-manager-devel
   xfce4-panel-devel
....(以下省略)....
```
这就是一个壁纸环境 (desktop environment) ，也就是一个窗口管理员。底下列出主要的与选择性 (optional) 的软件名称。
```
[root@www ~]# yum groupinstall XFCE-4.4
```
## 全系统自动升级
通过『 yum -y update 』来自动升级，-y 表示可以自动回答 yes 来开始下载和安装。然后再透过 crontab 的功能来处理即可。
```

[root@www ~]# vim /etc/crontab
....(前面省略并保留配置值)....
0  3 * * * root /usr/bin/yum -y update
```
如果升级的是核心软件 (kernel)，还是得要重新启动才会让安装的软件顺利运行。所以还是要分析登陆档，若有新核心安装，就重新启动，否则就让系统自动维持再最新较安全的环境。

# 管理的抉择：RPM 还是 Tarball
1. 优先选择原厂的 RPM 功能：  
   由于原厂释出的软件通常具有一段时间的维护期，举例来说， RHEL 与 CentOS 每一个版本至少提供五年以上的升级期限。这对于我们的系统安全性来说，是非常好的选项。 既然 yum 可以自动升级，加上原厂会持续维护软件升级，那么我们的系统就能够自己保持在软件最新的状态， 对于安全来说当然会比较好一些的。此外，由于 RPM 与 yum 具有容易安装/移除/升级等特点，且还提供查询与验证的功能，安装时更有数码签章的保护， 让你的软件管理变的更轻松！因此，当然首选就是利用 RPM 来处理。
2. 选择软件官网释出的 RPM 或者是提供的容器网址：  
   某些特殊软件原版厂商并不会提供，举例来说 CentOS 就没有提供 NTFS 的相关模块。此时你可以自行到官网去查阅，看看有没有提供相对到你的系统的 RPM 文件， 如果有提供容器网址，那就更好。可以修改 yum 配置档来加入该容器，就能够自动安装与升级该软件。
3. 利用 Tarball 安装特殊软件：  
   某些特殊用途的软件并不会特别帮你制作 RPM 文件的，此时建议你也不要妄想自行制作 SRPM 来转成 RPM 。因为你只有区区一部主机而已，若是你要管理相同的 100 部主机，那么将原始码转制作成 RPM 就有价值！ 单机版的特殊软件，例如学术网络常会用到的 MPICH/PVM 等平行运算函式库，这种软件建议使用 tarball 来安装即可， 不需要特别去搜寻 RPM 。
4. 用 Tarball 测试新版软件：  
   某些时刻你可能需要使用到新版的某个软件，但是原版厂商仅提供旧版软件，举例来说，我们的 CentOS 主要是定位于企业版，因此很多软件的要求是『稳』而不是『新』。 然担心新软件装好后产生问题，回不到旧软件，此时你可以用 tarball 安装新软件到 /usr/local 底下， 那么该软件就能够同时安装两个版本在系统上面了。而且大多数软件安装数种版本时还不会互相干扰。 

如果有 RPM 的话，那么优先权还是在于 RPM 安装上面，毕竟管理上比较便利，但是如果软件的架构差异性太大， 或者是无法解决相依属性的问题，那么与其花大把的时间与精力在解决属性相依的问题上，还不如直接以 tarball 来安装。