# 备份要点
## 备份数据的考量
### 造成系统损毁的问题——硬件问题
### 造成系统损毁的问题——软件问题
### 主机角色不同，备份任务也不同
如果是针对个人桌上型计算机使用的数据，那么 Norton 的『 Ghost 』应该算是一套较好的备份系统了。最主要是 Ghost 可以针对整个 partition 来进行备份，可以将 Windows 系统当中的整个 C 或者是整个 D 槽完整的备份下来。甚至在还原方面也是非常的快速，而且操作简便。另外，由于个人桌上型计算机所使用的数据量通常不大，所以当 ghost 完成之后，通常只要将数据烧录到光盘片当中，大约只要一至两片的光盘片也就绰绰有余。那么将光盘片保存好，这就是最简易的数据备份模式了。此外，由于个人的数据变动性不大，所以数据的备份频率方面也不需要非常的频繁。  
如果主机有提供 Internet 服务，备份频率需求。创建备份的策略 (频率、媒体、方法等) 是相当的重要的。
### 备份因素考量
* 备份哪些文件：  
  哪些数据对系统或使用者来说是重要的？那些数据就是值得备份的数据！例如 /etc/* 及 /home/* 等。
* 选择什么备份的媒介：  
  是可读写光盘、另一颗硬盘、同一颗硬盘的不同 partition、还是使用网络备援系统？ 哪一种的速度最快，最便宜，可将数据保存最久？这都可以考虑的。
* 考虑备份的方式：  
  是以完整备份(类似 ghost)来备份所有数据，还是使用差异备份仅备份有被更动过的数据即可？
* 备份的频率：  
  例如 MySQL 数据库是否天天备份、若完整备份，需要多久进行一次？
* 备份使用的工具为何：  
  是利用 tar 、 cpio 、 dd 还是 dump 等等的备份工具？
## 哪些 Linux 数据具有备份的意义
具有备份意义的文件通常可以粗分为两大类，一类是系统基本配置资讯、一类则是类似网络服务的内容数据。
### 操作系统本身需要备份的文件：
这方面的文件主要跟『帐号与系统配置档』有关系。 /etc/passwd, /etc/shadow, /etc/group, /etc/gshadow, /home 底下的使用者家目录等等， 而由于 Linux 默认的重要参数档都在 /etc/ 底下，所以只要将这个目录备份下来的话， 那么几乎所有的配置档都可以被保存。  
* /etc/ 整个目录
* /home 整个目录
* /var/spool/mail
* /boot
* /root
* 如果你自行安装过其他的套件，那么 /usr/local/ 或 /opt 也最好备份一下！
### 网络服务的数据库方面
* 软件本身的配置文件，例如：/etc/ 整个目录，/usr/local/ 整个目录
* 软件服务提供的数据，以 WWW 及 MySQL 为例：
* WWW 数据：/var/www 整个目录或 /srv/www 整个目录，及系统的使用者家目录
* MySQL ： /var/lib/mysql 整个目录
* 其他在 Linux 主机上面提供的服务之数据库文件
### 推荐需要备份的目录
* /boot
* /etc
* /home
* /root
* /usr/local(或者是 /opt 及 /srv 等)
* /var(注：这个目录当中有些缓存目录则可以不备份！)
### 不需要备份的目录
* /dev ：
* /proc：
* /mnt 与 /media：如果你没有在这个目录内放置你自己系统的东西，也不需要备份
* /tmp ：
## 备份用存储媒体的选择
### 异地备援系统
### 储存媒体的考量
* 备份速度要求——思考硬盘的用途  
* 储存容量——磁带备份考量
* 经费与数据可靠性——DVD 的使用，可保存 10 年左右

常见的装置代号：
* 光驱： /dev/cdrom (其实应该是 /dev/sdX 或 /dev/hdX)
* 磁带机： /dev/st0 (SCSI 介面), /dev/ht0 (IDE 介面)
* 软盘机： /dev/fd0, /dev/fd1
* 硬盘机： /dev/hd[a-d][1-63] (IDE), /dev/sd[a-p][1-16] (SCSI/SATA)
* 外接式 U盘 硬盘机： /dev/sd[a-p][1-16] (与 SCSI 相同)
* 打印机： /dev/lp[0-2]

# 备份的种类、频率与工具的选择
## 完整备份之累积备份 (Incremental backup)
### 还原的考量
如果是完整备份的话，只要将完整备份拿出来，整个给他倾倒回去硬盘， 有些时候 (例如使用 dd 命令) 甚至连系统都不需要重新安装。很多企业用来提供重要服务的主机都会使用完整备份， 若所提供的服务真的非常重要时，甚至会再架设一部一模一样的机器。若是原本的机器出问题，那就立刻将备份的机器拿出来接管，以使企业的网络服务不会中断。  
完整备份就是将根目录 (/) 整个系统通通备份下来。不过某些场合底下，完整备份也可以是备份一个文件系统 (filesystem)。
### 累积备份的原则
所谓的累积备份，指的是在系统在进行完第一次完整备份后，经过一段时间的运行， 比较系统与备份档之间的差异，仅备份有差异的文件而已。而第二次累积备份则与第一次累积备份的数据比较， 也是仅备份有差异的数据而已。如此一来，由于仅备份有差异的数据，因此备份的数据量小且快速，备份也很有效率。  
还原方面比较麻烦，需要从第一次备份开始，逐次往后还原。
### 累计备份使用的备份软件
完整备份常用的工具有 dd, cpio, dump/restore 等等。因为这些工具都能够备份装置与特殊文件。 dd 可以直接读取磁碟的磁区 (sector) 而不理会文件系统，是相当良好的备份工具，不过缺点就是慢很多。 cpio 是能够备份所有档名，不过，得要配合 find 或其他找档名的命令才能够处理妥当。以上两个都能够进行完整备份， 但累积备份就得要额外使用脚本程序来处理。可以直接进行累积备份的就是 dump 这个命令。
```
# 1. 用 dd 来将 /dev/sda 备份到完全一模一样的 /dev/sdb 硬盘上：
[root@www ~]# dd if=/dev/sda of=/dev/sdb
# 由于 dd 是读取磁区，所以 /dev/sdb 这颗磁碟可以不必格式化！非常的方便！
# 只是你会等非常非常久！因为 dd 的速度比较慢！

# 2. 使用 cpio 来备份与还原整个系统，假设储存媒体为 SATA 磁带机：
[root@www ~]# find / -print | cpio -covB > /dev/st0  <==备份到磁带机
[root@www ~]# cpio -iduv < /dev/st0                  <==还原
```
假设 /home 为一个独立的文件系统，而 /backupdata 也是一个独立的用来备份的文件系统，使用 dump 将 /home 完整的备份到 /backupdata 上
```
# 1. 完整备份
[root@www ~]# dump -0u -f /backupdata/home.dump /home

# 2. 第一次进行累积备份
[root@www ~]# dump -1u -f /backupdata/home.dump.1 /home
```
除了这些命令之外，其实 tar 也可以用来进行完整备份。举例来说，/backupdata 是个独立的文件系统， 你想要将整个系统通通备份起来时，可以这样考虑：将不必要的 /proc, /mnt, /tmp 等目录不备份，其他的数据则予以备份：
```
[root@www ~]# tar --exclude /proc --exclude /mnt --exclude /tmp --exclude /backupdata -jcvp -f /backupdata/system.tar.bz2 /
```
## 完整备份之差异备份 (Differential backup)
差异备份指的是：每次的备份都是与原始的完整备份比较的结果。所以系统运行的越久，离完整备份时间越长， 那么该次的差异备份数据可能就会越大。  
差异备份常用的工具与累积备份差不多，因为都需要完整备份。如果使用 dump 来备份的话，那么每次备份的等级 (level) 就都会是 level 1 的意思。也可以透过 tar 的 -N 选项来备份。
```
[root@www ~]# tar -N '2009-06-01' -jpcv -f /backupdata/home.tar.bz2 /home
# 只有在比 2009-06-01 还要新的文件，在 /home 底下的文件才会被打包进 home.bz2 中！
# 有点奇怪的是，目录还是会被记录下来，只是目录内的旧文件就不会备份。
```
此外，也可以通过 rsync 进行镜像备份。 rsync 可以对两个目录进行镜像 (mirror) ，算是一个非常快速的备份工具
```
[root@www ~]# rsync -av 来源目录 目标目录

# 1. 将 /home/ 镜像到 /backupdata/home/ 去
[root@www ~]# rsync -av /home /backupdata/
# 此时会在 /backupdata 底下产生 home 这个目录
[root@www ~]# rsync -av /home /backupdata/
# 再次进行会快很多！如果数据没有更动，几乎不会进行任何动作！
```
差异备份所使用的磁碟容量可能会比累积备份来的大，但是差异备份的还原较快， 因为只需要还原完整备份与最近一次的差异备份即可。
## 关键数据备份
如果想要分门别类的将各种不同的服务在不同的时间备份使用不同档名， tar 命令配合 date 命令是非常好用的工具
```
[root@www ~]# tar -jpcvf mysql.`date +%Y-%m-%d`.tar.bz2 /var/lib/mysql
```

# 作者的备份策略
将备份分为两大部分：一个是每日备份经常性变动的重要数据， 一个则是每周备份就不常变动的资讯。两个简单的 scripts ，分别来储存这些数据。
* 主机硬件：使用一个独立的 filesystem 来储存备份数据，此 filesystem 挂载到 /backup 当中；
* 每日进行：目前仅备份 MySQL 数据库；
* 每周进行：包括 /home, /var, /etc, /boot, /usr/local 等目录与特殊服务的目录；
* 自动处理：这方面利用 /etc/crontab 来自动提供备份的进行；
* 异地备援：每月定期的将数据分别 (a)烧录到光盘上面 (b)使用网络传输到另一部机器上面。
## 每周系统备份的 script
```
[root@www ~]# vi /backup/backupwk.sh
#!/bin/bash
# ====================================================================
# 使用者参数输入位置：
# basedir=你用来储存此脚本所预计备份的数据之目录(请独立文件系统)
basedir=/backup/weekly  <==您只要改这里就好了！

# ====================================================================
# 底下请不要修改了！用默认值即可！
PATH=/bin:/usr/bin:/sbin:/usr/sbin; export PATH
export LANG=C

# 配置要备份的服务的配置档，以及备份的目录
named=$basedir/named
postfixd=$basedir/postfix
vsftpd=$basedir/vsftp
sshd=$basedir/ssh
sambad=$basedir/samba
wwwd=$basedir/www
others=$basedir/others
userinfod=$basedir/userinfo
# 判断目录是否存在，若不存在则予以创建。
for dirs in $named $postfixd $vsftpd $sshd $sambad $wwwd $others $userinfod
do
	[ ! -d "$dirs" ] && mkdir -p $dirs
done

# 1. 将系统主要的服务之配置档分别备份下来，同时也备份 /etc 全部。
cp -a /var/named/chroot/{etc,var}	$named
cp -a /etc/postfix /etc/dovecot.conf	$postfixd
cp -a /etc/vsftpd/*			$vsftpd
cp -a /etc/ssh/*			$sshd
cp -a /etc/samba/*			$sambad
cp -a /etc/{my.cnf,php.ini,httpd}	$wwwd
cd /var/lib
  tar -jpc -f $wwwd/mysql.tar.bz2 	mysql
cd /var/www
  tar -jpc -f $wwwd/html.tar.bz2 	html cgi-bin
cd /
  tar -jpc -f $others/etc.tar.bz2	etc
cd /usr/
  tar -jpc -f $others/local.tar.bz2	local

# 2. 关于使用者参数方面
cp -a /etc/{passwd,shadow,group}	$userinfod
cd /var/spool
  tar -jpc -f $userinfod/mail.tar.bz2	mail
cd /
  tar -jpc -f $userinfod/home.tar.bz2	home
cd /var/spool
  tar -jpc -f $userinfod/cron.tar.bz2	cron at

[root@www ~]# chmod 700 /backup/backupwk.sh
[root@www ~]# /backup/backupwk.sh 
```
## 每日备份数据的 script
```
[root@www ~]# vi /backup/backupday.sh
#!/bin/bash
# =========================================================
# 请输入，你想让备份数据放置到那个独立的目录去
basedir=/backup/daily/  <==你只要改这里就可以了！

# =========================================================
PATH=/bin:/usr/bin:/sbin:/usr/sbin; export PATH
export LANG=C
basefile1=$basedir/mysql.$(date +%Y-%m-%d).tar.bz2
basefile2=$basedir/cgi-bin.$(date +%Y-%m-%d).tar.bz2
[ ! -d "$basedir" ] && mkdir $basedir

# 1. MysQL (数据库目录在 /var/lib/mysql)
cd /var/lib
  tar -jpc -f $basefile1 mysql

# 2. WWW 的 CGI 程序 (如果有使用 CGI 程序的话)
cd /var/www
  tar -jpc -f $basefile2 cgi-bin

[root@www ~]# chmod 700 /backup/backupday.sh
[root@www ~]# /backup/backupday.sh
```
```
[root@www ~]# vi /etc/crontab
30 3 * * 0 root /backup/backupwk.sh
30 2 * * * root /backup/backupday.sh
```
> 有些时候，你在进行备份时，被备份的文件可能同时间被其他的网络服务所修改。例如，备份 MySQL 数据库时，刚好有人利用你的数据库发表文章，此时， 可能会发生一些错误的信息。要避免这类的问题时，可以在备份前，将该服务先关掉， 备份完成后，再启动该服务即可。
## 远程备援的 script
### 使用 FTP 上次备份数据
```
[root@www ~]# vi /backup/ftp.sh
#!/bin/bash
# ===========================================
# 先输入系统所需要的数据
host="192.168.1.100"		# 远程主机
id="dmtsai"			# 远程主机的 FTP 帐号
pw='dmtsai.pass'		# 该帐号的口令
basedir="/backup/weekly"	# 本地端的欲被备份的目录
remotedir="/home/backup"	# 备份到远程的何处？

# ===========================================
backupfile=weekly.tar.bz2
cd $basedir/..
  tar -jpc -f $backupfile $(basename $basedir)

ftp -n "$host" > ${basedir}/../ftp.log 2>&1 <<EOF
user $id $pw
binary
cd $remotedir
put $backupfile
bye
EOF
```
### 使用 rsync 上传备份数据
必须要在你的服务器上面取得某个帐号使用权后， 并让该帐号可以不用口令即可登陆才行。
```
[root@www ~]# vi /backup/rsync.sh
#!/bin/bash
remotedir=/home/backup/
basedir=/backup/weekly
host=127.0.0.1
id=dmtsai

rsync -av -e ssh $basedir ${id}@${host}:${remotedir}
```
由于 rsync 可以透过 ssh 来进行镜像备份，所以没有变更的文件将不需要上传的。

# 灾难复原的考量
## 硬件损毁，且具有完整备份的数据时
不需要考虑系统软件的不稳定问题，所以可以直接将完整的系统复原回去即可。
## 由于软件的问题产生的被攻破资安事件
由于系统的损毁是因为被攻击，此时即使你恢复到正常的系统，那么这个系统既然会被攻破。，此时完整备份的复原可能不是个好方式，最好是需要这样进行：
1. 先拔除网络线，最好将系统进行完整备份到其他媒体上，以备未来查验
2. 开始查阅登录文件，尝试找出各种可能的问题
3. 开始安装新系统 (最好找最新的 distribution)
4. 进行系统的升级，与防火墙相关机制的制订
5. 根据 2 的错误，在安装完成新系统后，将那些 bug 修复
6. 进行各项服务与相关数据的恢复
7. 正式上线提供服务，并且开始测试