# zabbix 监控服务

## 概念简述

综合架构监控服务器常用命令

  CPU: top htop glances

  监控什么:

   us: user state      用户态信息 40%

   sy: system state  内核态信息 40% MySQL进程

   id: idle      空闲状态  20%

  内存: top htop free

   监控什么:

​     内存可用率:

   swap空间使用情况:

   磁盘: df iotop(检查磁盘io消耗) glances

   监控什么:

​     磁盘使用情况

   磁盘的IO消耗 

   网络: iftop glances

   监控什么:

​     网络带宽使用情况

   进程: top htop ps glances

   监控什么: 

   占用内存情况 tomcat(java)---内存占满(内存溢出)--服务出现僵死(重启服务)

   占用CPU情况: MySQL

   负载: w top uptime glances

   监控什么:

   10分钟负载 <CPU内核数

   15分钟负载

## 综合架构监控服务体系结构:

1) 硬件监控     服务器 路由器 交换机 防火墙(SNMP)

2) 系统监控   CPU 内存 磁盘 网络 进程 TCP(十一种状态)

3) 服务监控   nginx php tomcat redis memcache mysql

4) 网站监控   请求时间 响应时间 加载时间 页面监控

5) 日志监控   ELK（收集 存储 分析 展示） 日志易

     access --- 用户源IP地址 北京1000 上海100 深圳500   

6) 安全监控   Firewalld(4层和4层以上) WAF（Nginx+lua）(应用层面) 安全宝 牛盾云 安全狗

7) 网络监控   smokeping 监控宝 站长工具 奇云测 多机房

8) 业务监控   (数据库信息)活动产生多少流量 产生多少注册量 带来多少价值    

## zabbix监控服务说明

### zabbix软件结构组成

   zabbix-server:  监控服务端

   zabbix-agent:   监控客户端

   zabbix-web:     监控网站服务

   php:         处理动态请求

   mysql:        数据库存储监控数据

   zabbix-proxy:   负责收集agent信息汇总告知zabbix-server

 

### zabbix软件安装部署过程

   软件选择: 5.0 LTS(long time support 长期支持版)

![image-20210818225911493](./linux运维09 zabbix监控服务.assets\image-20210818225911493.png)

```bash
Install and configure Zabbix server for your platform

a. Install Zabbix repository
# rpm -Uvh https://repo.zabbix.com/zabbix/5.0/rhel/7/x86_64/zabbix-release-5.0-1.el7.noarch.rpm
# yum clean all


b. Install Zabbix server and agent
# yum install zabbix-server-mysql zabbix-agent


c. Install Zabbix frontend
Enable Red Hat Software Collections
# yum install centos-release-scl
ps: 版本选择错误删除
[root@m01 ~]# rpm -evh zabbix-release-5.0-1.el7.noarch

编辑配置文件 /etc/yum.repos.d/zabbix.repo and enable zabbix-frontend repository.
[zabbix-frontend]
...
enabled=1
...
```

![image-20210818230004716](./linux运维09 zabbix监控服务.assets\image-20210818230004716.png)

```bash
Install Zabbix frontend packages.
# yum install zabbix-web-mysql-scl zabbix-nginx-conf-scl

d. 创建初始数据库
Make sure you have database server up and running.
yum install mariadb-server -y
systemctl start mariadb 
systemctl enable mariadb
在数据库主机上运行以下代码。
# mysql -uroot -p
password
mysql> create database zabbix character set utf8 collate utf8_bin;
mysql> create user zabbix@localhost identified by 'password';
mysql> grant all privileges on zabbix.* to zabbix@localhost;
mysql> quit;
---------------------------------------------------------------------------------
[root@data01 ~]# mysql -uroot -p
Enter password: 
MariaDB [(none)]> create database zabbix character set utf8 collate utf8_bin;
MariaDB [(none)]> create user zabbix@localhost identified by 'password';
MariaDB [(none)]> grant all privileges on zabbix.* to zabbix@localhost;
MariaDB [(none)]> quit

zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -uzabbix -p zabbix
导入初始架构和数据，系统将提示您输入新创建的密码。
[root@m01 /etc/yum.repos.d]# scp /usr/share/doc/zabbix-server-mysql-5.0.10/create.sql.gz root@192.168.1.200:/tmp
create.sql.gz                                                               100% 1737KB  13.5MB/s   00:00    
[root@data01 ~]# zcat /tmp/create.sql.gz | mysql -uzabbix -p zabbix
Enter password:password  上面设置的zabbix密码

e. 为Zabbix server配置数据库
编辑配置文件 /etc/zabbix/zabbix_server.conf
DBPassword=password
```

![image-20210818230059525](./linux运维09 zabbix监控服务.assets\image-20210818230059525.png)

```bash
f. 为Zabbix前端配置PHP
编辑配置文件 /etc/opt/rh/rh-nginx116/nginx/conf.d/zabbix.conf, uncomment and set 'listen' and 'server_name' directives. 取消前面的注释#

# listen 80;
# server_name example.com;
编辑配置文件 /etc/opt/rh/rh-php72/php-fpm.d/zabbix.conf, add nginx to listen.acl_users directive.
listen.acl_users = apache,nginx

```

![image-20210818230235526](./linux运维09 zabbix监控服务.assets\image-20210818230235526.png)

```bash
Then uncomment and set the right timezone for you.
; php_value[date.timezone] = Europe/Riga
```

![image-20210818230309702](./linux运维09 zabbix监控服务.assets\image-20210818230309702.png)

```bash
php_value[date.timezone] = Asia/Shanghai


g. 启动Zabbix server和agent进程
启动Zabbix server和agent进程，并为它们设置开机自启：
# systemctl restart zabbix-server zabbix-agent rh-nginx116-nginx rh-php72-php-fpm
# systemctl enable zabbix-server zabbix-agent rh-nginx116-nginx rh-php72-php-fpm
```

![image-20210818230343844](./linux运维09 zabbix监控服务.assets\image-20210818230343844.png)

```bash
h. 配置Zabbix前端
连接到新安装的Zabbix前端： http://server_ip_or_name
```

以上是官网安装教程

 zabbix服务端部署软件流程:

```bash
第一个里程: 下载安装zabbix yum 源文件
LTS long time support
1) aliyun zabbix yum 源 
2) 清华源
rpm -ivh https://mirrors.tuna.tsinghua.edu.cn/zabbix/zabbix/4.0/rhel/7/x86_64/zabbix-release-4.0-1.el7.noarch.rpm
rpm -ivh https://mirrors.tuna.tsinghua.edu.cn/zabbix/zabbix/3.4/rhel/7/x86_64/zabbix-release-4.0-1.el7.noarch.rpm  -- 老版本

第二个里程: 下载安装zabbix服务端相关软件
zabbix服务程序软件: zabbix-server-mysql
zabbix服务web软件: zabbix-web-mysql httpd php
数据库服务软件: mariadb-server
yum install -y zabbix-server-mysql zabbix-web-mysql httpd php mariadb-server

第三个里程: 软件配置
vim /etc/zabbix/zabbix_server.conf
126 DBPassword=zabbix
vim /etc/httpd/conf.d/zabbix.conf
21         php_value date.timezone Asia/Shanghai

第四个里程: 编写配置数据库服务
systemctl start mariadb.service 
systemctl status mariadb.service
创建zabbix数据库--zabbix
create database zabbix character set utf8 collate utf8_bin;
创建数据库管理用户
grant all privileges on zabbix.* to zabbix@localhost identified by 'zabbix';
在zabbix数据库中导入相应的表信息
zcat /usr/share/doc/zabbix-server-mysql-4.0.0/create.sql.gz|mysql -uzabbix -pzabbix zabbix
 zgrep 

第五个里程: 启动zabbix程序相关服务
数据库服务 zabbix服务 httpd服务
systemctl start zabbix-server.service httpd mariadb.service
systemctl enable zabbix-server.service httpd mariadb.service
说明: 至此zabbix-server命令行操作结束
 LNMP: nginx php mysql 
 LAMP: apache(php模块) mysql

第六个里程: 登录zabbix服务端web界面, 进行初始化配置
http://10.0.0.71/zabbix/setup.php

10051  zabbix-server 服务端端口号
10050  zabbix-agent  客户端端口号
/etc/zabbix/web/zabbix.conf.php   -- 记录web页面初始化信息

第七个里程: 登录zabbix服务web页面
用户名Admin 密码zabbix

```

监控客户端部署流程

```bash
第一个里程: 下载安装zabbix yum 源文件
LTS long time support
1) aliyun zabbix yum 源 
2) 清华源
rpm -ivh https://mirrors.tuna.tsinghua.edu.cn/zabbix/zabbix/4.0/rhel/7/x86_64/zabbix-release-4.0-1.el7.noarch.rpm

第二个里程: 下载安装zabbix客户端软件
yum install -y zabbix-agent
或者
rpm -ivh https://mirrors.tuna.tsinghua.edu.cn/zabbix/zabbix/4.0/rhel/7/x86_64/zabbix-agent-4.0.0-2.el7.x86_64.rpm

第三个里程: 编写zabbix客户端配置文件
vim /etc/zabbix/zabbix_agentd.conf
98 Server=172.16.1.71

第四个里程: 启动zabbix-agent服务
[root@web01 ~]# systemctl start zabbix-agent
[root@web01 ~]# netstat -lntup|grep 10050
tcp      0     0 0.0.0.0:10050           0.0.0.0:*            LISTEN      4509/zabbix_agentd  
tcp      0     0 :::10050                :::*                 LISTEN      4509/zabbix_agentd
```

客户端安装

```bash
[root@data01 /server]#  rpm -Uvh https://repo.zabbix.com/zabbix/5.0/rhel/7/x86_64/zabbix-release-5.0-1.el7.noarch.rpm
[root@data01 /server]# yum clean all
[root@data01 /server]# yum install -y zabbix-agent
```

![image-20210818230600005](./linux运维09 zabbix监控服务.assets\image-20210818230600005.png)

```bash
[root@data01 /server]# systemctl start zabbix-agent
[root@data01 /server]# systemctl enable zabbix-agent
```

## 安装

![image-20210818230700052](./linux运维09 zabbix监控服务.assets\image-20210818230700052.png)

![image-20210818230707982](./linux运维09 zabbix监控服务.assets\image-20210818230707982.png)

![image-20210818230713301](./linux运维09 zabbix监控服务.assets\image-20210818230713301.png)

![image-20210818230720711](./linux运维09 zabbix监控服务.assets\image-20210818230720711.png)

![image-20210818230727915](./linux运维09 zabbix监控服务.assets\image-20210818230727915.png)

![image-20210818230733923](./linux运维09 zabbix监控服务.assets\image-20210818230733923.png)

**user:Admin password:zabbix**

![image-20210818230753460](./linux运维09 zabbix监控服务.assets\image-20210818230753460.png)

zabbix乱码

![image-20210818230809144](./linux运维09 zabbix监控服务.assets\image-20210818230809144.png)

```bash
[root@m01 ~]# yum install wqy-microhei-fonts
[root@m01 ~]# cp /usr/share/fonts/wqy-microhei/wqy-microhei.ttc /usr/share/fonts/dejavu/DejaVuSans.ttf
```

![image-20210818230824439](./linux运维09 zabbix监控服务.assets\image-20210818230824439.png)

### zabbix怎么使用？

  10051 zabbix-server 服务端端口号

  10050 zabbix-agent 客户端端口号

   /etc/zabbix/web/zabbix.conf.php  -- 记录web页面初始化信息

#### 实现zabbix默认第一台主机的监控

创建组

![image-20210818230901074](./linux运维09 zabbix监控服务.assets\image-20210818230901074.png)

![image-20210818230905802](./linux运维09 zabbix监控服务.assets\image-20210818230905802.png)

创建主机

![image-20210818230916269](./linux运维09 zabbix监控服务.assets\image-20210818230916269.png)

![image-20210818230922133](./linux运维09 zabbix监控服务.assets\image-20210818230922133.png)

![image-20210818230928218](./linux运维09 zabbix监控服务.assets\image-20210818230928218.png)

![image-20210818230935277](./linux运维09 zabbix监控服务.assets\image-20210818230935277.png)

![image-20210818230940620](./linux运维09 zabbix监控服务.assets\image-20210818230940620.png)

第一步: 配置---主机---创建主机(创建要监控的主机)

第二步: 配置监控的主机

​      主机信息中: 名称 主机组 监控的主机地址

​      模板信息中: 指定需要链接的模板信息

 第三步: 保存退出,进行监控检查        

​      检查主机有没有变绿

       监测--最新数据

#### 实现zabbix自定义配置监控

  监控项: 可以自定义监控收集主机的信息

​     应用集: 将多个类似的监控项进行整合 便于查看检查

​     模板:  将多个监控项 触发器 图形都配置在模板中, 方便多个监控的主机进行调用

​     动作:    指定将报警信息发送给谁OK/定义报警的信息ok/定义报警的类型OK(邮件 微信 短信电话)

       PS: 宏信息定义方法:

         https://www.zabbix.com/documentation/4.0/zh/manual/appendix/macros/supported_by_location

​     触发器: 可以实现报警提示(条件表达式),默认页面提示报警

​     图形:  将多个图整合成一张,便于分析数据

​     报警媒介: 定义报警的方式

 

##### 实现zabbix自定义监控---监控项作用

  简单的自定义监控配置(单一服务状态)

  需求: 监控nginx服务是否启动

1) 在zabbix-agent进行配置文件编写

​     第一个历程: 编写自定义监控命令

```sh
   ps -ef|grep -c [n]ginx
```

   第二个历程: 编写zabbix-agent配置文件

   第一种方法: 直接修改zabbix-agent配置文件参数

```sh
  UserParameter=
```

![image-20210818231108275](./linux运维09 zabbix监控服务.assets\image-20210818231108275.png)

第二种方法: 在zabbix_agentd.d/目录中编写自定义监控文件

   vim web_server.conf

   UserParameter=键(变量名),值(变量信息)

```bash
 UserParameter=web_state,ps -ef|grep -c [n]ginx
 
	 Format: UserParameter=<key>,<shell command>
	 
[root@data01 /etc/zabbix]# cat zabbix_agentd.d/nfs_server.conf
UserParameter=nfs_server,ps -ef|grep -c [n]fsd

```

第三个历程: 重启zabbix-agent服务

```sh
[root@data01 /etc/zabbix]# systemctl restart zabbix-agent.service  
```

 2) 在zabbix-server命令行进行操作

​     第一个历程: 检测自定义监控信息是否正确

```sh
[root@m01 ~]# yum install -y zabbix-get
[root@m01 ~]# zabbix_get -s 192.168.1.200 -k 'nfs_server'
9
[root@data01 /etc/zabbix]# ps -ef|grep -c [n]fsd
9   
```

   3) 在zabbix-server网站页面进行配置

​     第一个历程: 进入到创建监控项页面:

​     配置---主机---选择相应主机的监控项

![image-20210818231309885](./linux运维09 zabbix监控服务.assets\image-20210818231309885.png)

![image-20210818231314731](./linux运维09 zabbix监控服务.assets\image-20210818231314731.png)

第二个历程: 监控项页面如何配置

​     名称 键值 更新间隔时间 应用集

![image-20210818231335243](./linux运维09 zabbix监控服务.assets\image-20210818231335243.png)

 第三个历程: 检查是否收集到监控信息

![image-20210818231348236](./linux运维09 zabbix监控服务.assets\image-20210818231348236.png)

![image-20210818231357727](./linux运维09 zabbix监控服务.assets\image-20210818231357727.png)

自行研究: 监控远程服务是否正常启动

###  复杂的自定义监控配置(多个服务状态)

1) 在zabbix-agent端编写配置文件

```bash
vim server_state.conf 
UserParameter=server_state[*],netstat -lntup|grep -c $1

[root@data01 /etc/zabbix]# vim zabbix_agentd.d/nfs_server.conf
UserParameter=server_status[*],netstat -lntup|grep -c $1     
[root@data01 /etc/zabbix]# systemctl restart zabbix-agent.service    
```

2) 在zabbix-server命令测试

```sh
   zabbix_get -s 172.16.1.7 -k 'server_state[22]'
```

![image-20210818231530668](./linux运维09 zabbix监控服务.assets\image-20210818231530668.png)

![image-20210818231535605](./linux运维09 zabbix监控服务.assets\image-20210818231535605.png)

![image-20210818231540710](./linux运维09 zabbix监控服务.assets\image-20210818231540710.png)

3) 修改配置页面

  键值: server_state[22]

![image-20210818231602543](./linux运维09 zabbix监控服务.assets\image-20210818231602543.png)

![image-20210818231609624](./linux运维09 zabbix监控服务.assets\image-20210818231609624.png)

### 实现zabbix报警功能---触发器/动作

触发器

![image-20210818231627058](./linux运维09 zabbix监控服务.assets\image-20210818231627058.png)

![image-20210818231632137](./linux运维09 zabbix监控服务.assets\image-20210818231632137.png)

![image-20210818231638041](./linux运维09 zabbix监控服务.assets\image-20210818231638041.png)

根据实际情况写表达式

![image-20210818231651020](./linux运维09 zabbix监控服务.assets\image-20210818231651020.png)

![image-20210818231657223](./linux运维09 zabbix监控服务.assets\image-20210818231657223.png)

![image-20210818231704683](./linux运维09 zabbix监控服务.assets\image-20210818231704683.png)

```bash
[root@data01 /etc/zabbix]# systemctl start rpcbind
```

![image-20210818231725425](./linux运维09 zabbix监控服务.assets\image-20210818231725425.png)

这样就恢复了。但是一般情况不能够及时发现，完美需要报警声音或者短信，电话通知。

### 报警方式:

   01. 页面提示信息报警(值班运维)   

   02. 页面声音提示报警      

   03. 邮件信息报警            

   04. 微信功能报警              

   05. 短信报警/电话报警            

   

#### 页面提示和声音报警实践:

​     第一个历程: 创建触发器

​     配置---主机---选择相应监控主机触发器---创建触发器 

​     设置好表达式

   {web01:server_state[nginx].last()}<=2

​     {监控主机名称:键值名称.调用的表达式函数}<=2 

​     表达式总结:

   last()  收集到最新信息(数值) *****

​     max()     在一定周期内,收集到的最大值

  min()  在一定周期内,收集到的最小值

​     diff()  在一定时间内,判断收集的信息是否不同

​     change() 在一定时间内,判断收集的信息是否不同

   avg()  取一段时间的平均值  

------

企业工作遇见告警信息处理思路步骤:

   第一步: 看到告警提示信息,定位主机信息

   第二步: 看到主机信息之后,定位报警原因 获得监控项Key值

   第三步: 根据key值信息,最终获得报警原因

------

 上面触发器参考截图

![image-20210818231845057](./linux运维09 zabbix监控服务.assets\image-20210818231845057.png)

![image-20210818231851396](./linux运维09 zabbix监控服务.assets\image-20210818231851396.png)

至此: 已经看到提示报警

  

​     第二个历程: 修改网页配置,有提升声音报警

​     小人头---正在发送消息---前端信息勾选

   PS: 如何修改报警铃声:

​     1) 找到文件所在目录(在站点目录中找)

   find /usr/share/zabbix -type f -name "alarm_disaster.wav"

​     2) 将原有声音文件做替换

​     替换成指定声音文件

​     3) 需要清除浏览器缓存,进行测试

#### 邮件信息报警

​     第一个历程: 创建触发器

​     配置---主机---选择相应监控主机触发器---创建触发器 

​     设置好表达式

   {web01:server_state[nginx].last()}<=2

​     {监控主机名称:键值名称.调用的表达式函数}<=2 

 

  第二个历程: 修改动作配置

  配置---动作---将默认动作进行开启  

![image-20210818231933069](./linux运维09 zabbix监控服务.assets\image-20210818231933069.png)

第三个历程: 建立和163邮箱服务关系

​     管理---报警媒介类型---创建报警媒介

![image-20210818231945680](./linux运维09 zabbix监控服务.assets\image-20210818231945680.png)

![image-20210818231951017](./linux运维09 zabbix监控服务.assets\image-20210818231951017.png)

![image-20210818231956840](./linux运维09 zabbix监控服务.assets\image-20210818231956840.png)

 第四个历程: 定义接收报警的邮件地址

​     小人头--报警媒介--设置收件人信息

![image-20210818232010623](./linux运维09 zabbix监控服务.assets\image-20210818232010623.png)

![image-20210818232016377](./linux运维09 zabbix监控服务.assets\image-20210818232016377.png)

![image-20210818232023934](./linux运维09 zabbix监控服务.assets\image-20210818232023934.png)

![image-20210818232031307](./linux运维09 zabbix监控服务.assets\image-20210818232031307.png)

OK ，不管是故障还是恢复都会收到消息

![image-20210818232042975](./linux运维09 zabbix监控服务.assets\image-20210818232042975.png)

里面可以改成中文。

#### 微信报警

  第一个历程: 需要注册企业微信,并进行配置

   我的企业: 

   01. 获取企业id: ww32d68104ab5f51b0

​     02. 获取企业二维码: 允许员工加入

   管理工具:

   01. 成员加入---进行审核通过

​     应用小程序:

   01. 进行创建

   02. 收集程序信息

     AgentId: 1000006

      Secret: RvQYpaCjWbYMCcwhnPqg1ZYcEGB9cOQCvvlkn-ft6j4

      

   第二个历程: 编写脚本(python)

   cat /etc/zabbix/zabbix-server.conf 

   AlertScriptsPath=/usr/lib/zabbix/alertscripts --- 放置告警脚本

   

  执行脚本报错问题解决:

   01. 问题: No module named requests

   yum install -y python-pip

  pip install requests

   02. 问题: 脚本执行语法

   

  第三个历程: 修改添加报警媒介---定义了发微信配置

   

   第四个历程: 配置接收微信的人员

   

#### 短信和电话:

​     利用第三方短信电话报警平台

   01. 利用阿里大鱼(收费)

     https://yq.aliyun.com/articles/658524?spm=a2c4e.11155472.0.0.d821153fAjrH3q --- 自行研究

   02. 利用onealert发送告警

​     第一个历程: 配置报警平台

   01. 配置--应用--选择zabbix报警

   02. 配置--通知策略

   03. 配置--分派策略

实现zabbix图形配置---图形

配置--主机--图形--(监控项)





# 综合架构需求:

简单: yum本地仓库 jumpserver跳板机 pptpvpn 时间同步服务器 cobbler批量部署系统 

扩展: mysql高可用 nfs高可用 tomcat服务 实现防火墙配置(主机没有外网如何进行上网) 实现nginx缓存功能

   HTTPS访问网站 如何实现session会话功能(memcache)

高级: 如何利用ansible一键化部署

