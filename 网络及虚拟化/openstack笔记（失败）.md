*文中遇到拗口部分是因为使用翻译导致的。后期更改*



自动补全

```sh
[root@controller ~]# yum install -y bash-completion
[root@controller ~]# openstack complete |tee /etc/bash_completion.d/osc.bash_completion > /dev/null
[root@controller ~]# source /etc/bash_completion.d/osc.bash_completion >> ~/.bashrc 

```



# 概念

`简介`

==openstack== `是什么`？

openstack 是一个开源的云平台，国内很多云计算供应商都在使用，简单来说在网上购买一台云服务器就是使用openstack虚拟化出来的，基于kvm技术。与VMware vsphere 相似。都是Iaas 基础即服务解决方案，每个服务都是一个api

官方文档：https://docs.openstack.org/xena/

OpenStack 系统由几个单独安装的关键服务组成。这些服务根据您的云需求协同工作，包含：==Compute 计算, Identity 身份验证, Networking网络, Image镜像, Block Storage块存储, Object Storage对象存储, Telemetry数据收集服务, Orchestration编排技术, and Database services数据库服务.==您可以单独安装这些项目中的任何一个，并将它们单独配置或配置为连接的实体。



openstack 所提供的服务 ：https://www.openstack.org/software/project-navigator/openstack-components#openstack-services

## 概念架构

![openstack_kilo_conceptual_arch](openstack笔记.assets/openstack_kilo_conceptual_arch.png)

openstack 服务关系图

| 名称           | 说明                      | 备注        |
| :------------- | ------------------------- | ----------- |
| **keystone**   | 身份服务 identity service | shared 服务 |
| **ceilometer** |                           |             |
| **horizon**    |                           |             |
| **glance**     | 镜像服务 image service    | shared服务  |
| **nova**       | 计算服务                  |             |
| **sahara**     | 大数据处理框架            |             |
| **neutron**    | 网络服务                  |             |
| **lronic**     | 裸机供应服务              | 硬件周期    |
| **swift**      | 存储服务                  |             |
| **cinder**     | 块存储                    |             |
| **trove**      | 数据库服务                |             |
| **heat**       | 编排服务                  |             |





## 逻辑架构

![openstack-arch-kilo-logical-v1](openstack笔记.assets/openstack-arch-kilo-logical-v1.png)



要理解openstack 必须了解逻辑结构图上的工作流程

> 所有openstack 服务都必须通过公共身份服务进行身份验证 （**keystone**），各个api都是通过公共api相互交互，特殊的需要管理员。
>
> 在内部，OpenStack 服务由多个进程组成。所有服务都至少有一个 API 进程，它监听 API 请求，对它们进行预处理并将它们传递给服务的其他部分。除身份服务外，实际工作由不同的进程完成。
>
> 对于一个服务的进程之间的通信，使用了 AMQP 消息代理。服务的状态存储在数据库中。在部署和配置 OpenStack 云时，您可以在多种消息代理和数据库解决方案中进行选择，例如 RabbitMQ、MySQL、MariaDB 和 SQLite。

用户通过horizon 基于web的用户界面，命令行客户端以及通过浏览器插件curl等工具发出api请求访问openstack.





## 示例架构

演示最少需要两个节点来完成。

![image-20220303163811130](openstack笔记.assets/image-20220303163811130.png)

### 硬件要求

#### **controller** 

控制器节点运行身份服务、image服务、Placement服务、计算管理服务、网络管理部分、各种网络代理和仪表盘，还有支持服务：sql数据库，消息列队，NTP服务

可选部分：块存储、对象存储、编排和Telemetry 服务

至少需要两个网口



#### compute

计算节点运行计算实例管理程序部分，compute 默认使用kvm虚拟化程序，还运行一个网络服务代理，将实例连接到虚拟网络，并通过安全组为实例提供防火墙服务。



#### block storage

块存储，就是一台存储服务区（SDS 至少3n+1台节点）使用管理网络，生产环境单独的网络



#### object storage

对象存储用于存储账户，容器和对象的磁盘，也是使用管理网络。

该服务需要连个节点。每个节点至少一个网口。



#### networking

网络这里有公网和私网的区别。

阿里，腾讯，这些云厂商就是使用公网的IP，其他一般都是使用私网IP。

**专用网络**

![img](openstack笔记.assets/network1-services.png)

**私有网络**

![网络选项 2：自助服务网络 - 服务布局](openstack笔记.assets/network2-services.png)

> 也就是使用路由功能，为LBaaS和FWaaS 等高级服务奠定了基础

这些云计算架构中都不上面所描述的那样，需要根据客户的实际网络环境规划自定义。



# 环境

大多数环境都包含：身份、image、compute、network、仪表盘。存储自行搭建

这里有官网对象存储：https://docs.openstack.org/install-guide/environment.html

controller：1cpu,4GB内存,5GB存储

| 名称       | CPU  | 内存(GB) | 硬盘(GB) |
| ---------- | ---- | -------- | -------- |
| controller | 1    | 4        | 5        |
| compute    | 1    | 2        | 10       |
|            |      |          |          |

linux 系统最小安装，64bit，LVM逻辑卷

## 准备系统

```sh
-- 设置网络 net.ifnames=0 biosdevname=0  把网卡改成ethx样式的
[root@localhost ~]# cat /etc/sysconfig/network-scripts/ifcfg-eth0
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=eth0
UUID=6e8eb2af-cd15-48b6-b3a0-77d520ea5dd2
DEVICE=eth0
ONBOOT=yes
IPADDR=172.16.0.60
PREFIX=24
GATEWAY=172.16.0.1

[root@localhost ~]# cat /etc/sysconfig/network-scripts/ifcfg-eth1
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=eth1
UUID=ffcd65e5-f4fb-4782-b276-b7329c9f1782
DEVICE=eth1
ONBOOT=yes
IPADDR=192.168.10.200
PREFIX=24

[root@localhost ~]# cat /etc/resolv.conf
nameserver 114.114.114.114
nameserver 8.8.8.8

-- 重启网络服务
[root@localhost ~]# systemctl restart NetworkManager

-- 终端提示
[root@localhost ~]# vi /etc/bashrc 
 ...
  # You might want to have e.g. tty in prompt (e.g. more virtual machines)
  # and console windows
  # If you want to do so, just add e.g.
  # if [ "$PS1" ]; then
  #   PS1="[\u@\h:\l \W]\\$ "
      PS1='[\[\e[31;40m\]\u\[\e[33;40m\]@\[\e[34;40m\]\h \[\e[33;40m\]\w\[\e[0m\]]\$ '
  # fi
  # to your custom modification shell script in /etc/profile.d/ directory
...
[root@localhost ~]# source /etc/bashrc 

[root@localhost ~]# vim /etc/issue

---------------------------------------------------------------------------------------------
  
 #######  ########  ######## ##    ##  ######  ########    ###     ######  ##    ## 
##     ## ##     ## ##       ###   ## ##    ##    ##      ## ##   ##    ## ##   ##  
##     ## ##     ## ##       ####  ## ##          ##     ##   ##  ##       ##  ##   
##     ## ########  ######   ## ## ##  ######     ##    ##     ## ##       #####    
##     ## ##        ##       ##  ####       ##    ##    ######### ##       ##  ##   
##     ## ##        ##       ##   ### ##    ##    ##    ##     ## ##    ## ##   ##  
 #######  ##        ######## ##    ##  ######     ##    ##     ##  ######  ##    ## 

---------------------------------------------------------------------------------------------

                IP:\4                       TEL:18183510256

---------------------------------------------------------------------------------------------

-- 更新yum源
https://developer.aliyun.com/mirror/
[root@localhost ~]# mv /etc/yum.repos.d/CentOS-Base.repo{,.back}   #备份源
[root@localhost ~]# curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
[root@localhost ~]# sed -i -e '/mirrors.cloud.aliyuncs.com/d' -e '/mirrors.aliyuncs.com/d' /etc/yum.repos.d/CentOS-Base.repo

# epel
[root@localhost ~]# curl -o /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
[root@localhost ~]# yum makecache

-- 安装常用软件
[root@cotroller ~]# yum install -y tree nmap dos2unix lrzsz nc lsof wget tcpdump htop iftop iotop sysstat nethogs psmisc net-tools bash-completion vim-enhanced bind-utils

```

> **rockylinux更新yum源**  https://mirrors.sjtug.sjtu.edu.cn/docs/rocky
> sed -e 's|^mirrorlist=|#mirrorlist=|g' \
>     -e 's|^#baseurl=http://dl.rockylinux.org/$contentdir|baseurl=https://mirrors.sjtug.sjtu.edu.cn/rocky|g' \
>     -i.bak \
>     /etc/yum.repos.d/Rocky-*.repo
> --加载缓存    
> dnf makecache
> yum makecache    (选用其中一个)



阿里镜像源：https://developer.aliyun.com/mirror/rockylinux

```shell
sed -e 's|^mirrorlist=|#mirrorlist=|g' \
    -e 's|^#baseurl=http://dl.rockylinux.org/$contentdir|baseurl=https://mirrors.aliyun.com/rockylinux|g' \
    -i.bak \
    /etc/yum.repos.d/Rocky-*.repo

dnf makecache
```

看下上面优化的结果：这里我们可以做成角色名

![image-20220304164920833](openstack笔记.assets/image-20220304164920833.png)

![image-20220304164935311](openstack笔记.assets/image-20220304164935311.png)

修改个性化内容

```sh
[root@cotroller ~]# vim .vimrc
"设置行号"
"set nu"
"自动语法高亮"
"syntax on
"自动缩进"
"set autoindent"
"关闭兼容模式"
"set nocompatible
"激活鼠标"
"set mouse=c"
"开启语法"
"syntax enable
"tab缩进4个空格"
"set tabstop=4"
"设定<< >>移动宽度4"
"set shiftwidth=4
"自动缩进"
"set ai
"智能缩进"
"set si
"显示标尺"
"set ruler
"显示匹配的[]{}"
"set showmatch
"编码设置"
set encoding=utf-8
set fileencodings=utf-t
set termencoding=utf-8
"开启新行时使用智能自动缩进"
"set smartindent
"set cin
"set showmatch
"背景色"
"set background=dark"
"设置光标下划线"
set cursorline

map <F10> : set paste <cr>
map <F11> : set nopaste <cr>

autocmd BufNewFile *.sh exec ":call AddUsr()"
map <F7> ms:call AddTitle()<cr>'s

function AddAuthor()
        let n=1
        while n < 5
                let line = getline(n)
                if line =~'^\s*\*\s*\S*Last\s*modified\s*:\s*\S*.*$'
                        call UpdateTitle()
                        return
                endif
                let n = n + 1
        endwhile
        call AddTitle()
endfunction

function UpdateTitle()
        normal m'
        execute '/* Last modified\s*:/s@:.*$@\=strftime(": %Y-%m-%d %H:%M")@'
        normal "
        normal mk
        execute '/* Filename\s*:/s@:.*$@\=": ".expand("%:t")@'
        execute "noh"
        normal 'k
        echohl WarningMsg | echo "Successful in updating the copy right." | echohl None
endfunction

function AddTitle()
        call append(0,"#!/bin/bash")
        call append(1,"#----------------------------------------------")
        call append(2,"# Author        : 349925756")
        call append(3,"# Email         : 349925756@qq.com")
        call append(4,"# Last modified : ".strftime("%Y-%m-%d %H:%M"))
        call append(5,"# Filename      : ".expand("%:t"))
        call append(6,"# Description   : ")
        call append(7,"# Version       : 1.1 ")
        call append(8,"#----------------------------------------------")
        call append(9," ")
    	call append(10,"#Notes:  ")
        echohl WarningMsg | echo "Successful in adding the copyright." | echohl None

endfunction

function AddUsr()
        call append(0,"#!/bin/bash")
endfunction
```

其他操作

```sh
-- 修改主机名
[root@localhost ~]# hostnamectl set-hostname controller
[root@localhost ~]# hostnamectl set-hostname compute

[root@controller ~]# echo -e "192.168.10.200 controller\n192.168.10.201 compute\n" >> /etc/hosts

```

**互信脚本**

```sh
[root@controller ~]# mkdir -p /server/{soft,scripts}
[root@controller ~]# ssh-keygen -t rsa -b 4096
[root@controller ~]# vim /server/scripts/push_ssh.sh
#!/bin/bash
hosts=(controller compute)
rpm -qa | grep sshpass 2>&1

[ $? -ne 0 ] && yum install -y sshpass

for h in ${hosts[@]};
do
  echo -e "---------------\033[41;37m  HostName:$h pub-key start\033[0m-----------------";
  sshpass -p$1 ssh-copy-id -i ~/.ssh/id_rsa.pub ${h} "-o StrictHostKeyChecking=no" 2>/dev/null;  
  echo -e "---------------\033[44;37m  HostName:$h pub-key successfull\033[0m-----------------"; 
done                             


[root@cotroller ~]# . /server/scripts/push.sh **** #你的密码

-- 检查
[root@controller ~]# ssh compute hostname -s
compute
[root@controller ~]# ssh compute hostname -i
192.168.10.201

-- 传输脚本到计算节点
[root@controller ~]# scp -r /server/ compute:/
push_ssh.sh                 100%  425   142.0KB/s   00:00    
-- 节点也执行相同的操作

-- 验证
[root@compute ~]# for i in controller compute;do ssh $i hostname -s;done
cotroller
compute

```



## 安全

OpenStack 服务支持各种安全方法，包括密码、策略和加密。包括数据库服务器和消息代理在内的支持服务支持密码安全。

如果需要使用到随机密码可以使用下面的方法生成。

```sh
[root@cotroller ~]# openssl rand -hex 10   # 随机生成10个字符
f8ac7385f21616c25a35
```

用于**SERVICE_PASS**引用服务帐户密码和**SERVICE_DBPASS**引用数据库密码

*下表提供了需要密码的服务列表及其在指南中的相关参考*

| 密码名称                 | 描述                        |
| :----------------------- | :-------------------------- |
| 数据库密码（未使用变量） | 数据库的root密码            |
| ADMIN_PASS               | 用户密码**admin**           |
| CINDER_DBPASS            | 块存储服务的数据库密码      |
| CINDER_PASS              | 块存储服务用户密码cinder    |
| DASH_DBPASS              | 仪表板的数据库密码          |
| DEMO_PASS                | 用户密码demo                |
| GLANCE_DBPASS            | 镜像服务的数据库密码        |
| GLANCE_PASS              | 镜像服务用户密码glance      |
| KEYSTONE_DBPASS          | 身份服务的数据库密码        |
| METADATA_SECRET          | 元数据代理的秘密            |
| NEUTRON_DBPASS           | 网络服务的数据库密码        |
| NEUTRON_PASS             | 网络服务用户密码neutron     |
| NOVA_DBPASS              | 计算服务的数据库密码        |
| NOVA_PASS                | 计算服务用户密码nova        |
| PLACEMENT_PASS           | 安置服务用户的密码placement |
| RABBIT_PASS              | RabbitMQ 用户密码openstack  |

OpenStack 和支持服务在安装和操作期间需要管理权限。在某些情况下，服务对主机执行的修改可能会干扰 Ansible、Chef 和 Puppet 等部署自动化工具。



## 主机网络

安装操作系统后，您必须配置网络接口。建议禁用任何自动网络管理工具，并为您的分发手动编辑适当的配置文件。

网络分离：使用私有地址通过NAT或其他方法提供internet访问。

![image-20220304202048856](openstack笔记.assets/image-20220304202048856.png)

上面官网实例有两组网络，一组是内网，一组是外网。

ubuntu 上面网卡不是eth0 的修改方法

```sh
/etc/default/grub
GRUB_CMDLINE_LINUX="net.ifnames=0 biosdevname=0"

 sudo update-grub #更行
```

第一个接口使用内网

```
ipaddr=x.x.x.x
prefix=24
gateway=x.x.x.x
```

第二个接口使用自动获取（也可以手动设置static）

**ubuntu**

```sh
/etc/network/interfaces
# The provider network interface
auto INTERFACE_NAME
iface INTERFACE_NAME inet manual
up ip link set dev $IFACE up
down ip link set dev $IFACE down
```

**RHEL | CentOS**

```sh
/etc/sysconfig/network-scripts/ifcfg-INTERFACE_NAME
DEVICE=INTERFACE_NAME
TYPE=Ethernet
ONBOOT="yes"
BOOTPROTO="none"
```

**SUSE**

```
/etc/sysconfig/network/ifcfg-INTERFACE_NAME
STARTMODE='auto'
BOOTPROTO='static'
```

添加hosts名称解析

```sh
hostnamectl set-hostname cotroller #修改主机名
/etc/hosts
# controller
10.0.0.11       controller
# compute1
10.0.0.31       compute1
# block1
10.0.0.41       block1
# object1
10.0.0.51       object1
# object2
10.0.0.52       object2
```

**计算节点**与控制节点一样。

存储节点同上

最后相互ping 验证



验证

```sh
第一种
[root@cotroller ~]# ifconfig |grep -w inet|awk -F" " 'NR<3 {print $2}'
172.16.0.60
192.168.10.129

[root@compute ~]# ifconfig |grep -w inet|awk -F" " 'NR<3 {print $2}'
172.16.0.61
192.168.10.128

第二种
[root@cotroller ~]# ip a s |grep -w inet |awk -F" " 'NR>1 {print $2}'
172.16.0.60/24
192.168.10.129/24
```



## NTP

在节点之间正确同步服务，安装 Chrony

### controller

1、安装服务

```sh
#centos
[root@controller ~]# yum install chrony -y

#ubuntu
apt install chrony

#suse
zypper install chrony
```

2、编辑chrony.conf文件

```sh
[root@controller ~]# cp /etc/chrony.conf{,.bak}  #备份

-- 对于 RHEL、CentOS 或 SUSE，编辑/etc/chrony.conf文件：
-- 这里使用阿里云的NTP服务器
server ntp1.aliyun.com iburst
server ntp2.aliyun.com iburst
server ntp3.aliyun.com iburst
server ntp4.aliyun.com iburst
server ntp5.aliyun.com iburst
server ntp6.aliyun.com iburst

-- 对于 Ubuntu，编辑/etc/chrony/chrony.conf文件：
server NTP_SERVER iburst
```

允许其他节点访问到控制节点

```sh
allow 192.168.10.0/24     #允许使用的内网
```

3、重启NTP服务并设置开机启动

```sh
#ubuntu
service chrony restart

#Centos suse
[root@cotroller ~]# systemctl start chronyd ; systemctl enable chronyd

[root@cotroller ~]# systemctl status chronyd|grep Active|awk -F" " '{print $2,$3}'
active (running)
```

### 其他节点

操作

```sh
[root@compute ~]# yum install -y chrony   #安装服务
[root@compute ~]# cp /etc/chrony.conf{,.bak}   #备份配置文件
-- 编辑chrony.conf
#pool 2.pool.ntp.org iburst   #注释这行 
server cotroller iburst  #改成控制端的 这里注意主机名

-- 启动服务
[root@compute ~]# systemctl start chronyd;systemctl enable chronyd
[root@compute ~]# systemctl status chronyd|grep Active
   Active: active (running) since Fri 2022-03-04 21:54:41 CST; 29s ago

-- 验证
-- 在xshell 撰写输入date 看时间相同？
#不同 检查防火墙和selinux  可以不关闭selinux
[root@cotroller ~]# systemctl stop firewalld ;systemctl disable firewalld
Removed /etc/systemd/system/multi-user.target.wants/firewalld.service.
Removed /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service.
#关闭selinux 这里注意不要关闭了链接文件/etc/sysconfig/selinux 
[root@compute ~]# sed -i 's/SELINUX=.*/SELINUX=disabled/g' /etc/selinux/config

```

上面操作没有用，时间还是不同步

```sh
#查看时间同步
[root@cotroller ~]# timedatectl status
[root@cotroller ~]# timedatectl set-ntp true

#查看ntp_server状态
[root@cotroller ~]# chronyc sources -v

  .-- Source mode  '^' = server, '=' = peer, '#' = local clock.
 / .- Source state '*' = current best, '+' = combined, '-' = not combined,
| /             'x' = may be in error, '~' = too variable, '?' = unusable.
||                                                 .- xxxx [ yyyy ] +/- zzzz
||      Reachability register (octal) -.           |  xxxx = adjusted offset,
||      Log2(Polling interval) --.      |          |  yyyy = measured offset,
||                                \     |          |  zzzz = estimated error.
||                                 |    |           \
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
^- 120.25.115.20                 2   6    75    86  +5609us[+8634us] +/-   21ms
^* 203.107.6.88                  2   6   177    21    +95ms[  +98ms] +/-  141ms

M 这表示信号源的模式。^表示服务器，=表示对等方，＃表示本地连接的参考时钟。
S 此列指示源的状态
* 表示chronyd当前同步到的源。
+ 表示可接受的信号源，与选定的信号源组合在一起。
- 表示被合并算法排除的可接受源。
？ 指示已失去连接性或其数据包未通过所有测试的源。它也显示在启动时，直到从中至少收集了3个样本为止。
x 表示chronyd认为是虚假行情的时钟（即，其时间与大多数其他来源不一致）
〜 表示时间似乎具有太多可变性的来源。

Name/IP address  这显示了源的名称或IP地址，或参考时钟的参考ID。
Stratum 这显示了来源的层，如其最近收到的样本中所报告的那样。层1表示一台具有本地连接的参考时钟的计算机。与第1层计算机同步的计算机位于第2层。与第2层计算机同步的计算机位于第3层，依此类推。

Poll
这显示轮询源的速率，以秒为单位的时间间隔的以2为底的对数。因此，值为6表示每64秒进行一次测量。chronyd会根据当前情况自动更改轮询速率。

Reach
这显示了源的可达性寄存器以八进制数字打印。寄存器有8位，并在每个从源接收或丢失的数据包上更新。值377表示从最后八次传输中收到了对所有用户的有效答复。

LastRx
此列显示多长时间前从来源接收到了最后一个好的样本（在下一列中显示）。未通过某些测试的测量将被忽略。通常以秒为单位。字母m，h，d或y表示分钟，小时，天或年。

Last sample
此列显示上次测量时本地时钟与源之间的偏移。方括号中的数字表示实际测得的偏移量。可以用ns（表示纳秒），us （表示微秒），ms（表示毫秒）或s（表示秒）作为后缀。方括号左侧的数字表示原始测量值，已调整为允许此后施加于本地时钟的任何摆度。

+/-指示器后面的数字表示测量中的误差范围。正偏移表示本地时钟位于源时钟之前。

#状态
[root@cotroller ~]# chronyc sourcestats -v
                             .- Number of sample points in measurement set.
                            /    .- Number of residual runs with same sign.
                           |    /    .- Length of measurement set (time).
                           |   |    /      .- Est. clock freq error (ppm).
                           |   |   |      /           .- Est. error in freq.
                           |   |   |     |           /         .- Est. offset.
                           |   |   |     |          |          |   On the -.
                           |   |   |     |          |          |   samples. \
                           |   |   |     |          |          |             |
Name/IP Address            NP  NR  Span  Frequency  Freq Skew  Offset  Std Dev
==============================================================================
120.25.115.20               6   4   265    +12.578    188.009   +660us  4615us
203.107.6.88                8   5   266     -6.540    188.210  -6720us  8940us

#查看 ntp_servers 是否在线
[root@cotroller ~]# chronyc activity -v 

#查看 ntp 详细信息
[root@cotroller ~]# chronyc tracking -v
Reference ID    : 78197314 (120.25.115.20)
Stratum         : 3
Ref time (UTC)  : Fri Mar 04 14:43:13 2022
System time     : 0.000655383 seconds fast of NTP time
Last offset     : +0.000662724 seconds
RMS offset      : 0.003184171 seconds
Frequency       : 21.742 ppm fast
Residual freq   : +9.732 ppm
Skew            : 17.588 ppm
Root delay      : 0.053631224 seconds
Root dispersion : 0.035005637 seconds
Update interval : 129.4 seconds
Leap status     : Normal

#强制同步下系统时钟
[root@cotroller ~]# chronyc -a makestep

#查看日期时间、时区及 NTP 状态
[root@cotroller ~]# timedatectl
               Local time: Fri 2022-03-04 22:45:16 CST
           Universal time: Fri 2022-03-04 14:45:16 UTC
                 RTC time: Fri 2022-03-04 14:45:16
                Time zone: Asia/Shanghai (CST, +0800)
System clock synchronized: no
              NTP service: active
          RTC in local TZ: no

#查看时区列表
timedatectl list-timezones
timedatectl list-timezones |  grep  -E "Asia/S.*"

#修改时区
timedatectl set-timezone Asia/Shanghai

#修改日期时间（可以只修改其中一个）
timedatectl set-time "2021-04-15 15:50:20"

#开启 NTP
timedatectl set-ntp true/flase

vim /etc/sysconfig/ntpd
将系统时间写入BIOS，与 hwclock -w 效果相同
SYNC_HWCLOCK=yes
```

> 问题找到了，因为是双网卡一个手动一个DHCP，allow 允许成自动获取的导致hosts无法找到主机导致同步失败

![image-20220304234316228](openstack笔记.assets/image-20220304234316228.png)

最后同步成功了。内网IP设置为手动，外网设置为自动

如果不同步在下面设置这个

```sh
[root@cotroller ~]# chronyc sources
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
^+ 120.25.115.20                 2   6   277    42  +4909us[+4909us] +/-   26ms
^* 203.107.6.88                  2   6   377    42   -925us[  +32us] +/-   43ms

[root@compute ~]# chronyc sources
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
^* cotroller                     3   6   377    45  +1508us[+5123us] +/-   50ms
```



## openstack软件包

包一定要选一致的避免很多问题。 OpenStack 包的设置需要在所有节点上完成：控制器、计算和块存储节点。

> 禁用或删除任何自动更新服务，因为它们会影响您的 OpenStack 环境。

其他类型的系统参考：https://docs.openstack.org/install-guide/environment-packages.html

> 【警告】：openstack 从ussuri版本开始就需要centos8以上，这里使用的是rocky linux  

### 先决条件

其他版本：https://docs.openstack.org/install-guide/environment-packages-rdo.html

根据系统选择安装先决条件包

```sh
[root@controller ~]# mv /etc/yum.repos.d/epel.repo{,.bak}
```

> 我们建议在使用 RDO 包时禁用 EPEL，因为 EPEL 中的更新破坏了向后兼容性。`yum-versionlock`或者，最好使用插件固定封装版本。

### 启用openstack存储库

在 CentOS 上，extras存储库提供启用 OpenStack 存储库的 RPM。CentOSextras默认包含存储库，因此您只需安装该软件包即可启用 OpenStack 存储库。对于 CentOS8，您还需要启用 PowerTools 存储库。

```sh
[root@compute ~]# yum search centos-release-openstack
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
================================================ N/S matched: centos-release-openstack =================================================
centos-release-openstack-queens.noarch : OpenStack from the CentOS Cloud SIG repo configs
centos-release-openstack-rocky.noarch : OpenStack from the CentOS Cloud SIG repo configs
centos-release-openstack-stein.noarch : OpenStack from the CentOS Cloud SIG repo configs
centos-release-openstack-train.noarch : OpenStack from the CentOS Cloud SIG repo configs
```

可以看到centos下最新为**train**,后面都使用这个版本了

其他版本：https://docs.openstack.org/install-guide/environment-packages-rdo.html

```sh
# yum install centos-release-openstack-train -y
```



### 完成安装

其他版本的安装

```sh
[root@compute ~]# yum upgrade -y
[root@compute ~]# reboot
-- 升级系统包，包含kernel

--centos7
[root@controller ~]# yum install python-openstackclient -y

--centos8
[root@controller ~]# yum install python3-openstackclient

--RHEL 和 CentOS 默认启用SELinux。安装 openstack-selinux软件包以自动管理 OpenStack 服务的安全策略：
[root@controller ~]# yum install openstack-selinux -y
```





## 数据库

控制器节点

参考：https://docs.openstack.org/install-guide/environment-sql-database-rdo.html

openstack 服务需要数据库服务，数据库运行在控制节点controller上。常用数据库mariadb mysql postgresql.

```sh
-- 安装软件包
[root@controller ~]# yum install mariadb mariadb-server python2-PyMySQL -y

-- 创建openstack.cnf文件
[root@controller ~]# vim /etc/my.cnf.d/openstack.cnf
[mysqld]      
bind-address = 192.168.10.200

default-storage-engine = innodb
innodb_file_per_table = on
max_connections = 4096
collation-server = utf8_general_ci
character-set-server = utf8

[root@controller ~]# systemctl start mariadb.service 
[root@controller ~]# systemctl enable mariadb.service 

-- 使用脚本保护数据服务
[root@controller ~]# mysql_secure_installation

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none): 
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

You already have a root password set, so you can safely answer 'n'.

Change the root password? [Y/n] Y
New password: 
Re-enter new password: 
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] Y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] Y
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] Y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] Y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!


-- 测试数据库
[root@controller ~]# mysql -uroot -p -P3306
Enter password:
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
3 rows in set (0.001 sec)

MariaDB [(none)]> select host,user from mysql.user;
+-----------+------+
| host      | user |
+-----------+------+
| 127.0.0.1 | root |
| ::1       | root |
| localhost | root |
+-----------+------+
3 rows in set (0.000 sec)
```



## 消息列队

OpenStack 使用消息队列来协调服务之间的操作和状态信息。消息队列服务通常在**控制器节点**上运行。OpenStack 支持多种消息队列服务，包括RabbitMQ、 Qpid和ZeroMQ。但是，大多数打包 OpenStack 的发行版都支持特定的消息队列服务。

安装配置

```sh
-- 安装包
[root@controller ~]# yum install rabbitmq-server -y

-- 启动消息列队
[root@controller ~]# systemctl start rabbitmq-server.service ;systemctl enable rabbitmq-server.service 

-- 添加openstack用户
[root@controller ~]# rabbitmqctl add_user openstack openstack  #前面是用户，后面是密码
Creating user "openstack"

-- 允许用户进行配置、读写访问openstack
[root@controller ~]# rabbitmqctl set_permissions openstack ".*" ".*" ".*"
Setting permissions for user "openstack" in vhost "/"

```



## 内存缓存

服务的 Identity 服务认证机制使用 Memcached 来缓存令牌。memcached 服务通常在控制器节点上运行。对于生产部署，我们建议启用防火墙、身份验证和加密的组合来保护它。

配置

```sh
-- 安装软件
[root@controller ~]# yum install memcached python-memcached -y

-- 编辑配置文件
/etc/sysconfig/memcached
-- 将服务配置为使用控制器节点的管理 IP 地址。这是为了允许其他节点通过管理网络访问：

[root@controller ~]# vim /etc/sysconfig/memcached 

PORT="11211"
USER="memcached"
MAXCONN="1024"
CACHESIZE="64"
OPTIONS="-l 127.0.0.1,::1,controller"   

-- 启动服务
[root@controller ~]# systemctl start memcached.service ;systemctl enable memcached.service 
```



## Etcd

OpenStack 服务可以使用 Etcd，这是一种分布式可靠键值存储，用于分布式键锁定、存储配置、跟踪服务活动性和其他场景。kubernetes就使用etcd集群

```sh
[root@controller ~]# yum install etcd -y
```

编辑/etc/etcd/etcd.conf文件并将ETCD_INITIAL_CLUSTER, ETCD_INITIAL_ADVERTISE_PEER_URLS, ETCD_ADVERTISE_CLIENT_URLS, 设置ETCD_LISTEN_CLIENT_URLS为控制器节点的管理 IP 地址，以允许其他节点通过管理网络访问：

```sh
cat <<EOF |tee /etc/etcd/etcd.conf
#[Member]
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_PEER_URLS="http://192.168.10.200:2380"
ETCD_LISTEN_CLIENT_URLS="http://192.168.10.200:2379"
ETCD_NAME="controller"
#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.10.200:2380"
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.10.200:2379"
ETCD_INITIAL_CLUSTER="controller=http://192.168.10.200:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster-01"
ETCD_INITIAL_CLUSTER_STATE="new"
EOF
```

上面是需要修改的配置，集群参考kubernetes

```sh
[root@controller ~]# systemctl start etcd;systemctl enable etcd
```

> 以上就是安装的前提准备



# 安装openstack 服务

参考：https://docs.openstack.org/install-guide/openstack-services.html

这里特别要注意要跟上面环境里面的版本匹配，我上面是Train ，这里也必须是使用train版本

https://docs.openstack.org/train/install/

## Train 的最小部署

需要安装以下服务，按以下指定的顺序安装服务：

- identity   身份服务 keystone安装
- image     镜像服务 glance安装
- placement  安置服务  Placement安装
- compute  计算服务   nova安装
- networking  网络服务 neutron安装

附加组件

- dashboard  horizon 安装
- block storage  cinder 安装

## keystone 安装

OpenStack 系统由几个单独安装的关键服务组成。这些服务根据您的云需求协同工作，包括计算、身份、网络、镜像、块存储、对象存储、Telemetry、编排和数据库服务。您可以单独安装这些项目中的任何一个，并将它们单独配置或配置为连接的实体。

https://docs.openstack.org/keystone/train/install/index-rdo.html

### 概念

OpenStack Identity 服务提供了用于管理身份验证、授权和服务目录的单点集成

身份服务通常是用户与之交互的第一个服务。一旦通过身份验证，最终用户就可以使用他们的身份访问其他 OpenStack 服务。同样，其他 OpenStack 服务利用 Identity 服务来确保用户是他们所说的身份，并发现其他服务在部署中的位置。身份服务还可以与一些外部用户管理系统（例如 LDAP）集成。

用户和服务可以使用由 Identity 服务管理的服务目录来定位其他服务。每个服务可以有一个或多个端点，每个端点可以是以下三种类型之一：管理员、内部或公共。在生产环境中，出于安全原因，不同的端点类型可能驻留在暴露给不同类型用户的不同网络上。例如，公共 API 网络可能从 Internet 可见，因此客户可以管理他们的云。管理 API 网络可能仅限于管理云基础架构的组织内的操作员。内部 API 网络可能仅限于包含 OpenStack 服务的主机。OpenStack 支持多个区域以实现可扩展性。==RegionOne==地区。在身份服务中创建的区域、服务和端点共同构成了部署的服务目录。部署中的每个 OpenStack 服务都需要一个服务条目，其对应的端点存储在 Identity 服务中。这一切都可以在安装和配置身份服务后完成。

组件：

**====服务器：==**集中式服务器使用 RESTful 接口提供身份验证和授权服务

==**驱动程序：**==驱动程序或服务后端集成到集中式服务器。它们用于访问 OpenStack 外部存储库中的身份信息，并且可能已经存在于部署 OpenStack 的基础架构中（例如，SQL 数据库或 LDAP 服务器）。

**==模块：==**中间件模块在使用身份服务的 OpenStack 组件的地址空间中运行。这些模块拦截服务请求，提取用户凭据，并将其发送到集中式服务器进行授权。中间件模块和 OpenStack 组件之间的集成使用 Python Web 服务器网关接口。

### 安装和配置

在控制器节点上安装和配置 OpenStack Identity 服务，代号为 keystone。出于可扩展性目的，此配置部署了 Fernet 令牌和 Apache HTTP 服务器来处理请求。

#### 先决条件

在安装和配置身份服务之前，您必须创建一个数据库。

```sh
[root@controller ~]# mysql -uroot -p
Enter password: ***

MariaDB [(none)]> create database keystone charset utf8mb4;
Query OK, 1 row affected (0.004 sec)

MariaDB [(none)]> grant all  on keystone.* to 'keystone'@'localhost' identified by 'KEYSTONE_DBPASS';
Query OK, 0 rows affected (0.007 sec)

MariaDB [(none)]> GRANT ALL  ON keystone.* TO 'keystone'@'%' \ identified by 'KEYSTONE_DBPASS';
Query OK, 0 rows affected (0.001 sec)

```

```mysql
-- 验证
MariaDB [(none)]> select user,host from mysql.user;
+----------+-----------+
| user     | host      |
+----------+-----------+
| keystone | %         |
| root     | 127.0.0.1 |
| root     | ::1       |
| keystone | localhost |
| root     | localhost |
+----------+-----------+
5 rows in set (0.002 sec)
```

上面的密码可以更换成你自己的密码。不能忘记，忘记了也可以修改。

#### 安装和配置

```sh
--- 安装软件包
[root@controller ~]# yum install openstack-keystone httpd mod_wsgi -y

--- 编辑/etc/keystone/keystone.conf
[root@controller ~]# cp /etc/keystone/keystone.conf{,.bak} 
...
582 [database] #在该[database]部分中，配置数据库访问：
...
600 #connection = <None>
601 
602 connection = mysql+pymysql://keystone:KEYSTONE_DBPASS@controller/keystone
...
2448 [token]   #在该[token]部分中，配置 Fernet 令牌提供程序：
...
2477 #provider = fernet
2478 provider = fernet
...
```

> 注释掉或删除connection该部分中的任何其他选项 [database]
>
> 在此示例中，主机controller必须是可解析的。

```sh
--- 填充身份服务数据库：
[root@controller ~]# su -s /bin/sh -c "keystone-manage db_sync" keystone

-- 验证
[root@controller ~]# mysql -h controller -ukeystone -pKEYSTONE_DBPASS -e "use keystone;show tables;"
+------------------------------------+
| Tables_in_keystone                 |
+------------------------------------+
| access_rule                        |
| access_token                       |
| application_credential             |
| application_credential_access_rule |
| application_credential_role        |
| assignment                         |
| config_register                    |
| consumer                           |
| credential                         |
| endpoint                           |
| endpoint_group                     |
| federated_user                     |
| federation_protocol                |
| group                              |
| id_mapping                         |
| identity_provider                  |
| idp_remote_ids                     |
| implied_role                       |
| limit                              |
| local_user                         |
| mapping                            |
| migrate_version                    |
| nonlocal_user                      |
| password                           |
| policy                             |
| policy_association                 |
| project                            |
| project_endpoint                   |
| project_endpoint_group             |
| project_option                     |
| project_tag                        |
| region                             |
| registered_limit                   |
| request_token                      |
| revocation_event                   |
| role                               |
| role_option                        |
| sensitive_config                   |
| service                            |
| service_provider                   |
| system_assignment                  |
| token                              |
| trust                              |
| trust_role                         |
| user                               |
| user_group_membership              |
| user_option                        |
| whitelisted_config                 |
+------------------------------------+

```

初始化 Fernet 密钥存储库：

> --keystone-user和--keystone-group 用于运行 keystone 的操作系统的用户/组。在下面的示例中，我们调用 user & group keystone。

```sh
[root@controller ~]# keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
[root@controller ~]# keystone-manage credential_setup --keystone-user keystone --keystone-group keystone

-- 验证
[root@controller ~]# tree /etc/keystone/fernet-keys/
/etc/keystone/fernet-keys/
├── 0
└── 1

0 directories, 2 files

```

引导身份服务

> 在 Queens 发布之前，keystone 需要在两个单独的端口上运行以适应 Identity v2 API，后者通常在端口 35357 上运行单独的仅管理员服务。删除 v2 API 后，keystone 可以在同一端口上运行所有接口。

```sh
[root@controller ~]# keystone-manage bootstrap --bootstrap-password ADMIN_PASS \
  --bootstrap-admin-url http://controller:5000/v3/ \
  --bootstrap-internal-url http://controller:5000/v3/ \
  --bootstrap-public-url http://controller:5000/v3/ \
  --bootstrap-region-id RegionOne
```

ADMIN_PASS：可以更换成自定义的密码



#### 配置apache http服务

```sh
-- 编辑/etc/httpd/conf/httpd.conf文件并配置 ServerName选项以引用控制器节点
[root@controller ~]# vim /etc/httpd/conf/httpd.conf
 95 #ServerName www.example.com:80
 96 ServerName controller      #复制上面一条修改
 
-- 创建/usr/share/keystone/wsgi-keystone.conf文件的链接
[root@controller ~]# ln -s /usr/share/keystone/wsgi-keystone.conf /etc/httpd/conf.d/

[root@controller ~]# ll /etc/httpd/conf.d/
total 16
-rw-r--r--. 1 root root 2926 Jan 25 22:08 autoindex.conf
-rw-r--r--. 1 root root  366 Jan 25 22:09 README
-rw-r--r--. 1 root root 1252 Jan  8 00:08 userdir.conf
-rw-r--r--. 1 root root  824 Jan 14 01:38 welcome.conf
lrwxrwxrwx. 1 root root   38 Mar  6 16:08 wsgi-keystone.conf -> /usr/share/keystone/wsgi-keystone.conf
```

要安全性需要考虑使用ssl 支持https

完成

```sh
-- 启动
[root@controller ~]# systemctl enable httpd ;systemctl start httpd

-- 失败是selinux开启 
sed -i "s/SELINUX=.*/SELINUX=disabled/g" /etc/selinux/config 

-- 通过设置适当的环境变量来配置管理帐户：
$ export OS_USERNAME=admin
$ export OS_PASSWORD=ADMIN_PASS
$ export OS_PROJECT_NAME=admin
$ export OS_USER_DOMAIN_NAME=Default
$ export OS_PROJECT_DOMAIN_NAME=Default
$ export OS_AUTH_URL=http://controller:5000/v3
$ export OS_IDENTITY_API_VERSION=3
```

此处显示的这些值是从keystone-manage bootstrap引导程序创建的默认值。

将 ADMIN_PASS 替换为 keystone-install-configure-rdo 中 keystone-manage 引导命令中使用的密码。

```sh
[root@controller ~]# cat .bashrc 
# .bashrc

# User specific aliases and functions

alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'

# Source global definitions
if [ -f /etc/bashrc ]; then
	. /etc/bashrc
fi

export OS_USERNAME=admin
export OS_PASSWORD=ADMIN_PASS
export OS_PROJECT_NAME=admin
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_DOMAIN_NAME=Default
export OS_AUTH_URL=http://controller:5000/v3
export OS_IDENTITY_API_VERSION=3
```



### 创建域、项目、用户和角色

Identity 服务为每个 OpenStack 服务提供身份验证服务。**==身份验证服务使用域、项目、用户和角色的组合==**。

在前面的步骤keystone-manage引导已经创建了默认的域，下面还是要介绍创建新域。

```sh
-- 创建新域的正确方式
[root@controller ~]# openstack domain create --description "cloudb.pub" example
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | cloudb.pub                       |
| enabled     | True                             |
| id          | 27268c764a734361aff4c8572771fe15 |
| name        | example                          |
| options     | {}                               |
| tags        | []                               |
+-------------+----------------------------------+

-- 创建服务项目（添加到环境中的每项服务的唯一用户）
[root@controller ~]# openstack project create --domain default --description "Service Project" service
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Service Project                  |
| domain_id   | default                          |
| enabled     | True                             |
| id          | 9f2028aec79d4f7c8aafea6c74586019 |
| is_domain   | False                            |
| name        | service                          |
| options     | {}                               |
| parent_id   | default                          |
| tags        | []                               |
+-------------+----------------------------------+

-- 常规（非管理员）任务应使用非特权项目和用户。例如，本指南创建myproject项目和myuser 用户
--- 创建myproject项目
[root@controller ~]# openstack project create --domain default --description "Demo Project" myproject
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Demo Project                     |
| domain_id   | default                          |
| enabled     | True                             |
| id          | ab171cdc73f84df4ab572bf464cbe992 |
| is_domain   | False                            |
| name        | myproject                        |
| options     | {}                               |
| parent_id   | default                          |
| tags        | []                               |
+-------------+----------------------------------+
为该项目创建其他用户时，请勿重复此步骤。此步就是创建一个文件夹，里面的数据会被清空或者覆盖会导致数据丢失

--- 创建myuser用户
[root@controller ~]# openstack user create --domain default --password-prompt myuser
User Password:   myuser
Repeat User Password:   myuser
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 557b6c68221845cc924fbbb1f2e61688 |
| name                | myuser                           |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+

--- 创建myrole 角色
[root@controller ~]# openstack role create myrole
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | None                             |
| domain_id   | None                             |
| id          | 402ed58d008a42d2b2f4b99b5fb209e0 |
| name        | myrole                           |
| options     | {}                               |
+-------------+----------------------------------+

--- 将myrole角色添加到myproject 项目和myuser用户
[root@controller ~]# openstack role add --project myproject --user myuser myrole
```

重复上面的操作可以创建新的项目，

domain   、project 、 user  、role 是一套验证组合。

创建一个域，然后里面可以添加不同的项目（域组织），user（域用户），role（域权限）

### 验证操作

==在安装其他服务之前验证身份服务的操作==。controller执行

取消设置临时变量OS_AUTH_URL和OS_PASSWORD环境变量：

```sh
[root@controller ~]# unset OS_AUTH_URL OS_PASSWORD
```

作为admin用户，请求一个身份验证令牌：

```sh
[root@controller ~]# openstack --os-auth-url http://controller:5000/v3 --os-project-domain-name Default --os-user-domain-name Default --os-project-name admin --os-username admin token issue
Password: 
Password: 
+------------+-------------------------------------------------------------------------
| Field      | Value                                                                   
+------------+-------------------------------------------------------------------------
| expires    | 2022-03-06T15:07:57+0000       
| id         | gAAAAABiJMA99RJLRGII9d4vk0Hd3PvAo6fXcxJavM0VeuTV3XtB2zvBhQCgFUZyDhWpFT_5-CaTnJzaKGtyt8BnwDALPAJJLY-n2vaFxFWZ4lxnyG5kvIwr
| project_id | c22792043bcb41248ee21cc03f7d0ce2   
| user_id    | def6d7d6f7144af8914e9ec83fef0aa0   
+------------+-------------------------------------------------------------------------
```

![image-20220306220818716](openstack笔记.assets/image-20220306220818716.png)

> 此命令使用admin用户的密码。也就是环境变量中的那个密码。ADMIN_PASS

作为myuser在上一节中创建的用户，请求一个身份验证令牌

```sh
[root@controller ~]# openstack --os-auth-url http://controller:5000/v3 \
   --os-project-domain-name Default --os-user-domain-name Default \
   --os-project-name myproject --os-username myuser token issue
Password: 
Password: 
+------------+-------------------------------------------------------------------------
| Field      | Value   
+------------+-------------------------------------------------------------------------
| expires    | 2022-03-06T15:13:34+0000                                                
| id         | gAAAAABiJMGOjl_AjEF-LnUyJ7KmnLlSV1E6M-Al_PhZ9IrCzFLe1fZQxoB7mkr-v1CN-zHwQC-h-wc-SCtzKFgJSdnaogkXWMT5tD-Izzje6PSNcaxarBGCC65s6emqrQS_FZ7j0iaLA7W7nM9b-P9N46syvme_ggWOZzi_adOFTf6zneabYo8 
| project_id | ab171cdc73f84df4ab572bf464cbe992
| user_id    | 557b6c68221845cc924fbbb1f2e61688
+------------+-------------------------------------------------------------------------
```

> myuser 用户的密码myuser

从上面的获取令牌可以知道keystone服务搭建是成功的。



### 创建openstack客户端环境脚本

前面的部分使用环境变量和命令选项的组合通过 `openstack`客户端与身份服务进行交互。为了提高客户端操作的效率，OpenStack 支持简单的客户端环境脚本，也称为 OpenRC 文件。这些脚本通常包含所有客户端的通用选项，但也支持独特的选项。

#### 创建脚本

为项目和用户`admin`创建客户端环境脚本。`demo`在未来部分将参考这些脚本来为客户端操作加载适当的凭据。

>  客户端环境脚本的路径不受限制。为方便起见，您可以将脚本放置在任何位置，但请确保它们可访问并位于适合您的部署的安全位置，因为它们确实包含敏感凭据。

创建并编辑`admin-openrc`文件并添加以下内容：

> OpenStack 客户端也支持使用clouds.yaml文件

```sh
[root@controller ~]# vim /server/scripts/admin-openrc.sh
#!/bin/bash
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=ADMIN_PASS
export OS_AUTH_URL=http://controller:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2          
```

==ADMIN_PASS可以替换为你的admin自定义密码==

```sh
[root@controller ~]# vim /server/scripts/demo-openrc.sh
#!/bin/bash
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=myproject
export OS_USERNAME=myuser
export OS_PASSWORD=DEMO_PASS
export OS_AUTH_URL=http://controller:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
```

DEMO_PASS 密码改成上面自定义的myuser 

#### 使用脚本

要使用定义好的脚本环境必须加载。

```sh
[root@controller ~]# . /server/scripts/admin-openrc.sh 
[root@controller ~]# openstack token issue
+------------+-------------------------------------------------------------------------
| Field      | Value                                                                   
+------------+-------------------------------------------------------------------------
| expires    | 2022-03-06T15:35:32+0000 
| id         | gAAAAABiJMa0_1hehXD19zrgFI_PM-heIIxyyD2XgAt6Ijagtr1-wI_WQpWwtc7ylRYBd0rPGiM_bC9-2CV7yt7K2lz1unXdW5WBDYG8LgnLzJ5grsRkRsa1XFp0cVuY99IOHm80pt7Kc7o_rmK5k-fDY4PTXLB6FJwW2H52uO2aWqHFbPYcrX4 |
| project_id | c22792043bcb41248ee21cc03f7d0ce2 
| user_id    | def6d7d6f7144af8914e9ec83fef0aa0 
+------------+-------------------------------------------------------------------------
```

```sh
-- 客户端
[root@compute ~]# . /server/scripts/admin-openrc.sh 
[root@compute ~]# openstack token issue
+------------+-------------------------------------------------------------------------
| Field      | Value                                                                   
+------------+-------------------------------------------------------------------------
| expires    | 2022-03-06T15:38:54+0000                                                 
| id         | gAAAAABiJMd-ldIYaR_DakD7Oojxwh6I3T1wGS5nxg6lV_yrkYydhhP02zXMoFd7t_gmNz6Z3LyVaAg34_P8HCR7xWUp1edJixTh18cf6N9_8AZyKuUFMq6wcVKFfxYqSL4ToWx_fh5xHfOrvhpR_izhRRQ3NL8QPy0IVCJ-PeI8RtmaqG-70BU |
| project_id | c22792043bcb41248ee21cc03f7d0ce2                                         
| user_id    | def6d7d6f7144af8914e9ec83fef0aa0                                         
+------------+----------------------------------------------------------
```



## Glance 安装

Image 服务 (glance) 使用户能够发现、注册和检索虚拟机映像。它提供了一个 REST API，使您能够查询虚拟机映像元数据并检索实际映像。您可以将通过映像服务提供的虚拟机映像存储在各种位置，从简单的文件系统到 OpenStack 对象存储等对象存储系统。

> 为简单起见，将image服务配置为使用file后端，该后端上传并存储在托管image服务的控制器节点上的目录中。默认情况下，此目录为==`/var/lib/glance/images/`==.

OpenStack Image 服务是基础设施即服务 (IaaS) 的核心。它接受对磁盘或服务器映像的 API 请求，以及来自最终用户或 OpenStack 计算组件的元数据定义。它还支持在各种存储库类型上存储磁盘或服务器映像，包括 OpenStack 对象存储。

**glance-api**

接受用于image发现、检索和存储的image API 调用。

**glance-registry**    内部服务

存储、处理和检索有关image的元数据。元数据包括大小和类型等项目。

**Database**

存储image元数据，您可以根据自己的喜好选择数据库。大多数部署使用 MySQL 或 SQLite。

**Storage repository for image files**

支持各种存储库类型，包括普通文件系统（或安装在glance-api 控制器节点上的任何文件系统）、对象存储、RADOS 块设备、VMware 数据存储和HTTP。请注意，某些存储库仅支持只读使用。

**Metadata definition service**

供供应商、管理员、服务和用户有意义地定义自己的自定义元数据的通用 API。此元数据可用于不同类型的资源，如image、artifacts工件、volumes卷、flavors和aggregates聚合。定义包括新属性的键、描述、约束和可以关联的资源类型。

**python3下运行Glance**



### 安装

参考：https://docs.openstack.org/glance/train/install/install.html

#### 先决条件

在安装和配置映像服务之前，您必须创建数据库、服务凭证和 API 

1、创建数据

```sh
-- root 登录数据库
[root@controller ~]# mysql -uroot -p****

-- 创建glance数据库
MariaDB [(none)]> create database glance;
Query OK, 1 row affected (0.000 sec)

MariaDB [(none)]> grant all on glance.* to 'glance'@'localhost' identified by 'GLANCE_DBPASS';
Query OK, 0 rows affected (0.000 sec)

MariaDB [(none)]> grant all on glance.* to 'glance'@'%' identified by 'GLANCE_DBPASS';
Query OK, 0 rows affected (0.000 sec)
```

GLANCE_DBPASS可以自定义，这里采用默认的。

```sh
-- 验证
[root@controller ~]# mysql -hcontroller -uglance -pGLANCE_DBPASS -e "show databases;"
+--------------------+
| Database           |
+--------------------+
| glance             |
| information_schema |
+--------------------+

```

2、获取admin环境变量凭证

```sh
[root@controller ~]# . /server/scripts/admin-openrc.sh 
```

3、要创建**==服务凭证==**，完成下面的操作

```sh
-- 创建glance
[root@controller ~]# openstack user create --domain default --password-prompt glance
User Password:  glance
Repeat User Password:   glance
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 5472d65f1d914f6489814a0faee3fecc |
| name                | glance                           |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+

-- 将admin角色添加到glance用户和服务项目
[root@controller ~]# openstack role add --project service --user glance admin

-- 创建glance服务实体
[root@controller ~]# openstack service create --name glance --description "OpenStack Image" image
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Image                  |
| enabled     | True                             |
| id          | 08d071084dcf40c8832a256d4b8119cd |
| name        | glance                           |
| type        | image                            |
+-------------+----------------------------------+
```

4、创建image 服务API接口

```sh
-- 公共
[root@controller ~]# openstack endpoint create --region RegionOne image public http://controller:9292
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 7c537c18c91d4d83a835b27a24074b26 |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 08d071084dcf40c8832a256d4b8119cd |
| service_name | glance                           |
| service_type | image                            |
| url          | http://controller:9292           |
+--------------+----------------------------------+

-- 内部
[root@controller ~]# openstack endpoint create --region RegionOne image internal http://controller:9292
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | c6361a0e7c9f4ba7bbbaf6bdb8962f45 |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 08d071084dcf40c8832a256d4b8119cd |
| service_name | glance                           |
| service_type | image                            |
| url          | http://controller:9292           |
+--------------+----------------------------------+

-- 管理
[root@controller ~]# openstack endpoint create --region RegionOne image admin http://controller:9292
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | bf297e1fc398401593ae9ad1d9a0ba81 |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 08d071084dcf40c8832a256d4b8119cd |
| service_name | glance                           |
| service_type | image                            |
| url          | http://controller:9292           |
+--------------+----------------------------------+

openstack endpoint list 可以查看

```

#### 安装和配置组件

1、安装软件包

```sh
[root@controller ~]# yum install openstack-glance -y
```

2、编辑 ==/etc/glance/glance-api.conf==文件并完成以下操作

```sh
[root@controller ~]# cp /etc/glance/glance-api.conf{,.bak}
[root@controller ~]# vim /etc/glance/glance-api.conf
2071 [database]
...
2089 #connection = <None>
2090 connection = mysql+pymysql://glance:GLANCE_DBPASS@controller/glance 
     connection = mysql+pymysql://glance:GLANCE_DBPASS@controller/glance
...
# GLANCE_DBPASS 是glance数据库密码

-- 在[keystone_authtoken]和[paste_deploy]部分中，配置身份服务访问：
4859 [keystone_authtoken]
4874 www_authenticate_uri = http://controller:5000 
4890 auth_url = http://controller:5000
4937 memcached_servers = controller:11211  
5018 auth_type = password
5020 project_domain_name = Default
5021 user_domain_name = Default
5022 project_name = service
5023 username = glance
5024 password = glance

5504 [paste_deploy]
5534 flavor = keystone   

#GLANCE_PASS 是glance 的密码：glance
-- 注释掉或删除该 [keystone_authtoken]部分中的任何其他选项

-- 在该[glance_store]部分中，配置本地文件系统存储和image文件的位置：
3349 [glance_store]
3410 stores = file,http   
3464 default_store = file   
3749 filesystem_store_datadir = /var/lib/glance/images  
```

3、同步数据到数据库

```sh
[root@controller ~]# su -s /bin/sh -c "glance-manage db_sync" glance
# 忽略这里提示的弃用消息

-- 验证
[root@controller ~]# mysql -uglance -pGLANCE_DBPASS -e "use glance;show tables;"
+----------------------------------+
| Tables_in_glance                 |
+----------------------------------+
| alembic_version                  |
| image_locations                  |
| image_members                    |
| image_properties                 |
| image_tags                       |
| images                           |
| metadef_namespace_resource_types |
| metadef_namespaces               |
| metadef_objects                  |
| metadef_properties               |
| metadef_resource_types           |
| metadef_tags                     |
| migrate_version                  |
| task_info                        |
| tasks                            |
+----------------------------------+
```

#### 完成

```sh
[root@controller ~]# systemctl enable openstack-glance-api.service ;systemctl start openstack-glance-api.service 
```

### 验证

使用CirrOS验证映像服务的操作，这 是一个小型 Linux 映像，可帮助您测试 OpenStack 部署。

> 有关如何下载和构建镜像的更多信息，请参阅 [OpenStack Virtual Machine Image Guide](https://docs.openstack.org/image-guide/)。有关如何管理映像的信息，请参阅 [OpenStack 最终用户指南](https://docs.openstack.org/user-guide/common/cli-manage-images.html)。

在控制器节点上执行这些命令

```sh
-- 加载admin变量
[root@controller ~]# . /server/scripts/admin-openrc.sh 

-- 下载image
[root@controller ~]# wget http://download.cirros-cloud.net/0.4.0/cirros-0.4.0-x86_64-disk.img
```

使用 QCOW2 磁盘格式、裸容器格式和公共可见性将image上传到image服务，以便所有项目都可以访问它：

https://docs.openstack.org/glance/train/install/verify.html

https://docs.openstack.org/install-guide/openstack-services.html#minimal-deployment-for-train

```sh
-- 遇到的错误
[root@controller ~]# glance image-create --name "cirros" \
   --file cirros-0.4.0-x86_64-disk.img \
   --disk-format qcow2 --container-format bare \
   --visibility public
Error finding address for http://controller:9292/v2/schemas/image: Unable to establish connection to http://controller:9292/v2/schemas/image: HTTPConnectionPool(host='controller', port=9292): Max retries exceeded with url: /v2/schemas/image (Caused by NewConnectionError('<urllib3.connection.HTTPConnection object at 0x7f592b2da250>: Failed to establish a new connection: [Errno 111] Connection refused',))
[root@controller ~]# netstat -lntup |grep 9292
9292 端口没有开
auth_url = http://controller:5000   官网给的是uri 所以不能正常启动

[root@controller ~]# glance image-create --name "cirros" \
    --file cirros-0.4.0-x86_64-disk.img \
    --disk-format qcow2 --container-format bare \
    --visibility public
```

![image-20220309162833241](openstack笔记.assets/image-20220309162833241.png)

关于glanc的参数参阅openstack user --help

openstack --help 可以查看帮助

```sh
[root@controller ~]# glance image-list
+--------------------------------------+--------+
| ID                                   | Name   |
+--------------------------------------+--------+
| 2026248a-564b-4ca4-93be-200bb623b8e6 | cirros |
| 3b9781a4-0630-46ff-bd1d-dbc92b3210a9 | cirros |
+--------------------------------------+--------+
```



## Placement 安装

S版之前 placement  与 计算 REST API 代码 (nova-api) 一起在 Nova 中。

**1.部署API服务**

Placement 提供了一个placement-apiWSGI 脚本，用于使用 Apache、nginx 或其他支持 WSGI 的 Web 服务器运行服务。根据用于部署 OpenStack 的打包解决方案，WSGI 脚本可能位于/usr/bin 或/usr/local/bin.    placement-api，作为标准的 WSGI 脚本，提供了 application大多数 WSGI 服务器期望找到的模块级属性。这意味着可以在许多不同的服务器上运行它，从而在面对不同的部署场景时提供灵活性。常见的场景包括：

- 带有mod_wsgi的apache2

- apache2 与mod_proxy_uwsgi

- nginx与uwsgi

- nginx 与gunicorn

当放置第一次添加到 DevStack时，它使用了mod_wsgi样式。后来它被更新为使用mod_proxy_uwsgi。

DevStack 配置为在/placement  http 或 https (80或443) 的默认端口上托管放置，具体取决于是否使用 TLS。使用默认端口是可取的。

默认情况下，placement应用程序将从/etc/placement/placement.conf. 可以通过 OS_PLACEMENT_CONFIG_DIR在启动应用程序的进程的环境中设置来更改配置文件所在的目录。随着最近的版本oslo.config，配置选项也可以在环境中设置

> 当使用带有前端（例如，apache2 或 nginx）的 uwsgi 时，需要确保 uwsgi 进程正在运行。在 DevStack 中，这是通过[systemd](https://review.opendev.org/#/c/448323/)完成的。这是管理 uwsgi 的许多不同方法之一。

**2.同步数据库**

placement服务使用自己的数据库，在 placement_database配置部分定义。必须设置该 placement_database.connection选项，否则服务将无法启动。命令行工具placement-manage 可用于将数据库表迁移到正确的形式，包括创建它们。该选项描述的数据库必须已经存在并定义了适当的访问控制。connection

同步的另一个选项是在配置中 placement_database.sync_on_startup设置True。这将在放置 Web 服务启动时执行所有丢失的数据库迁移。您是选择自动同步还是使用命令行工具取决于您的环境和部署工具的限制。

**3. 创建账户并更新服务目录**

在 Keystone 中创建一个具有**管理员角色的****放置服务用户。**

放置 API 是一个单独的服务，因此应该在服务目录中的**放置**服务类型下注册。放置的客户端，例如 nova-compute 节点中的资源跟踪器，将使用服务目录来查找放置端点。

/placement  Devstack 在默认 HTTP 端口 (80) 上使用前缀而不是使用独立端口设置放置服务 。

### 安装包

Placement 作为数据模型上的 Web 服务运行。安装涉及创建必要的数据库以及安装和配置 Web 服务。这是一个简单的过程，但是有很多步骤可以将布局与 OpenStack 云的其余部分集成。

一些其他 OpenStack 服务需要Placement ，特别是 nova，因此它应该安装在其他服务之前，但在 Identity（keystone）之后。

### 安装

这里可以使用python安装：https://docs.openstack.org/placement/train/install/from-pypi.html

这里就跳过，还是使用centos 保持一致安装

在安装和配置placement服务之前，您必须创建数据库、服务凭证和 API 端点。

https://docs.openstack.org/placement/train/install/install-rdo.html

#### 先决条件

```mysql
MariaDB [(none)]> create database placement;
Query OK, 1 row affected (0.000 sec)

MariaDB [(none)]> grant all on placement.* to 'placement'@'localhost' identified by 'PLACEMENT_DBPASS';
Query OK, 0 rows affected (0.002 sec)

MariaDB [(none)]> grant all on placement.* to 'placement'@'%' identified by 'PLACEMENT_DBPASS';
Query OK, 0 rows affected (0.000 sec)

MariaDB [(none)]> select host,user from mysql.user;
+-----------+-----------+
| host      | user      |
+-----------+-----------+
| %         | glance    |
| %         | keystone  |
| %         | placement |
| 127.0.0.1 | root      |
| ::1       | root      |
| localhost | glance    |
| localhost | keystone  |
| localhost | placement |
| localhost | root      |
+-----------+-----------+
9 rows in set (0.001 sec)
```

PLACEMENT_DBPASS 替换为自定义密码

#### 配置用户和接口

```sh
-- 获取管理员环境变量
[root@controller ~]# . /server/scripts/admin-openrc.sh 

-- 使用您选择的 PLACEMENT_PASS 创建一个 Placement 服务用户
[root@controller ~]# openstack user create --domain default --password-prompt placement
[root@controller ~]# openstack user create --domain default --password-prompt placement
User Password:   placement
Repeat User Password:    placement
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | e64d178e39004642a906504dfb2faca5 |
| name                | placement                        |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+

-- 将 Placement 用户添加到具有管理员角色的服务项目中：
[root@controller ~]# openstack role add --project service --user placement admin

-- 在服务目录中创建 Placement API 条目：
[root@controller ~]# openstack service create --name placement --description "Placement API" placement
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Placement API                    |
| enabled     | True                             |
| id          | 28f0498358e645e2883e51a657040ca1 |
| name        | placement                        |
| type        | placement                        |
+-------------+----------------------------------+

```

创建 Placement API 服务端点：

> 根据您的环境，端点的 URL 会因端口（可能是 8780 而不是 8778，或者根本没有端口）和主机名而异。您有责任确定正确的 URL。

```sh
[root@controller ~]# openstack endpoint create --region RegionOne placement public http://controller:8778
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 8144c19843de4c70b34d6092687d3663 |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 28f0498358e645e2883e51a657040ca1 |
| service_name | placement                        |
| service_type | placement                        |
| url          | http://controller:8778           |
+--------------+----------------------------------+

[root@controller ~]# openstack endpoint create --region RegionOne  placement internal http://controller:8778
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 1f0468e870204532885851f1af8083b1 |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 28f0498358e645e2883e51a657040ca1 |
| service_name | placement                        |
| service_type | placement                        |
| url          | http://controller:8778           |
+--------------+----------------------------------+

[root@controller ~]# openstack endpoint create --region RegionOne placement admin http://controller:8778
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | ab50dda6aa01444487bf23af00d85590 |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 28f0498358e645e2883e51a657040ca1 |
| service_name | placement                        |
| service_type | placement                        |
| url          | http://controller:8778           |
+--------------+----------------------------------+

[root@controller ~]# openstack endpoint list
```

![image-20220310095101577](openstack笔记.assets/image-20220310095101577.png)

#### 安装和配置组件

```sh
1.安装
[root@controller ~]# yum install openstack-placement-api -y

2.修改配置文件/etc/placement/placement.conf
[root@controller ~]# cp /etc/placement/placement.conf{,.bak}
#修改
499 [placement_database]  
512 connection = mysql+pymysql://placement:PLACEMENT_DBPASS@controller/placement  

191 [api]
209 auth_strategy = keystone   

241 [keystone_authtoken] 
271 auth_url = http://controller:5000/v3
318 memcached_servers = controller:11211    
399 auth_type = password

401 project_domain_name = Default
402 user_domain_name = Default
403 project_name = service
404 username = placement
405 password = placement  

3.同步数据库
[root@controller ~]# su -s /bin/sh -c "placement-manage db sync" placement

4.查看下数据是否同步
[root@controller ~]# mysql -uplacement -pPLACEMENT_DBPASS -e "use placement;show tables;"
+------------------------------+
| Tables_in_placement          |
+------------------------------+
| alembic_version              |
| allocations                  |
| consumers                    |
| inventories                  |
| placement_aggregates         |
| projects                     |
| resource_classes             |
| resource_provider_aggregates |
| resource_provider_traits     |
| resource_providers           |
| traits                       |
| users                        |
+------------------------------+

5.重启httpd
[root@controller ~]# systemctl restart httpd.service 

[root@controller ~]# ss -lnt |grep 8778
LISTEN     0      128       [::]:8778                  [::]:*      
```

> project_domain_name 和 user_domain_name 的值需要与您的 keystone 配置同步。

#### 验证

```sh
1.加载admin变量
[root@controller ~]# . /server/scripts/admin-openrc.sh 

2.执行状态检查确保一切正常
[root@controller ~]# placement-status upgrade check
+----------------------------------+
| Upgrade Check Results            |
+----------------------------------+
| Check: Missing Root Provider IDs |
| Result: Success                  |
| Details: None                    |
+----------------------------------+
| Check: Incomplete Consumers      |
| Result: Success                  |
| Details: None                    |
+----------------------------------+

3.针对placement API 添加一些插件
- 安装osc-placement
#这里需要安装python3,因为安装pip需要使用到，产品使用的python2已经停止维护了
[root@controller ~]# yum install python3 -y
[root@controller ~]# wget https://bootstrap.pypa.io/pip/3.6/get-pip.py
#下载pip
[root@controller ~]# python3 get-pip.py 

-- 安装插件
[root@controller ~]# pip install osc-placement
这里有什么错误就需要排错。不熟悉python的还是有点困难
```

插件安装：https://docs.openstack.org/osc-placement/latest/



## compute 安装

OpenStack 计算是基础设施即服务 (IaaS) 系统的主要部分。主要模块是用 Python 实现的

OpenStack Compute 与用于身份验证的 OpenStack Identity、用于资源库存跟踪和选择的 OpenStack Placement、用于磁盘和服务器映像的 OpenStack Image 服务以及用于用户和管理界面的 OpenStack Dashboard 交互。image访问受项目和用户的限制；每个项目的配额是有限的（例如，实例的数量）。OpenStack Compute 可以在标准硬件上水平扩展，并下载image以启动实例。

**OpenStack Compute 由以下区域及其组件组成：**

==nova-api服务==

接受并响应最终用户计算 API 调用。该服务支持 OpenStack Compute API。它执行一些策略并启动大多数编排活动，例如运行实例。

==nova-api-metadata服务==

接受来自实例的元数据请求。nova-api-metadata当您以多主机模式运行安装时，通常会使用该服务nova-network 。

==nova-compute服务==

通过管理程序 API 创建和终止虚拟机实例的工作守护程序。例如：

- 用于 XenServer/XCP 的 XenAPI
- 用于 KVM 或 QEMU 的 libvirt
- 适用于 VMware 的 VMwareAPI

处理相当复杂。基本上，守护进程接受队列中的操作并执行一系列系统命令，例如启动 KVM 实例并更新其在数据库中的状态。

==nova-scheduler服务==

从队列中获取虚拟机实例请求并确定它在哪个计算服务器主机上运行。

==nova-conductor模块==

调解`nova-compute`服务和数据库之间的交互。它消除了服务对云数据库的直接访问 `nova-compute`。`nova-conductor`模块水平缩放。但是，请勿将其部署在==nova-compute==服务运行的节点上。

==nova-novncproxy守护进程==

为通过 VNC 连接访问正在运行的实例提供代理。支持基于浏览器的 novnc 客户端。

==nova-spicehtml5proxy守护进程==

为通过 SPICE 连接访问正在运行的实例提供代理。支持基于浏览器的 HTML5 客户端。

==nova-xvpvncproxy守护进程==

为通过 VNC 连接访问正在运行的实例提供代理。支持特定于 OpenStack 的 Java 客户端。

> 自 19.0.0 版起已弃用：**nova-xvpvnxproxy**自 19.0.0 (Stein) 起已弃用，并将在即将发布的版本中删除。

==消息队列==
用于在守护进程之间传递消息的中心枢纽。通常使用 RabbitMQ实现，但 也可以使用其他选项。

==SQL 数据库==

存储云基础架构的大多数构建时和运行时状态，包括：

- 可用的实例类型
- 正在使用的实例
- 可用网络
- 项目

理论上，OpenStack Compute 可以支持 SQLAlchemy 支持的任何数据库。常见的数据库是用于测试和开发工作的 SQLite3、MySQL、MariaDB 和 PostgreSQL。

### 安装和配置控制节点

在控制器上安装nova 的compute服务

#### 先决条件

数据库准备

```sh
-- 创建数据库  nova_api、nova和nova_cell0
MariaDB [(none)]> create database nova_api;
Query OK, 1 row affected (0.000 sec)

MariaDB [(none)]> create database nova;
Query OK, 1 row affected (0.001 sec)

MariaDB [(none)]> create database nova_cell0;
Query OK, 1 row affected (0.000 sec)

MariaDB [(none)]> grant all on nova_api.* to 'nova'@'localhost' identified by 'NOVA_DBPASS';
Query OK, 0 rows affected (0.001 sec)

MariaDB [(none)]> grant all on nova_api.* to 'nova'@'%' identified by 'NOVA_DBPASS';
Query OK, 0 rows affected (0.003 sec)

MariaDB [(none)]> grant all on nova.* to 'nova'@'localhost' identified by 'NOVA_DBPASS';
Query OK, 0 rows affected (0.000 sec)

MariaDB [(none)]> grant all on nova.* to 'nova'@'%' identified by 'NOVA_DBPASS';
Query OK, 0 rows affected (0.000 sec)

MariaDB [(none)]> grant all on nova_cell0.* to 'nova'@'localhost' identified by 'NOVA_DBPASS';
Query OK, 0 rows affected (0.000 sec)

MariaDB [(none)]> grant all on nova_cell0.* to 'nova'@'%' identified by 'NOVA_DBPASS';
Query OK, 0 rows affected (0.001 sec)
```

NOVA_DBPASS 是自定义的密码，

```sh
-- 加载admin环境变量
[root@controller ~]# . /server/scripts/admin-openrc.sh 

-- 创建计算服务凭证
[root@controller ~]# openstack user create --domain default --password-prompt nova
User Password:   nova
Repeat User Password:    nova
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | e2b59c78397f4a039a5223fabf4b9d41 |
| name                | nova                             |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+

-- 将nova添加到admin
[root@controller ~]# openstack role add --project service --user nova admin

-- 创建nova服务实体
[root@controller ~]# openstack service create --name nova --description "OpenStack Compute" compute
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Compute                |
| enabled     | True                             |
| id          | 848fcfbed03e4279a26e124012e67866 |
| name        | nova                             |
| type        | compute                          |
+-------------+----------------------------------+

```

创建计算API服务接口

```sh
[root@controller ~]# openstack endpoint create --region RegionOne compute public http://controller:8774/v2.1
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 3c8c4ca2f1b8401ba32fa88bb108547d |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 848fcfbed03e4279a26e124012e67866 |
| service_name | nova                             |
| service_type | compute                          |
| url          | http://controller:8774/v2.1      |
+--------------+----------------------------------+

[root@controller ~]# openstack endpoint create --region RegionOne compute internal http://controller:8774/v2.1
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 0efbbd5f049d433f865aa834dba26cbf |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 848fcfbed03e4279a26e124012e67866 |
| service_name | nova                             |
| service_type | compute                          |
| url          | http://controller:8774/v2.1      |
+--------------+----------------------------------+

[root@controller ~]# openstack endpoint create --region RegionOne compute admin http://controller:8774/v2.1
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | ade61ae0b43540cfbfdf7cbdb5010a8f |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 848fcfbed03e4279a26e124012e67866 |
| service_name | nova                             |
| service_type | compute                          |
| url          | http://controller:8774/v2.1      |
+--------------+----------------------------------+

```

#### 安装和配置组件

```sh
1.安装
[root@controller ~]# yum install openstack-nova-api openstack-nova-conductor openstack-nova-novncproxy openstack-nova-scheduler -y

2.编辑/etc/nova/nova.conf
[root@controller ~]# cp /etc/nova/nova.conf{,.bak}
1 [DEFAULT]   
1362 enabled_apis=osapi_compute,metadata 

1829 [api_database]
1844 connection=mysql+pymysql://nova:NOVA_DBPASS@controller/nova_api  

2315 [database] 
2334 connection=mysql+pymysql://nova:NOVA_DBPASS@controller/nova   

-- 在该[DEFAULT]部分中，配置RabbitMQ消息队列访问：
1 [DEFAULT]  
1587 transport_url=rabbit://openstack:RABBIT_PASS@controller:5672/  

(下面是我们上面安装rabbit时候设置的密码)
1587 transport_url=rabbit://openstack:openstack@controller:5672/  
（替换为您在 中为 帐户RABBIT_PASS选择的密码。openstackRabbitMQ）

在[api]和[keystone_authtoken]部分中，配置身份服务访问：
1629 [api]
1646 auth_strategy=keystone

3204 [keystone_authtoken]  
3219 www_authenticate_uri=http://controller:5000/ 
3235 auth_url=http://controller:5000/ 
3282 memcached_servers=controller:11211  
3363 auth_type=password 
3365 project_domain_name = Default
3366 user_domain_name = Default
3367 project_name = service
3368 username = nova
3369 password = NOVA_PASS
#NOVA_PASS是上面服务的密码nova

在该[DEFAULT]部分中，配置my_ip选项以使用控制器节点的管理接口 IP 地址：
1 [DEFAULT]  
508 my_ip=192.168.100.200 

在该[DEFAULT]部分中，启用对网络服务的支持：
1 [DEFAULT] 
816 use_neutron=true  
1169 firewall_driver=nova.virt.firewall.NoopFirewallDriver    
```

>  默认情况下，Compute 使用内部防火墙驱动程序。由于网络服务包含防火墙驱动程序，因此您必须使用防火墙驱动程序禁用计算防火墙驱动 `nova.virt.firewall.NoopFirewallDriver`程序。

配置**/etc/nova/nova.conf**`[neutron]`的部分。

```sh
-- 在该[vnc]部分中，将 VNC 代理配置为使用控制器节点的管理接口 IP 地址：

5885 [vnc]    
5899 enabled=true   
5923 server_listen=192.168.10.200  # $my_ip
5931 server_proxyclient_address=192.168.10.200   # $my_ip

-- 在该[glance]部分中，配置image服务 API 的位置：
2631 [glance]  
2642 api_servers=http://controller:9292  

-- 在该[oslo_concurrency]部分中，配置锁定路径：
4255 [oslo_concurrency] 
4269 lock_path=/var/lib/nova/tmp    

-- 在该[placement]部分中，配置对 Placement 服务的访问：
4761 [placement] 
4766 region_name = RegionOne
4767 project_domain_name = Default
4768 project_name = service
4769 auth_type = password
4770 user_domain_name = Default
4771 auth_url = http://controller:5000/v3
4772 username = placement
4773 password = PLACEMENT_PASS  #placement
```

> 替换为您为安装 [Placement](https://docs.openstack.org/placement/train/install/)`PLACEMENT_PASS`时创建的服务用户选择的密码 。注释掉或删除该部分中的任何其他选项。`placement[placement]`

填充==nova-api==数据库

```sh
[root@controller ~]# su -s /bin/sh -c "nova-manage api_db sync" nova
```

注册==cell0==数据库

```sh
[root@controller ~]# su -s /bin/sh -c "nova-manage cell_v2 map_cell0" nova
```

创建==cell1==单元格

```sh
[root@controller ~]# su -s /bin/sh -c "nova-manage cell_v2 create_cell --name=cell1 --verbose" nova
49351abd-5902-40ae-bee2-d6f7d8774aaa
```

填充==nova==数据库

```sh
[root@controller ~]# su -s /bin/sh -c "nova-manage db sync" nova
```

验证 nova cell0 和 cell1 是否正确注册：

```sh
[root@controller ~]# su -s /bin/sh -c "nova-manage cell_v2 list_cells" nova
```

![image-20220310163121763](openstack笔记.assets/image-20220310163121763.png)

查看数据库

```sh
[root@controller ~]# mysql -unova -pNOVA_DBPASS -e "use nova;show tables"

[root@controller ~]# mysql -unova -pNOVA_DBPASS -e "use nova_api;show tables"

[root@controller ~]# mysql -unova -pNOVA_DBPASS -e "use nova_cell0;show tables"
```

启动

```sh
[root@controller ~]# names=(api scheduler conductor novncproxy)
[root@controller ~]# for i in ${names[@]};do systemctl enable openstack-nova-${i};systemctl start openstack-nova-${i};done

-- 检查
[root@controller ~]# for i in ${names[@]};do systemctl status openstack-nova-${i};done

--- 官网
# systemctl enable \
    openstack-nova-api.service \
    openstack-nova-scheduler.service \
    openstack-nova-conductor.service \
    openstack-nova-novncproxy.service
# systemctl start \
    openstack-nova-api.service \
    openstack-nova-scheduler.service \
    openstack-nova-conductor.service \
    openstack-nova-novncproxy.service
```



### 安装和配置计算节点

==计算节点==上安装和配置计算服务

本部分假设您按照本指南中的说明逐步配置第一个计算节点。多节点依次配置

计算节点上使用 Quick EMUlator (QEMU) 管理程序和基于内核的 VM (KVM) 扩展。在旧硬件上，此配置使用通用 QEMU 管理程序。

```sh
1.安装软件
[root@controller ~]# yum install openstack-nova-compute -y

2.编辑/etc/nova/nova.conf
[root@compute ~]# cp /etc/nova/nova.conf{,.bak}
-- 在该[DEFAULT]部分中，仅启用计算和元数据 API：
1 [DEFAULT]
1361 enabled_apis=osapi_compute,metadata 

-- 在该[DEFAULT]部分中，配置RabbitMQ消息队列访问：
1587 transport_url=rabbit://openstack:openstack@controller
#替换为您在 中为 帐户RABBIT_PASS选择的密码。openstackRabbitMQ

-- 在[api]和[keystone_authtoken]部分中，配置身份服务访问
1629 [api]  
1646 auth_strategy=keystone

3202 [keystone_authtoken] 
3204 www_authenticate_uri = http://controller:5000/
3205 auth_url = http://controller:5000/
3206 memcached_servers = controller:11211
3207 auth_type = password
3208 project_domain_name = Default
3209 user_domain_name = Default
3210 project_name = service
3211 username = nova
3212 password = NOVA_PASS   #nova

-- 在该[DEFAULT]部分中，配置my_ip选项：
508 my_ip=MANAGEMENT_INTERFACE_IP_ADDRESS    
#替换为计算节点上管理网络接口的 IP 地址，对于示例架构MANAGEMENT_INTERFACE_IP_ADDRESS中的第一个节点，通常为 192.168.100.201 。

816 use_neutron=true
1169 firewall_driver=nova.virt.firewall.NoopFirewallDriver 
```

> 默认情况下，Compute 使用内部防火墙服务。由于网络包括防火墙服务，您必须使用`nova.virt.firewall.NoopFirewallDriver`防火墙驱动程序禁用计算防火墙服务。

配置**/etc/nova/nova.conf**`[neutron]`的部分https://docs.openstack.org/neutron/train/install/compute-install-rdo.html#configure-the-compute-service-to-use-the-networking-service

在该`[vnc]`部分中，启用和配置远程控制台访问：

```sh
5881 [vnc]  
5895 enabled=true 
5919 server_listen=0.0.0.0
5927 server_proxyclient_address=192.168.10.201   # $my_ip
5933 novncproxy_base_url=http://controller:6080/vnc_auto.html  
```

服务器组件侦听所有 IP 地址，代理组件仅侦听计算节点的管理接口 IP 地址。基本 URL 指示您可以使用 Web 浏览器访问此计算节点上实例的远程控制台的位置。

> 如果访问远程控制台的 Web 浏览器位于无法解析主机名的`controller`主机上，则必须替换 `controller`为控制器节点的管理接口 IP 地址。

```sh
-- [glance]部分
2629 [glance]   
2640 api_servers=http://controller:9292  

4251 [oslo_concurrency] 
4265 lock_path=/var/lib/nova/tmp    

4757 [placement]  
4759 region_name = RegionOne
4760 project_domain_name = Default
4761 project_name = service
4762 auth_type = password
4763 user_domain_name = Default
4764 auth_url = http://controller:5000/v3
4765 username = placement
4766 password = PLACEMENT_PASS   
```

替换为您在身份服务中`PLACEMENT_PASS`为用户选择的密码 。`placement`注释掉该`[placement]`部分中的任何其他选项。

#### 完成安装

```sh
-- 确定计算节点是否支持虚拟化
[root@compute ~]# egrep -c '(vmx|svm)' /proc/cpuinfo
0
#如果此命令返回 1 或更大的值，则您的计算节点支持硬件加速，这通常不需要额外配置。
#如果此命令返回值为零，则您的计算节点不支持硬件加速，您必须将 libvirt 配置为使用 QEMU 而不是 KVM。
编辑文件中的[libvirt]部分，/etc/nova/nova.conf
[libvirt]
# ...
virt_type = qemu

没有开启虚拟化返回0，那就需要修改配置文件
```

![image-20220310225525986](openstack笔记.assets/image-20220310225525986.png)

```sh
-- 经过打开之后在运行
[root@compute ~]# egrep -c '(vmx|svm)' /proc/cpuinfo 
1
```

启动服务

```sh
[root@compute ~]# systemctl enable libvirtd openstack-nova-compute;systemctl start libvirtd openstack-nova-compute
```

> 如果nova-compute服务无法启动，请检查 /var/log/nova/nova-compute.log. 该错误消息可能表明控制器节点上的防火墙正在阻止访问端口 5672。将防火墙配置为打开控制器节点上的端口 5672 并重新启动 计算节点上的服务。AMQP server on controller:5672 is unreachablenova-compute

#### 将计算节点添加到单元数据库

**在控制器**节点上运行以下命令。

```sh
1.获取管理员凭据以启用仅限管理员的 CLI 命令，然后确认数据库中有计算主机：
[root@controller ~]# . /server/scripts/admin-openrc.sh 
[root@controller ~]# openstack compute service list --service nova-compute
+----+--------------+---------+------+---------+-------+----------------------------+
| ID | Binary       | Host    | Zone | Status  | State | Updated At                 |
+----+--------------+---------+------+---------+-------+----------------------------+
|  5 | nova-compute | compute | nova | enabled | up    | 2022-03-10T15:03:54.000000 |
+----+--------------+---------+------+---------+-------+----------------------------+

-- 发现计算主机：
[root@controller ~]# su -s /bin/sh -c "nova-manage cell_v2 discover_hosts --verbose" nova
Found 2 cell mappings.
Skipping cell0 since it does not contain hosts.
Getting computes from cell 'cell1': 49351abd-5902-40ae-bee2-d6f7d8774aaa
Found 0 unmapped computes in cell: 49351abd-5902-40ae-bee2-d6f7d8774aaa

```

添加新计算节点时，必须在控制器节点上运行 nova-manage cell_v2 discover_hosts 以注册这些新计算节点。或者，您可以在 /etc/nova/nova.conf 中设置适当的时间间隔：

*这里配置好了但是计算节点网络不对。*还需要继续下一步



#### 验证

```sh
[root@controller ~]# . /server/scripts/admin-openrc.sh 

[root@controller ~]# openstack compute service list
```

![image-20220310231104019](openstack笔记.assets/image-20220310231104019.png)

此输出应指示在控制器节点上启用了两个服务组件，在计算节点上启用了一个服务组件。

列出身份服务中的 API 端点以验证与身份服务的连接

```sh
[root@controller ~]# openstack catalog list
```

![image-20220310231221935](openstack笔记.assets/image-20220310231221935.png)

列出image服务中的image以验证与image服务的连接性：

```sh
[root@controller ~]# openstack image list
```

![image-20220310231337002](openstack笔记.assets/image-20220310231337002.png)

检查单元格和放置 API 是否成功运行，以及其他必要的先决条件是否到位

```sh
[root@controller ~]# nova-status upgrade check
Error:
Traceback (most recent call last):
  File "/usr/lib/python2.7/site-packages/nova/cmd/status.py", line 398, in main
    ret = fn(*fn_args, **fn_kwargs)
  File "/usr/lib/python2.7/site-packages/oslo_upgradecheck/upgradecheck.py", line 102, in check
    result = func(self)
  File "/usr/lib/python2.7/site-packages/nova/cmd/status.py", line 164, in _check_placement
    versions = self._placement_get("/")
  File "/usr/lib/python2.7/site-packages/nova/cmd/status.py", line 154, in _placement_get
    return client.get(path, raise_exc=True).json()
  File "/usr/lib/python2.7/site-packages/keystoneauth1/adapter.py", line 386, in get
    return self.request(url, 'GET', **kwargs)
  File "/usr/lib/python2.7/site-packages/keystoneauth1/adapter.py", line 248, in request
    return self.session.request(url, method, **kwargs)
  File "/usr/lib/python2.7/site-packages/keystoneauth1/session.py", line 961, in request
    raise exceptions.from_response(resp, method, url)
Forbidden: Forbidden (HTTP 403)

```

错误了。这里问题应该是网络问题



## neutron 安装 （在compute上面）

OpenStack Networking (neutron) 允许您创建由其他 OpenStack 服务管理的接口设备并将其连接到网络。可以实施插件以适应不同的网络设备和软件，为 OpenStack 架构和部署提供灵活性。

**neutron-server**

接受 API 请求并将其路由到相应的 OpenStack Networking 插件以执行操作

**OpenStack Networking 插件和代理**

插入和拔出端口，创建网络或子网，并提供 IP 寻址。这些插件和代理因特定云中使用的供应商和技术而异。 OpenStack Networking 附带适用于 Cisco 虚拟和物理交换机、NEC OpenFlow 产品、Open vSwitch、Linux 桥接和 VMware NSX 产品的插件和代理。 常见的代理有 L3（第 3 层）、DHCP（动态主机 IP 寻址）和插件代理。

**消息队列**

大多数 OpenStack Networking 安装使用它在 neutron-server 和各种代理之间路由信息。还充当数据库来存储特定插件的网络状态。

OpenStack Networking 主要与 OpenStack Compute 交互，为其实例提供网络和连接。



### neutron概念

OpenStack Networking (neutron) 管理 OpenStack 环境中虚拟网络基础设施 (VNI) 和物理网络基础设施 (PNI) 的访问层方面的所有网络方面。OpenStack Networking 使项目能够创建高级虚拟网络拓扑，其中可能包括防火墙和虚拟专用网络 (VPN) 等服务。

网络提供网络、子网和路由器作为对象抽象。每个抽象都具有模仿其物理对应物的功能：网络包含子网，路由器在不同子网和网络之间路由流量。

任何给定的网络设置都至少有一个外部网络。与其他网络不同，外部网络不仅仅是一个虚拟定义的网络。相反，它代表了一个可以在 OpenStack 安装之外访问的物理外部网络切片的视图。外部网络上的任何人都可以访问外部网络上的 IP 地址。

除了外部网络，任何网络设置都有一个或多个内部网络。这些软件定义的网络直接连接到虚拟机。只有任何给定内部网络上的虚拟机，或通过接口连接到类似路由器的子网上的虚拟机，才能直接访问连接到该网络的虚拟机。

对于外部网络访问虚拟机，反之亦然，网络之间需要路由器。每台路由器都有一个连接外部网络的网关和一个或多个连接内部网络的接口。与物理路由器一样，子网可以访问连接到同一路由器的其他子网上的机器，机器可以通过路由器的网关访问外部网络。

此外，您可以将外部网络上的 IP 地址分配给内部网络上的端口。每当某物连接到子网时，该连接就称为端口。您可以将外部网络 IP 地址与 VM 的端口相关联。这样，外部网络上的实体就可以访问虚拟机。

网络还支持*安全组*。安全组使管理员能够在组中定义防火墙规则。一个 VM 可以属于一个或多个安全组，网络应用这些安全组中的规则来阻止或取消阻止该 VM 的端口、端口范围或流量类型。

Networking 使用的每个插件都有自己的概念。虽然对于操作 VNI 和 OpenStack 环境并不重要，但理解这些概念可以帮助您设置网络。所有网络安装都使用核心插件和安全组插件（或仅使用 No-Op 安全组插件）。此外，还提供防火墙即服务 (FWaaS)。

参考：https://docs.openstack.org/neutron/train/install/install-rdo.html



### 安装和配置控制器节点

#### 先决条件

创建数据库并收授权

```mysql
MariaDB [(none)]> create database neutron;
Query OK, 1 row affected (0.001 sec)

MariaDB [(none)]> grant all on neutron.* to 'neutron'@'localhost' identified by 'NEUTRON_DBPASS';
Query OK, 0 rows affected (0.004 sec)

MariaDB [(none)]> grant all on neutron.* to 'neutron'@'%' identified by 'NEUTRON_DBPASS';
Query OK, 0 rows affected (0.001 sec)

```

加载环境变量

```sh
[root@controller ~]# . /server/scripts/admin-openrc.sh 
```

创建服务凭证

```sh
[root@controller ~]# openstack user create --domain default --password-prompt neutron
User Password:   neutron
Repeat User Password:   neutron
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | ec8aa57b5c114b9baab7d9a8c8f5cdda |
| name                | neutron                          |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+

-- 将neutron添加到admin角色
[root@controller ~]# openstack role add --project service --user neutron admin

-- 创建服务实体
[root@controller ~]# openstack service create --name neutron --description "OpenStack Networking" network
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Networking             |
| enabled     | True                             |
| id          | e8777a6d02c14e84a45bba6445da2a0a |
| name        | neutron                          |
| type        | network                          |
+-------------+----------------------------------+
```

创建api

```sh
[root@controller ~]# openstack endpoint create --region RegionOne network public http://controller:9696+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 9cba8725160040dc84c49d08e7d172a9 |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | e8777a6d02c14e84a45bba6445da2a0a |
| service_name | neutron                          |
| service_type | network                          |
| url          | http://controller:9696           |
+--------------+----------------------------------+

[root@controller ~]# openstack endpoint create --region RegionOne network internal http://controller:9696
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | e7ad25a2065f43efac953a5404b5b103 |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | e8777a6d02c14e84a45bba6445da2a0a |
| service_name | neutron                          |
| service_type | network                          |
| url          | http://controller:9696           |
+--------------+----------------------------------+

[root@controller ~]# openstack endpoint create --region RegionOne network admin http://controller:9696
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | a5da90d553c24437a6a7a5aad4a9e376 |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | e8777a6d02c14e84a45bba6445da2a0a |
| service_name | neutron                          |
| service_type | network                          |
| url          | http://controller:9696           |
+--------------+----------------------------------+

-- 检查
[root@controller ~]# openstack user list
+----------------------------------+-----------+
| ID                               | Name      |
+----------------------------------+-----------+
| def6d7d6f7144af8914e9ec83fef0aa0 | admin     |
| 557b6c68221845cc924fbbb1f2e61688 | myuser    |
| 5472d65f1d914f6489814a0faee3fecc | glance    |
| e64d178e39004642a906504dfb2faca5 | placement |
| e2b59c78397f4a039a5223fabf4b9d41 | nova      |
| ec8aa57b5c114b9baab7d9a8c8f5cdda | neutron   |
+----------------------------------+-----------+
[root@controller ~]# openstack serv
server   service  
[root@controller ~]# openstack service list
+----------------------------------+-----------+-----------+
| ID                               | Name      | Type      |
+----------------------------------+-----------+-----------+
| 08d071084dcf40c8832a256d4b8119cd | glance    | image     |
| 28f0498358e645e2883e51a657040ca1 | placement | placement |
| 780eeccf32d94805b987f3b726fc7ec5 | keystone  | identity  |
| 848fcfbed03e4279a26e124012e67866 | nova      | compute   |
| caa4c027149247d595923d107a328b29 | glance    | image     |
| e8777a6d02c14e84a45bba6445da2a0a | neutron   | network   |
+----------------------------------+-----------+-----------+
[root@controller ~]# openstack endpoint list

```

![image-20220313215630649](openstack笔记.assets/image-20220313215630649.png)

#### 配置网络选项

- 公网

  该架构仅支持将实例附加到供应商（外部）网络。没有自助服务（专用）网络、路由器或浮动 IP 地址。只有该`admin`或其他特权用户可以管理提供商网络。

  https://docs.openstack.org/neutron/train/install/controller-install-option1-rdo.html

- 私网

  提供自助服务和供应商网络之间的连接的路由器，也就是内网转外网

  https://docs.openstack.org/neutron/train/install/controller-install-option2-rdo.html

  > 网络部署上IPV4都是私网NAT。IPV6才可能公网暴露，但是安全也是有极大的隐患

我们直接来内网的方式。

##### 安装组件

*在控制器*节点上安装和配置网络组件。

```sh
[root@controller ~]# yum install openstack-neutron openstack-neutron-ml2 openstack-neutron-linuxbridge ebtables -y
```

##### 配置

没有的自己添加

```sh
-- 编辑/etc/neutron/neutron.conf
[root@controller ~]# cp /etc/neutron/neutron.conf{,.bak}
[root@controller ~]# vim /etc/neutron/neutron.conf
254 [database]  
273 connection = mysql+pymysql://neutron:NEUTRON_DBPASS@controller/neutron 

-- 在该[DEFAULT]部分中，启用 Modular Layer 2 (ML2) 插件并禁用其他插件：
1 [DEFAULT]  #自己添加
3 core_plugin = ml2
4 service_plugins =    

-- transport_url = rabbit://openstack:RABBIT_PASS@controller  #替换为您在 RabbitMQ 中RABBIT_PASS为帐户选择的密码 。openstack
188 transport_url = rabbit://openstack:openstack@controller   
194 auth_strategy = keystone    #添加

365 [keystone_authtoken]     
381 www_authenticate_uri = http://controller:5000 
397 auth_url = http://controller:5000
445 memcached_servers = controller:11211
530 auth_type = password
531 project_domain_name = default
532 user_domain_name = default
533 project_name = service
534 username = neutron
535 password = neutron     
```

在`[DEFAULT]`和`[nova]`部分中，配置 Networking 以通知 Compute 网络拓扑更改

```sh
6 notify_nova_on_port_status_changes = true
7 notify_nova_on_port_data_changes = true

1063 [nova]
1064 auth_url = http://controller:5000
1065 auth_type = password
1066 project_domain_name = default
1067 user_domain_name = default
1068 region_name = RegionOne
1069 project_name = service
1070 username = nova
1071 password = nova

543 [oslo_concurrency]
557 lock_path = /var/lib/neutron/tmp   
```

##### 配置 Modular Layer 2 (ML2) 插件

ML2 插件使用 Linux 桥接机制为实例构建第 2 层（桥接和交换）虚拟网络基础架构。

编辑`/etc/neutron/plugins/ml2/ml2_conf.ini`文件并完成以下操作：

```sh
[root@controller ~]# cp /etc/neutron/plugins/ml2/ml2_conf.ini{,.bak}
[root@controller ~]# vim /etc/neutron/plugins/ml2/ml2_conf.ini
type_drivers = flat,vlan     #启用平面和 VLAN 网络
tenant_network_types =       #禁用自助服务网络
memcache_drivers = linuxbridge     #启用 Linux 桥接机制

extension_drivers = port_security  #启用端口安全扩展驱动程序
flat_networks = provider   #将提供者虚拟网络配置为平面网络
enable_ipset = true      #启用 ipset 以提高安全组规则的效率：
```

>  配置 ML2 插件后，删除 `type_drivers`选项中的值可能会导致数据库不一致。

##### 配置 Linux 网桥代理

Linux 桥接代理为实例构建第 2 层（桥接和交换）虚拟网络基础架构并处理安全组。

```sh
-- 编辑/etc/neutron/plugins/ml2/linuxbridge_agent.ini
[root@controller ~]# cp /etc/neutron/plugins/ml2/linuxbridge_agent.ini{,.bak} 


```



























# 问题

安装服务后重启出现报错

```sh
WARNING! The remote SSH server rejected X11 forwarding request.
Last login: Thu Mar 10 14:43:22 2022 from v.lan
net_mlx5: cannot load glue library: libibverbs.so.1: cannot open shared object file: No such file or directory
net_mlx5: cannot initialize PMD due to missing run-time dependency on rdma-core libraries (libibverbs, libmlx5)
PMD: net_mlx4: cannot load glue library: libibverbs.so.1: cannot open shared object file: No such file or directory
PMD: net_mlx4: cannot initialize PMD due to missing run-time dependency on rdma-core libraries (libibverbs, libmlx4)
net_mlx5: cannot load glue library: libibverbs.so.1: cannot open shared object file: No such file or directory
net_mlx5: cannot initialize PMD due to missing run-time dependency on rdma-core libraries (libibverbs, libmlx5)
PMD: net_mlx4: cannot load glue library: libibverbs.so.1: cannot open shared object file: No such file or directory
PMD: net_mlx4: cannot initialize PMD due to missing run-time dependency on rdma-core libraries (libibverbs, libmlx4)

[root@controller ~]# yum install -y libibverbs
```

