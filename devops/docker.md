#                                 Docker   

## docker 概念

  Docker 是一个开放源代码软件，是一个开放平台，用于开发应用、交付（shipping）应用、运行应用。 Docker允许用户将基础设施（Infrastructure）中的应用单独分割出来，形成更小的颗粒（容器），从而提高交付软件的速度。后期配合k8s使用很优秀.

docker 码头工人

container 容器

 ![image-20210819092530549](docker.assets/image-20210819092530549.png)

官网：https://www.docker.com/

仓库：https://hub.docker.com/    

username:xyz349925756 

password:----------

专门为容器开发的系统：photon os  fedora coreos

阿里docker镜像加速

```bash
https://cr.console.aliyun.com/cn-hangzhou/mirrors
https://c.163yun.com/dashboard?nowLang=zh#/overview
错误处理
WARNING: bridge-nf-call-iptables is disabled
WARNING: bridge-nf-call-ip6tables is disabled

[root@grafana ~]# vim /etc/sysctl.conf 
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
sysctl -p
```

![image-20210819092651883](docker.assets\image-20210819092651883.png)

阿里加速

```BASH
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://fosdg9zk.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

docker体系架构

![image-20210819092807126](docker.assets\image-20210819092807126.png)

## docker install

更新源

```bash
[root@Docker01 ~]# curl  http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -o /etc/yum.repos.d/docker-ce.repo
[root@Docker01 ~]# wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
[root@Docker01 ~]# curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
[root@Docker01 ~]# yum clean all
[root@Docker01 ~]# yum makecache
```

安装依赖包

```bash
[root@Docker01 ~]# yum install -y yum-utils device-mapper-persistent-data lvm2
[root@Docker01 ~]# yum clean all
[root@Docker01 ~]# yum makecache
```

正式安装开始

```bash
[root@Docker01 ~]# yum install -y docker-ce docker-ce-cli containerd.io
```

docker-ce-cli docker命令行工具包 

containerd.io  容器接口相关包

yum info 软件包直接查看一个包的具体作用

```bash
[root@Docker01 ~]# systemctl start docker && systemctl enable docker
[root@Docker01 ~]# docker version
[root@Docker01 ~]# docker info
Containers: 0
  Running: 0
  Paused: 0
  Stopped: 0
 Images: 0
…
Docker Root Dir: /var/lib/docker    默认的docker家目录，后期镜像也在这
 Debug Mode: false
 Registry: https://index.docker.io/v1/    默认去这个网站找docker镜像

daemon 守护进程
[root@Docker01 ~]# cat /etc/docker/daemon.json 
{
    "registry-mirrors": ["https://fosdg9zk.mirror.aliyuncs.com"]
}

[root@Docker01 ~]# systemctl daemon-reload && systemctl restart docker
```

## docker remove

```bash
#yum remove docker-ce docker-ce-cli containerd.io
#sudo rm -rf /var/lib/docker
#sudo rm -rf /var/lib/containerd
```

## 二进制安装

官方文档：https://docs.docker.com/engine/install/binaries/

下载页面：https://download.docker.com/linux/static/stable/

https://download.docker.com/linux/static/stable/x86_64/

```bash
#tar xzvf /path/to/<FILE>.tar.gz  #解压下载好的docker 文件
#cp docker/* /usr/bin/        #把docker下面的文件复制到/usr/bin
#dockerd &                   #启动docker 守护进程 很多时候可以把启动改成systemd的

[root@docker01 ~]# cat /usr/lib/systemd/system/docker.service 
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service containerd.service
Wants=network-online.target
Requires=docker.socket containerd.service

[Service]
Type=notify
# the default is not to use systemd for cgroups because the delegate issues still
# exists and systemd currently does not support the cgroup feature set required
# for containers run by docker
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
ExecReload=/bin/kill -s HUP $MAINPID
TimeoutSec=0
RestartSec=2
Restart=always

# Note that StartLimit* options were moved from "Service" to "Unit" in systemd 229.
# Both the old, and new location are accepted by systemd 229 and up, so using the old location
# to make them work for either version of systemd.
StartLimitBurst=3

# Note that StartLimitInterval was renamed to StartLimitIntervalSec in systemd 230.
# Both the old, and new name are accepted by systemd 230 and up, so using the old name to make
# this option work for either version of systemd.
StartLimitInterval=60s

# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity

# Comment TasksMax if your systemd version does not support it.
# Only systemd 226 and above support this option.
TasksMax=infinity

# set delegate yes so that systemd does not reset the cgroups of docker containers
Delegate=yes

# kill only the docker process, not all processes in the cgroup
KillMode=process
OOMScoreAdjust=-500

[Install]
WantedBy=multi-user.target

[root@docker01 ~]# cat /usr/lib/systemd/system/docker.socket 
[Unit]
Description=Docker Socket for the API

[Socket]
ListenStream=/var/run/docker.sock
SocketMode=0660
SocketUser=root
SocketGroup=docker

[Install]
WantedBy=sockets.target
#上面两个文件就是启动文件内容，参考
```

#### daemon.json 文件

```json
# daemon.json 配置介绍

 { 

 "graph": "/data/docker", 

 "storage-driver": "overlay2",  

"insecure-registries": ["registry.access.redhat.com","quay.io"], 

"registry-mirrors": ["https://q2gr04ke.mirror.aliyuncs.com"], 

 "bip": "172.24.38.1/24",  

"exec-opts": ["native.cgroupdriver=systemd"],  

"live-restore": true 

} 
```



 配置项注意点： 

graph: 该关键字未来将被弃用，可以采用 "data-root" 替代  

storage-driver: 存储驱动，即分层文件系统 

insecure-registries: 不安全的docker registries，即使用http协议推拉镜象 

 registry-mirrors: 加速站点，一般可以使用阿里、网易云、docker中国(https://registry.docker-cn.com)的地址

 bip: 指定docker bridge地址(不能以.0结尾)，生产中建议采用 172.xx.yy.1/24,其中xx.yy为宿主机ip后四位，方便定位问题 

 若启动失败，查看 /var/log/message 日志排错

daemon.josn完整例子(https://docs.docker.com/engine/reference/commandline/dockerd/#daemon-configuration-file)

```json
{
  "allow-nondistributable-artifacts": [], #把不可分配的工作推送到指定的注册中心
  "api-cors-header": "",   #在引擎API中设置CORS标头
  "authorization-plugins": [], #要加载的授权插件
  "bip": "",  #指定网桥IP
  "bridge": "",   #将容器附加到网桥
  "cgroup-parent": "",  #为所有容器设置父cgroup
  "cluster-advertise": "",  #要通告的地址或接口名称
  "cluster-store": "",  #分布式存储后端的URL
  "cluster-store-opts": {},  #设置集群存储选项（默认map []）
  "containerd": "/run/containerd/containerd.sock", #容器sock进程路径
  "containerd-namespace": "docker",  #容器namespace
  "containerd-plugin-namespace": "docker-plugins",   #container插件
  "data-root": "",    #docker运行时根路径默认 /var/lib/docker
  "debug": true,      #启用debug的模式，启用后，可以看到很多的启动信息。默认false
  "default-address-pools": [     #默认容器地址或者网段
    {
      "base": "172.80.0.0/16",
      "size": 24
    },
    {
      "base": "172.90.0.0/16",
      "size": 24
    }
  ],
  "default-cgroupns-mode": "private",  #默认cgroup
  "default-gateway": "",      #容器默认网关IPv4地址
  "default-gateway-v6": "",   #容器默认网关IPv6地址
  "default-runtime": "runc",   #容器的默认OCI运行时（默认为“ runc”）
  "default-shm-size": "64M",   
  "default-ulimits": {         #容器的默认ulimit（默认[]）
    "nofile": {
      "Hard": 64000,
      "Name": "nofile",
      "Soft": 64000
    }
  },
  "dns": [], #设定容器DNS的地址，在容器的 /etc/resolv.conf文件中可查看
  "dns-opts": [],   #容器 /etc/resolv.conf 文件，其他设置
  "dns-search": [],  #设定容器的搜索域，当设定搜索域为 .example.com 时，在搜索一个名为 host 的 主机时，DNS不仅搜索host，还会搜索host.example.com。注意：如果不设置，Docker 会默认用主机上的 /etc/resolv.conf来配置容器。
  "exec-opts": [], #运行时执行选项
  "exec-root": "",  #执行状态文件的根目录（默认为’/var/run/docker‘）
  "experimental": false, #是否开启试验性特性
  "features": {}, #特性
  "fixed-cidr": "",  #固定IP的IPv4子网
  "fixed-cidr-v6": "",   #固定IP的IPv6子网
  "group": "",    #Unix套接字的属组,仅指/var/run/docker.sock
  "hosts": [],   #设置容器hosts
  "icc": false,  #启用容器间通信（默认为true）
  "init": false,  #容器执行初始化，来转发信号或控制(reap)进程
  "init-path": "/usr/libexec/docker-init", #docker-init文件的路径
  "insecure-registries": [],  #配置docker的私库地址
  "ip": "0.0.0.0",  #绑定容器端口时的默认IP（默认0.0.0.0）
  "ip-forward": false,   #默认true, 启用 net.ipv4.ip_forward ,进入容器后使用sysctl -a|grepnet.ipv4.ip_forward查看
  "ip-masq": false, #启用IP伪装（默认为true）
  "iptables": false,  #启用iptables规则添加（默认为true）
  "ip6tables": false,  #启用ip6tables规则添加（默认为true）
  "ipv6": false,   #启用IPv6网络
  "labels": [],  #docker主机的标签，很实用的功能,例如定义：–label nodeName=host-121
  "live-restore": true, #在容器仍在运行时启用docker的实时还原
  "log-driver": "json-file",  #容器日志的默认驱动程序（默认为“ json-file”）
  "log-level": "",  #设置日志记录级别（“调试”，“信息”，“警告”，“错误”，“致命”）（默认为“信息”）
  "log-opts": {                  #日志的其他选项  容器默认日志驱动程序选项
    "cache-disabled": "false",     #日志缓存
    "cache-max-file": "5",      #最多文件数
    "cache-max-size": "20m",     #单个最大
    "cache-compress": "true",    #是否压缩默认true
    "env": "os,customer",        #用户环境
    "labels": "somelabel",       #标签
    "max-file": "5",             #最大文件数
    "max-size": "10m"           #单个10M
  },
  "max-concurrent-downloads": 3,    #设置每个请求的最大并发下载量（默认为3）
  "max-concurrent-uploads": 5,     #设置每次推送的最大同时上传数（默认为5）
  "max-download-attempts": 5,       #重试次数默认5
  "mtu": 0,                     #设置容器网络MTU
  "no-new-privileges": false,    #默认情况下，限制容器通过 suid 或 sgid 位获取附加权限。
  "node-generic-resources": [   #GPU 显卡
    "NVIDIA-GPU=UUID1",
    "NVIDIA-GPU=UUID2"
  ],
  "oom-score-adjust": -500,  #设置守护程序的oom_score_adj（默认值为-500）
  "pidfile": "",      #Docker守护进程的PID文件
  "raw-logs": false,     #全时间戳机制
  "registry-mirrors": [],  #镜像加速的地址，增加后在 docker info中可查看。
  "runtimes": {
    "cc-runtime": {
      "path": "/usr/bin/cc-runtime"
    },
    "custom": {
      "path": "/usr/local/bin/my-runc-replacement",
      "runtimeArgs": [
        "--debug"
      ]
    }
  },
  "seccomp-profile": "",
  "selinux-enabled": false, #默认 false，启用selinux支持
  "shutdown-timeout": 15,  #关闭时间
  "storage-driver": "",  #要使用的存储驱动程序
  "storage-opts": [],  #存储驱动程序选项，eg: “overlay2.override_kernel_check=true”,
                      #  “overlay2.size=15G”
  "swarm-default-advertise-addr": "",  #设置默认地址或群集广告地址的接口
  "tls": true,   #默认 false, 启动TLS认证开关
  "tlscacert": "",   #默认 ~/.docker/ca.pem，通过CA认证过的的certificate文件路径
  "tlscert": "",    #默认 ~/.docker/cert.pem ，TLS的certificate文件路径
  "tlskey": "",      #默认~/.docker/key.pem，TLS的key文件路径
  "tlsverify": true,   #默认false，使用TLS并做后台进程与客户端通讯的验证
  "userland-proxy": false, #使用userland代理进行环回流量（默认为true）
  "userland-proxy-path": "/usr/libexec/docker-proxy",
  "userns-remap": ""   #用户名称空间的用户/组设置
}
[root@docker01 ~]# man dockerd 
#可以查看到上面的信息
```

您不能在 daemon.json 中设置已经在 daemon 启动时设置为标志的选项。在使用 systemd 启动 Docker 守护进程的系统上，已经设置了-h，因此不能使用 daemon.json 中的 hosts 键来添加监听地址。https://docs.docker.com/config/daemon/systemd/

#### docker info 详解

```bash
[root@docker01 ~]# docker info
Client:                     #客户端
 Context:    default
 Debug Mode: false
 Plugins:
  app: Docker App (Docker Inc., v0.9.1-beta3)
  buildx: Build with BuildKit (Docker Inc., v0.5.1-docker)
  scan: Docker Scan (Docker Inc., v0.8.0)

Server:       #服务器端
 Containers: 1    #容器数量
  Running: 0      #正在运行的容器
  Paused: 0       #暂停的容器
  Stopped: 1      #停止的容器
 Images: 1         #镜像数量
 Server Version: 20.10.7     #docker 版本
 Storage Driver: overlay2    #存储引擎  overlay2
  Backing Filesystem: xfs     #磁盘文件格式 xfs
  Supports d_type: true    
  Native Overlay Diff: true
  userxattr: false
 Logging Driver: json-file   #logging 驱动
 Cgroup Driver: cgroupfs     #cgroups 驱动
 Cgroup Version: 1      
 Plugins:            #插件
  Volume: local     #存储：本地
  Network: bridge host ipvlan macvlan null overlay     #网络 桥接 主机ipvlan ...
  Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog
 Swarm: inactive
 Runtimes: io.containerd.runc.v2 io.containerd.runtime.v1.linux runc
 Default Runtime: runc
 Init Binary: docker-init
 containerd version: d71fcd7d8303cbf684402823e425e9dd2e99285d
 runc version: b9ee9c6314599f1b4a7f497e1f1f856fe433d3b7
 init version: de40ad0
 Security Options:
  seccomp
   Profile: default
 Kernel Version: 3.10.0-1160.el7.x86_64  #内核
 Operating System: CentOS Linux 7 (Core)   #宿主系统
 OSType: linux   #类型
 Architecture: x86_64   #64位 
 CPUs: 1    #1颗cpu
 Total Memory: 3.682GiB   #内存
 Name: docker01    #主机名
 ID: W5S6:QIXO:2VNW:E7AW:CF4Q:EAJC:6TRL:X3LP:2W22:NL6J:6PPS:FI34
 Docker Root Dir: /var/lib/docker   #主目录
 Debug Mode: false     #debug 模式关闭
 Registry: https://index.docker.io/v1/    #镜像仓库默认hub.docker.com
 Labels:   #标签
 Experimental: false
 Insecure Registries:   
  127.0.0.0/8
 Live Restore Enabled: false
```

## docker 镜像管理

docker镜像是一个典型的分层结构

- 只有最上面一层是可写的 其他都是只读的固化到镜像的

- 每次推送都是增量的

镜像名称的结构

```shell
${registry_ name}/${repository. name}/${image. name}:${tag. name}
```

例如:

```shell
docker.io/library/alpine:3.10.1
```







## docker ubuntu

```sh
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

$ echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  
 sudo apt-get update
 sudo apt-get install docker-ce docker-ce-cli containerd.io
 
 service docker start
 
 #docker-compose
 apt-get install python3-pip
 pip3 install docker-compose
 
[root@G /mnt/d]# docker-compose version
docker-compose version 1.29.2, build unknown
docker-py version: 5.0.3
CPython version: 3.8.10
OpenSSL version: OpenSSL 1.1.1f  31 Mar 2020
```











## docker 常用操作

```bash
[root@Docker01 ~]# docker search centos
```

> NAME          镜像名字           
>
> DESCRIPTION     镜像描述            
>
> STARS          受欢迎程度
>
> OFFICIAL        官方提供
>
> AUTOMATED      第三方提供

拉取镜像的方法

```bash
在线：
[root@Docker01 ~]# docker pull php:7.4-fpm   指定软件版本一般是debin系统安装好的
[root@Docker01 ~]# docker pull centos         默认拉取最新的
```

![image-20210819093122701](docker.assets\image-20210819093122701.png)

```bash
离线
把docker save  打包好的镜像上传到服务器上
[root@Docker01 ~]# ls /server/images/ -l
total 936160
-rw-r--r--. 1 root root 407502336 Apr 19 09:36 mariadb.tar
-rw-r--r--. 1 root root 137402880 Apr 19 09:34 nginx.tar
-rw-r--r--. 1 root root 413716992 Apr 19 09:37 php7.tar
[root@Docker01 /server/images]# docker load  -i mariadb.tar 
```

![image-20210819093148753](docker.assets\image-20210819093148753.png)

```bash
docker load < mariadb.tar  也可以恢复
```

```bash
[root@Docker01 /server/images]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
<none>       <none>    6209748b5c23   24 hours ago   401MB
[root@Docker01 /server/images]# docker tag 6209748b5c23 mariadb:mysqli
[root@Docker01 /server/images]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
mariadb      mysqli    6209748b5c23   24 hours ago   401MB
[root@Docker01 /server/images]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
php7         7.4-fpm   813ad233b3ee   24 hours ago   406MB
mariadb      mysqli    6209748b5c23   24 hours ago   401MB
nginx        1.19      bdf2cebd4597   24 hours ago   133MB
```

镜像详细信息查看

```bash
[root@docker /]# docker image inspect ubuntu:latest
[root@docker /]# docker image inspect 82f3b5f3c58f
```

![image-20210819093252765](docker.assets\image-20210819093252765.png)

### tab无法补全

yum方式可以这样安装，二进制方式需要下载tab包

```bash
[root@Docker01 /server/images]# yum install -y bash-completion
[root@Docker01 ~]# source /usr/share/bash-completion/completions/docker 
[root@Docker01 ~]# source /usr/share/bash-completion/bash_completion
```

## docker命令简述

### 生命周期

#### run

docker run 创建一个新的容器并运行

> 语法
>
> docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

```bash
OPTIONS说明：
-a stdin: 指定标准输入输出内容类型，可选 STDIN/STDOUT/STDERR 三项；
-d: 后台运行容器，并返回容器ID；
-i: 以交互模式运行容器，通常与 -t 同时使用；
-P: 随机端口映射，容器内部端口随机映射到主机的端口
-p: 指定端口映射，格式为：主机(宿主)端口:容器端口
-t: 为容器重新分配一个伪输入终端，通常与 -i 同时使用；
--name="nginx-lb": 为容器指定一个名称；
--dns 8.8.8.8: 指定容器使用的DNS服务器，默认和宿主一致；
--dns-search example.com: 指定容器DNS搜索域名，默认和宿主一致；
-h "mars": 指定容器的hostname；
-e username="ritchie": 设置环境变量；
--env-file=[]: 从指定文件读入环境变量；
--cpuset="0-2" or --cpuset="0,1,2": 绑定容器到指定CPU运行；
-m :设置容器使用内存最大值；
--net="bridge": 指定容器的网络连接类型，支持 bridge/host/none/container: 四种类型；
--link=[]: 添加链接到另一个容器；
--expose=[]: 开放一个端口或一组端口；
--volume , -v: 绑定一个卷
--rm 选项，这样在容器退出时就能够自动清理容器内部的文件系统。
显然，--rm 选项不能与 -d 同时使用（或者说同时使用没有意义），即只能自动清理 foreground 容器，不能自动清理detached容器。
注意，--rm 选项也会清理容器的匿名data volumes。
所以，执行 docker run 命令带 --rm命令选项，等价于在容器退出后，执行 docker rm -v。
```

#### start/stop/restart

docker start :启动一个或多个已经被停止的容器

docker stop :停止一个运行中的容器

docker restart :重启容器

> 语法
>
> docker start [OPTIONS] CONTAINER [CONTAINER...]
>
> docker stop [OPTIONS] CONTAINER [CONTAINER...]
>
> docker restart [OPTIONS] CONTAINER [CONTAINER...]

#### kill

docker kill :杀掉一个运行中的容器。

> docker kill [OPTIONS] CONTAINER [CONTAINER...]

OPTIONS说明：

-s :向容器发送一个信号

#### rm

docker rm ：删除一个或多个容器。

> 语法
>
> docker rm [OPTIONS] CONTAINER [CONTAINER...]

```bash
OPTIONS说明：
-f :通过 SIGKILL 信号强制删除一个运行中的容器。
-l :移除容器间的网络连接，而非容器本身。
-v :删除与容器关联的卷。
```

#### pause/unpasuse

docker pause :暂停容器中所有的进程。

docker unpause :恢复容器中所有的进程。

> 语法
>
> docker pause CONTAINER [CONTAINER...]
>
> docker unpause CONTAINER [CONTAINER...]

#### create

docker create ：创建一个新的容器但不启动它

用法同 docker run

> 语法

> docker create [OPTIONS] IMAGE [COMMAND] [ARG...]

#### exec

docker exec ：在运行的容器中执行命令

> 语法

> docker exec [OPTIONS] CONTAINER COMMAND [ARG...]

```bash
OPTIONS说明：
-d :分离模式: 在后台运行
-i :即使没有附加也保持STDIN 打开
-t :分配一个伪终端
子进程
容器操作
```

#### ps

docker ps : 列出容器

> 语法

> docker ps [OPTIONS]

```bash
OPTIONS说明：
-a :显示所有的容器，包括未运行的。
-f :根据条件过滤显示的内容。
--format :指定返回值的模板文件。
-l :显示最近创建的容器。
-n :列出最近创建的n个容器。
--no-trunc :不截断输出。
-q :静默模式，只显示容器编号。
-s :显示总的文件大小。
```

#### inspect

docker inspect : 获取容器/镜像的元数据。

> 语法
>
> docker inspect [OPTIONS] NAME|ID [NAME|ID...]

```bash
OPTIONS说明：
-f :指定返回值的模板文件。
-s :显示总的文件大小。
--type :为指定类型返回JSON。
```

#### top

docker top :查看容器中运行的进程信息，支持 ps 命令参数。

> 语法
>
> docker top [OPTIONS] CONTAINER [ps OPTIONS]

容器运行时不一定有/bin/bash终端来交互执行top命令，而且容器还不一定有top命令，可以使用docker top来实现查看container中正在运行的进程。

#### attach

docker attach :连接到正在运行中的容器。

> 语法
>
> docker attach [OPTIONS] CONTAINER

要attach上去的容器必须正在运行，可以同时连接上同一个container来共享屏幕（与screen命令的attach类似）。

官方文档中说attach后可以通过CTRL-C来detach，但实际上经过我的测试，如果container当前在运行bash，CTRL-C自然是当前行的输入，没有退出；如果container当前正在前台运行进程，如输出nginx的access.log日志，CTRL-C不仅会导致退出容器，而且还stop了。这不是我们想要的，detach的意思按理应该是脱离容器终端，但容器依然运行。好在attach是可以带上--sig-proxy=false来确保CTRL-D或CTRL-C不会关闭容器。

#### events

docker events : 从服务器获取实时事件

> 语法
>
> docker events [OPTIONS]

```bash
OPTIONS说明：
-f ：根据条件过滤事件；
--since ：从指定的时间戳后显示所有事件;
--until ：流水时间显示到指定的时间为止；
```

#### logs

docker logs : 获取容器的日志

> 语法
>
> docker logs [OPTIONS] CONTAINER

```bash
OPTIONS说明：
-f : 跟踪日志输出
--since :显示某个开始时间的所有日志
-t : 显示时间戳
--tail :仅列出最新N条容器日志
```

#### wait

docker wait : 阻塞运行直到容器停止，然后打印出它的退出代码。

```bash
语法
docker wait [OPTIONS] CONTAINER [CONTAINER...]
```

#### export

docker export :将文件系统作为一个tar归档文件导出到STDOUT。

> 语法
>
> docker export [OPTIONS] CONTAINER

```bash
OPTIONS说明：
-o :将输入内容写到文件。
```

#### port

docker port :列出指定的容器的端口映射，或者查找将PRIVATE_PORT NAT到面向公众的端口。

> 语法
>
> docker port [OPTIONS] CONTAINER [PRIVATE_PORT[/PROTO]]

### 容器rootfs命令

#### commit

docker commit :从容器创建一个新的镜像。

> 语法
>
> docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]

```bash
OPTIONS说明：
-a :提交的镜像作者；
-c :使用Dockerfile指令来创建镜像；
-m :提交时的说明文字；
-p :在commit时，将容器暂停。
```

#### cp

docker cp :用于容器与主机之间的数据拷贝。

> 语法
>
> docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH|-
>
> docker cp [OPTIONS] SRC_PATH|- CONTAINER:DEST_PATH

OPTIONS说明：

-L :保持源目标中的链接

#### diff

docker diff : 检查容器里文件结构的更改。

> 语法
>
> docker diff [OPTIONS] CONTAINER

### 镜像仓库

#### login

docker login : 登陆到一个Docker镜像仓库，如果未指定镜像仓库地址，默认为官方仓库 Docker Hub

docker logout : 登出一个Docker镜像仓库，如果未指定镜像仓库地址，默认为官方仓库 Docker Hub

> 语法
>
> docker login [OPTIONS] [SERVER]
>
> docker logout [OPTIONS] [SERVER]

```bash
OPTIONS说明：
-u :登陆的用户名
-p :登陆的密码
```

#### pull

docker pull : 从镜像仓库中拉取或者更新指定镜像

> 语法
>
> docker pull [OPTIONS] NAME[:TAG|@DIGEST] 

```bash
OPTIONS说明：
-a :拉取所有 tagged 镜像
--disable-content-trust :忽略镜像的校验,默认开启
```

#### push

docker push : 将本地的镜像上传到镜像仓库,要先登陆到镜像仓库

> 语法
>
> docker push [OPTIONS] NAME[:TAG]

```bash
OPTIONS说明：
--disable-content-trust :忽略镜像的校验,默认开启
```

#### search

docker search : 从Docker Hub查找镜像

> 语法
>
> docker search [OPTIONS] TERM

```bash
OPTIONS说明：
--automated :只列出 automated build类型的镜像；
--no-trunc :显示完整的镜像描述；
-f <过滤条件>:列出收藏数不小于指定值的镜像。
```

### 本地镜像管理

#### images

docker images : 列出本地镜像。

> 语法
>
> docker images [OPTIONS] [REPOSITORY[:TAG]]

```bash
OPTIONS说明：
-a :列出本地所有的镜像（含中间映像层，默认情况下，过滤掉中间映像层）；
--digests :显示镜像的摘要信息；
-f :显示满足条件的镜像；
--format :指定返回值的模板文件；
--no-trunc :显示完整的镜像信息；
```

#### rmi

docker rmi : 删除本地一个或多少镜像。

> 语法
>
> docker rmi [OPTIONS] IMAGE [IMAGE...]

```bash
OPTIONS说明：
-f :强制删除；
--no-prune :不移除该镜像的过程镜像，默认移除；
```

#### tag

docker tag : 标记本地镜像，将其归入某一仓库。

#### build

docker build 命令用于使用 Dockerfile 创建镜像。

> 语法
>
> docker build [OPTIONS] PATH | URL | -

```bash
OPTIONS说明：
--build-arg=[] :设置镜像创建时的变量；
--cpu-shares :设置 cpu 使用权重；
--cpu-period :限制 CPU CFS周期；
--cpu-quota :限制 CPU CFS配额；
--cpuset-cpus :指定使用的CPU id；
--cpuset-mems :指定使用的内存 id；
--disable-content-trust :忽略校验，默认开启；
-f :指定要使用的Dockerfile路径；
--force-rm :设置镜像过程中删除中间容器；
--isolation :使用容器隔离技术；
--label=[] :设置镜像使用的元数据；
-m :设置内存最大值；
--memory-swap :设置Swap的最大值为内存+swap，"-1"表示不限swap；
--no-cache :创建镜像的过程不使用缓存；
--pull :尝试去更新镜像的新版本；
--quiet, -q :安静模式，成功后只输出镜像 ID；
--rm :设置镜像成功后删除中间容器；
--shm-size :设置/dev/shm的大小，默认值是64M；
--ulimit :Ulimit配置。
--squash :将 Dockerfile 中所有的操作压缩为一层。
--tag, -t: 镜像的名字及标签，通常 name:tag 或者 name 格式；可以在一次构建中为一个镜像设置多个标签。
--network: 默认 default。在构建期间设置RUN指令的网络模式
```

#### history

docker history : 查看指定镜像的创建历史。

#### save

docker save : 将指定镜像保存成 tar 归档文件。

-o 指定保存到

#### load

docker load : 导入使用 docker save 命令导出的镜像。

```bash
--input , -i : 指定导入的文件，代替 STDIN。
--quiet , -q : 精简输出信息
```

#### import

docker import : 从归档文件中创建镜像。

```bash
-c :应用docker 指令创建镜像；
-m :提交时的说明文字；
```

### info|version

#### info

docker info : 显示 Docker 系统信息，包括镜像和容器数

#### version

docker version :显示 Docker 版本信息。



## docker 运行

### 启动&常见操作

```bash
[root@Docker01 ~]#docker run -it centos:latest 
i  以交互方式运行container 与-t 同时使用
t  为container分配一个伪终端
```

![image-20210819094557959](docker.assets\image-20210819094557959.png)

![image-20210819094602725](docker.assets\image-20210819094602725.png)

```bash
[root@Docker01 ~]#docker run -d centos:latest /bin/bash -c “while true ;do echo ‘hello world’;sleep 1;done”
-d   后台运行并返回ID
-c   后面跟待完成命令  不同系统container 能执行的命令不同 
```

![image-20210819094620962](docker.assets\image-20210819094620962.png)

很多时候后台运行了会出现exitd的状态,先启动再进入子进程。

```bash
连接容器的方式
[root@Docker01 ~]#docker start  containerID
[root@Docker01 ~]#docker exec -it containeID|name /bin/bash 
[root@docker /]# docker container attach nervous_allen

查看正在运行的容器
[root@Docker01 ~]#docker ps -a
[root@Docker01 ~]#docker container ls -a
```

![image-20210819094654325](docker.assets\image-20210819094654325.png)

```bash
删除正在运行的容器
[root@Docker01 ~]#docker rm -f  `docker ps -a -q`
[root@Docker01 ~]#docker rm -f  `docker container ls -a -q`

删除不需要的镜像
[root@Docker01 ~]#docker image rm containerID

一些特殊的删除需求可以使用awk判断none的或者是exit的
[root@Docker01 ~]#docker rm $(docker container ls -f “status=exited” -q)

一些临时任务可以使用—rm一起使用，端口映射
[root@docker ~]# docker container run -it --name="oldguo_cent76"  --rm 9f38484d220f
[root@docker /]# docker run -d -p 8080:80 --name="oldguo_nginx_80" nginx:1.14

```

CTRL+P+Q 可以使容器后台运行

```bash
手工数据交互
[root@docker opt]# docker container cp index.html n1:/usr/share/nginx/html/
[root@docker opt]# docker container cp n1:/usr/share/nginx/html/50x.html ./
```

![image-20210819094747677](docker.assets\image-20210819094747677.png)

```bash
Volume实现宿主机和容器的数据共享
[root@docker opt]# mkdir -p /opt/html
[root@docker ~]# docker run -d --name="nginx_3" -p 83:80 -v /opt/html:/usr/share/nginx/html nginx
```

![image-20210819095136073](docker.assets\image-20210819095136073.png)

数据直接保存到宿主机上面。删除docker container没有什么影响

(1)宿主机模拟数据目录

```bash
mkdir -p /opt/Volume/a
mkdir -p /opt/Volume/b
touch /opt/Volume/a/a.txt
touch /opt/Volume/b/b.txt
```

![image-20210819095204326](docker.assets\image-20210819095204326.png)

(2)启动数据卷容器

![image-20210819095218200](docker.assets\image-20210819095218200.png)

```bash
[root@docker01 /etc/docker]# docker container run -it --name "nginx_volumes" -v /opt/Volume/a:/opt/a -v /opt/Volume/b:/opt/b centos 
[root@9024c9dd7447 /]# ls /opt/
a  b
```

![image-20210819095239172](docker.assets\image-20210819095239172.png)

(3)使用数据卷容器

```bash
docker run -d  -p 8085:80 --volumes-from  nginx_volumes --name "n8085"  nginx
docker run -d  -p 8086:80 --volumes-from  nginx_volumes --name "n8086"  nginx
```

![image-20210819095304693](docker.assets\image-20210819095304693.png)

作用: 在集中管理集群中,大批量的容器都需要挂载相同的多个数据卷时,可以采用数卷容器进行统一管理

![image-20210819095315685](docker.assets\image-20210819095315685.png)

### 局域网yum源

```bash
1.安装vsftpd软件
[root@docker ~]# yum install -y vsftpd

2. 启动ftp 
[root@docker ~]# systemctl enable vsftpd
[root@docker ~]# systemctl start vsftpd

3. 上传系统进行到虚拟机
[root@docker01 ~]# ls /mnt/
CentOS-7-x86_64-DVD-2003.iso  debian-10.9.0-amd64-netinst.iso  hgfs

4. 配置yum仓库
mkdir -p /var/ftp/pub/centos6.9 
mkdir -p /var/ftp/ pub/centos7.5
[root@docker mnt]# mount -o loop /mnt/CentOS-6.9-x86_64-bin-DVD1.iso  /var/ftp/centos6.9/
```

![image-20210819095420334](docker.assets\image-20210819095420334.png)

```bash
cat >/yum.repos.d/ftp_6.repo <<EOF 
[ftp]
name=ftpbase
baseurl=ftp://10.0.0.100/ pub/centos6.9
enabled=1
gpgcheck=0
EOF


cat >/yum.repos.d/ftp_7.repo <<EOF 
[ftp]
name=ftpbase
baseurl=ftp://10.0.0.100/ pub/centos7.5
enabled=1
gpgcheck=0
EOF
```

```bash
yum repo文件编辑好之后需要的操作
yum clean all
yum makecache
```

### 容器的日志查看

docker logs name|containerID

```bash
[root@Docker01 ~]#docker logs centos:latest
```

这是排除错误方法之一

### 镜像制作

方法1：commit

保存container的当前状态到image后，生成对应的image

```bash
[root@Docker01 ~]#docker commit  containerID文件.tar  new container
[root@Docker01 ~]#docker load -I image
[root@Docker01 ~]#docker tag containerID name:tag 
```

![image-20210819095537309](docker.assets\image-20210819095537309.png)

方法2：Dockefile

```bash
[root@Docker01 ~]#mkdir /docker/docker-build -p
[root@Docker01 ~]#cd /docker/docker-build
[root@Docker01 docker/docker-build]#touch Dockerfile
```

编辑Dockerfile

```bash
FROM  centos:7.9.2009    基于那个镜像
RUN  yum -y install net-tools    安装软件
ADD  template /usr/share/nginx/html/   拷贝文件到指定目录
ADD  discuz.tar.gz /usr/share/nginx/html  解压文件到指定目录
EXPOSE 80 向外暴露的端口
CMD  /usr/bin/nginx -s reload     执行的命令或者sh文件，一个Dockerfile 只可以执行一个cmd多条只会执行最后一条

ENV  设定变量 
ENV CODEDIR /var/www/html/
ENV DATADIR /data/mysql/data
ADD bbs.tar.gz ${CODEDIR}
VOLUME ["${CODEDIR}","${DATADIR}"]

ENTRYPOINT

#CMD ["/bin/bash","/init.sh"]
ENTRYPOINT ["/bin/bash","/init.sh"]

说明： 
ENTRYPOINT 可以方式，在启动容器时，第一进程被手工输入的命令替换掉，防止容器秒起秒关
```

sh文件和template discuz.tar.gz 这些文件必须在Dockerfile目录中如果是sh注意

chmod a+x xxx.sh

```bash
[root@Docker01 docker/docker-build]#docker build -t 父镜像:镜像tag Dockerfile文件所在路径
[root@Docker01 ~]#docker images
```

docker image的发布

方法1：save image 

```bash
[root@Docker01 ~]#docker save -o 导出的镜像.tar 本地镜像:tag
```

方法2：

```bash
发布到公网，这里docker、阿里云等都提供类似的服务
[root@Docker01 ~]#docker login -u username -p password -e xyz349925756@hotmail.com

登录到官网，镜像上传之后发邮件告知
[root@Docker01 ~]#docker push centos:7.9.2009
```

## docker 搭建worpress discuz

### 超级权限获取

```bash
docker run --privileged -it -d centos:7.6.1810 /usr/sbin/init #注意2点：1. --privileged 参数必须加，2. /usr/sbin/init作为容器的启动命令
docker exec -it $containerID bash #进入容器的bash命令行
systemctl #验证systemctl 命令是否可用
```

![image-20210819100156620](docker.assets\image-20210819100156620.png)

```bash
docker pull php:7.4-fpm   (注意这里巨坑拉到php 很多功能没有)
docker pull nginx 
docker pull mariadb
```

### 搭建web

```bash
php
run -itd --name php_v1 -p 9000:9000 -v /opt/volume/html:/var/www/html php:7.4-fpm
```

```bash
nginx
run -itd --name nginx_v1 -p 80:80 -v /opt/volume/html:/usr/share/nginx/html --link php_v1:php nginx
```

```bash
mariadb
docker run -d -t --name mariadb_server -p 3306:3306 -v /opt/volume/mysql:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 mariadb:latest
```

```bash
nginx
[root@docker01 ~]# docker exec -it nginx_v1 bash 子进程后期常用
root@29e4e89d66b4:/etc/nginx/conf.d# cat www.conf 
server {
    listen       80;
    server_name  www.oldboy.com;
     
    location / {
        root   /usr/share/nginx/html;
        index  index.php index.html index.htm;
        charset utf-8;
   }
    location ~ \.php$ {
        root   /usr/share/nginx/html;
        fastcgi_pass php_v1:9000; #这里是docker php 那台服务器
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME /var/www/html/$fastcgi_script_name;
        include fastcgi_params;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
root@29e4e89d66b4:/etc/nginx/conf.d# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
root@29e4e89d66b4:/etc/nginx/conf.d# service nginx reload
[ ok ] Reloading nginx: nginx.
root@29e4e89d66b4:/etc/nginx/conf.d# exit
exit
root@29e4e89d66b4:/usr/share/nginx# chmod -R 777 html/
```

```bash
mariadb

docker exec -it mariadb_server bash
root@2fbadae6a565:/# mysql -uroot -p
Enter password:
MariaDB [(none)]> create database wordpress character utf-8;
MariaDB [(none)]> grant all on wordpress.* to 'wordpress'@'localhost' identified by '123456';
Query OK, 0 rows affected (0.001 sec)
MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.001 sec)
MariaDB [(none)]> quit
Bye
```

```bash
php
docker-php-ext-install mysqli
docker-php-ext-install pdo_mysql
```

小结：

目前问题Dockerfile还没有写

html权限问题

每个镜像再生成镜像不知道还要不要改什么？

其实会了就很简单

注意这里映射的卷只是安装目录。实际中。/etc/nginx/conf.d  /var/log/nginx

/var/log/mysql 等重要目录都要映射。

![image-20210819100414702](docker.assets\image-20210819100414702.png)

```bash
启动
[root@docker01 /opt/volume/html]# docker run -itd --name php_v1 -p 9000:9000 -v /opt/volume/html:/var/www/html php7:7.4-fpm

[root@docker01 /opt/volume/html]# docker run -itd --name nginx_v1 -p 80:80 -v /opt/volume/html:/usr/share/nginx/html --link php_v1:php nginx:1.19

[root@docker01 /opt/volume/html]# docker run -d -t --name mariadb_server -p 3306:3306 -v /opt/volume/mysql:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 mariadb:mysqli 


[root@docker01 /opt/volume/html]# docker ps -a 
CONTAINER ID   IMAGE            COMMAND                  CREATED              STATUS              PORTS                                       NAMES
b473e7651df3   mariadb:mysqli   "docker-entrypoint.s…"   6 seconds ago        Up 5 seconds        0.0.0.0:3306->3306/tcp, :::3306->3306/tcp   mariadb_server
b723be998eec   nginx:1.19       "/docker-entrypoint.…"   39 seconds ago       Up 37 seconds       0.0.0.0:80->80/tcp, :::80->80/tcp           nginx_v1
d792d660a786   php7:7.4-fpm     "docker-php-entrypoi…"   About a minute ago   Up About a minute   0.0.0.0:9000->9000/tcp, :::9000->9000/tcp   php_v1
```



![image-20210819100449417](docker.assets\image-20210819100449417.png)

![image-20210819100455808](docker.assets\image-20210819100455808.png)

## docker 私有仓库

创建本地仓库私人使用使用官网提供的工具docker-registry

```bash
[root@Docker01 ~]# docker pull registry
[root@Docker01 ~]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED      SIZE
php7         7.4-fpm   813ad233b3ee   2 days ago   406MB
mariadb      mysqli    6209748b5c23   2 days ago   401MB
nginx        1.19      bdf2cebd4597   2 days ago   133MB
registry     latest    1fd8e1b0bb7e   5 days ago   26.2MB
[root@Docker01 ~]# mkdir /opt/registry
创建一个registry存放目录
[root@Docker01 ~]# docker run -d -p 5000:5000 -v /opt/registry:/var/lib/registry registry:latest
启动registry
```

![image-20210819100525599](docker.assets\image-20210819100525599.png)

![image-20210819100531310](docker.assets\image-20210819100531310.png)

![image-20210819100536918](docker.assets\image-20210819100536918.png)

```bash
[root@Docker01 ~]# vim /etc/docker/daemon.json 
{
    "registry-mirrors": ["https://fosdg9zk.mirror.aliyuncs.com"],
    "insecure-registries":["192.168.1.120:5000"]                                                                          
}
[root@Docker01 ~]# systemctl daemon-reload && systemctl restart docker             
[root@Docker01 ~]# docker tag nginx:1.19 192.168.1.120:5000/nginx:1.19 
注意：上传到私有仓库必须添加IP端口信息。
```

上传遇到错误

```bash
[root@Docker01 ~]# docker push 192.168.1.120:5000/nginx
Using default tag: latest
The push refers to repository [192.168.1.120:5000/nginx]
Get http://192.168.1.120:5000/v2/: dial tcp 192.168.1.120:5000: connect: connection refused
[root@Docker01 ~]# getenforce 
Disabled
[root@Docker01 ~]# systemctl status firewalld
[root@Docker01 ~]# docker ps -a
CONTAINER ID   IMAGE             COMMAND                  CREATED          STATUS                     PORTS     NAMES
2e64e1ed8468   registry:latest   "/entrypoint.sh /etc…"   16 minutes ago   Exited (2) 7 minutes ago             dreamy_rhodes
原因就是docker 服务启动的时候
docker container registry 也停止了？
[root@Docker01 ~]# docker rm 2e64e1ed8468
2e64e1ed8468
删除重新创建
[root@Docker01 ~]# docker run -d -p 5000:5000 --restart=always -v /opt/registry:/var/lib/registry registry:latest    标红参数意识是开启服务自启容器跟随启动
[root@Docker01 ~]# docker push 192.168.1.120:5000/nginx:1.19
The push refers to repository [192.168.1.120:5000/nginx]
0974233efa7a: Pushed 
64ee8c6d0de0: Pushed 
974e9faf62f1: Pushed 
15aac1be5f02: Pushed 
23c959acc3d0: Pushed 
4dc529e519c4: Pushed 
7e718b9c0c8c: Pushed 
1.19: digest: sha256:53febd22382b6be8aa691d1fa1944b7465b3ae75203e790b0f02ab6ad913c166 size: 1778
上传成功
```

![image-20210819100615080](docker.assets\image-20210819100615080.png)

![image-20210819100723757](docker.assets\image-20210819100723757.png)

这个私有仓库并不是那么好用，从上述过程看很难看出上传的镜像就那个版本的，web查看也只是一个名称用起来不是那么方便。

```bash
[root@Docker01 ~]# yum install httpd-tools -y   安装httpd-tools工具
[root@Docker01 ~]# mkdir /opt/registry-auth/ -p   创建存放用户及密码的目录
[root@Docker01 ~]# htpasswd -Bbn test 123456 >/opt/registry-auth/htpasswd 生成一个test 的用户  密码为：123456
[root@Docker01 ~]# cat /opt/registry-auth/htpasswd 
test:$2y$05$SZu0onM1wxXTDwi/mquzIOi9ma4.uyAJgT8USjJBzQFmdDP8H9v5G
用户：密码
[root@Docker01 ~]# docker rm -f `docker ps -a -q`
[root@Docker01 ~]# docker run -d -p 5000:5000 -v /opt/registry:/var/lib/registry -v /opt/registry-auth/:/auth/ --name registry-auth -e "REGISTRY_AUTH=htpasswd" -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" -e "REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd" registry
[root@Docker01 ~]# docker login 192.168.1.120:5000
Username: test
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```

### 重启docker 服务，容器全部退出

```bash
方法一：docker run  --restart=always

方法二："live-restore": true
docker server配置文件/etc/docker/daemon.json参考
{
.....
......
 "live-restore": true
}
```

## docker harbor 

官网：https://goharbor.io/

github：https://github.com/goharbor/harbor/releases

compose

https://docs.docker.com/compose/install/ 最新安装帮助

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

github:https://github.com/docker/compose/releases/

```bash
[root@Docker01 ~]# chmod +x /usr/local/bin/docker-compose
[root@Docker01 ~]# ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
[root@Docker01 ~]# docker-compose --version
docker-compose version 1.29.1, build c34c88b2
下载并上传harbor
[root@Docker01 /server/soft]# tar xf harbor-offline-installer-v2.1.5-rc1.tgz -C /opt/
[root@Docker01 /server/soft]# ll /opt/harbor/
common.sh             harbor.yml.tmpl       LICENSE               
harbor.v2.1.5.tar.gz  install.sh            prepare             
```

![image-20210819100938393](docker.assets\image-20210819100938393.png)

```bash
[root@Docker01 /opt/harbor]# mkdir /data
[root@Docker01 /opt/harbor]# ls
common.sh  harbor.v2.1.5.tar.gz  harbor.yml  harbor.yml.tmpl  install.sh  LICENSE  prepare
[root@Docker01 /opt/harbor]#cp harbor.yml.tmpl harbor.yml
[root@Docker01 /opt/harbor]#vim harbor.yml
```

### http访问

![image-20210819101001599](docker.assets\image-20210819101001599.png)

```bash
[root@Docker01 /opt/harbor]# sh install.sh
```

![image-20210819101017530](docker.assets\image-20210819101017530.png)

![image-20210819101023834](docker.assets\image-20210819101023834.png)

![image-20210819101028456](docker.assets\image-20210819101028456.png)

![image-20210819101033430](docker.assets\image-20210819101033430.png)

```bash
[root@Docker01 /opt/harbor]# vim /etc/docker/daemon.json 
{
    "registry-mirrors": ["https://fosdg9zk.mirror.aliyuncs.com"],
    "insecure-registries":["192.168.1.120:5000","192.168.1.120"]     红色的是harbor的                                                     
}
[root@Docker01 /opt/harbor]# systemctl daemon-reload && systemctl restart docker
[root@Docker01 /opt/harbor]# docker-compose stop && docker-compose start
```

注意这里harbor被上面的重启也停止了需要重新激活

![image-20210819101059431](docker.assets\image-20210819101059431.png)

```bash
[root@Docker01 /opt/harbor]# docker login 192.168.1.120
Username: admin
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store
Login Succeeded
```

![image-20210819101113958](docker.assets\image-20210819101113958.png)

![image-20210819101118208](docker.assets\image-20210819101118208.png)

### https

参考kubernetes 里面有很详细的操作

![image-20210819101157944](docker.assets\image-20210819101157944.png)

```bash
注释了
#http  
#   port 80 
https
port 443
证书路径这些都在/data下
```

```bash
生成证书的脚本
[root@docker01 /opt/harbor]# cat create_cert.sh 
#!/bin/bash

# 在该目录下操作生成证书，正好供harbor.yml使用
mkdir -p /data/cert
cd /data/cert
openssl genrsa -out ca.key 4096
openssl req -x509 -new -nodes -sha512 -days 3650 -subj "/C=CN/ST=Beijing/L=Beijing/O=example/OU=Personal/CN=reg.local.com" -key ca.key -out ca.crt
openssl genrsa -out reg.local.com.key 4096
openssl req -sha512 -new -subj "/C=CN/ST=Beijing/L=Beijing/O=example/OU=Personal/CN=reg.local.com" -key reg.local.com.key -out reg.local.com.csr

cat > v3.ext <<-EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1=reg.local.com
DNS.2=harbor
DNS.3=ks-allinone
EOF
openssl x509 -req -sha512 -days 3650 -extfile v3.ext -CA ca.crt -CAkey ca.key -CAcreateserial -in reg.local.com.csr -out reg.local.com.crt
    
openssl x509 -inform PEM -in reg.local.com.crt -out reg.local.com.cert

cp reg.local.com.crt /etc/pki/ca-trust/source/anchors/reg.local.com.crt 
update-ca-trust

```

不用上面的脚本，就看下面操作

```bash
vim /etc/hosts
127.0.0.1 后面添加这个虚拟域名home.ca server.com  自定义一个就可以。
到data目录下创建存放密钥的目录
mkdir -p /data/cert
创建密钥
openssl genrsa-out ca.key 4096
openssl req -x509 -new -nodes -key ca.key -days 10000 -out ca.crt -subj “/CN=home.ca”
openssl genrsa -out server.key 4096

openssl req -new -key server.key -subj “/CN=server.com” -out server.csr
echo ‘subjectAltName = IP:127.0.0.1,IP:192.168.1.120’ > extfile.cnf
openssl x509 -req -sha256 -days 365 -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -extfile extfile.cnf

ls
要记录ca.rct和ca.key的存放位置然后编辑harbor.yml文件中对应的路径
然后
./prepare 检测环境
./install.sh  安装
```

### 错误解决

```bash
compost 采用yum方式也可以，官网不建议
误区：没有进入/opt/harbor目录下，改目录下有docker-compose.ym 文件
docker-compose 不能执行。
出现问题没有进入带有docker-compose.ym 文件的目录很大概率报错

方法一
docker-compose -f /opt/harbor/docker-compose.yml down -v           //移除Harbor服务容器同时保留镜像数据/数据库
docker-compose -f /usr/local/harbor/docker-compose.yml up -d             //重启私库

方法二
cd /opt/harbor/             ##目录跳转
docker-compose up -d              ##重启私库
docker-compose down -v            ##移除Harbor服务容器
```

## docker 网络

```bash
开启网络转发功能
[root@Docker01 ~]#vim /etc/sysctl.conf
net.ipv4.ip_forward = 1
[root@Docker01 ~]#sysctl -p 设置生效
[root@Docker01 ~]#cat /proc/sys/net/ipv4/ip_forward

指定映射(docker 会自动添加一条iptables规则来实现端口映射)
    -p hostPort:containerPort
    -p ip:hostPort:containerPort 
    -p ip::containerPort(随机端口:32768-60999)
    -p hostPort:containerPort/udp
    -p 81:80 –p 443:443
随机映射
docker run -P 80（随机端口）	 
```

![image-20210819101410211](docker.assets\image-20210819101410211.png)

### docker 静态IP地址

查看支持网络类型

```bash
[root@Docker01 ~]# docker network ls
NETWORK ID     NAME            DRIVER    SCOPE
344e133f97b5   bridge          bridge    local
a611aff4793e   harbor_harbor   bridge    local
3e5e4ed821f7   host            host      local
b67fd6c0fc61   none            null      local
```

> docker run --network=xxx 启动时候格式
>
> none : 无网络模式
>
> bridge ： 默认模式，相当于NAT
>
> host : 公用宿主机Network NameSapce
>
> container：与其他容器公用Network Namespace
>
> --network **network**       连接容器的网络
>
> --network-alias **list**      容器的网络别名

桥接模式中pipework 脚本分配固定IP但是重启之后IP会消失

配置桥接网络（违背docker的安全隔离原则）以下内容来自腾讯课堂**作为一种方法**

```bash
创建桥接设备br0

[root@xuegod63 ~]# rpm ivh /mnt/Packages/bridge-utils 1.5 9.el7.x86_64.rpm

把ens33 绑到 br0 桥设备上：
[root@xuegod63 ~]# cd /etc/sysconfig/network-scripts/
[root@xuegod63 network-ifcfgscripts]# cp ifcfg--ens33 /opt/ens33 /opt/ ##备份一下备份一下eth0
[root@xuegod63 network--scripts]# vim ifcfgfg--ens33 #编辑配置文件为以下内容编辑配置文件为以下内容
IPADDR="192.168.1.63" #删除IP地址相关内容，删除4行
PREFIX="24"
GATEWAY="192.168.1.1"
DNS1="8.8.8.8"
BRIDGE="br0" #在文件最后插入这一行，表示把ens33桥接到br0上

生成桥设备br0的配置文件
[root@xuegod63 network-scripts]# vim ifcfg--br0 ##创建创建ifcfg--br0文件，并写入以下内容文件，并写入以下内容
DEVICE="br0"
NM_CONTROLLED="yes"
ONBOOT="yes"
TYPE="Bridge"
BOOTPROTO=none
IPADDR=192.168.1.63
NETMASK=255.255.255.0
GATEWAY=192.168.1.1
DNS1=114.114.114.114
注：TYPE="Bridge"
[root@xuegod63 network-scripts]# service network restartnetwork
Restarting network (via systemctl): [ 确定]

测试
[root@xuegod63 network--scripts]# ping g.cn
```

### docker 跨主机网络

docker 原生的跨主机网络 **overlay**  **maclan**

#### overlay解决方案

为支持容器跨主机通信，Docker 提供了 overlay driver，使用户可以创建基于 VxLAN 的 overlay 网络。VxLAN 可将二层数据封装到 UDP 进行传输，VxLAN 提供与 VLAN 相同的以太网二层服务，但是拥有更强的扩展性和灵活性。

Docerk overlay 网络需要一个 key-value 数据库用于保存网络状态信息，包括 Network、Endpoint、IP 等。Consul、Etcd 和 ZooKeeper 都是 Docker 支持的 key-vlaue 软件，这里使用 Consul。

实验环境

| **docker01**      | **docker02**  | **docker03**  |
| ----------------- | ------------- | ------------- |
| **192.168.1.120** | 192.168.1.100 | 192.168.1.120 |
| **consul**        | node1         | node2         |

firewalld,selinux 都关闭的实验环境

docker01

```bash
[root@Docker01 ~]# docker search consul
[root@Docker01 ~]# docker pull progrium/consul
```

![image-20210819101616732](docker.assets\image-20210819101616732.png)

```bash
[root@docker01 ~]# docker run -d -p 8500:8500 -h consul --name consul --restart always progrium/consul -server -bootstrap
命令解释：-d：后台运行；-p 8500:8500：端口映射；-h consu：主机名（注意不是容器名）；--name consul：容器名；--restart always：跟随docker的启动而启动容器；-server：server端；-bootstrap：以单节点运行；
```

![image-20210819101637624](docker.assets\image-20210819101637624.png)

node01/02

```bash
[root@docker02 ~]# vim /usr/lib/systemd/system/docker.service
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --cluster-store=consul://192.168.1.120:8500 --cluster-advertise=eth0:2376     
[root@docker02 ~]# systemctl daemon-reload && systemctl restart docker
开启网卡混杂模式
[root@docker02 ~]# ifconfig eth0 promisc
```

node01

```bash
[root@docker02 ~]# docker network create -d overlay overlay_net01
aa89ebe4cbaa07a916589bfea67e114ba918191fb855930fadd571b7ac0f6b73
[root@docker02 ~]# docker network ls
NETWORK ID     NAME            DRIVER    SCOPE
1f7df498b933   bridge          bridge    local
21fca6e6183c   host            host      local
f3f04f7b3813   none            null      local
aa89ebe4cbaa   overlay_net01   overlay   global
[root@docker02 ~]# docker run -dit --name test --network overlay_net01 busybox
bf3d315d14444263505d9fe04e240e04bba3f0034e795d0472e78eada25a290d
[root@docker02 ~]# docker run -dit --name test1 --network overlay_net01 busybox
741fcca70a2fdb541c85eb6c6de54d4ab4f3ba5ec221935beec0a38e2e1cfb0a
[root@docker02 ~]# docker exec -it test1 ping -w 2 -c 2 test
PING test (10.0.0.2): 56 data bytes
64 bytes from 10.0.0.2: seq=0 ttl=64 time=0.285 ms
64 bytes from 10.0.0.2: seq=1 ttl=64 time=0.102 ms
--- test ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.102/0.193/0.285 ms
[root@docker02 ~]# docker exec -it test ping -w 2 -c 2 test1
PING test1 (10.0.0.3): 56 data bytes
64 bytes from 10.0.0.3: seq=0 ttl=64 time=0.072 ms
64 bytes from 10.0.0.3: seq=1 ttl=64 time=0.306 ms
--- test1 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.072/0.189/0.306 ms
[root@docker02 ~]# docker exec -it test ping -w 2 -c 2 test2
PING test2 (10.0.0.4): 56 data bytes
64 bytes from 10.0.0.4: seq=0 ttl=64 time=1.489 ms
64 bytes from 10.0.0.4: seq=1 ttl=64 time=1.686 ms
--- test2 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 1.489/1.587/1.686 ms
```

node2

```bash
[root@docker03 ~]# docker run -dit --name test2 --network overlay_net01 busybox
ea6f770b5c11cb91d2e724b3ab000a8d0f7d2320d3b4b6dc536dd9f4bba89b6d
[root@docker03 ~]# docker exec -it test2 ping -w 2 -c 3 test1
PING test1 (10.0.0.3): 56 data bytes
64 bytes from 10.0.0.3: seq=0 ttl=64 time=0.630 ms
64 bytes from 10.0.0.3: seq=1 ttl=64 time=0.599 ms
--- test1 ping statistics ---
3 packets transmitted, 2 packets received, 33% packet loss
round-trip min/avg/max = 0.599/0.614/0.630 ms
[root@docker03 ~]# docker exec -it test2 ping -w 2 -c 3 test
PING test (10.0.0.2): 56 data bytes
64 bytes from 10.0.0.2: seq=0 ttl=64 time=3.892 ms
64 bytes from 10.0.0.2: seq=1 ttl=64 time=1.099 ms
--- test ping statistics ---
3 packets transmitted, 2 packets received, 33% packet loss
round-trip min/avg/max = 1.099/2.495/3.892 ms
```

![image-20210819101727570](docker.assets\image-20210819101727570.png)

自定义网段的创建网络 busybox 是一个工具容器，docker中的瑞士军刀

node01

```bash
[root@docker02 ~]# docker network create -d overlay  --subnet 172.16.0.0/24 --gateway 172.16.0.254 overlay_net02
4eaa01a02a10aa5aaa9d5fd36c3cf216583f7c487035711b559b6f24fdd0edfa
[root@docker02 ~]# docker network ls
NETWORK ID     NAME              DRIVER    SCOPE
1f7df498b933   bridge            bridge    local
135d12932909   docker_gwbridge   bridge    local
21fca6e6183c   host              host      local
f3f04f7b3813   none              null      local
aa89ebe4cbaa   overlay_net01     overlay   global
4eaa01a02a10   overlay_net02     overlay   global
[root@docker02 ~]# docker run -dit --name ov_test0 --network overlay_net02 busybox
983b763401bb918a01b0375a162c7bdf8eab253d00e660fb2cb9eda82d713ea3
[root@docker02 ~]# docker exec ov_test0 ping -w 2 -c 2 ov_test1
PING ov_test1 (172.16.0.2): 56 data bytes
64 bytes from 172.16.0.2: seq=0 ttl=64 time=0.603 ms
64 bytes from 172.16.0.2: seq=1 ttl=64 time=1.309 ms
--- ov_test1 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.603/0.956/1.309 ms
```

node02

```bash
[root@docker03 ~]# docker run -dit --name ov_test1 --network overlay_net02 busybox
[root@docker03 ~]# docker exec -it ov_test1 ping -w 2 -c 3 ov_test0
PING ov_test0 (172.16.0.1): 56 data bytes
64 bytes from 172.16.0.1: seq=0 ttl=64 time=1.134 ms
64 bytes from 172.16.0.1: seq=1 ttl=64 time=2.577 ms
--- ov_test0 ping statistics ---
3 packets transmitted, 2 packets received, 33% packet loss
round-trip min/avg/max = 1.134/1.855/2.577 ms
```

#### macvlan解决方案

一些程序，特别是应用程序或者网络流量监控程序，期望直接连接到物理网络，这种情况下，可使用Macvlan网络模式，给每个容器的虚拟网络接口配置一个mac地址，使得连接容器，看起来是直接到一个物理主机上。这种情况下，需要在主机上 为macvlan驱动，指定一个物理接口，一起子网与默认网关，甚至使用不同的物理网络接口，隔离navlan网络。但必须了解如下几点：

（1）IP地址溢出，和虚拟网络传播（VLAN spreads）将很容易导致网络损坏。例如网络中有众多无效的唯一mac地址，

 （2）网络设备需要处理“混杂模式”， 多个mac地址关联一个物理接口

 （3）如果应用程序可使用 bridge 模式 或者 overlay 模式，从长远来讲，将是更好的选择。

环境

| **docker02**      | **docker03**  |
| ----------------- | ------------- |
| **192.168.1.100** | 192.168.1.121 |
| **node1**         | node2         |

node1

```bash
[root@docker02 ~]# docker network create -d macvlan --subnet 192.168.1.0/24 --gateway 192.168.1.254 -o parent=eth0 mac_net1
[root@docker02 ~]# docker run -dit --name wac_test0 --ip 192.168.1.122 --network mac_net1 busybox
[root@docker02 ~]# docker exec -it wac_test0 ping -w 2 -c 3 192.168.1.254
PING 192.168.1.254 (192.168.1.254): 56 data bytes
64 bytes from 192.168.1.254: seq=0 ttl=128 time=7.590 ms
64 bytes from 192.168.1.254: seq=1 ttl=128 time=1.670 ms
--- 192.168.1.254 ping statistics ---
3 packets transmitted, 2 packets received, 33% packet loss
round-trip min/avg/max = 1.670/4.630/7.590 ms
[root@docker02 ~]# docker exec -it wac_test0 ping -w 2 -c 3 192.168.1.120
PING 192.168.1.120 (192.168.1.120): 56 data bytes
64 bytes from 192.168.1.120: seq=0 ttl=64 time=5.996 ms
64 bytes from 192.168.1.120: seq=1 ttl=64 time=0.731 ms
--- 192.168.1.120 ping statistics ---
3 packets transmitted, 2 packets received, 33% packet loss
round-trip min/avg/max = 0.731/3.363/5.996 ms
[root@docker02 ~]# docker exec -it wac_test0 ping -w 2 -c 3 192.168.1.5
PING 192.168.1.5 (192.168.1.5): 56 data bytes
64 bytes from 192.168.1.5: seq=0 ttl=128 time=0.853 ms
64 bytes from 192.168.1.5: seq=1 ttl=128 time=1.402 ms
--- 192.168.1.5 ping statistics ---
3 packets transmitted, 2 packets received, 33% packet loss
round-trip min/avg/max = 0.853/1.127/1.402 ms
[root@docker02 ~]# docker exec -it wac_test0 ping -w 2 -c 3 192.168.1.123
PING 192.168.1.123 (192.168.1.123): 56 data bytes
64 bytes from 192.168.1.123: seq=0 ttl=64 time=5.303 ms
64 bytes from 192.168.1.123: seq=1 ttl=64 time=0.863 ms
--- 192.168.1.123 ping statistics ---
3 packets transmitted, 2 packets received, 33% packet loss
round-trip min/avg/max = 0.863/3.083/5.303 ms
```

node2

```bash
[root@docker03 ~]# docker network create -d macvlan --subnet 192.168.1.0/24 --gateway 192.168.1.254 -o parent=eth0 mac_net1
24e712ee2105f36ffd070dbecfad550e0b72d17b2839277f7c7243253c0e338f
[root@docker03 ~]# docker run -dit --name wac_test1 --ip 192.168.1.123 --network mac_net1 busybox
021782bf85f7a3a84834c4d3819a22528f5fe78a73e58f73410b1c0eef698948
[root@docker03 ~]# docker exec -it wac_test1 ping -w 2 -c 3 192.168.1.122  这里用主机名ping 找不到，只能IP地址
PING 192.168.1.122 (192.168.1.122): 56 data bytes
64 bytes from 192.168.1.122: seq=0 ttl=64 time=5.812 ms
64 bytes from 192.168.1.122: seq=1 ttl=64 time=1.093 ms
--- 192.168.1.122 ping statistics ---
3 packets transmitted, 2 packets received, 33% packet loss
round-trip min/avg/max = 1.093/3.452/5.812 ms
[root@docker03 ~]# docker exec -it wac_test1 ping -w 2 -c 3 192.168.1.254
PING 192.168.1.254 (192.168.1.254): 56 data bytes
64 bytes from 192.168.1.254: seq=0 ttl=128 time=5.418 ms
64 bytes from 192.168.1.254: seq=1 ttl=128 time=1.805 ms
--- 192.168.1.254 ping statistics ---
3 packets transmitted, 2 packets received, 33% packet loss
round-trip min/avg/max = 1.805/3.611/5.418 ms
[root@docker03 ~]# docker exec -it wac_test1 ping -w 2 -c 3 192.168.1.100
PING 192.168.1.100 (192.168.1.100): 56 data bytes
64 bytes from 192.168.1.100: seq=0 ttl=64 time=71.041 ms
64 bytes from 192.168.1.100: seq=1 ttl=64 time=0.424 ms
--- 192.168.1.100 ping statistics ---
3 packets transmitted, 2 packets received, 33% packet loss
round-trip min/avg/max = 0.424/35.732/71.041 ms
[root@docker03 ~]# docker exec -it wac_test1 ping -w 2 -c 3 192.168.1.5
PING 192.168.1.5 (192.168.1.5): 56 data bytes
64 bytes from 192.168.1.5: seq=0 ttl=128 time=0.985 ms
64 bytes from 192.168.1.5: seq=1 ttl=128 time=1.095 ms
--- 192.168.1.5 ping statistics ---
3 packets transmitted, 2 packets received, 33% packet loss
round-trip min/avg/max = 0.985/1.040/1.095 ms
```

看样子一台也可以使用

#### wacvlan多网段解决方案

一个网卡只能创建一个macvlan，想创建多个网段的macvlan需要使用sub-interface

开启网卡混杂模式

node1

```bash
[root@docker02 ~]# ifconfig eth0 promisc
[root@docker02 ~]# cd /etc/sysconfig/network-scripts/
[root@docker02 /etc/sysconfig/network-scripts]# cp ifcfg-eth0 ifcfg-eth0.10
[root@docker02 /etc/sysconfig/network-scripts]# cat ifcfg-eth0.10 
PHYSDEV=eth0	
VLAN=yes
TYPE=Vlan
VLAN_ID=10		
REORDER_HDR=yes
GVRP=no
MVRP=no
BOOTPROTO=none
IPADDR=192.168.2.100	
PREFIX=24
GATEWAY=192.168.2.254	
DEFROUTE=yes
IPV4_FAILURE_FATAL=yes
NAME=eth0.10
ONBOOT=yes   
[root@docker02 ~]# systemctl restart network
```

![image-20210819101918446](docker.assets\image-20210819101918446.png)

node2

```bash
[root@docker03 ~]# ifconfig eth0 promisc
[root@docker03 ~]# cd /etc/sysconfig/network-scripts/
[root@docker03 /etc/sysconfig/network-scripts]# cp ifcfg-eth0 ifcfg-eth0.10
[root@docker03 /etc/sysconfig/network-scripts]# vim ifcfg-eth0.10 
PHYSDEV=eth0    
VLAN=yes
TYPE=Vlan
VLAN_ID=10      
REORDER_HDR=yes
GVRP=no
MVRP=no
BOOTPROTO=none
IPADDR=192.168.2.101                                                                                                           
PREFIX=24
GATEWAY=192.168.2.254   
DEFROUTE=yes
IPV4_FAILURE_FATAL=yes
NAME=eth0.10
ONBOOT=yes   
[root@docker03 /etc/sysconfig/network-scripts]# systemctl restart network
```

![image-20210819101939745](docker.assets\image-20210819101939745.png)

开启路由转发

node1

```bash
[root@docker02 ~]# echo "net.ipv4.ip_forward = 1" >/etc/sysctl.conf
[root@docker02 ~]# sysctl -p
net.ipv4.ip_forward = 1
[root@docker02 ~]# docker network create -d macvlan --subnet=192.168.2.0/24 --gateway=192.168.2.254 -o parent=eth0.10 mac_net2
09f25e669383f52cf2b4c33e458d688400a58c42fc40f93b081a01a0d969ceac
[root@docker02 ~]# docker network ls
NETWORK ID     NAME              DRIVER    SCOPE
1f7df498b933   bridge            bridge    local
135d12932909   docker_gwbridge   bridge    local
21fca6e6183c   host              host      local
f2b6e7411d44   mac_net1          macvlan   local
09f25e669383   mac_net2          macvlan   local
f3f04f7b3813   none              null      local
aa89ebe4cbaa   overlay_net01     overlay   global
4eaa01a02a10   overlay_net02     overlay   global
[root@docker02 ~]# docker run -dit --name wac_test10 --ip=192.168.2.100 --network mac_net2 busybox
[root@docker02 ~]# docker exec -it wac_test10 ping -w 2 -c 3 192.168.2.100
PING 192.168.2.100 (192.168.2.100): 56 data bytes
64 bytes from 192.168.2.100: seq=0 ttl=64 time=0.077 ms
64 bytes from 192.168.2.100: seq=1 ttl=64 time=0.146 ms
--- 192.168.2.100 ping statistics ---
3 packets transmitted, 2 packets received, 33% packet loss
round-trip min/avg/max = 0.077/0.111/0.146 ms
[root@docker02 ~]# docker exec -it wac_test10 ping -w 2 -c 3 192.168.2.101
PING 192.168.2.101 (192.168.2.101): 56 data bytes
--- 192.168.2.101 ping statistics ---
3 packets transmitted, 0 packets received, 100% packet loss
```

node2

```bash
[root@docker03 ~]# vim /etc/sysctl.conf 
net.ipv4.ip_forward = 1                                  
[root@docker03 ~]# sysctl -p
net.ipv4.ip_forward = 1
[root@docker03 ~]# docker network create -d macvlan --subnet=192.168.2.0/24 --gateway=192.168.2.254 -o parent=eth0.10 mac_net3
19b8f32f3803fb5fd40c2a766baf7af7157c5317e0fa480231bbbed6045560c7
[root@docker03 ~]# docker network ls
NETWORK ID     NAME              DRIVER    SCOPE
49d80c0afd95   bridge            bridge    local
21c897a98628   docker_gwbridge   bridge    local
a611aff4793e   harbor_harbor     bridge    local
3e5e4ed821f7   host              host      local
24e712ee2105   mac_net1          macvlan   local
19b8f32f3803   mac_net3          macvlan   local
b67fd6c0fc61   none              null      local
aa89ebe4cbaa   overlay_net01     overlay   global
4eaa01a02a10   overlay_net02     overlay   global
[root@docker03 ~]# docker run -dit --name wac_test13 --ip=192.168.2.101 --network mac_net3 busybox
[root@docker03 ~]# docker exec -it wac_test13 ping -w 2 -c 3 192.168.2.100
PING 192.168.2.100 (192.168.2.100): 56 data bytes
64 bytes from 192.168.2.100: seq=0 ttl=64 time=7.242 ms
64 bytes from 192.168.2.100: seq=1 ttl=64 time=1.012 ms
--- 192.168.2.100 ping statistics ---
3 packets transmitted, 2 packets received, 33% packet loss
round-trip min/avg/max = 1.012/4.127/7.242 ms
[root@docker03 ~]# docker exec -it wac_test13 ping -w 2 -c 3 192.168.1.100
PING 192.168.1.100 (192.168.1.100): 56 data bytes
--- 192.168.1.100 ping statistics ---
3 packets transmitted, 0 packets received, 100% packet loss
[root@docker03 ~]# docker exec -it wac_test13 ping -w 2 -c 3 192.168.2.254
PING 192.168.2.254 (192.168.2.254): 56 data bytes
--- 192.168.2.254 ping statistics ---
3 packets transmitted, 0 packets received, 100% packet loss
```

不能ping通是因为docker network create 创建的虚拟交换机不同 需要ping通需要两台都创建一个网络mac_net2 或者mac_net3 

8021q封装功能

```bash
[root@docker02 ~]# modinfo 8021q
filename:       /lib/modules/3.10.0-1160.15.2.el7.x86_64/kernel/net/8021q/8021q.ko.xz
version:        1.8
license:        GPL
alias:          rtnl-link-vlan
retpoline:      Y
rhelversion:    7.9
srcversion:     1DD872AF3C7FF7FFD5B14D5
depends:        mrp,garp
intree:         Y
vermagic:       3.10.0-1160.15.2.el7.x86_64 SMP mod_unload modversions 
signer:         CentOS Linux kernel signing key
sig_key:        02:FB:1B:20:B0:39:E3:CD:C7:59:93:8B:A9:58:53:84:81:77:80:31
sig_hashalgo:   sha256
[root@docker02 ~]# modprobe 8021q
```

容器基本算是完成了，更多的解决方案需要自己在实践中应用。



