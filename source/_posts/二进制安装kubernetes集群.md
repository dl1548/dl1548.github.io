---
title: 二进制安装kubernetes集群
date: 2018-09-18 18:16:41
tags: 
    - k8s
categories:
    - linux
copyright: true
password: woshizhu
---

kubernetes集群部署

<img src="/images/k8s-web-ui.png" alt="k8s-web-ui" align=center />

<!--more-->



#### 集群规划

一主二从

master:192.168.1.245(k8s)

| 组件 | 版本 | 路径 |
| ---- | :--- | ---- |
| etcd | 3.3.8 | /usr/bin |
| flannel | 0.10.0 | /opt/flannel/ |
| cni | 0.7.1 | /opt/cni/ |
|kubernetes|1.10.7|/usr/bin/kube-apiserver,kube-controller-manager,kube-scheduler|


node-1:192.168.1.161(k8s01) 
node-2:192.168.1.162(k8s02)

| 组件 | 版本 | 路径 |
| ---- | :--- | ---- |
|docker|18.03.1-ce  |/usr/bin|
| etcd | 3.3.8 | /usr/bin |
| flannel | 0.10.0 | /opt/flannel/ |
| cni | 0.7.1 | /opt/cni/ |
|kubernetes|1.10.7|/usr/bin/kubelet,kube-proxy|



#### 安装包下载

  etcd：https://github.com/coreos/etcd/releases/

  flannel：https://github.com/coreos/flannel/releases/

  cni：https://github.com/containernetworking/plugins/releases 

  kubernetes：https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.10.md#v1107

前提,docker已安装  **docker安装不赘述.**



#### 服务安装

##### 前置工作

**主机名解析是必须的! **

设置好三台主机的主机名

` hostnamectl --static set-hostname k8s/k8s01/k8s02`

修改三台机器的`/etc/hosts` 添加修改如下

```
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4 k8s
#127的修改是自己的主机名,在最后添加即可

192.168.1.245 k8s
192.168.1.161 k8s01
192.168.1.162 k8s02
```



**关闭防火墙和selinux **

**关闭SWAP分区**

如果不关闭,kubelet将无法启动

`swapoff -a` 关闭. 然后`vi /etc/fsta`

```
#/dev/mapper/centos-swap swap                    swap    defaults        0 0
# 注释此行
```

**同步时间**

```
yum -y install ntpdate
ntpdate 0.cn.pool.ntp.org
```



##### 安装etcd集群

[etcd入门](yum install iptables-services)

```
tar -zxvf etcd-v3.3.8-linux-amd64.tar.gz
cp etcd etcdctl /usr/bin

mkdir -p /var/lib/etcd /etc/etcd   #创建相关文件夹
```

以上操作,三台机器都要做.

###### etcd配置文件

三台机器都要有，略有不同,主要是两个文件

`/usr/lib/systemd/system/etcd.service` 和 `/etc/etcd/etcd.conf`

etcd集群的主从节点关系与kubernetes集群的主从节点关系不是同的

etcd的配置文件只是表示三个etcd节点,etcd集群在启动和运行过程中会选举出主节点

因此，配置文件中体现的只是三个节点etcd-i，etcd-ii，etcd-iii

配置好三个节点的配置文件后,便可以启动etcd集群了



**ndoe1**

/usr/lib/systemd/system/etcd.service

```
[Unit]
Description=Etcd Server
After=network.target
 
[Service]
Type=notify
WorkingDirectory=/var/lib/etcd/
EnvironmentFile=/etc/etcd/etcd.conf
ExecStart=/usr/bin/etcd
 
[Install]
WantedBy=multi-user.target

```

 /etc/etcd/etcd.conf

```
# [member]
# 节点名称
ETCD_NAME=etcd-i
# 数据存放位置
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
# 监听其他Etcd实例的地址
ETCD_LISTEN_PEER_URLS="http://192.168.1.245:2380"
# 监听客户端地址
ETCD_LISTEN_CLIENT_URLS="http://192.168.1.245:2379,http://127.0.0.1:2379"
 
#[cluster]
# 通知其他Etcd实例地址
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.1.245:2380"
# 初始化集群内节点地址
ETCD_INITIAL_CLUSTER="etcd-i=http://192.168.1.245:2380,etcd-ii=http://192.168.1.161:2380,etcd-iii=http://192.168.1.162:2380"
# 初始化集群状态，new表示新建
ETCD_INITIAL_CLUSTER_STATE="new"
# 初始化集群token
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster-token"
# 通知客户端地址
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.1.245:2379,http://127.0.0.1:2379"

```

**node2**

/usr/lib/systemd/system/etcd.service

```
[Unit]
Description=Etcd Server
After=network.target
 
[Service]
Type=notify
WorkingDirectory=/var/lib/etcd/
EnvironmentFile=/etc/etcd/etcd.conf
ExecStart=/usr/bin/etcd
 
[Install]
WantedBy=multi-user.target
```

 /etc/etcd/etcd.conf

```
# [member]
# 节点名称
ETCD_NAME=etcd-ii
# 数据存放位置
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
# 监听其他Etcd实例的地址
ETCD_LISTEN_PEER_URLS="http://192.168.1.161:2380"
# 监听客户端地址
ETCD_LISTEN_CLIENT_URLS="http://192.168.1.161:2379,http://127.0.0.1:2379"
 
#[cluster]
# 通知其他Etcd实例地址
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.1.161:2380"
# 初始化集群内节点地址
ETCD_INITIAL_CLUSTER="etcd-i=http://192.168.1.245:2380,etcd-ii=http://192.168.1.161:2380,etcd-iii=http://192.168.1.162:2380"
# 初始化集群状态，new表示新建
ETCD_INITIAL_CLUSTER_STATE="new"
# 初始化集群token
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster-token"
# 通知客户端地址
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.1.161:2379,http://127.0.0.1:2379"
```

**node3**

/usr/lib/systemd/system/etcd.service

```
[Unit]
Description=Etcd Server
After=network.target
 
[Service]
Type=notify
WorkingDirectory=/var/lib/etcd/
EnvironmentFile=/etc/etcd/etcd.conf
ExecStart=/usr/bin/etcd
 
[Install]
WantedBy=multi-user.target
```

 /etc/etcd/etcd.conf

```
# [member]
# 节点名称
ETCD_NAME=etcd-iii
# 数据存放位置
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
# 监听其他Etcd实例的地址
ETCD_LISTEN_PEER_URLS="http://192.168.1.162:2380"
# 监听客户端地址
ETCD_LISTEN_CLIENT_URLS="http://192.168.1.162:2379,http://127.0.0.1:2379"
 
#[cluster]
# 通知其他Etcd实例地址
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.1.162:2380"
# 初始化集群内节点地址
ETCD_INITIAL_CLUSTER="etcd-i=http://192.168.1.245:2380,etcd-ii=http://192.168.1.161:2380,etcd-iii=http://192.168.1.162:2380"
# 初始化集群状态，new表示新建
ETCD_INITIAL_CLUSTER_STATE="new"
# 初始化集群token
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster-token"
# 通知客户端地址
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.1.162:2379,http://127.0.0.1:2379"
```

**启动etcd集群**

```
systemctl daemon-reload
systemctl start etcd.service
```

**检查集群状态**

```
[root@k8s ~]# etcdctl member list
29c11a00212167fb: name=etcd-iii peerURLs=http://192.168.1.162:2380 clientURLs=http://127.0.0.1:2379,http://192.168.1.162:2379 isLeader=false
393b6040645f784e: name=etcd-ii peerURLs=http://192.168.1.161:2380 clientURLs=http://127.0.0.1:2379,http://192.168.1.161:2379 isLeader=false
9d56a160db6a1fd1: name=etcd-i peerURLs=http://192.168.1.245:2380 clientURLs=http://127.0.0.1:2379,http://192.168.1.245:2379 isLeader=true

[root@k8s ~]# etcdctl cluster-health
member 29c11a00212167fb is healthy: got healthy result from http://127.0.0.1:2379
member 393b6040645f784e is healthy: got healthy result from http://127.0.0.1:2379
member 9d56a160db6a1fd1 is healthy: got healthy result from http://127.0.0.1:2379
cluster is healthy
[root@k8s ~]#
```

##### 安装flannel

集群机器均需操作

flannel服务依赖etcd，必须先安装好etcd，并配置etcd服务地址-etcd-endpoints

-etcd-prefix是etcd存储的flannel网络配置的键前缀

```
mkdir -p /opt/flannel/bin/
tar -xzvf flannel-v0.10.0-linux-amd64.tar.gz -C /opt/flannel/bin/
```

###### flannel配置文件

/usr/lib/systemd/system/flannel.service

```
[Unit]
Description=Flanneld overlay address etcd agent
After=network.target
After=network-online.target
Wants=network-online.target
After=etcd.service
Before=docker.service
 
[Service]
Type=notify
ExecStart=/opt/flannel/bin/flanneld -etcd-endpoints=http://192.168.1.245:2379,http://192.168.1.161:2379,http://192.168.1.162:2379 -etcd-prefix=coreos.com/network
ExecStartPost=/opt/flannel/bin/mk-docker-opts.sh -d /etc/docker/flannel_net.env -c
Restart=on-failure
 
[Install]
WantedBy=multi-user.target
RequiredBy=docker.service

```

执行一下命令设置flannel网络配置.(ip等信息可修改)

```
etcdctl mk /coreos.com/network/config '{"Network":"172.18.0.0/16", "SubnetMin": "172.18.1.0", "SubnetMax": "172.18.254.0",  "Backend": {"Type": "vxlan"}}'

#删除  etcdctl rm /coreos.com/network/config
```

###### 下载flannel

flannel服务依赖flannel镜像，所以要先下载flannel镜像，执行以下命令从阿里云下载，并创建镜像tag：

```
docker pull registry.cn-beijing.aliyuncs.com/k8s_images/flannel:v0.10.0-amd64
docker tag registry.cn-beijing.aliyuncs.com/k8s_images/flannel:v0.10.0-amd64 quay.io/coreos/flannel:v0.10.0
```

###### 配置docker

flannel配置中有一项

`ExecStartPost=/opt/flannel/bin/mk-docker-opts.sh -d /etc/docker/flannel_net.env -c`

flannel启动后执行mk-docker-opts.sh，并生成/etc/docker/flannel_net.env文件

flannel会修改docker网络,flannel_net.env是flannel生成的docker配置参数,因此,还要修改docker配置项



/usr/lib/systemd/system/docker.service

```
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
# After=network-online.target firewalld.service
After=network-online.target flannel.service
Wants=network-online.target
 
[Service]
Type=notify
# the default is not to use systemd for cgroups because the delegate issues still
# exists and systemd currently does not support the cgroup feature set required
# for containers run by docker
EnvironmentFile=/etc/docker/flannel_net.env
#ExecStart=/usr/bin/dockerd $DOCKER_OPTS
ExecStart=/usr/bin/dockerd
ExecReload=/bin/kill -s HUP $MAINPID
ExecStartPost=/usr/sbin/iptables -P FORWARD ACCEPT
# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
# Uncomment TasksMax if your systemd version supports it.
# Only systemd 226 and above support this version.
#TasksMax=infinity
TimeoutStartSec=0
# set delegate yes so that systemd does not reset the cgroups of docker containers
Delegate=yes
# kill only the docker process, not all processes in the cgroup
KillMode=process
# restart the docker process if it exits prematurely
Restart=on-failure
StartLimitBurst=3
StartLimitInterval=60s
 
[Install]
WantedBy=multi-user.target

```

After：flannel启动之后再启动docker

EnvironmentFile：配置docker的启动参数，由flannel生成

ExecStart：增加docker启动参数

ExecStartPost：在docker启动之后执行,会修改主机的iptables路由规则。    



###### 启动

```
systemctl daemon-reload
systemctl start flannel.service
systemctl restart docker.service
```



##### CNI配置

集群机器均需操作

CNI(Container Network Interface)容器网络接口，是Linux容器网络配置的一组标准和库，用户需要根据这些标准和库来开发自己的容器网络插件

```
mkdir -p /opt/cni/bin /etc/cni/net.d
tar -xzvf cni-plugins-amd64-v0.7.1.tgz -C /opt/cni/bin
```

/etc/cni/net.d/10-flannel.conflist

```
{
  "name":"cni0",
  "cniVersion":"0.3.1",
  "plugins":[
    {
      "type":"flannel",
      "delegate":{
        "forceAddress":true,
        "isDefaultGateway":true
      }
    },
    {
      "type":"portmap",
      "capabilities":{
        "portMappings":true
      }
    }
  ]
}
```



##### 安装k8s集群

###### CA证书



证书生成漏了一个,然后后续补上的.命令并不一定正确!!!  还未验证

建议去网上搜索其他ca生成教程.



|证书用途| 命名 |
| ------------------------------------------------ | -------------------------------------- |
| 根证书和私钥                                     | ca.crt、ca.key                         |
| kube-apiserver证书和私钥                         | server.crt、server.key                 |
| kube-controller-manager/kube-scheduler证书和私钥 | cs_client.crt、cs_client.key           |
| kubelet/kube-proxy证书和私钥                     | kubelet_client.crt、kubelet_client.key |

**master**

**创建证书目录**

`mkdir -p /etc/kubernetes/ca`

**生成根证书和私钥**

```
 openssl genrsa -out ca.key 2048
 openssl req -x509 -new -nodes -key ca.key -subj "/CN=k8s" -days 5000 -out ca.crt
 
 #/CN=主机名
```

**生成kube-apiserver证书和私钥**

新建master_ssl.conf

```
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = kubernetes
DNS.2 = kubernetes.default
DNS.3 = kubernetes.default.svc
DNS.4 = kubernetes.default.svc.cluster.local
DNS.5 = k8s                     #master hostname
IP.1 = 172.18.0.1                   #master clusterip 可通过kubectl get service获取
IP.2 = 192.168.1.245                #master ip

```

生成

这里的生成经网友提醒后添加的,刚开始忘记写了.时间悠久也忘记了当时的命令

所以这个生成命令可能并不正确.- .. 有空再去验证记录把,这里记录下.

```
# 生成 apiserver 私钥
openssl genrsa -out server.key 2048
# 生成签署请求
openssl req -new -key server.key -out server.csr -subj "/CN=k8s" -days 5000 -config master_ssl.conf
# 使用自建 CA 签署
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -days 5000 -extensions v3_req -extfile master_ssl.cnf

 #/CN=主机名
```





**生成kube-controller-manager/kube-scheduler证书和私钥**

```
openssl genrsa -out cs_client.key 2048
openssl req -new -key cs_client.key -subj "/CN=k8s" -out cs_client.csr
openssl x509 -req -in cs_client.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out cs_client.crt -days 5000

 #/CN=主机名
```



**node1**

其他node同! IP据情况而定

```
mkdir -p /etc/kubernetes/ca
# 将master节点的根证书和私钥拷贝到该目录下，执行以下命令生成证书和私钥：

openssl genrsa -out kubelet_client.key 2048
    openssl req -new -key kubelet_client.key -subj "/CN=192.168.1.161" -out kubelet_client.csr
    openssl x509 -req -in kubelet_client.csr  -CA ca.crt -CAkey ca.key -CAcreateserial -out kubelet_client.crt -days 5000

```



###### 安装(master)

```
#解压
tar -zxvf kubernetes-server-linux-amd64.tar.gz -C /opt

#复制bin文件
cd /opt/kubernetes/server/bin
cp -a `ls |egrep -v "*.tar|*_tag"` /usr/bin

#创建日志文件路径
mkdir -p /var/log/kubernetes
```



###### 配置kube-apiserver

 /usr/lib/systemd/system/kube-apiserver.service

```
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=etcd.service
Wants=etcd.service
 
[Service]
EnvironmentFile=/etc/kubernetes/apiserver.conf
ExecStart=/usr/bin/kube-apiserver $KUBE_API_ARGS
Restart=on-failure
Type=notify
LimitNOFILE=65536
 
[Install]
WantedBy=multi-user.target
```

/etc/kubernetes/apiserver.conf

```
KUBE_API_ARGS="\
    --storage-backend=etcd3 \
    --etcd-servers=http://192.168.1.245:2379,http://192.168.1.161:2379,http://192.168.1.162:2379 \
    --bind-address=0.0.0.0 \
    --secure-port=6443  \
    --service-cluster-ip-range=172.18.0.0/16 \
    --service-node-port-range=1-65535 \
    --kubelet-port=10250 \
    --advertise-address=192.168.1.245 \
    --allow-privileged=false \
    --anonymous-auth=false \
    --client-ca-file=/etc/kubernetes/ca/ca.crt \
    --tls-private-key-file=/etc/kubernetes/ca/server.key \
    --tls-cert-file=/etc/kubernetes/ca/server.crt \
    --enable-admission-plugins=NamespaceLifecycle,LimitRanger,NamespaceExists,SecurityContextDeny,ServiceAccount,DefaultStorageClass,ResourceQuota \
    --logtostderr=true \
    --log-dir=/var/log/kubernets \
    --v=2"

#############################
#解释说明
--etcd-servers  #连接到etcd集群
--secure-port  #开启安全端口6443
--client-ca-file、--tls-private-key-file、--tls-cert-file配置CA证书
--enable-admission-plugins  #开启准入权限
--anonymous-auth=false #不接受匿名访问,若为true,则表示接受,此处设置为false,便于dashboard访问

```



###### 配置kube-controller-manager

server 引用conf ,conf 里引用yaml



/etc/kubernetes/kube-controller-config.yaml

```
apiVersion: v1
kind: Config
users:
- name: controller
  user:
    client-certificate: /etc/kubernetes/ca/cs_client.crt
    client-key: /etc/kubernetes/ca/cs_client.key
clusters:
- name: local
  cluster:
    certificate-authority: /etc/kubernetes/ca/ca.crt
contexts:
- context:
    cluster: local
    user: controller
  name: default-context
current-context: default-context
```

/usr/lib/systemd/system/kube-controller-manager.service

```
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=kube-apiserver.service
Requires=kube-apiserver.service
 
[Service]
EnvironmentFile=/etc/kubernetes/controller-manager.conf
ExecStart=/usr/bin/kube-controller-manager $KUBE_CONTROLLER_MANAGER_ARGS
Restart=on-failure
LimitNOFILE=65536
 
[Install]
WantedBy=multi-user.target
```

/etc/kubernetes/controller-manager.conf

```
KUBE_CONTROLLER_MANAGER_ARGS="\
    --master=https://192.168.1.245:6443 \
    --service-account-private-key-file=/etc/kubernetes/ca/server.key \
    --root-ca-file=/etc/kubernetes/ca/ca.crt \
    --cluster-signing-cert-file=/etc/kubernetes/ca/ca.crt \
    --cluster-signing-key-file=/etc/kubernetes/ca/ca.key \
    --kubeconfig=/etc/kubernetes/kube-controller-config.yaml \
    --logtostderr=true \
    --log-dir=/var/log/kubernetes \
    --v=2"
    
    
#######################
master连接到master节点
service-account-private-key-file、root-ca-file、cluster-signing-cert-file、cluster-signing-key-file配置CA证书

kubeconfig是配置文件
```

###### 配置kube-scheduler

/etc/kubernetes/kube-scheduler-config.yaml

```
apiVersion: v1
kind: Config
users:
- name: scheduler
  user:
    client-certificate: /etc/kubernetes/ca/cs_client.crt
    client-key: /etc/kubernetes/ca/cs_client.key
clusters:
- name: local
  cluster:
    certificate-authority: /etc/kubernetes/ca/ca.crt
contexts:
- context:
    cluster: local
    user: scheduler
  name: default-context
current-context: default-context

```

/usr/lib/systemd/system/kube-scheduler.service

```
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=kube-apiserver.service
Requires=kube-apiserver.service
 
[Service]
User=root
EnvironmentFile=/etc/kubernetes/scheduler.conf
ExecStart=/usr/bin/kube-scheduler $KUBE_SCHEDULER_ARGS
Restart=on-failure
LimitNOFILE=65536
 
[Install]
WantedBy=multi-user.target

```

/etc/kubernetes/scheduler.conf

```
KUBE_SCHEDULER_ARGS="\
    --master=https://192.168.1.245:6443 \
    --kubeconfig=/etc/kubernetes/kube-scheduler-config.yaml \
    --logtostderr=true \
    --log-dir=/var/log/kubernetes \
    --v=2"
```

###### 启动master

```
systemctl daemon-reload
systemctl start kube-apiserver.service
systemctl start kube-controller-manager.service
systemctl start kube-scheduler.service
```

 日志查看.

```
journalctl -xeu kube-apiserver --no-pager
journalctl -xeu kube-controller-manager --no-pager
journalctl -xeu kube-scheduler --no-pager

# 实时查看加 -f
```



###### 安装(node)

server 包中已包含了节点二进制文件.

```
#解压
tar -zxvf kubernetes-server-linux-amd64.tar.gz -C /opt

#复制bin文件
cd /opt/kubernetes/server/bin
cp -a kubectl kubelet kube-proxy /usr/bin/

#创建日志文件路径
mkdir -p /var/log/kubernetes
```

touch /etc/sysctl.d/k8s.conf

```
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1

# 修改内核参数,iptables过滤规则生效.如果未用到可忽略.
# sysctl -p #配置生效
```



###### 配置kubelet

/etc/kubernetes/kubelet-config.yaml

```
apiVersion: v1
kind: Config
users:
- name: kubelet
  user:
    client-certificate: /etc/kubernetes/ca/kubelet_client.crt
    client-key: /etc/kubernetes/ca/kubelet_client.key
clusters:
- cluster:
    certificate-authority: /etc/kubernetes/ca/ca.crt
    server: https://192.168.1.245:6443
  name: local
contexts:
- context:
    cluster: local
    user: kubelet
  name: default-context
current-context: default-context
preferences: {}
```

/usr/lib/systemd/system/kubelet.service

```
[Unit]
Description=Kubelet Server
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=docker.service
Requires=docker.service
 
[Service]
EnvironmentFile=/etc/kubernetes/kubelet.conf
ExecStart=/usr/bin/kubelet $KUBELET_ARGS
Restart=on-failure
 
[Install]
WantedBy=multi-user.target
```

/etc/kubernetes/kubelet.conf

```
KUBELET_ARGS="\
    --kubeconfig=/etc/kubernetes/kubelet-config.yaml \
    --pod-infra-container-image=registry.aliyuncs.com/archon/pause-amd64:3.0 \
    --hostname-override=192.168.1.161 \
    --network-plugin=cni \
    --cni-conf-dir=/etc/cni/net.d \
    --cni-bin-dir=/opt/cni/bin \
    --logtostderr=true \
    --log-dir=/var/log/kubernetes \
    --v=2"


###################
--hostname-override  #配置node名称 建议使用node节点的IP
#--pod-infra-container-image=gcr.io/google_containers/pause-amd64:3.0 \
--pod-infra-container-image #指定pod的基础镜像 默认是google的,建议改为国内,或者FQ
或者 下载到本地重新命名镜像

docker pull registry.cn-hangzhou.aliyuncs.com/google-containers/pause-amd64:3.0
docker tag registry.cn-hangzhou.aliyuncs.com/google-containers/pause-amd64:3.0 gcr.io/google_containers/pause-amd64:3.0

--kubeconfig #为配置文件
```



###### 配置kube-proxy

/etc/kubernetes/proxy-config.yaml

```
apiVersion: v1
kind: Config
users:
- name: proxy
  user:
    client-certificate: /etc/kubernetes/ca/kubelet_client.crt
    client-key: /etc/kubernetes/ca/kubelet_client.key
clusters:
- cluster:
    certificate-authority: /etc/kubernetes/ca/ca.crt
    server: https://192.168.1.245:6443
  name: local
contexts:
- context:
    cluster: local
    user: proxy
  name: default-context
current-context: default-context
preferences: {}
```

/usr/lib/systemd/system/kube-proxy.service

```
[Unit]
Description=Kube-Proxy Server
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=network.target
Requires=network.service
 
[Service]
EnvironmentFile=/etc/kubernetes/proxy.conf
ExecStart=/usr/bin/kube-proxy $KUBE_PROXY_ARGS
Restart=on-failure
LimitNOFILE=65536
 
[Install]
WantedBy=multi-user.target
```

/etc/kubernetes/proxy.conf

```
KUBE_PROXY_ARGS="\
    --master=https://192.168.1.245:6443 \
    --hostname-override=192.168.1.161 \
    --kubeconfig=/etc/kubernetes/proxy-config.yaml \
    --logtostderr=true \
    --log-dir=/var/log/kubernetes \
    --v=2"
    
######################
--hostname-override  #配置node名称,要与kubelet对应，kubelet配置了，则kube-proxy也要配置
--master  #连接master服务
--kubeconfig #为配置文件
```



###### 启动node

```
systemctl daemon-reload
systemctl start kubelet.service
systemctl start kube-proxy.service
```

日志查看

```
journalctl -xeu kubelet --no-pager
journalctl -xeu kube-proxy --no-pager

# 实时查看加 -f
```



###### 查看节点

全部安装完成后.可在master上查看node

```
[root@k8s bin]# kubectl get nodes
NAME            STATUS    ROLES     AGE       VERSION
192.168.1.161   Ready     <none>    55m       v1.10.7
192.168.1.162   Ready     <none>    15s       v1.10.7
```



##### 集群测试

配置nginx 测试文件 (master)

nginx-rc.yaml

```
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx-rc
  labels:
    name: nginx-rc
spec:
  replicas: 2
  selector:
    name: nginx-pod
  template:
    metadata:
      labels: 
        name: nginx-pod
    spec:
      containers:
      - name: nginx
        image: nginx
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80

```

nginx-svc.yaml 

```
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  labels: 
    name: nginx-service
spec:
  type: NodePort
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
    nodePort: 30081
  selector:
    name: nginx-pod
```

启动

```
kubectl create -f nginx-rc.yaml
kubectl create -f nginx-svc.yaml


#查看pod创建情况：
kubectl get pod -o wide

http://node-ip:30081/

```

##### WEB UI

###### 下载dashboard yaml

```
wget https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
```
修改文件 kubernetes-dashboard.yaml

```
image 那里 要修改下.默认的地址被墙了

#image: k8s.gcr.io/kubernetes-dashboard-amd64:v1.10.0
image: mirrorgooglecontainers/kubernetes-dashboard-amd64:v1.8.3

```

```
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system
spec:
  ports:
    - port: 443
      targetPort: 8443
  selector:
    k8s-app: kubernetes-dashboard

##############修改后#############

kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system
spec:
  type: NodePort
  ports:
    - port: 443
      targetPort: 8443
      nodePort: 30000
  selector:
    k8s-app: kubernetes-dashboard

添加type:NodePort
暴露端口  ：30000
```

###### 创建权限控制yaml

dashboard-admin.yaml

```
apiVersion: rbac.authorization.k8s.io/v1beta1  
kind: ClusterRoleBinding  
metadata:  
  name: kubernetes-dashboard  
  labels:  
    k8s-app: kubernetes-dashboard  
roleRef:  
  apiGroup: rbac.authorization.k8s.io  
  kind: ClusterRole  
  name: cluster-admin  
subjects:  
- kind: ServiceAccount  
  name: kubernetes-dashboard  
  namespace: kube-system 
```



创建

```
kubectl create -f kubernetes-dashboard.yaml
kubectl create -f dashboard-admin.yaml
```



查看

```
[root@k8s test]# kubectl  get pods --all-namespaces -o wide
NAMESPACE     NAME                                    READY     STATUS    RESTARTS   AGE       IP             NODE
kube-system   kubernetes-dashboard-66c9d98865-78r5h   1/1       Running   0          6s        172.18.100.7   192.168.1.161
```

访问

成功后可以直接访问 https://NODE_IP:配置的端口 访问

我这边是 https://192.168.1.162:30000



访问会提示登录.我们采取token登录

```
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}') | grep token
```

 ```
[root@k8s test]# kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}') | grep token
Name:         admin-user-token-rlmgn
Type:  kubernetes.io/service-account-token
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9..(很多字符)

####
#将这些字符复制到前端登录即可.
 ```

