---
title: linux个人备忘手册
date: 1111-11-11 11:11:12
tags: -个人备忘手册
categories: linux
copyright: true
password: woshizhu
---

> 一些小命令的记录

<!--more-->

#### 按当前日期命名文件
```
创建备份Shell脚本:
输入/粘贴以下内容：
#!/bin/bash
mysqldump -uusername -ppassword DatabaseName > /home/backup/DatabaseName_$(date +%Y%m%d_%H%M%S).sql
对备份进行压缩：
#!/bin/bash
mysqldump -uusername -ppassword DatabaseName | gzip > /home/backup/DatabaseName_$(date +%Y%m%d_%H%M%S).sql.gz
```

#### 定时备份

```
#!/usr/bin/bash
cd /var/log/
tar -zcf /var/log/haproxy_log_bk/$(date +%Y%m%d_%H%M%S).tar.gz haproxy.log
echo "" > haproxy.log
```

#### -mtime
```
查找 并移动3天前的
find /net-log/ -mtime +3 -name "*.log" -exec mv {} /tmp/ \;
删除
find /net-log/ -mtime +3 -name "*.log" -exec rm -rf {} \;

命令可写到定时任务中,对长时间不读取文件进行删除
crontab -e
* * * * *  + 程序 +命令或脚本
#如果是bash命令 可直接写.
例:每天0点执行test.py
0 0 * * *  /usr/bin/python3   /script/test.py
```

#### ssh快捷登录
```
zili@Ubuntu:~$ cat ~/.ssh/config
ServerAliveInterval 60 #60S发送一次存活信息,以免断开
Host study
User root
Hostname 10.1.1.11
Port 21131
#默认为22 有修改则填写
#若有多个主机,直接复制修改即可
zili@Ubuntu:~$ ssh study
root@10.1.1.11's password:
```

#### mysql开启远程

`GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'youpassword' WITH GRANT OPTION;`


```
mysql> select user,host from mysql.user;
+--------+-----------+
| user   | host      |
+--------+-----------+
| root   | 127.0.0.1 |
| root   | localhost |
+--------+-----------+

mysql> show status like 'Threads%';
+-------------------+-------+
| Variable_name     | Value |
+-------------------+-------+
| Threads_cached    | 58    |
| Threads_connected | 57    |   ###这个数值指的是打开的连接数
| Threads_created   | 3676  |
| Threads_running   | 4     |   ###这个数值指的是激活的连接数，这个数值一般远低于connected数值
+-------------------+-------+

Threads_connected 跟show processlist结果相同，表示当前连接数。准确的来说，Threads_running是代表当前并发数

这是是查询数据库当前设置的最大连接数
mysql> show variables like '%max_connections%';
+-----------------+-------+
| Variable_name   | Value |
+-----------------+-------+
| max_connections | 1000  |
+-----------------+-------+

可以在/etc/my.cnf里面设置数据库的最大连接数
[mysqld]
max_connections = 10000
```

#### 开启snmp
`yum –y install net-snmp net-snmp-devel`
若要使用snmpwalk进行安装检测，则还需要
`yum –y install net-snmp-utils`

`vi /etc/snmp/snmpd.conf`
把`62`行中的`systemview`改为`mib2`
把`89行的`#`去掉。
然后在最后一行添加 `rwcommunity  ge.` 保存退出。
防火墙添加策略,重启服务即可
`snmpwalk -v 2c -c public localhost sysName.0`可做验证,默认社区号是`public`
若需要修改则`41`行中`public`换为指定字符串即可


#### openssl自签
Key是私用秘钥，通常是RSA算法
Csr是证书请求文件，用于申请证书。在制作csr文件时，必须使用自己的私钥来签署申，还可以设定一个密钥。
crt是CA认证后的证书文，签署人用自己的key给你签署凭证。
```
# 生成一个RSA密钥
openssl genrsa -des3 -out 33iq.key 1024

# 拷贝一个不需要输入密码的密钥文件
openssl rsa -in 33iq.key -out 33iq_nopass.key

# 生成一个证书请求
openssl req -new -key 33iq.key -out 33iq.csr

# 自己签发证书
openssl x509 -req -days 365 -in 33iq.csr -signkey 33iq.key -out 33iq.crt

```

#### pip源更换

pip国内的一些镜像
    - 阿里云 http://mirrors.aliyun.com/pypi/simple/
    - 中国科技大学 https://pypi.mirrors.ustc.edu.cn/simple/
    - 豆瓣(douban) http://pypi.douban.com/simple/
    - 清华大学 https://pypi.tuna.tsinghua.edu.cn/simple/
    - 中国科学技术大学 http://pypi.mirrors.ustc.edu.cn/simple/

临时使用：
可以在使用pip的时候在后面加上-i参数，指定pip源
`pip install scrapy -i https://pypi.tuna.tsinghua.edu.cn/simple`

永久修改：
    linux:
    修改 ~/.pip/pip.conf (没有就创建一个)，如下：
```
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
```


#### shell保留最近的N个文件

`cd 指定目录 && ls -lt | awk '{if(NR>=11){print $9}}' | xargs rm -f`



#### shell for循环计数比对

```
get_disk(){
    i=1
    for line in `iostat -d | awk -F " *" '{print $1}'`;do
            i=$((i+1))
    done

    ii=1
    printf "{\n"
    printf "\t\"data\":[\n"
    for line in `iostat -d | awk -F " *" '{print $1}'`;do
            ii=$((ii+1))
            disk=$line
            if [ "$ii" == $i ];
            then
                printf "\t{\t\t\"{#DISK}\":\"$disk\"\t}";
            else
                printf "\t{\t\t\"{#DISK}\":\"$disk\"\t},";
            fi
    done
    printf "\n\t]\n"
    printf "}\n"
}
echo $(get_disk)
```



#### shell 统计指定文件ip数

```
在/tmp下有大量文件access1.log,access2.log...，内容格式为:时间 IP,例:

a1.log
2018-01-01 127.0.0.1
2018-01-02 127.0.0.1
2018-01-02 127.0.0.1
2018-01-02 10.10.2.2
2018-01-03 192.168.1.1
...
...

取出最后十个文件,文件内容去重,并统计重复IP数,取第四行,存入count中


!/bin/bash
for filename in find /tmp-type f -name "access*.log" | tail -n 10
do
  sed -n '4p' $filename | awk '{print $2}' | uniq -c >>count
done

```



#### shell 系统判断

```
#!/bin/bash
check_os_release()
{
  while true
  do
    os_release=$(grep "Red Hat Enterprise Linux Server release" /etc/issue 2>/dev/null)
    os_release_2=$(grep "Red Hat Enterprise Linux Server release" /etc/redhat-release 2>/dev/null)
    if [ "$os_release" ] && [ "$os_release_2" ]
    then
      if echo "$os_release"|grep "release 5" >/dev/null 2>&1
      then
        os_release=redhat5
        echo "$os_release"
      elif echo "$os_release"|grep "release 6" >/dev/null 2>&1
      then
        os_release=redhat6
        echo "$os_release"
      elif echo "$os_release"|grep "release 7" >/dev/null 2>&1
      then
        os_release=redhat7
        echo "$os_release"
      else
        os_release=""
        echo "$os_release"
      fi
      break
    fi
    os_release=$(grep "Aliyun Linux release" /etc/issue 2>/dev/null)
    os_release_2=$(grep "Aliyun Linux release" /etc/aliyun-release 2>/dev/null)
    if [ "$os_release" ] && [ "$os_release_2" ]
    then
      if echo "$os_release"|grep "release 5" >/dev/null 2>&1
      then
        os_release=aliyun5
        echo "$os_release"
      elif echo "$os_release"|grep "release 6" >/dev/null 2>&1
      then
        os_release=aliyun6
        echo "$os_release"
      elif echo "$os_release"|grep "release 7" >/dev/null 2>&1
      then
        os_release=aliyun7
        echo "$os_release"
      else
        os_release=""
        echo "$os_release"
      fi
      break
    fi
    os_release_2=$(grep "CentOS" /etc/*release 2>/dev/null)
    if [ "$os_release_2" ]
    then
      if echo "$os_release_2"|grep "release 5" >/dev/null 2>&1
      then
        os_release=centos5
        echo "$os_release"
      elif echo "$os_release_2"|grep "release 6" >/dev/null 2>&1
      then
        os_release=centos6
        echo "$os_release"
      elif echo "$os_release_2"|grep "release 7" >/dev/null 2>&1
      then
        os_release=centos7
        echo "$os_release"
      else
        os_release=""
        echo "$os_release"
      fi
      break
    fi
    os_release=$(grep -i "ubuntu" /etc/issue 2>/dev/null)
    os_release_2=$(grep -i "ubuntu" /etc/lsb-release 2>/dev/null)
    if [ "$os_release" ] && [ "$os_release_2" ]
    then
      if echo "$os_release"|grep "Ubuntu 10" >/dev/null 2>&1
      then
        os_release=ubuntu10
        echo "$os_release"
      elif echo "$os_release"|grep "Ubuntu 12.04" >/dev/null 2>&1
      then
        os_release=ubuntu1204
        echo "$os_release"
      elif echo "$os_release"|grep "Ubuntu 12.10" >/dev/null 2>&1
      then
        os_release=ubuntu1210
        echo "$os_release"
      elif echo "$os_release"|grep "Ubuntu 14.04" >/dev/null 2>&1
      then
        os_release=ubuntu1204
        echo "$os_release"
      elif echo "$os_release"|grep "Ubuntu 16.04" >/dev/null 2>&1
      then
        os_release=ubuntu1604
        echo "$os_release" 
      else
        os_release=""
        echo "$os_release"
      fi
      break
    fi
    os_release=$(grep -i "debian" /etc/issue 2>/dev/null)
    os_release_2=$(grep -i "debian" /proc/version 2>/dev/null)
    if [ "$os_release" ] && [ "$os_release_2" ]
    then
      if echo "$os_release"|grep "Linux 6" >/dev/null 2>&1
      then
        os_release=debian6
        echo "$os_release"
      elif echo "$os_release"|grep "Linux 7" >/dev/null 2>&1
      then
        os_release=debian7
        echo "$os_release"
      else
        os_release=""
        echo "$os_release"
      fi
      break
    fi
    break
    done
}

os_release=$(check_os_release)
```

#### shell获取系统硬件资源信息

```
#获取centos信息
get_info(){
  os_sys=`uname -o`
  product_id=`dmidecode -t 1 | grep Serial`
  if [[ $os_release =~ 'ubuntu' ]];then
    os_version=`grep -i "ubuntu" /etc/issue | awk -F' ' '{print $1$2$3}'`
  elif [[ $os_release =~ 'centos' ]];then
    os_version=`cat /etc/redhat-release`
  elif [[ $os_release =~ 'redhat' ]];then
    os_version=`cat /etc/redhat-release`
  else
    os_version=`cat /etc/redhat-release`
  fi
  os_kernel=`uname -r`
  cpu_model=`grep 'model name' /proc/cpuinfo |uniq |awk -F : '{print $2}'`
  cpu_num=`cat /proc/cpuinfo| grep 'physical id'| sort| uniq| wc -l`
  cpu_core=`grep 'cpu cores' /proc/cpuinfo |uniq |awk -F : '{print $2}'`
  cpu_load=`cat /proc/loadavg | awk '{print $1}'`
  #mem_total=`free -m |grep -i mem |awk '{ print $2}'`
  #系统总内存,与物理不符,因为系统初始化会保留一部分内存
  mem_total=(`dmidecode -t memory | grep 'Installed Size' | awk -F : 'NR==1{ print $2 }'`)
  #disk_total=`lsblk | grep -E -i 'disk|磁盘' | grep -E -i 'sd|vd' | awk '{ print $4 }' | sed 's/G//g'`
  disk_total_o=`cat /proc/partitions | grep -w "0" |grep -E -i 'sd|vd' | awk '{print $3}'`
  disk_total=$[disk_total_o/1024/1024]

  # SUSE lsblk -m | grep -E -i 'disk|磁盘' | grep -E -i 'sd|vd'  | awk 'NR==1{ print $2 }'

#dict
  echo "{\
    \"os_sys\":\"$os_sys\",\
    \"os_version\":\"$os_version\",\
    \"os_kernel\":\"$os_kernel\",\
    \"cpu_model\":\"$cpu_model\",\
    \"cpu_num\":\"$cpu_num\",\
    \"cpu_core\":\"$cpu_core\",\
    \"mem_total\":\"$mem_total\",\
    \"product_id\":\"$product_id\",\
    \"disk_total\":\"$disk_total\"\
  }"
}

echo $(get_info)

```



#### win7 centos双系统 引导修复

先有centos7 再有win7 

```
# 修复grub引导
grub2-install --root-directory=/mnt/sysimage /dev/sda
sync
reboot
```

先有win7 再有centos7

需要安装`ntfs-3g`

```
yum -y install epel-release
yum -y install ntfs-3g

# 修复启动项
grub2-mkconfig -o /boot/grub2/grub.cfg



# 设置默认启动win7，"Windows 7"是根据文件/boot/grub2/grub.cfg中得到的

# 查找配置文件中的 menuentry 'xxxxxx' 的项就是了, 'xxxxxx'可随意更改。
grub2-set-default  "Windows 7"
grub2-editenv list
```











___
