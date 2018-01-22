---
title: ldap-zbx单点登录
date: 2018-01-21 20:31:43
tags: 
    - ldap
    - python
    - zabbix
categories: 
    - python
copyright: true
---

LDAP认证系统可再多系统间实现单点登录，免去多账户的烦恼。

<!--more-->

由于zabbix要实现单点登录，因此而学习了下。
zbx本身支持ldap认证，但要求zbx本身账户内用户存在并和LDAP用户名一致，所以要取出LDAP用户并插入zbx

#### 配置
[官方描述](https://www.zabbix.com/documentation/3.4/manual/web_interface/frontend_sections/administration/authentication)
zabbix管理--认证--LDAP，填写如下

|描述|内容|
|:-----|:-----|
|LDAP主机(LDAP host)|ldap://192.168.1.99|
|端口(Port)|389|
|基于DN(Base DN)|DC=zbx,DC=comLDAP|
|搜索属性(Search attribute)|sAMAccountName|
|绑定 DN(Bind DN)|cn=Admin,ou=zabbix,dc=zbx,dc=com|
|绑定密码(Bind password)|不进行任何操作|
|测试认证(Test authentication)|[必需为一个正确的LDAP用户]|
|登录(Login)|默认为当前登录用户|
|用户密码(User password)|当前用户密码|



#### 安装

yum -y install openldap-clients

#### 使用
查找某用户信息

```
[root@zabbix ~]# ldapsearch -x -W -D "cn=administrator,cn=users,dc=zbx,dc=com" -b "cn=administrator,cn=users,dc=zbx,dc=com" -h 192.168.1.xx
Enter LDAP Password: 
# extended LDIF
#
# LDAPv3
# base <cn=administrator,cn=users,dc=zbx,dc=com> with scope subtree
# filter: (objectclass=*)
# requesting: ALL
#

# Administrator, Users, zbx.com
dn: CN=Administrator,CN=Users,DC=zbx,DC=com
objectClass: top
objectClass: person
objectClass: organizationalPerson
objectClass: user
cn: Administrator
description:: 566h55CG6K6h566X5py6KOWfnynnmoTlhoXnva7luJDmiLc=
distinguishedName: CN=Administrator,CN=Users,DC=zbx,DC=com
instanceType: 4
...
...

```

查找ou下的用户信息
```
[root@zabbix openldap]# ldapsearch -H ldap://192.168.1.xx -x -D "cn=Admin,ou=zabbix,dc=zbx,dc=com" -b "ou=zabbix,dc=zbx,dc=com" -W -LLL | grep sAMAccountName 
Enter LDAP Password: 
sAMAccountName: zabbix-group
sAMAccountName: test01
sAMAccountName: Admin
[root@zabbix openldap]# 
```


#### 脚本
可自己编写脚本，执行相关命令，处理返回结果等。但是太过于繁杂。目前py有模块可用，python-ldap
此次脚本测试为LDAP是MS的AD环境。
安装模块环境：
Debian/Ubuntu:
`sudo apt-get install libsasl2-dev python-dev libldap2-dev libssl-dev`
RedHat/CentOS:
`yum -y install python-devel libevent-devel openldap-devel`
安装模块
`pip install gevent python-ldap`


```python
[root@zabbix ~]# cat get_ldap.py 
#coding: utf-8
import ldap

class LdapToZbx():
    def __init__(self,ldap_url,domain_name,admin_user,admin_pwd):
        self.ldap_url = ldap_url
        self.domain_name = domain_name
        self.admin_user = admin_user
        self.admin_pwd = admin_pwd
        ldap_user = '%s@%s' %(admin_user,domain_name)
        self.con = ldap.initialize(ldap_url) #初始连接
        self.con.protocol_version = ldap.VERSION3 #LDAP 版本
        try:
            self.con.simple_bind_s(ldap_user,admin_pwd)
        except ldap.LDAPError,err:
            self.ldap_error = 'Connect to %s failed, Error:%s.' %(ldap_url,err.message['desc'])
            print self.ldap_error

    def search_ou_user(self,baseDN):
        user_list=[]
        searchFilter = '(&(objectClass=person))' #只查找objectClass=person的
        results = self.con.search_s(baseDN,ldap.SCOPE_SUBTREE,searchFilter)
        if results is not None:
            for person in results:
                username = person[1]['sAMAccountName'][0]
                user_list.append(username)
            return user_list

if __name__ == "__main__":                
    ldap_url = 'ldap://192.168.1.xx:389'
    domain_name = u'zbx.com'
    admin_user = u'administrator'
    admin_pwd = u'xxxxxxx'

    baseDN = u'ou=zabbix,dc=zbx,dc=com'

    ldap_user =  LdapToZbx(ldap_url,domain_name,admin_user,admin_pwd)
    user = ldap_user.search_ou_user(baseDN)    
    print user

#执行结果
[root@zabbix ~]# python get_ldap.py 
['test01', 'Admin']
```


后续调用zbx接口，将用户插入进去，即可完成LDAP单点登录。

```python
#coding: utf-8
import requests
import urllib
import json

class Zbx():
    #auth=None
    zbx_host='192.168.1.100'
    api_url="http://%s/zabbix/api_jsonrpc.php"%zbx_host
    __user="xxxxxx"
    __password="xxxxxxx"


    def __init__(self,zbx_host=zbx_host,api_url=api_url,user=__user,password=__password):
        self.user=user
        self.password=password
        self.zbx_host=password
        self.api_url=api_url
    #登录，获取auth   
    def login(self):
        data = {"jsonrpc":"2.0","method":"user.login","params":{"user":self.user,"password":self.password},"id":0}
        headers={"Content-Type": "application/json"}
        try:
            req = requests.post(self.api_url,data=json.dumps(data),headers=headers,timeout=5)
            self.auth = req.json()["result"]
            #print(self.auth)
            return self.auth
        except Exception as exc:
            print(exc.args)
        print("Get auth error")
        return 1

    #获取指定组id
    def get_usrgrpid(self,groupname):
        self.login()
        data = {
            "jsonrpc": "2.0",
            "method": "usergroup.get",
            "params": {
                "output": "extend",
                "filter":{
                    "name":groupname
                },
                "status": 0
                },
            "auth": self.auth,
            "id": 1
            }
        try:
            req = requests.post(self.api_url, data=json.dumps(data), headers={"Content-Type": "application/json-rpc"},
                                timeout=5)
            res_json=req.json()
            #print res_json
            return res_json['result'][0]['usrgrpid']
                
        except Exception as exc:
            return 1
        return 1

    #创建用户
    def create_user(self,username,groupname):
        self.login()
        group_id=self.get_usrgrpid(groupname)
        data={
            "jsonrpc": "2.0",
            "method": "user.create",
            "params": {
                "alias": username,
                "passwd": "",
                "usrgrps": [
                    {
                        "usrgrpid": group_id
                    }
                ],
            },
            "auth": self.auth,
            "id": 1
        }
        try:
            req = requests.post(self.api_url, data=json.dumps(data), headers={"Content-Type": "application/json-rpc"},
                                timeout=5)
            res_json=req.json()
        except Exception as exc:
            return 1
        return 1

```

调用
```python
#!/usr/bin/python
#coding: utf-8

import get_ldap
import zbx_add_user

#ldap相关信息，根据实际情况修改
ldap_url = 'ldap://192.168.1.99:389'
domain_name = u'zbx.com'
admin_user = u'xxxxxxx'
admin_pwd = u'xxxxxxx'

#ou的层级  1：没有层级，直接写。2：有层级则从内到外开始写多个ou
baseDN = u'ou=zabbix,dc=zbx,dc=com'
#baseDN = u'ou=zabbix1,ou=zabbix,dc=zbx,dc=com'

#获取ldap user列表
ldap_conn = get_ldap.LdapToZbx(ldap_url,domain_name,admin_user,admin_pwd)
user_list = ldap_conn.search_ou_user(baseDN)


#zabbix相关信息,根据实际情况修改
zbx_host='192.168.1.100'
api_url="http://%s/zabbix/api_jsonrpc.php"%zbx_host
user="xxxxxx"
password="xxxxxx"

groupname = u'ldap' #ZABBX的组
#插入zbx
user = zbx_add_user.Zbx()
for username in user_list:
    user.create_user(username,groupname)
```

---