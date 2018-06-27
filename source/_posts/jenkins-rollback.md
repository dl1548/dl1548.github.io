---
title: jenkins rollback
date: 2018-05-08 13:35:18
tags: jenkins
categories:
    - 运维工具
    - jenkins
copyright: true
password: woshizhu
---

回滚之前,确保自己的项目是有备份的
<!--more-->

#### 新建项目
新建一个自由风格
 - general
    - 参数化构建过程(运行时参数)
        - 名称: ROLL_BACK (自定义)
        - 项目: 项目名字(此项目名字要写存在的,即,要回滚的项目)
        - 描述: 随便写
        - filter: 成功的构建.

其余不写.

#### 构建
添加执行shell模块
这里注意的是:ROLL_BACK对应的就是项目名字

可以
```
tmp=${ROLL_BACK%/*}
BUILD_NUMBER=${tmp##*/}
ssh root@192.168.1.55 /root/deploy.sh rollback monitor 80 /usr/local/tomcat-7.0.85 $BUILD_NUMBER
```

#### 脚本参考

```bash
#!/bin/bash
#JAVA
export RUN_AS_USER=root
export JAVA_HOME=/usr/local/jdk1.7.0_79
#how to use
#./deploy.sh <projectname> [tomcat port] [tomcat home dir] $BUILD_NUMBER
#~/deploy.sh deploy/rollback monitor 80 /usr/local/tomcat-7.0.85 $BUILD_NUMBER
#default var
action="$1"
PROJECT="$2"
TOMCAT_PORT="$3"
TOMCAT_HOME="$4"
VERSION="$5"
#dir exist?
if [ ! -d "$HOME/war/bak/" ];then
mkdir -p $HOME/war/bak/
fi
#args num
if [ $# -lt 4 ]; then
  echo "you must use like this : ./deploy.sh <action> <projectname> [tomcat port] [tomcat home dir]"
  exit
fi
#search tomcatpid
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

#path for bakdir
BAK_DIR=$HOME/war/bak/$PROJECT
if [ ! -d "$BAK_DIR" ];then
    mkdir -p $BAK_DIR
fi

if [ $action == 'deploy' ];then
  cp $HOME/war/$PROJECT.war "$BAK_DIR"/"$PROJECT"_"$VERSION".war
  #publish project
  echo "scan no tomcat pid,$PROJECT publishing...."
  sleep 10
  rm -rf "$TOMCAT_HOME"/webapps/$PROJECT
  rm -rf "$TOMCAT_HOME"/webapps/$PROJECT.war
  #this path is define by jenkins-after build-ssh-Remote directory
  mv $HOME/war/$PROJECT.war "$TOMCAT_HOME"/webapps/$PROJECT.war
fi

if [ $action == 'rollback' ];then
  #delete project
  rm -rf "$TOMCAT_HOME"/webapps/$PROJECT
  rm -rf "$TOMCAT_HOME"/webapps/$PROJECT.war
  cp "$BAK_DIR"/"$PROJECT"_"$VERSION".war "$TOMCAT_HOME"/webapps/$PROJECT.war
  #publish project
  echo "rollback to the project $PROJECT $VERSION"
  sleep 3
fi

if
#start tomcat
"$TOMCAT_HOME"/bin/startup.sh
echo "tomcat is starting,please try to access $PROJECT conslone url"

```
