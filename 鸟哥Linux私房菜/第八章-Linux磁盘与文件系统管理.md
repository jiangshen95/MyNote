# EXT2文件系统
## 硬盘的组成与分割
组成：
* 圆形的磁盘盘(主要记录数据的部分)；
* 机械手臂，与在机械手臂上的磁盘读取头(可擦写磁盘盘上的数据)；
* 主轴马达，可以转动磁盘盘，让机械手臂的读取头在磁盘盘上读写数据。

物理组成：
* 扇区(Sector)为最小的物理储存单位，每个扇区为 512 bytes；
* 将扇区组成一个圆，那就是磁柱(Cylinder)，磁柱是分割槽(partition)的最小单位；
* 第一个扇区最重要，里面有：(1)主要启动区(Master boot record, MBR)及分割表(partition table)， 其中 MBR 占有 446 bytes，而 partition table 则占有 64 bytes。

各种接口的磁盘在Linux中的文件名分别为：
* /dev/sd[a-p][1-15]：为SCSI, SATA, U盘, Flash闪盘等接口的磁盘文件名；
* /dev/hd[a-d][1-63]：为 IDE 接口的磁盘文件名；

分割表仅有64bytes而已， 因此最多只能记录四笔分割槽的记录，这四笔记录我们称为主要 (primary) 或延伸 (extended) 分割槽，其中扩展分配槽还可以再分割出逻辑分割槽 (logical) ， 而能被格式化的则仅有主要分割与逻辑分割。
* 主要分割与扩展分配最多可以有四笔(硬盘的限制)
* 扩展分配最多只能有一个(操作系统的限制)
* 逻辑分割是由扩展分配持续切割出来的分割槽；
* 能够被格式化后，作为数据存取的分割槽为主要分割与逻辑分割。扩展分配无法格式化；
* 逻辑分割的数量依操作系统而不同，在Linux系统中，IDE硬盘最多有59个逻辑分割(5号到63号)， SATA硬盘则有11个逻辑分割(5号到15号)。
## 文件系统特性
格式化为【文件系统格式(filesystem)】,Ext2(Linux second extended file system, ext2fs)  
由于新技术的利用，例如LVM与软件磁盘阵列(software raid)，可以将一个分割槽格式化为多个文件系统(LVM)，也能够将多个分割槽合成一个文件系统(LVM,RAID)。通常我们称呼一个可被挂载的数据为一个文件系统而不是一个分割槽。
文件系统包含实际数据，数据的权限与属性，通常将这两部分的数据分别存放在不同的区块，权限与属性放置到inode中，至于实际数据则放置到data block区块中。另外还有一个超级区块(superblock)会记录整个文件系统的整体信息，包括inode与block的总量、使用量、剩余量等。  
每个inode与block都有编号，这三个数据的意义：
* superblick：记录此 filesystem 的整体信息，包括inode/block的总量、使用量、剩余量， 以及文件系统的格式与相关信息等；
* inode：记录文件的属性，一个文件占用一个inode，同时记录此文件的数据所在的 block 号码；  
* block：实际记录文件的内容，若文件太大时，会占用多个 block。

每个文件都会占用一个inode，inode 内则有文件数据放置的 block 号码。  
文件系统先格式化出 inode 与 block 的区块，一口气将所有 block 内容读出来，索引式文件系统(indexed allocation)。FAT格式则是，没有 inode 存在，每个 block 号码都记录在前一个 block 中。碎片整理的原因就是文件写入的 block 太过于离散了，此时文件读取的效能将会变的很差所致。 这个时候可以透过碎片整理将同一个文件所属的 blocks 汇整在一起，这样数据的读取会比较容易。  
## Linux的EXT2文件系统(inode)：
文件系统一开始就将 inode 与 block 规划好了，除非重新格式化(或者利用 resize2fs 等命令变更文件大小)，否则 inode 与 block 固定后就不再变动。  
Ext2 文件系统在格式化的时候基本上是区分为多个区块群组(block group)的，每个区块群组都有独立的inode/block/superblock 系统。  
在整体规格中，文件系统最前面有一个启动扇区(boot sector)，这个启动扇区可以安装启动管理程序，如此我们就能将不同的启动管理程序安装到个别的文件系统最前端，而不用覆盖硬盘唯一的MBR，这样就能制作多重引导环境。
### data block（数据区块）
在 Ext2 文件系统中所支持的 block 大小有1K，2K 及 4K三种。在格式化时 block 的大小就固定了，且每个 block 都有编号方便 inode 记录。
Block 大小|1KB|2KB|4KB
:---:|:---:|:---:|:---:
最大单一文件限制|16GB|256GB|2TB
最大文件系统总容量|2TB|8TB|16TB
Ext2 文件系统的 block 基本限制如下:
* 原则上，block 的大小与数量在格式化完就不能够再改变了(除非重新格式化)；
* 每个 block 内最多只能够放置一个文件的数据；
* 承上，如果文件大于 block 的大小，则一个文件会占用多个 block 数量；
* 承上，若文件小于 block ，则该 block 的剩余容量就不能够再被使用了(磁盘空间会浪费)。
### inode table (inode 表格)
inode 记录的文件数据：
* 该文件的存取模式(read/write/excute)；
* 该文件的拥有者与群组(owner/group)；
* 该文件的容量；
* 该文件创建或状态改变的时间(ctime)；
* 最近一次的读取时间(atime)；
* 最近修改的时间(mtime)；
* 定义文件特性的旗标(flag)，如 SetUID...；
* 该文件真正内容的指向 (pointer)；

inode 的特性：
* 每个 inode 大小均固定为 128 bytes；
* 每个文件都仅会占用一个 inode 而已；
* 承上，因此文件系统能够创建的文件数量与 inode 的数量有关；
* 系统读取文件时需要先找到 inode，并分析 inode 所记录的权限与用户是否符合，若符合才能够开始实际读取 block 的内容。

文件系统将 inode 记录 block 号码的区域定义为12个直接，一个间接，一个双间接与一个三间接记录区。所谓间接就是再拿一个 block 当作记录 block 号码的记录区，如果文件太大时，就会使用简介的 block 来记录编号。双间接，第一个 block 仅指出下一个记录编号的 block 在哪里，实际记录的在第二个 block 中。  
以 1K block 为例计算 inode 能够指定的大小：
* 12 个直接指向： 12*1K=12K  
  由于是直接指向，所以总共可记录 12 笔记录，因此总额大小为如上所示
* 间接： 256*1K=256K  
  每笔 block 号码的记录会花去 4bytes，因此 1K 的大小能够记录 256 笔记录，因此一个间接可以记录的文件大小如上
* 双间接： 256*256*1K
  第一层 block 会指定 256 个第二层，每个第二层可以指定 256 个号码，因此总额大小如上；
* 三间接： 256*256*256*1K
  第一层 block 会指定 256 个第二层，每个第二层可以指定 256 个第三层，每个第三层可以指定 256 个号码，因此总额大小如上；
* 总额：相加得 16 GB

使用 2K 或 4K 的 block 时，会受到 Ext2 文件系统本身的限制，不能这么计算
### Superblock(超级区块)
Superblock 是记录整个 filesystem 相关信息的地方，主要记录的信息：
* block 与 inode 的总量
* 未使用与已使用的 inode/block 的数量
* block 与 inode 的大小(block 为 1，2，4K，inode 为 128 bytes)
* filesystem 的挂载时间、最近一次写入数据的时间、最近一次检验磁盘 (fsck) 的时间等文件系统的相关信息；
* 一个 valid bit 数值，若此文件系统已被挂载，则 valid bit 为 0 ，若未被挂载，则 valid bit 为 1 。

每个 block group 都可能含有 superblock，除了第一个 block group 内会含有 superbloc 之外，后续的不一定含有 superblock，若含有则为第一个的备份。
### Filesystem Description(文件系统描述说明)
可以描述每个 block group 的开始与结束的 block 号码，以及说明每个区段(superblock, bitmap, inodemap, data block)分别介于哪一个block号码之间
### block bitmap(区块对照表)
可以知道那些 block 是空的，文件删除时释放原本占用的 block ，此时 block bitmap 相应的该 block 号码的标志就要修改成【未使用】。记录 block 的使用情况。
### inode bitmap(inode 对照表)
记录 inode 的使用情况
***
每个区段与 superblock 的信息都可以使用 dump2fs 这个命令查询：
```
dumpe2fs [-bh] 装置文件名
选项与参数：
-b ：列出保留为坏轨的部分(一般用不到)
-h ：仅列出 superblock 的数据，不会列出其他的区段内容

df  <==这个命令可以叫出目前挂载的装置

dumpe2fs /dev/hdc2
dumpe2fs 1.39 (29-May-2006)
Filesystem volume name:   /1             <==这个是文件系统的名称(Label)
Filesystem features:      has_journal ext_attr resize_inode dir_index 
  filetype needs_recovery sparse_super large_file
Default mount options:    user_xattr acl <==默认挂载的参数
Filesystem state:         clean          <==这个文件系统是没问题的(clean)
Errors behavior:          Continue
Filesystem OS type:       Linux
Inode count:              2560864        <==inode的总数
Block count:              2560359        <==block的总数
Free blocks:              1524760        <==还有多少个 block 可用
Free inodes:              2411225        <==还有多少个 inode 可用
First block:              0
Block size:               4096           <==每个 block 的大小啦！
Filesystem created:       Fri Sep  5 01:49:20 2008
Last mount time:          Mon Sep 22 12:09:30 2008
Last write time:          Mon Sep 22 12:09:30 2008
Last checked:             Fri Sep  5 01:49:20 2008
First inode:              11
Inode size:               128            <==每个 inode 的大小
Journal inode:            8              <==底下这三个与下一小节有关
Journal backup:           inode blocks
Journal size:             128M

Group 0: (Blocks 0-32767) <==第一个 data group 内容, 包含 block 的启始/结束号码
  Primary superblock at 0, Group descriptors at 1-1  <==超级区块在 0 号 block
  Reserved GDT blocks at 2-626
  Block bitmap at 627 (+627), Inode bitmap at 628 (+628)
  Inode table at 629-1641 (+629)                     <==inode table 所在的 block
  0 free blocks, 32405 free inodes, 2 directories    <==所有 block 都用完了！
  Free blocks:
  Free inodes: 12-32416                              <==剩余未使用的 inode 号码
Group 1: (Blocks 32768-65535)
```
## 与目录树的关系
### 目录
当我们在 Linux 下的 ext2 文件系统创建一个目录时， ext2 会分配一个 inode 与至少一块 block 给该目录。其中，inode 记录该目录的相关权限与属性，并可记录分配到的那块 block 号码； 而 block 则是记录在这个目录下的文件名与该文件名占用的 inode 号码数据。  在目录底下的文件数如果太多而导致一个 block 无法容纳的下所有的档名与 inode 对照表时，Linux 会给予该目录多一个 block 来继续记录相关的数据；
使用 `ls -li` 这个命令可以查看目录下的文件所占用的 inode 号码
### 文件
当我们在 Linux 下的 ext2 创建一个一般文件时， ext2 会分配一个 inode 与相对于该文件大小的 block 数量给该文件。
### 目录树读取
inode 本身并不记录文件名，文件名的记录是在目录的 block 当中。
### filesystem 大小与硬盘读取效能
数据离散会降低读取效率
## EXT2/EXT3 文件的存取与日志式文件系统的功能
新增一个文件：
1. 先确定用户对于欲新增的文件的目录是否具有 w 与 x 的权限，有的话才能新增；
2. 根据 inode bitmap 找到没有使用的 inode 号码，并将新文件的权限/属性写入；
3. 根据 block bitmap 找到没有使用中的 block 号码，并将实际的数据写入 block 中，且升级 inode 的 block 指向数据；
4. 将刚刚写入的 inode 与 block 数据同步升级 inode bitmap 与 block bitmap，并升级 superblock 的内容。

一般来说，我们将 inode table 与 data table 称为数据存放区域，至于其他例如 superblock、block bitmap 与 inode bitmap 等区段就被称为 metadata (中介数据)，因为 superblock, inode bitmap 及 block bitmap 的数据经常变动，每次新增、移除、编辑时都可能会影响到这三个部分的数据，因此才被称为中介数据。
### 数据的不一致 (Inconsistent) 状态
文件写入文件系统时因为不知名的原因导致系统中的(例如停电，系统核心发生错误等)，写入的数据仅有 inode table 及 data block 而已，最后一个同步升级中介数据的步骤并没有做完，此时就会发生 metadata 的内容与实际数据存放区产生不一致 (Inconsistent) 的情况了。  
早期的 Ext2 文件系统中，如果发生这个问题， 那么系统在重新启动的时候，就会藉由 Superblock 当中记录的 valid bit (是否有挂载) 与 filesystem state (clean 与否) 等状态来判断是否强制进行数据一致性的检查。需要检查时则以 e2fsck 这支程序来进行。
### 日志式文件系统 (Journaling filesystem)
在 filesystem 当中规划出一个区块，专门记录写入或修订文件时的步骤：
1. 预备：当系统要写入一个文件时，会先在日志记录区块中纪录某个文件准备要写入的信息；
2. 实际写入：开始写入文件的权限与数据；开始升级 metadata 的数据；
3. 结束：完成数据与 metadata 的升级后，在日志记录区块当中完成该文件的纪录。

万一数据记录发生问题，只要检查日志记录区块，不必针对整个 filesystem 去检查，这样就可以达到快速修复 filesystem 的能力。  
Ext3 —— Ext2 的升级版本
## Linux 文件系统的运行
异步处理 (asynchronously) 的方式：  
当系统加载一个文件到内存后，如果该文件没有被更动过，则在内存区段的文件数据会被配置为干净(clean)的。 但如果内存中的文件数据被更改过了(例如你用 nano 去编辑过这个文件)，此时该内存中的数据会被配置为脏的 (Dirty)。此时所有的动作都还在内存中运行，并没有写入到磁盘中。 系统会不定时的将内存中配置为『Dirty』的数据写回磁盘，以保持磁盘与内存数据的一致性。 你也可以利用 `sync` 命令来手动强迫写入磁盘。  
文件系统与内存的关系：
* 系统会将常用的文件数据放置到主存储器的缓冲区，以加速文件系统的读/写；
* 承上，因此 Linux 的物理内存最后都会被用光！这是正常的情况！可加速系统效能；
* 可以手动使用 `sync` 来强迫内存中配置为 Dirty 的文件回写到磁盘中；
* 若正常关机时，关机命令会主动呼叫 sync 来将内存的数据回写入磁盘内；
* 但若不正常关机(如跳电、死机或其他不明原因)，由于数据尚未回写到磁盘内， 因此重新启动后可能会花很多时间在进行磁盘检验，甚至可能导致文件系统的损毁(非磁盘损毁)。
## 挂载点的意义 (mout point)
每个 filesystem 都有独立的 inode /block /superblock 等信息，这个文件系统能够链接到目录树才能被我们使用——挂载。挂载点一定是目录，该目录为进入该文件系统的入口，必须【挂载】到目录树的某个目录后，才能够使用该文件系统。   
 filesystem 最顶层的目录之 inode 一般为 2 号；
 ## 其他 Linux 支持的文件系统
 * 传统文件系统：ext2 / minix / MS-DOS / FAT(用 vfat 模块) / iso9660(光盘)等等；
 * 日志式文件系统：ext3 / ReiserFS / Windows'NTFS / IBM's JFS / SGI's XFS
 * 网络文件系统：NFS / SMBFS
 ### Linux VFS (Virtual Filesystem Switch)
 整个 Linux 的系统都是透过 Virtual Filesystem Switch 的核心功能去读取 filesystem 的。

 # 文件系统的简单操作
 ## 磁盘与目录的容量
* df 列出文件系统的整体磁盘使用量
  ```
  df [-ahikHTm] [目录或文件名]
  选项与参数：
  -a  ：列出所有的文件系统，包括系统特有的 /proc 等文件系统；
  -k  ：以 KBytes 的容量显示各文件系统；
  -m  ：以 MBytes 的容量显示各文件系统；
  -h  ：以人们较易阅读的 GBytes, MBytes, KBytes 等格式自行显示；
  -H  ：以 M=1000K 取代 M=1024K 的进位方式；
  -T  ：连同该 partition 的 filesystem 名称 (例如 ext3) 也列出；
  -i  ：不用硬盘容量，而以 inode 的数量来显示
  ```
  输出的结果信息：  
    * Filesystem：代表该文件系统是在哪个 partition ，所以列出装置名称；
    * 1k-blocks：说明底下的数字单位是 1KB，可利用 -h 或 -m 来改变；
    * Used：使用掉的硬盘空间
    * Available：也就是剩下的磁盘空间大小；
    * Use%：就是磁盘的使用率啦！如果使用率高达 90% 以上时， 最好需要注意一下了，免得容量不足造成系统问题
    * Mounted on：就是磁盘挂载的目录所在(挂载点)

  由于 df 主要读取的数据几乎都是针对一整个文件系统，因此读取的范围主要是在 Superblock 内的信息， 所以这个命令显示结果的速度非常的快速  
  /dev/shm/ 目录，是内存虚拟出来的磁盘空间。
* du 评估文件系统的磁盘使用量（常用在推估目录所占容量）  
  ```
  du [-ahskm] 文件或目录名称
  选项与参数：
  -a  ：列出所有的文件与目录容量，因为默认仅统计目录底下的文件量而已。
  -h  ：以人们较易读的容量格式 (G/M) 显示；
  -s  ：列出总量而已，而不列出每个各别的目录占用容量；
  -S  ：不包括子目录下的总计，与 -s 有点差别。
  -k  ：以 KBytes 列出容量显示；
  -m  ：以 MBytes 列出容量显示；

  # 直接输入 du 没有加任何选项时，则 du 会分析『目前所在目录』
  # 的文件与目录所占用的硬盘空间。但是，实际显示时，仅会显示目录容量(不含文件)，
  # 因此 . 目录有很多文件没有被列出来，所以全部的目录相加不会等于 . 的容量喔！
  # 此外，输出的数值数据为 1K 大小的容量单位。
  ```
  与 df 不一样的是，du 这个命令会直接到文件系统内去搜寻所有的文件数据，运行时间慢。

## 实体链接与符号链接：ln
Linux 下连结档有两种：一种是类似 Windows 的快捷方式，可以快速链接到目标文件或目录；另一种则是透过文件系统的 inode 连结来产生新档名，而不是产生新文件。这种称为实体链接(hard link)。
* Hard Link (实体链接，硬式连结或实际连结)
    * 每个文件都会占用一个 inode，文件内容由 inode 的记录来指向
    * 想要读取文件，必须经过目录记录的文件名来指向到正确的 inode 号码
  
  也就是说，其实文件名只与目录有关，但是文件内容则与 inode 有关。hard link 只是在某个目录下新增一笔档名链接到某 inode 号码的关连记录而已。  
  `ls -l` 的第二个字段，【有多少个档名链接到这个 inode 号码】  
  如果你将任何一个『档名』删除，其实 inode 与 block 都还是存在的  
  不论你使用哪个『档名』来编辑， 最终的结果都会写入到相同的 inode 与 block 中，因此均能进行数据的修改  
  hard link 使用限制：
    * 不能跨 Filesystem
    * 不能 link 目录
* Symbolic Link (符号链接，亦即是快捷方式)
创建一个独立的文件，而这个文件会让数据的读取指向他 link 的那个文件的档名。连结档的重要内容就是他会写上目标文件的【文件名】。
```
ln [-sf] 来源文件 目标文件
选项与参数：
-s  ：如果不加任何参数就进行连结，那就是hard link，至于 -s 就是symbolic link
-f  ：如果 目标文件 存在时，就主动的将目标文件直接移除后再创建！
```
修改 Linux 下的 symbolic link 文件时，实则更动的是【原始档】
* 关于目录的 link 数量：  
  空目录里面存在 . 与 ..  这两个目录。创建一个新目录时，【新的目录 link 数为 2，而上一级目录的 link 数会加一】

# 磁盘的分割、格式化、检验与挂载：
新增一颗硬盘时：
1. 对硬盘进行分割，以创建可用的 partition；
2. 对该 partition 进行格式化( format )，以创建系统可用的 filesystem；
3. 若想仔细一点，可以对刚刚创建好的 filesystem 进行检验
4. 在 Linux 系统上，需要创建挂载点(亦即是目录)，并将他挂载上来
## 磁盘分区： fdisk
```
fdisk [-l] 装置名称
选项与参数：
-l  ：输出后面接的装置所有的 partition 内容。若仅有 fdisk -l 时，
      则系统将会把整个系统内能够搜寻到的装置的 partition 均列出来。

fdisk /dev/hdc  <==仔细看，不要加上数字喔！
The number of cylinders for this disk is set to 5005.
There is nothing wrong with that, but this is larger than 1024,
and could in certain setups cause problems with:
1) software that runs at boot time (e.g., old versions of LILO)
2) booting and partitioning software from other OSs
   (e.g., DOS FDISK, OS/2 FDISK)

Command (m for help):     <==等待你的输入！

Command (m for help): m   <== 输入 m 后，就会看到底下这些命令介绍
Command action
   a   toggle a bootable flag
   b   edit bsd disklabel
   c   toggle the dos compatibility flag
   d   delete a partition            <==删除一个partition
   l   list known partition types
   m   print this menu
   n   add a new partition           <==新增一个partition
   o   create a new empty DOS partition table
   p   print the partition table     <==在屏幕上显示分割表
   q   quit without saving changes   <==不储存离开fdisk程序
   s   create a new empty Sun disklabel
   t   change a partition's system id
   u   change display/entry units
   v   verify the partition table
   w   write table to disk and exit  <==将刚刚的动作写入分割表
   x   extra functionality (experts only)

Command (m for help): p  <== 这里可以输出目前磁盘的状态

Disk /dev/hdc: 41.1 GB, 41174138880 bytes        <==这个磁盘的文件名与容量
255 heads, 63 sectors/track, 5005 cylinders      <==磁头、扇区与磁柱大小
Units = cylinders of 16065 * 512 = 8225280 bytes <==每个磁柱的大小

   Device Boot      Start         End      Blocks   Id  System
/dev/hdc1   *           1          13      104391   83  Linux
/dev/hdc2              14        1288    10241437+  83  Linux
/dev/hdc3            1289        1925     5116702+  83  Linux
/dev/hdc4            1926        5005    24740100    5  Extended
/dev/hdc5            1926        2052     1020096   82  Linux swap / Solaris
# 装置文件名 启动区否 开始磁柱    结束磁柱  1K大小容量 磁盘分区槽内的系统

Command (m for help): q
# 想要不储存离开，按下 q ，不要随便按 w 
```
fdisk 只用 root 才能运行，使用【装置文件名】不要加数字，因为 partition 是针对【整个硬件装置】而不是某个 patition。

### 删除磁盘分割槽
1. `fdisk /dev/sdb`：先进入 fdisk 画面；
2. p：先看一下分割槽的信息
3. d：选择一个 partition，输入数字
4. w (or) q
### 新增磁盘分区槽
```
新增一个 Primary 的分割槽，且指定为 4 号
Command (m for help): n
Command action            <==因为是全新磁盘，因此只会问extended/primary而已
   e   extended
   p   primary partition (1-4)
p                         <==选择 Primary 分割槽
Partition number (1-4): 4 <==配置为 4 号
First cylinder (1-5005, default 1): <==直接按下[enter]按键决定！
Using default value 1               <==启始磁柱就选用默认值！
Last cylinder or +size or +sizeM or +sizeK (1-5005, default 5005): +512M
# 我们知道 partition 是由 n1 到 n2 的磁柱号码 (cylinder)，
# 但磁柱的大小每颗磁盘都不相同，这个时候可以填入 +512M 来让系统自动帮我们找出
# 『最接近 512M 的那个 cylinder 号码』！因为不可能刚好等于 512MBytes
# 如上所示：这个地方输入的方式有两种：
# 1) 直接输入磁柱的号码，你得要自己计算磁柱/分割槽的大小才行；
# 2) 用 +XXM 来输入分割槽的大小，让系统自己捉磁柱的号码。
#    +与M是必须要有的，XX为数字

继续新增一个，Extended 分割槽
Command (m for help): n
Command action
   e   extended
   p   primary partition (1-4)
e    <==选择的是 Extended
Partition number (1-4): 1
First cylinder (64-5005, default 64): <=[enter]
Using default value 64
Last cylinder or +size or +sizeM or +sizeK (64-5005, default 5005): <=[enter]
Using default value 5005
# 还记得我们在第三章的磁盘分区表曾经谈到过的，扩展分配最好能够包含所有
# 未分割的区间；所以在这个练习中，我们将所有未配置的磁柱都给了这个分割槽
# 所以在开始/结束磁柱的位置上，按下两个[enter]用默认值即可！

新增一个 2GB 的分割槽
Command (m for help): n
Command action
   l   logical (5 or over)     <==因为已有 extended ，所以出现 logical 分割槽
   p   primary partition (1-4)
p   <=不能增加主分割槽
Partition number (1-4): 2
No free sectors available   <==因为没有多余的磁柱可供配置

Command (m for help): n
Command action
   l   logical (5 or over)
   p   primary partition (1-4)
l   <==乖乖使用逻辑分割槽吧！
First cylinder (64-5005, default 64): <=[enter]
Using default value 64
Last cylinder or +size or +sizeM or +sizeK (64-5005, default 5005): +2048M
```
* 1 - 4 号尚有剩余，且系统未有 extended：  
  此时可以挑选 Primary/Extended，且可以指定 1-4 号间的号码
* 1 - 4 号尚有剩余，且系统有 extended：  
  此时可挑选 Primary/Logical 的项目；若选择 p 则还需要指定 1-4 号间的号码；若选择 l(L的小写)则不需要配置号码，系统会自动指定逻辑分割槽的文件名号码
* 1 - 4 没有剩余，且系统有 extended：  
  此时不会让你挑选分割槽的类型，直接进入 logical 的分割槽形式

有时会存在，因为我们的磁盘无法卸除(因为含有根目录)，所以核心无法重新取得分割表信息， 因此此时系统会要求我们重新启动(reboot)以升级核心的分割表信息才行。也可以使用 `partprobe` 这个命令，告知核心必须要读取新的分割表。
### 操作环境说明
以 root 身份进行硬盘的 partition 时，最好在单人维护模式底下比较安全。如果该硬盘某个 partition 还在使用当中， 那么很有可能系统核心会无法重载硬盘的 partition table ，解决的方法就是将该使用中的 partition 给他卸除，然后再重新进入 fdisk 一遍，重新写入 partition table
### 注意事项
SATA 硬盘最多能够支持 15 号的分割槽，IDE 则可以支持到 63 号。  
`fdisk` 没有办法处理大于 2TB 以上的硬盘分区槽。此时使用 `parted` 命令

## 磁盘格式化
make filesystem -- mkfs
* mkfs
  ```
  mkfs [-t 文件系统格式] 装置文件名
  选项与参数：
  -t  ：可以接文件系统格式，例如 ext3, ext2, vfat 等(系统有支持才会生效)
  ```
  要指定：文件系统的标头(Label)、Block 的大小以及 inode 的数量，要使用 Ext2/Ext3 的公用程序，亦即 `mke2fs`这个命令
* mke2fs
  ```
  mke2fs [-b block大小] [-i block大小] [-L 标头] [-cj] 装置
  选项与参数：
  -b  ：可以配置每个 block 的大小，目前支持 1024, 2048, 4096 bytes 三种；
  -i  ：多少容量给予一个 inode 呢？
  -c  ：检查磁盘错误，仅下达一次 -c 时，会进行快速读取测试；
        如果下达两次 -c -c 的话，会测试读写(read-write)，会很慢
  -L  ：后面可以接标头名称 (Label)，这个 label 是有用的，e2label命令介绍会谈到～
  -j  ：本来 mke2fs 是 EXT2 ，加上 -j 后，会主动加入 journal 而成为 EXT3。

  # 若没有指定 -j 的时候，mke2fs 使用 ext2 为格式化文件格式，若加入 -j 时，则格式化为 ext3 这个 Journaling 的 filesystem
  ```
## 磁盘检验：fsck，badblocks
* fsck
  ```
  fsck [-t 文件系统] [-ACay] 装置名称
  选项与参数：
  -t  ：如同 mkfs 一样，fsck 也是个综合软件而已！因此我们同样需要指定文件系统。
        不过由于现今的 Linux 会自动的透过 superblock 去分辨文件系统，
        因此通常可以不需要这个选项！请看后续的范例说明。
  -A  ：依据 /etc/fstab 的内容，将需要的装置扫瞄一次。/etc/fstab 于下一小节说明，
        通常启动过程中就会运行此一命令了。
  -a  ：自动修复检查到的有问题的扇区，所以你不用一直按 y 
  -y  ：与 -a 类似，但是某些 filesystem 仅支持 -y 这个参数！
  -C  ：可以在检验的过程当中，使用一个直方图来显示目前的进度！

  EXT2/EXT3 的额外选项功能：(e2fsck 这支命令所提供)
  -f  ：强制检查！一般来说，如果 fsck 没有发现任何 unclean 的旗标，不会主动进入
        细部检查的，如果您想要强制 fsck 进入细部检查，就得加上 -f 旗标
  -D  ：针对文件系统下的目录进行优化配置。
  ```
  通常只有身为 root 且文件系统有问题时才使用这个命令，否自正常情况下使用，可能会对系统造成损害。  
  【运行 `fsck` 时，被检查的 partition 务必不可挂载到系统上，需要在卸除的状态】  
  系统实际运行的 fsck 命令，其实是呼叫 e2fsck 这个软件
* badblocks
  ```
  badblocks -[svw] 装置名称
  选项与参数：
  -s  ：在屏幕上列出进度
  -v  ：可以在屏幕上看到进度
  -w  ：使用写入的方式来测试，建议不要使用此一参数，尤其是待检查的装置已有文件时！
  ```
  fsck 是检查文件系统是否出错，而 badblocks 则是用来检测硬盘或软盘扇区有没有坏轨。其实是透过【`mke2fs -c 装置文件名`】在进行格式化的时候处理磁盘表面的读取测试，目前大多不使用此命令。
## 磁盘挂载与卸除
* 单一文件系统不应该被重复挂载在不同的挂载点(目录)中
* 单一目录不应该重复挂载多个文件系统
* 要作为挂载点的目录，理论上应该是空目录

如果不是空目录，原目录下的数据，会被暂时隐藏掉。
```
[root@www ~]# mount -a
[root@www ~]# mount [-l]
[root@www ~]# mount [-t 文件系统] [-L Label名] [-o 额外选项] [-n]  装置文件名  挂载点
选项与参数：
-a  ：依照配置文件 /etc/fstab 的数据将所有未挂载的磁盘都挂载上来
-l  ：单纯的输入 mount 会显示目前挂载的信息。加上 -l 可增列 Label 名称！
-t  ：与 mkfs 的选项非常类似的，可以加上文件系统种类来指定欲挂载的类型。
      常见的 Linux 支持类型有：ext2, ext3, vfat, reiserfs, iso9660(光盘格式),
      nfs, cifs, smbfs(此三种为网络文件系统类型)
-n  ：在默认的情况下，系统会将实际挂载的情况实时写入 /etc/mtab 中，以利其他程序
      的运行。但在某些情况下(例如单人维护模式)为了避免问题，会刻意不写入。
      此时就得要使用这个 -n 的选项了。
-L  ：系统除了利用装置文件名 (例如 /dev/hdc6) 之外，还可以利用文件系统的标头名称
      (Label)来进行挂载。最好为你的文件系统取一个独一无二的名称吧！
-o  ：后面可以接一些挂载时额外加上的参数！比方说账号、密码、读写权限等：
      ro, rw:       挂载文件系统成为只读(ro) 或可擦写(rw)
      async, sync:  此文件系统是否使用同步写入 (sync) 或异步 (async) 的
                    内存机制，请参考文件系统运行方式。默认为 async。
      auto, noauto: 允许此 partition 被以 mount -a 自动挂载(auto)
      dev, nodev:   是否允许此 partition 上，可创建装置文件？ dev 为可允许
      suid, nosuid: 是否允许此 partition 含有 suid/sgid 的文件格式？
      exec, noexec: 是否允许此 partition 上拥有可运行 binary 文件？
      user, nouser: 是否允许此 partition 让任何使用者运行 mount ？一般来说，
                    mount 仅有 root 可以进行，但下达 user 参数，则可让
                    一般 user 也能够对此 partition 进行 mount 。
      defaults:     默认值为：rw, suid, dev, exec, auto, nouser, and async
      remount:      重新挂载，这在系统出错，或重新升级参数时，很有用！
```
* 挂载 Ext2/Ext3 文件系统  
  【mount 装置文件名 挂载点】，Linux 可以透过 superblock 搭配 Linux 自己的驱动程序去测试挂载，如果成功套合了，就立刻自动的使用该类型的文件系统。可进行挂在测试参考：
    * /etc/filesystem：系统指定的测试挂载文件系统类型
    * /proc/filesystem：Linux 系统已经加载的文件系统类型

  Linux 支持的文件系统的驱动程序，都写在如下目录：
    * /lib/modules/$(uname -r)/kernel/fs/
  ```
  mount -l
  # /dev/hdc2 是挂载到 / 目录，文件系统类型为 ext3 ，且挂载为可擦写 (rw) ，另外，这个 filesystem 有标头，名字(label)为 /1 
  ```
* 挂载 CD 或 DVD 光盘  
  ```
  mount -t iso9660 /dev/cdrom /media/cdrom
  mount /dev/hdd /media/cdrom
  ```
  光驱挂载之后就无法退出光盘片了，除非将他卸载
* 挂载软盘
* 挂载闪盘(U 盘)  
  ```
  mount -t vfat -o iocharset=cp950 /dev/sda1 /mnt/flash
  # iocharset 用来指定语系，cp950为中文语系
  ```
* 重新挂载根目录与挂载不特定目录  
  根目录不能被卸除，如果要改变挂在参数，或者根目录出现只读状态，重新挂载
  ```
  mount -o remount,rw,auto /
  ```
  也可以利用 mount 来将某个目录挂载到另一个目录去，不是挂载文件系统。与 symbolic link 效果相同，应用于某些不支持符号链接的程序中
  ```
  将 /home 这个目录暂时挂载到 /mnt/home 下
  mount --bind /home /mnt/home
  ```
* umount (将装置文件卸除)
  ```
  umount [-fn] 装置文件名或挂载点
  选项与参数：
  -f：强制卸除，可用在类似网络文件系统 (NFS) 无法读取到的情况下
  -n：不升级 /etc/mtab 情况下卸除
  ```
* 使用 Label name 进行挂载——系统不必知道该文件所在的接口与磁盘文件名
  ```
  dumpe2fs -h /dev/hdc6
  Filesystem volume name:   vbird_logical

  mount -L "vbird_logical" /mnt/hdc6
  ```
## 磁盘参数修订
* mknod  
  Linux 下所有装置都以文件来代表，透过文件的 major 与 minor 数值。主要装置代码(Major) 次要装置代码(Minor)  
  基本上，Linux 核心 2.6 版本以后，硬件文件名都可以被系统自动的实时产生了，不需要手动创建。某些情况下还要手动处理装置文件，例如某些服务被关到特定的目录下时(chroot)。
  ```
  [root@www ~]# mknod 装置文件名 [bcp] [Major] [Minor]
  选项与参数：
  装置种类：
     b  ：配置装置名称成为一个周边储存设备文件，例如硬盘等；
     c  ：配置装置名称成为一个周边输入设备文件，例如鼠标/键盘等；
     p  ：配置装置名称成为一个 FIFO 文件；
  Major ：主要装置代码；
  Minor ：次要装置代码；
  ```
* e2label  
  修改文件系统标头(Label)
  ```
  e2label 装置名称  新的Label名称
  ```
* tune2fs  
  ```
  tune2fs [-jlL] 装置代号
  选项与参数：
  -l：类似 dumpe2fs -h 的功能——将 superblock 内的数据读出来
  -j：将 ext2 的 filesystem 转换为 ext3 的文件系统
  -L：类似 e2label 的功能，可以修改 filesystem 的 Label
  ```
* hdparm  
  如果时 IDE 接口，可以配置一些进阶参数，SATA 接口，只能用来测试。这个命令多用于测试效能。
  ```
  [root@www ~]# hdparm [-icdmXTt] 装置名称
  选项与参数：
  -i  ：将核心侦测到的硬盘参数显示出来！
  -c  ：配置 32-bit (32位)存取模式。这个 32 位存取模式指的是在硬盘在与 
        PCI 接口之间传输的模式，而硬盘本身是依旧以 16 位模式在跑的！
        默认的情况下，这个配置值都会被打开，建议直接使用 c1 即可！
  -d  ：配置是否激活 dma 模式， -d1 为启动， -d0 为取消；
  -m  ：配置同步读取多个 sector 的模式。一般来说，配置此模式，可降低系统因为
        读取磁盘而损耗的效能～不过， WD 的硬盘则不怎么建议配置此值～
        一般来说，配置为 16/32 是优化，不过，WD 硬盘建议值则是 4/8 。
        这个值的最大值，可以利用 hdparm -i /dev/hda 输出的 MaxMultSect
        来配置喔！一般如果不晓得，配置 16 是合理的！
  -X  ：配置 UtraDMA 的模式，一般来说， UDMA 的模式值加 64 即为配置值。
        并且，硬盘与主板芯片必须要同步，所以，取最小的那个。一般来说：
        33 MHz DMA mode 0~2 (X64~X66)
        66 MHz DMA mode 3~4 (X67~X68)
        100MHz DMA mode 5   (X69)
        如果您的硬盘上面显示的是 UATA 100 以上的，那么配置 X69 也不错！
  -T  ：测试缓存区 cache 的存取效能
  -t  ：测试硬盘的实际存取效能 （较正确！）
  ```

# 配置启动挂载
## 启动挂载 /etc/fstab 及 /etc/mtab
系统挂载的限制
* 根目录 / 是必须挂载的﹐而且一定要先于其它 mount point 被挂载进来。
* 其它 mount point 必须为已创建的目录﹐可任意指定﹐但一定要遵守必须的系统目录架构原则
* 所有 mount point 在同一时间之内﹐只能挂载一次。
* 所有 partition 在同一时间之内﹐只能挂载一次。
* 如若进行卸除﹐您必须先将工作目录移到 mount point(及其子目录) 之外。
```
[root@www ~]# cat /etc/fstab
# Device        Mount point   filesystem parameters    dump fsck
LABEL=/1          /           ext3       defaults        1 1
LABEL=/home       /home       ext3       defaults        1 2
LABEL=/boot       /boot       ext3       defaults        1 2
tmpfs             /dev/shm    tmpfs      defaults        0 0
devpts            /dev/pts    devpts     gid=5,mode=620  0 0
sysfs             /sys        sysfs      defaults        0 0
proc              /proc       proc       defaults        0 0
LABEL=SWAP-hdc5   swap        swap       defaults        0 0
```
/etc/fstab (filesystem table)就是将我么利用 mount 命令挂载时，将所有的选项与参数写入这个文件中。此外还加入了 dump 这个备份命令与启动时是否进行文件系统检查 fsck 等命令有关。  
各个字段的详细数据：
* 第一栏：磁盘装置文件名或该装置的 Label  
  默认使用的是 Label 名称，也可以取代称为装置名
* 第二栏：挂载点 (mount point)
* 第三栏：磁盘分区槽的文件系统  
  必须写入
* 第四栏：文件系统参数  
  参数|内容意义
  :---:|:---:
  async/sync<br>异步/同步|配置磁盘是否以异步方式运行！默认为 async(效能较佳)
  auto/noauto<br>自动/非自动|当下达 mount -a 时，此文件系统是否会被主动测试挂载。默认为 auto。
  rw/ro<br>可擦写/只读|让该分割槽以可擦写或者是只读的型态挂载上来，如果你想要分享的数据是不给用户随意变更的， 这里也能够配置为只读。则不论在此文件系统的文件是否配置 w 权限，都无法写入
  exec/noexec<br>可运行/不可运行|限制在此文件系统内是否可以进行『运行』的工作？如果是纯粹用来储存数据的， 那么可以配置为 noexec 会比较安全，相对的，会比较麻烦！
  user/nouser<br>允许/不允许使用者挂载|是否允许用户使用 mount 命令来挂载呢？一般而言，我们当然不希望一般身份的 user 能使用 mount ，因为太不安全了，因此这里应该要配置为 nouser
  suid/nosuid<br>具有/不具有 suid 权限|该文件系统是否允许 SUID 的存在？如果不是运行文件放置目录，也可以配置为 nosuid 来取消这个功能！
  usrquota|注意名称是『 usrquota 』不要拼错了！这个是在启动 filesystem 支持磁盘配额模式，更多数据我们在第四篇再谈。
  grpquota|注意名称是『grpquota』，启动 filesystem 对群组磁盘配额模式的支持。
  defaults|同时具有 rw, suid, dev, exec, auto, nouser, async 等参数。 基本上，默认情况使用 defaults 配置即可！
* 第五栏：能否被 dump 备份命令作用：  
  可以透过 fstab 指定哪个文件系统必须进行 dump 备份。0 代表不要做 dump 备份，1 代表每天进行 dump 的动作，2 代表其他不定日期的 dump 备份动作，通常这个数值不是 0 就是 1 。
* 第六栏：是否以 fsck 检查扇区：  
  启动过程中，系统默认会以 fsck 检验我们的 filesystem 是否完整 (clean)。不过某些 filesystem 不需要检验，例如内存交换空间(swap)，或者是特殊文件系统例如 /proc 与 /sys 等。0 是不要检验，1 表示最早检验 (一般只有根目录会配置为 1)，2 也是要检验，一般其他要检验的配置为 2。

/etc/fstab 是启动时的配置文件，实际 filesystem 的挂载时记录到 /etc/mtab 与 /proc/mounts 这两个文件中。每次更动 filesystem 的挂载时，也会同时更动这两个文件。   
万一发生 /etc/fstab 输入数据错误，导致无法顺利启动，而进入单人维护模式中，/ 是 read only 状态，无法修改 /etc/fstab。可以利用重新挂载命令：`mount -n -o remount,rw /`
## 特殊装置 loop 挂载 (映像档不刻录就挂载使用)
* 挂载光盘/DVD映像文件
  ```
  mount -o loop /root/centos5.2_x86_64.iso /mnt/centos_dvd

  umount /mnt/centos_dvd/
  ```
* 创建大文件以制作 loop 装置文件  
  可以解决很多系统分割不良的情况。或 Linux 下虚拟机的应用。
    * 创建大型文件  
      `dd`可以创建空的文件
      ```
      dd if=/dev/zero of=/home/loopdev bs=1M count=512
      # if 是 input file ，输入文件。那个 /dev/zero 是会一直输出 0 的装置！
      # of 是 output file ，将一堆零写入到后面接的文件中。
      # bs 是每个 block 大小，想文件系统那样的 block 意义
      # count 则是有几个 bs 的意思
      ```
   * 格式化  
     ```
     mkfs -t ext3 /home/loopdev
     ```
   * 挂载
     ```
     mount -o loop /home/loopdev /media/cdrom/
     ```

# 内存置换空间(swap)之建置
系统已经创建起来之后，没有建置 swap ，需要增加 swap
* 配置一个 swap partition
* 创建一个虚拟内存的文件
## 使用实体分割槽建置swap
1. 分割：先使用 fdisk 在磁盘中分割出一个分割槽给系统作为 swap。由于 Linux 的 fdisk 会默认将分割槽的 ID 配置为 Linux 的文件系统，所以还要配置一下 system ID。
2. 格式化：利用创建 swap 格式的 【mkswap 装置文件名】就能将分割槽格式化成 swap 格式
3. 使用：最后将该 swap 装置启动，方法为：【swapon 装置文件名】
4. 观察测试
* 分割：
  ```
  [root@www ~]# fdisk /dev/hdc
  Command (m for help): n
  First cylinder (2303-5005, default 2303):  <==这里按[enter]
  Using default value 2303
  Last cylinder or +size or +sizeM or +sizeK (2303-5005, default 5005): +256M

  Command (m for help): p

     Device Boot      Start         End      Blocks   Id  System
  .....中间省略.....
  /dev/hdc6            2053        2302     2008093+  83  Linux
  /dev/hdc7            2303        2334      257008+  83  Linux <==新增的项目

  Command (m for help): t             <==修改系统 ID
  Partition number (1-7): 7           <==从上结果看到的，七号partition
  Hex code (type L to list codes): 82 <==改成 swap 的 ID
  Changed system type of partition 7 to 82 (Linux swap / Solaris)

  Command (m for help): p

     Device Boot      Start         End      Blocks   Id  System
  .....中间省略.....
  /dev/hdc6            2053        2302     2008093+  83  Linux
  /dev/hdc7            2303        2334      257008+  82  Linux swap / Solaris

  Command (m for help): w
  # 此时就将 partition table 升级了！

  [root@www ~]# partprobe
  # 不要忘记让核心升级 partition table 
  ```
## 使用文件建置swap
* 使用 `dd` 命令新增一个大文件
* 使用 `mkswap` 将文件格式化为 swap 的文件格式
* 使用 `swapon` 启动
* 使用 `swapoff` 关掉 swap file
## swap 使用的限制
swap 主要的功能是当物理内存不够时，则某些在内存当中所占的程序会暂时被移动到 swap 当中，让物理内存可以被需要的程序来使用。另外，如果你的主机支持电源管理模式， 也就是说，你的 Linux 主机系统可以进入『休眠』模式的话，那么， 运行当中的程序状态则会被纪录到 swap 去，以作为『唤醒』主机的状态依据！ 另外，有某些程序在运行时，本来就会利用 swap 的特性来存放一些数据段， 所以，一般情况 swap 来是需要创建的  
限制：
* 在核心 2.4.10 版本以后，单一 swap 量已经没有 2GB 的限制了，
* 但是，最多还是仅能创建到 32 个 swap 的数量
* 而且，由于目前 x86_64 (64位) 最大内存寻址到 64GB， 因此， swap 总量最大也是仅能达 64GB 就是了

# 文件系统的特殊观察与操作
## boot sector 与 superbloc 的关系
可安装启动信息的 boot sector (启动扇区) 独立出来，并非放置到 superblock 中。
* superblock 大小为 1024 bytes;
* superblock 前面需要保留 1024 bytes 下来，让启动管理程序可以安装
### block 为 1024 bytes (1K) 时：
0 号 block 保留下来，留给 boot sector 使用，superblock 由 1 号 block 开始。
### block 大于 1024 bytes (2K, 4K) 时：
superblock 将会在 0 号。第一个 block 内含有 boot sector 与 superblock 两者。boot sector 占用 superblock 前面的 1024 bytes
## 磁盘空间的浪费
superblock, inode table 与其他中介数据等都会浪费磁盘容量。一个 block 只能放置一个文件，也造成浪费。  
查询目录所耗用的所有容量时，使用 `du -s` 命令，再使用 block 去测试，差别就是文件系统的耗费。
## 利用 GNU 的 parted 进行分割行为
`fdisk` 无法支持高于 2TB 以上的分割槽。
```
[root@www ~]# parted [装置] [命令 [参数]]
选项与参数：
命令功能：
新增分割：mkpart [primary|logical|extended] [ext3|vfat] 开始 结束
分割表  ：print
删除分割：rm [partition]

范例一：以 parted 列出目前本机的分割表数据
[root@www ~]# parted /dev/hdc print
Model: IC35L040AVER07-0 (ide)              <==硬盘接口与型号
Disk /dev/hdc: 41.2GB                      <==磁盘文件名与容量
Sector size (logical/physical): 512B/512B  <==每个扇区的大小
Partition Table: msdos                     <==分割表形式

Number  Start   End     Size    Type      File system  Flags
 1      32.3kB  107MB   107MB   primary   ext3         boot
 2      107MB   10.6GB  10.5GB  primary   ext3
 3      10.6GB  15.8GB  5240MB  primary   ext3
 4      15.8GB  41.2GB  25.3GB  extended
 5      15.8GB  16.9GB  1045MB  logical   linux-swap
 6      16.9GB  18.9GB  2056MB  logical   ext3
 7      18.9GB  19.2GB  263MB   logical   linux-swap
[  1 ]  [  2 ]  [  3  ] [  4  ] [  5  ]   [  6  ]

创建一个约为 512MB 容量的逻辑分割槽
parted /dev/hdc mkpart logical ext3 19.2GB 19.7GB
```
1. Number：这个就是分割槽的号码，举例来说，1号代表的是 /dev/hdc1 的意思；
2. Start：起始的磁柱位置在这颗磁盘的多少 MB 处？以容量作为单位
3. End：结束的磁柱位置在这颗磁盘的多少 MB 处？
4. Size：由上述两者的分析，得到这个分割槽有多少容量；
5. Type：就是分割槽的类型，有primary, extended, logical等类型；
6. File system：就如同 fdisk 的 System ID 之意。