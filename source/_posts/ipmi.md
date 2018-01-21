---
title: ipmi
date: 2018-01-21 17:59:13
tags:
    - ipmi
categories: linux
copyright: true
---
IPMI(Intelligent Platform Management Interface) 是独立于物理服务器CPU，电源，存储等独立工作的管理工具。其需要进入BISO先行设置。
IPMI可监控服务器的物理状态，如温度、电压、电扇、电源等，以及一些远程管控等
默认端口是623，UDP协议
<!--more-->

#### 安装
centos环境下
`yum -y install ipmitool`
也可以下载`ipmitool-version.tar.gz` 解压后`./configure && make && make install`
`service ipmi start`
执行 内核加载,另：虚机不存在
```bash
[root@localhost ~]# modprobe ipmi_watchdog
[root@localhost ~]# modprobe ipmi_poweroff
[root@localhost ~]# modprobe ipmi_devintf
[root@localhost ~]# modprobe ipmi_si
[root@localhost ~]# modprobe ipmi_msghandler
```

查资料显示服务器 `第一块网卡` `第一个口` 接`交换机`，默认这个口集成了IPMI，目前dell如此。
其他厂商可能会有变，使用前请确认，并去`BIOS`开启。网口启动与否并不影响使用。

#### 使用

远程获取服务器监控信息时，需要加上远程服务器的地址：
`ipmitool -I lanplus -H ipaddress -U username -P password command`
`-H`表示后面跟的是服务器的地址
`-U`表示后面跟着用户名
`-P`表示后面跟着用户密码
`command`命令

#### 参数
```
$ ipmitool -h

usage: ipmitool [options...] <command>

       -h             This help
       -V             Show version information
       -v             Verbose (can use multiple times)
       -c             Display output in comma separated format
       -d N           Specify a /dev/ipmiN device to use (default=0)
       -I intf        Interface to use
       -H hostname    Remote host name for LAN interface
       -p port        Remote RMCP port [default=623]
       -U username    Remote session username
       -f file        Read remote session password from file
       -z size        Change Size of Communication Channel (OEM)
       -S sdr         Use local file for remote SDR cache
       -D tty:b[:s]   Specify the serial device, baud rate to use
                      and, optionally, specify that interface is the system one
       -a             Prompt for remote password
       -Y             Prompt for the Kg key for IPMIv2 authentication
       -e char        Set SOL escape character
       -C ciphersuite Cipher suite to be used by lanplus interface
       -k key         Use Kg key for IPMIv2 authentication
       -y hex_key     Use hexadecimal-encoded Kg key for IPMIv2 authentication
       -L level       Remote session privilege level [default=ADMINISTRATOR]
                      Append a '+' to use name/privilege lookup in RAKP1
       -A authtype    Force use of auth type NONE, PASSWORD, MD2, MD5 or OEM
       -P password    Remote session password
       -E             Read password from IPMI_PASSWORD environment variable
       -K             Read kgkey from IPMI_KGKEY environment variable
       -m address     Set local IPMB address
       -b channel     Set destination channel for bridged request
       -t address     Bridge request to remote target address
       -B channel     Set transit channel for bridged request (dual bridge)
       -T address     Set transit address for bridge request (dual bridge)
       -l lun         Set destination lun for raw commands
       -o oemtype     Setup for OEM (use 'list' to see available OEM types)
       -O seloem      Use file for OEM SEL event descriptions
       -N seconds     Specify timeout for lan [default=2] / lanplus [default=1] interface
       -R retry       Set the number of retries for lan/lanplus interface [default=4]

Interfaces:
        open          Linux OpenIPMI Interface [default]
        imb           Intel IMB Interface 
        lan           IPMI v1.5 LAN Interface 
        lanplus       IPMI v2.0 RMCP+ LAN Interface 
        serial-terminal  Serial Interface, Terminal Mode 
        serial-basic  Serial Interface, Basic Mode 

Commands:
        raw           Send a RAW IPMI request and print response
        i2c           Send an I2C Master Write-Read command and print response
        spd           Print SPD info from remote I2C device
        lan           Configure LAN Channels
        chassis       Get chassis status and set power state
        power         Shortcut to chassis power commands
        event         Send pre-defined events to MC
        mc            Management Controller status and global enables
        sdr           Print Sensor Data Repository entries and readings
        sensor        Print detailed sensor information
        fru           Print built-in FRU and scan SDR for FRU locators
        gendev        Read/Write Device associated with Generic Device locators sdr
        sel           Print System Event Log (SEL)
        pef           Configure Platform Event Filtering (PEF)
        sol           Configure and connect IPMIv2.0 Serial-over-LAN
        tsol          Configure and connect with Tyan IPMIv1.5 Serial-over-LAN
        isol          Configure IPMIv1.5 Serial-over-LAN
        user          Configure Management Controller users
        channel       Configure Management Controller channels
        session       Print session information
        dcmi          Data Center Management Interface
        sunoem        OEM Commands for Sun servers
        kontronoem    OEM Commands for Kontron devices
        picmg         Run a PICMG/ATCA extended cmd
        fwum          Update IPMC using Kontron OEM Firmware Update Manager
        firewall      Configure Firmware Firewall
        delloem       OEM Commands for Dell systems
        shell         Launch interactive IPMI shell
        exec          Run list of commands from file
        set           Set runtime variable for shell and exec
        hpm           Update HPM components using PICMG HPM.1 file
        ekanalyzer    run FRU-Ekeying analyzer using FRU files
        ime           Update Intel Manageability Engine Firmware
```

#### 常用
电源操作
```bash 
ipmitool -I lanplus -H 192.168.1.231 -U username -P password chassis power off
ipmitool -I lanplus -H 192.168.1.231 -U username -P password chassis power reset
ipmitool -I lanplus -H 192.168.1.231 -U username -P password chassis power on
ipmitool -I lanplus -H 192.168.1.231 -U username -P password chassis power status
```

版本查看
`ipmitool -V` 
设备信息
`ipmitool -I lanplus -H 192.168.1.231 -U lenovo -P lenovo chassis status`


|命令选项  |  描述 |
|---|---|
|ipmitool -I lanplus -H myserver.example.com -P mypass chassis power on  |打开服务器的电源|
|ipmitool -I lanplus -H myserver.example.com -P mypass chassis power off |关闭服务器的电源|
|ipmitool -I lanplus -H myserver.example.com -P mypass chassis status    |检查服务器状态|
|ipmitool -I lanplus -H myserver.example.com -P mypass chassis power cycle   |关闭服务器的电源，然后重新打开|
|ipmitool -I lanplus -H myserver.example.com -P mypass sol activate  激活 SOL |系统控制台|
|ipmitool -I lanplus -H myserver.example.com -P mypass sol deactivate    |取消激活 SOL 系统控制台|
|ipmitool -I lanplus -H myserver.example.com -P mypass sel list|  返回错误日志|
|ipmitool -I lanplus -H myserver.example.com -P mypass sdr list  |列示所有传感器的状态|
|ipmitool -I lanplus -H myserver.example.com -P mypass sol set retry-interval value  |设置缺省重试时间间隔值（以毫秒计）|
|ipmitool -I lanplus -H myserver.example.com -P mypass fru print| 打印 FRU 信息|
|ipmitool -I lanplus -H myserver.example.com -P mypass user list |列示 IPMI 用户|


#### 详细
查询SDR(Print Sensor Data Repository entries and readings)传感器数据

```
[root@centos-01 ~]# ipmitool -I lanplus -H 192.168.1.231 -U lenovo -P lenovo sdr list
CPU1 Status      | 0x00              | ok
CPU2 Status      | 0x00              | ok
CPU1 Temp        | 42 degrees C      | ok
CPU2 Temp        | 57 degrees C      | ok
DIMM Zone1 Temp  | 33 degrees C      | ok
DIMM Zone2 Temp  | 30 degrees C      | ok
DIMM Zone3 Temp  | 28 degrees C      | ok
DIMM Zone4 Temp  | 31 degrees C      | ok
PCH Temp         | 48 degrees C      | ok
PSU Inlet Temp   | 31 degrees C      | ok
PCI Zone1 Temp   | 36 degrees C      | ok
PCI Zone2 Temp   | 32 degrees C      | ok
Inlet Amb Temp   | 17 degrees C      | ok
3.3V Standby     | 3.23 Volts        | ok
5V Standby       | 5.00 Volts        | ok
3.3V             | 3.23 Volts        | ok
5V               | 5.00 Volts        | ok
12V              | 11.96 Volts       | ok
3V Battery       | 3.12 Volts        | ok
12V Standby      | 11.70 Volts       | ok
Power Reading    | 228 Watts/hour    | ok
PSU1 Status      | 0x00              | ok
PSU2 Status      | 0x00              | ok
Power Unit       | 0x00              | ok
12V Ch A Fault   | 0x00              | ok
12V Ch B Fault   | 0x00              | ok
12V Ch C Fault   | 0x00              | ok
12V Ch D Fault   | 0x00              | ok
12V Ch E Fault   | 0x00              | ok
12V Ch F Fault   | 0x00              | ok
Watchdog         | 0x00              | ok
Memory ECC       | Not Readable      | ns
BIOS Event       | Not Readable      | ns
PCI Riser1 Temp  | 32 degrees C      | ok
PCI Riser2 Temp  | no reading        | ns
FAN1             | 3840 RPM          | ok
FAN2             | 3840 RPM          | ok
FAN3             | 3780 RPM          | ok
FAN4             | 3840 RPM          | ok
FAN5             | 3840 RPM          | ok
FAN6             | 3780 RPM          | ok
Sys Pwr Monitor  | 0x00              | ok
[root@centos-01 ~]# 

```

-v 查询详细

```bash
[root@centos-01 ~]# ipmitool -I lanplus -H 192.168.1.231 -U lenovo -P lenovo -v sdr list
Running Get PICMG Properties my_addr 0x20, transit 0, target 0x20
Error response 0xc1 from Get PICMG Properities
Running Get VSO Capabilities my_addr 0x20, transit 0, target 0x20
Invalid completion code received: Invalid command
Discovered IPMB address 0x0
Sensor ID              : CPU1 Status (0x1)
 Entity ID             : 3.1 (Processor)
 Sensor Type (Discrete): Processor (0x07)
 Sensor Reading        : 0h
 Event Message Control : Per-threshold
 States Asserted       : Processor
                         [Presence detected]
 Assertion Events      : Processor
                         [Presence detected]
 Assertions Enabled    : Processor
                         [Thermal Trip]
 OEM                   : 0

Sensor ID              : CPU2 Status (0x2)
 Entity ID             : 3.2 (Processor)
 Sensor Type (Discrete): Processor (0x07)
 Sensor Reading        : 0h
 Event Message Control : Per-threshold
 States Asserted       : Processor
                         [Presence detected]
 Assertion Events      : Processor
                         [Presence detected]
 Assertions Enabled    : Processor
                         [Thermal Trip]
 OEM                   : 0

Sensor ID              : CPU1 Temp (0x9)
 Entity ID             : 3.1 (Processor)
 Sensor Type (Threshold)  : Temperature (0x01)
 Sensor Reading        : 46 (+/- 0) degrees C
 Status                : ok
 Nominal Reading       : 35.000
 Upper critical        : 75.000
 Upper non-critical    : 70.000
 Positive Hysteresis   : Unspecified
 Negative Hysteresis   : Unspecified
 Minimum sensor range  : Unspecified
 Maximum sensor range  : Unspecified
 Event Message Control : Per-threshold
 Readable Thresholds   : unc ucr 
 Settable Thresholds   : 
 Threshold Read Mask   : unc ucr 
 Assertion Events      : 
 Assertions Enabled    : unc+ ucr+ 
 Deassertions Enabled  : unc+ ucr+ 


```

这里要注意`Sensor ID` 可根据其查询相关信息

```bash
[root@centos-01 ~]# ipmitool -I lanplus -H 192.168.1.231 -U lenovo -P lenovo sdr get "CPU1 Status"
Sensor ID              : CPU1 Status (0x1)
 Entity ID             : 3.1 (Processor)
 Sensor Type (Discrete): Processor (0x07)
 Sensor Reading        : 0h
 Event Message Control : Per-threshold
 States Asserted       : Processor
                         [Presence detected]
 Assertion Events      : Processor
                         [Presence detected]
 Assertions Enabled    : Processor
                         [Thermal Trip]
 OEM                   : 0
```

另外注意`Entity ID` ，也可根据查询相关信息

```bash
[root@centos-01 ~]# ipmitool -I lanplus -H 192.168.1.231 -U lenovo -P lenovo sdr entity 3.1
CPU1 Status      | 01h | ok  |  3.1 | Presence detected
CPU1 Temp        | 09h | ok  |  3.1 | 43 degrees C
```

这些命令可以通过帮助进行查看

```
[root@centos-01 ~]# ipmitool -I lanplus -H 192.168.1.231 -U lenovo -P lenovo sdr help
usage: sdr <command> [options]
               list | elist [option]
                     all           All SDR Records
                     full          Full Sensor Record
                     compact       Compact Sensor Record
                     event         Event-Only Sensor Record
                     mcloc         Management Controller Locator Record
                     fru           FRU Locator Record
                     generic       Generic Device Locator Record

               type [option]
                     <Sensor_Type> Retrieve the state of specified sensor.
                                   Sensor_Type can be specified either as
                                   a string or a hex value.
                     list          Get a list of available sensor types

               get <Sensor_ID>
                     Retrieve state of the first sensor matched by Sensor_ID

               info
                     Display information about the repository itself

               entity <Entity_ID>[.<Instance_ID>]
                     Display all sensors associated with an entity

               dump <file>
                     Dump raw SDR data to a file

               fill <option>
                     sensors       Creates the SDR repository for the current
                                   configuration
                     nosat         Creates the SDR repository for the current
                                   configuration, without satellite scan
                     file <file>   Load SDR repository from a file
                     range <range> Load SDR repository from a provided list
                                   or range. Use ',' for list or '-' for
                                   range, eg. 0x28,0x32,0x40-0x44
[root@centos-01 ~]# 
```

---