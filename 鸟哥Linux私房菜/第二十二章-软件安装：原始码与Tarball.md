# 开放源码的软件安装于升级简介
## 开放源码、编译器与可运行档
在 Linux 系统上面，一个文件能不能被运行看的是有没有可运行的那个权限 (具有 x permission)，不过 Linux 系统上真正认识的可运行档其实是二进位文件 ( binary program)。bash 本身也是一支二进位程序。
```
[root@www ~]# file /bin/bash
/bin/bash: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), for GNU/
Linux 2.6.9, dynamically linked (uses shared libs), for GNU/Linux 2.6.9, stripped

[root@www ~]# file /etc/init.d/syslog
/etc/init.d/syslog: Bourne-Again shell script text executable
```
如果是 binary 而且是可以运行的时候，他就会显示运行档类别 (ELF 32-bit LSB executable)， 同时会说明是否使用动态函式库 (shared libs)，而如果是一般的 script ，那他就会显示出 text executables 之类的字样。
> syslog 的数据显示出 Bourne-Again ... 那一行，是因为 scripts 上面第一行有宣告 #!/bin/bash，如果没有该宣告，那么不管 /etc/init.d/syslog 的权限为何，他其实显示的是 ASCII 文字档的资讯。

在编译的过程当中还会产生所谓的目标档 (Object file)，这些文件是以 *.o 的扩展名样式存在的。C 语言的原始码文件通常以 *.c 作为扩展名。此外，有的时候，我们会在程序当中『引用、呼叫』 其他的外部副程序，或者是利用其他软件提供的『函数功能』，这个时候，我们就必须要在编译的过程当中， 将该函式库给他加进去，如此一来，编译器就可以将所有的程序码与函式库作一个连结 (Link) 以产生正确的运行档。
* 开放源码：就是程序码，写给人类看的程序语言，但机器并不认识，所以无法运行；
* 编译器：将程序码转译成为机器看的懂得语言，就类似翻译者的角色；
* 可运行档：经过编译器变成二进位程序后，机器看的懂所以可以运行的文件。
## 函数库
函式库又分为动态与静态函式库。事实上， Linux 的核心提供很多的核心相关函式库与外部参数， 这些核心功能在设计硬件的驱动程序的时候很有用，这些核心相关资讯大多放置在 /usr/include, /lib, /usr/lib 里面。
* 函式库：就类似副程序的角色，可以被呼叫来运行的一段功能函数。
## make 与 configure
当运行 make 时，make 会在当时的目录下搜寻 Makefile (or makefile) 这个文字档，而 Makefile 里面则记录了原始码如何编译的详细资讯！ make 会自动的判别原始码是否经过变动了，而自动升级运行档，是软件工程师相当好用的一个辅助工具。  
通常软件开发商都会写一支侦测程序来侦测使用者的作业环境， 以及该作业环境是否有软件开发商所需要的其他功能，该侦测程序侦测完毕后，就会主动的创建这个 Makefile 的守则文件。通常这支侦测程序的档名为 configure 或者是 config 。  
般来说，侦测程序会侦测的数据大约有底下这些：
* 是否有适合的编译器可以编译本软件的程序码；
* 是否已经存在本软件所需要的函式库，或其他需要的相依软件；
* 操作系统平台是否适合本软件，包括 Linux 的核心版本；
* 核心的表头定义档 (header include) 是否存在 (驱动程序必须要的侦测)。

由于不同的 Linux distribution 的函式库文件所放置的路径，或者是函式库的档名订定， 或者是默认安装的编译器，以及核心的版本都不相同，因此理论上，无法在 CentOS 5.x 上面编译出 binary program 后，到 SuSE 上面运行。因为呼叫的目标函式库位置可能不同 ， 核心版本更不可能相同。所以同一套软件要在不同的平台上面运行时， 必须要重复编译。
## Tarball 的软件
所谓的 Tarball 文件，其实就是将软件的所有原始码文件先以 tar 打包，然后再以压缩技术来压缩，通常最常见的就是以 gzip 来压缩了。因为利用了 tar 与 gzip 的功能，所以 tarball 文件一般的扩展名就会写成 *.tar.gz 或者是简写为 *.tgz 。近来由于 bzip2 的压缩率较佳，所以 Tarball 渐渐的以 bzip2 的压缩技术来取代 gzip ，因此档名也会变成 *.tar.bz2 。Tarball 是一个软件包， 你将他解压缩之后，里面的文件通常就会有：
* 原始程序码文件；
* 侦测程序文件 (可能是 configure 或 config 等档名)；
* 本软件的简易说明与安装说明 (INSTALL 或 README)。
## 如何安装与升级
升级的原因：
* 需要新的功能，但旧有主机的旧版软件并没有，所以需要升级到新版的软件；
* 旧版本的软件上面可能有资安上的顾虑，所以需要升级到新版的软件；
* 旧版的软件运行效能不彰，或者运行的能力不能让管理者满足。

升级的方法：
* 直接以原始码透过编译来安装与升级；
* 直接以编译好的 binary program 来安装与升级。

预先编译好程序的机制，包括有 Red Hat 系统 (含 Fedora/CentOS 系列) 发展的 RPM 软件管理机制与 yum 线上升级模式； Debian 使用的 dpkg 软件管理机制与 APT 线上升级模式等等。由于 CentOS 系统是依循标准的 Linux distribution，所以可以使用 Tarball 直接进行编译的安装与升级， 当然也可以使用 RPM 相关的机制来进行安装与升级。
Tarball 的安装：
1. 将 Tarball 由厂商的网页下载下来；
2. 将 Tarball 解开，产生很多的原始码文件；
3. 开始以 gcc 进行原始码的编译 (会产生目标档 object files)；
4. 然后以 gcc 进行函式库、主、副程序的连结，以形成主要的 binary file；
5. 将上述的 binary file 以及相关的配置档安装至自己的主机上面。

3，4 步骤，可以通过 make 命令简化。

# 使用传统程序语言进行编译的简单范例
```
[root@www ~]# gcc hello.c
-rwxr-xr-x 1 root root 4725 Jun  5 02:41 a.out   <==此时会产生这个档名

[root@www ~]# gcc -c hello.c
-rw-r--r-- 1 root root 868 Jun  5 02:44 hello.o  <==就是被产生的目标档

[root@www ~]# gcc -o hello hello.o
-rwxr-xr-x 1 root root 4725 Jun  5 02:47 hello  <==这就是可运行档！ -o 的结果
-rw-r--r-- 1 root root   72 Jun  5 02:40 hello.c
-rw-r--r-- 1 root root  868 Jun  5 02:44 hello.o
```
## 主、副程序连结：副程序的编译
由于我们的原始码文件有时并非仅只有一个文件，所以我们无法直接进行编译。 这个时候就需要先产生目标档，然后再以连结制作成为 binary 可运行档。另外，如果有一天，你升级了 thanks_2.c 这个文件的内容，则你只要重新编译 thanks_2.c 来产生新的 thanks_2.o ，然后再以连结制作出新的 binary 可运行档即可。而不必重新编译其他没有更动过的原始码文件。
```
[root@www ~]# gcc -O -c thanks.c thanks_2.c  <== -O 为产生最佳化的参数

[root@www ~]# gcc -Wall -c thanks.c thanks_2.c  <== -Wall 为产生更详细的编译过程资讯。
```
## 呼叫外部函数库：加入连结的函数库
```
[root@www ~]# gcc sin.c -lm -L/lib -L/usr/lib  <==重点在 -lm 
```
* -l ：是『加入某个函式库(library)』的意思，
*  m ：则是 libm.so 这个函式库，其中， lib 与扩展名(.a 或 .so)不需要写

-L 后面接的路径表示：『函式库 libm.so 到 /lib 或 /usr/lib 里面搜寻』。由于 Linux 默认是将函式库放置在 /lib 与 /usr/lib 当中，所以没有写 -L/lib 与 -L/usr/lib 也没有关系。不过，如果使用的函式库并非放置在这两个目录下，那么 -L/path 就必须要写。  
可以使用底下的方式来定义出要读取的 include 文件放置的目录 (默认值是放置在 /usr/include 下)：
```
[root@www ~]# gcc sin.c -lm -I/usr/include
```
-I/path 后面接的路径( Path )就是配置要去搜寻相关的 include 文件的目录。
## gcc 的简易用法 (编译、参数与链结)
```
# 仅将原始码编译成为目标档，并不制作连结等功能：
[root@www ~]# gcc -c hello.c
# 会自动的产生 hello.o 这个文件，但是并不会产生 binary 运行档。

# 在编译的时候，依据作业环境给予最佳化运行速度
[root@www ~]# gcc -O hello.c -c
# 会自动的产生 hello.o 这个文件，并且进行最佳化

# 在进行 binary file 制作时，将连结的函式库与相关的路径填入
[root@www ~]# gcc sin.c -lm -L/usr/lib -I/usr/include
# 这个命令较常下达在最终连结成 binary file 的时候，
# -lm 指的是 libm.so 或 libm.a 这个函式库文件；
# -L 后面接的路径是刚刚上面那个函式库的搜寻目录；
# -I 后面接的是原始码内的 include 文件之所在目录。

# 将编译的结果输出成某个特定档名
[root@www ~]# gcc -o hello hello.c
# -o 后面接的是要输出的 binary file 档名

# 在编译的时候，输出较多的信息说明
[root@www ~]# gcc -o hello hello.c -Wall
# 加入 -Wall 之后，程序的编译会变的较为严谨一点，
# 所以警告信息也会显示出来！
```
另外，我们通常称 -Wall 或者 -O 这些非必要的参数为旗标 (FLAGS)，因为我们使用的是 C 程序语言，所以有时候也会简称这些旗标为 CFLAGS ，这些变量偶尔会被使用。

# 用 make 进行巨集编译
* 简化编译时所需要下达的命令；
* 若在编译完成之后，修改了某个原始码文件，则 make 仅会针对被修改了的文件进行编译，其他的 object file 不会被更动；
* 最后可以依照相依性来升级 (update) 运行档。
## makefile 的基本语法与变量
```
标的(target): 目标档1 目标档2
<tab>   gcc -o 欲创建的运行档 目标档1 目标档2
```
那个标的 (target) 就是我们想要创建的资讯，而目标档就是具有相关性的 object files ，那创建运行档的语法就是以 \<tab\> 按键开头的那一行。『命令列必须要以 tab 按键作为开头』。基本守则：
* 在 makefile 当中的 # 代表注解；
* \<tab\> 需要在命令行 (例如 gcc 这个编译器命令) 的第一个字节；
* 标的 (target) 与相依文件(就是目标档)之间需以『:』隔开。
```
# 1. 先编辑 makefile 来创建新的守则，此守则的标的名称为 clean ：
[root@www ~]# vi makefile
main: main.o haha.o sin_value.o cos_value.o
	gcc -o main main.o haha.o sin_value.o cos_value.o -lm
clean:
	rm -f main main.o haha.o sin_value.o cos_value.o

# 2. 以新的标的 (clean) 测试看看运行 make 的结果：
[root@www ~]# make clean  <==就是这里！透过 make 以 clean 为标的
rm -rf main main.o haha.o sin_value.o cos_value.o
```
makefile 里面就具有至少两个标的，分别是 main 与 clean ，如果我们想要创建 main 的话，输入『make main』，如果想要清除有的没的，输入『make clean』即可。先清除目标档再编译 main 这个程序，『make clean main』。
```
[root@www ~]# vi makefile
LIBS = -lm
OBJS = main.o haha.o sin_value.o cos_value.o
main: ${OBJS}
        gcc -o main ${OBJS} ${LIBS}
clean:
        rm -f main ${OBJS}
```
变量的基本用法为：
1. 变量与变量内容以『=』隔开，同时两边可以具有空格；
2. 变量左边不可以有 \<tab\> ，例如上面范例的第一行 LIBS 左边不可以是 \<tab\>；
3. 变量与变量内容在『=』两边不能具有『:』；
4. 在习惯上，变量最好是以『大写字母』为主；
5. 运用变量时，以 ${变量} 或 $(变量) 使用；
6. 在该 shell 的环境变量是可以被套用的，例如提到的 CFLAGS 这个变量！
7. 在命令列模式也可以给予变量。

由于 gcc 在进行编译的行为时，会主动的去读取 CFLAGS 这个环境变量，所以，你可以直接在 shell 定义出这个环境变量，也可以在 makefile 文件里面去定义，更可以在命令列当中给予。
```
[root@www ~]# CFLAGS="-Wall" make clean main
# 这个动作在上 make 进行编译时，会去取用 CFLAGS 的变量内容！
```
环境变量的取用守则：
1. make 命令列后面加上的环境变量为优先；
2. makefile 里面指定的环境变量第二；
3. shell 原本具有的环境变量第三。

特殊的变量：
* $@：代表目前的标的(target)
```
[root@www ~]# vi makefile
LIBS = -lm
OBJS = main.o haha.o sin_value.o cos_value.o
CFLAGS = -Wall
main: ${OBJS}
	gcc -o $@ ${OBJS} ${LIBS}   <== $@ 就是 main
clean:
	rm -f main ${OBJS}
```

# Tarball 的管理与建议
Tarball 的安装是可以跨平台的，因为 C 语言的程序码在各个平台上面是可以共通的， 只是需要的编译器可能并不相同而已。通过修改小部分的代码，就可以进行跨平台的移植了。在 Linux 底下写的程序『理论上，是可以在 Windows 上面编译的！』。
## 使用原始码管理软件所需要的基础软件
* gcc 或 cc 等 C 语言编译器 (compiler)：  
  Linux 上面有众多的编译器，其中以 GNU 的 gcc 是首选的自由软件编译器。事实上很多在 Linux 平台上面发展的软件的原始码，原本就是以 gcc 为底来设计的呢。
* make 及 autoconfig 等软件：  
  一般来说，以 Tarball 方式释出的软件当中，为了简化编译的流程，通常都是配合前几个小节提到的 make 这个命令来依据目标文件的相依性而进行编译。但是我们也知道说 make 需要 makefile 这个文件的守则，那由于不同的系统里面可能具有的基础软件环境并不相同， 所以就需要侦测使用者的作业环境，好自行创建一个 makefile 文件。这个自行侦测的小程序也必须要藉由 autoconfig 这个相关的软件来辅助才行。
* 需要 Kernel 提供的 Library 以及相关的 Include 文件：  
  include 文件的存在。很多的软件在发展的时候都是直接取用系统核心提供的函式库与 include 文件的，这样才可以与这个操作系统兼容。尤其是在『驱动程序方面的模块 』，例如网络卡、声卡、USB 等驱动程序在安装的时候，常常是需要核心提供的相关资讯的。在 Red Hat 的系统当中 (包含 Fedora/CentOS 等系列) ，这个核心相关的功能通常都是被包含在 kernel-source 或 kernel-header 这些软件名称当中，所以记得要安装这些软件。
 
Linux distribution 中安装 Development Tools 以及 Kernel Source Development 等相关字眼的软件群集。  
通过 yum 的软件群组安装功能：
* 如果是要安装 gcc 等软件发展工具，请使用『 yum groupinstall "Development Tools" 』
* 若待安装的软件需要图形介面支持，一般还需要『 yum groupinstall "X Software Development" 』
* 若安装的软件较旧，可能需要『 yum groupinstall "Legacy Software Development" 』
## Tarball 安装的基本步骤
先将 Tarball 解压缩，然后到原始码所在的目录下进行 makefile 的创建，再以 make 来进行编译与安装的动作。基础动作：
* 取得原始档：将 tarball 文件在 /usr/local/src 目录下解压缩；
* 取得步骤流程：进入新创建的目录底下，去查阅 INSTALL 与 README 等相关文件内容 (很重要的步骤！)；
* 相依属性软件安装：根据 INSTALL/README 的内容察看并安装好一些相依的软件 (非必要)；
* 创建 makefile：以自动侦测程序 (configure 或 config) 侦测作业环境，并创建 Makefile 这个文件；
* 编译：以 make 这个程序并使用该目录下的 Makefile 做为他的参数配置档，来进行 make (编译或其他) 的动作；
* 安装：以 make 这个程序，并以 Makefile 这个参数配置档，依据 install 这个标的 (target) 的指定来安装到正确的路径！

通常在每个软件在释出的时候，都会附上 INSTALL 或者是 README 这种档名的说明档，这些说明档请『确实详细的』 阅读过一遍，通常这些文件会记录这个软件的安装要求、软件的工作项目、 与软件的安装参数配置及技巧等。  
至于 makefile 在制作出来之后，里头会有相当多的标的 (target)，最常见的就是 install 与 clean 。通常『make clean』代表著将目标档 (object file) 清除掉，『make』则是将原始码进行编译而已。编译完成的可运行档与相关的配置档还在原始码所在的目录当中，因此，最后要进行『make install』来将编译完成的所有可运行档安装到正确的路径去。  
大部分 tarball 软件安装的命令下达方式：
* ./configure  
  这个步骤就是在创建 Makefile 文件。通常程序开发者会写一支 scripts 来检查你的 Linux 系统、相关的软件属性等等，这个步骤相当的重要， 因为未来你的安装资讯都是这一步骤内完成的！另外，这个步骤的相关资讯应该要参考一下该目录下的 README 或 INSTALL 相关的文件！
* make clean  
  make 会读取 Makefile 中关于 clean 的工作。这个步骤不一定会有，但是希望运行一下，因为他可以去除目标文件！因为谁也不确定原始码里面到底有没有包含上次编译过的目标文件 (*.o) 存在，所以当然还是清除一下比较妥当的。 至少等一下新编译出来的运行档我们可以确定是使用自己的机器所编译完成的。
* make  
  make 会依据 Makefile 当中的默认工作进行编译的行为。编译的工作主要是进行 gcc 来将原始码编译成为可以被运行的 object files ，但是这些 object files 通常还需要一些函式库之类的 link 后，才能产生一个完整的运行档！使用 make 就是要将原始码编译成为可以被运行的可运行档，而这个可运行档会放置在目前所在的目录之下， 尚未被安装到预定安装的目录中；
* make install  
  通常这就是最后的安装步骤了，make 会依据 Makefile 这个文件里面关于 install 的项目，将上一个步骤所编译完成的数据给他安装到预定的目录中，就完成安装了。

只要一个步骤无法成功，那么后续的步骤就完全没有办法进行。如果安装成功， 并且是安装在独立的一个目录中，例如 /usr/local/packages 这个目录中好了，那么你就必需手动的将这个软件的 man page 给他写入 /etc/man.config 里面去。

## 一般 Tarball 软件安装的建议事项 (如何移除，升级)
基本上，在默认的情况下，原本的 Linux distribution 释出安装的软件大多是在 /usr 里面的，而使用者自行安装的软件则建议放置在 /usr/local 里面。这是考量到管理使用者所安装软件的便利性。  
在默认的情况下， man 会去搜寻 /usr/local/man 里面的说明文件， 因此，如果我们将软件安装在 /usr/local 底下的话，那么自然安装完成之后， 该软件的说明文件就可以被找到了。  
建议将安装的软件放置在 /usr/local 下，原始码 (Tarball) 则建议放置在 /usr/local/src (src 为 source 的缩写)底下。  
Linux distribution 默认的安装软件的路径。以 apache (apache 是 WWW 服务器软件) 为例：
* /etc/httpd
* /usr/lib
* /usr/bin
* /usr/share/man

软件的内容大致上是摆在 etc, lib, bin, man 等目录当中，分别代表『配置档、函式库、运行档、线上说明档』。以 tarball 来安装时，如果是放在默认的 /usr/local 里面，由于 /usr/local 原本就默认这几个目录了，所以你的数据就会被放在：
* /usr/local/etc
* /usr/local/bin
* /usr/local/lib
* /usr/local/man

如果每个软件都选择在这个默认的路径下安装的话， 那么所有的软件的文件都将放置在这四个目录当中，因此，如果都安装在这个目录下的话， 那么未来再想要升级或移除的时候，就会比较难以追查文件的来源。而如果在安装的时候选择的是单独的目录，例如将 apache 安装在 /usr/local/apache 当中，那么你的文件目录就会变成：
* /usr/local/apache/etc
* /usr/local/apache/bin
* /usr/local/apache/lib
* /usr/local/apache/man

单一软件的文件都在同一个目录之下，那么要移除该软件， 只要将该目录移除即可视为该软件已经被移除。要移除 apache 只要下达『rm -rf /usr/local/apache』 就算移除这个软件了。实际安装的时候还是得视该软件的 Makefile 里头的 install 资讯才能知道到底他的安装情况为何。  
在运行某些命令的时候，与该命令是否在 PATH 这个环境变量所记录的路径有关，以上面为例，/usr/local/apache/bin 不在 PATH 里面的，所以运行 apache 的命令就得要利用绝对路径了，否则就得将这个 /usr/local/apache/bin 加入 PATH 里面。另外，那个 /usr/local/apache/man 也需要加入 man page 搜寻的路径当中。  
1. 最好将 tarball 的原始数据解压缩到 /usr/local/src 当中；
2. 安装时，最好安装到 /usr/local 这个默认路径下；
3. 考虑未来的反安装步骤，最好可以将每个软件单独的安装在 /usr/local 底下；
4. 为安装到单独目录的软件之 man page 加入 man path 搜寻：  
   如果你安装的软件放置到 /usr/local/software/ ，那么 man page 搜寻的配置中，可能就得要在 /etc/man.config 内的 40~50 行左右处，写入如下的一行：`MANPATH /usr/local/software/man`
## 一个简单的范例：利用 ntp 示范
```
[root@www ~]# cd /usr/local/src   <==切换目录
[root@www src]# tar -zxvf /root/ntp-4.2.4p7.tar.gz  <==解压缩到此目录
ntp-4.2.4p7/         <==会创建这个目录
ntp-4.2.4p7/libopts/
....(底下省略)....
[root@www src]# cd ntp-4.2.4p7/
[root@www ntp-4.2.4p7]# vi INSTALL  <==记得 README 也要看一下！
# 特别看一下 28 行到 54 行之间的安装简介！可以了解如何安装的流程

[root@www ntp*]# ./configure --help | more  <==查询可用的参数有哪些
  --prefix=PREFIX         install architecture-independent files in PREFIX
  --enable-all-clocks     + include all suitable non-PARSE clocks:
  --enable-parse-clocks   - include all suitable PARSE clocks:
# 上面列出的是比较重要的，或者是你可能需要的参数功能！

[root@www ntp*]# ./configure --prefix=/usr/local/ntp \
>  --enable-all-clocks --enable-parse-clocks  <==开始创建makefile
checking for a BSD-compatible install... /usr/bin/install -c
checking whether build environment is sane... yes
....(中间省略)....
checking for gcc... gcc           <==找到 gcc 编译器了
....(中间省略)....
config.status: creating Makefile  <== Makefile
config.status: creating config.h
config.status: executing depfiles commands
# 一般来说 configure 配置参数较重要的就是那个 --prefix=/path 了，--prefix 后面接的路径就是『这个软件未来要安装到那个目录去？』如果你没有指定 --prefix=/path 这个参数，通常默认参数就是 /usr/local 至于其他的参数意义就得要参考 ./configure --help 。这个动作完成之后会产生 makefile 或 Makefile 这个文件。这个侦测检查的过程会显示在屏幕上， 特别留意关于 gcc 的检查，还有最重要的是最后需要成功的创建起 Makefile 才行！

[root@www ntp*]# make clean; make
[root@www ntp*]# make check
[root@www ntp*]# make install
# 将数据安装在 /usr/local/ntp 底下
```
## 利用 patch 升级原始码
所谓的『升级原始码』常常是只有更改部分文件的小部分内容而已。首先，没有更动过的文件的目标档 (object file) 根本就不需要重新编译，而且有更动过的文件又可以利用 make 来自动 update (升级)，如此一来，我们原先的配置 (makefile 文件里面的守则) 将不需要重新改写或侦测。  
diff 命令，可以将『两个文件之间的差异性列出来』。可以通过 diff 比对出新旧版本之间的文字差异，然后再以 patch 命令来将旧版的文件升级。很多的软件开发商在升级了原始码之后，几乎都会释出所谓的 patch file，也就是直接将原始码 update 一个方式。  
patch 的基本语法如下：
```
patch -p数字 < patch_file
# 『 -p数字』就是与 patch_file 里面列出的档名有关的资讯。 -pxx 那个 xx 代表『拿掉几个斜线(/)』的意思。
```
如果 patch 错误，可以通过『 patch -R < ../main_0.1_to_0.2.patch 』还原。

# 函数库管理
很多的软件之间都会互相取用彼此提供的函式库来进行特殊功能的运行， 例如很多需要验证身份的程序都习惯利用 PAM 这个模块提供的验证机制来实作，而很多网络连线机制则习惯利用 SSL 函式库来进行连线加密的机制。 函式库又依照是否被编译到程序内部而分为动态与静态函式库。
## 动态与静态函数库
### 静态函数库的特点
* 扩展名：(扩展名为 .a)  
  这类的函式库通常扩展名为 libxxx.a 的类型；
* 编译行为：  
  这类函式库在编译的时候会直接整合到运行程序当中，所以利用静态函式库编译成的文件会比较大一些；
* 独立运行的状态：  
  这类函式库最大的优点，就是编译成功的可运行档可以独立运行，而不需要再向外部要求读取函式库的内容 (请参照动态函式库的说明)。
* 升级难易度：  
  虽然运行档可以独立运行，但因为函式库是直接整合到运行档中， 因此若函式库升级时，整个运行档必须要重新编译才能将新版的函式库整合到程序当中。 也就是说，在升级方面，只要函式库升级了，所有将此函式库纳入的程序都需要重新编译！
### 动态函数库的特点
* 扩展名：(扩展名为 .so)  
  这类函式库通常扩展名为 libxxx.so 的类型；
* 编译行为：  
  动态函式库与静态函式库的编译行为差异挺大的。 与静态函式库被整个捉到程序中不同的，动态函式库在编译的时候，在程序里面只有一个『指向 (Pointer)』的位置而已。也就是说，动态函式库的内容并没有被整合到运行档当中，而是当运行档要使用到函式库的机制时， 程序才会去读取函式库来使用。由于运行档当中仅具有指向动态函式库所在的指标而已， 并不包含函式库的内容，所以他的文件会比较小一点。
* 独立运行的状态：  
  这类型的函式库所编译出来的程序不能被独立运行， 因为当我们使用到函式库的机制时，程序才会去读取函式库，所以函式库文件『必须要存在』才行，而且，函式库的『所在目录也不能改变』，因为我们的可运行档里面仅有『指标』亦即当要取用该动态函式库时， 程序会主动去某个路径下读取。所以动态函式库可不能随意移动或删除，会影响很多相依的程序软件。
* 升级难易度：  
  虽然这类型的运行档无法独立运行，然而由于是具有指向的功能， 所以，当函式库升级后，运行档根本不需要进行重新编译的行为，因为运行档会直接指向新的函式库文件 (前提是函式库新旧版本的档名相同)。

目前的 Linux distribution 比较倾向于使用动态函式库，最重要的一点， 就是因为函式库的升级方便。由于 Linux 系统里面的软件相依性太复杂了，如果使用太多的静态函式库，那么升级某一个函式库时， 都会对整个系统造成很大的冲击，因为其他相依的运行档也需要重新编译。  
绝大多数的函式库都放置在：/usr/lib, /lib 目录下。此外，Linux 系统里面很多的函式库其实 kernel 就提供了，kernel 的函式库放在 /lib/modules 里面。不同版本的核心提供的函式库差异性是很大的。
## ldconfig 与 /etc/ld.so.conf
将常用到的动态函式库先加载内存当中 (缓存, cache)，当软件要取用动态函式库时，就不需要从头由硬盘里面读出，可以增进动态函数库的读取速度。需要 ldconfig 与 /etc/ld.so.conf 协助。  
将动态函数库加载到高速缓存中：
1. 首先，我们必须要在 /etc/ld.so.conf 里面写下『 想要读入高速缓存当中的动态函式库所在的目录』，注意是目录而不是文件；
2. 接下来则是利用 ldconfig 这个运行档将 /etc/ld.so.conf 的数据读入缓存当中；
3. 同时也将数据记录一份在 /etc/ld.so.cache 这个文件当中。

ldconfig 还可以用来判断动态函数库的连结信息。
```
[root@www ~]# ldconfig [-f conf] [ -C cache]
[root@www ~]# ldconfig [-p]
选项与参数：
-f conf ：那个 conf 指的是某个文件名称，也就是说，使用 conf 作为 libarary 
	  函式库的取得路径，而不以 /etc/ld.so.conf 为默认值
-C cache：那个 cache 指的是某个文件名称，也就是说，使用 cache 作为缓存缓存
	  的函式库数据，而不以 /etc/ld.so.cache 为默认值
-p	：列出目前有的所有函式库数据内容 (在 /etc/ld.so.cache 内的数据！)

范例一：假设我的 MySQL 数据库函式库在 /usr/lib/mysql 当中，如何读进 cache ？
[root@www ~]# vi /etc/ld.so.conf
include ld.so.conf.d/*.conf
/usr/lib/mysql   <==新增这一行

[root@www ~]# ldconfig  <==画面上不会显示任何的资讯

[root@www ~]# ldconfig -p
530 libs found in cache `/etc/ld.so.cache'
        libz.so.1 (libc6) => /usr/lib/libz.so.1
        libxslt.so.1 (libc6) => /usr/lib/libxslt.so.1
....(底下省略)....
#       函式库名称 => 该函式库实际路径
```
在某些时候，如果想要自行加入某些 Tarball 安装的动态函式库，而你想要让这些动态函式库的相关连结可以被读入到缓存当中， 这个时候你可以将动态函式库所在的目录名称写入 /etc/ld.so.conf 当中，然后运行 ldconfig 即可。
## 程序的动态函数库解析：ldd
```
[root@www ~]# ldd [-vdr] [filename]
选项与参数：
-v ：列出所有内容资讯；
-d ：重新将数据有遗失的 link 点显示出来
-r ：将 ELF 有关的错误内容显示出来

范例一：找出 /usr/bin/passwd 这个文件的函式库数据
[root@www ~]# ldd /usr/bin/passwd
....(前面省略)....
        libaudit.so.0 => /lib/libaudit.so.0 (0x00494000)     <==SELinux
        libselinux.so.1 => /lib/libselinux.so.1 (0x00101000) <==SELinux
        libc.so.6 => /lib/libc.so.6 (0x00b99000)
        libpam.so.0 => /lib/libpam.so.0 (0x004ab000)         <==PAM 模块
....(底下省略)....

范例二：找出 /lib/libc.so.6 这个函式的相关其他函式库！
[root@www ~]# ldd -v /lib/libc.so.6
        /lib/ld-linux.so.2 (0x00ab3000)
        linux-gate.so.1 =>  (0x00636000)

        Version information:  <==使用 -v 选项，添加显示其他版本资讯！
        /lib/libc.so.6:
                ld-linux.so.2 (GLIBC_PRIVATE) => /lib/ld-linux.so.2
                ld-linux.so.2 (GLIBC_2.3) => /lib/ld-linux.so.2
                ld-linux.so.2 (GLIBC_2.1) => /lib/ld-linux.so.2
```
使用 -v 这个参数还可以得知该函式库来自于哪一个软件。上例中，就可以得到该 libc.so.6 其实可以支持 GLIBC_2.1 等的版本。

# 检验软件正确性
md5sum 与 sha1sum —— 文件指纹，验证文件的正确性。
## md5sum / sha1sum
目前有多种机制可以计算文件的指纹码，我们选择使用较为广泛的 MD5 与 SHA1 加密机制来处理。
* CentOS-5.3-i386-netinstall.iso：CentOS 5.3 的网络安装映像档；
* md5sum.txt： MD5 指纹编码
* sha1sum.txt： SHA1 指纹编码

如果你下载了 CentOS-5.3-i386-netinstall.iso 后，再以 md5sum 与 sha1sum 去检验这个文件时， 文件所回传的指纹码应该要与网站上面提供的文件指纹码相同才对！我们由网站上面提供的指纹码知道这个映像档的指纹为：
* MD5 : 6ae4077a9fc2dcedca96013701bd2a43
* SHA1: a0c640ae0c68cc0d9558cf4f8855f24671b3dadb
```
[root@www ~]# md5sum/sha1sum [-bct] filename
[root@www ~]# md5sum/sha1sum [--status|--warn] --check filename
选项与参数：
-b ：使用 binary 的读档方式，默认为 Windows/DOS 文件型态的读取方式；
-c ：检验文件指纹；
-t ：以文字型态来读取文件指纹。
```
要在 Linux 系统上为这些重要的文件进行指纹数据库的创建，将底下这些文件创建数据库：
* /etc/passwd
* /etc/shadow( 假如你不让使用者改口令了 )
* /etc/group
* /usr/bin/passwd
* /sbin/portmap
* /bin/login
* /bin/ls
* /bin/ps
* /usr/bin/top

可以替这些文件创建指纹数据库 (就是使用 md5sum 检查一次，将该文件指纹记录下来，然后常常以 shell script 的方式由程序自行来检查指纹表是否不同了)，那么对于文件系统会比较安全。