---
title: nexus
date: 2018-03-15 15:20:01
tags: nexus
categories: linux
copyright: true
---

[nexus官网](https://www.sonatype.com/)
[下载地址](https://www.sonatype.com/download-oss-sonatype)

2.4 Nexus介绍
maven的仓库只有两大类：1.本地仓库 2.远程仓库，在远程仓库中又分成了3种：

#### 安装java
并配置相关java环境变量
```
#java1.8
export RUN_AS_USER=root
export JAVA_HOME=/usr/local/jdk1.8.0_45
export CLASS_HOME=/usr/local/jdk1.8.0_45/lib:$JAVA_HOME/jre/lib
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib:$CLASSPATH
export PATH=$PATH:$JAVA_HOME/bin
```

#### 启动
PATH/bin/nexus start/stop/run(有日志输出)

#### 开机启动

ln -s PATH/bin/nexus /etc/init.d/nexus
chkconfig nexus on
chkconfig --list nexus 检查

#### 访问

默认url: ip:8081/nexus
账号: admin
密码: admin123

#### 仓库介绍

1 中央仓库
2 私服
3 其它公共库。

私服是一种特殊的远程仓库，它是架设在局域网内的仓库服务，私服代理广域网上的远程仓库，供局域网内的Maven用户使用。当Maven需要下载构件的时候，它从私服请求，如果私服上不存在该构件，则从外部的远程仓库下载，缓存在私服上之后，再为Maven的下载请求提供服务。我们还可以把一些无法从外部仓库下载到的构件上传到私服上。


Nexus 的仓库分为这么几类：

hosted 宿主仓库：主要用于部署无法从公共仓库获取的构件（如 oracle 的 JDBC 驱动）以及自己或第三方的项目构件；

proxy 代理仓库：代理公共的远程仓库；

virtual 虚拟仓库：用于适配 Maven 1；

group 仓库组：Nexus 通过仓库组的概念统一管理多个仓库，这样我们在项目中直接请求仓库组即可请求到仓库组管理的多个仓库。




#### component介绍
```
component name的一些说明：
- maven-central：maven中央库，默认从https://repo1.maven.org/maven2/拉取jar
- maven-releases：私库发行版jar
- maven-snapshots：私库快照（调试版本）jar
- maven-public：仓库分组，把上面三个仓库组合在一起对外提供服务，在本地maven基础配置settings.xml中使用。

component type的一些说明：
- hosted：类型的仓库，内部项目的发布仓库
- releases：内部的模块中release模块的发布仓库
- snapshots：发布内部的SNAPSHOT模块的仓库
- 3rd party：第三方依赖的仓库，这个数据通常是由内部人员自行下载之后发布上去
- proxy：类型的仓库，从远程中央仓库中寻找数据的仓库

```

#### 上传jar包
配置setting
```
<server>   
<id>jk_jar</id>   
<username>admin</username>
<password>123xxxxx</password>   
</server>
```

```
mvn deploy:deploy-file -DgroupId=com.jingkunsystem -DartifactId=zabbix4j -Dversion=0.1.2 -Dpackaging=jar -Dfile=/jk_jar/zabbix4j-0.1.2.jar -Durl=http://192.168.1.111:8081/repository/jk_jar/ -DrepositoryId=jk_jar


DgroupId和DartifactId构成了该jar包在pom.xml的坐标，项目就是依靠这两个属性定位。自己起名字也行。
Dfile表示需要上传的jar包的绝对路径。
Durl私服上仓库的位置，打开nexus——>repositories菜单，可以看到该路径。
DrepositoryId服务器的表示id，在nexus的configuration可以看到。
Dversion表示版本信息
解压该包，会发现一个叫MANIFEST.MF的文件，这个文件就有描述该包的版本信息。
比如Manifest-Version: 1.0可以知道该包的版本了。
```
