---
title: rpm打包
date: 2018-02-28 08:42:06
tags: rpm
categories: linux
copyright: true
---

> RPM（Redhat Package Manager）是用于Redhat、CentOS、Fedora等Linux 分发版（distribution）的常见的软件包管理器。因为它允许分发已编译的软件，所以用户只用一个命令就可以安装软件.

<!--more-->

#### 安装
```
yum install rpm-build
yum install rpmdevtools
```

#### 使用
`rpmdev-setuptree`命令,可在`$HOme`下生产一个标注的工作空间'rpmbuild'
结构如下
```
$ tree rpmbuild
rpmbuild
├── BUILD
├── RPMS
├── SOURCES
├── SPECS
└── SRPMS
```

如果未安装`rpmdevtools` 手动创建也是可以的
`mkdir -p ~/rpmbuild/{BUILD,RPMS,SOURCES,SPECS,SRPMS}`

#### 目录

|目录名| 说明 | macros中的宏名|
| - | - | - |
|~/rpmbuild/|rpmbuild文件夹|%_topdir|
|~/rpmbuild/BUILD  | 编译rpm包的临时目录| %_builddir|
|~/rpmbuild/BUILDROOT   |编译后生成的软件临时安装目录|  %_buildrootdir|
|~/rpmbuild/RPMS    |最终生成的可安装rpm包的所在目录|   %_rpmdir|
|~/rpmbuild/SOURCES |所有源代码和补丁文件的存放目录| %_sourcedir|
|~/rpmbuild/SPECS   |存放SPEC文件的目录(重要)| %_specdir|
|~/rpmbuild/SRPMS   |软件最终的rpm源码格式存放路径(暂时忽略掉，别挂在心上)|   %_srcrpmdir|

#### 流程

1. 源代码放在`%_sourcedir`下,一般是`.tar.gz`之类,或文件
2. 编译,在`%_builddir`中进行,源码包解压后复制到此目录即可
3. 安装,把软件包应该包含的内容（比如二进制文件、配置文件、man文档等）复制到`%_buildrootdir`中，并按照实际安装后的目录结构组装，比如二进制命令可能会放在`/usr/bin`下，那么就在`%_buildrootdir`下也按照同样的目录结构放置
4. 配置,安装前,中,后,的一些工作,以及卸载前的工作.
5. 检测,软件是否正常运行. 可选
6. 生成rpm包,放到`%_rpmdir`下

以上流程均是通过编写SPEC文件进行操作.

|阶段  |读取的目录  | 写入的目录  | 具体动作|
| - | - | - | - |
|%prep  | %_sourcedir| %_builddir | 读取位于 %_sourcedir 目录的源代码和 patch 。之后，解压源代码至 %_builddir 的子目录并应用所有 patch|
|%build | %_builddir | %_builddir  |编译位于 %_builddir 构建目录下的文件。通过执行类似 ./configure && make 的命令实现|
|%install  |  %_builddir | %_buildrootdir|  读取位于 %_builddir 构建目录下的文件并将其安装至 %_buildrootdir 目录。这些文件就是用户安装 RPM 后，最终得到的文件。注意一个奇怪的地方: 最终安装目录 不是 构建目录。通过执行类似 make install 的命令实现|
|%check | %_builddir | %_builddir | 检查软件是否正常运行。通过执行类似 make test 的命令实现。很多软件包都不需要此步。|
|bin |%_buildrootdir | %_rpmdir  |  读取位于 %_buildrootdir 最终安装目录下的文件，以便最终在 %_rpmdir 目录下创建 RPM 包。在该目录下，不同架构的 RPM 包会分别保存至不同子目录， noarch 目录保存适用于所有架构的 RPM 包。这些 RPM 文件就是用户最终安装的 RPM 包。|
|src |%_sourcedir |%_srcrpmdir |创建源码 RPM 包（简称 SRPM，以.src.rpm 作为后缀名），并保存至 %_srcrpmdir 目录。SRPM 包通常用于审核和升级软件包。|



#### SPEC 
zabbix agentd rpm包构建参考
```
%define zabbix_user zabbix                  
#自定义宏，名字为zabbix_user值为zabbix,%{zabbix_user}引用

Name:   zabbix                              
#软件包的名字，后面可用%{name}引用

Version:    3.0.14                           
#软件的实际版本号，可使用%{version}引用

Release:    1%{?dist}                       
#发布序列号，标明第几次打包  

Summary:    zabbix_agentd                   
#软件包内容概要

Group:      zabbix                          
#软件包分组

License:    GPL
#授权许可方式

URL:        blog.dl1548.com
#软件的主页

#源代码包，可以有Source0,Source1等源,对应%install下
Source0:    zabbix-3.0.14.tar.gz
Source1:    discover_disk.pl
Source2:    tcp_conn_status.sh
Source3:    zbx_parse_iostat_values.sh
Source4:    tcp-status-params.conf
Source5:    zbx_parse_iostat_values.conf

BuildRequires:      net-tools
#gcc, gcc-c++            
#制作rpm包时，所依赖的基本库

Requires:      net-tools
#gcc, gcc-c++       
#安装rpm包时，所依赖的软件包

%description                                
#定义rpm包的描述信息
Zabbix agentd
creata by jingkunsystem-lizili

%pre                                        
#rpm包安装前执行的脚本
grep zabbix /etc/passwd > /dev/null
if [ $? != 0 ] 
then useradd zabbix -M -s /sbin/nologin
fi
[ -d /usr/local/zabbix ]||rm -rf /usr/local/zabbix*
rm -rf /etc/zabbix*


%post                                       
#rpm包安装后执行的脚本

sed -i "/BASEDIR=\/usr\/local/c\BASEDIR=\/usr\/local\/%{name}" /etc/init.d/zabbix_agentd

%preun                                      
#rpm卸载前执行的脚本
/etc/init.d/zabbix_agentd stop

%postun                                     
#rpm卸载后执行的脚本
userdel  zabbix
rm -rf /usr/local/zabbix*

%prep

%setup -q                                   

%build                                      
#定义编译软件包时的操作
./configure --prefix=/usr/local/%{name}  --enable-agent
make -j8 %{?_smp_mflags}

%install                                    
#定义安装软件包，使用默认值即可
test -L %{buildroot}/usr/local/%{name} && rm -f %{buildroot}/usr/local/%{name}
install -d %{buildroot}/etc/profile.d
install -d %{buildroot}/etc/init.d
make install DESTDIR=%{buildroot}

#其他脚本
install -p -D -m 0755 %{SOURCE1}        %{buildroot}/usr/local/%{name}/lib/discover_disk.pl
install -p -D -m 0755 %{SOURCE2}        %{buildroot}/usr/local/%{name}/lib/tcp_conn_status.sh
install -p -D -m 0755 %{SOURCE3}        %{buildroot}/usr/local/%{name}/lib/zbx_parse_iostat_values.sh
#其他配置文件
install -p -D -m 0755 %{SOURCE4}        %{buildroot}/usr/local/%{name}/etc/zabbix_agentd.conf.d/tcp-status-params.conf
install -p -D -m 0755 %{SOURCE5}        %{buildroot}/usr/local/%{name}/etc/zabbix_agentd.conf.d/zbx_parse_iostat_values.conf

echo 'export PATH=/etc/zabbix/bin:/etc/zabbix/sbin:$PATH' >> %{buildroot}/etc/profile.d/%{name}.sh
cp %{_builddir}/%{name}-%{version}/misc/init.d/fedora/core/zabbix_agentd       %{buildroot}/etc/init.d/zabbix_agentd

%files
%defattr (-,root,root,0755)                                      
#定义rpm包安装时创建的相关目录及文件
#在该选项中%defattr (-,root,root)一定要注意。它是指定安装文件的属性，分别是(mode,owner,group)，-表示默认值，对文本文件是0644，可执行文件是0755。
/usr/local/%{name}/*
/usr/local/zabbix/lib/*
/usr/local/zabbix/etc/zabbix_agentd.conf.d/*
/etc/init.d/zabbix_agentd
/etc/profile.d/%{name}.sh

%changelog                                  
#主要用于软件的变更日志。该选项可有可无

%clean 
rm -rf %{buildroot}                         
#清理临时文件

```

脚本参考(主要针对centos7 内核)
```
#/usr/bin/bash
#ip= ip addr |grep inet |egrep -v "inet6|127.0.0.1" |awk '{print $2}' |awk -F "/" '{print $1}'

# get server ip

echo -e "\n NOTICE! Check the package net-tools is installed \n "
read -p  "zabbix server ip：" zabbix_server
if [ ! -n "$zabbix_server" ]; then
    echo "you have not input server ip, exit"
    exit 1
fi

# get agent ip
read -p  "local ip: " local_ip
if [ ! -n "$local_ip" ]; then
    echo "you have not input local ip, exit"
    exit 1
fi

echo "start to install..."
sleep 1
#version centos7
#cat /etc/*release | egrep -i "centos|rhel|red hat|redhat"

echo "check os version..."
sleep 1


#version centos7
cat /etc/*release | grep -q -i "centos\|rhel\|red hat\|redhat"
if [[ $? -eq 0 ]];then
    cat /etc/*release | grep -q -i "7\."
    if [[ $? -eq 0 ]];then
        sleep 1
        rpm -ivh zabbix-3.0.14-1.el7.centos.x86_64.rpm
        echo "modify the agentd configuration"
        sleep 1
        sed -i "/^Server=/c\Server=${zabbix_server}" /usr/local/zabbix/etc/zabbix_agentd.conf
        sed -i "/^ServerActive=/c\ServerActive=${zabbix_server}" /usr/local/zabbix/etc/zabbix_agentd.conf
        sed -i "/^Hostname=/c\Hostname=${local_ip}" /usr/local/zabbix/etc/zabbix_agentd.conf
        sed -i "/Timeout=3/c\Timeout=30" /usr/local/zabbix/etc/zabbix_agentd.conf
        sed -i "260i\Include=/usr/local/zabbix/etc/zabbix_agentd.conf.d/*.conf" /usr/local/zabbix/etc/zabbix_agentd.conf
        systemctl daemon-reload
        /etc/init.d/zabbix_agentd start
        chkconfig zabbix_agentd on
        echo "SUCCESSFUL !"
        exit
    fi
fi

```