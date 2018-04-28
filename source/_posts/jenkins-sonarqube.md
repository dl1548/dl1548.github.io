---
title: jenkins+sonarqube
date: 2018-04-18 17:46:57
tags: jenkins
categories:
    - 运维工具
    - jenkins
copyright: true
---

SonarQube是一个用于代码质量管理的开源平台
<!--more-->

#### 环境准备
系统版本:centos7x64

|应用|版本|
|-|-|
|java|1.8.0_xx|
|mysql|5.6+|
|sonarqube|6.7LTS|
|JENKINS|2.117|

jenkins安装插件`SonarQube` ,插件对jenkins有版本要求.

mysql编译安装,[建议阅读](http://blog.dl1548.site/2018/03/26/mysql5-7%E6%BA%90%E7%A0%81%E5%AE%89%E8%A3%85/)

jenkins安装,[建议阅读](http://blog.dl1548.site/2017/11/09/jenkins%E5%AE%89%E8%A3%85%E5%92%8C%E9%83%A8%E7%BD%B2%E9%A1%B9%E7%9B%AE/)

#### sonarqube 安装

下载安装包: 　[官方下载](https://www.sonarqube.org/downloads/)

`unzip  sonarqube-6.7.zip -d /usr/local/`

由于es的原因,不能用root用户启动,所以要新建用户
```
useradd sonar
chown -R sonar /usr/local/sonarqube-6.7/
```

配置环境变量

```
#sonarqube
export  SONAR_HOME=/usr/local/sonarqube-6.7
export  PATH=${SONAR_HOME}/bin:${PATH}
```

#### 数据库配置
如果不用mysql 可直接启动即可.系统默认使用内置数据库

`create database sonar character set utf8 collate utf8_general_ci;`
授权
`GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'youpassword' WITH GRANT OPTION;`
`flush privileges`


#### 编辑sonar配置

`vim conf/sonar.properties`

```
sonar.jdbc.username= root
sonar.jdbc.password= xxxxx
#..3306/sonar..? 这里的sonar 是dbname
sonar.jdbc.url= jdbc:mysql://192.168.1.56:3306/sonar?useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true&useConfigs=maxPerformance&useSSL=false

#可选
sonar.web.host=本机IP
#默认为9000,可修改
sonar.web.port=9000

```

启动sonar
`su sonar ./bin/linux-x86-64/sonar.sh start`

登录
在浏览器输入：http:// IP：PORT  即可 admin/admin

安装插件: Administrator--->Markeptlace

`Chinese Pack` `Checkstyle`


#### jenkins-SonarQube


`系统设置` ---> `SonarQube servers` ---> `SonarQube installations`
    - Name : 自定义
    - Server URL : sonar地址
    - Server authentication token : snoar中生成.(我的账户--安全--生成令牌)


##### 方法一
修改maven配置.

```
<settings>
    <pluginGroups>
        #添加此行
        <pluginGroup>org.sonarsource.scanner.maven</pluginGroup>
    </pluginGroups>
    <profiles>
        #添加以下,地址视情况而定
        <profile>
            <id>sonar</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            <properties>
                <sonar.host.url>
                  http://192.168.x.xx:9000
                </sonar.host.url>
            </properties>
        </profile>

```

登录 jenkins 在maven项目中.

`BUild` --->`Goals and options`
`clean install`修改为:
`clean install org.sonarsource.scanner.maven:sonar-maven-plugin:3.3.0.603:sonar`

再次执行即可看到项目console输出日志中对代码的分析
```
...
...
[INFO] --- sonar-maven-plugin:3.3.0.603:sonar (default-cli) @ user ---
...
...
[INFO] Working dir: /root/.jenkins/workspace/XXXXXXXX
[INFO] Source paths: src/main/webapp, pom.xml, src/main/java
[INFO] Source encoding: UTF-8, default locale: zh_CN
[INFO] Index files
[INFO] 421 files indexed
[INFO] Quality profile for java: Sonar way
[INFO] Quality profile for js: Sonar way
[INFO] Quality profile for xml: Sonar way
[INFO] Sensor JavaSquidSensor [java]
...
...
```

##### 方法二
添加插件.jenkins安装插件`SonarQube`

这里需要额外安装`sonar-scanner` 这个软件,如果不想自己安装,则在
`全局工具配置` ---> `SonarQube Scanner` 勾选自动安装.并定义个名字


项目配置:

`Pre Steps` ---> `add pre build step`
选择JDK
`Analysis properties` 输入如下

```
#自定义,唯一. 最好和项目一致,这样后续可调用变量通过邮件发送URL
sonar.projectKey=JKSTACK
sonar.projectName=JKSTACK

sonar.projectVersion=1.0
sonar.language=java
sonar.scm.disabled=true
sonar.sourceEncoding=UTF-8

#相对于项目而言的目录
sonar.sources=moni/src/main/java
sonar.java.binaries=moni/target/classes

```


其他参考.
```
#required metadata
#projectKey项目的唯一标识，不能重复
sonar.projectKey=zzzzz
sonar.projectName=zzzzz 
sonar.projectVersion=1.0 
sonar.sourceEncoding=UTF-8
sonar.modules=java-module,javascript-module,html-module

# Java module
java-module.sonar.projectName=Java Module
java-module.sonar.language=java
# .表示projectBaseDir指定的目录
java-module.sonar.sources=.
java-module.sonar.projectBaseDir=src
sonar.binaries=classes

# JavaScript module
javascript-module.sonar.projectName=JavaScript Module
javascript-module.sonar.language=js
javascript-module.sonar.sources=js
javascript-module.sonar.projectBaseDir=webRoot

# Html module
html-module.sonar.projectName=Html Module
html-module.sonar.language=web
html-module.sonar.sources=pages
html-module.sonar.projectBaseDir=webRoot

sonar.projectKey=org.codehaus.sonar:php-sonar-runner-unit-tests
sonar.projectName=PHP project analyzed with the SonarQube Runner reusing PHPUnit reports
sonar.projectVersion=1.0
sonar.sources=src
sonar.tests=tests
sonar.language=php
sonar.sourceEncoding=UTF-8
# Reusing PHPUnit reports
sonar.php.coverage.reportPath=reports/phpunit.coverage.xml
sonar.php.tests.reportPath=reports/phpunit.xml
```


