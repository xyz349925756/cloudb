# 架构说明

中小规模网站架构组成

1) 顾客--用户

  访问网站的人员

2) 保安--防火墙 (firewalld)

  进行访问策略控制

3) 迎宾--负载均衡服务器 (nginx)

  对用户的访问请求进行调度处理

4) 服务员---web服务器  (nginx)

  处理用户的请求

5) 厨师---数据库服务器  (mysql)

  存储的字符数据 (耳机 500  索尼 黑色 北京地址 订单时间2019-05-05 13:00)

6) 厨师---存储服务器   (nfs)

  存储图片 音频 视频 附件等数据信息

7) 厨师---备份服务器   (rsync+crond-定时备份 rsync+sersync--实时备份)

  存储网站所有服务器的重要数据

8) 厨师---缓存服务器   (memcache redis mongodb) 

  a 将数据信息存储到内存中

  b 减缓服务器的压力       

9) 经理---批量管理服务器 (ansible)

  批量管理多台服务器主机

 

部署网站架构:

1) 需要解决网站架构单点问题

  迎宾:       高可用服务---keepalived

  数据库:  高可用服务---mha

  存储服务: 高可用服务---keepalived实现

​                高可用服务---分布式存储

  备份服务: 

  面试题: 公司的数据是如何备份

1) 利用开源软件实现数据备份 rsync(免费)

2) 利用企业网盘进行数据备份 七牛云存储

3) 利用自建备份存储架构   两地三中心 

  缓存服务: 高可用服务--- 缓存服务集群/哨兵模式

2) 内部员工如何远程访问架构

  部署搭建VPN服务器 PPTP vpn

  https://blog.oldboyedu.com/pptp-l2tp/

3) 内部员工操作管理架构服务器要进行审计

  跳板机服务器 jumpserver

  https://jumpserver.readthedocs.io/zh/docs/setup_by_centos.html

4) 架构中服务器出现问题需要进行提前报警告知

  部署监控服务器 zabbix

## 综合架构规划

主机名称和IP地址规划

1. 防火墙服务器    firewalld      10.0.0.81(外网地址)     172.16.1.81(内外地址)   软件: firewalld

2. 负载均衡服务器   lb01         10.0.0.5          172.16.1.5           软件: nginx keepalived

3. 负载均衡服务器   lb02       10.0.0.6          172.16.1.6           软件: nginx keepalived
4. web服务器     web01      10.0.0.7          172.16.1.7         软件: nginx

5. web服务器     web02       10.0.0.8          172.16.1.8         软件: nginx

6. web服务器     web03       10.0.0.9(存储)       172.16.1.9         软件: nginx

7. 数据库服务器    db01        10.0.0.51         172.16.1.51         软件: mysql(慢) mariaDB

8. 存储服务器      nfs01       10.0.0.31         172.16.1.31         软件: nfs 

9. 备份服务器         backup       10.0.0.41         172.16.1.41         软件: rsync

10. 批量管理服务器  m01        10.0.0.61         172.16.1.61         软件: ansible

11. 跳板机服务器    jumpserver     10.0.0.71(61)       172.16.1.71         软件: jumpserver

12. 监控服务器     zabbix       10.0.0.72(61)       172.16.1.72         软件: zabbix

先把路走通,再进行变通

10. 缓存服务器   忽略



# 计划任务(定时任务)

定时任务：就是完成管理员指定的功能任务的程序。

企业常用定时任务：清理磁盘 自动更新时间ntpdate xxx 很重要的功能。

cronie  实现定时任务功能*****

at    实现定时任务功能 只能一次设置定时功能 

anacron 实现定时任务功能 应用在家用电脑 7*24服务器

```bash

[root@lnmp ~/oldboy]# yum install at -y
[root@lnmp ~/oldboy]# systemctl start atd
[root@lnmp ~/oldboy]# systemctl enable atd
[root@lnmp ~/oldboy]# systemctl status atd
[root@lnmp ~/oldboy]# at 21:10
at> date > ./e.log
at> <EOT>
job 4 at Thu Sep 17 21:10:00 2020
[root@lnmp ~/oldboy]# ll
total 12
-rw-r--r--. 1 root root  17 Sep 13 10:26 1.txt
-rw-r--r--. 1 root root 308 Sep 13 13:39 2.txt
-rw-r--r--. 1 root root  29 Sep 17 21:10 e.log
drwxr-xr-x. 2 root root   6 Sep 17 19:46 test
[root@lnmp ~/oldboy]# tail -F e.log 
Thu Sep 17 21:10:01 CST 2020
^C
You have new mail in /var/spool/mail/root
Cronie

[root@lnmp ~/oldboy]# rpm -qa cronie
cronie-1.4.11-23.el7.x86_64
[root@lnmp ~/oldboy]# rpm -ql cronie
/etc/cron.d
/etc/cron.d/0hourly
/etc/cron.deny
/etc/pam.d/crond
/etc/sysconfig/crond
/usr/bin/crontab
/usr/lib/systemd/system/crond.service
/usr/sbin/crond
/usr/share/doc/cronie-1.4.11
/usr/share/doc/cronie-1.4.11/AUTHORS
/usr/share/doc/cronie-1.4.11/COPYING
/usr/share/doc/cronie-1.4.11/ChangeLog
/usr/share/doc/cronie-1.4.11/INSTALL
/usr/share/doc/cronie-1.4.11/README
/usr/share/man/man1/crontab.1.gz
/usr/share/man/man5/crontab.5.gz
/usr/share/man/man8/cron.8.gz
/usr/share/man/man8/crond.8.gz
/var/spool/cron
```

定时任务实现方法:

> 日志文件需要定期进行切割处理

周一     secure  100M

周二(00:00) mv secure secure-`date +%F` 100M 切割后的文件

​       touch secure

系统特殊目录:

系统定时任务周期：每小时  控制定时任务目录：/etc/cron.hourly

系统定时任务周期：每一天  控制定时任务目录：/etc/cron.daily  00:00-23:59

系统定时任务周期：每一周  控制定时任务目录：/etc/cron.weekly 7天

系统定时任务周期：每个月  控制定时任务目录：/etc/cron.monthly 30 28 31

 

> 用户定时任务

每天的02:30进行数据备份

a 用户定时任务查看  crontab -l（list）

说明: 列表查看定时任务信息（cron table）

 

b 用户定时任务编辑  crontab -e（edit）

说明: 编辑配置定时任务信息

crontab -e 编写定时任务  vi /var/spool/cron/    定时任务配置文件保存目录

​             /var/spool/cron/root  root用户设置的定时任务配置文件

                  /var/spool/cron/oldboy oldboy用户设置的定时任务配置文件

visudo   对普通用户提权  vi /etc/sudoers

```BASH
[root@lnmp ~/oldboy]# cat /etc/crontab 
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root
# For details see man 4 crontabs
# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed
 分  时  日 月 周
定时任务实际编写方法
定时任务服务环境准备  
  定时任务服务是否启动/是否开机自动启动	 
  [root@oldboyedu ~]# systemctl status crond
```

```css
写法:
01. 用数值表示时间信息
00 02 *  *  *   备份文件
每隔2小时执行备份文件操作

02. 利用特殊符号表示时间信息
    *      *      *     *     *   备份文件 
每分钟  每小时 每天  每月  每周


  PS: 定时任务最短执行的周期为每分钟
  */5     */5    */5    
  每隔5分钟  每隔5小时	 每隔5天
    01-05   02    *    *    *	 	 
    01到05   02    *    *    *
  指定时间的范围
    00      14,20  *   *    *
指定不连续的时间信息
  
 测验01: 每天下午02:30分钟 起来学习
         30 14 * * *
 测验02: 每隔3天 夜里2点   执行数据备份
         00 02 */3  *  * 
 测验03: */10  01,03  *  *   *   ??? 
         01点 每隔10分钟 
	    03点 每隔10分钟  
         每天   凌晨1点和凌晨3点  每隔10分钟0点整 -->  01:00 03:00	    
 测验04: */10   01-03  *  *   *
         每天  凌晨1点至凌晨三点 每10分钟
 测验05: *      01,03  *  *   *   ???   
        每天  1点，3点  
 测验06: *      01-03	  *  *   *   
        每天 1点至3点 
 测验07: 00     02     28  */2  6 	 ???   
         分     时     天   每2月   周六
        每隔两个月的28号的星期6的凌晨2点。不一定28号是周六
         02:00  28 每隔两个月  星期6
结论: 
 01. 在写时间信息的时候, 如果想表示每隔多久执行什么任务
     /上面尽量用*号表示, 不要写具体数值
 02. 时间信息由左到右依次书写, 尽量不要跳步
 03. 当编写定时任务时,日期信息不要和星期信息同时出现
 补充说明:
 20/10  01,03 * * *  
 从20分开始，每隔10分钟
 01:20 01:30 01:40       01:59
 03:20 03:30 01:40      03:59
实际编写定时任务
1) 每天凌晨两点备份 /data目录到 /backup
第一个历程: 写上时间信息
00 02 * * *
第二个历程: 写上完成任务的具体命令
cp -a /data /backup
第三个历程: 编写定时任务
crontab -e 
00 02 * * *  cp -a /data /backup	   
定时任务排查方法:
```

```bash
01. 检查是否有定时任务配置文件
cat /var/spool/cron/root 
00 02 * * *  cp -a /data /backup
02. 检查定时任务日志文件
ll /var/log/cron
-rw-------. 1 root root 14050 Apr 25 15:49 /var/log/cron
日志信息说明
Apr 25 15:53:22 oldboyedu crontab[3893]: (root) BEGIN EDIT (root)
Apr 25 15:54:06 oldboyedu crontab[3934]: (oldboy) BEGIN EDIT (oldboy)
Apr 25 15:54:48 oldboyedu crontab[3893]: (root) REPLACE (root)
Apr 25 15:54:48 oldboyedu crontab[3893]: (root) END EDIT (root)
Apr 25 15:55:01 oldboyedu crond[905]: (root) RELOAD (/var/spool/cron/root)
Apr 25 15:55:01 oldboyedu CROND[3939]: (root) CMD (cp -a /data /backup)
Apr 25 15:55:01 oldboyedu CROND[3937]: (root) MAIL (mailed 55 bytes of output but got 
  执行时间      主机名   编辑定时任务    以什么用户编辑或执行定时任务/干了什么事情
                        执行定时任务

```

时任务编写注意事项:(规范)

 编写定时任务要有注释说明

 编写定时任务路径信息尽量使用绝对路径

 编写定时任务命令需要采用绝对路径执行 /usr/sbin/useradd

 命令执行成功条件:

 useradd ---> $PATH ---> /sbin/useradd ---> 命令执行成功

 定时任务执行时,识别的PATH信息只有: /usr/bin:/bin

 useradd命令--->usr/sbin/useradd

 编写定时任务时,可以将输出到屏幕上的信息保存到黑洞中,避免占用磁盘空间

```bash
 * * * * *  sh test.sh &>/dev/null
```

说明: 定时任务中执行命令,如果产生输出到屏幕的信息,都会以邮件方式告知用户

```bash
 /var/spool/mail/root     不断变大占用磁盘空间    占用的block空间
```

 解决方法: 将邮件服务关闭

```bash
 systemctl stop postfix
```

 

 /var/spool/postfix/maildrop/ 不断产生小文件占用磁盘空间 占用的inode空间

 解决方法: 删除小文件

```bash
 rm -f /var/spool/postfix/maildrop/*
 systemctl start postfix
```

 编写定时任务, 尽量不要产生屏幕输出信息

 

```sh
cp -a /data /backup  
 tar zcvf /backup/data.tar.gz /data  有信息输出
 cd / 
 tar zcf /backup/data.tar.gz ./data  没有信息输出
```

 当需要多个命令完成一个定时任务需求时,可以利用脚本编写定时

 ```bash
 vim backup.sh 
  cp -a /data /backup	 
  tar zcvf /backup/data.tar.gz  /data
 ```

```bash
crontab -e 
 # xxxxx
 * * * * *  /bin/sh /server/scripts/backup.sh &>/dev/null
```

定时任务编写的注意事项:

\1) 定时任务编写时需要加注释信息

\2) 文件的路径尽量采用绝对路径

\3) 命令信息最好也要用绝对路径 

\4) 编写定时任务尽量在后面加上重定向黑洞 &>/dev/null

  定时任务中有输出到屏幕上的信息:

  如果开启邮件服务 postfix: 输出的信息 >> /var/spool/mail/root       

  如果关闭邮件服务 postfix: 输出的信息 >> /var/spool/postfix/maildrop/小文件

\5) 尽可能让命令不要产生正确或错误的输出信息

  tar zcvf --> tar zcf 

\6) 多个定时任务命令,最好使用脚本实现

\7) 定时任务中无法识别任务中的一些特殊符号 

  解决方式一: 利用转义符号        

```bash
  \* * * * * /bin/date "+\%F \%T" >/tmp/time.txt
```

  解决方式二: 利用脚本编写任务

  ```bash
   vim /oldboy/date.sh
    /bin/date "+%F %T" 
    * * * * * /bin/sh /oldboy/date.sh &>/dev/null
  ```





