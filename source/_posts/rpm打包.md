---
title: rpm打包
date: 2018-02-28 08:42:06
tags: rpm
categories: linux
copyright: true
password: woshizhu
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



