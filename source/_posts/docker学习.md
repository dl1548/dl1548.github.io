---
title: docker学习
date: 2017-12-15 21:15:06
tags: docker
categories:
    - linux
copyright: true
password: woshizhu
---

docker
<!--more-->

Docker三个基本概念
- 镜像（image）
- 容器（container）
- 仓库（rrepository)

#### 安装
不同系统安装方式不同，[详见](https://github.com/yeasy/docker_practice/blob/master/install/README.md)
以`centos`为例(仅支持64位kernel >=3.10)建议使用国内源
**卸载原有版本**
`yum remove docker docker-common docker-selinux docker-engine`
yum-util 提供yum-config-manager功能，另外两个是devicemapper驱动依赖
`yum install -y yum-utils device-mapper-persistent-data lvm2`
**添加yum源**
```
#中科大
sudo yum-config-manager --add-repo https://mirrors.ustc.edu.cn/docker-ce/linux/centos/docker-ce.repo
# 官方源
# sudo yum-config-manager  --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```
查看版本
` yum list docker-ce --showduplicates | sort -r`
可选择版本进行安装。默认仓库只开放稳定版
```
sudo yum install docker-ce #默认安装最新稳定版
sudo yum install docker-ce-17.12.0.ce #安装指定版本
```
然后启动并开机启动，`docker version`即可查看版本

#### 创建用户组
为了安全起见，docker只允许root和docker用户组的用户进行访问
`groupadd docker`
`usermod -G docker username`

 #### 镜像加速
若文件不存在,则手动创建
`vim /etc/docker/daemon.json`
```
#docker国内镜像加速应该是挂了
{
  "registry-mirrors": ["https://registry.docker-cn.com"]
}

#阿里云镜像加速
登陆[阿里云](https://cr.console.aliyun.com),获取专属加速地址
类似 https://xxxx.mirror.aliyuncs.com 替换上面的地址

```
systemctl daemon-reload
systemctl restart docker



#### image
获取镜像
`docker pull [选项] [ 仓库地址]  仓库名/标签` 默认从`docker hub` 中拉取
例如 ： docker pull centos #默认就是从docker hub下载最新的
如果需要下载制定tag的，可去官网查询
`docker search xxxx` 不显示tag
`docker pull centos:7.4.1708` 下载指定版本

`docker images -a `  列出镜像 (显示包括中间层镜像)
中间层镜像：是其他镜像所依赖的镜像，不能删除，否则会导致上层镜像因为依赖丢失报错，事实上也无需删除，相同的层只会存储一遍。

`docker system df`  查看镜像,容器,数据卷占用空间

`docker images -f since(或before)=image name` 列出某个image之前(之后)的镜像

`docker image prune`删除虚悬镜像(dangling)
 虚悬镜像：也就是新旧镜像重名而导致的，pull和build都可能会出现这种情况，`docker image ls -f dangling=true`可 查看，虚悬镜像都是无价值的，可删除

`docker rmi $(docker images -q -f dangling=true)`  批量删除dangling

`docker run --name webserver -d -p 80:80 nginx`  以nginx镜像为基础，运行容器，也就是新建个容器

##### 构建镜像
##### Dockerfile
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
`RUN`命令常用来安装软件包


###### 构建
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

###### 实例解析
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
###### build中的点`.`
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
或者这样理解,`CMD` 执行的是默认容器启动后执行的命令以及参数.一般`只允许使用一次CMD,常用于文件最后`

命令格式上推荐使用`exec` , 默认情况下 shell 是会被封装成为`sh -c`
如:
`CMD echo $HOME` --> `CMD ["sh", "-c", "echo $HOME"]`
注意! `exec`格式 一定要用双引号!因为在解析时会被解析成`json`

docker不是虚拟机,容器的应用也应该是以前台执行的,容器不存在后台的概念.不能以虚拟机的方式去执行后台服务等
所以 `CMD  service nginx start`肯定是错误的.会转换为 `CMD ["sh", "-c", "service nginx start"]` 显而易见,主进程是`sh`,所以这种格式的命令,会导致容器退出.
正确的方式 是`CMD ["nginx","-g","daemon off"]`

###### ENTRYPOINT
同样支持两种格式: `exec` 和`shell`

当指定了`ENTRYPOINT`后`CMD`的含义就发生了变化,它不在直接运行命令,而是将内容传送给`ENTYRPOINT`运行
同`CMD`,一个文件只写一次.


###### ENV
环境变量
`ENV KEY VALUE` `ENV KEY=VALUE` 两种方式都可以
下列指令可以支持环境变量展开：
`ADD、COPY、ENV、EXPOSE、LABEL、USER、WORKDIR、VOLUME、STOPSIGNAL、ONBUILD`
例:(node官方)
```
ENV NODE_VERSION 7.2.0

RUN curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-x64.tar.xz" \
  && curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc" \
  && gpg --batch --decrypt --output SHASUMS256.txt SHASUMS256.txt.asc \
  && grep " node-v$NODE_VERSION-linux-x64.tar.xz\$" SHASUMS256.txt | sha256sum -c - \
  && tar -xJf "node-v$NODE_VERSION-linux-x64.tar.xz" -C /usr/local --strip-components=1 \
  && rm "node-v$NODE_VERSION-linux-x64.tar.xz" SHASUMS256.txt.asc SHASUMS256.txt \
  && ln -s /usr/local/bin/node /usr/local/bin/nodejs
```

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
用来指定工作目录,WORKDIR类似命令cd。
WORKDIR参数后可以是相对路径或者带/的绝对路径，使用相对路径就依据上一个WORKDIR参数决定下面的Dockerfile工作目录。
可以重复定义，以切换Dockerfile的工作目录。

```
RUN cd  /app
RUN echo    "hello" >   world.txt
```
两个RUN 其实是运行了两个容器.第一个RUN并不会对第二个产生任何影响.所以第二个将会找不到路径,此时就需要设置工作目录 `WORKDIR /app`

###### ONBUILD
该命令实际上是个触发器,其参数是任意一个Dockerfile 指令
`ONBUILD RUN mkdir /testdir`
当我们在一个Dockerfile文件中加上ONBUILD指令，该指令对利用该Dockerfile构建镜像（比如为A镜像）不会产生实质性影响。
但是当我们编写一个新的Dockerfile文件来基于A镜像构建一个镜像（比如为B镜像）时，这时构造A镜像的Dockerfile文件中的ONBUILD指令就生效了.



###### USER
`USER username`
`USER`会改变以后命令的执行用户.或者说,他就是切换用户的.前提,用户是存在的,否则失败.

如果以 root 执行的脚本，在执行期间希望改变身份，比如希望以某个已经建立好的用户来运行某个服务进程
不要使用 su 或者 sudo，这些都需要比较麻烦的配置，而且在 TTY 缺失的环境下经常出错。建议使用 gosu。
```
# 建立 redis 用户，并使用 gosu 换另一个用户执行命令
RUN groupadd -r redis && useradd -r -g redis redis
# 下载 gosu
RUN wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/1.7/gosu-amd64" \
    && chmod +x /usr/local/bin/gosu \
    && gosu nobody true
# 设置 CMD，并以另外的用户执行
CMD [ "exec", "gosu", "redis", "redis-server" ]
```

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

###### Dockerfile多阶段构建

```
FROM muninn/glide:alpine AS build-env
ADD . /go/src/app
WORKDIR /go/src/app
RUN glide install
RUN go build -v -o /go/src/app/app-server

FROM alpine
RUN apk add -U tzdata
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai  /etc/localtime
COPY --from=build-env /go/src/app/app-server /usr/local/bin/app-server
EXPOSE 80
CMD ["app-server"]
```
首先，第一个 `FROM` 后边多了个 `AS` 关键字，可以给这个阶段起个名字。
第二部分用了官方的 alpine 镜像，改变时区到中国
注意`COPY` 关键字，它现在可以接受 --from= 这样的参数，从上个我们起名字的阶段复制文件过来。



#### container

##### 新建容器
`docker run` 此命令是用来新建容器
例如：
`docker run ubuntu /bin/echo 'test'` 会输出test后终止容器，终止容器并不是删除容器，所以容器还是存在的，添加`--rm` 则会临时性的执行，完后删除容器`docker run --rm ....`

`docker run -it ubuntu /bin/bash` 会生成一个伪终端。可通过终端进行简单的操作。同样，此容器也是存在的。docker ps -a查看

docker run流程
- 检查本地是否存在指定的镜像，不存在就从公有仓库下载
- 利用镜像创建并启动一个容器
- 分配一个文件系统，并在只读的镜像层外面挂载一层可读写层
- 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去
- 从地址池配置一个 ip 地址给容器
- 执行用户指定的应用程序
- 执行完毕后容器被终止,并不是删除，容器还可以再次启动

##### 启动容器
`docker container start -i container_id`会重新启动一个已终止的容器。

**docker container 还有很多命令，建议使用 --help查看使用说明**

##### 容器后台运行
`docker run -d`
不是用后台运行时，结果输出会到当前主机
```
[root@zili ~]# docker run ubuntu /bin/sh -c "while true; do echo test -d; sleep 1; done"
test -d
test -d
test -d
test -d
```

使用后台运行时候,容器将后台运行，那么如何查看结果呢？
```
[root@zili ~]# docker run -d ubuntu /bin/sh -c "while true; do echo test -d; sleep 1; done"
69931b53bfea873daf9cfeb82c926be84980e41a3c0f62f966b039ffbaa0b1c1
```

`docker container log container_id` 可以来查看相关的容器输出
```
[root@zili ~]# docker container logs 699
test -d
test -d
test -d
test -d
test -d
test -d
...
```
需要注意的是，容器是否能长久运行和指定的命令有关系，也就是说，命令结束，容器停止，和`-d`参数并无关系，它只是用来让容器后台运行而已。
停止容器运行使用`docker container stop container_id`

##### 进入容器
`docker attach`

不建议使用,因为使用此命令,如果容器是`-d`后台运行,stdin退出时,容器运行状态将终止.也就是说,此命令进入容器,退出时会导致后台运行的容器终止

`docker exec -it `  不会导致后台运行中的容器终止.推荐使用~!
参数说明请使用 `docker exec --help`




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

##### 其他命令
`docker diff  container name` 查看容器变化
`docker history container:tag` 查看container 历史


#### Repository
官方repository [Docker hub](https://hub.docker.com/)

`docker search` 命令查找官方repository的镜像.
`docker pull` 命令把镜像拉取下来

 `docker login` 登录docker hub
`docker logout` 退出docker hub
`docker push` 推送镜像

##### 私有仓库
[略](https://yeasy.gitbooks.io/docker_practice/content/repository/registry.html)


#### 数据管理
容器管理数据主要有两种方式,数据卷（Volumes）和挂载主机目录(Bind mounts)

##### 数据卷(Volumes)
数据卷可以在容器之间共享和重用,
数据卷 的修改会立马生效且数据卷 的更新，不会影响镜像
数据卷 默认会一直存在，即使容器被删除

- 注意：数据卷 的使用，类似于 Linux 下对目录或文件进行 mount，镜像中的被指定为挂载点的目录中的文件会隐藏掉，能显示看的是挂载的 数据卷。


数据卷有 `--mount`  `-v` 或者 `--volume` 但是推荐使用 `--mount` 参数

##### 创建/删除volume

`docker volume create webtest` 创建一个webtest的数据卷
`docker volume inspect webtest` 查看数据卷
```
[root@docker ~]# docker volume inspect webtest
[
    {
        "CreatedAt": "2018-04-15T10:54:40+08:00",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/webtest/_data",
        "Name": "webtest",
        "Options": {},
        "Scope": "local"
    }
]

```
`docker volume rm webtest` 删除数据卷,前提数据卷没有被容器使用

数据卷是独立于容器的,用来做数据初始化的,所以容器的删除不会影响数据卷,若删除容器时想要删除数据卷 则使用`docker rm -v`
清理无主的数据卷 `docker volume prune`

涉及删除的操作,`思之,慎之而行之`


##### 启动挂载数据卷的容器
`docker run`时，使用 `--mount` 标记来将数据卷挂载到容器里,在一次 docker run 中可以挂载多个数据卷

`-v后面的映射关系是"宿主机文件/目录:容器里对应的文件/目录"，其中，宿主机上的文件/目录是要提前存在的，容器里对应的文件/目录会自动创建。`

两种方式本质上没有区别,mount更加清晰直观.
` docker run -it --name web -d -p 80:80 --mount source=webtest,target=/webdir nginx`
`docker run -it --name web -d -p 80:80 -v webtest:/webdir nginx`

**查看数据卷具体信息**
`docker inspect web` 数据卷的信息在Mounts下
```
"Mounts": [
            {
                "Type": "volume",
                "Name": "webtest",
                "Source": "/var/lib/docker/volumes/webtest/_data",
                "Destination": "/webdir",
                "Driver": "local",
                "Mode": "z",
                "RW": true,
                "Propagation": ""
            }
        ],
```


`docker exec -it container /bin/bash` 可进入容器查看挂载的目录并新建文件等
```
root@f24f73d240d9:/# cd webdir/
root@f24f73d240d9:/webdir# touch 123
root@f24f73d240d9:/webdir# df -h
Filesystem              Size  Used Avail Use%   Mounted on
/dev/mapper/centos-root  50G  1.8G   49G   4%   /webdir

然后退出可在宿主机上看到此文件
[root@docker ~]# cd /var/lib/docker/volumes/webtest/_data/
[root@docker _data]# ls
123

```


##### 挂载目录(Bind mounts)
多个目录的挂载 多写几次`--mount`即可

`[root@docker web]# docker run --name dirbind -d -p 80:80 --mount type=bind,source=/root/web,target=/usr/share/nginx/html nginx`
首先我在root下新建web目录并写个index.html文件将其挂到由nginx镜像生成的容器`dirbind`的html下.

```
[root@docker web]# docker run --name dirbind -d -p 80:80 --mount type=bind,source=/root/web,target=/usr/share/nginx/html nginx
2648c6f9e8ff3660e37dc64b0483bc906e5987082224fb0f8604e2bceef2da65
[root@docker web]# docker exec -it 264 /bin/bash
#此时已进入容器
root@2648c6f9e8ff:/# cd /usr/share/nginx/html
root@2648c6f9e8ff:/usr/share/nginx/html# ls
index.html
#index文件已经挂载上来了. 再次访问nginx就能看到index的内容了.
```
查看容器信息
`docker inspect dirbind` #Mounts部分可看到挂载信息,注意这里,`RW: true`
说明是读写权限的,其实还可以挂载个只读的文件.后有说明.
```
"Mounts": [
            {
                "Type": "bind",
                "Source": "/root/web",
                "Destination": "/usr/share/nginx/html",
                "Mode": "",
                "RW": true,
                "Propagation": "rprivate"
            }
        ],

```
只读目录挂载
`docker run --name dirbind -d -p 80:80 --mount type=bind,source=/root/web,target=/usr/share/nginx/html,readonly nginx`

##### 挂载单个文件
`docker run --name bindfile -d -p 80:80 \
--mount type=bind,source=/root/web,target=/usr/share/nginx/html \
--mount type=bind,source=/root/a.html,target=/usr/share/nginx/html/index.html  nginx`
第二个`--mount`是挂载单个文件,其会覆盖挂载的第一个目录下的index文件.
- 单个文件的挂载要求容器内必须也存在此文件


#### docker网络
docker 网络分为 外部访问和容器互联两种情况
##### 外部访问

这个已经并不陌生了就是在`docker run`的时候加上`-p`参数即可指定本地端口和容器端口
一个指定端口上只可以绑定一个容器
格式有` ip:hostPort:containerPort | ip::containerPort | hostPort:containerPort`
 还可以指定端口格式(tcp或udp)`127.0.0.1:1111:1111/udp`

 - 注意:若需多个端口绑定,重复只用`-p` 即可

`docker port container_id` 可查看容器端口映射情况

使用`-P` 大写的P默认会自动分配端口进行映射.

##### 容器互联
**新建网络**
`docker network create -d bridge web-net` 新建一个`web-net`的网络
`-d` 参数指定 Docker 网络类型，有 `bridge` `overlay`。其中 `overlay` 网络类型用于 `Swarm mode`
```
[root@docker ~]# docker network create -d bridge web-net
3a06ef1bde3ea75c16afe1d3024d1a161d33c3c9499646521f30a74992607407
[root@docker ~]# docker network inspect web-net
[
    {
        "Name": "web-net",
        "Id": "3a06ef1bde3ea75c16afe1d3024d1a161d33c3c9499646521f30a74992607407",
        "Created": "2018-04-15T14:08:13.044203584+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
```
**容器关联网络**
通过`--network` 参数指定容器网络
`docker run --name web1 -d -P  --network web-net nginx`
`docker run --name web2 -d -P  --network web-net nginx`
```
[root@docker ~]# docker network inspect web-net
[
    {
        "Name": "web-net",
        "Id": "3a06ef1bde3ea75c16afe1d3024d1a161d33c3c9499646521f30a74992607407",
        "Created": "2018-04-15T14:08:13.044203584+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "21250caa4838de429eeda541aab9d4e6e0697a4b9038d9656e0ee318db9f2636": {
                "Name": "web2",
                "EndpointID": "46cac28abeef408daa1351bd2c376376928c82c1bb2836cb729adcd7379022ce",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            },
            "aca3aa6558188c41a2ab1f79993d845d6cea392fcc8c61d8acc0bc9864550834": {
                "Name": "web1",
                "EndpointID": "26e27856326d3bc04f29ecaf083dab789c80b0f64ce7c09159df31e896052ca6",
                "MacAddress": "02:42:ac:12:00:03",
                "IPv4Address": "172.18.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
```
进入容器中`docker exec -it /bin/bash`
ubuntu容器没有ping则安装
```
apt-get update
apt install net-tools       # ifconfig
apt install iputils-ping     # ping
```
```
root@0b8f77731024:~# ping web2
PING net2 (172.18.0.2) 56(84) bytes of data.
64 bytes from web2.web-net (172.18.0.2): icmp_seq=1 ttl=64 time=0.085 ms
64 bytes from web2.web-net (172.18.0.2): icmp_seq=2 ttl=64 time=0.043 ms

```

如果有过个容器要互联,如果你有多个容器之间需要互相连接，`Docker Compose`了解一下.

##### DNS
进入容器中执行`mount`
```
root@0b8f77731024:~# mount
/dev/mapper/centos-root on /etc/resolv.conf type xfs (rw,relatime,attr2,inode64,noquota)
/dev/mapper/centos-root on /etc/hostname type xfs (rw,relatime,attr2,inode64,noquota)
/dev/mapper/centos-root on /etc/hosts type xfs (rw,relatime,attr2,inode64,noquota)
```
可以看到,dns,主机名,hosts文件是被挂载上去的,这样只要宿主机有变更,那么容器就能立即得到更新.
同理,如果有需要可单独见文件进行挂载,并指向容器内相关文件.

**还有一种方式** 通过`/etc/docker/daemon.json`来配置,这样影响所有容器
```
{
  "dns" : [
    "114.114.114.114",
    "8.8.8.8"
  ]
}
```
如果想要手动指定的话.在`docker run`的时候可以添加参数
`-h HOSTNAME `或者 `--hostname=HOSTNAME `设定容器的主机名,但它在容器外部看不到，既不会在 docker container ls 中显示，也不会在其他的容器的 /etc/hosts 看到。
`--dns=IP_ADDRESS` 添加 DNS 服务器到容器的` /etc/resolv.conf` 中，让容器用这个服务器来解析所有不在 /etc/hosts 中的主机名。
`--dns-search=DOMAIN `设定容器的搜索域，当设定搜索域为 `.example.com `时，在搜索一个名为 host 的主机时，DNS 不仅搜索 host，还会搜索 `host.example.com`


##### 高级网络配置
[略](https://yeasy.gitbooks.io/docker_practice/content/advanced_network/)

#### Docker Compose
Docker Compose 是 Docker 官方编排（Orchestration）项目之一，负责快速的部署分布式应用
