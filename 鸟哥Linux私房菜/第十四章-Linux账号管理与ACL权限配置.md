# Linux 账号与群组
## 使用者标识符：UID 与 GID
每个登陆的使用者至少都会取得两个 ID ，一个是使用者 ID (User ID ，简称 UID)、一个是群组 ID (Group ID ，简称 GID)。  
每个文件都会有所谓的拥有着 ID 与拥有群组 ID，那么我们要显示文件属性的需求时，系统会依据 /etc/passwd 与 /etc/group 的内容，找到 UID/GID 对应的账号与组名再显示出来。
## 使用者账号
输入账号口令之后。
1. 先找寻 /etc/passwd 里面是否有你输入的账号？如果没有则跳出，如果有的话则将该账号对应的 UID 与 GID (在 /etc/group 中) 读出来，另外，该账号的家目录与 shell 配置也一并读出；
2. 再来则是核对口令表，这时 Linux 会进入 /etc/shadow 里面找出对应的账号与 UID，然后核对一下你刚刚输入的口令与里头的口令是否一致
3. 如果一切都 OK 的话，就进入 Shell 控管的阶段

跟使用者账号有关的有两个非常重要的文件，一个是管理使用者 UID/GID 重要参数的 /etc/passwd ，一个则是专门管理口令相关数据的 /etc/shadow 。
### /etc/passwd 文件结构
每一行都代表一个账号，里面有很多账号本来就是系统正常运行所必须要的，简称系统账号，例如 bin, daemon, adn, nobody 等。
```
head -n 1 /etc/passwd
root:x:0:0:root:/root:/bin/bash
```
1. 账号名称  
   用来对应 UID，例如 root 的 UID 对应就是 0 (第三字段)；
2. 口令：  
   早期 Unix 系统的口令就是放在此字段。因为这个文件的特性是所有的程序都能够读取，这样很容易造成口令数据被窃取，因此后来就将这个字段的口令数据改放到 /etc/shadow 中了，这里会看到一个 【x】
3. UID  
   Linux 对 UID 的几个限制
   id 范围|该 ID 使用者特性
   :---:|:---:
   0<br>(系统管理员)|当 UID 是 0 时，代表这个账号是『系统管理员』！ 所以当你要让其他的账号名称也具有 root 的权限时，将该账号的 UID 改为 0 即可。 这也就是说，一部系统上面的系统管理员不见得只有 root ， 不过，不建议有多个账号的 UID 是 0 
   1~499<br>(系统账号)|保留给系统使用的 ID，其实除了 0 之外，其他的 UID 权限与特性并没有不一样。默认 500 以下的数字让给系统作为保留账号只是一个习惯。<br><br>由于系统上面启动的服务希望使用较小的权限去运行，因此不希望使用 root 的身份去运行这些服务， 所以我们就得要提供这些运行中程序的拥有者账号才行。这些系统账号通常是不可登陆的， 所以才会有我们在第十一章提到的 /sbin/nologin 这个特殊的 shell 存在。<br><br>根据系统账号的由来，通常系统账号又约略被区分为两种：<br>1~99：由 distributions 自行创建的系统账号；<br>100~499：若用户有系统账号需求时，可以使用的账号 UID。
   500~65535<br>(可登陆账号)|给一般使用者用的。事实上，目前的 linux 核心 (2.6.x 版)已经可以支持到 4294967295 (2^32-1) 的 UID 号码
4. GID  
   与 /etc/group 有关。用来规范组名与 GID 的对应。
5. 用户信息说明栏：  
   只是用来解释这个账号的意义而已。如果您提供使用 finger 的功能时， 这个字段可以提供很多的信息
6. 家目录：
7. Shell  
   /sbin/nologin 可以让账号无法取得 shell 环境。可以用来制作纯 pop 邮件账号者的数据。
### /etc/shadow 文件结构 (口令——密码)
很多程序的运行都与权限有关，而权限与 UID/GID 有关，因此各程序需要读取 /etc/passwd 来了解不同账号的权限。因此 /etc/passwd 的权限需配置为 -rw-r--r--。虽然早期的口令也有加密过，但却放置到 /etc/passwd 的第二个字段上，这样一来很容易被有心人士所窃取的， 加密过的口令也能够透过暴力破解法去 try and error (试误) 找出来。  
后来发展出将口令移动到 /etc/shadow 这个文件分隔开来的技术， 而且还加入很多的口令限制参数在 /etc/shadow 里头。
```
[root@www ~]# head -n 4 /etc/shadow
root:$1$/30QpE5e$y9N/D0bh6rAACBEz.hqo00:14126:0:99999:7:::
bin:*:14126:0:99999:7:::
daemon:*:14126:0:99999:7:::
adm:*:14126:0:99999:7:::
```
九个字段的用途。
1. 账号名称：
2. 口令  
   这个字段内的数据才是真正的口令，而且是经过编码的口令 (加密)。只会看到一些特殊符号的字母。这个文件的默认权限是【-rw-------】或者【-r--------】，亦即只有 root 才可以读取。  
   由于各种口令编码的技术不一样，不同的编码系统会造成这个字段的长度不同。由于固定的编码系统产生的口令长度必须一致，因此『当你让这个字段的长度改变后，该口令就会失效(算不出来)』。 很多软件透过这个功能，在此字段前加上 ! 或 * 改变口令字段长度，就会让口令『暂时失效』。
3. 最近更动口令的日期  
   记录【更动口令那一天】的日期。Linux 日期的计算以 1970 年 1 月 1 日 作为 1 累加而来。
4. 口令不可被更动的天数: (与第 3 字段相比)  
   第四个字段记录了：这个账号的口令在最近一次被更改后需要经过几天才可以再被变更。如果是 0 的话， 表示口令随时可以更动的意思。
5. 口令需要重新变更的天数：(与第 3 字段相比)  
   为了强制要求用户变更口令，这个字段可以指定在最近一次更改口令后， 在多少天数内需要再次的变更口令才行。你必须要在这个天数内重新配置你的口令，否则这个账号的口令将会『变为过期特性』。 而如果是 99999 (计算为 273 年) 的话，那就表示，口令的变更没有强制性之意。
6. 口令需要变更期限前的警告天数：(与第 5 字段相比)  
   当账号的口令有效期限快要到的时候 (第 5 字段)，系统会依据这个字段的配置，发出『警告』言论给这个账号，提醒他『再过 n 天你的口令就要过期了，请尽快重新配置你的口令』。
7. 口令过期后的账号宽限时间(口令失效日)：(与第 5 字段相比)  
   口令有效日期为『升级日期(第3字段)』+『重新变更日期(第5字段)』，过了该期限后用户依旧没有升级口令，那该口令就算过期了。虽然口令过期但是该账号还是可以用来进行其他工作的，包括登陆系统取得 bash 。不过如果口令过期了， 那当你登陆系统时，系统会强制要求你必须要重新配置口令才能登陆继续使用，这就是口令过期特性。  
   这个字段表示，在口令过期几天后，如果使用者还是没有登陆更改口令，那么这个账号的口令将会『失效』， 亦即该账号再也无法使用该口令登陆了。要注意口令过期与口令失效并不相同。
8. 账号失效日期：  
   这个日期跟第三个字段一样，都是使用 1970 年以来的总日数配置。这个字段表示： 这个账号在此字段规定的日期之后，将无法再使用。 就是所谓的『账号失效』，此时不论你的口令是否有过期，这个『账号』都不能再被使用。这个字段会被使用通常应该是在『收费服务』的系统中，你可以规定一个日期让该账号不能再使用。
9. 保留

忘记口令的解决方法。
* 一般用户的口令忘记了：请系统管理员帮忙。利用 root 身份使用 passwd 命令来处理。
* root 口令忘了：可以使用各种可行的方法启动进入 Linux 修改 /etc/shadow 。例如重启进入单人维护模式后，系统会主动给予 root 权限的 bash 接口，此时再以 passwd 修改口令。或用 Live CD 启动后挂载根目录去修改 /etc/shadow，将里面的 root 口令字段清空，再重新启动后 root 将不用口令即可登陆，登陆后再用 passwd 命令配置 root 口令。
## 关于群组：有效与初始群组、groups、newgrp
### /etc/group 文件结构
```
[root@www ~]# head -n 4 /etc/group
root:x:0:root
bin:x:1:root,bin,daemon
daemon:x:2:root,bin,daemon
sys:x:3:root,bin,adm
```
四个字段。
1. 组名
2. 群组口令：  
   通常不需要配置，这个配置通常是给『群组管理员』使用的，目前很少有这个机会配置群组管理员了。同样的，口令已经移动到 /etc/gshadow 去，因此这个字段只会存在一个『x』。
3. GID  
   /etc/passwd 的第四个字段使用的 GID 对应的群组吗，就是由此对应来的。
4. 此群组支持的账号名称：  
   一个账号可以加入多个群组，那某个账号想要加入此群组时，将该账号填入这个字段即可。以【,】分隔，不要有空格。
### 有效群组(effective group)与初始群组(initial group)
每个使用者在他的 /etc/passwd 里面的第四栏的 GID 就是所谓的『初始群组 (initial group) 』。也就是说，用户一登陆系统，立刻就拥有这个群组的相关权限。不需要在 /etc/group 的第四个字段写入该账号。  
非 initial group 的其他群组，需要在 /etc/group 这个文件，代表群组的那一行，加入该账号，才能够加入这个群组。  
只要是用户所属群组拥有的功能和权限，这个用户都能使用。不过，如果要创建新的文件后者是目录，新文件的所属群组是当时的有效群组(effective group)。
### groups：有效与支持群组的观察
输入 groups 命令，可得到用户所属的群组。
```
groups
dmtsai users
```
第一个输出的群组即为有效群组 (effective group)。
### newgrp：有效群组的切换
使用 newgrp 是有限制的，那就是你想要切换的群组必须是你已经有支持的群组即所属的群组。  
newgrp 这个命令可以变更目前用户的有效群组， 而且是另外以一个 shell 来提供这个功能的，新的 shell 给予用户有效 GID 为切换的群组。  
虽然用户的环境配置(例如环境变量等等其他数据)不会有影响，但是使用者的『群组权限』将会重新被计算。 但是需要注意，由于是新取得一个 shell ，因此如果你想要回到原本的环境中，请输入 exit 回到原本的 shell。  
加入一个群组有两个方式，一个是透过系统管理员 (root) 利用 usermod 帮你加入，第二如果你的系统有配置群组管理员，那么你可以透过群组管理员以 gpasswd 帮你加入他所管理的群组中。
### /etc/gshadow
```
[root@www ~]# head -n 4 /etc/gshadow
root:::root
bin:::root,bin,daemon
daemon:::root,bin,daemon
sys:::root,bin,adm
```
这个文件几乎与 /etc/group 一样。区别在第二个字段，口令栏。如果口令栏上面是【!】时，表示该群组不具有群组管理员。第四个字段是支持的账号。
1. 组名
2. 口令栏，同样的，开头为 ! 表示无合法口令，所以无群组管理员
3. 群组管理员的账号 (相关信息在 gpasswd 中介绍)
4. 该群组的所属账号 (与 /etc/group 内容相同)

以系统管理员的角度来说，这个 gshadow 最大的功能就是创建群组管理员

# 账号管理
## 新增与移除使用者：useradd，相关配置文件，passwd，usermod，userdel
### useradd
```
[root@www ~]# useradd [-u UID] [-g 初始群组] [-G 次要群组] [-mM]\
>  [-c 说明栏] [-d 家目录绝对路径] [-s shell] 使用者账号名
选项与参数：
-u  ：后面接的是 UID ，是一组数字。直接指定一个特定的 UID 给这个账号；
-g  ：后面接的那个组名就是我们上面提到的 initial group
      该群组的 GID 会被放置到 /etc/passwd 的第四个字段内。
-G  ：后面接的组名则是这个账号还可以加入的群组。
      这个选项与参数会修改 /etc/group 内的相关数据
-M  ：强制！不要创建用户家目录！(系统账号默认值)
-m  ：强制！要创建用户家目录！(一般账号默认值)
-c  ：这个就是 /etc/passwd 的第五栏的说明内容，可以随便配置
-d  ：指定某个目录成为家目录，而不要使用默认值。务必使用绝对路径
-r  ：创建一个系统的账号，这个账号的 UID 会有限制 (参考 /etc/login.defs)
-s  ：后面接一个 shell ，若没有指定则默认是 /bin/bash 
-e  ：后面接一个日期，格式为『YYYY-MM-DD』此项目可写入 shadow 第八字段，亦即账号失效日的配置项目
-f  ：后面接 shadow 的第七字段项目，指定口令是否会失效。0为立刻失效，-1 为永远不失效(口令只会过期而强制于登陆时重新配置而已。)
```
其实系统已经帮我们规定很多默认值了，可以简单使用【useradd 账号】来创建使用者。CentOS 这些默认值主要会帮我们处理几个项目。
* 在 /etc/passwd 里面创建一行与账号相关的数据，包括创建 UID/GID/家目录等；
* 在 /etc/shadow 里面将此账号的口令相关参数填入，但是尚未有口令；
* 在 /etc/group 里面加入一个与账号名称一模一样的组名；
* 在 /home 底下创建一个与账号同名的目录作为用户家目录，且权限为 700

由于在 /etc/shadow 内仅会有口令参数而不会有加密过的口令数据，因此我们在创建使用者账号时， 还需要使用『 passwd 账号 』来给予口令才算是完成了用户创建的流程。  
若创建时指定一个已存在的群组作为使用者的初始群组，/etc/group 里面就不会主动的创建于账号同名的群组了。  
用户自己创建的系统账号 UID 一般由 100 号以后起算。系统账号默认不会主动创建家目录。  
使用 useradd 创建使用者账号时，会更改的地方。
* 用户账号与口令参数方面的文件：/etc/passwd, /etc/shadow
* 使用者群组相关方面的文件：/etc/group, /etc/gshadow
* 用户的家目录：/home/账号名称

### useradd 参考档
useradd 的默认值可以使用 `useradd -D` 呼出
```
[root@www ~]# useradd -D
GROUP=100		<==默认的群组
HOME=/home		<==默认的家目录所在目录
INACTIVE=-1		<==口令失效日，在 shadow 内的第 7 栏
EXPIRE=			<==账号失效日，在 shadow 内的第 8 栏
SHELL=/bin/bash		<==默认的 shell
SKEL=/etc/skel		<==用户家目录的内容数据参考目录
CREATE_MAIL_SPOOL=yes   <==是否主动帮使用者创建邮件信箱(mailbox)
```
这个数据由 /etc/default/useradd 呼叫出来。这些配置项目所造成的行为分别是：
* GROUP=100：新建账号的初始群组使用 GID 为 100 者  
  系统上面 GID 为 100 者即是 users 这个群组，此配置项目指的就是让新设使用者账号的初始群组为 users 这一个的意思。 针对群组的角度有两种不同的机制：
    * 私有群组机制：系统会创建一个与账号一样的群组给使用者作为初始群组。 这种群组的配置机制会比较有保密性，这是因为使用者都有自己的群组，而且家目录权限将会配置为 700 (仅有自己可进入自己的家目录) 之故。使用这种机制将不会参考 GROUP=100 这个配置值。代表性的 distributions 有 RHEL, Fedora, CentOS 等；
    * 公共群组机制：就是以 GROUP=100 这个配置值作为新建账号的初始群组，因此每个账号都属于 users 这个群组， 且默认家目录通常的权限会是『 drwxr-xr-x ... username users ... 』，由于每个账号都属于 users 群组，因此大家都可以互相分享家目录内的数据之故。代表 distributions 如 SuSE等。
* HOME=/home：用户家目录的基准目录(basedir)
* INACTIVE=-1：口令过期后是否会失效的配置值  
  如果是 0 代表口令过期立刻失效， 如果是 -1 则是代表口令永远不会失效，如果是数字，如 30 ，则代表过期 30 天后才失效。shadow 文件结构的第七个字段。
* EXPIRE=：账号失效的日期  
  shadow 的第八个字段，可以直接配置账号在哪个日期后就直接失效，而不理会口令的问题。通常不会配置此项
* SHELL=/bin/bash：默认使用的 shell 程序文件名  
  假如你的系统为 mail server ，你希望每个账号都只能使用 email 的收发信件功能， 而不许用户登陆系统取得 shell ，那么可以将这里配置为 /sbin/nologin ，如此一来，新建的使用者默认就无法登陆！ 也免去后续使用 usermod 进行修改的手续。
* CREATE_MAIL_SPOOL=yes：创建使用者的 mailbox

除了这些基本的账号配置之外，UID/GID 还有口令参数，可以参考 /etc/login.defs 文件。
```
MAIL_DIR        /var/spool/mail	<==用户默认邮件信箱放置目录

PASS_MAX_DAYS   99999	<==/etc/shadow 内的第 5 栏，多久需变更口令日数
PASS_MIN_DAYS   0	<==/etc/shadow 内的第 4 栏，多久不可重新配置口令日数
PASS_MIN_LEN    5	<==口令最短的字符长度，已被 pam 模块取代，失去效用！
PASS_WARN_AGE   7	<==/etc/shadow 内的第 6 栏，过期前会警告的日数

UID_MIN         500	<==使用者最小的 UID，意即小于 500 的 UID 为系统保留
UID_MAX       60000	<==使用者能够用的最大 UID
GID_MIN         500	<==使用者自定义组的最小 GID，小于 500 为系统保留
GID_MAX       60000	<==使用者自定义组的最大 GID

CREATE_HOME     yes	<==在不加 -M 及 -m 时，是否主动创建用户家目录？
UMASK           077     <==用户家目录创建的 umask ，因此权限会是 700
USERGROUPS_ENAB yes     <==使用 userdel 删除时，是否会删除初始群组
MD5_CRYPT_ENAB yes      <==口令是否经过 MD5 的加密机制处理
```
* mailbox 所在目录：  
  用户的默认 mailbox 文件放置的目录在 /var/spool/mail，所以 vbird1 的 mailbox 就是在 /var/spool/mail/vbird1。
* shadow 口令第 4, 5, 6 字段内容：  
  透过 PASS_MAX_DAYS 等等配置值来指定，因此何默认的 /etc/shadow 内每一行都会有『 0:99999:7 』，不过要注意的是，由于目前我们登陆时改用 PAM 模块来进行口令检验，所以那个 PASS_MIN_LEN 是失效的！
* UID/GID 指定数值：  
  虽然 Linux 核心支持的账号可高达 232 这么多个，不过一部主机要作出这么多账号在管理上很麻烦， 所以在这里就针对 UID/GID 的范围进行规范。上表中的 UID_MIN 指的就是可登陆系统的一般账号的最小 UID ，至于 UID_MAX 则是最大 UID 之意。  
  要注意的是，系统给予一个账号 UID 时，他是 (1)先参考 UID_MIN 配置值取得最小数值； (2)由 /etc/passwd 搜寻最大的 UID 数值， 将 (1) 与 (2) 相比，找出最大的那个再加一就是新账号的 UID 了。  
  如果我是想要创建系统用的账号，所以使用 useradd -r sysaccount 这个 -r 的选项时，就会找『比 500 小的最大的那个 UID + 1 』。
* 用户家目录配置值：  
  『CREATE_HOME = yes』配置值时，系统默认会帮用户创建家目录。这个配置值会让你在使用 useradd 时， 主动加入『 -m 』这个产生家目录的选项！如果不想要创建用户家目录，就只能强制加上『 -M 』的选项在 useradd 命令运行时！创建家目录的权限配置透过 umask 这个配置，因为是 077 的默认配置，因此用户家目录默认权限才会是『 drwx------(700) 』
* 用户删除与口令配置值：  
  使用『USERGROUPS_ENAB yes』这个配置值的功能是： 如果使用 userdel 去删除一个账号时，且该账号所属的初始群组已经没有人隶属于该群组了， 那么就删除掉该群组，举例来说，我们刚刚有创建 vbird4 这个账号，他会主动创建 vbird4 这个群组。 若 vbird4 这个群组并没有其他账号将他加入支持的情况下，若使用 userdel vbird4 时，该群组也会被删除的意思。 至于『MD5_CRYPT_ENAB yes』则表示使用 MD5 来加密口令明文，而不使用旧式的 DES。

useradd 这支程序在创建 Linux 的账号时，至少会参考：
* /etc/default/useradd
* /etc/login.defs
* /etc/skel/*
### passwd
使用 useradd 创建了账号之后，在默认的情况下，该账号是暂时被封锁的， 也就是说，该账号是无法登陆的，/etc/shadow 的第二个字段未配置。需要使用 passwd 配置口令。
```
[root@www ~]# passwd [--stdin]  <==所有人均可使用来改自己的口令
[root@www ~]# passwd [-l] [-u] [--stdin] [-S] \
>  [-n 日数] [-x 日数] [-w 日数] [-i 日期] 账号 <==root 功能
选项与参数：
--stdin ：可以透过来自前一个管线的数据，作为口令输入，对 shell script 有帮助！
-l  ：是 Lock 的意思，会将 /etc/shadow 第二栏最前面加上 ! 使口令失效；
-u  ：与 -l 相对，是 Unlock 的意思！
-S  ：列出口令相关参数，亦即 shadow 文件内的大部分信息。
-n  ：后面接天数，shadow 的第 4 字段，多久不可修改口令天数
-x  ：后面接天数，shadow 的第 5 字段，多久内必须要更动口令
-w  ：后面接天数，shadow 的第 6 字段，口令过期前的警告天数
-i  ：后面接『日期』，shadow 的第 7 字段，口令失效日期

root 给予 vbird2 口令
passwd vbird2

修改自己的口令
passwd
```
当我们要给予用户口令时，透过 root 来配置即可。 root 可以配置各式各样的口令，系统几乎一定会接受。  
要帮一般账号创建口令需要使用『 passwd 账号 』的格式，使用『 passwd 』表示修改自己的口令  
一般账号在更改口令时需要先输入自己的旧口令 (亦即 current 那一行)，然后再输入新口令 (New 那一行)。 要注意的是，口令的规范是非常严格的，尤其新的 distributions 大多使用 PAM 模块来进行口令的检验，包括太短、 口令与账号相同、口令为字典常见字符串等，都会被 PAM 模块检查出来而拒绝修改口令，此时会再重复出现『 New 』这个关键词。   
 root 并不需要知道旧口令就能够帮用户或 root 自己创建新口令  
使用 PAM 模块来管理口令，这个管理的机制写在 /etc/pam.d/passwd 当中。而该文件与口令有关的测试模块就是使用：pam_cracklib.so，这个模块会检验口令相关的信息， 并且取代 /etc/login.defs 内的 PASS_MIN_LEN 的配置。理论上，口令最好符合如下要求：
* 口令不能与账号相同；
* 口令尽量不要选用字典里面会出现的字符串；
* 口令需要超过 8 个字符；
* 口令不要使用个人信息，如身份证、手机号码、其他电话号码等；
* 口令不要使用简单的关系式，如 1+1=2， Iamvbird 等；
* 口令尽量使用大小写字符、数字、特殊字符($,_,-等)的组合。

```
使用 standard input 创建用户命令
echo "abc543asdf" | passwd --stdin vbird2

# 会直接升级用户口令，不用再次手动配置。缺点是这个口令会保留在命令中， 未来若系统被攻破，人家可以在 /root/.bash_history 找到这个口令，这个动作通常尽在 shell script 的大量创建使用者账号当中。
```
```
管理 vbird2 的口令使口令具有 60 天变更、10 天口令失效的配置
[root@www ~]# passwd -S vbird2
vbird2 PS 2009-02-26 0 99999 7 -1 (Password set, MD5 crypt.)
# 上面说明口令创建时间 (2009-02-26)、0 最小天数、99999 变更天数、7 警告日数
# 与口令不会失效 (-1) 。

[root@www ~]# passwd -x 60 -i 10 vbird2
[root@www ~]# passwd -S vbird2
vbird2 PS 2009-02-26 0 60 7 10 (Password set, MD5 crypt.)

让 vbird2 的账号暂时失效，观察完毕后再解锁
[root@www ~]# passwd -l vbird2
[root@www ~]# passwd -S vbird2
vbird2 LK 2009-02-26 0 60 7 10 (Password locked.)
# 状态变成『 LK, Lock 』了，无法登陆
[root@www ~]# grep vbird2 /etc/shadow
vbird2:!!$1$50MnwNFq$oChX.0TPanCq7ecE4HYEi.:14301:0:60:7:10::
# 其实是在口令字段前加上 !!

[root@www ~]# passwd -u vbird2
[root@www ~]# grep vbird2 /etc/shadow
vbird2:$1$50MnwNFq$oChX.0TPanCq7ecE4HYEi.:14301:0:60:7:10::
# 口令字段恢复正常
```
### chage
更详细的口令参数显示功能。
```
[root@www ~]# chage [-ldEImMW] 账号名
选项与参数：
-l ：列出该账号的详细口令参数；
-d ：后面接日期，修改 shadow 第三字段(最近一次更改口令的日期)，格式 YYYY-MM-DD
-E ：后面接日期，修改 shadow 第八字段(账号失效日)，格式 YYYY-MM-DD
-I ：后面接天数，修改 shadow 第七字段(口令失效日期)
-m ：后面接天数，修改 shadow 第四字段(口令最短保留天数)
-M ：后面接天数，修改 shadow 第五字段(口令多久需要进行变更)
-W ：后面接天数，修改 shadow 第六字段(口令过期前警告日期)

[root@www ~]# chage -l vbird2
Last password change                               : Feb 26, 2009
Password expires                                   : Apr 27, 2009
Password inactive                                  : May 07, 2009
Account expires                                    : never
Minimum number of days between password change     : 0
Maximum number of days between password change     : 60
Number of days of warning before password expires  : 7
```
chage 可以实现『使用者在第一次登陆时， 强制他们一定要更改口令后才能够使用系统资源』
```
创建一个名为 agetest 的账号，该账号第一次登陆后使用默认口令，但必须要更改过口令后，使用新口令才能够登陆系统使用 bash 环境
[root@www ~]# useradd agetest
[root@www ~]# echo "agetest" | passwd --stdin agetest
[root@www ~]# chage -d 0 agetest
# 此时账号的口令创建时间会被改为 1970/1/1，所以会有问题。
# 使用此账号登陆时，会被强制要求修改口令。更改口令完成后就会被踢出系统。再次登陆时就能够使用新口令登陆了
```
### usermod
```
[root@www ~]# usermod [-cdegGlsuLU] username
选项与数：
-c  ：后面接账号的说明，即 /etc/passwd 第五栏的说明栏，可以加入一些账号的说明。
-d  ：后面接账号的家目录，即修改 /etc/passwd 的第六栏；
-e  ：后面接日期，格式是 YYYY-MM-DD 也就是在 /etc/shadow 内的第八个字段数据
-f  ：后面接天数，为 shadow 的第七字段。
-g  ：后面接初始群组，修改 /etc/passwd 的第四个字段，亦即是 GID 的字段
-G  ：后面接次要群组，修改这个使用者能够支持的群组，修改的是 /etc/group 
-a  ：与 -G 合用，可『添加次要群组的支持』而非『配置』
-l  ：后面接账号名称。亦即是修改账号名称， /etc/passwd 的第一栏！
-s  ：后面接 Shell 的实际文件，例如 /bin/bash 或 /bin/csh 等等。
-u  ：后面接 UID 数字，即 /etc/passwd 第三栏的数据；
-L  ：暂时将用户的口令冻结，让他无法登陆。其实仅改 /etc/shadow 的口令栏。
-U  ：将 /etc/shadow 口令栏的修改去掉，解冻。

创建的系统账号时，没有给予家目录，创建家目录
[root@www ~]# ll -d ~vbird3
ls: /home/vbird3: No such file or directory  <==确认一下，确实没有家目录的存在！
[root@www ~]# cp -a /etc/skel /home/vbird3
[root@www ~]# chown -R vbird3:vbird3 /home/vbird3
[root@www ~]# chmod 700 /home/vbird3
[root@www ~]# ll -a ~vbird3
drwx------  4 vbird3 vbird3 4096 Sep  4 18:15 .  <==用户家目录权限
drwxr-xr-x 11 root   root   4096 Feb 26 11:45 ..
-rw-r--r--  1 vbird3 vbird3   33 May 25  2008 .bash_logout
-rw-r--r--  1 vbird3 vbird3  176 May 25  2008 .bash_profile
-rw-r--r--  1 vbird3 vbird3  124 May 25  2008 .bashrc
drwxr-xr-x  3 vbird3 vbird3 4096 Sep  4 18:11 .kde
drwxr-xr-x  4 vbird3 vbird3 4096 Sep  4 18:15 .mozilla
# 使用 chown -R 是为了连同家目录底下的用户/群组属性都一起变更的意思；
# 使用 chmod 没有 -R ，是因为我们仅要修改目录的权限而非内部文件的权限！
```
usermod 也是用来微调 useradd 添加的使用者参数。
### userdel
删除用户的相关数据，用户的数据有：
* 用户账号/口令相关参数：/etc/passwd, /etc/shadow
* 使用者群组相关参数：/etc/group, /etc/gshadow
* 用户个人文件数据： /home/username, /var/spool/mail/username..
```
[root@www ~]# userdel [-r] username
选项与参数：
-r  ：连同用户的家目录也一起删除
```
通常我们要移除一个账号的时候，你可以手动的将 /etc/passwd 与 /etc/shadow 里头的该账号取消即可！一般而言，如果该账号只是『暂时不激活』的话，那么将 /etc/shadow 里头账号失效日期 (第八字段) 配置为 0 就可以让该账号无法使用，但是所有跟该账号相关的数据都会留下来。使用 userdel 的通常是『你真的确定不要让该用户在主机上面使用任何数据了』。  
另外，其实用户如果在系统上面操作过一阵子了，那么该用户其实在系统内可能会含有其他文件的。 举例来说，他的邮件信箱 (mailbox) 或者是例行性工作排程 (crontab, 十六章) 之类的文件。 所以，如果想要完整的将某个账号完整的移除，最好可以在下达 `userdel -r username` 之前， 先以『 find / -user username 』查出整个系统内属于 username 的文件，然后再加以删除。
## 用户功能
useradd/usermod/userdel ，那都是系统管理员所能够使用的命令。一般身份用户常用的账号数据变更于查询命令。
### finger
```
[root@www ~]# finger [-s] username
选项与参数：
-s  ：仅列出用户的账号、全名、终端机代号与登陆时间等等；
-m  ：列出与后面接的账号相同者，而不是利用部分比对 (包括全名部分)
```
finger 列出的几乎都是 /etc/passwd 文件里的数据。信息说明如下：
* Login：为使用者账号，亦即 /etc/passwd 内的第一字段；
* Name：为全名，亦即 /etc/passwd 内的第五字段(或称为批注)；
* Directory：就是家目录了；
* Shell：就是使用的 Shell 文件所在；
* Never logged in.：figner 还会调查用户登陆主机的情况！
* No mail.：调查 /var/spool/mail 当中的信箱数据；
* No Plan.：调查 ~vbird1/.plan 文件，并将该文件取出来说明！

是否能够查阅到 Mail 与 Plan 则与权限有关了！因为 Mail / Plan 都是与使用者自己的权限配置有关。
### chfn
change finger?
```
[root@www ~]# chfn [-foph] [账号名]
选项与参数：
-f  ：后面接完整的大名；
-o  ：您办公室的房间号码；
-p  ：办公室的电话号码；
-h  ：家里的电话号码！

# 其实就是改到第五个字段，该字段里面用多个『 , 』分隔。
```
### chsh
change shell
```
[vbird1@www ~]$ chsh [-ls]
选项与参数：
-l  ：列出目前系统上面可用的 shell ，其实就是 /etc/shells 的内容！
-s  ：配置修改自己的 Shell
```
不论是 chfn 于 chsh ，都能够让一般用户修改 /etc/passwd 这个系统文件，这两个文件的权限，一定是 SUID。
### id
查询某人或自己相关的 UID/GID 等信息。也可以判断出系统上面有无某账号。
```
[root@www ~]# id [username]
```
## 新增于移除群组
基本上，群组的内容都与这两个文件有关：/etc/group, /etc/gshadow。都是上面两个文件的新增、修改与移除而已。
### groupadd
```
[root@www ~]# groupadd [-g gid] [-r] 组名
选项与参数：
-g  ：后面接某个特定的 GID ，用来直接给予某个 GID
-r  ：创建系统群组，与 /etc/login.defs 内的 GID_MIN 有关。

# 非系统群组，群组的 GID 默认也是由 500 以上最大 GID + 1 来决定。
```
### groupmod
与 usermod 类似，仅进行 group 相关参数的修改而已。
```
[root@www ~]# groupmod [-g gid] [-n group_name] 群组名
选项与参数：
-g  ：修改既有的 GID 数字；
-n  ：修改既有的组名

[root@www ~]# groupmod -g 201 -n mygroup group1
[root@www ~]# grep mygroup /etc/group /etc/gshadow
/etc/group:mygroup:x:201:
/etc/gshadow:mygroup:!::
# 不要随意更动 GID ，容易造成系统资源的错乱。
```
### groupdel
删除群组
```
[root@www ~]# groupdel [groupname]
```
『有某个账号 (/etc/passwd) 的 initial group 使用该群组，不能被删除』 
### gpasswd：群组管理员功能
```
# 关于系统管理员(root)做的动作：
[root@www ~]# gpasswd groupname
[root@www ~]# gpasswd [-A user1,...] [-M user3,...] groupname
[root@www ~]# gpasswd [-rR] groupname
选项与参数：
    ：若没有任何参数时，表示给予 groupname 一个口令(/etc/gshadow)
-A  ：将 groupname 的主控权交由后面的使用者管理(该群组的管理员)
-M  ：将某些账号加入这个群组当中！
-r  ：将 groupname 的口令移除
-R  ：让 groupname 的口令栏失效

# 关于群组管理员(Group administrator)做的动作：
[someone@www ~]$ gpasswd [-ad] user groupname
选项与参数：
-a  ：将某位使用者加入到 groupname 这个群组当中！
-d  ：将某位使用者移除出 groupname 这个群组当中。
```
群组管理员可以有多个。
## 账号管理实例

# 主机的细部权限规划：ACL 的使用
## ACL 机制
ACL 是 Access Control List 的缩写，主要的目的是在提供传统的 owner, group, others 的 read, write, execute 权限之外的细部权限配置。ACL 可以针对单一使用者，单一文件或目录来进行 r,w,x 的权限规范，对于需要特殊权限的使用状况非常有帮助。  
主要针对以下几个项目：
* 使用者 (user)：可以针对使用者来配置权限；
* 群组 (group)：针对群组为对象来配置其权限；
* 默认属性 (mask)：还可以针对在该目录下在创建新文件/目录时，规范新数据的默认权限；
## 如何启动 ACL
由于 ACL 是传统的 Unix-like 操作系统权限的额外支持项目，因此要使用 ACL 必须要有文件系统的支持才行。目前绝大部分的文件系统都有支持 ACL 的功能，包括 ReiserFS, EXT2/EXT3, JFS, XFS 等等。
目前新的 distributions 常常会主动加入 acl 支持，若要自己开启。
```
[root@www ~]# mount -o remount,acl /
[root@www ~]# mount
/dev/hda2 on / type ext3 (rw,acl)
# 这样就加入了！但是如果想要每次启动都生效，那就这样做：

[root@www ~]# vi /etc/fstab
LABEL=/1   /   ext3    defaults,acl    1 1
```
## ACL 的配置技巧：getfacl, setfacl
* getfacl：取得某个文件/目录的 ACL 配置项目
* setfacl：配置某个文件/目录的 ACL 规范
### setfacl 命令用法
```
[root@www ~]# setfacl [-bkRd] [{-m|-x} acl参数] 目标文件名
选项与参数：
-m ：配置后续的 acl 参数给文件使用，不可与 -x 合用；
-x ：删除后续的 acl 参数，不可与 -m 合用；
-b ：移除所有的 ACL 配置参数；
-k ：移除默认的 ACL 参数，关于所谓的『默认』参数于后续范例中介绍；
-R ：递归配置 acl ，亦即包括次目录都会被配置起来；
-d ：配置『默认 acl 参数』的意思！只对目录有效，在该目录新建的数据会引用此默认值
```
对单一使用者的配置方式：
```
# 配置规范：『 u:[使用者账号列表]:[rwx] 』
[root@www ~]# touch acl_test1
[root@www ~]# ll acl_test1
-rw-r--r-- 1 root root 0 Feb 27 13:28 acl_test1
[root@www ~]# setfacl -m u:vbird1:rx acl_test1
[root@www ~]# ll acl_test1
-rw-r-xr--+ 1 root root 0 Feb 27 13:28 acl_test1
# 权限部分多了个 + ，且与原本的权限 (644) 看起来差异很大

[root@www ~]# setfacl -m u::rwx acl_test1
[root@www ~]# ll acl_test1
-rwxr-xr--+ 1 root root 0 Feb 27 13:28 acl_test1
# 无使用者列表，代表配置该文件拥有者
```
『 u:使用者:权限 』的方式，最简单的配置方式。如果一个文件配置了 ACL 参数后，他的权限部分就会多出一个 + 号，但是此时你看到的权限与实际权限可能就会有点误差，透过 getfacl 观察。
### getfacl 命令用法
```
[root@www ~]# getfacl filename
选项与参数：
getfacl 的选项几乎与 setfacl 相同

[root@www ~]# getfacl acl_test1
# file: acl_test1   <==说明档名
# owner: root       <==说明此文件的拥有者，亦即 ll 看到的第三使用者字段
# group: root       <==此文件的所属群组，亦即 ll 看到的第四群组字段
user::rwx           <==使用者列表栏是空的，代表文件拥有者的权限
user:vbird1:r-x     <==针对 vbird1 的权限配置为 rx ，与拥有者并不同！
group::r--          <==针对文件群组的权限配置仅有 r 
mask::r-x           <==此文件默认的有效权限 (mask)
other::r--          <==其他人拥有的权限
```
```
针对特定群组的 ACL 配置方式
# 配置规范：『 g:[群组列表]:[rwx] 』
[root@www ~]# setfacl -m g:mygroup1:rx acl_test1
[root@www ~]# getfacl acl_test1
# file: acl_test1
# owner: root
# group: root
user::rwx
user:vbird1:r-x
group::r--
group:mygroup1:r-x  <==这里就是新增的部分！多了这个群组的权限配置
mask::r-x
other::r--
```
mask 的意义：使用者或者群组所配置的权限必须要存在于 mask 的权限配置范围内才会生效，此即『有效权限 (effective permission)』。
```
针对有效权限 mask 的配置方式：
# 配置规范：『 m:[rwx] 』
[root@www ~]# setfacl -m m:r acl_test1
[root@www ~]# getfacl acl_test1
# file: acl_test1
# owner: root
# group: root
user::rwx
user:vbird1:r-x        #effective:r-- <==vbird1 和 mask 都存在的权限，仅有 r
group::r--
group:mygroup1:r-x     #effective:r--
mask::r--
other::r--
```
可以透过使用 mask 来规范最大允许的权限，就能够避免不小心开放某些权限给其他使用者或群组了。  
如果想要让 ACL 在目录下的数据都有继承的功能
```
针对默认权限的配置方式
# 配置规范：『 d:[ug]:使用者列表:[rwx] 』
[root@www ~]# setfacl -m d:u:myuser1:rx /srv/projecta
```

# 使用者身份切换
* 使用一般账号：系统平日操作的好习惯
* 用较低的权限启动系统服务
* 软件本身的限制  
  远古时代 telnet 程序，默认不允许 root 的身份登陆，telnet 会判断登陆者的 UID，若为 0 则拒绝登陆。ssh 也可以配置拒绝 root 登陆。

通常使用一般账号登陆系统的，等有需要进行系统维护或软件升级时才转为 root 的身份来动作。切换方式：
* 以『 su - 』直接将身份变成 root 即可，但是这个命令却需要 root 的口令，也就是说，如果你要以 su 变成 root 的话，你的一般使用者就必须要有 root 的口令才行。
* 以『 sudo 命令 』运行 root 的命令串，由于 sudo 需要事先配置妥当，且 sudo 需要输入用户自己的口令， 因此多人共管同一部主机时， sudo 要比 su 好。
## su
su 是最简单的身份切换命令，可以进行任何身份的切换
```
[root@www ~]# su [-lm] [-c 命令] [username]
选项与参数：
-   ：单纯使用 - 如『 su - 』代表使用 login-shell 的变量文件读取方式来登陆系统；
      若使用者名称没有加上去，则代表切换为 root 的身份。
-l  ：与 - 类似，但后面需要加欲切换的使用者账号！也是 login-shell 的方式。
-m  ：-m 与 -p 是一样的，表示『使用目前的环境配置，而不读取新使用者的配置文件』
-c  ：仅进行一次命令，所以 -c 后面可以加上命令
```
```
使用 non-login shell 的方式变成 root
[vbird1@www ~]$ su       <==注意提示字符，是 vbird1 的身份
Password:                <==这里输入 root 的口令
[root@www vbird1]# id    <==提示字符的目录是 vbird1
uid=0(root) gid=0(root) groups=0(root),1(bin),...   <==确实是 root 的身份！
[root@www vbird1]# env | grep 'vbird1'
USER=vbird1
PATH=/usr/local/bin:/bin:/usr/bin:/home/vbird1/bin  <==这个影响最大！
MAIL=/var/spool/mail/vbird1                         <==收到的 mailbox 是 vbird1
PWD=/home/vbird1                                    <==并非 root 的家目录
LOGNAME=vbird1
# 虽然你的 UID 已经是具有 root 的身份，但是, 
# 还是有一堆变量为原本 vbird1 的身份，所以很多数据还是无法直接利用。
[root@www vbird1]# exit   <==这样可以离开 su 的环境！
```
单纯使用『 su 』切换成为 root 的身份，读取的变量配置方式为 non-login shell 的方式，这种方式很多原本的变量不会被改变， 尤其是 PATH 变量，由于没有改变成为 root 的环境 ( /sbin, /usr/sbin 等目录都没有被包含进来)， 因此很多 root 惯用的命令就只能使用绝对路径来运行。其他的还有 MAIL 这个变量，你输入 mail 时， 收到的邮件还是 vbird1 的，而不是 root 本身的邮件。
```
使用 login shell 的方式切换为 root 的身份
[vbird1@www ~]$ su -
Password:   <==这里输入 root 的口令
[root@www ~]# env | grep root
USER=root
MAIL=/var/spool/mail/root
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin
PWD=/root
HOME=/root
LOGNAME=root
[root@www ~]# exit
```
如果只是想要运行『一个只有 root 才能进行的命令，且运行完毕就恢复原本的身份』，使用 -c 选项
```
[vbird1@www ~]$ su - -c "head -n 3 /etc/shadow"
```
root 用户或其他用户，相互切换
```
su -l dmtsai
```
* 若要完整的切换到新使用者的环境，必须要使用『 su - username 』或『 su -l username 』， 才会连同 PATH/USER/MAIL 等变量都转成新用户的环境；
* 如果仅想要运行一次 root 的命令，可以利用『 su - -c "命令串" 』的方式来处理；
* 使用 root 切换成为任何使用者时，并不需要输入新用户的口令；
## sudo
相对于 su 需要了解新切换的用户口令 (常常是需要 root 的口令)， sudo 的运行则仅需要自己的口令即可。 sudo 可以让你以其他用户的身份运行命令 (通常是使用 root 的身份来运行命令)，因此并非所有人都能够运行 sudo ， 而是仅有规范到 /etc/sudoers 内的用户才能够运行 sudo 这个命令。
### sudo 的命令用法
开始系统仅有 root 可以运行 sudo，先观察以 root 身份运行
```
[root@www ~]# sudo [-b] [-u 新使用者账号]
选项与参数：
-b  ：将后续的命令放到背景中让系统自行运行，而不与目前的 shell 产生影响
-u  ：后面可以接欲切换的使用者，若无此项则代表切换身份为 root 。

要以 sshd 的身份在 /tmp 底下创建一个名为 mysshd 的文件
[root@www ~]# sudo -u sshd touch /tmp/mysshd
[root@www ~]# ll /tmp/mysshd
-rw-r--r-- 1 sshd sshd 0 Feb 28 17:42 /tmp/mysshd

以 vbird1 的身份创建 ~vbird1/www 并于其中创建 index.html 文件
[root@www ~]# sudo -u vbird1 sh -c "mkdir ~vbird1/www; cd ~vbird1/www; \
>  echo 'This is index.html file' > index.html"
```
因为我们无法使用『 su - sshd 』去切换系统账号 (因为系统账号的 shell 是 /sbin/nologin)，需要使用 sudo。  
sudo 的运行流程：
1. 当用户运行 sudo 时，系统于 /etc/sudoers 文件中搜寻该使用者是否有运行 sudo 的权限；
2. 若使用者具有可运行 sudo 的权限后，便让使用者『输入用户自己的口令』来确认；
3. 若口令输入成功，便开始进行 sudo 后续接的命令(但 root 运行 sudo 时，不需要输入口令)；
4. 若欲切换的身份与运行者身份相同，那也不需要输入口令。

『能否使用 sudo 必须要看 /etc/sudoers 的配置值， 而可使用 sudo 者是透过输入用户自己的口令来运行后续的命令串』，因为该文件的内容是有一定规范的，需要透过 visudo 去修改这个文件。
### visudo 与 /etc/sudoers
除了 root 之外的其他账号，若想要使用 sudo 运行属于 root 的权限命令，则 root 需要先使用 visudo 去修改 /etc/sudoers ，让该账号能够使用全部或部分的 root 命令功能。/etc/sudoers 是有配置语法的，如果配置错误那会造成无法使用 sudo 命令的不良后果。因此才会使用 visudo 去修改， 并在结束离开修改画面时，系统会去检验 /etc/sudoers 的语法就是了。
1. 单一用户可进行 root 所有命令，与 sudoers 文件语法：  
   ```
    [root@www ~]# visudo
    ....(前面省略)....
    root    ALL=(ALL)       ALL  <==找到这一行，大约在 76 行左右
    vbird1  ALL=(ALL)       ALL  <==这一行是你要新增的！
   ```
   其实 visudo 只是利用 vi 将 /etc/sudoers 文件呼叫出来进行修改而已，所以这个文件就是 /etc/sudoers 。
   ```
    使用者账号  登陆者的来源主机名=(可切换的身份)  可下达的命令
    root                         ALL=(ALL)           ALL   <==这是默认值
   ```
   这四个组件的意义是：
   1. 系统的哪个账号可以使用 sudo 这个命令的意思，默认为 root 这个账号；
   2. 当这个账号由哪部主机联机到本 Linux 主机，意思是这个账号可能是由哪一部网络主机联机过来的， 这个配置值可以指定客户端计算机(信任用户的意思)。默认值 root 可来自任何一部网络主机
   3. 这个账号可以切换成什么身份来下达后续的命令，默认 root 可以切换成任何人；
   4. 可用该身份下达什么命令？这个命令请务必使用绝对路径撰写。 默认 root 可以切换任何身份且进行任何命令之意。

   ALL 是特殊的关键词，代表任何身份、主机或命令的意思。
2. 利用群组以及免口令的功能处理 visudo  
   ```
   [root@www ~]# visudo  <==同样的，请使用 root 先配置
    ....(前面省略)....
    %wheel     ALL=(ALL)    ALL <==大约在 84 行左右，请将这行的 # 拿掉！
    # 在最左边加上 % ，代表后面接的是一个『群组』之意！改完请储存后离开
    ```
    『任何加入 wheel 这个群组的使用者，就能够使用 sudo 切换任何身份来操作任何命令』
    ```
    不需要口令即可是使用 sudo
    [root@www ~]# visudo  <==同样的，请使用 root 先配置
    ....(前面省略)....
    %wheel     ALL=(ALL)   NOPASSWD: ALL <==大约在 87 行左右，请将 # 拿掉！
    # 在最左边加上 % ，代表后面接的是一个『群组』之意！改完请储存后离开
    ```
3. 有限制的命令操作  
   上面两点都会让使用者能够利用 root 的身份进行任何事情，如果想要让用户仅能够进行部分系统任务， 比方说，系统上面的 myuser1 仅能够帮 root 修改其他用户的口令时，亦即『当使用者仅能使用 passwd 这个命令帮忙 root 修改其他用户的口令』时
   ```
   [root@www ~]# visudo  <==注意是 root 身份
   myuser1	ALL=(root)  /usr/bin/passwd  <==最后命令务必用绝对路径
   ```
   配置值指的是『myuser1 可以切换成为 root 使用 passwd 这个命令』的意思。其中要注意的是： 命令字段必须要填写绝对路径。如果按上述配置，myuser1 将能够 root 的口令。要限制用户的命令参数。
   ```  
   [root@www ~]# visudo  <==注意是 root 身份
   myuser1	ALL=(root)  !/usr/bin/passwd, /usr/bin/passwd [A-Za-z]*, \
                        !/usr/bin/passwd root
   ```
   加上惊叹号『 ! 』代表『不可运行』的意思。
4. 透过别名建置 visudo  
   visudo 的别名可以是『命令别名、帐户别名、主机别名』等。  
   假设我的 pro1, pro2, pro3 与 myuser1, myuser2 要加入上述的口令管理员的 sudo 列表中， 那我可以创立一个帐户别名称为 ADMPW 的名称，然后将这个名称处理一下即可。处理的方式如下：
   ```
   [root@www ~]# visudo  <==注意是 root 身份
   User_Alias ADMPW = pro1, pro2, pro3, myuser1, myuser2
   Cmnd_Alias ADMPWCOM = !/usr/bin/passwd, /usr/bin/passwd [A-Za-z]*, \
                         !/usr/bin/passwd root
   ADMPW   ALL=(root)  ADMPWCOM
   ```
   我透过 User_Alias 创建出一个新账号，这个账号名称一定要使用大写字符来处理，包括 Cmnd_Alias(命令别名)、Host_Alias(来源主机名别名) 都需要使用大写字符。
5. sudo 的时间间隔问题：  
   如果我使用同一个账号在短时间内重复操作 sudo 来运行命令的话， 在第二次运行 sudo 时，并不需要输入自己的口令。两次运行 sudo 的间隔在五分钟内，那么再次运行 sudo 时就不需要再次输入口令。  
   使用一般账号时，理论上不会使用到 /sbin, /usr/sbin 等目录内的命令，所以 $PATH 变量不会含有这些目录，因此很多管理命令需要使用绝对路径来下达比较妥当。
6. sudo 搭配 su 的使用方式  
   ```
   [root@www ~]# visudo
   User_Alias  ADMINS = pro1, pro2, pro3, myuser1
   ADMINS ALL=(root)  /bin/su -
   ```
   接下来，上述的 pro1, pro2, pro3, myuser1 这四个人，只要输入『 sudo su - 』并且输入『自己的口令』后， 立刻变成 root 的身份。『这些你加入的使用者，全部都是你能够信任的用户』。

# 使用者的特殊 shell 与 PAM 模块
## 特殊的 shell，/sbin/nologin
系统账号的 shell 就是使用 /sbin/nologin。『这个使用者无法使用 bash 或其他 shell 来登陆系统』，并不是说这个账号无法使用其他的系统资源。各个系统账号，打印作业由 lp 这个账号在管理， WWW 服务由 apache 这个账号在管理， 他们都可以进行系统程序的工作，但是『就是无法登陆主机』。  
如果想要让某个具有 /sbin/nologin 的使用者知道，他们不能登陆主机时， 其实我可以创建『 /etc/nologin.txt 』这个文件， 并且在这个文件内说明不能登陆的原因，那么下次当这个用户想要登陆系统时， 屏幕上出现的就会是 /etc/nologin.txt 这个文件的内容，而不是默认的内容了。
## PAM 模块简介
PAM (Pluggable Authentication Modules, 嵌入式模块) 机制。  
PAM 是一套应用程序编程接口 (Application Programing Interface, API)，他提供了一连串的验证机制，只要使用者将验证阶段的需求告知 PAM 后，PAM 就能够回报使用者验证的结果 (成功或失败)。由于 PAM 仅是一套验证的机制，又可以提供给其他程序所呼叫引用，因此不论你使用什么程序，都可以使用 PAM 来进行验证，如此一来，就能够让账号口令或者是其他方式的验证具有一致的结果。  
 PAM 是一个独立的 API 存在，只要任何程序有需求时，可以向 PAM 发出验证要求的通知， PAM 经过一连串的验证后，将验证的结果回报给该程序，然后该程序就能够利用验证的结果来进行可登陆或显示其他无法使用的信息。 这也就是说，你可以在写程序的时候将 PAM 模块的功能加入，就能够利用 PAM 的验证功能。  
 PAM 用来进行验证的数据称为模块 (Modules)，每个 PAM 模块的功能都不太相同。举例来说，使用 passwd 命令时，如果随便输入字典上面找的到的字符串， passwd 就会回报错误信息，就是 PAM 的 pam_cracklib.so 模块的功能。他能够判断该口令是否在字典里面！ 并回报给口令修改程序，此时就能够了解你的口令强度了。
## PAM 模块配置语法
PAM 藉由一个与程序相同文件名的配置文件来进行一连串的认证分析需求。我们同样以 passwd 这个命令的呼叫 PAM 来说明。运行 passwd 后，呼叫 PAM 的流程是：
1. 用户开始运行 /usr/bin/passwd 这支程序，并输入口令；
2. passwd 呼叫 PAM 模块进行验证；
3. PAM 模块会到 /etc/pam.d/ 找寻与程序 (passwd) 同名的配置文件；
4. 依据 /etc/pam.d/passwd 内的配置，引用相关的 PAM 模块逐步进行验证分析；
5. 将验证结果 (成功、失败以及其他信息) 回传给 passwd 这支程序；
6. passwd 这支程序会根据 PAM 回传的结果决定下一个动作 (重新输入新口令或者通过验证！)

```
[root@www ~]# cat /etc/pam.d/passwd
#%PAM-1.0  <==PAM版本的说明而已！
auth       include      system-auth <==每一行都是一个验证的过程
account    include      system-auth
password   include      system-auth
验证类别   控制标准     PAM 模块与该模块的参数
```
除了第一行宣告 PAM 版本之外，其他任何『 # 』开头的都是批注，而每一行都是一个独立的验证流程， 每一行可以区分为三个字段，分别是验证类别(type)、控制标准(flag)、PAM的模块与该模块的参数。
【include】这个关键词，代表的是，【请呼叫后面的文件作为这个类别的验证】，所以，上述的每一行都要重复呼叫 /etc/pam.d/system-auth 那个文件来进行验证。
### 第一个字段：验证类别 (Type)
验证类别主要分为四种：
* auth  
  是 authentication (认证) 的缩写，所以这种类别主要用来检验使用者的身份验证，这种类别通常是需要口令来检验的， 所以后续接的模块是用来检验用户的身份。
* account  
  account (账号) 则大部分是在进行 authorization (授权)，这种类别则主要在检验使用者是否具有正确的权限。举例来说，当你使用一个过期的口令来登陆时，当然就无法正确的登陆了。
* session  
  session 是会议期间的意思，所以 session 管理的就是使用者在这次登陆 (或使用这个命令) 期间，PAM 所给予的环境配置。这个类别通常用在记录用户登陆与注销时的信息。例如，如果你常常使用 su 或者是 sudo 命令的话， 那么应该可以在 /var/log/secure 里面发现很多关于 pam 的说明，而且记载的数据是『session open, session close』的信息。
* password  
  这种类别主要在提供验证的修订工作，举例来说，就是修改/变更口令。

这四个验证的类型通常是有顺序的。原因是，(1)我们总是得要先验证身份 (auth) 后， (2)系统才能够藉由用户的身份给予适当的授权与权限配置 (account)，而且(3)登陆与注销期间的环境才需要配置， 也才需要记录登陆与注销的信息 (session)。如果在运行期间需要口令修订时，(4)才给予 password 的类别。
### 第二个字段：验证的控制旗标 (control flag)
『验证通过的标准』。这个字段管控该验证的放行方式，主要也分为四种控制方式：
* required  
  此验证若成功则带有 success (成功) 的标志，若失败则带有 failure 的标志，但不论成功或失败都会继续后续的验证流程。由于后续的验证流程可以继续进行，因此相当有利于数据的登录 (log) ，这也是 PAM 最常使用 required 的原因。
* requisite  
  若验证失败则立刻回报原程序 failure 的标志，并终止后续的验证流程。若验证成功则带有 success 的标志并继续后续的验证流程。这个项目与 required 最大的差异，就是在于失败的时候要不要继续验证下去，由于 requisite 是失败就终止，因此失败时产生的 PAM 信息就无法透过后续的模块来记录了。
* sufficient  
  若验证成功则立刻回传 success 给原程序，并终止后续的验证流程；若验证失败则带有 failure 标志并继续后续的验证流程。
* optional  
  这个模块控件目的大多是在显示信息而已，并不是用在验证方面的

程序运行过程中遇到验证时才会去呼叫 PAM ，而 PAM 验证又分很多类型与控制，不同的控制旗标所回报的信息并不相同。 验证结束后所回报的信息，通常是【success 或 failure】，后续的流程需要该程序的判断来继续运行。
## 常用模块简介
* /etc/pam.d/*：每个程序个别的 PAM 配置文件；
* /lib/security/*：PAM 模块文件的实际放置目录；
* /etc/security/*：其他 PAM 环境的配置文件
* /user/share/doc/pam-*/：详细的 PAM 说明文件

几个较常用的模块：
* pam_securetty.so:  
  限制系统管理员 (root) 只能够从安全的 (secure) 终端机登陆；写在 /etc/securetty 这个文件中。
* pam_nologin.so:  
  这个模块可以限制一般用户是否能够登陆主机之用。当 /etc/nologin 这个文件存在时，则所有一般使用者均无法再登陆系统。若 /etc/nologin 存在，则一般使用者在登陆时， 在他们的终端机上会将该文件的内容显示出来！所以，正常的情况下，这个文件应该是不能存在系统中的。 但这个模块对 root 以及已经登陆系统中的一般账号并没有影响。
* pam_selinux.so:  
  SELinux 是个针对程序来进行细部管理权限的功能，由于 SELinux 会影响到用户运行程序的权限，因此我们利用 PAM 模块，将 SELinux 暂时关闭，等到验证通过后， 再予以启动！
* pam_console.so:  
  当系统出现某些问题，或者是某些时刻你需要使用特殊的终端接口 (例如 RS232 之类的终端联机设备) 登陆主机时， 这个模块可以帮助处理一些文件权限的问题，让使用者可以透过特殊终端接口 (console) 顺利的登陆系统。
* pam_loginuid.so  
  系统账号与一般账号的 UID 是不同的！一般账号 UID 均大于 500 才合理。 因此，为了验证使用者的 UID 真的是我们所需要的数值，可以使用这个模块来进行规范！
* pam_env.so  
  用来配置环境变量的一个模块，如果你有需要额外的环境变量配置，可以参考 /etc/security/pam_env.conf 这个文件的详细说明。
* pam_unix.so:  
  这是个很复杂且重要的模块，这个模块可以用在验证阶段的认证功能，可以用在授权阶段的账号许可证管理， 可以用在会议阶段的登录文件记录等，甚至也可以用在口令升级阶段的检验。
* pam_cracklib.so:  
  可以用来检验口令的强度！包括口令是否在字典中，口令输入几次都失败就断掉此次联机等功能，都是这模块提供的
* pam_limit.so:  
  ulimit 就是这个模块提供的能力，更多细部的配置参考 /etc/security/limits.conf 内的说明。

login 的 PAM 验证机制流程：
1. 验证阶段 (auth)：首先，(a)会先经过 pam_securetty.so 判断，如果使用者是 root 时，则会参考 /etc/securetty 的配置； 接下来(b)经过 pam_env.so 配置额外的环境变量；再(c)透过 pam_unix.so 检验口令，若通过则回报 login 程序；若不通过则(d)继续往下以 pam_succeed_if.so 判断 UID 是否大于 500 ，若小于 500则回报失败，否则再往下 (e)以 pam_deny.so 拒绝联机。
2. 授权阶段 (account)：(a)先以 pam_nologin.so 判断 /etc/nologin 是否存在，若存在则不许一般使用者登陆； (b)接下来以 pam_unix 进行账号管理，再以 (c) pam_succeed_if.so 判断 UID 是否小于 500 ，若小于 500 则不记录登录信息。(d)最后以 pam_permit.so 允许该账号登陆。
3. 口令阶段 (password)：(a)先以 pam_cracklib.so 配置口令仅能尝试错误 3 次；(b)接下来以 pam_unix.so 透过 md5, shadow 等功能进行口令检验，若通过则回报 login 程序，若不通过则 (c)以 pam_deny.so 拒绝登陆。
4. 会议阶段 (session)：(a)先以 pam_selinux.so 暂时关闭 SELinux；(b)使用 pam_limits.so 配置好用户能够操作的系统资源； (c)登陆成功后开始记录相关信息在登录文件中； (d)以 pam_loginuid.so 规范不同的 UID 权限；(e)开启 pam_selinux.so 的功能。

依据验证类别 (type) 来看，然后先由 login 的配置值去查阅，如果出现【include system-auth】就转到 system-auth 文件中的相同类别，去取得额外的验证流程。然后再到下一个验证类别，最终所有的验证跑完。
## 其他相关文件
除了 /etc/securetty 会影响到 root 可登陆的安全终端机，/etc/nologin 会影响到一般使用者能否登陆的功能之外，PAM 相关的配置文件在 /etc/pam.d，说明文件在 /usr/share/doc/pam-(版本)，模块实际在 /lib/security/。还有一些相关的 PAM 文件，在 /etc/security 这个目录内。
### limits.conf
ulimit 的功能，除了修改使用者的 ~/.bashrc 配置文件之外，其实系统管理员可以统一藉由 PAM 来管理。就是 /etc/security/limits.conf 这个文件配置。
```
vbird1 这个用户只能创建 100MB 的文件，且大于 90MB 会警告
[root@www ~]# vi /etc/security/limits.conf
vbird1	soft		fsize		 90000
vbird1	hard		fsize		100000
#账号   限制依据	限制项目 	限制值
# 第一字段为账号，或者是群组！若为群组则前面需要加上 @ ，例如 @projecta
# 第二字段为限制的依据，是严格(hard)，还是仅为警告(soft)；
# 第三字段为相关限制，此例中限制文件容量，
# 第四字段为限制的值，在此例中单位为 KB。

限制 pro1 这个群组，每次仅能有一个用户登陆系统 (maxlogins)
[root@www ~]# vi /etc/security/limits.conf
@pro1   hard   maxlogins   1
# 如果要使用群组功能的话，这个功能似乎对初始群组才有效
# 而如果你尝试多个 pro1 的登陆时，第二个以后就无法登陆了。
# 而且在 /var/log/secure 文件中还会出现如下的信息：
# pam_limits(login:session): Too many logins (max 1) for pro1
```
这个文件配置完成就生效了，不用重启任何服务。但是 PAM 有个特殊的地方，由于他是在程序呼叫时才予以配置，因此修改完成的数据，对于已登陆系统中的用户是没有效果的，要等他再次登陆才会生效。
### /var/log/secur, /var/log/messages
如果发生任何无法登陆或者是产生一些你无法预期的错误时，由于 PAM 模块都会将数据记载在 /var/log/secure 当中，所以发生了问题请务必到该文件内去查询一下问题点。

# Linux 主机上的用户信息传递
## 查询使用者：w, who, last, lastlog
使用 last 可以查询登陆者信息。  
要知道目前已登陆在系统上面的用户，可以通过 w 或 who 来查询。
```
[root@www ~]# w
 13:13:56 up 13:00,  1 user,  load average: 0.08, 0.02, 0.01
USER   TTY    FROM            LOGIN@   IDLE   JCPU   PCPU WHAT
root   pts/1  192.168.1.100   11:04    0.00s  0.36s  0.00s -bash
vbird1 pts/2  192.168.1.100   13:15    0.00s  0.06s  0.02s w
# 第一行显示目前的时间、启动 (up) 多久，几个用户在系统上平均负载等；
# 第二行只是各个项目的说明，
# 第三行以后，每行代表一个使用者。如上所示，root 登陆并取得终端机名 pts/1 之意。

[root@www ~]# who
root     pts/1        2009-03-04 11:04 (192.168.1.100)
vbird1   pts/2        2009-03-04 13:15 (192.168.1.100)
```
如果要知道每个账号的最近登陆时间，可以使用 lastlog 这个命令。lastlog 回去读取 /var/log/lastlog 文件。
```
[root@www ~]# lastlog
Username    Port   From           Latest
root        pts/1  192.168.1.100  Wed Mar  4 11:04:22 +0800 2009
bin                                        **Never logged in**
....(中间省略)....
vbird1      pts/2  192.168.1.100  Wed Mar  4 13:15:56 +0800 2009
....(以下省略)....
```
## 使用者对谈：write, mesg, wall
write 可以直接将信息传给接收者。
```
[root@www ~]# write 使用者账号 [用户所在终端接口]
```
如果不想要接受任何信息
```
[vbird1@www ~]$ mesg n
[vbird1@www ~]$ mesg
is n
```
不过，这个 mesg 的功能对 root 传送来的信息没有抵挡的能力。如果想要解开的话，再次下达 `mesg y` 即可。要知道目前 mesg 的状态，直接下达 `mesg` 即可。  
『对所有系统上面的用户传送简讯 (广播)』，使用 wall。
```
[root@www ~]# wall "I will shutdown my linux server..."
```
## 使用者邮件信箱：mail
一般来说， mailbox 都会放置在 /var/spool/mail 里面，一个账号一个 mailbox (文件)。  
直接使用 mail 这个命令，寄出信件。【`mail username@localhost -s "邮件标题"`】，一般来说，如果是寄给本机上的使用者，基本上，不用写『 @localhost 』
```
[root@www ~]# mail vbird1 -s "nice to meet you"
Hello, D.M. Tsai
Nice to meet you in the network.
You are so nice.  byebye!
.    <==这里很重要喔，结束时，最后一行输入小数点 . 即可！
Cc:  <==这里是所谓的『副本』，不需要寄给其他人，所以直接 [Enter]
[root@www ~]#  <==出现提示字符，表示输入完毕了！
```
可以使用数据流重导向，` mail vbird1 -s "nice to meet you" < filename `。  
收信同样也使用 mail。
```
[vbird1@www ~]$ mail
Mail version 8.1 6/6/93.  Type ? for help.
"/var/spool/mail/vbird1": 1 message 1 new
>N  1 root@www.vbird.tsai   Wed Mar  4 13:36  18/663   "nice to meet you"
&  <==这里可以输入很多的命令，如果要查阅，输入 ? 即可！
```
 这封信件的前面那个 > 代表目前处理的信件，而在大于符号的左边那个 N 代表该封信件尚未读过，mail 内部的命令。
 ```
 & ?
    Mail   Commands
t <message list>                type messages
n                               goto and type next message
e <message list>                edit messages
f <message list>                give head lines of messages
d <message list>                delete messages
s <message list> file           append messages to file
u <message list>                undelete messages
R <message list>                reply to message senders
r <message list>                reply to message senders and all recipients
pre <message list>              make messages go back to /usr/spool/mail
m <user list>                   mail to specific users
q                               quit, saving unresolved messages in mbox
x                               quit, do not remove system mailbox
h                               print out active message headers
!                               shell escape
cd [directory]                  chdir to directory or home if none given
# <message list>指的是每封邮件左边的数字，编号
```
命令|意义
:---:|:---:
h|列出信件标头；如果要查阅 40 封信件左右的信件标头，可以输入『 h 40 』
d|删除后续接的信件号码，删除单封是『 d10 』，删除 20~40 封则为『 d20-40 』。 不过，这个动作要生效的话，必须要配合 q 这个命令才行(参考底下说明)！
s|将信件储存成文件。例如我要将第 5 封信件的内容存成 ~/mail.file:『s 5 ~/mail.file』
x|或者输入 exit 都可以。这个是『不作任何动作离开 mail 程序』的意思。 不论你刚刚删除了什么信件，或者读过什么，使用 exit 都会直接离开 mail，所以刚刚进行的删除与阅读工作都会无效。 如果您只是查阅一下邮件而已的话，一般来说，建议使用这个离开啦！除非你真的要删除某些信件。
q|相对于 exit 是不动作离开， q 则会进行两项动作： 1. 将刚刚删除的信件移出 mailbox 之外； 2. 将刚刚有阅读过的信件存入 ~/mbox ，且移出 mailbox 之外。
读取 ~/mbox 使用 【`mail -f /home/vbird/mbox`】

# 手动新增使用者
## 一些检查工具
### pwck
pwck 这个命令在检查 /etc/passwd 这个账号配置文件内的信息，与实际的家目录是否存在等信息，还可以对比 /etc/passwd /etc/shadow 的信息是否一致。另外，如果 /etc/passwd 内的数据字段错误时，会提示使用者修订。一般来说，只是利用它来检查输入。
```
[root@www ~]# pwck
user adm: directory /var/adm does not exist
user uucp: directory /var/spool/uucp does not exist
user gopher: directory /var/gopher does not exist
```
系统账号没有家目录，可以忽略。对应的群组检查可以使用 grpck。
### pwconv
这个命令主要的目的是在『将 /etc/passwd 内的账号与口令，移动到 /etc/shadow 当中！』。早期的 Unix 系统当中并没有 /etc/shadow，用户的登陆口令早期是在 /etc/passwd 的第二栏，后来为了系统安全，才将口令数据移动到 /etc/shadow 内的。使用 pwconv 后，可以：
* 比对 /etc/passwd 及 /etc/shadow ，若 /etc/passwd 内存在的账号并没有对应的 /etc/shadow 口令时，则 pwconv 会去 /etc/login.defs 取用相关的口令数据，并创建该账号的 /etc/shadow 数据；
* 若 /etc/passwd 内存在加密后的口令数据时，则 pwconv 会将该口令栏移动到 /etc/shadow 内，并将原本的 /etc/passwd 内相对应的口令栏变成 x 

一般来说，正常使用 useradd 添加使用者时，pwconv 并不会有任何动作。
### pwunconv
相对于 pwconv ， pwunconv 则是『将 /etc/shadow 内的口令栏数据写回 /etc/passwd 当中， 并且删除 /etc/shadow 文件。』不建议使用
### chpasswd
可以『读入未加密前的口令，并且经过加密后， 将加密后的口令写入 /etc/shadow 当中。』这个命令很常被使用在大量建置账号的情况中。可以由 Standard input 读入数据，每笔数据的格式是『 username:password 』。 
```
echo "dmtsai:abcdefg" | chpasswd -m
```
在默认的情况中， chpasswd 使用的是 DES 加密方法来加密， 我们可以使用 chpasswd -m 来使用 CentOS 5.x 默认的 MD5 加密方法。CentOS 5.x 其实已经提供了『 passwd --stdin 』的选项，这个 chpasswd 可以不必使用了。
## 特殊账号：如纯数字账号的手工创建
为了系统安全起见，不建议使用纯数字的账号。
* 先创建所需要的群组 ( vi /etc/group )；
* 将 /etc/group 与 /etc/gshadow 同步化 ( grpconv )；
* 创建账号的各个属性 ( vi /etc/passwd )；
* 将 /etc/passwd 与 /etc/shadow 同步化 ( pwconv )；
* 创建该账号的口令 ( passwd accountname )；
* 创建用户家目录 ( cp -a /etc/skel /home/accountname )；
* 更改用户家目录的属性 ( chown -R accountname.group /home/accountname )。
## 大量建置账号范本(适用 passwd --stdin 选项)
```
[root@www ~]# vi account1.sh
#!/bin/bash
# 这支程序用来创建新增账号，功能有：
# 1. 检查 account1.txt 是否存在，并将该文件内的账号取出；
# 2. 创建上述文件的账号；
# 3. 将上述账号的口令修订成为『强制第一次进入需要修改口令』的格式。
# 2009/03/04    VBird
export PATH=/bin:/sbin:/usr/bin:/usr/sbin

# 检查 account1.txt 是否存在
if [ ! -f account1.txt ]; then
        echo "所需要的账号文件不存在，请创建 account1.txt ，每行一个账号名称"
        exit 1
fi

usernames=$(cat account1.txt)

for username in $usernames
do
        useradd $username                         <==新增账号
        echo $username | passwd --stdin $username <==与账号相同的口令
        chage -d 0 $username                      <==强制登陆修改口令
done
```
## 大量建置账号的范例(适用于连续数字，如学号)
* 默认不允许使用纯数字方式创建账号；
* 可加入年级来区分账号；
* 可配置账号的起始号码与账号数量；
* 有两种口令创建方式，可以与账号相同或程序自行以随机数创建口令文件。
```
#!/bin/bash
#
# 这支程序主要在帮您创建大量的账号之用，更多的使用方法请参考：
#
# History:
# 2005/09/05    VBird 
# 2009/03/04    VBird   加入一些语系的修改与说明，修改口令产生方式 (用 openssl)
export LANG=zh_TW.big5
export PATH=/sbin:/usr/sbin:/bin:/usr/bin
accountfile="user.passwd"

# 1. 进行账号相关的输入先！
echo ""
echo "例如我们昆山四技的学号为： 4960c001 到 4960c060 ，那么："
echo "账号开头代码为         ：4"
echo "账号层级或年级为       ：960c"
echo "号码数字位数为(001~060)：3"
echo "账号开始号码为         ：1"
echo "账号数量为             ：60"
echo ""
read -p "账号开头代码 ( Input title name, ex> std )======> " username_start
read -p "账号层级或年级 ( Input degree, ex> 1 or enter )=> " username_degree
read -p "号码部分的数字位数 ( Input \# of digital )======> " nu_nu
read -p "起始号码 ( Input start number, ex> 520 )========> " nu_start
read -p "账号数量 ( Input amount of users, ex> 100 )=====> " nu_amount
read -p "口令标准 1) 与账号相同 2)随机数自定义 ==============> " pwm
if [ "$username_start" == "" ]; then
        echo "没有输入开头的代码，不给你运行哩！" ; exit 1
fi
# 判断数字系统
testing0=$(echo $nu_nu     | grep '[^0-9]' )
testing1=$(echo $nu_amount | grep '[^0-9]' )
testing2=$(echo $nu_start  | grep '[^0-9]' )
if [ "$testing0" != "" -o "$testing1" != "" -o "$testing2" != "" ]; then
        echo "输入的号码不对啦！有非为数字的内容！" ; exit 1
fi
if [ "$pwm" != "1" ]; then
        pwm="2"
fi

# 2. 开始输出账号与口令文件！
[ -f "$accountfile" ] && mv $accountfile "$accountfile"$(date +%Y%m%d)
nu_end=$(($nu_start+$nu_amount-1))
for (( i=$nu_start; i<=$nu_end; i++ ))
do
        nu_len=${#i}
        if [ $nu_nu -lt $nu_len ]; then
                echo "数值的位数($i->$nu_len)已经比你配置的位数($nu_nu)还大！"
                echo "程序无法继续"
                exit 1
        fi
        nu_diff=$(( $nu_nu - $nu_len ))
        if [ "$nu_diff" != "0" ]; then
                nu_nn=0000000000
                nu_nn=${nu_nn:1:$nu_diff}
                # 表示从 $nunn 这个变量 (参数) 的第一个字符开始，取 $nu_diff 个字符
        fi
        account=${username_start}${username_degree}${nu_nn}${i}
        if [ "$pwm" == "1" ]; then
                password="$account"
        else
                password=$(openssl rand -base64 6)
        fi
        echo "$account":"$password" | tee -a "$accountfile"
done

# 3. 开始创建账号与口令！
cat "$accountfile" | cut -d':' -f1 | xargs -n 1 useradd -m
chpasswd < "$accountfile"
pwconv
echo "OK！创建完成！"
```
如果有需要创建同一班级具有同一群组的话，可以先使用 groupadd 创建群组后， 将该群组加入『 cat "$accountfile" | cut -d':' -f1 | xargs -n 1 useradd -m -g groupname 』那行。  
将刚刚创建的使用者全部删除：
```
[root@www ~]# vi delaccount2.sh
#!/bin/bash
usernames=$(cat user.passwd | cut -d ':' -f 1)
for username in $usernames
do
	echo "userdel -r $username"
	userdel -r $username
done
[root@www ~]# sh delaccount2.sh
```