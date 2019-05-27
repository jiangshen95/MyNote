# Shell scripts
shell script 是利用 shell 的功能所写的一个『程序 (program)』，这个程序是使用纯文字档，将一些 shell 的语法与命令(含外部命令)写在里面， 搭配正规表示法、管线命令与数据流重导向等功能，以达到我们所想要的处理目的。  
shell script 更提供阵列、回圈、条件与逻辑判断等重要功能，让使用者也可以直接以 shell 来撰写程序  
shell script 可以简单的被看成是批量档， 也可以被说成是一个程序语言，且这个程序语言由於都是利用 shell 与相关工具命令， 所以不需要编译即可运行，且拥有不错的除错 (debug) 工具，所以，他可以帮助系统管理员快速的管理好主机。 
* 自动化管理的重要依据
* 追踪与管理系统的重要工作  
  Linux 系统的服务 (services) 启动介面是在 /etc/init.d/ 这个目录下，目录下的所有文件都是 scripts；另外，包括启动 (booting) 过程也都是利用 shell script 来帮忙搜寻系统的相关配置数据， 然后再代入各个服务的配置参数
* 简单入侵侦测功能  
  当我们的系统有异状时，大多会将这些异状记录在系统记录器，也就是我们常提到的『系统登录档』， 那么我们可以在固定的几分钟内主动的去分析系统登录档，若察觉有问题，就立刻通报管理员， 或者是立刻加强防火墙的配置守则，如此一来，你的主机可就能够达到『自我保护』。
* 连续命令单一化：  
  『汇整一些在 command line 下达的连续命令，将他写入 scripts 当中，而由直接运行 scripts 来启动一连串的 command line 命令输入！』
* 简易的数据处理：
* 跨平台支持与学习历程较短

shell script 用在系统管理上面是很好的一项工具，但是用在处理大量数值运算上， 就不够好了，因为 Shell scripts 的速度较慢，且使用的 CPU 资源较多，造成主机资源的分配不良。
## 第一支 script 的撰写与运行
1. 命令的运行是从上而下、从左而右的分析与运行；
2. 命令的下达就如同第五章内提到的： 命令、选项与参数间的多个空白都会被忽略掉；
3. 空白行也将被忽略掉，并且 [tab] 按键所推开的空白同样视为空白键；
4. 如果读取到一个 Enter 符号 (CR) ，就尝试开始运行该行 (或该串) 命令；
5. 如果一行的内容太多，则可以使用『 \[Enter] 』来延伸至下一行；
6. 『 # 』可做为注解！任何加在 # 后面的数据将全部被视为注解文字而被忽略。

运行 script 文件：
* 直接命令下达： shell.sh 文件必须要具备可读与可运行 (rx) 的权限，然后：
    * 绝对路径：使用 /home/dmtsai/shell.sh 来下达命令；
    * 相对路径：假设工作目录在 /home/dmtsai/ ，则使用 ./shell.sh 来运行
    * 变量『PATH』功能：将 shell.sh 放在 PATH 指定的目录内，例如： ~/bin/
* 以 bash 程序来运行：透过『 bash shell.sh 』或『 sh shell.sh 』来运行

可以利用 sh 的参数，如 -n 及 -x 来检查与追踪 shell.sh 的语法正确与否。
### 撰写第一支 script
1. 第一行 #!/bin/bash 在宣告这个 script 使用的 shell 名称：  
   因为我们使用的是 bash ，所以，必须要以『 #!/bin/bash 』来宣告这个文件内的语法使用 bash 的语法！那么当这个程序被运行时，他就能够加载 bash 的相关环境配置档 (一般来说就是 non-login shell 的 ~/.bashrc)， 并且运行 bash 来使我们底下的命令能够运行！这很重要的！(在很多状况中，如果没有配置好这一行， 那么该程序很可能会无法运行，因为系统可能无法判断该程序需要使用什么 shell 来运行)
2. 程序内容的说明  
   整个 script 当中，除了第一行的『 #! 』是用来宣告 shell 的之外，其他的 # 都是『注解』用途！ 所以上面的程序当中，第二行以下就是用来说明整个程序的基本数据。一般来说， 建议要养成说明该 script 的：1. 内容与功能； 2. 版本资讯； 3. 作者与联络方式； 4. 建档日期；5. 历史纪录 等等。这将有助于未来程序的改写与 debug
3. 主要环境变量的宣告：  
   建议务必要将一些重要的环境变量配置好，PATH 与 LANG (如果有使用到输出相关的资讯时) 是当中最重要的！ 如此一来，则可让我们这支程序在进行时，可以直接下达一些外部命令，而不必写绝对路径
4. 主要程序部分
5. 运行成果告知(定义回传值)  
   可以利用 exit 这个命令来让程序中断，并且回传一个数值给系统。exit 0 代表离开 script 并且回传一个 0 给系统， 所以我运行完这个 script 后，若接著下达 echo $? 则可得到 0 的值。
## 撰写 shell script 的良好习惯
在每个 script 的档头处记录
* script 的功能；
* script 的版本资讯；
* script 的作者与联络方式；
* script 的版权宣告方式；
* script 的 History (历史纪录)；
* script 内较特殊的命令，使用『绝对路径』的方式来下达；
* script 运行时需要的环境变量预先宣告与配置。
* 较为特殊的程序码部分，最好加上注释说明

# 简单的 shell script 练习
## script 的运行方式差异 (source, sh script, ./script)
### 利用直接运行的方式来运行 script
直接命令下达 (不论是绝对路径/相对路径还是 $PATH 内)，或者是利用 bash (或 sh) 来下达脚本时， 该 script 都会使用一个新的 bash 环境来运行脚本内的命令！也就是说，使用者种运行方式时， 其实 script 是在子程序的 bash 内运行的。『当子程序完成后，在子程序内的各项变量或动作将会结束而不会传回到父程序中』。
### 利用 source 来运行脚本：在父程序中运行

# 判断式
## 利用 test 命令的测试功能
当要检测系统上面某些文件或者相关的属性时，可以利用 test 命令。
```
检查一个文件是否存在
test -e /dmtsai && echo "exist" || echo "Not exist"
```
还有以下判断标志
<table>
<tbody><tr><th>测试的标志</th><th>代表意义</th></tr>
<tr><td colspan="2"><b>1. 关于某个档名的『文件类型』判断，如 test -e filename 表示存在否</b></td></tr>
<tr><td align="center">-e</td><td>该『档名』是否存在？(常用)</td></tr>
<tr><td align="center">-f</td><td>该『档名』是否存在且为文件(file)？(常用)</td></tr>
<tr><td align="center">-d</td><td>该『档名』是否存在且为目录(directory)？(常用)</td></tr>
<tr><td align="center">-b</td><td>该『档名』是否存在且为一个 block device 装置？</td></tr>
<tr><td align="center">-c</td><td>该『档名』是否存在且为一个 character device 装置？</td></tr>
<tr><td align="center">-S</td><td>该『档名』是否存在且为一个 Socket 文件？</td></tr>
<tr><td align="center">-p</td><td>该『档名』是否存在且为一个 FIFO (pipe) 文件？</td></tr>
<tr><td align="center">-L</td><td>该『档名』是否存在且为一个连结档？</td></tr>
<tr><td colspan="2"><b>2. 关于文件的权限侦测，如 test -r filename 表示可读否 (但 root 权限常有例外)</b></td></tr>
<tr><td align="center">-r</td><td>侦测该档名是否存在且具有『可读』的权限？</td></tr>
<tr><td align="center">-w</td><td>侦测该档名是否存在且具有『可写』的权限？</td></tr>
<tr><td align="center">-x</td><td>侦测该档名是否存在且具有『可运行』的权限？</td></tr>
<tr><td align="center">-u</td><td>侦测该档名是否存在且具有『SUID』的属性？</td></tr>
<tr><td align="center">-g</td><td>侦测该档名是否存在且具有『SGID』的属性？</td></tr>
<tr><td align="center">-k</td><td>侦测该档名是否存在且具有『Sticky bit』的属性？</td></tr>
<tr><td align="center">-s</td><td>侦测该档名是否存在且为『非空白文件』？</td></tr>
<tr><td colspan="2"><b>3. 两个文件之间的比较，如： test file1 -nt file2</b></td></tr>
<tr><td align="center">-nt</td><td>(newer than)判断 file1 是否比 file2 新</td></tr>
<tr><td align="center">-ot</td><td>(older than)判断 file1 是否比 file2 旧</td></tr>
<tr><td align="center">-ef</td><td>判断 file1 与 file2 是否为同一文件，可用在判断 hard link 的判定上。主要意义在判定，两个文件是否均指向同一个 inode </td></tr>
<tr><td colspan="2"><b>4. 关于两个整数之间的判定，例如 test n1 -eq n2</b></td></tr>
<tr><td align="center">-eq</td><td>两数值相等 (equal)</td></tr>
<tr><td align="center">-ne</td><td>两数值不等 (not equal)</td></tr>
<tr><td align="center">-gt</td><td>n1 大于 n2 (greater than)</td></tr>
<tr><td align="center">-lt</td><td>n1 小于 n2 (less than)</td></tr>
<tr><td align="center">-ge</td><td>n1 大于等于 n2 (greater than or equal)</td></tr>
<tr><td align="center">-le</td><td>n1 小于等于 n2 (less than or equal)</td></tr>
<tr><td colspan="2"><b>5. 判定字串的数据</b></td></tr>
<tr><td align="center">test -z string</td><td>判定字串是否为 0 ？若 string 为空字串，则为 true</td></tr>
<tr><td align="center">test -n string</td><td>判定字串是否非为 0 ？若 string 为空字串，则为 false。<br>注： -n 亦可省略</td></tr>
<tr><td align="center">test str1 = str2</td><td>判定 str1 是否等於 str2 ，若相等，则回传 true</td></tr>
<tr><td align="center">test str1 != str2</td><td>判定 str1 是否不等於 str2 ，若相等，则回传 false</td></tr>
<tr><td colspan="2"><b>6. 多重条件判定，例如： test -r filename -a -x filename</b></td></tr>
<tr><td align="center">-a</td><td>(and)两状况同时成立！例如 test -r file -a -x file，则 file 同时具有 r 与 x 权限时，才回传 true。</td></tr>
<tr><td align="center">-o</td><td>(or)两状况任何一个成立！例如 test -r file -o -x file，则 file 具有 r 或 x 权限时，就可回传 true。</td></tr>
<tr><td align="center">!</td><td>反相状态，如 test ! -x file ，当 file 不具有 x 时，回传 true</td></tr>
</tbody></table>

## 利用判断符号 [  ]
```
判断 $HOME 变量是否为空
[ -z "$HOME" ];echo $?
```
中括号用作 shell 的判断式时，必须要注意中括号的两端需要有空白字节来分隔。
* 在中括号 [] 内的每个组件都需要有空白键来分隔；
* 在中括号内的变量，最好都以双引号括号起来；
* 在中括号内的常数，最好都以单或双引号括号起来。

中括号的用法与 test 几乎一样，中括号比较常用在条件判断是 if...then...fi 中。
## Shell script 的默认变量($0, $1...)
script 针对参数已经有配置好一些变量名称了。对应如下。
```
/path/to/scriptname  opt1  opt2  opt3  opt4 
       $0             $1    $2    $3    $4
```
除了这些数字的变量之外，我们还有一些特殊的变量，可以在 script 内使用来呼叫这些参数。
* $# ：代表后接的参数『个数』，以上表为例这里显示为『 4 』；
* $@ ：代表『 "$1" "$2" "$3" "$4" 』之意，每个变量是独立的(用双引号括起来)；
* $* ：代表『 "$1c$2c$3c$4" 』，其中 c 为分隔字节，默认为空白键， 所以本例中代表『 "$1 $2 $3 $4" 』之意。
### shift: 造成参数变量号码偏移
shift 会移动变量，而且 shift 后面可以接数字，代表拿掉最前面的几个参数的意思

# 条件判断式
## 利用 if ... then
### 单层、简单条件判断式
```
if [ 条件判断式 ]; then
	当条件判断式成立时，可以进行的命令工作内容；
fi   <==将 if 反过来写，结束 if 之意！
```
当有多个条件需要判别时，可以用多个中括号隔开，括号与括号之间以 && 或 || 来隔开。
* && 代表 AND;
* || 代表 or;
### 多重、复杂条件判断式
```
# 一个条件判断，分成功进行与失败进行 (else)
if [ 条件判断式 ]; then
	当条件判断式成立时，可以进行的命令工作内容；
else
	当条件判断式不成立时，可以进行的命令工作内容；
fi
```
```
# 多个条件判断 (if ... elif ... elif ... else) 分多种不同情况运行
if [ 条件判断式一 ]; then
	当条件判断式一成立时，可以进行的命令工作内容；
elif [ 条件判断式二 ]; then
	当条件判断式二成立时，可以进行的命令工作内容；
else
	当条件判断式一与二均不成立时，可以进行的命令工作内容；
fi
```
【netstat -tuln】来取得目前主机有启动的服务
```
[root@www ~]# netstat -tuln
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address     Foreign Address   State
tcp        0      0 0.0.0.0:111       0.0.0.0:*         LISTEN
tcp        0      0 127.0.0.1:631     0.0.0.0:*         LISTEN
tcp        0      0 127.0.0.1:25      0.0.0.0:*         LISTEN
tcp        0      0 :::22             :::*              LISTEN
udp        0      0 0.0.0.0:111       0.0.0.0:*
udp        0      0 0.0.0.0:631       0.0.0.0:*
#封包格式           本地IP:端口       远程IP:端口       是否监听
```
重点是『Local Address (本地主机的IP与端口对应)』那个栏位，他代表的是本机所启动的网络服务。若为 127.0.0.1 则是仅针对本机开放，若是 0.0.0.0 或 ::: 则代表对整个 Internet 开放。每个端口 (port) 都有其特定的网络服务，几个常见的 port 与相关网络服务的关系是：
* 80：WWW
* 22: ssh
* 21: ftp
* 25: mail
* 111: RPC(远程程序呼叫)
* 631: CPUS(列印服务功能)

## 利用 case ... esac 判断
```
case  $变量名称 in   <==关键字为 case ，还有变量前有钱字号
  "第一个变量内容")   <==每个变量内容建议用双引号括起来，关键字则为小括号 )
	程序段
	;;            <==每个类别结尾使用两个连续的分号来处理！
  "第二个变量内容")
	程序段
	;;
  *)                  <==最后一个变量内容都会用 * 来代表所有其他值
	不包含第一个变量内容与第二个变量内容的其他程序运行段
	exit 1
	;;
esac                  <==最终的 case 结尾！『反过来写』
```
系统中很多服务的启动 scripts 都是使用这种写法。  
一般来说，使用『 case $变量 in 』这个语法中，当中的那个『 $变量 』大致有两种取得的方式：
* 直接下达式：例如上面提到的，利用『 script.sh variable 』 的方式来直接给予 $1 这个变量的内容，这也是在 /etc/init.d 目录下大多数程序的设计方式。
* 互动式：透过 read 这个命令来让使用者输入变量的内容。
## 利用 function 功能
函数可以在 shell script 当中做出一个类似自定运行命令。
```
function fname() {
	程序段
}
```
因为 shell script 的运行方式是由上而下，由左而右， 因此在 shell script 当中的 function 的配置一定要在程序的最前面，这样才能够在运行时被找到可用的程序段。  
function 拥有内建变量，他的内建变量与 shell script 很类似，函数名称代表示 $0，而后续接的变量也是以 $1, $2 ... 来取代的

# 回圈 (loop)/循环
回圈可以不断的运行某个程序段落，直到使用者配置的条件达成为止。除了依据判断式达成与否的不定回圈之外，还有一种固定跑多少次的固定回圈形态。
## while don done, until do done (不定回圈)
```
while [ condition ]  <==中括号内的状态就是判断式
do            <==do 是回圈的开始！
	程序段落
done          <==done 是回圈的结束
```
『当 condition 条件成立时，就进行回圈，直到 condition 的条件不成立才停止』
```
until [ condition ]
do
	程序段落
done
```
与 while 相反，『当 condition 条件成立时，就终止回圈， 否则就持续进行回圈的程序段。』
## for...do...done (固定回圈)
知道要进行几次循环
```
for var in con1 con2 con3 ...
do
	程序段
done

for sitenu in $(seq 1 100) # seq 为 sequence(连续) 的缩写之意
```
## for...do...done 的数值处理
除了上述方法之外，for 循环还有另一种写法
```
for (( 初始值; 限制值; 运行步阶 ))
do
	程序段
done
```
* 初始值：某个变量在回圈当中的起始值，直接以类似 i=1 配置好；
* 限制值：当变量的值在这个限制值的范围内，就继续进行回圈。例如 i<=100；
* 运行步阶：每作一次回圈时，变量的变化量。例如 i=i+1。如果每次添加 1 ，则可以使用类似『i++』的方式

# shell script 的追踪与 debug
利用 bash 的参数进行 debug
```
[root@www ~]# sh [-nvx] scripts.sh
选项与参数：
-n  ：不要运行 script，仅查询语法的问题；
-v  ：在运行 sccript 前，先将 scripts 的内容输出到屏幕上；
-x  ：将使用到的 script 内容显示到屏幕上。可以将运行过程全部列出来。

sh -x sh15.sh
# 输出的信息中，在加号后面的数据其实都是命令串，由于 sh -x 的方式来将命令运行过程也显示出来， 如此使用者可以判断程序码运行到哪一段时会出现相关的资讯
```