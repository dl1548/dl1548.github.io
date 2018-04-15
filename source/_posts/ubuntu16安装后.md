---
title: Ubuntu16安装后
date: 2017-04-22 20:20:49
tags: ubuntu
categories: linux
copyright: true
---

<!--more-->

#### 更新root密码
```
sudo su
passwd root
```
#### 触摸板控制
```
重启后失效
关闭触摸板： sudo modprobe -r psmouse 
打开触摸板： sudo modprobe psmouse

方案二,安装应用进行管理
sudo add-apt-repository ppa:atareao/atareao
sudo apt-get update
sudo apt-get install touchpad-indicator


```
#### 更新补丁
```
sudo apt-get update 
sudo apt-get upgrade
```
#### 卸载libreOffice
```
sudo apt-get remove libreoffice-common  
```

#### 删Amazon的链接
```
sudo apt-get remove unity-webapps-common 
```

#### 删除不常用软件
```
注意按需删除，如无强迫症不建议删除。比如雷鸟邮件客户端，其实也挺好用的。另：删除后如果需要以后可在安装
sudo apt-get remove thunderbird totem rhythmbox empathy brasero simple-scan gnome-mahjongg aisleriot 
sudo apt-get remove gnome-mines cheese transmission-common gnome-orca webbrowser-app gnome-sudoku  landscape-client-ui-install  

sudo apt-get remove onboard deja-dup 

```
#### 安装Unity Tweak Tool 
可用来启动点击图标最小化，调整主题等功能
```
sudo apt install unity-tweak-tool
```
#### 安装wps
```
sudo apt install wps-office
#消除wps打开提示字体缺失
链接: https://pan.baidu.com/s/1jIR4R8U 密码: 1ka9
#下载解压，切换到目录内
sudo cp -rp * /usr/share/fonts/wps-office/
#生成字体的索引信息：
sudo mkfontscale
sudo mkfontdir
#更新字体缓存
sudo fc-cache
```

#### 安装chrome
```
官网下载chrome的Linux版本，或者
链接: http://pan.baidu.com/s/1i59oUpB 密码: 5rnt

sudo apt-get install libappindicator1 libindicator7 
sudo dpkg -i google-chrome-stable_current_amd64.deb 
sudo apt-get -f install 
```
#### 安装flash
```
系统设置--软件和更新--其他软件----（勾选第一个）
sudo apt-get install flashplugin-installer 
```
#### 安装下载应用
```
sudo apt-get install uget  #下载工具
sudo apt-get install aria2   #插件
#打开软件
编辑---设置---插件，启用aria2，启用aria2插件
分类---属性---默认设置，设置默认的下载路径。最大连接数等
```


####安装第三方wechat（electronic-wechat-linux）
```
自行搜索安装包，或者
链接: http://pan.baidu.com/s/1dFalmVn 密码: q4es
开箱即用
```
#### 安装网易云音乐
```
去网易官网下载包或者
链接: https://pan.baidu.com/s/1pLM9QR5 密码: 5jrd
sudo dpkg -i 包名
sudo apt-get -f  install 包名（-f修复依赖关系）
```
#### 安装便签
```
sudo apt-get install  xpad
```

#### 安装搜狗输入法
```
添加源 sudo vim /etc/apt/sources.list.d/ubuntukylin.list
deb http://archive.ubuntukylin.com:10006/ubuntukylin trusty main

sudo apt-get update
sudo apt-get install sogoupinyin
```
#### 安装大小写提示
```
sudo add-apt-repository ppa:tsbarnes/indicator-keylock
sudo apt-get update
sudo apt-get install indicator-keylock
```
#### 安装vim git filezilla
```
sudo apt-get install vim git filezilla
```

#### 安装sublime 3
```
sudo add-apt-repository ppa:webupd8team/sublime-text-3 
sudo apt-get update  
sudo apt-get install sublime-text

#注册
—– BEGIN LICENSE —–
Michael Barnes
Single User License
EA7E-821385
8A353C41 872A0D5C DF9B2950 AFF6F667
C458EA6D 8EA3C286 98D1D650 131A97AB
AA919AEC EF20E143 B361B1E7 4C8B7F04
B085E65E 2F5F5360 8489D422 FB8FC1AA
93F6323C FD7F7544 3F39C318 D95E6480
FCCC7561 8A4A1741 68FA4223 ADCEDE07
200C25BE DBBC4855 C4CFB774 C5EC138C
0FEC1CEF D9DCECEC D3A5DAD1 01316C36
—— END LICENSE ——

win系统
----- BEGIN LICENSE -----
TwitterInc
200 User License
EA7E-890007
1D77F72E 390CDD93 4DCBA022 FAF60790
61AA12C0 A37081C5 D0316412 4584D136
94D7F7D4 95BC8C1C 527DA828 560BB037
D1EDDD8C AE7B379F 50C9D69D B35179EF
2FE898C4 8E4277A8 555CE714 E1FB0E43
D5D52613 C3D12E98 BC49967F 7652EED2
9D2D2E61 67610860 6D338B72 5CF95C69
E36B85CC 84991F19 7575D828 470A92AB
------ END LICENSE ------



ctrl + 飘（ESC下面）然后输入以下代码
import urllib.request,os; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); open(os.path.join(ipp, pf), 'wb').write(urllib.request.urlopen( 'http://sublime.wbond.net/' + pf.replace(' ','%20')).read())

win系统
import urllib.request,os,hashlib; h = 'eb2297e1a458f27d836c04bb0cbaf282' + 'd0e7a3098092775ccb37ca9d6b2e4b7d'; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); by = urllib.request.urlopen( 'http://packagecontrol.io/' + pf.replace(' ', '%20')).read(); dh = hashlib.sha256(by).hexdigest(); print('Error validating download (got %s instead of %s), please try manual install' % (dh, h)) if dh != h else open(os.path.join( ipp, pf), 'wb' ).write(by)

如果不成功，下载包 手动安装
https://packagecontrol.io/installation

然后按下Ctrl+Shift+P调出命令面板
输入install 调出 Install Package 选项并回车，然后在列表中选中要安装的插件。

ChineseLocalizations #中文汉化

Boxy Theme  #主题
SFTP   #如其名
Sublime CodeIntel #代码提示和补全插件
Bracket Highlighter #匹配括号
Emmet  #前端神器（前身Zen Coding）
Alignment  #代码对齐
SidebarEnhancements #侧边栏功能增加

Markdown Editing // Markdown编辑和语法高亮支持 
Markdown Preview// Markdown导出html预览支持 
shift+ctrl+p  输入mp可浏览器浏览

align arguments
ChineseLocalIzations
ConvertToUTF8   #解决中文乱码
Ctags
filediff
pretty json
wordHighlight
zzzzzzzz-localization


Perference---->Package Settings------->SFTP------->setting user
{
    "email": "xiaosong@xiaosong.me",
    "product_key": "d419f6-de89e9-0aae59-2acea1-07f92a"
}

##个人配置
    "font_size": 12,
    "ignored_packages":
    [
        "Vintage"
    ],
     // 设置tab的大小为4
    "tab_size":4,
    // 使用空格代替tab
    "translate_tabs_to_spaces": true,
 
    // 添加行宽标尺
    "rulers": [80, 100],
    // 显示空白字符
    "draw_white_space": "all",
    // 保存时自动去除行末空白
    "trim_trailing_white_space_on_save": true,
    // 保存时自动增加文件末尾换行
    "ensure_newline_at_eof_on_save": true,
    // 默认编码格式
    "default_encoding": "UTF-8"

#解决无法输入中文的问题
git clone https://github.com/lyfeyaj/sublime-text-imfix.git
或链接: https://pan.baidu.com/s/1gf1l1jP 密码: piug
移动subl到/usr/bin/，移动sublime-imfix.so到/opt/sublime_text/（sublime的安装目录）
命令行 输入 subl 如果启动成功，即可输入中文
```
#### 经典下拉菜单指示器
```
sudo add-apt-repository ppa:diesch/testing
sudo apt-get update 
sudo apt-get install classicmenu-indicator
```
#### 安装截图
```
自带的也很好用，但是shutter更丰富。
sudo apt-get install shutter
```
#### 安装unrar
```
用来解压rar
sudo apt-get install unrar
```
#### vpn安装
(本人根据公司情况安装,并不适合每个人)
```
http://support.arraynetworks.com.cn/troubleshooting
下载相关软件以及说明书
或 链接: https://pan.baidu.com/s/1hr9GD7Y 密码: d2kk

```

#### 安装kvm虚拟机

```
1. 检查CPU是否支持虚拟化技术
egrep "(svm|vmx)" /proc/cpuinfo
有结果输出,类似如下信息,则可以安装
flags       : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc aperfmperf eagerfpu pni pclmulqdq dtes64 monitor ds_cpl vmx est tm2 ssse3 sdbg fma cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm abm epb tpr_shadow vnmi flexpriority ept vpid fsgsbase tsc_adjust bmi1 avx2 smep bmi2 erms invpcid xsaveopt dtherm ida arat pln pts

2. 安装kvm依赖
sudo apt-get install qemu-kvm
sudo apt-get install qemu
sudo apt-get install virt-manager
sudo apt-get install virt-viewer 
sudo apt-get install libvirt-bin 
sudo apt-get install bridge-utils

3. 配置网络
sudo cp /etc/network/interfaces /etc/network/interfaces-bak   #备份配置
sudo vim /etc/network/interfaces
#修改配置如下
auto br0
iface br0 inet static
address 192.168.1.130
network 192.168.1.0
netmask 255.255.255.0
broadcast 192.168.1.255
#gateway 192.168.1.1
dns-nameservers 223.5.5.5
bridge_ports eth0
bridge_stp off

4. 重启
sudo reboot
5. 打开程序
virt-manager

```

#### 美化
```
#Flatabulous主题
sudo add-apt-repository ppa:noobslab/themes
sudo apt-get update
sudo apt-get install flatabulous-theme
#图包
sudo add-apt-repository ppa:noobslab/icons
sudo apt-get update
sudo apt-get install ultra-flat-icons

#Numix主题 （图包好看 可配Flatabulous主题）
sudo add-apt-repository ppa:numix/ppa
sudo apt-get update
sudo apt-get install numix-gtk-theme numix-icon-theme-circle

#壁纸更换
sudo apt-get install variety
```

___
