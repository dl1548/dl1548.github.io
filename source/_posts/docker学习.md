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
暂时跳过



#### 容器相关

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







列出容器
`docker ps -a`
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
docker提供volume用于存储,commit可以将container存储层的数据保存下来成为image,换种说法就是在原有的image上加上container存储层,构成新的image

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

#### 数据卷
volume 是一个可提供给一个或者多个容器使用的特殊目录.
1. volume可在容器之间共享和重用
2. 对volume的修改,立刻生效
3. volume的更新,不会影响image
4. volume 默认一致存在,即使container被删除.

##### -v和--mount
新手应该使用--mount .经验丰富的使用-v或者--volume ,但是推荐使用--mount



##### 创建volume
```
[root@docker ~]# docker volume create hc
hc
[root@docker ~]# docker volume ls
DRIVER              VOLUME NAME
local               hc
```
查看volume信息
```
[root@docker ~]# docker volume 
create   inspect  ls       prune    rm       
[root@docker ~]# docker volume inspect hc
[
    {
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/hc/_data",
        "Name": "hc",
        "Options": {},
        "Scope": "local"
    }
]
[root@docker ~]# 
```
##### 挂载卷
使用`docker run`的时候,添加`--mount` 参数,将volume 挂载到容器.一次`docker run` 可以挂载多个`volume`
```
[root@docker ~]# docker run \ 
--name volume-test -d -p 80:80 \  #容器名字
--mount source=hc,target=/volume-test \ #加载数据卷hc到容器的/volume-test 下
nginx
```
```
[root@docker ~]# docker exec -it volume-test bash
root@310825d9bce9:/# df -h
Filesystem               Size  Used Avail Use% Mounted on
overlay                   45G  2.9G   42G   7% /
tmpfs                    3.9G     0  3.9G   0% /dev
tmpfs                    3.9G     0  3.9G   0% /sys/fs/cgroup
/dev/mapper/centos-root   45G  2.9G   42G   7% /volume-test
shm                       64M     0   64M   0% /dev/shm
tmpfs                    3.9G     0  3.9G   0% /sys/firmware

#创建个文件
root@310825d9bce9:/# cd /volume-test/
root@310825d9bce9:/volume-test# touch 123
#退出去卷组查看
[root@docker _data]# pwd
/var/lib/docker/volumes/hc/_data
[root@docker _data]# ls
123
```
##### 删除卷
`docker volume rm NAME`
volume是为了数据的持久化的.所以它独立于容器,容器删除,volume并不会被删除.如果需要删除容器的同时删除数据卷 添加 `-v`参数.
`docker rm -v 容器名`


##### 查看volume具体信息
```
[root@docker ~]# docker inspect volume-test 
...
"Mounts": [
            {
                "Type": "volume",
                "Name": "hc",
                "Source": "/var/lib/docker/volumes/hc/_data",
                "Destination": "/volume-test",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            }
        ],
...
```

#### 挂载主机目录
将本地目录`/root/qr` 挂载到container中的`/usr/share/nginx/html`
本地路径必须是`绝对路径`,若不存在则会自动创建
```
docker run --name dir-test -d -p 80:80 \
--mount type=bind,source=/root/qr,target=/usr/share/nginx/html \  #可添加readonly.文件挂载为只读模式
nginx
```
两个目录已可做到同步
```
[root@docker qr]# docker exec -it dir-test bash
root@dad7c498fdb4:/# cd /usr/share/nginx/html/
root@dad7c498fdb4:/usr/share/nginx/html# ls
css  fonts  images  index.html  js  test
root@dad7c498fdb4:/usr/share/nginx/html# echo '123' > test 
root@dad7c498fdb4:/usr/share/nginx/html# exit 
exit
[root@docker qr]# ls
css  fonts  images  index.html  js  test
[root@docker qr]# cat test 
123
[root@docker qr]# 
```

##### 挂载文件到主机
`
--mount type=bind,source=$HOME/.bash_history,target=/root/.bash_history
`
这样可以记录容器中的历史命令/

#### 使用网络
##### 指定/随机映射
`docker network OPTION`
docker run 的时候有用过`-p`参数.
`docker run --name webserver -d -p 80:80 nginx`
`-p` 指定的就是端口映射.其实还有个参数`-P` ,大写P的会随机映射中的一个端口到内部容器开放的网络端口中.

`-p`可多次使用,指定多个映射
```
    docker  run -d  \
                -p  5000:5000   \
                -p  3000:80 \
```

```bash
[root@docker ~]# docker run -d -P --name net-test nginx
5493456c2472597ca711a30e3a49df795d4b00986ed93e73b3899d70c295254e
[root@docker ~]# docker container ls 
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                   NAMES
5493456c2472        nginx               "nginx -g 'daemon ..."   10 seconds ago      Up 10 seconds       0.0.0.0:32768->80/tcp   net-test
```

##### 指定地址和端口的映射
默认类似`80:80`这样的映射,细心应该能发现,ip是`0.0.0.0`
指定地址映射格式如下
`docker run --name webserver -d -p 127.0.0.1:80:80 nginx`

##### 指定地址随机端口
`docker run --name webserver -d -p 127.0.0.1::80 nginx`

`UDP`标记指定端口为`UDP`端口

`docker run --name webserver -d -p 127.0.0.1:5000:5000/udp nginx`

##### 查看端口
`docker port NAMES`


#### 容器互联
通过自定义网络来连接多个容器

新建网络
`docker network create -d bridge my-net`
新建容器1
`docker run  -it --rm --name test1 --network my-net centos bash`
新建容器2
`docker run  -it --rm --name test1 --network my-net centos bash`
查看容器
```
[root@docker ~]# docker container ps
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS                    NAMES
f76c89032d7f        centos              "bash"                   53 seconds ago       Up 52 seconds                                test2
8e52a4bfce63        centos              "bash"                   About a minute ago   Up About a minute                            test1
```
互PING
```
[root@8e52a4bfce63 /]# ping test2
PING test2 (172.18.0.3) 56(84) bytes of data.
64 bytes from test2.my-net (172.18.0.3): icmp_seq=1 ttl=64 time=0.085 ms
64 bytes from test2.my-net (172.18.0.3): icmp_seq=2 ttl=64 time=0.082 ms

[root@f76c89032d7f /]# ping test1
PING test1 (172.18.0.2) 56(84) bytes of data.
64 bytes from test1.my-net (172.18.0.2): icmp_seq=1 ttl=64 time=0.049 ms
64 bytes from test1.my-net (172.18.0.2): icmp_seq=2 ttl=64 time=0.073 ms
```

#### 高级网络设置
docker 启动时候会自动创建一个`docker0`的网卡,可以理解为系统的一个桥接网卡.并会随机分配个地址给docker0,此后,容器的启动,ip地址均和docker0在同网段,
容器创建的时候会创建一个`vethxxxx`格式的网络接口.用于和`docker0`通信.`veth`之间也是互通的,这样就做到了主机和容器,容器和容器间的通讯.
把docker0 想象为一台交换机,vteh为接口,容器为设备即可.

##### 配置指南
1. 只能在docker服务启动时使用

`-h HOSTNAME or --hostname=HOSTNAME` --配置容器主机名
`--link=CONTAINER_NAME:ALIAS` --添加到另一个容器的连接,推荐使用`docker network`创建网络并指定
`--net=bridge|none|container:NAME_or_ID|host` --配置容器的桥接模式
`-p SPEC or --publish=SPEC` --映射容器端口到宿主主机
`-P or --publish-all=true|false` --映射容器所有端口到宿主主机

2. 既可以在启动服务时指定，也可以 Docker 容器启动（ docker run ）时候指定。在Docker 服务启动的时候指定则会成为默认值，后面执行 docker run 时可以覆盖设置的默认值.
`--dns=IP_ADDRESS...` --使用指定的DNS服务器
`--dns-search=DOMAIN...` --指定DNS搜索域

3. 只能在`docker run`的时候使用.
`-h HOSTNAME or --hostname=HOSTNAME` --配置容器主机名
`--link=CONTAINER_NAME:ALIAS` --添加到另一个容器的连接
`--net=bridge|none|container:NAME_or_ID|host` --配置容器的桥接模式
`-p SPEC or --publish=SPEC` --映射容器端口到宿主主机
`-P or --publish-all=true|false` --映射容器所有端口到宿主主机

##### 配置DNS

默认通过文件的挂载,来进行相关的配置
容器中通过`mount`来查看挂载信息
```
/dev/mapper/centos-root on /etc/resolv.conf type xfs (rw,relatime,attr2,inode64,noquota)
/dev/mapper/centos-root on /etc/hostname type xfs (rw,relatime,attr2,inode64,noquota)
/dev/mapper/centos-root on /etc/hosts type xfs (rw,relatime,attr2,inode64,noquota)
```
这种机制可以让主机的DNS发生变化后,容器的能及时刷新.

如果想要**手动指定**,可以`docker run`时使用如下配置
`-h HOSTNAME or --hostname=HOSTNAME`
设定容器的主机名，它会被写到容器内的 `/etc/hostname` 和 `/etc/hosts`。但它在容器外部看不到，既不会在 `docker ps` 中显示，也不会在其他的容器的 `etc/hosts` 看到。

`--dns=IP_ADDRESS`
添加 DNS 服务器到容器的 `/etc/resolv.conf `中，让容器用这个服务器来解析所有不在 `/etc/hosts` 中的主机名

`--dns-search=DOMAIN`
设定容器的搜索域，当设定搜索域为 `.example.com` 时，在搜索一个名为 host 的主机时，DNS 不仅搜索host，还会搜索 `host.example.com`  注意：如果没有上述最后 2 个选项，Docker 会默认用主机上的 `/etc/resolv.conf` 来配置容器。

##### 容器访问控制
访问控制,主要通过iptables来进行控制.

**容器访问外部网络**
容器想要访问外网,需要本地的转发支持,检查转发功能是否打开.
```
[root@docker ~]# sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 1
```
`1`为打开,`0`为关闭.需要手动打开`sysctl -w net.ipv4.ip_forward=1`

如果在启动docker服务的时候设定`--ip-forward=true`,docker会自动设定系统转发为开启`1`

**容器之间访问**
1. 容器间互联
2. iptables 放行

**访问所有端口**
当启动 Docker 服务时候，默认会添加一条转发策略到 iptables 的 FORWARD 链上。策略为通过`（ACCEPT）`还是禁止`（DROP）`取决于配置`--icc=true`（缺省值）还是` --icc=false`。当然，如果手动指定` --iptables=false` 则不会添加 iptables 规则。
可见，默认情况下，不同容器之间是允许网络互通的。如果为了安全考虑，可以在` /etc/default/docker `文件中配置` DOCKER_OPTS=--icc=false` 来禁止它。

**访问指定端口**
在通过 `-icc=false` 关闭网络访问后，还可以通过 `--link=CONTAINER_NAME:ALIAS` 选项来访问容器的开放端口。
例如，在启动 Docker 服务时，可以同时使用 icc=false --iptables=true 参数来关闭允许相互的网络访问，并让 Docker 可以修改系统中的 iptables 规则。
此时，系统中的 iptables 规则可能是类似
```
...
Chain FORWARD (policy ACCEPT)
target     prot opt source               destination
DROP   
```
之后，启动容器（docker run）时使用` --link=CONTAINER_NAME:ALIAS` 选项。Docker 会在 iptable 中为 两个容器分别添加一条 ACCEPT 规则，允许相互访问开放的端口（取决于 Dockerfile 中的 EXPOSE 行）。

当添加了` --link=CONTAINER_NAME:ALIAS` 选项后，添加了 iptables 规则。
```
...
Chain FORWARD (policy ACCEPT)
target     prot opt source               destination
ACCEPT     tcp  --  172.17.0.2           172.17.0.3           tcp spt:80
ACCEPT     tcp  --  172.17.0.3           172.17.0.2           tcp dpt:80
DROP       all  --  0.0.0.0/0            0.0.0.0/0
```
`--link=CONTAINER_NAME:ALIAS` 中的 `CONTAINER_NAME` 目前必须是 Docker 分配的名字，或使用 `-name `参数指定的名字。主机名则不会被识别。

##### 端口映射实现