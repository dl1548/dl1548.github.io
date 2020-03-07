---
title: shell手册
date: 2020-02-23 12:06:52
tags: shell
categories: linux
password:
---



​    转载

​    手册制作: 雪松

​    github更新下载地址:  https://github.com/liquanzhou/ops_doc



<!--more-->



#### 文件

##### 基础命令

```shell
    ls -rtl                 # 按时间倒叙列出所有目录和文件 ll -rt
    touch file              # 创建空白文件
    rm -rf 目录名            # 不提示删除非空目录(-r:递归删除 -f强制)
    dos2unix                # windows文本转linux文本
    unix2dos                # linux文本转windows文本
    enca filename           # 查看编码  安装 yum install -y enca
    md5sum                  # 查看md5值
    ln 源文件 目标文件         # 硬链接
    ln -s 源文件 目标文件      # 符号连接
    readlink -f /data       # 查看连接真实目录
    cat file | nl |less     # 查看上下翻页且显示行号  q退出
    head                    # 查看文件开头内容
    head -c 10m             # 截取文件中10M内容
    split -C 10M            # 将文件切割大小为10M -C按行
    tail -f file            # 查看结尾 监视日志文件
    tail -F file            # 监视日志并重试, 针对文件被mv的情况可以持续读取
    file                    # 检查文件类型
    umask                   # 更改默认权限
    uniq                    # 删除重复的行
    uniq -c                 # 重复的行出现次数
    uniq -u                 # 只显示不重复行
    paste a b               # 将两个文件合并用tab键分隔开
    paste -d'+' a b         # 将两个文件合并指定'+'符号隔开
    paste -s a              # 将多行数据合并到一行用tab键隔开
    chattr +i /etc/passwd   # 不得任意改变文件或目录 -i去掉锁 -R递归
    more                    # 向下分面器
    locate 字符串            # 搜索
    wc -l file              # 查看行数
    cp filename{,.bak}      # 快速备份一个文件
    \cp a b                 # 拷贝不提示 既不使用别名 cp -i
    rev                     # 将行中的字符逆序排列
    comm -12 2 3            # 行和行比较匹配
    echo "10.45aa" |cksum                   # 字符串转数字编码，可做校验，也可用于文件校验
    iconv -f gbk -t utf8 原.txt > 新.txt     # 转换编码
    xxd /boot/grub/stage1                   # 16进制查看
    hexdump -C /boot/grub/stage1            # 16进制查看
    rename 原模式 目标模式 文件                 # 重命名 可正则
    watch -d -n 1 'df; ls -FlAt /path'      # 实时某个目录下查看最新改动过的文件
    cp -v  /dev/dvd  /rhel4.6.iso9660       # 制作镜像
    diff suzu.c suzu2.c  > sz.patch         # 制作补丁
    patch suzu.c < sz.patch                 # 安装补丁
```


#####  sort排序

```shell
    -t  # 指定排序时所用的栏位分隔字符
    -n  # 依照数值的大小排序
    -r  # 以相反的顺序来排序
    -f  # 排序时，将小写字母视为大写字母
    -d  # 排序时，处理英文字母、数字及空格字符外，忽略其他的字符
    -c  # 检查文件是否已经按照顺序排序
    -b  # 忽略每行前面开始处的空格字符
    -M  # 前面3个字母依照月份的缩写进行排序
    -k  # 指定域
    -m  # 将几个排序好的文件进行合并
    -T  # 指定临时文件目录,默认在/tmp
    +<起始栏位>-<结束栏位>   # 以指定的栏位来排序，范围由起始栏位到结束栏位的前一栏位。
    -o  # 将排序后的结果存入指定的文

    sort -n               # 按数字排序
    sort -nr              # 按数字倒叙
    sort -u               # 过滤重复行
    sort -m a.txt c.txt   # 将两个文件内容整合到一起
    sort -n -t' ' -k 2 -k 3 a.txt     # 第二域相同，将从第三域进行升降处理
    sort -n -t':' -k 3r a.txt         # 以:为分割域的第三域进行倒叙排列
    sort -k 1.3 a.txt                 # 从第三个字母起进行排序
    sort -t" " -k 2n -u  a.txt        # 以第二域进行排序，如果遇到重复的，就删除
```

##### find查找

```shell
    # linux文件无创建时间
    # Access 使用时间
    # Modify 内容修改时间
    # Change 状态改变时间(权限、属主)
    # 时间默认以24小时为单位,当前时间到向前24小时为0天,向前48-72小时为2天
    # -and 且 匹配两个条件 参数可以确定时间范围 -mtime +2 -and -mtime -4
    # -or 或 匹配任意一个条件

    find /etc -name "*http*"     # 按文件名查找
    find . -type f               # 查找某一类型文件
    find / -perm                 # 按照文件权限查找
    find / -user                 # 按照文件属主查找
    find / -group                # 按照文件所属的组来查找文件
    find / -atime -n             # 文件使用时间在N天以内
    find / -atime +n             # 文件使用时间在N天以前
    find / -mtime +n             # 文件内容改变时间在N天以前
    find / -ctime +n             # 文件状态改变时间在N天前
    find / -mmin +30             # 按分钟查找内容改变
    find / -size +1000000c -print                           # 查找文件长度大于1M字节的文件
    find /etc -name "*passwd*" -exec grep "xuesong" {} \;   # 按名字查找文件传递给-exec后命令
    find . -name 't*' -exec basename {} \;                  # 查找文件名,不取路径
    find . -type f -name "err*" -exec  rename err ERR {} \; # 批量改名(查找err 替换为 ERR {}文件
    find 路径 -name *name1* -or -name *name2*               # 查找任意一个关键字
```

##### vim编辑

```shell

    gconf-editor           # 配置编辑器
    /etc/vimrc             # 配置文件路径
    vim +24 file           # 打开文件定位到指定行
    vim file1 file2        # 打开多个文件
    vim -O2 file1 file2    # 垂直分屏
    vim -on file1 file2    # 水平分屏
    Ctrl+ U                # 向前翻页
    Ctrl+ D                # 向后翻页
    Ctrl+ww                # 在窗口间切换
    Ctrl+w +or-or=         # 增减高度
    :sp filename           # 上下分割打开新文件
    :vs filename           # 左右分割打开新文件
    :set nu                # 打开行号
    :set nonu              # 取消行号
    :nohl                  # 取消高亮
    :set paste             # 取消缩进
    :set autoindent        # 设置自动缩进
    :set ff                # 查看文本格式
    :set binary            # 改为unix格式
    :%s/字符1/字符2/g       # 全部替换
    :200                   # 跳转到200  1 文件头
    G                      # 跳到行尾
    dd                     # 删除当前行 并复制 可直接p粘贴
    11111dd                # 删除11111行，可用来清空文件
    r                      # 替换单个字符
    R                      # 替换多个字符
    u                      # 撤销上次操作
    *                      # 全文匹配当前光标所在字符串
    $                      # 行尾
    0                      # 行首
    X                      # 文档加密
    v =                    # 自动格式化代码
    Ctrl+v                 # 可视模式
    Ctrl+v I ESC           # 多行操作
    Ctrl+v s ESC           # 批量取消注释

```

##### 归档压缩

```shell
    tar zxvpf gz.tar.gz -C 放到指定目录 包中的目录       # 解包tar.gz 不指定目录则全解压
    tar zcvpf /$path/gz.tar.gz * # 打包gz 注意*最好用相对路径
    tar zcf /$path/gz.tar.gz *   # 打包正确不提示
    tar ztvpf gz.tar.gz          # 查看gz
    tar xvf 1.tar -C 目录         # 解包tar
    tar -cvf 1.tar *             # 打包tar
    tar tvf 1.tar                # 查看tar
    tar -rvf 1.tar 文件名         # 给tar追加文件
    tar --exclude=/home/dmtsai --exclude=*.tar -zcvf myfile.tar.gz /home/* /etc      # 打包/home, /etc ，但排除 /home/dmtsai
    tar -N "2005/06/01" -zcvf home.tar.gz /home      # 在 /home 当中，比 2005/06/01 新的文件才备份
    tar -zcvfh home.tar.gz /home                     # 打包目录中包括连接目录
    tar zcf - ./ | ssh root@IP "tar zxf - -C /xxxx"  # 一边压缩一边解压
    zgrep 字符 1.gz               # 查看压缩包中文件字符行
    bzip2  -dv 1.tar.bz2         # 解压bzip2
    bzip2 -v 1.tar               # bzip2压缩
    bzcat                        # 查看bzip2
    gzip A                       # 直接压缩文件 # 压缩后源文件消失
    gunzip A.gz                  # 直接解压文件 # 解压后源文件消失
    gzip -dv 1.tar.gz            # 解压gzip到tar
    gzip -v 1.tar                # 压缩tar到gz
    unzip zip.zip                # 解压zip
    zip zip.zip *                # 压缩zip
    # rar3.6下载:  http://www.rarsoft.com/rar/rarlinux-3.6.0.tar.gz
    rar a rar.rar *.jpg          # 压缩文件为rar包
    unrar x rar.rar              # 解压rar包
    7z a 7z.7z *                 # 7z压缩
    7z e 7z.7z                   # 7z解压	
```

##### 文件ACL控制

```shell
getfacl 1.test                      # 查看文件ACL权限
setfacl -R -m u:xuesong:rw- 1.test  # 对文件增加用户的读写权限 -R 递归

```

##### git操作

```shell
# 编译安装git-1.8.4.4
./configure --with-curl --with-expat
make
make install

git clone git@10.10.10.10:gittest.git  ./gittest/  # 克隆项目到指定目录
git status                                         # Show the working tree(工作树) status
git log -n 1 --stat                                # 查看最后一次日志文件
git branch -a                                      # 列出远程跟踪分支(remote-tracking branches)和本地分支
git checkout developing                            # 切换到developing分支
git checkout -b release                            # 切换分支没有从当前分支创建
git checkout -b release origin/master              # 从远程分支创建本地镜像分支
git push origin --delete release                   # 从远端删除分区，服务端有可能设置保护不允许删除
git push origin release                            # 把本地分支提交到远程
git pull                                           # 更新项目 需要cd到项目目录中
git fetch                                          # 抓取远端代码但不合并到当前
git reset --hard origin/master                     # 和远端同步分支
git add .                                          # 更新所有文件
git commit -m "gittest up"                         # 提交操作并添加备注
git push                                           # 正式提交到远程git服务器
git push [-u origin master]                        # 正式提交到远程git服务器(master分支)
git tag [-a] dev-v-0.11.54 [-m 'fix #67']          # 创建tag,名为dev-v-0.11.54,备注fix #67
git tag -l dev-v-0.11.54                           # 查看tag(dev-v-0.11.5)
git push origin --tags                             # 提交tag
git reset --hard                                   # 本地恢复整个项目
git rm -r -n --cached  ./img                       # -n执行命令时,不会删除任何文件,而是展示此命令要删除的文件列表预览
git rm -r --cached  ./img                          # 执行删除命令 需要commit和push让远程生效
git init --bare smc-content-check.git              # 初始化新git项目  需要手动创建此目录并给git用户权限 chown -R git:git smc-content-check.git
git config --global credential.helper store        # 记住密码
git config [--global] user.name "your name"        # 设置你的用户名, 希望在一个特定的项目中使用不同的用户或e-mail地址, 不要--global选项
git config [--global] user.email "your email"      # 设置你的e-mail地址, 每次Git提交都会使用该信息
git config [--global] user.name                    # 查看用户名
git config [--global] user.email                   # 查看用户e-mail
git config --global --edit                         # 编辑~/.gitconfig(User-specific)配置文件, 值优先级高于/etc/gitconfig(System-wide)
git config --edit                                  # 编辑.git/config(Repository specific)配置文件, 值优先级高于~/.gitconfig
git cherry-pick  <commit id>                       # 用于把另一个本地分支的commit修改应用到当前分支 需要push到远程
git log --pretty=format:'%h: %s' 9378b62..HEAD     # 查看指定范围更新操作 commit id

从远端拉一份新的{
# You have not concluded your merge (MERGE_HEAD exists)  git拉取失败
git fetch --hard origin/master
git reset --hard origin/master
}
```

##### rm删除文件恢复

```shell
# debugfs针对 ext2   # ext3grep针对 ext3   # extundelete针对 ext4
df -T   # 首先查看磁盘分区格式
umount /data/     # 卸载挂载,数据丢失请首先卸载挂载,或重新挂载只读
ext3grep /dev/sdb1 --ls --inode 2         # 记录信息继续查找目录下文件inode信息
ext3grep /dev/sdb1 --ls --inode 131081    # 此处是inode
ext3grep /dev/sdb1 --restore-inode 49153  # 记录下inode信息开始恢复目录
```





#### 软件

##### rpm

```shell
rpm -ivh lynx          # rpm安装
rpm -e lynx            # 卸载包
rpm -e lynx --nodeps   # 强制卸载
rpm -qa                # 查看所有安装的rpm包
rpm -qa | grep lynx    # 查找包是否安装
rpm -ql                # 软件包路径
rpm -Uvh               # 升级包
rpm --test lynx        # 测试
rpm -qc                # 软件包配置文档
rpm --import  /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6     # 导入rpm的签名信息
rpm --initdb           # 初始化rpm 数据库
rpm --rebuilddb        # 重建rpm数据库  在rpm和yum无响应的情况使用 先 rm -f /var/lib/rpm/__db.00* 在重建
```

##### yum

```shell
yum list                 # 所有软件列表
yum install 包名          # 安装包和依赖包
yum -y update            # 升级所有包版本,依赖关系，系统版本内核都升级
yum -y update 软件包名    # 升级指定的软件包
yum -y upgrade           # 不改变软件设置更新软件，系统版本升级，内核不改变
yum search mail          # yum搜索相关包
yum grouplist            # 软件包组
yum -y groupinstall "Virtualization"   # 安装软件包组
repoquery -ql gstreamer  # 不安装软件查看包含文件
yum clean all            # 清除var下缓存
```

##### 编译

```shell
源码安装{

    ./configure --help                   # 查看所有编译参数
    ./configure  --prefix=/usr/local/    # 配置参数
    make                                 # 编译
    # make -j 8                          # 多线程编译,速度较快,但有些软件不支持
    make install                         # 安装包
    make clean                           # 清除编译结果

}

perl程序编译{

    perl Makefile.PL
    make
    make test
    make install

}

python程序编译{

    python file.py

    # 源码包编译安装
    python setup.py build
    python setup.py install

}

编译c程序{

    gcc -g hello.c -o hello

}
```



#### 系统

##### 基础命令

```shell
wall        　  　                            # 给其它用户发消息
whereis ls                                    # 查找命令的目录
which                                         # 查看当前要执行的命令所在的路径
clear                                         # 清空整个屏幕
reset                                         # 重新初始化屏幕
cal                                           # 显示月历
echo -n 123456 | md5sum                       # md5加密
mkpasswd                                      # 随机生成密码   -l位数系统org: public ntp time server for everyone(http://www.pool.ntp.org/zh/)
tzselect                                      # 选择时区 #+8=(5 9 1 1) # (TZ='Asia/Shanghai'; export TZ)括号内写入 /etc/profile
/sbin/hwclock -w                              # 时间保存到硬件
/etc/shadow                                   # 账户影子文件
LANG=en                                       # 修改语言
vim /etc/sysconfig/i18n                       # 修改编码  LANG="en_US.UTF-8"
export LC_ALL=C                               # 强制字符集
vi /etc/hosts                                 # 查询静态主机名
alias                                         # 别名
watch uptime                                  # 监测命令动态刷新 监视
ipcs -a                                       # 查看Linux系统当前单个共享内存段的最大值
ldconfig                                      # 动态链接库管理命令
ldd `which cmd`                               # 查看命令的依赖库
dist-upgrade                                  # 会改变配置文件,改变旧的依赖关系，改变系统版本
/boot/grub/grub.conf                          # grub启动项配置
ps -mfL <PID>                                 # 查看指定进程启动的线程 线程数受 max user processes 限制
ps uxm |wc -l                                 # 查看当前用户占用的进程数 [包括线程]  max user processes
top -p  PID -H                                # 查看指定PID进程及线程
lsof |wc -l                                   # 查看当前文件句柄数使用数量  open files
lsof |grep /lib                               # 查看加载库文件
sysctl -a                                     # 查看当前所有系统内核参数
sysctl -p                                     # 修改内核参数/etc/sysctl.conf，让/etc/rc.d/rc.sysinit读取生效
strace -p pid                                 # 跟踪系统调用
ps -eo "%p %C  %z  %a"|sort -k3 -n            # 把进程按内存使用大小排序
strace uptime 2>&1|grep open                  # 查看命令打开的相关文件
grep Hugepagesize /proc/meminfo               # 内存分页大小
mkpasswd -l 8  -C 2 -c 2 -d 4 -s 0            # 随机生成指定类型密码
echo 1 > /proc/sys/net/ipv4/tcp_syncookies    # 使TCP SYN Cookie 保护生效  # "SYN Attack"是一种拒绝服务的攻击方式
grep Swap  /proc/25151/smaps |awk '{a+=$2}END{print a}'    # 查询某pid使用的swap大小
redir --lport=33060 --caddr=10.10.10.78 --cport=3306       # 端口映射 yum安装 用supervisor守护

```



##### 开机启动脚本顺序

```shell
/etc/profile
/etc/profile.d/*.sh
~/bash_profile
~/.bashrc
/etc/bashrc

```

##### 进程管理

###### 常用

```shell
ps -eaf               # 查看所有进程
kill -9 PID           # 强制终止某个PID进程
kill -15 PID          # 安全退出 需程序内部处理信号
cmd &                 # 命令后台运行
nohup cmd &           # 后台运行不受shell退出影响
ctrl+z                # 将前台放入后台(暂停)
jobs                  # 查看后台运行程序
bg 2                  # 启动后台暂停进程
fg 2                  # 调回后台进程
pstree                # 进程树
vmstat 1 9            # 每隔一秒报告系统性能信息9次
sar                   # 查看cpu等状态
lsof file             # 显示打开指定文件的所有进程
lsof -i:32768         # 查看端口的进程
renice +1 180         # 把180号进程的优先级加1
exec sh a.sh          # 子进程替换原来程序的pid， 避免supervisor无法强制杀死进程

```

###### top

```shell
前五行是系统整体的统计信息。
第一行: 任务队列信息，同 uptime 命令的执行结果。内容如下：
    01:06:48 当前时间
    up 1:22 系统运行时间，格式为时:分
    1 user 当前登录用户数
    load average: 0.06, 0.60, 0.48 系统负载，即任务队列的平均长度。
    三个数值分别为 1分钟、5分钟、15分钟前到现在的平均值。

第二、三行:为进程和CPU的信息。当有多个CPU时，这些内容可能会超过两行。内容如下：
    Tasks: 29 total 进程总数
    1 running 正在运行的进程数
    28 sleeping 睡眠的进程数
    0 stopped 停止的进程数
    0 zombie 僵尸进程数
    Cpu(s): 0.3% us 用户空间占用CPU百分比
    1.0% sy 内核空间占用CPU百分比
    0.0% ni 用户进程空间内改变过优先级的进程占用CPU百分比
    98.7% id 空闲CPU百分比
    0.0% wa 等待输入输出的CPU时间百分比
    0.0% hi
    0.0% si

第四、五行:为内存信息。内容如下：
    Mem: 191272k total 物理内存总量
    173656k used 使用的物理内存总量
    17616k free 空闲内存总量
    22052k buffers 用作内核缓存的内存量
    Swap: 192772k total 交换区总量
    0k used 使用的交换区总量
    192772k free 空闲交换区总量
    123988k cached 缓冲的交换区总量。
    内存中的内容被换出到交换区，而后又被换入到内存，但使用过的交换区尚未被覆盖，
    该数值即为这些内容已存在于内存中的交换区的大小。
    相应的内存再次被换出时可不必再对交换区写入。

进程信息区,各列的含义如下:  # 显示各个进程的详细信息

序号 列名    含义
a   PID      进程id
b   PPID     父进程id
c   RUSER    Real user name
d   UID      进程所有者的用户id
e   USER     进程所有者的用户名
f   GROUP    进程所有者的组名
g   TTY      启动进程的终端名。不是从终端启动的进程则显示为 ?
h   PR       优先级
i   NI       nice值。负值表示高优先级，正值表示低优先级
j   P        最后使用的CPU，仅在多CPU环境下有意义
k   %CPU     上次更新到现在的CPU时间占用百分比
l   TIME     进程使用的CPU时间总计，单位秒
m   TIME+    进程使用的CPU时间总计，单位1/100秒
n   %MEM     进程使用的物理内存百分比
o   VIRT     进程使用的虚拟内存总量，单位kb。VIRT=SWAP+RES
p   SWAP     进程使用的虚拟内存中，被换出的大小，单位kb。
q   RES      进程使用的、未被换出的物理内存大小，单位kb。RES=CODE+DATA
r   CODE     可执行代码占用的物理内存大小，单位kb
s   DATA     可执行代码以外的部分(数据段+栈)占用的物理内存大小，单位kb
t   SHR      共享内存大小，单位kb
u   nFLT     页面错误次数
v   nDRT     最后一次写入到现在，被修改过的页面数。
w   S        进程状态。
    D=不可中断的睡眠状态
    R=运行
    S=睡眠
    T=跟踪/停止
    Z=僵尸进程 父进程在但并不等待子进程
x   COMMAND  命令名/命令行
y   WCHAN    若该进程在睡眠，则显示睡眠中的系统函数名
z   Flags    任务标志，参考 sched.h

```

###### ps

```shell
ps aux |grep -v USER | sort -nk +4 | tail       # 显示消耗内存最多的10个运行中的进程，以内存使用量排序.cpu +3
# USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
%CPU     # 进程的cpu占用率
%MEM     # 进程的内存占用率
VSZ      # 进程虚拟大小,单位K(即总占用内存大小,包括真实内存和虚拟内存)
RSS      # 进程使用的驻留集大小即实际物理内存大小
START    # 进程启动时间和日期
占用的虚拟内存大小 = VSZ - RSS

ps -eo pid,lstart,etime,args         # 查看进程启动时间

```

###### 列出占用sawp的进程

```shell
#!/bin/bash
echo -e "PID\t\tSwap\t\tProc_Name"
# 拿出/proc目录下所有以数字为名的目录（进程名是数字才是进程，其他如sys,net等存放的是其他信息）
for pid in `ls -l /proc | grep ^d | awk '{ print $9 }'| grep -v [^0-9]`
do
    # 让进程释放swap的方法只有一个：就是重启该进程。或者等其自动释放。放
    # 如果进程会自动释放，那么我们就不会写脚本来找他了，找他都是因为他没有自动释放。
    # 所以我们要列出占用swap并需要重启的进程，但是init这个进程是系统里所有进程的祖先进程
    # 重启init进程意味着重启系统，这是万万不可以的，所以就不必检测他了，以免对系统造成影响。
    if [ $pid -eq 1 ];then continue;fi
    grep -q "Swap" /proc/$pid/smaps 2>/dev/null
    if [ $? -eq 0 ];then
        swap=$(grep Swap /proc/$pid/smaps \
            | gawk '{ sum+=$2;} END{ print sum }')
        proc_name=$(ps aux | grep -w "$pid" | grep -v grep \
            | awk '{ for(i=11;i<=NF;i++){ printf("%s ",$i); }}')
        if [ $swap -gt 0 ];then
            echo -e "${pid}\t${swap}\t${proc_name}"
        fi
    fi
done | sort -k2 -n | awk -F'\t' '{
    pid[NR]=$1;
    size[NR]=$2;
    name[NR]=$3;
}
END{
    for(id=1;id<=length(pid);id++)
    {
        if(size[id]<1024)
            printf("%-10s\t%15sKB\t%s\n",pid[id],size[id],name[id]);
        else if(size[id]<1048576)
            printf("%-10s\t%15.2fMB\t%s\n",pid[id],size[id]/1024,name[id]);
        else
            printf("%-10s\t%15.2fGB\t%s\n",pid[id],size[id]/1048576,name[id]);
    }
}'

```

###### 系统信号

```shell
kill -l                    # 查看linux提供的信号
trap "echo aaa"  2 3 15    # shell使用 trap 捕捉退出信号

# 发送信号一般有两种原因:
#   1(被动式)  内核检测到一个系统事件.例如子进程退出会像父进程发送SIGCHLD信号.键盘按下control+c会发送SIGINT信号
#   2(主动式)  通过系统调用kill来向指定进程发送信号
# 进程结束信号 SIGTERM 和 SIGKILL 的区别:  SIGTERM 比较友好，进程能捕捉这个信号，根据您的需要来关闭程序。在关闭程序之前，您可以结束打开的记录文件和完成正在做的任务。在某些情况下，假如进程正在进行作业而且不能中断，那么进程可以忽略这个SIGTERM信号。
# 如果一个进程收到一个SIGUSR1信号，然后执行信号绑定函数，第二个SIGUSR2信号又来了，第一个信号没有被处理完毕的话，第二个信号就会丢弃。

SIGHUP  1          A     # 终端挂起或者控制进程终止
SIGINT  2          A     # 键盘终端进程(如control+c)
SIGQUIT 3          C     # 键盘的退出键被按下
SIGILL  4          C     # 非法指令
SIGABRT 6          C     # 由abort(3)发出的退出指令
SIGFPE  8          C     # 浮点异常
SIGKILL 9          AEF   # Kill信号  立刻停止
SIGSEGV 11         C     # 无效的内存引用
SIGPIPE 13         A     # 管道破裂: 写一个没有读端口的管道
SIGALRM 14         A     # 闹钟信号 由alarm(2)发出的信号
SIGTERM 15         A     # 终止信号,可让程序安全退出 kill -15
SIGUSR1 30,10,16   A     # 用户自定义信号1
SIGUSR2 31,12,17   A     # 用户自定义信号2
SIGCHLD 20,17,18   B     # 子进程结束自动向父进程发送SIGCHLD信号
SIGCONT 19,18,25         # 进程继续（曾被停止的进程）
SIGSTOP 17,19,23   DEF   # 终止进程
SIGTSTP 18,20,24   D     # 控制终端（tty）上按下停止键
SIGTTIN 21,21,26   D     # 后台进程企图从控制终端读
SIGTTOU 22,22,27   D     # 后台进程企图从控制终端写

缺省处理动作一项中的字母含义如下:
    A  缺省的动作是终止进程
    B  缺省的动作是忽略此信号，将该信号丢弃，不做处理
    C  缺省的动作是终止进程并进行内核映像转储(dump core),内核映像转储是指将进程数据在内存的映像和进程在内核结构中的部分内容以一定格式转储到文件系统，并且进程退出执行，这样做的好处是为程序员提供了方便，使得他们可以得到进程当时执行时的数据值，允许他们确定转储的原因，并且可以调试他们的程序。
    D  缺省的动作是停止进程，进入停止状况以后还能重新进行下去，一般是在调试的过程中（例如ptrace系统调用）
    E  信号不能被捕获
    F  信号不能被忽略
```

###### 系统性能状态

````shell
vmstat 1 9

r      # 等待执行的任务数。当这个值超过了cpu线程数，就会出现cpu瓶颈。
b      # 等待IO的进程数量,表示阻塞的进程。
swpd   # 虚拟内存已使用的大小，如大于0，表示机器物理内存不足，如不是程序内存泄露，那么该升级内存。
free   # 空闲的物理内存的大小
buff   # 已用的buff大小，对块设备的读写进行缓冲
cache  # cache直接用来记忆我们打开的文件,给文件做缓冲，(把空闲的物理内存的一部分拿来做文件和目录的缓存，是为了提高 程序执行的性能，当程序使用内存时，buffer/cached会很快地被使用。)
inact  # 非活跃内存大小，即被标明可回收的内存，区别于free和active -a选项时显示
active # 活跃的内存大小 -a选项时显示
si   # 每秒从磁盘读入虚拟内存的大小，如果这个值大于0，表示物理内存不够用或者内存泄露，要查找耗内存进程解决掉。
so   # 每秒虚拟内存写入磁盘的大小，如果这个值大于0，同上。
bi   # 块设备每秒接收的块数量，这里的块设备是指系统上所有的磁盘和其他块设备，默认块大小是1024byte
bo   # 块设备每秒发送的块数量，例如读取文件，bo就要大于0。bi和bo一般都要接近0，不然就是IO过于频繁，需要调整。
in   # 每秒CPU的中断次数，包括时间中断。in和cs这两个值越大，会看到由内核消耗的cpu时间会越多
cs   # 每秒上下文切换次数，例如我们调用系统函数，就要进行上下文切换，线程的切换，也要进程上下文切换，这个值要越小越好，太大了，要考虑调低线程或者进程的数目,例如在apache和nginx这种web服务器中，我们一般做性能测试时会进行几千并发甚至几万并发的测试，选择web服务器的进程可以由进程或者线程的峰值一直下调，压测，直到cs到一个比较小的值，这个进程和线程数就是比较合适的值了。系统调用也是，每次调用系统函数，我们的代码就会进入内核空间，导致上下文切换，这个是很耗资源，也要尽量避免频繁调用系统函数。上下文切换次数过多表示你的CPU大部分浪费在上下文切换，导致CPU干正经事的时间少了，CPU没有充分利用。
us   # 用户进程执行消耗cpu时间(user time)  us的值比较高时，说明用户进程消耗的cpu时间多，但是如果长期超过50%的使用，那么我们就该考虑优化程序算法或其他措施
sy   # 系统CPU时间，如果太高，表示系统调用时间长，例如是IO操作频繁。
id   # 空闲 CPU时间，一般来说，id + us + sy = 100,一般认为id是空闲CPU使用率，us是用户CPU使用率，sy是系统CPU使用率。
wt   # 等待IOCPU时间。Wa过高时，说明io等待比较严重，这可能是由于磁盘大量随机访问造成的，也有可能是磁盘的带宽出现瓶颈。

如果 r 经常大于4，且id经常少于40，表示cpu的负荷很重。
如果 pi po 长期不等于0，表示内存不足。
如果 b 队列经常大于3，表示io性能不好。

````

##### 日志管理

```shell
history                      # 历时命令默认1000条
HISTTIMEFORMAT="%Y-%m-%d %H:%M:%S "   # 让history命令显示具体时间
history  -c                  # 清除记录命令
cat $HOME/.bash_history      # 历史命令记录文件
lastb -a                     # 列出登录系统失败的用户相关信息  清空二进制日志记录文件 echo > /var/log/btmp
last                         # 查看登陆过的用户信息  清空二进制日志记录文件 echo > /var/log/wtmp   默认打开乱码
who /var/log/wtmp            # 查看登陆过的用户信息
lastlog                      # 用户最后登录的时间
tail -f /var/log/messages    # 系统日志
tail -f /var/log/secure      # ssh日志
```

##### selinux

```shell
sestatus -v                    # 查看selinux状态
getenforce                     # 查看selinux模式
setenforce 0                   # 设置selinux为宽容模式(可避免阻止一些操作)
semanage port -l               # 查看selinux端口限制规则
semanage port -a -t http_port_t -p tcp 8000  # 在selinux中注册端口类型
vi /etc/selinux/config         # selinux配置文件
SELINUX=enfoceing              # 关闭selinux 把其修改为  SELINUX=disabled

```

##### 系统信息

```shell
uname -a              # 查看Linux内核版本信息
cat /proc/version     # 查看内核版本
cat /etc/issue        # 查看系统版本
lsb_release -a        # 查看系统版本  需安装 centos-release
locale -a             # 列出所有语系
locale                # 当前环境变量中所有编码
hwclock               # 查看时间
who                   # 当前在线用户
w                     # 当前在线用户
whoami                # 查看当前用户名
logname               # 查看初始登陆用户名
uptime                # 查看服务器启动时间
sar -n DEV 1 10       # 查看网卡网速流量
dmesg                 # 显示开机信息
lsmod                 # 查看内核模块
```

##### 硬件信息

```shell
more /proc/cpuinfo                                       # 查看cpu信息
lscpu                                                    # 查看cpu信息
cat /proc/cpuinfo | grep name | cut -f2 -d: | uniq -c    # 查看cpu型号和逻辑核心数
getconf LONG_BIT                                         # cpu运行的位数
cat /proc/cpuinfo | grep 'physical id' |sort| uniq -c    # 物理cpu个数
cat /proc/cpuinfo | grep flags | grep ' lm ' | wc -l     # 结果大于0支持64位
cat /proc/cpuinfo|grep flags                             # 查看cpu是否支持虚拟化   pae支持半虚拟化  IntelVT 支持全虚拟化
more /proc/meminfo                                       # 查看内存信息
dmidecode                                                # 查看全面硬件信息
dmidecode | grep "Product Name"                          # 查看服务器型号
dmidecode | grep -P -A5 "Memory\s+Device" | grep Size | grep -v Range       # 查看内存插槽
cat /proc/mdstat                                         # 查看软raid信息
cat /proc/scsi/scsi                                      # 查看Dell硬raid信息(IBM、HP需要官方检测工具)
lspci                                                    # 查看硬件信息
lspci|grep RAID                                          # 查看是否支持raid
lspci -vvv |grep Ethernet                                # 查看网卡型号
lspci -vvv |grep Kernel|grep driver                      # 查看驱动模块
modinfo tg2                                              # 查看驱动版本(驱动模块)
ethtool -i em1                                           # 查看网卡驱动版本
ethtool em1                                              # 查看网卡带宽
```

##### limits.conf

```shell
ulimit -SHn 65535  # 临时设置文件描述符大小 进程最大打开文件柄数 还有socket最大连接数, 等同配置 nofile
ulimit -SHu 65535  # 临时设置用户最大进程数
ulimit -a          # 查看

/etc/security/limits.conf

# 文件描述符大小  open files
# lsof |wc -l   查看当前文件句柄数使用数量
* soft nofile 16384         # 设置太大，进程使用过多会把机器拖死
* hard nofile 32768

# 用户最大进程数  max user processes
# echo $((`ps uxm |wc -l`-`ps ux |wc -l`))  查看当前用户占用的进程数 [包括线程]
user soft nproc 16384
user hard nproc 32768

# 如果/etc/security/limits.d/有配置文件，将会覆盖/etc/security/limits.conf里的配置
# 即/etc/security/limits.d/的配置文件里就不要有同样的参量设置
/etc/security/limits.d/90-nproc.conf    # centos6.3的默认这个文件会覆盖 limits.conf
user soft nproc 16384
user hard nproc 32768

sysctl -p    # 修改配置文件后让系统生效
```

##### libc.so故障修复

```shell

# 由于升级glibc导致libc.so不稳定,突然报错,幸好还有未退出的终端
grep: error while loading shared libraries: /lib64/libc.so.6: ELF file OS ABI invalid

# 看看当前系统有多少版本 libc.so
ls /lib64/libc-[tab]

# 更改环境变量指向其他 libc.so 文件测试
export LD_PRELOAD=/lib64/libc-2.7.so    # 如果不改变LD_PRELOAD变量,ln不能用,需要使用 /sbin/sln 命令做链接

# 当前如果好使了，在执行下面强制替换软链接。如不好使，测试其他版本的libc.so文件
ln -f -s /lib64/libc-2.7.so /lib64/libc.so.6

```

##### sudo

```shell
echo myPassword | sudo -S ls /tmp  # 直接输入sudo的密码非交互,从标准输入读取密码而不是终端设备
visudo                             # sudo命令权限添加  /etc/sudoers
用户  别名(可用all)=NOPASSWD:命令1,命令2
user  ALL=NOPASSWD:/bin/su         # 免root密码切换root身份
wangming linuxfan=NOPASSWD:/sbin/apache start,/sbin/apache restart
UserName ALL=(ALL) ALL
UserName ALL=(ALL) NOPASSWD: ALL
peterli        ALL=(ALL)       NOPASSWD:/sbin/service
Defaults requiretty                # sudo不允许后台运行,注释此行既允许
Defaults !visiblepw                
```

##### iptables

```
内建三个表：nat mangle 和 filter
filter预设规则表，有INPUT、FORWARD 和 OUTPUT 三个规则链
vi /etc/sysconfig/iptables    # 配置文件
INPUT    # 进入
FORWARD  # 转发
OUTPUT   # 出去
ACCEPT   # 将封包放行
REJECT   # 拦阻该封包
DROP     # 丢弃封包不予处理
-A       # 在所选择的链(INPUT等)末添加一条或更多规则
-D       # 删除一条
-E       # 修改
-p       # tcp、udp、icmp    0相当于所有all    !取反
-P       # 设置缺省策略(与所有链都不匹配强制使用此策略)
-s       # IP/掩码    (IP/24)    主机名、网络名和清楚的IP地址 !取反
-j       # 目标跳转，立即决定包的命运的专用内建目标
-i       # 进入的（网络）接口 [名称] eth0
-o       # 输出接口[名称]
-m       # 模块
--sport  # 源端口
--dport  # 目标端口

iptables -F                        # 将防火墙中的规则条目清除掉  # 注意: iptables -P INPUT ACCEPT
iptables-restore < 规则文件        # 导入防火墙规则
/etc/init.d/iptables save          # 保存防火墙设置
/etc/init.d/iptables restart       # 重启防火墙服务
iptables -L -n                     # 查看规则
iptables -t nat -nL                # 查看转发

```

###### iptables实例

```shell
iptables -L INPUT                   # 列出某规则链中的所有规则
iptables -X allowed                 # 删除某个规则链 ,不加规则链，清除所有非内建的
iptables -Z INPUT                   # 将封包计数器归零
iptables -N allowed                 # 定义新的规则链
iptables -P INPUT DROP              # 定义过滤政策
iptables -A INPUT -s 192.168.1.1    # 比对封包的来源IP   # ! 192.168.0.0/24  ! 反向对比
iptables -A INPUT -d 192.168.1.1    # 比对封包的目的地IP
iptables -A INPUT -i eth0           # 比对封包是从哪片网卡进入
iptables -A FORWARD -o eth0         # 比对封包要从哪片网卡送出 eth+表示所有的网卡
iptables -A INPUT -p tcp            # -p ! tcp 排除tcp以外的udp、icmp。-p all所有类型
iptables -D INPUT 8                 # 从某个规则链中删除一条规则
iptables -D INPUT --dport 80 -j DROP         # 从某个规则链中删除一条规则
iptables -R INPUT 8 -s 192.168.0.1 -j DROP   # 取代现行规则
iptables -I INPUT 8 --dport 80 -j ACCEPT     # 插入一条规则
iptables -A INPUT -i eth0 -j DROP            # 其它情况不允许
iptables -A INPUT -p tcp -s IP -j DROP       # 禁止指定IP访问
iptables -A INPUT -p tcp -s IP --dport port -j DROP               # 禁止指定IP访问端口
iptables -A INPUT -s IP -p tcp --dport port -j ACCEPT             # 允许在IP访问指定端口
iptables -A INPUT -p tcp --dport 22 -j DROP                       # 禁止使用某端口
iptables -A INPUT -i eth0 -p icmp -m icmp --icmp-type 8 -j DROP   # 禁止icmp端口
iptables -A INPUT -i eth0 -p icmp -j DROP                         # 禁止icmp端口
iptables -t filter -A INPUT -i eth0 -p tcp --syn -j DROP                  # 阻止所有没有经过你系统授权的TCP连接
iptables -A INPUT -f -m limit --limit 100/s --limit-burst 100 -j ACCEPT   # IP包流量限制
iptables -A INPUT -i eth0 -s 192.168.62.1/32 -p icmp -m icmp --icmp-type 8 -j ACCEPT  # 除192.168.62.1外，禁止其它人ping我的主机
iptables -A INPUT -p tcp -m tcp --dport 80 -m state --state NEW -m recent --update --seconds 5 --hitcount 20 --rttl --name WEB --rsource -j DROP  # 可防御cc攻击(未测试)

```

###### 常用配置实例

```shell
# 允许某段IP访问任何端口
iptables -A INPUT -s 192.168.0.3/24 -p tcp -j ACCEPT
# 设定预设规则 (拒绝所有的数据包，再允许需要的,如只做WEB服务器.还是推荐三个链都是DROP)
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT
# 注意: 直接设置这三条会掉线
# 开启22端口
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
# 如果OUTPUT 设置成DROP的，要写上下面一条
iptables -A OUTPUT -p tcp --sport 22 -j ACCEPT
# 注:不写导致无法SSH.其他的端口一样,OUTPUT设置成DROP的话,也要添加一条链
# 如果开启了web服务器,OUTPUT设置成DROP的话,同样也要添加一条链
iptables -A OUTPUT -p tcp --sport 80 -j ACCEPT
# 做WEB服务器,开启80端口 ,其他同理
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
# 做邮件服务器,开启25,110端口
iptables -A INPUT -p tcp --dport 110 -j ACCEPT
iptables -A INPUT -p tcp --dport 25 -j ACCEPT
# 允许icmp包通过,允许ping
iptables -A OUTPUT -p icmp -j ACCEPT (OUTPUT设置成DROP的话)
iptables -A INPUT -p icmp -j ACCEPT  (INPUT设置成DROP的话)
# 允许loopback!(不然会导致DNS无法正常关闭等问题)
IPTABLES -A INPUT -i lo -p all -j ACCEPT (如果是INPUT DROP)
IPTABLES -A OUTPUT -o lo -p all -j ACCEPT(如果是OUTPUT DROP)
```

###### 网段转发

```
# 例如通过vpn上网
echo 1 > /proc/sys/net/ipv4/ip_forward       # 在内核里打开ip转发功能
iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -j MASQUERADE  # 添加网段转发
iptables -t nat -A POSTROUTING -s 10.0.0.0/255.0.0.0 -o eth0 -j SNAT --to 192.168.10.158  # 原IP网段经过哪个网卡IP出去
iptables -t nat -nL                # 查看转发

```

###### 端口映射

```shell
# 内网通过有外网IP的机器映射端口
# 内网主机添加路由
route add -net 10.10.20.0 netmask 255.255.255.0 gw 10.10.20.111     # 内网需要添加默认网关，并且网关开启转发
# 网关主机
echo 1 > /proc/sys/net/ipv4/ip_forward       # 在内核里打开ip转发功能
iptables -t nat -A PREROUTING -d 外网IP  -p tcp --dport 9999 -j DNAT --to 10.10.20.55:22    # 进入
iptables -t nat -A POSTROUTING -s 10.10.20.0/24 -j SNAT --to 外网IP                         # 转发回去
iptables -t nat -nL                # 查看转发
```

#### 网络

##### 常用

```shell
rz                                                                    # 通过ssh上传小文件
sz                                                                    # 通过ssh下载小文件
ifconfig eth0 down                                                    # 禁用网卡
ifconfig eth0 up                                                      # 启用网卡
ifup eth0:0                                                           # 启用网卡
mii-tool em1                                                          # 查看网线是否连接
traceroute www.baidu.com                                              # 测试跳数
vi /etc/resolv.conf                                                   # 设置DNS  nameserver IP 定义DNS服务器的IP地址
nslookup www.moon.com                                                 # 解析域名IP
dig -x www.baidu.com                                                  # 解析域名IP
dig +trace -t A domainname                                            # 跟踪dns
dig +short txt hacker.wp.dg.cx                                        # 通过 DNS 来读取 Wikipedia 的hacker词条
host -t txt hacker.wp.dg.cx                                           # 通过 DNS 来读取 Wikipedia 的hacker词条
lynx                                                                  # 文本上网
wget -P path -O name url                                              # 下载  包名:wgetrc   -q 安静 -c 续传
dhclient eth1                                                         # 自动获取IP
mtr -r www.baidu.com                                                  # 测试网络链路节点响应时间 # trace ping 结合
ipcalc -m "$ip" -p "$num"                                             # 根据IP和主机最大数计算掩码
curl -I www.baidu.com                                                 # 查看网页http头
curl -s www.baidu.com                                                 # 不显示进度
queryperf -d list -s DNS_IP -l 2                                      # BIND自带DNS压力测试  [list 文件格式:www.turku.fi A]
telnet ip port                                                        # 测试端口是否开放,有些服务可直接输入命令得到返回状态
echo "show " |nc $ip $port                                            # 适用于telnet一类登录得到命令返回
nc -l -p port                                                         # 监听指定端口
nc -nv -z 10.10.10.11 1080 |grep succeeded                            # 检查主机端口是否开放
curl -o /dev/null -s -m 10 --connect-timeout 10 -w %{http_code} $URL  # 检查页面状态
curl -X POST -d "user=xuesong&pwd=123" http://www.abc.cn/Result       # 提交POST请求
curl -s http://20140507.ip138.com/ic.asp                              # 通过IP138取本机出口外网IP
curl http://IP/ -H "X-Forwarded-For: ip" -H "Host: www.ttlsa.com"     # 连到指定IP的响应主机,HTTPserver只看 Host字段
ifconfig eth0:0 192.168.1.221 netmask 255.255.255.0                   # 增加逻辑IP地址
echo 1 > /proc/sys/net/ipv4/icmp_echo_ignore_all                      # 禁ping
net rpc shutdown -I IP_ADDRESS -U username%password                   # 远程关掉一台WINDOWS机器
wget --random-wait -r -p -e robots=off -U Mozilla www.example.com     # 递归方式下载整个网站
sshpass -p "$pwd" rsync -avzP /dir  user@$IP:/dir/                    # 指定密码避免交互同步目录
rsync -avzP --delete /dir user@$IP:/dir                               # 无差同步目录 可以快速清空大目录
rsync -avzP -e "ssh -p 22 -e -o StrictHostKeyChecking=no" /dir user@$IP:/dir         # 指定ssh参数同步
```

##### 网卡流量查看

```shell
watch more /proc/net/dev    # 实时监控流量文件系统 累计值
iptraf                      # 网卡流量查看工具
nethogs -d 5 eth0 eth1      # 按进程实时统计网络流量 epel源nethogs
iftop -i eth0 -n -P         # 实时流量监控

sar {
-n参数有6个不同的开关: DEV | EDEV | NFS | NFSD | SOCK | ALL
DEV显示网络接口信息
EDEV显示关于网络错误的统计数据
NFS统计活动的NFS客户端的信息
NFSD统计NFS服务器的信息
SOCK显示套 接字信息
ALL显示所有5个开关

sar -n DEV 1 10

rxpck/s   # 每秒钟接收的数据包
txpck/s   # 每秒钟发送的数据包
rxbyt/s   # 每秒钟接收的字节数
txbyt/s   # 每秒钟发送的字节数
rxcmp/s   # 每秒钟接收的压缩数据包
txcmp/s   # 每秒钟发送的压缩数据包
rxmcst/s  # 每秒钟接收的多播数据包

}

```

##### ss

```shell
# netstat是遍历/proc下面每个PID目录，ss直接读/proc/net下面的统计信息。所以ss执行的时候消耗资源以及消耗的时间都比netstat少很多
ss -s                          # 列出当前socket详细信息
ss -l                          # 显示本地打开的所有端口
ss -tnlp                       # 显示每个进程具体打开的socket
ss -ant                        # 显示所有TCP socket
ss -u -a                       # 显示所有UDP Socekt
ss dst 192.168.119.113         # 匹配远程地址
ss dst 192.168.119.113:http    # 匹配远程地址和端口号
ss dst 192.168.119.113:3844    # 匹配远程地址和端口号
ss src 192.168.119.103:16021   # 匹配本地地址和端口号
ss -o state established '( dport = :smtp or sport = :smtp )'        # 显示所有已建立的SMTP连接
ss -o state established '( dport = :http or sport = :http )'        # 显示所有已建立的HTTP连接
ss -x src /tmp/.X11-unix/*         # 找出所有连接X服务器的进程

```

##### ssh

```shell
ssh -p 22 user@192.168.1.209                            # 从linux ssh登录另一台linux
ssh -p 22 root@192.168.1.209 CMD                        # 利用ssh操作远程主机
scp -P 22 file root@ip:/dir                             # 把本地文件拷贝到远程主机
scp -l 100000  file root@ip:/dir                        # 传输文件到远程，限制速度100M
sshpass -p 'pwd' ssh -n root@$IP "echo hello"           # 指定密码远程操作
ssh -o StrictHostKeyChecking=no $IP                     # ssh连接不提示yes
ssh -t "su -"                                           # 指定伪终端 客户端以交互模式工作
scp root@192.168.1.209:/RemoteDir /localDir             # 把远程指定文件拷贝到本地
pscp -h host.ip /a.sh /opt/sbin/                        # 批量传输文件
ssh -N -L2001:remotehost:80 user@somemachine            # 用SSH创建端口转发通道
ssh -t host_A ssh host_B                                # 嵌套使用SSH
ssh -t -p 22 $user@$Ip /bin/su - root -c {$Cmd};        # 远程su执行命令 Cmd="\"/sbin/ifconfig eth0\""
ssh-keygen -t rsa                                       # 生成密钥
ssh-copy-id -i xuesong@10.10.10.133                     # 传送key
vi $HOME/.ssh/authorized_keys                           # 公钥存放位置
sshfs name@server:/path/to/folder /path/to/mount/point  # 通过ssh挂载远程主机上的文件夹
fusermount -u /path/to/mount/point                      # 卸载ssh挂载的目录
ssh user@host cat /path/to/remotefile | diff /path/to/localfile -                # 用DIFF对比远程文件跟本地文件
su - user -c "ssh user@192.168.1.1 \"echo -e aa |mail -s test mail@163.com\""    # 切换用户登录远程发送邮件
pssh -h ip.txt -i uptime                                # 批量执行ssh yum install pssh

SSH反向连接{

# 外网A要控制内网B

ssh -NfR 1234:localhost:2223 user1@123.123.123.123 -p22    # 将A主机的1234端口和B主机的2223端口绑定，相当于远程端口映射
ss -ant   # 这时在A主机上sshd会listen本地1234端口
# LISTEN     0    128    127.0.0.1:1234       *:*
ssh localhost -p1234    # 在A主机连接本地1234端口

}
```

##### 路由

```
route                           # 查看路由表
route add default  gw 192.168.1.1  dev eth0                        # 添加默认路由
route add -net 172.16.0.0 netmask 255.255.0.0 gw 10.39.111.254     # 添加静态路由网关
route del -net 172.16.0.0 netmask 255.255.0.0 gw 10.39.111.254     # 删除静态路由网关


    静态路由{

        vim /etc/sysconfig/static-routes
        any net 192.168.12.0/24 gw 192.168.0.254
        any net 192.168.13.0/24 gw 192.168.0.254

    }
```

#### 磁盘

##### 常用

````shell
df -Ph                                          # 查看硬盘容量
df -T                                           # 查看磁盘分区格式
df -i                                           # 查看inode节点   如果inode用满后无法创建文件
du -h dir                                       # 检测目录下所有文件大小
du -sh *                                        # 显示当前目录中子目录的大小
mount -l                                        # 查看分区挂载情况
fdisk -l                                        # 查看磁盘分区状态
fdisk /dev/hda3                                 # 分区
mkfs -t ext3  /dev/hda3                         # 格式化分区
fsck -y /dev/sda6                               # 对文件系统修复
lsof |grep delete                               # 释放进程占用磁盘空间  列出进程后，查看文件是否存在，不存在则kill掉此进程
tmpwatch -afv 10   /tmp                         # 删除10小时内未使用的文件  勿在重要目录使用
cat /proc/filesystems                           # 查看当前系统支持文件系统
mount -o remount,rw /                           # 修改只读文件系统为读写
iotop                                           # 进程占用磁盘IO情况   yum install iotop
smartctl -H /dev/sda                            # 检测硬盘状态  # yum install smartmontools
smartctl -i /dev/sda                            # 检测硬盘信息
smartctl -a /dev/sda                            # 检测所有信息
e2label /dev/sda5                               # 查看卷标
e2label /dev/sda5 new-label                     # 创建卷标
ntfslabel -v /dev/sda8 new-label                # NTFS添加卷标
tune2fs -j /dev/sda                             # ext2分区转ext3分区
tune2fs -l /dev/sda                             # 查看文件系统信息
mke2fs -b 2048 /dev/sda5                        # 指定索引块大小
dumpe2fs -h /dev/sda5                           # 查看超级块的信息
mount -t iso9660 /dev/dvd  /mnt                 # 挂载光驱
mount -t ntfs-3g /dev/sdc1 /media/yidong        # 挂载ntfs硬盘
mount -t nfs 10.0.0.3:/opt/images/  /data/img   # 挂载nfs 需要重载 /etc/init.d/nfs reload  重启需要先启动 portmap 服务
mount -o loop  /software/rhel4.6.iso   /mnt/    # 挂载镜像文件
````

##### IO系能

````
iostat -x 1 10

% user       # 显示了在用户级(应用程序)执行时生成的 CPU 使用率百分比。
% system     # 显示了在系统级(内核)执行时生成的 CPU 使用率百分比。
% idle       # 显示了在 CPU 空闲并且系统没有未完成的磁盘 I/O 请求时的时间百分比。
% iowait     # 显示了 CPU 空闲期间系统有未完成的磁盘 I/O 请求时的时间百分比。

rrqm/s       # 每秒进行 merge 的读操作数目。即 delta(rmerge)/s
wrqm/s       # 每秒进行 merge 的写操作数目。即 delta(wmerge)/s
r/s          # 每秒完成的读 I/O 设备次数。即 delta(rio)/s
w/s          # 每秒完成的写 I/O 设备次数。即 delta(wio)/s
rsec/s       # 每秒读扇区数。即 delta(rsect)/s
wsec/s       # 每秒写扇区数。即 delta(wsect)/s
rkB/s        # 每秒读K字节数。是 rsect/s 的一半，因为每扇区大小为512字节。(需要计算)
wkB/s        # 每秒写K字节数。是 wsect/s 的一半。(需要计算)
avgrq-sz     # 平均每次设备I/O操作的数据大小 (扇区)。delta(rsect+wsect)/delta(rio+wio)
avgqu-sz     # 平均I/O队列长度。即 delta(aveq)/s/1000 (因为aveq的单位为毫秒)。
await        # 平均每次设备I/O操作的等待时间 (毫秒)。即 delta(ruse+wuse)/delta(rio+wio)
svctm        # 平均每次设备I/O操作的服务时间 (毫秒)。即 delta(use)/delta(rio+wio)
%util        # 一秒中有百分之多少的时间用于 I/O 操作，或者说一秒中有多少时间 I/O 队列是非空的。即 delta(use)/s/1000 (因为use的单位为毫秒)

````

标准

```
1、 如果 %util 接近 100%，说明产生的I/O请求太多，I/O系统已经满负荷，该磁盘可能存在瓶颈。
2、 idle 小于70% IO压力就较大了,一般读取速度有较多的wait.
3、 同时可以结合 vmstat 查看查看b参数(等待资源的进程数)和wa参数(IO等待所占用的CPU时间的百分比,高过30%时IO压力高)
4、 svctm 一般要小于 await (因为同时等待的请求的等待时间被重复计算了),svctm 的大小一般和磁盘性能有关,CPU/内存的负荷也会对其有影响,请求过多也会间接导致 svctm 的增加. await 的大小一般取决于服务时间(svctm) 以及 I/O 队列的长度和 I/O 请求的发出模式. 如果 svctm 比较接近 await,说明 I/O 几乎没有等待时间;如果 await 远大于 svctm,说明 I/O 队列太长,应用得到的响应时间变慢,如果响应时间超过了用户可以容许的范围,这时可以考虑更换更快的磁盘,调整内核 elevator 算法,优化应用,或者升级 CPU
5、 队列长度(avgqu-sz)也可作为衡量系统 I/O 负荷的指标，但由于 avgqu-sz 是按照单位时间的平均值，所以不能反映瞬间的 I/O 洪水。

```

##### iotop

```shell

# 监视进程磁盘I/O

yum install iotop

-o        # 只显示有io操作的进程
-b        # 批量显示，无交互，主要用作记录到文件。
-n NUM    # 显示NUM次，主要用于非交互式模式。
-d SEC    # 间隔SEC秒显示一次。
-p PID    # 监控的进程pid。
-u USER   # 监控的进程用户。

# 左右箭头：改变排序方式，默认是按IO排序。
r         # 改变排序顺序。
o         # 只显示有IO输出的进程。
p         # 进程/线程的显示方式的切换。
a         # 显示累积使用量。
q         # 退出。

```

创建swap文件

```shell
dd if=/dev/zero of=/swap bs=1024 count=4096000            # 创建一个足够大的文件
# count的值等于1024 x 你想要的文件大小, 4096000是4G
mkswap /swap                      # 把这个文件变成swap文件
swapon /swap                      # 启用这个swap文件
/swap swap swap defaults 0 0      # 在每次开机的时候自动加载swap文件, 需要在 /etc/fstab 文件中增加一行
cat /proc/swaps                   # 查看swap
swapoff -a                        # 关闭swap
swapon -a                         
```

##### 新硬盘挂载

```shell
fdisk /dev/sdc
p    #  打印分区
d    #  删除分区
n    #  创建分区，（一块硬盘最多4个主分区，扩展占一个主分区位置。p主分区 e扩展）
w    #  保存退出
mkfs.ext4 -L 卷标  /dev/sdc1            # 格式化相应分区
mount /dev/sdc1  /mnt                  # 挂载
vi /etc/fstab                          # 添加开机挂载分区
LABEL=/data            /data                   ext4    defaults        1 2      # 用卷标挂载
/dev/sdb1              /data4                  ext4    defaults        1 2      # 用真实分区挂载
/dev/sdb2              /data4                  ext4    noatime,defaults        1 2

第一个数字"1"该选项被"dump"命令使用来检查一个文件系统应该以多快频率进行转储，若不需要转储就设置该字段为0
第二个数字"2"该字段被fsck命令用来决定在启动时需要被扫描的文件系统的顺序，根文件系统"/"对应该字段的值应该为1，其他文件系统应该为2。若该文件系统无需在启动时扫描则设置该字段为0
当以 noatime 选项加载（mount）文件系统时，对文件的读取不会更新文件属性中的atime信息。设置noatime的重要性是消除了文件系统对文件的写操作，文件只是简单地被系统读取。由于写操作相对读来说要更消耗系统资源，所以这样设置可以明显提高服务器的性能.wtime信息仍然有效，任何时候文件被写，该信息仍被更新。

mount -a    # 自动加载 fstab 文件挂载，避免配置错误，系统无法重启






#centos6
fdisk -l /dev/sda
fdisk /dev/sda
n p 3 enter enter  w
partx -a /dev/sda
pvcreate /dev/sda3
vgextend vg_centos6 /dev/sda3
lvextend /dev/vg_centos6/lv_root /dev/sda3
resize2fs /dev/vg_centos6/lv_root

#centos7
fdisk -l /dev/sda
fdisk /dev/sda
n p 3 enter enter t 3 8e w
partprobe
mkfs -t xfs /dev/sda3
pvcreate /dev/sda3
vgextend centos /dev/sda3
lvextend -L +399.99G /dev/centos/root /dev/sda3
xfs_growfs /dev/centos/root

lvcreate -l 100%FREE /dev/vg_adj



##添加硬盘不重启
echo "- - -" > /sys/class/scsi_host/host0/scan
```

##### 2T和16T分区

```shell
parted /dev/sdb                # 针对磁盘分区
(parted) mklabel gpt           # 设置为 gpt
(parted) print
(parted) mkpart  primary 0KB 22.0TB        # 指定分区大小
Is this still acceptable to you?
Yes/No? Yes
Ignore/Cancel? Ignore
(parted) print
Model: LSI MR9271-8i (scsi)
Disk /dev/sdb: 22.0TB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Number  Start   End     Size    File system  Name     Flags
1      17.4kB  22.0TB  22.0TB               primary
(parted) quit

mkfs.ext4 /dev/sdb1        # e2fsprogs升级后支持大于16T硬盘

# 大于16T的单个分区ext4格式化报错，需要升级e2fsprogs
Size of device /dev/sdb1 too big to be expressed in 32 bits using a blocksize of 4096.

yum -y install xfsprogs
mkfs.xfs -f /dev/sdb1              # 大于16T单个分区也可以使用XFS分区,但inode占用很大,对大量的小文件支持不太好

```

##### raid原理

````
        raid0至少2块硬盘.吞吐量大,性能好,同时读写,但损坏一个就完蛋
        raid1至少2块硬盘.相当镜像,一个存储,一个备份.安全性比较高.但是性能比0弱
        raid5至少3块硬盘.分别存储校验信息和数据，坏了一个根据校验信息能恢复
        raid6至少4块硬盘.两个独立的奇偶系统,可坏两块磁盘,写性能非常差

````

#### 用户

```shell
users                                      # 显示所有的登录用户
groups                                     # 列出当前用户和他所属的组
who -q                                     # 显示所有的登录用户
groupadd                                   # 添加组
useradd user                               # 建立用户
passwd username                            # 修改密码
userdel -r                                 # 删除帐号及家目录
chown -R user:group                        # 修改目录拥有者(R递归)
chown y\.li:mysql                          # 修改所有者用户中包含点"."
umask                                      # 设置用户文件和目录的文件创建缺省屏蔽值
chgrp                                      # 修改用户组
finger                                     # 查找用户显示信息
echo "xuesong" | passwd user --stdin       # 非交互修改密码
useradd -g www -M  -s /sbin/nologin  www   # 指定组并不允许登录的用户,nologin允许使用服务
useradd -g www -M  -s /bin/false  www      # 指定组并不允许登录的用户,false最为严格
useradd -d /data/song -g song song         # 创建用户并指定家目录和组
usermod -l newuser olduser                 # 修改用户名
usermod -g user group                      # 修改用户所属组
usermod -d dir -m user                     # 修改用户家目录
usermod -G group user                      # 将用户添加到附加组
gpasswd -d user group                      # 从组中删除用户
su - user -c " #cmd1; "                    # 切换用户执行
```

##### 恢复密码

```shell
# 即进入单用户模式: 在linux出现grub后，在安装的系统上面按"e"，然后出现grub的配置文件，按键盘移动光标到第二行"Ker……"，再按"e"，然后在这一行的结尾加上：空格 single或者空格1回车，然后按"b"重启，就进入了"单用户模式"

```

#### 脚本

