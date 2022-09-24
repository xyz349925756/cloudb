web标准：https://w3techs.com/technologies/overview/web_server

# nginx

## 概念

HTTP协议资源信息

URL: 全称为Uniform Resource Location，中文翻译为统一资源定位符

URI: 全称为Uniform Resource Identifier，中文翻译为统一资源标识符

docs.ansible.com       /      ansible/latest/user_guide/playbooks_reuse_roles.html

​                  URL                         URI

评测网站好坏的指标:

IP: 根据用户IP地址数量进行统计  300万~400万 一个星期

局域网多个用户访问: 网站服务器只记录一个IP访问  NAT技术

断电了家用电脑会重新拨号: wan--公网地址

PV: 页面访问量          700~800万   一个星期

参考值:

UV: 记录独立访客数量

cookie: 标识用户身份信息, 会保存在用户客户端本地           内存中

session: 记录用户的一些会话操作, eg: 记录用户登录信息 记录在服务端   内存中

网站的并发: (压测技术 1 10 100 1000 10000)

A种理解：网站服务器每秒能够接收的最大用户请求数。

B种理解：网站服务器每秒能够响应的最大用户请求数。

备注：很多地方宽带现在采用局中局很多家庭使用一个公网IP地址，可能是1000家/IP网站访问需要重新评估定义

## Nginx的配置优化及常见问题

## nginx的安装

安装nginx

下载nginx http://nginx.org/en/download.html

### 方法一：yum安装

配置yum源

http://nginx.org/en/linux_packages.html

```bash
sudo yum install yum-utils
```

 **/etc/yum.repos.d/nginx.repo**

```BASH
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
```

```bash
sudo yum-config-manager --enable nginx-mainline
```

```bash
sudo yum install nginx
```

### 方法二：编译安装

安装组件,安装依赖包

```bash
[root@localhost~]#yum install -y pcre pcre-devel openssl openssl-devel
```

创建nginx虚拟用户和组并且不赋予登录

```bash
[root@localhost~]# useradd -s /sbin/nologin -M nginx
[root@localhost~]# tar xf nginx-1.16.1.tar.gz
[root@localhost~]# mkdir /application/nginx-1.16.1
[root@localhost~]# cd nginx-1.16.1
[root@localhost nginx-1.16.1]# ./configure --user=nginx --group=nginx --prefix=/application/nginx-1.16.1/ --with-http_stub_status_module --with-http_ssl_module
```

```bash
a 进行配置操作
        ./configure --prefix=  --user=USER 
        --prefix=PATH     set installation prefix    
	 	                   指定程序安装路径
	 	 --user=USER       set non-privileged user for worker processes
	 	                   设置一个虚拟用户管理worker进程(安全)
        --group=GROUP     set non-privileged group for worker processes
                          设置一个虚拟用户组管理worker进程(安全)
b 进行软件的编译过程:
        make 编译(翻译)	 
        C(英国人)       代码(任务文件)  系统识别  翻译  系统(中国人)处理代码
	 	 翻译效率比较低  翻译官(gcc)
	 	 python(日本人)  代码(任务文件)  系统识别  翻译  系统(中国人)处理代码
        翻译效率比较高  百度翻译(python解释器)  
	 	 
c 编译安装过程
	 	 make install
```

创建nginx目录软连接，防止后期误删

```bash
[root@localhost nginx-1.16.1]# make && make install
[root@localhost nginx-1.16.1]# ln -s /application/nginx-1.16.1 /application/nginx
```

语法检查

```BASH
[root@localhost tools]# /application/nginx/sbin/nginx -t
```

创建nginx命令软连接，方便后期直接使用，还有一种方法是直接再path中添加环境路径

```bash
[root@localhost tools]# ln -sv /application/nginx/sbin/nginx /usr/bin/nginx
[root@localhost tools]# nginx
```

![image-20210818191957025](./linux运维07web服务及负载均衡.assets\image-20210818191957025.png)

```bash
[root@localhost tools]# lsof -i :80
COMMAND  PID  USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
nginx   5972  root    6u  IPv4  52132      0t0  TCP *:http (LISTEN)
nginx   6222 nginx    6u  IPv4  52132      0t0  TCP *:http (LISTEN)
[root@localhost tools]# netstat -lnt|grep 80
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN     
[root@localhost tools]# wget 172.16.0.200

[root@localhost tools]# wget 127.0.0.1

[root@localhost cmatrix-1.2a]# grep html /application/nginx/conf/nginx.conf
            root   html;
            index  index.html index.htm;
        #error_page  404              /404.html;
        # redirect server error pages to the static page /50x.html
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        #    root           html;
    #        root   html;
    #        index  index.html index.htm;
    #        root   html;
    #        index  index.html index.htm;
这个就是存放html的目录。

[root@localhost nginx]# ls html/
50x.html    index.html  

我们把需要的网站源代码放到这个目录就可以看到网站了。
看下初始的nginx.default 文件已经去除了#号。

[root@localhost nginx]# egrep -v '#|^$' conf/nginx.conf.default 
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       80;
        server_name  localhost;
        location / {
            root   html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}

```



## nginx日志定时切割

### 方法一

**/etc/logrotate.d/**

```bash
[root@web01 /server/tools]# ls /etc/logrotate.d/
bootlog  chrony  firewalld  nginx  syslog  wpa_supplicant  yum
```

> Linux系统默认安装logrotate工具，它默认的配置文件在：
>
> /etc/logrotate.conf
>
> /etc/logrotate.d/

logrotate.conf 是主要的配置文件，logrotate.d 是一个目录，该目录里的所有文件都会被主动的读入/etc/logrotate.conf中执行。

另外，如果 /etc/logrotate.d/ 里面的文件中没有设定一些细节，则会以/etc/logrotate.conf这个文件的设定来作为默认值。

Logrotate是基于CRON来运行的，其脚本是/etc/cron.daily/logrotate，日志轮转是系统自动完成的。

实际运行时，Logrotate会调用配置文件/etc/logrotate.conf。

可以在/etc/logrotate.d目录里放置自定义好的配置文件，用来覆盖Logrotate的缺省值。

可以切割很多服务的日志

```bash
[root@web01 /server/tools]# cat /etc/logrotate.d/nginx 
/var/log/nginx/*.log {
        daily
        missingok
        rotate 52
        compress
        delaycompress
        notifempty
        create 640 nginx adm
        sharedscripts
        postrotate
                if [ -f /var/run/nginx.pid ]; then
                        kill -USR1 `cat /var/run/nginx.pid`
                fi
        endscript
}
```

解析：

```bash
[root@web01 /server/tools]# cat /etc/logrotate.conf 
# see "man logrotate" for details  查看详细帮助
# rotate log files weekly
weekly
定义默认日志切割的周期
# keep 4 weeks worth of backlogs
rotate 4
定义只保留切割后的几个文件
# create new (empty) log files after rotating old ones
create
创建一个相同的源文件
# use date as a suffix of the rotated file
dateext
定义角标（扩展名称信息）
# uncomment this if you want your log files compressed
#compress
是否对切割后文件压缩
# RPM packages drop log rotation information into this directory
include /etc/logrotate.d
加载/etc/logrotate.d目录中配置文件
# no packages own wtmp and btmp -- we'll rotate them here
/var/log/wtmp {     单独对某个文件切割配置
    monthly
    create 0664 root utmp
	 minsize 1M      最小大小为1M，小于1M不进行切割
    rotate 1
}

/var/log/btmp {
    missingok
    monthly
    create 0600 root utmp
    rotate 1
}

# system-specific logs may be also be configured here.

```

```bash
[root@web01 /server/tools]# logrotate /etc/logrotate.conf 
[root@web01 /server/tools]# date -s "20210314"
2021年 03月 14日 星期日 00:00:00 CST
[root@web01 /server/tools]# logrotate /etc/logrotate.conf 
[root@web01 /server/tools]# date -s "20210315"
2021年 03月 15日 星期一 00:00:00 CST
[root@web01 /server/tools]# logrotate /etc/logrotate.conf 
[root@web01 /server/tools]# ls /var/log/nginx/
access.log           access.log-20210314  error.log            error.log-20210314   
[root@web01 /server/tools]# ls /var/log/nginx/
access.log  access.log-20210314  error.log  error.log-20210314
```

如果等不及cron自动执行日志轮转，想手动强制切割日志，需要加-f参数；不过正式执行前最好通过Debug选项来验证一下（-d参数），这对调试也很重要

```bash
# /usr/sbin/logrotate -f /etc/logrotate.d/nginx
# /usr/sbin/logrotate -d -f /etc/logrotate.d/nginx
logrotate命令格式：
logrotate [OPTION...] <configfile>
-d, --debug ：debug模式，测试配置文件是否有错误。
-f, --force ：强制转储文件。
-m, --mail=command ：压缩日志后，发送日志到指定邮箱。
-s, --state=statefile ：使用指定的状态文件。
-v, --verbose ：显示转储过程。
```

#### logrotate 日志自动发送邮件

邮件配置

```BASH
[root@lnmp ~]# yum install sendmail
[root@lnmp ~]# yum install mailx
[root@lnmp ~]# vi /etc/mail.rc
set from=wadshong5220@126.com
set smtp=smtp.126.com
set smtp-auth-user=wadshong5220@126.com
set smtp-auth-password=***********     #这里是设置密码
set smtp-auth=login
```

nginx配置

```BASH
[root@web01 /var/log/nginx]# cat /etc/logrotate.d/nginx 
/var/log/nginx/*.log {
        daily
        missingok
        rotate 52
        mail 349925756@qq.com
        mailfirst      默认是过期日志才发送邮件
        compress
        delaycompress
        notifempty
        create 640 nginx adm
        sharedscripts
        postrotate
                if [ -f /var/run/nginx.pid ]; then
                        kill -USR1 `cat /var/run/nginx.pid`
                fi
        endscript
}
[root@web01 /var/log/nginx]# logrotate -f /etc/logrotate.conf
强制执行分割

```

![image-20210818192718102](./linux运维07web服务及负载均衡.assets\image-20210818192718102.png)

#### 重要参数

```bash
compress                                   通过gzip 压缩转储以后的日志
nocompress                                不做gzip压缩处理
copytruncate                              用于还在打开中的日志文件，把当前日志备份并截断；是先拷贝再清空的方式，拷贝和清空之间有一个时间差，可能会丢失部分日志数据。
nocopytruncate                           备份日志文件不过不截断
create mode owner group             轮转时指定创建新文件的属性，如create 0777 nobody nobody
nocreate                                    不建立新的日志文件
delaycompress                           和compress 一起使用时，转储的日志文件到下一次转储时才压缩
nodelaycompress                        覆盖 delaycompress 选项，转储同时压缩。
missingok                                 如果日志丢失，不报错继续滚动下一个日志
errors address                           专储时的错误信息发送到指定的Email 地址
ifempty                                    即使日志文件为空文件也做轮转，这个是logrotate的缺省选项。
notifempty                               当日志文件为空时，不进行轮转
mail address                             把转储的日志文件发送到指定的E-mail 地址
nomail                                     转储时不发送日志文件
olddir directory                         转储后的日志文件放入指定的目录，必须和当前日志文件在同一个文件系统
noolddir                                   转储后的日志文件和当前日志文件放在同一个目录下
sharedscripts                           运行postrotate脚本，作用是在所有日志都轮转后统一执行一次脚本。如果没有配置这个，那么每个日志轮转后都会执行一次脚本
prerotate                                 在logrotate转储之前需要执行的指令，例如修改文件的属性等动作；必须独立成行
postrotate                               在logrotate转储之后需要执行的指令，例如重新启动 (kill -HUP) 某个服务！必须独立成行
daily                                       指定转储周期为每天
weekly                                    指定转储周期为每周
monthly                                  指定转储周期为每月
rotate count                            指定日志文件删除之前转储的次数，0 指没有备份，5 指保留5 个备份
dateext                                  使用当期日期作为命名格式
dateformat .%s                       配合dateext使用，紧跟在下一行出现，定义文件切割后的文件名，必须配合dateext使用，只支持 %Y %m %d %s 这四个参数
size(或minsize) log-size            当日志文件到达指定的大小时才转储，log-size能指定bytes(缺省)及KB (sizek)或MB(sizem).
当日志文件 >= log-size 的时候就转储。 以下为合法格式：（其他格式的单位大小写没有试过）
size = 5 或 size 5 （>= 5 个字节就转储）
size = 100k 或 size 100k
size = 100M 或 size 100M
```

### 方法二：脚本

利用脚本实现切割

```bash

#!/bin/bash
mv /var/log/nginx/access.log  /var/log/nginx/access_$(date +%F).log
systemctl restart nginx

```

```bash
[root@www ~]# cat -n /application/nginx/conf/cut_nginx_log.sh 
 #!/bin/sh
 Dateformat=$(date +%Y_%m_%d)
 Basedir="/application/nginx"
 Nginxlogdir="$Basedir/logs"
 Logname='access_www'
 [ -d $Nginxlogdir ] && cd $Nginxlogdir||exit 1
 [ -f ${Logname}.log ]||exit 1
 /bin/mv ${Nginxlogdir}/${Logname}.log ${Nginxlogdir}/${Logname}_${Dateformat}.log
 $Basedir/sbin/nginx -s reload
 
[root@www conf]# sh cut_nginx_log.sh 
[root@www conf]# ls ../logs/
access_bbs.log   access_www_2020_05_15.log         nginx.pid
access_blog.log   access.log       access_www.log             error.log
[root@www ~]# curl www.xxx.org
http://www.xxx.org
[root@www conf]# cat ../logs/access_www.log 
172.16.0.200 - - [15/May/2020:21:49:46 +0800] "GET / HTTP/1.1" 200 19 "-" "curl/7.29.0" "-"
[root@www conf]# sh cut_nginx_log.sh 
[root@www conf]# cat ../logs/access_www_2020_05_15.log 
172.16.0.200 - - [15/May/2020:21:49:46 +0800] "GET / HTTP/1.1" 200 19 "-" "curl/7.29.0" "-"
这里是手动执行的是不是很费时。那么可以结合计划任务来实现自动功能。
[root@www ~]# cat >>/var/spool/cron/root <<EOF
> #cut nginx access log by oldboy
> 00 00 * * * /bin/sh /application/nginx/conf/cut_nginx_log.sh >/dev/null 2>&1
> EOF

[root@www ~]# crontab -l
#cut nginx access log by oldboy
00 00 * * * /bin/sh /application/nginx/conf/cut_nginx_log.sh >/dev/null 2>&1

nginx 常用日志工具有 rsyslog awstats flume ELK storm等。
```

## nginx服务配置文件

```BASH
[root@web01 /server/tools]# cat /etc/nginx/nginx.conf 

user  nginx;
worker_processes  1;             

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;

----------------------------第一部分

events {
    worker_connections  1024;
}
----------------------------第二部分

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;

----------------------------第三部分

server {
    listen       8080;                --- 指定监听的端口
    server_name  www.oldboy.com;      --- 指定网站域名               
    root   /usr/share/nginx/html;     --- 定义站点目录的位置
    index  index.html index.htm;      --- 定义首页文件
    error_page   500 502 503 504  /50x.html;   --- 优雅显示页面信息
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
} 

第三部分里面包含第四部分

----------------------------第四部分

```

```bash
/etc/nginx/nginx.conf      	  --- 主配置文件
第一个部分: 配置文件主区域配置
user  www;               	  --- 定义worker进程管理的用户
补充: nginx的进程
master process:  主进程	 	 ---管理服务是否能够正常运行   boss
worker process:  工作进程	 ---处理用户的访问请求         员工  
worker_processes  2;        ---定义有几个worker进程  == CPU核数 / 核数的2倍
error_log  /var/log/nginx/error.log warn;   --- 定义错误日志路径信息
pid        /var/run/nginx.pid;              --- 定义pid文件路径信息

第二个部分: 配置文件事件区域
events {                    
    worker_connections  1024;   --- 一个worker进程可以同时接收1024访问请求
}

第三个部分: 配置http区域
http {
    include       /etc/nginx/mime.types;      --- 加载一个配置文件
    default_type  application/octet-stream;   --- 指定默认识别文件类型
    log_format  oldboy  '$remote_addr - $remote_user [$time_local] "$request" '
                        '$status $body_bytes_sent "$http_referer" '
                        '"$http_user_agent" "$http_x_forwarded_for"';
	                   --- 定义日志的格式	 	 
    access_log  /var/log/nginx/access.log  oldboy;
	                   --- 指定日志路径          
    sendfile        on;   ???
    #tcp_nopush     on;   ???
    keepalive_timeout  65;   --- 超时时间
    #gzip  on;
    include /etc/nginx/conf.d/*.conf;        --- 加载一个配置文件
}


/etc/nginx/nginx.d/default  --- 扩展配置(虚拟主机配置文件)
第四个部分: server区域信息(配置一个网站 www/bbs/blog -- 一个虚拟主机)
server {
    listen       8080;                --- 指定监听的端口
    server_name  www.oldboy.com;      --- 指定网站域名                     
    root   /usr/share/nginx/html;     --- 定义站点目录的位置
    index  index.html index.htm;      --- 定义首页文件
    error_page   500 502 503 504  /50x.html;   --- 优雅显示页面信息
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}

```

### 虚拟主机

#### 1、基于域名的虚拟主机

```bash
[root@localhost nginx]# egrep -v '#|^$' conf/nginx.conf.default > conf/nginx.conf
[root@localhost nginx]# cat conf/nginx.conf
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       80;
        server_name  localhost;
        location / {
            root   html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
[root@localhost nginx]# vi conf/nginx.conf
[root@localhost nginx]# cat conf/nginx.conf
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       80;
        server_name  www.xxx.org;
        location / {
             root   html/xxx.org;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```

直接使用命令把deflaut里面没有#的文件弄过来做nginx.conf配置文件。修改上面标红部分既可。

把这个域名下的网站代码放进去。为了规范使用可以在html目录下重新创建对应域名的文件夹以备使用。

```bash
[root@localhost nginx]# nginx -t
[root@localhost nginx]# nginx -s reload
[root@localhost nginx]# ps -ef|grep nginx
root       5972      1  0 10:37 ?        00:00:00 nginx: master process /application/nginx-1.16.1/sbin/nginx
nginx     19265   5972  0 14:58 ?        00:00:00 nginx: worker process
root      19318   2124  0 14:59 pts/0    00:00:00 grep --color=auto nginx
[root@localhost nginx]# netstat -lntup |grep 80
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      5972/nginx: master  
检查域名
[root@localhost nginx]# echo "172.16.0.200 www.xxx.org" >> /etc/hosts 
[root@localhost nginx]# curl www.xxx.org
这里是linux下面的测试的，windows的需要修改hosts文件。
```

##### 基于多个主机的域名虚拟主机配置。

```bash
[root@www conf]# cat nginx.conf
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       80;
        server_name  www.xxx.org;
        location / {
            root   html/www;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
    server {
        listen       80;
        server_name  blog.xxx.org;
        location / {
            root   html/blog;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
     }
    server {
        listen       80;
        server_name  bbs.xxx.org;
        location / {
            root   html/bbs;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
   }
}

[root@www conf]# tree ../html
../html
├── 50x.html
├── bbs
│   └── index.html
├── blog
│   └── index.html
├── index.html
├── www
│   └── index.html
└── xxx.org
    └── index.html

4 directories, 6 files
[root@www conf]# echo "172.16.0.200 www.xxx.org" >> /etc/hosts
[root@www conf]# echo "172.16.0.200 blog.xxx.org" >> /etc/hosts
[root@www conf]# echo "172.16.0.200 bbs.xxx.org" >> /etc/hosts
[root@www conf]# nginx -t
[root@www conf]# nginx -s reload

再验证一下这三个网站
[root@www conf]# for n in www bbs blog; 
do  
mkdir -p ../html/$n;
 echo "http://${n}.xxx.org" > ../html/$n/index.html;
 cat ../html/$n/index.html;
done
http://www.xxx.org
http://bbs.xxx.org
http://blog.xxx.org
[root@www conf]# curl www.xxx.org
http://www.xxx.org
[root@www conf]# curl bbs.xxx.org
http://bbs.xxx.org
[root@www conf]# curl blog.xxx.org
http://blog.xxx.org
```

#### 2、基于端口的虚拟主机

基于端口的虚拟主机大部分是在**内网使用**。少部分外网是管理员不会nginx。

```bash
[root@www conf]# cat nginx.conf
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       80;
        server_name  www.xxx.org;
        location / {
            root   html/www;
            index  index.html index.htm;
        }
…

如下配置
server {
        listen       80;
        server_name  www.xxx.org;
        location / {
            root   html/www;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
    server {
        listen       81;
        server_name  blog.xxx.org;
        location / {
            root   html/blog;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
     }
    server {
        listen       82;
        server_name  bbs.xxx.org;
        location / {
            root   html/bbs;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

[root@www conf]# nginx -t
[root@www conf]# nginx -s reload
```

![image-20210818193807572](./linux运维07web服务及负载均衡.assets\image-20210818193807572.png)

![image-20210818193818833](./linux运维07web服务及负载均衡.assets\image-20210818193818833.png)

#### 3、基于IP地址的虚拟主机

这个也很简单。直接看配置文件

![image-20210818193852931](./linux运维07web服务及负载均衡.assets\image-20210818193852931.png)

访问参考上面基于端口访问。

##### nginx的检测策略

```bash
#!/bin/sh
#author:oldboy
. /etc/init.d/functions
function checkURL()
{
    checkUrl=$1
    echo 'check url start ...'
    judge=($(curl -I -s --connect-timeout 2 ${checkUrl}|head -1|tr " " "\n"))
    if [[ "${judge[1]}"=='200' && "${judge[2]}"=='OK' ]]
       then
           action "${checkUrl}" /bin/true
    else
           action "${checkUrl}" /bin/false
           echo -n "retrying again...";sleep 3;
           judgeagain=($(curl -I -s --connect-timeout 2 ${checkUrl}|head -1|tr "\r" "\n"))
           if [[ "${judgeagain[1]}" == '200' && "${judgeagain[2]}" == 'OK' ]]
              then
                  action "${checkUrl},retried again" /bin/true
           else
                  action "${checkUrl},retried again" /bin/false
           fi
    fi
    sleep 1;

}
#usage method
checkURL http://www.xxx.org
checkURL http://bbs.xxx.org
checkURL http://blog.xxx.org
shell 编程中很多地方需要注意空格。
[root@www conf]# sh check_url.sh 
check url start ...
http://www.xxx.org                                         [  确定  ]
check url start ...
http://bbs.xxx.org                                         [  确定  ]
check url start ...
http://blog.xxx.org                                        [  确定  ]

```

## 部署搭建网站常见错误:

1. 网站服务配置文件编写不正确

  404 错误

   解决方法一: 修改nginx配置文件---location

   解决方法二: 在站点目录中创建相应目录或文件数据信息

   403 错误

   解决方法一: 不要禁止访问

   解决方法二: 因为没有首页文件 

2. DNS信息配置不正确

3. nginx配置文件修改一定要重启服务;

  站点目录中代码文件信息调整,不需要重启服务

​     4.访问异常排查思路

  首先检查站点所属用户组，根据nginx.conf 里面的user 默认是nginx

对应的html 所有者 chown -R nginx.nginx /html/

权限对应  chmod -R 744 或者其他，根据实际情况调整

对于访问先看端口是否启动lsof -I :80 / netstat -lantup |grep nginx

5. 服务配置文件中涉及到地址修改,必须重启nginx服务,不能平滑重启

### nginx常用功能实战

#### 优化规范nginx的配置文件

就是把虚拟主机的子文件放到extra目录中去。这里使用的是include参数

include 语法：include file|mask;

```bash
[root@www conf]# cat -n nginx.conf
     1	 worker_processes  1;
     2	 events {
     3	     worker_connections  1024;
     4	 }
     5	 http {
     6	     include       mime.types;
     7	     default_type  application/octet-stream;
     8	     sendfile        on;
     9	     keepalive_timeout  65;
    10	     server {
    11	         listen       80;
    12	         server_name  www.xxx.org;
    13	         location / {
    14	             root   html/www;
    15	             index  index.html index.htm;
    16	         }
    17	         error_page   500 502 503 504  /50x.html;
    18	         location = /50x.html {
    19	             root   html;
    20	         }
    21	     }
    22	     server {
    23	         listen       81;
    24	         server_name  blog.xxx.org;
    25	         location / {
    26	             root   html/blog;
    27	             index  index.html index.htm;
    28	         }
    29	         error_page   500 502 503 504  /50x.html;
    30	         location = /50x.html {
    31	             root   html;
    32	         }
    33	      }
    34	     server {
    35	         listen       82;
    36	         server_name  bbs.xxx.org;
    37	         location / {
    38	             root   html/bbs;
    39	             index  index.html index.htm;
    40	         }
    41	         error_page   500 502 503 504  /50x.html;
    42	         location = /50x.html {
    43	             root   html;
    44	         }
    45	    }
    46	 }

```

```bash
[root@www conf]# sed -n '10,21p' nginx.conf
    server {
        listen       80;
        server_name  www.xxx.org;
        location / {
            root   html/www;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
[root@www conf]# sed -n '10,21p' nginx.conf > extra/www.conf
[root@www conf]# sed -n '22,33p' nginx.conf > extra/blog.conf
[root@www conf]# sed -n '34,45p' nginx.conf > extra/bbs.conf
[root@www conf]# sed -i '10,45d' nginx.conf   删除10，45行
[root@www conf]# cat -n nginx.conf
     1	 worker_processes  1;
     2	 events {
     3	     worker_connections  1024;
     4	 }
     5	 http {
     6	     include       mime.types;
     7	     default_type  application/octet-stream;
     8	     sendfile        on;
     9	     keepalive_timeout  65;
     10	 }
     
关联三个虚拟主机文件
[root@www conf]# cat nginx.conf
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    include extra/www.conf；
    include extra/blog.conf；
    include extra/bbs.conf；
}
[root@www conf]# nginx -t
[root@www conf]# nginx -s reload
[root@www conf]# sed -i "10 i include extra/www.conf;\ninclude extra/blog.conf;\ninclude extra/bbs.conf;" nginx.conf
[root@www conf]# cat -n nginx.conf
     1	 worker_processes  1;
     2	 events {
     3	     worker_connections  1024;
     4	 }
     5	 http {
     6	     include       mime.types;
     7	     default_type  application/octet-stream;
     8	     sendfile        on;
     9	     keepalive_timeout  65;
    10	 include extra/www.conf;
    11	 include extra/blog.conf;
    12	 include extra/bbs.conf;
    13	 }

nginx status 
状态模块，可以查看基本访问信息
[root@www ~]# nginx -V
nginx version: nginx/1.16.1
built by gcc 10.1.0 (GCC) 
built with OpenSSL 1.0.2k-fips  26 Jan 2017
TLS SNI support enabled
configure arguments: --user=nginx --group=nginx --prefix=/application/nginx-1.16.1/ --with-http_stub_status_module --with-http_ssl_module
标红模块必须增加。
[root@www ~]# cat >>/application/nginx/conf/extra/status.conf<<EOF
###status
server{
      listen 80;
      server_name status.xxx.org;
      location / {
         stub_status on;
         access_log  off;
      }
    }
EOF
[root@www ~]# cat -n /application/nginx/conf/nginx.conf
     1	 worker_processes  1;
     2	 events {
     3	     worker_connections  1024;
     4	 }
     5	 http {
     6	     include       mime.types;
     7	     default_type  application/octet-stream;
     8	     sendfile        on;
     9	     keepalive_timeout  65;
    10	 include extra/www.conf;
    11	 include extra/blog.conf;
    12	 include extra/bbs.conf;
    13	 }
[root@www ~]# sed -i "13 i include extra/status.conf;" /application/nginx/conf/nginx.conf
[root@www ~]# nginx -t
[root@www ~]# nginx -s reload
```

![image-20210818194324749](./linux运维07web服务及负载均衡.assets\image-20210818194324749.png)

#### 统计访问量

```bash
###status
server{
      listen 84;
      server_name status.xxx.org;
      location / {
         stub_status on;
         access_log  off;
         allow 172.16.0.0/24;    只允许172.16.0.0网段的访问
         deny all;              禁止所有用户访问
      }
    }
```

![image-20210818194344392](./linux运维07web服务及负载均衡.assets\image-20210818194344392.png)

server 表示nginx启动到现在处理了4个请求
accepts 表示nginx启动到现在创建了4次握手
handled requests 表示处理了3次请求
请求丢失数=握手数-连接数 上图并没有数据丢失
reading 读取客户端的header信息数
writing 返回给客户的header信息数
waiting 正在等待下一次请求指令的驻留链接。开启keep-ative的情况下
active-(reading +writing)



#### nginx错误日志

日志是记录软件在运行过程中出现故障的凭证，是Linux下很重要的排错信息。
错误日志属于核心模块功能之一。ngc_core_module的参数。
语法: 
error_log        file             level
关键字        日志文件         错误日志级别
关键字不能变，日志文件就是日志的路径  
错误日志级别常见的有[debug|info|notice|warn|error|crit|alert|emerg]
warn  error   crit 这三个是最常见的配置INFO级别会导致磁盘IO消耗。
error_log  logs/error.log  error
放置标签为，main,http,server,location

```bash
[root@www conf]# sed -i "2 i error_log  logs/error.log;" nginx.conf
[root@www conf]# cat -n nginx.conf
     1	 worker_processes  1;
     2	 error_log  logs/error.log;
     3	 events {
     4	     worker_connections  1024;
     5	 }
     6	 http {
     7	     include       mime.types;
     8	     default_type  application/octet-stream;
     9	     sendfile        on;
    10	     keepalive_timeout  65;
    11	 include extra/www.conf;
    12	 include extra/blog.conf;
    13	 include extra/bbs.conf;
    14	 include extra/status.conf;
    15	 }
```

```bash
error_log  /var/log/nginx/error.log warn;
错误级别:
debug	 	 :调试级别, 服务运行的状态信息和错误信息详细显示     信息越多
info        :信息级别, 只显示重要的运行信息和错误信息
notice      :通知级别: 更加重要的信息进行通知说明
warn        :警告级别: 可能出现了一些错误信息,但不影响服务运行
error	 	 :错误级别: 服务运行已经出现了错误,需要进行纠正      推荐选择
crit        :严重级别: 必须进行修改调整
alert       :严重警告级别: 即警告,而且必须进行错误修改
emerg       :灾难级别: 服务已经不能正常运行                      信息越少
```

#### nginx访问日志

nginx记录访问者的信息到指定文件中，方便后期分析。由ngx_http_log_module负责。

参数

log_format  用来定义记录日志格式（可以定义多种，取名隔离）

access_log  用来指定日志文件路径及何种格式记录日志

 ```bash
 default :
 log_format main  ‘$remote_addr -$remote_user [$time_local] “$request” ’
           ‘$status $body_bytes_sent “$http_referer” ’
           ‘ ”$http_user_agent” “$http_x_forwarded_for” ’;
 ```

access_log logs/access.log  main; 

| Nginx变量             | 说明                                     |
| --------------------- | ---------------------------------------- |
| $remote_addr          | 地址                                     |
| $remote_user          | 远程客户端用户名                         |
| $time_local           | 记录访问时间与时区                       |
| $request              | 用户http请求起始信息                     |
| $status               | 状态 200   404                           |
| $body_bytes_sent      | 服务器发给客户端相应body字节数           |
| $http_referer         | 记录此次请求是从那个地方访问过来的       |
| $http_user_agent      | 记录客户端信息，手机或者电脑，IE或者火狐 |
| $http_x_forwarded_for | 记录代理服务器                           |

access_log参数说明

语法：

access_log path [format [buffer=size [flush=time]][if=condition]];

access_log path format gzip[=level] [buffer=size] [flush=time] [if=condition];

access_log syslog:server=address[,parameter=value] [format [if=condition]];

buffer=size 为存放访问日志的缓冲区大小

flush=time 将缓冲区的日志刷到磁盘的时间

gzip[=level] 表示压缩等级

[if=condition] 表示其他条件

一般情况下这些不需要配置，极端情况才需要配置。



deflaut

access_log logs/access.log combined;

放置位置：http,server,location,if in location,limit_except中。

```bash
[root@www conf]# cat nginx.conf
worker_processes  1;
error_log  logs/error.log;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    sendfile        on;
    keepalive_timeout  65;
include extra/www.conf;
include extra/blog.conf;
include extra/bbs.conf;
include extra/status.conf;
}

[root@www conf]# sed -n '21,23 s/#//gp' nginx.conf.default 
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
[root@www conf]# vi nginx.conf
[root@www conf]# cat nginx.conf
worker_processes  1;
error_log  logs/error.log;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    sendfile        on;
    keepalive_timeout  65;
include extra/www.conf;
include extra/blog.conf;
include extra/bbs.conf;
include extra/status.conf;
}

[root@www conf]# vi extra/www.conf 
[root@www conf]# vi extra/blog.conf 
[root@www conf]# vi extra/bbs.conf.conf 
[root@www conf]# vi extra/bbs.conf
[root@www conf]# vi extra/blog.conf 
[root@www conf]# nginx -t
[root@www conf]# nginx -s reload
[root@www conf]# cat extra/www.conf 
    server {
        listen       80;
        server_name  www.xxx.org;
        location / {
            root   html/www;
            index  index.html index.htm;
        }
        access_log logs/access_www.log main gzip buffer=32k flush=5s;  高并发情况下使用提高网站访问性能。
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
}
[root@www conf]# curl www.xxx.org
http://www.xxx.org
[root@www conf]# cat -n ../logs/access_www.log 
     1	 172.16.0.200 - - [14/May/2020:21:48:51 +0800] "GET / HTTP/1.1" 200 19 "-" "curl/7.29.0" "-

```

#### nginx location

```bash
nginx location
location 语法：
location [=|~|~*|^~] uri {…}
指令     匹配标识  匹配网址  匹配URI后需要执行的配置段

~    区分大小写 
~*   不区分大小写
上面两个可以结合 ！使用，即取反。   ！~不区分大小写  ！~*区分大小写
^~  意思是在进行常规的字符串匹配检查之后，不做正则表达式的检查。


localtion =/ {                       =精准匹配  优先级01  最高
[configuration A]
}

localtion /  {                         默认匹配 优先级04 最低
[configuration B]
}

localtion /documents/ {               按照目录进行匹配  优先级03
[configuration C]
}

localtion ^~ /images/ {               优先匹配/不识别uri信息中符号信息 优先级02
[configuration D]
}

localtion ~* \.(gif|jpg|jpeg)$  {          不区分大小写匹配 优先级03
[configuration E]
}

```

| 用户请求的URI            | 完整的URL地址                                | 匹配的配置      |
| ------------------------ | -------------------------------------------- | --------------- |
| =/                       | http://www.xxx.org                           | configuration A |
| /                        | http://www.xxx.org/index.html                | configuration B |
| /documents/document.html | http://www.xxx.org/  documents/document.html | configuration C |
| /images/1.gif            | http://www.xxx.org/images/1.gif              | configuration D |
| /documents/1.jpg         | http://www.xxx.org/documents/1.jpg           | configuration E |

URI特殊字符组合匹配是有顺序的。

=/ 精准匹配

^~ 匹配常规字符串，不做正则匹配检查

~* \.(gif|jpg|jpeg)$ 正则匹配

/docation/documents/ 匹配常规字符串，如果有正则，则优先匹配正则

/  所有location 都不能匹配后的默认匹配



#### nginx rewrite

nginx rewrite

URL重定向

语法：rewrite  regex    replacement[flag];

   关键字 正则表达   重定向到replacement  flag结尾标记

默认：none

应用位置：server,location,if

rewrite ^/(.*) htpp://www.xxx.org/$1 permanent;

表示匹配成功之后跳转到301页面，url显示跳转之后的地址。

regex 常用正则表达式字符

| 字符        | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| \           | 转义字符，\n换行                                             |
| ^           | 匹配字符串的起始位置                                         |
| $           | 匹配字符串的结束位置                                         |
| *           | 匹配前面的字符零次或多次                                     |
| +           | 匹配前面的字符一次或多次                                     |
| ?           | 匹配前面的字符0次或1次                                       |
| .           | 匹配除\n之外的任何单个字符                                   |
| （pattern） | 匹配括号内的pattern，并可以在后获取对应的匹配常用$0-$9匹配小括号里面的内容 |

FLAG 标记参数说明

| flag标记  | 说明                                  |
| --------- | ------------------------------------- |
| last      | 匹配完当前规则，继续匹配下面的URI规则 |
| break     | 本条规则匹配完即终止                  |
| redirect  | 返回302重定向，浏览器跳转后URL地址    |
| permanent | 返回301重定向，浏览器跳转后URL地址    |

 ```bash
 [root@www conf]# cat extra/www.conf 
     server {
         listen       80;
         server_name  www.xxx.org  xxx.org;
 rewrite ^/(.*)  http://www.xxx.org/$1 permanent;
         location / {
             root   html/www;
             index  index.html index.htm;
         }
         access_log logs/access_www.log main gzip buffer=32k flush=5s;  高并发情况下使用提高网站访问性能。
         error_page   500 502 503 504  /50x.html;
         location = /50x.html {
             root   html;
         }
 }
 ```

```bash
rewrite模块是跳转功能:  http_rewrite_module
Syntax:	 rewrite regex replacement [flag];   rewite  匹配的正则信息  替换成什么信息
Default:	 —
Context:	 server, location, if

rewrite www.oldboy.com/(.*) http://www.oldboy.com/$1 permanent;   重写规则配置
                    ^/ (.*)  baidu.com  / oldboy.html  跳转方式

www.baidu.com/oldboy.html

跳转方式:
永久跳转:  permanent   301    会将跳转信息进项缓存
临时跳转:  redirect    302    不会缓存跳转信息

出现无限跳转如何解决:
第一种方法: 利用不同server区块配置打破循环
server {
   server_name  oldboy.com;
   rewrite ^/(.*) http://www.oldboy.com/$1 permanent;
}
第二种方法: 利用if判断实现打破循环
if ($host ~* "^oldboy.com$") {
  rewrite ^/(.*) http://www.oldboy.com/$1 permanent;
}

www.oldboy.com/oldboy01/oldboy02/oldboy.jpg   --- www.oldboy.com/oldboy.jpg

```

#### nginx不同域名的URL跳转

```bash
[root@www conf]# cat extra/bbs.conf 
    server {
        listen       82;
        server_name  bbs.xxx.org;
        location / {
            root   html/bbs;
            index  index.html index.htm;
        }
        if ( $http_host ~* "^(.*)\.xxx\.org$") {
            set $domain $1;
            rewrite ^(.*) http://blog.xxx.org/$domain/oldboy.html break;
         }
        access_log logs/access_bbs.log main;
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
   }
oldboy.htm 文件必须存在并具有访问权限，不然返回302错误。
```

#### nginx 访问加密

```bash

   server {
        listen       80;
        server_name  www.xxx.org;
        location / {
            root   html/www;
            index  index.html index.htm;
            auth_basic   "test testtest";
            auth_basic_user_file /application/nginx/conf/htpasswd;
        }


        access_log logs/access_www.log main;
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
}
[root@www conf]# which htpasswd   没有使用
#yum install -y httpd 去安装一个
/usr/bin/htpasswd
[root@www conf]# htpasswd -bc /application/nginx
nginx/        nginx-1.16.1/ 
[root@www conf]# htpasswd -bc /application/nginx/conf/htpasswd test testtest
Adding password for user test
[root@www conf]# chmod 400 /application/nginx/conf/htpasswd 
[root@www conf]# chown nginx /application/nginx/conf/htpasswd 
[root@www conf]# cat htpasswd 
test:$apr1$NZm1Kw7e$Wdq1VLgIYOda1HVAf3gKF0
[root@www conf]# nginx -t
nginx: the configuration file /application/nginx-1.16.1//conf/nginx.conf syntax is ok
nginx: configuration file /application/nginx-1.16.1//conf/nginx.conf test is successful
[root@www conf]# nginx -s reload
```

```bash
htpasswd命令参数说明:
-c  Create a new file.  *****
    创建一个密码文件
-n  Don't update file; display results on stdout.
    不会更新文件; 显示文件内容信息
-b  Use the password from the command line rather than prompting for it. *****
    免交互方式输入用户密码信息
-i  Read password from stdin without verification (for script usage).
    读取密码采用标准输入方式,并不做检查 ???
-m  Force MD5 encryption of the password (default).
    md5的加密算法
-B  Force bcrypt encryption of the password (very secure).
    使用bcrypt对密码进行加密  
-C  Set the computing time used for the bcrypt algorithm
    (higher is more secure but slower, default: 5, valid: 4 to 31).
  使用bcrypt algorithm对密码进行加密
-d  Force CRYPT encryption of the password (8 chars max, insecure).
    密码加密方式
-s  Force SHA encryption of the password (insecure).
    加密方式
-p  Do not encrypt the password (plaintext, insecure).
    不进行加密
-D  Delete the specified user.
    删除指定用户
-v  Verify password for the specified user.

修改密码文件权限: 
chmod 600 ./htpasswd
```

![image-20210818201042275](./linux运维07web服务及负载均衡.assets\image-20210818201042275.png)

输入用户test 和密码testtest 就可以访问该站点了。

tengine 是淘宝开源的nginx分支。

## 企业应用

```bash
利用nginx服务搭建网站文件共享服务器
第一个步骤: 编写配置文件(www.conf)
nginx模块功能: ngx_http_autoindex_module

Syntax:	 autoindex on | off;
Default:	 
autoindex off;
Context:	 http, server, location

server {
   listen        80;
   server_name   www.oldboy.com;
   location / {
     root  /html/www;
     auth_basic      "oldboy-sz-01";
     auth_basic_user_file password/htpasswd;
     autoindex on;    --- 开启nginx站点目录索引功能
}
PS: 1. 需要将首页文件进行删除
    2. mime.types媒体资源类型文件作用
	    文件中有的扩展名信息资源,   进行访问时会直接看到数据信息
	    文件中没有的扩展名信息资源, 进行访问时会直接下载资源
	    
网站页面目录数据,中文出现乱码,如何解决:
location / {
   root  /html/www;
   #index index.html;
   auth_basic      "oldboy-sz-01";
   auth_basic_user_file password/htpasswd;
   autoindex on;
   charset utf-8;   --- 修改目录结构中出现的中文乱码问题
 }
```

### 安装MySQL数据库

#### Mysql安装方法

因为mysql有不同的产品线，所以安装方式也就有下面几种。

 

| 序号 | Mysql安装方式           | 特点说明                                                     |
| ---- | ----------------------- | ------------------------------------------------------------ |
| 1    | yum/rpm                 | 简单，速度快，没法定制安装，入门和新手常用                   |
| 2    | 二进制安装              | 解压软件，简单配置后就可以使用，不用安装，速度快，专业DBA使用方式 |
| 3    | 源码编译安装            | 定制安装，安装时间长                                         |
| 4    | 源码软件结合yum/rpm安装 | 把源码制作成符合要求的RPM，放到yum仓库里，然后通过yum来安装，结合1，3的优点。需要具备更深功力 |

2、安装步骤介绍

```bash
[root@localhost ~]# find / -name mysql |xargs rm -rf  //批量查找并删除
```

#### A、使用rpm方法安装

获取mysql安装包https://dev.mysql.com/downloads/mysql/ 官网下载最安全。

![image-20210818203247214](./linux运维07web服务及负载均衡.assets\image-20210818203247214.png)

清理机器上面不需要的mariadb数据库软件

```bash
[root@localhost ~]# rpm -qa |grep mariadb
mariadb-libs-5.5.64-1.el7.x86_64
[root@localhost ~]# rpm -e --nodeps mariadb-libs-5.5.64-1.el7.x86_64
[root@localhost ~]# rpm -qa |grep mariadb
[root@localhost ~]# rpm -qa |grep mysql
```

1.通过rpm -qa | grep mariadb命令查看mariadb的安装包

2.如果有信息，通过rpm -e --nodeps mariadb-libs-x.x.xx-x.xxx.x86_64命令卸载

3.通过rpm -qa | grep mysql命令查看mysql的安装包

4.如果有信息，通过yum remove mysql-community-***命令依次删除

5.通过find / -name mysql命令查找mysql配置文件

6.如果有信息，通过rm -rf 文件路径命令依次删除

```bash
[root@localhost tools]# useradd -s /sbin/nologin -M mysql
[root@localhost tools]# id mysql
uid=1002(mysql) gid=1002(mysql) 组=1002(mysql)
// 创建mysql 用户和组并且禁止mysql登陆Linux.
[root@localhost tools]# tail -1 /etc/passwd
mysql:x:1002:1002::/home/mysql:/sbin/nologin
[root@localhost tools]# rpm -qa|grep mysql
//查看本机是否安装了mysql
[root@localhost tools]# tar -xvf mysql-8.0.20-1.el7.x86_64.rpm-bundle.tar
[root@localhost tools]# ls
mysql-8.0.20-1.el7.x86_64.rpm-bundle.tar  
mysql-community-libs-compat-8.0.20-1.el7.x86_64.rpm
mysql-community-client-8.0.20-1.el7.x86_64.rpm           
mysql-community-server-8.0.20-1.el7.x86_64.rpm
mysql-community-common-8.0.20-1.el7.x86_64.rpm           
mysql-community-test-8.0.20-1.el7.x86_64.rpm
mysql-community-devel-8.0.20-1.el7.x86_64.rpm            
mysql-community-embedded-compat-8.0.20-1.el7.x86_64.rpm  
mysql-community-libs-8.0.20-1.el7.x86_64.rpm
nginx-1.16.1
nginx-1.16.1.tar.gz
//解压从官网下载好的安装包
[root@localhost tools]# rpm -ivh mysql-community-common-8.0.20-1.el7.x86_64.rpm 
[root@localhost tools]# rpm -ivh mysql-community-libs-8.0.20-1.el7.x86_64.rpm 
[root@localhost tools]# rpm -ivh mysql-community-client-8.0.20-1.el7.x86_64.rpm 
[root@localhost tools]# rpm -ivh mysql-community-server-8.0.20-1.el7.x86_64.rpm 
[root@localhost tools]# rpm -qa|grep mysql
mysql-community-libs-8.0.20-1.el7.x86_64
mysql-community-common-8.0.20-1.el7.x86_64
mysql-community-client-8.0.20-1.el7.x86_64
mysql-community-server-8.0.20-1.el7.x86_64
[root@localhost tools]# whereis mysql
mysql: /usr/bin/mysql /usr/lib64/mysql /usr/share/man/man1/mysql.1.gz
[root@localhost tools]# chown -R mysql.mysql /usr/bin/mysql
//不添加权限会导致mysqld.service无法启动
[root@localhost tools]# which mysql
/usr/bin/mysql
[root@localhost ~]# ll -d /usr/bin/mysql
-rwxr-xr-x. 1 mysql mysql 23809944 3月  26 22:18 /usr/bin/mysql
//初始化数据库
[root@localhost tools]# mysqld --initialize --user=mysql
[root@localhost tools]# cat /var/log/mysqld.log 
2020-05-07T12:39:18.975181Z 0 [System] [MY-013169] [Server] /usr/sbin/mysqld (mysqld 8.0.20) initializing of server in progress as process 13836
2020-05-07T12:39:18.986441Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
2020-05-07T12:39:20.413645Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
2020-05-07T12:39:21.908797Z 6 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: iruqzQNgm4!C   //初始化之后的root登录密码
[root@localhost tools]# systemctl start mysqld.service  //启动mysqld
#service mysqld start 也可以启动

[root@localhost tools]# systemctl status mysqld.service  //查看mysql状态
[root@localhost tools]# mysql -u root -p
Enter password:                //输入刚才复制的密码
mysql> alter user'root'@'localhost'identified by '123456';
//修改密码为123456
//开启mysql的远程登陆
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
mysql> use mysql;
mysql> update user set host='%'where user='root';
mysql> select user,host from user;
+------------------+-----------+
| user             | host      |
+------------------+-----------+
| root             | %       |
| mysql.infoschema | localhost |
| mysql.session    | localhost |
| mysql.sys        | localhost |
+------------------+-----------+
```

#### systemd管理

```bash
查看服务是否开机启动：systemctl is-enabled firewalld.service
查看已启动的服务列表：systemctl list-unit-files|grep enabled
查看启动失败的服务列表：systemctl –failed
端口操作
添加
firewall-cmd --zone=public --add-port=80/tcp –permanent
（--permanent永久生效，没有此参数重启后失效）
重新载入
firewall-cmd --reload
```

#### B、使用二进制方法安装

下载二进制软件https://dev.mysql.com/downloads/mysql/

![image-20210818204755389](./linux运维07web服务及负载均衡.assets\image-20210818204755389.png)

```BASH
[root@localhost tools]# useradd -s /sbin/nologin -M mysql
[root@localhost tools]# tail -1 /etc/passwd
mysql:x:1002:1002::/home/mysql:/sbin/nologin
[root@localhost tools]# mkdir /application/mysql-8.0.20
[root@localhost tools]# ls
mysql-8.0.20-linux-glibc2.12-x86_64.tar.xz  nginx-1.16.1  nginx-1.16.1.tar.gz  rpm
[root@localhost tools]# tar xvf mysql-8.0.20-linux-glibc2.12-x86_64.tar.xz
[root@localhost tools]# mv mysql-8.0.20-linux-glibc2.12-x86_64 /application/mysql-8.0.20
[root@localhost tools]# cd /application/mysql-8.0.20/
[root@localhost mysql-8.0.20]# ln -s /application/mysql-8.0.20/ /application/mysql
[root@localhost mysql-8.0.20]# ll /application/
总用量 0
lrwxrwxrwx.  1 root root  26 5月   8 11:07 mysql -> /application/mysql-8.0.20/
drwxr-xr-x.  9 root root 129 5月   8 11:05 mysql-8.0.20
drwxr-xr-x. 11 root root 171 4月  25 14:27 nginx-1.16.1
[root@localhost mysql-8.0.20]# cd /application/mysql
[root@localhost mysql]# ls support-files/
mysqld_multi.server  mysql-log-rotate  mysql.server
[root@localhost mysql]# chown -R mysql.mysql /application/mysql
[root@localhost mysql]# mkdir /var/log/mysql
[root@localhost mysql]# chown -R mysql.mysql /var/log/mysql
[root@localhost mysql]# mkdir /application/data
[root@localhost mysql]# chown -R mysql.mysql /application/data/

[root@localhost mysql]#  vi /etc/my.cnf
[mysqld]
port=3306
basedir=/application/mysql
datadir=/application/data
log-error=/var/log/mysql/error.log
user=mysql
default_authentication_plugin=mysql_native_password
skip_host_cache
skip-name-resolve=1
####################
#skip-grant-tables
[client]
socket=/tmp/mysql.sock
初始化数据库
[root@localhost mysql]# /application/mysql/bin/mysqld --initialize --user=mysql --basedir=/application/mysql/ --datadir=/application/data/
制作启动文件
[root@localhost mysql]# cp -a /application/mysql/support-files/mysql.server /etc/init.d/mysqld
[root@localhost mysql]# vi /etc/init.d/mysqld
basedir=/application/mysql
basedir=/application/mysql/bin
datadir=/application/data
这里必须把文件中所有这个参数替换了，防止后期出错。
[root@localhost mysql]# chown -R 755 /etc/init.d/mysqld
//[root@localhost mysql]# chmod +x /etc/init.d/mysqld
添加可执行权限
[root@localhost mysql]# /etc/init.d/mysqld start
Starting MySQL SUCCESS!
[root@localhost mysql]# ln -vs /application/mysql/bin/* /usr/bin 
创建一个mysql命令链接
[root@localhost mysql]# mysql
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: NO)
提示这个错误，去/etc/my.cnf 文件中把#skip-grant-tables的#号取消。
[root@localhost mysql]# vi /etc/my.cnf
[root@localhost mysql]# /etc/init.d/mysqld restart
[root@localhost mysql]# mysql
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
mysql> flush privileges;
Query OK, 0 rows affected (0.01 sec)
mysql> alter user'root'@'localhost'identified by '123456';
Query OK, 0 rows affected (0.00 sec)
//修改密码为123456
//开启mysql的远程登陆
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.00 sec)

mysql> use mysql;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> update user set host='%'where user='root';
Query OK, 1 row affected (0.01 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> select user,host from user;
+------------------+-----------+
| user             | host      |
+------------------+-----------+
| root             | %       |
| mysql.infoschema | localhost |
| mysql.session    | localhost |
| mysql.sys        | localhost |
+------------------+-----------+
4 rows in set (0.00 sec)
mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
mysql> exit
Bye
然后再修改/etc/my.cnf 文件中把#skip-grant-tables的#加上。
[root@localhost mysql]# /etc/init.d/mysqld restart
[root@localhost mysql]# mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 12
Server version: 8.0.20 MySQL Community Server - GPL

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
把启动命令放到/etc/rc.local里面去。

touch /var/lock/subsys/local
/etc/init.d/mysqld start
chmod +x /etc/rc.d/rc.local
到此二进制安装方式完成。

```

#### C、编译方式安装

下载地址：https://dev.mysql.com/downloads/mysql/

![image-20210818204921536](./linux运维07web服务及负载均衡.assets\image-20210818204921536.png)

在使用源码安装时候必须准备好的软件

```BASH
cmake-3.17.2 、gcc-5.3以上，本文使用10.1.0
yum -y install wget  cmake gcc gcc-c++ ncurses  ncurses-devel  libaio-devel  openssl openssl-devel 

cmake的安装

[root@localhost mysql]# cmake . -DCMAKE_INSTALL_PREFIX=/application/mysql -DMYSQL_DATADIR=/application/data -DSYSCONDIR=/etc -DMYSQL_TCP_PORT=3306 -DWITH_BOOT=/application/mysql/boot
-- Running cmake version 2.8.12.2
CMake Warning at CMakeLists.txt:54 (MESSAGE):
  Please use cmake3 rather than cmake on this platform
-- Found /usr/bin/cmake3
CMake Error at CMakeLists.txt:84 (CMAKE_MINIMUM_REQUIRED):
  CMake 3.5.1 or higher is required.  You are running version 2.8.12.2
-- Configuring incomplete, errors occurred!
cmake版本太低必须升级到3.5.1以上版本。
这里使用cmake 时候提示yum的cmake版本过低。下载
https://cmake.org/files/
[root@localhost tools]# tar xvf cmake-3.17.2.tar.gz
[root@localhost cmake-3.17.2]# yum install -y gcc gcc-c++ openssl-devel
[root@localhost cmake-3.17.2]# ./bootstrap --prefix=/usr/local/cmake
[root@localhost cmake-3.17.2]# ./configure 二选一
[root@localhost cmake-3.17.2]# make
[root@localhost cmake-3.17.2]# make install
[root@localhost cmake-3.17.2]# /usr/local/bin/cmake --version
cmake version 3.17.2
[root@localhost cmake-3.17.2]# ln -s /usr/local/bin/cmake /usr/bin
[root@localhost cmake-3.17.2]# cmake --version
cmake version 3.17.2
[root@localhost cmake-3.17.2]#vi /etc/profile
export PATH=$PATH:/usr/local/cmake/bin
安装之前需要安装下面的这些组件
[root@localhost ~]yum install -y gcc gcc-c++ openssl-devel
[root@localhost ~]tar xvf cmake-3.17.2.tar.gz
[root@localhost ~]cd cmake-3.17.2
[root@localhost cmake-3.17.2]#./configrue
[root@localhost cmake-3.17.2]#make
[root@localhost cmake-3.17.2]#make install
[root@localhost cmake-3.17.2]# whereis cmake
cmake: /usr/local/bin/cmake
[root@localhost cmake-3.17.2]# /usr/local/bin/cmake --version
cmake version 3.17.2
CMake suite maintained and supported by Kitware (kitware.com/cmake).
[root@localhost cmake-3.17.2]# ln -s /usr/local/bin/cmake /usr/bin
gcc
CentOS 7为4.8。gcc 4.8最主要的一个特性就是全面支持C++11，如果不清楚什么用的也没关系，简单说一些C++11标准的程序都需要gcc 4.8以上版本的gcc编译器编译，如MySQL 8.0版本(8.0.16以上版本是C++14标准，需gcc 5.3以上版本)。
http://ftp.gnu.org/gnu/gcc/   下载地址
yum install gcc gcc-c++  不能卸载以前的（这里就是共存，我们最后把/usr/local/bin下面的命令软连接到/usr/bin下面去。实现共存不然会出现很多问题）
进入软件目录
[root@localhost ~]#tar xvf gcc-10.1.0.tar.gz
[root@localhost ~]#cd gcc-10.1.0
[root@localhost gcc-10.1.0]#./contrib/download_prerequisites
[root@localhost gcc-10.1.0]#./configure --enable-checking=release --enable-languages=c,c++ --disable-multilib  
[root@localhost gcc-10.1.0]#make   需要一个小时以上，虚拟机4-5个小时
make install
[root@localhost gcc-10.1.0]# cd /usr/bin/
[root@localhost bin]# mv gcc gcc_bak
[root@localhost bin]# mv g++ g++_bak
[root@localhost bin]# which gcc
/usr/local/bin/gcc
[root@localhost bin]# /usr/local/bin/gcc --version
gcc (GCC) 10.1.0
Copyright © 2020 Free Software Foundation, Inc.
本程序是自由软件；请参看源代码的版权声明。本软件没有任何担保；
包括没有适销性和某一专用目的下的适用性担保。
[root@localhost bin]# ln -sf /usr/local/bin/* /usr/bin/
[root@localhost bin]# ln -s /usr/local/bin/g
g++         gcc         gcc-ar      gcc-nm      gcc-ranlib  gcov        gcov-dump   gcov-tool   
[root@localhost bin]# ln -s /usr/local/bin/gcc gcc
[root@localhost bin]# ln -s /usr/local/bin/g++ g++
[root@localhost bin]# gcc --version
gcc (GCC) 10.1.0
Copyright © 2020 Free Software Foundation, Inc.
本程序是自由软件；请参看源代码的版权声明。本软件没有任何担保；
包括没有适销性和某一专用目的下的适用性担保。
[root@localhost bin]#
尝试很多种方法都是错误不断。在经过一天的努力下终于把遇到的各种错误都解决了。下面是完成版的整理。

```

【错误集合】

1、配置报错：Please install the appropriate openssl developer package

**解决方法**：安装openssl-devel

yum -y install openssl-devel

2、配置报错：Could NOT find Curses

解决方法：安装ncurses-devel

yum -y install ncurses-devel

3、编译报错：Please do not build in-source.  Out-of source builds are highly

​       ![image-20210818205415015](./linux运维07web服务及负载均衡.assets\image-20210818205415015.png)                 

**原因**：是它建议你不要构建源代码。

**解决办法**：在配置的时候加入字段：-DFORCE_INSOURCE_BUILD=1

4、配置报错：The C compiler identification is unknown

**解决办法**：做一个gcc的软链接。

ln -s /usr/local/gcc/bin/gcc /usr/bin/cc

5、编译报错：/usr/lib/libstdc++.so.6: version `CXXABI_1.3.9' not found

因为libstdc++.so.6的库是使用的以前的老库。

**解决办法**：修改软链接

find / -name libstdc++.so.6
 mv /usr/lib64/libstdc++.so.6 /usr/lib64/libstdc++.so.6_old
 ln -s /usr/local/gcc/lib64/libstdc++.so.6.0.25 /usr/lib64/libstdc++.so.6 需要下载新版本

6、编译报错：'SYS_gettid' has not been declared in this scope

解决方法：在该文件上添加一个头文件。

vim /usr/local/src/mysql-8.0.16/storage/innobase/buf/buf0buf.cc

​    \#在第一行添加

​    \#include "sys/syscall.h"

7、编译报错：‘os_compare_and_swap_thread_id’ Has not been declared in this scope

解决办法：修改报错文件内容

vim /usr/local/src/mysql-8.0.16/storage/innobase/lock/lock0lock.cc

   \#将 “os_compare_and_swap_thread_id” 修改为

   os_compare_and_swap_lint

![image-20210818205437480](./linux运维07web服务及负载均衡.assets\image-20210818205437480.png)

8、编译报错：/usr/bin/ar: ../../archive_output_directory/libz.a

解决方法：重新配置编译

make clean

cmake .....(你的配置参数)

make

 

以上全是编译过程中遇到的问题和解决方法。

mysql正式开始了。

```bash
[root@localhost ~]#useradd -s /sbin/nologin -M mysql
[root@localhost ~]#mkdir /application/mysql
[root@localhost ~]#mkdir /application/data
[root@localhost ~]#tar -zxf mysql-boost-8.0.11.tar.gz -C /application/mysql-8.0.20
[root@localhost ~]#tcd /application/mysql-8.0.20
[root@localhost ~]#cmake . -DCMAKE_INSTALL_PREFIX=/application/mysql -DMYSQL_DATADIR=/application/data -DSYSCONFDIR=/etc -DMYSQL_TCP_PORT=3306 -DWITH_BOOST=/application/mysql-8.0.20/boost -DFORCE_INSOURCE_BUILD=1
[root@localhost ~]# make  && make install
这里跟二进制的配置完全一样
[root@localhost mysql]# chown -R mysql.mysql /application/mysql
[root@localhost mysql]# mkdir /var/log/mysql
[root@localhost mysql]# chown -R mysql.mysql /var/log/mysql
[root@localhost mysql]# mkdir /application/data
[root@localhost mysql]# chown -R mysql.mysql /application/data/
[root@localhost mysql]#  vi /etc/my.cnf
[mysqld]
port=3306
basedir=/application/mysql
datadir=/application/data
log-error=/var/log/mysql/error.log
user=mysql
default_authentication_plugin=mysql_native_password
skip_host_cache
skip-name-resolve=1
####################
#skip-grant-tables
[client]
socket=/tmp/mysql.sock
初始化数据库
[root@localhost mysql]# /application/mysql/bin/mysqld --initialize --user=mysql --basedir=/application/mysql/ --datadir=/application/data/
制作启动文件
[root@localhost mysql]# cp -a /application/mysql/support-files/mysql.server /etc/init.d/mysqld
[root@localhost mysql]# vi /etc/init.d/mysqld
basedir=/application/mysql
basedir=/application/mysql/bin
datadir=/application/data
这里必须把文件中所有这个参数替换了，防止后期出错。
[root@localhost mysql]# chown -R 755 /etc/init.d/mysqld
//[root@localhost mysql]# chmod +x /etc/init.d/mysqld
添加可执行权限
[root@localhost mysql]# /etc/init.d/mysqld start
Starting MySQL SUCCESS!
[root@localhost mysql]# ln -vs /application/mysql/bin/mysql /usr/bin 
创建一个mysql命令链接
[root@localhost mysql]# mysql
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: NO)
提示这个错误，去/etc/my.cnf 文件中把#skip-grant-tables的#号取消。
[root@localhost mysql]# vi /etc/my.cnf
[root@localhost mysql]# /etc/init.d/mysqld restart
[root@localhost mysql]# mysql
mysql> flush privileges;
mysql> alter user'root'@'localhost'identified by '123456';
//修改密码为123456
//开启mysql的远程登陆
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
mysql> use mysql;
mysql> update user set host='%'where user='root';
mysql> select user,host from user;
+------------------+-----------+
| user             | host      |
+------------------+-----------+
| root             | %       |
| mysql.infoschema | localhost |
| mysql.session    | localhost |
| mysql.sys        | localhost |
+------------------+-----------+
mysql> flush privileges;
mysql> exit
Bye
然后再修改/etc/my.cnf 文件中把#skip-grant-tables的#加上。
[root@localhost mysql]# /etc/init.d/mysqld restart
[root@localhost mysql]# mysql -u root -p
Enter password: 
mysql>
```

总结：一般情况不要轻易去使用源码编译，非常复杂费事费时！！！

### mariadb 安装

```bash
第一个历程: 安装数据库软件
	 yum install mariadb-server mariadb -y
	 
	 补充: 数据库初始化过程 mysql_install_db
	 --basedir=path       The path to the MariaDB installation directory.
	                      指定mysql程序目录
	 --datadir=path       The path to the MariaDB data directory.
	                      指定数据信息保存的目录
	 --user=mysql         让mysql管理数据目录  700	 
	 
	 信息输出内容:
	 创建数据库的密码信息:
	 PLEASE REMEMBER TO SET A PASSWORD FOR THE MySQL root USER !
    To do so, start the server, then issue the following commands:
    /application/mysql/bin/mysqladmin -u root          password 'new-password'   --- 给本地数据库设置密码
    /application/mysql/bin/mysqladmin -u root -h web01 password 'new-password'   --- 给远程数据库设置密码
	 
第二个历程: 启动数据库服务
	 systemctl start mariadb.service 
	 systemctl enable mariadb.service

第三个历程: 给mysql数据库服务设置密码
    mysqladmin -u root  password 'oldboy123'    --- 设置密码  	 
	 mysql -u root -poldboy123

创建数据库: create database wordpress;  
检查: show databases;
创建数据库管理用户: grant all on wordpress.* to 'wordpress'@'localhost' identified by 'oldboy123';
检查: select user,host from mysql.user;

```

### PHP的安装及常见问题

linux下fastCGI接口为socket，也可以是文件socket 也可以是IP socket。

确认运行库

```bash
[root@www ~]# yum install -y zlib-devel libxml2-devel libjpeg-devel libjpeg-trubo-devel libiconv-devel
[root@www ~]# yum install -y freetype-devel libpng-devel gd-devel libcurl-devel libxslt-devel


[root@www ~]# rpm -qa zlib-devel libxml2-devel libjpeg-turbo-devel libiconv-devel
zlib-devel-1.2.7-18.el7.x86_64
libxml2-devel-2.9.1-6.el7.4.x86_64
libjpeg-turbo-devel-1.2.90-8.el7.x86_64
[root@www ~]# rpm -qa freetype-devel libpng-devel gd-devel libcurl-devel libxslt-devel
libxslt-devel-1.1.28-5.el7.x86_64
freetype-devel-2.8-14.el7.x86_64
libcurl-devel-7.29.0-57.el7.x86_64
libpng-devel-1.5.13-7.el7_2.x86_64
gd-devel-2.0.35-26.el7.x86_64

libiconv-devel需要使用编译方式来安装了
下载地址：https://ftp.gnu.org/pub/gnu/libiconv/
下载好放到软件目录里面

[root@www tools]# ls
cmake-3.17.2         erjinzhi              mysql-8.0.20-linux-glibc2.12-x86_64.tar.xz
cmake-3.17.2.tar.gz  gcc-10.1.0            nginx-1.16.1
cmatrix-1.2a         gcc-10.1.0.tar.gz     nginx-1.16.1.tar.gz
cmatrix-1.2a.tar.gz  libiconv-1.16.tar.gz  rpm

cd /home/oldboy/tools/
wget  https://ftp.gnu.org/pub/gnu/libiconv/libiconv-1.16.tar.gz
tar -xf libiconv-1.16.tar.gz 
cd libiconv-1.16
./configure --prefix=/usr/local/libiconv
make && make install

libmcrypt库
libmcrypt是一个使用动态加载的模块
[root@www tools]# yum install -y libmcrypt-devel
[root@www tools]# rpm -qa libmcrypt
libmcrypt-2.5.8-13.el7.x86_64

mhash库
[root@www tools]# yum install -y mhash
[root@www tools]# rpm -qa mhash
mhash-0.9.9.9-10.el7.x86_64

mcrypt加密扩展库
[root@www tools]# yum install -y mcrypt

解压安装PHP
[root@www tools]# tar xzf php-7.4.6.tar.gz 
[root@www tools]# cd php-7.4.6
[root@www php-7.4.6]# ./configure --prefix=/application/php-7.4.6 \
--with-pdo-mysql=/application/mysql \
--with-iconv-div=/usr/local/libiconv \
--with-freetype \
--with-jpeg \
--with-zlib \
--with-libxml \
--disable-rpath \
--enable-bcmath \
--enable-shmop \
--enable-sysvsem \
--with-curl \
--enable-fpm \
--enable-mbstring \
--enable-gd \
--with-openssl \
--with-mhash \
--enable-pcntl \
--enable-sockets \
--with-xmlrpc \
--with-zip \
--with-soap \
--disable-short-tags \
--with-xsl \
--with-fpm-user=nginx \
--with-fpm-group=nginx \
--enable-ftp

这条等价于上面
./configure --prefix=/application/php-7.4.6 --with-pdo-mysql=/application/mysql --with-iconv-div=/usr/local/libiconv --with-freetype --with-jpeg --with-zlib --with-libxml --disable-rpath --enable-bcmath --enable-shmop --enable-sysvsem --with-curl --enable-fpm --enable-mbstring --enable-gd --with-openssl --with-mhash --enable-pcntl --enable-sockets --with-xmlrpc --with-zip --with-soap --disable-short-tags --with-xsl --with-fpm-user=nginx --with-fpm-group=nginx --enable-ftp

加入了mysql
./configure --prefix=/application/php-7.4.6 --with-curl --with-freetype --enable-gd --with-jpeg --with-gettext --with-iconv-dir=/usr/local --with-kerberos --with-libdir=lib64 --with-libxml --with-pdo-mysql=/application/mysql --with-openssl --with-pdo-sqlite --with-pear --enable-sockets --with-mhash --with-ldap-sasl --with-xmlrpc --with-xsl --with-zlib --enable-fpm --enable-bcmath --enable-inline-optimization --enable-mbregex --enable-mbstring --enable-opcache --enable-pcntl --enable-shmop --enable-soap --enable-sockets --enable-sysvsem --enable-xml --with-zip  -with-bz2 --enable-inline-optimization --enable-sysvsem --with-fpm-user=nginx --with-fpm-group=nginx --with-pdo_mysql

遇到的错误解决：
1.checking for sqlite3 > 3.7.4... no
configure: error: Package requirements (sqlite3 > 3.7.4) were not met:
No package 'sqlite3' found
Consider adjusting the PKG_CONFIG_PATH environment variable if you
installed software in a non-standard prefix.
Alternatively, you may set the environment variables SQLITE_CFLAGS
and SQLITE_LIBS to avoid the need to call pkg-config.
See the pkg-config man page for more details.

#yum install -y sqlite-devel

2. configure: error: Package requirements (oniguruma) were not met:
Package 'oniguruma', required by 'virtual:world', not found
Consider adjusting the PKG_CONFIG_PATH environment variable if you
installed software in a non-standard prefix.
Alternatively, you may set the environment variables ONIG_CFLAGS
and ONIG_LIBS to avoid the need to call pkg-config.
See the pkg-config man page for more details.

[root@yjweb source]# wget https://github.com/kkos/oniguruma/archive/v6.9.4.tar.gz -O oniguruma-6.9.4.tar.gz 
[root@yjweb source]# tar -zxvf oniguruma-6.9.4.tar.gz
[root@yjweb source]# cd oniguruma-6.9.4/
[root@yjweb oniguruma-6.9.4]# ./autogen.sh && ./configure --prefix=/usr
Generating autotools files.
./autogen.sh: line 6: autoreconf: command not found
[root@yjweb oniguruma-6.9.4]# yum install autoconf automake libtool
[webop@yjweb ~]$ cat /etc/redhat-release
CentOS Linux release 8.0.1905 (Core) 

3．configure: error: Package requirements (libzip >= 0.11) were not met:

Package 'libzip', required by 'virtual:world', not found

Consider adjusting the PKG_CONFIG_PATH environment variable if you
installed software in a non-standard prefix.

[root@localhost php-7.4.6]# yum install libzip-devel

```

#### 配置

```BASH
配置php文件php.ini
[root@localhost ~]# ln -s /application/php-7.4.6 /application/php
[root@localhost ~]# ll /application/
总用量 4
drwxr-xr-x   6 mysql mysql 4096 5月  26 20:42 data
lrwxrwxrwx   1 mysql mysql   26 5月  26 04:54 mysql -> /application/mysql-8.0.20/
drwxr-xr-x   9 root  root   129 5月  26 04:43 mysql-8.0.20
lrwxrwxrwx.  1 nginx nginx   25 5月  26 04:16 nginx -> /application/nginx-1.16.1
drwxr-xr-x. 11 root  root   151 5月  26 04:17 nginx-1.16.1
lrwxrwxrwx   1 root  root    22 5月  26 20:46 php -> /application/php-7.4.6
drwxr-xr-x   9 root  root    88 5月  26 09:43 php-7.4.6
[root@localhost php-7.4.6]# ls php.ini-
php.ini-development  php.ini-production   
[root@localhost php-7.4.6]# cp php.ini-production /application/php/lib/php.ini
配置PHP  fastcgi的配置文件
[root@localhost php-7.4.6]# cd /application/php/etc/
[root@localhost etc]# cp php-fpm.conf.default php-fpm.conf
[root@localhost etc]# ls
pear.conf  php-fpm.conf  php-fpm.conf.default  php-fpm.d  php.ini

启动php服务
[root@localhost etc]# /application/php/sbin/php-fpm 
[root@localhost etc]# ln -vs /application/php/sbin/php-fpm /usr/bin/php-fpm
'/usr/bin/php-fpm' -> '/application/php/sbin/php-fpm'
[root@localhost etc]# ps -ef|grep php-fpm
root       5579      1  0 20:57 ?        00:00:00 php-fpm: master process (/application/php-7.4.6/etc/php-fpm.conf)
nginx      5580   5579  0 20:57 ?        00:00:00 php-fpm: pool www
nginx      5581   5579  0 20:57 ?        00:00:00 php-fpm: pool www
root       5587   5525  0 20:58 pts/0    00:00:00 grep --color=auto php-fpm
[root@localhost etc]# lsof -i:9000
COMMAND  PID  USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
php-fpm 5579  root   10u  IPv4  33721      0t0  TCP localhost:cslistener (LISTEN)
php-fpm 5580 nginx    5u  IPv4  33721      0t0  TCP localhost:cslistener (LISTEN)
php-fpm 5581 nginx    5u  IPv4  33721      0t0  TCP localhost:cslistener (LISTEN)

添加开机启动
[root@localhost etc]# vi /etc/rc.d/rc.local
touch /var/lock/subsys/local
mysqld1 start
nginx
php-fpm
~              
配置nginx支持php请求
[root@localhost nginx]# vi conf/extra/www.conf 

    server {
        listen       80;
        server_name  localhost;
        location / {
            root   html;
            index  index.html index.htm;
        }
        location ~ .*/.(php|php5)?${
            root   html/blog;
            fastcgi_pass 127.0.0.1:9000;
            fastcgi_index index.php;
            include fastcgi.conf;
         }
        access_log logs/access_www.log main gzip buffer=32k flush=5s;
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
配置文件如上。
添加测试php页面
[root@localhost nginx]# mkdir html/blog
[root@localhost nginx]# cd html/blog/
[root@localhost blog]# echo "<?php php.info();  ?>" >test_php.php
[root@localhost blog]# cat test_php.php 
<?php php.info();  ?>

开始不能访问修改index.php 
[root@localhost conf]# cat ../html/blog/index.php 
<?php
  phpinfo();
?>

```

![image-20210818212152905](./linux运维07web服务及负载均衡.assets\image-20210818212152905.png)

#### discuz搭建

```bash
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '123456';
mysql加密方式需要使用最新的。
cat -n /var/log/mysql/error.log 查看日志得到的解决办法。

create database discuz character set utf8;  创建一个discuz数据库
create user'discuz'@'localhost'IDENTIFIED by '123456';  创建一个用户
grant all privileges on discuz.* to 'discuz'@'localhost';   授权用户
flush privileges; 刷新缓存
grant all on asd.* to 'wanghz'@'192.168.1.%' identified by 'w123'； 创建用户并授权
select user,host from user;
```

![image-20210818212318898](./linux运维07web服务及负载均衡.assets\image-20210818212318898.png)

### yum安装方式

```bash
PHP服务部署流程:
	 第一个历程: 更新yum源/卸载系统自带的PHP软件
# yum remove php-mysql php php-fpm php-common
# rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
# rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
	 
	 第二个历程: 安装php软件
# yum install -y php71w php71w-cli php71w-common php71w-devel php71w-embedded  php71w-gd php71w-mcrypt php71w-mbstring php71w-pdo php71w-xml php71w-fpm php71w-mysqlnd php71w-opcache  php71w-pecl-memcached php71w-pecl-redis php71w-pecl-mongodb
	 
	 第三个历程: 编写配置文件
# vim /etc/php-fpm.d/www.conf
# user = www   nginx---www
# group = www
	 PS: 保证nginx进程的管理用户和php服务进程的管理用户保持一致
	 
	 第四个历程: 启动php服务
[root@web01 ~]# systemctl status php-fpm.service 
[root@web01 ~]# systemctl enable php-fpm.service 
```

#### php.ini参数优化设置

```BASH
安全参数优化
safe_mode = on     开启安全模式
expose_php = off     关闭版本信息
display_errors = off    错误信息控制，调试时候打开
error_reporting = E_ALL & ~E_DEPRECATED    报错级别
log_errors = on           错误日志开启
error_log = /app/logs/php_errors.log        日志路径
register_globals = off               关闭全局变量，默认关闭
magic_quotes_gpc =on         防止SQL注入
allow_url_fopen = off          打开远程打开（禁止）
cgi.fix_pathinfo=0           防止nginx 文件类型错误解析漏洞

业务参数
max_execution_time = 30       单个脚本最大运行时间单位秒
max_input_time = 60           大哥脚本输出等待时间
memory_limit = 128M           单个脚本内存最大使用
upload_max_filesize = 2M        上传文件最大许可
max_file_uploads = 20           可以通过单个请求上传上载的最大文件数
```

#### PHP服务器缓存加速优化

xcache eaccelerator APC zendopcache 这四种是最常见的缓存加速器

xcache 效率高，速度更快。社区活跃

eaccelerator 安装简单，速度不错，社区不活跃  

已经停止更新10年了。

zendopcache php5.4以下使用 5.5已经整合到php里面只需要编译时指定—with-opcache即可。

#### php缓存加速器安装环境准备

```BASH
[root@localhost ~]# nginx -v
nginx version: nginx/1.16.1  
查看nginx版本

[root@localhost ~]# /application/php/bin/php -v
PHP Warning:  PHP Startup: Unable to load dynamic library 'pdo_mysql' (tried: /application/php-7.4.6/lib/php/extensions/no-debug-non-zts-20190902/pdo_mysql (/application/php-7.4.6/lib/php/extensions/no-debug-non-zts-20190902/pdo_mysql: cannot open shared object file: No such file or directory), /application/php-7.4.6/lib/php/extensions/no-debug-non-zts-20190902/pdo_mysql.so (/application/php-7.4.6/lib/php/extensions/no-debug-non-zts-20190902/pdo_mysql.so: cannot open shared object file: No such file or directory)) in Unknown on line 0
PHP 7.4.6 (cli) (built: May 26 2020 09:42:01) ( NTS )
Copyright (c) The PHP Group
Zend Engine v3.4.0, Copyright (c) Zend Technologies
查看php版本

[root@localhost ~]# mysqladmin -uroot -p123456 version
Server version	 	 8.0.20
Protocol version	 10
Connection	 	 Localhost via UNIX socket
UNIX socket	 	 /tmp/mysql.sock
Uptime:	 	 	 5 min 30 sec

查看mysql版本
不管是apache or nginx 最后都是通过PHP提供动态程序解析的，优化是一致的。
加速软件perl编译问题。
[root@localhost ~]# echo "export LC_ALL=C" >> /etc/profile  设置变量环境解决perl问题
[root@localhost ~]# tail -1 /etc/profile
export LC_ALL=C 
[root@localhost ~]# source /etc/profile 使增加的环境生效
[root@localhost ~]# echo $LC_ALL
C

[root@localhost ~]# /application/php/bin/phpize 
Cannot find config.m4. 
Make sure that you run '/application/php/bin/phpize' in the top level source directory of the module

[root@localhost ~]# find / -name openssl
/etc/pki/ca-trust/extracted/openssl
/usr/bin/openssl
/usr/lib64/python3.6/site-packages/cryptography/hazmat/backends/openssl
/usr/lib64/python3.6/site-packages/cryptography/hazmat/bindings/openssl
/usr/share/licenses/openssl
/usr/share/doc/openssl
/usr/share/ruby/openssl
/usr/share/gems/gems/openssl-2.1.2/lib/openssl
/usr/include/openssl
/soft/nginx-1.16.1/auto/lib/openssl
/soft/php-7.4.6/ext/openssl
[root@localhost ~]# cd /soft/php-7.4.6/ext/openssl/
[root@localhost openssl]# ll
total 1848
-rw-rw-r-- 1 root root      65 May 12 04:09 CREDITS
-rw-rw-r-- 1 root root     330 May 12 04:09 config.w32
-rw-rw-r-- 1 root root    1246 May 12 04:09 config0.m4
-rw-rw-r-- 1 root root  189426 May 12 04:09 openssl.c
-rw-r--r-- 1 root root     324 May 26 09:21 openssl.lo
-rw-r--r-- 1 root root 1135064 May 26 09:21 openssl.o
-rw-rw-r-- 1 root root    6489 May 12 04:09 php_openssl.h
drwxrwxr-x 2 root root    8192 May 12 04:09 tests
-rw-rw-r-- 1 root root   81124 May 12 04:09 xp_ssl.c
-rw-r--r-- 1 root root     321 May 26 09:21 xp_ssl.lo
-rw-r--r-- 1 root root  436496 May 26 09:21 xp_ssl.o
[root@localhost openssl]# cp config0.m4 config.m4
[root@localhost openssl]# /application/php/bin/phpize 
Configuring for:
PHP Api Version:         20190902
Zend Module Api No:      20190902
Zend Extension Api No:   320190902

[root@localhost openssl]# yum install -y perl-devel  

```

# 实现LNMP之间建立关系

## 1. 实现nginx + php 建立关系

```bash
第一个历程: 编写nginx文件
编写nginx配置文件
location ~ \.php$ {
         root /www;
         fastcgi_index index.php;          url               uri
         fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
         fastcgi_pass  127.0.0.1:9000;
         include fastcgi_params;   变量配置文件
}
重启nginx服务
```

![image-20210818213833285](./linux运维07web服务及负载均衡.assets\image-20210818213833285.png)

![image-20210818213850969](./linux运维07web服务及负载均衡.assets\image-20210818213850969.png)

第二个历程: 编写动态资源文件

```BASH
 [root@web01 conf.d]# cat /html/blog/test_php.php 
   <?php
   phpinfo();
   ?>
```

第三个历程: 进行访问测试

```bash
blog.oldboy.com/test_php.php
[root@web01 ~]# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
[root@web01 ~]# systemctl restart nginx
```

2. 实现php + mysql 建立关系

```bash
编写php代码文件
   [root@web01 blog]# vim test_mysql.php
   <?php
    $servername = "localhost";
    $username = "root";
    $password = "oldboy123";
    //$link_id=mysql_connect('主机名','用户','密码');
    //mysql -u用户 -p密码 -h 主机
    $conn = mysqli_connect($servername, $username, $password);
    if ($conn) {
          echo "mysql successful by root !\n";
       }else{
          die("Connection failed: " . mysqli_connect_error());
       }
   ?>
```

### worpress的安装

首先进入mysql添加wordpress数据库，授权wordpress用户控制

```bash
[root@web01 /etc/nginx/conf.d]# cat blog.conf 
server {
    listen       80;
    server_name  blog.oldboy.com;
    location / {
        root    /html/blog/wordpress;
        index   index.php;
 
               }


    location ~ \.php$ {
        root   /html/blog/wordpress;
        #index  index.php index.html index.htm;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_pass 127.0.0.1:9000;
        include fastcgi_params;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
[root@web01 /etc/nginx/conf.d]# ll /html/blog/
drwxr--r-- 5 nginx nginx     4096 3月  15 10:00 wordpress
-rwxr--r-- 1 root  root  16430366 3月  16 16:22 wordpress-5.7-zh_CN.tar.gz
[root@web01 /etc/nginx/conf.d]# ll /html/blog/wordpress
-rwxr--r--  1 nginx nginx   405 2月   6 2020 index.php
-rwxr--r--  1 nginx nginx 19915 1月   1 08:19 license.txt
-rwxr--r--  1 nginx nginx  6885 3月  15 10:00 readme.html
-rwxr--r--  1 nginx nginx  7165 1月  21 09:37 wp-activate.php
drwxr--r--  9 nginx nginx  4096 3月  15 10:00 wp-admin
-rwxr--r--  1 nginx nginx   351 2月   6 2020 wp-blog-header.php
-rwxr--r--  1 nginx nginx  2328 2月  17 21:08 wp-comments-post.php
-rwxr--r--  1 nginx nginx  2782 3月  15 10:00 wp-config-sample.php
drwxr--r--  5 nginx nginx    69 3月  16 16:58 wp-content
-rwxr--r--  1 nginx nginx  3939 7月  31 2020 wp-cron.php
drwxr--r-- 25 nginx nginx  8192 3月  15 10:00 wp-includes
-rwxr--r--  1 nginx nginx  2496 2月   6 2020 wp-links-opml.php
-rwxr--r--  1 nginx nginx  3313 1月  11 03:28 wp-load.php
-rwxr--r--  1 nginx nginx 44993 2月   3 02:13 wp-login.php
-rwxr--r--  1 nginx nginx  8509 4月  14 2020 wp-mail.php
-rwxr--r--  1 nginx nginx 21125 2月   2 08:10 wp-settings.php
-rwxr--r--  1 nginx nginx 31328 1月  28 05:03 wp-signup.php
-rwxr--r--  1 nginx nginx  4747 10月  9 05:15 wp-trackback.php
-rwxr--r--  1 nginx nginx  3236 6月   9 2020 xmlrpc.php

```

![image-20210818214129860](./linux运维07web服务及负载均衡.assets\image-20210818214129860.png)

![image-20210818214137167](./linux运维07web服务及负载均衡.assets\image-20210818214137167.png)

![image-20210818214149760](./linux运维07web服务及负载均衡.assets\image-20210818214149760.png)

http://blog.oldboy.com/wp-admin/

上传wordpress主题,报413错误,如何解决?

   总结:

   第一个历程: 修改nginx配置文件

>   vim blog.conf

>   server {

>    client_max_body_size 50m;  --- 指定用户上传数据的大小限制(默认1M)

>   }



  第二个历程: 修改php.ini配置文件

>   upload_max_filesize = 50M   --- 使PHP接收用户上传的更大的数据(默认2M)  

###   diacuz  wecenter

![image-20210818214349112](./linux运维07web服务及负载均衡.assets\image-20210818214349112.png)

![image-20210818214357973](./linux运维07web服务及负载均衡.assets\image-20210818214357973.png)

------

### LNMP注意事项

a selinux必须关闭 防火墙关闭

b /tmp 1777 mysql服务无法启动

c 出现web安装页无样式，修改nginx.conf/blog.conf

```bash
 location / {
        root    /html/blog/wordpress;
        index   index.php;
               }
```

