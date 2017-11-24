---
title: centos密码破解
date: 2017-04-24 20:26:01
tags: 
categories: linux
copyright: true
---
> 破解centos密码

<!--more-->


##### centos7 

```
rd.break方法：
- 启动的时候，在启动界面，相应启动项，内核名称上按“e”；
- 进入后，找到linux16开头的地方，按“end”键到最后，输入rd.break，按ctrl+x进入；
- 进去后输入命令mount，发现根为/sysroot/，并且不能写，只有ro=readonly权限；
- mount -o remount,rw /sysroot/，重新挂载，之后mount，发现有了r,w权限；
- chroot /sysroot/ 改变根；
- passwd  root 修改root密码
- 如果开了seLinux.记得 touch /.autorelabel 这句是为了selinux生效,否则系统无法正常启动
- exit 退出重启

init方法：

- 启动系统，并在GRUB2启动屏显时，按下e键进入编辑模式。
- 在linux16/linux/linuxefi所在参数行尾添加以下内容：init=/bin/sh
-  按Ctrl+x启动到shell。
- 挂载文件系统为可写模式：mount –o remount,rw /
- 运行passwd,并按提示修改root密码。
- 如何之前系统启用了selinux，必须运行以下命令，否则将无法正常启动系统：touch /.autorelabel
-  运行命令exec /sbin/init来正常启动，或者用命令exec /sbin/reboot重启

```
##### centos6
```

 - 重启系统，在开机启动的时候能看到引导目录，用上下方向键选择你忘记密码的那个系统，然后按e。
 - 选择第二项—kernel，然后继续按”E”
 - 在rhgb quiet后回车输入single或者1，然后回车
 - 然后回车后，回到该界面，然后按b进行重新引导系统
 - 启动后，我们发现直接进入系统，无需要输入账户及密码
 - 进入后，我们可以根据passwd root来修改密码了
 ```



___