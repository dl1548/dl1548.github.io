---
title: jenkins-Publish Over SSH Plugin
date: 2018-04-11 15:24:01
tags: jenkins
categories:
    - 运维工具
    - jenkins
copyright: true
---
jenkins通过SSH部署项目到tomcat下
<!--more-->

#### 安装
插件名就是`Publish Over SSH` 安装过程不赘述.

#### 配置

系统管理-->系统配置-->Publish Over SSH

说明:
`Passphrase` ：密码（key的密码，未设置则空）
`Path to key`：key私钥路径`/root/.ssh/id_rsa`
    - 这个key是要手动生的,命令`ssh-keygen -t rsa`这里会提示是否对key加密.也就是`Passphrase` 
    - 免密设置`ssh-copy-id 192.168.xx.xx`
`Key`：私钥内容复制到这里
`Disable exec`：禁止运行命令, 不勾选

SSH Servers配置,有多台可以加多台
    - SSH Server Name：标识的名字（随便取,建议IP）
    - Hostname：需要连接ssh的主机名或ip地址（建议ip）
    - Username：用户名
    - Remote Directory：远程目录(后续有些相对目录的指定就是以此为基准的)
    - Use password authentication, or use a different (√)
私有配置的高级：
    - Port：端口（默认22）
    - Timeout (ms)：超时时间（毫秒）默认即可
    - Disable exec：禁止运行命令
    - Test Configuration：测试连接

至此,相关的基础配置已完毕.

#### 使用

点进一个`maven`项目
`构建环境处`
Build
    - Root POM : moni/pom.xml (pom文件是在项目根目录下,如果分支就一个项目则不需要写项目名)
    - Goals and options : clean install




在`构建后操作`选项
点击增加构建后操作步骤,选择`send build artifacts over SSH` 会增加相关选项界面.

SSH Publishers
    - SSH Server
        - NAME (选择刚刚配置的那个即可)
        - Transfers
            - Source files : moni/target/*.war(上传的文件,相对于工作区的路径,可写多个，默认用,分隔)
            - Remove prefix：moni/target/（只能指定Transfer Set Source files中的目录,不易出,web路径会变）
            - Remote directory：远程目录,这个是基于配置中的那个而言的相对路径
            - Exec command：~/deploy.sh monitor 80 /usr/local/tomcat-7.0.85(也可直接写脚本内容,后会有个脚本参考,具体参数看根据项目而定)
            -
            高级部分
            - Exclude files：排除文件（传输目录的时候很有用，使用通配符，例如：**/*.log,**/*.tmp,.git/）

            其他:略


##### 脚本参考
来自网友,稍加修改,出处不知.
前提要在目标主机新建文件夹`mkdir -p ~/war/bak`
然后将脚本`scp`到目标主机,具体位置,根据上一步中`Exec command：~/deploy.sh monitor 80 /usr/local/tomcat-7.0.85` 而定.

```
#!/usr/bin/bash

#导入JAVA
export RUN_AS_USER=root
export JAVA_HOME=/usr/local/jdk1.7.0_79

#默认变量值
PROJECT="$1"
TOMCAT_PORT="$2"
TOMCAT_HOME="$3"

#参数检验./deploy.sh <projectname> [tomcat port] [tomcat home dir]

if [ $# -lt 3 ]; then
  echo "you must use like this : ./deploy.sh <projectname> [tomcat port] [tomcat home dir]"  
  exit
fi
#根据端口查找tomcatpid,可能有多个,循环中判断
tomcat_pid=`netstat -anp | grep $TOMCAT_PORT | awk '{printf $7}' | cut -d "/" -f 1`
echo "current :" $tomcat_pid
while [ -n "$tomcat_pid" ]
do
 sleep 5
 #进一步筛选
 tomcat_pid=`ps -ef | grep $tomcat_pid |grep $TOMCAT_HOME | grep -v 'grep\|tail\|more\|bash\|less'| awk '{print $2}'`
 echo "scan tomcat pid :" $tomcat_pid
 if [ -n "$tomcat_pid" ]; then
   echo "kill tomcat :" $tomcat_pid
   kill -9 $tomcat_pid
 fi
done

#备份路径
BAK_DIR=$HOME/war/bak/$PROJECT/`date +%Y%m%d`
mkdir -p "$BAK_DIR"
cp "$TOMCAT_HOME"/webapps/$PROJECT.war "$BAK_DIR"/"$PROJECT"_`date +%H%M%S`.war

#publish project
echo "scan no tomcat pid,$PROJECT publishing"

rm -rf "$TOMCAT_HOME"/webapps/$PROJECT
rm -rf "$TOMCAT_HOME"/webapps/$PROJECT.war
#cp $HOME/war/$PROJECT.war "$TOMCAT_HOME"/webapps/$PROJECT.war
mv $HOME/war/$PROJECT.war "$TOMCAT_HOME"/webapps/$PROJECT.war
#remove tmp
#rm -rf $HOME/war/$PROJECT.war

sleep 10
#start tomcat
"$TOMCAT_HOME"/bin/startup.sh
echo "tomcat is starting,please try to access $PROJECT conslone url"
```