---
title: zabbix-agent安装
date: 2017-04-18 16:15:37
tags:
    - zabbix
    - zabbix agent
categories: zabbix
copyright: true
---
监控路由交换等,由于本身不能安装软件,所以通常使用SNMP协议进行数据的采集.
但是,对于系统的监控最好还是采用agent进行采集,方便.
<!--more-->

>Zabbix3.2 CentOS7 x64

#### Yum源配置
```
#centos7
rpm -ivh http://repo.zabbix.com/zabbix/3.2/rhel/7/x86_64/zabbix-release-3.2-1.el7.noarch.rpm

#centos6 
rpm -ivh http://repo.zabbix.com/zabbix/3.2/rhel/6/x86_64/zabbix-release-3.2-1.el6.noarch.rpm
```
#### zabbix_agent安装配置
##### centos
```
yum install zabbix-agent -y
vim /etc/zabbix/zabbix_agentd.conf 
Server= 服务端IP #用于被动模式
ServerActive= 服务端IP#用于主动模式

Hostname=本机Ip #不要用127.0.0.1 
#或者 弄个唯一名字，这里的名字在服务端添加主机的时候要用！建议使用IP
主动模式失败，  `Received empty response from Zabbix Agent at [10.1.93.252]. Assuming that ...` 不知为何，


systemctl start zabbix-agent#启动zabbix agent
```
主动和被动的区别。以agent端为出发，agent主动发送就是主动，被动接受就是被动。

#####  Ubuntu amd64

```
#下载软件包
wget http://repo.zabbix.com/zabbix/3.2/ubuntu/pool/main/z/zabbix/zabbix-agent_3.2.4-1+trusty_amd64.deb

#然后安装
dpkg -i zabbix....deb

##再修改配置，同centos
```
##### windows agent安装

下载agent包 http://www.zabbix.com/download


拷贝zabbix_agents_3.2.0.win\bin\win64下所有文件（非文件夹）到C:\Program Files\zabbix下
32位到 C:\Program Files x86\zabbix下


新建文件C:\Program Files\zabbix\zabbix_agent.conf
复制代码，主要修改地方同linux。不要写错，否则服务启动不起来s
```
第一种：
Server= 服务端IP #用于被动模式
ServerActive= 服务端IP#用于主动模式
Hostname=本机Ip #不要用127.0.0.1 

如果有特殊需求

第二种：

# This is a configuration file for Zabbix agent service (Windows)
# To get more information about Zabbix, visit http://www.zabbix.com

############ GENERAL PARAMETERS #################

### Option: LogType
#   Specifies where log messages are written to:
#       system  - Windows event log
#       file    - file specified with LogFile parameter
#       console - standard output
#
# Mandatory: no
# Default:
# LogType=file

### Option: LogFile
#   Log file name for LogType 'file' parameter.
#
# Mandatory: no
# Default:
# LogFile=

LogFile=c:\zabbix_agentd.log

### Option: LogFileSize
#   Maximum size of log file in MB.
#   0 - disable automatic log rotation.
#
# Mandatory: no
# Range: 0-1024
# Default:
# LogFileSize=1

### Option: DebugLevel
#   Specifies debug level:
#   0 - basic information about starting and stopping of Zabbix processes
#   1 - critical information
#   2 - error information
#   3 - warnings
#   4 - for debugging (produces lots of information)
#   5 - extended debugging (produces even more information)
#
# Mandatory: no
# Range: 0-5
# Default:
# DebugLevel=3

### Option: SourceIP
#   Source IP address for outgoing connections.
#
# Mandatory: no
# Default:
# SourceIP=

### Option: EnableRemoteCommands
#   Whether remote commands from Zabbix server are allowed.
#   0 - not allowed
#   1 - allowed
#
# Mandatory: no
# Default:
# EnableRemoteCommands=0

### Option: LogRemoteCommands
#   Enable logging of executed shell commands as warnings.
#   0 - disabled
#   1 - enabled
#
# Mandatory: no
# Default:
# LogRemoteCommands=0

##### Passive checks related

### Option: Server
#   List of comma delimited IP addresses (or hostnames) of Zabbix servers.
#   Incoming connections will be accepted only from the hosts listed here.
#   If IPv6 support is enabled then '127.0.0.1', '::127.0.0.1', '::ffff:127.0.0.1' are treated equally.
#
# Mandatory: no
# Default:
# Server=

Server=127.0.0.1

### Option: ListenPort
#   Agent will listen on this port for connections from the server.
#
# Mandatory: no
# Range: 1024-32767
# Default:
# ListenPort=10050

### Option: ListenIP
#       List of comma delimited IP addresses that the agent should listen on.
#       First IP address is sent to Zabbix server if connecting to it to retrieve list of active checks.
#
# Mandatory: no
# Default:
# ListenIP=0.0.0.0

### Option: StartAgents
#   Number of pre-forked instances of zabbix_agentd that process passive checks.
#   If set to 0, disables passive checks and the agent will not listen on any TCP port.
#
# Mandatory: no
# Range: 0-100
# Default:
# StartAgents=3

##### Active checks related

### Option: ServerActive
#   List of comma delimited IP:port (or hostname:port) pairs of Zabbix servers for active checks.
#   If port is not specified, default port is used.
#   IPv6 addresses must be enclosed in square brackets if port for that host is specified.
#   If port is not specified, square brackets for IPv6 addresses are optional.
#   If this parameter is not specified, active checks are disabled.
#   Example: ServerActive=127.0.0.1:20051,zabbix.domain,[::1]:30051,::1,[12fc::1]
#
# Mandatory: no
# Default:
# ServerActive=

ServerActive=127.0.0.1

### Option: Hostname
#   Unique, case sensitive hostname.
#   Required for active checks and must match hostname as configured on the server.
#   Value is acquired from HostnameItem if undefined.
#
# Mandatory: no
# Default:
# Hostname=

Hostname=Windows host

### Option: HostnameItem
#   Item used for generating Hostname if it is undefined. Ignored if Hostname is defined.
#   Does not support UserParameters or aliases.
#
# Mandatory: no
# Default:
# HostnameItem=system.hostname

### Option: HostMetadata
#   Optional parameter that defines host metadata.
#   Host metadata is used at host auto-registration process.
#   An agent will issue an error and not start if the value is over limit of 255 characters.
#   If not defined, value will be acquired from HostMetadataItem.
#
# Mandatory: no
# Range: 0-255 characters
# Default:
# HostMetadata=
HostMetadata=wonders_windows_1

### Option: HostMetadataItem
#   Optional parameter that defines an item used for getting host metadata.
#   Host metadata is used at host auto-registration process.
#   During an auto-registration request an agent will log a warning message if
#   the value returned by specified item is over limit of 255 characters.
#   This option is only used when HostMetadata is not defined.
#
# Mandatory: no
# Default:
# HostMetadataItem=

### Option: RefreshActiveChecks
#   How often list of active checks is refreshed, in seconds.
#
# Mandatory: no
# Range: 60-3600
# Default:
# RefreshActiveChecks=120

### Option: BufferSend
#   Do not keep data longer than N seconds in buffer.
#
# Mandatory: no
# Range: 1-3600
# Default:
# BufferSend=5

### Option: BufferSize
#   Maximum number of values in a memory buffer. The agent will send
#   all collected data to Zabbix server or Proxy if the buffer is full.
#
# Mandatory: no
# Range: 2-65535
# Default:
# BufferSize=100

### Option: MaxLinesPerSecond
#   Maximum number of new lines the agent will send per second to Zabbix Server
#   or Proxy processing 'log', 'logrt' and 'eventlog' active checks.
#   The provided value will be overridden by the parameter 'maxlines',
#   provided in 'log', 'logrt' or 'eventlog' item keys.
#
# Mandatory: no
# Range: 1-1000
# Default:
# MaxLinesPerSecond=20

############ ADVANCED PARAMETERS #################

### Option: Alias
#   Sets an alias for an item key. It can be used to substitute long and complex item key with a smaller and simpler one.
#   Multiple Alias parameters may be present. Multiple parameters with the same Alias key are not allowed.
#   Different Alias keys may reference the same item key.
#   For example, to retrieve paging file usage in percents from the server:
#   Alias=pg_usage:perf_counter[\Paging File(_Total)\% Usage]
#   Now shorthand key pg_usage may be used to retrieve data.
#   Aliases can be used in HostMetadataItem but not in HostnameItem or PerfCounter parameters.
#
# Mandatory: no
# Range:
# Default:

### Option: Timeout
#   Spend no more than Timeout seconds on processing.
#
# Mandatory: no
# Range: 1-30
# Default:
# Timeout=3

### Option: PerfCounter
#   Syntax: <parameter_name>,"<perf_counter_path>",<period>
#   Defines new parameter <parameter_name> which is an average value for system performance counter <perf_counter_path> for the specified time period <period> (in seconds).
#   For example, if you wish to receive average number of processor interrupts per second for last minute, you can define new parameter "interrupts" as following:
#   PerfCounter = interrupts,"\Processor(0)\Interrupts/sec",60
#   Please note double quotes around performance counter path.
#   Samples for calculating average value will be taken every second.
#   You may run "typeperf -qx" to get list of all performance counters available in Windows.
#
# Mandatory: no
# Range:
# Default:

### Option: Include
#   You may include individual files in the configuration file.
#
# Mandatory: no
# Default:
# Include=

# Include=c:\zabbix\zabbix_agentd.userparams.conf
# Include=c:\zabbix\zabbix_agentd.conf.d\
# Include=c:\zabbix\zabbix_agentd.conf.d\*.conf

####### USER-DEFINED MONITORED PARAMETERS #######

### Option: UnsafeUserParameters
#   Allow all characters to be passed in arguments to user-defined parameters.
#   The following characters are not allowed:
#   \ ' " ` * ? [ ] { } ~ $ ! & ; ( ) < > | # @
#   Additionally, newline characters are not allowed.
#   0 - do not allow
#   1 - allow
#
# Mandatory: no
# Range: 0-1
# Default:
# UnsafeUserParameters=0

### Option: UserParameter
#   User-defined parameter to monitor. There can be several user-defined parameters.
#   Format: UserParameter=<key>,<shell command>
#
# Mandatory: no
# Default:
# UserParameter=

####### TLS-RELATED PARAMETERS #######

### Option: TLSConnect
#   How the agent should connect to server or proxy. Used for active checks.
#   Only one value can be specified:
#       unencrypted - connect without encryption
#       psk         - connect using TLS and a pre-shared key
#       cert        - connect using TLS and a certificate
#
# Mandatory: yes, if TLS certificate or PSK parameters are defined (even for 'unencrypted' connection)
# Default:
# TLSConnect=unencrypted

### Option: TLSAccept
#   What incoming connections to accept.
#   Multiple values can be specified, separated by comma:
#       unencrypted - accept connections without encryption
#       psk         - accept connections secured with TLS and a pre-shared key
#       cert        - accept connections secured with TLS and a certificate
#
# Mandatory: yes, if TLS certificate or PSK parameters are defined (even for 'unencrypted' connection)
# Default:
# TLSAccept=unencrypted

### Option: TLSCAFile
#   Full pathname of a file containing the top-level CA(s) certificates for
#   peer certificate verification.
#
# Mandatory: no
# Default:
# TLSCAFile=

### Option: TLSCRLFile
#   Full pathname of a file containing revoked certificates.
#
# Mandatory: no
# Default:
# TLSCRLFile=

### Option: TLSServerCertIssuer
#      Allowed server certificate issuer.
#
# Mandatory: no
# Default:
# TLSServerCertIssuer=

### Option: TLSServerCertSubject
#      Allowed server certificate subject.
#
# Mandatory: no
# Default:
# TLSServerCertSubject=

### Option: TLSCertFile
#   Full pathname of a file containing the agent certificate or certificate chain.
#
# Mandatory: no
# Default:
# TLSCertFile=

### Option: TLSKeyFile
#   Full pathname of a file containing the agent private key.
#
# Mandatory: no
# Default:
# TLSKeyFile=

### Option: TLSPSKIdentity
#   Unique, case sensitive string used to identify the pre-shared key.
#
# Mandatory: no
# Default:
# TLSPSKIdentity=

### Option: TLSPSKFile
#   Full pathname of a file containing the pre-shared key.
#
# Mandatory: no
# Default:
# TLSPSKFile=
```

#### 注册zabbix服务
管理员身份运行cmd

切换到zabbix目录下
```
zabbix_agentd.exe -c .C:\Program Files\zabbix\zabbix_agentd.win.conf -i

 -c  表示配置文件路径 -i  表示安装 
删除的话： -d  表示卸载
zabbix_agentd.exe  -s #启动
或者去服务里面手动启动
  ```
完
___
