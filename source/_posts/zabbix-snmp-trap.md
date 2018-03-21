---
title: zabbix snmp trap
date: 2018-03-18 15:08:24
tags: zabbix
categories: zabbix
copyright: true
---
了解snmp trap之前,先了解snmp以及MIB.
snmp trap 是 SNMP 的一部分,简单来说就是被管理设备主动发送消息给管理设备.
<!--more-->

zabbix开启 snmp trap 比较简单,官方提供了perl脚本.
[官方文档](https://zabbix.org/wiki/Start_with_SNMP_traps_in_Zabbix)

#### 获取脚本
前置条件,也可源码安装,按个人喜好
`yum install -y net-snmp-utils net-snmp-perl net-snmp`

下载源码
[下载地址](https://www.zabbix.com/download_sources)
解压
`tar -zxvf zabbix-VERSION.tar.gz`
复制脚本并给予权限
```
cp ./zabbix-VERSION/misc/snmptrap/zabbix_trap_receiver.pl /usr/bin
chmod +x /usr/bin/zabbix_trap_receiver.pl
```

#### 配置trap
`vi /etc/snmp/snmptrapd.conf`
修改配置如下
```
authCommunity execute public
perl do "/usr/bin/zabbix_trap_receiver.pl";
```

开启server的trapper功能
`vi /etc/zabbix/zabbix_server.conf`
修改配置如下
```
StartSNMPTrapper=1
SNMPTrapperFile=/tmp/zabbix_traps.tmp 
#此路径和脚本中的要一致,否则server无法读取发送过来的信息
```
重启服务
`/etc/init.d/zabbix-server restart`

#### 添加mib库
默认情况下snmp内置了一下mib库文件`/usr/share/snmp/mibs`
如果有需要可添加第三方mib文件到此目录下.并新建`/etc/snmp/snmp.conf`
添加mib库信息,格式如下
```
#mibs +库名
mibs +JUNIPER-MIB:JUNIPER-FABRIC-CHASSIS:BGP4-MIB
```

#### 启动
centos7
`systemctl enable snmptrapd`
`systemctl restart snmptrapd.service`
小记:`snmptrapd -C -c /etc/snmp/snmptrapd.conf` 指定配置文件启动

#### web使用
现在就可以在web端进行操作了.
添加开启了snmptrap的主机即可

##### 创建监控项目
可以针对单个主机,也可创建模板然后关联主机.

1. 名称: 自定义
2. 类型: snmp trap
3. key: snmptrap.fallback(表示所有没有被正则匹配到的,都会到这个下面)
```
官网有明显的解释以及案例
比如新建个监控项
     名称为 :Name: Unmatched SNMP trap received from {HOST.NAME}
     key为 : snmptrap.fallback
那么表达式就可以写成 类似 {snmptrap.fallback.nodata(300)}=0
表示五分钟没有数据就触发告警
```
4. 信息类型: 日志
主要就这几点.

重点讲下这个`key`

除了`snmptrap.fallback`,还有`snmptrap[regexp]`这个支持正则匹配,
由于监控的是日志,获取的也是所有的日志信息,所有要取想要的值就要进行正则匹配.
给出官网例子,便于理解
```
Example 1
Key: snmptrap["SNMPv2-MIB::coldStart"]
Instead of OID (numeric or textual), you can use any word / phrase from a trap text:

Key: snmptrap["No route to host"]
in this case, Zabbix catches all SNMP traps from a corresponding address that contains "No route to host".

Note that you can create item for each trap (example above) or a single item for multiple traps.
-------------------------
Example 2
Key: snmptrap["cpqRackPowerSubsystem(NotRedundant|LineVoltageProblem|OverloadCondition)"]

in this case, Zabbix catches all SNMP traps from a corresponding address that contains "cpqRackPowerSubsystemNotRedundant" or "cpqRackPowerSubsystemLineVoltageProblem" or "cpqRackPowerSubsystemOverloadCondition".
```
其实就是说,可以进行正则匹配,匹配到,server就会捕获相应的snmp trap信息.
感觉上就像是个日志监控

再给个官网例子
```
  Name: Link status trap for {#SNMPVALUE}
  Type: SNMP trap
  Key: snmptrap["(IF-MIB::linkDown|IF-MIB::linkUp)(.|[[:space:]])*{#SNMPVALUE}"]
  Type of information: Log
```
可以看出,这个key就是去匹配链路的up/down的.也给出了触发器的表达式
```
 Name: Interface {#SNMPVALUE} on {HOST.NAME} is down
  Expression: {Template SNMP Interfaces:snmptrap["(IF-MIB::linkDown|IF-MIB::linkUp)(.|[[:space:]])*{#SNMPVALUE}"].str(linkDown)}=1
```
可以看出,匹配到了`linkDown` 就返回1 然后触发告警.`str返回两个结果,1和0 1代匹配,0表示未匹配`

整个流程大体就是这样,具体的监控项还是要熟悉mib,然后来定义.

---
