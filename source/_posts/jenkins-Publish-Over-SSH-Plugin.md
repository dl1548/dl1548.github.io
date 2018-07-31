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

```bash
#!/bin/bash
#jenkins
#jenkins需要做配置 send..ssh

#Remote directory : tmp/ 
#这个相对路径取决于系统配置信息里面的ssh主机的 Remote Directory,建议tmp,因为备份在tmp  

#Exec command : ~/deploy.sh deploy cmdb $BUILD_NUMBER

#tomcat
t_home_bin=`find / -name catalina.sh`
t_home=${t_home_bin%*/bin/*}
t_port=`cat $t_home/conf/server.xml | grep 'HTTP/1.1' | grep protocol | grep Connector |awk '{print $2}' | tr -cd "[0-9]"`

#export
source /etc/profile
source $HOME/.bashrc
#how to use
#./deploy.sh <projectname> [tomcat port] [tomcat home dir] $BUILD_NUMBER
#default var
ACTION="$1"
PROJECT="$2"
TOMCAT_PORT=$t_port
TOMCAT_HOME=$t_home
VERSION="$3"
#dir exist?
if [ ! -d "/tmp/war/bak/" ];then
mkdir -p /tmp/war/bak/
fi
#args num
if [ $# -lt 3 ]; then
  echo "you must use like this : ./deploy.sh <action> <projectname> <version>"
  exit
fi
#search tomcatpid
#tomcat_pid=`netstat -anp | grep $TOMCAT_PORT | awk '{printf $7}' | cut -d "/" -f 1`
tomcat_p=`netstat -anp | grep $TOMCAT_PORT | awk '{printf $7}' | cut -d "/" -f 1`
tomcat_pid=${tomcat_p#*-}
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

#path for bakdir
BAK_DIR=/tmp/war/bak/$PROJECT
if [ ! -d "$BAK_DIR" ];then
    mkdir -p $BAK_DIR
fi

if [ $ACTION == 'deploy' ];then
  cp /tmp/$PROJECT.war "$BAK_DIR"/"$PROJECT"_"$VERSION".war
  #publish project
  echo "+++++++++++++++++++++++++++++++++++++++++++++++++++"
  echo "scan no tomcat pid,$PROJECT publishing...."
  sleep 10
  rm -rf "$TOMCAT_HOME"/webapps/$PROJECT
  rm -rf "$TOMCAT_HOME"/webapps/$PROJECT.war
  #this path is define by jenkins-after build-ssh-Remote directory
  mv /tmp/$PROJECT.war "$TOMCAT_HOME"/webapps/$PROJECT.war
fi

if [ $ACTION == 'rollback' ];then
  #delete project
  rm -rf "$TOMCAT_HOME"/webapps/$PROJECT
  rm -rf "$TOMCAT_HOME"/webapps/$PROJECT.war
  cp "$BAK_DIR"/"$PROJECT"_"$VERSION".war "$TOMCAT_HOME"/webapps/$PROJECT.war
  #publish project
  echo "+++++++++++++++++++++++++++++++++++++++++++++++++++"
  echo "rollback to the project $PROJECT $VERSION"
  echo "+++++++++++++++++++++++++++++++++++++++++++++++++++"
  sleep 3
fi

#start tomcat
"$TOMCAT_HOME"/bin/startup.sh
echo "+++++++++++++++++++++++++++++++++++++++++++++++++++"
echo "tomcat is starting,please try to access $PROJECT conslone url"

#del war
echo "+++++++++++++++++++++++++++++++++++++++++++++++++++"
echo `date +%Y%m%d` "delete invalid war"
find $BAK_DIR -mtime +7 -name "*.war";
find $BAK_DIR -mtime +7 -name "*.war" -exec rm -rf {} \;
echo "+++++++++++++++++++++++++++++++++++++++++++++++++++"
echo "host address:" `/sbin/ifconfig -a|grep inet|grep -v 127.0.0.1|grep -v inet6 | awk '{print $2}' | tr -d "addr:"`
echo "+++++++++++++++++++++++++++++++++++++++++++++++++++"

```