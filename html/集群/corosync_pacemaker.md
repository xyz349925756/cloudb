# corosync pacemaker

https://clusterlabs.org/

https://access.redhat.com/documentation/zh-cn/red_hat_enterprise_linux/7/html-single/high_availability_add-on_reference/index#s1-ov-newfeatures-7.1-HAAR

http://crmsh.github.io/

https://access.redhat.com/documentation/zh-cn/red_hat_enterprise_linux/  redhat 资源文档库

https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/managing_systems_using_the_rhel_7_web_console/getting-started-with-the-rhel-web-console_system-management-using-the-rhel-7-web-console#installing-the-web-console_getting-started-with-the-web-console

管理平台

## 配置主机

```bash
[root@ceph01 ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
172.16.0.31 ceph01
172.16.0.32 ceph02
172.16.0.33 ceph03

#再添加心跳主机
echo -e "192.168.10.31 ceph01\n192.168.10.32 ceph02\n192.168.10.33 ceph03" >>/etc/hosts

[root@ceph03 ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
172.16.0.31 ceph01
172.16.0.32 ceph02
172.16.0.33 ceph03
192.168.10.31 ceph01
192.168.10.32 ceph02
192.168.10.33 ceph03
```

## ISCSI配置

生产很少使用，生产直接接ceph or glusterfs

这里做一个了解

```bash
[root@ceph01 ~]# yum install targetcli -y
[root@ceph01 ~]# systemctl start target;systemctl enable target
[root@ceph01 ~]# fdisk /dev/sdb 
Command (m for help): p
Disk /dev/sdb: 10.7 GB, 10737418240 bytes, 20971520 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0xc27fdeba
   Device Boot      Start         End      Blocks   Id  System

Command (m for help): n
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p
Partition number (1-4, default 1): 
First sector (2048-20971519, default 2048): 
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-20971519, default 20971519): 
Using default value 20971519
Partition 1 of type Linux and of size 10 GiB is set

Command (m for help): p

Disk /dev/sdb: 10.7 GB, 10737418240 bytes, 20971520 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0xc27fdeba

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048    20971519    10484736   83  Linux

Command (m for help): w
The partition table has been altered!

[root@ceph01 ~]# lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sdd               8:48   0   10G  0 disk 
sdb               8:16   0   10G  0 disk 
└─sdb1            8:17   0   10G  0 part 
sr0              11:0    1  4.4G  0 rom  
sdc               8:32   0   10G  0 disk 
sda               8:0    0   60G  0 disk 
├─sda2            8:2    0   59G  0 part 
│ ├─centos-swap 253:1    0    2G  0 lvm  [SWAP]
│ └─centos-root 253:0    0   57G  0 lvm  /
└─sda1            8:1    0    1G  0 part /boot

[root@ceph01 ~]# targetcli
Warning: Could not load preferences file /root/.targetcli/prefs.bin.
targetcli shell version 2.1.53
Copyright 2011-2013 by Datera, Inc and others.
For help on commands, type 'help'.

/> /backstores/block create cluster_voll /dev/sdb1   #使用命令创建存储
Created block storage object cluster_voll using /dev/sdb1.
/> /iscsi create iqn.2021-09.com.mwdserver:iscsimwd1   #配置ISCSI target
Created target iqn.2021-09.com.mwdserver:iscsimwd1.
Created TPG 1.
Global pref auto_add_default_portal=true
Created default portal listening on all IPs (0.0.0.0), port 3260.

/> /iscsi/iqn.2021-09.com.mwdserver:iscsimwd1/tpg1/luns create /backstores/block/cluster_voll  #创建LUM
Created LUN 0.

#创建ACLs  iqn.2021-09.com.test:client2是客户端名称标识
/> cd /iscsi/iqn.2021-09.com.mwdserver:iscsimwd1/tpg1/acls 
/iscsi/iqn.20...wd1/tpg1/acls> create iqn.2021-09.com.test:client1
Created Node ACL for iqn.2021-09.com.test:client1
Created mapped LUN 0.
/iscsi/iqn.20...wd1/tpg1/acls> create iqn.2021-09.com.test:client2
Created Node ACL for iqn.2021-09.com.test:client2
Created mapped LUN 0.
/iscsi/iqn.20...wd1/tpg1/acls> create iqn.2021-09.com.test:client3
Created Node ACL for iqn.2021-09.com.test:client3
Created mapped LUN 0.

#设置账号密码
/iscsi/iqn.20...wd1/tpg1/acls> cd iqn.2021-09.com.test:client1
/iscsi/iqn.20....test:client1> pwd
/iscsi/iqn.2021-09.com.mwdserver:iscsimwd1/tpg1/acls/iqn.2021-09.com.test:client1
/iscsi/iqn.20....test:client1> set auth userid=test
Parameter userid is now 'test'.
/iscsi/iqn.20....test:client1> set auth password=test
Parameter password is now 'test'.

#IP地址与端口
/iscsi/iqn.20....test:client1> cd /iscsi/iqn.2021-09.com.mwdserver:iscsimwd1/tpg1/portals/
/iscsi/iqn.20.../tpg1/portals> ls
o- portals .......................................................................................... [Portals: 1]
  o- 0.0.0.0:3260 ........................................................................................... [OK]

#如果没有开放则使用下面创建
/iscsi/iqn.2021-09.com.mwdserver:iscsimwd1/tpg1/portals create x.x.x.x:3260

#退出保存
/iscsi/iqn.20.../tpg1/portals> exit
Global pref auto_save_on_exit=true
Configuration saved to /etc/target/saveconfig.json

#重启服务
[root@ceph01 ~]# systemctl restart target

```

### ISCSI接入cluster

```bash
[root@ceph01 ~]# yum install -y iscsi-initiator-utils   #节点都安装

[root@ceph01 ~]# echo "InitiatorName=iqn.2021-09.com.test:client1" >/etc/iscsi/initiatorname.iscsi
#三节点都操作
```

设置验证账户和密码（三节点修改）

```bash
[root@ceph01 ~]# vim /etc/iscsi/iscsid.conf 
node.session.auth.authmethod = CHAP
node.session.auth.username = test
node.session.auth.password = test  
```

启动服务

```BASH
systemctl start iscsi ;systemctl enable iscsi
```

查找ISCSI设备（节点都操作）

```BASH
[root@ceph01 ~]# iscsiadm -m discovery -t sendtargets -p 172.16.0.31:3260
172.16.0.31:3260,1 iqn.2021-09.com.mwdserver:iscsimwd1
```

登录ISCSI设备

```BASH
[root@ceph01 ~]# iscsiadm -m node --login
Logging in to [iface: default, target: iqn.2021-09.com.mwdserver:iscsimwd1, portal: 172.16.0.31,3260] (multiple)
Login to [iface: default, target: iqn.2021-09.com.mwdserver:iscsimwd1, portal: 172.16.0.31,3260] successful.
```

开机挂载ISCSI

```bash
[root@ceph01 ~]# iscsiadm -m node -T iqn.2021-09.com.mwdserver:iscsimwd1 -p 172.16.0.31:3260 -o update -n  node.startup -v automatic

[root@ceph01 ~]# lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sdd               8:48   0   10G  0 disk 
sdb               8:16   0   10G  0 disk 
└─sdb1            8:17   0   10G  0 part 
sr0              11:0    1  4.4G  0 rom  
sde               8:64   0   10G  0 disk 
sdc               8:32   0   10G  0 disk 
sda               8:0    0   60G  0 disk 
├─sda2            8:2    0   59G  0 part 
│ ├─centos-swap 253:1    0    2G  0 lvm  [SWAP]
│ └─centos-root 253:0    0   57G  0 lvm  /
└─sda1            8:1    0    1G  0 part /boot

sde就是挂载的ISCSI
如果其他节点没有出来reboot一下就好了

```





## cluster的安装

```bash
[root@ceph01 ~]# yum install pcs fence-agents-all -y

[root@ceph01 ~]# rpm -qa |grep fence
fence-agents-common-4.2.1-41.el7_9.4.x86_64
fence-agents-apc-4.2.1-41.el7_9.4.x86_64
fence-agents-ipdu-4.2.1-41.el7_9.4.x86_64
fence-agents-rsa-4.2.1-41.el7_9.4.x86_64
fence-agents-eps-4.2.1-41.el7_9.4.x86_64
fence-agents-ilo-mp-4.2.1-41.el7_9.4.x86_64
fence-agents-wti-4.2.1-41.el7_9.4.x86_64
fence-agents-vmware-rest-4.2.1-41.el7_9.4.x86_64
fence-agents-mpath-4.2.1-41.el7_9.4.x86_64
fence-agents-all-4.2.1-41.el7_9.4.x86_64
fence-agents-emerson-4.2.1-41.el7_9.4.x86_64
fence-agents-ilo-ssh-4.2.1-41.el7_9.4.x86_64
fence-agents-intelmodular-4.2.1-41.el7_9.4.x86_64
fence-agents-sbd-4.2.1-41.el7_9.4.x86_64
fence-agents-ifmib-4.2.1-41.el7_9.4.x86_64
fence-agents-bladecenter-4.2.1-41.el7_9.4.x86_64
fence-agents-eaton-snmp-4.2.1-41.el7_9.4.x86_64
fence-agents-ibmblade-4.2.1-41.el7_9.4.x86_64
fence-agents-amt-ws-4.2.1-41.el7_9.4.x86_64
fence-agents-ilo-moonshot-4.2.1-41.el7_9.4.x86_64
fence-agents-hpblade-4.2.1-41.el7_9.4.x86_64
fence-agents-rsb-4.2.1-41.el7_9.4.x86_64
fence-agents-cisco-mds-4.2.1-41.el7_9.4.x86_64
fence-agents-cisco-ucs-4.2.1-41.el7_9.4.x86_64
fence-agents-apc-snmp-4.2.1-41.el7_9.4.x86_64
fence-agents-ilo2-4.2.1-41.el7_9.4.x86_64
fence-agents-vmware-soap-4.2.1-41.el7_9.4.x86_64
fence-agents-redfish-4.2.1-41.el7_9.4.x86_64
fence-virt-0.3.2-16.el7.x86_64
fence-agents-rhevm-4.2.1-41.el7_9.4.x86_64
fence-agents-ipmilan-4.2.1-41.el7_9.4.x86_64
fence-agents-kdump-4.2.1-41.el7_9.4.x86_64
fence-agents-brocade-4.2.1-41.el7_9.4.x86_64
fence-agents-scsi-4.2.1-41.el7_9.4.x86_64
fence-agents-drac5-4.2.1-41.el7_9.4.x86_64
fence-agents-heuristics-ping-4.2.1-41.el7_9.4.x86_64
fence-agents-compute-4.2.1-41.el7_9.4.x86_64

[root@ceph01 ~]# rpm -qa |grep pacemaker
pacemaker-libs-1.1.23-1.el7_9.1.x86_64
pacemaker-cli-1.1.23-1.el7_9.1.x86_64
pacemaker-cluster-libs-1.1.23-1.el7_9.1.x86_64
pacemaker-1.1.23-1.el7_9.1.x86_64
[root@ceph01 ~]# rpm -qa |grep corosync
corosync-2.4.5-7.el7_9.1.x86_64
corosynclib-2.4.5-7.el7_9.1.x86_64
```

> 如果需要使用Cluster LVM和GFS， yum install lvm2-cluster gfs2-utils

### 设置密码

所有节点设置

密码这里是看不到的，我设置的是admin

```bash
[root@ceph01 ~]# passwd hacluster 
Changing password for user hacluster.
New password: ==admin==
BAD PASSWORD: The password is shorter than 8 characters
Retype new password: ==admin==
passwd: all authentication tokens updated successfully.
[root@ceph01 ~]# 
```

### 启动服务

all node

```bash
[root@ceph01 ~]# systemctl start pcsd.service;systemctl enable pcsd.service 
```

节点之间互信

```bash
[root@ceph01 ~]# pcs cluster auth ceph01 ceph02 ceph03
Username: hacluster
Password: ==admin==
ceph02: Authorized
ceph03: Authorized
ceph01: Authorized
```

```bash
[root@ceph01 ~]# netstat -lntup
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1114/sshd           
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      1202/master         
tcp6       0      0 :::2224                 :::*                    LISTEN      2078/ruby           
tcp6       0      0 :::22                   :::*                    LISTEN      1114/sshd           
tcp6       0      0 ::1:25                  :::*                    LISTEN      1202/master         
```

https://172.16.0.31:2224/login

![image-20210908232938235](corosync_pacemaker.assets\image-20210908232938235.png)

![image-20210908233013112](corosync_pacemaker.assets\image-20210908233013112.png)

### 创建集群

![image-20210910164408304](corosync_pacemaker.assets\image-20210910164408304.png)

这步操作完成会生成corosync.conf配置文件，这份文件需要保存好

![image-20210908233401798](corosync_pacemaker.assets\image-20210908233401798.png)

这里使用相同密码

![image-20210908233456841](corosync_pacemaker.assets\image-20210908233456841.png)



![image-20210908233525540](corosync_pacemaker.assets\image-20210908233525540.png)

### 配置文件corosync

我们看下corosync.conf文件

```BASH
[root@ceph01 ~]# cat /etc/corosync/corosync.conf
totem {
    version: 2
    cluster_name: cluster_rbd
    secauth: off
    transport: udpu
    rrp_mode: passive
}

nodelist {
    node {
        ring0_addr: ceph01
        ring1_addr: 192.168.10.31
        nodeid: 1
    }

    node {
        ring0_addr: ceph02
        ring1_addr: 192.168.10.32
        nodeid: 2
    }

    node {
        ring0_addr: ceph03
        ring1_addr: 192.168.10.33
        nodeid: 3
    }
}

quorum {
    provider: corosync_votequorum
}

logging {
    to_logfile: yes
    logfile: /var/log/cluster/corosync.log
    to_syslog: yes
}

```

![image-20210908233820367](corosync_pacemaker.assets\image-20210908233820367.png)

按图操作

CLI操作

```bash
[root@ceph01 ~]# pcs cluster start --all
ceph01: Starting Cluster (corosync)...
ceph02: Starting Cluster (corosync)...
ceph03: Starting Cluster (corosync)...
ceph03: Starting Cluster (pacemaker)...
ceph02: Starting Cluster (pacemaker)...
ceph01: Starting Cluster (pacemaker)...
[root@ceph01 ~]# pcs cluster enable --all
ceph01: Cluster Enabled
ceph02: Cluster Enabled
ceph03: Cluster Enabled
```

判断集群是否合法

> 总节点数  <  2 * 节点存活数
>
> 4节点    4<2*4
>
> 3节点    4<2*3
>
> 2节点    4<2*2   不成立这时候随便挂一台就故障了。

按照上面的计算2节点是不可用的，但是很多情况只能2节点。这个时候就需要修改计算机制了

两节点修改方法：

> pcs property set no-quorum-policy=ignore  #忽略quorum选票
>
> pcs resource defaults migration-threshold=1    #同时开启集群故障时的迁移服务

集群两台默认已经设置了上面的参数，如果是宕机之后只有两台才需要手动设置

```bash
[root@ceph01 ~]# pcs cluster status
Cluster Status:
 Stack: corosync
 Current DC: ceph02 (version 1.1.23-1.el7_9.1-9acf116022) - partition with quorum
 Last updated: Thu Sep  9 11:33:13 2021
 Last change: Wed Sep  8 23:34:30 2021 by hacluster via crmd on ceph03
 3 nodes configured
 0 resource instances configured

PCSD Status:
  ceph02: Online
  ceph03: Online
  ceph01: Online
```

在命令行检查配置文件

```bash
[root@ceph01 ~]# crm_verify -L -V
   error: unpack_resources:	Resource start-up disabled since no STONITH resources have been defined
   error: unpack_resources:	Either configure some or disable STONITH with the stonith-enabled option
   error: unpack_resources:	NOTE: Clusters with shared data need STONITH to ensure data integrity
Errors found during check: config not valid

```

正常的是没有显示。有问题就像上面这样

这个错误需要关闭stonith

```BASH
[root@ceph01 ~]# pcs property set stonith-enabled=false  #命令关闭
```

GUI关闭

![image-20210909120216165](corosync_pacemaker.assets\image-20210909120216165.png)

```BASH
[root@ceph01 ~]# crm_verify -L -V
[root@ceph01 ~]# 
```

这个就是正常的

## 设置VIP

添加resource(VIP)

![image-20210909144618617](corosync_pacemaker.assets\image-20210909144618617.png)

VIP就是keepalived里面的VIP。

命令创建

```BASH
[root@ceph01 ~]# pcs resource create VIP ocf:heartbeat:IPaddr2 ip=172.16.0.30 cidr_netmask=24 op monitor interval=30s
```

## 添加resource

LVM这个适应ISCSI 不适合生产环境不使用

### 添加filesystem



![image-20210910165643798](corosync_pacemaker.assets\image-20210910165643798.png)



![image-20210910165817306](corosync_pacemaker.assets\image-20210910165817306.png)

这里报警了。删了，因为ceph02 ceph03节点没有挂载rbd1

```bash
[root@ceph01 ~]# scp /usr/local/bin/rbd-mount ceph02:/usr/local/bin/

[root@ceph01 ~]# scp /usr/local/bin/rbd-mount ceph03:/usr/local/bin/
 
[root@ceph01 ~]# scp /etc/systemd/system/rbd-mount.service ceph02:/etc/systemd/system/

[root@ceph01 ~]# scp /etc/systemd/system/rbd-mount.service ceph03:/etc/systemd/system/

#ceph02 ceph03 都操作下面的
[root@ceph02 ~]# systemctl enable rbd-mount.service 

[root@ceph02 ~]# rbd-mount m
/dev/rbd0

[root@ceph03 ~]# df -h /mnt/rbd
Filesystem      Size  Used Avail Use% Mounted on
/dev/rbd0        30G   33M   30G   1% /mnt/rbd

```

![image-20210910204305547](corosync_pacemaker.assets\image-20210910204305547.png)

## 创建service group

![image-20210910212157531](corosync_pacemaker.assets\image-20210910212157531.png)

```bash
[root@ceph01 ~]# pcs resource group  add testvip VIP_rbd
[root@ceph03 ~]# pcs status
Cluster name: cluster_rbd
Stack: corosync
Current DC: ceph01 (version 1.1.23-1.el7_9.1-9acf116022) - partition with quorum
Last updated: Fri Sep 10 21:23:11 2021
Last change: Fri Sep 10 21:21:54 2021 by hacluster via cibadmin on ceph01

3 nodes configured
2 resource instances configured

Online: [ ceph01 ceph02 ceph03 ]

Full list of resources:

 rbd1	(ocf::heartbeat:Filesystem):	Started ceph02
 Resource Group: testvip
     VIP_rbd	(ocf::heartbeat:IPaddr2):	Started ceph01

Daemon Status:
  corosync: active/enabled
  pacemaker: active/enabled
  pcsd: active/enabled
```

## 故障模拟和恢复

断开VIP，模拟集群切换

```bash
[root@ceph01 ~]# ip a s eth0
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:ed:17:97 brd ff:ff:ff:ff:ff:ff
    inet 172.16.0.31/24 brd 172.16.0.255 scope global noprefixroute eth0
       valid_lft forever preferred_lft forever
    inet 172.16.0.30/24 brd 172.16.0.255 scope global secondary eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::28be:ce7a:20d6:66d/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```

参数

```BASH
[root@ceph01 ~]# pcs resource meta VIP_rbd failure-timeout=30 
# 失效30秒后切换资源
```

![image-20210910215335632](corosync_pacemaker.assets\image-20210910215335632.png)

设置failure-timeout失效多少秒后切回资源到失效的主机

```BASH
[root@ceph01 ~]# pcs resource meta testvip failure-timeout=30
```



```bash
[root@ceph01 ~]# pcs resource meta rbd1 failure-timeout=30
```

![image-20210910220047762](corosync_pacemaker.assets\image-20210910220047762.png)

设置切换时间





### 如何解决回切问题

（主机恢复后，VIP回归原主机）这个在生产中不能使用，来回切换服务会中断。

```BASH
[root@ceph01 ~]# pcs resource defaults resource-stickiness=100
Warning: Defaults do not apply to resources which override them with their own defined values
[root@ceph01 ~]# pcs resource defaults
resource-stickiness=100
```

resource-stickiness表示资源对主机的黏贴性。

配置资源优先级

```BASH
[root@ceph01 ~]# pcs constraint location testvip prefers ceph01=200
```

![image-20210910224821431](corosync_pacemaker.assets\image-20210910224821431.png)

查看资源的优先级

```BASH
[root@ceph01 ~]# crm_simulate  -sL

Current cluster status:
Online: [ ceph01 ceph02 ceph03 ]

 rbd1	(ocf::heartbeat:Filesystem):	Started ceph02
 Resource Group: testvip
     VIP_rbd	(ocf::heartbeat:IPaddr2):	Started ceph01

Allocation scores:
pcmk__native_allocate: rbd1 allocation score on ceph01: 0
pcmk__native_allocate: rbd1 allocation score on ceph02: 100
pcmk__native_allocate: rbd1 allocation score on ceph03: 0
pcmk__group_allocate: testvip allocation score on ceph01: 200
pcmk__group_allocate: testvip allocation score on ceph02: 0
pcmk__group_allocate: testvip allocation score on ceph03: 0
pcmk__group_allocate: VIP_rbd allocation score on ceph01: 300
pcmk__group_allocate: VIP_rbd allocation score on ceph02: 0
pcmk__group_allocate: VIP_rbd allocation score on ceph03: 0
pcmk__native_allocate: VIP_rbd allocation score on ceph01: 300
pcmk__native_allocate: VIP_rbd allocation score on ceph02: 0
pcmk__native_allocate: VIP_rbd allocation score on ceph03: 0

Transition Summary:
[root@ceph01 ~]# pcs config show
Cluster Name: cluster_rbd
Corosync Nodes:
 ceph01 ceph02 ceph03
Pacemaker Nodes:
 ceph01 ceph02 ceph03

Resources:
 Resource: rbd1 (class=ocf provider=heartbeat type=Filesystem)
  Attributes: device=/dev/rbd0 directory=/mnt/rbd fstype=xfs
  Meta Attrs: failure-timeout=30
  Operations: monitor interval=20s timeout=40s (rbd1-monitor-interval-20s)
              notify interval=0s timeout=60s (rbd1-notify-interval-0s)
              start interval=0s timeout=60s (rbd1-start-interval-0s)
              stop interval=0s timeout=60s (rbd1-stop-interval-0s)
 Group: testvip
  Meta Attrs: failure-timeout=30
  Resource: VIP_rbd (class=ocf provider=heartbeat type=IPaddr2)
   Attributes: ip=172.16.0.30
   Meta Attrs: failure-timeout=30
   Operations: monitor interval=10s timeout=20s (VIP_rbd-monitor-interval-10s)
               start interval=0s timeout=20s (VIP_rbd-start-interval-0s)
               stop interval=0s timeout=20s (VIP_rbd-stop-interval-0s)

Stonith Devices:
Fencing Levels:

Location Constraints:
  Resource: testvip
    Enabled on: ceph01 (score:200) (id:location-testvip-ceph01-200)
Ordering Constraints:
Colocation Constraints:
Ticket Constraints:

Alerts:
 No alerts defined

Resources Defaults:
 resource-stickiness=100
Operations Defaults:
 No defaults set

Cluster Properties:
 cluster-infrastructure: corosync
 cluster-name: cluster_rbd
 dc-version: 1.1.23-1.el7_9.1-9acf116022
 have-watchdog: false
 stonith-enabled: false

Quorum:
  Options:

```

### 断开心跳测试（脑裂的防范）

```BASH
[root@ceph01 ~]# ifconfig eth1 down

[root@ceph01 ~]# ifdown eth1

[root@ceph01 ~]# ip link set eth1 down
[root@ceph01 ~]# nmcli con  Down/Up
[root@ceph01 ~]# nmtui
```

断开心跳线

```BASH
[root@ceph01 ~]# ifup eth1   #激活网卡
[root@ceph01 ~]# ifconfig eth1 up
[root@ceph01 ~]# ip link set eth1 up
```

![image-20210910231153357](corosync_pacemaker.assets\image-20210910231153357.png)

```BASH
[root@ceph02 ~]# pcs status
Cluster name: cluster_rbd
Stack: corosync
Current DC: ceph02 (version 1.1.23-1.el7_9.1-9acf116022) - partition with quorum
Last updated: Fri Sep 10 23:10:54 2021
Last change: Fri Sep 10 22:53:05 2021 by hacluster via cibadmin on ceph01

3 nodes configured
2 resource instances configured

Online: [ ceph02 ceph03 ]
OFFLINE: [ ceph01 ]

Full list of resources:

 rbd1	(ocf::heartbeat:Filesystem):	Started ceph02
 Resource Group: testvip
     VIP_rbd	(ocf::heartbeat:IPaddr2):	Started ceph03

Daemon Status:
  corosync: active/enabled
  pacemaker: active/enabled
  pcsd: active/enabled
```

脑裂就是像上面的这种情况，心跳线停止了，但是两边有检测到自己存活就变成两个集群了。

双心跳线是解决方案，上面我们已经使用了双心跳线。

```BASH
pcs cluster stop -all
pcs cluster start -all
```

### stonith设置（Fence设置）

fence是集群中某个节点故障使其关机，重启及卸载集群

1.打开fence设备的设置

```BASH
[root@ceph01 ~]# pcs property set stonith-enabled=true
```

![image-20210910233729074](corosync_pacemaker.assets\image-20210910233729074.png)

2.查看本系统支持的fence

```bash
[root@ceph01 ~]# pcs stonith list
fence_amt_ws - Fence agent for AMT (WS)
fence_apc - Fence agent for APC over telnet/ssh
fence_apc_snmp - Fence agent for APC, Tripplite PDU over SNMP
fence_bladecenter - Fence agent for IBM BladeCenter
fence_brocade - Fence agent for HP Brocade over telnet/ssh
fence_cisco_mds - Fence agent for Cisco MDS
fence_cisco_ucs - Fence agent for Cisco UCS
fence_compute - Fence agent for the automatic resurrection of OpenStack compute instances
fence_drac5 - Fence agent for Dell DRAC CMC/5
fence_eaton_snmp - Fence agent for Eaton over SNMP
fence_emerson - Fence agent for Emerson over SNMP
fence_eps - Fence agent for ePowerSwitch
fence_evacuate - Fence agent for the automatic resurrection of OpenStack compute instances
fence_heuristics_ping - Fence agent for ping-heuristic based fencing
fence_hpblade - Fence agent for HP BladeSystem
fence_ibmblade - Fence agent for IBM BladeCenter over SNMP
fence_idrac - Fence agent for IPMI
fence_ifmib - Fence agent for IF MIB
fence_ilo - Fence agent for HP iLO
fence_ilo2 - Fence agent for HP iLO
fence_ilo3 - Fence agent for IPMI
fence_ilo3_ssh - Fence agent for HP iLO over SSH
fence_ilo4 - Fence agent for IPMI
fence_ilo4_ssh - Fence agent for HP iLO over SSH
fence_ilo5 - Fence agent for IPMI
fence_ilo5_ssh - Fence agent for HP iLO over SSH
fence_ilo_moonshot - Fence agent for HP Moonshot iLO
fence_ilo_mp - Fence agent for HP iLO MP
fence_ilo_ssh - Fence agent for HP iLO over SSH
fence_imm - Fence agent for IPMI
fence_intelmodular - Fence agent for Intel Modular
fence_ipdu - Fence agent for iPDU over SNMP
fence_ipmilan - Fence agent for IPMI
fence_kdump - fencing agent for use with kdump crash recovery service
fence_mpath - Fence agent for multipath persistent reservation
fence_redfish - I/O Fencing agent for Redfish
fence_rhevm - Fence agent for RHEV-M REST API
fence_rsa - Fence agent for IBM RSA
fence_rsb - I/O Fencing agent for Fujitsu-Siemens RSB
fence_sbd - Fence agent for sbd
fence_scsi - Fence agent for SCSI persistent reservation
fence_virt - Fence agent for virtual machines
fence_vmware_rest - Fence agent for VMware REST API
fence_vmware_soap - Fence agent for VMWare over SOAP API
fence_wti - Fence agent for WTI
fence_xvm - Fence agent for virtual machines
```

3.查看即将使用的fence设备的相关信息

```BASH
[root@ceph01 ~]# pcs stonith describe fence_ipmilan
fence_ipmilan - Fence agent for IPMI

fence_ipmilan is an I/O Fencing agentwhich can be used with machines controlled by IPMI.This agent calls support software ipmitool (http://ipmitool.sf.net/). WARNING! This fence agent might report success before the node is powered off. You should use -m/method onoff if your fence device works correctly with that option.

Stonith options:
  auth: IPMI Lan Auth type.
  cipher: Ciphersuite to use (same as ipmitool -C parameter)
  hexadecimal_kg: Hexadecimal-encoded Kg key for IPMIv2 authentication
  ipaddr: IP address or hostname of fencing device
  ipport: TCP/UDP port to use for connection with device
  lanplus: Use Lanplus to improve security of connection
  login: Login name
  method: Method to fence
  passwd: Login password or passphrase
  passwd_script: Script to run to retrieve password
  port: IP address or hostname of fencing device (together with --port-as-ip)
  privlvl: Privilege level on IPMI device
  target: Bridge IPMI requests to the remote target address
  quiet: Disable logging to stderr. Does not affect --verbose or --debug-file or logging to syslog.
  verbose: Verbose mode
  debug: Write debug information to given file
  delay: Wait X seconds before fencing is started
  ipmitool_path: Path to ipmitool binary
  login_timeout: Wait X seconds for cmd prompt after login
  port_as_ip: Make "port/plug" to be an alias to IP address
  power_timeout: Test X seconds for status change after ON/OFF
  power_wait: Wait X seconds after issuing ON/OFF
  shell_timeout: Wait X seconds for cmd prompt after issuing command
  retry_on: Count of attempts to retry power on
  sudo: Use sudo (without password) when calling 3rd party software
  sudo_path: Path to sudo binary
  pcmk_host_map: A mapping of host names to ports numbers for devices that do not support host names. Eg.
                 node1:1;node2:2,3 would tell the cluster to use port 1 for node1 and ports 2 and 3 for node2
  pcmk_host_list: A list of machines controlled by this device (Optional unless pcmk_host_check=static-list).
  pcmk_host_check: How to determine which machines are controlled by the device. Allowed values: dynamic-list
                   (query the device via the 'list' command), static-list (check the pcmk_host_list
                   attribute), status (query the device via the 'status' command), none (assume every device
                   can fence every machine)
  pcmk_delay_max: Enable a random delay for stonith actions and specify the maximum of random delay. This
                  prevents double fencing when using slow devices such as sbd. Use this to enable a random
                  delay for stonith actions. The overall delay is derived from this random delay value adding
                  a static delay so that the sum is kept below the maximum delay.
  pcmk_delay_base: Enable a base delay for stonith actions and specify base delay value. This prevents double
                   fencing when different delays are configured on the nodes. Use this to enable a static
                   delay for stonith actions. The overall delay is derived from a random delay value adding
                   this static delay so that the sum is kept below the maximum delay.
  pcmk_action_limit: The maximum number of actions can be performed in parallel on this device Pengine
                     property concurrent-fencing=true needs to be configured first. Then use this to specify
                     the maximum number of actions can be performed in parallel on this device. -1 is
                     unlimited.

Default operations:
  monitor: interval=60s

```

`实际环境中fence_impmilan必须有lanplus="true"`

4.生产环境中的初始配置文件

```bash
[root@ceph01 ~]# pcs cluster cib stonith_cfg
```

5.为节点配置fence

```bash
[root@ceph01 ~]# pcs -f stonith_cfg stonith create ipmi-fence-ceph01 fence_ipmilan lanplus="true" pcmk_host_list="ceph01" pcmk_host_check="static-list" action="reboot" ipaddr="192.168.20.1" login=userid passwd=password op monitor interval=60s
```

这里不是物理服务器上面信息根据实际情况设置，每个节点都需要操作

ipmi-fence-ceph01  这个是创建的fence设备名称

pcmk_host_check="static-list"将节点与IP地址对应

6.检查stonith_cfg中的stonith的配置信息

```BASH
[root@ceph01 ~]# pcs -f stonith_cfg stonith
```

7.上文关闭了stonish,现在开启stonith

```BASH
[root@ceph01 ~]# pcs -f stonith_cfg property set stonith-enabled=true
```

8.检查stonith_cfg中stonith是否已经开启

```BASH
[root@ceph01 ~]# pcs -f stonith_cfg property
Cluster Properties:
 cluster-infrastructure: corosync
 cluster-name: cluster_rbd
 dc-version: 1.1.23-1.el7_9.1-9acf116022
 have-watchdog: false
 stonith-enabled: true
```

9.将stonith_cfg写入cib.xml

```BASH
[root@ceph01 ~]# pcs cluster cib-push stonith_cfg 
CIB updated
```

10.测试Fence

```BASH
stonith_admin --reboot ceph01
stonith_admin --reboot ceph02
```

如果主机重启了表示完成成功。

web 设置

![image-20210911000740246](corosync_pacemaker.assets\image-20210911000740246.png)

### 备份恢复集群

```BASH
[root@ceph01 ~]# pcs config backup backfile1
```

上面是备份当前集群的文件

恢复集群

--local选项只会恢复当前节点中的文件

```BASH
[root@ceph01 ~]# pcs config restore --local backfile1
```



# DRBD

解决方案：https://linbit.com/solution-and-product-documentation/ （参考）

文档：https://linbit.com/drbd-user-guide/

云计算：https://github.com/LINBIT/linstor-server （用于容器、云和虚拟化的高性能软件定义块存储。与 Docker、Kubernetes、Openstack、Proxmox 等完全集成。）

VMware SDS :https://linbit.com/linbit-vsan-software-defined-storage-for-vmware%e2%80%8b/?_ga=2.247458372.1366182388.1631324552-1241064990.1631324552

