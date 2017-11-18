---
title: shell基础
date: 2017-03-12 15:13:38
tags: shell
categories: linux
copyright: true
---

<img src="/images/shell.png" width = "50%" height = "30%" alt="shell" align=center />
#### 前言
Shell 是一个用 C 语言编写的程序，它是用户使用 Linux 的桥梁。Shell 既是一种命令语言，又是一种程序设计语言。
Shell 是指一种应用程序，这个应用程序提供了一个界面，用户通过这个界面访问操作系统内核的服务。
<!-- more -->
#### Shell基本介绍

##### shell学习必备基础

1.  Linux的基本使用
2.  如何在bash上执行程序
3.  简单的管道传输
4.  使用 &将程序放在后台执行

##### 入门

**为什么要使用shell脚本?**

>   使用脚本编程语言的好处是,脚本语言多半运行在比编译语言还高得层级,能够轻易处理文件与目录之类的对象.
缺点:一般情况下,效率比较低.不过权衡之下,脚本的执行速度已经很快,快到足以让人感觉不到性能不高了.
常用的脚本编程语言有:shelll,Ruby,JavaScript等.
>
>   shell似乎是不同版本的linux系统之间的通用功能.shell脚本只要用心写,就能应用到很多系统上.

shell脚本的过人之处
>   简单性:shell是高级语言
>   可移植性:通过POSIX(可移植操作系统接口，是IEEE为要在各种UNIX操作系统上运行的软件，而定义API的一系列互相关联的标准的总称)所定义的功能,可以在不同的系统上执行,无需需改.
>   开发容易:短时间即可完成一个功能强大又好用的脚本(字啊以后的学习中就能看到)

##### 简单的shell脚本

`who` 能看到以下内容

>   [root/shell] ]$cat 1.sh 
>   \#!/bin/bash
>   who
>
>   [root/shell] ]$./1.sh 
>   root     :0           2016-09-19 13:53 (:0)
>   root     pts/0        2017-02-13 12:49 (:0)
>   root     pts/2        2017-02-19 20:57 (192.168.247.1)

表示系统当前有多少人登陆，登陆用户-使用的终端-登陆时间&登出时间

`who | wc -l`  统计当前登陆了多少人



##### 脚本第一行

`#/bin/bash` 指定运行此脚本的程序

也可以执行一些独立的程序 比如：`#/bin/awk -f`



>   以下是几个初级的陷阱:
>   1.对#!这一行的长度尽量不要超过64个字符
>   2.脚本的可移植性取决于是否有完整的路径名称
>   3.不要在选项之后放置任何空白,因为空白也会跟着选项一起传递给被引用的程序.
>   4.需要知道解释器的完成路径的名称.这样可以规避可移植性的问题,厂商不同,同样的东西可能放在不同的地方
>   5.一些较久的系统,内核不具备#!的能力,有些shell会自行处理,这些shell对于#!与紧随其后的解释器名称之间是否可以有空白,可能有不同的解释.
>
>   查看当前发行版本可以使用的shell:cat  /etc/shells
>   查看系统默认的shell:echo $SHELL:一般情况下是输出/bin/bash.
>   如果想切换shell的版本,只需要直接输入shell的版本.例如想使用csh,直接输入csh即可,使用exit退出当前shell回到原shell.



##### shell的基本元素

**shell识别三种基本命令:**

1.  内建命令:就是linux的命令,例如cd,ls,mkdir等,这些命令是由于其必要性才内建的,内外一种命令的村子啊是为了效率,其中最典型的就是test,
2.  shell函数:功能健全的一系列程序代码,用shell语言写成,可以像使用命令一样使用,就是在C++中调用函数.
3.  外部命令:是由shell的副本(新的进程)所执行的命令,还是命令.



**shell中的变量**
在shell中,变量值可以是(通常是)空值,也就是不含有任何字符.这是合理的,也是常见的,好用的特性.空值就是null
在shell中变量名的长度无限制,所能保存的字符数同样没有限制.
变量的赋值方式:变量名=值,中间不能有任何的空格,如果想去除shell变量的值时,需要在变量名前加上$字符.当所赋予的值包含空格的时候,需要将值用单引号或者双引号包起来,用单引号包起来的后果是单引号里面的所有特殊符号都不具备特殊含义,用双引号包起来代表特殊符号有特殊含义.
例如:
```
name=syx;
echo 'name'  #输出 name
echo "$name" #输出 syx
```
如果想将 name1=syx,name2=zsf合并,成syxzsf
则`name=${name1}${name2},echo $name name=syxzsf`,貌似还有其他的合并方法,个人觉得这一种最好.
至于变量的四种类型什么的,暂时不学。

#### 简单的echo输出

echo的作用就是产生输出,可以提示用户,或者用来产生数据提供用户,或者产生数据进一步处理.

以前的echo只能将参数打印到shell交互界面上,参数之间以一个空格隔开,并以换行符号结尾.
后来又衍生出了-n选项,省略结尾的换行符号.
>    etho的语法:
>    etho [string......]

用途是产生shell脚本的输出,没有什么主要选项.行为模式是将参数打印到标准输出,参数之间用空格隔开,并以换行符结尾.转义序列用来表示特殊字符,以及控制其行为模式.

参　　 数：

-n 不要在最后自动换行
-e 若字符串中出现以下字符，则特别加以处理，而不会将它当成一般文字输出：
常用的转移序列:

```
\a  #发出警告声
\b  #删除前一个字符
\c  # 最后不加上换行符号；
\f  #换行但光标仍旧停留在原来的位置
\n  #换行
\r  #回车
\t  #水平制表符
\v  #垂直制表符
\\  #反斜杠字符
```



#### 华丽的printf 输出

如同echo命令,printf命令可以输出简单的字符串，但是printf 不提供自动换行

##### 语法

`printf format-string [arguments...]`

```
printf "Hello, Shell\n"
Hello, Shell
```

```
# format-string为双引号
printf "%d %s\n" 1 "abc"
1 abc
# 单引号与双引号效果一样 
printf '%d %s\n' 1 "abc" 
1 abc
# 没有引号也可以输出
printf %s abcdef
abcdef
# 格式只指定了一个参数，但多出的参数仍然会按照该格式输出，format-string 被重用
printf %s abc def
abcdef
printf "%s\n" abc def
abc
def
printf "%s %s %s\n" a b c d e f g h i j
a b c
d e f
g h i
j
# 如果没有 arguments，那么 %s 用NULL代替，%d 用 0 代替
printf "%s and %d \n" 
and 0
# 如果以 %d 的格式来显示字符串，那么会有警告，提示无效的数字，此时默认置为 0
printf "The first program always prints'%s,%d\n'" Hello Shell
-bash: printf: Shell: invalid number
The first program always prints 'Hello,0'
```



#### I/O重定向

##### 标准的输入输出

在了解重定向之前,需要先了解一下标准的输入输出,总的来说,所有的数据都有来源,也都应该都重点,默认的标准输入输出就是终端.
例如:
我们只是输入  cat命令,并不指定任何参数,接着我们输入hello world,就是打印helloworld 到终端.

>   [root~] ]$cat
>   test #输入的
>   test #输出的

**所谓的I/O重定向就是通过与终端交互,或是在shell脚本里设置,重新安排从哪里输入或者输出到哪里.**



##### 重定向与管道

**<**    改变标准输入 
**\>**  改变标准输出

重定向符号在目的地文件不存在的时候会新建一个文件，如果存在，则会覆盖！
**>>** 此重定向符号为追加
**|** 管道可以将 标准输出 改为标准输入  如 `cat /etc/paswd | wc -l`

>   在构造管道的时候,应该试着让每个阶段的数据量变少,也就是说,把会让数据变少的命令放在前边,提高后面的命令的执行效率.例如,sort之前,先用grep找出相关的行,这样可以让sort少做些事.



#### /dev/null和/dev/tty

**/dev/null**

>   当被用作重定向输出时，程序的输出被直接丢弃。该文件用在那些不关心程序输出的地方。
>   当被用作重定向输入时，输入则是文件结束。
**/dev/tty**
>   当被用作重定向时，表示重定向到终端。

#### shell基本命令查找与添加

>   shell会沿着\$PATH来寻找命令.\$PATH是一个以冒号分割的目录列表,你可以在列表所指定的目录下找到所要执行的命令.命令可能是shell脚本,也可能是编译后的可执行文件,从用户角度来看,二者并无不同.
>
>   默认路径至少包含/bin和/usr/bin,或许还包含其他的.
>   名称为bin的目录用来保存可执行文件.
>   如果要编写自己的脚本,最好准备一个自己的bin目录来存放他们,并且让shell能够自动找到他们.
>   要想永久生效,在/etc/profile文件中把你的bin目录加入到\$PATH,而每次登陆时Shell都将读取.profile文件.
>   PATH=\$PATH:[你存放脚本的目录]   如“\$PATH:/usr/local/myshell
>   $PATH里的空项目表示当前项目.空项目位于路径中间时,可以用两个连续的冒号来表示,如果将冒号直接置于最前端或尾端,分别表示查找的时候最先查找或最后查找当前目录.

>   \$PATH=:/bin:/usr/bin            先找当前目录
>   \$PATH=/bin::/usr/bin            当前目录居中
>   \$PATH=/bin:/usr/bin:            最后找当前目录
>   不应该在查找路径中放进当前项目.

#### 访问脚本的参数

比如我们想查看当前某个用户是否登陆，那么新建脚本 finduser.sh

```
#!/bin/bash
who | grep $1
```

```
./finduser/sh root
$1 就是指执行脚本传入的第一个参数 root
root     :0           2016-09-19 13:53 (:0)
root     pts/0        2017-02-13 12:49 (:0)
root     pts/2        2017-02-20 20:45 (192.168.247.1)
```

理想状态下，是这样的，但是如果用户不输入参数，脚本就会报错了，所以更好的情况是把脚本可能出现的事件都做个判断以及友情的提示，更加智能化



#### 基本的正则表达式(BRE)

##### 简单的正则举例
```
tolstoy: 匹配一行上任意位置的7个字母:tolstoy
^tolstoy: 7个字母tolstoy,出现在一行的开头
tolstoy$: 出现在一行的结尾
^tolstoy$: 正好包含这7个字母的一行,没有其他的任何字符.
[tT]olstoy: 在一行的任意位居中,含有Tolstoy或者tolstoy
tol.toy:在一行的任意位居中,含有tol这三个字母,加上一个特殊字符,在接着toy这三个字母
tol.*toy:在一行的任意位居中,含有tol这三个字母,加上任意的0或者多个字符,
 再继续toy这三个字母(例如:toltoy,tolstoy,tolWHOtoy都是满足要求的).
```
##### shell中的通配符

```
*: 代表0个或者多个任意字符
?: 代表一定有一个的任意字符
[]: 代表一定有一个在括号内的字符(非任意字符).例如[abcd]代表一定有一个字符,可能是abcd这四个选项的任意一个.
[-]: 代表在编码顺序内的所有自负.例如:0-9代表0到9之间的所有数字,因为数字的语系编码是连续的.
[^]: 若括号内的第一个字符为指数字符\(^) 表示反选,例如:\^abc代表是非abc的其他字符就可以.
```


#####shell中的特殊字符
```
#:   注释字符
\:   将特殊字符或者通配符还原成一般字符
|:   管道符,分割两个管线命令的界定
;:   连续命令下达分隔符
~:   用户的家目录
$:   放在变量前面,正确使用变量
&:   工作控制,将命令编程背景下工作
!:   非(!)的意思,逻辑运算符
>,>>:   输出重定向,分别是覆盖和追加
><,<<:   输入重定向
>'':   单引号,不具有变量置换的功能
>"":   双引号,具有变量置换的功能
>():   在中间的为子shell的起始与结束
>{]:   在中间为命令块的组合
```

#####shell中正则表达式的控制字符

```
^:     匹配行首位置
$:     匹配行尾位置
.:     匹配任意祖父
*:     对*之前的匹配整体或字符匹配任意次(包括0次)
\?:     对\?之前的匹配整体或字符匹配0次或1次
\{n\}:     对 \ { 之前的匹配整体或字符匹配n次
\{m,\}:     对 \ { 之前的匹配整体或字符匹配至少m次
\{m,n}:     对 \ { 之前的匹配整体或字符匹配m到n次 
[abcdef]:   对单字符而言匹配中的字符
[a-z];   对单字符而言,匹配任意一个小写字母
[^a-z]:   不匹配括号中的内容
```

##### 基本正则表达式

###### 匹配单个字符

>   1.匹配一般字符:一般字符是指无特殊含义的字符,包括所有文本和数字字符,绝大多数的空白字符以及标点符号字符,因此,正则a,匹配a.

>   2.如果相匹配*,因为*是特殊字符,所以需要用 \ 转义,正则\*,匹配*.
>
>   3.\.(点号)字符意即”任意字符”,例如a.c匹配于abc,aac.
>
>   4.使用方括号表达式.例如x[abcdefg]z,可以匹配xaz,xbz,等,方括号里如果存在(^),表示取反的意思,就是说不匹配列表里的任意字符.
>
>   [0123456789]表示所数字,但是这样写太麻烦,我们可以用[0-9]来表示,[abcdefg]同样可以用[a-g]

###### 单个表达式匹配多字符

>   最简单的办法就是把它们一一列出来:正则abc匹配于abc.
>   虽然(.)meta字符与方括号表达式都提供了依次匹配一个字符的很好方式,单正则真正强大而有力地功能是修饰符meta字符的使用上.
>   最常用的修饰符是(\*),表示匹配0个或多个前面的单个字符.因此ab\*c表示”匹配一个a,0个或多个b字符以及a空c”.这个正则匹配的有ac,abc,abbcabbbbc.
>   匹配0或多个,不表示匹配其他的某一个.例如正则ab*c,文本aQc是不匹配的.但是ac是匹配的.
>   (*)修饰符虽然好用,但是他没有限制,如要只要指定次数,使用一个复杂的方括号表达式虽然也能指定次数,但是太过麻烦.我们就引入了区间表达式.所谓的区间表达式有三种变化
>   `\{n\}`  前置正则表达式所得结果重现n次
>   `\{n,\}` 前置正则表达式所得结果至少出现n次
>   `\{n,m\}` 出现n到m次
>   例如我们想要表达”重现5个`a' =>a\{5\}'`,重现10到42个`q'=>q\{10,42\}'`;

###### 文本匹配锚点

>   两个meta字符是脱节符号(^),与货币字符($),他们叫做锚点,因为其用途在限制正则表达式匹配时,针对要被匹配字符的开始或者结尾处进行匹配,
>   假定有一串字符串:abcABCdefDEF

| 模式                | 是否匹配 | 理由         |
| ----------------- | ---- | ---------- |
| ABC               | 是    | 居中的456匹配   |
| ^ABC              | 否    | 起始不是ABC    |
| def               | 是    | 居中的def匹配   |
| def$              | 否    | 结尾不是def    |
| [[:upper:]]\{3\}  | 是    | 结尾的大写ABC匹配 |
| [[:upper:]]\{3\}$ | 是    | 结尾的大写DEF匹配 |
| ^[[:alpha:]]\{3\} | 是    | 结尾的大写ABC匹配 |

#####BRE运算优先级,由高到低
```
[..]\[==][::] -----用于字符拍的方括号符号
\metacharacter ----转移的meta字符
[] --------------- 方括号表达式
\{ \}  ------------子表达式
*  \{ \ } ---------前置单个字符重现的正则表达式
无符号--------------连续
^$  -------------- 锚点
```


#### 扩展正则表达式(ERE)

  BRE与ERE在大多数的meta字符与功能应用上几乎是完全一致,单ERE理由写meta字符看起来与BRE类似,却具有完全不同的类型.

>   扩展正则表达式与基础正则表达式的唯一区别在于：? + () {} 这几个字符。
>   基础正则表达式中，如果你想? + () {}表示特殊含义，你需要将他们转义
>   而扩展正则表达式中，如果你想? + () {} 不表示特殊含义，你需要将他们转义。
>   转义符号，都是一样的，\符号。
>   所谓特殊含义，就是正则表达式中的含义。非特殊含义，就是这个符号本身。

例如 
```
   echo aaa|grep 'a?';
   echo aaa|grep 'a\?'; \#aaa
   \#egrep使用的是扩展正则表达式
   echo aaa|egrep 'a?';     \#aaa
   echo aaa|egrep 'a\?';
```


##### 打印

>   如果希望打印文件,最好预先处理一下,包括调整边距,设置行高,设置标题等,这样打印出来的文件更加美观.当然,不处理也能打印,但是可能会比较丑陋.

  pr命令
pr命令就是转换文件格式的,可以把较大的文件分割成多个页面进行打印,并未每个页面添加标题.
 语法:
``` pr option(s) filename(s)```

常见选项:
```
>   -k:分成激烈打印,默认为1
>   -d:两倍行距(并不是所有版本的pr都有效)
>   -h “title” 设置每个文件的标题
>   -l PAGE_LENGTH :每页显示多少行.默认是每个页面一共66行.
>   -o MARGIN:每行缩进的空格数
>   -w PAGE_WIDTH:多列输出时,设置页面宽度,默认是72个字符.
```
例如我有一个文件food,里面的内容为:
>   Sweet Tooth
>   Bangkok Wok
>   Mandalay
>   Afghani Cuisine
>   Isle of Java
>   Big Apple Deli
>   Sushi and Sashimi
>   Tio Pepe's Peppers

```
   使用命令:pr -2 -h "food" food
   输出结果为:
   2015-06-22 12:27                      food                      
   weet Tooth                          Isle of Java
   Bangkok Wok                         Big Apple Deli
   Mandalay                            Sushi and Sashimi
   Afghani Cuisine                     Tio Pepe's Peppers'
```

 解释:
>pr会以文件的修改时间作为页面标题的时间戳;如果输入时自管道而来,则使用当前的时间,接上文件名称(如果输入的数据内容在管道中,则为空)以及页码.


>   lp 和 lpr 命令将文件传送到打印机进行打印。使用 pr 命令将文件格式化后就可以使用这两个命令来打印。

例如:
``` pr -2 -h "food" food | lpr```
>   命令成功执行会返回一个表示打印任务的ID，通过这个ID可以取消打印或者查看打印状态。
>
>   如果你希望打印多份文件，可以使用 lp 的 -nNum 选项，或者 lpr 命令的 -Num 选项。Num 是一个数字，可以随意设置。
>
>   如果系统连接了多台打印机，可以使用 lp 命令的 -dprinter 选项，或者 lpr 命令的 -Pprinter 选项来选择打印机。printer 为打印机名称。
>   lpstat 和 lpq 命令
>   lpstat 命令可以查看打印机的缓存队列（有多少个文件等待打印），包括任务ID、所有者、文件大小、请求时间和请求状态。
>
>   提示：等待打印的文件会被放到打印机的的缓存队列中。
>
>   使用 lpstat -o 命令查看打印机中所有等待打印的文件，lpstat -o 命令按照打印顺序输出队列中的文件。
>
>   cancel 和 lprm 分别用来终止 lp 和 lpr 的打印请求。使用这两个命令，需要指定ID（由 lp 或 lpq 返回）或打印机名称。
>   lprm 命令用来取消当前用户的正在等待打印的文件，使用任务号作为参数可以取消指定文件，使用横线(-)作为参数可以取消所有文件。
>   lprm 会返回被取消的文件名。
>



#### 提取文件开头或者结尾

##### head & tail

>   个人觉得最好用的显示文本文件的头几行最好用的是 head -n [file(s)]

  head的常用选项:
```
>   -q: 隐藏文件名
>   -v: 显示文件名
>   -c<字节>: 显示字节数
>   -n<行数>: 显式的行数
```
>   在交互式shell通信期中,有时需要监控某个文件的输出----如日志这类持续写入状态的文件.-f选项这时就派上用场了,他可以要求tail显示指定的文件结尾行数,接着进入无止境的循环中----休息一秒后又再度醒来并检查是否需要显示更多的输出结果.再设置-f的状态下,tail只有当你中断它时才会停止----通常是输入Ctrl+C来中断;

```tail -n 25 -f /var/log/messages ```    此选项不可用于shell脚本.
>   直到按了ctrl+c选项后才停止.
>   由于tail加上-f选项之后便不会自己中断,所以此选项不能用于shell脚本.使用-f选项有实时监听的效果.

 head案例:
```
>   使用命令:head -n 3 /etc/passwd结果是显示文件的头三行,
>   如果命令为:head -n -3 /etc/passwd 结果是显示除了最后三行都显示,注意到区别没有?
>   相似的,显示文件的前n个字节,以及除了最后n个字节以外的内容也没问题了.
>   head和tail如果组合使用:
>   head -n 5 /etc/passwd | tail -n 3
>   输出/etc/passwd的第三道第五行.
```


#### shell变量与算数

test.sh

name=vic #定义一个变量

readonly name #变量只读

unset name  #删除变量

sleep 5 #等待5秒

##### 参数的展开

```
var="hello world"
echo ${var}
#hello world
#所谓的参数的展开，基本上就是变量，shell中，参数是变量的超集，只不过变量不能以数字开头，而参数可以，比如$1，表示传递的第一个参数
```

##### 位置参数

```
$0           #程序名
$1到$9       #直接表示
${10}        #大于9的时候要用花括号

$*          #接收所有参数，
$@          #传递给脚本或函数的所有参数。被双引号(" ")包含时，与 $* 稍有不同
#但是当它们被双引号(" ")包含时，"$*" 会将所有的参数作为一个整体！以"$1 $2 … $n"的形式输出所有参数；"$@" 会将各个参数分开，以"$1" "$2" … "$n" 的形式输出所有参数。

$?          #上个命令的退出状态，或函数的返回值
$$          #SHELL的进程ID
$#          #参数个数
```





##### 展开运算符（参数的运算符）

```
${varname:-zili}
#如果varname存在且不是NULL，返回varname，反之，设为zili。可用于设置变量默认值！！

${varname:+zili}
#如果varname存在且不是NULL，返回varname，反之，返回null
```

##### 模式运算符

```
var=br1br2ead
echo ${var$$*br}
输出:2ead
echo ${var#*br}
输出:1br2ead

${parameter%word}或${parameter%%word}
作用:与前例相似,唯一不同的是从$parameter的为不开始匹配.
var="La.Maison.en.Petits.Cubes.avi"
echo ${var%.*}
输出:La.Maison.en.Petits.Cubes
echo ${var%%.*}
输出:La

分析:匹配案例中的”.”时,shell会从$var的尾部开始查找”.”,如果是最短匹配(echo ${var%.*}),则会找到第一个”.”就停止,否则(echo ${var%%.*})会一直找到最后一个”.”才停止.可以看到,这种用法可以分方便的去掉文件后缀,从而得到文件名.
```

#### 退出状态和 if 语句

##### 退出状态

>   每一条命令,不管是内置的,shell函数,还是外部的,当它退出时,都会返回一个小的整数值给引用它的程序,这就是程序的退出状态.在shell下执行进程有很多方式可取用程序的退出状态.

>   以管理状态来说，0 表示成功，也就是当$? 返回的值为0的时候，证明程序执行成功，其他为失败

```
0       #命令成功地退出
>0      #在重定向或单词展开期间(~,变量,命令,算数展开,以及单词切割)失败
1-125   #命令不成功的退出.特定的退出值的含义,是由各个单独的命令定义的.
126     #命令找到了,但文件无法执行
127     #命令没找到
>128    #命令因受到信号而死亡
```

###### exit

```
cd $(dirname $0) || exit 1
##进入脚本所在目录 否则退出
```

```
if [ $# -ne "2" ];
    then
    echo ''
    exit 2
fi
```

##### if语法

```
if [xxxx];then
    xxxx
fi


if [xxxx];then
    xxxx
    else
        xxxx
fi


if [xxxx];then
    xxxx
    elif [xxxx];then
    xxxx
....
....
else
    xxxx
fi
```

```
#!/bin/bash
read -p "what is your backup directoy : " BakDir
if [ -d $BakDir ];then
        echo "$BakDir alerdy exist"
else
        echo "$BakDir is not exist,will make it"
        mkdir $BakDir
fi
```

```
#!/bin/bash
UserNum=`who | wc -l` 
if [ $UserNum -gt 3 ];
        then
        echo "Alert, too many login users ( Total: $UserNum)."
else
        echo "Login Users:"
        who | awk '{print $1,$2}'
fi
```

**注意**

> if 与[ 之间必须有空格
>
>  [ ]与判断条件之间也必须有空格
>
> ]与; 之间不能有空格


#### 逻辑判断

##### 字符串判断

```
str1 = str2　 　　当两个串有相同内容、长度时为真
str1 != str2　　　当串str1和str2不等时为真
-n str1　　　　　 当串的长度大于0时为真(串非空)
-z str1　　　　　 当串的长度为0时为真(空串)
```



##### 数字的判断

```
int1 -eq int2　　　　两数相等为真
int1 -ne int2　　　　两数不等为真
int1 -gt int2int1大于int2为真
int1 -ge int2int1大于等于int2为真
int1 -lt int2int1小于int2为真
int1 -le int2int1小于等于int2为真
```

##### 文件的判断

```
-r file　　　　　用户可读为真
-w file　　　　　用户可写为真
-x file　　　　　用户可执行为真
-b file            若文件存在且是一个块特殊文件，则为真
-c file            若文件存在且是一个字符特殊文件，则为真
-d file            若文件存在且是一个目录，则为真
-e file            若文件存在，则为真
-f file            若文件存在且是一个规则文件，则为真
-g file            若文件存在且设置了SGID位的值，则为真
-h file            若文件存在且为一个符合链接，则为真
-k file            若文件存在且设置了"sticky"位的值
-p file            若文件存在且为一已命名管道，则为真
-s file            若文件存在且其大小大于零，则为真
-u file            若文件存在且设置了SUID位，则为真
-o file            若文件存在且被有效用户ID所拥有，则为真
```

##### 复杂的逻辑判断

```
-a 　 　　　　　与
-o　　　　　　　或
!　　　　　　　 非
```

#####test的使用

```
#!/bin/bash
cd /bin
if test -e ./bash   //其实这里相当于if [ -e ./bahs ]
then
        echo 'the file already exist!'
else
        echo 'the file not exist!'
fi
输出结果为:the file already exist!
```

##### 注意和简写与扩展

###### 注意

```
if [ -n "$str" -a -f "$file" ]      一个test命令,两种条件
if [-n "str"] && [ -f "$file" ]      两个命令,一块接方式计算
if [-n "$str" && -f "$file"]        语法错误！！！
```

###### 简写

```
[1 eq1 ] &&echo'OK'
输出:ok

[ 2 < 1 ] &&echo 'OK'
输出:-bash: 1: No such file or directory
使用命令:[ 2 \< 1 ] &&echo 'OK'这样就可以了

使用命令:[ 2 -gt 1 -a 3 -lt 4 ]&&echo 'Ok'
输出:Ok

使用命令:[ 2 -gt 1 && 3 -lt 4 ]&&echo 'Ok'  
输出:-bash: [: missing `]'

注意：在[] 表达式中，常见的>,<需要加转义字符，表示字符串大小比较，以acill码位置作为比较。不直接支持<>运算符，还有逻辑运算符 || 和 && 它需要用-a[and] –o[or]表示。
```

###### 扩展 [[]]

```
[[ 2 < 3 ]]&&echo 'OK'
输出OK.

[[ 2 < 3 && 4 < 5 ]] && echo 'ok'
输出:ok

注意：[[]] 运算符只是[]运算符的扩充。能够支持<,>符号运算不需要转义符，它还是以字符串比较大小。里面支持逻辑运算符 || 和 && bash 的条件表达式中有三个几乎等效的符号和命令：test，[]和[[]]。通常，大家习惯用if [];then这样的形式。而[[]]的出现，根据ABS所说，是为了兼容><之类的运算符。

不考虑对低版本bash和对sh的兼容的情况下，用[[]]是兼容性强，而且性能比较快，在做条件运算时候，可以使用该运算符。
```

#### case

```shell
#!/bin/sh
read -p 'input a number 1 to 4' unum
case $unum in
    1) echo 'Number is 1'
    ;;
    2|3) echo 'Number is 2 or 3'
    ;;
    4) echo 'Number is 4'
    ;;
    *) echo '1 to 4,Please'
    ;;
esac

```

>  *) 相当于其他语言中的default。
>  除了*)模式，各个分支中;;是必须的，;;相当于其他语言中的break
>   | 分割多个模式，相当于or



#### 循环

##### for

>   for循环是将串行的元素取出,依次放入指定的变量中,重复执行在do和done之间的命令,直到所有元素取尽为止

```
for  变量 in  串行
do
....
done


for((i=0;i<100;i++))
do
....
done
```

```
#!/bin/sh
for i in $(seq 1 10)
do
    mkdir /tmp/test${i}
    cd /tmp/test${i}
    for j in $(seq 1 5)
        do  
            touch test${j}
        done
done
```

```
#!/bin/sh
dir="/var"
cd $dir
for i in $(ls $dir)
do  
    if [ -d $i ];then
        du -sh $i
     fi  
done
```

##### while

```
while 条件测试
do
....
done
```

```
#!/bin/sh
sum=100
i=0
while [ $i -le 100 ]
    do
        sum=$(($sum+$i))
        i=$(($i+1))
    done
echo $sum     
```

##### until

```
until 条件测试
do
....
done

#条件为真时，循环停止
```

```
sum=0
i=1
until ((i>100))
do
        sum=$(( $sum+$i ))
        i=$(( $i+1 ))
done
echo $sum
```

#### 退出/跳出循环

##### break

>   直接打断循环

##### continue

>   跳过本次循环

##### shift

>   shift每执行一次，参数指针像右移动一位，

```
#!/bin/bash
until [ $# -eq 0 ]
do
    echo "第一个参数为: $1 参数个数为: $#"
    shift
don

第一个参数为: 1 参数个数为: 4
第一个参数为: 2 参数个数为: 3
第一个参数为: 3 参数个数为: 2
第一个参数为: 4 参数个数为: 1
```

##### getopts

>   以后研究




































#### Linux相关命令详解

##### grep

######语法

grep (option) [file]

######选项

>   -A 1 表示找到所有匹配行，并显示所有匹配行后的一行
>   -B 1  表示找到所有匹配行，并显示所有匹配行的前面一行
>   -C 1表示找到所有匹配行，并显示所有匹配行的前一行，后一行
>   -V:显示不匹配的行
>   -a 表示把所有文件当作ASCII文件来处理 搜索二进制文件
>   -b 表示显示match的字符串在文件中的offset
>   -c 显示有多少行match
>   -e 后面跟一个正则表达式，指定多个正则表达式的时候很有用
>   -f可以指定pattern在我们的文件中   pattern文件中的每一行都会来进行匹配
>   -i:模式匹配时忽略大小写
>   -l 出匹配模式的文件名称,而不是打印匹配的行
>   -m 最多匹配几个后，就停止，这样速度会比较快
>   -n 匹配之后，在前面打印行号，这个还是有用的
>   -o 只打印匹配的内容
>   -R 搜索子目录

######参数

文件，文档等

###### 实例


>grep -n 'root' /etc/passwd
>
>grep -m 1 'root' /etc/passwd






##### tr

######语法
`tr (选项) (参数)`

######选项

>   -c或——complerment：取代所有不属于第一字符集的字符；
>   -d或——delete：删除所有属于第一字符集的字符；
>   -s或——squeeze-repeats：把连续重复的字符以单独一个字符表示；
>   -t或——truncate-set1：先删除第一字符集较第二字符集多出的字符。

######参数

>   字符集1：指定要转换或删除的原字符集。当执行转换操作时，必须使用参数“字符集2”指定转换的目标字符集。但执行删除操作时，不需要参数“字符集2”；
>   字符集2：指定要转换成的目标字符集。

######实例

输入的字符串大写转成小写：

```
echo "HELLO WORLD" | tr 'A-Z' 'a-z'
hello world
#由于是转换，所以必须要有字符集2 也就是'a-z'
```

用tr压缩字符，可以压缩输入中重复的字符：

```
echo "thissss is      a text linnnnnnne." | tr -s ' sn'
this is a text line.
```

##### sed

>   

######语法

>   sed的命令格式： sed [option] 'sed command'filename
>   sed的脚本格式：sed [option] -f 'sed script'filename

######选项

>   -n ：只打印模式匹配的行
>   -e ：直接在命令行模式上进行sed动作编辑，此为默认选项
>   -f ：将sed的动作写在一个文件内，用–f filename 执行filename内的sed动作
>   -r ：支持扩展表达式
>   -i ：直接修改文件内容



###### 实例

```
sed -n '5p' /etc/passwd         #打印某一行
sed -n '5,8P' /etc/passwd       #打印5-8行
sed -n '/root/p' /etc/passwd     #匹配某字符的行
sed -n '/root/,5p' /etc/passwd   #以匹配某字符的行 到某行
sed -n '1,/adm/p' /etc/passwd    #匹配从哪里开始 以某关键字结尾的行
sed -n '1,4{=;p}' /etc/passwd    #打印带行号的
sed -n '1,4!{=;p}' /etc/passwd   # ！感叹号取反 不打印1-4行
```

###### sed 匹配正则 -r


####### 选项

```
^       锚点行首的符合条件的内容，用法格式"^pattern"
$       锚点行首的符合条件的内容，用法格式"pattern$"
^$      空白行
.       匹配任意单个字符
*       匹配紧挨在前面的字符任意次(0,1,多次)
.*      匹配任意长度的任意字符
?       匹配紧挨在前面的字符0次或1次
\{m,n\}  匹配其前面的字符至少m次，至多n次
\{m,\}   匹配其前面的字符至少m次
\{m\}    精确匹配前面的m次\{0,n\}:0到n次
\<      锚点词首----相当于 \b，用法格式：\<pattern
\>      锚点词尾，   用法格式:\>pattern
\<pattern\> 单词锚点

分组，用法格式：pattern，引用\1,\2
[]          匹配指定范围内的任意单个字符
[^]         匹配指定范围外的任意单个字符
[:digit:]    所有数字, 相当于0-9， [0-9]---> [[:digit:]]
[:lower:]    所有的小写字母
```



#######实例
```
#######sed的匹配模式支持正则表达式#####################  
sed '5 q' /etc/passwd               #打印前5行  
sed -n '/r*t/p' /etc/passwd          #打印匹配r有0个或者多个，后接一个t字符的行  
sed -n '/.r.*/p' /etc/passwd         #打印匹配有r的行并且r后面跟任意字符  
sed -n '/o*/p' /etc/passwd          #打印o字符重复任意次  
sed -n '/o\{1,\}/p' /etc/passwd      #打印o字重复出现一次以上  
sed -n '/o\{1,3\}/p' /etc/passwd     #打印o字重复出现一次到三次之间以上  
```

###### sed编辑命令

```
p   打印匹配行（和-n选项一起合用）
=   显示文件行号
a\  在定位行号后附加新文本信息
i\  在定位行号前插入新文本信息
d   删除定位行
c\  用新文本替换定位文本
w filename  写文本到一个文件，类似输出重定向 >
r filename  从另一个文件中读文本，类似输入重定向 <
s   使用替换模式替换相应模式
q   第一个模式匹配完成后退出或立即退出
l   显示与八进制ACSII代码等价的控制符
{}  在定位行执行的命令组，用分号隔开
n   从另一个文件中读文本下一行，并从下一条命令而不是第一条命令开始对其的处理
N   在数据流中添加下一行以创建用于处理的多行组
g   将模式2粘贴到/pattern n/
y   传送字符，替换单个字符
```

####### 实例

```
sed -n '/^#/p' /etc/profile      #打印以井号开头的 '/^#/!P'加感叹号表示没有注释的行
sed -n '/^#/!{/^$/!p}' /etc/profile #这样过滤所有的#开头和空白行
sed -e '/^#/d' -e '/^$/d' /etc/profile  #等同上面，-e 支持对单个文件进行不同的操作

sed '/root/s/^/SSSSSSSSSSSSS/' /etc/passwd  #匹配root的行，行首添加SSSSSS...
sed '/root/s/$/SSSSSSSSSSSSS/' /etc/passwd  #匹配root的行，行尾添加SSSSSS...

sed 's/root/& LLL/' /etc/passwd     #匹配root关键字，前面添加 LLL
sed 's/root/LLL &/' /etc/passwd     #匹配root关键字，后面添加 LLL

sed '/zabbix/i ZZZ' /etc/passwd     #匹配root关键字，前一行添加 ZZZ
sed '/zabbix/a ZZZ' /etc/passwd     #匹配root关键字，后一行添加 ZZZ
sed '/zabbix/a ZZZ \n OOO' /etc/passwd  #\n 可以用来换行

sed 's/^/DDDDDDDD/' /etc/passwd     #每行的开头添加 DDDDDD....
sed 's/$/DDDDDDDD/' /etc/passwd     #每行的结尾添加 DDDDDD....
sed '1，3s/^/#/' /etc/passwd     #每行的开头#  这个用来添加注释
```



###### sed删除

```
sed '/^#/d'     #删除#开头的
sed '/^#/!d'    #删除非#开头的
sed '/root/d'   #删除匹配root的字符
sed '/\<you>\/'  #删除包含you这个单词的行 \<>\用来定位单词
...
```

###### sed替换(脚本中较多使用)

```
#================源文件里面的内容===============================  
[root@jie1 ~]# cat test  
anonymous_enable=YES  
write_enable=YES  
local_umask=022  
xferlog_enable=YES  
connect_from_port_20=YES  
root:x:0:0:root:/root:/bin/bash  
bin:x:1:1:bin:/bin:/sbin/nologin  
daemon:x:2:2:daemon:/sbin:/sbin/nologin  
adm:x:3:4:adm:/var/adm:/sbin/nologin  
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin  
DEVICE="eth0"  
BOOTPROTO="static"  
HWADDR="00:0C:29:90:79:78"  
ONBOOT="yes"  
IPADDR=172.16.22.1  
NETMASK=255.255.0.0  
#======================================================================  
[root@jie1 ~]# sed -i '/DEVICE/c\Ethernet' test   
        #匹配DEVICE的行，替换成Ethernet这行  
[root@jie1 ~]# sed -i 's/static/dhcp/' test       
        #把static替换成dhcp(/,@,#都是前面所说的地址定界符)  
[root@jie1 ~]# sed -i '/IPADDR/s@22\.1@10.12@' test  
        #匹配IPADDR的行，把22.1替换成10.12由于.号有特殊意义所有需要转义  
[root@jie1 ~]# sed -i '/connect/s#YES#NO#' test   
        #匹配connect的行，把YES替换成NO  
[root@jie1 ~]# sed -i 's/bin/tom/2g' test         
        #把所有匹配到bin的行中第二次及第二次之后出现bin替换成tom  
[root@jie1 ~]# sed -i 's/daemon/jerry/2p' test    
        #把所有匹配到bin的行中第二次出现的daemon替换成jerry，并在生产与匹配行同样的行  
[root@jie1 ~]# sed -i 's/adm/boss/2' test         
        #把所有匹配到adm的行中仅仅只是第二次出现的adm替换成boss  
[root@jie1 ~]# sed -i '/root/{s/bash/nologin/;s/0/1/g}' test  
        #匹配root的行，把bash替换成nologin，且把0替换成1  
[root@jie1 ~]# sed -i 's/root/(&)/g' test                   
        #把root用括号括起来，&表示引用前面匹配的字符  
[root@jie1 ~]# sed -i 's/BOOTPROTO/#BOOTPROTO/' test        
        #匹配BOOTPROTO替换成#BOOTPROTO，在配置文件中一般用于注释某行  
[root@jie1 ~]# sed -i 's/ONBOOT/#&/' test                   
        #匹配ONBOOT的行的前面添加#号，在配置文件中也表示注释某行  
[root@jie1 ~]# sed -i '/ONBOOT/s/#//' test                  
        #匹配ONBOOT的行，把#替换成空，即去掉#号，也一般用作去掉#注释  
#================执行以上sed命令之后文件显示的内容====================  
[root@jie1 ~]# cat test  
anonymous_enable=YES  
write_enable=YES  
local_umask=022  
xferlog_enable=YES  
connect_from_port_20=NO  
(root):x:1:1:(root):/(root):/bin/nologin  
bin:x:1:1:tom:/tom:/stom/nologin  
daemon:x:2:2:jerry:/sbin:/stom/nologin  
daemon:x:2:2:jerry:/sbin:/stom/nologin  
adm:x:3:4:boss:/var/adm:/sbin/nologin  
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin  
Ethernet  
#BOOTPROTO="dhcp"  
HWADDR="00:0C:29:90:79:78"  
ONBOOT="yes"  
IPADDR=172.16.10.12  
NETMASK=255.255.0.0  
```

###### sed 引用变量  (脚本中也较多使用)

**第一种当sed命令里面没有默认的变量时可以把单引号改成双引号**
**第二种当sed命令里面有默认的变量时，自己定义的变量需加单引号，且sed里面的语句必须用单引**

```
  #!/bin/sh
  my_name=li
  sed -i 's/'bbb' /'$my_name'/' /shell/myfile  
```

###### sed的其他用法

**操作一个文件，并写入到另一个文件**
```
[root@jie1 ~]# cat test   #sed操作的文件中的内容  
Ethernet  
#BOOTPROTO="dhcp"  
HWADDR="00:0C:29:90:79:78"  
ONBOOT="yes"  
IPADDR=172.16.10.12  
NETMASK=255.255.0.0  
[root@jie1 ~]# sed -i 's/IPADDR/ip/w ip.txt' test  
       #把sed操作的文件内容保存到另外一个文件中，w表示保存，ip.txt文件名  
[root@jie1 ~]# cat ip.txt  #查看新文件的内容  
ip=172.16.10.12
```

**读取一个文件到正在用sed操作的文件中**

```
[root@jie1 ~]# cat myfile   #文件内容  
hello world  
i am li  
how are you  
li  
[root@jie1 ~]# cat test  #将用sed操作的文件的内容  
Ethernet  
#BOOTPROTO="dhcp"  
HWADDR="00:0C:29:90:79:78"  
ONBOOT="yes"  
IPADDR=172.16.10.12  
NETMASK=255.255.0.0  
[root@jie1 ~]# sed  -i '/Ethernet/r myfile' test  
      #在匹配Ethernet的行，读进来另一个文件的内容，读进来的文件的内容会插入到匹配Ethernet的行后  
[root@jie1 ~]# cat test  #再次查看用sed命令操作的行  
Ethernet  
hello world  
i am li  
how are you  
li  
#BOOTPROTO="dhcp"  
HWADDR="00:0C:29:90:79:78"  
ONBOOT="yes"  
IPADDR=172.16.10.12  
NETMASK=255.255.0.0  
[root@jie1 ~]#  
```

###### sed的经典例子

```
[root@jie1 ~]# cat myfile   #文件内容  
hello world  
i am li  
how are you  
li  
[root@jie1 ~]# cat test  #将用sed操作的文件的内容  
Ethernet  
#BOOTPROTO="dhcp"  
HWADDR="00:0C:29:90:79:78"  
ONBOOT="yes"  
IPADDR=172.16.10.12  
NETMASK=255.255.0.0  
[root@jie1 ~]# sed  -i '/Ethernet/r myfile' test  
      #在匹配Ethernet的行，读进来另一个文件的内容，读进来的文件的内容会插入到匹配Ethernet的行后  
[root@jie1 ~]# cat test  #再次查看用sed命令操作的行  
Ethernet  
hello world  
i am li  
how are you  
li  
#BOOTPROTO="dhcp"  
HWADDR="00:0C:29:90:79:78"  
ONBOOT="yes"  
IPADDR=172.16.10.12  
NETMASK=255.255.0.0  
[root@jie1 ~]#  
```



##### AWK

###### 基础语法

```
    awk [options] file ...
    
    #逐行读取并打印出来，默认以空格分割
    awk '{print $0}' /etc/passwd  
    
    #读取每行的第一个域的内容，并打印出来
    awk -F: '{print $1}' /etc/passwd 
    
    #匹配的列,只要有匹配就打印，不分域
    awk '/root/ {print $1}' /etc/passwd
    
    #统计行数
    awk '{count++}END{print  count}' /etc/passwd
    #count是自定义变量,这里没有初始化count,虽然默认是0,但是妥当的做法还是初始化为0.
    awk 'BEGIN{count=0}{count=count+1}END{print count}' /etc/passwd
    
    #字符长度大于N的行
    awk -F: 'length($1)>5' /etc/passwd
    
    #统计某个文件夹下的文件占用的字节数
    ls -l |awk 'BEGIN {size=0;} {size=size+$5;} END{print "[end]size is ", size}'
    #按照M为单位显示
    ls -l |awk 'BEGIN {size=0;} {size=size+$5;} END{print "[end]size is ", size/1024/1024,"M"}' 
```



###### 内置函数

```
ARGC               命令行参数个数
ARGV               命令行参数排列
ENVIRON            支持队列中系统环境变量的使用
FILENAME           awk浏览的文件名
FNR                浏览文件的记录数
FS                 设置输入域分隔符，等价于命令行 -F选项
NF                 浏览记录的域的个数
NR                 已读的记录数
OFS                输出域分隔符
ORS                输出记录分隔符
RS                 控制记录分隔符
```

```
#统计/etc/passwd:文件名，每行的行号，每行的列数，对应的完整行内容:
awk  -F: '{print "filename:" FILENAME ",linenumber:" NR ",columns:" NF ",linecontent:"$0}' /etc/passwd

#命令行参数个数
awk '{print ARGC}' file file2...
```

**awk可以做很多事情，赋值，运算，变量等等...具体就不一一说明了，自行搜索.**



##### sort

###### 基本语法

```
sort [option] [files...]
```

###### 选项

```
-b：忽略每行前面开始处的空格字符；
-c：检查文件是否已经按照顺序排序，排序过为真；
-d：排序时，处理英文字母、数字和空格字符，以字典顺序排序。忽略其他所有字符；
-f：排序时，将小写字母视为大写字母；
-i：排序时，处理040~176之间的ASCII字符，忽略其他所有字符；
-m：将几个排序好的文件进行合并；
-M：将前面3个字母按月份的缩写进行排序；
-n：按照数值大小进行排序；
-o outfile.txt：将排序后的结果存入outfile.txt；
-r：以相反的顺序进行排序；
-k：指定需要排序的列数（栏数）；就是指定按第几列进行排序
-t 分隔符：指定排序时所用到的栏位分隔符；
-u 去重 不过一般使用 uniq操作
```

例如,如下文本

>   one_two
>   one_two_three
>   one_two_four
>   one_two_five

```
### 以下划线为分割 第一第二列进行排序,那么默认第三列会按顺序进行排序，所以结果会被打乱
sort -t_ -k1,1 -k2,2 filename
```
> [root/shell] ]$sort -t _ -k1,1 -k2,2 c
> one_two
> one_two_five
> one_two_four
> one_two_three

```
#所以，sort提供了--stable参数来进行补救。
sort -t_ -k1,1 -k2,2 filename
```

##### uniq

###### 选项

```
 -c, --count              //在每行前加上表示相应行目出现次数的前缀编号
 -d, --repeated          //只输出重复的行
 -D, --all-repeated      //只输出重复的行，不过有几行输出几行
 -f, --skip-fields=N     //-f 忽略的段数，-f 1 忽略第一段
 -i, --ignore-case       //不区分大小写
 -s, --skip-chars=N      //根-f有点像，不过-s是忽略，后面多少个字符 -s 5就忽略后面5个字符
 -u, --unique            //去除重复的后，全部显示出来，根MySQL的distinct功能上有点像
 -z, --zero-terminated   end lines with 0 byte, not newline
 -w, --check-chars=N      //对每行第N 个字符以后的内容不作对照
 --help              //显示此帮助信息并退出
 --version              //显示版本信息并退出
```

###### 注意

>   二点!
>   1，对文本操作时，它一般会和sort命令进行组合使用，因为uniq 不会检查重复的行，除非它们是相邻的行。如果您想先对输入排序，使用sort -u。
>   2，对文本操作时，若域中为先空字符(通常包括空格以及制表符)，然后非空字符，域中字符前的空字符将被跳过

##### fmt文本块操作

###### 语法

```
fmt [option] [file-list]
```

###### 选项

```
-s              截断长行，但不合并
-t               除每个段落的第1行外都缩进
-u              改变格式化，使字之间出现一个空格，句子之间出现两个空格
-w n           将输出的行宽改为n个字符。不带该选项时，fmt输出的行宽度为75个字符
```

```
例如,我有一个文件demo,内容为:
A long time ago, there was a huge apple tree.         A little boy loved to come and play around it every day. He climbed to the tree top, ate the apples, took a nap under the shadow… He loved the tree and the tree loved to play with him. 
 
使用命令 fmt -s demo,输出为:
 
A long time ago, there was a huge apple tree.         A little boy loved
to come and play around it every day. He climbed to the tree top, ate
the apples, took a nap under the shadow… He loved the tree and the
tree loved to play with him.
该命令的含义是节段2长行.
 
使用fmt -t demo命令的意思是说排除首行的缩进,结果为:
A long time ago, there was a huge apple tree.         A little boy loved
   to come and play around it every day. He climbed to the tree top,
   ate the apples, took a nap under the shadow… He loved the tree and
   the tree loved to play with him.
 
 
使用fmt -u demo命令的意思是说格式化单词和句子的间隔.输出为:
A long time ago, there was a huge apple tree.  A little boy loved to come
and play around it every day. He climbed to the tree top, ate the apples,
took a nap under the shadow… He loved the tree and the tree loved to
play with him.
显然A little boy前面的多个空格变成了两个.
 
使用命令fmt -w 40 demo意思是说指定行的宽度,这里的行宽为40个字符.所以输出为:
A long time ago, there was a huge
apple tree.         A little boy
loved to come and play around it
every day. He climbed to the tree top,
ate the apples, took a nap under the
shadow… He loved the tree and the
tree loved to play with him.
 
仅作切割的选项： -s , 在你想将长的行绕回，短的行保持不动时很好用，这么做也能使结果与原始版本间的差异达到最小,例如:
fmt -s -w 10 << EOF
one two three four five
six
seven
eight
输出为:
one two
three
four five
six
seven
eight
 
fmt的小案例:
下面以拼音字典为例：
字典文件：/usr/dict/words或者/usr/share/dict/words。
sed -n -e 9991,10010p /usr/share/dict/words | fmt
sed -n -e 9991,10010p /usr/share/dict/words | fmt -w 30
```

##### wc

###### 选项

```
-c:统计字节数
-l:统计行数
-m:统计字符数.这个标志不能与-c标志一起使用
-w:统计字数.一个字被定义为由空白,挑个或换行字符分隔的字符串.
-L:打印最常行的长度
-help:显示帮助信息
--version:显示版本信息
```

#### seq

```
-s 指定分隔符，默认是换行
-w 等位补全，就是宽度相等，不足的前面补 0
-f 格式化输出，就是指定打印的格式
```

```
seq 1 10        #输出1到10
seq -s "--" 1 3     #输出1--2--3
seq -w 1 3      #等宽输出
seq -f %03g 1 20    #格式化为五位，不足用0补齐
# %后面指定数字的位数 默认是%g，%3g那么数字位数不足部分是空格。
```
