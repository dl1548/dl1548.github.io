---
title: ldapsearch
date: 2018-01-21 20:31:43
tags: 
    - ldap
    - python
categories: 
    - python
copyright: true
---

LDAP认证系统可再多系统间实现单点登录，免去多账户的烦恼。

<!--more-->

由于zabbix要实现单点登录，因此而学习了下。
zbx本身支持ldap认证，但要求zbx本身账户内用户存在并和LDAP用户名一致，所以要取出LDAP用户并插入zbx



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


```
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

---