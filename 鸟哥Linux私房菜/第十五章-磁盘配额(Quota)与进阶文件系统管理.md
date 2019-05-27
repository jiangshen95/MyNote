# 磁盘配额 (Quota) 的应用与实作
使用 quota ，多少容量限制，来让磁碟的容量使用较为公平。
## 什么是 Quota
在 Linux 系统中，由于是多人多工的环境，所以会有多人共同使用一个磁盘空间的情况发生， 如果其中有少数几个使用者大量的占掉了磁盘空间的话，那势必压缩其他使用者的使用权力！ 因此管理员应该适当的限制硬盘的容量给使用者，以妥善的分配系统资源。
### Quota 的一般用途
* 针对 WWW server ，例如：每个人的网页空间的容量限制！
* 针对 mail server，例如：每个人的邮件空间限制。
* 针对 file server，例如：每个人最大的可用网络磁盘空间

上述是针对网络服务的设计，如果针对 Linux 系统主机上面的配置：
* 限制某一群组所能使用的最大磁碟配额 (使用群组限制)：
* 限制某一使用者的最大磁碟配额 (使用使用者限制)：
* 以 Link 的方式，来使邮件可以作为限制的配额 (更改 /var/spool/mail 这个路径)：  
  通常在原本磁盘分区的规划不好，但又不想更动原有主机架构的情况。

### Quota 的使用限制
* 仅能针对整个 filesystem：  
  quota 实际在运行的时候，是针对『整个 filesystem』进行限制的， 例如：如果你的 /dev/sda5 是挂载在 /home 底下，那么在 /home 底下的所有目录都会受到限制！
* 核心必须支持 quota ：
* Quota 的记录档：
* 只对一般身份使用者有效：

### Quota 的规范配置项目
quota 针对整个 filesystem 的限制项目主要分为如下几个部分：
* 容量限制或文件数量限制 (block 或 inode)：  
    * 限制 inode 用量：可以管理使用者可以创建的【文件数量】
    * 限制 block 用量：管理使用者磁碟容量的限制，较为常见的为这种方式。
* 柔性劝导与硬性设定 (soft/hard)：  
  不管是 inode/block ，限制值都有两个，分别是 soft 与 hard。通常 hard 限制值要比 soft 还要高。  
    * hard：表示使用者的用量绝对不会超过这个限制值，若超过这个值则系统会锁住该用户的磁碟使用权；
    * soft：表示使用者在低于 soft 限值时，可以正常使用磁碟，但若超过 soft 且低于 hard 的限值，每次使用者登陆系统时，系统会主动发出磁碟即将爆满的警告信息， 且会给予一个宽限时间 (grace time)。不过，若使用者在宽限时间倒数期间就将容量再次降低于 soft 限值之下， 则宽限时间会停止。
* 会倒计时的宽限时间 (grace time)：  
  一般默认的宽限时间为七天，如果七天内你都不进行任何磁碟管理，那么 soft 限制值会即刻取代 hard 限值来作为 quota 的限制。

## 实作 Quota 流程-1：文件系统支持
```
mount -o remount,usrquota,grpquota /home

写入配置档中
[root@www ~]# vi /etc/fstab
LABEL=/home   /home  ext3   defaults,usrquota,grpquota  1 2
```
## 实作 Quota 流程-2：创建 quota 记录档
Quota 是透过分析整个文件系统中，每个使用者(群组)拥有的文件总数与总容量， 再将这些数据记录在该文件系统的最顶层目录，然后在该记录档中再使用每个帐号(或群组)的限制值去规范磁碟使用量的。建置 Quota 记录档就非常重要。扫瞄有支持 Quota 参数 (usrquota, grpquota) 的文件系统， 就使用 quotacheck 这个命令。
### quotacheck：扫描文件系统并创建 Quota 的记录档
```
[root@www ~]# quotacheck [-avugfM] [/mount_point]
选项与参数：
-a  ：扫瞄所有在 /etc/mtab 内，含有 quota 支持的 filesystem，加上此参数后， 
      /mount_point 可不必写，因为扫瞄所有的 filesystem
-u  ：针对使用者扫瞄文件与目录的使用情况，会创建 aquota.user
-g  ：针对群组扫瞄文件与目录的使用情况，会创建 aquota.group
-v  ：显示扫瞄过程的资讯；
-f  ：强制扫瞄文件系统，并写入新的 quota 配置档 (危险)
-M  ：强制以读写的方式扫瞄文件系统，只有在特殊情况下才会使用。
```
quotacheck 的选项你只要记得『 -avug 』一起下达即可。 -f 与 -M 是在文件系统可能已经启动 quota 了， 但是你还想要重新扫瞄文件系统时，系统会要求你加入那两个选项。
```
针对整个系统含有 usrquota, grpquota 参数的文件系统进行 quotacheck 扫瞄
[root@www ~]# quotacheck -avug
quotacheck: Scanning /dev/hda3 [/home] quotacheck: Cannot stat old user quota
file: No such file or directory <==有找到文件系统，但尚未制作记录档！
quotacheck: Cannot stat old group quota file: No such file or directory
quotacheck: Cannot stat old user quota file: No such file or directory
quotacheck: Cannot stat old group quota file: No such file or directory
done  <==上面三个错误只是说明记录档尚未创建而已，可以忽略不理！
quotacheck: Checked 130 directories and 107 files <==实际搜寻结果
quotacheck: Old file not found.
quotacheck: Old file not found.
# 若运行这个命令却出现如下的错误信息，表示你没有任何文件系统有启动 quota 支持！
# quotacheck: Can't find filesystem to check or filesystem not mounted with 
# quota option.

[root@www ~]# ll -d /home/a*
-rw------- 1 root root 8192 Mar  6 11:58 /home/aquota.group
-rw------- 1 root root 9216 Mar  6 11:58 /home/aquota.user
# /home 是独立的文件系统，因此搜寻结果会将两个记录档放在 
# /home 底下。这两个文件就是 Quota 最重要的资讯了！
```
若要强制重新进行 quotacheck 的动作。
```
如果因为特殊需求需要强制扫瞄已挂载的文件系统时
[root@www ~]# quotacheck -avug -mf
quotacheck: Scanning /dev/hda3 [/home] done
quotacheck: Checked 130 directories and 109 files
# 数据要简洁很多，因为有记录档存在，所以警告信息不会出现。
```
创建的两个文件是 quota 自己的数据文件，并不是纯文字档，且会一直变动。
## 实作 Quota 流程-3：Quota 启动，关闭与限制值配置
### quotaon：启动 quota 的服务
```
[root@www ~]# quotaon [-avug]
[root@www ~]# quotaon [-vug] [/mount_point]
选项与参数：
-u  ：针对使用者启动 quota (aquota.user)
-g  ：针对群组启动 quota (aquota.group)
-v  ：显示启动过程的相关信息；
-a  ：根据 /etc/mtab 内的 filesystem 配置启动有关的 quota ，若不加 -a 的话，
      则后面就需要加上特定的那个 filesystem.
```
这个『 quotaon -auvg 』的命令几乎只在第一次启动 quota 时才需要进行！因为下次等你重新启动系统时， 系统的 /etc/rc.d/rc.sysinit 这个初始化脚本就会自动的下达这个命令。
### quotaoff：关闭 quota 的服务
```
[root@www ~]# quotaoff [-a]
[root@www ~]# quotaoff [-ug] [/mount_point]
选项与参数：
-a  ：全部的 filesystem 的 quota 都关闭 (根据 /etc/mtab)
-u  ：仅针对后面接的那个 /mount_point 关闭 user quota
-g  ：仅针对后面接的那个 /mount_point 关闭 group quota
```
### edquota：编辑账号/群组的限制值与宽限时间
```
[root@www ~]# edquota [-u username] [-g groupname]
[root@www ~]# edquota -t  <==修改宽限时间
[root@www ~]# edquota -p 范本帐号 -u 新帐号
选项与参数：
-u  ：后面接帐号名称。可以进入 quota 的编辑画面 (vi) 去配置 username 的限制值；
-g  ：后面接群组名称。可以进入 quota 的编辑画面 (vi) 去配置 groupname 的限制值；
-t  ：可以修改宽限时间。
-p  ：复制范本。那个 范本帐号 为已经存在并且已配置好 quota 的使用者，
      意义为『将 范本帐号 这个人的 quota 限制值复制给 新帐号 』！
```
```
配置 dmtsai 这个使用者的 quota 限制值
[root@www ~]# edquota -u myquota1
Disk quotas for user myquota1 (uid 710):
  Filesystem    blocks  soft   hard  inodes  soft  hard
  /dev/hda3         80     0      0      10     0     0
```
第一行在说明针对哪个帐号 (myquota1) 进行 quota 的限额配置，第二行则是标头行，里面共分为七个栏位， 七个栏位分别的意义为：
* 文件系统 (filesystem)：说明该限制值是针对哪个文件系统 (或 partition)；
* 磁碟容量 (blocks)：这个数值是 quota 自己算出来的，单位为 Kbytes，请不要更动他；
* soft：磁碟容量 (block) 的 soft 限制值，单位亦为 KB
* hard：block 的 hard 限制值，单位 KB；
* 文件数量 (inodes)：这是 quota 自己算出来的，单位为个数，请不要更动他；
* soft：inode 的 soft 限制值；
* hard：inode 的 hard 限制值；

当 soft/hard 为 0 时，表示没有限制的意思。
## 实作 Quota 流程-4：Quota 限制值的报表
quota 的报表主要有两种模式，一种是针对每个个人或群组的 quota 命令，一个是针对整个文件系统的 repquota 命令。
### quota：单一用户的 quota 报表
```
[root@www ~]# quota [-uvs] [username]
[root@www ~]# quota [-gvs] [groupname]
选项与参数：
-u  ：后面可以接 username ，表示显示出该使用者的 quota 限制值。若不接 username 
      ，表示显示出运行者的 quota 限制值。
-g  ：后面可接 groupname ，表示显示出该群组的 quota 限制值。
-v  ：显示每个用户在 filesystem 的 quota 值；
-s  ：使用 1024 为倍数来指定单位，会显示如 M 之类的单位！
```
### repquota：针对文件系统的限额做报表
```
[root@www ~]# repquota -a [-vugs]
选项与参数：
-a  ：直接到 /etc/mtab 搜寻具有 quota 标志的 filesystem ，并报告 quota 的结果；
-v  ：输出的数据将含有 filesystem 相关的细部资讯；
-u  ：显示出使用者的 quota 限值 (这是默认值)；
-g  ：显示出个别群组的 quota 限值。
-s  ：使用 M, G 为单位显示结果
```
## 实作 Quota 流程-5：测试与管理
### warnquota：对超过限额者发出警告信
可以依据 /etc/warnquota.conf 的配置，然后找出目前系统上面 quota 用量超过 soft (就是有 grace time 出现的) 的帐号，透过 email 的功能将警告信件发送到使用者的电子邮件信箱。 warnquota 并不会自动运行，所以我们需要手动去运行他。单纯运行『 warnquota 』之后，他会发送两封信出去， 一封给 myquota1 一封给 root 。
```
[root@www ~]# warnquota
```
这个方法并不适用在 /var/spool/mail 也爆表的 quota 控管中，因为如果使用者在这个 filesystem 的容量已经爆表，那么新的信件就收不到。让系统自动运行 warnquota。
```
[root@www ~]# vi /etc/cron.daily/warnquota
/usr/sbin/warnquota
# 只要这一行，且将运行档以绝对路径的方式写入即可！

[root@www ~]# chmod 755 /etc/cron.daily/warnquota
```
### setquota：直接于命令中配置 quota 限额
如果想要使用 script 的方法来创建大量的账号，并将所有的账号都在创建时就给予 quota。
* 先创建一个原始 quota 账号，再以『 edquota -p old -u new 』写入 script 中；
* 直接以 setquota 创建用户的 quota 配置值。

不同于 edquota 时呼叫 vi 来进行配置，setquota 直接由命令输入所必要的各项限制值。
```
[root@www ~]# setquota [-u|-g] 名称 block(soft) block(hard) inode(soft) inode(hard) 文件系统
```
## 不更动既有系统的 quota 实例
如果你的主机原先没有想到要配置成为邮件主机，所以并没有规划将邮件信箱所在的 /var/spool/mail/ 目录独立成为一个 partition ，并且不想新增或分割出新的分割槽。假设已经有 /home 这个独立的分割槽了，可进行以下操作：
1. 将 /var/spool/mail 这个目录完整的移动到 /home 底下；
2. 利用 ln -s /home/mail /var/spool/mail 来创建连结数据；
3. 将 /home 进行 quota 限额配置
> 目前新的 distributions 大多有使用 SELinux 的机制， 因此你要进行如同上面的目录搬移时，在许多情况下可能会有使用上的限制，或许你得要先暂时关闭 SELinux 才能测试， 也或许你得要自行修改 SELinux 的守则。

# 软件磁盘阵列 (Software RAID)
## 什么是 RAID
磁盘阵列全名是【Redundant Arrays of Indexpensive Disks, RAID】，容错式廉价磁盘阵列。 RAID 可以透过一个技术(软件或硬件)，将多个较小的磁碟整合成为一个较大的磁碟装置； 而这个较大的磁碟功能不止是储存，还具有数据保护的功能。真个 RAID 由于选择的等级(level)不同，而使得整合后的磁碟具有不同的功能。
### RAID-0 (等量模式：stripe)：效能最佳
这种模式如果使用相同型号与容量的磁碟来组成时，效果较佳。这种模式的 RAID 会将磁碟先切出等量的区块 (举例来说， 4KB)， 然后当一个文件要写入 RAID 时，该文件会依据区块的大小切割好，之后再依序放到各个磁碟里面去。由于每个磁碟会交错的存放数据， 因此当你的数据要写入 RAID 时，数据会被等量的放置在各个磁碟上面。  
 当有数据要写入 RAID 时，数据会先被切割成符合小区块的大小，然后再依序一个一个的放置到不同的磁碟去。 由于数据已经先被切割并且依序放置到不同的磁碟上面，因此每颗磁碟所负责的数据量都降低了。  
 只是使用此等级你必须要自行负担数据损毁的风险，由于文件是被切割成为适合每颗磁盘分区区块的大小， 然后再依序放置到各个磁碟中。如果某一颗磁碟损毁了，那么文件数据将缺一块，此时这个文件就损毁了。 由于每个文件都是这样存放的，因此 RAID-0 只要有任何一颗磁碟损毁，在 RAID 上面的所有数据都会遗失而无法读取。  
 另外，如果使用不同容量的磁碟来组成 RAID-0 时，由于数据是一直等量的依序放置到不同磁碟中，当小容量磁碟的区块被用完了， 那么所有的数据都将被写入到最大的那颗磁碟去。效能就变差了。
 ### RAID-1 (映射模式：mirror)：完整备份
 这种模式也是需要相同的磁碟容量的，最好是一模一样的磁碟。如果是不同容量的磁碟组成 RAID-1 时，那么总容量将以最小的那一颗磁碟为主。这种模式主要是『让同一份数据，完整的保存在两颗磁碟上头』。  
 由于同一份数据会被分别写入到其他不同磁碟，因此如果要写入 100MB 时，数据传送到 I/O 汇流排后会被复制多份到各个磁碟， 结果就是数据量感觉变大了！因此在大量写入 RAID-1 的情况下，写入的效能可能会变的非常差。 好在如果你使用的是硬件 RAID (磁盘阵列卡) 时，磁盘阵列卡会主动的复制一份而不使用系统的 I/O 汇流排，效能方面则还可以。 如果使用软件磁盘阵列，可能效能就不好了。  
 RAID-1 最大的优点大概就在于数据的备份。由于磁碟容量有一半用在备份， 因此总容量会是全部磁碟容量的一半。虽然 RAID-1 的写入效能不佳，不过读取的效能较佳，这是因为数据有两份在不同的磁碟上面，如果多个 processes 在读取同一笔数据时， RAID 会自行取得最佳的读取平衡。
 ### RAID 0+1, RAID 1+0
 所谓的 RAID 0+1 就是： (1)先让两颗磁碟组成 RAID 0，并且这样的配置共有两组； (2)将这两组 RAID 0 再组成一组 RAID 1。反过来说，RAID 1+0 就是先组成 RAID-1 再组成 RAID-0 的意思。
 ### RAID 5：效能与数据备份的均衡考量
 RAID-5 至少需要三颗以上的磁碟才能够组成这种类型的磁盘阵列。这种磁盘阵列的数据写入有点类似 RAID-0 ， 不过每个循环的写入过程中，在每颗磁碟还加入一个同位检查数据 (Parity) ，这个数据会记录其他磁碟的备份数据， 用于当有磁碟损毁时的救援。  
 每个循环写入时，都会有部分的同位检查码 (parity) 被记录起来，并且记录的同位检查码每次都记录在不同的磁碟， 因此，任何一个磁碟损毁时都能够藉由其他磁碟的检查码来重建原本磁碟内的数据。由于有同位检查码，因此 RAID 5 的总容量会是整体磁碟数量减一颗。以上图为例， 原本的 3 颗磁碟只会剩下 (3-1)=2 颗磁碟的容量。而且当损毁的磁碟数量大于等于两颗时，这整组 RAID 5 的数据就损毁了。 因为 RAID 5 默认仅能支持一颗磁碟的损毁情况。  
 另外，由于 RAID 5 仅能支持一颗磁碟的损毁，因此近来还有发展出另外一种等级，就是 RAID 6 ，这个 RAID 6 则使用两颗磁碟的容量作为 parity 的储存，因此整体的磁碟容量就会少两颗，但是允许出错的磁碟数量就可以达到两颗，也就是在 RAID 6 的情况下，同时两颗磁碟损毁时，数据还是可以救回来。
 ### Spare Disk：预备磁碟的功能：
 当磁盘阵列的磁碟损毁时，就得要将坏掉的磁碟拔除，然后换一颗新的磁碟。换成新磁碟并且顺利启动磁盘阵列后， 磁盘阵列就会开始主动的重建 (rebuild) 原本坏掉的那颗磁碟数据到新的磁碟上！然后你磁盘阵列上面的数据就复原了！ 这就是磁盘阵列的优点。不过，我们还是得要动手拔插硬盘，此时通常得要关机才能这么做。  
 为了让系统可以即时的在坏掉硬盘时主动的重建，因此就需要预备磁碟 (spare disk) 的辅助。所谓的 spare disk 就是一颗或多颗没有包含在原本磁盘阵列等级中的磁碟，这颗磁碟平时并不会被磁盘阵列所使用， 当磁盘阵列有任何磁碟损毁时，则这颗 spare disk 会被主动的拉进磁盘阵列中，并将坏掉的那颗硬盘移出磁盘阵列。然后立即重建数据系统。若磁盘阵列支持热插拔，那么可以直接将坏掉的那颗磁碟拔除换一颗新的，再将新的配置成为 spare disk。  
### 磁盘阵列的优点
* 数据安全与可靠性：指的并非资讯安全，而是当硬件 (指磁碟) 损毁时，数据是否还能够安全的救援或使用之意；
* 读写效能：例如 RAID 0 可以加强读写效能，让你的系统 I/O 部分得以改善；
* 容量：可以让多颗磁碟组合起来，故单一文件系统可以有相当大的容量。
## software, hardware RAID
所谓的硬件磁盘阵列 (hardware RAID) 是透过磁盘阵列卡来达成阵列的目的。 磁盘阵列卡上面有一块专门的芯片在处理 RAID 的任务，因此在效能方面会比较好。在很多任务 (例如 RAID 5 的同位检查码计算) 磁盘阵列并不会重复消耗原本系统的 I/O 汇流排，理论上效能会较佳。此外目前一般的中高阶磁盘阵列卡都支持热拔插， 亦即在不关机的情况下抽换损坏的磁碟，对于系统的复原与数据的可靠性方面非常的好用。  
软件磁盘阵列主要是透过软件来模拟阵列的任务， 因此会损耗较多的系统资源，比如说 CPU 的运算与 I/O 汇流排的资源等。  
CentOS 提供的软件磁盘阵列为 mdadm 这套软件，这套软件会以 partition 或 disk 为磁碟的单位，也就是说，你不需要两颗以上的磁碟，只要有两个以上的分割槽 (partition) 就能够设计你的磁盘阵列了。此外， mdadm 还支持 RAID0/RAID1/RAID5/spare disk 等！ 而且提供的管理机制还可以达到类似热拔插的功能，可以线上 (文件系统正常使用) 进行分割槽的抽换，使用上也非常的方便。  
硬件磁盘阵列再 Linux 底下看起来就是一颗实际的大磁盘，因此硬件磁盘阵列的装置档名为 /dev/sd[a-p]，因为使用 SCSI 的模块之故。至于软件磁盘阵列则是系统模拟的，因此使用的装置档名是系统的装置档， 档名为 /dev/md0, /dev/md1...，两者的装置档名并不相同。
## 软件磁盘阵列的配置
```
[root@www ~]# mdadm --detail /dev/md0
[root@www ~]# mdadm --create --auto=yes /dev/md[0-9] --raid-devices=N \
> --level=[015] --spare-devices=N /dev/sdx /dev/hdx...
选项与参数：
--create ：为创建 RAID 的选项；
--auto=yes ：决定创建后面接的软件磁盘阵列装置，亦即 /dev/md0, /dev/md1...
--raid-devices=N ：使用几个磁碟 (partition) 作为磁盘阵列的装置
--spare-devices=N ：使用几个磁碟作为备用 (spare) 装置
--level=[015] ：配置这组磁盘阵列的等级。支持很多，不过建议只要用 0, 1, 5 即可
--detail ：后面所接的那个磁盘阵列装置的详细资讯
```
最后面会接许多的装置档名，这些装置档名可以是整颗磁碟，例如 /dev/sdb ， 也可以是分割槽，例如 /dev/sdb1 之类。不过，这些装置档名的总数必须要等于 --raid-devices 与 --spare-devices 的个数总和才行！
### 以 mdadm 建置 RAID
```
[root@www ~]# mdadm --create --auto=yes /dev/md0 --level=5 \
> --raid-devices=4 --spare-devices=1 /dev/hda{6,7,8,9,10}

[root@www ~]# mdadm --detail /dev/md0
/dev/md0:                                        <==RAID 装置档名
        Version : 00.90.03
  Creation Time : Tue Mar 10 17:47:51 2009       <==RAID 被创建的时间
     Raid Level : raid5                          <==RAID 等级为 RAID 5
     Array Size : 2963520 (2.83 GiB 3.03 GB)     <==此 RAID 的可用磁碟容量
  Used Dev Size : 987840 (964.85 MiB 1011.55 MB) <==每个装置的可用容量
   Raid Devices : 4                              <==用作 RAID 的装置数量
  Total Devices : 5                              <==全部的装置数量
Preferred Minor : 0
    Persistence : Superblock is persistent

    Update Time : Tue Mar 10 17:52:23 2009
          State : clean
 Active Devices : 4                              <==启动的(active)装置数量
Working Devices : 5                              <==可动作的装置数量
 Failed Devices : 0                              <==出现错误的装置数量
  Spare Devices : 1                              <==预备磁碟的数量

         Layout : left-symmetric
     Chunk Size : 64K      <==就是图2.1.4内的小区块

           UUID : 7c60c049:57d60814:bd9a77f1:57e49c5b <==此装置(RAID)识别码
         Events : 0.2

    Number   Major   Minor   RaidDevice State
       0       3        6        0      active sync   /dev/hda6
       1       3        7        1      active sync   /dev/hda7
       2       3        8        2      active sync   /dev/hda8
       3       3        9        3      active sync   /dev/hda9

       4       3       10        -      spare   /dev/hda10
# 最后五行就是这五个装置目前的情况，包括四个 active sync 一个 spare ！
# 至于 RaidDevice  指的则是此 RAID 内的磁碟顺序
```
出此命令外，也可以查阅 /proc/mdstat 来观察系统软件磁盘阵列的情况：
```
[root@www ~]# cat /proc/mdstat
Personalities : [raid6] [raid5] [raid4]
md0 : active raid5 hda9[3] hda10[4](S) hda8[2] hda7[1] hda6[0]    <==第一行
      2963520 blocks level 5, 64k chunk, algorithm 2 [4/4] [UUUU] <==第二行

unused devices: <none>
```
* 第一行部分：指出 md0 为 raid5 ，且使用了 hda9, hda8, hda7, hda6 等四颗磁碟装置。每个装置后面的中括号 [] 内的数字为此磁碟在 RAID 中的顺序 (RaidDevice)；至于 hda10 后面的 [S] 则代表 hda10 为 spare 之意。
* 第二行：此磁盘阵列拥有 2963520 个block(每个 block 单位为 1K)，所以总容量约为 3GB， 使用 RAID 5 等级，写入磁碟的小区块 (chunk) 大小为 64K，使用 algorithm 2 磁盘阵列演算法。 [m/n] 代表此阵列需要 m 个装置，且 n 个装置正常运行。因此本 md0 需要 4 个装置且这 4 个装置均正常运行。 后面的 [UUUU] 代表的是四个所需的装置 (就是 [m/n] 里面的 m) 的启动情况，U 代表正常运行，若为 _ 则代表不正常。
### 格式化与挂载使用 RAID
```
[root@www ~]# mkfs -t ext3 /dev/md0   # /dev/md0 做为装置被格式化

[root@www ~]# mkdir /mnt/raid
[root@www ~]# mount /dev/md0 /mnt/raid
```
## 模拟 RAID 错误的救援模式
```
[root@www ~]# mdadm --manage /dev/md[0-9] [--add 装置] [--remove 装置] [--fail 装置] 
选项与参数：
--add ：会将后面的装置加入到这个 md 中！
--remove ：会将后面的装置由这个 md 中移除
--fail ：会将后面的装置配置成为出错的状态
```
### 配置磁碟为错误 (fault)
```
[root@www ~]# mdadm --manage /dev/md0 --fail /dev/hda8
mdadm: set /dev/hda8 faulty in /dev/md0

[root@www ~]# mdadm --detail /dev/md0
....(前面省略)....
          State : clean, degraded, recovering
 Active Devices : 3
Working Devices : 4
 Failed Devices : 1  <==出错的磁碟有一个！
  Spare Devices : 1
....(中间省略)....
    Number   Major   Minor   RaidDevice State
       0       3        6        0      active sync   /dev/hda6
       1       3        7        1      active sync   /dev/hda7
       4       3       10        2      spare rebuilding   /dev/hda10
       3       3        9        3      active sync   /dev/hda9

       5       3        8        -      faulty spare   /dev/hda8
# 这的动作要快做才会看到 /dev/hda10 启动了而 /dev/hda8 死掉了

[root@www ~]# cat /proc/mdstat
Personalities : [raid6] [raid5] [raid4]
md0 : active raid5 hda9[3] hda10[4] hda8[5](F) hda7[1] hda6[0]
      2963520 blocks level 5, 64k chunk, algorithm 2 [4/3] [UU_U]
      [>.......]  recovery =  0.8% (9088/987840) finish=14.3min speed=1136K/sec
```
RAID 5 重建完毕
```
[root@www ~]# mdadm --detail /dev/md0
....(前面省略)....
    Number   Major   Minor   RaidDevice State
       0       3        6        0      active sync   /dev/hda6
       1       3        7        1      active sync   /dev/hda7
       2       3       10        2      active sync   /dev/hda10
       3       3        9        3      active sync   /dev/hda9

       4       3        8        -      faulty spare   /dev/hda8

[root@www ~]# cat /proc/mdstat
Personalities : [raid6] [raid5] [raid4]
md0 : active raid5 hda9[3] hda10[2] hda8[4](F) hda7[1] hda6[0]
      2963520 blocks level 5, 64k chunk, algorithm 2 [4/4] [UUUU]
```
### 将出错的磁碟移除并加入新磁碟
```
创建新的分割槽
[root@www ~]# fdisk /dev/hda
Command (m for help): n
First cylinder (2668-5005, default 2668): <==这里按 [enter]
Using default value 2668
Last cylinder or +size or +sizeM or +sizeK (2668-5005, default 5005): +1000M

Command (m for help): w

[root@www ~]# partprobe
# 此时系统会多一个 /dev/hda11 的分割槽

# 4. 加入新的拔除有问题的磁碟
[root@www ~]# mdadm --manage /dev/md0 --add /dev/hda11 --remove /dev/hda8
mdadm: added /dev/hda11
mdadm: hot removed /dev/hda8

[root@www ~]# mdadm --detail /dev/md0
....(前面省略)....
       0       3        6        0      active sync   /dev/hda6
       1       3        7        1      active sync   /dev/hda7
       2       3       10        2      active sync   /dev/hda10
       3       3        9        3      active sync   /dev/hda9

       4       3       11        -      spare   /dev/hda11
```
## 启动自动启动 RAID 并自动挂载
新的 distribution 大多会自己搜寻 /dev/md[0-9] 然后在启动的时候给予配置好所需要的功能。software RAID 的配置档在 /etc/mdadm.conf。
```
[root@www ~]# mdadm --detail /dev/md0 | grep -i uuid
        UUID : 7c60c049:57d60814:bd9a77f1:57e49c5b
# 后面那一串数据，就是这个装置向系统注册的 UUID 识别码！

# 开始配置 mdadm.conf
[root@www ~]# vi /etc/mdadm.conf
ARRAY /dev/md0 UUID=7c60c049:57d60814:bd9a77f1:57e49c5b
#     RAID装置      识别码内容

# 开始配置启动自动挂载并测试
[root@www ~]# vi /etc/fstab
/dev/md0    /mnt/raid    ext3    defaults     1 2

[root@www ~]# umount /dev/md0; mount -a
[root@www ~]# df /mnt/raid
Filesystem           1K-blocks      Used Available Use% Mounted on
/dev/md0               2916920    188464   2580280   7% /mnt/raid
```
## 关闭软件 RAID
如果你只是将 /dev/md0 卸载，然后忘记将 RAID 关闭， 未来你在重新分割 /dev/hdaX 时可能会出现一些莫名的错误状况。
```
# 1. 先卸载且删除配置档内与这个 /dev/md0 有关的配置：
[root@www ~]# umount /dev/md0
[root@www ~]# vi /etc/fstab
/dev/md0    /mnt/raid     ext3    defaults      1 2
# 将这一行删除掉！或者是注解掉也可以！

# 2. 直接关闭 /dev/md0 的方法！
[root@www ~]# mdadm --stop /dev/md0
mdadm: stopped /dev/md0

[root@www ~]# cat /proc/mdstat
Personalities : [raid6] [raid5] [raid4]
unused devices: <none>  <==确实不存在任何阵列装置

[root@www ~]# vi /etc/mdadm.conf
ARRAY /dev/md0 UUID=7c60c049:57d60814:bd9a77f1:57e49c5b
# 删除或是注解
```
> 最好由多颗磁碟来组成软件 RAID，效能较佳，而且某颗磁碟损坏时数据才能藉由其他磁碟挽救。

# 逻辑卷轴管理员 (Logical Volume Manager)
LVM 的重点在于『可以弹性的调整 filesystem 的容量！』 LVM 可以整合多个实体 partition 在一起， 让这些 partitions 看起来就像是一个磁碟一样！而且，还可以在未来新增或移除其他的实体 partition 到这个 LVM 管理的磁碟当中。 如此一来，整个磁碟空间的使用上，具有相当的弹性。
## 什么是 LVM：PV, PE, VG, LV 的意义
LVM 的全名是 Logical Volume Manager，中文可以翻译作逻辑卷轴管理员。LVM 的作法是将几个实体的 partitions (或 disk) 透过软件组合成为一块看起来是独立的大磁碟 (VG) ，然后将这块大磁碟再经过分割成为可使用分割槽 (LV)， 最终就能够挂载使用了。
### Physical Volume, PV, 实体卷轴
我们实际的 partition 需要调整系统识别码 (system ID) 成为 8e (LVM 的识别码)，然后再经过 pvcreate 的命令将他转成 LVM 最底层的实体卷轴 (PV) ，之后才能够将这些 PV 加以利用！ 调整 system ID 的方是就是透过 fdisk。
### Volume Group, VG, 卷轴群组
所谓的 LVM 大磁碟就是将许多 PV 整合成这个 VG 的东西。所以 VG 就是 LVM 组合起来的大磁碟！这个大磁碟最大能够达到的容量，与底下要说明的 PE 有关。因为每个 VG 最多仅能包含 65534 个 PE 。如果使用 LVM 默认的参数，则一个 VG 最大可达 256GB 的容量。
### Physical Extend, PE, 实体延申区块
LVM 默认使用 4MB 的 PE 区块，而 LVM 的 VG 最多仅能含有 65534 个 PE ，因此默认的 LVM VG 会有 4M*65534/(1024M/G)=256G。PE 是整个 LVM 最小的储存区块，也就是说，其实我们的文件数据都是藉由写入 PE 来处理的。 简单的说，这个 PE 就有点像文件系统里面的 block 大小。
### Logical Volume, LV, 逻辑卷轴
最终的 VG 还会被切成 LV，这个 LV 就是最后可以被格式化使用的类似分割槽。LV 的大小就与在此 LV 内的 PE 总数有关。 为了方便使用者利用 LVM 来管理其系统，因此 LV 的装置档名通常指定为『 /dev/vgname/lvname 』的样式。  
透过『交换 PE 』来进行数据转换， 将原本 LV 内的 PE 移转到其他装置中以降低 LV 容量，或将其他装置的 PE 加到此 LV 中以加大容量
### 实作流程
透过 PV, VG, LV 的规划之后，再利用 mkfs 就可以将你的 LV 格式化成为可以利用的文件系统了！而且这个文件系统的容量在未来还能够进行扩充或减少， 而且里面的数据还不会被影响。  
数据写入 LV 时，根据写入机制不同，由两种方式写入硬盘中：
* 线性模式 (linear)：假如我将 /dev/hda1, /dev/hdb1 这两个 partition 加入到 VG 当中，并且整个 VG 只有一个 LV 时，那么所谓的线性模式就是：当 /dev/hda1 的容量用完之后，/dev/hdb1 的硬盘才会被使用到， 这也是我们所建议的模式。
* 交错模式 (triped)：将一笔数据拆成两部分，分别写入 /dev/hda1 与 /dev/hdb1 的意思，感觉上有点像 RAID 0 。一份数据用两颗硬盘来写入，理论上讲，读写效能会较佳。

基本上，LVM 最主要的用处是在实现一个可以弹性调整容量的文件系统上， 而不是在创建一个效能为主的磁碟上，所以，我们应该利用的是 LVM 可以弹性管理整个 partition 大小的用途上，而不是著眼在效能上的。因此， LVM 默认的读写模式是线性模式。若使用 triped 模式，当任何一个 partition 损毁时，所有数据都会损毁，如果强调效能与备份，使用 RAID 较合理。
## LVM 实作流程
LVM 必需要核心有支持且需要安装 lvm2 这个软件。
```
[root@www ~]# fdisk -l
Disk /dev/hda: 41.1 GB, 41174138880 bytes
255 heads, 63 sectors/track, 5005 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes

   Device Boot      Start         End      Blocks   Id  System
/dev/hda1   *           1          13      104391   83  Linux
/dev/hda2              14        1288    10241437+  83  Linux
/dev/hda3            1289        1925     5116702+  83  Linux
/dev/hda4            1926        5005    24740100    5  Extended
/dev/hda5            1926        2052     1020096   82  Linux swap / Solaris
/dev/hda6            2053        2235     1469916   8e  Linux LVM
/dev/hda7            2236        2418     1469916   8e  Linux LVM
/dev/hda8            2419        2601     1469916   8e  Linux LVM
/dev/hda9            2602        2784     1469916   8e  Linux LVM
```
8e 会导致 system 变成【Linux LVM】，没有配置成 8e，某些 LVM 的侦测命令可能会侦测不到该 partition。
### PV 阶段
* pvcreate：将实体 partition 创建成为 PV;
* pvscan：搜寻目前系统里面任何具有 PV 的磁碟；
* pvdisplay：显示出目前系统上面的 PV 状态；
* pvremove：将 PV 属性移除，让该 partition 不具有 PV 属性。
```
root@www ~]# pvscan
  No matching physical volumes found <==找不到任何的 PV 存在喔！

[root@www ~]# pvcreate /dev/hda{6,7,8,9}
  Physical volume "/dev/hda6" successfully created
  Physical volume "/dev/hda7" successfully created
  Physical volume "/dev/hda8" successfully created
  Physical volume "/dev/hda9" successfully created

[root@www ~]# pvscan
  PV /dev/hda6         lvm2 [1.40 GB]
  PV /dev/hda7         lvm2 [1.40 GB]
  PV /dev/hda8         lvm2 [1.40 GB]
  PV /dev/hda9         lvm2 [1.40 GB]
  Total: 4 [5.61 GB] / in use: 0 [0   ] / in no VG: 4 [5.61 GB]
# 这就分别显示每个 PV 的资讯与系统所有 PV 的资讯。尤其最后一行，显示的是：
# 整体 PV 的量 / 已经被使用到 VG 的 PV 量 / 剩余的 PV 量

# 2. 更详细的列示出系统上面每个 PV 的个别资讯：
[root@www ~]# pvdisplay
  "/dev/hda6" is a new physical volume of "1.40 GB"
  --- NEW Physical volume ---
  PV Name               /dev/hda6  <==实际的 partition 装置名称
  VG Name                          <==因为尚未分配出去，所以空白！
  PV Size               1.40 GB    <==就是容量说明
  Allocatable           NO         <==是否已被分配，结果是 NO
  PE Size (KByte)       0          <==在此 PV 内的 PE 大小
  Total PE              0          <==共分割出几个 PE
  Free PE               0          <==没被 LV 用掉的 PE
  Allocated PE          0          <==尚可分配出去的 PE 数量
  PV UUID               Z13Jk5-RCls-UJ8B-HzDa-Gesn-atku-rf2biN
....(底下省略)....
# 由于 PE 是在创建 VG 时才给予的参数，因此在这里看到的 PV 里头的 PE 都会是 0
# 而且也没有多余的 PE 可供分配 (allocatable)。
```
### VG 阶段
* vgcreate：主要创建 VG 的命令
* vgscan：搜寻系统上面是否有 VG 存在
* vgdisplay：显示目前系统上面的 VG 状态
* vgextend：在 VG 内添加额外的 PV
* vgreduce：在 VG 内移除 PV
* vgchange：配置 VG 是否启动 (active)
* vgremove：删除一个 VG

与 PV (partition 的装置档名) 不同，VG 的名称是自订的。
```
[root@www ~]# vgcreate [-s N[mgt]] VG名称 PV名称
选项与参数：
-s ：后面接 PE 的大小 (size) ，单位可以是 m, g, t (大小写均可)

# 1. 将 /dev/hda6-8 创建成为一个 VG，且指定 PE 为 16MB
[root@www ~]# vgcreate -s 16M vbirdvg /dev/hda{6,7,8}
  Volume group "vbirdvg" successfully created

[root@www ~]# vgscan
  Reading all physical volumes.  This may take a while...
  Found volume group "vbirdvg" using metadata type lvm2
# 确实存在这个 vbirdvg 的 VG

[root@www ~]# pvscan
  PV /dev/hda6   VG vbirdvg   lvm2 [1.39 GB / 1.39 GB free]
  PV /dev/hda7   VG vbirdvg   lvm2 [1.39 GB / 1.39 GB free]
  PV /dev/hda8   VG vbirdvg   lvm2 [1.39 GB / 1.39 GB free]
  PV /dev/hda9                lvm2 [1.40 GB]
  Total: 4 [5.57 GB] / in use: 3 [4.17 GB] / in no VG: 1 [1.40 GB]
# 有三个 PV 被用去，剩下一个 /dev/hda9 的 PV 没被用掉

[root@www ~]# vgdisplay
  --- Volume group ---
  VG Name               vbirdvg
  System ID
  Format                lvm2
  Metadata Areas        3
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                3
  Act PV                3
  VG Size               4.17 GB   <==整体的 VG 容量有这么大
  PE Size               16.00 MB  <==内部每个 PE 的大小
  Total PE              267       <==总共的 PE 数量共有这么多！
  Alloc PE / Size       0 / 0
  Free  PE / Size       267 / 4.17 GB
  VG UUID               4VU5Jr-gwOq-jkga-sUPx-vWPu-PmYm-dZH9EO
# 最后那三行指的就是 PE 能够使用的情况！由于尚未切出 LV，因此所有的 PE 
# 均可自由使用。

# 2. 将剩余的 PV (/dev/hda9) 添加到 vbirdvg
[root@www ~]# vgextend vbirdvg /dev/hda9
  Volume group "vbirdvg" successfully extended

[root@www ~]# vgdisplay
....(前面省略)....
  VG Size               5.56 GB
  PE Size               16.00 MB
  Total PE              356
  Alloc PE / Size       0 / 0
  Free  PE / Size       356 / 5.56 GB
  VG UUID               4VU5Jr-gwOq-jkga-sUPx-vWPu-PmYm-dZH9EO
# 这样就可以抽换整个 VG 的大小
```
### LV 阶段
创建分割区 (LV) 。
* lvcreate：创建 LV
* lvscan：查询系统上面的 LV
* lvdisplay：显示系统上面的 LV 状态
* lvextend：在 LV 里面添加容量
* lvreduce：在 LV 里面减少容量
* lvremove：删除一个 LV
* lvresize：对 LV 进行容量大小的调整
```
[root@www ~]# lvcreate [-L N[mgt]] [-n LV名称] VG名称
[root@www ~]# lvcreate [-l N] [-n LV名称] VG名称
选项与参数：
-L  ：后面接容量，容量的单位可以是 M,G,T 等，要注意的是，最小单位为 PE，
      因此这个数量必须要是 PE 的倍数，若不相符，系统会自行计算最相近的容量。
-l  ：后面可以接 PE 的『个数』，而不是数量。若要这么做，得要自行计算 PE 数。
-n  ：后面接的就是 LV 的名称
更多的说明应该可以自行查阅 man lvcreate 

[root@www ~]# lvcreate -l 356 -n vbirdlv vbirdvg
  Logical volume "vbirdlv" created
# 由于本案例中每个 PE 为 16M ，因此上述的命令也可以使用如下的方式来创建：
# lvcreate -L 5.56G -n vbirdlv vbirdvg

[root@www ~]# ll /dev/vbirdvg/vbirdlv
lrwxrwxrwx 1 root root 27 Mar 11 16:49 /dev/vbirdvg/vbirdlv ->
/dev/mapper/vbirdvg-vbirdlv

[root@www ~]# lvdisplay
  --- Logical volume ---
  LV Name                /dev/vbirdvg/vbirdlv  <==这个才是 LV 的全名！
  VG Name                vbirdvg
  LV UUID                8vFOPG-Jrw0-Runh-ug24-t2j7-i3nA-rPEyq0
  LV Write Access        read/write
  LV Status              available
  # open                 0
  LV Size                5.56 GB               <==这个 LV 的容量这么大
  Current LE             356
  Segments               4
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0
```
VG 的名称为 vbirdvg ， 但是 LV 的名称必须使用全名！亦即是 /dev/vbirdvg/vbirdlv。
### 文件系统阶段
```
[root@www ~]# mkfs -t ext3 /dev/vbirdvg/vbirdlv <==注意 LV 全名！
[root@www ~]# mkdir /mnt/lvm
[root@www ~]# mount /dev/vbirdvg/vbirdlv /mnt/lvm
[root@www ~]# df
Filesystem           1K-blocks      Used Available Use% Mounted on
/dev/hda2              9920624   3858984   5549572  42% /
/dev/hda3              4956316   1056996   3643488  23% /home
/dev/hda1               101086     21408     74459  23% /boot
tmpfs                   371332         0    371332   0% /dev/shm
/dev/mapper/vbirdvg-vbirdlv
                       5741020    142592   5306796   3% /mnt/lvm
```
实际上 LVM 使用的装置是放置到 /dev/mapper/ 目录下，将 LV 的名称建置成 /dev/vbirdvg/vbirdlv 是为了方便使用者找到需要的数据。
## 方法 LV 容量
1. 用 fdisk 配置新的具有 8e system ID 的 partition
2. 利用 pvcreate 建置 PV
3. 利用 vgextend 将 PV 加入我们的 vbirdvg
4. 利用 lvresize 将新加入的 PV 内的 PE 加入 vbirdlv 中
5. 透过 resize2fs 将文件系统的容量确实添加！

由于整个文件系统在最初格式化的时候就创建了 inode/block/superblock 等资讯，要改变这些资讯是很难的。不过因为文件系统格式化的时候建置的是多个 block group ，因此我们可以透过在文件系统当中添加 block group 的方式来增减文件系统的量。而增减 block group 就是利用 resize2fs。最后一步是针对文件系统来处理的。
```
[root@www ~]# fdisk /dev/hda <==其他的动作请自行处理
[root@www ~]# partprobe
[root@www ~]# fdisk -l
   Device Boot      Start         End      Blocks   Id  System
....(中间省略)....
/dev/hda10           2785        3150     2939863+  8e  Linux LVM

[root@www ~]# pvcreate /dev/hda10
  Physical volume "/dev/hda10" successfully created
[root@www ~]# pvscan
  PV /dev/hda6    VG vbirdvg   lvm2 [1.39 GB / 0    free]
  PV /dev/hda7    VG vbirdvg   lvm2 [1.39 GB / 0    free]
  PV /dev/hda8    VG vbirdvg   lvm2 [1.39 GB / 0    free]
  PV /dev/hda9    VG vbirdvg   lvm2 [1.39 GB / 0    free]
  PV /dev/hda10                lvm2 [2.80 GB]
  Total: 5 [8.37 GB] / in use: 4 [5.56 GB] / in no VG: 1 [2.80 GB]
# 可以看到 /dev/hda10 是新加入并且尚未被使用

[root@www ~]# vgextend vbirdvg /dev/hda10
  Volume group "vbirdvg" successfully extended
[root@www ~]# vgdisplay
  --- Volume group ---
  VG Name               vbirdvg
  System ID
  Format                lvm2
  Metadata Areas        5
  Metadata Sequence No  4
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               1
  Max PV                0
  Cur PV                5
  Act PV                5
  VG Size               8.36 GB
  PE Size               16.00 MB
  Total PE              535
  Alloc PE / Size       356 / 5.56 GB
  Free  PE / Size       179 / 2.80 GB
  VG UUID               4VU5Jr-gwOq-jkga-sUPx-vWPu-PmYm-dZH9EO
# 不但整体 VG 变大了！而且剩余的 PE 共有 179 个，容量则为 2.80G

[root@www ~]# lvresize -l +179 /dev/vbirdvg/vbirdlv
  Extending logical volume vbirdlv to 8.36 GB
  Logical volume vbirdlv successfully resized
# 放大 LV，lvresize 基本上透过 -l 或 -L 来添加。若要添加使用 +，若要减少则使用 -。

[root@www ~]# lvdisplay
  --- Logical volume ---
  LV Name                /dev/vbirdvg/vbirdlv
  VG Name                vbirdvg
  LV UUID                8vFOPG-Jrw0-Runh-ug24-t2j7-i3nA-rPEyq0
  LV Write Access        read/write
  LV Status              available
  # open                 1
  LV Size                8.36 GB
  Current LE             535
  Segments               5
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0

[root@www ~]# df /mnt/lvm
Filesystem           1K-blocks      Used Available Use% Mounted on
/dev/mapper/vbirdvg-vbirdlv
                       5741020    261212   5188176   5% /mnt/lvm
```
LV 放大到了 8.36GB，但是文件系统没有相对添加。
```
# 先看一下原本的文件系统内的 superblock 记录情况
[root@www ~]# dumpe2fs /dev/vbirdvg/vbirdlv
dumpe2fs 1.39 (29-May-2006)
....(中间省略)....
Block count:              1458176    <==这个filesystem的 block 总数
....(中间省略)....
Blocks per group:         32768      <==多少个 block 配置成为一个 block group
Group 0: (Blocks 0-32767)            <==括号内为 block 的号码
....(中间省略)....
Group 44: (Blocks 1441792-1458175)   <==这是本系统中最后一个 group
....(后面省略)....

[root@www ~]# resize2fs [-f] [device] [size]
选项与参数：
-f      ：强制进行 resize 的动作！
[device]：装置的文件名称；
[size]  ：可以加也可以不加。如果加上 size 的话，那么就必须要给予一个单位，
          譬如 M, G 等等。如果没有 size 的话，那么默认使用『整个 partition』
          的容量来处理！

```
## 缩小 LV 容量
```
# 1. 先找出 /dev/hda6 的容量大小，并尝试计算文件系统需缩小到多少
[root@www ~]# pvdisplay
  --- Physical volume ---
  PV Name               /dev/hda6
  VG Name               vbirdvg
  PV Size               1.40 GB / not usable 11.46 MB
  Allocatable           yes (but full)
  PE Size (KByte)       16384
  Total PE              89
  Free PE               0
  Allocated PE          89
  PV UUID               Z13Jk5-RCls-UJ8B-HzDa-Gesn-atku-rf2biN
# 从这里可以看出 /dev/hda6 有多大，而且含有 89 个 PE 的量
# 那如果要使用 resize2fs 时，则总量减去 1.40GB

[root@www ~]# pvscan
  PV /dev/hda6    VG vbirdvg   lvm2 [1.39 GB / 0    free]
  PV /dev/hda7    VG vbirdvg   lvm2 [1.39 GB / 0    free]
  PV /dev/hda8    VG vbirdvg   lvm2 [1.39 GB / 0    free]
  PV /dev/hda9    VG vbirdvg   lvm2 [1.39 GB / 0    free]
  PV /dev/hda10   VG vbirdvg   lvm2 [2.80 GB / 0    free]
  Total: 5 [8.36 GB] / in use: 5 [8.36 GB] / in no VG: 0 [0   ]
# 从上面可以发现如果扣除 /dev/hda6 则剩余容量有：1.39*3+2.8=6.97

# 2. 直接降低文件系统的容量
[root@www ~]# resize2fs /dev/vbirdvg/vbirdlv 6900M
resize2fs 1.39 (29-May-2006)
Filesystem at /dev/vbirdvg/vbirdlv is mounted on /mnt/lvm; on-line resizing
On-line shrinking from 2191360 to 1766400 not supported.
# 容量好像不能够写小数点位数，因此 6.9G 是错误的，就使用 6900M 了。
# 此外，放大可以线上直接进行，缩小文件系统似乎无法支持, 所以要这样做：

[root@www ~]# umount /mnt/lvm
[root@www ~]# resize2fs /dev/vbirdvg/vbirdlv 6900M
resize2fs 1.39 (29-May-2006)
Please run 'e2fsck -f /dev/vbirdvg/vbirdlv' first.
# 要先进行磁碟检查

[root@www ~]# e2fsck -f /dev/vbirdvg/vbirdlv
e2fsck 1.39 (29-May-2006)
Pass 1: Checking inodes, blocks, and sizes
Pass 2: Checking directory structure
Pass 3: Checking directory connectivity
Pass 4: Checking reference counts
Pass 5: Checking group summary information
/dev/vbirdvg/vbirdlv: 2438/1087008 files (0.1% non-contiguous), 

[root@www ~]# resize2fs /dev/vbirdvg/vbirdlv 6900M
resize2fs 1.39 (29-May-2006)
Resizing the filesystem on /dev/vbirdvg/vbirdlv to 1766400 (4k) blocks.
The filesystem on /dev/vbirdvg/vbirdlv is now 1766400 blocks long.
# 再来 resize2fs 一次就能够成功了！如上所示

[root@www ~]# mount /dev/vbirdvg/vbirdlv /mnt/lvm
[root@www ~]# df /mnt/lvm
Filesystem           1K-blocks      Used Available Use% Mounted on
/dev/mapper/vbirdvg-vbirdlv
                       6955584    262632   6410328   4% /mnt/lvm
```
然后再将 LV 的容量降低。
```
# 3. 降低 LV 的容量，同时我们知道 /dev/hda6 有 89 个 PE
[root@www ~]# lvresize -l -89 /dev/vbirdvg/vbirdlv
  WARNING: Reducing active and open logical volume to 6.97 GB
  THIS MAY DESTROY YOUR DATA (filesystem etc.)
Do you really want to reduce vbirdlv? [y/n]: y
  Reducing logical volume vbirdlv to 6.97 GB
  Logical volume vbirdlv successfully resized
# 会有警告信息！但是我们的实际数据量还是比 6.97G 小

[root@www ~]# lvdisplay
  --- Logical volume ---
  LV Name                /dev/vbirdvg/vbirdlv
  VG Name                vbirdvg
  LV UUID                8vFOPG-Jrw0-Runh-ug24-t2j7-i3nA-rPEyq0
  LV Write Access        read/write
  LV Status              available
  # open                 1
  LV Size                6.97 GB
  Current LE             446
  Segments               5
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0
```
这样就将 LV 缩小了，接下来要将 /dev/hda6 移出 vbirdvg 这个 VG。先要确定 /dev/hda6 里面的 PE 完全不被使用后，才能够将 /dev/hda6 抽离。
```
# 4.1 先确认 /dev/hda6 是否将 PE 都移除了！
[root@www ~]# pvdisplay
  --- Physical volume ---
  PV Name               /dev/hda6
  VG Name               vbirdvg
  PV Size               1.40 GB / not usable 11.46 MB
  Allocatable           yes (but full)
  PE Size (KByte)       16384
  Total PE              89
  Free PE               0
  Allocated PE          89
  PV UUID               Z13Jk5-RCls-UJ8B-HzDa-Gesn-atku-rf2biN
....(中间省略)....

  --- Physical volume ---
  PV Name               /dev/hda10
  VG Name               vbirdvg
  PV Size               2.80 GB / not usable 6.96 MB
  Allocatable           yes
  PE Size (KByte)       16384
  Total PE              179
  Free PE               89
  Allocated PE          90
  PV UUID               7MfcG7-y9or-0Jmb-H7RO-5Pa5-D3qB-G426Vq
# 未被使用的 PE 在 /dev/hda10，需要搬移 PE

[root@www ~]# pvmove /dev/hda6 /dev/hda10
# pvmove 来源PV 目标PV ，可以将 /dev/hda6 内的 PE 通通移动到 /dev/hda10
# 尚未被使用的 PE 去 (Free PE)。

# 4.2 将 /dev/hda6 移出 vbirdvg 中！
[root@www ~]# vgreduce vbirdvg /dev/hda6
  Removed "/dev/hda6" from volume group "vbirdvg"

[root@www ~]# pvscan
  PV /dev/hda7    VG vbirdvg   lvm2 [1.39 GB / 0    free]
  PV /dev/hda8    VG vbirdvg   lvm2 [1.39 GB / 0    free]
  PV /dev/hda9    VG vbirdvg   lvm2 [1.39 GB / 0    free]
  PV /dev/hda10   VG vbirdvg   lvm2 [2.80 GB / 0    free]
  PV /dev/hda6                 lvm2 [1.40 GB]
  Total: 5 [8.37 GB] / in use: 4 [6.97 GB] / in no VG: 1 [1.40 GB]

[root@www ~]# pvremove /dev/hda6
  Labels on physical volume "/dev/hda6" successfully wiped
```
## LVM 的系统快照
LVM 还有一个重要的能力，即系统快照 (snapshot)。快照就是将当时的系统资讯记录下来，就好像照相记录一般！ 未来若有任何数据更动了，则原始数据会被搬移到快照区，没有被更动的区域则由快照区与文件系统共享。  
LVM 会预留一个区域作为数据存放处。 此时快照区内并没有任何数据，而快照区与系统区共享所有的 PE 数据， 因此你会看到快照区的内容与文件系统是一模一样的。等到系统运行一阵子后，假设 A 区域的数据被更动了，则更动前系统会将该区域的数据移动到快照区，而其他 B 到 I 的区块则还是与文件系统共享。  
LVM 的系统快照是非常棒的『备份工具』，因为他只有备份有被更动到的数据， 文件系统内没有被变更的数据依旧保持在原本的区块内，但是 LVM 快照功能会知道那些数据放置在哪里， 因此『快照』当时的文件系统就得以『备份』下来，且快照所占用的容量又非常小。  
由于快照区与原本的 LV 共享很多 PE 区块，因此快照区与被快照的 LV 必须要在同一个 VG 上头。
### 快照区的创建
```
# 先观察 VG 还剩下多少容量
[root@www ~]# vgdisplay
  --- Volume group ---
  VG Name               vbirdvg
....(其他省略)....
  VG Size               6.97 GB
  PE Size               16.00 MB
  Total PE              446
  Alloc PE / Size       446 / 6.97 GB
  Free  PE / Size       0 / 0  <==没有多余的 PE 可用！

# 2. 将刚刚移除的 /dev/hda6 加入这个 VG
[root@www ~]# pvcreate /dev/hda6
  Physical volume "/dev/hda6" successfully created
[root@www ~]# vgextend vbirdvg /dev/hda6
  Volume group "vbirdvg" successfully extended
[root@www ~]# vgdisplay
  --- Volume group ---
  VG Name               vbirdvg
....(其他省略)....
  VG Size               8.36 GB
  PE Size               16.00 MB
  Total PE              535
  Alloc PE / Size       446 / 6.97 GB
  Free  PE / Size       89 / 1.39 GB  <==多出了 89 个可用 PE

# 3. 利用 lvcreate 创建系统快照区，我们取名为 vbirdss，且给予 60 个 PE
[root@www ~]# lvcreate -l 60 -s -n vbirdss /dev/vbirdvg/vbirdlv
  Logical volume "vbirdss" created
# 上述的命令中最重要的是那个 -s 的选项！代表是 snapshot 快照功能之意！
# -n 后面接快照区的装置名称， /dev/.... 则是要被快照的 LV 完整档名。
# -l 后面则是接使用多少个 PE 来作为这个快照区使用。

[root@www ~]# lvdisplay
  --- Logical volume ---
  LV Name                /dev/vbirdvg/vbirdss
  VG Name                vbirdvg
  LV UUID                K2tJ5E-e9mI-89Gw-hKFd-4tRU-tRKF-oeB03a
  LV Write Access        read/write
  LV snapshot status     active destination for /dev/vbirdvg/vbirdlv
  LV Status              available
  # open                 0
  LV Size                6.97 GB    <==被快照的原 LV 磁碟容量
  Current LE             446
  COW-table size         960.00 MB  <==快照区的实际容量
  COW-table LE           60         <==快照区占用的 PE 数量
  Allocated to snapshot  0.00%
  Snapshot chunk size    4.00 KB
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:1
```
/dev/vbirdvg/vbirdss 这个快照区就被创建起来了。他的 VG 量与原本的 /dev/vbirdvg/vbirdlv 相同，也就是说，如果挂载这个装置，看到的数据会跟原本的 vbirdlv 相同。
### 利用快照区复原系统
要复原的数据量不能够高于快照区所能负载的实际容量。由于原始数据会被搬移到快照区， 如果你的快照区不够大，若原始数据被更动的实际数据量比快照区大，那么快照区当然容纳不了，这时候快照功能会失效。
```
[root@www ~]# lvdisplay /dev/vbirdvg/vbirdss
  --- Logical volume ---
  LV Name                /dev/vbirdvg/vbirdss
  VG Name                vbirdvg
....(中间省略)....
  Allocated to snapshot  12.22%
....(底下省略)....
# 从这里也看得出来，快照区已经被使用了 12.22% ！因为原始的文件系统有更动

利用快照区将原本的 filesystem 备份
[root@www ~]# mount /dev/vbirdvg/vbirdss /mnt/snapshot
[root@www ~]# df
Filesystem           1K-blocks      Used Available Use% Mounted on
/dev/mapper/vbirdvg-vbirdlv
                       6955584    370472   6302488   6% /mnt/lvm
/dev/mapper/vbirdvg-vbirdss
                       6955584    262632   6410328   4% /mnt/snapshot
# 两者不同

[root@www ~]# mkdir -p /backups <==确认真的有这个目录！
[root@www ~]# cd /mnt/snapshot
[root@www snapshot]# tar -jcv -f /backups/lvm.tar.bz2 *
# 此时你就会有一个备份数据，亦即是 /backups/lvm.tar.bz2 了！
```
不能够格式化 /dev/vbirdvg/vbirdlv 然后直接复制，因为个时候整个 vbirdlv 时，原本文件系统所有的数据都会被搬移到 vbirdss，如果 vbirdss 的容量不够大(通常真的不够大)，那么部分数据将无法复制到 vbirdss 内，数据无法全部还原。  
可以对比 /mnt/lvm 与 /mnt/snapshot 的内容，找出最近的修改。还原 vbirdlv 的内容：
```
将 vbirdss 卸载并移除 (因为里面的内容已经备份起来了)
[root@www ~]# umount /mnt/snapshot
[root@www ~]# lvremove /dev/vbirdvg/vbirdss
Do you really want to remove active logical volume "vbirdss"? [y/n]: y
  Logical volume "vbirdss" successfully removed

[root@www ~]# umount /mnt/lvm
[root@www ~]# mkfs -t ext3 /dev/vbirdvg/vbirdlv
[root@www ~]# mount /dev/vbirdvg/vbirdlv /mnt/lvm
[root@www ~]# tar -jxv -f /backups/lvm.tar.bz2 -C /mnt/lvm
[root@www ~]# ll /mnt/lvm
drwxr-xr-x 105 root root 12288 Mar 11 16:59 etc
drwxr-xr-x  17 root root  4096 Mar 11 14:17 log
drwx------   2 root root 16384 Mar 11 16:59 lost+found
```
### 利用快照区进行各项练习与测试的任务，再以原系统还原快照
换个角度来想想，我们将原本的 vbirdlv 当作备份数据，然后将 vbirdss 当作实际在运行中的数据， 任何测试的动作都在 vbirdss 这个快照区当中测试，那么当测试完毕要将测试的数据删除时，只要将快照区删去即可！ 而要复制一个 vbirdlv 的系统，再作另外一个快照区即可！
## LVM 相关命令汇整与 LVM 的关闭
任务|PV 阶段|VG 阶段|LV 阶段
:---:|:---:|:---:|:---:
搜寻(scan)|pvscan|vgscan|lvscan
创建(create)|pvcreate|vgcreate|lvcreate
列出(display)|pvdisplay|vgdisplay|lvdisplay
添加(extend)||vgextend|lvextend (lvresize)
减少(reduce)||vgreduce|lvreduce (lvresize)
删除(remove)|pvremove|vgremove|lvremove
改变容量(resize)|||lvresize
改变属性(attribute)|pvchange|vgchange|lvchange
至于文件系统阶段 (filesystem 的格式化处理) 部分，还需要以 resize2fs 来修订文件系统实际的大小。虽然 LVM 可以弹性的管理你的磁碟容量，但是要注意，如果你想要使用 LVM 管理您的硬盘时，那么在安装的时候就得要做好 LVM 的规划了， 否则未来还是需要先以传统的磁碟添加方式来添加后，移动数据后，才能够进行 LVM 的使用。  
如果实体 partition 已经被使用到 LVM 中去，如果将来没有将 LVM 关闭就直接将那些 partition 删除或转为其他用途的话，系统会发生很大问题。
1. 先卸载系统上面的 LVM 文件系统 (包括快照与所有 LV)；
2. 使用 lvremove 移除 LV ；
3. 使用 vgchange -a n VGname 让 VGname 这个 VG 不具有 Active 的标志；
4. 使用 vgremove 移除 VG：
5. 使用 pvremove 移除 PV；
6. 最后，使用 fdisk 修改 ID 回来