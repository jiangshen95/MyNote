# 远程联机服务器
## 什么是远程联机服务器
远程联机服务器在提供你由远程透过文字或图形接口的方式来登入系统， 让你在远程的工作机前面登入 Linux 主机以取得可操控主机之接口 (shell)，而登入后的操作感觉上就像坐在系统前面一样。
### 联机服务器的功能作用之一：分享 Unix Like 主机的运算能力
### 服务器类型 (Server)：有限度的开放联机
在一般对因特网开放服务的服务器中，由于开放的服务可能会有较为重要的信息，而远程联机程序连进主机之后， 可以进行的工作又太多了(几乎就像在主机前面工作一般)，因此服务器的远程联机程序通常仅针对少部分系统维护者开放。除非必要，否则 Server 类型的主机不建议开放联机的服务。
### 工作站类型 (Workstation)：只对内网开放
所谓的工作站就是不提供因特网服务的主机，仅提供大量的运算能力给使用者。
## 有哪些可供登入的类型
以登入的联机界面来分类，基本上有文字接口与图形接口两种：
* 文字接口明码： telnet, rsh 等为主，目前非常少用；
* 文字接口密码： ssh 为主，已经取代上述的 telnet, rsh 等明码方式；
* 图形接口： Xdmcp, VNC, RDP 等较为常见

图形接口的联机服务器，比较简单的有 Xdmcp (X Display Manager Control Protocol)，架设 Xdmcp 很简单， 不过客户端的软件比较少。另外一款目前很常见的图形联机服务器，就是 VNC (Virtual Network Computing)， 透过 VNC server/client 软件来进行连接。如果你想要使用类似 Windows 的远程桌面联机，该功能使用的是 RDP (Remote Desktop Protocol)，那你可得要架设 RDP 服务器才行。
### 数据传送的明码与密码
所谓的明码就是： 『当我们的数据封包在网络上传输时，该数据封包的内容为数据的原始格式』。  
由于明码传输的 telnet, rsh 等联机服务器已经被 ssh 取代，并且在一些实际应用上已经很少看到 telnet 与 rsh 了。

# 文字接口联机服务器：SSH 服务器
SSH 是 Secure SHell protocol 的简写 (安全的壳程序协议)，它可以透过数据封包加密技术，将等待传输的封包加密后再传输到网络上。SSH 可以用来取代较不安全的 finger, R Shell (rcp, rlogin, rsh 等), talk 及 telnet 等联机模式。  
这个 SSH 协议，在预设的状态中，本身就提供两个服务器功能：
1. 一个就是类似 telnet 的远程联机使用 shell 的服务器，亦即是俗称的 ssh ；
2. 另一个就是类似 FTP 服务的 sftp-server ！提供更安全的 FTP 服务。
## 联机加密技术简介
目前常见的网络封包加密技术通常是藉由所谓的『非对称密钥系统』来处理的。 主要是透过两把不一样的公钥与私钥 (Public and Private Key) 来进行加密与解密的过程。由于这两把钥匙是提供加解密的功用， 所以在同一个方向的联机中，这两把钥匙当然是需要成对的！它的功用分别如下：
* 公钥 (public key)：提供给远程主机进行数据加密的行为，也就是说，大家都能取得你的公钥来将数据加密的意思；
* 私钥 (private key)：远程主机使用你的公钥加密的数据，在本地端就能够使用私钥来进行解密。由于私钥是这么的重要， 因此私钥是不能够外流的！只能保护在自己的主机上。

由于每部主机都应该有自己的密钥 (公钥与私钥)，且公钥用来加密而私钥用来解密， 其中私钥不可外流。但因为网络联机是双向的，所以，每个人应该都要有对方的『公钥』。  
站在客户端的角度来看，那么，首先你必须要取得服务器端的公钥，然后将自己的公钥发送给服务器端， 最终在客户端上面的密钥会是『服务器的公钥加上客户端我自己的私钥』来组成的。
> 目前在 SSH 使用上，主要是利用 RSA/DSA/Diffie-Hellman 等机制。

目前 SSH 的协议版本有两种，分别是 version 1 与 version 2 ，其中 V2 由于加上了联机检测的机制， 可以避免联机期间被插入恶意的攻击码，因此比 V1 还要更加的安全。
### SSH 的联机行为简介
1. 服务器建立公钥档： 每一次启动 sshd 服务时，该服务会主动去找 /etc/ssh/ssh_host* 的档案，若系统刚刚安装完成时，由于没有这些公钥档案，因此 sshd 会主动去计算出这些需要的公钥档案，同时也会计算出服务器自己需要的私钥档；
2. 客户端主动联机要求： 若客户端想要联机到 ssh 服务器，则需要使用适当的客户端程序来联机，包括 ssh, pietty 等客户端程序；
3. 服务器传送公钥档给客户端： 接收到客户端的要求后，服务器便将第一个步骤取得的公钥档案传送给客户端使用 (此时应是明码传送，公钥本来就是给大家使用的)；
4. 客户端记录/比对服务器的公钥数据及随机计算自己的公私钥： 若客户端第一次连接到此服务器，则会将服务器的公钥数据记录到客户端的用户家目录内的 ~/.ssh/known_hosts 。若是已经记录过该服务器的公钥数据，则客户端会去比对此次接收到的与之前的记录是否有差异。若接受此公钥数据， 则开始计算客户端自己的公私钥数据；
5. 回传客户端的公钥数据到服务器端： 用户将自己的公钥传送给服务器。此时服务器：『具有服务器的私钥与客户端的公钥』，而客户端则是： 『具有服务器的公钥以及客户端自己的私钥』，你会看到，在此次联机的服务器与客户端的密钥系统 (公钥+私钥) 并不一样，所以才称为非对称式密钥系统；
6. 开始双向加解密： (1)服务器到客户端：服务器传送数据时，拿用户的公钥加密后送出。客户端接收后，用自己的私钥解密； (2)客户端到服务器：客户端传送数据时，拿服务器的公钥加密后送出。服务器接收后，用服务器的私钥解密。

在上述的第 4 步骤中，客户端的密钥是随机运算产生于本次联机当中的，所以你这次的联机与下次的联机的密钥可能就会不一样。此外在客户端的用户家目录下的 ~/.ssh/known_hosts 会记录曾经联机过的主机的 public key ，用以确认我们是连接上正确的那部服务器。  
产生新的服务器端的 ssh 公钥与服务器自己使用的成对私钥：
```
[root@www ~]# rm /etc/ssh/ssh_host*  <==删除密钥档
[root@www ~]# /etc/init.d/sshd restart
正在停止 sshd:                         [  确定  ]
正在产生 SSH1 RSA 主机密钥:            [  确定  ]
正在产生 SSH2 RSA 主机密钥:            [  确定  ]
正在产生 SSH2 DSA 主机密钥:            [  确定  ]
正在激活 sshd:                         [  确定  ]
[root@www ~]# date; ll /etc/ssh/ssh_host*
Mon Jul 25 11:36:12 CST 2011
-rw-------. 1 root root  668 Jul 25 11:35 /etc/ssh/ssh_host_dsa_key
-rw-r--r--. 1 root root  590 Jul 25 11:35 /etc/ssh/ssh_host_dsa_key.pub
-rw-------. 1 root root  963 Jul 25 11:35 /etc/ssh/ssh_host_key
-rw-r--r--. 1 root root  627 Jul 25 11:35 /etc/ssh/ssh_host_key.pub
-rw-------. 1 root root 1675 Jul 25 11:35 /etc/ssh/ssh_host_rsa_key
-rw-r--r--. 1 root root  382 Jul 25 11:35 /etc/ssh/ssh_host_rsa_key.pub
# 看一下上面输出的日期与档案的建立时间，刚刚建立的新公钥、私钥系统！
```
## 启动 SSH 服务
Linux 系统当中，默认就已经含有 SSH 的所有需要的软件了，这包含了可以产生密码等协议的 OpenSSL 软件与 OpenSSH 软件。在目前的 Linux Distributions 当中，都是预设启动 SSH 的。直接启动就是以 SSH daemon ，简称为 sshd 来启动的，所以，手动可以这样启动：
```
[root@www ~]# /etc/init.d/sshd restart
[root@www ~]# netstat -tlnp | grep ssh
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address  Foreign Address  State   PID/Program name
tcp        0      0 :::22          :::*             LISTEN  1539/sshd
```
需要注意的是，SSH 不但提供了 shell 给我们使用，亦即是 ssh protocol 的主要目的，同时亦提供了一个较为安全的 FTP server ，亦即是 ssh-ftp server 给我们当成是 FTP 来使用。所以，这个 sshd 可以同时提供 shell 与 ftp ，而且都是架构在 port 22 上面的。
## ssh 客户端联机程序 - Linux 用户
### ssh：直接登入远程主机的指令
SSH 在 client 端使用的是 ssh 这个指令，这个指令可以指定联机的版本 (version1, version2)， 还可以指定非正规的 ssh port (正规 ssh port 为 22)。不过，一般的用法可以使用底下的方式：
```
[root@www ~]# ssh [-f] [-o 参数项目] [-p 非正规埠口] [账号@]IP [指令]
选项与参数：
-f ：需要配合后面的 [指令] ，不登入远程主机直接发送一个指令过去而已；
-o 参数项目：主要的参数项目有：
	ConnectTimeout=秒数 ：联机等待的秒数，减少等待的时间
	StrictHostKeyChecking=[yes|no|ask]：预设是 ask，若要让 public key
           主动加入 known_hosts ，则可以设定为 no 即可。
-p ：如果你的 sshd 服务启动在非正规的埠口 (22)，需使用此项目；
[指令] ：不登入远程主机，直接发送指令过去。但与 -f 意义不太相同。

# 1. 直接联机登入到对方主机的方法 (以登入本机为例)：
[root@www ~]# ssh 127.0.0.1
The authenticity of host '127.0.0.1 (127.0.0.1)' can't be established.
RSA key fingerprint is eb:12:07:84:b9:3b:3f:e4:ad:ba:f1:85:41:fc:18:3b.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '127.0.0.1' (RSA) to the list of known hosts.
root@127.0.0.1's password: <==在这里输入 root 的密码即可！
Last login: Mon Jul 25 11:36:06 2011 from 192.168.1.101
[root@www ~]# exit  <==离开这次的 ssh 联机
# 由于 ssh 后面没有加上账号，因此预设使用当前的账号来登入远程服务器
```
一般使用 ssh 登入远程主机，都会填写『 ssh 账号@主机IP 』的格式， 意思是说，使用该主机的某账号登入的意思。如果不写账号的话，亦即使用『 ssh 主机IP 』，那么会以本地端计算机的账号来尝试登入远程。 也就是说，如果近端与远程具有相同的账号，那么不写账号也没有关系。  
上面出现的讯息中，开头 RSA 的那行后面接的就是远程服务器的公钥指纹码，如果确定该指纹码没有问题，那么你就得要输入 yes 来将该指纹码写入服务器公钥记录文件 (~/.ssh/known_hosts)，以方便未来比对该服务器的正确性之用。注意是要写 yes ，单纯输入 Y 或 y 是不会被接受的。此外， 由于该主机的公钥已经被记录，因此未来重复使用 ssh 登入此主机时，就不会出现这个指纹码提示了。
```
# 2. 使用 student 账号登入本机
[root@www ~]# ssh student@127.0.0.1
student@127.0.0.1's password:
[student@www ~]$ exit
# 由于加入账号，因此切换身份成为 student 了！另外，因为 127.0.0.1 曾登入过，
# 所以就不会再出现提示你要增加主机公钥的讯息

# 3. 登入对方主机执行过指令后立刻离开的方式：
[root@www ~]# ssh student@127.0.0.1 find / &> ~/find1.log
student@localhost's password:
# 此时你会发现怎么画面卡住了？这是因为上头的指令会造成，你已经登入远程主机，
# 但是执行的指令尚未跑完，因此你会在等待当中。那如何指定系统自己跑？

# 4. 与上题相同，但是让对方主机自己跑该指令，你立刻回到近端主机继续工作：
[root@www ~]# ssh -f student@127.0.0.1 find / &> ~/find1.log
# 此时你会立刻注销 127.0.0.1 ，但 find 指令会自己在远程服务器跑
```
第四个范例，比较有用。如果你想要让远程主机进行关机的指令，如果不加上 -f 的参数， 那你会等待对方主机关机完毕再将你踢出联机。因此，加上 -f 就很重要。
```
# 5. 删除掉 known_hosts 后，重新使用 root 联机到本机，且自动加上公钥记录
[root@www ~]# rm ~/.ssh/known_hosts
[root@www ~]# ssh -o StrictHostKeyChecking=no root@localhost
Warning: Permanently added 'localhost' (RSA) to the list of known hosts.
root@localhost's password:
# 如上所示，不会问你 yes 或 no 啦！直接会写入 ~/.ssh/known_hosts 当中！
```
加上这个 StrictHostKeyChecking=no ，他会不询问自动加入主机的公钥到档案中，对于一般使用者帮助不大，对于程序脚本来说比较有用。
### 服务器公钥记录文件：~/.ssh/known_hosts
当你登入远程服务器时，本机会主动的用接收到的服务器的 public key 去比对 ~/.ssh/known_hosts 有无相关的公钥， 然后进行底下的动作：
* 若接收的公钥尚未记录，则询问用户是否记录。若要记录 (范例中回答 yes 的那个步骤) 则写入 ~/.ssh/known_hosts 且继续登入的后续工作；若不记录 (回答 no) 则不写入该档案，并且离开登入工作；
* 若接收到的公钥已有记录，则比对记录是否相同，若相同则继续登入动作；若不相同，则出现警告信息， 且离开登入的动作。这是客户端的自我保护功能，避免你的服务器是被别人伪装的。
### 模拟 FTP 的文件传输方式：sftp
sftp 或 scp，这两个指令也都是使用 ssh 的通道 (port 22)，只是模拟成 FTP 与复制的动作而已。sftp ，这个指令的用法与 ssh 很相似，只是 ssh 是用在登入而 sftp 在上传/下载文件而已。
```
[root@www ~]# sftp student@localhost
Connecting to localhost...
student@localhost's password: <== 这里请输入密码
sftp> exit  <== 这里就是在等待你输入 ftp 相关指令的地方了！
```
进入到 sftp 之后，那就跟在一般 FTP 模式下的操作方法没有两样了，sftp 这个接口下的使用指令：
<table>
		<tbody><tr><td colspan="2">针对远方服务器主机 (Server) 之行为</td></tr>
		<tr><td>变换目录到 /etc/test 或其他目录</td>
			<td>cd /etc/test<br>cd PATH</td></tr>
		<tr><td>列出目前所在目录下的文件名</td>
			<td>ls<br>dir</td></tr>
		<tr><td>建立目录</td>
			<td>mkdir directory</td></tr>
		<tr><td>删除目录</td>
			<td>rmdir directory</td></tr>
		<tr><td>显示目前所在的目录</td>
			<td>pwd</td></tr>
		<tr><td>更改档案或目录群组</td>
			<td>chgrp groupname PATH</td></tr>
		<tr><td>更改档案或目录拥有者</td>
			<td>chown username PATH</td></tr>
		<tr><td>更改档案或目录的权限</td>
			<td>chmod 644 PATH<br>其中，644 与权限有关</td></tr>
		<tr><td>建立连结档</td>
			<td>ln oldname newname</td></tr>
		<tr><td>删除档案或目录</td>
			<td>rm PATH</td></tr>
		<tr><td>更改档案或目录名称</td>
			<td>rename oldname newname</td></tr>
		<tr><td>离开远程主机</td>
			<td>exit (or) bye (or) quit</td></tr>
		<tr><td colspan="2">针对本机 (Client) 之行为(都加上 l, L 的小写 )</td></tr>
		<tr><td>变换目录到本机的 PATH 当中</td>
			<td>lcd PATH</td></tr>
		<tr><td>列出目前本机所在目录下的文件名</td>
			<td>lls</td></tr>
		<tr><td>在本机建立目录</td>
			<td>lmkdir</td></tr>
		<tr><td>显示目前所在的本机目录</td>
			<td>lpwd</td></tr>
		<tr><td colspan="2">针对资料上传/下载的行为</td></tr>
		<tr><td>将档案由本机上传到远程主机</td>
			<td>put [本机目录或档案] [远程]<br>
			put [本机目录或档案]<br>
			如果是这种格式，则档案会放置到目前远程主机的目录下！</td></tr>
		<tr><td>将档案由远程主机下载回来</td>
			<td>get [远程主机目录或档案] [本机]<br>
			get [远程主机目录或档案]<br>
			若是这种格式，则档案会放置在目前本机所在的目录当中！可以使用通配符，例如：<br>
			get *<br>get *.rpm<br>亦是可以的格式！</td></tr>
		</tbody></table>

就整体而言， sftp 在 Linux 底下，如果不考虑图形接口，那么他已经可以取代 FTP 了，因此，在不考虑到图形接口的 FTP 软件时，可以直接关掉 FTP 的服务，而改以 sftp-server 来提供 FTP 的服务。
### 档案异地直接复制：scp
通常使用 sftp 是因为可能不知道服务器上面有什么档名的档案存在，如果已经知道服务器上的档案档名了， 那么最简单的文件传输则是透过 scp 这个指令：
```
[root@www ~]# scp [-pr] [-l 速率] file  [账号@]主机:目录名 <==上传
[root@www ~]# scp [-pr] [-l 速率] [账号@]主机:file  目录名 <==下载
选项与参数：
-p ：保留原本档案的权限数据；
-r ：复制来源为目录时，可以复制整个目录 (含子目录)
-l ：可以限制传输的速度，单位为 Kbits/s ，例如 [-l 800] 代表传输速限 100Kbytes/s

# 1. 将本机的 /etc/hosts* 全部复制到 127.0.0.1 上面的 student 家目录内
[root@www ~]# scp /etc/hosts* student@127.0.0.1:~
student@127.0.0.1's password: <==输入 student 密码
hosts                        100%  207         0.2KB/s   00:00
hosts.allow                  100%  161         0.2KB/s   00:00
hosts.deny                   100%  347         0.3KB/s   00:00
# 文件名显示                   进度  容量(bytes) 传输速度  剩余时间
# 你可以仔细看，出现的讯息有五个字段，意义如上所示。

# 2. 将 127.0.0.1 这部远程主机的 /etc/bashrc 复制到本机的 /tmp 底下
[root@www ~]# scp student@127.0.0.1:/etc/bashrc /tmp
```
其实上传或下载的重点是那个冒号 (:) ，连接在冒号后面的就是远程主机的档案。如果冒号在前，代表的就是从远程主机下载下来，如果冒号在后，则代表本机数据上传。 而如果想要复制目录的话，那么可以加上 -r 的选。
## ssh 客户端联机程序 - Windows 用户
常见的软件主要有 pietty, psftp 及 filezilla 等。
### 直接联机的 pietty
在 Linux 底下想要连接 SSH 服务器，可以直接利用 ssh 这个指令，在 Windows 操作系统底下就得要使用 pietty 或 putty：
* putty 官方网站：http://www.chiark.greenend.org.uk/~sgtatham/putty/
* pietty 官方网站：http://www.csie.ntu.edu.tw/~piaip/pietty/

在 putty 的官方网站上有很多的软件可以使用的，包括 putty/pscp/psftp 等等。他们分别对应了 ssh/scp/sftp 这三个指令。pietty 则是台湾的林弘德先生根据 putty 所改版而成的。由于 pietty 除了完整的兼容于 putty 之外，还提供了选单与较为完整的文字编码。  
编码的问题。要解决这个问题时，你必须要牢记下面的三个跟语系编码有关的数据要相同才行：
* 文本文件本身在存档时所挑选的语系；
* Linux 程序 (如 bash 软件) 本身所使用的语系 (可用 LANG 变量调整)；
* pietty 所使用的语系。
### 使用 sftp-server 的功能：psftp
在 putty 的官方网站上也提供 psftp 这支程序。这一支程序的重点则在使用 sftp-server。使用的方式可以直接点选 psftp 这个档案，让他直接启动，则会出现下面的图样：
```
psftp: no hostname specified; use "open host.name" to connect
psftp>

psftp: no hostname specified; use "open host.name" to connect
psftp> open 192.168.100.254
login as: root
root@192.168.100.254's password:
Remote working directory is /root
psftp> <== 这里就在等待你输入 FTP 的指令了！
```
### 图形化接口的 sftp 客户端软件：Filezila
## ssdh 服务器细部设定
基本上，所有的 sshd 服务器详细设定都放在 /etc/ssh/sshd_config 里面，不过，每个 Linux distribution 的预设设定都不太相同。在预设的档案内，只要是预设有出现且被批注的设定值 (设定值前面加 #)，即为『默认值！』，你可以依据它来修改：
```
[root@www ~]# vim /etc/ssh/sshd_config
# 1. 关于 SSH Server 的整体设定，包含使用的 port ，以及使用的密码演算方式
# Port 22
# SSH 预设使用 22 这个port，也可以使用多个port，即重复使用 port 这个设定项目！
# 例如想要开放 sshd 在 22 与 443 ，则多加一行内容为：『 Port 443 』
# 然后重新启动 sshd 这样就好了！不过，不建议修改 port number

Protocol 2
# 选择的 SSH 协议版本，可以是 1 也可以是 2 ，CentOS 5.x 预设是仅支援 V2。
# 如果想要支持旧版 V1 ，就得要使用『 Protocol 2,1 』才行。

# ListenAddress 0.0.0.0
# 监听的主机适配器！举个例子来说，如果你有两个 IP，分别是 192.168.1.100 及 
# 192.168.100.254，假设你只想要让 192.168.1.100 可以监听 sshd ，那就这样写：
# 『 ListenAddress 192.168.1.100 』默认值是监听所有接口的 SSH 要求

# PidFile /var/run/sshd.pid
# 可以放置 SSHD 这个 PID 的档案！上述为默认值

# LoginGraceTime 2m
# 当使用者连上 SSH server 之后，会出现输入密码的画面，在该画面中，
# 在多久时间内没有成功连上 SSH server 就强迫断线！若无单位则默认时间为秒！

# Compression delayed
# 指定何时开始使用压缩数据模式进行传输。有 yes, no 与登入后才将数据压缩 (delayed)

# 2. 说明主机的 Private Key 放置的档案，预设使用下面的档案即可！
# HostKey /etc/ssh/ssh_host_key        # SSH version 1 使用的私钥
# HostKey /etc/ssh/ssh_host_rsa_key    # SSH version 2 使用的 RSA 私钥
# HostKey /etc/ssh/ssh_host_dsa_key    # SSH version 2 使用的 DSA 私钥
# 还记得我们在主机的 SSH 联机流程里面谈到的，这里就是 Host Key

# 3. 关于登录文件的讯息数据放置与 daemon 的名称！
SyslogFacility AUTHPRIV
# 当有人使用 SSH 登入系统的时候，SSH 会记录信息，这个信息要记录在什么 daemon name
# 底下？预设是以 AUTH 来设定的，即是 /var/log/secure 里面！什么？忘记了！
# 回到 Linux 基础去翻一下。其他可用的 daemon name 为：DAEMON,USER,AUTH,
# LOCAL0,LOCAL1,LOCAL2,LOCAL3,LOCAL4,LOCAL5,

# LogLevel INFO
# 登录记录的等级！嘿嘿！任何讯息！同样的，忘记了就回去参考！

# 4. 安全设定项目！极重要！
# 4.1 登入设定部分
# PermitRootLogin yes
# 是否允许 root 登入！预设是允许的，但是建议设定成 no！

# StrictModes yes
# 是否让 sshd 去检查用户家目录或相关档案的权限数据，
# 这是为了担心使用者将某些重要档案的权限设错，可能会导致一些问题所致。
# 例如使用者的 ~.ssh/ 权限设错时，某些特殊情况下会不许用户登入

# PubkeyAuthentication yes
# AuthorizedKeysFile      .ssh/authorized_keys
# 是否允许用户自行使用成对的密钥系统进行登入行为，仅针对 version 2。
# 至于自制的公钥数据就放置于用户家目录下的 .ssh/authorized_keys 内

PasswordAuthentication yes
# 密码验证当然是需要的！所以这里写 yes 

# PermitEmptyPasswords no
# 若上面那一项如果设定为 yes 的话，这一项就最好设定为 no ，
# 这个项目在是否允许以空的密码登入！当然不许！

# 4.2 认证部分
# RhostsAuthentication no
# 本机系统不使用 .rhosts，因为仅使用 .rhosts太不安全了，所以这里一定要设定为 no

# IgnoreRhosts yes
# 是否取消使用 ~/.ssh/.rhosts 来做为认证！当然是！

# RhostsRSAAuthentication no #
# 这个选项是专门给 version 1 用的，使用 rhosts 档案在 /etc/hosts.equiv
# 配合 RSA 演算方式来进行认证！不要使用

# HostbasedAuthentication no
# 这个项目与上面的项目类似，不过是给 version 2 使用的！

# IgnoreUserKnownHosts no
# 是否忽略家目录内的 ~/.ssh/known_hosts 这个档案所记录的主机内容？
# 不要忽略，所以这里就是 no 

ChallengeResponseAuthentication no
# 允许任何的密码认证！所以，任何 login.conf 规定的认证方式，均可适用！
# 但目前我们比较喜欢使用 PAM 模块帮忙管理认证，因此这个选项可以设定为 no

UsePAM yes
# 利用 PAM 管理使用者认证有很多好处，可以记录与管理。
# 所以这里我们建议你使用 UsePAM 且 ChallengeResponseAuthentication 设定为 no 
　
# 4.3 与 Kerberos 有关的参数设定！因为我们没有 Kerberos 主机，所以底下不用设定！
# KerberosAuthentication no
# KerberosOrLocalPasswd yes
# KerberosTicketCleanup yes
# KerberosTgtPassing no
　
# 4.4 底下是有关在 X-Window 底下使用的相关设定！
X11Forwarding yes
# X11DisplayOffset 10
# X11UseLocalhost yes
# 比较重要的是 X11Forwarding 项目，他可以让窗口的数据透过 ssh 信道来传送
# 在本章后面比较进阶的 ssh 使用方法中会谈到。

# 4.5 登入后的项目：
# PrintMotd yes
# 登入后是否显示出一些信息呢？例如上次登入的时间、地点等等，预设是 yes
# 亦即是打印出 /etc/motd 这个档案的内容。但是，如果为了安全，可以考虑改为 no ！

# PrintLastLog yes
# 显示上次登入的信息！可以啊！预设也是 yes ！

# TCPKeepAlive yes
# 当达成联机后，服务器会一直传送 TCP 封包给客户端藉以判断对方式否一直存在联机。
# 不过，如果联机时中间的路由器暂时停止服务几秒钟，也会让联机中断
# 在这个情况下，任何一端死掉后，SSH可以立刻知道！而不会有僵尸程序的发生！
# 但如果你的网络或路由器常常不稳定，那么可以设定为 no 的

UsePrivilegeSeparation yes
# 是否权限较低的程序来提供用户操作。我们知道 sshd 启动在 port 22 ，
# 因此启动的程序是属于 root 的身份。那么当 student 登入后，这个设定值
# 会让 sshd 产生一个属于 sutdent 的 sshd 程序来使用，对系统较安全

MaxStartups 10
# 同时允许几个尚未登入的联机画面？当我们连上 SSH ，但是尚未输入密码时，
# 这个时候就是我们所谓的联机画面。在这个联机画面中，为了保护主机，
# 所以需要设定最大值，预设最多十个联机画面，而已经建立联机的不计算在这十个当中

# 4.6 关于用户抵挡的设定项目：
DenyUsers *
# 设定受抵挡的使用者名称，如果是全部的使用者，那就是全部挡
# 若是部分使用者，可以将该账号填入！例如下列！
DenyUsers test

DenyGroups test
# 与 DenyUsers 相同！仅抵挡几个群组而已！

# 5. 关于 SFTP 服务与其他的设定项目！
Subsystem       sftp    /usr/lib/ssh/sftp-server
# UseDNS yes
# 一般来说，为了要判断客户端来源是正常合法的，因此会使用 DNS 去反查客户端的主机名
# 不过如果是在内网互连，这项目设定为 no 会让联机达成速度比较快。
```
建议你 (1)将 root 的登入权限取消； (2)将 ssh 版本设定为 2 。如果你修改过上面这个档案(/etc/ssh/sshd_config)，那么就必需要重新启动一次 sshd 这个 daemon 才行：
```
/etc/init.d/sshd restart
```
## 制作不用密码可立即登入的 ssh 用户
我们可以将 Client 产生的 Key 给他拷贝到 Server 当中，所以， 以后 Client 登入 Server 时，由于两者在 SSH 要联机的讯号传递中，就已经比对过 Key 了， 因此，可以立即进入数据传输接口中，而不需要再输入密码：
1. 客户端建立两把钥匙：想一想，在密钥系统中，私钥比较重要，私钥才是解密的关键。这两把钥匙当然得在发起联机的客户端建置才对。利用的指令为 ssh-keygen 这个命令；
2. 客户端放置好私钥档案：将 Private Key 放在 Client 上面的家目录，亦即 $HOME/.ssh/ ， 并且得要注意权限
3. 将公钥放置服务器端的正确目录与文件名去：最后，将那把 Public Key 放在任何一个你想要用来登入的服务器端的某 User 的家目录内之 .ssh/ 里面的认证档案即可完成整个程序。
### 1. 客户端建立两把钥匙
在 clientlinux.centos.vbird 这部主机上面以 vbirdtsai 的身份来建立两把钥匙即可。 不过，需要注意的是，我们有多种密码算法，如果不指定特殊的算法，则默认以 RSA 算法来处理：
```
[vbirdtsai@clientlinux ~]$ ssh-keygen [-t rsa|dsa] <==可选 rsa 或 dsa
[vbirdtsai@clientlinux ~]$ ssh-keygen  <==用预设的方法建立密钥
Generating public/private rsa key pair.
Enter file in which to save the key (/home/vbirdtsai/.ssh/id_rsa): <==按 enter
Created directory '/home/vbirdtsai/.ssh'. <==此目录若不存在则会主动建立
Enter passphrase (empty for no passphrase): <==按 Enter 不给密码
Enter same passphrase again: <==再输入一次 Enter 
Your identification has been saved in /home/vbirdtsai/.ssh/id_rsa. <==私钥档
Your public key has been saved in /home/vbirdtsai/.ssh/id_rsa.pub. <==公钥档
The key fingerprint is:
0f:d3:e7:1a:1c:bd:5c:03:f1:19:f1:22:df:9b:cc:08 vbirdtsai@clientlinux.centos.vbird

[vbirdtsai@clientlinux ~]$ ls -ld ~/.ssh; ls -l ~/.ssh
drwx------. 2 vbirdtsai vbirdtsai 4096 2011-07-25 12:58 /home/vbirdtsai/.ssh
-rw-------. 1 vbirdtsai vbirdtsai 1675 2011-07-25 12:58 id_rsa      <==私钥档
-rw-r--r--. 1 vbirdtsai vbirdtsai  416 2011-07-25 12:58 id_rsa.pub  <==公钥档
```
我的身份是 vbirdtsai ，所以当我执行 ssh-keygen 时，才会在我的家目录底下的 .ssh/ 这个目录里面产生所需要的两把 Keys ，分别是私钥 (id_rsa) 与公钥 (id_rsa.pub)。 ~/.ssh/ 目录必须要是 700 的权限才行。id_rsa 的档案权限必须要是 -rw------- 且属于 vbirdtsai 自己才行。否则在未来密钥比对的过程当中，可能会被判定为危险而无法成功的以公私钥成对档案的机制来达成联机。 其实，建立私钥后预设的权限与文件名放置位置都是正确的，你只要检查过没问题即可。
### 2. 将公钥档案数据上传到服务器上：
因为我们要登入 www.centos.vbird 是以 dmtsai 的身份，因此我们就得要将上个步骤建立的公钥 (id_rsa.pub) 上传到服务器上的 dmtsai 用户才行。
```
[vbirdtsai@clientlinux ~]$ scp ~/.ssh/id_rsa.pub dmtsai@192.168.100.254:~
# 上传到 dmtsai 的家目录底下即可。
```
### 3. 将公钥放置服务器端的正确目录与文件名：
还记得 sshd_config 里面的 AuthorizedKeysFile 这个设定值，就是在指定公钥数据应该要放置的文件名。所以，我们必须要到服务器端的 dmtsai 这个用户身份下， 将刚刚上传的 id_rsa.pub 数据附加到 authorized_keys 这个档案内才行。
```
# 1. 建立 ~/.ssh 档案，注意权限需要为 700 喔！
[dmtsai@www ~]$ ls -ld .ssh
ls: .ssh: 没有此一档案或目录
# 由于可能是新建的用户，因此这个目录不存在。不存在才作底下建立目录的行为

[dmtsai@www ~]$ mkdir .ssh; chmod 700 .ssh
[dmtsai@www ~]$ ls -ld .ssh
drwx------. 2 dmtsai dmtsai 4096 Jul 25 13:06 .ssh
# 权限设定中，务必是 700 且属于使用者本人的账号与群组才行！

# 2. 将公钥档案内的数据使用 cat 转存到 authorized_keys 内
[dmtsai@www ~]$ ls -l *pub
-rw-r--r--. 1 dmtsai dmtsai 416 Jul 25 13:05 id_rsa.pub <==确实有存在

[dmtsai@www ~]$ cat id_rsa.pub >> .ssh/authorized_keys
[dmtsai@www ~]$ chmod 644 .ssh/authorized_keys
[dmtsai@www ~]$ ls -l .ssh
-rw-r--r--. 1 dmtsai dmtsai 416 Jul 25 13:07 authorized_keys
# 这个档案的权限设定中，就得要是 644 才可以！不可以搞混了！
```
以后你从 clientlinux.centos.vbird 的 vbirdtsai 登入到 www.centos.vbird 的 dmtsai 用户时， 就不需要任何的密码了。
* Client 必须制作出 Public & Private 这两把 keys，且 Private 需放到 ~/.ssh/ 内；
* Server 必须要有 Public Key ，且放置到用户家目录下的 ~/.ssh/authorized_keys，同时目录的权限 (.ssh/) 必须是 700 而档案权限则必须为 644 ，同时档案的拥有者与群组都必须与该账号吻合才行。

未来，当你还想要登入其他的主机时，只要将你的 public key (就是 id_rsa.pub 这个档案) 给他 copy 到其他主机上面去，并且新增到某账号的 ~/.ssh/authorized_keys 这个档案中即可。
## 简易安全设定
* 服务器软件本身的设定强化：/etc/ssh/sshd_config
* TCP wrapper 的使用：/etc/hosts.allow, /etc/hosts.deny
* iptables 的使用： iptables.rule, iptables.allow
### 服务器软件本身的设定强化：/etc/ssh/sshd_config
一般而言，这个档案的默认项目就已经很完备了，如果你有些使用者方面的顾虑，那么可以这样修正一些问题：
1. 禁止 root 这个账号使用 sshd 的服务；
2. 禁止 nossh 这个群组的用户使用 sshd 的服务；
3. 禁止 testssh 这个用户使用 sshd 的服务；
```
# 1. 先观察一下所需要的账号是否存在
[root@www ~]# for user in sshnot1 sshnot2 sshnot3 testssh student; do \
> id $user | cut -d ' ' -f1-3 ; done
uid=507(sshnot1) gid=509(sshnot1) groups=509(sshnot1),508(nossh)
uid=508(sshnot2) gid=510(sshnot2) groups=510(sshnot2),508(nossh)
uid=509(sshnot3) gid=511(sshnot3) groups=511(sshnot3),508(nossh)
uid=511(testssh) gid=513(testssh) groups=513(testssh)
uid=505(student) gid=506(student) groups=506(student)
# 若上述账号并不存在你的系统，请自己建置出来

# 2. 修改 sshd_config 并且重新启动 sshd 
[root@www ~]# vim /etc/ssh/sshd_config
PermitRootLogin no  <==约在第 39 行，请拿掉批注且修改成这样
DenyGroups  nossh   <==底下这两行可以加在档案的最后面
DenyUsers   testssh

[root@www ~]# /etc/init.d/sshd restart

# 3. 测试与观察相关的账号登入情况
[root@www ~]# ssh root@localhost  <==并请输入正确的密码
[root@www ~]# tail /var/log/secure
Jul 25 13:14:05 www sshd[2039]: pam_unix(sshd:auth): authentication failure; 
logname= uid=0 euid=0 tty=ssh ruser= rhost=localhost  user=root
# 你会发现出现这个错误讯息，而不是密码输入错误而已。

[root@www ~]# ssh sshnot1@localhost  <==并请输入正确的密码
[root@www ~]# tail /var/log/secure
Jul 25 13:15:53 www sshd[2061]: User sshnot1 from localhost not allowed because
a group is listed in DenyGroups

[root@www ~]# ssh testssh@localhost  <==并请输入正确的密码
[root@www ~]# tail /var/log/secure
Jul 25 13:17:16 www sshd[2074]: User testssh from localhost not allowed 
because listed in DenyUsers
```
### /etc/hosts.allow 及 /etc/hosts.deny
sshd 只想让本机以及区网内的主机来源能够登入的话，那就这样作：
```
[root@www ~]# vim /etc/hosts.allow
sshd: 127.0.0.1 192.168.1.0/255.255.255.0 192.168.100.0/255.255.255.0

[root@www ~]# vim /etc/hosts.deny
sshd : ALL 
```
### iptables 封包过滤防火墙
你应该在 iptables.rule 内将 port 22 的放行功能取消，然后再到 iptables.allow 里面新增这行：
```
[root@www ~]# vim /usr/local/virus/iptables/iptables.allow
iptables -A INPUT -i $EXTIF -s 192.168.1.0/24 -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -i $EXTIF -s 192.168.100.0/24 -p tcp --dport 22 -j ACCEPT

[root@www ~]# /usr/local/virus/iptables/iptables.rule
```

# 最原始图形接口：Xdmcp 服务的启用
## X Window 的 Server/Client 架构与各组件
Linux 使用的图形接口是 X-Window System ，是能够跨平台的，目前在 Linux 上头开发的图形接口软件，几乎都是使用这个 X 的架构来处理。  
X Window System 在运作的过程中，又因控制的数据不同而分为 X Server 与 X Client 两种程序，虽然说是 X Server/Client ， 但是他的作用却与网络主机的 Server/Client 架构大异其趣。
* X Server： 这组程序主要负责的是屏幕画面的绘制与显示。 X Server 可以接收来自 X client 的数据，将这些数据绘制呈现为图面在屏幕上。 此外，我们移动鼠标、点击数据、由键盘输入数据等等，也会透过 X Server 来传达到 X Client 端，而由 X Client 来加以运算出应绘制的数据；
* X Client： 这组程序主要负责的是数据的运算。 X Client 在接受到 X Server 传来的数据后 (例如移动鼠标、点击 icon 等动作)，会经由本身的运算而得到鼠标应该要如何移动、 点击的结果应该要出现什么样的数据、键盘输入的结果应该要如何呈现等等，然后将这些结果告知 X Server ，让他自行去绘制到屏幕上。

由于每一支 X client 都是独立存在的程序，因此在图形显示会发生一些迭图的问题，因此，后来就有一组特殊的 X client 在进行管理所有的其他 X client 程序，就是 Window Manager 。
* Window Manager (WM)：是一组控制所有 X client 的管理程序，并同时提供例如任务栏、 背景桌面、虚拟桌面、窗口大小、窗口移动与重迭显示等任务。Window manager 主要由一些大型的计划案所开发而来，常见的有 GNOME, KDE, XFCE 等
* Display Manager (DM)：提供使用者登入的画面以让用户可以藉由图形接口登入。 在使用者登入后，可透过 display manager 的功能去呼叫其他的 Window manager ，让用户在图形接口的登入过程变得更简单。 由于 DM 也是启动一个等待输入账号密码的图形数据，因此 DM 会主动去唤醒一个 X Server 然后在上头加载等待输入的画面就是了。

在目前新释出的 Linux distributions 中，通常启动图形接口让用户登入的方式中，都是先执行 Display Manager 程序， 该程序会主动加载一个 X Server 程序，然后再提供一个等待输入账号密码的接口程序，之后再根据用户的选择去启动所需要的 Window Manager 程序，最后就由用户直接操作 WM 来玩图形接口。
### X Window System 用在网络上的方式： XDMCP
此时你得先在客户端启动一个 X server 将图形接口绘图所需要的硬件装置配置好， 并且启动一个 X server 常见的接收埠口 (通常是 port 6000)，然后再由服务器端的 X client 取得绘图数据，再将数据绘制成图。透过这个机制，你可以在任何一部启动 X server 登入服务器。  
Xdmcp 启动后会在服务器的 udp 177 开始监听，然后当客户端的 X server 联机到服务器的 port 177 之后， 我们的 Xdmcp 就会在客户端的 X server 放上用户输入账密的图形接口程序。这样就能透过这个 Xdmcp 去加载服务器所提供的类似 Window Manager 的相关 X client 。取得图形接口的远程联机服务器。
## 设定 gdm 的 XDMCP 服务
Xdmcp 协议是由 DM 程序所提供的。我们的 CentOS 预设的 DM 为 GNOME 这个计划所提供的 gdm 。因此，你想要启动 Xdmcp 服务，那就得要针对 gdm 这个程序来设定。gdm 的设定数据都放置在 /etc/gdm/ 目录下，而我们所要修改的配置文件其实仅是一个 /etc/gdm/custom.conf 档案而已。
> X11 提供的 display manager 为 xdm ，而著名的 KDE 与 GNOME 也都有自己的 display manager 管理程序，分别是 kdm 与 gdm 。你可以透过三者中任何一者的 display manager 的配置文件来启动 xdmcp 这个协定

不过，因为我们安装的基准是『Basic server』，所以很多图形接口软件并没有被安装起来。因此，在实作 Xdmcp 之前，我们得先安装图形接口才行：
```
# 先检查看看与 X 相关的软件群组有哪些？
[root@www ~]# yum grouplist
   Desktop
   Desktop Platform
   X Window System
# gdm 是在 Destop 中！

[root@www ~]# yum groupinstall "Desktop" "Desktop Platform" "X Window System"
```
```
[root@www ~]# vim /etc/gdm/custom.conf
[security]           <==在与资安方面有关的信息，大多指登录相关事宜
AllowRemoteRoot=yes  <==xdmcp 预设不许 root 登入，得用这个项目才能以 root 登入
DisallowTCP=false    <==这个项目在允许客户端使用 TCP 的方式联机到 xdmcp

[xdmcp]              <==就是这个小节的重点之一
Enable=true          <==启动 xdmcp 的最重要项目

[root@www ~]# init 5
# 上述这个指令会切换到 X 图形画面，如果确定要使用 gdm，runlevel 得调整到 5 才好
# 果真如此的话，那就得要调整 /etc/inittab 

[root@www ~]# netstat -tulnp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address  Foreign Address   State    PID/Program name
tcp        0      0 0.0.0.0:6000   0.0.0.0:*         LISTEN   4557/Xorg
tcp        0      0 :::6000        :::*              LISTEN   4557/Xorg
udp        0      0 0.0.0.0:177    0.0.0.0:*                  4536/gdm-binary
# 上述的 port 6000 是由 DisallowTCP=false 项目启动的，port 177 才是我们要的
```
如果你是在 runlevel 3 底下并且不希望变更成为 runlevel 5 ，可以这样启动 xdmcp：
```
[root@www ~]# init 3
[root@www ~]# runlevel
5 3 <==左边的是前一个 runlevel，右边的是目前的，因此目前是 runlevel 3
[root@www ~]# gdm   <==这样就启动 xdmcp
[root@www ~]# vim /etc/rc.d/rc.local
/usr/sbin/gdm
```
如果是 runlevel 5 ，因为在 /etc/inittab 就已经有自动启动 gdm 了， 所以你只要顺利启动 runlevel 5 即可。但如果你是在 runlevel 3 的话，因为这样 gdm 就不会被系统的启动流程启动， 那你只好自己在 /etc/rc.d/rc.local 里面指定启动他。接下来，你得要开放客户端对你的 port 177 联机才行！ 请自行修改你的防火墙规则，开放 udp port 177 。
```
[root@www ~]# vim /usr/local/virus/iptables/iptables.rule
iptables -A INPUT -p UDP -i $EXTIF --dport 177 --sport 1024:65534 \
 -s 192.168.100.0/24 -j ACCEPT #xdmcp
# 特点是使用 UDP 埠口以及加入来源端 IP 网域的控管

[root@www ~]# /usr/local/virus/iptables/iptables.rule
[root@www ~]# iptables-save | grep 177
-A INPUT -s 192.168.100.0/24 -i eth0 -p udp -m udp --sport 1024:65534 --dport 177 -j ACCEPT
# 确实有开放 port 177 ，而且是 udp 的埠口！要注意这两个项目。
```
## 用户系统为 Linux 的登入方式
由于 Linux 本身的窗口就是由 X server 提供来的，因此使用 Linux 登入远程的图形服务器是很简单。但是因为启动 X 的方式不同而已数种启动方式，底下我们就讲讲两个常见的启动方式：
### 在不同的 X 环境下启动联机： 直接用 X
如果你的客户端已经在 runlevel 5 了，因此其实你已经有一个 X 窗口的环境，这个环境的显示终端机就称为『 :0 』。在 CentOS 6.x 的环境中，如果原本就是 runlevel 5 的环境，那么这个图形接口的 :0 是在 tty1 终端机；如果是由 runlevel 3 启动图形接口，那就是在 tty7 。由于已经有一个 X 了，因此你必须要在另外的终端机启动另一个 X 才行，那个新的 X 就称为 :1 接口，其实通常就在 tty7 或 tty8 。但因为 X server 要接受 X client 必须要有授权才行， 所以你得先在窗口接口开放接受来自服务器的 X client 数据。  
此外，虽然你在客户端是以主动的方式连接到服务器的 udp port 177 ，但是服务器的 X client 却会主动的连接到你客户端的 X server，因此，你必须要开放来自服务器端主动对你的 TCP port 6001 (因为是 :1 界面) 的防火墙联机才行
```
# 1. 放行 X client 传来的资料：在 X Window 的画面当中启用 shell 输入：
[root@clientlinux ~]# xhost + 192.168.100.254
192.168.100.254 being added to access control list
# 注意！你是客户端！且假设我刚刚那部 Linux 主机的 IP 为 192.168.100.254

# 2. 开始放行防火墙，因为我们启动 port 6001 ，所以你在客户端这样作：
[root@clientlinux ~]# vim /usr/local/virus/iptables/iptables.allow
iptables -A INPUT -i $EXTIF -s 192.168.100.0/24 -p tcp --dport 6001 -j ACCEPT

[root@clientlinux ~]# /usr/local/virus/iptables/iptables.rule
[root@clientlinux ~]# iptables-save
-A INPUT -s 192.168.100.0/24 -p tcp -m tcp --dport 6001 -j ACCEPT
# 要能看到上面这一行才行

# 3. 在文字接口 (例如 tty1) 下输入如下的指令：
[root@clientlinux ~]# X -query 192.168.100.254 :1
# 进入 X Window 
```
如果想要回到本机的窗口接口， 就回到 tty7 (:0) 即可切换成功 (在 runlevel 5 时，:0 在 tty1 ，而 :1 在 tty7 )。想要关闭 tty8 ，不能够在 tty8 注销，因为注销后，系统会重新开一个等待登入的画面，你还是没办法关闭的。你得要回到刚刚启动 X 的 tty1 然后按下 [ctrl]-c 中断联机即可。
### 在同一个 X 底下启动另一个 X：使用 Xnest
直接在 tty7 启动另一个窗口来加载远程服务器的图形接口，透过 Xnest 。这指令需要在 X 的环境下使用：
```
[root@www ~]# Xnest -query 主机名 -geometry 分辨率 :1
选项与参数：
-query    ：后面接 xdmcp 服务器的主机名或 IP 
-geometry ：后面接画面的分辨率，例如 1024x768 或 800x600 等之类的分辨率

# 根据上述数据，使用 800x600 连上 192.168.100.254 那部主机：
[root@www ~]# yum install xorg-x11-server-Xnest
[root@www ~]# Xnest -query 192.168.100.254 -geometry 640x480 :1
```
要关闭这个 X ，直接按下关闭，或者是中断那个 Xnest 的程序即可。
## 用户系统为 Windows 的登入方式：Xming
由于 Windows 本身并没有提供预设的 X server ，因此我们得要自行安装 X server 在 Windows 上面才行。 目前常见的 X server 有底下这几个：
* X-Win32 (http://www.xwin32.tw/)
* Exceed (http://www.hummingbird.com/products/nc/exceed/index.html?cks=y)
* Xming (http://sourceforge.net/projects/xming/)

其中 X-Win32 与 Exceed 都属于商业软件，而 Xming 则属于轻量级的自由软件。
### 重点在 Server 与 Client 的防火墙上
XDMCP 不论是在 Server 还是 Client 的设定上面都很简单，但是有时候你就是会发现， 明明所有的动作都做完了，但是就是没有办法连上 Xdmcp 服务器。最容易发生错误的其实就是防火墙。因为虽然我们客户端启动 X server 后，会主动联机到服务器端的 Xdmcp (port 177)，但是，接下来却是服务器主动联机到我们客户端的 X server (可能是 port 6000~6010)。 因此，如果你只是设定了服务器的防火墙而已，那么很可能出现问题的应该就是客户端的防火墙忘记打开提供服务器主动联机的规则。

# 华丽的图形接口：VNC 服务器
## 预设的 VNC 服务器：使用 twm window manager
VNC server 会在服务器端启动一个监听用户要求的端口，一般端口号码在 5901 ~ 5910 之间。当客户端启动 X server 联机到 5901 之后， VNC server 再将一堆预先设定好的 X client 透过这个联机传递到客户端上，最终就能够在客户端显示服务器的图形接口了。  
不过需要注意的是，预设的 VNC server 都是独立提供给『单一』一个客户端来联机的，因此当你要使用 VNC 时， 再联机到服务器去启动 VNC server 即可。所以，一般来说， VNC server 都是使用手动启动的，然后使用完毕后， 再将 VNC server 关闭即可。
```
[root@www ~]# vncserver [:号码] [-geometry 分辨率] [options]
[root@www ~]# vncserver [-kill :号码]
选项与参数：
:号码     ：就是将 VNC server 开在哪个埠口，如果是 :1 则代表 VNC 5901 埠口
-geometry ：就是分辨率，例如 1024x768 或 800x600 之类的
options   ：其他 X 相关的选项，例如 -query localhost 之类的
-kill     ：将已经启动的 VNC 埠口删除！依据身份控制

[root@www ~]# yum install tigervnc-server
# 这个是必须要的服务器软件，注意软件的名称与之前的版本不同！

# 将 VNC server 启动在 5903 埠口
[root@www ~]# vncserver :3

You will require a password to access your desktops.

Password:  <==输入 VNC 的联机密码，这是建立 VNC 时所需要的
Verify:    <==再输入一次相同的密码
xauth:  creating new authority file /root/.Xauthority

New 'www.centos.vbird:3 (root)' desktop is www.centos.vbird:3

Creating default startup script /root/.vnc/xstartup
Starting applications specified in /root/.vnc/xstartup
Log file is /root/.vnc/www.centos.vbird:3.log

[root@www ~]# netstat -tulnp | grep X
tcp        0      0 0.0.0.0:5903   0.0.0.0:*      LISTEN      4361/Xvnc
tcp        0      0 0.0.0.0:6000   0.0.0.0:*      LISTEN      1755/Xorg
tcp        0      0 0.0.0.0:6003   0.0.0.0:*      LISTEN      4361/Xvnc
tcp        0      0 :::6000        :::*           LISTEN      1755/Xorg
tcp        0      0 :::6003        :::*           LISTEN      4361/Xvnc
# 已经启动所需要的埠口
```
1. 密码至少需要六个字符
2. 依据使用 vncserver 的身份，将刚刚建立的密码放置于该账号家目录下。例如上述的身份是使用 root 身份，因此密码文件会放在 /root/.vnc/passwd 这个档案中但是若该档案已经存在，则不会出现建立密码的画面。
3. 当客户端联机成功后，服务器将会传送 /root/.vnc/startx 内的 X client 给客户端

修改 VNC 密码，使用 vncpasswd ：
```
[root@www ~]# ls -l /root/.vnc/passwd
-rw-------. 1 root root 8 Jul 26 15:08 /root/.vnc/passwd
[root@www ~]# vncpasswd
Password:  <==就是这里开始输入新的密码
Verify:
[root@www ~]# ls -l /root/.vnc/passwd
-rw-------. 1 root root 8 Jul 26 15:15 /root/.vnc/passwd
# 时间有更新喔，这个档案的内容更动过
```
接下来开始放行 5903 这个埠口的联机防火墙规则，因为预计可能会开放 11 个 VNC 的埠口：
```
[root@www ~]# vim /usr/local/virus/iptables/iptables.allow
iptables -A INPUT -i $EXTIF -s 192.168.100.0/24 -p tcp --dport 5900:5910 -j ACCEPT

[root@www ~]# /usr/local/virus/iptables/iptables.rule
[root@www ~]# iptables-save
-A INPUT -s 192.168.100.0/24 -i eth0 -p tcp -m tcp --dport 5900:5910 -j ACCEPT
```
## VNC 的客户端联机软件
### Linux 客户端程序：vncviewer
```
[root@clientlinux ~]# yum install tigervnc
[root@clientlinux ~]# vncviewer 192.168.10.254:3
# 这个指令请一定一定要在图形接口上面执行才行
```
输入的密码是 VNC 的联机密码，而不是 root 的登入密码。
### Windows 客户端程序：realvnc
## VNC 搭配本机的 Xdmcp 画面
如果因为某些特殊因素，你得要使用 VNC 来搭配 xdmcp 的输出时，那就直接在服务器透过底下的指令来处理即可。必须要已经启动了 xdmcp 了。
```
# 1. 要确定 xdmcp 已经启动了才可以：
[root@www ~]# netstat -tlunp | grep 177
udp        0      0 0.0.0.0:177   0.0.0.0:*      1734/gdm-binary

# 2. 切换成 student，并且启动 VNC server 在 :5
[root@www ~]# su - student
[student@www ~]$ vncserver :5 -query localhost
You will require a password to access your desktops.

Password:
Verify:
xauth:  creating new authority file /home/student/.Xauthority

New 'www.centos.vbird:5 (student)' desktop is www.centos.vbird:5

Creating default startup script /home/student/.vnc/xstartup
Starting applications specified in /home/student/.vnc/xstartup
Log file is /home/student/.vnc/www.centos.vbird:5.log

# 3. 取消 xstartup 的启动内容
[student@www ~]$ vim /home/student/.vnc/xstartup
....(前面省略)....
#xterm -geometry 80x24+10+10 -ls -title "$VNCDESKTOP Desktop" &
#twm &
# 将这个档案的内容，全部都加上 # 批注掉

# 4. 重新启动 vncserver
[student@www ~]$ vncserver -kill :5
[student@www ~]$ vncserver :5 -query localhost
```
接下来请使用 root 的身份加入 5905 的端口防火墙规则，然后自行使用 Linux 的 vncviewer 或 Windows 的 RealVNC 来联机。我们这只 VNC 的联机程序是 student 身份，但是我们却可以透过 xdmcp 的登入功能来登入 root 身份。因为在服务器上面的 Xvnc 程序是 student 拥有。
## 开机就启动 VNC server 的方法
不要将 vncserver 的指令写入在 /etc/rc.d/rc.local 中，否则可能会产生 localhost 无法登入的问题。要修改一下配置文件，才能实现开机启动。底下使用 student 的身份启动 VNC server，而启动的方式为使用 xdmcp 登入画面，启动的埠口就定在 5901 ：
```
[root@www ~]# vim /etc/sysconfig/vncservers
VNCSERVERS="1:student"
VNCSERVERARGS[1]="-query localhost"
# 上述两行的 1 指的就是那个埠口 5901 

[root@www ~]# /etc/init.d/vncserver restart
[root@www ~]# chkconfig vncserver on
```
## 同步的 VNC：可以透过图示同步教学
Linux 本身提供多个 VNC server ，她们是各自独立的，所以不会与 tty7 的画面同步。想要与 Linux 的 tty7 同步的话，可以利用 VNC 释出的给 X Server 使用的模块来加以设定即可。  
使用这个模块可以让两个图形接口在 server/client 都是一样的， 所以，如果你想要教你的朋友你是如何设定的，那就可以透过这个机制来处理，你的朋友在远程就能够知道你一步一步进行的过程。
```
[root@www ~]# yum install tigervnc-server-module
[root@www ~]# vim /etc/X11/xorg.conf
Section "Screen"
        Identifier "Screen0"
        Device     "Videocard0"
        DefaultDepth     24
        # VBird
        Option "passwordFile" "/home/student/.vnc/passwd"
        SubSection "Display"
                Viewport   0 0
                Depth     24
        EndSubSection
EndSection

# VBird
Section "Module"
    Load    "vnc"
EndSection
# 假设你的 vnc 密码档案放置在 /home/student/.vnc/passwd 里头，
# 这个时候就得要将密码文件内容写到 Screen 这个 section 当中了

[root@www ~]# init 3 ; init 5
[root@www ~]# netstat -tlunp | grep X
tcp        0      0 0.0.0.0:5900   0.0.0.0:*      LISTEN      7445/Xorg
tcp        0      0 0.0.0.0:6000   0.0.0.0:*      LISTEN      7445/Xorg
tcp        0      0 :::6000        :::*           LISTEN      7445/Xorg
# 这几个 port 启动的 PID 都一样喔！所以会启动一个 port 5900 
```
之后你可以使用『 vncviewer 192.168.100.254 』来联机即可，不需要加上 :0 之类的埠口。客户端与服务器端的图形接口，画面会同步运作。不过这个动作还是只允许一条 VNC 联机，不能让所有客户端都连到 port 5900 。

# 仿真的远程桌面系统：XRDP 服务器
使用上面的图形接口的联机服务器都有一个问题，除了联机机制的不同之外，上头的 Xdmcp 与 VNC 原则上，资料都没有加密。 因此上面的动作大多仅适合局域网络内运作。Windows 的远程桌面 (Remote Desktop Procotol, RDP) 其实是具有联机加密功能的，Linux 上面提供 RDP Server 的就是 XRDP 服务器。  
```
[root@www ~]# vim /etc/yum.repos.d/fedora_epel.repo
[epel]
name=CentOS-$releasever - Epel
baseurl=http://download.fedora.redhat.com/pub/epel/6/x86_64/
gpgcheck=0
enabled=1

[root@www ~]# yum clean all
[root@www ~]# yum install xrdp
```
在一般的主机上面安装好这个 xrdp 之后，你根本不需要调整任何配置文件，保留好配置文件就好了，然后启动它，并且设定开机后启动，未来只要用远程联机连到这部主机， 系统就会启动 5910~5920 以上的 VNC 埠口，然后你就能够透过 RDP 的协议取得 VNC 的画面，最后就能够登入系统。
```
[root@www ~]# /etc/init.d/xrdp start
[root@www ~]# chkconfig xrdp on
[root@www ~]# netstat  | grep xrdp
tcp        0      0 127.0.0.1:3350  0.0.0.0:*     LISTEN    6615/xrdp-sesman
tcp        0      0 0.0.0.0:3389    0.0.0.0:*     LISTEN    6611/xrdp
# 远程桌面的埠口是 3389 ，但是 xrdp 会再连到本机的 3350 去唤醒一个 VNC 的联机。
# 但是尚未联机之前，并不会起动任何的 VNC 埠口。
```
因为 xrdp 最终会自动启用 VNC ，因此你还是必须要安装 tigervnc-server 才行。

# SSH 服务器的进阶应用
使用 ssh 的加密通道就能够在客户端启动图形接口。
## 启动 ssh 在非正规埠口 (非 port 22)
### 设定 ssh 在 port 22 及 23 两个埠口的设定方式
```
[root@www ~]# vim /etc/ssh/sshd_config
Port 22
Port 23    <==要有两个 Port 的设定才行

[root@www ~]# /etc/init.d/sshd restart
```
这一版的 CentOS 却将 SSH 规范 port 仅能启动于 22 而已，所以此时会出现一个 SELinux 的错误，根据 setroubleshoot 的提示，我们必须要自行定义一个 SELinux 的规则放行模块。
```
# 1. 于 /var/log/audit/audit.log 找出与 ssh 有关的 AVC 信息，并转为本地模块
[root@www ~]# cat /var/log/audit/audit.log | grep AVC | grep ssh | \
>  audit2allow -m sshlocal > sshlocal.te  <==扩展名要是 .te 才行
[root@www ~]# grep sshd_t /var/log/audit/audit.log | \
>  audit2allow -M sshlocal  <==sshlocal 就是刚刚建立的 .te 档名
******************** IMPORTANT ***********************
To make this policy package active, execute:
semodule -i sshlocal.pp   <==这个指令会编译出这个重要的 .pp 模块！

# 2. 将这个模块加载系统的 SELinux 管理当中！
[root@www ~]# semodule -i sshlocal.pp

# 3. 再重新启动 sshd 并且观察埠口
[root@www ~]# /etc/init.d/sshd restart
[root@www ~]# netstat -tlunp | grep ssh
tcp        0      0 0.0.0.0:22   0.0.0.0:*    LISTEN      7322/sshd
tcp        0      0 0.0.0.0:23   0.0.0.0:*    LISTEN      7322/sshd
tcp        0      0 :::22        :::*         LISTEN      7322/sshd
tcp        0      0 :::23        :::*         LISTEN      7322/sshd
```
### 非正规埠口的联机方式
```
root@www ~]# ssh -p 23 root@localhost
root@localhost's password:
Last login: Tue Jul 26 14:07:41 2011 from 192.168.1.101
[root@www ~]# netstat -tnp | grep 23
tcp  0  0 ::1:23               ::1:56645              ESTABLISHED 7327/2
tcp  0  0 ::1:56645            ::1:23                 ESTABLISHED 7326/ssh
# 因为网络是双向的，因此自己连自己 (localhost)，就会抓到两只联机！
```
不要将 port 开放在某些既知的埠口上
## 以 rsync 进行同步镜像备份
rsync 最早是想要取代 rcp 这个指令的，因为 rsync 不但传输的速度快，而且他在传输时， 可以比对本地端与远程主机欲复制的档案内容，而仅复制两端有差异的档案而已，所以传输的时间就相对的降低很多。rsync 的传输方式至少可以透过三种方式来运作：
* 在本机上直接运作，用法就与 cp 几乎一模一样，例如：  
  rsync -av /etc /tmp (将 /etc/ 的数据备份到 /tmp/etc 内)
* 透过 rsh 或 ssh 的信道在 server / client 之间进行数据传输，例如：  
  rsync -av -e ssh user@rsh.server:/etc /tmp (将 rsh.server 的 /etc 备份到本地主机的 /tmp 内)
* 直接透过 rsync 提供的服务 (daemon) 来传输，此时 rsync 主机需要启动 873 port：  
    1. 你必须要在 server 端启动 rsync ， 看 /etc/xinetd.d/rsync 即可；
    2. 你必须编辑 /etc/rsyncd.conf 配置文件；
    3. 你必须设定好 client 端联机的密码数据；
    4. 在 client 端可以利用：rsync -av user@hostname::/dir/path /local/path

其实三种传输模式差异在于有没有冒号 (:) 而已，本地端传输不需要冒号，透过 ssh 或 rsh 时，就得要利用一个冒号 (:)， 如果是透过 rsync daemon 的话，就得要两个冒号 (::) 。
```
[root@www ~]# rsync [-avrlptgoD] [-e ssh] [user@host:/dir] [/local/path]
选项与参数：
-v ：观察模式，可以列出更多的信息，包括镜像时的档案档名等；
-q ：与 -v  相反，安静模式，略过正常信息，仅显示错误讯息；
-r ：递归复制！可以针对『目录』来处理！很重要！
-u ：仅更新 (update)，若目标档案较新，则保留新档案不会覆盖；
-l ：复制链接文件的属性，而非链接的目标源文件内容；
-p ：复制时，连同属性 (permission) 也保存不变！
-g ：保存源文件的拥有群组；
-o ：保存源文件的拥有人；
-D ：保存源文件的装置属性 (device)
-t ：保存源文件的时间参数；
-I ：忽略更新时间 (mtime) 的属性，档案比对上会比较快速；
-z ：在数据传输时，加上压缩的参数！
-e ：使用的信道协议，例如使用 ssh 通道，则 -e ssh
-a ：相当于 -rlptgoD ，所以这个 -a 是最常用的参数了！
更多说明请参考 man rsync 的解说！

# 1. 将 /etc 的数据备份到 /tmp 底下：
[root@www ~]# rsync -av /etc /tmp
....(前面省略)....
sent 21979554 bytes  received 25934 bytes  4000997.82 bytes/sec
total size is 21877999  speedup is 0.99
[root@www ~]# ll -d /tmp/etc /etc
drwxr-xr-x. 106 root root 12288 Jul 26 16:10 /etc
drwxr-xr-x. 106 root root 12288 Jul 26 16:10 /tmp/etc
# 第一次运作时会花比较久的时间，因为首次建立，如果再次备份

[root@www ~]# rsync -av /etc /tmp
sent 55716 bytes  received 240 bytes  111912.00 bytes/sec
total size is 21877999  speedup is 390.99
# 比较一下两次 rsync 的传输与接受数据量，你就会发现立刻就跑完了！
# 传输的数据也很少！因为再次比对，仅有差异的档案会被复制。

# 2. 利用 student 的身份登入 clientlinux.centos.vbird 将家目录复制到本机 /tmp
[root@www ~]# rsync -av -e ssh student@192.168.100.10:~ /tmp 
student@192.168.100.10's password:  <==输入对方主机的 student 密码
receiving file list ... done
student/
student/.bash_logout
....(中间省略)....
sent 110 bytes  received 697 bytes  124.15 bytes/sec
total size is 333  speedup is 0.41

[root@www ~]# ll -d /tmp/student
drwx------. 4 student student 4096 Jul 26 16:52 /tmp/student
```
## 通过 ssh 通道加密原本无加密的服务
假设服务器上面有启动了 VNC 服务在 port 5901 ，客户端则使用 vncviewer 要联机到服务器上的 port 5901 就是了。 那现在我们在客户端计算机上面启动一个 5911 的埠口，然后再透过本地端的 ssh 联机到服务器的 sshd 去，而服务器的 sshd 再去连接服务器的 VNC port 5901 。  
假设你已经透过上述各个小节建立好服务器 (www.centos.vbird) 上面的 VNC port 5901 ，而客户端则没有启动任何的 VNC 埠口。
```
[root@clientlinux ~]# ssh -L 本地埠口:127.0.0.1:远程端口 [-N] 远程主机
选项与参数：
-N ：仅启动联机通道，不登入远程 sshd 服务器
本地埠口：就是开启 127.0.0.1 上面一个监听的埠口
远程埠口：指定联机到后面远程主机的 sshd 后，sshd 该连到哪个埠口进行传输

# 1. 在客户端启动所需要的端口进行的指令
[root@clientlinux ~]# ssh -L 5911:127.0.0.1:5901 -N 192.168.100.254
root@192.168.100.254's password:
   <==登入远程仅是开启一个监听埠口，所以停止不能动作

# 2. 在客户端在另一个终端机测试看看，这个动作不需要作，只是查阅而已
[root@clientlinux ~]# netstat -tnlp| grep ssh
tcp  0   0 0.0.0.0:22           0.0.0.0:*            LISTEN      1330/sshd
tcp  0   0 127.0.0.1:5911       0.0.0.0:*            LISTEN      3347/ssh
tcp  0   0 :::22                :::*                 LISTEN      1330/sshd
[root@clientlinux ~]# netstat -tnap| grep ssh
tcp  0   0 192.168.100.10:55490 192.168.100.254:22   ESTABLISHED 3347/ssh
# 在客户端启动 5911 的埠口是 ssh 启动的，同一个 PID 也联机到远程
```
接下来你就可以在客户端 (192.168.100.10, clientlinux.centos.vbird) 使用『 vncviewer localhost:5911 』来联机， 但是该联机却会连到 www.centos.vbird (192.168.100.254) 那部主机的 port 5901 。
```
# 3. 在服务器端测试看看，这个动作不需要作，只是查阅而已
[root@www ~]# netstat -tnp | grep ssh
tcp   0  0 127.0.0.1:59442     127.0.0.1:5901        ESTABLISHED 7623/sshd: root
tcp   0  0 192.168.100.254:22  192.168.100.10:55490  ESTABLISHED 7623/sshd: root
# 明显的看到 port 22 的程序同时联机到 port 5901
```
要取消这个联机，先关闭 VNC 之后，然后再将 clientlinux.centos.vbird 的第一个动作 (ssh -L ...) 按下 [ctrl]-c 就中断这个加密通道了。
## 以 ssh 信道配合 X server 传递图形接口
