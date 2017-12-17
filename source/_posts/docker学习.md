---
title: docker学习(未完待续)
date: 2017-12-15 21:15:06
tags: docker
categories:
    - linux
copyright: true
password: woshizhu
---

docker
<!--more-->

#### image
docker images 
-a (显示包括中间层镜像)

列出某个image之前(之后)的镜像
`docker images -f since(或before)=image name`

删除虚悬镜像(dangling)
`docker image prune`
批量删除dangling
`docker rmi $(docker images -q -f dangling=true)`

运行容器,以某个image为基础
`docker run --name webserver -d -p 80:80 nginx`

#### 容器相关
列出容器
`docker ps`
进入
`docker exec option`
进入容器,并生成伪终端,执行bash命令
`docker exec -it webserver bash`
查看容器变化
`docker diff  container name`
查看container 历史
`docker history container:tag`

#### docker commit 
> 慎用!后有说明(制作镜像推荐使用 Dockerfile)
docker提供volume用于存储,commit可以将container存储层的数据保存下来成为image,换种说法就是在原有的image上加上container存储,构成新的image

语法:
`docker commit [option] <container ID or name> [<repository >[:<tag>]]`
指定了作者和信息,可省略
`docker commit  --author 'lizili<lizili@wondersgroup.com>' --message 'modify index.html' webserver nginx:v2`

查看container 历史
`docker history container:tag`

以新的image 运行 container
`docker run --name  web2    -d  -p  81:80   nginx:v2`

为什么要慎用docker commit
> 如果仔细观察之前的`docker  diff    webserver` 的结果,你会发现除了真正想要修改的`index.html`文件外,由于命令的执行,还有很多文件被改动或添加了。这还仅仅是最简单的操作,如果是安装软件包、编译构建,那会有大量的无关内容被添加进来,如果不小心清理,将会导致镜像极为臃肿。
此外,使用    `docker    commit`意味着所有对镜像的操作都是黑箱操作,生成的镜像也被称为黑箱镜像,换句话说,就是除了制作镜像的人知道执行过什么命令、怎么生成的镜像,别人根本无从得知。而且,即使是这个制作镜像的人,过一段时间后也无法记清具体在操作的。虽然`docker diff`   或许可以告诉得到一些线索,但是远远不到可以确保生成一致镜像的地步。这种黑箱镜像的维护工作是非常痛苦的.
而且,回顾之前提及的镜像所使用的分层存储的概念,除当前层外,之前的每一层都是不会发生改变的,换句话说,任何修改的结果仅仅是在当前层进行标记、添加、修改,而不会改动上一层。如果使用`docker commit ` 制作镜像,以及后期修改的话,每一次修改都会让镜像
更加臃肿一次,所删除的上一层的东西并不会丢失,会一直如影随形的跟着这个镜像,即使根本无法访问到。这会让镜像更加臃肿

#### Dockerfile
```
$   mkdir   mynginx
$   cd  mynginx
$   touch   Dockerfile
cat Dockerfile
FROM nginx
RUN echo '<h1>Dockerfile</h1>' >    > /usr/share/nginx/html/index.html
```
`FROM image` 
必写,定制镜像也是以镜像为基础的.这里就是选定一个基础镜像,可以以相关服务镜像为基础(nginx,mysql等),也可是是系统(ubuntu,centos等).
另外,dicker还提供`FROM scratch` 意味不以任何镜像为基础.
`RUN`
shell格式 : 用来执行命令行命令.
exec格式: `RUN ["可执行文件","参数1",    "参数2"]`这像是函数调用

##### 构建
`docker build [选项] <上下文路径/URL/->`
在文件目录下执行`docker build -t nginx:v3   .`
```
[root@docker docker]# docker build -t nginx:v3 .
Sending build context to Docker daemon  2.048kB
Step 1/2 : FROM nginx
 ---> b8efb18f159b
Step 2/2 : RUN echo '<h1> Dockerfile!</h1>' >> /usr/share/nginx/html/index.html
 ---> Running in 8b921720af4c
 ---> 2cd0174a0aea
Removing intermediate container 8b921720af4c
Successfully built 2cd0174a0aea
Successfully tagged nginx:v3
```

##### 实例解析
```
FROM    debian:jessie
RUN apt-get update
RUN apt-get install -y  gcc libc6-dev   make
RUN wget    -O  redis.tar.gz    "http://download.redis.io/releases/redis-3.2.5.tar.gz"
RUN mkdir   -p  /usr/src/redis
RUN tar -xzf    redis.tar.gz    -C  /usr/src/redis  --strip-components=1
RUN make    -C  /usr/src/redis
RUN make    -C  /usr/src/redis  install
```
Dockerfile中一个RUN指令,就会建立一层.就和cimmit类似,一层层的叠加然后生成新的image.这样没有意义,很多无用的东西都装进了image.编译环境,软件包等.而且UnionFS是有层数限制的的.
所以,正确的写法应该是这样
```
FROM    debian:jessie
RUN buildDeps='gcc  libc6-dev   make'   \
                &&  apt-get update  \
                &&  apt-get install -y  $buildDeps  \
                &&  wget    -O  redis.tar.gz    "http://download.redis.io/releases/redis-3.2.5.tar.gz"  \
                &&  mkdir   -p  /usr/src/redis  \
                &&  tar -xzf    redis.tar.gz    -C  /usr/src/redis  --strip-components=1    \
                &&  make    -C  /usr/src/redis  \
                &&  make    -C  /usr/src/redis  install \
                &&  rm  -rf /var/lib/apt/lists/*    \
                &&  rm  redis.tar.gz    \
                &&  rm  -r  /usr/src/redis  \
                &&  apt-get purge   -y  --auto-remove   $buildDeps
```
一层一个目的,而不是一层一条命令,我们是在构建image并不是在写shell脚本.并且要记得做相关清理工作.
##### build中的点`.`
docker build 命令最后有个点`.`,它表示当前目录,而Dockerfile就在当前目录.所以会让人认为点`.`指的就是Dockerfile的路径,其实是不准确的.如果对照bulid格式,会发现,这里的点其实指的是上下文.
理解上下文要理解下docker工作原理,docker运行时,分为server和client.我们其实是使用client控制docker服务端,所有的操作其实是服务端完成的.也就是C/S结构.
所以我们构建image的时候,可以使用ADD.COPY等指令,将本地文件复制进镜像.构建是在服务端.服务端如何获取本地文件呢?这里就用到了上下文.用户指定了上下文路径,build的时候就会将路径下的内容打包.然后上传到Docker server.当Docker server收到后展开就能获得所需的一切文件
比如
`COPY ./package.json /app`
这并不是要复制执行docker build命令所在的目录下的package.json   ,也不是复制Dockerfile所在目录下的package.json而是复制上下文(context)目录下的p
ackage.json
所以,这里的路径是相对的,   `COPY   ../package.json /app`或者`COPY    /opt/xxxx   /app` 其实都已经超出了上下文的范围.那当我们需要context范围外的文件怎么办?很简单,先把文件cp到上下文目录中.
默认情况下,Dockerfile会将上下文目录下的Dockerfile文件作为Dockerfile.所以很多人会误认为 `.`就是指Dockerfile所在目录
通常习惯上我们也不会去更改.

##### Dockerfile其他指令
###### COPY
```
    COPY    <源路径>...    <目标路径>  
    COPY    ["<源路径1>",...   "<目标路径>"]   
```
源路径可以指定多个,也可以是还是用通配符.
此COPY 自带 -p.会保留源文件的属性.

###### ADD
基本和COPY是一致的.但是他俩的使用建议遵守一个原则
文件复制使用`COPY`,需要解压缩使用`ADD`

###### CMD
此命令类似`RUN` 支持两种格式 `shell` 和 `exec`
shell    格式: `CMD   <命令>`
exec格式: `CMD    ["可执行文件",   "参数1",  "参数2"...]`  
参数列表格式: `CMD["参数1", "参数2"...]`在指定了  `ENTRYPOINT`        指令后,用    CMD指定具体的参数。

Docker不是虚拟机,容器是进程.所以容器的启动需要指定运行程序和参数.`CMD` 就是用于指定默认容易主进程的启动命令

命令格式上推荐使用`exec` , 默认情况下 shell 是会被封装成为`sh -c`
如:
`CMD echo $HOME` --> `CMD ["sh", "-c", "echo $HOME"]`
注意! `exec`格式 一定要用双引号!因为在解析时会被解析成`json`

docker不是虚拟机,容器的应用也应该是以前台执行的,容器不存在后台的概念.不能以虚拟机的方式去执行后台服务等
所以 `CMD  service nginx start`肯定是错误的.会转换为 `CMD ["sh", "-c", "service nginx start"]` 显而易见,主进程是`sh`,所以这种格式的命令,会导致容器退出.
正确的方式 是`CMD ["nginx","-g","daemon off"]`

###### ENTRYPOINT
同样支持两种格式: `exec` 和`shell`

当指定了`ENTRYPOINT`后`CMD`的含义就发生了变化,它不在直接运行命令,而是将内容传送给`ENTYRPOINT`运行,

###### ENV
环境变量
`ENV KEY VALUE` `ENV KEY=VALUE` 两种方式都可以

###### ARG变量

`ARG key `  `ARG key=value  `
ARG指令定义的参数，在docker build命令中以--build-arg  key=value形式赋值。
ARG变量不像ENV变量始终存在于镜像中。不过ARG变量会以类似的方式对构建缓存产生影响。如果Dockerfile中定义的ARG变量的值与之前定义的变量值不一样，那么就有可能产生“cache miss”。比如RUN指令使用ARG定义的变量时，ARG变量的值变了之后，就会导致缓存失效。

##### VOLUME
`VOLUME path` `VOLUME ["path1","path2"]`
容器运行,尽量让存储层不发生写操作,通常数据都存放在卷中,此命令可实现指定挂载某目录,以防用户忘记将动态文件目录挂载为卷.这样可以保证容器的正常运行,并且不会向存储写数据

###### EXPOSE
`EXPOSE 80`
`EXPOSE 22`
声明运行容器时提供服务端口,单单只是声明,并不会真的去开启这个端口.当使用`docker run -P`时,会自动随机映射EXPOSE的端口

###### WORKDIR
指定工作目录
`WORKDIR dir`
用来指定工作目录
比如
```
RUN cd  /app
RUN echo    "hello" >   world.txt
```
两个RUN 其实是运行了两个容器.第一个RUN并不会对第二个产生任何影响.所以第二个将会找不到路径,此时就需要设置工作目录 `WORKDIR /app`

###### USER
`USER username`
`USER`会改变以后命令的执行用户.或者说,他就是切换用户的.前提,用户是存在的,否则失败.

###### HEALTHCHECK
`HEALTHCHECK [option] CMD <command>`
`HEALTHCHECK NONE` 可屏蔽基础镜像的健康检测指令

其命令和ENTTYPOINT类似.
`--interval` 间隔时间,两次检测时间间隔,默认30s
`--timeout` 超时.健康检测运行超时时间.默认30s
`--retries` 重试.默认3次.
返回值:`0`:成功,`1`:失败 `2`:保留


```
FROM    nginx
RUN apt-get update  &&  apt-get install -y  curl    &&  rm  -rf /var/lib/apt/lists/*
HEALTHCHECK --interval=15s  --timeout=5s    \
        CMD curl    -fs http://localhost/   ||  exit    1
```

#### 操作容器
`docker run` 1.13run + 版本推荐使用 `docker container run`
运行docker run的时候.后台默认会执行如下操作.
 - 检查image是否存在,不存在则去仓库下载
- 利用image 创建并启动一个container
- 分配一个文件系统,并在只读镜像外挂载一个可读写层
- 从宿主机配置的网桥接口中,桥接一个虚拟接口到container
- 从地址池分配一个地址给container
- 执行用户指定的程序
- 执行完后,终止container

##### 启动
1: 新建并启动container

运行一个容器 输出'hello docker'后,终止容器
```
[root@docker ~]# docker run ubuntu:16.04 /bin/echo 'hello docker'
hello docker
```
启动一个bash终端,并允许用户交互.
`-t` 分配一个伪终端,并绑定到container的标准输入上.
`-i` 让container的标准输入保持打开
```
[root@docker ~]# docker run -t -i ubuntu:16.04 /bin/bash
root@932cd726849d:/# pwd
/
root@932cd726849d:/# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@932cd726849d:/# 
```





2: 启动已存在的 container
`docker start` 
`docker ps -a`  可查看到容器的想关信息.我们可通过容器`NAMES`启动容器
```
[root@docker ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                      PORTS               NAMES
932cd726849d        ubuntu:16.04        "/bin/bash"              13 minutes ago      Up 5 seconds                                    awesome_pare
3d7a51ae9af8        ubuntu:16.04        "/bin/echo 'hello ..."   16 minutes ago      Exited (0) 16 minutes ago                       sad_franklin
f6d27f6015e3        helcheck            "/bin/bash"              43 hours ago        Exited (0) 43 hours ago                         web1

```
`docker container log NAMES` 可查看容器日志
```
[root@docker ~]# docker container logs sad_franklin 
hello docker
[root@docker ~]# docker container start sad_franklin
sad_franklin
#这里可以看出,我们启动一遍,就多执行了一次echo....
[root@docker ~]# docker container logs sad_franklin 
hello docker
hello docker
```

##### 后台启动
多数情况下,我们希望容器在后台运行,而不是执行完,结果输出到终端就终止了.那么可以通过 `-d` 参数来实现

```
[root@docker ~]# docker container run ubuntu:16.04 /bin/sh -c 'while true; do echo hello docker; sleep 3; done'
hello docker
hello docker
hello docker
[root@docker ~]# docker container run -d ubuntu:16.04 /bin/sh -c 'while true; do echo hello docker; sleep 3; done'
fd109346347ef4145e02acd0cc89bc4c97d4ac59e00d0c170adca406c8915869

#两次执行的结果有明显的区别.一个输出结果,一个不输出结果.

[root@docker ~]# docker container logs vigorous_aryabhata 
hello docker
hello docker
hello docker
hello docker
hello docker
...
```

##### 停止
`docker stop NAMES`  1.13+版本 `docker container stop NAMES`

启动的交互终端的容器,输入exit 或者 clrt + d 就会退出容器(终止容器)

另外,`docker restart` 命令会让容器先终止,然后再启动.


##### 进入容器
`docker attach` 

不建议使用,因为使用此命令,如果容器是-d后台运行,stdin退出时,容器运行状态将终止.

`docker exec`  参数 `-it` 

不会导致后台运行中的容器终止.

##### 导入和导出
`docker container export` 导出
```
[root@docker ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
0828d964cc2f        ubuntu:16.04        "/bin/bash"         29 minutes ago      Exited (0) 29 minutes ago                       attach
[root@docker ~]# docker container export attach > attach.tar
[root@docker ~]# ls
anaconda-ks.cfg  attach.tar  docker  myip
```
`docker image   import` 导入
```
[root@docker ~]# cat attach.tar | docker import - test/ubuntu
sha256:e9b17f2d8f7546a1d3e143f684b234ebf2301b95193782d8685d2cdb2f05b2f5
[root@docker ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
test/ubuntu         latest              e9b17f2d8f75        4 seconds ago       98.4MB
```
也可以通过URL导入
`docker import  http://example.com/exampleimage.tgz example/imagerepo`

##### 删除容器
`docker rm`
`docker rm -f` 删除运行中的容器
`docker container prune` 删除所有终止的容器

#### 访问仓库
官方repository [Docker hub](https://hub.docker.com/)

`docker search` 命令查找官方repository的镜像.
`docker pull` 命令把镜像拉取下来

 `docker login` 登录docker hub
`docker logout` 退出docker hub
`docker push` 推送镜像

重命名tag
```bash
[root@docker ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
helcheck            latest              2e024362b1d1        2 days ago          139MB
[root@docker ~]# docker tag helcheck helcheck:hc
[root@docker ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
helcheck            hc                  2e024362b1d1        2 days ago          139MB

```


