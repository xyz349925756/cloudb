

# Docker

容器本身没有价值，有价值的是“容器编排”。

容器，其实是一种特殊的进程而已。

因为容器具备如下优点，所以很受欢迎：

- 灵活性：即使是最复杂的应用程序也可以容器化。
- 轻量化：容器共享主机内核，使得它们远比虚拟机高效。
- 便携性：可以做到本地编译，到处运行。
- 松耦合：容器自我封装，一个容器被替换或升级不会打断别的容器。
- 安全性：容器对进程进行了严格的限制和隔离，而无需用户进行任何配置。

# Linux 容器的具体实现方式

一个“容器”，实际上是一个由 Linux Namespace、Linux Cgroups 和 rootfs 三种技术构建出来的进程的隔离环境。

1. **Namespace** 的作用是“隔离”，它让应用进程只能看到该 Namespace 内的“世界”；修改进程视图的主要方法
2. **Cgroups** 的作用是“限制”，它给这个“世界”围上了一圈看不见的墙。制造约束的主要手段

​    3.一组联合挂载在 /var/lib/docker/aufs/mnt 上的 **rootfs**，这一部分我们称为“容器镜像”（Container Image），是容器的静态视图；一个由 Namespace+Cgroups 构成的隔离环境，这一部分我们称为“容器运行时”（Container Runtime），是容器的动态视图。

## namespace机制

在容器中看到只有自己一个进程 

但是这个进程其实是在宿主机上的 只不过是一个独立的namespace 

定义了5个命名空间结构体，多个进程可以使用同一个namespace

1. UTS： 运行内核的名称、版本、底层体系结构类型等信息（UNIX Timesharing System）
2. IPC： 与进程间通信（IPC)有关

1. MNT： 已经装载的文件系统的视图 Mount Namespace，用于让被隔离进程只看到当前 Namespace 里的挂载点信息；
2. PID：有关进程ID的信息

1. NET：网络相关的命名空间参数 Network Namespace，用于让被隔离进程看到当前 Namespace 里的网络设备和配置。

在 Linux 内核中，有很多资源和对象是不能被 Namespace 化的，最典型的例子就是：**时间**。

## cgroup技术

Linux Cgroups 就是 Linux 内核中用来为进程设置资源限制的一个重要功能。

Linux Cgroups 的全称是 Linux Control Group。它最主要的作用，就是限制一个进程组能够使用的资源上限，包括 CPU、内存、磁盘、网络带宽等等。

此外，Cgroups 还能够对进程进行优先级设置、审计，以及将进程挂起和恢复等操作

一个正在运行的 Docker 容器，其实就是一个启用了多个 Linux Namespace 的应用进程，而这个进程能够使用的资源量，则受 Cgroups 配置的限制。这也是容器技术中一个非常重要的概念，即：容器是一个“单进程”模型。

centos ubuntu等本来就是用的相同的linux内核，但是其他操作系统的一些可执行文件等等就是各个linux发行版不同的地方了。

## 容器化的基础

### Chroot

如果需要在一个宿主机上运行多个容器，且容器之间相互隔离，那么第一个就需要系统库文件的依赖，对于一个容器而言，需要将其需要的系统文件单独复制出来一份，放到指定目录，并且需要让进程认为这就是根目录，而不是去调用宿主机系统上的库文件。Chroot就是一个切换根目录的方式。

### NameSpaces

为了让多个容器以沙盒的方式在宿主机上运行，就需要提前定义好各个容器能看到的边界。由于各个容器都是直接运行在宿主机系统上，因此需要内核对各个容器的上下文进行修改，让他们看上去是一个独立的操作系统。比如，指定PID为1的进程，指定网卡设备，指定文件系统挂载，指定用户等等。

Linux操作系统内核从底层实现了为各个进程创建独立用户空间的功能，不同用户空间似于一个个独立的虚拟机系统，用户空间内部进程不能感知到其它用户空间中的进程状态。内核提供了六种Namespaces:

| UTS   | hostname and domainname  | 主机名和域名隔离                           | 内核版本：2.6.19 |
| ----- | ------------------------ | ------------------------------------------ | ---------------- |
| User  |                          | 用户隔离。运行进程的用户和组               | 内核版本：3.8.x  |
| Mount |                          | 挂载点隔离。即挂载点隔离，主要指根目录     | 内核版本：2.4.19 |
| IPC   | Inter-process-connection | 进程间通信隔离。消息队列、共享内容、信号量 | 内核版本：2.6.19 |
| Pid   | Process                  |                                            |                  |
| ID    | PID隔离                  | 内核版本：2.6.24                           |                  |
| Net   | Network                  | 网络隔离。网络设备、协议栈、端口           | 内核版本：2.6.29 |

### Cgroups

Namespaceses通过障眼法实现了用户空间的隔离，但是没办法对硬件资源进行限制，当一个容器进行CPU密集型操作时，会消耗掉整个宿主机的CPU资源，进而影响了其它容器的正常运行。

因此在Namespaceses之上，还需要对各个容器实现硬件资源限制，比如CPU，Memory,diskio等等。

Cgroups技术针对进程而言的，在centos7系统上，可以通过以下方式来实现对进程的资源限制:



```bash
[root@docker01 ~]# while :;do :; done &   #后台模拟一个死循环
[1] 1688
[root@docker01 ~]# pidstat -u -p 1688 2   #没有cgroups 跑满单个CPU核心
Linux 3.10.0-1160.el7.x86_64 (docker01) 	06/17/2021 	_x86_64_	(1 CPU)

09:52:02 AM   UID       PID    %usr %system  %guest    %CPU   CPU  Command
09:52:04 AM     0      1688   99.50    0.00    0.00   99.50     0  bash
09:52:06 AM     0      1688  100.00    0.00    0.00  100.00     0  bash
09:52:08 AM     0      1688  100.00    0.00    0.00  100.00     0  bash
09:52:10 AM     0      1688   99.50    0.00    0.00   99.50     0  bash
09:52:12 AM     0      1688  100.00    0.00    0.00  100.00     0  bash
09:52:14 AM     0      1688  100.00    0.00    0.00  100.00     0  bash
[root@docker01 ~]# mount -t cgroup  #查看当前cgroup路径
cgroup on /sys/fs/cgroup/systemd type cgroup (rw,nosuid,nodev,noexec,relatime,xattr,release_agent=/usr/lib/systemd/systemd-cgroups-agent,name=systemd)
cgroup on /sys/fs/cgroup/perf_event type cgroup (rw,nosuid,nodev,noexec,relatime,perf_event)
cgroup on /sys/fs/cgroup/cpu,cpuacct type cgroup (rw,nosuid,nodev,noexec,relatime,cpuacct,cpu)
cgroup on /sys/fs/cgroup/blkio type cgroup (rw,nosuid,nodev,noexec,relatime,blkio)
cgroup on /sys/fs/cgroup/hugetlb type cgroup (rw,nosuid,nodev,noexec,relatime,hugetlb)
cgroup on /sys/fs/cgroup/freezer type cgroup (rw,nosuid,nodev,noexec,relatime,freezer)
[root@docker01 ~]# mkdir /sys/fs/cgroup/cpu/loop   #创建一个loop
[root@docker01 ~]# cat /sys/fs/cgroup/cpu/loop/cpu.cfs_quota_us  #定额
-1
[root@docker01 ~]# cat /sys/fs/cgroup/cpu/loop/cpu.cfs_period_us   #周期
100000
[root@docker01 ~]# echo 10000 >/sys/fs/cgroup/cpu/loop/cpu.cfs_quota_us  #修改CPU限制在10%
[root@docker01 ~]# echo 1726 >/sys/fs/cgroup/cpu/loop/tasks # 指定限制的进程
[root@docker01 ~]# pidstat -u -p 1726 2    #验正结果
Linux 3.10.0-1160.el7.x86_64 (docker01) 	06/17/2021 	_x86_64_	(1 CPU)
10:06:30 AM   UID       PID    %usr %system  %guest    %CPU   CPU  Command
10:06:32 AM     0      1726   10.81    0.00    0.00   10.81     0  bash
10:06:34 AM     0      1726   10.87    0.00    0.00   10.87     0  bash
10:06:36 AM     0      1726   10.87    0.00    0.00   10.87     0  bash
10:06:38 AM     0      1726   10.81    0.00    0.00   10.81     0  bash
[root@docker01 ~]# jobs -l   #调出隐藏进程
[1]+  1726 Running                 while :; do
    :;
done &
[root@docker01 ~]# kill % 1    #2是暂停
-bash: kill: %: no such job
```

### Docker组件

Docker服务有三个部分组成，分别是Client，Docker Host，Registry。当创建新的容器时，会向Docker Daemon发送指令，Docker Daemon通过本地镜像文件创建容器，当本地不存在镜像时，将从Registry下载镜像。

Registry由两个部分组成：

Repostitory

- 由特定的docker镜像的所有迭代版本组成一个镜像仓库
- 一个Registry可以包括多个Repostitory

- Repostitory包含顶层仓库和用户仓库

1. 顶层仓库: 仓库名:标签, nginx:latest
2. 用户仓库: 用户名/仓库名:标签, xxxx/nginx:1.18.1

- 一个镜像可以有多个标签，如最新版的nginx,可以是nginx:latest,nginx:1.18.1

Index

- 提供用户认证、镜像检索功能

### Docker镜像

镜像(Image)是一堆只读层(read-only layer)的统一视角。如下图所示:



左边的是多个只读层，他们相互堆叠在一起。除了最下层之外，其它每一层都会有一个指针指向下一层。这些层是Docker内部的实现细节，并且能够在宿主机的文件系统上访问到。

统一文件系统（union file system,aufs）技术(新版用overlay2)能够将不同的层整合成一个文件系统，为这些层提供了一个统一的视角，这样就隐藏了多层的存在，在用户的角度看来，只存在一个文件系统。我们可以在图片的右边看到这个视角的形式。

每一层都包含了当前层的ID，Metadata，Pointer（指向上一层）三层，最底层不包含Pointer。

### Docker容器

Docker容器包含静止状态和运行状态两种，这两种状态下的层级不一样。

静态状态的容器仅仅是在镜像状态下增加一个可读写的层级，运行状态中的容器包含了进程和对应的进程空间:





docker 是一个典型的C/S架构

Docker在2017年以前时使用大版本号+小版本号来名，在2017年之后，采用YY.MM.N-xx格式，如 19.03.1-ce表示2019年3月份的第2个ce版本。以CentOS 7安装docker-ce版本为例

准备基础环境，下载常用软件和环境变量

```bash
[root@localhost ~]# yum install -y vim wget net-tools bash-completion
[root@localhost ~]# cat /etc/issue 开机界面显示IP地址
[root@localhost ~]# cat /etc/bashrc #环境变量修改
   PS1='[\[\e[31;40m\]\u\[\e[33;40m\]@\[\e[34;40m\]\h \[\e[33;40m\]\w\[\e[0m\]]\$ '
[root@localhost ~]# source /etc/bashrc
[root@localhost ~]# mv vimrc .vimrc   #编辑vim功能
[root@localhost ~]# source /usr/share/bash-completion/completions/docker  #加载docker tab 补全

```

配置国内yum源

```shell
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
或者
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
[root@docker01 ~]# yum clean all
[root@docker01 ~]# yum makecache
```

## docker install

基础组件的安装

https://docs.docker.com/engine/install/centos/

```bash
[root@docker01 ~]# yum install -y yum-utils
[root@docker01 ~]# yum-config-manager --add-repo \
                  https://download.docker.com/linux/centos/docker-ce.repo
[root@docker01 ~]# yum install docker-ce docker-ce-cli containerd.io -y
#查看仓库里面docker版本
[root@docker01 ~]# yum list docker-ce --showduplicates | sort -r
docker-ce.x86_64            3:20.10.7-3.el7                    docker-ce-stable 
docker-ce.x86_64            3:20.10.7-3.el7                    @docker-ce-stable
docker-ce.x86_64            3:20.10.6-3.el7                    docker-ce-stable 
docker-ce.x86_64            3:20.10.5-3.el7                    docker-ce-stable 
docker-ce.x86_64            3:20.10.4-3.el7                    docker-ce-stable 
#如果您需要安装到某个稳定版本就可以使用
[root@docker01 ~]#yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io
[root@docker01 ~]#systemctl start docker && systemctl enable docker
#启动，并配置开机自启
[root@docker01 ~]# docker run hello-world
#运行一个容器测试下docker 是否安装正确
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
b8dfde127a29: Pull complete 
Digest: sha256:9f6ad537c5132bcce57f7a0a20e317228d382c3cd61edae14650eec68b2b345c
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

```

rpm 软件包安装

#跟上面yum 安装是一样的只是yum 是在线安装，rpm是离线安装。

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

# docker 镜像管理

![image-20210617200847523](C:\Users\goo\AppData\Roaming\Typora\typora-user-images\image-20210617200847523.png)

- docker镜像是一个典型的分层结构
- 只有最上面一层是可写的 其他都是只读的固化到镜像的

- 每次推送都是增量的

![image-20210617201022774](C:\Users\goo\AppData\Roaming\Typora\typora-user-images\image-20210617201022774.png)

镜像名称的结构

```shell
${registry_ name}/${repository. name}/${image. name}:${tag. name}
```

例如:

```shell
docker.io/library/alpine:3.10.1
```





### 登录docker仓库

```bash
[root@docker01 ~]# docker login docker.io     #登录到docker官方仓库
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: xyz349925756
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
[root@docker01 ~]# cat /root/.docker/config.json  #授权信息保存文件
{
	"auths": {
		"https://index.docker.io/v1/": {
			"auth": "eHl6MzQ5OTI1NzU2OmhvbmdmZWkwMDc="
		}
	}
}
[root@docker01 ~]# echo "eHl6MzQ5OTI1NzU2OmhvbmdmZWkwMDc=" |base64 -d
xyz349925756:********** #解密为明文密码了
[root@docker01 ~]# 

#搜索某个镜像
[root@docker01 ~]# docker search alpine
NAME                  DESCRIPTION                                  STARS     OFFICIAL   AUTOMATED
alpine               A minimal Docker image based on Alpine Linux…   7554      [OK]       
mhart/alpine-node    Minimal Node.js built on Alpine Linux           484                  
anapsix/alpine-java  Oracle Java 8 (and 7) with GLIBC 2.28 over A…   470                  [OK]

#NAME        名称
#DESCRIPTION 说明
#STARS       点赞数（下载数）
#OFFICIAL    官方镜像
#AUTOMATED   第三方镜像

#没有指定版本都是latest
[root@docker01 ~]# docker pull alpine
Using default tag: latest
latest: Pulling from library/alpine
5843afab3874: Pull complete 
Digest: sha256:234cb88d3020898631af0ccbbcca9a66ae7306ecd30c9720690858c1b007d2a0
Status: Downloaded newer image for alpine:latest
docker.io/library/alpine:latest

[root@docker01 ~]# docker pull busybox   #linux 里面的工具集
Using default tag: latest
latest: Pulling from library/busybox
b71f96345d44: Pull complete 
Digest: sha256:930490f97e5b921535c153e0e7110d251134cc4b72bbb8133c6a5065cc68580d
Status: Downloaded newer image for busybox:latest
docker.io/library/busybox:latest

[root@docker01 ~]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
alpine       latest    d4ff818577bc   38 hours ago   5.6MB
busybox      latest    69593048aa3a   9 days ago     1.24MB
nginx        latest    d1a364dc548d   3 weeks ago    133MB

#拉取官方镜像不必加docker.io  默认就是这个仓库。但是拉取私有的必须使用全称docker.io/xyz349925756/alpine

#打标签
[root@docker01 ~]# docker tag 69593048aa3a docker.io/xyz349925756/busybox:1.33.1

[root@docker01 ~]# docker images
REPOSITORY             TAG       IMAGE ID       CREATED        SIZE
alpine                 latest    d4ff818577bc   39 hours ago   5.6MB
busybox                latest    69593048aa3a   9 days ago     1.24MB
xyz349925756/busybox   1.33.1    69593048aa3a   9 days ago     1.24MB
nginx                  latest    d1a364dc548d   3 weeks ago    133MB

#上传镜像到docker.io仓库
[root@docker01 ~]# docker push docker.io/xyz349925756/busybox:1.33.1
The push refers to repository [docker.io/xyz349925756/busybox]
5b8c72934dfc: Mounted from library/busybox 
1.33.1: digest: sha256:dca71257cd2e72840a21f0323234bb2e33fea6d949fa0f21c5102146f583486b size: 527

#删除镜像
[root@docker01 ~]# docker image rm -f 69593048aa3a
Untagged: busybox:latest
Untagged: busybox@sha256:930490f97e5b921535c153e0e7110d251134cc4b72bbb8133c6a5065cc68580d
Untagged: xyz349925756/busybox:1.33.1
Untagged: xyz349925756/busybox@sha256:dca71257cd2e72840a21f0323234bb2e33fea6d949fa0f21c5102146f583486b
Deleted: sha256:69593048aa3acfee0f75f20b77acb549de2472063053f6730c4091b53f2dfb02
Deleted: sha256:5b8c72934dfc08c7d2bd707e93197550f06c0751023dabb3a045b723c5e7b373

#确认
[root@docker01 ~]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
alpine       latest    d4ff818577bc   39 hours ago   5.6MB
nginx        latest    d1a364dc548d   3 weeks ago    133MB

#拉取刚才上传的镜像
[root@docker01 ~]# docker pull xyz349925756/busybox:1.33.1
1.33.1: Pulling from xyz349925756/busybox
b71f96345d44: Pull complete 
Digest: sha256:dca71257cd2e72840a21f0323234bb2e33fea6d949fa0f21c5102146f583486b
Status: Downloaded newer image for xyz349925756/busybox:1.33.1
docker.io/xyz349925756/busybox:1.33.1

#查看
[root@docker01 ~]# docker images
REPOSITORY             TAG       IMAGE ID       CREATED        SIZE
alpine                 latest    d4ff818577bc   39 hours ago   5.6MB
xyz349925756/busybox   1.33.1    69593048aa3a   9 days ago     1.24MB
nginx                  latest    d1a364dc548d   3 weeks ago    133MB

```

![image-20210617212230844](C:\Users\goo\AppData\Roaming\Typora\typora-user-images\image-20210617212230844.png)

## docker 基本操作

```bash
root@docker01 ~]# docker images           #查看本地镜像
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
hello-world   latest    d1165f221234   3 months ago   13.3kB

[root@docker01 ~]# docker image rm -f d1165f221234   #删除不要的镜像
Untagged: hello-world:latest
Untagged: hello-world@sha256:9f6ad537c5132bcce57f7a0a20e317228d382c3cd61edae14650eec68b2b345c
Deleted: sha256:d1165f2212346b2bab48cb01c1e39ee8ad1be46b87873d9ca7a4e434980a7726

[root@docker01 ~]# docker rm -f `docker ps -a|awk 'NR==2 {print $1}'`  #删除退出的容器

[root@docker01 ~]#docker rm -f  `docker ps -a -q`   #删除退出的container,也会删除正在运行的container

[root@docker01 ~]# docker run -itd --name=nginx1 -p 80:80 nginx:latest 
1656d7eeeae7c6df2ff3fa0c318f0a32fd6d69ca280c1033e3c7db0218f06ef5
#运行一个container 
```

### container的常规操作

```bash
[root@docker01 ~]# docker container ls -a
CONTAINER ID   IMAGE          COMMAND                  CREATED        STATUS                    PORTS     NAMES
1656d7eeeae7   nginx:latest   "/docker-entrypoint.…"   24 hours ago   Exited (0) 24 hours ago             nginx1

[root@docker01 ~]# docker ps -a
CONTAINER ID   IMAGE          COMMAND                  CREATED        STATUS                    PORTS     NAMES
1656d7eeeae7   nginx:latest   "/docker-entrypoint.…"   24 hours ago   Exited (0) 24 hours ago             nginx1

#一个是新的一个是旧的加上container的是新语法，旧的有可能在未来版本优化了

[root@docker01 ~]# docker run -itd --rm --name=busybox busybox:1.33.1 /bin/sh
633c1b00fa2394c0e0a9863edc5da48a8315499733b7ee888fb5af100ca3a8be
#这个container stop 之后就会被删除 

[root@docker01 ~]# docker ps -a
CONTAINER ID   IMAGE            COMMAND                  CREATED         STATUS                    PORTS     NAMES
633c1b00fa23   busybox:1.33.1   "/bin/sh"                3 seconds ago   Up 2 seconds                        busybox
1656d7eeeae7   nginx:latest     "/docker-entrypoint.…"   24 hours ago    Exited (0) 24 hours ago             nginx1

#进入container 看下基本操作和网络
[root@docker01 ~]# docker exec -it 633c1b00fa23 /bin/sh
/ # ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:AC:11:00:02  
          inet addr:172.17.0.2  Bcast:172.17.255.255  Mask:255.255.0.0

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
/ # pwd
/
/ # whoami
root
/ # ping 172.16.0.1
PING 172.16.0.1 (172.16.0.1): 56 data bytes
64 bytes from 172.16.0.1: seq=0 ttl=63 time=2.341 ms
^C
--- 172.16.0.1 ping statistics ---
4 packets transmitted, 4 packets received, 0% packet loss
round-trip min/avg/max = 2.047/2.799/4.665 ms
/ # ping -w 2 -c 3 www.baidu.com
PING www.baidu.com (183.232.231.174): 56 data bytes
64 bytes from 183.232.231.174: seq=0 ttl=55 time=37.085 ms
64 bytes from 183.232.231.174: seq=1 ttl=55 time=40.521 ms

--- www.baidu.com ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 37.085/38.803/40.521 ms
#能连接到外网可以正常上网

[root@docker01 ~]# docker stop 633c1b00fa23
633c1b00fa23
#手动停止这个container

[root@docker01 ~]# docker ps -a
CONTAINER ID   IMAGE          COMMAND                  CREATED        STATUS                    PORTS     NAMES
1656d7eeeae7   nginx:latest   "/docker-entrypoint.…"   24 hours ago   Exited (0) 24 hours ago             nginx1

```

**上面命令讲解**

docker  run  [option] image [command] [arg]

-i :启动一个交互式的container，并持续打开标准输入

-t ：伪装成一台终端

-d：后台运行

--rm  ：容器退出后旧删除

--name  ：容器别名，容器删除后消失

image ：指定调用的container

command ：指定container 要执行的命令

arg：参数

```bash
docker run --rm -td  --name=busybox_1 busybox:1.33.1 ping -w 2 -c 3 www.baidu.com
#一闪而过就执行完成了
[root@docker01 ~]# docker run -itd  --name=busybox_1 busybox:1.33.1 ping -w 2 -c 3 www.baidu.com >/tmp/1.txt
执行完之后container就 exited  因为container 中 init 1 的进程没有运行，它执行完成之后就退出了
通常情况会在后面加命令   /bin/sleep 300  这样可以持续300秒
ps aux |grep [s]leep 这样可以查看到后台进程


#container 的进入 两种方法containerID or names
[root@docker01 ~]# docker exec  -it objective_nash /bin/sh
[root@docker01 ~]# docker exec -it 633c1b00fa23 /bin/sh

#container 的启动重启停止  containerID or names
docker start 06fbbee401aa
docker restart 06fbbee401aa
docker stop 06fbbee401aa

删除这些exited的container
[root@docker01 ~]# docker rm -f `docker ps -a -q`

[root@docker01 ~]# docker run -dt --name=test01 busybox:1.33.1 /bin/sh
8445a97af41f9a2fc2e957bf3b09886115ad20a55468736028d5855eb986a90c
[root@docker01 ~]# docker ps -a
CONTAINER ID   IMAGE            COMMAND     CREATED         STATUS         PORTS     NAMES
8445a97af41f   busybox:1.33.1   "/bin/sh"   6 seconds ago   Up 5 seconds             test01
[root@docker01 ~]# docker exec  -it test01 /bin/sh
/ # ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:AC:11:00:02  
          inet addr:172.17.0.2  Bcast:172.17.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:8 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:656 (656.0 B)  TX bytes:0 (0.0 B)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

/ # ifconfig >/tmp/1.txt
/ # ls /tmp/
1.txt


Usage:  docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH|-
	    docker cp [OPTIONS] SRC_PATH|- CONTAINER:DEST_PATH

[root@docker01 ~]# ls
anaconda-ks.cfg  uuid.sh

#拉取container里面的文件到本地
[root@docker01 ~]# docker cp 8445a97af41f:/tmp/1.txt ./
[root@docker01 ~]# ls
1.txt  anaconda-ks.cfg  uuid.sh

#推送本地文件到container指定目录
[root@docker01 ~]# docker cp uuid.sh 8445a97af41f:/tmp/
[root@docker01 ~]# docker exec  -it test01 /bin/sh
/ # ls /tmp/
1.txt    uuid.sh
/ # 
```

container 里面写入的文件不会保存退出就不在了

```bash
#固化文件到container
[root@docker01 ~]# docker commit -p test01 docker.io/xyz349925756/busybox:1.33.1_1.txt
sha256:ddee1359ecbccdde9d28ea70bc8aa42ba2f09ca1531036c38372a7cda9ba3b2b

[root@docker01 ~]# docker images
REPOSITORY             TAG            IMAGE ID       CREATED          SIZE
xyz349925756/busybox   1.33.1_1.txt   ddee1359ecbc   12 seconds ago   1.24MB
alpine                 latest         d4ff818577bc   40 hours ago     5.6MB
busybox                1.33.1         69593048aa3a   9 days ago       1.24MB
xyz349925756/busybox   1.33.1         69593048aa3a   9 days ago       1.24MB
nginx                  latest         d1a364dc548d   3 weeks ago      133MB
[root@docker01 ~]# docker run -it --rm xyz349925756/busybox:1.33.1_1.txt 
/ # ls /tmp
1.txt    uuid.sh
/ # 

-p 参数 保存到执行命令这一时的内容
```



### 镜像的导出导入

```bash
[root@docker01 ~]# docker save ddee1359ecbc > /data/docker/busybox.1.33.1_1.txt.tar
[root@docker01 ~]# ls /data/docker/
busybox.1.33.1_1.txt.tar
[root@docker01 ~]# docker image rm ddee1359ecbc
Untagged: xyz349925756/busybox:1.33.1_1.txt
Deleted: sha256:ddee1359ecbccdde9d28ea70bc8aa42ba2f09ca1531036c38372a7cda9ba3b2b
Deleted: sha256:f633bbb261ba1188b9a1863ef784bdc944ff92c523a217a7601164a7c2080521
[root@docker01 ~]# docker images
REPOSITORY             TAG       IMAGE ID       CREATED        SIZE
alpine                 latest    d4ff818577bc   40 hours ago   5.6MB
xyz349925756/busybox   1.33.1    69593048aa3a   9 days ago     1.24MB
busybox                1.33.1    69593048aa3a   9 days ago     1.24MB
nginx                  latest    d1a364dc548d   3 weeks ago    133MB
[root@docker01 ~]# docker load < /data/docker/busybox.1.33.1_1.txt.tar 
5a281580ed23: Loading layer [==================================================>]  6.144kB/6.144kB
Loaded image ID: sha256:ddee1359ecbccdde9d28ea70bc8aa42ba2f09ca1531036c38372a7cda9ba3b2b
[root@docker01 ~]# docker images
REPOSITORY             TAG       IMAGE ID       CREATED          SIZE
<none>                 <none>    ddee1359ecbc   13 minutes ago   1.24MB
alpine                 latest    d4ff818577bc   40 hours ago     5.6MB
busybox                1.33.1    69593048aa3a   9 days ago       1.24MB
xyz349925756/busybox   1.33.1    69593048aa3a   9 days ago       1.24MB
nginx                  latest    d1a364dc548d   3 weeks ago      133MB
[root@docker01 ~]# docker tag ddee1359ecbc docker.io/xyz349925756/busybox:latest
[root@docker01 ~]# docker images
REPOSITORY             TAG       IMAGE ID       CREATED          SIZE
xyz349925756/busybox   latest    ddee1359ecbc   14 minutes ago   1.24MB
alpine                 latest    d4ff818577bc   40 hours ago     5.6MB
busybox                1.33.1    69593048aa3a   9 days ago       1.24MB
xyz349925756/busybox   1.33.1    69593048aa3a   9 days ago       1.24MB
nginx                  latest    d1a364dc548d   3 weeks ago      133MB

```

Usage:  docker save [OPTIONS] IMAGE [IMAGE...]   **#保存镜像**

Usage:  docker load [OPTIONS]                                   **#恢复镜像**

​              -i  

```bash
[root@docker01 ~]# docker save xyz349925756/busybox:1.33.1 > /data/docker/xyz349925756_busybox\:1.33.1.tar
```

**从上面实践得出如果文件名中有特殊符号使用\ 转义。但是/ 不能转义改成_使用**



### 日志查看

```BASH
[root@docker01 ~]# docker logs test01 

Usage:  docker logs [OPTIONS] CONTAINER

Options:
      --details        Show extra details provided to logs
  -f, --follow         Follow log output
      --since string   Show logs since timestamp (e.g. 2013-01-02T13:23:37Z) or relative (e.g. 42m for 42 minutes)
  -n, --tail string    Number of lines to show from the end of the logs (default "all")
  -t, --timestamps     Show timestamps
      --until string   Show logs before a timestamp (e.g. 2013-01-02T13:23:37Z) or relative (e.g. 42m for 42
                       minutes)

```



### 查看详细信息

```BASH
[root@docker01 ~]# docker inspect test01 
[
    {
        "Id": "3caeb5ca76ae3b74aa7d644df477ac80170a072ac16bc94d534e8d5c3b9cfe18",
        "Created": "2021-06-17T14:39:37.841342263Z",
        "Path": "/bin/sh",
        "Args": [],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 5580,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2021-06-17T14:39:38.605909225Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:ddee1359ecbccdde9d28ea70bc8aa42ba2f09ca1531036c38372a7cda9ba3b2b",
        "ResolvConfPath": "/var/lib/docker/containers/3caeb5ca76ae3b74aa7d644df477ac80170a072ac16bc94d534e8d5c3b9cfe18/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/3caeb5ca76ae3b74aa7d644df477ac80170a072ac16bc94d534e8d5c3b9cfe18/hostname",
        "HostsPath": "/var/lib/docker/containers/3caeb5ca76ae3b74aa7d644df477ac80170a072ac16bc94d534e8d5c3b9cfe18/hosts",
        "LogPath": "/var/lib/docker/containers/3caeb5ca76ae3b74aa7d644df477ac80170a072ac16bc94d534e8d5c3b9cfe18/3caeb5ca76ae3b74aa7d644df477ac80170a072ac16bc94d534e8d5c3b9cfe18-json.log",
        "Name": "/test01",
        "RestartCount": 0,
        "Driver": "overlay2",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": null,
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "default",
            "PortBindings": {},
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "CapAdd": null,
            "CapDrop": null,
            "CgroupnsMode": "host",
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "private",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "ConsoleSize": [
                0,
                0
            ],
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": [],
            "BlkioDeviceReadBps": null,
            "BlkioDeviceWriteBps": null,
            "BlkioDeviceReadIOps": null,
            "BlkioDeviceWriteIOps": null,
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DeviceRequests": null,
            "KernelMemory": 0,
            "KernelMemoryTCP": 0,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": null,
            "OomKillDisable": false,
            "PidsLimit": null,
            "Ulimits": null,
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "MaskedPaths": [
                "/proc/asound",
                "/proc/acpi",
                "/proc/kcore",
                "/proc/keys",
                "/proc/latency_stats",
                "/proc/timer_list",
                "/proc/timer_stats",
                "/proc/sched_debug",
                "/proc/scsi",
                "/sys/firmware"
            ],
            "ReadonlyPaths": [
                "/proc/bus",
                "/proc/fs",
                "/proc/irq",
                "/proc/sys",
                "/proc/sysrq-trigger"
            ]
        },
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/ef2fa373cb0309aa310438487e0a80a29441a9171d9c93535213cdfd315a4458-init/diff:/var/lib/docker/overlay2/fc1b1577b7f8363044c86bce7554312aad88fb4b2164e40d24caac2967c3dbb7/diff:/var/lib/docker/overlay2/f1288d5425205c92d20bb3147721fc56561f07537f73d7e46b4414dc27d62f19/diff",
                "MergedDir": "/var/lib/docker/overlay2/ef2fa373cb0309aa310438487e0a80a29441a9171d9c93535213cdfd315a4458/merged",
                "UpperDir": "/var/lib/docker/overlay2/ef2fa373cb0309aa310438487e0a80a29441a9171d9c93535213cdfd315a4458/diff",
                "WorkDir": "/var/lib/docker/overlay2/ef2fa373cb0309aa310438487e0a80a29441a9171d9c93535213cdfd315a4458/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [],
        "Config": {
            "Hostname": "3caeb5ca76ae",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": true,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/bin/sh"
            ],
            "Image": "xyz349925756/busybox:latest",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {}
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "bfaf8824404a32349d2ae93a9314928ef8bfe8934bf30c901f6938e4d42eb8c7",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {},
            "SandboxKey": "/var/run/docker/netns/bfaf8824404a",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "94292a568558e488f7000d93c7fbe9ffe07a51d465b14649314dd666a1316903",
            "Gateway": "172.17.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.17.0.2",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:11:00:02",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "ee8bedd19a8c8dab274d5116c693a771dfa28ff511dc9a1f895f48ab5f9f449d",
                    "EndpointID": "94292a568558e488f7000d93c7fbe9ffe07a51d465b14649314dd666a1316903",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:02",
                    "DriverOpts": null
                }
            }
        }
    }
]

```

很详细的信息，这里就不一一标注了。

### 端口映射|目录

```BASH
[root@docker01 ~]# docker run -dt --name=nginx_01 -p 80:80 -v /data/docker/html:/usr/share/nginx/html/ nginx:latest 
d6f2f9393c71805ca167aceb2f488b4d078ad874409d878bfdd8ad1f6cfecbe9
[root@docker01 ~]# docker ps -a
CONTAINER ID   IMAGE                         COMMAND                  CREATED         STATUS         PORTS                               NAMES
d6f2f9393c71   nginx:latest                  "/docker-entrypoint.…"   9 seconds ago   Up 8 seconds   0.0.0.0:80->80/tcp, :::80->80/tcp   nginx_01
3caeb5ca76ae   xyz349925756/busybox:latest   "/bin/sh"                7 minutes ago   Up 7 minutes                                       test01
```

![image-20210617224811002](C:\Users\goo\AppData\Roaming\Typora\typora-user-images\image-20210617224811002.png)

这里为什么403？是因为存储里面什么都没有

```BASH
[root@docker01 ~]# echo "hello world" >/data/docker/html/index.html
```

![image-20210617224958115](C:\Users\goo\AppData\Roaming\Typora\typora-user-images\image-20210617224958115.png)

```bash
-v host_path:container_path
[root@docker01 ~]# wget www.baidu.com -O index.html 
[root@docker01 ~]# mv index.html /data/docker/html/

```

![image-20210617225443954](C:\Users\goo\AppData\Roaming\Typora\typora-user-images\image-20210617225443954.png)

```BASH 
[root@docker01 ~]# docker inspect nginx_01 |grep -A 9 'Mounts'
        "Mounts": [
            {
                "Type": "bind",
                "Source": "/data/docker/html",
                "Destination": "/usr/share/nginx/html",
                "Mode": "",
                "RW": true,
                "Propagation": "rprivate"
            }
        ],
#查看挂载信息
```



### 传递环境变量

```shell
-e　``variate_name＝``variate_value
[root@docker01 ~]# docker run --rm -e E_OPT=test docker.io/xyz349925756/busybox:latest printenv
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=e5998a2ea417
E_OPT=test
HOME=/root
#多变量多个-e  跟  -p  -v 一样使用
```

### 容器内下载软件

redhat  centos fedora   yum  dnf

ubuntu apt-get

alpine apt

```BASH
# docker exec -dt nginx_with_baidu /bin/bash
# docker exec -it nginx_with_baidu /bin/bash
# tee /etc/apt/sources.list << EOF
deb http://mirrors.163.com/debian/ jessie main non-free contrib
deb http://mirrors.163.com/debian/ jessie-updates main non-free contrib
EOF
#容器内配置yum 源
#yum update && yum install curl -y 
# curl -k https://www.baidu.com
[root@docker01 ~]# docker commit -p acf79798ce19 xyz349925756/nginx:curl
[root@docker01 ~]#docker push docker.io/xyz349925756/nginx:curl 
```



## Dockerfile

Dockerfile 是一个文本文件，其内包含了一条条的 **指令(Instruction)**，每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建。

```BASH
[root@docker01 /data/docker/html]# vim Dockerfile 
[root@docker01 /data/docker/html]# cat Dockerfile 
FROM nginx
COPY index.html /usr/share/nginx/html

[root@docker01 /data/docker/html]# docker build -t nginx:v1 /data/docker/html/
Sending build context to Docker daemon   5.12kB
Step 1/2 : FROM nginx
 ---> d1a364dc548d
Step 2/2 : COPY index.html /usr/share/nginx/html
 ---> 7091a9edba22
Successfully built 7091a9edba22
Successfully tagged nginx:v1

[root@docker01 /data/docker/html]# docker images |grep v1
nginx                  v1        7091a9edba22   About a minute ago   133MB

      test01
[root@docker01 /data/docker/html]# docker run -dt --name=nginx_test_01 -p 80:80 nginx:v1
24a122bea30ad8c064d0f80a31a31297df9774e2754f8f13124a8ea8d2b818ab
[root@docker01 /data/docker/html]# docker ps -a
CONTAINER ID   IMAGE                         COMMAND                  CREATED          STATUS                      PORTS                               NAMES
24a122bea30a   nginx:v1                      "/docker-entrypoint.…"   10 seconds ago   Up 8 seconds                0.0.0.0:80->80/tcp, :::80->80/tcp   nginx_test_01

[root@docker01 /data/docker/html]# curl -I 172.16.0.150:80
HTTP/1.1 200 OK
Server: nginx/1.21.0
Date: Fri, 18 Jun 2021 01:53:24 GMT
Content-Type: text/html
Content-Length: 2381
Last-Modified: Thu, 17 Jun 2021 14:53:48 GMT
Connection: keep-alive
ETag: "60cb61fc-94d"
Accept-Ranges: bytes

```



### docker build 语法

```bash
[root@docker01 /data/docker/html]# docker build --help
Usage:  docker build [OPTIONS] PATH | URL | -
Build an image from a Dockerfile
Options:
      --add-host list           Add a custom host-to-IP mapping (host:ip)
      --build-arg list          Set build-time variables
      --cache-from strings      Images to consider as cache sources
      --cgroup-parent string    Optional parent cgroup for the container
      --compress                Compress the build context using gzip
      --cpu-period int          Limit the CPU CFS (Completely Fair Scheduler) period
      --cpu-quota int           Limit the CPU CFS (Completely Fair Scheduler) quota
  -c, --cpu-shares int          CPU shares (relative weight)
      --cpuset-cpus string      CPUs in which to allow execution (0-3, 0,1)
      --cpuset-mems string      MEMs in which to allow execution (0-3, 0,1)
      --disable-content-trust   Skip image verification (default true)
  -f, --file string             Name of the Dockerfile (Default is 'PATH/Dockerfile')
      --force-rm                Always remove intermediate containers
      --iidfile string          Write the image ID to the file
      --isolation string        Container isolation technology
      --label list              Set metadata for an image
  -m, --memory bytes            Memory limit
      --memory-swap bytes       Swap limit equal to memory plus swap: '-1' to enable unlimited swap
      --network string          Set the networking mode for the RUN instructions during build (default "default")
      --no-cache                Do not use cache when building the image
      --pull                    Always attempt to pull a newer version of the image
  -q, --quiet                   Suppress the build output and print image ID on success
      --rm                      Remove intermediate containers after a successful build (default true)
      --security-opt strings    Security options
      --shm-size bytes          Size of /dev/shm
  -t, --tag list                Name and optionally a tag in the 'name:tag' format
      --target string           Set the target build stage to build.
      --ulimit ulimit           Ulimit options (default [])
```



### FROM

FROM 以指定镜像为基础构建

一个dockerfile  中第一条必须的语句。

FROM 对象是各种构建好的服务或者官方提供的系统：nginx 、PHP、httpd、centos、ubuntu等

scratch  表示的是空镜像虚拟镜像，不存在实体file，GO语言经常使用这种构建镜像



### RUN

run 指令是用来执行命令行命令的。常用命令之一

shell 格式： RUN echo 'xxxx' >/path

​                    RUN <命令>

exec格式:   RUN [“可执行文件”，“参数1”，“参数2”]

注意：不能使用多条RUN 这样系统会认为一条run一层镜像，导致系统臃肿不堪

如果多条命令可以使用 && 逻辑关系运算符,下面例子

```dockerfile
FROM debian:stretch
RUN buildDeps='gcc libc6-dev make wget' \
    && apt-get update \
    && apt-get install -y $buildDeps \
    && wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz" \
    && mkdir -p /usr/src/redis \
    && tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1 \
    && make -C /usr/src/redis \
    && make -C /usr/src/redis install \
    && rm -rf /var/lib/apt/lists/* \
    && rm redis.tar.gz \
    && rm -r /usr/src/redis \
    && apt-get purge -y --auto-remove $buildDeps
```

Dockerfile 支持 Shell 类的行尾添加 `\` 的命令换行方式，以及行首 `#` 进行注释的格式。良好的格式，比如换行、缩进、注释等，会让维护、排障更为容易，这是一个比较好的习惯。

#### 镜像构建上下文路径问题

刚开始编辑dockerfile出现

```bash
COPY /data/html/index.html /usr/share/nginx/html
```

发现执行之后报错，这是因为copy 格式错误

**正确的应该是**

```BASH
#vim Dockerfile
FROM nginx
COPY index.html /usr/share/nginx/html

#docker build -t nginx:v1 /data/html
```

然后使用构建命令构建，这样是不是就更明白了。

```BASH
#vim nginx_01
FROM nginx
COPY index.html /usr/share/nginx/html
#docker build -f /data/dockerfile/nginx_01 /data/html
上面是有问题的。dockerfile 中提及的文件必须跟dockerfile在同一个目录下
#cp /data/html/index.html /data/dockerfile/
#docker build -f /data/dockerfile/nginx_01 /data/dockerfile/
这样才不会报错。
```

build 还支持URL 构建

```HTML
 docker build https://github.com/twang2218/gitlab-ce-zh.git#:11.1
```

这行命令指定了构建所需的 Git repo，并且指定默认的 `master` 分支，构建目录为 `/11.1/`，然后 Docker 就会自己去 `git clone` 这个项目、切换到指定分支、并进入到指定目录后开始构建。

build 支持tar

```BASH
docker build http://server/context.tar.gz
```

如果所给出的 URL 不是个 Git repo，而是个 `tar` 压缩包，那么 Docker 引擎会下载这个包，并自动解压缩，以其作为上下文，开始构建。

#### 从标准输入中读取dockefile

```bash
[root@docker01 /data/docker/html]# cat Dockerfile |docker build .
Sending build context to Docker daemon   5.12kB
Step 1/2 : FROM nginx
 ---> d1a364dc548d
Step 2/2 : COPY index.html /usr/share/nginx/html
 ---> Using cache
 ---> 7091a9edba22
Successfully built 7091a9edba22
[root@docker01 /data/docker/html]# docker build . < Dockerfile
Sending build context to Docker daemon   5.12kB
Step 1/2 : FROM nginx
 ---> d1a364dc548d
Step 2/2 : COPY index.html /usr/share/nginx/html
 ---> Using cache
 ---> 7091a9edba22
Successfully built 7091a9edba22

docker build - 这种输入要没有上下文的情况使用。
上下文就是上面提到过的路径问题。
[root@docker01 /data/docker/html]# cat Dockerfile |docker build -
Sending build context to Docker daemon  2.048kB
Step 1/2 : FROM nginx
 ---> d1a364dc548d
Step 2/2 : COPY index.html /usr/share/nginx/html
COPY failed: file not found in build context or excluded by .dockerignore: stat index.html: file does not exist
报错了
```

从标准输入中读取上下文压缩包进行构建

```BASH
docker build - < context.tar.gz
支持：gzip、bzip2 以及 xz
```

### COPY

语法：

​         COPY  [--chown=<user>:<group>]   <src> ... <dest>

​         COPY  [--chown=<user>:<group>]   [<src1>,<src2> ... <dest>]

`COPY` 指令将从构建上下文目录中 `<源路径>` 的文件/目录复制到新的一层的镜像内的 `<目标路径>` 位置

`<源路径>` 可以是多个，甚至可以是通配符，其通配符规则要满足 Go 的 [`filepath.Match`](https://golang.org/pkg/path/filepath/#Match) 规则

```BASH
COPY index.html /usr/share/nginx/html
COPY index.* /usr/share/nginx/html
COPY ind?.html /usr/share/nginx/html
```

`<目标路径>` 可以是容器内的绝对路径，也可以是相对于工作目录的相对路径（工作目录可以用 `WORKDIR` 指令来指定）。目标路径不需要事先创建，如果目录不存在会在复制文件前先行创建缺失目录。

还需要注意一点，使用 `COPY` 指令，源文件的各种元数据都会保留。比如读、写、执行权限、文件变更时间等。这个特性对于镜像定制很有用。特别是构建相关文件都在使用 Git 进行管理的时候。

在使用该指令的时候还可以加上 `--chown=<user>:<group>` 选项来改变文件的所属用户及所属组。

```BASH
COPY --chown=55:mygroup files* /mydir/
COPY --chown=bin files* /mydir/
COPY --chown=1 files* /mydir/
COPY --chown=10:11 files* /mydir/
```

### ADD

COPY 的进阶版`<源路径>` 可以是一个 `URL`，这种情况下，Docker 引擎会试图去下载这个链接的文件放到 `<目标路径>` 去。下载文件权限默认为600，如果不是想要的权限还需要更改提交一个RUN，多一层，不建议使用

不如RUN  &&  wget   |  curl  最后清理下载的文件，带解压功能支持tar gzip xz bzip 。

```dockerfile
FROM scratch
ADD ubuntu-xenial-core-cloudimg-amd64-root.tar.gz /
```

ADD指令会导致构建缓存失败，需要解压的环境才使用

一样可以加权限选项使用  --chown <user>:<group>

### CMD

shell : CMD <command>

exec :CMD ["execfile" ,"arg1","arg2"...]

用 CMD 指定具体的参数。
之前介绍容器的时候曾经说过，Docker 不是虚拟机，容器就是进程。既然是进程，那么在启动容器的时候，需要指定所运行的程序及参数。CMD 指令就是用于指定默认的容器主进程的启动命令的。

在运行时可以指定新的命令来替代镜像设置中的这个默认命令，比如，ubuntu 镜像默认的 CMD 是 /bin/bash，如果我们直接 docker run -it ubuntu 的话，会直接进入 bash。我们也可以在运行时指定运行别的命令，如 **docker run -it ubuntu cat /etc/os-release**。这就是用 cat /etc/os-release 命令替换了默认的 /bin/bash 命令了，输出了系统版本信息。

在指令格式上，一般推荐使用 exec 格式，这类格式在解析时会被解析为 JSON 数组，因此一定要使用双引号 "，而不要使用单引号。

如果使用 shell 格式的话，实际的命令会被包装为 sh -c 的参数的形式进行执行。

```CMD
CMD echo $HOME
```

在实际执行中，会将其变更为：

```shell
 CMD [ "sh", "-c", "echo $HOME" ]
```

这就是为什么我们可以使用环境变量的原因，因为这些环境变量会被 shell 进行解析处理。

提到 `CMD` 就不得不提容器中应用在前台执行和后台执行的问题。这是初学者常出现的一个混淆。

Docker 不是虚拟机，容器中的应用都应该以前台执行，而不是像虚拟机、物理机里面那样，用 `systemd` 去启动后台服务，容器内没有后台服务的概念。

一些初学者将 `CMD` 写为：

```bash
CMD service nginx start
```

然后发现容器执行后就立即退出了。甚至在容器内去使用 `systemctl` 命令结果却发现根本执行不了。这就是因为没有搞明白前台、后台的概念，没有区分容器和虚拟机的差异，依旧在以传统虚拟机的角度去理解容器。

对于容器而言，其启动程序就是容器应用进程，容器就是为了主进程而存在的，主进程退出，容器就失去了存在的意义，从而退出，其它辅助进程不是它需要关心的东西。

而使用 `service nginx start` 命令，则是希望 upstart 来以后台守护进程形式启动 `nginx` 服务。而刚才说了 `CMD service nginx start` 会被理解为 `CMD [ "sh", "-c", "service nginx start"]`，因此主进程实际上是 `sh`。那么当 `service nginx start` 命令结束后，`sh` 也就结束了，`sh` 作为主进程退出了，自然就会令容器退出。

正确的做法是直接执行 `nginx` 可执行文件，并且要求以前台形式运行。比如：

```bash
CMD ["nginx", "-g", "daemon off;"]
```

### ENTRYPOINT 

`CMD`和`ENTRYPOINT`这两个指令用于在`Dockerfile`和`Docker Compose files`里配置容器的运行命令。

**`Entrypoint`指令用于设定容器启动时第一个运行的命令及其参数**。

任何使用`docker run <image>`命令传入的参数都会附加在`entrypoint`指令之后，并且用此命令传入的参数会覆盖在Dockerfile中使用`CMD`指令设定的值。比如`docker run <image> bash`命令会将`bash`命令附加在`entrypoint`指令设定的值的后面。

##### Dockerfile ENTRYPOINT

Dockerfiles使用`entrypoint`全大写的形式来标识此指令。有如下几种不同的方式来定义它。

###### The exec syntax

使用`exec`形式时，你需要将命令和其参数以JSON数组的格式书写。这意味着你需要使用双引号，例如：

```bash
ENTRYPOINT ["executable", "param1", "param2"]
```

使用这种语法，Docker不会使用shell来运行。这意味着通常的shell处理过程不会发生。如果你需要在shell环境中运行，那么你可以这样做：

```bash
ENTRYPOINT [ "sh", "-c", "echo $HOME" ]
```

###### Using an entrypoint script

另外一种形式是使用一个脚本作为`ENTRYPOINT`的值。按照惯例来说，脚本名中通常包含`entrypoint`关键字。在这个脚本中，你可以做相关的配置，设置环境变量等，例如下面代码：

```objectivec
COPY ./docker-entrypoint.sh /
ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["postgres"]
```

例如，下面就是[Postgres官方镜像](https://links.jianshu.com/go?to=https%3A%2F%2Fhub.docker.com%2F_%2Fpostgres%2F)中的`ENTRYPOINT`脚本中的内容：

```bash
#!/bin/bash
set -e
if [ "$1" = 'postgres' ]; then
    chown -R postgres "$PGDATA"
    if [ -z "$(ls -A "$PGDATA")" ]; then
        gosu postgres initdb
    fi
    exec gosu postgres "$@"
fi
exec "$@"
```

##### Docker Compose entrypoint

在docker compose中使用entrypoint指令的方法与在Dockerfiles中一样，唯一不同的是在compose中`entrypoint`使用全小写的形式。

你也可以在`docker-compose.yml`文件中以列表的形式来定义它的值：

```bash
entrypoint:
    - php
    - -d
    - zend_extension=/usr/local/lib/php/xdebug.so
    - -d
    - memory_limit=-1
    - vendor/bin/phpunit
```

##### Overriding Entrypoint

你可以通过使用命令`docker run --entrypoint`或`docker-compose run --entrypoint`来覆盖镜像中的`ENTRYPOINT`指令的内容



#### ENV设置环境变量

ENV <key> <value>

ENV <key1>=<value1> <key2>=<value2>...

设置环境变量上面调用的run  cmd 都可以直接使用这里定义的变量

```dockerfile
ENV VERSION=1.0 DEBUG=on \   
    NAME="Happy Feet" #有空格用引号
```

## ARG 构建参数

构建参数，与 ENV 作用一至。不过作用域不一样。ARG 设置的环境变量仅对 Dockerfile 内有效，也就是说只有 docker build 的过程中有效，构建好的镜像内不存在此环境变量。

构建命令 docker build 中可以用 --build-arg <参数名>=<值> 来覆盖。

```BASH
ARG <参数名>[=<默认值>]
```



### volume 定义匿名卷

定义匿名数据卷。在启动容器时忘记挂载数据卷，会自动挂载到匿名卷。

作用：

- 避免重要的数据，因容器重启而丢失，这是非常致命的。
- 避免容器不断变大。

```BASH
VOLUME ["<路径1>", "<路径2>"...]
VOLUME <路径>
```

### EXPOSE

暴露端口

格式为 `EXPOSE <端口1> [<端口2>...]`。

`EXPOSE` 指令是声明运行时容器提供服务端口，这只是一个声明，在运行时并不会因为这个声明应用就会开启这个端口的服务。在 Dockerfile 中写入这样的声明有两个好处，一个是帮助镜像使用者理解这个镜像服务的守护端口，以方便配置映射；另一个用处则是在运行时使用随机端口映射时，也就是 `docker run -P` 时，会自动随机映射 `EXPOSE` 的端口。

### WORKDIR

格式为 `WORKDIR <工作目录路径>`。

使用 `WORKDIR` 指令可以来指定工作目录（或者称为当前目录），以后各层的当前目录就被改为指定的目录，如该目录不存在，`WORKDIR` 会帮你建立目录。

*已存在目录不知道会不会被覆盖*

### USER 

格式：`USER <用户名>[:<用户组>]`

`USER` 指令和 `WORKDIR` 相似，都是改变环境状态并影响以后的层。`WORKDIR` 是改变工作目录，`USER` 则是改变之后层的执行 `RUN`, `CMD` 以及 `ENTRYPOINT` 这类命令的身份。

当然，和 `WORKDIR` 一样，`USER` 只是帮助你切换到指定用户而已，**这个用户必须是事先建立好的，否则无法切换。**

```dockerfile
RUN groupadd -r redis && useradd -r -g redis redis
USER redis
RUN [ "redis-server" ]
```

如果以 `root` 执行的脚本，在执行期间希望改变身份，比如希望以某个已经建立好的用户来运行某个服务进程，不要使用 `su` 或者 `sudo`，这些都需要比较麻烦的配置，而且在 TTY 缺失的环境下经常出错。建议使用 `gosu`。

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

为什么要用gosu？

- gosu启动命令时只有一个进程，所以docker容器启动时使用gosu，那么该进程可以做到PID等于1；
- sudo启动命令时先创建sudo进程，然后该进程作为父进程去创建子进程，1号PID被sudo进程占据；

https://github.com/tianon/gosu/releases

### HEALTHCHECK 健康检查

格式：

- `HEALTHCHECK [选项] CMD <命令>`：设置检查容器健康状况的命令
- `HEALTHCHECK NONE`：如果基础镜像有健康检查指令，使用这行可以屏蔽掉其健康检查指令

`HEALTHCHECK` 指令是告诉 Docker 应该如何进行判断容器的状态是否正常，这是 Docker 1.12 引入的新指令。

在没有 `HEALTHCHECK` 指令前，Docker 引擎只可以通过容器内主进程是否退出来判断容器是否状态异常。很多情况下这没问题，但是如果程序进入死锁状态，或者死循环状态，应用进程并不退出，但是该容器已经无法提供服务了。在 1.12 以前，Docker 不会检测到容器的这种状态，从而不会重新调度，导致可能会有部分容器已经无法提供服务了却还在接受用户请求。

而自 1.12 之后，Docker 提供了 `HEALTHCHECK` 指令，通过该指令指定一行命令，用这行命令来判断容器主进程的服务状态是否还正常，从而比较真实的反应容器实际状态。

当在一个镜像指定了 `HEALTHCHECK` 指令后，用其启动容器，初始状态会为 `starting`，在 `HEALTHCHECK` 指令检查成功后变为 `healthy`，如果连续一定次数失败，则会变为 `unhealthy`。

`HEALTHCHECK` 支持下列选项：

- `--interval=<间隔>`：两次健康检查的间隔，默认为 30 秒；
- `--timeout=<时长>`：健康检查命令运行超时时间，如果超过这个时间，本次健康检查就被视为失败，默认 30 秒；
- `--retries=<次数>`：当连续失败指定次数后，则将容器状态视为 `unhealthy`，默认 3 次。

和 `CMD`, `ENTRYPOINT` 一样，`HEALTHCHECK` 只可以出现一次，如果写了多个，只有最后一个生效。

在 `HEALTHCHECK [选项] CMD` 后面的命令，格式和 `ENTRYPOINT` 一样，分为 `shell` 格式，和 `exec` 格式。命令的返回值决定了该次健康检查的成功与否：`0`：成功；`1`：失败；`2`：保留，不要使用这个值。

假设我们有个镜像是个最简单的 Web 服务，我们希望增加健康检查来判断其 Web 服务是否在正常工作，我们可以用 `curl` 来帮助判断，其 `Dockerfile` 的 `HEALTHCHECK` 可以这么写：

```Dockerfile
FROM nginx
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*
HEALTHCHECK --interval=5s --timeout=3s \
  CMD curl -fs http://localhost/ || exit 1
```

这里我们设置了每 5 秒检查一次（这里为了试验所以间隔非常短，实际应该相对较长），如果健康检查命令超过 3 秒没响应就视为失败，并且**使用 `curl -fs http://localhost/ || exit 1` 作为健康检查命令**。好像之前学习的ps -ef |grep [n]ginx 小于0 然后执行xxx 。这里使用exit 返回。没有使用 $?

使用 `docker build` 来构建这个镜像：

```bash
$ docker build -t myweb:v1 .
```

构建好了后，我们启动一个容器：

```bash
$ docker run -d --name web -p 80:80 myweb:v1
```

当运行该镜像后，可以通过 `docker container ls` 看到最初的状态为 `(health: starting)`：

```bash
$ docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                            PORTS               NAMES
03e28eb00bd0        myweb:v1            "nginx -g 'daemon off"   3 seconds ago       Up 2 seconds (health: starting)   80/tcp, 443/tcp     web
```

在等待几秒钟后，再次 `docker container ls`，就会看到健康状态变化为了 `(healthy)`：

```bash
$ docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                    PORTS               NAMES
03e28eb00bd0        myweb:v1            "nginx -g 'daemon off"   18 seconds ago      Up 16 seconds (healthy)   80/tcp, 443/tcp     web
```

如果健康检查连续失败超过了重试次数，状态就会变为 `(unhealthy)`。

为了帮助排障，健康检查命令的输出（包括 `stdout` 以及 `stderr`）都会被存储于健康状态里，可以用 `docker inspect` 来查看。

```bash
$ docker inspect --format '{{json .State.Health}}' web | python -m json.tool
{
    "FailingStreak": 0,
    "Log": [
        {
            "End": "2016-11-25T14:35:37.940957051Z",
            "ExitCode": 0,
            "Output": "<!DOCTYPE html>\n<html>\n<head>\n<title>Welcome to nginx!</title>\n<style>\n    body {\n        width: 35em;\n        margin: 0 auto;\n        font-family: Tahoma, Verdana, Arial, sans-serif;\n    }\n</style>\n</head>\n<body>\n<h1>Welcome to nginx!</h1>\n<p>If you see this page, the nginx web server is successfully installed and\nworking. Further configuration is required.</p>\n\n<p>For online documentation and support please refer to\n<a href=\"http://nginx.org/\">nginx.org</a>.<br/>\nCommercial support is available at\n<a href=\"http://nginx.com/\">nginx.com</a>.</p>\n\n<p><em>Thank you for using nginx.</em></p>\n</body>\n</html>\n",
            "Start": "2016-11-25T14:35:37.780192565Z"
        }
    ],
    "Status": "healthy"
}
```



### ONBUILD

ONBUILD指令可以为镜像添加触发器。其参数是任意一个Dockerfile 指令。

当我们在一个Dockerfile文件中加上ONBUILD指令，该指令对利用该Dockerfile构建镜像（比如为A镜像）不会产生实质性影响。

但是当我们编写一个新的Dockerfile文件来基于A镜像构建一个镜像（比如为B镜像）时，这时构造A镜像的Dockerfile文件中的ONBUILD指令就生效了，在构建B镜像的过程中，首先会执行ONBUILD指令指定的指令，然后才会执行其它指令。

需要注意的是，如果是再利用B镜像构造新的镜像时，那个ONBUILD指令就无效了，也就是说只能再构建子镜像中执行，对孙子镜像构建无效。其实想想是合理的，因为在构建子镜像中已经执行了，如果孙子镜像构建还要执行，相当于重复执行，这就有问题了。

利用ONBUILD指令,实际上就是相当于创建一个模板镜像，后续可以根据该模板镜像创建特定的子镜像，需要在子镜像构建过程中执行的一些通用操作就可以在模板镜像对应的dockerfile文件中用ONBUILD指令指定。 从而减少dockerfile文件的重复内容编写。

我们来看一个简单例子。

1、先编写一个Dockerfile文件，内容如下：

```
#test
FROM ubuntu
ONBUILD RUN mkdir mydir
```

利用上面的dockerfile文件构建镜像： docker build -t imagea .
利用imagea镜像创建容器： docker run --name test1 -it imagea /bin/bash

我们发现test1容器的根目录下并没有mydir目录。说明ONBUILD指令指定的指令并不会在自己的构建中执行。

2、再编写一个新的Dockerfile文件，内容 如下

```
#test
FROM imagea
```

注意，该构建准备使用的基础镜像是上面构造出的镜像imagea
利用上面的dockerfile文件构建镜像： docker build -t imageb .
利用imagea镜像创建容器： docker run --name test2 -it imageb /bin/bash

我们发现test2容器的根目录下有mydir目录，说明触发器执行了。 这个其实从构建imageb的输出日志就可看出。日志如下：

```shell
xxx@ubuntu:~/myimage$ docker build -t imageb .
Sending build context to Docker daemon 15.87 kB
Step 1 : FROM imagea
# Executing 1 build trigger...
Step 1 : RUN mkdir mydir
 ---> Running in e16c35c94b03
 ---> 4b393d1610a6
Removing intermediate container e16c35c94b03
Successfully built 0f63b8e04d82
```

FROM指令执行之后，就立即执行的是触发器（ONBUILD指令指定的指令)

# 镜像优化

**第一步优化：使用轻量化基础镜像**

相较于基于其他 Linux 发行版（例如 Ubuntu）的镜像，基于 Alpine 或 BusyBox 的镜像非常小。这是因为 Alpine 镜像和类似的其他镜像都经过了优化，其中仅包含最少的必须的软件包。

**第二步优化：多阶段构建**

通过多阶段构建，我们可以在 Dockerfile 中使用多个基础镜像，并将编译成品、配置文件等从一个阶段复制到另一个阶段，这样我们就可以丢弃不需要的东西。

通过将 Dockerfile 修改为如下内容，我们最终得到的镜像大小为 91.5MB。请记住，来自第一阶段（第 1-4 行）的镜像不会被自动删除，Docker 将它保存在 cache 中，如果我们在另一个构建镜像过程中执行了相同的阶段，就可以使镜像构建更快。所以你必须手动删除第一阶段镜像。

# Docker多阶段构建



https://wordpress.org/latest.tar.gz

后期补上。

实现的想法  php  nginx 在一个容器。



# Docker 的四种网络模型

主机，桥接，NAT 

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

docker 静态IP地址
查看支持网络类型
[root@Docker01 ~]# docker network ls
NETWORK ID     NAME            DRIVER    SCOPE
344e133f97b5   bridge          bridge    local
a611aff4793e   harbor_harbor   bridge    local
3e5e4ed821f7   host            host      local
b67fd6c0fc61   none            null      local

docker run --network=xxx  启动时候格式
none : 无网络模式
bridge ： 默认模式，相当于NAT
host : 公用宿主机Network NameSapce
container：与其他容器公用Network Namespace
--network network              连接容器的网络
--network-alias list            容器的网络别名
```

桥接模式中pipework 脚本分配固定IP但是重启之后IP会消失

配置桥接网络（违背docker的安全隔离原则）以下内容来自腾讯课堂**作为一种方法**

```BASH
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

```BASH
overlay解决方案
为支持容器跨主机通信，Docker 提供了 overlay driver，使用户可以创建基于 VxLAN 的 overlay 网络。VxLAN 可将二层数据封装到 UDP 进行传输，VxLAN 提供与 VLAN 相同的以太网二层服务，但是拥有更强的扩展性和灵活性。

Docerk overlay 网络需要一个 key-value 数据库用于保存网络状态信息，包括 Network、Endpoint、IP 等。Consul、Etcd 和 ZooKeeper 都是 Docker 支持的 key-vlaue 软件，这里使用 Consul。

实验环境
docker01	    docker02	    docker03
192.168.1.120	192.168.1.100	192.168.1.120
consul	        node1	        node2

firewalld,selinux 都关闭的实验环境

docker01

[root@Docker01 ~]# docker search consul
[root@Docker01 ~]# docker pull progrium/consul
 
[root@docker01 ~]# docker run -d -p 8500:8500 -h consul --name consul --restart always progrium/consul -server -bootstrap
命令解释：-d：后台运行；-p 8500:8500：端口映射；-h consu：主机名（注意不是容器名）；--name consul：容器名；--restart always：跟随docker的启动而启动容器；-server：server端；-bootstrap：以单节点运行；

node01/02
[root@docker02 ~]# vim /usr/lib/systemd/system/docker.service
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --cluster-store=consul://192.168.1.120:8500 --cluster-advertise=eth0:2376     
[root@docker02 ~]# systemctl daemon-reload && systemctl restart docker
开启网卡混杂模式
[root@docker02 ~]# ifconfig eth0 promisc

node01
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

node2
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

自定义网段的创建网络 busybox 是一个工具容器，docker中的瑞士军刀
node01
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

node02
[root@docker03 ~]# docker run -dit --name ov_test1 --network overlay_net02 busybox
[root@docker03 ~]# docker exec -it ov_test1 ping -w 2 -c 3 ov_test0
PING ov_test0 (172.16.0.1): 56 data bytes
64 bytes from 172.16.0.1: seq=0 ttl=64 time=1.134 ms
64 bytes from 172.16.0.1: seq=1 ttl=64 time=2.577 ms
--- ov_test0 ping statistics ---
3 packets transmitted, 2 packets received, 33% packet loss
round-trip min/avg/max = 1.134/1.855/2.577 ms

详细信息可以参考另外笔记文档
```

```BASH
macvlan解决方案
一些程序，特别是应用程序或者网络流量监控程序，期望直接连接到物理网络，这种情况下，可使用Macvlan网络模式，给每个容器的虚拟网络接口配置一个mac地址，使得连接容器，看起来是直接到一个物理主机上。这种情况下，需要在主机上 为macvlan驱动，指定一个物理接口，一起子网与默认网关，甚至使用不同的物理网络接口，隔离navlan网络。但必须了解如下几点：
（1）IP地址溢出，和虚拟网络传播（VLAN spreads）将很容易导致网络损坏。例如网络中有众多无效的唯一mac地址，
 （2）网络设备需要处理“混杂模式”， 多个mac地址关联一个物理接口
 （3）如果应用程序可使用 bridge 模式 或者 overlay 模式，从长远来讲，将是更好的选择。



环境
docker02	docker03
192.168.1.100	192.168.1.121
node1	node2

node1
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

node2
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
看样子一台也可以使用

wacvlan多网段解决方案
一个网卡只能创建一个macvlan，想创建多个网段的macvlan需要使用sub-interface
开启网卡混杂模式
node1
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


node2
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
 
开启路由转发
node1
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



node2
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

不能ping通是因为docker network create 创建的虚拟交换机不同 需要ping通需要两台都创建一个网络mac_net2 或者mac_net3 
8021q封装功能
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

容器基本算是完成了，更多的解决方案需要自己在实践中应用。
```



# 如何构建一个自己定制的容器 wordpress环境

准备实验环境

```bash
[root@docker01 ~]# docker image rm -f `docker images | awk '{if (NR>1){print $3}}'`
```













