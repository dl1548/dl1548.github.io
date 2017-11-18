---
title: jenkins安装和部署项目
date: 2017-11-09 20:43:25
tags: jenkins
categories:
    - 运维工具
    - jenkins
copyright: true
---
大致说下jenkins自动化的流程:
```
 程序员--------->远程仓库
        提交至     |                           远程主机
                  | 提取                           |
                  |                               | 部署
                  ↓       构建[编译,打包]           |
               Jenkins ------------------->Jenkins Server
```
<!--more-->

#### 安装git
(根据环境选择是否安装)
[推荐阅读](https://dl1548.github.io/2017/07/12/git客户端最新版安装/)

#### 安装maven
(根据环境选择是否安装)
maven用于构建,如果你的环境不需要构建可以不安装
[选择版本](http://maven.apache.org/download.cgi)下载
`wget http://mirrors.hust.edu.cn/apache/maven/maven-3/3.5.2/binaries/apache-maven-3.5.2-bin.tar.gz`

*解压*
`tar -zxvf 安装包 -C /usr/local`
`cd /usr/local`
` mv apache-maven-3.5.2 /usr/local/maven` #可选

*添加环境变量*
`/etc/profile 或者~/.bash_profile`
```bash
#maven目录
export M2_HOME=/usr/local/maven
#maven的bin目录
export M2=$M2_HOME/bin
#将bin加到PATH变量中
export PATH=$M2:$PATH
```
*配置生效*
`source /etc/profile`
*查看*
`mvn -version`
```
[root@jenkins /]# mvn -version
Apache Maven 3.5.2 (138edd61fd100ec658bfa2d307c43b76940a5d7d; 2017-10-18T15:58:13+08:00)
Maven home: /usr/local/maven
Java version: 1.8.0_151, vendor: Oracle Corporation
Java home: /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.151-1.b12.el7_4.x86_64/jre
Default locale: zh_CN, platform encoding: UTF-8
OS name: "linux", version: "3.10.0-327.el7.x86_64", arch: "amd64", family: "unix"
```
#### 安装jenkins
(根据环境必须安装)
[官网下载](https://jenkins.io/download/)以下摘自官方
```
#下载仓库源,导入公钥
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key

#安装
yum -y install jenkins
#运行(/usr/lib/jenkins)
java -jar jenkins.war
```
#### 访问
http://ipaddress:8080/  
若出现如下内容
> Unlock Jenkins
>To ensure Jenkins is securely set up by the administrator, a password has been written to the log (not sure where to find it?) and this file on the server:
/root/.jenkins/secrets/initialAdminPassword
Please copy the password from either location and paste it below.
Administrator password

根据提示,复制密码解锁即可.
##### 通过tomcat启动
*安装tomcat*
http://tomcat.apache.org/
下载软件包
```
tar -zxvf apache-tomcat-7.0.75.tar.gz 
mv apache-tomcat-7.0.75 /usr/local/tomcat
mv /usr/lib/jenkins/jenkins.war /usr/local/tomcat/webapps
#启动
/usr/local/tomcat/bin/startup.sh

```
访问
http://ipaddress:8080/ jenkins

##### 注意
构建,部署需要两个插件(常用于java)
>Deploy to container Plugin 和  Maven Integration plugin 
使用git的要安装Git Plugin

进入jenkins,警告如下
>Your container doesn’t use UTF-8 to decode URLs. If you use non-ASCII characters as a job name etc, this will cause problems. See Containers and Tomcat i18n for more details.

去tomcat的conf/下修改serverl.xml
```xml
<Connector port="8080" protocol="HTTP/1.1"
                URIEncoding="UTF-8"         #添加此参数
               connectionTimeout="20000"
               redirectPort="8443" />
```

jenkins环境至此搭建完毕.

#### 熟悉jenkins
**系统设置**
   - 包括一些全局设置,工具的设置,证书创建,插件管理等等


#### 部署静态页面
##### 部署到httpd
温习下流程:
>1. 程序员提交代码到远程仓库.
>2. jenkins 从远程仓库下载代码.
>3. jenkins 构建(编译/打包)
>4. jenkins 部署
>5. 完毕

代码的下载(git)要用到git plug插件(svn要用到Subversion Plug-in),部署到远端服务器需要Publish Over SSH(httpd,nginx等).如果要部署到tomcat,jboss建议使用Deploy to container Plugin,当然ssh(通过脚本)也可以

- 新建一个自由风格的项目 命名为free(名字自定义)
- `General`页面,点开高级选项卡,手动指定此项目的工作空间
```
/usr/local/jenkins/workspace/free #自定义路径
```
- `源码管理` 页面选择代码托管方式
  - 一般会有三个选择 None,Git,Subversion.根据情况选择.我选择GIT.根据情况填写仓库URL以及添加秘钥.(秘钥可以在**Credentials** 下先添加,这里选择.)
此时jenkins和git打通.
- `构建` 页面选择**增加构建步骤**--`execute shell`用来执行脚本
```
cd /usr/local/jenkins/workspace/free;  #切换到项目工作空间
tar cf free.tar *;   #执行shell 打包文件
```
- `构建后操作`页面**增加构建后步骤**--`send build artifacts over SSH`(通过SSH将构建的东西发送到远端主机) 这个选择需要安装上文提到的ssh插件.
  - **name**: 这个选择需要去`系统管理`--`系统设置`--`Publish over SSH` 中添加ssh服务器.(name:自定义.hostname:主机地址.username:用户名,Remote Directory:远端主机目录,不建议填写,建议在项目下填写.点开`高级`,输入密码.点击`test configuration`,显示返回成功即可)
  - **Transfer Set Source files** :`*.tar` 注意这个路径是针对工作空间而言的相对路径.
  - **Remote directory** : 远端主机目录 `/var/www/html/`
  - **Exec command**: 传输后执行的命令,针对远端主机的
```
cd  /var/www/html/;
tar xvf free.tar; 
保存,执行构建任务即可.
```
  
___
