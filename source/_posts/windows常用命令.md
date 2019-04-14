---
title: windows常用命令
date: 1111-11-11 11:11:13
tags: -个人备忘手册
categories: windows
copyright: true
password: woshizhu
---

#### wmic

##### 先决条件

a. 启动Windows Management Instrumentation服务，开放TCP135端口。
b. 本地安全策略的“网络访问: 本地帐户的共享和安全模式”应设为“经典-本地用户以自己的身份验证”。

1. wmic /node:"192.168.1.20" /user:"domain\administrator" /password:"123456"

##### 硬件信息(管理)

获取磁盘资料：
`wmic DISKDRIVE get deviceid,Caption,size,InterfaceType`

获取分区资料：
`wmic LOGICALDISK get name,Description,filesystem,size,freespace`

获取CPU资料:
`wmic cpu get name,addresswidth,processorid`

获取主板资料:
`wmic BaseBoard get Manufacturer,Product,Version,SerialNumber`

获取内存数:
`wmic memlogical get totalphysicalmemory`

获得品牌机的序列号:
`wmic csproduct get IdentifyingNumber`

获取声卡资料:
`wmic SOUNDDEV get ProductName`

获取屏幕分辨率
`wmic DESKTOPMONITOR where Status='ok' get ScreenHeight,ScreenWidth`

##### PROCESS(进程管理)：

列出进程
`wmic process list brief`
(Full显示所有、Brief显示摘要、Instance显示实例、Status显示状态)

wmic 获取进程路径: 
`wmic process where name="jqs.exe" get executablepath`

wmic 创建新进程 

```
wmic process call create notepad
wmic process call create "C:\Program Files\Tencent\QQ\QQ.exe" 
wmic process call create "shutdown.exe -r -f -t 20"
```

wmic 删除指定进程: 

```
wmic process where name="qq.exe" call terminate 
wmic process where processid="2345" delete 
wmic process 2345 call terminate
```

wmic 删除可疑进程

```
wmic process where "name='explorer.exe' and executablepath<>'%SystemDrive%\windows\explorer.exe'" delete

wmic process where "name='svchost.exe' and ExecutablePath<>'C:\WINDOWS\system32\svchost.exe'" call Terminate

```

##### USERACCOUNT(账号管理)：

更改当前用户名 

```
WMIC USERACCOUNT where "name='%UserName%'" call rename newUserName 

WMIC USERACCOUNT create /?
```



##### SHARE(共享管理)：

建立共享

`WMIC SHARE CALL Create "","test","3","TestShareName","","c:\test",0`
(可使用 WMIC SHARE CALL Create /? 查看create后的参数类型)

删除共享
`WMIC SHARE where name="C$" call delete`
`WMIC SHARE where path='c:\\test' delete`

##### SERVICE(服务管理)：

更改telnet服务启动类型[Auto|Disabled|Manual]
`wmic SERVICE where name="tlntsvr" set startmode="Auto"`

运行telnet服务
`wmic SERVICE where name="tlntsvr" call startservice`

停止ICS服务
`wmic SERVICE where name="ShardAccess" call stopservice`

删除test服务
`wmic SERVICE where name="test" call delete`

##### FSDIR(目录管理)

列出c盘下名为test的目录
`wmic FSDIR where "drive='c:' and filename='test'" list`

删除c:\good文件夹
`wmic fsdir "c:\\test" call delete`

重命名c:\test文件夹为abc
`wmic fsdir "c:\\test" rename "c:\abc"`
`wmic fsdir where (name='c:\\test') rename "c:\abc"`

复制文件夹
`wmic fsdir where name='d:\\test' call copy "c:\\test"`

##### datafile(文件管理)

重命名
`wmic datafile "c:\\test.txt" call rename c:\abc.txt`

##### 任务计划：

`wmic job call create "notepad.exe",0,0,true,false,********154800.000000+480`
`wmic job call create "explorer.exe",0,0,1,0,********154600.000000+480`



##### 自动脚本

```
@echo off
::get os data
wmic os get Caption,Version,SerialNumber,InstallDate /value
echo ---
:: get ip data
wmic nicconfig where "IPEnabled  = True" get Description,DefaultIPGateway,IPAddress,IPSubnet,MACAddress /value
echo ---
:: get system data
wmic COMPUTERSYSTEM get Name,NumberOfLogicalProcessors,NumberOfProcessors,TotalPhysicalMemory /value
echo ---
:: get cpu data
wmic CPU get NAME,DeviceID /value
echo ---
:: get phydisk data
wmic diskdrive get serialnumber,DeviceID,Size /value
echo ---
:: get logicaldisk data
wmic logicaldisk get DeviceID,FreeSpace,Size,VolumeSerialNumber /value


::wmic nicconfig where "IPEnabled  = True" get Description,DefaultIPGateway,IPAddress,IPSubnet,MACAddress /value|findstr [1-9]
:: 可去除不相关的回车换行等
```



#### cmd(bat)

##### 取ip

```
@echo off 
for /f "tokens=2 delims=:" %%b in ('ipconfig^|find /i "ip"') do set minion=%%b
echo %minion%
```

#### powershell

##### 获取系统硬件信息

```
$os_version=Get-CimInstance -ClassName Win32_OperatingSystem | Select-Object -ExpandProperty Caption
$os_kernel=Get-CimInstance -ClassName Win32_OperatingSystem | Select-Object -ExpandProperty version
$cpu_model=Get-CimInstance Win32_processor | Select-Object -ExpandProperty name -Unique
$cpu_num=(Get-CimInstance Win32_processor | Select-Object -ExpandProperty name).count
$cpu_core=(Get-CimInstance Win32_processor | Select-Object -ExpandProperty processorid).count
$mem_total=(get-CimInstance -class Win32_PhysicalMemory   -namespace "root\cimv2").Capacity
$disk_total=Get-CimInstance -class win32_logicaldisk | Measure-Object -Sum size | Select-Object -ExpandProperty sum
$product_id=Get-CimInstance -ClassName Win32_OperatingSystem | Select-Object -ExpandProperty SerialNumber

$os_info = @"
{"os_sys":"Windows","os_version":"$os_version","os_kernel":"$os_kernel","cpu_model":"$cpu_model","cpu_num":"$cpu_num","cpu_core":"$cpu_core","mem_total":"$mem_total","disk_total":"$disk_total","product_id":"$product_id"}
"@

$os_info
```

