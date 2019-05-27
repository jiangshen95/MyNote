# 什么是 X Window System
## X Window 的发展简史
X Window 系统最早是由 MIT (Massachusetts Institute of Technology, 麻省理工学院) 在 1984 年发展出来的， 当初 X 就是在 Unix 的 System V 这个操作系统版本上面开发出来的。在开发 X 时，开发者就希望这个窗口介面不要与硬件有强烈的相关性，这是因为如果与硬件的相关性高，那就等于是一个操作系统了， 如此一来的应用性会比较局限。因此 X 在当初就是以应用程序的概念来开发的，而非以操作系统来开发。  
1992 年 XFree86 (http://www.xfree86.org/) 计画顺利展开， 该计画持续在维护 X11R6 的功能性，包括对新硬件的支持以及更多新增的功能等等。当初定名为 XFree86 其实是根据『 X + Free software + x86 硬件 』而来的。早期 Linux 所使用的 X Window 的主要核心都是由 XFree86 这个计画所提供的。  
不过由于一些授权的问题导致 XFree86 无法继续提供类似 GPL 的自由软件，后来 Xorg 基金会就接手 X11R6 的维护！ Xorg (http://www.x.org/) 利用当初 MIT 发布的类似自由软件的授权， 将 X11R6 拿来进行维护，并且在 2004 年发布了 X11R6.8 版本，更在 2005 年后发表了 X11R7.x 版。 现在我们 CentOS 5.x 使用的 X 就是 Xorg 提供的 X11R7 。  
* 在 Unix Like 上面的图形使用者介面 (GUI) 被称为 X 或 X11；
* X11 是一个『软件』而不是一个操作系统；
* X11 是利用网络架构来进行图形介面的运行与绘制；
* 较著名的 X 版本为 X11R6 这一版，目前大部分的 X 都是这一版演化出来的 (包括 X11R7)；
* 现在大部分的 distribution 使用的 X 都是由 Xorg 基金会所提供的 X11 软件；
* X11 使用的是 MIT 授权，为类似 GPL 的自由软件授权方式。
## 主要组件：X Server/X Client/Window Manager/Display Manager
X Window system 是个利用网络架构的图形使用者介面软件。基本上是分成 X Server 与 X Client 两个组件。其中 X Server 在管理硬件，而 X Client 则是应用程序。 在运行上，X Client 应用程序会将所想要呈现的画面告知 X Server ，最终由 X server 来将结果透过他所管理的硬件绘制出来。  
X Server 的重点就是在管理用户端的硬件，包括接受键盘/鼠标等设备的输入资讯， 并且将图形绘制到屏幕上。X Client 提供要绘制的图形数据。
### X Server：硬件管理、屏幕绘制与提供字型功能：
XFree86 计画及 Xorg 基金会，主要提供的就是 X Server 。X Server 管理的设备主要与输入/输出有关，包括键盘、鼠标、手写板、显示器 (monitor) 、屏幕解析度与色彩深度、显卡 (包含驱动程序) 与显示的字型等等，都是 X Server 管理的。  
『每部用户端主机都需要安装 X Server，而服务器端则是提供 X Client 软件， 以提供用户端绘图所需要的数据数据』。  
X Server / X Client 的互动并非仅有 client --> server，两者其实有互动的。X Server 还有一个重要的工作，那就是将来自输入装置 (如键盘、鼠标等) 的动作告知 X Client 。
### X Client：负责 X Server 要求的【事件】之处理
X Server 主要是管理显示介面与在屏幕上绘图，同时将输入装置的行为告知 X Client， 此时 X Client 就会依据这个输入装置的行为来开始处理，最后 X Client 会得到『这个输入装置的行为会产生某个图示』，然后将这个图示的显示数据回传给 X Server ， X server 再根据 X Client 传来的绘图数据将他描图在自己的屏幕上，来得到显示的结果。  
也就是说， X Client 最重要的工作就是处理来自 X Server 的动作，将该动作处理成为绘图数据， 再将这些绘图数据传回给 X Server 。由于 X Client 的目的在产生绘图的数据，因此我们也称呼 X Client 为 X Application (X 应用程序)。而且，每个 X Client 并不知道其他 X Client 的存在， 意思是说，如果有两个以上的 X client 同时存在时，两者并不知道对方到底传了什么数据给 X Server ， 因此 X Client 的绘图常常会互相重叠而产生困扰。  
举个例子来说，当我们在 X Window 的画面中，将鼠标向右移动，首先， X server 会侦测到鼠标的移动，此时，他将鼠标的这个动作告知 X Client， X Client 就会去运算，要将鼠标指标向右移动几个位素，然后将这个结果告知 X server ， 接下来，您就会看到 X Server 将鼠标指标向右移动。  
这样处理，最大的好处是， X Client 不需要知道 X Server 的硬件配备与操作系统。因为 X Client 单纯就是在处理绘图的数据而已，本身是不绘图的。所以，用户端的 X Server 的硬件，操作系统，服务器端的 X Client 不需要知道。
### X Window Manager：特殊的 X Client ，负责管理所有的 X Client 软件
由于 X Client 彼此不知道对方在屏幕中的位置，就需要 Window Manager (WM, 窗口管理员)。窗口管理员也是 X client ，只是他主要在负责全部 X client 的控管，还包括提供某些特殊的功能，例如：
* 提供许多的控制元素，包括工作列、背景壁纸的配置等等；
* 管理虚拟壁纸 (virtual desktop)；
* 提供窗口控制参数，这包括窗口的大小、窗口的重叠显示、窗口的移动、窗口的最小化等等。

我们常常听到的 KDE, GNOME, XFCE 还有极简易的 twm 等等，都是一些窗口管理员的专案计画。这些专案计画中，每种窗口管理员所用以开发的显示引擎都不太相同，所著重的方向也不一样， 因此才会说，在 Linux 底下，每套 Window Manager 都是独特存在的，不是换了壁纸与显示效果而已， 而是连显示的引擎都不会一样。常见的窗口管理员全名与链接：
* GNOME (GNU Network Object Model Environment)：http://www.gnome.org/
* KDE (K Desktop Enviroment)：http://kde.org/
* twm (Tab Window Manager)：http://xwinman.org/vtwm.php
* XFCE (XForms Common Environment)：http://www.xfce.org/

窗口管理员上面还有提供非常多的 X client 软件， 包括办公室生产力软件 (Open Office) 以及常用的网络功能 (firefox 浏览器、 Thunderbird 收发信件软件) 等。  
由于我们要在本机端启动 X Window system ，因此，在我们的 CentOS 主机上面必须要有 Xorg 的 X server 核心， 这样才能够提供屏幕的绘制，然后为了让窗口管理更方便，于是就加装了 GNOME 这个计画的 window manager ， 然后为了让自己的使用更方便，于是就在 GNOME 上面加上更多的窗口应用软件，包括输入法等等的， 最后就建构出我们的 X Window System。
### Display Manager：提供登陆需求
display manager 最大的任务就是提供登陆的环境， 并且加载使用者选择的 Window Manager 与语系等数据。几乎所有的大型窗口管理员专案计画都会提供 display manager ，在 CentOS 上面我们主要利用的是 GNOME 的 GNOME Display Manager (gdm) 这支程序来提供 tty7 的图形介面登陆。至于登陆后取得的窗口管理员， 则可以在 gdm 上面进行选择。
## X Window 的启动流程
可以透过登陆本机的文字介面后，输入 startx 来启动 X 窗口；也能够透过 display manager (如果有启动 runlevel 5) 提供的登陆画面，输入你的帐号口令来登陆与取得 X 窗口的。
### 在文字界面启动 X：透过 startx 命令
Linux 是个多人多工的操作系统，所以，X 窗口也是可以根据不同的使用者而有不同的配置。也就是说，每个用户启动 X 时， X server 的解析度、启动 X client 的相关软件及 Window Manager 的选择可能都不一样。  
当在纯文字介面且并没有启动 X 窗口的情况下来输入 startx 时，这个 startx 的作用就是在帮你配置好上头提到的这些动作。startx 其实是一个 shell script ，他是一个比较亲和的程序，会主动的帮忙使用者创建起他们的 X 所需要引用的配置档而已。  
startx 最重要的任务就是找出使用者或者是系统默认的 X server 与 X client 的配置档，而使用者也能够使用 startx 外接参数来取代配置档的内容。这个意思是说：startx 可以直接启动，也能够外接参数，例如底下格式的启动方式：
```
[root@www ~]# startx [X client 参数] -- [X server 参数]

# 范例：以色彩深度为 16 bit 启动 X
[root@www ~]# startx  --  -depth 16
```
startx 后面接的参数以两个减号『--』隔开，前面的是 X Client 的配置，后面的是 X Server 的配置。范例是让 X server 以色彩深度 16 bit 色 (亦即每一像素占用 16 bit ，也就是 65536 色) 显示， 因为色彩深度是与 X Server 有关的，所以参数是写在 -- 后面。  
事实上启动 X 的是 xinit 这支程序， startx 仅是在帮忙找出配置值而已。startx 找到的配置值可用顺序基本上是：
* X server 的参数方面：
    1. 使用 startx 后面接的参数；
    2. 若无参数，则找寻使用者家目录的文件，亦即 ~/.xserverrc
    3. 若无上述两者，则以 /etc/X11/xinit/xserverrc
    4. 若无上述三者，则单纯运行 /usr/bin/X (此即 X server 运行档)
* X client 的参数方面：
    1. 使用 startx 后面接的参数；
    2. 若无参数，则找寻使用者家目录的文件，亦即 ~/.xinitrc
    3. 若无上述两者，则以 /etc/X11/xinit/xinitrc
    4. 若无上述三者，则单纯运行 xterm (此为 X 底下的终端机软件)

接下来 startx 会去呼叫 xinit 这支程序来启动我们所需要的 X 窗口系统整体。
### 由 startx 呼叫运行的 xinit
事实上，当 startx 找到需要的配置值后，就呼叫 xinit 实际启动 X 的。他的语法是：
```
[root@www ~]# xinit [client option] -- [server or display option]
```
通过 startx 找到适当的 xinitrc 与 xserverrc 后，就交给 xinit 来运行。 在默认的情况下 (使用者尚未有 ~/.xinitrc 等文件时)，你输入 startx ， 就等于进行 xinit /etc/X11/xinit/xinitrc -- /etc/X11/xinit/xserverrc 这个命令。但由于 xserverrc 也不存在，因此实际上的命令是：xinit /etc/X11/xinit/xinitrc -- /usr/bin/X。  
在单纯只是运行 xinit 时，系统的默认 X Client 与 X Server 内容是：
```
xinit xterm  -geometry  +1+1  -n  login  -display  :0 --  X  :0
```
在 X client 方面：那个 xterm 是 X 窗口底下的虚拟终端机，后面接的参数则是这个终端机的位置与登陆与否。 最后面会接一个『 -display :0 』表示这个虚拟终端机是启动在『第 :0 号的 X 显示介面』的意思。至于 X Server 方面， 而我们启动的 X server 程序就是 X 。其实 X 就是 Xorg 的连结档，亦即是 X Server 的主程序。直接运行 X 而已，同时还指定 X 启动在第 :0 个 X 显示介面。
### 启动 X server 的文件：xserverrc
X 窗口最先需要启动的就是 X server ，X server 启动的脚本与参数是透过 /etc/X11/xinit/ 里面的 xserverrc 。CentOS 5.x 根本就没有 xserverrc 这个文件，所以就运行 /usr/bin/X 这个命令，这个命令也是最原始的 X Server 运行档。  
在启动 X Server 时，Xorg 会去读取 /etc/X11/xorg.conf 这个配置档。如果一切顺利，那么 X 就会顺利的在 tty7 的环境中启动了 X 。 单纯的 X 启动时，你只会看到画面一片漆黑，然后中心有个鼠标的光标而已。  
Linux 可以『同时启动多个 X』，第一个 X 的画面会在 :0 亦即是 tty7 ，第二个 X 则是 :1 亦即是 tty8 。 后续还可以有其他的 X 存在。所以，xterm 在加载时，也必须要使用 -display 来说明。X server 未注明加载的介面时，默认是使用 :0 ～ 但是 X client 未注明时，则无法运行。
### 启动 X Client 的文件：xinitrc
假设你的家目录并没有 ~/.xinitrc ，则此时 X Client 会以 /etc/X11/xinit/xinitrc 来作为启动 X Client 的默认脚本。xinitrc 这个文件会将很多其他的文件参数引进来， 包括 /etc/X11/xinit/xinitrc-common 与 /etc/X11/xinit/Xclients 还有 /etc/sysconfig/desktop 。  
最后，其实最终就是加载 KDE 或者是 GNOME 而已。最终在 XClient 文件当中会有两个命令的搜寻， 包括 startkde 与 gnome-session 这两个，这也是 CentOS 默认会提供的两个主要的 Window Manager 。可以透过修改 /etc/sysconfig/desktop 内的 DESKTOP=GNOME 或 DESKTOP=KDE 来决定默认使用哪个窗口管理员的。  
如果你的 .xinitrc 配置档里面有启动的 x client 很多的时候，千万注意将除了最后一个 window manager 或 X Client 之外，都放到背景里面去运行。
```
xclock -geometry 100x100-5+5 &
xterm -geometry 80x50-50+150 &
exec /usr/bin/twm
```
如果忘记加上 & 的符号，就会让系统等待，而无法一次就登陆 X 。
### X 启动的端口
在文字介面底下启动 X 时，直接使用 startx 来找到 X server 与 X client 的参数或配置档， 然后再呼叫 xinit 来启动 X 窗口系统。xinit 先加载 X server 到默认的 :0 这个显示介面 (默认在 tty7)，然后再加载 X client 到这个 X 显示介面上。而 X client 通常就是 GNOME 或 KDE ，这两个配置也能够在 /etc/sysconfig/desktop 里面作好配置。X 是可以跨网络的，会启动一个端口。  
CentOS 由于考虑 X 窗口是在本机上面运行，因此将端口改为插槽档 (socket) 了。事实上， X server 应该是要启动一个 port 6000 来与 X client 进行沟通。由于系统上面也可能有多个 X 存在，因此我们就会有 port 6001, port 6002... 等等。这也就是说：
X 窗口系统|显示介面号码|默认终端机|网络监听端口
:---:|:---:|:---:|:---:
第一个 X|hostname:0|tty7|port 6000
第二个 X|hostname:1|tty8|port 6001
在 X Window System 的环境下，我们称 port 6000 为第 0 个显示介面，亦即为 hostname:0 ， 那个主机名称通常可以不写，所以就成了 :0 即可。在默认的情况下，第一个启动的 X (不论是启动在第几个 port number) 是在 tty7 ，亦即按下 [ctrl]+[Alt]+[F7] 那个画面。  而起动的第二个 X，则默认在 tty8 亦即 [ctrl]+[Alt]+[F8] 那个画面。  
因为主机上的 X 可能有多个同时存在，因此，当我们在启动 X Server / Client 时， 应该都要注明该 X Server / Client 主要是提供或接受来自哪个 display 的 port number 才行。
## X 启动流程测试
## 是否需要激活 X Window System
一般来说，如果你的 Linux 主机定位为网络服务器的话，那么由于 Linux 里面的主要服务的配置档都是纯文字的格式文件，不需要 X Window 存在，因为 X Window 仅是 Linux 系统内的一个软件。  
如果 Linux 主机作为桌面计算机使用，就需要 X Window 的功能。

# X Server 配置档解析与配置
基本上， X Server 管理的是显卡、屏幕解析度、鼠标按键对应等等。X server 的配置档都是默认放置在 /etc/X11 目录下，而相关的显示模块或上面提到的总总模块，则主要放置在 /usr/lib/xorg/modules 底下。比较重要的是字型档与芯片组，主要放置在:
* 提供的萤幕字型: /usr/share/X11/fonts/
* 显卡的芯片组: /usr/lib/xorg/modules/drivers/

在 CentOS 底下，我们可以透过 chkfontpath 这个命令来取得目前系统有的字型文件目录。X Server 的配置档为 /etc/X11/xorg.conf 。
## 解析 xorg.conf 配置
```
[root@www ~]# X -version
X Window System Version 7.1.1
Release Date: 12 May 2006
X Protocol Version 11, Revision 0, Release 7.1.1
Build Operating System: Linux 2.6.18-53.1.14.el5PAE i686 Red Hat, Inc.
Current Operating System: Linux localhost.localdomain 2.6.18-128.1.14.el5 #1 
SMP Wed Jun 17 06:40:54 EDT 2009 i686
Build Date: 21 January 2009
Build ID: xorg-x11-server 1.1.1-48.52.el5
        Before reporting problems, check http://wiki.x.org
        to make sure that you have the latest version.
Module Loader present
```
/etc/X11/xorg.conf 这个文件的内容是分成数个段落的，每个段落以 Section 开始，以 EndSection 结束， 里面含有该 Section (段落) 的相关配置值，例如:
```
Section  "section name"
…… <== 与这个 section name 有关的配置项目
……
EndSection
```
常见的 section name 主要有：
1. Module: 被加载到 X Server 当中的模块 (某些功能的驱动程序)；
2. InputDevice: 包括输入的 1. 键盘的格式 2. 鼠标的格式，以及其他相关输入设备；
3. Files: 配置字型所在的目录位置等；
4. Monitor: 监视器的格式， 主要是配置水平、垂直的升级频率，与硬件有关；
5. Device: 这个重要，就是显卡芯片组的相关配置了；
6. Screen: 这个是在萤幕上显示的相关解析度与色彩深度的配置项目，与显示的行为有关；
7. ServerLayout: 上述的每个项目都可以重覆配置，这里则是此一 X server 要取用的哪个项目值的配置。
```
[root@www ~]# cd /etc/X11
[root@www X11]# cp -a xorg.conf xorg.conf.20090713  <== 备份
[root@www X11]# vim xorg.conf
Section "Module"
        Load  "dbe"
        Load  "extmod"
        Load  "record"
        Load  "dri"
        Load  "xtrap"
        Load  "glx"
        Load  "vnc"
EndSection
# 上面这些模块是 X Server 启动时，希望能够额外获得的相关支持的模块。
# 关于更多模块可以搜寻一下 /usr/lib/xorg/modules/extensions/ 这个目录

Section "InputDevice"
        Identifier  "Keyboard0"
        Driver      "kbd"
        Option      "XkbModel" "pc105"
        Option      "XkbLayout" "us"  <==注意，是 us 美式键盘对应
EndSection
# 这个玩意儿是键盘的对应配置数据，重点在于 XkbLayout 那一项，
# 特别注意到 Identifier (定义) 那一项，那个是在说明，我这个键盘的配置档，被定义为名称是 Keyboard0 的意思，这个名称最后会被用于 ServerLayout 中

Section "InputDevice"
        Identifier  "Mouse0"
        Driver      "mouse"
        Option      "Protocol" "auto"
        Option      "Device" "/dev/input/mice"
        Option      "ZAxisMapping" "4 5 6 7" <==滚轮支持
EndSection
# 这个则主要在配置鼠标功能，重点在那个 Protocol 项目，
# 那个是可以指定鼠标介面的配置值，我这里使用的是自动侦测！不论是 U盘/PS2。

Section "Files"
        RgbPath      "/usr/share/X11/rgb"
        ModulePath   "/usr/lib/xorg/modules"
        FontPath     "unix/:7100"  <==使用另外的服务来提供字型定义
        FontPath     "built-ins"
EndSection
# 我们的 X Server 很重要的一点就是必须要提供字型，这个 Files 
# 的项目就是在配置字型，你的主机必须要有字型档才行。一般字型文件在：
# /usr/share/X11/fonts/ 目录中。至于那个 Rgb 是与色彩有关的项目。

Section "Monitor"
        Identifier   "Monitor0"
        VendorName   "Monitor Vendor"
        ModelName    "Monitor Model"
        HorizSync    30.0 - 80.0
        VertRefresh  50.0 - 100.0
EndSection
# 萤幕监视器的配置仅有一个地方要注意，那就是垂直与水平的升级频率。
# 在上面的 HorizSync 与 VerRefresh 的配置上，要注意，不要配置太高，
# 这个玩意儿与实际的监视器功能有关，请查询你的监视器手册说明来配置

Section "Device"   <==显卡的驱动程序项目
        Identifier  "Card0"
        Driver      "vesa"  <==实际的驱动程序
        VendorName  "Unknown Vendor"
        BoardName   "Unknown Board"
        BusID       "PCI:0:2:0"
EndSection
# 这地方重要了，这就是显卡的芯片模块加载的配置区域。由于作者使用 Virtualbox
# 模拟器模拟这个测试机，因此这个地方显示的驱动程序为通用的 vesa 模块。
# 更多的显示芯片模块可以参考 /usr/lib/xorg/modules/drivers/

Section "Screen"    <==与显示的画面有关，解析度与色彩深度
        Identifier "Screen0"
        Device     "Card0"     <==使用哪个显卡来提供显示
        Monitor    "Monitor0"  <==使用哪个监视器
        SubSection "Display"   <==此阶段的附属配置项目
                Viewport   0 0
                Depth     16   <==就是色彩深度
		Modes    "1024x768" "800x600" "640x480" <==解析度
        EndSubSection
        SubSection "Display"
                Viewport   0 0
                Depth     24
		Modes    "1024x768" "800x600"
        EndSubSection
EndSection
# Monitor 与实际的显示器有关，而 Screen 则是与显示的画面解析度、色彩深度有关。
# 我们可以配置多个解析度，实际应用时可以让使用者自行选择想要的解析度来呈现。
# 不过，为了避免困扰，通常只指定一到两个解析度而已。

Section "ServerLayout"   <==实际选用的配置值
        Identifier     "X.org Configured"
        Screen      0  "Screen0" 0 0              <==解析度等
        InputDevice    "Mouse0" "CorePointer"     <==鼠标
        InputDevice    "Keyboard0" "CoreKeyboard" <==键盘
EndSection
# 我们上面配置了这么多的项目之后，最后整个 X Server 要用的项目，写入这里，包括键盘、鼠标以及显示介面。
# 其中 screen 的部分还牵涉到显卡、监视器萤幕等配置值
```
如果你的 Files 那个项目用的是直接写入字型的路径， 那就不需要启动 XFS (X Font Server)，如果是使用 font server 时，就要先启动 xfs ：
```
[root@www ~]# /etc/init.d/xfs start
```
## X Font Server (XFS) 与加入额外中文字型
这个服务的目的在提供 X server 字型库。X server 所使用的字型其实是 XFS 这个服务所提供的，因此没有启动 XFS 服务时，你的 X server 是无法顺利启动。  
XFS 的主配置档在 /etc/X11/fs/config ，而字型档则在 /usr/share/X11/fonts/ ，启动的脚本则在 /etc/init.d/xfs 。
```
[root@www ~]# vi /etc/X11/fs/config
client-limit = 10  <==最多允许几个 X server 向我要求字型(因为跨网络)
clone-self = on    <==与效能有关，若 xfs 达到限制值，启动新的 xfs
catalogue = /usr/share/X11/fonts/misc:unscaled,
        /usr/share/X11/fonts/75dpi:unscaled,
        /usr/share/X11/fonts/100dpi:unscaled,
        /usr/share/X11/fonts/Type1,
        /usr/share/X11/fonts/TTF,
        /usr/share/fonts/default/Type1,
# 字型文件的所在！如果你有新字型，可以放置在该目录。

default-point-size = 120            <==默认字型大小，单位为 1/10 点字 (point)
default-resolutions = 75,75,100,100 <==这个则是显示的字型像素 (pixel)
deferglyphs = 16                    <==延迟显示的字型，此为 16 bits 字型
use-syslog = on                     <==启动支持错误登录
no-listen = tcp                     <==启动 xfs 于 socket 而非 TCP
```
可以使用 chkfontpath 这个命令来列出目前支持的字型文件，也可以直接修改。  
```
# 1. 先安装中文字形软件，亦即 fonts-chinese 这个软件名
[root@www ~]# yum install fonts-chinese

# 2. 查阅 taipei 字型的所在目录位置：
[root@www ~]# rpm -ql fonts-chinese | grep taipei
/usr/share/fonts/chinese/misc/taipei16.pcf.gz  <==重点在目录！
/usr/share/fonts/chinese/misc/taipei20.pcf.gz
/usr/share/fonts/chinese/misc/taipei24.pcf.gz

# 3. 创建字型档的目录架构
[root@www ~]# cd /usr/share/fonts/chinese/misc
[root@www ~]# mkfontdir
# 这个命令在建置 fonts.dir 这个文件，提供字型文件目录的说明。

# 4. 将上述的目录加入 xfs 的支持之中：
[root@www ~]# chkfontpath -a /usr/share/fonts/chinese/misc/
[root@www ~]# chkfontpath
....(前面省略)....
/usr/share/fonts/chinese/misc:unscaled
/usr/share/fonts/chinese/misc  <==这两行会被新增出来！
[root@www ~]# /etc/init.d/xfs restart

# 5. 在 X window 底下启动终端机，测试一下有没有捉到该字型？
[root@www ~]# xlsofnts | grep taipei
# 如果顺利的话，你会看到有几个 taipeiXX 的字样在萤幕上出现！
```
这个时候的 X server 已经有新支持的中文字形了，不过，想要让 X client 可以使用额外的字型的话，还得要使用 fontconfig 的软件提供的 fc-cache 来创建字型缓存档才行。
### 让窗口管理员可以使用额外的字型
如果想要使用额外的字型的话，你可以自行取得某些字型来处理的。
```
# 1. 将上述的三个文件放置到系统配置目录，亦即底下的目录中：
[root@www ~]# cd /usr/share/fonts/
[root@www ~]# mkdir windows
[root@www ~]# cp /root/*.tt[fc] /usr/share/fonts/windows

# 2. 使用 fc-cache 将上述的文件加入字型的支持中：
[root@www ~]# fc-cache -f -v
....(前面省略)....
/usr/share/fonts/windows: caching, 4 fonts, 0 dirs
....(中间省略)....
fc-cache: succeeded
# -v 仅是列出目前的字型数据， -f 则是强制重新创建字型缓存！

# 3. 透过 fc-list 列出已经被使用的文件看看：
[root@www ~]# fc-list : file  <==找出被缓存住的档名
....(前面省略)....
/usr/share/fonts/windows/kaiu.ttf:
/usr/share/fonts/windows/times.ttf:
/usr/share/fonts/windows/mingliu.ttc:
....(后面省略)....
```
通过 fc-cache 以及 fc-list 去确认过字型确实存在后，就能够使用窗口管理员的功能去检查字型档了。在『系统』-->『偏好配置』-->『字型』点选后，就会出现可以调整的字型， 接下来你就会发现多出了『标楷体、细明体、新细明体』等字体可以选择。
## 配置档重建与显示参数微调
使用 Xorg 重新制作出配置档。使用 root 身份运行
```
[root@www ~]# Xorg -configure :1
```
此时 X 会主动的以内建的模块进行系统硬件的探索，并将硬件与字型的侦测结果写入 /root/xorg.conf.new 这个文件里面去，这就是 xorg.conf 的重制结果。不过，这个新建的文件不见得真的能够启动 X server ， 所以我们必须要使用底下的命令来测试一下这个新的配置档是否能够顺利的运行：
```
[root@www ~]# X -config /root/xorg.conf.new :1
```
如果一切顺利的话，就可以将 /root/xorg.conf.new 复制成为 /etc/X11/xorg.conf 覆盖掉修改错误的文件，然后重新启动 X 。
### 关于屏幕解析度与升级率
```
[root@www ~]# gtf 水平像素 垂直像素 升级频率 [-xv]
选项与参数：
水平像素：就是解析度的 X 轴
垂直像素：就是解析度的 Y 轴
升级频率：与显示器有关，一般可以选择 60, 75, 80, 85 等频率
-x      ：使用 Xorg 配置档的模式输出，这是默认值
-v      ：显示侦测的过程

[root@www ~]# gtf 1024 768 75 -x
# 1024x768 @ 75.00 Hz (GTF) hsync: 60.15 kHz; pclk: 81.80 MHz
Modeline "1024x768_75.00"  81.80  1024 1080 1192 1360  768 769 772 802  -HSync +Vsync

将上述的数据输入 xorg.conf 内的 Monitor 项目中：
[root@www ~]# vim /etc/X11/xorg.conf
Section "Monitor"
    Identifier   "Monitor0"
    VendorName   "Monitor Vendor"
    ModelName    "Monitor Model"
    Modeline "1024x768_75.00"  81.80  1024 1080 1192 1360  768 769 772 802  -HSync +Vsync
EndSection
```
然后，重启 X ，就能够选择新的解析度。重启 X 的方法，一个是『 init 3 ; init 5 』从文字模式与图形模式的运行等级去切换，另一个比较简单， 如果原本就是 runlevel 5 的话，那么在 X 的画面中按下『 [alt] + [crtl] + [backspace] 』三个组合按键， 就能够重新启动 X 窗口。

# 显卡驱动程序安装范例
## Nvidia
### 下载驱动程序
### 开始安装驱动程序
```
[root@www ~]# sh NVIDIA-Linux-x86_64-185.18.14-pkg2.run
```
## ATI (AMD)
## Intel
由于 Intel 针对 Linux 的图形介面驱动程序已经开放成为 Open source 了，所以理论上不需要重新安装 Intel 的显卡驱动程序的。