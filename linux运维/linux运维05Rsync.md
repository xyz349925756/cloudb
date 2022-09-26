# Rsync 服务（备份服务器）

## 1、简介

Rsync是一个文件传输程序，可以实现全量增量的本地或远程数据同步备份。

1.1安装

```bash
[root@lnmp ~]# yum install -y rsync
```

Linux常用拷贝对比

a 本地备份数据 cp

```bash
[root@nfs01 backup]# cp /etc/hosts /tmp
[root@nfs01 backup]# ll /tmp/hosts
-rw-r--r-- 1 root root 371 May  6 16:11 /tmp/hosts
[root@nfs01 backup]# rsync /etc/hosts /tmp/host_rsync
[root@nfs01 backup]# ll /tmp/host_rsync
-rw-r--r-- 1 root root 371 May  6 16:12 /tmp/host_rsync
```

b 远程备份数据 scp

```bash
scp -rp /etc/hosts root@172.16.1.41:/backup
root@172.16.1.41's password: 
hosts         100%  371    42.8KB/s   00:00
```

> -r  --- 递归复制传输数据
>
> -p  --- 保持文件属性信息不变

```bash
[root@nfs01 ~]# rsync -rp /etc/hosts 172.16.1.41:/backup/hosts_rsync
root@172.16.1.41's password: 	 
[root@nfs01 ~]# rsync -rp /oldboy/ 172.16.1.41:/backup  --- 备份的目录后面有 / 
root@172.16.1.41's password:
[root@backup ~]# ll /backup
total 0
-rw-r--r-- 1 root root 0 May  6 16:24 01.txt
-rw-r--r-- 1 root root 0 May  6 16:24 02.txt
-rw-r--r-- 1 root root 0 May  6 16:24 03.txt
```

> 总结: 在使用rsync备份目录时:
>
> 备份目录后面有 / -- /oldboy/ : 只将目录下面的内容进行备份传输 
>
> 备份目录后面没有/ -- /oldboy : 会将目录本身以及下面的内容进行传输备份

## 2、Rsync 帮助命令

```bash
[root@lnmp ~]# rsync --help
Usage: rsync [OPTION]... SRC [SRC]... DEST
  or   rsync [OPTION]... SRC [SRC]... [USER@]HOST:DEST
  or   rsync [OPTION]... SRC [SRC]... [USER@]HOST::DEST
  or   rsync [OPTION]... SRC [SRC]... rsync://[USER@]HOST[:PORT]/DEST
  or   rsync [OPTION]... [USER@]HOST:SRC [DEST]
  or   rsync [OPTION]... [USER@]HOST::SRC [DEST]
  or   rsync [OPTION]... rsync://[USER@]HOST[:PORT]/SRC [DEST]
```

```bash
rsync [选项]…源…目标
The ':' usages connect via remote shell, while '::' & 'rsync://' usages connect
to an rsync daemon, and require SRC or DEST to start with a module name.
':'用法通过远程shell连接，而'::'和'rsync://'用法则通过
到 rsync 守护进程，并要求 SRC 或 DEST 以模块名开头。
Options
 -v, --verbose               increase verbosity
    冗余
     --info=FLAGS            fine-grained informational verbosity
     --debug=FLAGS           fine-grained debug verbosity
     --msgs2stderr           special output handling for debugging
 -q, --quiet                 suppress non-error messages
抑制非错误信息
     --no-motd               suppress daemon-mode MOTD (see manpage caveat)
 -c, --checksum              skip based on checksum, not mod-time & size
 -a, --archive               archive mode; equals -rlptgoD (no -H,-A,-X)
存档模式 相当于-rlptgoD（无-H -A -X）
     --no-OPTION             turn off an implied OPTION (e.g. --no-D)
 -r, --recursive             recurse into directories
递归到目录中
 -R, --relative              use relative path names
递归到相对路径
     --no-implied-dirs       don't send implied dirs with --relative
 -b, --backup                make backups (see --suffix & --backup-dir)
     --backup-dir=DIR        make backups into hierarchy based in DIR
     --suffix=SUFFIX         set backup suffix (default ~ w/o --backup-dir)
创建备份//以目录为基准备份//设置备份后缀
 -u, --update                skip files that are newer on the receiver
     --inplace               update destination files in-place (SEE MAN PAGE)
     --append                append data onto shorter files
     --append-verify         like --append, but with old data in file checksum
跳过接收器上较新的文件//更新目标文件//增量备份//
 -d, --dirs                  transfer directories without recursing
免递归
 -l, --links                 copy symlinks as symlinks
拷贝为系统连接
 -L, --copy-links            transform symlink into referent file/dir
     --copy-unsafe-links     only "unsafe" symlinks are transformed
     --safe-links            ignore symlinks that point outside the source tree
     --munge-links           munge symlinks to make them safer (but unusable)
将符号链接转换为引用文件/目录//只有不安全的符号链接才会被转换//	 忽略指向源代码树之外的符号链接。//屏蔽符号链接，使其无法使用。简单讲就是针对链接文件备份选择复制还是剔除。
 -k, --copy-dirlinks         transform symlink to a dir into referent dir
将指向目录的符号链接转换为引用目录
 -K, --keep-dirlinks         treat symlinked dir on receiver as dir
将接收机上的符号链接目录当作目录
 -H, --hard-links            preserve hard links
保留硬链
 -p, --perms                 preserve permissions
保留权限
 -E, --executability         preserve the file's executability
     --chmod=CHMOD           affect file and/or directory permissions
保留文件的可执行性//更改文件和/或目录的权限
 -A, --acls                  preserve ACLs (implies --perms)
控制策略
 -X, --xattrs                preserve extended attributes
保留扩展属性
 -o, --owner                 preserve owner (super-user only)
-g, --group                 preserve group
     --devices               preserve device files (super-user only)
     --copy-devices          copy device contents as regular file
     --specials              preserve special files
 -D                          same as --devices --specials
 -t, --times                 preserve modification times
保留修改时间
 -O, --omit-dir-times        omit directories from –times
 -J, --omit-link-times       omit symlinks from --times
     --super                 receiver attempts super-user activities
     --fake-super            store/recover privileged attrs using xattrs
 -S, --sparse                handle sparse files efficiently
     --preallocate           allocate dest files before writing them
高效地处理稀疏文件//写入之前备份好文件
 -n, --dry-run               perform a trial run with no changes made
 -W, --whole-file            copy files whole (without delta-xfer algorithm)
全盘复制
 -x, --one-file-system       don't cross filesystem boundaries
 -B, --block-size=SIZE       force a fixed checksum block-size
强制规定一个固定的校验大小的块
 -e, --rsh=COMMAND           specify the remote shell to use
指定使用远程shell
     --rsync-path=PROGRAM    specify the rsync to run on the remote machine
指定要在远程机器上运行的rsync。
     --existing              skip creating new files on receiver
跳过在接收器上创建新文件
     --ignore-existing       skip updating files that already exist on receiver
忽略已存在文件
     --remove-source-files   sender removes synchronized files (non-dirs)
同步完成删除源文件
     --del                   an alias for --delete-during
     --delete                delete extraneous files from destination dirs
删除目标目录下的无关文件
     --delete-before         receiver deletes before transfer, not during
在传输前删除
     --delete-during         receiver deletes during the transfer
在传输过程删除
     --delete-delay          find deletions during, delete after
     --delete-after          receiver deletes after transfer, not during
传输之后删除
     --delete-excluded       also delete excluded files from destination dirs
也可以从目标目录中删除被排除的文件
     --ignore-missing-args   ignore missing source args without error
     --delete-missing-args   delete missing source args from destination
删除丢失的源参数
     --ignore-errors         delete even if there are I/O errors
     --force                 force deletion of directories even if not empty
强制删除非空文件夹
     --max-delete=NUM        don't delete more than NUM files
     --max-size=SIZE         don't transfer any file larger than SIZE
     --min-size=SIZE         don't transfer any file smaller than SIZE
不传输设置大小的文件
     --partial               keep partially transferred files
     --partial-dir=DIR       put a partially transferred file into DIR
     --delay-updates         put all updated files into place at transfer's end
传输文件位置或者指定目录
 -m, --prune-empty-dirs      prune empty directory chains from the file-list
     --numeric-ids           don't map uid/gid values by user/group name
     --usermap=STRING        custom username mapping
     --groupmap=STRING       custom groupname mapping
     --chown=USER:GROUP      simple username/groupname mapping
     --timeout=SECONDS       set I/O timeout in seconds
     --contimeout=SECONDS    set daemon connection timeout in seconds
 -I, --ignore-times          don't skip files that match in size and mod-time
 -M, --remote-option=OPTION  send OPTION to the remote side only
     --size-only             skip files that match in size
     --modify-window=NUM     compare mod-times with reduced accuracy
 -T, --temp-dir=DIR          create temporary files in directory DIR
 -y, --fuzzy                 find similar file for basis if no dest file
     --compare-dest=DIR      also compare destination files relative to DIR
     --copy-dest=DIR         ... and include copies of unchanged files
     --link-dest=DIR         hardlink to files in DIR when unchanged
 -z, --compress              compress file data during the transfer
传输过程压缩数据
     --compress-level=NUM    explicitly set compression level
     --skip-compress=LIST    skip compressing files with a suffix in LIST
 -C, --cvs-exclude           auto-ignore files the same way CVS does
 -f, --filter=RULE           add a file-filtering RULE
增加一个文件过滤规则
 -F                          same as --filter='dir-merge /.rsync-filter'
                             repeated: --filter='- .rsync-filter'
     --exclude=PATTERN       exclude files matching PATTERN
     --exclude-from=FILE     read exclude patterns from FILE
排除指定文件，多个文件使用一个文件来匹配所有排除文件
     --include=PATTERN       don't exclude files matching PATTERN
     --include-from=FILE     read include patterns from FILE
     --files-from=FILE       read list of source-file names from FILE
 -0, --from0                 all *-from/filter files are delimited by 0s
 -s, --protect-args          no space-splitting; only wildcard special-chars
     --address=ADDRESS       bind address for outgoing socket to daemon
     --port=PORT             specify double-colon alternate port number
     --sockopts=OPTIONS      specify custom TCP options
     --blocking-io           use blocking I/O for the remote shell
     --stats                 give some file-transfer stats
 -8, --8-bit-output          leave high-bit chars unescaped in output
 -h, --human-readable        output numbers in a human-readable format
     --progress              show progress during transfer
 -P                          same as --partial --progress
 -i, --itemize-changes       output a change-summary for all updates
     --out-format=FORMAT     output updates using the specified FORMAT
     --log-file=FILE         log what we're doing to the specified FILE
     --log-file-format=FMT   log updates using the specified FMT
     --password-file=FILE    read daemon-access password from FILE
     --list-only             list the files instead of copying them
     --bwlimit=RATE          limit socket I/O bandwidth
     --outbuf=N|L|B          set output buffering to None, Line, or Block
     --write-batch=FILE      write a batched update to FILE
批量更新文件
     --only-write-batch=FILE like --write-batch but w/o updating destination
     --read-batch=FILE       read a batched update from FILE
     --protocol=NUM          force an older protocol version to be used
     --iconv=CONVERT_SPEC    request charset conversion of filenames
     --checksum-seed=NUM     set block/file checksum seed (advanced)
 -4, --ipv4                  prefer IPv4
 -6, --ipv6                  prefer IPv6
     --version               print version number
(-h) --help                  show this help (-h is --help only if used alone)

重要参数
-v, --verbose     显示详细的传输信息
-a, --archive     命令的归档参数 包含: rtopgDl
-r, --recursive   递归参数
-t, --times       保持文件属性信息时间信息不变(修改时间)
-o, --owner       保持文件属主信息不变
-g, --group       保持文件属组信息不变
PS: 如何让-o和-g参数生效,需要将配置文件uid和gid改为root,需要将 fake super参数进行注释
-p, --perms       保持文件权限信息不变
-D,               保持设备文件信息不变
-l, --links       保持链接文件属性不变
-L,               保持链接文件数据信息不变（生产中建议使用L不建议l因为l只是备份了连接文件，并没有备份文件内容，L就可以备份文件内容）
-P,               显示数据传输的进度信息
--exclude=PATTERN   排除指定数据不被传输
--exclude-from=file 排除指定数据不被传输(批量排除)
--bwlimit=RATE    显示传输的速率  100Mb/8=12.5MB
                  企业案例:    马路(带宽-半)   人人网地方 
--delete          无差异同步参数(慎用)
                  我有的你也有,我没有的你也不能有
```

```bash
rsync命令语法格式
SYNOPSIS
Local:  rsync [OPTION...] SRC... [DEST]
本地备份数据: 
src: 要备份的数据信息
dest: 备份到什么路径中

远程备份数据:
Access via remote shell:
Pull: rsync [OPTION...] [USER@]HOST:SRC... [DEST]
[USER@]    --- 以什么用户身份拉取数据(默认以当前用户)
hosts      --- 指定远程主机IP地址或者主机名称
SRC        --- 要拉取的数据信息
dest       --- 保存到本地的路径信息

Push: rsync [OPTION...] SRC... [USER@]HOST:DEST
SRC        --- 本地要进行远程传输备份的数据
[USER@]    --- 以什么用户身份推送数据(默认以当前用户)
hosts      --- 指定远程主机IP地址或者主机名称
dest       --- 保存到远程的路径信息

守护进程方式备份数据 备份服务 
01. 可以进行一些配置管理
02. 可以进行安全策略管理
03. 可以实现自动传输备份数据
Access via rsync daemon:
Pull: rsync [OPTION...] [USER@]HOST::SRC... [DEST]
      rsync [OPTION...] rsync://[USER@]HOST[:PORT]/SRC... [DEST]
Push: rsync [OPTION...] SRC... [USER@]HOST::DEST
      rsync [OPTION...] SRC... rsync://[USER@]HOST[:PORT]/DEST
```

## 3、应用

替代删除命令

rm命令

```bash
[root@nfs01 ~]# rsync -rp --delete /null/ 172.16.1.41:/backup
root@172.16.1.41's password: 
```

> --delete  实现无差异同步数据

面试题: 有一个存储数据信息的目录, 目录中数据存储了50G数据, 如何将目录中的数据快速删除

rm /目录/* -rf

替代查看文件命令 ls 

```bash
[root@backup ~]# ls /etc/hosts
/etc/hosts
[root@backup ~]# rsync /etc/hosts
-rw-r--r--            371 2019/05/06 11:55:22 hosts
```

## 4、部署

rsync守护进程部署方式

客户端---服务端 

rsync守护进程服务端配置:

```bash
第一个历程: 下载安装软件
rpm -qa|grep rsync
yum install -y rsync 

第二个历程: 编写配置文件
man rsyncd.conf    可以查看rsyncd.conf 文件的高级功能
vim /etc/rsyncd.conf    编辑这个配置文件
[root@lnmp ~]# cp /etc/rsyncd.conf{,.bak}
[root@lnmp ~]# ls /etc/rsyncd.conf
rsyncd.conf      rsyncd.conf.bak  
##created by HQ at 2017
###rsyncd.conf start##
uid = rsync       --- 指定管理备份目录的用户  
gid = rsync       --- 指定管理备份目录的用户组
port = 873        --- 定义rsync备份服务的网络端口号
fake super = yes  --- 将rsync虚拟用户伪装成为一个超级管理员用户 
use chroot = no   --- 和安全相关的配置
max connections = 200  --- 最大连接数  同时只能有200个客户端连接到备份服务器
timeout = 300          --- 超时时间(单位秒)
pid file = /var/run/rsyncd.pid   --- 记录进程号码信息 1.让程序快速停止进程 2. 判断一个服务是否正在运行
lock file = /var/run/rsync.lock  --- 锁文件
log file = /var/log/rsyncd.log   --- rsync服务的日志文件 用于排错分析问题
ignore errors                    --- 忽略传输中的简单错误
read only = false                --- 指定备份目录是可读可写
list = false                     --- 使客户端可以查看服务端的模块信息
hosts allow = 172.16.1.0/24      --- 允许传输备份数据的主机(白名单)
hosts deny = 0.0.0.0/32          --- 禁止传输备份数据的主机(黑名单)
auth users = rsync_backup        --- 指定认证用户 
secrets file = /etc/rsync.password   --- 指定认证用户密码文件 用户名称:密码信息
[backup]                         --- 模块信息
comment = "backup dir by oldboy"  
path = /backup                   --- 模块中配置参数 指定备份目录
[root@lnmp ~]# cat /etc/rsyncd.conf
#created by HQ at 2021
####rsyncd.conf start##
uid = rsync    
gid = rsync   
port = 873   
fake super = yes
use chroot = no 
max connections = 200 
timeout = 300        
pid file = /var/run/rsyncd.pid 
lock file = /var/run/rsync.lock 
log file = /var/log/rsyncd.log 
ignore errors                 
read only = false            
list = false                
hosts allow = 192.168.1.0/24
hosts deny = 0.0.0.0/32   
auth users = rsync_backup
secrets file = /etc/rsync.password  
[backup]                        
comment = "backup dir by oldboy"  
path = /backup                  
查看进程是否启动
[root@lnmp ~]# systemctl status rsyncd
[root@lnmp ~]# ps -ef|grep rsync
root       1967      1  0 17:19 ?        00:00:00 /usr/bin/rsync --daemon --no-detach
root       1976   1312  0 17:20 pts/0    00:00:00 grep --color=auto rsync
[root@lnmp ~]# cat /var/run/rsyncd.pid 
1967  
[root@lnmp ~]#kill ` cat /var/run/rsyncd.pid `  杀死这个守护进程
第三个历程: 创建rsync服务的虚拟用户
[root@lnmp ~]# useradd rsync -M -s /sbin/nologin
[root@lnmp ~]# id rsync
第四个历程: 创建备份服务认证密码文件
[root@lnmp ~]# echo "rsync_backup:123456" >> /etc/rsync.password
[root@lnmp ~]# cat /etc/rsync.password 
rsync_backup:123456
[root@lnmp ~]# ll /etc/rsync.password 
-rw-r--r--. 1 root root 20 Jan 12 19:21 /etc/rsync.password
[root@lnmp ~]# chmod 600 /etc/rsync.password 
[root@lnmp ~]# ll /etc/rsync.password 
-rw-------. 1 root root 20 Jan 12 19:21 /etc/rsync.password
第五个历程: 创建备份目录并修改属主属组信息
[root@lnmp ~]# mkdir /backup
[root@lnmp ~]# ll -d /backup/
drwxr-xr-x. 2 root root 6 Jan 12 17:14 /backup/
[root@lnmp ~]# chown rsync.rsync /backup/
[root@lnmp ~]# ll -d /backup/
drwxr-xr-x. 2 rsync rsync 6 Jan 12 17:14 /backup/

第六个历程: 启动备份服务
systemctl start rsyncd
systemctl enable rsyncd
systemctl status rsyncd
[root@lnmp ~]# systemctl enable rsyncd
Created symlink from /etc/systemd/system/multi-user.target.wants/rsyncd.service to /usr/lib/systemd/system/rsyncd.service.
[root@lnmp ~]# systemctl start rsyncd
[root@lnmp ~]# systemctl status rsyncd

```

> 测试
>
> 需要熟悉rsync守护进程名称语法:
>
> Access via rsync daemon:
>
> 客户端做拉的操作: 恢复数据
>
> Pull: rsync [OPTION...] [USER@]HOST::SRC... [DEST]
>
>    rsync [OPTION...] rsync://[USER@]HOST[:PORT]/SRC... [DEST]
>
> 客户端做推的操作: 备份数据
>
> Push: rsync [OPTION...] SRC... [USER@]HOST::DEST
>
>    src: 要推送备份数据信息
>
> [USER@]: 指定认证用户信息
>
> ​      HOST: 指定远程主机的IP地址或者主机名称
>
> ​      ::DEST: 备份服务器的模块信息
>
> ​    rsync [OPTION...] SRC... rsync://[USER@]HOST[:PORT]/DEST

```bash
[root@lnmp ~]# rsync -avz /etc/hosts rsync_backup@192.168.1.199::backup
Password: 
sending incremental file list
hosts
sent 140 bytes  received 43 bytes  52.29 bytes/sec
total size is 158  speedup is 0.86
```

## **错误**

```bash
错误1：
[root@lnmp ~]# rsync -avz /etc/hosts rsync_backup@192.168.1.199::backup
Password: 
sending incremental file list
hosts
rsync: mkstemp ".hosts.fnUtMe" (in backup) failed: Permission denied (13)
//没有权限
sent 140 bytes  received 121 bytes  74.57 bytes/sec
total size is 158  speedup is 0.61
rsync error: some files/attrs were not transferred (see previous errors) (code 23) at main.c(1179) [sender=3.1.2]
原因
1、服务端/backup 所属用户不是rsync    chown -R rsync.rsync /backup
2、/backup  权限 chmod -R 755 /backup
3、selinux 没有关闭   
[root@lnmp /backup]# getenforce 
Enforcing
[root@lnmp /backup]# setenforce 0
[root@lnmp /backup]# getenforce 
Permissive
[root@lnmp /backup]# vim /etc/sysconfig/selinux 
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
#SELINUX=enforcing
SELINUX=disabled
# SELINUXTYPE= can take one of three values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are prot
ected. 
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
4、服务端/etc/rsyncd.conf   
uid = rsync
gid = rsync

错误2：
[root@lnmp ~]# rsync -avz /etc/hosts rsync_backuo@192.168.1.199::backup
Password: 
@ERROR: auth failed on module backup
rsync error: error starting client-server protocol (code 5) at main.c(1649) [sender=3.1.2]
因为/etc/rsync.passwd  权限不是600

错误3
[root@lnmp ~]# rsync -avz /etc/hosts rsync_backuo@192.168.1.199::backup
rsync: failed to connect to 192.168.1.199 (192.168.1.199): Connection refused (111)
rsync error: error in socket IO (code 10) at clientserver.c(125) [sender=3.1.2]
rsync服务没有启动
其他错误参考错误提示。

```

> 小结
>
> /etc/rsyncd.conf 文件中参数一定要正确，/etc/rsync.passwd文件中结尾注意不能有空格，可以使用cat -A /etc/rsync.passwd 查看$结尾没有空格才对

## 5、免交互配置（不输密码）

rsync守护进程客户端配置:

第一个历程: 创建一个秘密文件

```bash
[root@lnmp ~]# echo "oldboy123" >/etc/rsync.password
[root@lnmp ~]# chmod 600 /etc/rsync.password
```

第二个历程: 进行免交互传输数据测试

```bash
[root@lnmp ~]# rsync -avz /etc/hosts rsync_backup@192.168.1.199::backup --password-file=/etc/rsync.password
sending incremental file list
sent 42 bytes  received 20 bytes  124.00 bytes/sec
total size is 158  speedup is 2.55
```

搭配计划任务实现客户端定时向服务端推送

```bash
[root@lnmp ~]# crontab -e
#*/1 * * * * date >> /root/oldboy/date.log
*/1 * * * * rsync -avz /etc/hosts rsync_backup@192.168.1.199::backup --password-file=/etc/rsync.password
[root@lnmp ~]# systemctl restart crond.service
```

```bash
显示传输速率
[root@lnmp ~]# dd if=/dev/zero of=/tmp/1G bs=100M count=10
10+0 records in
10+0 records out
1048576000 bytes (1.0 GB) copied, 51.443 s, 20.4 MB/s
[root@lnmp ~]# rsync -avzP /tmp/1G rsync_backup@192.168.1.199::backup
Password: 
sending incremental file list
1G
  1,048,576,000 100%   25.20MB/s    0:00:39 (xfr#1, to-chk=0/1)

sent 1,019,827 bytes  received 43 bytes  20,195.45 bytes/sec
total size is 1,048,576,000  speedup is 1,028.15
[root@lnmp ~]# ll /backup/
total 1024004
-rw-r--r-- 1 rsync rsync 1048576000 Jan 14 09:28 1G
-rw-r--r-- 1 rsync rsync        158 Jun  7  2013 hosts

无差异化传输（慎用）
[root@lnmp /back]# touch {a..c}.txt
[root@lnmp /back]# ll
total 0
-rw-r--r-- 1 root root 0 Jan 14 09:32 a.txt
-rw-r--r-- 1 root root 0 Jan 14 09:32 b.txt
-rw-r--r-- 1 root root 0 Jan 14 09:32 c.txt
[root@lnmp /back]# rsync -avz --delete /back/ rsync_backup@192.168.1.199::backup
Password: 
sending incremental file list
deleting hosts
deleting 1G
./
a.txt
b.txt
c.txt

sent 215 bytes  received 99 bytes  69.78 bytes/sec
total size is 0  speedup is 0.00
You have new mail in /var/spool/mail/root
[root@lnmp /back]# rm -f b.txt 
[root@lnmp /back]# ll
total 0
-rw-r--r-- 1 root root 0 Jan 14 09:32 a.txt
-rw-r--r-- 1 root root 0 Jan 14 09:32 c.txt
[root@lnmp /back]# rsync -avz --delete /back/ rsync_backup@192.168.1.199::backup
Password: 
sending incremental file list
deleting hosts
deleting b.txt
./

sent 95 bytes  received 45 bytes  31.11 bytes/sec
total size is 0  speedup is 0.00
[root@lnmp /back]# rsync -avz --delete /back/ rsync_backup@192.168.1.199:/ 
可以删了系统。。。
```

```bash
守护进程服务企业应用:
a.	守护进程多模块功能配置
服务端
[root@lnmp /backup]# mkdir -p /{dba,dev_data}
[root@lnmp /backup]# chown -R rsync.rsync /{dba,dev_data}
[root@lnmp /backup]# ll -d /{dba,dev_data}
drwxr-xr-x  2 rsync rsync    6 Jan 13 20:07 /dba
drwxr-xr-x 19 rsync rsync 3220 Jan 13 12:46 /dev_data
[root@lnmp /backup]# vim /etc/rsyncd.conf
…
[backup]
comment = "backup dir by oldboy"
path = /backup
[dba]
comment = "backup dir by oldboy"
path = /dba
[dev]
comment = "backup dir by oldboy"
path = /dev_data
客户端
[root@lnmp /back]# touch {dev_data,dba}.txt
[root@lnmp /back]# ll
total 0
-rw-r--r-- 1 root root 0 Jan 14 09:32 a.txt
-rw-r--r-- 1 root root 0 Jan 14 09:32 c.txt
-rw-r--r-- 1 root root 0 Jan 14 10:40 dba.txt
-rw-r--r-- 1 root root 0 Jan 14 10:40 dev_data.txt
[root@lnmp /back]# rsync -avz dba.txt rsync_backup@192.168.1.199::dba --password-file=/etc/rsync.password
sending incremental file list
dba.txt

sent 89 bytes  received 43 bytes  264.00 bytes/sec
total size is 0  speedup is 0.00
[root@lnmp /back]# rsync -avz dev_data.txt rsync_backup@192.168.1.199::dev_data --password-file=/etc/rsync.password
sending incremental file list
dev_data.txt

sent 94 bytes  received 43 bytes  274.00 bytes/sec
total size is 0  speedup is 0.00
服务端
[root@lnmp /backup]# ll /{dev_data,dba}
/dba:
total 0 
-rw-r--r-- 1 rsync rsync 0 Jan 14 10:40 dba.txt

/dev_data:
total 0
-rw-r--r-- 1 rsync rsync 0 Jan 14 10:40 dev_data.txt


b. 守护进程的排除功能实践
准备环境:
[root@nfs01 /]# mkdir -p /oldboy
[root@nfs01 /]# mkdir -p /oldboy/{a..c}
[root@nfs01 /]# touch /oldboy/{a..c}/{1..3}.txt
[root@nfs01 /]# tree /oldboy
/oldboy
├── 01.txt
├── 02.txt
├── a
│   ├── 1.txt
│   ├── 2.txt
│   └── 3.txt
├── b
│   ├── 1.txt
│   ├── 2.txt
│   └── 3.txt
└── c
    ├── 1.txt
    ├── 2.txt
    └── 3.txt

需求01: 将/oldboy目录下面 a目录数据全部备份 b目录不要备份1.txt文件 c整个目录不要做备份
--exclude=PATTERN
绝对路径方式:
[root@lnmp /back]# rsync -avz /oldboy --exclude=/oldboy/b/1.txt --exclude=/oldboy/c/ --password-file=/etc/rsync.password rsync_backup@192.168.1.199::backup
sending incremental file list
oldboy/
oldboy/a/
oldboy/a/1.txt
oldboy/a/2.txt
oldboy/a/3.txt
oldboy/b/
oldboy/b/2.txt
oldboy/b/3.txt

sent 379 bytes  received 135 bytes  342.67 bytes/sec
total size is 0  speedup is 0.00
相对路径方式:
[root@lnmp /back]# rsync -avz /oldboy --exclude=b/1.txt --exclude=c/ --password-file=/etc/rsync.password rsync_backup@192.168.1.199::backup
sending incremental file list
oldboy/
oldboy/a/
oldboy/a/1.txt
oldboy/a/2.txt
oldboy/a/3.txt
oldboy/b/
oldboy/b/2.txt
oldboy/b/3.txt

sent 379 bytes  received 135 bytes  146.86 bytes/sec
total size is 0  speedup is 0.00

需求02: 将/oldboy目录下面 a目录数据全部备份 b目录不要备份1.txt文件 c整个目录1.txt 3.txt文件不要备份
--exclude-from=file  --- 批量排除 
第一个历程: 编辑好一个排除文件
[root@nfs01 /]# cat /back/exclude.txt 
b/1.txt
c/1.txt
c/3.txt
exclude.txt   如果exclude.txt在需要备份的目录里面也需要排除！

第二个历程: 实现批量排除功能
[root@lnmp /back]# rsync -avz /oldboy/ --exclude-from=/back/exclude.txt rsync_backup@192.168.1.199::backup --password-file=/etc/rsync.password
sending incremental file list
./
a/
a/1.txt
a/2.txt
a/3.txt
b/
b/2.txt
b/3.txt
c/
c/2.txt

sent 436 bytes  received 157 bytes  395.33 bytes/sec
total size is 0  speedup is 0.00



 守护进程来创建备份目录
[root@nfs01 /]# rsync -avz /etc/hosts  rsync_backup@172.16.1.41::backup/10.0.0.31/ 
sending incremental file list
created directory 10.0.0.31
hosts

sent 226 bytes  received 75 bytes  602.00 bytes/sec
total size is 371  speedup is 1.23
错误：
[root@lnmp /back]# rsync -avz /etc/hosts rsync_backup@192.168.1.199::backup/192.168.1.200/ --password-file=/etc/rsync.password
sending incremental file list
rsync: ERROR: cannot stat destination "192.168.1.200/" (in backup): Not a directory (20)
rsync error: errors selecting input/output files, dirs (code 3) at main.c(635) [Receiver=3.1.2]
因为服务端/backup目录  没有继承权限
以后设置服务备份目录使用  chmod -R 755 /backup
PS: 客户端无法在服务端创建多级目录

守护进程的访问控制配置
守护进程白名单和黑名单功能
PS: 建议只使用一种名单
```

守护进程的列表功能配置

```bash
[root@nfs01 /]# rsync rsync_backup@172.16.1.41::
backup         	 "backup dir by oldboy"
dba            	 "backup dir by oldboy"
dev            	 "backup dir by oldboy"
```

# 全网备份数据项目

具体要求：

1、 所有服务器的备份目录必须都为/backup

2、 要备份的系统配置文件包括但不限于：

A、 定时任务服务的配置文件(/var/spool/cron/root)适合于web和nfs服务器

B、 开机自启动的配置文件(/etc/rc.local)适合于web和nfs服务器

C、 日常脚本的目录/server/scripts

D、 防火墙iptables的配置文件(/etc/sysconfig/iptables)

E、 其他还有什么需要备份呢？

3、 web服务器站点目录(/var/html/www)

4、 web服务器A访问日志路径(/app/logs)

5、 web服务器保留打包后的7天的备份数据即可（本地不能保存多于7天，太多服务器硬盘会满）

6、 备份服务器上，保留每周一的所有数据副本，其他要保留6个月的数据副本。（也就是180天前的，只保留周一的数据，周一的数据在23：59分才是哦，180天内的必须保留全部副本）

0 0 * * 2  tar  $(date +%F_%A_%w)每周二的0时0分执行tar备份。

0 0 * * 0,1,3,4,5,6 tar 每天备份

0 0 * * *  find /back -name “*.tar.gz” -maxdepth 1 -path '*._monday' -prune -o -print  -mtime +180 -exec rm -rf {} \;

0 0 * * *  find /back -type f -mtime +180 **! -name “\*.Monday.\*”**-exec rm -rf {} \;

7、 备份服务器上要按照备份数据服务器的内网IP为目录保存备份，备份的文件按照实践命名保存

8、 需要确保备份的数据尽量完整正确，在备份服务器上对备份数据进行检查MD5，把备份的成功与失败结果信息发给系统管理员邮箱中。

## 服务器info一览表

| 名称   | IP地址                                               | 服务              | 文件路径                                                     | 备注 |
| ------ | ---------------------------------------------------- | ----------------- | ------------------------------------------------------------ | ---- |
| web1   | 内网地址：192.168.1.191     外网地址：  172.16.1.180 | nginx  mysql  php | /var/html/www  /var/html/  /application/nginx/conf/nginx.conf  /application/nginx/conf/extra/  /application/nginx/logs  /application/mysql/data/  /application/php/php.ini  /var/spool/cron/root  /etc/rc.local  /server/scripts  /etc/sysconfig/iptables  /backup |      |
| web2   | 内网地址：192.168.1.192     外网地址：  172.16.1.181 | nginx  mysql  php | /var/html/www  /var/html/  /application/nginx/conf/nginx.conf  /application/nginx/conf/extra/  /application/nginx/logs  /application/mysql/data/  /application/php/php.ini  /var/spool/cron/root  /etc/rc.local  /server/scripts  /etc/sysconfig/iptables  /backup |      |
| nfs1   | 内网地址：192.168.1.195                              | nfs               | /etc/exports  /nfs_data  /var/spool/cron/root  /etc/rc.local  /server/scripts  /etc/sysconfig/iptables  /backup |      |
| nfs2   | 内网地址：192.168.1.196                              | nfs               | /etc/exports  /nfs_data  /var/spool/cron/root  /etc/rc.local  /server/scripts  /etc/sysconfig/iptables  /backup |      |
| rsync1 | 内网地址：192.168.1.199                              | rsync             | /backup  /backup/192.168.1.191  /backup/192.168.1.192  /backup/192.168.1.195  /backup/192.168.1.196  /var/spool/cron/root |      |
| rsync2 | 内网地址：192.168.1.200                              | rsync             | /backup  /backup/192.168.1.191  /backup/192.168.1.192  /backup/192.168.1.195  /backup/192.168.1.196  /var/spool/cron/root |      |

 

先在本地服务器上面备份，然后再备份到rsync1服务器。Web空余时间推送，本地/backup 文件不超过7天，备份到rsync服务器上保留180天，180天以前的只留周一的数据。rsync2再拉取或者rsync1再推送所有备份数据。

## 操作步骤：

1、 首先确定所需备份文件信息

详见以上一览表

2、 安装rsync服务端和客户端

```bash
#rpm -qa |grep rsync 
#yum install -y rsync (如果没有直接yum安装，二进制安装和源码编译此处省略)
```

服务端：rsync1,rsync2

客户端：web1，web2,nfs1,nfs2

 

服务器端配置如下：

```bash
#/etc/rsyncd.conf
#created by HQ at 2021
####rsyncd.conf start##
uid = rsync    
gid = rsync   
port = 873   
fake super = yes
use chroot = no 
max connections = 200 
timeout = 300        
pid file = /var/run/rsyncd.pid 
lock file = /var/run/rsync.lock 
log file = /var/log/rsyncd.log 
ignore errors                 
read only = false            
list = false                
hosts allow = 192.168.1.0/24
#hosts deny = 0.0.0.0/32   
auth users = rsync_backup
secrets file = /etc/rsync.password  
[backup]                        
comment = "backup dir by rsync1"(第二台rsync2)  
path = /backup                  
```

客户端

```bash
#rsync /etc/hosts /backup/local-hosts-$(date +%F)    （备份到本地）
#echo “123456” >> /etc/rsync.password   （方便后期免密备份）

[root@lnmp /backup]# mkdir /back
[root@lnmp /backup]# tar cvzPf /back/${h}_$(date +%F).tar.gz /etc/hosts
/etc/hosts
[root@lnmp /backup]# ll /back
total 4
-rw-r--r-- 1 root root 170 Jan 15 12:14 hosts_2021-01-15.tar.gz
[root@lnmp /backup]# rsync -avz /back /backup/
sending incremental file list
back/
back/hosts_2021-01-15.tar.gz

sent 306 bytes  received 39 bytes  690.00 bytes/sec
total size is 170  speedup is 0.49
[root@lnmp /backup]# ll 
total 0
drwxr-xr-x 2 root root 37 Jan 15 12:14 back
[root@lnmp /backup]# rsync -avz /back/ /backup/
sending incremental file list
./
hosts_2021-01-15.tar.gz

sent 293 bytes  received 38 bytes  662.00 bytes/sec
total size is 170  speedup is 0.51
[root@lnmp /backup]# tree
.
├── back
│   └── hosts_2021-01-15.tar.gz
└── hosts_2021-01-15.tar.gz
1 directory, 2 files
#注意：tar 要加h参数，有些文件是快捷方式，备份过去不能还原。
tar cvzhPf /back/${h}_$(date +%F).tar.gz /etc/hosts
/etc/hosts
```

### 被玩坏的for

使用for来完成批量创建文件和查看文件

```bash
[root@lnmp ~]# for line in `cat 1.txt`
 do
 echo $line     其实可以不要
 mkdir -p $line
 done    
 
/var/html/www
/var/html/
/application/nginx/conf/nginx.conf
/application/nginx/conf/extra/
/application/nginx/logs
/application/mysql/data/
/application/php/php.ini
/var/spool/cron/root
/etc/rc.local
/server/scripts
/etc/sysconfig/iptables
/backup
[root@lnmp ~]# for line in `cat 1.txt`; do  ll -d $line; done    其实可以合并到上面
drwxr-xr-x 2 root root 6 Jan 16 16:21 /var/html/www
drwxr-xr-x 3 root root 17 Jan 16 16:21 /var/html/
drwxr-xr-x 2 root root 6 Jan 16 16:21 /application/nginx/conf/nginx.conf
drwxr-xr-x 2 root root 6 Jan 16 16:21 /application/nginx/conf/extra/
drwxr-xr-x 2 root root 6 Jan 16 16:21 /application/nginx/logs
drwxr-xr-x 2 root root 6 Jan 16 16:21 /application/mysql/data/
drwxr-xr-x 2 root root 6 Jan 16 16:21 /application/php/php.ini
-rw------- 1 root root 148 Jan 13 16:44 /var/spool/cron/root
lrwxrwxrwx. 1 root root 13 Jun 13  2020 /etc/rc.local -> rc.d/rc.local
drwxr-xr-x 2 root root 6 Jan 16 16:21 /server/scripts
drwxr-xr-x 2 root root 6 Jan 16 16:21 /etc/sysconfig/iptables
drwxr-xr-x 3 root root 76 Jan 15 12:14 /backup

# for line in `cat 1.txt`; do  mkdir -p $line ;ll -d $line; done   应该是可以这样写的。
[root@lnmp ~]# for line in `cat 1.txt`; do tar cvzPf /back/$(basename $line)_$(date +%F).tar.gz $line; done    打包指定目录并按时间戳命令。
[root@lnmp ~]# ll /back
total 52
-rw-r--r-- 1 root root 444 Jan 16 16:40 backup_2021-01-16.tar.gz
-rw-r--r-- 1 root root 125 Jan 16 16:40 data_2021-01-16.tar.gz
-rw-r--r-- 1 root root 131 Jan 16 16:40 extra_2021-01-16.tar.gz
-rw-r--r-- 1 root root 170 Jan 15 12:14 hosts_2021-01-15.tar.gz
-rw-r--r-- 1 root root 134 Jan 16 16:40 html_2021-01-16.tar.gz
-rw-r--r-- 1 root root 126 Jan 16 16:40 iptables_2021-01-16.tar.gz
-rw-r--r-- 1 root root 125 Jan 16 16:40 logs_2021-01-16.tar.gz
-rw-r--r-- 1 root root 128 Jan 16 16:40 nginx.conf_2021-01-16.tar.gz
-rw-r--r-- 1 root root 124 Jan 16 16:40 php.ini_2021-01-16.tar.gz
-rw-r--r-- 1 root root 118 Jan 16 16:40 rc.local_2021-01-16.tar.gz
-rw-r--r-- 1 root root 230 Jan 16 16:40 root_2021-01-16.tar.gz
-rw-r--r-- 1 root root 118 Jan 16 16:40 scripts_2021-01-16.tar.gz
-rw-r--r-- 1 root root 117 Jan 16 16:40 www_2021-01-16.tar.gz

生成MD5
[root@lnmp /back]# find /back -name "*.gz"|sort |xargs md5sum >>md5num.md5   把打包好的gz文件生成md5码。
[root@lnmp /back]# ll
total 56
-rw-r--r-- 1 root root 444 Jan 16 16:54 backup_2021-01-16.tar.gz
-rw-r--r-- 1 root root 125 Jan 16 16:54 data_2021-01-16.tar.gz
-rw-r--r-- 1 root root 131 Jan 16 16:54 extra_2021-01-16.tar.gz
-rw-r--r-- 1 root root 170 Jan 15 12:14 hosts_2021-01-15.tar.gz
-rw-r--r-- 1 root root 166 Jan 16 16:54 html_2021-01-16.tar.gz
-rw-r--r-- 1 root root 126 Jan 16 16:54 iptables_2021-01-16.tar.gz
-rw-r--r-- 1 root root 125 Jan 16 16:54 logs_2021-01-16.tar.gz
-rw-r--r-- 1 root root 842 Jan 16 19:15 md5num.md5
-rw-r--r-- 1 root root 128 Jan 16 16:54 nginx.conf_2021-01-16.tar.gz
-rw-r--r-- 1 root root 124 Jan 16 16:54 php.ini_2021-01-16.tar.gz
-rw-r--r-- 1 root root 118 Jan 16 16:54 rc.local_2021-01-16.tar.gz
-rw-r--r-- 1 root root 230 Jan 16 16:54 root_2021-01-16.tar.gz
-rw-r--r-- 1 root root 118 Jan 16 16:54 scripts_2021-01-16.tar.gz
-rw-r--r-- 1 root root 117 Jan 16 16:54 www_2021-01-16.tar.gz
[root@lnmp /back]# cat md5num.md5 
87b968d431f117460c1061c9aa7703e6  /back/backup_2021-01-16.tar.gz
aef6362cc36b55b2afa2db7aa75d20e0  /back/data_2021-01-16.tar.gz
38984f3ca2a2015288c765f2066d3a69  /back/extra_2021-01-16.tar.gz
48fec8afab0e63d6fbefef3b9f04af9f  /back/hosts_2021-01-15.tar.gz
adb72d2fd8dba4ada8ea0d8abac36c41  /back/html_2021-01-16.tar.gz
5a853d0895936ffeb186057a53ac9b87  /back/iptables_2021-01-16.tar.gz
c0e340edf20e392c13ace1e557ce6dd8  /back/logs_2021-01-16.tar.gz
4a2b4787a5a562c4d0bb3971419469e3  /back/nginx.conf_2021-01-16.tar.gz
96602d1b16d515d09809397d72e9e258  /back/php.ini_2021-01-16.tar.gz
ffed9f84d506db43a64d8db9b2d2a075  /back/rc.local_2021-01-16.tar.gz
0ae71551f9e2d01da47380ec5404f139  /back/root_2021-01-16.tar.gz
b08cca2de5d9be44afbe128416c0e734  /back/scripts_2021-01-16.tar.gz
50cb4277fa7da6e6129753f76bfa0ee2  /back/www_2021-01-16.tar.gz
[root@lnmp /backup]# rsync -avz /back/* /backup/$(date +%F)  把备份过来的文件以时间戳命令的文件夹
sending incremental file list
created directory /backup/2021-01-16
backup_2021-01-16.tar.gz
data_2021-01-16.tar.gz
extra_2021-01-16.tar.gz
hosts_2021-01-15.tar.gz
html_2021-01-16.tar.gz
iptables_2021-01-16.tar.gz
logs_2021-01-16.tar.gz
md5num.md5
nginx.conf_2021-01-16.tar.gz
php.ini_2021-01-16.tar.gz
rc.local_2021-01-16.tar.gz
root_2021-01-16.tar.gz
scripts_2021-01-16.tar.gz
www_2021-01-16.tar.gz
sent 3,560 bytes  received 323 bytes  7,766.00 bytes/sec
total size is 2,964  speedup is 0.76
[root@lnmp /backup]# find /backup/$(date +%F) -name "*.gz"|sort |xargs md5sum >>/backup/$(date +%F)/copy_md5.md5
查找并生成当天md5码
[root@lnmp /backup/2021-01-16]# vimdiff md5num.md5 copy_md5.md5
```

![](linux运维05Rsync.assets/image-20210818151840588-16510242817681.png)

```bash
[root@lnmp /backup/2021-01-16]# md5sum -c md5num.md5   检查结果
/back/backup_2021-01-16.tar.gz: OK
/back/data_2021-01-16.tar.gz: OK
/back/extra_2021-01-16.tar.gz: OK
/back/hosts_2021-01-15.tar.gz: OK
/back/html_2021-01-16.tar.gz: OK
/back/iptables_2021-01-16.tar.gz: OK
/back/logs_2021-01-16.tar.gz: OK
/back/nginx.conf_2021-01-16.tar.gz: OK
/back/php.ini_2021-01-16.tar.gz: OK
/back/rc.local_2021-01-16.tar.gz: OK
/back/root_2021-01-16.tar.gz: OK
/back/scripts_2021-01-16.tar.gz: OK
/back/www_2021-01-16.tar.gz: OK
[root@lnmp /back]# find /back/ -type d -mtime +7 -exec rm -rf {} \;
 查找并删除7天以前的文件夹。
```

> $(date +%F _week%w) 显示周几的信息
>
> [root@lnmp ~]# date +%F_%A_%w
>
> 2021-01-21_Thursday_4
>
> 生成的MD5一起传输过来，直接使用md5sum -c 验证这个文件即可。
>
> 遇到多个md5文件检查可以使用
>
> find /backup -type f -name “md5.txt” |xargs md5sum -c 

### 告警邮件发送

如何发送邮件

```bash
[root@lnmp ~]# yum install sendmail
[root@lnmp ~]# yum install mailx
[root@lnmp ~]# vi /etc/mail.rc
set from=wadshong5220@126.com
set smtp=smtp.126.com
set smtp-auth-user=wadshong5220@126.com
set smtp-auth-password=***********     #这里是设置密码
set smtp-auth=login
```

![image-20210818151946796](linux运维05RSYNC.assets\image-20210818151946796.png)

![image-20210818151954082](linux运维05RSYNC.assets\image-20210818151954082.png)

```bash
[root@lnmp ~]# vi test.txt
how are you!
[root@lnmp ~]# mail -s "test" 349925756@qq.com < /root/test.txt
```

![image-20210818152016938](linux运维05RSYNC.assets\image-20210818152016938.png)

```bash
[root@lnmp ~]# ll /backup/ > mail.txt
[root@lnmp ~]# mail -s "ll" 349925756@qq.com < /root/mail.txt
[root@mysql03 ~]# ll |mail -s "ls -l" 349925756@qq.com
```

![image-20210818152033221](linux运维05RSYNC.assets\image-20210818152033221.png)

### 编写脚本

准备工作

客户端：

```bash
[root@lnmp ~]# echo "192.168.1.199 lnmp" >> /etc/hosts
[root@lnmp ~]# hostname -i
192.168.1.199
[root@lnmp ~]# mkdir /server/scripts -p
[root@lnmp ~]# cat /server/scripts/backup_file.txt  把需要备份的文件路径添加到一个文件中，后期增删改查都很方便。
/var/html/www
/var/html/
/application/nginx/conf/nginx.conf
/application/nginx/conf/extra/
/application/nginx/logs
/application/mysql/data/
/application/php/php.ini
/var/spool/cron/root
/etc/rc.local
/server/scripts
/etc/sysconfig/iptables
/backup
[root@lnmp ~]#for line in `cat /server/scripts/backup_file.txt`; do  mkdir $line -p; done  创建模拟文件

客户端脚本
[root@lnmp /server/scripts]# vim backup_date.sh 
#!/bin/bash
#var
hosname=$(hostname -i)
dat=$(date +%F_%A)
backup_dir="/backup/$hosname/$dat"
mkdir -p $backup_dir


#read backup data file
backup_date_file=`cat /server/scripts/backup_file.txt`

#mkdir -p /backup/$hosname/$(date +%F_%A)
#creat tar name 
for line in $backup_date_file
do
tar cvhzPf $backup_dir/$(basename $line)——$dat.tar.gz $line

#del 7 day ago data
#find $backup_dir -type d -mtime +7|xargs rm 2>/dev/null  这里路径搜不到文件夹就导致无法删除。
find /backup/$hosname -type d -mtime +7 |xargs rm -rf 2>/dev/null
done

#creat MD5
find $backup_dir -type f -name "*.tar.gz" |xargs md5sum > $backup_dir/check_$(date +%F_%A).md5
#rsync
rsync -az /backup/ rsync_backup@192.168.1.199::backup --password-file=/etc/rsync.password

定时任务
[root@lnmp /server/scripts]# crontab -e
#backup data
0 0 * * * /bin/sh /server/scripts/backup_date.sh &>/dev/null
[root@lnmp /server/scripts]# systemctl restart crond.service

服务端
[root@lnmp /server/scripts]# vim backup_check.sh 
#!/bin/bash
backup_dir="/backup"
file_dir=$(ls -I "*.txt" $backup_dir >$backup_dir/file_name.txt)
dat=$(date +%F_%A)
#del 180 day ago data
for line in `cat $backup_dir/file_name.txt`
do
find $backup_dir/$line -type d ! -name "*_Monday" -mtime +180|xargs rm -rf 2>/dev/null

#check data
find $backup_dir/$line/$dat -type f -name "*.md5" |xargs md5sum -c >$backup_dir/md5sum.txt 2>/dev/null

done

mail -s "backup data chenck" 349925756@qq.com < $backup_dir/md5sum.txt
```

![image-20210818152127484](linux运维05RSYNC.assets\image-20210818152127484.png)

![image-20210818152136903](linux运维05RSYNC.assets\image-20210818152136903.png)

添加到计划任务

```bash
[root@lnmp /server/scripts]# crontab -e
#backup data
0 5 * * * /bin/sh /server/scripts/backup_check.sh &>/dev/null
```

小结：到此为止自动化全网备份项目就完成了。以上脚本可以通用。

服务器

```bash
[root@lnmp /server/scripts]# tree -L 2 /backup/
/backup/
├── 192.168.1.200
│   ├── 2021-01-25_Monday
│   ├── 2021-01-27_Wednesday
│   ├── 2021-01-28_Thursday
│   ├── 2021-01-29_Friday
│   ├── 2021-01-30_Saturday
│   ├── 2021-01-31_Sunday
│   └── 2021-02-01_Monday
客户端
[root@lnmp ~]# tree -L 2 /backup/
/backup/
└── 192.168.1.200
    ├── 2021-01-24_Sunday
    ├── 2021-01-25_Monday
    ├── 2021-01-26_Tuesday
    ├── 2021-01-27_Wednesday
    ├── 2021-01-28_Thursday
    ├── 2021-01-29_Friday
    ├── 2021-01-30_Saturday
    ├── 2021-01-31_Sunday
    └── 2021-02-01_Monday

10 directories, 0 files
[root@lnmp ~]# tree /backup/
/backup/
└── 192.168.1.200
    ├── 2021-01-23_Saturday
    │   ├── check_2021-01-23_Saturday.md5
    │   ├── data_2021-01-23_Saturday.tar.gz
    │   ├── extra_2021-01-23_Saturday.tar.gz
    │   ├── html_2021-01-23_Saturday.tar.gz
    │   ├── iptables_2021-01-23_Saturday.tar.gz
    │   ├── logs_2021-01-23_Saturday.tar.gz
    │   ├── nginx.conf_2021-01-23_Saturday.tar.gz
    │   ├── php.ini_2021-01-23_Saturday.tar.gz
    │   ├── rc.local_2021-01-23_Saturday.tar.gz
    │   ├── root_2021-01-23_Saturday.tar.gz
    │   ├── scripts_2021-01-23_Saturday.tar.gz
│   └── www_2021-01-23_Saturday.tar.gz
```

从上就可以看出备份文件的日期时间，归档到文件夹方便后期查找使用。

## 存储服务

分布式存储：

moosefs(mfs) 年代久远

GlusterFS 软件即存储

FastDFS    企业常用

存储是什么？存储提供什么样的服务，linux下存储服务有哪些

| 名称  | 功能                       |
| ----- | -------------------------- |
| ftp   | 提供少量用户数据存储和共享 |
| ssh   | SFTP                       |
| samba | windows和linux之间共享数据 |
| NFS   | linux-linux 提供共享数据   |

# NFS

实现数据共享、编写数据操作管理、节约服务器开销。

## 服务器端部署：

第一步：安装NFS服务

```bash
[root@lnmp ~]# rpm -qa |grep nfs
[root@lnmp ~]# rpm -qa |grep rpcbind
[root@lnmp ~]# rpm -qa|grep -E "nfs|rpc"
[root@lnmp ~]# yum search nfs rpcbind
[root@lnmp ~]# yum install -y nfs-utils rpcbind
[root@lnmp ~]# rpm -qa|grep -E "nfs|rpc"
nfs-utils-1.3.0-0.68.el7.x86_64
libtirpc-0.2.4-0.16.el7.x86_64
libnfsidmap-0.25-19.el7.x86_64
rpcbind-0.2.0-49.el7.x86_64
```

第二步：配置nfs服务

设置数据存储目录

设置访问白名单

配置存储权限

```bash
[root@lnmp ~]#man exports 获取帮助信息
[root@lnmp ~]# vim /etc/exports
/data    192.168.1.0/24(rw,sync)
```

第三步：创建存储路径

```bash
[root@lnmp ~]# mkdir /data
[root@lnmp ~]# ll -d /data/
drwxr-xr-x 2 root root 6 Jan 26 19:38 /data/
[root@lnmp ~]# chown nfsnobody.nfsnobody /data/
[root@lnmp ~]# ll -d /data/
drwxr-xr-x 2 nfsnobody nfsnobody 6 Jan 26 19:38 /data
```

第四步：启动服务及相关依赖服务

```bash
[root@lnmp ~]# systemctl start rpcbind.service 
[root@lnmp ~]# systemctl enable rpcbind.service 
[root@lnmp ~]# systemctl start nfs
[root@lnmp ~]# systemctl enable nfs
Created symlink from /etc/systemd/system/multi-user.target.wants/nfs-server.service to /usr/lib/systemd/system/nfs-server.service.
```

## 客户端部署：

第一步：安装NFS服务

```bash
[root@lnmp ~]# yum -y install nfs-utils
[root@lnmp ~]# rpm -qa |grep nfs
nfs-utils-1.3.0-0.68.el7.x86_64
libnfsidmap-0.25-19.el7.x86_64
```

第二步：挂载存储目录

```bash
[root@lnmp ~]# mount -t nfs 192.168.1.199:/data /mnt
[root@lnmp ~]# df -h
Filesystem           Size  Used Avail Use% Mounted on
devtmpfs             900M     0  900M   0% /dev
tmpfs                910M     0  910M   0% /dev/shm
tmpfs                910M  9.5M  901M   2% /run
tmpfs                910M     0  910M   0% /sys/fs/cgroup
/dev/sda2             56G  2.0G   54G   4% /
/dev/sda1            509M  174M  335M  35% /boot
tmpfs                182M     0  182M   0% /run/user/0
192.168.1.199:/data   56G  2.0G   54G   4% /mnt
```

NFS工作依赖RPC 111端口

```bash
[root@lnmp ~]# rpcinfo
[root@lnmp ~]# rpcinfo 192.168.1.199
```

## NFS服务端详细说明

NFS服务端详细说明

多网段主机挂载

第一种方法：

```bash
/data   172.16.1.0/24(rw,sync) 10.0.0.0/24(rw,sync)
```

第二种方法：

```bash
/data   172.16.1.0/24(rw,sync) 
/data   10.0.0.0/24(rw,sync)
```

NFS共享目录的权限和那些因素有关系

1、 和共享存储本身权限有关(755 属主nfsnobody)

```bash
[root@lnmp ~]# id nfsnobody
uid=65534(nfsnobody) gid=65534(nfsnobody) groups=65534(nfsnobody)
[root@lnmp ~]# ll -d /data/
drwxr-xr-x 2 nfsnobody nfsnobody 6 Jan 26 19:38 /data/
```

2、 /etc/exports 配置有关

```bash
rw/ro   xxx_squash    anonuid/anongid
```

3、 和客户端挂载命令有关

```bash
mount -t nfs rw 192.168.1.199:/data /mnt
```

### NFS配置参数详解

**rw**        --存储目录是否有读写权限

**ro**         --存储目录是否是只读权限

**sync**        --同步方式存储数据，直接将数据存储到硬盘（数据安全性高，并发IO会很慢）

**async**        --异步方式存储数据，数据直接存储到内存（并发IO很高，效率高。数据安全性堪忧，突然断电数据丢）

**no_root_squash** --不要将root身份进行转换nfsnobody

**root_squash**   --将root身份进行转换

all_squash    --将所有用户身份转换

**no_all_squash**  --不将所有用户身份转换

```bash
[root@lnmp ~]# vim /etc/exports
/data    192.168.1.0/24(rw,sync,root_squash)
[root@lnmp ~]# mount -t nfs 192.168.1.199:/data /mnt
[root@lnmp ~]# cd /mnt/
[root@lnmp /mnt]# touch test.txt
[root@lnmp /mnt]# ll
total 0
-rw-r--r-- 1 nfsnobody nfsnobody 0 Jan 27 15:45 test.txt
把root转换成nfsnobody了
[root@lnmp ~]# vim /etc/exports
/data    192.168.1.0/24(rw,sync,no_root_squash)
[root@lnmp /mnt]# touch test1.txt
[root@lnmp /mnt]# ll
total 0
-rw-r--r-- 1 root      root      0 Jan 27 15:46 test1.txt
-rw-r--r-- 1 nfsnobody nfsnobody 0 Jan 27 15:45 test.txt
不转换root用户

[root@lnmp ~]# vim /etc/exports
/data    192.168.1.0/24(rw,sync,all_squash)
[root@lnmp /mnt]# useradd test 
[root@lnmp /mnt]# passwd test
Changing password for user test.
New password: 
BAD PASSWORD: The password is shorter than 8 characters
Retype new password: 
passwd: all authentication tokens updated successfully.
[test@lnmp ~]$ df -h
Filesystem           Size  Used Avail Use% Mounted on
devtmpfs             900M     0  900M   0% /dev
tmpfs                910M  9.5M  901M   2% /run
tmpfs                910M     0  910M   0% /sys/fs/cgroup
/dev/sda2             56G  2.0G   54G   4% /
192.168.1.199:/data   56G  2.0G   54G   4% /mnt
[test@lnmp ~]$ cd /mnt/
[test@lnmp /mnt]$ touch test2.txt
[test@lnmp /mnt]$ ll
total 0
-rw-r--r-- 1 root      root      0 Jan 27 15:46 test1.txt
-rw-rw-r-- 1 nfsnobody nfsnobody 0 Jan 27 18:55 test2.txt
-rw-r--r-- 1 nfsnobody nfsnobody 0 Jan 27 15:45 test.txt

[root@lnmp ~]# vim /etc/exports
/data    192.168.1.0/24(rw,sync,no_all_squash)
[test@lnmp /mnt]$ touch test3.txt
touch: cannot touch ‘test3.txt’: Permission denied
[test@lnmp /mnt]$ ll -d /mnt
drwxr-xr-x 2 nfsnobody nfsnobody 56 Jan 27 18:55 /mnt
[test@lnmp /mnt]$ rm -rf test.txt 
rm: cannot remove ‘test.txt’: Permission denied
可以看出no_all_squash 普通用户是不能写入，删除数据的
要想普通用户也可以写入nfs目录权限加上写入权限，这样做很不安全，不建议
chmod o+w /data/
```

>  保证网站存储服务器用户数据安全性：
>
> ​     no_all_squash 需要进行配置  共享目录权限为www（确保客户端用户 服务端用户 uid数值一致）
>
>    root_squash  需要进行配置  root---nfsnobody  data目录---www
>
> ​     以上默认配置（很多服务默认配置都是从安全角度出发）
>
> ​     如何查看nfs默认配置
>
>    cat /var/lib/nfs/etab  --- 记录nfs服务的默认配置记录信息
>
> /data 172.16.1.0/24(rw,sync,wdelay,hide,nocrossmnt,secure,root_squash,no_all_squash,no_subtree_check,secure_locks,acl,no_pnfs,anonuid=65534,anongid=65534,sec=sys,rw,secure,root_squash,no_all_squash)
>
> 修改映射用户：www=1002
>
>    /data  172.16.1.0/24(rw,sync,anonuid=1002,anongid=1002)
>
>  
>
> 企业中如何编辑nfs配置文件
>
>    01. 通用方法 *****
>
>    /data  172.16.1.0/24(rw,sync)
>
> ​     02. 特殊情况 （让部分人员不能操作存储目录 可以看目录中的数据）
>
>    /data  10.0.0.0/24(ro,sync)
>
> ​     03. 修改默认的匿名用户
>
>    /data  10.0.0.0/24(ro,sync,anonuid=xxx,anongid=xxx)

### 如何实现自动挂载

   01. 利用rc.local

```sh
   echo "mount -t nfs 172.16.1.31:/data /mnt" **>>**/etc/rc.local
```

   rc.local是连接文件，初始系统需要添加权限，否则不执行

```sh
chmod +x /etc/rc.d/rc.local
[test@lnmp /mnt]$ ll /etc/rc.local 
lrwxrwxrwx. 1 root root 13 Jun 13  2020 /etc/rc.local -> rc.d/rc.local
[test@lnmp /mnt]$ ll /etc/rc.d/rc.local 
-rw-r--r--. 1 root root 473 May 12  2020 /etc/rc.d/rc.local 注意这里权限是不执行
```

  2. 利用fstab文件

```bash
 vim /etc/fstab
	 172.16.1.31:/data             /mnt               nfs     defaults        0 0
[root@lnmp ~]# cat /etc/fstab
```

文件也不一定成功.linux启动顺序。fstab 优先于network那么网络不通的情况下上面是否能挂载？不能

```bash
172.16.1.31:/data             /mnt               nfs     auto       0 0
```

![image-20210818153546577](linux运维05RSYNC.assets\image-20210818153546577.png)

### 客户端mount命令参数

    rw  --- 实现挂载后挂载点目录可读可写 （默认）

​     ro  --- 实现挂载后挂载点目录可读可写

   suid --- 在共享目录中可以让setuid权限位生效 （默认）

 nosuid --- 在共享目录中可以让setuid权限位失效  提供共享目录的安全性

​     exec --- 共享目录中的执行文件可以直接执行

 noexec --- 共享目录中的执行文件可以无法直接执行 提供共享目录的安全性

   auto --- 可以实现自动挂载   mount -a 实现加载fstab文件自动挂载

 noauto --- 不可以实现自动挂载

 nouser --- 禁止普通用户可以卸载挂载点

  user --- 允许普通用户可以卸载挂载点

### 客户端如何卸载

>  umount -lf /mnt  --- 强制卸载挂载点
>
> ​     -l 不退出挂载点目录进行卸载 
>
> ​     -f 强制进行卸载操作
>
> showmount -e 172.16.1.31 检查服务提供的服务目录

### 查看那些服务开机自启

```bash
ll /etc/systemd/system/multi-user.target.wants/
```

![image-20210818153656079](linux运维05RSYNC.assets\image-20210818153656079.png)



# 综合架构实时同步

上面介绍了rsync计划同步，如果当天发生故障那么还是有很多数据会丢失，怎么做才能实时同步数据呢？

这里使用sersync，sersync主要用于服务器同步，web镜像等功能。基于boost1.43.0,inotify api,rsync command.开发。目前使用的比较多的同步解决方案是**inotify-tools+rsync** ，另外一个是google开源项目Openduckbill（依赖于inotify- tools），这两个都是基于脚本语言编写的。相比较上面两个项目

sersync优点是：

sersync是使用c++编写，结合rsync同步的时候，节省了运行时耗和网络资源。因此更快。

相比较上面两个项目，sersync配置起来很简单，其中bin目录下已经有基本上静态编译的2进制文件，配合bin目录下的xml配置文件直接使用即可。

另外sersync项目相比较其他脚本开源项目，使用多线程进行同步，尤其在同步较大文件时，能够保证多个服务器实时保持同步状态。

sersync项目有出错处理机制，通过失败队列对出错的文件重新同步，如果仍旧失败，则按设定时长对同步失败的文件重新同步。

sersync项目自带crontab功能，只需在xml配置文件中开启，即可按您的要求，隔一段时间整体同步一次。无需再额外配置crontab功能。

sersync项目socket与http插件扩展，满足您二次开发的需要。

针对上图的设计架构，这里做几点说明，来帮助大家阅读和理解该图

1 ) 线程组线程是等待线程队列的守护线程，当事件队列中有事件产生的时候，线程组守护线程就会逐个唤醒同步线程。当队列中 Inotify 事件较多的时候，同步线程就会被全部唤醒一起工作。这样设计的目的是为了能够同时处理多个 Inotify 事件，从而提升服务器的并发同步能力。同步线程的最佳数量=核数 x 2 + 2。 2 ) 那么之所以称之为线程组线程，是因为每个线程在工作的时候，会根据服务器上新写入文件的数量去建立子线程，子线程可以保证所有的文件与各个服务器同时同步。当要同步的文件较大的时候，这样的设计可以保证每个远程服务器都可以同时获得需要同步的文件。 3 ) 服务线程的作用有三个：

处理同步失败的文件，将这些文件再次同步，对于再次同步失败的文件会生成 rsync_fail_log.sh 脚本，记录失败的事件。

每隔10个小时执行 rsync_fail_log.sh 脚本一次，同时清空脚本。(第二次失败删除rsync_fail_log.sh那么之前失败的也就不能继续了，这个是很棘手的错误。)

crontab功能，可以每隔一定时间，将所有路径整体同步一次。

4 ) 过滤队列的建立是为了过滤短时间内产生的重复的inotify信息，例如在删除文件夹的时候，inotify就会同时产生删除文件夹里的文件与删除文件夹的事件，通过过滤队列，当删除文件夹事件产生的时候，会将之前加入队列的删除文件的事件全部过滤掉，这样只产生一条删除文件夹的事件，从而减轻了同步的负担。同时对于修改文件的操作的时候，会产生临时文件的重复操作。

实时同步服务原理/概念   

  1）需要部署好rsync守护进程服务，实现数据传输

  2）需要部署好inotify服务，实现目录中数据变化监控

  3）将rsync服务和inotify服务建立联系，将变化的数据进行实时备份传输    

## 部署环境

服务器A（主服务端）

服务器B（从服务器/备服务器）

rsync默认tcp端口873

sersync https://github.com/wsgzao/sersync

​        https://rsync.samba.org/

### 服务器A：

```bash
cd /app/local
wget  http://rsync.samba.org/ftp/rsync/src/rsync-3.1.1.tar.gz
tar zxf rsync-3.1.1.tar.gz
cd rsync-3.1.1
./configure
make && make install
直接使用yum install -y rsync 即可
```

#### 安装inotify-tools

```bash
cd /app/local
wget http://github.com/downloads/rvoicilas/inotify-tools/inotify-tools-3.14.tar.gz
tar zxf inotify-tools-3.14.tar.gz
cd inotify-tools-3.14
./configure --prefix=/app/local/inotify 
make && make install
yum install -y inotify-tools
[root@lnmp ~]# rpm -qa |grep inotify-tools
inotify-tools-3.14-9.el7.x86_64
```

#### inotifywait  监控目录数据信息变化

```css
Usage: inotifywait [ options ] file1 [ file2 ] [ file3 ] [ ... ]
Options:
	 -h|--help     	 Show this help text.
	 @<file>       	 Exclude the specified file from being watched.
排除指定文件
	 --exclude <pattern>    排除指定文件中的文件
	               	 Exclude all events on files matching the
	               	 extended regular expression <pattern>.
	 --excludei <pattern>
	               	 Like --exclude but case insensitive. 不区分大小写排除
	 -m|--monitor  	 Keep listening for events forever.  Without
	               	 this option, inotifywait will exit after one
	               	 event is received.
	 	 	 	 	 实现一直监控目录数据变化
	 -d|--daemon   	 Same as --monitor, except run in the background
	               	 logging events to a file specified by --outfile.
	               	 Implies --syslog.
	 	 	 	 	 类似系统日志
	 -r|--recursive	 Watch directories recursively. 递归监控
	 --fromfile <file>
	               	 Read files to watch from <file> or `-' for stdin.
	 	 	 	 	 从文件中读取监控文件，或者从stdin（终端）中读取
	 -o|--outfile <file>
	               	 Print events to <file> rather than stdout.
	 	 	 	 	 将事件打印到文件中而不是stdout（终端）
	 -s|--syslog   	 Send errors to syslog rather than stderr.
	 	 	 	 	 将事件发送到syslog而不是stderr（终端）
	 -q|--quiet    	 Print less (only print events).
	 	 	 	 	 只打印事件
	 -qq           	 Print nothing (not even events).
	 	 	 	 	 不打印任何东西
	 --format <fmt>	 Print using a specified printf-like format
	               	 string; read the man page for more details.
	 	 	 	 	 指定输出格式
	 --timefmt <fmt>	 strftime-compatible format string for use with
	               	 %T in --format string.
	 	 	 	 	 指定输出的时间信息格式
	 -c|--csv      	 Print events in CSV format.
	 	 	 	 	 以CSV格式打印
	 -t|--timeout <seconds>
	               	 When listening for a single event, time out after
	               	 waiting for an event for <seconds> seconds.
	               	 If <seconds> is 0, inotifywait will never time out.
	 	 	 	 	 当监控的事件等待时间为0那么inotifywait永不超时
	 -e|--event <event1> [ -e|--event <event2> ... ]
	 	 Listen for specific event(s).  If omitted, all events are 
	 	 listened for.
	 	 	 	 	 指定监控的事件信息

Exit status:
	 0  -  An event you asked to watch for was received.
	 1  -  An event you did not ask to watch for was received
	       (usually delete_self or unmount), or some error occurred.
	 2  -  The --timeout option was given and no events occurred
	       in the specified interval of time.

Events:
	 access	 	 file or directory contents were read
	 modify	 	 file or directory contents were written
	 attrib	 	 file or directory attributes changed
	 close_write	 file or directory closed, after being opened in
	            	 writeable mode
	 close_nowrite	 file or directory closed, after being opened in
	            	 read-only mode
	 close	 	 file or directory closed, regardless of read/write mode
	 open	 	 file or directory opened
	 moved_to	 file or directory moved to watched directory
	 moved_from	 file or directory moved from watched directory
	 move	 	 file or directory moved to or from watched directory
	 create	 	 file or directory created within watched directory
	 delete	 	 file or directory deleted within watched directory
	 delete_self	 file or directory was deleted
	 unmount	 	 file system containing file or directory unmounted
```

```bash
[root@lnmp ~]# /usr/bin/inotifywait -m /backup/
Setting up watches.
Watches established.   输出对应的操作
/backup/ CREATE 4.txt     创建文件
/backup/ OPEN 4.txt       打开文件
/backup/ ATTRIB 4.txt     修改文件的属性信息
/backup/ CLOSE_WRITE,CLOSE 4.txt	 	 保存关闭一个文件

/backup/ DELETE 2.txt     监控输出删除

/backup/ OPEN,ISDIR    打开目录索引 终端执行ll命令
/backup/ CLOSE_NOWRITE,CLOSE,ISDIR   关闭目录索引    tab建也会输出这个提示

/backup/ MODIFY 3.txt  修改临时文件
/backup/ OPEN 3.txt     打开文件
/backup/ MODIFY 3.txt   修改文件
/backup/ CLOSE_WRITE,CLOSE 3.txt   保存并退出

[root@lnmp ~]# /usr/bin/inotifywait -m /backup/4.txt @   #可以不要@
Setting up watches.
Watches established.
/backup/4.txt MODIFY 
/backup/4.txt OPEN 
/backup/4.txt MODIFY 
/backup/4.txt CLOSE_WRITE,CLOSE 
只监控指定文件

[root@lnmp ~]# /usr/bin/inotifywait -m /backup --exclude=4.txt
Setting up watches.
Watches established.
/backup/ MODIFY 3.txt
/backup/ OPEN 3.txt
/backup/ MODIFY 3.txt
/backup/ CLOSE_WRITE,CLOSE 3.txt
排除指定文件

[root@lnmp ~]# /usr/bin/inotifywait -mrq --timefmt "%F" --format "%T  %w %f 事件信息:%e" /backup 
2021-01-29  /backup/ 3.txt 事件信息:MODIFY
2021-01-29  /backup/ 3.txt 事件信息:OPEN
2021-01-29  /backup/ 3.txt 事件信息:MODIFY
2021-01-29  /backup/ 3.txt 事件信息:CLOSE_WRITE,CLOSE
```

create创建、delete删除、moved_to移入、close_write修改 是重点监视对象

> 企业应用：
>
>    防止系统重要文件被破坏需要用到inotify进行实时一直监控 /etc passwd /var/spool/cron/root

#### inotifywatch 对监控的变化信息进行统计

```css
Gather filesystem usage statistics using inotify.
Usage: inotifywatch [ options ] file1 [ file2 ] [ ... ]
Options:
	 -h|--help    	 Show this help text.
	 -v|--verbose 	 Be verbose.
	 @<file>       	 Exclude the specified file from being watched.
	 --fromfile <file>
	 	 Read files to watch from <file> or `-' for stdin.
	 --exclude <pattern>
	 	 Exclude all events on files matching the extended regular
	 	 expression <pattern>.
	 --excludei <pattern>
	 	 Like --exclude but case insensitive.
	 -z|--zero
	 	 In the final table of results, output rows and columns even
	 	 if they consist only of zeros (the default is to not output
	 	 these rows and columns).
	 -r|--recursive	 Watch directories recursively.
	 -t|--timeout <seconds>
	 	 Listen only for specified amount of time in seconds; if
	 	 omitted or 0, inotifywatch will execute until receiving an
	 	 interrupt signal.
	 -e|--event <event1> [ -e|--event <event2> ... ]
	 	 Listen for specific event(s).  If omitted, all events are 
	 	 listened for.
	 -a|--ascending <event>
	 	 Sort ascending by a particular event, or `total'.
	 -d|--descending <event>
	 	 Sort descending by a particular event, or `total'.

Exit status:
	 0  -  Exited normally.
	 1  -  Some error occurred.

Events:
	 access	 	 file or directory contents were read
	 modify	 	 file or directory contents were written
	 attrib	 	 file or directory attributes changed
	 close_write	 file or directory closed, after being opened in
	            	 writeable mode
	 close_nowrite	 file or directory closed, after being opened in
	            	 read-only mode
	 close	 	 file or directory closed, regardless of read/write mode
	 open	 	 file or directory opened
	 moved_to	 file or directory moved to watched directory
	 moved_from	 file or directory moved from watched directory
	 move	 	 file or directory moved to or from watched directory
	 create	 	 file or directory created within watched directory
	 delete	 	 file or directory deleted within watched directory
	 delete_self	 file or directory was deleted
	 unmount	 	 file system containing file or directory unmounted
```

#### 安装sersync

```bash
cd /app/local
wget https://github.com/wsgzao/sersync/blob/master/sersync2.5.4_64bit_binary_stable_final.tar.gz
tar zxf sersync2.5.4_64bit_binary_stable_final.tar.gz
mv /app/local/GNU-Linux-x86/ /app/local/sersync
cd /app/local/sersync
方法二
https://github.com/wsgzao/sersync
下载
https://github.com/wsgzao/sersync/blob/master/sersync2.5.4_64bit_binary_stable_final.tar.gz
[root@lnmp ~]# mkdir /server/tools/
[root@lnmp ~]# rz -y   上传下载好的sersync软件
[root@lnmp ~]# yum install -y unzip
[root@lnmp ~]# unzip /server/tools/sersync-master.zip
[root@lnmp ~]# tree /server/tools/
/server/tools/
├── sersync-master
│   ├── inotify-tools-3.14.tar.gz
│   ├── README.md
│   ├── rsync-3.1.1.tar.gz
│   └── sersync2.5.4_64bit_binary_stable_final.tar.gz
└── sersync-master.zip
[root@lnmp ~]# tar -zxvf /server/tools/sersync-master/sersync2.5.4_64bit_binary_stable_final.tar.gz
[root@lnmp ~]#mv /server/tools/GNU-Linux-x86/ /usr/local/sersync
[root@lnmp ~]# tree /usr/local/sersync/
/usr/local/sersync/
├── confxml.xml   sersync配置文件
└── sersync2    sersync软件命令目录

#配置下密码文件，因为这个密码是要访问服务器B需要的密码和上面服务器B的密码必须一致
[root@lnmp ~]# echo "123456" > /etc/rsync.password
[root@lnmp ~]# cat /etc/rsync.password 
123456
#修改权限
[root@lnmp ~]#chmod 600 /etc/rsync.password
[root@lnmp ~]# ll /etc/rsync.password 
-rw-------. 1 root root 7 Jan 29 10:10 /etc/rsync.password
#修改confxml.conf
[root@lnmp ~]# vim /usr/local/sersync/confxml.xml
<?xml version="1.0" encoding="ISO-8859-1"?>
  <head version="2.5">
       <host hostip="localhost" port="8008"></host>
       <debug start="false"/>
       <fileSystem xfs="false"/>
       <filter start="false">
           <exclude expression="(.*)\.svn"></exclude>
           <exclude expression="(.*)\.gz"></exclude>
           <exclude expression="^info/*"></exclude>
        <exclude expression="^static/*"></exclude>
    </filter>
     <!--说明：排除指定数据信息不要进行实时传输同步-->
    <inotify>
        <delete start="true"/>
        <createFolder start="false"/>
        <createFile start="true"/>
        <closeWrite start="true"/>
        <moveFrom start="true"/>
        <moveTo start="true"/>
        <attrib start="false"/>
        <modify start="false"/>
    </inotify>
<!--说明：定义inotify程序需要监控的事件-->
    <sersync>
        <localpath watch="/backup">
<!- 这里填写服务器A要同步的文件夹路径-->
             <remote ip="192.168.1.200" name="backup"/>
             <!-- 这里填写服务器B的IP地址和模块名-->
             <!--<remote ip="192.168.8.39" name="tongbu"/>-->
             <!--<remote ip="192.168.8.40" name="tongbu"/>-->
         </localpath>
         <rsync>
             <commonParams params="-artuz"/>
            <auth start="true" users="rsync_backup" passwordfile="/etc/rsync.password"/>
<!-- rsync+密码文件 这里填写服务器B的认证信息-->
              <userDefinedPort start="false" port="873"/><!-- port=874 -->
              <timeout start="false" time="100"/><!-- timeout=100 -->
              <ssh start="false"/>
          </rsync>
          <failLog path="/tmp/rsync_fail_log.sh" timeToExecute="60"/><!--default every 60mins execut
e once-->
<!-- 修改失败日志记录（可选）-->
         <crontab start="false" schedule="600"><!--600mins-->
             <crontabfilter start="false">
                 <exclude expression="*.php"></exclude>
                 <exclude expression="info/*"></exclude>
             </crontabfilter>
         </crontab>
         <plugin start="false" name="command"/>
     </sersync>
  <!-- 下面这些有关于插件你可以忽略了 -->
     <plugin name="command">
         <param prefix="/bin/sh" suffix="" ignoreError="true"/>  <!--prefix /opt/tongbu/mmm.sh suff
    ix-->
         <filter start="false">
             <include expression="(.*)\.php"/>
             <include expression="(.*)\.sh"/>
         </filter>
     </plugin>
 
     <plugin name="socket">
         <localpath watch="/opt/tongbu">
             <deshost ip="192.168.138.20" port="8009"/>
         </localpath>
     </plugin>
     <plugin name="refreshCDN">
         <localpath watch="/data0/htdocs/cms.xoyo.com/site/">
             <cdninfo domainname="ccms.chinacache.com" port="80" username="xxxx" passwd="xxxx"/>
              <sendurl base="http://pic.xoyo.com/cms"/>
             <regexurl regex="false" match="cms.xoyo.com/site([/a-zA-Z0-9]*).xoyo.com/images"/>
        </localpath>
    </plugin>
</head>
[root@lnmp ~]# vim /etc/rsyncd.conf
uid = rsync
gid = rsync
port = 874

```

#### 启动sersync 服务

```bash
[root@lnmp ~]# export PATH="$PATH:/usr/local/sersync"
[root@lnmp ~]# echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/var/lib/snapd/snap/bin:/root/bin:/usr/local/sersync
[root@lnmp ~]# sersync2 -h
set the system param
execute：echo 50000000 > /proc/sys/fs/inotify/max_user_watches
execute：echo 327679 > /proc/sys/fs/inotify/max_queued_events
parse the command param
```

> 参数-d:启用守护进程模式
>
> 参数-r:在监控前，将监控目录与远程主机用rsync命令推送一遍
>
> c参数-n: 指定开启守护线程的数量，默认为10个
>
> 参数-o:指定配置文件，默认使用confxml.xml文件
>
> 参数-m:单独启用其他模块，使用 -m refreshCDN 开启刷新CDN模块
>
> 参数-m:单独启用其他模块，使用 -m socket 开启socket模块
>
> 参数-m:单独启用其他模块，使用 -m http 开启http模块
>
> 不加-m参数，则默认执行同步程序

```bash
[root@lnmp ~]# sersync2 -dro /usr/local/sersync/confxml.xml  启动服务
set the system param
execute：echo 50000000 > /proc/sys/fs/inotify/max_user_watches
execute：echo 327679 > /proc/sys/fs/inotify/max_queued_events
parse the command param
option: -d 	 run as a daemon
option: -r 	 rsync all the local files to the remote servers before the sersync work
option: -o 	 config xml name：  /usr/local/sersync/confxml.xml
daemon thread num: 10
parse xml config file
host ip : localhost	 host port: 8008
daemon start，sersync run behind the console 
use rsync password-file :
user is	 rsync_backup
passwordfile is 	 /etc/rsync.password
config xml parse success
please set /etc/rsyncd.conf max connections=0 Manually
sersync working thread 12  = 1(primary thread) + 1(fail retry thread) + 10(daemon sub threads) 
Max threads numbers is: 22 = 12(Thread pool nums) + 10(Sub threads)
please according your cpu ，use -n param to adjust the cpu rate
------------------------------------------
rsync the directory recursivly to the remote servers once
working please wait...
execute command: cd /backup && rsync -artuz -R --delete ./ rsync_backup@192.168.1.200::backup --password-file=/etc/rsync.password >/dev/null 2>&1 
run the sersync: 
watch path is: /backup
sersync2 -dro  /usr/local/sersync/conf/confxml.xml   启动实时同步服务
killall sersync                                     停止实时同步服务
/etc/rc.local <-- sersync2 -dro  /usr/local/sersync/confxml.xml   开机自动启动
[root@lnmp ~]# echo "sersync2 -dro  /usr/local/sersync/confxml.xml" >> /etc/rc.local 
[root@lnmp ~]# cat /etc/rc.local 
#!/bin/bash
touch /var/lock/subsys/local
sersync2 -dro  /usr/local/sersync/confxml.xml
[root@lnmp ~]# chmod +x /etc/rc.d/rc.local
[root@lnmp ~]# ll /etc/rc.d/rc.local
-rwxr-xr-x. 1 root root 519 Jan 29 21:44 /etc/rc.d/rc.local
[root@lnmp ~]# ps -ef |grep sersync
root       1955      1  0 21:38 ?        00:00:00 sersync2 -dro /usr/local/sersync/confxml.xml
root       1980   1311  0 21:41 pts/0    00:00:00 grep --color=auto sersync
[root@lnmp ~]# killall sersync2
[root@lnmp ~]# ps -ef |grep sersync
root       1986   1311  0 21:42 pts/0    00:00:00 grep --color=auto sersync
```

### 服务器B：

```bash
cd /app/local
wget  http://rsync.samba.org/ftp/rsync/src/rsync-3.1.1.tar.gz
tar zxf rsync-3.1.1.tar.gz
cd rsync-3.1.1
./configure
make && make install
[root@lnmp /backup]# rpm -qa |grep rsync
rsync-3.1.2-10.el7.x86_64
```

设置rsync的配置文件

```bash
vi /etc/rsyncd.conf
#服务器B上的rsyncd.conf文件内容
uid=root
gid=root
#最大连接数
max connections=36000
#默认为true，修改为no，增加对目录文件软连接的备份 
use chroot=no
#定义日志存放位置
log file=/var/log/rsyncd.log
#忽略无关错误
ignore errors = yes
#设置rsync服务端文件为读写权限
read only = no 
#认证的用户名与系统帐户无关在认证文件做配置，如果没有这行则表明是匿名
auth users = rsync
#密码认证文件，格式(虚拟用户名:密码）
secrets file = /etc/rsync.passwd
#这里是认证的模块名，在client端需要指定，可以设置多个模块和路径
[backup]
#自定义注释
comment  = backup
#同步到B服务器的文件存放的路径
path=/backup


#创建rsync认证文件  可以设置多个，每行一个用户名:密码，注意中间以“:”分割
echo "rsync_backup:123456" > /etc/rsync.passwd
#设置文件所有者读取、写入权限
chmod 600 /etc/rsyncd.conf  
chmod 600 /etc/rsync.pass  
#启动服务器B上的rsync服务
systemctl start rsyncd
#监听端口873
netstat -an | grep 873
lsof -i tcp:873
COMMAND   PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
rsync   31445 root    4u  IPv4 443872      0t0  TCP *:rsync (LISTEN)
rsync   31445 root    5u  IPv6 443873      0t0  TCP *:rsync (LISTEN)

注意：
B服务器：
[root@lnmp /backup]# vim /etc/rsyncd.conf
uid = rsync
gid = rsync
port = 874
A服务器
<rsync>
           <!--<commonParams params="-artuz"/>-->
            <commonParams params="-az"/>
            <auth start="true" users="rsync_backup" passwordfile="/etc/rsync.password"/>
            <userDefinedPort start="true" port="874"/><!-- port=874 -->
            <timeout start="false" time="100"/><!-- timeout=100 -->
            <ssh start="false"/>
        </rsync>

```

```bash
[root@lnmp /backup]# sersync2 -dro /usr/local/sersync/confxml.xml
set the system param
execute：echo 50000000 > /proc/sys/fs/inotify/max_user_watches
execute：echo 327679 > /proc/sys/fs/inotify/max_queued_events
parse the command param
option: -d 	 run as a daemon
option: -r 	 rsync all the local files to the remote servers before the sersync work
option: -o 	 config xml name：  /usr/local/sersync/confxml.xml
daemon thread num: 10
parse xml config file
host ip : localhost	 host port: 8008
daemon start，sersync run behind the console 
use rsync password-file :
user is	 rsync_backup
passwordfile is 	 /etc/rsync.password
config xml parse success
please set /etc/rsyncd.conf max connections=0 Manually
sersync working thread 12  = 1(primary thread) + 1(fail retry thread) + 10(daemon sub threads) 
Max threads numbers is: 22 = 12(Thread pool nums) + 10(Sub threads)
please according your cpu ，use -n param to adjust the cpu rate
------------------------------------------
rsync the directory recursivly to the remote servers once
working please wait...
execute command: cd /backup && rsync -az -R --delete ./  --port=874 rsync_backup@192.168.1.200::backup --password-file=/etc/rsync.password >/dev/null 2>&1 
run the sersync: 
watch path is: /backup
[root@lnmp /backup]# cat /etc/rc.local 
touch /var/lock/subsys/local
sersync2 -dro  /usr/local/sersync/confxml.xml
A服务器
[root@lnmp ~]# cd /backup/
[root@lnmp /backup]# ll
total 0
drwxr-xr-x 12 rsync rsync 266 Jan 23 19:12 192.168.1.200
-rw-r--r--  1 root  root    0 Jan 30 16:46 a.txt
[root@lnmp /backup]# rm -f a.txt
B服务
[root@lnmp /backup]# ll
total 0
drwxr-xr-x 12 rsync rsync 266 Jan 23 19:12 192.168.1.200

```

> 这里实时备份的只有A服务器备份到B服务器，B服务器创建的文件会在下次a服务器创建文件之后—delete 参数删除。这里切记！！

```bash
运行sersync
nohup /app/local/sersync/sersync2 -r -d -o /app/local/sersync/confxml.xml >/app/local/sersync/rsync.log 2>&1 &
rsync 这里是监控模块的名称
nohup /app/local/sersync/sersync2 -r -d -o /app/local/sersync/img.xml >/app/local/sersync/img.log 2>&1 &
```

> ```
> -d:启用守护进程模式
> -r:在监控前，将监控目录与远程主机用rsync命令推送一遍
> -n: 指定开启守护线程的数量，默认为10个
> -o:指定配置文件，默认使用confxml.xml文件
> ```

```bash
[root@lnmp /backup]# nohup /usr/local/sersync/sersync2 -rdo /usr/local/sersync/confxml.xml >/usr/local/sersync/rsync$(date +%F).log 2>&1 &
[3] 1329
[2]   Exit 1                  nohup /usr/local/sersync/sersync2 -rdo /usr/local/sersync/confxml.xml > /app/local/sersync/rsync$(date +%F).log 2>&1
[root@lnmp /backup]# jobs -l 看不到后台运行的
[root@lnmp /backup]# ps -ef |grep sersync2
root        813      1  0 16:47 ?        00:00:00 sersync2 -dro /usr/local/sersync/confxml.xml
root       1333      1  0 16:58 ?        00:00:00 /usr/local/sersync/sersync2 -rdo /usr/local/sersync/confxml.xml
root       1368   1267  0 17:00 pts/0    00:00:00 grep --color=auto sersync2
```

#### nohup和&后台运行，进程查看及终止

1.nohup

用途：不挂断地运行命令。

语法：nohup Command [ Arg … ] [　& ]

　　无论是否将 nohup 命令的输出重定向到终端，输出都将附加到当前目录的 nohup.out 文件中。

　　如果当前目录的 nohup.out 文件不可写，输出重定向到 $HOME/nohup.out 文件中。

　　如果没有文件能创建或打开以用于追加，那么 Command 参数指定的命令不可调用。

退出状态：该命令返回下列出口值： 　

　　可以查找但不能调用 Command 参数指定的命令。 　

　　nohup 命令发生错误或不能查找由 Command 参数指定的命令。 　

　　否则，nohup 命令的退出状态是 Command 参数指定命令的退出状态。

2.&

用途：在后台运行

一般两个一起用

nohup command &