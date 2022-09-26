# linux

## 编码

```bash
[root@grafana ~]# locale
locale: Cannot set LC_CTYPE to default locale: No such file or directory
locale: Cannot set LC_MESSAGES to default locale: No such file or directory
locale: Cannot set LC_ALL to default locale: No such file or directory
LANG=C.UTF-8
LC_CTYPE="C.UTF-8"
LC_NUMERIC="C.UTF-8"
LC_TIME="C.UTF-8"
LC_COLLATE="C.UTF-8"
LC_MONETARY="C.UTF-8"
LC_MESSAGES="C.UTF-8"
LC_PAPER="C.UTF-8"
LC_NAME="C.UTF-8"
LC_ADDRESS="C.UTF-8"
LC_TELEPHONE="C.UTF-8"
LC_MEASUREMENT="C.UTF-8"
LC_IDENTIFICATION="C.UTF-8"
LC_ALL=
```

UnicodeEncodeError: 'ascii' codec can't encode character

划词搜索

```
https://www.baidu.com/search?q==$(CURRENT_WORD)
```

## cockpit管理平台

https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/managing_systems_using_the_rhel_7_web_console/getting-started-with-the-rhel-web-console_system-management-using-the-rhel-7-web-console#installing-the-web-console_getting-started-with-the-web-console

安装

```BASH
yum install cockpit
```

启动

```BASH
systemctl enable --now cockpit.socket
```

访问

```BASH
https://172.16.0.31:9090/
```

## ubuntu 更新镜像源

## 更新镜像源

阿里镜像源：https://developer.aliyun.com/mirror/ 

https://developer.aliyun.com/mirror/ubuntu?spm=a2c6h.13651102.0.0.3e221b1106rTyE



```sh
[root@G /mnt/d]# cp /etc/apt/sources.list{,.bak}

[root@G /mnt/d]# lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 20.04.3 LTS
Release:        20.04
Codename:       focal

[root@G /mnt/d]# vim /etc/apt/sources.list
deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
```

```sh
 PS1='[\[\e[31;40m\]\u\[\e[33;40m\]@\[\e[34;40m\]\h \[\e[33;40m\]\w\[\e[0m\]]\$ '
```

```sh
LS_COLORS=$LS_COLORS:'ow=1;32:'   #取消背景颜色
export PATH=/mnt/d/mysql5.7/bin:$PATH    #mysql 的环境
```

```sh
[root@G /mnt/c/Users/xyz34]# neofetch
            .-/+oossssoo+/-.               root@G
        `:+ssssssssssssssssss+:`           ------
      -+ssssssssssssssssssyyssss+-         OS: Ubuntu 20.04.3 LTS on Windows 10 x86_64
    .ossssssssssssssssssdMMMNysssso.       Kernel: 5.10.16.3-microsoft-standard-WSL2
   /ssssssssssshdmmNNmmyNMMMMhssssss/      Uptime: 11 mins
  +ssssssssshmydMMMMMMMNddddyssssssss+     Packages: 707 (dpkg)
 /sssssssshNMMMyhhyyyyhmNMMMNhssssssss/    Shell: bash 5.0.17
.ssssssssdMMMNhsssssssssshNMMMdssssssss.   Terminal: /dev/pts/1
+sssshhhyNMMNyssssssssssssyNMMMysssssss+   CPU: Intel i5-5200U (4) @ 2.194GHz
ossyNMMMNyMMhsssssssssssssshmmmhssssssso   Memory: 74MiB / 9487MiB
ossyNMMMNyMMhsssssssssssssshmmmhssssssso
+sssshhhyNMMNyssssssssssssyNMMMysssssss+
.ssssssssdMMMNhsssssssssshNMMMdssssssss.
 /sssssssshNMMMyhhyyyyhdNMMMNhssssssss/
  +sssssssssdmydMMMMMMMMddddyssssssss+
   /ssssssssssshdmNNNNmyNMMMMhssssss/
    .ossssssssssssssssssdMMMNysssso.
      -+sssssssssssssssssyyyssss+-
        `:+ssssssssssssssssss+:`
            .-/+oossssoo+/-.

```

