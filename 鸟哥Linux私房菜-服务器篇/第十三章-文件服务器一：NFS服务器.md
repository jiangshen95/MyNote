# NFS 的由来与其功能
NFS 是藉由网络分享文件系统的服务。最大的问题在于『权限』方面的概念，因为在客户端与服务器端可能必须要具备相同的账号才能够存取某些目录或档案。 另外，NFS 的启动需要透过所谓的远程过程调用 (RPC)，也就是说，我们并不是只要启动 NFS 就好了， 还需要启动 RPC 这个服务才行。
## 什么是 NFS (Network FileSystem)
NFS 就是 Network FileSystem 的缩写，最早之前是由 Sun 这家公司所发展出来的。 它最大的功能就是可以透过网络，让不同的机器、不同的操作系统、可以彼此分享个别的档案 (share files)。所以，可以简单的将他看做是一个文件服务器 (file server) 。这个 NFS 服务器可以让你的 PC 来将网络远程的 NFS 服务器分享的目录，挂载到本地端的机器当中， 在本地端的机器看起来，那个远程主机的目录就好像是自己的一个磁盘分区槽一样 (partition)。使用上面相当的便利。  
当我们的 NFS 服务器设定好了分享出来的 /home/sharefile 这个目录后，其他的 NFS 客户端就可以将这个目录挂载到自己系统上面的某个挂载点 (挂载点可以自定义)。基本上 NFS 这个服务的埠口开在 2049 ，但是由于文件系统非常复杂，因此 NFS 还有其他的程序去启动额外的端口。预设 NFS 用来传输的埠口是随机选择小于 1024 以下的埠口来使用的。此时就得要 远程过程调用 (Remote Procedure Call, RPC) 的协定来辅助，客户端获取服务器使用的端口。
## 什么是 RPC (Remote Procedure Call)
因为 NFS 支持的功能相当的多，而不同的功能都会使用不同的程序来启动， 每启动一个功能就会启用一些端口来传输数据，因此， NFS 的功能所对应的端口才没有固定住， 而是随机取用一些未被使用的小于 1024 的埠口来作为传输之用。  
RPC 最主要的功能就是在指定每个 NFS 功能所对应的 port number ，并且回报给客户端，让客户端可以连结到正确的埠口上去。当服务器在启动 NFS 时会随机取用数个埠口，并主动的向 RPC 注册，因此 RPC 可以知道每个埠口对应的 NFS 功能，然后 RPC 又是固定使用 port 111 来监听客户端的需求并回报客户端正确的埠口。
> 要启动 NFS 之前，RPC 就要先启动了，否则 NFS 会无法向 RPC 注册。 另外，RPC 若重新启动时，原本注册的数据会不见，因此 RPC 重新启动后，它管理的所有服务都需要重新启动来重新向 RPC 注册。

当客户端有 NFS 档案存取需求时，向服务器端要求数据：
1. 客户端会向服务器端的 RPC (port 111) 发出 NFS 档案存取功能的询问要求；
2. 服务器端找到对应的已注册的 NFS daemon 埠口后，会回报给客户端；
3. 客户端了解正确的埠口后，就可以直接与 NFS daemon 来联机。

由于 NFS 的各项功能都必须要向 RPC 来注册，如此一来 RPC 才能了解 NFS 这个服务的各项功能之 port number, PID, NFS 在服务器所监听的 IP 等等，而客户端才能够透过 RPC 的询问找到正确对应的埠口。 也就是说，NFS 必须要有 RPC 存在时才能成功的提供服务，因此我们称 NFS 为 RPC server 的一种。事实上，有很多这样的服务器都是向 RPC 注册的，举例来说，NIS (Network Information Service) 也是 RPC server 的一种。不论是客户端还是服务器端，要使用 NFS 时，两者都需要启动 RPC 才行。
## NFS 启动的 RPC daemons
NFS 服务器主要的任务是进行文件系统的分享，文件系统的分享则与权限有关。 所以 NFS 服务器启动时至少需要两个 daemons ，一个管理客户端是否能够登入的问题， 一个管理客户端能够取得的权限。如果你还想要管理 quota 的话，那么 NFS 还得要再加载其他的 RPC 程序。以较单纯的 NFS 服务器来说：
* rpc.nfsd：  
  最主要的 NFS 服务器服务提供商。这个 daemon 主要的功能就是在管理客户端是否能够使用服务器文件系统挂载信息等， 其中还包含这个登入者的 ID 的判别
* rpc.mountd  
  这个 daemon 主要的功能，则是在管理 NFS 的文件系统。当客户端顺利的通过 rpc.nfsd 而登入服务器之后，在他可以使用 NFS 服务器提供的档案之前，还会经过档案权限 (就是那个 -rwxrwxrwx 与 owner, group 那几个权限) 的认证程序。他会去读 NFS 的配置文件 /etc/exports 来比对客户端的权限，当通过这一关之后客户端就可以取得使用 NFS 档案的权限了。(注：这个也是我们用来管理 NFS 分享之目录的权限与安全设定的地方)
* rpc.lockd (非必要)  
  可以用在管理档案的锁定 (lock) 用途。 因为既然分享的 NFS 档案可以让客户端使用，那么当多个客户端同时尝试写入某个档案时， 就可能对于该档案造成一些问题。这个 rpc.lockd 则可以用来克服这个问题。 但 rpc.lockd 必须要同时在客户端与服务器端都开启才行，此外， rpc.lockd 也常与 rpc.statd 同时启用。
* rpc.statd (非必要)  
  可以用来检查档案的一致性，与 rpc.lockd 有关。若发生因为客户端同时使用同一档案造成档案可能有所损毁时， rpc.statd 可以用来检测并尝试回复该档案。与 rpc.lockd 同样的，这个功能必须要在服务器端与客户端都启动才会生效。

上述这几个 RPC 所需要的程序，其实都已经写入到两个基本的服务启动脚本中了，那就是 nfs 以及 nfslock 。亦即是在 /etc/init.d/nfs, /etc/init.d/nfslock，与服务器较有关的写入在 nfs 服务中，而与客户端的 rpc.lockd 之类的，就设定于 nfslock 服务中。
## NFS 的档案访问权限
NFS 本身的服务并没有进行身份登入的识别， 所以说，当你在客户端以 dmtsai 的身份想要存取服务器端的文件系统时， 服务器端会以客户端的使用者 UID 与 GID 等身份来尝试读取服务器端的文件系统。  
当我以 dmtsai 这个一般身份使用者要去存取来自服务器端的档案时，你要先注意到的是： 文件系统的 inode 所记录的属性为 UID, GID 而非账号与群组名。 那一般 Linux 主机会主动的以自己的 /etc/passwd, /etc/group 来查询对应的使用者、组名。 所以当 dmtsai 进入到该目录后，会参照 NFS client 1 的使用者与组名。 但是由于该目录的档案主要来自 NFS server ，所以可能就会发现几个情况：
* NFS server/NFS client 刚好有相同的账号与群组  
  则此时使用者可以直接以 dmtsai 的身份进行服务器所提供的文件系统之存取。
* NFS server 的 501 这个 UID 账号对应为 vbird  
  若 NFS 服务器上的 /etc/passwd 里面 UID 501 的使用者名称为 vbird 时， 则客户端的 dmtsai 可以存取服务器端的 vbird 这个使用者的档案。只因为两者具有相同的 UID 而已。这就造成很大的问题了！因为没有人可以保证客户端的 UID 所对应的账号会与服务器端相同， 那服务器所提供的数据就可能会被错误的使用者乱改。
* NFS server 并没有 501 这个 UID  
  另一个极端的情况是，在服务器端并没有 501 这个 UID 的存在，则此时 dmtsai 的身份在该目录下会被压缩成匿名者， 一般 NFS 的匿名者会以 UID 为 65534 为其使用者，早期的 Linux distributions 这个 65534 的账号名称通常是 nobody ，我们的 CentOS 则取名为 nfsnobody 。但有时也会有特殊的情况，例如在服务器端分享 /tmp 的情况下， dmtsain 的身份还是会保持 501 但建立的各项数据在服务器端来看，就会属于无拥有者的资料。
* 如果使用者身份是 root 时  
  在预设的情况下， root 的身份会被主动的压缩成为匿名者。

客户端使用者能做的事情是与 UID 及其 GID 有关的，那当客户端与服务器端的 UID 及账号的对应不一致时， 可能就会造成文件系统使用上的困扰，这个就是 NFS 文件系统在使用上面的一个很重要的地方。在了解使用者账号与 UID 及文件系统的关系之后，要实际在客户端以 NFS 取用服务器端的文件系统时， 你还得需要具有：
* NFS 服务器有开放可写入的权限 (与 /etc/exports 设定有关)；
* 实际的档案权限具有可写入 (w) 的权限。

当你满足了 (1)使用者账号，亦即 UID 的相关身份； (2)NFS 服务器允许有写入的权限； (3)文件系统确实具有 w 的权限时，你才具有该档案的可写入权限。NFS 通常需要与 NIS (十四章) 这一个可以确认客户端与服务器端身份一致的服务搭配使用，以避免身份的错乱。

# NFS Server 端的设定
## 所需要的软件
以 CentOS 6.x 为例的话，要设定好 NFS 服务器我们必须要有两个软件才行，分别是：
* RPC 主程序：rpcbind  
  NFS 其实可以被视为一个 RPC 服务，而要启动任何一个 RPC 服务之前，我们都需要做好 port 的对应 (mapping) 的工作才行，这个工作其实就是『 rpcbind 』这个服务所负责的。也就是说， 在启动任何一个 RPC 服务之前，我们都需要启动 rpcbind 才行。 (在 CentOS 5.x 以前这个软件称为 portmap，在 CentOS 6.x 之后才称为 rpcbind 的！)
* NFS 主程序：nfs-utils  
  就是提供 rpc.nfsd 及 rpc.mountd 这两个 NFS daemons 与其他相关 documents 与说明文件、执行文件等的软件！这个就是 NFS 服务所需要的主要软件。

## NFS 的软件结构
* 主要配置文件：/etc/exports  
  系统并没有默认值，所以这个档案『 不一定会存在』，你可能必须要使用 vim 主动的建立起这个档案。
* NFS 文件系统维护指令：/usr/sbin/exportfs  
  这个是维护 NFS 分享资源的指令，我们可以利用这个指令重新分享 /etc/exports 变更的目录资源、将 NFS Server 分享的目录卸除或重新分享等等，这个指令是 NFS 系统里面相当重要的一个。
* 分享资源的登录档：/var/lib/nfs/*tab  
  在 NFS 服务器的登录文件都放置到 /var/lib/nfs/ 目录里面，在该目录下有两个比较重要的登录档， 一个是 etab ，主要记录了 NFS 所分享出来的目录的完整权限设定值；另一个 xtab 则记录曾经链接到此 NFS 服务器的相关客户端数据。
* 客户端查询服务器分享资源的指令：/usr/sbin/showmount  
  这是另一个重要的 NFS 指令。exportfs 是用在 NFS Server 端，而 showmount 则主要用在 Client 端。这个 showmount 可以用来察看 NFS 分享出来的目录资源
## /etc/exports 配置文件的语法与参数
NFS 会直接使用到核心功能，所以你的核心必须要有支持 NFS 才行。  
NFS 服务器的架设，只要编辑好主要配置文件 /etc/exports 之后，先启动 rpcbind (若已经启动了，就不要重新启动)，然后再启动 nfs ，你的 NFS 就成功了。不过这样的设定能否对客户端生效，那就得要考虑你权限方面的设定能力了。
```
[root@www ~]# vim /etc/exports
/tmp         192.168.100.0/24(ro)   localhost(rw)   *.ev.ncku.edu.tw(ro,sync)
[分享目录]   [第一部主机(权限)]     [可用主机名]    [可用通配符]
```
每一行最前面是要分享出来的目录，注意是以目录为单位。 然后这个目录可以依照不同的权限分享给不同的主机，上面的例子说明是： 要将 /tmp 分别分享给三个不同的主机或网域的意思。记得主机后面以小括号 () 设计权限参数， 若权限参数不止一个时，则以逗号 (,) 分开。且主机名与小括号是连在一起的。在这个档案内也可以利用 # 来批注。  
主机名的设定主要有几个方式：  
* 可以使用完整的 IP 或者是网域，例如 192.168.100.10 或 192.168.100.0/24 ，或 192.168.100.0/255.255.255.0 都可以接受！
* 也可以使用主机名，但这个主机名必须要在 /etc/hosts 内，或可使用 DNS 找到该名称才行。反正重点是可找到 IP 就是了。如果是主机名的话，那么他可以支持通配符，例如 * 或 ? 均可接受。

至于权限方面 (就是小括号内的参数) 常见的参数则有：
参数值|内容说明
:---:|:----:
rw<br>ro|该目录分享的权限是可擦写 (read-write) 或只读 (read-only)，但最终能不能读写，还是与文件系统的 rwx 及身份有关。
sync<br>async|sync 代表数据会同步写入到内存与硬盘中，async 则代表数据会先暂存于内存当中，而非直接写入硬盘
no_root_squash<br>root_squash|客户端使用 NFS 文件系统的账号若为 root 时，系统该如何判断这个账号的身份？预设的情况下，客户端 root 的身份会由 root_squash 的设定压缩成 nfsnobody， 如此对服务器的系统会较有保障。但如果你想要开放客户端使用 root 身份来操作服务器的文件系统，那么这里就得要开 no_root_squash 才行！
all_squash|论登入 NFS 的使用者身份为何， 他的身份都会被压缩成为匿名用户，通常也就是 nobody(nfsnobody)
anonuid<br>anongid|anon 意指 anonymous (匿名者) 前面关于 *_squash 提到的匿名用户的 UID 设定值，通常为 nobody(nfsnobody)，但是你可以自行设定这个 UID 的值！当然，这个 UID 必需要存在于你的 /etc/passwd 当中！ anonuid 指的是 UID 而 anongid 则是群组的 GID 。
* 案例一：让 root 保有 root 的权限  
  我想将 /tmp 分享出去给大家使用，由于这个目录本来就是大家都可以读写的，因此想让所有的人都可以存取。此外，我要让 root 写入的档案还是具有 root 的权限
  ```
  [root@www ~]# vim /etc/exports
  # 任何人都可以用我的 /tmp ，用通配符来处理主机名，重点在 no_root_squash
  /tmp  *(rw,no_root_squash)
  ```
  主机名可以使用通配符，上头表示无论来自哪里都可以使用我的 /tmp 这个目录。 再次提醒，『 *(rw,no_root_squash) 』这一串设定值中间是没有空格符的。而 /tmp 与 *(rw,no_root_squash) 则是有空格符来隔开的！特别注意到 no_root_squash 的功能。在这个例子中，如果你是客户端，而且你是以 root 的身份登入你的 Linux 主机，那么当你 mount 上我这部主机的 /tmp 之后，你在该 mount 的目录当中，将具有『root 的权限』
* 案例二：同一目录针对不同范围开放不同权限  
  我要将一个公共的目录 /home/public 公开出去，但是只有限定我的局域网络 192.168.100.0/24 这个网域且加入 vbirdgroup 的用户才能够读写，其他来源则只能读取。
  ```
  [root@www ~]# mkdir /home/public
  [root@www ~]# setfacl -m g:vbirdgroup:rwx /home/public
  [root@www ~]# vim /etc/exports
  /tmp          *(rw,no_root_squash)
  /home/public  192.168.100.0/24(rw)    *(ro)
  # 继续累加在后面，将主机与网域分为两段 (用空白隔开)
  ```
  上面的例子说的是，当我的 IP 是在 192.168.100.0/24 这个网段的时候，那么当我在 Client 端挂载了 Server 端的 /home/public 后，针对这个被我挂载的目录我就具有可以读写的权限。至于如果我不是在这个网段之内，那么这个目录的数据我就仅能读取而已，亦即为只读的属性。  
  需要注意的是，通配符仅能用在主机名的分辨上面，IP 或网段就只能用 192.168.100.0/24 的状况， 不可以使用 192.168.100.* 。
* 案例三：仅给某个单一主机使用的目录设定  
  我要将一个私人的目录 /home/test 开放给 192.168.100.10 这个 Client 端的机器来使用时，假设使用者的身份是 dmtsai 才具有完整的权限
  ```
  [root@www ~]# mkdir /home/test
  [root@www ~]# setfacl -m u:dmtsai:rwx /home/test
  [root@www ~]# vim /etc/exports
  /tmp          *(rw,no_root_squash)
  /home/public  192.168.100.0/24(rw)    *(ro)
  /home/test    192.168.100.10(rw)
  # 只要设定 IP 正确即可
  ```
  只有 192.168.100.10 这部机器才能对 /home/test 这个目录进行存取
* 案例四：开放匿名登录的情况  
  我要让 *.centos.vbird 网域的主机，登入我的 NFS 主机时，可以存取 /home/linux ，但是他们存数据的时候，我希望他们的 UID 与 GID 都变成 45 这个身份的使用者，假设我 NFS 服务器上的 UID 45 与 GID 45 的用户/组名为 nfsanon。
  ```
  [root@www ~]# groupadd -g 45 nfsanon
  [root@www ~]# useradd -u 45 -g nfsanon nfsanon
  [root@www ~]# mkdir /home/linux
  [root@www ~]# setfacl -m u:nfsanon:rwx /home/linux
  [root@www ~]# vim /etc/exports
  /tmp          *(rw,no_root_squash)
  /home/public  192.168.100.0/24(rw)    *(ro)
  /home/test    192.168.100.10(rw)
  /home/linux   *.centos.vbird(rw,all_squash,anonuid=45,anongid=45)
  # 如果要开放匿名，那么重点是 all_squash，并且要配合 anonuid
  ```
  特别注意到 all_squash 与 anonuid, anongid 的功能。如此一来，当 clientlinux.centos.vbird 登入这部 NFS 主机，并且在 /home/linux 写入档案时，该档案的所有人与所有群组，就会变成 /etc/passwd 里面对应的 UID 为 45 的那个身份的使用者了。
### 客户端与服务器具有相同的 UID 与账号：
假设我在 192.168.100.10 登入这部 NFS (IP 假设为 192.168.100.254) 服务器，并且我在 192.168.100.10 的账号为 dmtsai 这个身份，同时，在这部 NFS 上面也有 dmtsai 这个账号， 并具有相同的 UID ，果真如此的话，那么：
1. 由于 192.168.100.254 这部 NFS 服务器的 /tmp 权限为 -rwxrwxrwt ，所以我 (dmtsai 在 192.168.100.10 上面) 在 /tmp 底下具有存取的权限，并且写入的档案所有人为 dmtsai ；
2. 在 /home/public 当中，由于我有读写的权限，所以如果在 /home/public 这个目录的权限对于 dmtsai 有开放写入的话，那么我就可以读写，并且我写入的档案所有人是 dmtsai 。但是万一 /home/public 对于 dmtsai 这个使用者并没有开放可以写入的权限时， 那么我还是没有办法写入档案
3. 在 /home/test 当中，我的权限与 /home/public 相同的状态！还需要 NFS 服务器的 /home/test 对于 dmtsai 有开放权限；
4. 在 /home/linux 当中就比较麻烦！因为不论你是何种 user ，你的身份一定会被变成 UID=45 这个账号！所以，这个目录就必需要针对 UID = 45 的那个账号名称，修改他的权限才行！
### 客户端与服务器端的账号并未相同时：
假如我在 192.168.100.10 的身份为 vbird (uid 为 600)，但是 192.168.100.254 这部 NFS 主机却没有 uid=600 的账号时，情况会变成：
1. 我在 /tmp 底下还是可以写入，只是该档案的权限会保持为 UID=600 ，因此服务器端看起来就会比较奇怪， 因为找不到 UID=600 这个账号的显示，故档案拥有者会填上 600 。
2. 我在 /home/public 里面是否可以写入，还需要视 /home/public 的权限而定，不过，由于没有加上 all_squash 的参数， 因此在该目录下会保留客户端的使用者 UID，同上一点所示。
3. /home/test 的观点与 /home/public 相同
4. /home/linux 底下，我的身份就被变成 UID = 45 那个使用者了。
### 当客户端的身份为 root 时：
1. 我在 /tmp 里面可以写入，并且由于 no_root_squash 的参数，改变了预设的 root_squash 设定值，所以在 /tmp 写入的档案所有人为 root
2. 我在 /home/public 底下的身份还是被压缩成为 nobody 了。因为默认属性里面都具有 root_squash ，所以，如果 /home/public 有针对 nobody 开放写入权限时，那么我就可以写入，但是档案所有人变成 nobody 了。
3. /home/test 与 /home/public 相同；
4. /home/linux 的情况中，我 root 的身份也被压缩成为 UID = 45 的那个使用者了。
## 启动 NFS
```
[root@www ~]# /etc/init.d/rpcbind start
# 如果 rpcbind 本来就已经在执行了，那就不需要启动

[root@www ~]# /etc/init.d/nfs start
# 有时候某些 distributions 可能会出现如下的警告讯息：
exportfs: /etc/exports [3]: No 'sync' or 'async' option specified 
for export "192.168.100.10:/home/test".
  Assuming default behaviour ('sync').
# 上面的警告讯息仅是在告知因为我们没有指定 sync 或 async 的参数，
# 则 NFS 将默认会使用 sync 的信息而已。你可以不理他，也可以加入 /etc/exports。

[root@www ~]# /etc/init.d/nfslock start
[root@www ~]# chkconfig rpcbind on
[root@www ~]# chkconfig nfs on
[root@www ~]# chkconfig nfslock on
```
rpcbind 其实不需要设定，只要直接启动它即可。启动之后，会出现一个 port 111 的 sunrpc 的服务，那就是 rpcbind 。至于 nfs 则会启动至少两个以上的 daemon 出现，然后就开始在监听 Client 端的需求。要注意屏幕上的输出信息，因为如果配置文件写错的话，屏幕上会显示出错误的地方。  
此外，如果你想要增加一些 NFS 服务器的数据一致性功能时，可能需要用到 rpc.lockd 及 rpc.statd 等 RPC 服务，或许要增加 nfslock 服务。启动之后，到 /var/log/messages 里面看看有没有被正确的启动：
```
[root@www ~]# tail /var/log/messages
Jul 27 17:10:39 www kernel: Installing knfsd (copyright (C) 1996 okir@monad.swb.de).
Jul 27 17:10:54 www kernel: NFSD: Using /var/lib/nfs/v4recovery as the NFSv4 state 
recovery directory
Jul 27 17:10:54 www kernel: NFSD: starting 90-second grace period
Jul 27 17:11:32 www rpc.statd[3689]: Version 1.2.2 starting
```
NFS 开了那些埠口：
```
[root@www ~]# netstat -tulnp| grep -E '(rpc|nfs)'
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address  Foreign Address  State   PID/Program name
tcp        0      0 0.0.0.0:875    0.0.0.0:*        LISTEN  3631/rpc.rquotad
tcp        0      0 0.0.0.0:111    0.0.0.0:*        LISTEN  3601/rpcbind
tcp        0      0 0.0.0.0:48470  0.0.0.0:*        LISTEN  3647/rpc.mountd
tcp        0      0 0.0.0.0:59967  0.0.0.0:*        LISTEN  3689/rpc.statd
tcp        0      0 0.0.0.0:2049   0.0.0.0:*        LISTEN  -
udp        0      0 0.0.0.0:875    0.0.0.0:*                3631/rpc.rquotad
udp        0      0 0.0.0.0:111    0.0.0.0:*                3601/rpcbind
udp        0      0 0.0.0.0:897    0.0.0.0:*                3689/rpc.statd
udp        0      0 0.0.0.0:46611  0.0.0.0:*                3647/rpc.mountd
udp        0      0 0.0.0.0:808    0.0.0.0:*                3601/rpcbind
udp        0      0 0.0.0.0:46011  0.0.0.0:*                3689/rpc.statd
```
主要的埠口是：
* rpcbind 启动的 port 在 111 ，同时启动在 UDP 与 TCP；
* nfs 本身的服务启动在 port 2049 上
* 其他 rpc.* 服务启动的 port 则是随机产生的，因此需向 port 111 注册。

使用 rpcinfo 观察每个 RPC 服务的注册情况
```
[root@www ~]# rpcinfo -p [IP|hostname]
[root@www ~]# rpcinfo -t|-u  IP|hostname 程序名称
选项与参数：
-p ：针对某 IP (未写则预设为本机) 显示出所有的 port 与 porgram 的信息；
-t ：针对某主机的某支程序检查其 TCP 封包所在的软件版本；
-u ：针对某主机的某支程序检查其 UDP 封包所在的软件版本；

# 1. 显示出目前这部主机的 RPC 状态
[root@www ~]# rpcinfo -p localhost
   program vers proto   port  service
    100000    4   tcp    111  portmapper
    100000    3   tcp    111  portmapper
    100000    2   tcp    111  portmapper
    100000    4   udp    111  portmapper
    100000    3   udp    111  portmapper
    100000    2   udp    111  portmapper
    100011    1   udp    875  rquotad
    100011    2   udp    875  rquotad
    100011    1   tcp    875  rquotad
    100011    2   tcp    875  rquotad
    100003    2   tcp   2049  nfs
....(底下省略)....
# 程序代号 NFS版本 封包类型 埠口  服务名称

# 2. 针对 nfs 这个程序检查其相关的软件版本信息 (仅察看 TCP 封包)
[root@www ~]# rpcinfo -t localhost nfs
program 100003 version 2 ready and waiting
program 100003 version 3 ready and waiting
program 100003 version 4 ready and waiting
# 可发现提供 nfs 的版本共有三种，分别是 2, 3, 4 版，新版的 NFS 传输速度较快
```
如果 rpcinfo 无法输出，那就表示注册的数据有问题，可能需要重新启动 rpcbind 与 nfs 。
## NFS 的联机观察
```
[root@www ~]# showmount [-ae] [hostname|IP]
选项与参数：
-a ：显示目前主机与客户端的 NFS 联机分享的状态；
-e ：显示某部主机的 /etc/exports 所分享的目录数据。

# 1. 请显示出刚刚我们所设定好的相关 exports 分享目录信息
[root@www ~]# showmount -e localhost
Export list for localhost:
/tmp         *
/home/linux  *.centos.vbird
/home/test   192.168.100.10
/home/public (everyone)
```
当你要扫瞄某一部主机提供的 NFS 分享的目录时，就使用 showmount -e IP (或hostname) 即可，这也是 NFS client 端最常用的指令。另外， NFS 关于目录权限设定的数据非常之多。在 /etc/exports 只是比较特别的权限参数而已，还有很多预设参数，这些预设参数的位置，可以检查 /var/lib/nfs/etab 。
```
[root@www ~]# tail /var/lib/nfs/etab
/home/public    192.168.100.0/24(rw,sync,wdelay,hide,nocrossmnt,secure,root_squash,
no_all_squash,no_subtree_check,secure_locks,acl,anonuid=65534,anongid=65534)
# 上面是同一行，可以看出除了 rw, sync, root_squash 等等，
# 其实还有 anonuid 及 anongid 等等的设定！
```
另外，如果有其他客户端挂载了你的 NFS 文件系统时，那么该客户端与文件系统信息就会被记录到 /var/lib/nfs/xtab 里。  
要重新处理 /etc/exports 档案，当重新设定完 /etc/exports 后可以不要重启 nfs ，使用 exportfs 这个指令：
```
[root@www ~]# exportfs [-aruv]
选项与参数：
-a ：全部挂载(或卸除) /etc/exports 档案内的设定
-r ：重新挂载 /etc/exports 里面的设定，此外，亦同步更新 /etc/exports
     及 /var/lib/nfs/xtab 的内容
-u ：卸除某一目录
-v ：在 export 的时候，将分享的目录显示到屏幕上

# 1. 重新挂载一次 /etc/exports 的设定
[root@www ~]# exportfs -arv
exporting 192.168.100.10:/home/test
exporting 192.168.100.0/24:/home/public
exporting *.centos.vbird:/home/linux
exporting *:/home/public
exporting *:/tmp

# 2. 将已经分享的 NFS 目录资源，通通都卸除
[root@www ~]# exportfs -auv
# 这时如果你再使用 showmount -e localhost 就会看不到任何资源了！
```
如果你仅有处理配置文件，但并没有相对应的目录 (/home/public 等目录) 可以提供使用，可能会出现一些警告讯息。所以记得要建立分享的目录。
## NFS 的安全性
### 防火墙的设定问题与解决方案
一般来说， NFS 的服务仅会对内部网域开放，不会对因特网开放。因为除了固定的 port 111, 2049 之外， 还有很多不固定的埠口是由 rpc.mountd, rpc.rquotad 等服务所开启，所以防火墙规则比较难以设定。  
为了解决这个问题， CentOS 6.x 有提供一个固定特定 NFS 服务的埠口配置文件，那就是 /etc/sysconfig/nfs 。在这个档案里面就能够指定特定的埠口，这样每次启动 nfs 时，相关服务启动的埠口就会固定。这个配置文件内容很多，绝大部分的数据都不要更改，只要改跟 PORT 这个关键词有关的数据即可。 需要更改的 rpc 服务主要有 mountd, rquotad, nlockmgr 这三个，所以你应该要这样改：
```
[root@www ~]# vim /etc/sysconfig/nfs
RQUOTAD_PORT=1001   <==约在 13 行左右
LOCKD_TCPPORT=30001 <==约在 21 行左右
LOCKD_UDPPORT=30001 <==约在 23 行左右
MOUNTD_PORT=1002    <==约在 41 行左右
# 记得设定值最左边的批注服务要拿掉之外，埠口的值你也可以自行决定。

[root@www ~]# /etc/init.d/nfs restart
[root@www ~]# rpcinfo -p | grep -E '(rquota|mount|nlock)'
    100011    2   udp   1001  rquotad
    100011    2   tcp   1001  rquotad
    100021    4   udp  30001  nlockmgr
    100021    4   tcp  30001  nlockmgr
    100005    3   udp   1002  mountd
    100005    3   tcp   1002  mountd
```
放行 NFS 的防火墙规则设置：
```
[root@www ~]# vim /usr/local/virus/iptables/iptables.allow
iptables -A INPUT -i $EXTIF -p tcp -s 120.114.140.0/24 -m multiport \
         --dport 111,2049,1001,1002,30001 -j ACCEPT
iptables -A INPUT -i $EXTIF -p udp -s 120.114.140.0/24 -m multiport \
         --dport 111,2049,1001,1002,30001 -j ACCEPT

[root@www ~]# /usr/local/virus/iptables/iptables.rule
```
### 使用 /etc/exports 设定安全的权限：
善用 root_squash 及 all_squash 等功能，再利用 anonuid 等等的设定来规范登入你主机的用户身份。
### 更安全的 partition 规则
如果你的工作环境中，具有多部的 Linux 主机，并且预计彼此分享出目录时，那么在安装 Linux 的时候，最好就可以规划出一块 partition 作为预留之用。因为『 NFS 可以针对目录来分享』，因此，你可以将预留的 partition 挂载在任何一个挂载点，再将该挂载点 (就是目录) 由 /etc/exports 的设定中分享出去，那么整个工作环境中的其他 Linux 主机就可以使用该 NFS 服务器的那块预留的 partition 了。所以，在主机的规划上面，主要需要留意的只有 partition 而已。此外，由于分享的 partition 可能较容易被入侵，最好可以针对该 partition 设定比较严格的参数在 /etc/fstab 当中。
### NFS 服务器关机前的注意事项
由于 NFS 使用的这个 RPC 服务，当客户端连上服务器时，如果你的服务器上面还有客户端在联机，那么你要关机， 可能得要等到数个钟头才能够正常的关机成功。  
建议你的 NFS Server 想要关机之前，能先『关掉 rpcbind 与 nfs 』这两个服务。如果无法正确的将这两个 daemons 关掉，那么先以 netstat -utlp 找出 PID ，然后以 kill 将他关掉。这样才有办法正常的关机成功。  
也可以利用 showmount -a localhost 来查出来那个客户端还在联机，或者是查阅 /var/lib/nfs/rmtab 或 xtab 等档案来检查亦可。

# NFS 客户端的设定
## 手动挂载 NFS 服务器分享的资源
1. 确认本地端已经启动了 rpcbind 服务！
2. 扫瞄 NFS 服务器分享的目录有哪些，并了解我们是否可以使用 (showmount)；
3. 在本地端建立预计要挂载的挂载点目录 (mkdir)；
4. 利用 mount 将远程主机直接挂载到相关目录。
```
# 1. 启动必备的服务：若没有启动才启动，有启动则保持原样不动。
[root@clientlinux ~]# /etc/init.d/rpcbind start
[root@clientlinux ~]# /etc/init.d/nfslock start
# 一般来说，系统默认会启动 rpcbind
# 另外，如果服务器端有启动 nfslock 的话，客户端也要启动才能生效

# 2. 查询服务器提供哪些资源给我们使用
[root@clientlinux ~]# showmount -e 192.168.100.254
Export list for 192.168.100.254:
/tmp         *
/home/linux  *.centos.vbird
/home/test   192.168.100.10
/home/public (everyone)   <==这是等一下我们要挂载的目录

# 3. 建立挂载点，并且挂载
[root@clientlinux ~]# mkdir -p /home/nfs/public
[root@clientlinux ~]# mount -t nfs 192.168.100.254:/home/public \
> /home/nfs/public
# 注意一下挂载的语法『 -t nfs 』指定文件系统类型，
# IP:/dir 则是指定某一部主机的某个提供的目录，另外，如果出现如下错误：
mount: 192.168.100.254:/home/public failed, reason given by server: No such file 
or directory
# 这代表你在 Server 上面并没有建立 /home/public 

# 4. 总是得要看看挂载之后的情况如何，可以使用 df 或 mount 
[root@clientlinux ~]# df
文件系统               1K-区段      已用     可用 已用% 挂载点
....(中间省略)....
192.168.100.254:/home/public
                       7104640    143104   6607104   3% /home/nfs/public
```
以后，只要你进入你的目录 /home/nfs/public 就等于到了 192.168.100.254 那部远程主机的 /home/public 那个目录中。使用 umount 卸除挂载的 NFS 目录。
```
[root@clientlinux ~]# umount /home/nfs/public
```
## 客户端可处理的挂载参数与开机挂载
mount 指令的参数
参数|参数代表意义|系统默认值
:---:|:---:|:---:
suid<br>nosuid|如果挂载的 partition 上面有任何 SUID 的 binary 程序时， 你只要使用 nosuid 就能够取消 SUID 的功能了。|suid
rw<br>ro|你可以指定该文件系统是只读 (ro) 或可擦写。服务器可以提供给你可擦写， 但是客户端可以仅允许只读的参数设定值|rw
dev<br>nodev|是否可以保留装置档案的特殊功能？一般来说只有 /dev 这个目录才会有特殊的装置，因此你可以选择 nodev|dev
exec<br>noexec|是否具有执行 binary file 的权限？ 如果你想要挂载的仅是数据区 (例如 /home)，那么可以选择 noexec|exec
user<br>nouser|是否允许使用者进行档案的挂载与卸除功能？ 如果要保护文件系统，最好不要提供使用者进行挂载与卸除|nouser
auto<br>noauto|这个 auto 指的是『mount -a』时，会不会被挂载的项目。 如果你不需要这个 partition 随时被挂载，可以设定为 noauto。|auto
一般来说，如果你的 NFS 服务器所提供的只是类似 /home 底下的个人资料， 应该不需要可执行、SUID 与装置档案，因此当你在挂载的时候，可以这样下达指令：
```
[root@clientlinux ~]# umount /home/nfs/public
[root@clientlinux ~]# mount -t nfs -o nosuid,noexec,nodev,rw \
> 192.168.100.254:/home/public /home/nfs/public

[root@clientlinux ~]# mount | grep addr
192.168.100.254:/home/public on /home/nfs/public type nfs (rw,noexec,nosuid,
nodev,vers=4,addr=192.168.100.254,clientaddr=192.168.100.10)
```
这样一来你所挂载的这个文件系统就只能作为资料存取之用，相对来说，对于客户端是比较安全一些的。
### 关于 NFS 特殊的挂载参数
除了上述的 mount 参数之外，其实针对 NFS 服务器，Linux 还提供不少有用的额外参数。
参数|参数功能|预设参数
:---:|:---:|:---:
fg<br>bg|当执行挂载时，该挂载的行为会在前景 (fg) 还是在背景 (bg) 执行？ 若在前景执行时，则 mount 会持续尝试挂载，直到成功或 time out 为止，若为背景执行， 则 mount 会在背景持续多次进行 mount ，而不会影响到前景的程序操作。 如果你的网络联机有点不稳定，或是服务器常常需要开关机，那建议使用 bg 比较妥当。|fg
soft<br>hard|如果是 hard 的情况，则当两者之间有任何一部主机脱机，则 RPC 会持续的呼叫，直到对方恢复联机为止。如果是 soft 的话，那 RPC 会在 time out 后『重复』呼叫，而非『持续』呼叫， 因此系统的延迟会比较不这么明显。同上，如果你的服务器可能开开关关，建议用 soft|hard
intr|当你使用上头提到的 hard 方式挂载时，若加上 intr 这个参数， 则当 RPC 持续呼叫中，该次的呼叫是可以被中断的 (interrupted)。|没有
rsize<br>wsize|读出(rsize)与写入(wsize)的区块大小 (block size)。 这个设定值可以影响客户端与服务器端传输数据的缓冲记忆容量。一般来说， 如果在局域网络内 (LAN) ，并且客户端与服务器端都具有足够的内存，那这个值可以设定大一点， 比如说 32768 (bytes) 等，提升缓冲记忆区块将可提升 NFS 文件系统的传输能力。但要注意设定的值也不要太大，最好是达到网络能够传输的最大值为限。|rsize=1024<br>wsize=1024
通常如果你的 NFS 是用在高速运作的环境当中的话，那么可以建议加上这些参数：
```
[root@clientlinux ~]# umount /home/nfs/public
[root@clientlinux ~]# mount -t nfs -o nosuid,noexec,nodev,rw \
> -o bg,soft,rsize=32768,wsize=32768 \
> 192.168.100.254:/home/public /home/nfs/public
```
> 某些大型的模式运算并不允许 soft 这个参数
### 将 NFS 开机即挂载
NFS 不能写入 /etc/fstab 当中，因为网络的启动是在本机挂载之后，因此当你利用 /etc/fstab 尝试挂载 NFS 时，系统由于尚未启动网络，所以无法挂载成功。写入 /etc/rc.d/rc.local:
```
[root@clientlinux ~]# vim /etc/rc.d/rc.local
mount -t nfs -o nosuid,noexec,nodev,rw,bg,soft,rsize=32768,wsize=32768 \
192.168.100.254:/home/public /home/nfs/public
```
## 无法挂载的原因分析
### 客户端的主机名或 IP 网段不被允许使用：
比如， /home/test 只能提供 192.168.100.0/24 这个网域，所以如果我在 192.168.100.254 这部服务器中，以 localhost (127.0.0.1) 来挂载时，就会无法挂载上。
### 服务器或客户端某些服务未启动：
比如忘记了启动 rpcbind 这个服务，如果你在客户端发现 mount 的讯息是这样：
```
[root@clientlinux ~]# mount -t nfs 192.168.100.254:/home/test /mnt
mount: mount to NFS server '192.168.100.254' failed: System Error: Connection refused.
# 如果你使用 ping 却发现网络与服务器都是好的，那么这个问题就是 rpcbind 没有开

[root@clientlinux ~]# mount -t nfs 192.168.100.254:/home/test /home/nfs
mount: mount to NFS server '192.168.100.254' failed: RPC Error: Program not registered.
# 注意看最后面的数据，确实有连上 RPC ，但是服务器的 RPC 告知我们，该程序无注册
```
### 被防火墙挡掉了：
## 自动挂载 autofs 的使用
在一般 NFS 文件系统的使用情况中，如果客户端要使用服务器端所提供的 NFS 文件系统时，在 /etc/rc.d/rc.local 当中设定开机时挂载，或登入系统后手动利用 mount 来挂载。 此外，客户端得要预先手动的建立好挂载点目录，然后挂载上来。
### NFS 文件系统与网络联机的困扰
* 可不可以让客户端在有使用到 NFS 文件系统的需求时才让系统自动挂载？
* 当 NFS 文件系统使用完毕后，可不可以让 NFS 自动卸除，以避免可能的 RPC 错误？
### autofs 的设定概念：
autofs 这个服务在客户端计算机上面，会持续的侦测某个指定的目录， 并预先设定当使用到该目录下的某个次目录时，将会取得来自服务器端的 NFS 文件系统资源，并进行自动挂载的动作。  
autofs 主要配置文件为 /etc/auto.master，这个档案的内容很简单，我只要定义出最上层目录 (/home/nfsfile) 即可，这个目录就是 autofs 会一直持续侦测的目录。后续的档案则是该目录底下各次目录的对应。在 /etc/auto.nfs (这个档案的档名可自定义) 里面则可以定义出每个次目录所欲挂载的远程服务器的 NFS 目录资源。  
举例来说：『当我们在客户端要使用 /home/nfsfile/public 的数据时，此时 autofs 才会去 192.168.100.254 服务器上挂载 /home/public 』且『当隔了 5 分钟没有使用该目录下的数据后，则客户端系统将会主动的卸除 /home/nfsfile/public 』。
### 建立主配置文件 /etc/auto.master，并指定侦测的特定目录
这个主要配置文件的内容很简单，只要有要被持续侦测的目录及『数据对应文件』即可。 那个数据对应文件的文件名是可以自行设定的：
```
[root@clientlinux ~]# vim /etc/auto.master
/home/nfsfile  /etc/auto.nfs
```
/home/nfsfile 目录不需要存在，因为 autofs 会主动的建立该目录。如果你建立了，可能反而会出问题。
### 建立数据对应文件内 (/etc/auto.nfs) 的挂载信息与服务器对应资源
```
[本地端次目录]  [-挂载参数]  [服务器所提供的目录]
选项与参数：
[本地端次目录] ：指的就是在 /etc/auto.master 内指定的目录之次目录
[-挂载参数]    ：就是前一小节提到的 rw,bg,soft 等等的参数，可有可无；
[服务器所提供的目录] ：例如 192.168.100.254:/home/public 等

[root@clientlinux ~]# vim /etc/auto.nfs
public   -rw,bg,soft,rsize=32768,wsize=32768  192.168.100.254:/home/public
testing  -rw,bg,soft,rsize=32768,wsize=32768  192.168.100.254:/home/test
temp     -rw,bg,soft,rsize=32768,wsize=32768  192.168.100.254:/tmp
# 参数部分，只要最前面加个 - 符号即可！
```
那些 /home/nfsfile/public 是不需要事先建立的，autofs 会视情况处理。
### 实际运作与观察
启动 autofs
```
[root@clientlinux ~]# /etc/init.d/autofs stop
[root@clientlinux ~]# /etc/init.d/autofs start
```
要进入 /home/nfsfile/public 时：
```
[root@clientlinux ~]# ll -d /home/nfsfile
drwxr-xr-x. 2 root root 0 2011-07-28 00:07 /home/nfsfile
# /home/nfsfile 容量是 0 ，因为是 autofs 建立的

[root@clientlinux ~]# cd /home/nfsfile/public
[root@clientlinux public]# mount | grep nfsfile
192.168.100.254:/home/public on /home/nfsfile/public type nfs (rw,soft,rsize=32768,
wsize=32768,sloppy,vers=4,addr=192.168.100.254,clientaddr=192.168.100.10)
# 自动挂载

[root@clientlinux public]# df  /home/nfsfile/public
文件系统               1K-区段      已用     可用 已用% 挂载点
192.168.100.254:/home/public
                       7104640    143104   6607040   3% /home/nfsfile/public
# 档案的挂载也出现没错
```

# 案例演练
### 模拟的环境状态中，服务器端的想法如下：
假设服务器的 IP 为 192.168.100.254 这一部；
1. /tmp 分享为可擦写，并且不限制使用者身份的方式，分享给所有 192.168.100.0/24 这个网域中的所有计算机；
2. /home/nfs 分享的属性为只读，可提供除了网域内的工作站外，向 Internet 亦提供数据内容；
3. /home/upload 做为 192.168.100.0/24 这个网域的数据上传目录，其中，这个 /home/upload 的使用者及所属群组为 nfs-upload 这个名字，他的 UID 与 GID 均为 210；
4. /home/andy 这个目录仅分享给 192.168.100.10 这部主机，以提供该主机上面 andy 这个使用者来使用，也就是说， andy 在 192.168.100.10 及 192.168.100.254 均有账号，且账号均为 andy ，所以预计开放 /home/andy 给 andy 使用他的家目录
### 服务器端设定的实地演练：
```
[root@www ~]# vim /etc/exports
/tmp         192.168.100.0/24(rw,no_root_squash)
/home/nfs    192.168.100.0/24(ro)  *(ro,all_squash)
/home/upload 192.168.100.0/24(rw,all_squash,anonuid=210,anongid=210)
/home/andy   192.168.100.10(rw)

# 1. /tmp
[root@www ~]# ll -d /tmp
drwxrwxrwt. 12 root root 4096 2011-07-27 23:49 /tmp

# 2. /home/nfs
[root@www ~]# mkdir -p /home/nfs
[root@www ~]# chmod 755 -R /home/nfs
# 修改较为严格的档案权限将目录与档案设定成只读！不能写入的状态，会更保险一点！

# 3. /home/upload
[root@www ~]# groupadd -g 210 nfs-upload
[root@www ~]# useradd -g 210 -u 210 -M nfs-upload
# 先建立对应的账号与组名及 UID 
[root@www ~]# mkdir -p /home/upload
[root@www ~]# chown -R nfs-upload:nfs-upload /home/upload
# 修改拥有者！如此，则用户与目录的权限都设定妥当

# 4. /home/andy
[root@www ~]# useradd andy
[root@www ~]# ll -d /home/andy
drwx------. 4 andy andy 4096 2011-07-28 00:15 /home/andy

[root@www ~]# /etc/init.d/nfs restart

# 1. 确认远程服务器的可用目录：
[root@clientlinux ~]# showmount -e 192.168.100.254
Export list for 192.168.100.254:
/home/andy   192.168.100.10
/home/upload 192.168.100.0/24
/home/nfs    (everyone)
/tmp         192.168.100.0/24

# 2. 建立挂载点：
[root@clientlinux ~]# mkdir -p /mnt/{tmp,nfs,upload,andy}

# 3. 实际挂载：
[root@clientlinux ~]# mount -t nfs 192.168.100.254:/tmp         /mnt/tmp
[root@clientlinux ~]# mount -t nfs 192.168.100.254:/home/nfs    /mnt/nfs
[root@clientlinux ~]# mount -t nfs 192.168.100.254:/home/upload /mnt/upload
[root@clientlinux ~]# mount -t nfs 192.168.100.254:/home/andy   /mnt/andy
```