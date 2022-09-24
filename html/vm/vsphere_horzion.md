vsphere



# IP规划表

| 名称       | IP            | 域名       | 服务               |
| ---------- | ------------- | ---------- | ------------------ |
| **ESXI01** | 192.168.10.10 | cloudb.com | esxi               |
| **ESXI02** | 192.168.10.11 | cloudb.com | esxi               |
| **ESXI03** | 192.168.10.12 | cloudb.com | esxi               |
| VCSA       | 192.168.10.13 | cloudb.com | vcenter            |
| **DC**     | 192.168.10.9  | cloudb.com | ad,dns,dhcp,ca,sql |
| CONNETION  | 192.168.10.8  | cloudb.com |                    |
| UAG        | 192.168.10.7  | cloudb.com |                    |
| **NFS**    | 192.168.10.5  | -          | nfs                |
|            |               |            |                    |



---

> windows hosts配置
>
> 192.168.10.10 esxi01.cloudb.com
> 192.168.10.11 esxi02.cloudb.com
> 192.168.10.12 esxi03.cloudb.com
> 192.168.10.13 vcsa.cloudb.com
>
> 192.168.10.9 dc.cloudb.com
> 192.168.10.8 connetion.cloudb.com
> 192.168.10.7 uag.cloudb.com

## AD

默认不配置的不做截图！先部署ad再一一部署其他服务

![image-20211011154141636](vsphere.assets/image-20211011154141636.png)

![image-20211011154249720](vsphere.assets/image-20211011154249720.png)

![image-20211011154317387](vsphere.assets/image-20211011154317387.png)

等待重启

![image-20211011155451350](vsphere.assets/image-20211011155451350.png)

### 配置DHCP

![image-20211011161931727](vsphere.assets/image-20211011161931727.png)

![image-20211011162036775](vsphere.assets/image-20211011162036775.png)

![image-20211011162207402](vsphere.assets/image-20211011162207402.png)

![image-20211011162233746](vsphere.assets/image-20211011162233746.png)

![image-20211011162740607](vsphere.assets/image-20211011162740607.png)

![image-20211011162648108](vsphere.assets/image-20211011162648108.png)

### DNS

![image-20211011161657115](vsphere.assets/image-20211011161657115.png)

![image-20211011162922038](vsphere.assets/image-20211011162922038.png)

![image-20211012230203638](vsphere.assets/image-20211012230203638.png)

![image-20211014185525510](vsphere.assets/image-20211014185525510.png)

如果在vcsa第二阶段卡住0%很长时间是因为没有设置DNS记录！



### CA

![image-20211011162946704](vsphere.assets/image-20211011162946704.png)



![image-20211011163139489](vsphere.assets/image-20211011163139489.png)

![image-20211011163148491](vsphere.assets/image-20211011163148491.png)

#### 申请证书

![image-20211011163259794](vsphere.assets/image-20211011163259794.png)



![image-20211011163412822](vsphere.assets/image-20211011163412822.png)



### ISCSI准备

![image-20211011164219814](vsphere.assets/image-20211011164219814.png)

![image-20211011163748952](vsphere.assets/image-20211011163748952.png)

![image-20211011163836385](vsphere.assets/image-20211011163836385.png)



![image-20211011164317974](vsphere.assets/image-20211011164317974.png)

![image-20211011164338825](vsphere.assets/image-20211011164338825.png)

![image-20211011164414919](vsphere.assets/image-20211011164414919.png)

![image-20211011164428989](vsphere.assets/image-20211011164428989.png)

![image-20211011164503116](vsphere.assets/image-20211011164503116.png)

![image-20211011164548475](vsphere.assets/image-20211011164548475.png)

![image-20211011164628308](vsphere.assets/image-20211011164628308.png)

![image-20211011164754849](vsphere.assets/image-20211011164754849.png)

![image-20211011164942769](vsphere.assets/image-20211011164942769.png)



==windows加入ad 之后提示：c:\windowssystem32rundll32.exe windows无法访问指定设备、路径或文件==

> 域控管理员账号登录Windows Server 2016服务器，鼠标点击声音、图标等设置报错 rundll32.exe Windows无法访问指定设备、路径或文件。

![image-20211011194432492](vsphere.assets/image-20211011194432492.png)

![image-20211011194510632](vsphere.assets/image-20211011194510632.png)

![image-20211011194539387](vsphere.assets/image-20211011194539387.png)

![image-20211011194555152](vsphere.assets/image-20211011194555152.png)



![image-20211011194707400](vsphere.assets/image-20211011194707400.png)

![image-20211011194729436](vsphere.assets/image-20211011194729436.png)



![image-20211011194816574](vsphere.assets/image-20211011194816574.png)

上面的设置只能本计算机

![image-20211011195538603](vsphere.assets/image-20211011195538603.png)

设置完成 gpupdate 一下即可（没有起作用，重新启动才可以）



## 存储

这里的存储使用ceph 或者glusterfs是最好的。但是实验就使用NFS了。

```sh
[root@localhost ~]# lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda               8:0    0   40G  0 disk 
├─sda1            8:1    0    1G  0 part /boot
└─sda2            8:2    0   39G  0 part 
  ├─centos-root 253:0    0 35.1G  0 lvm  /
  └─centos-swap 253:1    0  3.9G  0 lvm  [SWAP]
sdb               8:16   0  600G  0 disk 
└─sdb1            8:17   0  600G  0 part 
sdc               8:32   0  350G  0 disk 
└─sdc1            8:33   0  350G  0 part 
sr0              11:0    1 1024M  0 rom  
[root@localhost ~]# mkfs.xfs /dev/sdb1
meta-data=/dev/sdb1              isize=512    agcount=4, agsize=39321536 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=157286144, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=76799, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
[root@localhost ~]# mkfs.xfs /dev/sdc1
meta-data=/dev/sdc1              isize=512    agcount=4, agsize=22937536 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=91750144, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=44799, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0

[root@localhost ~]# mkdir -p /data/{hdd,ssd}

[root@localhost ~]# mount /dev/sdb1 /data/hdd/
[root@localhost ~]# df -h
Filesystem               Size  Used Avail Use% Mounted on
devtmpfs                 1.9G     0  1.9G   0% /dev
tmpfs                    1.9G     0  1.9G   0% /dev/shm
tmpfs                    1.9G   12M  1.9G   1% /run
tmpfs                    1.9G     0  1.9G   0% /sys/fs/cgroup
/dev/mapper/centos-root   36G  1.8G   34G   5% /
/dev/sda1               1014M  151M  864M  15% /boot
tmpfs                    378M     0  378M   0% /run/user/0
/dev/sdb1                600G   33M  600G   1% /data/hdd

[root@localhost ~]# vim /usr/local/bin/mount
...机密不能发布到web上面,完整版参考笔记

[root@localhost ~]# systemctl enable rbd-mount
reboot之后再次查看NFS是否挂载

[root@localhost ~]# df -h
Filesystem               Size  Used Avail Use% Mounted on
devtmpfs                 1.9G     0  1.9G   0% /dev
tmpfs                    1.9G     0  1.9G   0% /dev/shm
tmpfs                    1.9G   12M  1.9G   1% /run
tmpfs                    1.9G     0  1.9G   0% /sys/fs/cgroup
/dev/mapper/centos-root   36G  1.8G   34G   5% /
/dev/sda1               1014M  151M  864M  15% /boot
/dev/sdb1                600G   33M  600G   1% /data/hdd
/dev/sdc1                350G   33M  350G   1% /data/ssd
tmpfs                    378M     0  378M   0% /run/user/0


```

NFS

```sh
[root@localhost ~]# yum install nfs-utils rpcbind -y
[root@localhost ~]# systemctl start rpcbind.service ;systemctl start nfs
[root@localhost ~]# systemctl enable rpcbind.service ;systemctl enable nfs
[root@localhost ~]# vim /etc/exports
/data/hdd 192.168.10.5(rw,sync,all_squash) 
/data/ssd 192.168.10.5(rw,sync,all_squash) 
[root@localhost ~]# chown -R nfsnobody:nfsnobody /data/
[root@nfs /]# ll /data/
total 0
drwxr-xr-x 2 nfsnobody nfsnobody 6 Oct 15 08:17 hdd
drwxr-xr-x 2 nfsnobody nfsnobody 6 Oct 15 08:17 ssd

[root@localhost ~]# exportfs -rv
exporting 192.168.10.0/24:/data/ssd
exporting 192.168.10.0/24:/data/hdd
[root@localhost ~]# showmount -e 192.168.10.5
Export list for 192.168.10.5:
/data/ssd 192.168.10.0/24
/data/hdd 192.168.10.0/24

测试挂载
[root@localhost ~]# mount -t nfs 192.168.10.5:/data/hdd /mnt

```

![image-20211011231906548](vsphere.assets/image-20211011231906548.png)

NFS 服务器注意防火墙和selinux





### esxi 挂载一下看看

![image-20211011232038597](vsphere.assets/image-20211011232038597.png)

```sh
[root@esxi01:~] esxcli storage nfs list
Volume Name  Host          Share      Accessible  Mounted  Read-Only   isPE  Hardware Acceleration
-----------  ------------  ---------  ----------  -------  ---------  -----  ---------------------
ssd          192.168.10.5  /data/ssd        true     true      false  false  Not Supported
hdd          192.168.10.5  /data/hdd        true     true      false  false  Not Supported
[root@esxi01:~] df -h
Filesystem   Size   Used Available Use% Mounted on
NFS        349.8G  32.2M    349.8G   0% /vmfs/volumes/ssd
NFS        599.7G  32.2M    599.7G   0% /vmfs/volumes/hdd
VMFS-L      41.8G   2.9G     38.9G   7% /vmfs/volumes/OSDATA-6163df77-cdfdda28-2e5c-000c29acd0b5
vfat         4.0G 198.8M      3.8G   5% /vmfs/volumes/BOOTBANK1
vfat         4.0G  64.0K      4.0G   0% /vmfs/volumes/BOOTBANK2

```

三台ESXI都一样的设置

![image-20211011232508089](vsphere.assets/image-20211011232508089.png)

## VCSA

![image-20211011233017227](vsphere.assets/image-20211011233017227.png)

![image-20211011233129063](vsphere.assets/image-20211011233129063.png)

注意！DNS要设置一个可以上网的，不然部署很慢

![image-20211011233236059](vsphere.assets/image-20211011233236059.png)

等待第一阶段完成

![image-20211011233407750](vsphere.assets/image-20211011233407750.png)

测试硬盘吞吐量

```sh
[root@localhost ~]# yum -y install hdparm
[root@localhost ~]# hdparm -Tt --direct /dev/sdb1

/dev/sdb1:
 Timing O_DIRECT cached reads:   2912 MB in  2.00 seconds = 1456.38 MB/sec
 Timing O_DIRECT disk reads: 2560 MB in  3.00 seconds = 852.93 MB/sec
[root@localhost ~]# hdparm -Tt --direct /dev/sdc1

/dev/sdc1:
 Timing O_DIRECT cached reads:   5034 MB in  2.00 seconds = 2519.95 MB/sec
 Timing O_DIRECT disk reads: 8870 MB in  3.00 seconds = 2956.61 MB/sec

```

```sh
-t 评估硬盘的读取效率。
-T 评估硬盘快取的读取效率。
```

存储消耗内存，cpu基本不消化多少

![image-20211011234622900](vsphere.assets/image-20211011234622900.png)

![image-20211011234659904](vsphere.assets/image-20211011234659904.png)

![image-20211011235917959](vsphere.assets/image-20211011235917959.png)

**第二阶段**

![image-20211012000306410](vsphere.assets/image-20211012000306410.png)

![image-20211012000324159](vsphere.assets/image-20211012000324159.png)

![image-20211012000409839](vsphere.assets/image-20211012000409839.png)

![image-20211012153356915](vsphere.assets/image-20211012153356915.png)

### 配置VCSA

![image-20211012230836008](vsphere.assets/image-20211012230836008.png)



创建集群

![image-20211012232124671](vsphere.assets/image-20211012232124671.png)

添加主机

![image-20211012231812948](vsphere.assets/image-20211012231812948.png)

![image-20211012231838938](vsphere.assets/image-20211012231838938.png)

![image-20211012232328724](vsphere.assets/image-20211012232328724.png)

![image-20211012233806008](vsphere.assets/image-20211012233806008.png)

集群已经配置好





## UAG

![image-20211014085740266](vsphere.assets/image-20211014085740266.png)



![image-20211014085833397](vsphere.assets/image-20211014085833397.png)

![image-20211014085901446](vsphere.assets/image-20211014085901446.png)



![image-20211014085911156](vsphere.assets/image-20211014085911156.png)

![image-20211014090003565](vsphere.assets/image-20211014090003565.png)

![image-20211014090024732](vsphere.assets/image-20211014090024732.png)

![image-20211014090050332](vsphere.assets/image-20211014090050332.png)

![image-20211014090113409](vsphere.assets/image-20211014090113409.png)

![image-20211014091132540](vsphere.assets/image-20211014091132540.png)

![image-20211014091241723](vsphere.assets/image-20211014091241723.png)



![image-20211014091306666](vsphere.assets/image-20211014091306666.png)

![image-20211014091335933](vsphere.assets/image-20211014091335933.png)



![image-20211014091352450](vsphere.assets/image-20211014091352450.png)

打开UAG 电源

https://192.168.10.7:9443/

![image-20211014100517188](vsphere.assets/image-20211014100517188.png)

![image-20211014100630099](vsphere.assets/image-20211014100630099.png)

![image-20211014100749874](vsphere.assets/image-20211014100749874.png)

![image-20211014100814302](vsphere.assets/image-20211014100814302.png)

![image-20211014101333800](vsphere.assets/image-20211014101333800.png)

在CA根服务器上操作

![image-20211014101402948](vsphere.assets/image-20211014101402948.png)

![image-20211014101515849](vsphere.assets/image-20211014101515849.png)

![image-20211014101733912](vsphere.assets/image-20211014101733912.png)

打开连接服务地址

![image-20211014102152200](vsphere.assets/image-20211014102152200.png)



![image-20211014102221525](vsphere.assets/image-20211014102221525.png)

![image-20211014102237568](vsphere.assets/image-20211014102237568.png)

e75c776850532eff0e7fdfb1683445183ac56314

![image-20211014102330275](vsphere.assets/image-20211014102330275.png)

![image-20211014102356376](vsphere.assets/image-20211014102356376.png)



![image-20211014102903465](vsphere.assets/image-20211014102903465.png)

![image-20211014103057235](vsphere.assets/image-20211014103057235.png)



![image-20211014130723426](vsphere.assets/image-20211014130723426.png)



![image-20211014104844084](vsphere.assets/image-20211014104844084.png)

这里稍微等一下，不行的话开启下UDP再关闭就激活了。

![image-20211014130759146](vsphere.assets/image-20211014130759146.png)













## windows10模板

模板制作基本一致这里使用w10

![image-20211013225758189](vsphere.assets/image-20211013225758189.png)

模板常用软件制作好之后使用sysprep 重新打包避免SID重复不能加入AD的问题。打包好一定要关机

![image-20211013230148467](vsphere.assets/image-20211013230148467.png)





## 安装connetion

先配置证书

![image-20211013235256589](vsphere.assets/image-20211013235256589.png)



![image-20211013235318171](vsphere.assets/image-20211013235318171.png)

![image-20211013235420426](vsphere.assets/image-20211013235420426.png)

![image-20211013235635929](vsphere.assets/image-20211013235635929.png)

![image-20211013235737195](vsphere.assets/image-20211013235737195.png)

![image-20211014000615043](vsphere.assets/image-20211014000615043.png)

1G2NK-AF3DP-M84U1-02CZ4-1LHP8

![image-20211014000903795](vsphere.assets/image-20211014000903795.png)

![image-20211014001002626](vsphere.assets/image-20211014001002626.png)

![image-20211014001142260](vsphere.assets/image-20211014001142260.png)

![image-20211014001231806](vsphere.assets/image-20211014001231806.png)

![image-20211014001240956](vsphere.assets/image-20211014001240956.png)

![image-20211014001345136](vsphere.assets/image-20211014001345136.png)

![image-20211014001358856](vsphere.assets/image-20211014001358856.png)



## SQL

![image-20211014002032461](vsphere.assets/image-20211014002032461.png)

![image-20211014002123515](vsphere.assets/image-20211014002123515.png)

![image-20211014002210426](vsphere.assets/image-20211014002210426.png)

![image-20211014002249163](vsphere.assets/image-20211014002249163.png)

![image-20211014002327055](vsphere.assets/image-20211014002327055.png)

![image-20211014002439101](vsphere.assets/image-20211014002439101.png)

回到connetion

![image-20211014002524017](vsphere.assets/image-20211014002524017.png)

![image-20211014002602582](vsphere.assets/image-20211014002602582.png)

# 桌面池



## 自动

准备父VM

选择组织非个人所拥有

![image-20211015092916360](vsphere.assets/image-20211015092916360.png)

![image-20211015092942924](vsphere.assets/image-20211015092942924.png)

![image-20211015093020219](vsphere.assets/image-20211015093020219.png)

根据用户设置用户名

agent 默认安装就可以了

确定好之后关机做快照。

![image-20211015100003441](vsphere.assets/image-20211015100003441.png)

![image-20211015100017848](vsphere.assets/image-20211015100017848.png)

![image-20211015100143649](vsphere.assets/image-20211015100143649.png)

![image-20211015100324831](vsphere.assets/image-20211015100324831.png)

![image-20211015100555631](vsphere.assets/image-20211015100555631.png)

![image-20211015100613049](vsphere.assets/image-20211015100613049.png)

![image-20211015101132744](vsphere.assets/image-20211015101132744.png)

![image-20211015101223908](vsphere.assets/image-20211015101223908.png)

![image-20211015103041975](vsphere.assets/image-20211015103041975.png)

![image-20211015103130778](vsphere.assets/image-20211015103130778.png)

![image-20211015103241681](vsphere.assets/image-20211015103241681.png)

这里使用用户组授权，那个用户先登录就是那个用户使用

![image-20211015105945892](vsphere.assets/image-20211015105945892.png)

![image-20211015110424486](vsphere.assets/image-20211015110424486.png)

这里出现置备会出现桌面不可用的情况

![image-20211015110536902](vsphere.assets/image-20211015110536902.png)

然后登陆![image-20211015110608481](vsphere.assets/image-20211015110608481.png)

客户端

![image-20211015111201697](vsphere.assets/image-20211015111201697.png)

专用桌面

![image-20211015132454831](vsphere.assets/image-20211015132454831.png)

创建跟浮动差不多。

![image-20211015132645808](vsphere.assets/image-20211015132645808.png)

慢慢等待

![image-20211015132705234](vsphere.assets/image-20211015132705234.png)

这里会自动开机并且完成设置会自动关闭

![image-20211015153410891](vsphere.assets/image-20211015153410891.png)

完全克隆

![image-20211015192022493](vsphere.assets/image-20211015192022493.png)

模板机需要加AD安装agent

## 手动

![image-20211016090157172](vsphere.assets/image-20211016090157172.png)

![image-20211016090218205](vsphere.assets/image-20211016090218205.png)

![image-20211016090315619](vsphere.assets/image-20211016090315619.png)

![image-20211016090339079](vsphere.assets/image-20211016090339079.png)

![image-20211016090446469](vsphere.assets/image-20211016090446469.png)

等待状态为可用

![image-20211016090559731](vsphere.assets/image-20211016090559731.png)

这个是因为有用户登录到虚拟机里面了。注销即可

![image-20211016090736408](vsphere.assets/image-20211016090736408.png)

![image-20211016090933673](vsphere.assets/image-20211016090933673.png)

![image-20211016091340859](vsphere.assets/image-20211016091340859.png)

## RDS

上传一个ISO

![image-20211016091649741](vsphere.assets/image-20211016091649741.png)

NFS上传速度53M/s

![image-20211016104642338](vsphere.assets/image-20211016104642338.png)

![image-20211016104711508](vsphere.assets/image-20211016104711508.png)

![image-20211016104738075](vsphere.assets/image-20211016104738075.png)

![image-20211016122412215](vsphere.assets/image-20211016122412215.png)

![image-20211016122511647](vsphere.assets/image-20211016122511647.png)

![image-20211016122549115](vsphere.assets/image-20211016122549115.png)

![image-20211016122647477](vsphere.assets/image-20211016122647477.png)

![image-20211016122755269](vsphere.assets/image-20211016122755269.png)

![image-20211016123152836](vsphere.assets/image-20211016123152836.png)

![](vsphere.assets/image-20211016123354594.png)

![image-20211016123417497](vsphere.assets/image-20211016123417497.png)

![image-20211016123512518](vsphere.assets/image-20211016123512518.png)

![image-20211016123524030](vsphere.assets/image-20211016123524030.png)

![image-20211016123540256](vsphere.assets/image-20211016123540256.png)

![image-20211016123626471](vsphere.assets/image-20211016123626471.png)

![image-20211016124332496](vsphere.assets/image-20211016124332496.png)

**数量**可自己选择，最大为500 。**企业协议号码**可使用**6565792,4954438,6879321或者5296992。**

![image-20211016124422296](vsphere.assets/image-20211016124422296.png)

![image-20211016124442719](vsphere.assets/image-20211016124442719.png)

![image-20211016124611890](vsphere.assets/image-20211016124611890.png)

![image-20211016124620586](vsphere.assets/image-20211016124620586.png)



![image-20211016125034032](vsphere.assets/image-20211016125034032.png)

![image-20211016125021513](vsphere.assets/image-20211016125021513.png)

打开组策略

gpedit.msc 运行输入

![image-20211016125343359](vsphere.assets/image-20211016125343359.png)



![image-20211016125505637](vsphere.assets/image-20211016125505637.png)

![image-20211016125530837](vsphere.assets/image-20211016125530837.png)

![image-20211016125545084](vsphere.assets/image-20211016125545084.png)

![image-20211016125608128](vsphere.assets/image-20211016125608128.png)

![image-20211016125650216](vsphere.assets/image-20211016125650216.png)

![image-20211016125742675](vsphere.assets/image-20211016125742675.png)

![image-20211016125839524](vsphere.assets/image-20211016125839524.png)

![image-20211016130156806](vsphere.assets/image-20211016130156806.png)



![image-20211016130142134](vsphere.assets/image-20211016130142134.png)

![image-20211016130249759](vsphere.assets/image-20211016130249759.png)

![image-20211016130234229](vsphere.assets/image-20211016130234229.png)

安装agent

![image-20211016131907777](vsphere.assets/image-20211016131907777.png)

重启之后再装agent

![image-20211016133406349](vsphere.assets/image-20211016133406349.png)

![image-20211016133937894](vsphere.assets/image-20211016133937894.png)

### 创建RDS桌面

![image-20211016134153787](vsphere.assets/image-20211016134153787.png)

![image-20211016134213160](vsphere.assets/image-20211016134213160.png)

![image-20211016134226496](vsphere.assets/image-20211016134226496.png)

![image-20211016134234512](vsphere.assets/image-20211016134234512.png)

![image-20211016134309390](vsphere.assets/image-20211016134309390.png)

![image-20211016134323837](vsphere.assets/image-20211016134323837.png)

![image-20211016134335297](vsphere.assets/image-20211016134335297.png)

![](vsphere.assets/image-20211016134426905.png)

![image-20211016134442627](vsphere.assets/image-20211016134442627.png)

客户端登陆

![image-20211016134551514](vsphere.assets/image-20211016134551514.png)

![image-20211016134652967](vsphere.assets/image-20211016134652967.png)

返回RDS主机设置

![image-20211016134824710](vsphere.assets/image-20211016134824710.png)

![image-20211016135017224](vsphere.assets/image-20211016135017224.png)

![image-20211016135107115](vsphere.assets/image-20211016135107115.png)

![image-20211016135134721](vsphere.assets/image-20211016135134721.png)

![image-20211016135315750](vsphere.assets/image-20211016135315750.png)

![image-20211016135340473](vsphere.assets/image-20211016135340473.png)

![image-20211016135411290](vsphere.assets/image-20211016135411290.png)



## 应用桌面池

![image-20211016135723209](vsphere.assets/image-20211016135723209.png)

手动添加的就是麻烦，适用于单个定制app

自动发现

![image-20211016140503980](vsphere.assets/image-20211016140503980.png)

![image-20211016140627678](vsphere.assets/image-20211016140627678.png)

![image-20211016140819741](vsphere.assets/image-20211016140819741.png)

![image-20211016140841189](vsphere.assets/image-20211016140841189.png)

![image-20211016140901253](vsphere.assets/image-20211016140901253.png)

![image-20211016140936392](vsphere.assets/image-20211016140936392.png)

![image-20211016141022652](vsphere.assets/image-20211016141022652.png)

vmware horzion VDI基本完成。
