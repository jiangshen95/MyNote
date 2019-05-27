# 什么是程序 (Progress)
Linux 底下所有的命令与你能够进行的动作都与权限有关。在 Linux 系统当中：『触发任何一个事件时，系统都会将他定义成为一个程序，并且给予这个程序一个 ID ，称为 PID，同时依据启发这个程序的使用者与相关属性关系，给予这个 PID 一组有效的权限配置。』 从此以后，这个 PID 能够在系统上面进行的动作，就与这个 PID 的权限有关了。
## 程序与程序 (precess & program)
通过『运行一个程序或命令』就可以触发一个事件而取得一个 PID ，产生一个程序。而运行的程序 binary file，就是 program。  
『不同的使用者身份运行这个 program 时，系统给予的权限也都不相同！』  
程序一般是放置在实体磁碟中，然后透过使用者的运行来触发。触发后会加载到内存中成为一个个体，那就是程序。 为了操作系统可管理这个程序，因此程序有给予运行者的权限/属性等参数，并包括程序所需要的命令码与数据或文件数据等， 最后再给予一个 PID 。系统就是透过这个 PID 来判断该 process 是否具有权限进行工作的！  
当我们登陆并运行 bash 时，系统已经给我们一个 PID 了，这个 PID 就是依据登陆者的 UID/GID (/etc/passwd) 来的， /bin/bash 是一个程序 (program)，一个程序衍生出来的其他程序在一般状态下，也会沿用这个程序的相关权限：
* 程序 (program)：通常为 binary program ，放置在储存媒体中 (如硬盘、光盘、软盘、磁带等)， 为实体文件的型态存在；
* 程序 (process)：程序被触发后，运行者的权限与属性、程序的程序码与所需数据等都会被加载内存中， 操作系统并给予这个内存内的单元一个识别码 (PID)，可以说，程序就是一个正在运行中的程序。
### 子程序与父程序
当我们登陆系统后，会取得一个 bash 的 shell ，然后，我们用这个 bash 提供的介面去运行另一个命令，例如 /usr/bin/passwd 或者是 touch 等等，那些另外运行的命令也会被触发成为 PID ，后来运行命令才产生的 PID 就是『子程序』，而在原本的 bash 环境下，就称为『父程序』。  
程序之间是有相关性的。因为每个程序都有一个 PID ，程序的父程序透过 Parent PID (PPID) 来判断。子程序可以取得父程序的环境变量。
### fork and exec：程序呼叫的流程
在 Linux 的程序呼叫通常称为 fork-and-exec 的流程，程序都会藉由父程序以复制 (fork) 的方式产生一个一模一样的子程序， 然后被复制出来的子程序再以 exec 的方式来运行实际要进行的程序，最终就成为一个子程序的存在。
### 系统或网络服务：常驻在内存的程序
常驻在内存当中的程序通常都是负责一些系统所提供的功能以服务使用者各项任务，因此这些常驻程序就会被我们称为：服务 (daemon)。系统的服务非常的多， 不过主要大致分成系统本身所需要的服务，例如刚刚提到的 crond 及 atd ，还有 syslog 等等的。还有一些则是负责网络连线的服务，例如 Apache, named, postfix, vsftpd... 等等的。这些网络服务比较有趣的地方，在于这些程序被运行后，他会启动一个可以负责网络监听的端口 (port) ，以提供外部用户端 (client) 的连线要求。
## Linux 的多人多工环境
在 Linux 底下运行一个命令时，系统会将相关的权限、属性、程序码与数据等均加载内存， 并给予这个单元一个程序识别码 (PID)，最终该命令可以进行的任务则与这个 PID 的权限有关。
* 多人环境：  
  在 Linux 系统上面具有多种不同的帐号， 每种帐号都有都有其特殊的权限，只有一个人具有至高无上的权力，那就是 root (系统管理员)。除了 root 之外，其他人都必须要受一些限制的！而每个人进入 Linux 的环境配置都可以随著每个人的喜好来配置。每个人登陆后取得的 shell 的 PID 不同。
* 多工行为：  
  CPU 切换程序的工作，与这些工作进入到 CPU 运行的排程 (CPU 排程，非 crontab 排程) 会影响到系统的整体效能！ 目前 Linux 使用的多工切换行为，可以对每个用户都有较好的效能。
* 多重登陆环境的七个基本终端窗口：  
  在 Linux 当中，默认提供了六个文字界面登陆窗口，以及一个图形界面，你可以使用 [Alt]+[F1].....[F7] 来切换不同的终端机界面，而且每个终端机界面的登陆者还可以不同人。  
  Linux 默认会启动六个终端机登陆环境的程序，所以我们就会有六个终端机介面。也可以减少启动的终端机程序，详细的数据查阅 /etc/inittab 这个文件。
* 特殊的程序管理行为：  
  Linux 可以在任何时候，将某个被困住的程序杀掉，然后再重新运行而不是重新启动。如果在 Linux 下以文字界面登陆，在屏幕当中显示错误信息后就不能操作了，可以随意的再按 [Alt]+[F1].....[F7] 来切换到其他的终端机界面，然后以 ps -aux 找出刚刚的错误程序，然后给他 kill 掉，回到刚刚的终端机界面，即可恢复正常。  
  每个程序之间可能是独立的，也可能有相依性， 只要到独立的程序当中，删除有问题的那个程序，当然他就可以被系统移除掉了。
* bash 环境下的工作管理 (job control)  
  登陆 bash 之后， 就是取得一个名为 bash 的 PID 了，而在这个环境底下所运行的其他命令， 就几乎都是所谓的子程序。在单一的 bash 界面下，依然可以『同时』进行多个工作
  ```
  [root@www ~]# cp file1 file2 &
  ```
  & 符号表示将命令放置到背景中运行，也就是说，运行这一命令之后，在这一个终端界面任然可以做其他工作。而当这一命令运行完毕之后，系统会在终端界面显示完成的消息。
* 多人多工的系统资源分配问题考虑：  
  
# 工作管理 (job control)
这个工作管理 (job control) 是用在 bash 环境下的，也就是说：『当我们登陆系统取得 bash shell 之后，在单一终端机介面下同时进行多个工作的行为管理 』。
## 什么是工作管理
『进行工作管理的行为中， 其实每个工作都是目前 bash 的子程序，亦即彼此之间是有相关性的。 我们无法以 job control 的方式由 tty1 的环境去管理 tty2 的 bash 』。  
出现提示字节可操作的环境称为前景 (foreground)，其他工作就可以放入背景 (background) 去暂停或运行。要注意的是，放入背景的工作想要运行时， 他必须不能够与使用者互动。而且放入背景的工作是不可以使用 [ctrl]+c 来终止的』！  
要进行 bash 的 job control 要注意的限制是：
* 这些工作所触发的程序必须来自于你 shell 的子程序(只管理自己的 bash)；
* 前景：你可以控制与下达命令的这个环境称为前景的工作 (foreground)；
* 背景：可以自行运行的工作，你无法使用 [ctrl]+c 终止他，可使用 bg/fg 呼叫该工作；
* 背景中『运行』的程序不能等待 terminal/shell 的输入(input)
## job control 的管理
### 直接将命令丢到背景中【运行】的 &
```
[root@www ~]# tar -zpcf /tmp/etc.tar.gz /etc &
[1] 8400  <== [job number] PID 
[root@www ~]# tar: Removing leading `/' from member names 
# 在中括号内的号码为工作号码 (job number)，该号码与 bash 的控制有关。
# 后续的 8400 则是这个工作在系统中的 PID。至于后续出现的数据是 tar 运行的数据流，
# 由于我们没有加上数据流重导向，所以会影响画面！不过不会影响前景的操作
```
背景中的工作完成之后，会显示
```
[1]+  Done                    tar -zpcf /tmp/etc.tar.gz /etc  # 后面接该工作的命令
```
最好使用数据流重导向，将输出数据导向到某个文件中。
```
[root@www ~]# tar -zpcvf /tmp/etc.tar.gz /etc > /tmp/log.txt 2>&1 &
[1] 8429
```
### 将【目前】的工作丢到背景中【暂停】：[ctrl]-z
比如在 vi 下工作时。
```
# 在 vi 的一般模式下，按下 [ctrl]-z 这两个按键
[1]+  Stopped                 vim ~/.bashrc
```
+ 代表最近一个被丢进背景的工作，且目前在背景下默认会被取用的那个工作 (与 fg 这个命令有关 )。而 Stopped 代表目前这个工作的状态。在默认的情况下，使用 [ctrl]-z 丢到背景当中的工作都是『暂停』的状态。
### 观察目前的背景工作状态：jobs
```
[root@www ~]# jobs [-lrs]
选项与参数：
-l  ：除了列出 job number 与命令串之外，同时列出 PID 的号码；
-r  ：仅列出正在背景 run 的工作；
-s  ：仅列出正在背景当中暂停 (stop) 的工作。

[root@www ~]# jobs -l
[1]- 10314 Stopped                 vim ~/.bashrc
[2]+ 10833 Stopped                 find / -print
```
\+ 代表默认的取用工作。如果仅输入 fg 时，那么 [2] 会被拿到前景当中来处理。- 代表最近最后第二个被放置到背景中的工作号码。 而超过最后第三个以后的工作，就不会有 +/- 符号存在了。
### 将背景工作拿到前景来处理：fg
``` 
[root@www ~]# fg %jobnumber
选项与参数：
%jobnumber ：jobnumber 为工作号码(数字)。注意，那个 % 是可有可无的

[root@www ~]# fg      <==默认取出那个 + 的工作
```
带 + 号的默认取用工作，一般是最后一个放入背景中的工作。输入『 fg - 』 则代表将 - 号的那个工作号码拿出来。
### 让工作在背景下的状态变成运行中：bg
 [ctrl]-z 可以将目前的工作丢到背景底下去『暂停』。让一个工作在背景下【Run】
 ```
 [root@www ~]# jobs ; bg %3 ; jobs
[1]-  Stopped                 vim ~/.bashrc
[2]   Stopped                 find / -print
[3]+  Stopped                 find / -perm +7000 > /tmp/text.txt
[3]+ find / -perm +7000 > /tmp/text.txt &  <==用 bg%3 的情况！
[1]+  Stopped                 vim ~/.bashrc
[2]   Stopped                 find / -print
[3]-  Running                 find / -perm +7000 > /tmp/text.txt &
 ```
 ### 管理背景当中的工作：kill
 ```
[root@www ~]# kill -signal %jobnumber
[root@www ~]# kill -l
选项与参数：
-l  ：这个是 L 的小写，列出目前 kill 能够使用的讯号 (signal) 有哪些？
signal ：代表给予后面接的那个工作什么样的指示罗！用 man 7 signal 可知：
  -1 ：重新读取一次参数的配置档 (类似 reload)；
  -2 ：代表与由键盘输入 [ctrl]-c 同样的动作；
  -9 ：立刻强制删除一个工作；
  -15：以正常的程序方式终止一项工作。与 -9 是不一样的。
 ``` 
-9 这个 signal 通常是用在『强制删除一个不正常的工作』时所使用的， -15 则是以正常步骤结束一项工作(15也是默认值)，两者之间并不相同。举例来说，vi 在背景运行的时候，使用 -15 这个 signal 时，vi 会尝试以正常的步骤来结束掉该 vi 的工作，所以 .filename.swp 会主动的被移除，而使用 -9 这个 signal 时，由于 vi 工作会被强制移除掉，因此 .filename.swp 会继续存在文件系统中。  
killall 与 kill 相同的用法。kill 后面接的数字默认会是 PID ，如果想要管理 bash 的工作控制，就得要加上 %数字 了。
## 离线管理问题
我们在工作管理当中提到的『背景』指的是在终端机模式下可以避免 [crtl]-c 中断的一个情境， 并不是放到系统的背景去。所以，工作管理的背景依旧与终端机有关。在这样的情况下，如果你是以远程连线方式连接到你的 Linux 主机，并且将工作以 & 的方式放到背景去，在工作尚未结束的情况下离线，工作不会继续进行，而会被中断掉。  
此种情况，可以使用 at 来处理，at 是将工作放置到系统背景，而与终端机无关。也可以使用 nohup 这个命令，可以在离线注销系统后，工作继续进行。
```
[root@www ~]# nohup [命令与参数]   <==在终端机前景中工作
[root@www ~]# nohup [命令与参数] & <==在终端机背景中工作
```
nohup 并不支持 bash 内建的命令，命令必须是外部命令才行。

# 程序管理
## 程序的观察
利用静态的 ps 或者动态的 top ，查阅系统上面正在运行的程序。还能以 pstree 查阅程序树之间的关系。
### ps：将某个时间点的程序运行情况撷取下来
```
[root@www ~]# ps aux  <==观察系统所有的程序数据
[root@www ~]# ps -lA  <==也是能够观察所有系统的数据
[root@www ~]# ps axjf <==连同部分程序树状态
选项与参数：
-A  ：所有的 process 均显示出来，与 -e 具有同样的效用；
-a  ：不与 terminal 有关的所有 process ；
-u  ：有效使用者 (effective user) 相关的 process ；
x   ：通常与 a 这个参数一起使用，可列出较完整资讯。
输出格式规划：
l   ：较长、较详细的将该 PID 的的资讯列出；
j   ：工作的格式 (jobs format)
-f  ：做一个更为完整的输出。
```
一个是只能查阅自己 bash 程序的『 ps -l 』一个则是可以查阅所有系统运行的程序『 ps aux 』.
* 仅观察自己的 bash 相关程序：ps -l  
  ```
  [root@www ~]# ps -l
  F S   UID   PID  PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD
  4 S     0 13639 13637  0  75   0 -  1287 wait   pts/1    00:00:00 bash
  4 R     0 13700 13639  0  77   0 -  1101 -      pts/1    00:00:00 ps
  ```
  使用 ps -l 仅列出与你的操作环境 (bash) 有关的程序而已， 亦即最上一级的父程序会是你自己的 bash 而没有延伸到 init 这支程序去。
    * F：代表这个程序旗标 (process flags)，说明这个程序的总结权限，常见号码有：  
        * 若为 4 表示此程序的权限为 root;
        * 若为 1 表示此子程序仅进行复制(fork)而没有实际运行(exec)
    * S：代表这个程序的状态 (STAT)，主要状态有：  
        * R (Running)：该程序正在运行中；
        * S (Sleep)：该程序目前正在睡眠状态(idle)，但可以被唤醒(signal)。
        * D：不可被唤醒的睡眠状态，通常这只程序可能在等待 I/O 的情况(ex>列印)
        * T：停止状态(stop)，可能是在工作控制(背景暂停)或除错(traced)状态；
        * Z (Zombie)：僵尸状态，程序已经终止但却无法被移除至内存外。
    * UID/PID/PPID：代表『此程序被该 UID 所拥有/程序的 PID 号码/此程序的父程序 PID 号码』
    * C：代表 CPU 使用率，单位为百分比；
    * PRI/NI：Priority/Nice 的缩写，代表此程序被 CPU 所运行的优先顺序，数值越小代表该程序越快被 CPU 运行。
    * ADDR/SZ/WCHAN：都与内存有关，ADDR 是 kernel function，指出该程序在内存的哪个部分，如果是个 running 的程序，一般就会显示『 - 』 / SZ 代表此程序用掉多少内存 / WCHAN 表示目前程序是否运行中，同样的， 若为 - 表示正在运行中。
    * TTY：登陆者的终端机位置，若为远程登陆则使用动态终端介面 (pts/n)；
    * TIME：使用掉的 CPU 时间，注意，是此程序实际花费 CPU 运行的时间，而不是系统时间；
    * CMD：就是 command 的缩写，造成此程序的触发程序之命令为何。
* 观察系统所有程序：ps aux
  ```
  [root@www ~]# ps aux
  USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
  root         1  0.0  0.0   2064   616 ?        Ss   Mar11   0:01 init [5]
  root         2  0.0  0.0      0     0 ?        S<   Mar11   0:00 [migration/0]
  root         3  0.0  0.0      0     0 ?        SN   Mar11   0:00 [ksoftirqd/0]
  .....(中间省略).....
  root     13639  0.0  0.2   5148  1508 pts/1    Ss   11:44   0:00 -bash
  root     14232  0.0  0.1   4452   876 pts/1    R+   15:52   0:00 ps aux
  root     18593  0.0  0.0   2240   476 ?        Ss   Mar14   0:00 /usr/sbin/atd
  ```
  格栏位的意义为：
  * USER：该 process 属于那个使用者帐号的？
  * PID ：该 process 的程序识别码。
  * %CPU：该 process 使用掉的 CPU 资源百分比；
  * %MEM：该 process 所占用的实体内存百分比；
  * VSZ ：该 process 使用掉的虚拟内存量 (Kbytes)
  * RSS ：该 process 占用的固定的内存量 (Kbytes)
  * TTY ：该 process 是在那个终端机上面运行，若与终端机无关则显示 ?，另外， tty1-tty6 是本机上面的登陆者程序，若为 pts/0 等等的，则表示为由网络连接进主机的程序。
  * STAT：该程序目前的状态，状态显示与 ps -l 的 S 旗标相同 (R/S/T/Z)
  * START：该 process 被触发启动的时间；
  * TIME ：该 process 实际使用 CPU 运行的时间。
  * COMMAND：该程序的实际命令为何？

  一般来说，ps aux 会依照 PID 的顺序来排序显示
  ```
  pa -lA
  # 每个栏位与 ps -l 的输出情况相同，但显示的程序包括系统所有的程序

  ps axjf
  # 列出类似程序树的程序显示
  ```
  ```
  找出与 cron 与 syslog 这两个服务有关的 PID 号码？
  [root@www ~]# ps aux | egrep '(cron|syslog)'
  ```
  通常，造成僵尸程序的成因是因为该程序应该已经运行完毕，或者是因故应该要终止了， 但是该程序的父程序却无法完整的将该程序结束掉，而造成那个程序一直存在内存当中。 某个程序的 CMD 后面还接上 \<defunct\> 时，就代表该程序是僵尸程序。  
  当系统不稳定的时候就容易造成所谓的僵尸程序，可能是因为程序写的不好，或者是使用者的操作习惯不良等等所造成。 如果系统中发现很多僵尸程序，要找出父程序，然后追踪和进行主机环境优化。  
  事实上，通常僵尸程序都已经无法控管，而直接是交给 init 这支程序来负责了，偏偏 init 是系统第一支运行的程序， 他是所有程序的父程序！我们无法杀掉该程序，所以，如果产生僵尸程序，而系统过一阵子还没有办法通过核心非经常性的特殊处理来将该程序删除时，只能透过 reboot 的方式将该程序抹去。
### top：动态观察程序的变化
```
[root@www ~]# top [-d 数字] | top [-bnp]
选项与参数：
-d  ：后面可以接秒数，就是整个程序画面升级的秒数。默认是 5 秒；
-b  ：以批量的方式运行 top ，还有更多的参数可以使用
      通常会搭配数据流重导向来将批量的结果输出成为文件。
-n  ：与 -b 搭配，意义是，需要进行几次 top 的输出结果。
-p  ：指定某些个 PID 来进行观察监测而已。
在 top 运行过程当中可以使用的按键命令：
	? ：显示在 top 当中可以输入的按键命令；
	P ：以 CPU 的使用资源排序显示；
	M ：以 Memory 的使用资源排序显示；
	N ：以 PID 来排序
	T ：由该 Process 使用的 CPU 时间累积 (TIME+) 排序。
	k ：给予某个 PID 一个讯号  (signal)
	r ：给予某个 PID 重新制订一个 nice 值。
	q ：离开 top 软件的按键。
```
```
[root@www ~]# top -d 2
top - 17:03:09 up 7 days, 16:16,  1 user,  load average: 0.00, 0.00, 0.00
Tasks:  80 total,   1 running,  79 sleeping,   0 stopped,   0 zombie
Cpu(s):  0.5%us,  0.5%sy,  0.0%ni, 99.0%id,  0.0%wa,  0.0%hi,  0.0%si,  0.0%st
Mem:    742664k total,   681672k used,    60992k free,   125336k buffers
Swap:  1020088k total,       28k used,  1020060k free,   311156k cached
    <==如果加入 k 或 r 时，就会有相关的字样出现在这里喔！
  PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND     
14398 root      15   0  2188 1012  816 R  0.5  0.1   0:00.05 top
    1 root      15   0  2064  616  528 S  0.0  0.1   0:01.38 init
    2 root      RT  -5     0    0    0 S  0.0  0.0   0:00.00 migration/0
    3 root      34  19     0    0    0 S  0.0  0.0   0:00.00 ksoftirqd/0
```
 top 这个程序可以持续的监测整个系统的程序工作状态。 在默认的情况下，每次升级程序资源的时间为 5 秒，不过，可以使用 -d 来进行修改。 top 主要分为两个画面，上面的画面为整个系统的资源使用状态，基本上总共有六行，显示的内容依序是：
* 第一行(top...)：这一行显示的资讯分别为：
    * 目前的时间，亦即是 17:03:09 那个项目；
    * 启动到目前为止所经过的时间，亦即是 up 7days, 16:16 那个项目；
    * 已经登陆系统的使用者人数，亦即是 1 user项目；
    * 系统在 1, 5, 15 分钟的平均工作负载。我们在第十六章谈到的 batch 工作方式为负载小于 0.8 就是这个负载，代表的是 1, 5, 15 分钟，系统平均要负责运行几个程序(工作)的意思。 越小代表系统越闲置，若高于 1 得要注意你的系统程序是否太过繁复了
* 第二行(Tasks...)：显示的是目前程序的总量与个别程序在什么状态(running, sleeping, stopped, zombie)。 比较需要注意的是最后的 zombie 那个数值，如果不是 0 ！好好看看到底是那个 process 变成僵尸了
* 第三行(Cpus...)：显示的是 CPU 的整体负载，每个项目可使用 ? 查阅。需要特别注意的是 %wa ，那个项目代表的是 I/O wait， 通常你的系统会变慢都是 I/O 产生的问题比较大！因此这里得要注意这个项目耗用 CPU 的资源， 另外，如果是多核心的设备，可以按下数字键『1』来切换成不同 CPU 的负载率。
* 第四行与第五行：表示目前的实体内存与虚拟内存 (Mem/Swap) 的使用情况。 再次重申，要注意的是 swap 的使用量要尽量的少！如果 swap 被用的很大量，表示系统的实体内存实在不足！
* 第六行：这个是当在 top 程序当中输入命令时，显示状态的地方。

top 下半部分的画面，则是每个 process 使用的资源情况。需要注意的是：
* PID ：每个 process 的 ID
* USER：该 process 所属的使用者；
* PR ：Priority 的简写，程序的优先运行顺序，越小越早被运行；
* NI ：Nice 的简写，与 Priority 有关，也是越小越早被运行；
* %CPU：CPU 的使用率；
* %MEM：内存的使用率；
* TIME+：CPU 使用时间的累加；

top 默认使用 CPU 使用率 (%CPU) 作为排序的重点，如果你想要使用内存使用率排序，则可以按下『M』， 若要回复则按下『P』即可。如果想要离开 top 则按下『 q 』吧！如果你想要将 top 的结果输出成为文件时， 可以这样做：
```
将 top 的资讯进行 2 次，然后将结果输出到 /tmp/top.txt
[root@www ~]# top -b -n 2 > /tmp/top.txt
```
可以将某个时段 top 观察到的结果存成文件，可以用在你想要在系统背景底下运行，不受终端机屏幕限制，可以得到全部的程序画面。
```
# 使用 top 持续观察当前 bash ， bash PID 可由 $$ 变量取得
[root@www ~]# echo $$
13639  <=== bash 的 PID
[root@www ~]# top -d 2 -p 13639
top - 17:31:56 up 7 days, 16:45,  1 user,  load average: 0.00, 0.00, 0.00
Tasks:   1 total,   0 running,   1 sleeping,   0 stopped,   0 zombie
Cpu(s):  0.0%us,  0.0%sy,  0.0%ni,100.0%id,  0.0%wa,  0.0%hi,  0.0%si,  0.0%st
Mem:    742664k total,   682540k used,    60124k free,   126548k buffers
Swap:  1020088k total,       28k used,  1020060k free,   311276k cached

  PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND
13639 root      15   0  5148 1508 1220 S  0.0  0.2   0:00.18 bash
```
```
# 修改 NI 值
在 top 界面中，直接按 r ，然后输入 PID 号码，修改该程序的 NICE 值
```
### pstree
```
[root@www ~]# pstree [-A|U] [-up]
选项与参数：
-A  ：各程序树之间的连接以 ASCII 字节来连接；
-U  ：各程序树之间的连接以万国码的字节来连接。在某些终端介面下可能会有错误；
-p  ：并同时列出每个 process 的 PID；
-u  ：并同时列出每个 process 的所属帐号名称。

列出目前系统上面所有的程序树的相关性：
[root@www ~]# pstree -A
init-+-acpid
     |-atd
     |-auditd-+-audispd---{audispd}  <==这行与底下一行为 auditd 分出来的子程序
     |        `-{auditd}
     |-automount---4*[{automount}]   <==默认情况下，相似的程序会以数字显示
....(中间省略)....
     |-sshd---sshd---bash---pstree   <==就是我们命令运行的那个相依性！

[root@www ~]# pstree -Aup    # 同时列出 PID 与 users
```
一般连结符号可以使用 ASCII 码即可，但有时因为语系问题会主动的以 Unicode 的符号来连结， 但因为可能终端机无法支持该编码，或许会造成乱码问题。因此可以加上 -A 选项来克服此类线段乱码问题。  
由 pstree 的输出我们也可以很清楚的知道，所有的程序都是依附在 init 这支程序底下的，这支程序的 PID 是 1 号。因为他是由 Linux 核心所主动呼叫的第一支程序。  
如果子程序挂掉或者总是杀不掉子程序，可以使用 pstree 查找父程序。
## 程序的管理
程序之间是可以互相控制的。通过给予这个程序一个讯号 (signal) 去告诉该程序需要让他做什么。  
要给予某个已经存在背景中的工作某些动作时，是直接给予一个讯号给该工作号码即可。可以使用 kill -l (小写的 L ) 或者是 man 7 signal 查询信号。主要的讯号代号与名称对应及内容是：
代号|名称|内容
:---:|:---:|:---:
1|SIGHUP|启动被终止的程序，可让该 PID 重新读取自己的配置档，类似重新启动
2|SIGINT|相当于用键盘输入 [ctrl]-c 来中断一个程序的进行
9|SIGKILL|代表强制中断一个程序的进行，如果该程序进行到一半， 那么尚未完成的部分可能会有『半产品』产生，类似 vim 会有 .filename.swp 保留下来。
15|SIGTERM|以正常的结束程序来终止该程序。由于是正常的终止， 所以后续的动作会将他完成。不过，如果该程序已经发生问题，就是无法使用正常的方法终止时， 输入这个 signal 也是没有用的。
17|SIGSTOP|相当于用键盘输入 [ctrl]-z 来暂停一个程序的进行
### kill -signal PID
kill 可以帮我们将这个 signal 传送给某个工作 (%jobnumber) 或者是某个 PID (直接输入数字)。kill 后面直接加数字与加上 %number 的情况是不同的，% 是专门用在工作控制。
### killall -signal 命令名称
由于 kill 后面必须要加上 PID (或者是 job number)，所以，通常 kill 都会配合 ps, pstree 等命令，用来找到程序的 PID。使用 killall 可以直接利用『下达命令的名称』来给予讯号。
```
[root@www ~]# killall [-iIe] [command name]
选项与参数：
-i  ：interactive 的意思，互动式的，若需要删除时，会出现提示字节给使用者；
-e  ：exact 的意思，表示『后面接的 command name 要一致』，但整个完整的命令
      不能超过 15 个字节。
-I  ：命令名称(可能含参数)忽略大小写。

给予 syslogd 这个命令启动的 PID 一个 SIGHUP 的讯号
[root@www ~]# killall -1 syslogd
# 如果用 ps aux 仔细看一下，syslogd 才是完整的命令名称。但若包含整个参数，则 syslogd -m 0 才是完整的

强制终止所有以 httpd 启动的程序
[root@www ~]# killall -9 httpd
```
总之，要删除某个程序，我们可以使用 PID 或者是启动该程序的命令名称， 而如果要删除某个服务，最简单的方法就是利用 killall ， 因为他可以将系统当中所有以某个命令名称启动的程序全部删除。
## 关于程序的运行顺序
>  CPU 排程指的是每支程序被 CPU 运行的演算守则， 而例行性工作排程则是将某支程序安排在某个时间再交由系统运行。 CPU 排程与操作系统较具有相关性
### Priority 与 Nice 值
『优先运行序 (priority, PRI)』， PRI 值越低代表越优先的意思。不过这个 PRI 值是由核心动态调整的， 使用者无法直接调整 PRI 值。  
如果想要调整程序的优先运行序，就得要透过 Nice 值。一般来说，PRI 与 NI 的相关性如下：  
`PRI (new) = PRI (old) + nice`
因为 PRI 是系统『动态』决定的，所以，虽然 nice 值是可以影响 PRI ，不过， 最终的 PRI 仍是要经过系统分析后才会决定的。nice 的值是有正负的，而既然 PRI 越小越早被运行， 所以，当 nice 值为负值时，那么该程序就会降低 PRI 值，亦即会变的较优先被处理。此外，必须留意：
* nice 值可调整的范围为 -20 ~ 19 ；
* root 可随意调整自己或他人程序的 Nice 值，且范围为 -20 ~ 19 ；
* 一般使用者仅可调整自己程序的 Nice 值，且范围仅为 0 ~ 19 (避免一般用户抢占系统资源)；
* 一般使用者仅可将 nice 值越调越高，例如本来 nice 为 5 ，则未来仅能调整到大于 5；

这也就是说，要调整某个程序的优先运行序，就是『调整该程序的 nice 值』。有两种方式，给予某个程序 nice 值，分别是：
* 一开始运行程序就立即给予一个特定的 nice 值：用 nice 命令；
* 调整某个已经存在的 PID 的 nice 值：用 renice 命令。
### nice：新运行的命令即给予新的 nice 值
```
[root@www ~]# nice [-n 数字] command
选项与参数：
-n  ：后面接一个数值，数值的范围 -20 ~ 19。

用 root 给一个 nice 值为 -5 ，用于运行 vi ，并观察该程序！
[root@www ~]# nice -n -5 vi &
[1] 18676
[root@www ~]# ps -l
F S   UID   PID  PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD
4 S     0 18625 18623  0  75   0 -  1514 wait   pts/1    00:00:00 bash
4 T     0 18676 18625  0  72  -5 -  1242 finish pts/1    00:00:00 vi
4 R     0 18678 18625  0  77   0 -  1101 -      pts/1    00:00:00 ps
# 原本的 bash PRI 为 75  ，所以 vi 默认应为 75。不过由于给予 nice  为 -5 ，
# 因此 vi 的 PRI 降低了！但并非降低到 70 ，因为核心还会动态调整！

[root@www ~]# kill -9 %1 <==测试完毕将 vi 关闭
```
系统背景工作中，某些比较不重要的程序，例如备份工作，由于备份工作相当消耗系统资源，这个时候就可以将备份的命令之 nice 值调大一些，可以使系统的资源分配更为公平。
### renice：已存在的 nice 重新调整
```
[root@www ~]# renice [number] PID
选项与参数：
PID ：某个程序的 ID
```
整个 nice 值是可以在父程序 --> 子程序之间传递的，除了 renice 之外 top 也可以调整 nice 的值。
## 系统资源的观察
### free：观察内存使用情况
```
[root@www ~]# free [-b|-k|-m|-g] [-t]
选项与参数：
-b  ：直接输入 free 时，显示的单位是 Kbytes，我们可以使用 b(bytes), m(Mbytes)
      k(Kbytes), 及 g(Gbytes) 来显示单位喔！
-t  ：在输出的最终结果，显示实体内存与 swap 的总量。
```
total 是总量， used 是已被使用的量， free 则是剩余可用的量。 后面的 shared/buffers/cached 则是在已被使用的量当中，用来作为缓冲及缓存的量。  
通常系统充分使用内存，为了系统的存取效能加速。不过，一般来说，swap 最好不要被使用，尤其 swap 最好不要被使用超过 20% 以上，如果超过，说明实体内存不足。
> Linux 系统为了要加速系统效能，所以会将最常使用到的或者是最近使用到的文件数据缓存 (cache) 下来， 这样未来系统要使用该文件时，就直接由内存中搜寻取出，而不需要重新读取硬盘
### uname：查阅系统与核心相关资讯
```
[root@www ~]# uname [-asrmpi]
选项与参数：
-a  ：所有系统相关的资讯，包括底下的数据都会被列出来；
-s  ：系统核心名称
-r  ：核心的版本
-m  ：本系统的硬件名称，例如 i686 或 x86_64 等；
-p  ：CPU 的类型，与 -m 类似，只是显示的是 CPU 的类型！
-i  ：硬件的平台 (ix86)
```
### uptime：观察系统启动时间与工作负载
显示出目前系统已经启动多久的时间，以及 1, 5, 15 分钟的平均负载。就是 top 最上面一行。
```
[root@www ~]# uptime
 15:39:13 up 8 days, 14:52,  1 user,  load average: 0.00, 0.00, 0.00
```
### netstat：追踪网络或插槽档
这个命令比较常被用在网络的监控方面。基本上， netstat 的输出分为两大部分，分别是网络与系统自己的程序相关性部分：
```
[root@www ~]# netstat -[atunlp]
选项与参数：
-a  ：将目前系统上所有的连线、监听、Socket 数据都列出来
-t  ：列出 tcp 网络封包的数据
-u  ：列出 udp 网络封包的数据
-n  ：不以程序的服务名称，以埠号 (port number) 来显示；
-l  ：列出目前正在网络监听 (listen) 的服务；
-p  ：列出该网络服务的程序 PID 

[root@www ~]# netstat
Active Internet connections (w/o servers) <==与网络较相关的部分
Proto Recv-Q Send-Q Local Address        Foreign Address      State
tcp        0    132 192.168.201.110:ssh  192.168.:vrtl-vmf-sa ESTABLISHED
Active UNIX domain sockets (w/o servers)  <==与本机的程序自己的相关性(非网络)
Proto RefCnt Flags       Type       State         I-Node Path
unix  20     [ ]         DGRAM                    9153   /dev/log
unix  3      [ ]         STREAM     CONNECTED     13317  /tmp/.X11-unix/X0
unix  3      [ ]         STREAM     CONNECTED     13233  /tmp/.X11-unix/X0
unix  3      [ ]         STREAM     CONNECTED     13208  /tmp/.font-unix/fs7100
```
在上面的结果当中，显示了两个部分，分别是网络的连线以及 linux 上面的 socket 程序相关性部分。网络连线部分：
* Proto ：网络的封包协议，主要分为 TCP 与 UDP 封包，相关数据请参考服务器篇；
* Recv-Q：非由使用者程序连结到此 socket 的复制的总 bytes 数；
* Send-Q：非由远程主机传送过来的 acknowledged 总 bytes 数；
* Local Address ：本地端的 IP:port 情况
* Foreign Address：远程主机的 IP:port 情况
* State ：连线状态，主要有创建(ESTABLISED)及监听(LISTEN)；

除了网络上的连线之外，Linux 系统上面的程序是可以接收不同程序所发送来的资讯，那就是 Linux 上头的插槽档 (socket file)。socket file 可以沟通两个程序之间的资讯，因此程序可以取得对方传送过来的数据。由于有 socket file，因此类似 X Window 这种需要透过网络连接的软件，目前新版的 distributions 就以 socket 来进行窗口介面的连线沟通了。上表中 socket file 的输出栏位有：
* Proto ：一般就是 unix
* RefCnt：连接到此 socket 的程序数量；
* Flags ：连线的旗标；
* Type ：socket 存取的类型。主要有确认连线的 STREAM 与不需确认的 DGRAM 两种；
* State ：若为 CONNECTED 表示多个程序之间已经连线创建。
* Path ：连接到此 socket 的相关程序的路径！或者是相关数据输出的路径。

利用 netstat 查看那些程序有启动那些网络的【后门】
```
找出目前系统上已在监听的网络连线及其 PID
[root@www ~]# netstat -tlnp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address    Foreign Address  State   PID/Program name
tcp        0      0 127.0.0.1:2208   0.0.0.0:*        LISTEN  4566/hpiod
tcp        0      0 0.0.0.0:111      0.0.0.0:*        LISTEN  4328/portmap
tcp        0      0 127.0.0.1:631    0.0.0.0:*        LISTEN  4597/cupsd
tcp        0      0 0.0.0.0:728      0.0.0.0:*        LISTEN  4362/rpc.statd
tcp        0      0 127.0.0.1:25     0.0.0.0:*        LISTEN  4629/sendmail: 
tcp        0      0 127.0.0.1:2207   0.0.0.0:*        LISTEN  4571/python
tcp        0      0 :::22            :::*             LISTEN  4586/sshd
# 除了可以列出监听网络的介面与状态之外，最后一个栏位还能够显示此服务的 PID 号码以及程序的命令名称。
```
不论主机提供什么样的服务， 一定必须要有相对应的 program 在主机上面运行。例如提供 WWW 服务，就是 Apache 提供的功能，要关闭 WWW 服务，关掉该程序即可。
### dmesg：分析核心产生的信息
所有核心侦测的信息，如侦测硬件，不管是启动时候还是系统运行过程中，反正只要是核心产生的信息，都会被记录到内存中的某个保护区段。 dmesg 这个命令就能够将该区段的信息读出来的，因为信息过多，所以运行时可以加入这个管线命令『 | more 』来使画面暂停。
```
输出所有的核心启动时的资讯
[root@www ~]# dmesg | more

搜寻启动的时候，硬盘的相关资讯
[root@www ~]# dmesg | grep -i hd
```
### vmstat：侦测系统资源变化
vmstat 可以侦测『 CPU / 内存 / 磁碟输入输出状态 』等等
```
[root@www ~]# vmstat [-a] [延迟 [总计侦测次数]] <==CPU/内存等资讯
[root@www ~]# vmstat [-fs]                      <==内存相关
[root@www ~]# vmstat [-S 单位]                  <==配置显示数据的单位
[root@www ~]# vmstat [-d]                       <==与磁碟有关
[root@www ~]# vmstat [-p 分割槽]                <==与磁碟有关
选项与参数：
-a  ：使用 inactive/active(活跃与否) 取代 buffer/cache 的内存输出资讯；
-f  ：启动到目前为止，系统复制 (fork) 的程序数；
-s  ：将一些事件 (启动至目前为止) 导致的内存变化情况列表说明；
-S  ：后面可以接单位，让显示的数据有单位。例如 K/M 取代 bytes 的容量；
-d  ：列出磁碟的读写总量统计表
-p  ：后面列出分割槽，可显示该分割槽的读写总量统计表

统计目前主机 CPU 状态，每秒一次，共计三次！
[root@www ~]# vmstat 1 3
procs -----------memory---------- ---swap-- -----io---- --system-- -----cpu------
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 0  0     28  61540 137000 291960    0    0     4     5   38   55  0  0 100  0  0
 0  0     28  61540 137000 291960    0    0     0     0 1004   50  0  0 100  0  0
 0  0     28  61540 137000 291964    0    0     0     0 1022   65  0  0 100  0  0
```
使用类似『 vmstat 5 』代表每五秒钟升级一次，直到你按下 [ctrl]-c 为止。各个栏位的意义：
* 内存栏位 (procs) 的项目分别为：  
  r ：等待运行中的程序数量；b：不可被唤醒的程序数量。这两个项目越多，代表系统越忙碌 (因为系统太忙，所以很多程序就无法被运行或一直在等待而无法被唤醒之故)。
* 内存栏位 (memory) 项目分别为：  
  swpd：虚拟内存被使用的容量； free：未被使用的内存容量； buff：用于缓冲内存； cache：用于高速缓存。 这部份则与 free 是相同的。
* 内存置换空间 (swap) 的项目分别为：  
  si：由磁碟中将程序取出的量； so：由于内存不足而将没用到的程序写入到磁碟的 swap 的容量。 如果 si/so 的数值太大，表示内存内的数据常常得在磁碟与主内存之间传来传去，系统效能会很差！
* 磁碟读写 (io) 的项目分别为：  
  bi：由磁碟写入的区块数量； bo：写入到磁碟去的区块数量。如果这部份的值越高，代表系统的 I/O 非常忙碌！
* 系统 (system) 的项目分别为：  
  in：每秒被中断的程序次数； cs：每秒钟进行的事件切换次数；这两个数值越大，代表系统与周边设备的沟通非常频繁！ 这些周边设备当然包括磁碟、网络卡、时间钟等。
* CPU 的项目分别为：  
  us：非核心层的 CPU 使用状态； sy：核心层所使用的 CPU 状态； id：闲置的状态； wa：等待 I/O 所耗费的 CPU 状态； st：被虚拟机器 (virtual machine) 所盗用的 CPU 使用状态 (2.6.11 以后才支持)。
```
系统上面所有的磁碟的读写状态
[root@www ~]# vmstat -d
disk- ------------reads------------ ------------writes----------- -----IO------
       total merged sectors      ms  total merged sectors      ms    cur    sec
ram0       0      0       0       0      0      0       0       0      0      0
....(中间省略)....
hda   144188 182874 6667154 7916979 151341 510244 8027088 15244705      0    848
hdb        0      0       0       0      0      0       0       0      0      0
```

# 特殊文件与程序
## 具有 SUID/SGID 权限的命令运行状态
SUID 的权限与程序的相关性非常大。SUID 被一般使用者运行：
* SUID 权限仅对二进位程序(binary program)有效；
* 运行者对于该程序需要具有 x 的可运行权限；
* 本权限仅在运行该程序的过程中有效 (run-time)；
* 运行者将具有该程序拥有者 (owner) 的权限。

整个 SUID 的权限会生效是由于『具有该权限的程序被触发』，一个程序被触发会变成程序，运行者可以具有程序拥有者的权限就是在该程序变成程序时。比如运行 passwd 具有 root 权限，是因为在触发 passwd 后，会取得一个新的程序与 PID，该 PID 产生时透过 SUID 来给予该 PID 特殊的权限配置。
```
[dmtsai@www ~]$ passwd
Changing password for user dmtsai.
Changing password for dmtsai
(current) UNIX password: <==这里按下 [ctrl]-z 并且按下 [enter]
[1]+  Stopped                 passwd

[dmtsai@www ~]$ pstree -u
init-+-acpid
....(中间省略)....
     |-sshd---sshd---sshd(dmtsai)---bash-+-more
     |                                   |-passwd(root)
     |                                   `-pstree
```
使用 `find / -perm +6000` 来查询具有 SUID/SGID 权限的文件。
## /proc/* 代表的意义
触发的程序都是在内存中，而内存中的数据都是写入到 /proc/* 这个而目录下。基本上，目前主机上面的各个程序的 PID 都是以目录的型态存在于 /proc 当中。举例来说，我们启动所运行的第一支程序 init 他的 PID 是 1 ， 这个 PID 的所有相关资讯都写入在 /proc/1/* 当中。若直接观察 PID 为 1 的数据：
```
[root@www ~]# ll /proc/1
dr-xr-xr-x 2 root root 0 Mar 12 11:04 attr
-r-------- 1 root root 0 Mar 17 14:32 auxv
-r--r--r-- 1 root root 0 Mar 17 14:32 cmdline  <==就是命令串
-rw-r--r-- 1 root root 0 Mar 17 14:32 coredump_filter
-r--r--r-- 1 root root 0 Mar 17 14:32 cpuset
lrwxrwxrwx 1 root root 0 Mar 17 14:32 cwd -> /
-r-------- 1 root root 0 Mar 17 14:32 environ  <==一些环境变量
lrwxrwxrwx 1 root root 0 Mar 17 14:32 exe -> /sbin/init  <==实际运行的命令
....(以下省略)....
```
* cmdline：这个程序被启动的命令串；
* environ：这个程序的环境变量内容。
```
[root@www ~]# cat /proc/1/cmdline
init [5]
```
/proc 底下相关的文件与对应的内容：
档名|文件内容
:---:|:---:
/proc/cmdline|加载 kernel 时所下达的相关参数！查阅此文件，可了解系统是如何启动的！
/proc/cpuinfo|本机的 CPU 的相关资讯，包含时脉、类型与运算功能等
/proc/devices|这个文件记录了系统各个主要装置的主要装置代号，与 mknod 有关呢！
/proc/filesystems|目前系统已经加载的文件系统
/proc/interrupts|目前系统上面的 IRQ 分配状态。
/proc/ioports|目前系统上面各个装置所配置的 I/O 位址。
/proc/kcore|这个就是内存的大小，不要读他
/proc/loadavg|还记得 top 以及 uptime 的三个平均数值就是记录在此！
/proc/meminfo|使用 free 列出的内存资讯，在这里也能够查阅到
/proc/modules|目前我们的 Linux 已经加载的模块列表，也可以想成是驱动程序
/proc/mounts|系统已经挂载的数据，就是用 mount 这个命令呼叫出来的数据
/proc/swaps|到底系统挂加载的内存在哪里？使用掉的 partition 就记录在此
/proc/partitions|使用 fdisk -l 会出现目前所有的 partition ，在这个文件当中也有纪录
/proc/pci|在 PCI 汇流排上面，每个装置的详细情况！可用 lspci 来查阅！
/proc/uptime|就是用 uptime 的时候，会出现的资讯
/proc/version|核心的版本，就是用 uname -a 显示的内容
/proc/bus/*|一些汇流排的装置，还有 U盘 的装置也记录在此
## 查询已开启文件或已运行程序开启之文件
### fuser：藉由文件 (或文件系统) 找出正在使用该文件的程序
```
[root@www ~]# fuser [-umv] [-k [i] [-signal]] file/dir
选项与参数：
-u  ：除了程序的 PID 之外，同时列出该程序的拥有者；
-m  ：后面接的那个档名会主动的上提到该文件系统的最顶层，对 umount 不成功很有效！
-v  ：可以列出每个文件与程序还有命令的完整相关性！
-k  ：找出使用该文件/目录的 PID ，并试图以 SIGKILL 这个讯号给予该 PID；
-i  ：必须与 -k 配合，在删除 PID 之前会先询问使用者意愿！
-signal：例如 -1 -15 等等，若不加的话，默认是 SIGKILL (-9)

找出目前所在目录的使用 PID/所属帐号/权限
[root@www ~]# fuser -uv .
                     USER        PID ACCESS COMMAND
.:                   root      20639 ..c.. (root)bash
```
ACCESS 代表的意义为：
* c ：此程序在当前的目录下(非次目录)；
* e ：可被触发为运行状态；
* f ：是一个被开启的文件；
* r ：代表顶层目录 (root directory)；
* F ：该文件被开启了，不过在等待回应中；
* m ：可能为分享的动态函式库；

如果你想要查阅某个文件系统底下有多少程序正在占用该文件系统时，要使用 -m 的选项。
```
[root@www ~]# fuser -uv /proc
# 不会显示任何数据，因为没有任何程序会去使用 /proc 这个目录会被用到的是 /proc 底下的文件

[root@www ~]# fuser -mvu /proc
                     USER        PID ACCESS COMMAND
/proc:               root       4289 f.... (root)klogd
                     root       4555 f.... (root)acpid
                     haldaemon  4758 f.... (haldaemon)hald
                     root       4977 F.... (root)Xorg
```
```
# 针对单一文件
[root@www ~]# fuser -uv /var/gdm/.gdmfifo
                     USER        PID ACCESS COMMAND
/var/gdm/.gdmfifo:   root       4892 F.... (root)gdm-binary

试图删除该 PID ，且不要删除
[root@www ~]# fuser -ki /var/gdm/.gdmfifo
/var/gdm/.gdmfifo:    4892
Kill process 4892 ? (y/N) n
```
### lsof：列出被程序所开启的文件档名
```
[root@www ~]# lsof [-aUu] [+d]
选项与参数：
-a  ：多项数据需要『同时成立』才显示出结果时！
-U  ：仅列出 Unix like 系统的 socket 文件类型；
-u  ：后面接 username，列出该使用者相关程序所开启的文件；
+d  ：后面接目录，亦即找出某个目录底下已经被开启的文件！

列出目前系统上面所有已经被开启的文件与装置：
[root@www ~]# lsof
COMMAND PID  USER   FD  TYPE  DEVICE   SIZE     NODE NAME
init      1  root  cwd   DIR     3,2   4096        2 /
init      1  root  rtd   DIR     3,2   4096        2 /
init      1  root  txt   REG     3,2  38620  1426405 /sbin/init
# 默认情况下，lsof 会将目前系统上面已经开启的文件全部列出来。

# lsof -u root -a -U 与 lsof -u root -U 不同，-a 表示两个项目同时成立。

[root@www ~]# lsof -u root | grep bash
```
### pidof：找出某支正在运行的程序的 PID
```
[root@www ~]# pidof [-sx] program_name
选项与参数：
-s  ：仅列出一个 PID 而不列出所有的 PID
-x  ：同时列出该 program name 可能的 PPID 那个程序的 PID

列出目前系统上面 init 以及 syslogd 这两个程序的 PID
[root@www ~]# pidof init syslogd
1 4286
```

# SELinux 初探
## 什么是 SELinux
Security Enhanced Linux，安全强化 Linux。
### 设计目标：避免资源的误用
系统出现问题的原因大部分都在于『内部员工的资源误用』所导致的，实际由外部发动的攻击反而没有这么严重。  
这也就是说：其实 SELinux 是在进行程序、文件等细部权限配置依据的一个核心模块，由于启动网络服务的也是程序，所以也能够控制网络服务能否存取系统资源。
### 传统的文件权限与账号关系：自主式存取控制, DAC
当某个程序想要对文件进行存取时， 系统就会根据该程序的拥有者/群组，并比对文件的权限，若通过权限检查，就可以存取该文件了。  
这种存取文件系统的方式被称为『自主式存取控制 (Discretionary Access Control, DAC)』，基本上，就是依据程序的拥有者与文件资源的 rwx 权限来决定有无存取的能力。其问题表现在：
* root 具有最高的权限
* 使用者可以取得程序来变更文件资源的存取权限
### 以政策守则订定特定程序读取特定文件：委任式存取控制, MAC
SELinux 导入了委任式存取控制 (Mandatory Access Control, MAC)。  
针对特定的程序与特定的文件资源来进行权限的控管，即使 root 用户，使用不同的程序时，取得的权限并不一定是 root ，而要看程序的配置而定。如此一来，我们针对控制的【主体】变成了【程序】而不是使用者。此外，这个主体程序也不能任意使用系统文件资源，因为每个文件资源也有针对该主体程序配置可取用的权限。SELinux 提供一些默认的政策 (Policy) ，并在该政策内提供多个守则 (rule) ，让你可以选择是否激活该控制守则。  
在委任式存取控制的配置下，程序能够活动的空间就变小了。
## SELinux 的运行方式
SELinux 是透过 MAC 的方式来控管程序，他控制的主体是程序， 而目标则是该程序能否读取的『文件资源』。
* 主体 (Subject)：  
  程序，process
* 目标 (Object)：  
  主体程序能否存取的『目标资源』一般就是文件系统。因此这个目标项目可以等文件系统划上等号；
* 政策 (Policy)：  
  由于程序与文件数量庞大，因此 SELinux 会依据某些服务来制订基本的存取安全性政策。这些政策内还会有详细的守则 (rule) 来指定不同的服务开放某些资源的存取与否。在目前的 CentOS 5.x 里面仅有提供两个主要的政策，分别是：
    * targeted：针对网络服务限制较多，针对本机限制较少，是默认的政策；
    * strict：完整的 SELinux 限制，限制方面较为严格；

  建议使用默认的 targeted 政策即可。
* 安全性本文 (security context)：  
  主体能不能存取目标除了政策指定之外，主体与目标的安全性本文必须一致才能够顺利存取。安全性本文 (security context) 类似文件系统的 rwx 。如果配置错误，某些服务(主体程序)就无法存取文件系统(目标资源)，一直提示【权限不符】的错误信息。  
  最终能否存取目标还与文件系统的 rwx 权限配置有关。
### 安全性本文 (Security Context)
CentOS 5.x 已经帮我们制订好非常多的守则了，这部份你只要知道如何开启/关闭某项守则的放行与否即可。  
安全性本文是放置到文件的 inode 内的，因此主体程序想要读取目标文件资源时，同样需要读取 inode ， 这 inode 内就可以比对安全性本文以及 rwx 等权限值是否正确，而给予适当的读取权限依据。  
查看安全性文本：
```
 [root@www ~]# ls -Z
drwxr-xr-x  root root root:object_r:user_home_t   Desktop
-rw-r--r--  root root root:object_r:user_home_t   install.log
-rw-r--r--  root root root:object_r:user_home_t   install.log.syslog

# 安全性本文主要用冒号分为三个栏位：
# Identify:role:type
# 身份识别:角色:类型
```
* 身份识别 (Identify)：  
    * root：表示 root 的帐号身份
    * system_u：表示系统程序方面的识别，通常就是程序
    * user_u：代表的是一般使用者帐号相关的身份。

  身份识别中，除了 root 之外，其他的识别后面都会加上『 _u 』的字样。身份识别重点在让我们了解该数据为何种身份所有。系统上大部分的数据都会是 system_u 或 root ，如果是 /home 底下的数据，大部分应该会是 user_u
* 角色 (Role)：  
  透过角色栏位，我们可以知道这个数据是属于程序、文件资源还是代表使用者。一般的角色有：
    * object_r：代表的是文件或目录等文件资源，这应该是最常见的
    * system_r：代表的就是程序，一般使用者也会别指定为 system_r

  『 _r 』，代表 role
* 类型 (Type)：  
  在默认的 targeted 政策中， Identify 与 Role 栏位基本上是不重要的，重要的在于类型 (type) 栏位。基本上，一个主体程序能不能读取到这个文件资源，与类型栏位有关。而类型栏位在文件与程序的定义不太相同，分别是：
    * type：在文件资源 (Object) 上面称为类型 (Type)；
    * domain：在主体程序 (Subject) 则称为领域 (domain)。

  domain 需要与 type 搭配，则该程序才能够顺利的读取文件资源。
### 程序与文件 SELinux type 栏位的相关性
基本上，这些对应数据在 targeted 政策下的对应如下：
身份识别|角色|该对应在 targeted 的意义
:---:|:---:|:---:
root|system_r|代表供 root 帐号登陆时所取得的权限
system_u|system_r|由于为系统帐号，因此是非交谈式的系统运行程序
user_u|system_r|一般可登陆使用者的程序
最重要的栏位是类型栏位，主体与目标之间是否具有可以读写的权限，与程序的 domain 及文件的 type 有关。以达成 WWW 服务器功能的 httpd 这支程序与 /var/www/html 这个网页放置的目录来说明。 
```
[root@www ~]# ll -Zd /usr/sbin/httpd /var/www/html
-rwxr-xr-x  root root system_u:object_r:httpd_exec_t   /usr/sbin/httpd
drwxr-xr-x  root root system_u:object_r:httpd_sys_content_t /var/www/html
# 两者的角色栏位都是 object_r ，代表都是文件！而 httpd 属于 httpd_exec_t 类型，
# /var/www/html 则属于 httpd_sys_content_t 这个类型！
```
httpd 属于 httpd_exec_t 这个可以运行的类型，而 /var/www/html 则属于 httpd_sys_content_t 这个可以让 httpd 领域 (domain) 读取的类型。
1. 首先，我们触发一个可运行的目标文件，那就是具有 httpd_exec_t 这个类型的 /usr/sbin/httpd 文件；
2. 该文件的类型会让这个文件所造成的主体程序 (Subject) 具有 httpd 这个领域 (domain)， 我们的政策针对这个领域已经制定了许多守则，其中包括这个领域可以读取的目标资源类型；
3. 由于 httpd domain 被配置为可以读取 httpd_sys_content_t 这个类型的目标文件 (Object)， 因此你的网页放置到 /var/www/html/ 目录下，就能够被 httpd 那支程序所读取了；
4. 但最终能不能读到正确的数据，还得要看 rwx 是否符合 Linux 权限的规范！

## SELinux 的启动、关闭与观察
并非所有的 Linux distributions 都支持 SELinux 。目前 SELinux 支持三种模式，分别是：
* enforcing：强制模式，代表 SELinux 运行中，且已经正确的开始限制 domain/type 了；
* permissive：宽容模式：代表 SELinux 运行中，不过仅会有警告信息并不会实际限制 domain/type 的存取。这种模式可以运来作为 SELinux 的 debug 之用；
* disabled：关闭，SELinux 并没有实际运行。

透过 getenforce 观察目前 SELinux 模式。
```
[root@www ~]# getenforce
Enforcing
```
使用 sestatus 观察 SELinux 的政策 (Policy)
```
[root@www ~]# sestatus [-vb]
选项与参数：
-v  ：检查列于 /etc/sestatus.conf 内的文件与程序的安全性本文内容；
-b  ：将目前政策的守则布林值列出，亦即某些守则 (rule) 是否要启动 (0/1) 之意；

列出目前的 SELinux 使用哪个政策 (Policy)？
[root@www ~]# sestatus
SELinux status:                 enabled    <==是否启动 SELinux
SELinuxfs mount:                /selinux   <==SELinux 的相关文件数据挂载点
Current mode:                   enforcing  <==目前的模式
Mode from config file:          enforcing  <==配置档指定的模式
Policy version:                 21
Policy from config file:        targeted   <==目前的政策
```
SELinux 的配置档是 /etc/selinux/config
```
[root@www ~]# vi /etc/selinux/config
SELINUX=enforcing     <==调整 enforcing|disabled|permissive
SELINUXTYPE=targeted  <==目前仅有 targeted 与 strict
```
### SELinux 的启动与关闭
如果改变了政策则需要重新启动；如果由 enforcing 或 permissive 改成 disabled ，或由 disabled 改成其他两个，那也必须要重新启动。这是因为 SELinux 是整合到核心里面去的， 你只可以在 SELinux 运行下切换成为强制 (enforcing) 或宽容 (permissive) 模式，不能够直接关闭 SELinux 。同时，由 SELinux 关闭 (disable) 的状态到开启的状态也需要重新启动。  
在 /boot/grub/menu.lst 查看核心是否关闭 SELinux
```
[root@www ~]# vi /boot/grub/menu.lst
default=0
timeout=5
splashimage=(hd0,0)/grub/splash.xpm.gz
hiddenmenu
title CentOS (2.6.18-92.el5)
        root (hd0,0)
        kernel /vmlinuz-2.6.18-92.el5 ro root=LABEL=/1 rhgb quiet selinux=0
        initrd /initrd-2.6.18-92.el5.img
# 如果要启动 SELinux ，则不可以出现 selinux=0 的字样在 kernel 后面
```
selinux=0 指定给核心时， 则核心会自动的忽略 /etc/selinux/config 的配置值，而直接略过 SELinux 的加载。如果从 disable 转到启动 SELinux 的模式时， 由于系统必须要针对文件写入安全性本文的资讯，因此启动过程会花费不少时间在等待重新写入 SELinux 安全性本文 (有时也称为 SELinux Label) ，而且在写完之后还得要再次的重新启动一次。  
让 SELinux 模式在 enforcing 与 permissive 之间切换的方法为：
```
[root@www ~]# setenforce [0|1]
选项与参数：
0 ：转成 permissive 宽容模式；
1 ：转成 Enforcing 强制模式

# setenforce 无法在 Disable 的模式下进行模式切换
```
## SELinux 网络服务运行范例
使用 WWW 服务器来说明一下 SELinux 的运行方式。
### 网络服务的启动与观察
启动 httpd 这支服务，一般服务启动的脚本会在 /etc/init.d/ 底下：
```
[root@www ~]# /etc/init.d/httpd start
正在启动 httpd:          [  确定  ]

观察有无此程序，并且观察此程序的 SELinux 安全性本文数据
[root@www ~]# pstree | grep httpd
     |-httpd---8*[httpd]   <==httpd 会产生很多子程序来负责网络服务喔！

[root@www ~]# ps aux -Z |grep http
root:system_r:httpd_t root   24089 0.2 1.2 22896 9256 ? Ss 16:06 0:00 /usr/sbin/httpd
root:system_r:httpd_t apache 24092 0.0 0.6 22896 4752 ? S  16:06 0:00 /usr/sbin/httpd
root:system_r:httpd_t apache 24093 0.0 0.6 22896 4752 ? S  16:06 0:00 /usr/sbin/httpd
```
ps -Z 这个『 -Z 』的选项可以让我们查阅程序的安全性本文。
### 错误的 SELinux 安全性本文
导致权限错误，不能读取。
### 重设 SELinux 安全性本文
```
[root@www ~]# chcon [-R] [-t type] [-u user] [-r role] 文件
[root@www ~]# chcon [-R] --reference=范例档 文件
选项与参数：
-R  ：连同该目录下的次目录也同时修改；
-t  ：后面接安全性本文的类型栏位！例如 httpd_sys_content_t ；
-u  ：后面接身份识别，例如 system_u；
-r  ：后面街角色，例如 system_r；
--reference=范例档：拿某个文件当范例来修改后续接的文件的类型！

[root@www ~]# chcon -t httpd_sys_content_t /var/www/html/index.html

[root@www ~]# chcon --reference=/etc/passwd /var/www/html/index.html
```
chcon 是透过直接指定的方式来处理安全性本文的类型数据。那我们知道其实系统默认的目录都有特殊的 SELinux 安全性本文， 举例来说， /var/www/html 原本就是 httpd 可以读取的目录。可以使用 `restorecon` 还原默认的安全性本文。
```
[root@www ~]# restorecon [-Rv] 文件或目录
选项与参数：
-R  ：连同次目录一起修改；
-v  ：将过程显示到萤幕上

[root@www ~]# restorecon -Rv /var/www/html/index.html
```
类型 (type) 配置错误的原因很可能是因为该文件由其他位置复制或移动过来所导致的。
## SELinux 所需的服务
### setroubleshoot --> 错误信息写入 /var/log/messages
几乎所有 SELinux 相关的程序都会以 se 为开头。troubleshoot 就是错误克服。这个服务会将关于 SELinux 的错误信息与克服方法记录到 /var/log/messages 里头，所以必须要启动。
```
[root@www ~]# chkconfig --list setroubleshoot
setroubleshoot  0:off  1:off  2:off 3:on  4:on  5:on  6:off
# 我们的 Linux 运行模式是在 3 或 5 号，因此这两个要 on 即可。

[root@www ~]# chkconfig setroubleshoot on
# 关于 chkconfig 我们会在后面章节介绍， --list 是列出目前的运行等级是否有启动，
# 如果加上 on ，则是在启动时启动，若为 off 则启动时不启动。
```
错误信息：
```
[root@www ~]# cat /var/log/messages | grep setroubleshoot
Mar 23 17:18:44 www setroubleshoot: SELinux is preventing the httpd from using 
potentially mislabeled files (/var/www/html/index.html). For complete SELinux 
messages. run sealert -l 6c028f77-ddb6-4515-91f4-4e3e719994d4


[root@www ~]# sealert -l 6c028f77-ddb6-4515-91f4-4e3e719994d4
Summary:

SELinux is preventing the httpd from using potentially mislabeled files
(/var/www/html/index.html). <==就是刚刚 /var/log/messages 的信息

Detailed Description:       <==底下是更完整的描述！要看！

SELinux has denied httpd access to potentially mislabeled file(s)
(/var/www/html/index.html). This means that SELinux will not allow httpd to use
these files. It is common for users to edit files in their home directory or tmp
directories and then move (mv) them to system directories. The problem is that
the files end up with the wrong file context which confined applications are not
allowed to access.

Allowing Access:            <==若要允许存取，你需要进行的动作！

If you want httpd to access this files, you need to relabel them using
restorecon -v '/var/www/html/index.html'. You might want to relabel the entire
directory using restorecon -R -v '/var/www/html'.
```
### auditd --> 详细数据写入 /var/log/audit/audit.log
audit 是稽核的意思，这个 auditd 会将 SELinux 发生的错误资讯写入 /var/log/audit/audit.log 中！ 与上个服务相同的，最好在启动时就配置这服务为启动的模式
```
[root@www ~]# chkconfig --list auditd
auditd      0:off  1:off  2:on   3:on   4:on   5:on   6:off

[root@www ~]# chkconfig auditd on
# 若 3:off 及 5:off 时，才需要进行！
```
与 setroubleshoot 不同的是， auditd 会将许多的 SELinux 资讯都记录下来，不只是错误信息而已， 因此登录档 /var/log/audit/audit.log 非常的庞大。SELinux 有提供一个 audit2why 的命令来让我们查询错误信息。
```
[root@www ~]# audit2why < /var/log/audit/audit.log
# 意思是，将登录档的内容读进来分析，并输出分析的结果！结果有点像这样：
type=AVC msg=audit(1237799959.349:355): avc:  denied  { getattr } for  pid=24094 
comm="httpd" path="/var/www/html/index.html" dev=hda2 ino=654685 scontext=root:s
ystem_r:httpd_t:s0 tcontext=root:object_r:user_home_t:s0 tclass=file
    Was caused by:
       Missing or disabled TE allow rule.
       Allow rules may exist but be disabled by boolean settings; check boolean
settings.
       You can see the necessary allow rules by running audit2allow with this
audit message as input.
```
AVC 是 access vector cache 的缩写， 目的是记录所有与 SELinux 有关的存取统计数据。
## SELinux 的政策与守则管理
一个主体程序能否读取到目标文件资源的重点在于 SELinux 的政策以及政策内的各项守则， 然后再透过该守则的定义去处理各目标文件的安全性本文，尤其是『类型』的部分。
### 政策查阅
可以透过 seinfo 查询
```
[root@www ~]# seinfo [-Atrub]
选项与参数：
-A  ：列出 SELinux 的状态、守则布林值、身份识别、角色、类别等所有资讯
-t  ：列出 SELinux 的所有类别 (type) 种类
-r  ：列出 SELinux 的所有角色 (role) 种类
-u  ：列出 SELinux 的所有身份识别 (user) 种类
-b  ：列出所有守则的种类 (布林值)

# 想要找到有 httpd 字样的安全性本文类别时， 可以使用『 seinfo -t | grep httpd 』来查询
```
如果查询到相关的类别或者是布林值后，想要知道详细的守则时， 就得要使用 sesearch 这个命令。
```
[root@www ~]# sesearch [-a] [-s 主体类别] [-t 目标类别] [-b 布林值]
选项与参数：
-a  ：列出该类别或布林值的所有相关资讯
-t  ：后面还要接类别，例如 -t httpd_t
-b  ：后面还要接布林值的守则，例如 -b httpd_enable_ftp_server

找出目标文件资源类别为 httpd_sys_content_t 的有关资讯
[root@www ~]# sesearch -a -t httpd_sys_content_t
Found 74 av rules:
   allow readahead_t httpd_sys_content_t : file { ioctl read getattr lock };
   allow readahead_t httpd_sys_content_t : dir { ioctl read getattr lock search };
....(底下省略)....
# 『 allow  主体程序安全性本文类别  目标文件安全性本文类别 』
# 如上，说明这个类别可以被那个主题程序的类别所读取，以及目标文件资源的格式。

找出主体程序为 httpd_t 且目标文件类别为 httpd 相关的所有资讯
[root@www ~]# sesearch -s httpd_t -t httpd_* -a
Found 163 av rules:
....(中间省略)....
   allow httpd_t httpd_sys_content_t : file { ioctl read getattr lock };
   allow httpd_t httpd_sys_content_t : dir { ioctl read getattr lock search };
   allow httpd_t httpd_sys_content_t : lnk_file { ioctl read getattr lock };
....(后面省略)....
# 从上面的数据就可以看出当程序为 httpd_t 这个类别，是可以读取 
# httpd_sys_content_t 的！
```
实际规范这些守则的，就是布林值的项目，也就是守则。主体程序能否对某些目标文件进行存取，与这个布林值非常有关系。布林值可以将守则配置为启动 (1) 或者是关闭 (0)。  
实际的政策数据都是放置到 /etc/selinux/targeted/policy/ 底下， 事实上，所有与 targetd 相关的资讯都是放置到 /etc/selinux/targeted 里面，包括安全性本文相关的资讯。
### 布林值的查询与修改
Subject 与 Object 能否有存取的权限，是与布林值有关的， 系统有多少布林值可以透过 seinfo -b 来查询。查询每个布林值是启动或关闭。
```
[root@www ~]# getsebool [-a] [布林值条款]
选项与参数：
-a  ：列出目前系统上面的所有布林值条款配置为开启或关闭值
```
```
[root@www ~]# setsebool [-P] 布林值=[0|1]
选项与参数：
-P  ：直接将配置值写入配置档，该配置数据未来会生效的！
```
setsebool 最好加上 -P 的选项，这样才能将此配置写入配置档。
### 默认目录的安全性本文查询与修改
每个目录或文件都会有默认的安全性本文，会制订目录的安全性本文，是因为系统的一些服务所放置文件的目录已经是确定的，当然有默认的安全性本文管理上较方便。查询这些目录的默认安全性本文，使用 `semanage`。
```
[root@www ~]# semanage {login|user|port|interface|fcontext|translation} -l
[root@www ~]# semanage fcontext -{a|d|m} [-frst] file_spec
选项与参数：
fcontext ：主要用在安全性本文方面的用途， -l 为查询的意思；
-a ：添加的意思，你可以添加一些目录的默认安全性本文类型配置；
-m ：修改的意思；
-d ：删除的意思。

查询一下 /var/www/html 的默认安全性本文配置为何！
[root@www ~]# semanage fcontext -l
SELinux fcontext    type          Context
....(前面省略)....
/var/www(/.*)?      all files     system_u:object_r:httpd_sys_content_t:s0
```
目录的配置可以使用正规表示法去指定一个范围。
```
[root@www ~]# semanage fcontext -l | grep '/srv'
/srv/.*                     all files   system_u:object_r:var_t:s0
/srv/([^/]*/)?ftp(/.*)?     all files   system_u:object_r:public_content_t:s0
/srv/([^/]*/)?www(/.*)?     all files   system_u:object_r:httpd_sys_content_t:s0
/srv/([^/]*/)?rsync(/.*)?   all files   system_u:object_r:public_content_t:s0
/srv/gallery2(/.*)?         all files   system_u:object_r:httpd_sys_content_t:s0
/srv                        directory   system_u:object_r:var_t:s0 <==看这里！
# 上面则是默认的 /srv 底下的安全性本文数据，没有指定到 /srv/samba

[root@www ~]# semanage fcontext -a -t public_content_t "/srv/samba(/.*)?"
[root@www ~]# semanage fcontext -l | grep '/srv/samba'
/srv/samba(/.*)?            all files   system_u:object_r:public_content_t:s0

[root@www ~]# cat /etc/selinux/targeted/contexts/files/file_contexts.local
# This file is auto-generated by libsemanage
# Please use the semanage command to make changes
/srv/samba(/.*)?    system_u:object_r:public_content_t:s0
# 所谓默认值，是写入到这个文件

[root@www ~]# restorecon -Rv /srv/samba* <==尝试恢复默认值
[root@www ~]# ll -Zd /srv/samba
drwxr-xr-x  root root system_u:object_r:public_content_t /srv/samba/
# 有默认值后，使用用 restorecon 来修改。
```