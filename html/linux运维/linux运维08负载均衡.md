# 负载均衡环境

## 设备一览表

| 名称          | IP            | 服务                  |
| ------------- | ------------- | --------------------- |
| web01         | 192.168.1.197 | nginx,php             |
| web02         | 192.168.1.196 | nginx,php             |
| web03         | 192.168.1.195 | nginx,php             |
| nfs and mysql | 192.168.1.200 | nfs mysql             |
| ha1           | 192.168.1.193 | nginx                 |
| ha2           | 192.168.1.192 |                       |
| zabbix        | 192.168.1.191 | zabbix,apache,mariadb |

企业中NFS 和mysql单独服务器

```sh
yum install -y mariadb-server mariadb nfs-utils rpcbind
```



### NFS-MYSQL服务器操作

```bash
[root@data01 /server/scripts]# vim check_nfs_mariadb.sh 
#!/bin/bash
#rpm -qa mariadb-server mariadb nfs-utils rpcbind
 
Nfs=`rpm -qa |grep -c nfs-utils`
Rpcbind=`rpm -qa|grep -c rpcbind`
Mariadb=`rpm -qa mariadb-server mariadb|wc -l`
if [ $Nfs -ne 1 ];then                                                  
     echo "nfs not installed!"
     yum install -y nfs-utils
else
     echo "nfs is installed!"
fi
if [ $Rpcbind -ne 1 ];then
     echo "rpcbind not installed!"
     yum install -y rpcbind
else
     echo "rpcbind is installed"
fi
if [ $Mariadb -ne 2 ];then
     echo "mariadb not installed" 
     yum install -y mariadb-server mariadb
else
     echo "mariadb is installed"
 fi

[root@data01 /server/scripts]# sh check_nfs_mariadb.sh 
nfs is installed!
rpcbind is installed
mariadb is installed

[root@data01 /server/scripts]# systemctl start nfs && systemctl enable nfs
[root@data01 /server/scripts]# systemctl start rpcbind && systemctl enable rpcbind
[root@data01 /server/scripts]# systemctl start mariadb.service && systemctl enable mariadb.service 

[root@data01 /server/scripts]# echo "/data 192.168.1.0/24(rw,sync)" >/etc/exports
[root@data01 /server/scripts]# mkdir /data
[root@data01 /server/scripts]# chown -R nfsnobody.nfsnobody /data/
[root@data01 /server/scripts]# ll /data/
总用量 0
[root@data01 /server/scripts]# ll -d /data/
drwxr-xr-x. 2 nfsnobody nfsnobody 6 4月  10 10:52 /data/

[root@data01 /server/scripts]# mysqladmin -u root passwo
[root@data01 /server/scripts]# mysql -uroot -p123456
MariaDB [(none)]> create database wordpress;
MariaDB [(none)]> create database discuz;
MariaDB [(none)]> create database wecenter;
MariaDB [(none)]> create database wenkuicms;
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| discuz             |
| mysql              |
| performance_schema |
| test               |
| wecenter           |
| wenkuicms          |
| wordpress          |
+--------------------+
MariaDB [(none)]> select user.host from mysql.user;
+-----------+
| host      |
+-----------+
| 127.0.0.1 |
| ::1       |
| data01    |
| data01    |
| localhost |
| localhost |
+-----------+
```

### WEB 服务器操作

#### 怎么把本地存储和NFS建立关系

注意：nfs服务器是否关闭firewalld selinux

##### 1、如何找出网站存储数据的位置

上传一篇文章或者已有图片找到源码或者右键属性

![image-20210818215337891](./linux运维08负载均衡.assets\image-20210818215337891.png)

![image-20210818215348902](./linux运维08负载均衡.assets\image-20210818215348902.png)

这里就可以看到位于服务器wp-content/uploads 目录下

```bash
[root@web01 ~]# tree /html/blog/wordpress/wp-content/ -L 1
/html/blog/wordpress/wp-content/
├── index.php
├── languages
├── plugins
├── themes
├── upgrade
└── uploads
[root@web01 ~]# find /html -type f -mmin -30
/html/blog/wordpress/wp-content/uploads/2021/04/nginx网络架构.png
/html/blog/wordpress/wp-content/uploads/2021/04/nginx网络架构-300x212.png
/html/blog/wordpress/wp-content/uploads/2021/04/nginx网络架构-1024x724.png
/html/blog/wordpress/wp-content/uploads/2021/04/nginx网络架构-150x150.png
/html/blog/wordpress/wp-content/uploads/2021/04/nginx网络架构-768x543.png
/html/blog/wordpress/wp-content/uploads/2021/04/nginx网络架构-330x190.png
/html/blog/wordpress/wp-content/uploads/2021/04/nginx网络架构-1110x444.png
/html/blog/wordpress/wp-content/uploads/2021/04/nginx网络架构-570x350.png
/html/blog/wordpress/wp-content/uploads/2021/04/nginx网络架构-270x250.png
/html/blog/wordpress/wp-content/uploads/2021/04/nginx网络架构-1123x500.png
/html/blog/wordpress/wp-content/uploads/2021/04/nginx网络架构-1110x500.png
/html/blog/wordpress/wp-content/uploads/2021/04/nginx网络架构-510x510.png
/html/blog/wordpress/wp-content/uploads/2021/04/nginx网络架构-1110x624.png
/html/blog/wordpress/wp-content/uploads/2021/04/nginx网络架构-528x297.png
/html/blog/wordpress/wp-content/uploads/2021/04/nginx网络架构-517x235.png
```

```bash
inotifywait -mrq /html/blog
```

##### 2、存储服务器创建对应的目录

```bash
[root@data01 ~]# mkdir /data/{wwb,blog,bbs} -p
[root@data01 ~]# >/etc/exports
[root@data01 ~]# ll /data/
总用量 0
drwxr-xr-x 2 nfsnobody root 6 4月  10 13:58 bbs
drwxr-xr-x 2 nfsnobody root 6 4月  10 13:58 blog
drwxr-xr-x 2 nfsnobody root 6 4月  10 16:23 www
[root@data01 ~]# for i in {bbs,blog,www}
> do
> echo "/data/$i 192.168.1.0/24(rw,sync)" >>/etc/exports
> done
[root@data01 ~]# cat /etc/exports
/data/bbs 192.168.1.0/24(rw,sync)
/data/blog 192.168.1.0/24(rw,sync)
/data/www 192.168.1.0/24(rw,sync)
[root@data01 ~]# systemctl restart rpcbind
[root@data01 ~]# systemctl restart nfs
[root@data01 ~]# showmount -e 192.168.1.200
Export list for 192.168.1.200:
/data/www  192.168.1.0/24
/data/blog 192.168.1.0/24
/data/bbs  192.168.1.0/24
```

```bash
[root@web01 ~]# mount -t nfs 192.168.1.200:/data/blog /html/blog/wordpress/wp-content/uploads/
[root@web01 ~]# df -h
文件系统                  容量  已用  可用 已用% 挂载点
devtmpfs                  476M     0  476M    0% /dev
tmpfs                     487M     0  487M    0% /dev/shm
tmpfs                     487M  7.7M  479M    2% /run
tmpfs                     487M     0  487M    0% /sys/fs/cgroup
/dev/sda3                  18G  2.5G   16G   15% /
/dev/sda1                 509M  153M  356M   31% /boot
tmpfs                      98M     0   98M    0% /run/user/0
192.168.1.200:/data/blog   18G  2.0G   16G   12% /html/blog/wordpress/wp-content/uploads
[root@web01 ~]# mv /tmp/2021/ /html/blog/wordpress/wp-content/uploads/
mv: 无法保留"/html/blog/wordpress/wp-content/uploads/2021/uploads/2021/03/1-2.jpg" 的所有者: 不允许的操作
mv: 无法保留"/html/blog/wordpress/wp-content/uploads/2021/uploads/2021/03/1-2-300x169.jpg" 的所有者: 不允许的操作
mv: 无法保留"/html/blog/wordpress/wp-content/uploads/2021/uploads/2021/03/1-2-1024x576.jpg" 的所有者: 不允许的操作
mv: 无法保留"/html/blog/wordpress/wp-content/uploads/2021/uploads/2021/03/1-2-150x150.jpg" 的所有者: 不允许的操作

[root@web01 ~]# mount -t nfs 192.168.1.200:/data/blog /mnt
mount.nfs: access denied by server while mounting 192.168.1.200:/data/blog
服务器端没有加子网掩码 /24 导致

```

```bash
[root@data01 ~]# groupadd nginx -g 995
[root@data01 ~]# useradd nginx -s /sbin/nologin -M -u 997 -g 995
[root@data01 ~]# id nginx
uid=997(nginx) gid=995(nginx) 组=995(nginx)
[root@data01 ~]# chown -R nginx /data/
[root@data01 ~]# ll /data/
总用量 0
drwxr-xr-x 2 nginx root  6 4月  10 13:58 bbs
drwxr-xr-x 3 nginx root 18 4月  10 16:57 blog
drwxr-xr-x 2 nginx root  6 4月  10 16:23 www
[root@data01 ~]# sed -ri 's/(sync)/\1,anonuid=997,anongid=995/g' /etc/exports
[root@data01 ~]# cat /etc/exports
/data/bbs 192.168.1.0/24(rw,sync,anonuid=997,anongid=995)
/data/blog 192.168.1.0/24(rw,sync,anonuid=997,anongid=995)
/data/www 192.168.1.0/24(rw,sync,anonuid=997,anongid=995)

```

web01 root不能移动或者复制备份的文件到nfs共享的/data/blog里面。只有web01把之前备份的数据打包 tar zcvpf 然后scp 上传到nfs服务器，再进入nfs root解压把数据恢复。

因为nfs权限是755 也就是web01只有所属用户nginx才可以写入删除等操作。

```bash
[root@web01 /html/blog/wordpress/wp-content/uploads]# tar zcvpf /tmp/2021.tar.gz /tmp/2021
[root@web01 /html/blog/wordpress/wp-content/uploads]# scp /tmp/2021.tar.gz root@192.168.1.200:/tmp
```

```bash
[root@data01 /data/blog]# ls /tmp/
2021.tar.gz                
[root@data01 /data/blog]# tar -xvf /tmp/2021.tar.gz /tmp
[root@data01 /data/blog]#mv /tmp/2021/* /data/blog          
[root@data01 /data/blog]# ls
2021  nimble_css
[root@data01 /data/blog]# tree -L 2
.
├── 2021
│   ├── 03
│   └── 04
└── nimble_css
    ├── index.php
    └── skp__home.css
```

nfs如何在web01开机自启

/etc/rc.loacl

```bash
[root@web01 ~]# cat /etc/fstab 
UUID=b0f4c029-ea02-4787-a691-68a52721693b /          xfs     defaults        0 0
UUID=e6b89be4-d434-4096-b094-8a31dc3260d6 /boot      xfs     defaults        0 0
UUID=52a98cfb-db68-4cf3-bb0f-16bea01335fd swap       swap    defaults        0 0
192.168.1.200:/data/blog   /html/blog/wordpress/wp-content/uploads nfs    user,auto       0 0
```

![image-20210818215922513](./linux运维08负载均衡.assets\image-20210818215922513.png)

小结：存储到NFS服务器权限问题要特别注意。data服务器上面的nginx 用户要和web上面的一致。用同样的方法把discuz wecenter一起调试了。

下面错误是因为没有安装nfs-utils

```bash
[root@web02 ~]# mount -t nfs 192.168.1.200:/data/blog /html/blog/wordpress/wp-content/uploads/
mount: 文件系统类型错误、选项错误、192.168.1.200:/data/blog 上有坏超级块、
       缺少代码页或助手程序，或其他错误
       (对某些文件系统(如 nfs、cifs) 您可能需要
       一款 /sbin/mount.<类型> 助手程序)
       
       有些情况下在 syslog 中可以找到一些有用信息- 请尝试
       dmesg | tail  这样的命令看看。
```

```bash
[root@web03 ~]# yum install -y nfs-utils
```

#### discuz

![image-20210818220131836](./linux运维08负载均衡.assets\image-20210818220131836.png)

```bash
[root@web01 ~]# find /html/bbs -type f -mmin -5
/html/bbs/data/attachment/forum/202104/index.html
/html/bbs/data/attachment/forum/202104/10/index.html
/html/bbs/data/attachment/forum/202104/10/200216zffnfxhf2knhfj2f.png
/html/bbs/data/attachment/image/index.htm
/html/bbs/data/attachment/image/000/index.html
/html/bbs/data/attachment/image/000/00/index.html
/html/bbs/data/attachment/image/000/00/00/index.html
/html/bbs/data/attachment/image/000/00/00/02_320_320.jpg
/html/bbs/data/attachment/image/000/00/00/02_720_720.jpg
/html/bbs/data/sendmail.lock
[root@web01 ~]# rm -rf /tmp/*
[root@web01 ~]# cd /html/bbs/data/attachment/
[root@web01 /html/bbs/data/attachment]# ls
album  category  common  forum  group  image  index.htm  portal  profile  swfupload  temp
[root@web01 /html/bbs/data/attachment]# ls forum/
202103  202104  index.htm
[root@web01 /html/bbs/data/attachment]# mkdir /tmp/bbs
[root@web01 /html/bbs/data/attachment]# mv forum/* /tmp/bbs
[root@web01 /html/bbs/data/attachment]# ls forum/
[root@web01 /html/bbs/data/attachment]# ls /tmp/bbs/
202103  202104  index.htm
[root@web01 /html/bbs/data/attachment]# tar czvf /tmp/bbs.tar.gz /tmp/bbs/
tar: 从成员名中删除开头的“/”
/tmp/bbs/
/tmp/bbs/202103/
/tmp/bbs/202103/index.html
/tmp/bbs/202103/16/
/tmp/bbs/202103/16/index.html
/tmp/bbs/202103/16/195447gddvgz5n6969tbvn.jpg
/tmp/bbs/202104/
/tmp/bbs/202104/index.html
/tmp/bbs/202104/10/
/tmp/bbs/202104/10/index.html
/tmp/bbs/202104/10/200216zffnfxhf2knhfj2f.png
/tmp/bbs/index.htm
[root@web01 /html/bbs/data/attachment]# ll /tmp/
总用量 508
drwxr-xr-x 4 root root     51 4月  10 20:07 bbs
-rw-r--r-- 1 root root 519995 4月  10 20:12 bbs.tar.gz
[root@web01 /html/bbs/data/attachment]# scp /tmp/bbs.tar.gz root@192.168.1.200:/tmp

-----------------------------------------------
[root@data01 ~]# rm -rf /tmp/*
[root@data01 ~]# ls /tmp
[root@data01 ~]# cd /tmp
[root@data01 /tmp]# ls
bbs.tar.gz
[root@data01 /tmp]# tar xf bbs.tar.gz 
[root@data01 /tmp]# ls
bbs.tar.gz  tmp
[root@data01 /tmp]# ls tmp/bbs/
202103  202104  index.htm
[root@data01 /tmp]# mv tmp/bbs/* /data/bbs/
----------------------------------------------------
[root@web01 /html/bbs/data/attachment]# ls forum/
202103  202104  index.htm
[root@web01 /html/bbs/data/attachment]# umount -lf ./forum/
[root@web01 /html/bbs/data/attachment]# df -h
文件系统                  容量  已用  可用 已用% 挂载点
devtmpfs                  476M     0  476M    0% /dev
tmpfs                     487M     0  487M    0% /dev/shm
tmpfs                     487M  7.7M  479M    2% /run
tmpfs                     487M     0  487M    0% /sys/fs/cgroup
/dev/sda3                  18G  2.5G   16G   15% /
/dev/sda1                 509M  153M  356M   31% /boot
192.168.1.200:/data/blog   18G  2.0G   16G   12% /html/blog/wordpress/wp-content/uploads
tmpfs                      98M     0   98M    0% /run/user/0
[root@web01 /html/bbs/data/attachment]# ls forum/
[root@web01 /html/bbs/data/attachment]# mount -t nfs 192.168.1.200:/data/bbs ./forum/
[root@web01 /html/bbs/data/attachment]# df -h
文件系统                  容量  已用  可用 已用% 挂载点
devtmpfs                  476M     0  476M    0% /dev
tmpfs                     487M     0  487M    0% /dev/shm
tmpfs                     487M  7.7M  479M    2% /run
tmpfs                     487M     0  487M    0% /sys/fs/cgroup
/dev/sda3                  18G  2.5G   16G   15% /
/dev/sda1                 509M  153M  356M   31% /boot
192.168.1.200:/data/blog   18G  2.0G   16G   12% /html/blog/wordpress/wp-content/uploads
tmpfs                      98M     0   98M    0% /run/user/0
192.168.1.200:/data/bbs    18G  2.0G   16G   12% /html/bbs/data/attachment/forum
-----------------------------------------
```

经过测试完全没有问题

```bash
[root@web01 /html/bbs/data/attachment]# cat /etc/fstab 
UUID=b0f4c029-ea02-4787-a691-68a52721693b /        xfs     defaults        0 0
UUID=e6b89be4-d434-4096-b094-8a31dc3260d6 /boot    xfs     defaults        0 0
UUID=52a98cfb-db68-4cf3-bb0f-16bea01335fd swap     swap    defaults        0 0
192.168.1.200:/data/blog   /html/blog/wordpress/wp-content/uploads nfs    user,auto       0 0
192.168.1.200:/data/bbs    /html/bbs/data/attachment/forum         nfs    user,auto       0 0
[root@web01 /html/bbs/data/attachment]# mount -a
[root@web02 ~]# cat /tmp/fstab 
UUID=b0f4c029-ea02-4787-a691-68a52721693b /                       xfs     defaults        0 0
UUID=e6b89be4-d434-4096-b094-8a31dc3260d6 /boot                   xfs     defaults        0 0
UUID=52a98cfb-db68-4cf3-bb0f-16bea01335fd swap                    swap    defaults        0 0
192.168.1.200:/data/blog   /html/blog/wordpress/wp-content/uploads nfs    user,auto       0 0
192.168.1.200:/data/bbs    /html/bbs/data/attachment/forum         nfs    user,auto       0 0
[root@web02 ~]# cp /tmp/fstab /etc/
cp：是否覆盖"/etc/fstab"？ y
[root@web02 ~]# mount -a

[root@web03 ~]# cp /tmp/fstab /etc/
cp：是否覆盖"/etc/fstab"？ y
[root@web03 ~]# mount -a
[root@web03 ~]# df -h
文件系统                  容量  已用  可用 已用% 挂载点
devtmpfs                  476M     0  476M    0% /dev
tmpfs                     487M     0  487M    0% /dev/shm
tmpfs                     487M  7.7M  479M    2% /run
tmpfs                     487M     0  487M    0% /sys/fs/cgroup
/dev/sda3                  18G  2.5G   16G   15% /
/dev/sda1                 509M  153M  356M   31% /boot
tmpfs                      98M     0   98M    0% /run/user/0
192.168.1.200:/data/blog   18G  2.0G   16G   12% /html/blog/wordpress/wp-content/uploads
192.168.1.200:/data/bbs    18G  2.0G   16G   12% /html/bbs/data/attachment/forum

```

#### 怎么把数据库迁移到远程服务器

```bash
第一个历程: 将web服务器本地数据库数据进行备份
 mysqldump -uroot -poldboy123 --all-database >/tmp/web_back.sql

第二个历程: 将备份数据进行迁移
 scp -rp /tmp/web_back.sql 172.16.1.51:/tmp
 
 第三个历程: 恢复数据信息
 yum install -y mariadb-server mariadb
 mysql -uroot -poldboy123 </tmp/web_back.sql

 第四个历程: 修改数据库服务器中数据库用户信息
 MariaDB [(none)]> select user,host from mysql.user;
+-----------+-----------+
| user      | host      |
+-----------+-----------+
| root      | 127.0.0.1 |
| root      | ::1       |
|           | localhost |
| root      | localhost |
| wordpress | localhost |
|           | web01     |
| root      | web01     |
+-----------+-----------+
7 rows in set (0.00 sec)
 
 优化: 删除无用的用户信息
 delete from mysql.user where user="" and host="localhost";
delete from mysql.user where user="" and host="web01";
 
 添加: 添加新的用户信息
 grant all on wordpress.* to 'wordpress'@'172.16.1.%' identified by 'oldboy123';
 flush privileges;
 第五个历程: 修改web服务器代码文件信息
 vim wp-config.php
 /** MySQL hostname */
define( 'DB_HOST', '172.16.1.51' );
 
 第六个历程: 停止web服务器上数据库服务
[root@web01 /html/bbs/config]# systemctl stop mariadb
[root@web01 /html/bbs/config]# systemctl disable mariadb
Removed symlink /etc/systemd/system/multi-user.target.wants/mariadb.service.

```

```bash
[root@web01 ~]# mysqldump -uroot -p123456 --all-database >/tmp/web01.sql
[root@web01 ~]# ls /tmp/
bbs  bbs.tar.gz  web01.sql
[root@web01 ~]# scp /tmp/web01.sql root@192.168.1.200:/tmp
[root@data01 /tmp]# ls
bbs.tar.gz  tmp  web01.sql
[root@data01 /tmp]# mysql -uroot -p123456 </tmp/web01.sql 
[root@data01 /tmp]# mysql -uroot -p123456
MariaDB [(none)]> select user,host from mysql.user;
+-----------+-----------+
| user      | host      |
+-----------+-----------+
| root      | 127.0.0.1 |
| root      | ::1       |
|           | localhost |
| discuz    | localhost |
| root      | localhost |
| wecenter  | localhost |
| wenkucms  | localhost |
| wordpress | localhost |
|           | web01     |
| root      | web01     |
+-----------+-----------+
MariaDB [(none)]> delete from mysql.user where user="wenkuicms" and host="localhost";
MariaDB [(none)]> delete from mysql.user where user="" and host="localhost";
MariaDB [(none)]> delete from mysql.user where user="" and host="web01";
MariaDB [(none)]> grant all on wordpress.* to 'wordpress'@'192.168.1.%' identified by '123456';
MariaDB [(none)]> grant all on discuz.* to 'discuz'@'192.168.1.%' identified by '123456';
MariaDB [(none)]> flush privileges;
MariaDB [(none)]>
[root@web01 ~]# vim /html/blog/wordpress/wp-config.php
```

![image-20210818220914646](./linux运维08负载均衡.assets\image-20210818220914646.png)

```bash
[root@web01 /html/bbs/config]# vim config_global.php
```

![image-20210818220935903](./linux运维08负载均衡.assets\image-20210818220935903.png)

```bash
[root@web01 /html/bbs/config]# vim config_ucenter.php
```

![image-20210818220953213](./linux运维08负载均衡.assets\image-20210818220953213.png)

```bash
[root@web01 /html/bbs/config]# systemctl stop mariadb && systemctl disable mariadb
[root@web02 ~]# systemctl stop mariadb && systemctl disable mariadb
[root@web03 ~]# systemctl stop mariadb && systemctl disable mariadb
```

上面修改好的文件传到web02 web03去。

```bash
[root@web01 ~]# scp /html/blog/wordpress/wp-config.php root@192.168.1.196:/tmp
[root@web01 ~]# scp /html/blog/wordpress/wp-config.php root@192.168.1.195:/tmp
[root@web01 ~]# scp /html/bbs/config/config_global.php root@192.168.1.195:/tmp
[root@web01 ~]# scp /html/bbs/config/config_ucenter.php root@192.168.1.195:/tmp
[root@web01 ~]# scp /html/bbs/config/config_ucenter.php root@192.168.1.196:/tmp
[root@web01 ~]# scp /html/bbs/config/config_global.php root@192.168.1.196:/tmp
```

```bash
[root@web02 ~]# cp /tmp/wp-config.php /html/blog/wordpress/
cp：是否覆盖"/html/blog/wordpress/wp-config.php"？ y
[root@web02 ~]# cp /tmp/config_global.php /html/bbs/config/
cp：是否覆盖"/html/bbs/config/config_global.php"？ y
[root@web02 ~]# cp /tmp/config_ucenter.php /html/bbs/config/
cp：是否覆盖"/html/bbs/config/config_ucenter.php"？ y
```

```bash
[root@web03 ~]# cp /tmp/wp-config.php /html/blog/wordpress/
cp：是否覆盖"/html/blog/wordpress/wp-config.php"？ y
[root@web03 ~]# cp /tmp/config_global.php /html/bbs/config/
cp：是否覆盖"/html/bbs/config/config_global.php"？ y
[root@web03 ~]# cp /tmp/config_ucenter.php /html/bbs/config/
cp：是否覆盖"/html/bbs/config/config_ucenter.php"？ y
```

   问题01:

   数据库服务没有正确启动: Error establishing a database connection 连接不上3306端口

   问题02: 

​    PHP服务没有开启,报502错误

​     网站换域名要特别注意！！！

​     web01代码信息迁移到web02服务器,并且修改了网站域名无法正确访问

​     访问新域名会自动跳转到老的域名

   方法一: 

​     修改wordpres后台设置信息,将后台中老的域名改为新的域名

   方法二:

​     修改数据库中的一个表, 在表中修改一个和域名有关的条目信息 (update phpmyadmin)

### (反向代理)负载均衡的概念说明

#### 概念简述

 什么是集群?

   完成相同任务或工作的一组服务器 (web01 web02 web03 -- web集群)

​     什么是负载均衡?

​     1) 实现用户访问请求进行调度分配

​     2) 实现用户访问压力分担

​     什么是反向代理?

   反向代理:   外网 ---> (eth0外网) 代理服务器 (eth1内网) ---> 公司网站服务器web(内网)

            外网用户(客户端)  --- 代理服务器 (服务端)

            代理服务器(客户端) --- web服务器(服务端)  

   正向代理:  内网(局域网主机)    --- (内网)代理服务器(外网) --- 互联网 --- web服务器(日本)

  

#### 准备负载均衡的环境

  集群服务器部署:

​     PS: 集群中每天服务器的配置一模一样

   企业中: 

1. 先部署好一台LNMP服务器,上传代码信息

       2. 进行访问测试

3. 批量部署多台web服务器

4. 将nginx配置文件进行分发

5. 将站点目录分发给所有主机

   教学中:

   01. 将web01作为模板主机克隆

```sh
  sed -i 's#\.7#.8#g' /etc/sysconfig/network-scripts/ifcfg-eth[01]
   hostnamectl set-hostname web02
```

​     利用手动方式实现负载均衡:

  修改hosts主机地址和域名映射文件 

  负载均衡服务器部署:

第一个历程: 安装部署nginx软件

```bash
[root@ha01 ~]# vim /etc/yum.repos.d/nginx.repo
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true

[nginx-mainline]
name=nginx mainline repo
baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
gpgcheck=1
enabled=0
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true

[root@ha01 ~]# yum install -y nginx
[root@ha01 ~]# systemctl start nginx
[root@ha01 ~]# systemctl enable nginx
[root@ha01 ~]# ps -ef |grep [n]ginx
root       1296      1  0 15:16 ?        00:00:00 nginx: master process /usr/sbin/nginx -c /etc/nginx/nginx.conf
nginx      1297   1296  0 15:16 ?        00:00:00 nginx: worker process

```

第二个历程: 编写nginx负载服务配置文件

  ngx_http_upstream_module  --- upstream  负载均衡 

  ngx_http_proxy_module    --- proxy_pass 反向代理

```bash
[root@ha01 /etc/nginx/conf.d]# vim www.conf
upstream www {
    server 192.168.1.197:80;
    server 192.168.1.196:80;
    server 192.168.1.195:80;
}
server {
      listen    80;
      server_name www.oldboy.com;
      location / {
          proxy_pass http://www;
                 }
      }
[root@ha01 /etc/nginx/conf.d]# nginx -t
[root@ha01 /etc/nginx/conf.d]# systemctl restart nginx
```

 第三个历程: 实现负载功能测试

​     搭建集群测试环境:

```bash
[root@web01 /etc/nginx/conf.d]# echo "www.oldboy.com 192.168.1.197" >>/html/www/index.php

[root@web02 /etc/nginx/conf.d]# echo "www.oldboy.com 192.168.1.196" >>/html/www/index.php

[root@web03 /etc/nginx/conf.d]# echo "www.oldboy.com 192.168.1.195" >>/html/www/index.php

```

 修改windows解析文件

![image-20210818221711671](./linux运维08负载均衡.assets\image-20210818221711671.png)

![image-20210818221722864](./linux运维08负载均衡.assets\image-20210818221722864.png)

![image-20210818221732071](./linux运维08负载均衡.assets\image-20210818221732071.png)

![image-20210818221740237](./linux运维08负载均衡.assets\image-20210818221740237.png)

可以看出手动负载均衡成功。

试试其他的站点

```bash
[root@ha01 ~]# vim /etc/nginx/conf.d/www.conf 
upstream www {
    server 192.168.1.197:80;
    server 192.168.1.196:80;
    server 192.168.1.195:80;
}
server {
      listen    80;
      server_name www.oldboy.com oldboy.com;   这个叫别名
      location / {
          proxy_pass http://www;
                 }
      }
```

 负载均衡访问网站异常排错思路:

​     第一步: 负载均衡 测试后端web节点服务器是否能够正常访问

```bash
 [root@lb01 conf.d]# curl -H host:www.oldboy.com 10.0.0.7/wenwen.html
www 10.0.0.7
[root@lb01 conf.d]# curl -H host:www.oldboy.com 10.0.0.8/wenwen.html
www 10.0.0.8
[root@lb01 conf.d]# curl -H host:www.oldboy.com 10.0.0.9/wenwen.html
www 10.0.0.9
```

 第二步: 负载均衡 利用curl命令访问负载均衡服务器

​     查看两个配置文件

   第三步: 打开一个xshell连接 ping www.oldboy.com

   第四步: 配置文件编写不正确

#### 负载均衡配置模块详细说明

```bash
 ngx_http_upstream_module   --- upstream
 实现不同调度功能
 1. 轮询分配请求(平均)
 2. 权重分配请求(能力越强责任越重)
    upstream oldboy {
      server 10.0.0.7:80 weight=3;
      server 10.0.0.8:80 weight=2;
      server 10.0.0.9:80 weight=1;
   }
 3. 实现热备功能(备胎功能)
   	 upstream oldboy {
      server 10.0.0.7:80;
      server 10.0.0.8:80;
      server 10.0.0.9:80 backup;
   }
 4. 定义最大失败次数                 	 健康检查参数
    max_fails=5
 5. 定义失败之后重发的间隔时间	 	 	 健康检查参数
    fail_timeout=10s  会给失败的服务器一次机会
 
 实现不同调度算法
 1. rr  轮询调度算法
 2. wrr 权重调度算法
 3. ip_hash 算法  (出现反复登录的时候)
 4. least_conn  根据服务器连接数分配资源
 
 ngx_http_proxy_module	    --- proxy_pass
 01. 访问不同的网站地址,不能显示不同的网站页面  (面试题)
 proxy_set_header Host $host;

```

```bash
[root@ha01 /etc/nginx/conf.d]# vim www.conf 
  upstream www {
      server 192.168.1.197:80;
      server 192.168.1.196:80;
      server 192.168.1.195:80;
  }
  server {
        listen    80;
        server_name www.oldboy.com;
        location / {
            proxy_pass http://www;
            proxy_set_header Host $host;
                   }
        }
  server {
        listen    80;
        server_name bbs.oldboy.com;
        location / {
            proxy_pass http://www;
            proxy_set_header Host $host;
                   }
        }
  server {
        listen    80;
        server_name blog.oldboy.com;
        location / {
           proxy_pass http://www;
           proxy_set_header Host $host;
                  }
       }
```

  访问网站用户地址信息无法进行分析统计    (面试题)

```sh
   proxy_set_header X-Forwarded-For $remote_addr;
```

![image-20210818222138195](./linux运维08负载均衡.assets\image-20210818222138195.png)

都一样无法分析

![image-20210818222200392](./linux运维08负载均衡.assets\image-20210818222200392.png)

```bash
[root@ha01 /etc/nginx/conf.d]# vim www.conf 
upstream www {
    server 192.168.1.197:80;
    server 192.168.1.196:80;
    server 192.168.1.195:80;
}
server {
      listen    80;
      server_name www.oldboy.com;
      location / {
          proxy_pass http://www;
          proxy_set_header Host $host;
          proxy_set_header X-Forwarded-For $remote_addr;
                 }
      }
server {
      listen    80;
      server_name bbs.oldboy.com;
      location / {
          proxy_pass http://www;
          proxy_set_header Host $host;
          proxy_set_header X-Forwarded-For $remote_addr;
                 }
      }
server {
      listen    80;
      server_name blog.oldboy.com;
      location / {
          proxy_pass http://www;
          proxy_set_header Host $host;
          proxy_set_header X-Forwarded-For $remote_addr;
                 }
      }
```

  访问负载均衡会出现错误页面,影响用户体验

```sh
   proxy_next_upstream error timeout http_404 http_502 http_403;
```

#### 负载均衡企业实践应用

##### 1) 根据用户访问的uri信息进行负载均衡

```bash
 第一个历程: 架构环境规划
	 /upload	   集群-192.168.1.197:80	 html/www/upload	    upload服务器集群
	 /static   集群-192.168.1.196:80  html/www/static    static服务器集群
	 /         集群-192.168.1.195:80  html/www           default服务器集群
```

 web01上进行环境部署:

```bash
[root@web01 /html]# mkdir /html/www/upload
[root@web01 /html]# echo "upload-web HA-192.168.1.197" >/html/www/upload/oldboy.html 
```

Web02

```bash
[root@web02 ~]# mkdir /html/www/static
[root@web02 ~]# echo "static-web HA-192.168.1.196" >/html/www/static/oldboy.html
```

03

```bash
[root@web03 ~]# echo "default-web HA-192.168.1.195" >/html/www/oldboy.html
```

第二个历程: 编写负载均衡配置文件

```bash
[root@ha01 /etc/nginx/conf.d]# cat www.conf 
upstream upload {
    server 192.168.1.197:80;
} 
upstream static {
    server 192.168.1.196:80;
}
upstream default {
    server 192.168.1.195:80;
}
server {
      listen    80;
      server_name www.oldboy.com;
      location / {
          proxy_pass http://default;
          proxy_set_header Host $host;
          proxy_set_header X-Forwarded-For $remote_addr;
          proxy_next_upstream error timeout http_404 http_502 http_403;
                 }
      location /upload {
          proxy_pass http://upload;
          proxy_set_header Host $host;
          proxy_set_header X-Forwarded-For $remote_addr;
          proxy_next_upstream error timeout http_404 http_502 http_403;
                 }
      location /static {
          proxy_pass http://static;
          proxy_set_header Host $host;
          proxy_set_header X-Forwarded-For $remote_addr;
          proxy_next_upstream error timeout http_404 http_502 http_403;
                 }
      }	 
```

![image-20210818222538460](./linux运维08负载均衡.assets\image-20210818222538460.png)

根据用户的URI 进行动静分离

![image-20210818222555225](./linux运维08负载均衡.assets\image-20210818222555225.png)

![image-20210818222602860](./linux运维08负载均衡.assets\image-20210818222602860.png)

总结: 实现网站集群动静分离

​     01. 提高网站服务安全性

​     02. 管理操作工作简化

​     03. 可以换分不同人员管理不同集群服务器

##### 2) 根据用户访问的终端信息显示不同页面

```bash
 第一个历程: 准备架构环境
	 iphone   www.oldboy.com  --- iphone_access 192.168.1.197:80  mobile移动端集群
	 谷歌     www.oldboy.com  --- google_access 192.168.1.196:80  web端集群
	 IE 360   www.oldboy.com  --- default_access 192.168.1.195:80 default端集群
```

```bash
 web01:
[root@web01 /html]# echo "iphone-192.168.1.197" >/html/www/oldboy1.html
web02:
[root@web02 ~]# echo "google 192.168.1.196" >/html/www/oldboy1.html
web03:
[root@web03 ~]# echo "default1 192.168.1.195" >/html/www/oldboy1.html
```

第二个历程: 编写负载均衡配置文件

```bash
[root@ha01 /etc/nginx/conf.d]# cat moble.conf 
upstream mobile {
    server 192.168.1.197:80;
} 
upstream web {
    server 192.168.1.196:80;
}
upstream default {
    server 192.168.1.195:80;
}
server {
      listen    80;
      server_name www.oldboy.com;
      location / {
          if ($http_user_agent ~* iphone){
              proxy_pass http://mobile;
          }
          if ($http_user_agent ~* Chrome){
              proxy_pass http://web;
           }
          proxy_pass http://default;
          proxy_set_header Host $host;
          proxy_set_header X-Forwarded-For $remote_addr;
          proxy_next_upstream error timeout http_404 http_502 http_403;
                 }
}    

[root@ha01 /etc/nginx/conf.d]# nginx -t
nginx: [warn] conflicting server name "www.oldboy.com" on 0.0.0.0:80, ignored
nginx: [warn] conflicting server name "www.oldboy.com" on 0.0.0.0:80, ignored
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

```

因为实验环境配置相同导致。

![image-20210818222746150](./linux运维08负载均衡.assets\image-20210818222746150.png)

![image-20210818222755326](./linux运维08负载均衡.assets\image-20210818222755326.png)

# 高可用

高可用服务介绍说明

避免负载均衡服务出现单点问题   

高可用原理

![image-20210818222834457](./linux运维08负载均衡.assets\image-20210818222834457.png)

如何实现部署高可用服务

  利用keepalived软件实现

   作用:

 1.为LVS服务而诞生出来的  k8s + 容器技术docker 容器<--镜像<--仓库 春节抢红包

​                keepalived + LVS负载均衡软件(4层)

 2. 实现高可用服务功能

## 高可用keepalived服务部署流程

  第一个历程: 准备高可用服务架构（两台机器）

  第二个历程: 安装部署keepalived软件 (lb01 lb02)

```BASH
[root@ha01 ~]# yum install -y keepalived
[root@ha02 ~]# yum install -y keepalived
```

 第三个历程: 编写keepalived配置文件

```bash
 vim /etc/keepalived/keepalived.conf
    GLOBAL	 CONFIGURATION	 	 	 --- 全局配置部分
	 VRRPD 	 CONFIGURATION	 	 	 --- VRRP协议配置部分
      LVS 	 CONFIGURATION	 	 	 --- LVS服务管理配置部分
```

```bash
  [root@lb01 ~]# cat /etc/keepalived/keepalived.conf 
    ! Configuration File for keepalived
   
    global_defs {                  	 --- 全局配置部分
       notification_email {	 	 	 --- 设置发送邮件信息的收件人
         acassen@firewall.loc
         failover@firewall.loc
         sysadmin@firewall.loc
       }
       notification_email_from oldboy@163.com   --- 设置连接的邮件服务器信息
       smtp_server 163.smtp.xxx_
       smtp_connect_timeout 30 
       router_id LVS_DEVEL           --- 高可用集群主机身份标识(集群中主机身份标识名称不能重复)
    }
   
    vrrp_instance oldboy {           --- Vrrp协议家族 oldboy
        state MASTER                 --- 标识所在家族中的身份 (MASTER/BACKUP)
        interface eth0               --- 指定虚拟IP地址出现在什么网卡上
        virtual_router_id 51         --- 标识家族身份信息 多台高可用服务配置要一致 
        priority 100                 --- 设定优先级 优先级越高,就越有可能成为主
        advert_int 1                 --- 定义组播包发送的间隔时间(秒)  主和备配置一样  1
        authentication {             --- 实现通讯需要有认证过程
            auth_type PASS
            auth_pass 1111
        }
        virtual_ipaddress {          --- 配置虚拟IP地址信息
            192.168.200.16
            192.168.200.17
            192.168.200.18
        }
    }

```

 ha01配置信息:

```bash
[root@ha01 ~]# cat /etc/keepalived/keepalived.conf
! Configuration File for keepalived

global_defs {
   router_id ha01
}

vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 150
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.1.190/24
    }
}
```

 lb02配置信息:

```bash
[root@ha02 ~]# cat /etc/keepalived/keepalived.conf 
! Configuration File for keepalived

global_defs {
   router_id ha02
}

vrrp_instance VI_1 {
    state BACKUP
    interface eth0
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.1.190/24
    }
}
```

 第三个历程: 启动keepalived服务

```bash
[root@ha01 ~]# systemctl start keepalived.service 
[root@ha01 ~]# systemctl enable keepalived.service 
```

```bash
[root@ha02 ~]# systemctl start keepalived.service 
[root@ha02 ~]# systemctl enable keepalived.service 
```

 第四个历程: 修改域名和IP地址解析关系

![image-20210818223852265](./linux运维08负载均衡.assets\image-20210818223852265.png)

![image-20210818223859119](./linux运维08负载均衡.assets\image-20210818223859119.png)

这里注意ha02没有安装nginx 导致ha01断开后无法迁移

```bash
[root@ha01 ~]# scp /etc/yum.repos.d/nginx.repo root@192.168.1.192:/etc/yum.repos.d/
[root@ha02 ~]# cat /etc/yum.repos.d/nginx.repo 
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true

[nginx-mainline]
name=nginx mainline repo
baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
gpgcheck=1
enabled=0
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true

[root@ha02 ~]# yum install -y nginx
[root@ha02 ~]# systemctl start nginx
[root@ha02 ~]# systemctl enable nginx

[root@ha01 ~]# scp -r /etc/nginx/backup root@192.168.1.192:/etc/nginx/
[root@ha01 ~]# scp -r /etc/nginx/conf.d/moble.conf root@192.168.1.192:/etc/nginx/conf.d/

[root@ha01 ~]# mv /etc/nginx/conf.d/moble.conf /etc/nginx/backup/
[root@ha01 ~]# mv /etc/nginx/backup/bbs.conf /etc/nginx/conf.d/
[root@ha01 ~]# systemctl reload nginx

[root@ha02 ~]# mv /etc/nginx/conf.d/moble.conf /etc/nginx/backup/
[root@ha02 ~]# mv /etc/nginx/backup/bbs.conf /etc/nginx/conf.d/
[root@ha02 ~]# systemctl reload nginx
```

![image-20210818224119902](./linux运维08负载均衡.assets\image-20210818224119902.png)

![image-20210818224131609](./linux运维08负载均衡.assets\image-20210818224131609.png)

非常完美。是这样的？

![image-20210818224151252](./linux运维08负载均衡.assets\image-20210818224151252.png)



#### 高可用服务企业应用

1. 高可用服务常见异常问题---脑裂问题

   出现原因: 

​     高可用备服务器接收不到主服务器发送的组播包,备服务器上会自动生成VIP（虚拟192.168.1.190/24）地址

  物理原因:

​     高可用集群之间通讯线路出现问题

   逻辑原因:

​     有安全策略阻止

​     如何解决脑裂问题:

#### 01. 进行监控**,**发出告警

​     备服务器出现VIP地址的原因:

​     a 主服务器出现故障

​     b 出现脑裂问题

```bash
[root@ha01 ~]# cat /server/scripts/keepalived_naolie.sh 
#!/bin/bash
vip="192.168.1.190"
ip a s eth0|grep "$vip" &>/dev/null
if [ $? -eq 0 ]
then
    echo "keepalived server abnormal"|mail -s "abnormal-keepalived" 349925756@qq.com
fi	 
```

![image-20210818224329050](./linux运维08负载均衡.assets\image-20210818224329050.png)

直接关闭一台服务器的keepalived服务

#### 02. 如何实现keepalived服务自动释放vip地址资源

 nginx(皇帝) + keepalived(妃子): nginx服务停止(皇帝死了),keepalived也必须停止(殉情)

```bash
一个历程: 编写监控nginx服务状态监控
#!/bin/bash
num=`ps -ef|grep -c [n]ginx`
if [ $num -lt 2 ]
then
   systemctl stop keepalived
 fi
 第二个历程: 测试监控脚本
第三个历程: 实时监控nginx服务状态---keepalived配置文件
check_web=/server/scripts/check_web.sh
vrrp_script check_web {
   script "/server/scripts/check_web.sh"   --- 定义需要监控脚本(脚本是执行权限)   
   interval 2                              --- 执行脚本的间隔时间(秒)  
   weight 2                                --- ???
 }
$check_web
track_script {                             
     check_web                             --- 调用执行你的脚本信息
 }	 

```

 keepalived信息配置  

```bash
 [root@lb01 scripts]# cat /etc/keepalived/keepalived.conf
! Configuration File for keepalived
 global_defs {
   router_id lb01
}

vrrp_script check_web {
 script "/server/scripts/check_web.sh"  
 interval 3   
 weight 2
}

vrrp_instance oldboy {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 150
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
     10.0.0.3/24
    }
   track_script {
         check_web
   }
}
```

 这里有个问题需要写脚本执行

注意：ha01 02不需要安装php服务，只需要 keepalived nginx即可

不然ps -ef |grep nginx 会导致上面脚本失效。因为nginx用户后台在启动php。

当nginx停止的时候需要一起停止keepalived不然会出现不能访问web的问题。shell即可解决。

####  03.如何高可用集群双主配置

   第一个历程: 编写lb01服务器keepalived配置文件

```bash
 [root@lb01 ~]# cat /etc/keepalived/keepalived.conf
! Configuration File for keepalived
 global_defs {
   router_id lb01
}


vrrp_instance oldboy {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 150
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
     10.0.0.3/24
    }
}
vrrp_instance oldgirl {
    state BACKUP
    interface eth0
    virtual_router_id 52
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
     10.0.0.4/24
    }
}
```

 第二个历程: 编写lb02服务器keepalived配置文件  

```BASH
 ! Configuration File for keepalived
global_defs {
   router_id lb02
}

vrrp_instance oldboy {
    state BACKUP
    interface eth0
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        10.0.0.3/24
    }
}
vrrp_instance oldgirl {
    state MASTER
    interface eth0
    virtual_router_id 52
    priority 150
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        10.0.0.4/24
    }
}
```

第三个历程: 编写域名和IP地址解析信息

​     进行抓包查看:

```bash
 www.oldboy.com --- 10.0.0.3(10.0.0.5)
	 10.0.0.1       --- 10.0.0.3
	 10.0.0.5       --- 10.0.0.7
	 10.0.0.7       --- 10.0.0.5
	 10.0.0.3       --- 10.0.0.1
	 
	 bbs.oldboy.com --- 10.0.0.4(10.0.0.6)
	 10.0.0.1       --- 10.0.0.4
	 10.0.0.6       --- 10.0.0.7
	 10.0.0.7       --- 10.0.0.6
	 10.0.0.4       --- 10.0.0.1
```

#### 04、高可用服务安全访问配置(负载均衡服务)

  第一个历程: 修改nginx负载均衡文件

```bash
upstream oldboy {
   server 10.0.0.7:80;
   server 10.0.0.8:80;
   server 10.0.0.9:80;
   }
server {
    listen       10.0.0.3:80;
    server_name  www.oldboy.com;
    location / {
       proxy_pass http://oldboy;
       proxy_set_header Host $host;
       proxy_set_header X-Forwarded-For $remote_addr;
       proxy_next_upstream error timeout http_404 http_502 http_403;
    }
}
server {
    listen       10.0.0.4:80;   这里监测会提示调整内核信息即可，参考下面异常问题
    server_name  bbs.oldboy.com;
    location / {
       proxy_pass http://oldboy;
       proxy_set_header Host $host;
       proxy_set_header X-Forwarded-For $remote_addr;
    }
}
 
 第二个历程: 修改内核文件
```

##### 异常问题:

​     01. 如何设置监听网卡上没有的地址

​     解决: 需要修改内核信息

```bash
echo 'net.ipv4.ip_nonlocal_bind = 1' >>/etc/sysctl.conf
sysctl -p 
```

 第三个历程: 重启nginx负载均衡服务

```sh
systemctl restart nginx
```

高可用服务总结

1) 负载均衡服务扩展补充

​        根据用uri信息进行负载均衡(动静分离架构)

​        根据user_agent信息进行负载均衡(手机用户和浏览器访问用户页面信息可以不一致)

2) 高可用服务作用(避免出现单点故障)

3) keepalived高可用服务

        1. 管理LVS负载均衡软件

        2. 实现高可用功能(vrrp原理)

4) keepalived服务配置文件 

5) keepalived服务企业应用

        1. 可能出现脑裂问题      --- 脚本编写

        2. 如何实现自动释放资源   --- 脚本(监控web服务) 修改keepalived文件

        3. 如何实现双主配置       --- 编写keepalived配置文件 编写多个vrrp实例

   ​     4. 如何实现负载均衡安全访问 --- 编写负载均衡nginx配置文件 配置监听vip地址信息

