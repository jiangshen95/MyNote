# 什么是例行性工作排程
Linux 可以设置例行性工作排程，提醒我们一些任务。Linux 的例行性工作排程，就是通过 crontab 与 at 实现的。
## Linux 工作排程的种类：at, cron
两种工作排程方式：
* 一种是例行性的，就是每隔一定的周期要来办的事项；
* 一种是突发性的
Linux 下使用 at 与 crontab 来达成这两个功能。
* at：at 是个可以处理仅运行一次就结束排程的命令，不过要运行 at 时， 必须要有 atd 这个服务 (第十八章) 的支持才行。在某些新版的 distributions 中，atd 可能默认并没有启动，那么 at 这个命令就会失效。
* crontab：crontab 这个命令所配置的工作将会循环的一直进行下去！ 可循环的时间为分钟、小时、每周、每月或每年等。crontab 除了可以使用命令运行外，亦可编辑 /etc/crontab 来支持。 至于让 crontab 可以生效的服务则是 crond 这个服务。
## Linux 上常见的例行性工作
Linux 会主动的帮我们进行一些工作呢！ 比方说自动的进行线上升级 (on-line update)、自动的进行 updatedb (第七章谈到的 locate 命令) 升级档名数据库、自动的作登录档分析 (所以 root 常常会收到标题为 logwatch 的信件) 等等。这是由于系统要正常运行的话， 某些在背景底下的工作必须要定时进行的缘故。基本上 Linux 系统常见的例行性任务有：
* 进行登录档的轮替 (log rotate)：  
  Linux 会主动的将系统所发生的各种资讯都记录下来，这就是登录档 (第十九章)。 由于系统会一直记录登录资讯，所以登录档将会越来越大。大型文件不但占容量还会造成读写效能的困扰， 因此适时的将登录档数据挪一挪，让旧的数据与新的数据分别存放，则比较可以有效的记录登录资讯。这就是 log rotate 的任务！这也是系统必要的例行任务；
* 登录档分析 logwatch 的任务：  
  如果系统发生了软件问题、硬件错误、资安问题等，绝大部分的错误资讯都会被记录到登录档中， 因此系统管理员的重要任务之一就是分析登录档。但你不可能手动透过 vim 等软件去检视登录档，因为数据太复杂了！ 我们的 CentOS 提供了一只程序『 logwatch 』来主动分析登录资讯，所以你会发现，你的 root 老是会收到标题为 logwatch 的信件，那是正常的
* 创建 locate 的数据库：  
  locate 命令是透过已经存在的档名数据库来进行系统上档名的查询。我们的档名数据库是放置到 /var/lib/mlocate/ 中。系统会主动的进行 updatedb 。
* whatis 数据库的创建：  
  与 locate 数据库类似的，whatis 也是个数据库，这个 whatis 是与 man page 有关的一个查询命令，不过要使用 whatis 命令时， 必须要拥有 whatis 数据库，而这个数据库也是透过系统的例行性工作排程来自动运行。
* RPM 软件登录档的创建：  
  RPM (第二十三章) 是一种软件管理的机制。由于系统可能会常常变更软件， 包括软件的新安装、非经常性升级等，都会造成软件档名的差异。为了方便未来追踪，系统也帮我们将档名作排序的记录，有时候系统也会透过排程来帮忙 RPM 数据库的重新建置
* 移除缓存档  
  某些软件在运行中会产生一些缓存档，但是当这个软件关闭时，这些缓存档可能并不会主动的被移除。 有些缓存档则有时间性，如果超过一段时间后，这个缓存档就没有效用了，此时移除这些缓存档就是一件重要的工作。系统透过例行性工作排程运行名为 tmpwatch 的命令来删除这些缓存档。
* 与网络服务有关的分析行为：  
  如果你有安装类似 WWW 服务器软件 (一个名为 apache 的软件)，那么你的 Linux 系统通常就会主动的分析该软件的登录档。 同时某些凭证与认证的网络资讯是否过期的问题，我们的 Linux 系统也会很亲和的帮你进行自动检查！

# 仅运行一次的工作排程
## atd 的启动与 at 运行的方式
激活 atd 服务
```
[root@www ~]# /etc/init.d/atd restart
正在停止 atd:                          [  确定  ]
正在启动 atd:                          [  确定  ]

# 再配置一下启动时就启动这个服务，免得每次重新启动都得再来一次！
[root@www ~]# chkconfig atd on
```
### at 的运行方式
我们使用 at 这个命令来产生所要运行的工作，并将这个工作以文字档的方式写入 /var/spool/at/ 目录内，该工作便能等待 atd 这个服务的取用与运行了。  
并不是所有的人都可以进行 at 工作排程，考虑安全问题，认可的账号才能使用 at 。
利用 /etc/at.allow 与 /etc/at.deny 这两个文件来进行 at 的使用限制。加上这两个文件后，at 的工作情况如下：
1. 先找寻 /etc/at.allow 这个文件，写在这个文件中的使用者才能使用 at ，没有在这个文件中的使用者则不能使用 at (即使没有写在 at.deny 当中)；
2. 如果 /etc/at.allow 不存在，就寻找 /etc/at.deny 这个文件，若写在这个 at.deny 的使用者则不能使用 at ，而没有在这个 at.deny 文件中的使用者，就可以使用 at
3. 如果两个文件都不存在，那么只有 root 可以使用 at 这个命令。

/etc/at.allow 是管理较为严格的方式，而 /etc/at.deny 则较为松散。在一般的 distributions 当中，由于假设系统上的所有用户都是可信任的， 因此系统通常会保留一个空的 /etc/at.deny 文件，意思是允许所有人使用 at 命令的意思。如果不希望某些使用者使用 at 的话，将那个使用者的账号写入 /etc/at.deny 即可。
## 实际运行单一工作排程
```
[root@www ~]# at [-mldv] TIME
[root@www ~]# at -c 工作号码
选项与参数：
-m  ：当 at 的工作完成后，即使没有输出信息，亦以 email 通知使用者该工作已完成。
-l  ：at -l 相当于 atq，列出目前系统上面的所有该使用者的 at 排程；
-d  ：at -d 相当于 atrm ，可以取消一个在 at 排程中的工作；
-v  ：可以使用较明显的时间格式列出 at 排程中的工作列表；
-c  ：可以列出后面接的该项工作的实际命令内容。

TIME：时间格式，这里可以定义出『什么时候要进行 at 这项工作』的时间，格式有：
  HH:MM				ex> 04:00
	在今日的 HH:MM 时刻进行，若该时刻已超过，则明天的 HH:MM 进行此工作。
  HH:MM YYYY-MM-DD		ex> 04:00 2009-03-17
	强制规定在某年某月的某一天的特殊时刻进行该工作！
  HH:MM[am|pm] [Month] [Date]	ex> 04pm March 17
	也是一样，强制在某年某月某日的某时刻进行！
  HH:MM[am|pm] + number [minutes|hours|days|weeks]
	ex> now + 5 minutes	ex> 04pm + 3 days
	就是说，在某个时间点『再加几个时间后』才进行。
```
```
再过五分钟后，将 /root/.bashrc 寄给 root 自己
[root@www ~]# at now + 5 minutes  <==单位要加 s
at> /bin/mail root -s "testing at job" < /root/.bashrc
at> <EOT>   <==这里输入 [ctrl] + d 就会出现 <EOF> 的字样！代表结束！
job 4 at 2009-03-14 15:38
# 上面这行资讯在说明，第 4 个 at 工作将在 2009/03/14 的 15:38 进行！
# 而运行 at 会进入所谓的 at shell 环境，让你下达多重命令等待运行！

将上述的第 4 项工作内容列出来查阅
[root@www ~]# at -c 4
#!/bin/sh               <==透过 bash shell
# atrun uid=0 gid=0
# mail     root 0
umask 22
....(中间省略许多的环境变量项目)....
cd /root || {           <==可以看出，会到下达 at 时的工作目录去运行命令
         echo 'Execution directory inaccessible' >&2
         exit 1
}

/bin/mail root -s "testing at job" < /root/.bashrc
# 你可以看到命令运行的目录 (/root)，还有多个环境变量与实际的命令内容

由于机房预计于 2009/03/18 停电，我想要在 2009/03/17 23:00 关机
[root@www ~]# at 23:00 2009-03-17
at> /bin/sync
at> /bin/sync
at> /sbin/shutdown -h now
at> <EOT>
job 5 at 2009-03-17 23:00
# at 可以在一个工作内输入多个命令。
```
当我们使用 at 时会进入一个 at shell 的环境来让使用者下达工作命令，此时，建议你最好使用绝对路径来下达你的命令。由于命令的下达与 PATH 变量有关， 同时与当时的工作目录也有关连 (如果有牵涉到文件的话)，因此使用绝对路径来下达命令，比较不会出问题。因为 at 在运行时，会跑到当时下达 at 命令的那个工作目录。  
at 的运行与终端机环境无关，而所有 standard output/standard error output 都会传送到运行者的 mailbox 去，所以在终端机当然看不到任何资讯，可以透过终端机的装置来处理，假如你在 tty1 登陆，则可以使用『 echo "Hello" > /dev/tty1 』来取代。
> 如果在 at shell 内的命令并没有任何的信息输出，那么 at 默认不会发 email 给运行者的。 如果你想要让 at 无论如何都发一封 email 告知你是否运行了命令，那么可以使用『 at -m 时间格式 』来下达命令。at 就会传送一个信息给运行者，而不论该命令运行有无信息输出了！

at 有另外一个很棒的优点，那就是『背景运行』的功能，与 bash 的 nohup (第十七章) 类似。  
由于 at 工作排程的使用上，系统会将该项 at 工作独立出你的 bash 环境中， 直接交给系统的 atd 程序来接管，因此，当你下达了 at 的工作之后就可以立刻离线了， 剩下的工作就完全交给 Linux 管理即可。所以，如果有长时间的网络工作时，使用 at 可以免除网络断线的困扰。
### at 工作管理
```
[root@www ~]# atq
[root@www ~]# atrm (jobnumber)

# 利用 atq 查询，利用 atrm 来删除错误的命令，利用 at 来直接下达单一工作排程。
```
### batch：系统有空时才进行背景任务
其实 batch 是利用 at 来进行命令的下达啦！只是加入一些控制参数而已。 batch 的特别之处在于：他会在 CPU 工作负载小于 0.8 的时候，才进行你所下达的工作任务。这个负载的意思是： CPU 在单一时间点所负责的工作数量，而非 CPU 的使用率。  
当 CPU 的工作负载越大，代表 CPU 必须要在不同的工作之间进行频繁的工作切换。
```
# batch 下达命令与 at 相同
[root@www ~]# batch 23:00 2009-3-17
at> sync
at> sync
at> shutdown -h now
at> <EOT>
job 6 at 2009-03-17 23:00

[root@www ~]# atq
6       2009-03-17 23:00 b root
[root@www ~]# atrm 6
```

# 循环运行的例行性工作排程
循环运行的例行性工作排程则是由 cron (crond) 这个系统服务来控制的。Linux 系统上面原本就有非常多的例行性工作，因此这个系统服务是默认启动的。由于使用者自己也可以进行例行性工作排程， Linux 也提供使用者控制例行性工作排程的命令 (crontab)。
## 使用者的配置。
使用者想要创建循环型工作排程时，使用的是 crontab 这个命令，与 at 相同，可以限制使用 crontab 的使用者账号。
* /etc/cron.allow：  
  将可以使用 crontab 的帐号写入其中，若不在这个文件内的使用者则不可使用 crontab；
* /etc/cron.deny：  
  将不可以使用 crontab 的帐号写入其中，若未记录到这个文件当中的使用者，就可以使用 crontab 。

同样的，以优先顺序来说， /etc/cron.allow 比 /etc/cron.deny 要优先， 而判断上面，这两个文件只选择一个来限制，因此，建议只要保留一个即可。一般来说，系统默认是保留 /etc/cron.deny ， 你可以将不想让他运行 crontab 的那个使用者写入 /etc/cron.deny 当中，一个帐号一行。  
当使用者使用 crontab 这个命令来创建工作排程之后，该项工作就会被纪录到 /var/spool/cron/ 里面去了，而且是以帐号来作为判别的，举例来说， dmtsai 使用 crontab 后， 他的工作会被纪录到 /var/spool/cron/dmtsai 里头去。但请注意，不要使用 vi 直接编辑该文件， 因为可能由于输入语法错误，会导致无法运行 cron。另外， cron 运行的每一项工作都会被纪录到 /var/log/cron 这个登录档中，所以，如果你的 Linux 不知道是否被植入木马时，也可以搜寻以下 /var/log/cron 这个登陆档。
```
[root@www ~]# crontab [-u username] [-l|-e|-r]
选项与参数：
-u  ：只有 root 才能进行这个任务，亦即帮其他使用者创建/移除 crontab 工作排程；
-e  ：编辑 crontab 的工作内容
-l  ：查阅 crontab 的工作内容
-r  ：移除所有的 crontab 的工作内容，若仅要移除一项，请用 -e 去编辑。

用 dmtsai 的身份在每天的 12:00 发信给自己
[dmtsai@www ~]$ crontab -e
# 此时会进入 vi 的编辑画面让您编辑工作！注意到，每项工作都是一行。
0   12  *  *  * mail dmtsai -s "at 12:00" < /home/dmtsai/.bashrc
#分 时 日 月 周 |<==============命令串========================>|
```
默认情况下，任何使用者只要不被列入 /etc/cron.deny 中，就可以直接下达【crontab -e】去编辑自己的例行性命令。每项工作 (每行) 的格式都是具有六个栏位，这六个栏位的意义为：
代表意义|分钟|小时|日期|月份|周|命令
:---:|:---:|:---:|:---:|:---:|:---:|:---:
数字范围|0-59|0-23|1-31|1-12|0-7||
周的数字为 0 或 7  时，都代表【星期天】的意义。还有一些辅助字符：
特殊字符|代表意义
:---:|:---:
*(星号)|代表任何时刻都接受的意思！举例来说，范例一内那个日、月、周都是 * ， 就代表著『不论何月、何日的礼拜几的 12:00 都运行后续命令』的意思
,(逗号)|代表分隔时段的意思。举例来说，如果要下达的工作是 3:00 与 6:00 时，就会是：<br>0 3,6 * * * command<br>时间参数还是有五栏，不过第二栏是 3,6 ，代表 3 与 6 都适用！
-(减号)|代表一段时间范围内，举例来说， 8 点到 12 点之间的每小时的 20 分都进行一项工作：<br>20 8-12 * * * command<br>代表 8,9,10,11,12 都适用的意思
/n(斜线)|那个 n 代表数字，亦即是『每隔 n 单位间隔』的意思，例如每五分钟进行一次，则：<br>*/5 * * * * command<br>用 * 与 /5 来搭配，也可以写成 0-59/5 ，相同意思
那个 crontab 每个人都只有一个文件存在，在 /var/spool/cron 里，建议：『命令下达时，最好使用绝对路径』  
『如果只是要删除某个 crontab 的工作项目，那么请使用 crontab -e 来重新编辑即可！』如果使用 -r 的参数，会将所有的 crontab 数据内容都删掉。
## 系统的配置档：/etc/crontab
管理『系统的例行性任务』，只要编辑 /etc/crontab 这个文件。需要注意，crontab -e 这个 crontab 其实是 /usr/bin/crontab 这个运行档，但是 /etc/crontab 是一个『纯文字档』。使用 root 身份编辑这个文件。  
基本上， cron 这个服务的最低侦测限制是『分钟』，所以『 cron 会每分钟去读取一次 /etc/crontab 与 /var/spool/cron 里面的数据内容 』，因此，只要你编辑完 /etc/crontab 这个文件，并且将他储存之后，那么 cron 的配置就自动的会来运行了。
> Linux 底下的 crontab 会帮我们每分钟重新读取一次 /etc/crontab 的例行工作事项，但是某些原因或者其他的 Unix 系统中，由于 crontab 是读到内存中的，修改完 /etc/crontab 之后，可能不会马上运行，需要重启 crond 这个服务，『/etc/init.d/crond restart』

```
[root@www ~]# cat /etc/crontab
SHELL=/bin/bash                     <==使用哪种 shell 介面
PATH=/sbin:/bin:/usr/sbin:/usr/bin  <==运行档搜寻路径
MAILTO=root                         <==若有额外STDOUT，以 email将数据送给谁
HOME=/                              <==默认此 shell 的家目录所在

# run-parts
01  *  *  *  *   root      run-parts /etc/cron.hourly   <==每小时
02  4  *  *  *   root      run-parts /etc/cron.daily    <==每天
22  4  *  *  0   root      run-parts /etc/cron.weekly   <==每周日
42  4  1  *  *   root      run-parts /etc/cron.monthly  <==每个月 1 号
分 时 日 月 周 运行者身份  命令串
```
* MAILTO-root：  
  这个项目是说，当 /etc/crontab 这个文件中的例行性工作的命令发生错误时，或者是该工作的运行结果有 STDOUT/STDERR 时，会将错误信息或者屏幕显示的信息传给谁。默认由系统直接寄发一封 mail 给 root 。可以将这个 e-mail 改成自己的账号，以便随时了解系统的状况。
* PATH=.....：
* 01 * * * * root run-parts /etc/cron.hourly：  
  这个 /etc/crontab 里面预配置义出四项工作任务，分别是每小时、每天、每周及每个月分别进行一次的工作！ 但是在五个栏位后面接的并不是命令，而是一个新的栏位，那就是『运行后面那串命令的身份』.与使用者的 crontab -e 不同。由于使用者自己的 crontab 并不需要指定身份，但 /etc/crontab 里面就指定身份了。以上表的内容表示，系统默认的例行性工作是以 root 的身份来进行的。  
  『 which run-parts 』发现 run-parts 是一个 bash script。这支命令会将后面接的『目录』内的所有文件捉出来运行。也就是说『 如果你想让系统每小时主动帮你运行某个命令，将该命令写成 script，并将该文件放置到 /etc/cron.hourly/ 目录下即可』。  
  
由于 CentOS 提供的 run-parts 这个 script 的辅助，因此 /etc/crontab 这个文件里面支持两种下达命令的方式， 一种是直接下达命令，一种则是以目录来规划：
* 命令形态  
  01 * * * * dmtsai mail -s "testing" kiki < /home/dmtsai/test.txt  
  以 dmtsai 这个使用者的身份，在每小时运行一次 mail 命令。
* 目录规划  
  */5 * * * * root run-parts /root/runcron  
  创建一个 /root/runcron 的目录，将要每隔五分钟运行的『可运行档』都写到该目录下， 就可以让系统每五分钟运行一次该目录下的所有可运行档。
## 一些注意事项
### 资源分配不均的问题
当大量使用 crontab 的时候，总是会有问题发生的，最严重的问题就是『系统资源分配不均』的问题。
```
[root@www ~]# vi /etc/crontab
1,6,11,16,21,26,31,36,41,46,51,56 * * * * root  CMD1
2,7,12,17,22,27,32,37,42,47,52,57 * * * * root  CMD2
3,8,13,18,23,28,33,38,43,48,53,58 * * * * root  CMD3
4,9,14,19,24,29,34,39,44,49,54,59 * * * * root  CMD4
```
可以将五分钟工作的流程分别在不同的时刻来工作。
### 取消不要的输出项目
『 当有运行成果或者是运行的项目中有输出的数据时，该数据将会 mail 给 MAILTO 配置的帐号 』。直接以『命令重导向』将输出的结果输出到 /dev/null 这个垃圾桶当中。
### 安全的检验
可以藉由检查 /var/log/cron 的内容来视察是否有『非您配置的 cron 被运行了』，检查木马。
### 周与日月不可同时并存

# 可唤醒停机期间的工作任务
如果你的 Linux 主机是作为 24 小时全天、全年无休的服务器之用，那么你只要有 atd 与 crond 这两个服务来管理你的例行性工作排程即可。如果你的服务器并非 24 小时无间断的启动，可以使用 anacron。
## 什么是 anacron
anacron 并不是用来取代 crontab 的，anacron 存在的目的是在处理非 24 小时一直启动的 Linux 系统的 crontab 的运行。所以 anacron 并不能指定何时运行某项任务， 而是以天为单位或者是在启动后立刻进行 anacron 的动作，他会去侦测停机期间应该进行但是并没有进行的 crontab 任务，并将该任务运行一遍后，anacron 就会自动停止了。  
由于 anacron 会以一天、七天、一个月为期去侦测系统未进行的 crontab 任务，因此对于某些特殊的使用环境非常有帮助。   
anacron 通过读取时间记录档 (timestamps) 获知系统的关机时间，anacron 会去分析现在的时间与时间记录档所记载的上次运行 anacron 的时间，两者比较后若发现有差异， 那就是在某些时刻没有进行 crontab。此时 anacron 就会开始运行未进行的 crontab 任务。所以 anacron 其实也是透过 crontab 来运行的！因此 anacron 运行的时间通常有两个，一个是系统启动期间运行，一个是写入 crontab 的排程中。这样才能够在特定时间分析系统未进行的 crontab 工作。
## anacron 与 /etc/anacrontab
anacron 其实是一支程序并非一个服务！这支程序在 CentOS 当中已经进入 crontab 的排程。
```
[root@www ~]# anacron [-sfn] [job]..
[root@www ~]# anacron -u [job]..
选项与参数：
-s  ：开始一连续的运行各项工作 (job)，会依据时间记录档的数据判断是否进行；
-f  ：强制进行，而不去判断时间记录档的时间戳记；
-n  ：立刻进行未进行的任务，而不延迟 (delay) 等待时间；
-u  ：仅升级时间记录档的时间戳记，不进行任何工作。
job ：由 /etc/anacrontab 定义的各项工作名称。
```
CentOS 的 /etc/cron.daily/0anacron 仅进行时间戳记的升级，而没有进行任何 anacron 的动作。anacron 的进行其实是在启动完成后才进行的一项工作任务，你也可以将 anacron 排入 crontab 的排程中。但是为了担心 anacron 误判时间参数，因此 /etc/cron.daily/ 里面的 anacron 才会在档名之前加个 0 (0anacron)，让 anacron 最先进行！就是为了让时间戳记先升级！以避免 anacron 误判 crontab 尚未进行任何工作的意思。
```
[root@www ~]# cat /etc/anacrontab
SHELL=/bin/sh
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root

1       65      cron.daily     run-parts /etc/cron.daily
7       70      cron.weekly    run-parts /etc/cron.weekly
30      75      cron.monthly   run-parts /etc/cron.monthly
天数   延迟时间 工作名称定义   实际要进行的命令串
# 天数单位为天；延迟时间单位为分钟；工作名称定义可自订；
# 命令串则通常与 crontab 的配置相同！
```
anacron 若下达『 anacron -s cron.daily 』时，他会这样运行的：
1. 由 /etc/anacrontab 分析到 cron.daily 这项工作名称的天数为 1 天；
2. 由 /var/spool/anacron/cron.daily 取出最近一次运行 anacron 的时间戳记；
3. 由上个步骤与目前的时间比较，若差异天数为 1 天以上 (含 1 天)，就准备进行命令；
4. 若准备进行命令，根据 /etc/anacrontab 的配置，将延迟 65 分钟
5. 延迟时间过后，开始运行后续命令，亦即『 run-parts /etc/cron.daily 』这串命令；
6. 运行完毕后， anacron 程序结束。

anacron 是透过该记录与目前的时间差异，了解到是否应该要进行某项任务的工作。  
anacron 并不需要额外的配置，使用默认值即可！ CentOS 只有在启动时才会运行 anacron 。如果要确定 anacron 是否启动时会主动的运行，你可以下达下列命令：
```
[root@www ~]# chkconfig --list anacron
anacron      0:off   1:off   2:on    3:on    4:on    5:on    6:off
# 详细的 chkconfig 说明我们会在后续章节提到，注意看 3, 5
# 的项目，都是 on ！启动时才会运行的意思！
```