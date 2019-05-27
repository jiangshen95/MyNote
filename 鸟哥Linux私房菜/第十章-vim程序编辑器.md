# vi 与 vim
* 所有的 Unix Like 系统都会内建 vi 文书编辑器，其他的文书编辑器则不一定会存在；
* 很多个别软件的编辑接口都会主动呼叫 vi (例如未来会谈到的 crontab, visudo, edquota 等指令)；
* vim 具有程序编辑的能力，可以主动的以字体颜色辨别语法的正确性，方便程序设计；
* 因为程序简单，编辑速度相当快速。
# vi 的使用
* 一般模式：  
  默认模式。在这个模式中，可以使用【上下左右】按键来移动光标，【删除字符】或【删除整行】来处理档案内容，也可以使用【复制、粘贴】来处理文件数据
* 编辑模式  
  一般模式下可以进行删除、复制、粘贴等动作，但是无法编辑文件内容，等按下【i,l,o,O,a,A,r,R】等任何一个字母后，才会进入编辑模式。通常在 Linux 中，按下这些按键时，画面左下方会出现【INSERT 或 REPLACE】的字样，此时才可以进行编辑。要回到一般模式，必须要按下【Esc】即可退出编辑模式。
* 指令列命令模式：
  在一般模式中，输入【: / ?】三个中的任何一个按钮，就可以将光标移动到最底下一行。在这个模式中，可以提供【搜索】的动作，而读取，存储，大量取代字符，离开 vi，显示行号等动作也是在此模式中达成的。

一般模式可与编辑模式及指令列模式切换，但编辑模式与指令列模式之间不可互相切换。
* 直接输入【vi 档名】就能进入 vi 的一般模式了。vi 后面一定要加档名，不管档名存在与否。  
  如果档案权限不对，可能会无法写入，此时可以使用【强制写入】的方式【:wq!】，不过要在权限可以改变的情况下
## 按键说明
### 第一部分：一般模式可用的按钮说明。光标移动、复制粘贴、搜索替代等
<table>
<tr>
<th colspan="2">移动光标的方法</th>
</tr>
<tr>
<td style="text-align: center;">h 或 向左箭头键(←)</td>
<td>光标向左移动一个字符</td>
</tr>
<tr>
<td style="text-align: center;">j 或 向下箭头键(↓)</td>
<td>光标向下移动一个字符</td>
</tr>
<tr>
<td style="text-align: center;">k 或 向上箭头键(↑)</td>
<td>光标向上移动一个字符</td>
</tr>
<tr>
<td style="text-align: center;">l 或 向右箭头键(→)</td>
<td>光标向右移动一个字符</td>
</tr>
<tr>
<td colspan="2">
如果你将右手放在键盘上的话，你会发现 hjkl 是排列在一起的，因此可以使用这四个按钮来移动光标。 如果想要进行多次移动的话，例如向下移动 30 行，可以使用 "30j" 或 "30↓" 的组合按键， 亦即加上想要进行的次数(数字)后，按下动作即可！
</td>
<tr>
<tr>
<td style="text-align: center;">[Ctrl] + [f]</td>
<td>屏幕『向下』移动一页，相当于 [Page Down]按键(常用)</td></tr>
<tr><td style="text-align: center;">[Ctrl] + [b]</td>
	<td>屏幕『向上』移动一页，相当于 [Page Up] 按键(常用)</td></tr>
<tr><td style="text-align: center;">[Ctrl] + [d]</td>
	<td>屏幕『向下』移动半页</td></tr>
<tr><td style="text-align: center;">[Ctrl] + [u]</td>
	<td>屏幕『向上』移动半页</td></tr>
<tr><td style="text-align: center;">+</td>
	<td>光标移动到非空格符的下一列</td></tr>
<tr><td style="text-align: center;">-</td>
	<td>光标移动到非空格符的上一列</td></tr>
<tr><td style="text-align: center;">n&lt;space&gt;</td>
	<td>那个 n 表示『数字』，例如 20 。按下数字后再按空格键，光标会向右移动这一行的 n 个字符。例如 20&lt;space&gt; 则光标会向后面移动 20 个字符距离。</td></tr>
<tr><td style="text-align: center;">0 或功能键[Home]</td>
	<td>这是数字『 0 』：移动到这一行的最前面字符处(常用)</td></tr>
<tr><td style="text-align: center;">$ 或功能键[End]</td>
	<td>移动到这一行的最后面字符处(常用)</td></tr>
<tr><td style="text-align: center;">H</td>
	<td>光标移动到这个屏幕的最上方那一行的第一个字符</td></tr>
<tr><td style="text-align: center;">M</td>
	<td>光标移动到这个屏幕的中央那一行的第一个字符</td></tr>
<tr><td style="text-align: center;">L</td>
	<td>光标移动到这个屏幕的最下方那一行的第一个字符</td></tr>
<tr><td style="text-align: center;">G</td>
	<td>移动到这个档案的最后一行(常用)</td></tr>
<tr><td style="text-align: center;">nG</td>
	<td>n 为数字。移动到这个档案的第 n 行。例如 20G 则会移动到这个档案的第 20 行(可配合 :set nu)</td></tr>
<tr><td style="text-align: center;">gg</td>
	<td>移动到这个档案的第一行，相当于 1G(常用)</td></tr>
<tr><td style="text-align: center;">n&lt;Enter&gt;</td>
	<td>n 为数字。光标向下移动 n 行(常用)</td></tr>
<tr><td colspan="2" style="text-align: center;"><b>搜寻与取代</b></td></tr>
<tr><td style="text-align: center;">/word</td>
	<td>向光标之下寻找一个名称为 word 的字符串。例如要在档案内搜寻 vbird 这个字符串，就输入 /vbird 即可
(常用)</td></tr>
<tr><td style="text-align: center;">?word</td>
	<td>向光标之上寻找一个字符串名称为 word 的字符串。</td></tr>
<tr><td style="text-align: center;">n</td>
	<td>这个 n 是英文按键。代表『<u>重复前一个搜寻的动作</u>』。举例来说，
		如果刚刚我们执行 /vbird 去向下搜寻 vbird 这个字符串，则按下 n 后，会向下继续搜寻下一个名称为 vbird 的字符串。如果是执行 ?vbird 的话，那么按下 n 则会向上继续搜寻名称为 vbird 的字符串！</td></tr>
<tr><td style="text-align: center;">N</td>
	<td>这个 N 是英文按键。与 n 刚好相反，为『反向』进行前一个搜寻动作。例如 /vbird 后，按下 N 则表示『向上』搜寻 vbird 。</td></tr>
<tr><td colspan="2">使用 /word 配合 n 及 N 是非常有帮助的！可以让你重复的找到一些你搜寻的关键词！</td></tr>
<tr><td style="text-align: center;">:n1,n2s/word1/word2/g</td>
		<td>n1 与 n2 为数字。在第 n1 与 n2 行之间寻找 word1 这个字符串，并将该字符串取代为
		word2 ！举例来说，在 100 到 200 行之间搜寻 vbird 并取代为 VBIRD 则：<br>『:100,200s/vbird/VBIRD/g』。(常用)</td></tr>
<tr><td style="text-align: center;">:1,$s/word1/word2/g</td>
	<td>从第一行到最后一行寻找 word1 字符串，并将该字符串取代为word2 ！(常用)</td></tr>
<tr><td style="text-align: center;">:1,$s/word1/word2/gc</td>
	<td>从第一行到最后一行寻找 word1 字符串，并将该字符串取代为 word2 ！且在取代前显示提示字符给用户确认 (confirm)是否需要取代！(常用)</td></tr>
<tr><td colspan="2" style="text-align: center;"><b>删除、复制与粘贴</b></td></tr>
<tr><td style="text-align: center;">x, X</td>
	<td>在一行字当中，x 为向后删除一个字符 (相当于 [del] 按键)，X 为向前删除一个字符(相当于 [backspace] 亦即是退格键)(常用)</td></tr>
<tr><td style="text-align: center;">nx</td>
	<td>n 为数字，连续向后删除 n 个字符。举例来说，我要连续删除 10 个字符，『10x』。</td></tr>
<tr><td style="text-align: center;">dd</td>
	<td>删除游标所在的那一整列(常用)</td></tr>
<tr><td style="text-align: center;">ndd</td>
	<td>n 为数字。删除光标所在的向下 n 列，例如 20dd 则是删除 20 列(常用)</td></tr>
<tr><td style="text-align: center;">d1G</td>
	<td>删除光标所在到第一行的所有数据</td></tr>
<tr><td style="text-align: center;">dG</td>
	<td>删除光标所在到最后一行的所有数据</td></tr>
<tr><td style="text-align: center;">d$</td>
	<td>删除游标所在处，到该行的最后一个字符</td></tr>
<tr><td style="text-align: center;">d0</td>
	<td>那个是数字的 0 ，删除游标所在处，到该行的最前面一个字符</td></tr>
<tr><td style="text-align: center;">yy</td>
	<td>复制游标所在的那一行(<span class="text_import2">常用</span>)</td></tr>
<tr><td style="text-align: center;">nyy</td>
	<td>n 为数字。复制光标所在的向下 n 列，例如 20yy 则是复制 20 列(常用)</td></tr>
<tr><td style="text-align: center;">y1G</td>
	<td>复制游标所在列到第一列的所有数据</td></tr>
<tr><td style="text-align: center;">yG</td>
	<td>复制游标所在列到最后一列的所有数据</td></tr>
<tr><td style="text-align: center;">y0</td>
	<td>复制光标所在的那个字符到该行行首的所有数据</td></tr>
<tr><td style="text-align: center;">y$</td>
	<td>复制光标所在的那个字符到该行行尾的所有数据</td></tr>
<tr><td style="text-align: center;">p, P</td>
	<td>p 为将已复制的数据在光标下一行贴上，P 则为贴在游标上一行！举例来说，我目前光标在第 20 行，且已经复制了 10 行数据。则按下 p 后，那 10 行数据会贴在原本的 20 行之后，亦即由 21 行开始贴。但如果是按下 P 呢？那么原本的第 20 行会被推到变成 30 行。(常用)</td></tr>
<tr><td style="text-align: center;">J</td>
	<td>将光标所在列与下一列的数据结合成同一列</td></tr>
<tr><td style="text-align: center;">c</td>
	<td>重复删除多个数据，例如向下删除 10 行，[ 10cj ]</td></tr>
<tr><td style="text-align: center;">u</td>
	<td>复原前一个动作。(常用)</td></tr>
<tr><td style="text-align: center;">[Ctrl]+r</td>
	<td>重做上一个动作。(常用)</td></tr>
<tr><td colspan="2">这个 u 与 [Ctrl]+r 是很常用的指令！一个是复原，另一个则是重做一次</td></tr>
<tr><td style="text-align: center;">.</td>
	<td>小数点！意思是重复前一个动作的意思。如果你想要重复删除、重复贴上等等动作，按下小数点『.』就好了！(常用)</td></tr>
</table>

### 第二部分：一般模式切换到编辑模式的可用的按钮说明

<table>
<tr><td colspan="2" style="text-align: center;"><b>进入插入或取代的编辑模式</b></td></tr>
<tr><td style="text-align: center;">i, I</td>
		<td>进入插入模式(Insert mode)：<br>
		i 为『从目前光标所在处插入』， I 为『在目前所在行的第一个非空格符处开始插入』。(常用)</td></tr>
<tr><td style="text-align: center;">a, A</td>
	<td>进入插入模式(Insert mode)：<br>
	a 为『从目前光标所在的下一个字符处开始插入』， A 为『从光标所在行的最后一个字符处开始插入』。(常用)</td></tr>
<tr><td style="text-align: center;">o, O</td>
	<td>进入插入模式(Insert mode)：<br>
	这是英文字母 o 的大小写。o 为『在目前光标所在的下一行处插入新的一行』；O 为在目前光标所在处的上一行插入新的一行！(常用)</td></tr>
<tr><td style="text-align: center;">r, R</td>
	<td>进入取代模式(Replace mode)：<br>
	r 只会取代光标所在的那一个字符一次；R会一直取代光标所在的文字，直到按下 ESC 为止；(常用)</td></tr>
<tr><td colspan="2">上面这些按键中，在 vi 画面的左下角处会出现『--INSERT--』或『--REPLACE--』的字样。由名称就知道该动作了吧！！特别注意的是，我们上面也提过了，你想要在档案里面输入字符时，一定要在左下角处看到 INSERT 或 REPLACE 才能输入</td></tr>
<tr><td style="text-align: center;">[Esc]</td>
	<td>退出编辑模式，回到一般模式中(常用)</td></tr>
</table>

### 第三部分：一般模式切换到指令列模式的可用的按钮说明
<table>
<tr><td colspan="2" style="text-align: center;"><b>指令列的储存、离开等指令</b></td></tr>
<tr><td style="text-align: center;">:w</td>
	<td>将编辑的数据写入硬盘档案中(常用)</td></tr>
<tr><td style="text-align: center;">:w!</td>
	<td>若文件属性为『只读』时，强制写入该档案。不过，到底能不能写入，还是跟你对该档案的档案权限有关</td></tr>
<tr><td style="text-align: center;">:q</td>
	<td>离开 vi (常用)</td></tr>
<tr><td style="text-align: center;">:q!</td>
	<td>若曾修改过档案，又不想储存，使用 ! 为强制离开不储存档案。</td></tr>
<tr><td colspan="2">注意一下啊，那个惊叹号 (!) 在 vi 当中，常常具有『强制』的意思～</td></tr>
<tr><td style="text-align: center;">:wq</td>
	<td>储存后离开，若为 :wq! 则为强制储存后离开(常用)</td></tr>
<tr><td style="text-align: center;">ZZ</td>
	<td>这是大写的 Z 若档案没有更动，则不储存离开，若档案已经被更动过，则储存后离开！</td></tr>
<tr><td style="text-align: center;">:w [filename]</td>
	<td>将编辑的数据储存成另一个档案（类似另存新档）</td></tr>
<tr><td style="text-align: center;">:r [filename]</td>
	<td>在编辑的数据中，读入另一个档案的数据。亦即将 『filename』这个档案内容加到游标所在行后面</td></tr>
<tr><td style="text-align: center;">:n1,n2 w [filename]</td>
	<td>将 n1 到 n2 的内容储存成 filename 这个档案。</td></tr>
<tr><td style="text-align: center;">:! command</td>
	<td>暂时离开 vi 到指令列模式下执行 command 的显示结果！例如<br>『:! ls /home』即可在 vi 当中察看 /home 底下以 ls 输出的档案信息！</td></tr>
<tr><td colspan="2" style="text-align: center;"><b>vim 环境的变更</b></td></tr>
<tr><td style="text-align: center;">:set nu</td>
	<td>显示行号，设定之后，会在每一行的前缀显示该行的行号</td></tr>
<tr><td style="text-align: center;">:set nonu</td>
	<td>与 set nu 相反，为取消行号！</td></tr>
<tr><td style="text-align: center;">:e!</td>
	<td>恢复成档案的原始状态</td></tr>
</table>

## vim 的暂存档、救援恢复与开启时的警告讯息
使用 vim 编辑时，vim 会在与被编辑的档案目录下，再建立一个名为 .filename.swp 的档案（暂存档），对文件所作的动作就会被记录到这个 .man.config.swp 当中。  
当我们使用 vim 的一般模式下按下 [ctrl]-z 的组合按键时，vim 会被丢到背景去执行。  
当存在文件的 .swp 文件时，vim 会主动判断这个档案可能有问题。 vim 提示两点主要问题与解决方案：
* 问题一：可能有其他人或程序同时在编辑这个档案  
  如果在多人共同编辑的情况下，万一大家同时储存， vim 会出现这个警告窗口。解决方法
    * 找到另外的那个程序或人员，请他将该 vim 的工作结束，然后再继续处理
	* 如果只是查看档案内容并不会有任何修改编辑的行为，可以选择开启为只读(O Open Read-Only)档案
* 问题二：在前一个 vim 环境中，可能会因为某些不知名的原因导致 vim 中断(crashed)
    * 如果之前的 vim 处理动作尚未储存，此时应该要按下【R】，亦即使用 Recover 的项目，此时 vim 会载入 .man.config.swp 的内容，让你自己来决定要不要存储，这样就能挽救回来之前未储存的工作。不过那个 .swp 并不会在你档案结束 vim 后自动删除，所以离开 vim 后要自行删除，才能避免每次打开这个档案都会出现这样的警告。
    * 如果确定这个暂存档是没用的，直接按下【D（Delete）】删除掉这个暂存档。此时会直接载入文件，并且将旧的 .swp 删除后，建立这次会使用的新的 .swp 文件。

六个可用按钮：
* [O]pen Read-Only：打开此档案成为只读档， 可以用在你只是想要查阅该档案内容并不想要进行编辑行为时。
* (E)dit anyway：还是用正常的方式打开你要编辑的那个档案， 并不会载入暂存档的内容。不过很容易出现两个使用者互相改变对方的档案等问题
* (R)ecover：就是加载暂存盘的内容，用在你要救回之前未储存的工作。 不过当你救回来并且储存离开 vim 后，还是要手动自行删除那个暂存档
* (D)elete it：你确定那个暂存档是无用的！那么开启档案前会先将这个暂存盘删除！ 这个动作其实是比较常做的！因为你可能不确定这个暂存档是怎么来的，一般会选择删除
* (Q)uit：按下 q 就离开 vim ，不会进行任何动作回到命令提示字符。
* (A)bort：忽略这个编辑行为，感觉上与 quit 非常类似！ 也会送你回到命令提示字符。

# vim 的额外功能
颜色显示，程序除错(debug)
## 区块选择(Visual Block)
<table>
<tr><th colspan="2">区块选择的按键意义</th></tr>
<tr><td>v</td><td>字符选择，会将光标经过的地方反白选择</td></tr>
<tr><td>V</td><td>行选择，会将光标经过的行反白选择</td></tr>
<tr><td>[Ctrl] + v</td><td>区块选择，可以用长方形的方式选择资料</td></tr>
<tr><td>y</td><td>将反白的地方复制起来</td></tr>
<tr><td>d</td><td>将反白的地方删除掉</td></tr>
</table>

## 多档案编辑
<table>
<tr><th colspan="2">多档案编辑的按键</th></tr>
<tr><td>:n</td><td>编辑下一个档案</td></tr>
<tr><td>:N</td><td>编辑上一个档案</td></tr>
<tr><td>:files</td><td>列出目前这个 vim 的开启的所有档案</td></tr>
</table>

* 通过 【vim 文件1 文件2】  指令来使用一个 vim 开启两个档案
* 在 vim 中使用 【:files】 查看编辑的档案数据

## 多窗口功能
【分割窗口】 在指令模式下输入 【:sp {filename}】，那个 filename 可有可无，想要在新窗口中启动一个档案，就加入档名，否则仅输入 :sp 时，出现的则是同一个档案在两个窗口间。  
利用『[ctrl]+w+↑』及『[ctrl]+w+↓』 在两个窗口之间移动。分割窗口相关指令功能有很多，只要记得几个即可。
<table>
<tr><th colspan="2">多窗口情况下的按键功能</th></tr>
<tr><td>:sp&nbsp;[filename]</td><td>开启一个新窗口，如果有加 filename，表示在新窗口开启一个新档案，否则表示两个窗口为同一个档案内容(同步显示)</td></tr>
<tr><td>[ctrl] + w + j<br>[ctrl] + w + ↓</td><td>按键的按法是：先按下 [ctrl] 不放， 再按下 w 后放开所有的按键，然后再按下 j (或向下箭头键)，则光标可移动到下方的窗口。</td></tr>
<tr><td>[ctrl] + w + k<br>[ctrl] + w + ↑</td><td>同上，不过移动到上面的窗口</td></tr>
<tr><td>[ctrl] + w + q</td><td>其实就是 :q 结束离开。如果想要结束下方的窗口，那么移动到下方窗口后，按下 :q 即可离开，也可以按下 [ctrl] + w + q</td></tr>
</table>

## vim 环境设定与记录：~/.vimrc, ~/.viminfo
vim 会主动将你曾经做过的行为记录下来，方便下次作业，记录在 ~/.viminfo 中  
vim 预设环境，环境设定。要查看目前的设定值，可以在一般模式时输入 【:set all】 查阅。
<table>
<tr><th colspan="2">vim 的环境设定参数</th></tr>
<tr><td>:set nu<br>:set nonu</td><td>设定与取消行号</td></tr>
<tr><td>:set hlsearch<br>:set nohlsearch</td><td>hlsearch 就是 high light search(高亮度搜寻)。 这个就是设定是否将搜寻的字符串反白的设定值。默认值是 hlsearch</td></tr>
<tr><td>:set autoindent<br>:set noautoindent</td><td>是否自动缩排？autoindent 就是自动缩排</td></tr>
<tr><td>:set backup</td><td>是否自动储存备份档？一般是 nobackup 的， 如果设定 backup 的话，那么当你更动任何一个档案时，则源文件会被另存成一个档名为 filename~ 的档案。</td></tr>
<tr><td>:set ruler</td><td>右下角的一些状态栏说明吗？ 这个 ruler 就是在显示或不显示该设定值</td></tr>
<tr><td>:set showmode</td><td>是否要显示 --INSERT-- 之类的字眼在左下角的状态栏</td></tr>
<tr><td>:set backspace=(0 1 2)</td><td>一般来说， 如果我们按下 i 进入编辑模式后，可以利用退格键 (backspace) 来删除任意字符的。 但是，某些 distribution 则不许如此。此时，我们就可以透过 backspace 来设定，当 backspace 为 2 时，就是可以删除任意值；0 或 1 时，仅可删除刚刚输入的字符， 而无法删除原本就已经存在的文字了！</td></tr>
<tr><td>:set all</td><td>显示目前所有的环境参数设定值。</td></tr>
<tr><td>:set</td><td>显示与系统默认值不同的设定参数， 一般来说就是你有自行变动过的设定参数</td></tr>
<tr><td>:syntax on<br>:syntax off</td><td>是否依据程序相关语法显示不同颜色</td></tr>
<tr><td>:set bg=dard<br>:set bg=light</td><td>可用以显示不同的颜色色调，预设是『 light 』</td></tr>
</table>

整体 vim 的设定值一般放置在 /etc/vimrc 这个档案，不过不建议修改，可以修改 ~/.vimrc (预设不存在，需要自行手动建立)，将所希望预设的设定值写入。这个档案中，设定参数命令，前面有没有冒号 【:】 效果都相同，双引号则是批注符号。

# 使用 vim 的注意事项
## 中文编码问题
中文编码有 big5 与 utf8 两种
1. Linux 系统默认终端支持的语系数据：与 /etc/sysconfig/il8n 有关；
2. 终端介面 (bash) 的语系：与 LANG 这个变数有关
3. 档案原本的编码
4. 开启终端机的软件，如 GNOME 下的窗口接口
## DOS 与 Linux 的断行字符
 DOS(Windows 系统) 使用的断行字符为 ^M$ ，我们称为 CR 与 LF 两个符号。 而在 Linux 底下，则是仅有 LF ($) 这个断行符号  
 在 Linux 底下指令开始执行时，判断依据是 【Enter】，而 Linux 的 Enter 为 LF 符号，若是 DOS 的断行符号 CRLF 则多处一个 ^M 符号，如果是一个 shell script 程序文件，将造成【程序无法执行】  
断行符号格式转换：
```
[root@www ~]# dos2unix [-kn] file [newfile]
[root@www ~]# unix2dos [-kn] file [newfile]
选项与参数：
-k  ：保留该档案原本的 mtime 时间格式 (不更新档案上次内容经过修订的时间)
-n  ：保留原本的旧档，将转换后的内容输出到新档案，如： dos2unix -n old new
```
## 语系编码转换
iconv 指令
```
[root@www ~]# iconv --list
[root@www ~]# iconv -f 原本编码 -t 新编码 filename [-o newfile]
选项与参数：
--list ：列出 iconv 支持的语系数据
-f     ：from ，亦即来源之意，后接原本的编码格式；
-t     ：to ，亦即后来的新编码要是什么格式；
-o file：如果要保留原本的档案，那么使用 -o 新档名，可以建立新编码档案。
```
```
将繁体中文的 utf8 转换成简体中文的 utf8 编码
iconv -f utf8 -t big5 vi.utf8 | iconv -f big5 -t gb2312 | iconv -f gb2312 -t utf8 -o vi.gb.utf8
```