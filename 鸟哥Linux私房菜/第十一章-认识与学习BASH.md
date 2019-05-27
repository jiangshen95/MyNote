# 简介
shell 是操作系统提供给用户的一个接口，通过这接口可以呼叫其他应用软件，与系统交互。能够操作应用程序的接口都能够称为壳程序。狭义的壳程序指的是命令列方面的软件，包括本章要介绍的 bash 等。 广义的壳程序则包括图形接口的软件。  
Linux 使用的 Shell 为 【Bourne Again Shell(简称 bash)】，是 Bourne Shell 增强版本，基于 GNU 架构发展。  
Linux 系统支持的 合法的 shells 记录在 /etc/shells 文件
* /bin/sh (已经被 /bin/bash 所取代)
* /bin/bash (就是 Linux 默认的 shell)
* /bin/ksh (Kornshell 由 AT&T Bell lab. 发展出来的，兼容于 bash)
* /bin/tcsh (整合 C Shell ，提供更多的功能)
* /bin/csh (已经被 /bin/tcsh 所取代)
* /bin/zsh (基于 ksh 发展出来的，功能更强大的 shell)

举例来说，某些 FTP 网站会去检查使用者的可用 shell ，而如果你不想要让这些使用者使用 FTP 以外的主机资源时，可能会给予该使用者一些特殊的 shell，让使用者无法以其他服务登陆主机。 这个时候，你就得将那些特殊的 shell 写到 /etc/shells 当中了。 比如 /sbin/nologin。  
使用者在登陆时，默认取得的 shell 记录在 /etc/passwd 中，即每一行的最后一个数据
## Bash shell 的功能
### 命令编修能力 （history）
可以记忆使用过的命令。记录在家目录内的 .bash_history 中，需留意的是，~/.bash_history 记录的是前一次登陆以前所运行过的命令，而这一次登陆所运行的命令都被缓存在内存中，当你成功注销系统之后，该命令的记忆才会记录到 .bash_history 当中。 可以查询曾经做过的动作。
### 命令与文件补全功能：([tab] 按键)
少打字，和确认输入的数据是正确的
* [Tab] 接在一串命令的第一个字的后面，则为命令补全；
* [Tab] 接在一串命令的第二个字以后时，则为『文件补齐』！
### 命令别名配置功能：(alias)
在命令列输入 alias 就可以知道目前的命令别名有哪些。也可以直接下达 `alias lm='ls -al'`
### 工作控制、前景背景控制：(job control, foreground, background)
可以将工作放到背景中运行，放置被 [ctrl] + c 停掉，也可以在单一登陆环境中，达到多任务的目的
### 程序化脚本：(shell scripts)
可以将平时管理系统常需要下达的连续命令写成一个文件，该文件可以透过对谈交互式的方式进行主机的侦测工作，也可以藉由 shell 提供的环境变量及相关命令来进行设计。
### 通配符：(Wildcard)
除了完整的字符串外，bash 还支持许多通配符来帮助用户查询与命令下达。比如万能字符 【*】
## Bash shell 的内建命令：type
为了方便 shell 操作，bash 已经内建了许多命令，例如 cd, umask 等。  
利用 `type` 命令查看命令是来源于外部命令(其他非 bash 所提供的命令)或者是内建在 bash 当中的。
```
[root@www ~]# type [-tpa] name
选项与参数：
    ：不加任何选项与参数时，type 会显示出 name 是外部命令还是 bash 内建命令
-t  ：当加入 -t 参数时，type 会将 name 以底下这些字眼显示出他的意义：
      file    ：表示为外部命令；
      alias   ：表示该命令为命令别名所配置的名称；
      builtin ：表示该命令为 bash 内建的命令功能；
-p  ：如果后面接的 name 为外部命令时，才会显示完整文件名；
-a  ：会由 PATH 变量定义的路径中，将所有含 name 的命令都列出来，包含 alias
```
由于利用 type 搜寻后面的名称时，如果后面接的名称并不能以运行档的状态被找到， 那么该名称是不会被显示出来的。
## 命令的下达
如果命令太长可以使用【\Enter】换行，\表示跳脱紧接着的下一个字符，中间不能有空格之类的，紧跟 Enter 表示让 Enter 按键不再具有【开始运行】的功能，顺利跳脱 [Enter] 后，下一行最前面就会主动出现 > 的符号，继续输入。

# Shell 的变量功能
## 变量
让某一特定字符串代表不固定的内容。变量就是以一组文字或符号等，来取代一些配置或者是一串保留的数据
### 变量的可变性与方便性
例如不同用户登陆系统时，MAIL 这个变量就代表不同的目录，相应用户的信箱目录。
### 影响 bash 环境操作的变量
环境变量，例如 PATH、HOME、MAIL、SHELL等等，为了区分其与自定义变量的不同，环境变量通常以大写字母表示。
### 脚本程序设计 (shell script) 自定义变量

## 变量的取用与配置：echo，变量配置守则：unset
变量被取用时，前面必须加上【$】符号。
### 变量的取用：echo
```
echo $PATH
echo ${PATH}
```
配置变量，使用【=】连接他与他的内容即可。在 bash 当中，当一个变量名尚未被配置时，默认的内容是【空】的，变量在配置时，要符合某些规定。
### 变量的配置守则
1. 变量与变量的内容以一个等号 【=】 来连接
2. 等号两边不能直接接空格符
3. 变量的名称只能是英文字母与数字，但是开头字符不能是数字
4. 变量内容若有空格符可使用双引号【"】或单引号【'】将变量内容结合起来，但是：
    * 双引号内的特殊字符如 $ 等，可以保有原本的特性，如  
     『var="lang is $LANG"』则『echo $var』可得『lang is en_US』
    * 单引号内的特殊字符则仅为一般字符(纯文本)，例如：  
      『var='lang is $LANG'』则『echo $var』可得『lang is $LANG』
5. 可用跳脱字符 【\】 将特殊符号 (如 [enter], $, \, 空格符 等) 变成一般字符
6. 在一串命令中，还需要藉由其他的命令提供的信息，可以使用反单引号 【\`命令\`】 或 【$(命令)】。` 是键盘上方的数字键 1 左边那个按键，而不是单引号。例如要取得核心版本的配置：  
   『version=$(uname -r)』再『echo $version』可得『2.6.18-128.el5』
7. 若该变量为扩增变量内容时，则可用 “$变量名称” 或 ${变量名称}累加内容，如：  
   【PATH="$PATH":/home/bin】
8. 若该变量需要在其他子程序运行，则需要以 export 来使变量变成环境变量：  
   【export PATH】
9. 通常大写字符为系统默认变量，自行配置变量可以使用小写字符，方便判断
10. 取消变量的方法为使用 unset：【unset 变量名称】

子程序：从目前的 shell 去激活另一个新的 shell ，新的 shell 就是子程序。在一般状态下，父程序的自定义变量是无法在子程序内使用的。但是透过 export 变成环境变量后，就能够在子程序底下应用了。
```
在一串命令中，在 ` 之内的命令将会被先运行，而其运行出来的结果将做为外部的输入信息
进入目前核心的目录
cd /lib/modules/`uname -r`/kernel
cd /lib/modules/$(uname -r)/kernel
```
## 环境变量
### 用 env 查看环境变量与常见环境变量说明
env(environment)
* HOME
* SHELL  
  目前环境使用的 SHELL 是哪支程序，Linux 默认使用 /bin/bash
* HISTSIZE  
  这个与『历史命令』有关，亦即是， 我们曾经下达过的命令可以被系统记录下来，而记录的『笔数』则是由这个值来配置的。
* MAIL  
  当我们使用 mail 这个命令在收信时，系统会去读取的邮件信箱文件 (mailbox)。
* PATH  
  就是运行文件搜寻的路径啦～目录与目录中间以冒号(:)分隔， 由于文件的搜寻是依序由 PATH 的变量内的目录来查询，所以，目录的顺序也是重要的。
* LANG  
  语系数据
* RANDOM  
  【随机随机数】变量。目前大多数的 distributions 都会有随机数生成器，那就是 /dev/random 这个文件。 我们可以透过这个随机数文件相关的变量 ($RANDOM) 来随机取得随机数值。在 BASH 的环境下，这个 RANDOM 变量的内容，介于 0~32767 之间，也可以利用 declare 宣告数值类型。例如取得 0-9 之间的随机数：  
  ```
  declare -i number=$RANDOM*10/32768 ; echo $number
  ```
### 用 set 查看所有变量 (含环境变量与自定义变量)
环境变量，与 bash 操作接口有关的变量，以及用户自定义的变量。
```
[root@www ~]# set
BASH=/bin/bash           <== bash 的主程序放置路径
BASH_VERSINFO=([0]="3" [1]="2" [2]="25" [3]="1" [4]="release" 
[5]="i686-redhat-linux-gnu")      <== bash 的版本
BASH_VERSION='3.2.25(1)-release'  <== 也是 bash 的版本
COLORS=/etc/DIR_COLORS.xterm      <== 使用的颜色纪录文件
COLUMNS=115              <== 在目前的终端机环境下，使用的字段有几个字符长度
HISTFILE=/root/.bash_history      <== 历史命令记录的放置文件，隐藏档
HISTFILESIZE=1000        <== 存起来(与上个变量有关)的文件之命令的最大纪录笔数。
HISTSIZE=1000            <== 目前环境下，可记录的历史命令最大笔数。
HOSTTYPE=i686            <== 主机安装的软件主要类型。我们用的是 i686 兼容机器软件
IFS=$' \t\n'             <== 默认的分隔符
LINES=35                 <== 目前的终端机下的最大行数
MACHTYPE=i686-redhat-linux-gnu    <== 安装的机器类型
MAILCHECK=60             <== 与邮件有关。每 60 秒去扫瞄一次信箱有无新信！
OLDPWD=/home             <== 上个工作目录。我们可以用 cd - 来取用这个变量。
OSTYPE=linux-gnu         <== 操作系统的类型！
PPID=20025               <== 父程序的 PID (会在后续章节才介绍)
PS1='[\u@\h \W]\$ '      <== PS1 就厉害了。这个是命令提示字符，也就是我们常见的
                             [root@www ~]# 或 [dmtsai ~]$ 的配置值，可以更动的！
PS2='> '                 <== 如果你使用跳脱符号 (\) 第二行以后的提示字符也
name=VBird               <== 刚刚配置的自定义变量也可以被列出来
$                        <== 目前这个 shell 所使用的 PID
?                        <== 刚刚运行完命令的回传值。
```
基本上，在 Linux 默认的情况中，使用{大写的字母}来配置的变量一般为系统内定需要的变量。
* PS1: (提示字符的配置)
  PS1 (数字1)，【命令提示字符】。当我们每次按下 [Enter] 按键去运行某个命令后，最后要再次出现提示字符时， 就会主动去读取这个变量值了。PS1 的特殊字符：
    * \d ：可显示出『星期 月 日』的日期格式，如："Mon Feb 2"
    * \H ：完整的主机名。举例来说，鸟哥的练习机为『www.vbird.tsai』
    * \h ：仅取主机名在第一个小数点之前的名字，如鸟哥主机则为『www』后面省略
    * \t ：显示时间，为 24 小时格式的『HH:MM:SS』
    * \T ：显示时间，为 12 小时格式的『HH:MM:SS』
    * \A ：显示时间，为 24 小时格式的『HH:MM』
    * \@ ：显示时间，为 12 小时格式的『am/pm』样式
    * \u ：目前使用者的账号名称，如『root』；
    * \v ：BASH 的版本信息，如鸟哥的测试主板本为 3.2.25(1)，仅取『3.2』显示
    * \w ：完整的工作目录名称，由根目录写起的目录名称。但家目录会以 ~ 取代；
    * \W ：利用 basename 函数取得工作目录名称，所以仅会列出最后一个目录名。
    * \# ：下达的第几个命令。
    * \$ ：提示字符，如果是 root 时，提示字符为 # ，否则就是 $ 
* $: (关于本 shell 的 PID)  
  代表【目前这个 shell 的线程代号】，亦即所谓的 PID (Process ID)。【echo $$】显示出来
* ?: (关于上个运行命令的回传值)  
  当我们运行某些命令时， 这些命令都会回传一个运行后的代码。一般来说，如果成功的运行该命令， 则会回传一个 0 值，如果运行过程发生错误，就会回传『错误代码』，一般就是以非为 0 的数值来取代。
* OSTYPE, HOSTTYPE, MACHTYPE: (主机硬件与核心的等级)  
  目前个人计算机的 CPU 主要分为 32/64 位，其中 32 位又可分为 i386, i586, i686，而 64 位则称为 x86_64。 由于不同等级的 CPU 命令集不太相同，因此你的软件可能会针对某些 CPU 进行优化，以求取较佳的软件性能。
### export: 自定义变量转成环境变量
环境变量与自定义变量的差异，『 该变量是否会被子程序所继续引用』。  
当你登陆 Linux 并取得一个 bash 之后，你的 bash 就是一个独立的程序，被称为 PID 的就是。 接下来你在这个 bash 底下所下达的任何命令都是由这个 bash 所衍生出来的，那些被下达的命令就被称为子程序了。如果在原本的 bash 底下运行另一个 bash，操作的环境接口就会跑到第二个 bash 中去，原本的 bash 会进入睡眠，若要回到原本的 bash 去，只有将第二个 bash 结束掉 (exit 或 logout)。  
```
export 变量名称
```
分享自己的变量配置给后来呼叫的文件或其他程序。主控文件后面呼叫其他附属文件(类似函数的功能)，附属文件可以使用主控文件的变量。  
export 后面没有接变量，会将所有的【环境变量】显示出来。

## 影响显示结果的语系变量 (locale)
```
locale -a
# 显示 Linux 底下支持的语系，这些语系文件都放在 /usr/lib/locale/ 目录中

与语系有关的变量数据
[root@www ~]# locale  <==后面不加任何选项与参数即可！
LANG=en_US                   <==主语言的环境
LC_CTYPE="en_US"             <==字符(文字)辨识的编码
LC_NUMERIC="en_US"           <==数字系统的显示信息
LC_TIME="en_US"              <==时间系统的显示数据
LC_COLLATE="en_US"           <==字符串的比较与排序等
LC_MONETARY="en_US"          <==币值格式的显示等
LC_MESSAGES="en_US"          <==信息显示的内容，如菜单、错误信息等
LC_ALL=                      <==整体语系的环境
```
如果其他的语系变量都未配置， 且你有配置 LANG 或者是 LC_ALL 时，则其他的语系变量就会被这两个变量所取代
## 变量的有效范围
环境变量为什么能够被子程序引用：
* 当启动一个 shell，操作系统会分配一记忆区块给 shell 使用，此内存内之变量可让子程序取用
* 若在父程序利用 export 功能，可以让自定义变量的内容写到上述的记忆区块当中(环境变量)；
* 当加载另一个 shell 时 (亦即启动子程序，而离开原本的父程序了)，子 shell 可以将父 shell 的环境变量所在的记忆区块导入自己的环境变量区块当中。
## 变量键盘读取、数组与宣告：read, array, declare
### read
要读取来自键盘输入的变量，就使用 read 这个命令。最常被用在 shell script 的撰写当中。
```
[root@www ~]# read [-pt] variable
选项与参数：
-p  ：后面可以接提示字符！
-t  ：后面可以接等待的『秒数！』

让用户由键盘输入一内容，将该内容变成名为 atest 的变量
[root@www ~]# read atest
This is a test        <==此时光标会等待你输入
[root@www ~]# echo $atest
This is a test          <==刚刚输入的数据已经变成一个变量内容

提示使用者 30 秒内输入自己的大名，将该输入字符串作为名为 named 的变量内容
[root@www ~]# read -p "Please keyin your name: " -t 30 named
Please keyin your name: VBird Tsai   <==会有提示字符
[root@www ~]# echo $named
VBird Tsai
```
### declare / typeset
declare 或 typeset 是一样的功能，就是在【宣告变量的类型】。如果 declare 后面没有接任何参数，那么 bash 就会主动将所有的变量名称与内容显示出来，类似 set。
```
[root@www ~]# declare [-aixr] variable
选项与参数：
-a  ：将后面名为 variable 的变量定义成为数组 (array) 类型
-i  ：将后面名为 variable 的变量定义成为整数数字 (integer) 类型
-x  ：用法与 export 一样，就是将后面的 variable 变成环境变量； # 将 - 变成 + 可以进行【取消】动作
-r  ：将变量配置成为 readonly 类型，该变量不可被更改内容，也不能 unset

-p  ：可以单独列出变量的类型
```
在默认情况下，bash 对于变量有几个基本的定义：
* 变量类型默认为【字符串】，所以若不指定变量类型，则 1+2 为一个【字符串】而不是【计算式】。
* bash 环境中的数值运算，默认最多仅能达到整数形态。

若果将变量配置为【只读】，通常要注销再登陆才能复原该变量的类型
### 数组 (array) 变量类型
数组的配置方式：`var[index]=content`。bash 提供的是一维数组。  
一般来说，建议直接以 ${数组} 的方式来读取

## 与文件系统及程序的限制关系：ulimit
bash 可以【限制用户的某些系统资源】，包括可以开启的文件数量，可以使用 CPU 时间，可以使用的内存总量等等。
```
[root@www ~]# ulimit [-SHacdfltu] [配额]
选项与参数：
-H  ：hard limit ，严格的配置，必定不能超过这个配置的数值；
-S  ：soft limit ，警告的配置，可以超过这个配置值，但是若超过则有警告信息。
      在配置上，通常 soft 会比 hard 小，举例来说，soft 可配置为 80 而 hard 
      配置为 100，那么你可以使用到 90 (因为没有超过 100)，但介于 80~100 之间时，
      系统会有警告信息通知你！
-a  ：后面不接任何选项与参数，可列出所有的限制额度；
-c  ：当某些程序发生错误时，系统可能会将该程序在内存中的信息写成文件(除错用)，
      这种文件就被称为核心文件(core file)。此为限制每个核心文件的最大容量。
-f  ：此 shell 可以创建的最大文件容量(一般可能配置为 2GB)单位为 Kbytes
-d  ：程序可使用的最大断裂内存(segment)容量；
-l  ：可用于锁定 (lock) 的内存量
-t  ：可使用的最大 CPU 时间 (单位为秒)
-u  ：单一用户可以使用的最大程序(process)数量。
```
想要复原 ulimit 的配置最简单的方法就是注销再登陆，否则就是得要重新以 ulimit 配置才行！ 不过，要注意的是，一般身份使用者如果以 ulimit 配置了 -f 的文件大小， 那么他『只能继续减小文件容量，不能添加文件容量』。
## 变量内容的删除、取代与替换
### 变量内容的删除与取代
```
先让小写的 path 自定义变量配置的与 PATH 内容相同
[root@www ~]# path=${PATH}
[root@www ~]# echo $path
/usr/kerberos/sbin:/usr/kerberos/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin

所以要将前两个目录删除掉
[root@www ~]# echo ${path#/*kerberos/bin:}
/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin

${variable#/*kerberos/bin:}    # ${}是关键词，必须存在，variable 是原本的变量，# 代表【从变量内容的最前面开始向右删除】，且仅删除最短的那个。/*kerberos/bin: 代表要删除的部分，由于 # 代表由前面开始删除，所以由 / 开始写起。可以通过通配符 * 代表 0 到无穷多个任意字符。
```
```
想要删除前面所有的目录，仅保留最后一个目录
[root@www ~]# echo ${path#/*:}
/usr/kerberos/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin
# 由于一个 # 仅删除掉最短的那个，因此他删除的情况，仅删除了第一个 /usr/kerberos/sbin:

[root@www ~]# echo ${path##/*:}
/root/bin
# 多加了一个 # 变成 ## 之后，他变成『删除掉最长的那个数据』！亦即是：删除前面的所有目录，仅剩最后一个(最后一个没有:)
```
* \# ：符合取代文字的『最短的』那一个；
* ##：符合取代文字的『最长的』那一个
```
想要删除最后面那个目录
[root@www ~]# echo ${path%:*bin}
/usr/kerberos/sbin:/usr/kerberos/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
# 这个 % 符号代表由最后面开始向前删除，删掉了 /root/bin

只要保留第一个目录呢
[root@www ~]# echo ${path%%:*bin}
/usr/kerberos/sbin
# 同样的， %% 代表的则是最长的符合字符串
```
取代：
```
将 path 的变量内容内的 sbin 取代成大写 SBIN：
[root@www ~]# echo ${path/sbin/SBIN}
/usr/kerberos/SBIN:/usr/kerberos/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin
# 关键词在于那两个斜线，两斜线中间的是旧字符串，后面的是新字符串。

[root@www ~]# echo ${path//sbin/SBIN}
/usr/kerberos/SBIN:/usr/kerberos/bin:/usr/local/SBIN:/usr/local/bin:/SBIN:/bin:/usr/SBIN:/usr/bin:/root/bin
# 如果是两条斜线，那么就变成所有符合的内容都会被取代
```
变量配置方式|说明
:---:|:---:
${变量#关键词}<br>${变量##关键词}|若变量内容从头开始的数据符合『关键词』，则将符合的最短数据删除<br>若变量内容从头开始的数据符合『关键词』，则将符合的最长数据删除
${变量%关键词}<br>${变量%%关键词}|若变量内容从尾向前的数据符合『关键词』，则将符合的最短数据删除<br>若变量内容从尾向前的数据符合『关键词』，则将符合的最长数据删除
${变量/旧字符串/新字符串}<br>${变量//旧字符串/新字符串}|若变量内容符合『旧字符串』则『第一个旧字符串会被新字符串取代』<br>若变量内容符合『旧字符串』则『全部的旧字符串会被新字符串取代』
### 变量的测试与内容替换
```
测试一下是否存在 username 这个变量，若不存在则给予 username 内容为 root
[root@www ~]# echo $username
           <==由于出现空白，所以 username 可能不存在，也可能是空字符串
[root@www ~]# username=${username-root}
[root@www ~]# echo $username
root       <==因为 username 没有配置，所以主动给予名为 root 的内容。
[root@www ~]# username="vbird tsai" <==主动配置 username 的内容
[root@www ~]# username=${username-root}
[root@www ~]# echo $username
vbird tsai <==因为 username 已经配置了，所以使用旧有的配置而不以 root 取代

重点在【-】后面接的关键词
new_var=${old_var-content}
# 新的变量，主要用来取代旧变量。新旧变量名称其实常常是一样的
# ${} 是关键词部分，必须要存在，old_var 旧的变量，被测试的项目，content 变量的【内容】，【给与未配置变量的内容】
```
```
若 username 未配置或为空字符串，则将 username 内容配置为 root
[root@www ~]# username=""
[root@www ~]# username=${username-root}
[root@www ~]# echo $username
      <==因为 username 被配置为空字符串了！所以当然还是保留为空字符串！
[root@www ~]# username=${username:-root}
[root@www ~]# echo $username
root  <==加上『 : 』后若变量内容为空或者是未配置，都能够以后面的内容替换！

# 加上冒号后，被测试的变量未被配置或者是已被配置为空字符串时， 都能够用后面的内容来替换与配置
# 一般来说，str: 代表【str 没配置成空的字符串时】；至于 str 则仅为 【没有该变量】。
```
变量配置方式|str 没有配置|str 为空字符串|str 已配置非为空字符串
:---:|:---:|:---:|:---:
var=${str-expr}|var=expr|var=|var=$str
var=${str:-expr}|var=expr|var=expr|var=$str
var=${str+expr}|var=|var=expr|var=expr
var=${str:+expr}|var=|var=|var=expr
var=${str=expr}|str=expr<br>var=expr|str 不变<br>var=|str 不变<br>var=$str
var=${str:=expr}|str=expr<br>var=expr|str=expr<br>var=expr|str 不变<br>var=$str
var=${str?expr}|expr 输出至 stderr|var=|var=$str
var=${str:?expr}|expr 输出至 stderr|expr 输出至 stderr|var=$str

# 命令别名与历史命令
## 命令别名配置：alias, unalias
『alias 的定义守则与变量定义守则几乎相同』，alias 命令别名的配置还可以取代既有的命令。  
直接使用 `alias` 后面不接任何内容，则是列出目前系统中存在的别名。  
取消命令别名，使用 `unalias 别名`
## 历史命令：history
```
[root@www ~]# history [n]
[root@www ~]# history [-c]
[root@www ~]# history [-raw] histfiles
选项与参数：
n   ：数字，意思是『要列出最近的 n 笔命令行表』的意思
-c  ：将目前的 shell 中的所有 history 内容全部消除
-a  ：将目前新增的 history 命令新增入 histfiles 中，若没有加 histfiles ，
      则默认写入 ~/.bash_history
-r  ：将 histfiles 的内容读到目前这个 shell 的 history 记忆中；
-w  ：将目前的 history 记忆内容写入 histfiles 中！

# history 后面不接参数，表示列出目前内存中所有 history 记忆
```
正常情况下，历史命令的读取与记录：
* 以 bash 登陆 Linux 主机之后，系统会主动由家目录的 ~/.bash_history 读取以前曾经下过的命令，那么 ~/.bash_history 会记录的数据，与 bash 的 HISTFILESIZE 这个变量配置值有关。
* 历史命令在注销时，会将最近的 HISTFILESIZE 笔记录到我的记录文件当中。
* 也可以用 history -w 强制立刻写入。 ~/.bash_historoy 记录的笔数永远都是 HISTFILESIZE ，旧的信息会被主动拿掉。仅保留最新的。

```
[root@www ~]# !number
[root@www ~]# !command
[root@www ~]# !!
选项与参数：
number  ：运行第几笔命令的意思；
command ：由最近的命令向前搜寻『命令串开头为 command』的那个命令，并运行；
!!      ：就是运行上一个命令(相当于按↑按键后，按 Enter)
```
使用 history 需注意安全问题，尤其是 root 的历史纪录文件，会记录很多重要的数据。
### 同一个账号多次登陆的 history 写入问题
注销的时候都会升级记录文件，所以最后记录的为最后一个注销的 bash 的记录。  
使用单一 bash 登陆，再用工作控制 (job control) 来切换不同工作，才能够将所有曾经下达过的命令都记录下来，方便未来系统管理员进行命令的 debug。
### 无法记录时间
历史命令无法记录命令下达的时间，仅有依序记录。可以透过 ~/.bash_logout 来进行 history 的记录，并加上 date 来添加时间参数。

# Bash shell 的操作环境
登陆的时候给予用户一些信息或者欢迎文字，载入习惯的环境变量、命令别名等。
## 路径与命令搜寻顺序
命令运行的顺序：
1. 以相对 / 绝对路径运行命令，例如【 /bin/ls 】或【 ./ls 】；
2. 由 alias 找到的命令来运行；
3. 由 bash 内建的 (builtin) 命令来运行；
4. 透过 $PATH 这个变量的顺序搜寻到的第一个命令来运行。

可以通过 `type -a 命令` 查询顺序
## bash 的进站与欢迎信息：/etc/issue, /etc/motd
issue 这个文件的内容也可以使用反斜杠作为变量取用。
|issue 内的各代码的意义|
|:---:|
|\d 本地端时间的日期；|
|\l 显示第几个终端机接口；|
|\m 显示硬件的等级 (i386/i486/i586/i686...)；|
|\n 显示主机的网络名称；|
|\o 显示 domain name；|
|\r 操作系统的版本 (相当于 uname -r)|
|\t 显示本地端时间的时间；|
|\s 操作系统的名称；|
|\v 操作系统的版本。|
除了 /etc/issue 之外还有个 /etc/issue.net ，提供给 telnet 这个远程登陆程序用的。当我们使用 telnet 远程连接到主机时，主机的登陆画面就会显示 /etc/issue.net。  
如果想要让使用者登陆后取得一些信息，可以将信息加入 /etc/motd 里。所有登陆者登陆时都会看到。
## bash 的环境配置文件
这些配置文件可以分为全体系统的配置文件亦即用户个人偏好配置文件。一般情况下，命令别名，自定义变量之类的，注销 bash 后就会失效，想要保留配置，要将这些配置写入配置文件才行。
### login 与 non-login shell
* login shell：取得 bash 时需要完整的登陆流程，就称为 login shell。
* non-login shell：取得 bash 接口的方法不需要重复登陆举动。举例来说，(1)以 X window 登陆 Linux 后， 再以 X 的图形化接口启动终端机，此时那个终端接口并没有需要再次的输入账号与密码，那个 bash 的环境就称为 non-login shell了。(2)你在原本的 bash 环境下再次下达 bash 这个命令，同样的也没有输入账号密码， 那第二个 bash (子程序) 也是 non-login shell 。

这两个取得 bash 的情况中，读取的配置文件数据并不一样。。一般来说，login shell 会读取以下两个配置文件：
1. etc/profile：这是系统整体的配置，最好不要修改
2. ~/.bash_profile 或 ~/.bash_login 或 ~/.profile：属于使用者个人的配置
### /etc/profile (login shell 才会读取)
这个配置文件可以利用使用者的标识符 (UID) 来决定很多重要的变量数据，这个也是每个使用者登陆取得 bash 时一定会读取的配置文件。要对所有使用者配置整体环境，则修改这个文件。这个文件配置的变量主要有：
* PATH：会依据 UID 决定 PATH 变量要不要含有 sbin 的系统命令目录；
* MAIL：依据账号配置好使用者的 mailbox 到 /var/spool/mail/账号名；
* USER：依据用户的账号配置此一变量的内容；
* HOSTNAME：依据主机的 hostname 命令决定此一变量内容；
* HISTSIZE：历史命令记录笔数。

/etc/profile 还会呼叫外部的配置数据，下面这些数据会依序被呼叫进来：
* /etc/inputrc  
  /etc/profile 会主动判断使用者有没有自定义输入的按键功能，如果没有的话，/etc/profile 就会决定配置【INPUTRC=/etc/inputrc】这个变量。此文件内容为 bash 的热键、[tab]是否有声音等等数据。
* /etc/profile.d/*.sh  
  这是个目录内的众多文件，只要在 /etc/profile.d/ 这个目录内且扩展名为 .sh ，另外，使用者能够具有 r 的权限， 那么该文件就会被 /etc/profile 呼叫进来。在 CentOS 5.x 中，这个目录底下的文件规范了 bash 操作接口的颜色、 语系、ll 与 ls 命令的命令别名、vi 的命令别名、which 的命令别名等等。如果你需要帮所有使用者配置一些共享的命令别名时， 可以在这个目录底下自行创建扩展名为 .sh 的文件，并将所需要的数据写入即可。
* /etc/sysconfig/il8n  
  这个文件是由 /etc/profile.d/lang.sh 呼叫进来的，这也是我们决定 bash 默认使用何种语系的重要配置文件，文件里最重要的就是 LANG 这个变量的配置。

bash 的 login shell 情况下所读取的整体环境配置文件其实只有 /etc/profile，其会呼叫出其他的配置文件。
### ~/.bash_profile (login shell 才会读)
bash 在读完了整体环境配置的 /etc/profile 并藉此呼叫其他配置文件后，接下来则是会读取使用者的个人配置文件。 在 login shell 的 bash 环境中，所读取的个人偏好配置文件其实主要有三个，依序分别是：
1. ~/.bash_profile
2. ~/.bash_login
3. ~/.profile

其实 bash 的 login shell 配置只会读取上面三个文件的其中一个，而读取的顺序则是依照上面的顺序。
```
[root@www ~]# cat ~/.bash_profile
# .bash_profile

# Get the aliases and functions
if [ -f ~/.bashrc ]; then   <==底下这三行在判断并读取 ~/.bashrc
        . ~/.bashrc  <== ~/.bash_profile 会再呼叫 ~/.bashrc
fi

# User specific environment and startup programs
PATH=$PATH:$HOME/bin        <==底下这几行在处理个人化配置
export PATH
unset USERNAME

# 累加的方式添加到用户目录下的 ~/bin/ 为额外的运行文件放置目录到 PATH 环境变量，可以将自己创建的运行档放置到自己家目录下的 ~/bin/ 目录。就能够直接运行，而不需要使用绝对/相对路径来运行。
```
最终被读取的配置文件是【~/.bashrc】这个文件，可以将自己的偏好配置写入该文件即可。
### source：读入环境配置文件的命令
由于 /etc/profile 与 ~/.bash_profile 都是在取得 login shell 的时候才会读取的配置文件，所以， 如果你将自己的偏好配置写入上述的文件后，通常都是得注销再登陆后，该配置才会生效。。若要直接读取配置文件而不注销登陆，利用 source 命令。
```
source 配置文件档名

将家目录的 ~/.bashrc 的配置读入目前的 bash 环境中
[root@www ~]# source ~/.bashrc  <==底下这两个命令是一样的！
[root@www ~]#  .  ~/.bashrc
```
一个工作环境分多种情况时，需要改变环境时，直接【source 变量文件】。
### ~/.bashrc (non=login shell 会读)
取得 non-login shell 时，该 bash 配置文件仅会读取 ~/.bashrc。
```
[root@www ~]# cat ~/.bashrc
# .bashrc

# User specific aliases and functions
alias rm='rm -i'             <==使用者的个人配置
alias cp='cp -i'
alias mv='mv -i'

# Source global definitions
if [ -f /etc/bashrc ]; then  <==整体的环境配置
        . /etc/bashrc
fi
```
/etc/bashrc 定义以下数据：
* 依据不同的 UID 规范出 umask 的值
* 依据不同的 UID 规范出提示字符 (就是 PS1 变量)
* 呼叫 /etc/profile.d/*.sh 的配置

这个 /etc/bashrc 是 CentOS 特有的 (其实是 Red Hat 系统特有的)，其他不同的 distributions 可能会放置在不同的档名。/etc/skel/.bashrc 是作为默认备份的 bashrc 文件
### 其他相关配置文件
* /etc/man.config  
  [规范了使用 man 的时候，man page 的路径去哪里寻找，数据的路径配置]  
  如果以 tarball 方式安装的数据，man page 可能会放置在 /usr/local/softpackage/man 里， softpackage 是的套件名称， 这个时候需要以手动的方式将该路径加到 /etc/man.config 里。  
  MANPATH 这个变量配置
* ~/.bash_history  
  记录历史命令。每次登陆后，bash 会先读取这个文件，将所有的历史命令载入内存。命令条数 HISTFILESIZE
* ~/.bash_logout  
  记录了【注销 bash 后，系统需要完成的动作】

## 终端机的环境配置：stty, set
stty (setting tty) 查阅目前的一些按键内容，也可以帮助配置终端机的输入按键代表意义：
```
stty [-a]
选项与参数：
-a：将目前所有的 stty 参数列出来。^ 代表 [Ctrl] 的意思
```
* eof   : End of file 的意思，代表『结束输入』。
* erase : 向后删除字符，
* intr  : 送出一个 interrupt (中断) 的讯号给目前正在 run 的程序；
* kill  : 删除在目前命令列上的所有文字；
* quit  : 送出一个 quit 的讯号给目前正在 run 的程序；
* start : 在某个程序停止后，重新启动他的 output
* stop  : 停止目前屏幕的输出；
* susp  : 送出一个 terminal stop 的讯号给正在 run 的程序。

set 除了可以显示变量之外，还可以配置整个命令输出/输入环境，例如记录历史命令，显示错误内容等
```
[root@www ~]# set [-uvCHhmBx]
选项与参数：
-u  ：默认不激活。若激活后，当使用未配置变量时，会显示错误信息；
-v  ：默认不激活。若激活后，在信息被输出前，会先显示信息的原始内容；
-x  ：默认不激活。若激活后，在命令被运行前，会显示命令内容(前面有 ++ 符号)
-h  ：默认激活。与历史命令有关；
-H  ：默认激活。与历史命令有关；
-m  ：默认激活。与工作管理有关；
-B  ：默认激活。与中括号 [] 的作用有关；
-C  ：默认不激活。若使用 > 等，则若文件存在时，该文件不会被覆盖。

显示目前所有的 set 配置值
[root@www ~]# echo $-
himBH
# 那个 $- 变量内容就是 set 的所有配置，bash 默认是 himBH 

配置 "若使用未定义变量时，则显示错误信息" 
[root@www ~]# set -u
[root@www ~]# echo $vbirding
-bash: vbirding: unbound variable
# 默认情况下，未配置/未宣告 的变量都会是『空的』，不过，若配置 -u 参数，
# 那么当使用未配置的变量时，就会有输出错误信息，很多的 shell 都默认激活 -u 参数。
# 若要取消这个参数，输入 set +u 即可

运行前，显示该命令内容。
[root@www ~]# set -x
[root@www ~]# echo $HOME
+ echo /root
/root
++ echo -ne '\033]0;root@www:~'
# 要输出的命令都会先被打印到屏幕上，前面会多出 + 的符号
```
在 /etc/inputrc 这个文件也可以配置其他按键的功能。
组合按键|运行结果
:---:|:---:
Ctrl + C|终止目前的命令
Ctrl + D|输入结束 (EOF)，例如邮件结束的时候；
Ctrl + M|就是 Enter 
Ctrl + S|暂停屏幕的输出
Ctrl + Q|恢复屏幕的输出
Ctrl + U|在提示字符下，将整列命令删除
Ctrl + Z|『暂停』目前的命令
## 通配符与特殊符号
常用通配符
符号|意义
:---:|:---:
*|代表『 0 个到无穷多个』任意字符
?|代表『一定有一个』任意字符
[ ]|同样代表『一定有一个在括号内』的字符(非任意字符)。例如 [abcd] 代表『一定有一个字符， 可能是 a, b, c, d 这四个任何一个』
[ - ]|若有减号在中括号内时，代表『在编码顺序内的所有字符』。例如 [0-9] 代表 0 到 9 之间的所有数字，因为数字的语系编码是连续的！
[^ ]|若中括号内的第一个字符为指数符号 (^) ，那表示『反向选择』，例如 [^abc] 代表 一定有一个字符，只要是非 a, b, c 的其他字符就接受的意思。
特殊符号

符号|内容
:----:|:----:
\#|批注符号：这个最常被使用在 script 当中，视为注释，在后的数据均不运行
\\|跳脱符号：将『特殊字符或通配符』还原成一般字符
\||管线 (pipe)：分隔两个管线命令的界定(后两节介绍)；
;|连续命令下达分隔符：连续性命令的界定 (注意！与管线命令并不相同)
~|用户的家目录
$|取用变量前导符：亦即是变量之前需要加的变量取代值
&|工作控制 (job control)：将命令变成背景下工作
!|逻辑运算意义上的『非』 not 的意思！
/|目录符号：路径分隔的符号
>, >>|数据流重导向：输出导向，分别是『取代』与『累加』
<, <<|数据流重导向：输入导向 (这两个留待下节介绍)
' '|单引号，不具有变量置换的功能
" "|具有变量置换的功能，引用的变量置换为其值
\` \`|两个『 \` 』中间为可以先运行的命令，亦可使用 $( )
( )|在中间为子 shell 的起始与结束
{ }|在中间为命令区块的组合！

# 数据流重导向
将某个命令运行后应该要出现在屏幕上的数据，传输到其他的地方，例如文件后者装置(例如打印机之类的)。
## 数据流重导向的概念
### standard output (标准输出) 与 standard error output (标准错误输出)
标准输出指的是『命令运行所回传的正确的信息』，而标准错误输出可理解为『 命令运行失败后，所回传的错误信息』。  
数据流重导向可以将 standard output (简称 stdout) 与 standard error output (简称 stderr) 分别传送到其他的文件或装置去，而分别传送所用的特殊字符则如下所示：
* 标准输入　　(stdin) ：代码为 0 ，使用 < 或 << ；
* 标准输出　　(stdout)：代码为 1 ，使用 > 或 >> ；
* 标准错误输出(stderr)：代码为 2 ，使用 2> 或 2>> ；

文件的创建方式：
1. 该文件若不存在，系统会自动的将他创建起来，但是
2. 当这个文件存在的时候，那么系统就会先将这个文件内容清空，然后再将数据写入！
3. 也就是若以 > 输出到一个已存在的文件中，那个文件就会被覆盖掉

如果想要累加数据而不是覆盖，则使用 >> 符号。
* 1> ：以覆盖的方法将『正确的数据』输出到指定的文件或装置上；
* 1>>：以累加的方法将『正确的数据』输出到指定的文件或装置上；
* 2> ：以覆盖的方法将『错误的数据』输出到指定的文件或装置上；
* 2>>：以累加的方法将『错误的数据』输出到指定的文件或装置上；
```
find /home -name .bashrc > list_right 2> list_error
# 搜索到的文件列表会输出到 list_right 文件中，而错误信息会输出到 list_error 中
```
### /dev/null 垃圾桶黑洞装置与特殊写法
这个 /dev/null 黑洞装置可以吃掉任何导向这个装置的额信息。将数据导向该装置即可。  
```
将命令的数据全部写入同一个文件中
find /home -name .bashrc > list 2> list <==错误，数据写入会混乱
find /home -name .bashrc > list 2>&l    <==正确
find /home -name .bashrc &> list        <==正确
```
### standard input：< 与 <<
< 符号将原本需要由键盘输入的数据，改由文件内容来取代。  
<< 则代表【结束的输入字符】
```
cat > catfile << "eof"
> This is a test.
> OK now stop.
> eof  <==输入这个关键词，立刻结束，而不需要输入 [ctrl]+d
```
使用命令输出重导向的场景：
* 屏幕输出的信息很重要，而且我们需要将他存下来的时候；
* 背景运行中的程序，不希望他干扰屏幕正常的输出结果时；
* 一些系统的例行命令 (例如写在 /etc/crontab 中的文件) 的运行结果，希望他可以存下来时；
* 一些运行命令的可能已知错误信息时，想以『 2> /dev/null 』将他丢掉时；
* 错误信息与正确信息需要分别输出时。
## 命令运行的判断依据：; , &&, ||
### cmd ; cmd (不考虑命令相关性的连续命令下达)
在命令与命令中间利用分号(;)来隔开，分号前的命令运行完后就会立刻接着运行后面的命令了。
### $?(命令回传值) 与 && 或 ||
两个命令之间有相依性，主要判断的地方在于前一个命令运行的结果是否正确。【若前一个命令运行的结果为正确，在 Linux 底下会回传一个 $? = 0 的值】，再由 【&&】 及 【||】 处理。
命令下达情况|说明
:---:|:---:
cmd1 && cmd2|1. 若 cmd1 运行完毕且正确运行($?=0)，则开始运行 cmd2。<br>2. 若 cmd1 运行完毕且为错误 ($?≠0)，则 cmd2 不运行。
cmd1 \|\| cmd2|1. 若 cmd1 运行完毕且正确运行($?=0)，则 cmd2 不运行。<br>2. 若 cmd1 运行完毕且为错误 ($?≠0)，则开始运行 cmd2。
```
ls /tmp/abc || mkdir /tmp/abc && touch /tmp/abc/test
# (1)若 /tmp/abc 不存在故回传 $?!=0，则(2)因为 || 遇到为非0 的 $? 故开始 mkdir 会成功进行
# 所以回传 $?=0 (3)因为 && 遇到 S?=0 故会运行 touch -。
# (1)若 /tmp/abc 存在故回传 $?=0，则(2)因为 || 遇到为0 的 $? 不会进行，$?=0 继续向后传
# 故 (3)因为 && 遇到 S?=0 故会运行 touch -。
```
命令是一个接一个运行的，使用 && 与 || 不能搞错命令的顺序。一般来说判断时有三个：  
`command1 && command2 || command3`  
而且顺序通常不会变，一般来说，command2 与 command3 会放置肯定可以运行成功的命令

# 管线命令 (pipe)
使用 | 这个界定符号。【|】仅能处理经由前面一个命令传来的正确信息，也就是 standard output 的信息，对于 standard error 并没有直接处理的能力。  
每个管线后面接的第一个数据必定是【命令】，而且这个命令必须能够接受 standard input 的数据才行，这样的命令才可以是【管线命令】。例如 less, more, head, tail 等都是可以接受 standard input 的管线命令啦。至于例如 ls, cp, mv 等就不是管线命令。
* 管线命令仅会处理 standard output，对于 standard error output 会予以忽略
* 管线命令必须要能够接受来自前一个命令的数据成为 standard input 继续处理才行。
## 撷取命令：cur, grep
将一段数据经过分析后，去除我们想要的。或者经由分析关键词，取得我们想要的那一行。一般来说，撷取信息通常针对【一行一行】来分析，并不是整篇信息分析的。
### cut
将一段信息的某一段【切】出来，处理信息以【行】为单位。
```
[root@www ~]# cut -d'分隔字符' -f fields <==用于有特定分隔字符
[root@www ~]# cut -c 字符区间            <==用于排列整齐的信息
选项与参数：
-d  ：后面接分隔字符。与 -f 一起使用；
-f  ：依据 -d 的分隔字符将一段信息分割成为数段，用 -f 取出第几段的意思；
-c  ：以字符 (characters) 的单位取出固定字符区间；

找出 PATH 的第3和5个路径
echo $PATH | cut =d ':' -f 3,5

取得 export 输出的信息每一行的第 12 字符以后的所有字符串
export | cut -c 12-
```
cut 主要的用途在于将【同一行里面的数据进行分解】最常用再分析一些数据或文字数据的时候。很多时候我们以某些字符当作分割的参数，将数据切割获得我们想要的数据。不过 cut 再处理多空格相连的数据时，会比较吃力。
### grep
分析一行信息，若当中有我们所需要的信息，就将该行拿出来
```
[root@www ~]# grep [-acinv] [--color=auto] '搜寻字符串' filename
选项与参数：
-a ：将 binary 文件以 text 文件的方式搜寻数据
-c ：计算找到 '搜寻字符串' 的次数
-i ：忽略大小写的不同，所以大小写视为相同
-n ：顺便输出行号
-v ：反向选择，亦即显示出没有 '搜寻字符串' 内容的那一行
--color=auto ：可以将找到的关键词部分加上颜色的显示

last | grep 'root' | cur -d ' ' -f 1
```
## 排序命令：sort, wc, uniq
### sort
排序，而且可以依据不同的数据形态来排序。
```
[root@www ~]# sort [-fbMnrtuk] [file or stdin]
选项与参数：
-f  ：忽略大小写的差异，例如 A 与 a 视为编码相同；
-b  ：忽略最前面的空格符部分；
-M  ：以月份的名字来排序，例如 JAN, DEC 等等的排序方法；
-n  ：使用『纯数字』进行排序(默认是以文字型态来排序的)；
-r  ：反向排序；
-u  ：就是 uniq ，相同的数据中，仅出现一行代表；
-t  ：分隔符，默认是用 [tab] 键来分隔；
-k  ：以那个区间 (field) 来进行排序的意思

/etc/passwd 内容以 : 来分割的，以第三栏排序
cat /etc/passwd | sort -t ':' -k 3
```
### uniq
将重复的数据仅列出一个显示，【重复的行删除掉只显示一个】
```
[root@www ~]# uniq [-ic]
选项与参数：
-i  ：忽略大小写字符的不同；
-c  ：进行计数

last | cut -d ' ' -f 1 | sort | uniq -c
```
### wc
```
[root@www ~]# wc [-lwm]
选项与参数：
-l  ：仅列出行；
-w  ：仅列出多少字(英文单字)；
-m  ：多少字符；

[root@www ~]# cat /etc/man.config | wc 
    141     722    4617
# 输出的三个数字中，分别代表： 『行、字数、字符数』
```
## 双向重导向：tee
tee 会同时数据流分送到文件与屏幕(screen)：而输出到屏幕的其实就是 stdout，可以让下一个命令继续处理。
```
[root@www ~]# tee [-a] file
选项与参数：
-a  ：以累加 (append) 的方式，将数据加入 file 当中！
```
## 字符转换命令：tr, col, join, paste, expand
### tr
tr 可以用来删除一段信息当中的文字，或者进行文字信息的替换
```
[root@www ~]# tr [-ds] SET1 ...
选项与参数：
-d  ：删除信息当中的 SET1 这个字符串；
-s  ：取代掉重复的字符！

将 last 输出的信息中，所有的小写变成大写字符：
last | tr '[a-z]' '[A-Z]'

去除 DOS 文件的断行符号 ^M
cat /root/passwd | tr -d '\r' > /root/passwd.linux
```
### col
```
[root@www ~]# col [-xb]
选项与参数：
-x  ：将 tab 键转换成对等的空格键
-b  ：在文字内有反斜杠 (/) 时，仅保留反斜杠最后接的那个字符

将 col 的 man page 转存成 /root/col.man 的纯文本
man col | col -b > /root/col.man
```
### join
主要处理【两个文件当中，有“相同数据”的那一行，才将他加在一起】
```
[root@www ~]# join [-ti12] file1 file2
选项与参数：
-t  ：join 默认以空格符分隔数据，并且比对『第一个字段』的数据，
      如果两个文件相同，则将两笔数据联成一行，且第一个字段放在第一个！
-i  ：忽略大小写的差异；
-1  ：这个是数字的 1 ，代表『第一个文件要用那个字段来分析』的意思；
-2  ：代表『第二个文件要用那个字段来分析』的意思。

用 root 的身份，将 /etc/passwd 与 /etc/shadow 相关数据整合成一栏
[root@www ~]# head -n 3 /etc/passwd /etc/shadow
==> /etc/passwd <==
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin

==> /etc/shadow <==
root:$1$/3AQpE5e$y9A/D0bh6rElAs:14120:0:99999:7:::
bin:*:14126:0:99999:7:::
daemon:*:14126:0:99999:7:::
# 由输出的数据可以发现这两个文件的最左边字段都是账号！且以 : 分隔

[root@www ~]# join -t ':' /etc/passwd /etc/shadow
root:x:0:0:root:/root:/bin/bash:$1$/3AQpE5e$y9A/D0bh6rElAs:14120:0:99999:7:::
bin:x:1:1:bin:/bin:/sbin/nologin:*:14126:0:99999:7:::
daemon:x:2:2:daemon:/sbin:/sbin/nologin:*:14126:0:99999:7:::
# 透过上面这个动作，我们可以将两个文件第一字段相同者整合成一行！
# 第二个文件的相同字段并不会显示(因为已经在第一行了)

我们知道 /etc/passwd 第四个字段是 GID ，那个 GID 记录在 
        /etc/group 当中的第三个字段，请问如何将两个文件整合？
[root@www ~]# head -n 3 /etc/passwd /etc/group
==> /etc/passwd <==
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin

==> /etc/group <==
root:x:0:root
bin:x:1:root,bin,daemon
daemon:x:2:root,bin,daemon
# 从上面可以看到，确实有相同的 GID passwd 为 ":" 分割的第四栏，group 为 ":" 分割的 第三栏

[root@www ~]# join -t ':' -1 4 /etc/passwd -2 3 /etc/group
0:root:x:0:root:/root:/bin/bash:root:x:root
1:bin:x:1:bin:/bin:/sbin/nologin:bin:x:root,bin,daemon
2:daemon:x:2:daemon:/sbin:/sbin/nologin:daemon:x:root,bin,daemon
# 同样的，相同的字段部分被移动到最前面了！所以第二个文件的内容就没再显示。
```
在使用 join 之前，所需的文件应该要事先经过排序 (sort) 处理，否则有些比对的项会被略过。
### paste
【直接将两行贴在一起，且中间以 [tab] 键隔开】
```
[root@www ~]# paste [-d] file1 file2
选项与参数：
-d  ：后面可以接分隔字符。默认是以 [tab] 来分隔的！
-   ：如果 file 部分写成 - ，表示来自 standard input 的数据的意思。

先将 /etc/group 读出(用 cat)，然后与 /etc/passwd /etc/shadow 贴在一起，且仅取出前三行
[root@www ~]# cat /etc/group|paste /etc/passwd /etc/shadow -|head -n 3
```
### expand
将 [tab] 按键转成空格键
```
[root@www ~]# expand [-t] file
选项与参数：
-t  ：后面可以接数字。一般来说，一个 tab 按键可以用 8 个空格键取代。
      我们也可以自行定义一个 [tab] 按键代表多少个字符
```
unexpand 将空白转成 [tab]
## 分割命令：split
可以将一个大文件，依据文件大小或行数来分割，就可以将大文件分割成小文件
```
[root@www ~]# split [-bl] file PREFIX
选项与参数：
-b  ：后面可接欲分割成的文件大小，可加单位，例如 b, k, m 等；
-l  ：以行数来进行分割。
PREFIX ：代表前导符的意思，可作为分割文件的前导文字。

# 小文件会以 xxxaa, xxxab, xxxac 等方式来创建

split -b 300k /etc/termcp termcp

将分割成的小文件合成一个文件
cat termcap* >> termcapback

使用 ls -al / 输出信息中，每十行记录成一个文件
ls -al / | split -l 10 - lsroot
# - 符号，如果需要 stdout / stdin 时，又没有文件，使用 - 就会被当成 stdin 或 stdout
```
## 参数代换：xargs
x 代表乘号，args 则是 arguments (参数) 的意思。xargs 就是产生某个命令的参数的意思。 xargs 可以读入 stdin 的数据，并且以空格符或断行符作为分辨，将 stdin 的数据分割成 arguments。如果一些档名或者其他意义的名词内含有空格符的时候，xargs 可能会误判。
```
[root@www ~]# xargs [-0epn] command
选项与参数：
-0  ：如果输入的 stdin 含有特殊字符，例如 `, \, 空格键等等字符时，这个 -0 参数
      可以将他还原成一般字符。这个参数可以用于特殊状态
-e  ：这个是 EOF (end of file) 的意思。后面可以接一个字符串，当 xargs 分析到
      这个字符串时，就会停止继续工作！
-p  ：在运行每个命令的 argument 时，都会询问使用者的意思；
-n  ：后面接次数，每次 command 命令运行时，要使用几个参数的意思。
当 xargs 后面没有接任何的命令时，默认是以 echo 来进行输出

将 /etc/passwd 内的第一栏去除，仅取三行，使用 finger 命令将账号内容显示出来
cut -d':' -f1 /etc/passwd | head -n 3 | xargs -p finger

find /sbin -perm +7000 | xargs ls -l
```
会使用 xargs 的原因是，很多命令并不支持管线命令，因此我们可以透过 xargs 来提供命令引用 standard input 。
## 关于减号 - 的用途
在管线命令中，常常会使用前一个命令的 stdout 作为这次的 stdin ，某些命令需要用到文件名来处理时，该 stdin 与 stdout 可以利用减号 “-” 来替代。
```
tar -cvf - /home | tar -xvf -
# 将 /home 里面的文件打包，但打包的数据不是记录到文件，而是传送到 stdout；经过管线后，将 tar -cvf - /home 传送给后面的 tar -xvf -。后面的这个 - 则是取用前一个命令的 stdout。
```