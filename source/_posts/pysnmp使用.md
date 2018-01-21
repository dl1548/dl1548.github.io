---
title: pysnmp使用
date: 2018-01-01 22:15:08
tags: 
    - pysnmp
    - python
categories: python
copyright: true
---
`pysnmp` ,python的一个库文件,可通过它获取设备snmp信息,从而提取想要的资料.
建议,使用前,先了解下snmp,MIB,oid等 [pysnmp官网](http://snmplabs.com/pysnmp/contents.html)
<!--more-->

安装使用 `pip` 就可以了.

#### 官方例子
##### 获取信息
SNMP GET
官方例子
```python
"""
SNMPv1
++++++

Send SNMP GET request using the following options:

  * with SNMPv1, community 'public'
  * over IPv4/UDP
  * to an Agent at demo.snmplabs.com:161
  * for two instances of SNMPv2-MIB::sysDescr.0 MIB object,

Functionally similar to:

| $ snmpget -v1 -c public demo.snmplabs.com SNMPv2-MIB::sysDescr.0

"""#
from pysnmp.hlapi import *

errorIndication, errorStatus, errorIndex, varBinds = next(
    getCmd(SnmpEngine(),
           CommunityData('public', mpModel=0),
           UdpTransportTarget(('demo.snmplabs.com', 161)),
           ContextData(),
           ObjectType(ObjectIdentity('SNMPv2-MIB', 'sysDescr', 0)))
)

if errorIndication:
    print(errorIndication)
elif errorStatus:
    print('%s at %s' % (errorStatus.prettyPrint(),
                        errorIndex and varBinds[int(errorIndex) - 1][0] or '?'))
else:
    for varBind in varBinds:
        print(' = '.join([x.prettyPrint() for x in varBind]))
```
执行后的结果
`SNMPv2-MIB::sysDescr."0" = SunOS zeus.snmplabs.com 4.1.3_U1 1 sun4m`

##### 方法解释
大概说下作用,如开头所说,建议先了解下snmp,MIB,oid等

1. `getCmd()` 通过此方法来执行SNMP命令.
返回由`errorIndication`, `errorStatus`, `errorIndex`, `varBinds`组成的一个tuple
一共有四个方法`bulkCmd` `getCmd` `nextCmd` `setCmd`都有各自的用法.
2. `SnmpEngine()` 创建了个SNMP引擎,用来保障后续的操作.
3. `CommunityData()` 实例化SNMP协议以及community
SNMP协议分为`v1` `v2c` `v3` v3暂时不管.`CommunityData()`需要两个参数用来实例化
其中 `0`代表`v1`, `1`代表`v2c`  v3是另外的方法`UsmUserData`
4. `UdpTransportTarget` 其实还有个 `Udp6TransportTarget` 针对IPv6的
它用来设置传输目标
5. `ContextData()`,由于SNMP的消息头是`v3`的,这个类是用来初始snmp上下文的,
6. `ObjectType` 理解为一个容器对象,使`ObjectIdentity`和SNMP信息更像是`key:value`
[MIB库ASN1](http://mibs.snmplabs.com/asn1/) 里都是MIB信息,格式类似
```
sysUpTime OBJECT-TYPE
    SYNTAX      TimeTicks
    MAX-ACCESS  read-only
    STATUS      current
    DESCRIPTION
            "The time (in hundredths of a second) since
            the network management portion of the system
            was last re-initialized."
    ::= { system 3 }
```
这些信息由`ObjectIdentity`转换为`1.3.6.1....`,然后再由`ObjectType` 将其和值对应.
使结果更为友好,使用`prettyPrint()`打印出
7. `ObjectIdentity` MIB对象标识,格式化处理MIB对象为OID,反之亦然.所以也可直接输出OID

以上步骤基本就可以按需求提出SNMP信息了.

#### 简便方法
使用模块`pysnmp.entity.rfc3413.oneliner.cmdgen`

```python
from pysnmp.entity.rfc3413.oneliner import cmdgen

def info(self,oid,ip,commu):
        cmdGen = cmdgen.CommandGenerator()
        errorIndication, errorStatus, errorIndex, varBindTable = cmdGen.nextCmd(
            cmdgen.CommunityData(commu),
            cmdgen.UdpTransportTarget((ip, 161)),
            oid,
        )

        if errorIndication:
            print(errorIndication)
        else:
            if errorStatus:
                print('%s at %s' % (
                    errorStatus.prettyPrint(),
                    errorIndex and varBindTable[-1][int(errorIndex)-1][0] or '?'
                    )
                )
            else:
                for varBindTableRow in varBindTable:
                    for name, val in varBindTableRow:
                        return ('%s = %s' % (name.prettyPrint(), val.prettyPrint()))
```

#### 简单例子

```python
#!/usr/bin/env python
#coding: utf-8
#author: lizili

#from pysnmp.hlapi import *
from pysnmp.entity.rfc3413.oneliner import cmdgen
import json

class GetSnmp():
    #oid列表
    def make_list(self,*oid):
        oid_list = []
        for o in oid:
            oid_list.append(o)
        return oid_list 

    #获取snmp信息
    def info(self,oid,ip,commu):
        cmdGen = cmdgen.CommandGenerator()
        errorIndication, errorStatus, errorIndex, varBindTable = cmdGen.nextCmd(
            cmdgen.CommunityData(commu),
            cmdgen.UdpTransportTarget((ip, 161)),
            oid,
        )
        if errorIndication:
            print(errorIndication)
        else:
            if errorStatus:
                print('%s at %s' % (
                    errorStatus.prettyPrint(),
                    errorIndex and varBindTable[-1][int(errorIndex)-1][0] or '?'
                    )
                )
            else:
                var_dict={}
                for varBindTableRow in varBindTable:
                    for name, val in varBindTableRow:
                        var_dict[name.prettyPrint()]=str(val.prettyPrint())
                return var_dict

    #循环oid表，提取整理信息
    def get_info(self,oid,ip,commu='public'):
        info_dict={}
        for o in oid:
            info = self.info(o,ip,commu)
            info_dict[o]=info
        info_json=json.dumps(info_dict,indent=4)
        return info_json

if __name__ == "__main__":
    sysName  = "1.3.6.1.2.1.1.5"
    sysDescr = "1.3.6.1.2.1.1.1"
    ifNumber = "1.3.6.1.2.1.2.1"
    ifDescr = "1.3.6.1.2.1.2.2.1.2"
    ifInOctet = "1.3.6.1.2.1.2.2.1.10"
    ifOutOctet = "1.3.6.1.2.1.2.2.1.16"
    ifInUcastPkts = "1.3.6.1.2.1.2.2.1.11"
    ifOutUcastPkts = "1.3.6.1.2.1.2.2.1.17"
    ipNetToMediaPhysAddress = "1.3.6.1.2.1.4.22.1.2"
    ipOperStatus = "1.3.6.1.2.1.2.2.1.8"
    #实例化类
    test = GetSnmp()

    #生成list
    oid_list = test.make_list(
    #    sysName,
    #    sysDescr,
    #    ifNumber,
    #    ifDescr,
    #    ifInOctet,
    #    ifOutOctet,
    #    ifInUcastPkts,
    #    ifOutUcastPkts,
        ipNetToMediaPhysAddress,
        ipOperStatus,
        )
    #输出信息 
    info = test.get_info(oid_list,"192.168.1.235")
    print info

```