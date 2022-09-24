**富在术数不在劳身，利在势局不在力耕。**

# 常用信息

## 月份英文对照表

|  January  | February |  March   |  April   |
| :-------: | :------: | :------: | :------: |
|   一月    |   二月   |   三月   |   四月   |
|    May    |   June   |   July   |  August  |
|   五月    |   六月   |   七月   |   八月   |
| September | October  | November | December |
|   九月    |   十月   |  十一月  |  十二月  |

## 星期英文对照表

| 星期一 | 星期二  |  星期三   |  星期四  | 星期五 |  星期六  | 星期日 |
| :----: | :-----: | :-------: | :------: | :----: | :------: | :----: |
| Monday | Tuesday | Wednesday | Thursday | Friday | Saturday | Sunday |

 

## 批量注释

 

### 方法一 ：块选择模式

> Ctrl + v 进入块选择模式，然后移动光标选中你要注释的行，再按大写的I进入行首插入模式输入注释符号如 // 或 #，输入完毕之后，按两下ESC，Vim会自动将你选中的所有行首都加上注释，保存退出完成注释。



### 方法二 替换命令

> 使用下面命令在指定的行首添加注释：

> :起始行号,结束行号s/^/注释符/g

> 取消注释：

> :起始行号,结束行号s/^注释符//g

> 例子：

> 在10 - 20行添加 // 注释

> :10,20s#^#//#g

> 在10 - 20行删除 // 注释

> :10,20s#^//##g

 

## VIM粘贴格式错乱

运行如下命令，进入 paste 模式：

==:set paste==

进入 paste 模式后，按 i 键进入插入模式，然后再粘帖，文本格式不会错乱了。但粘帖后还需要按 <ESC> 进入普通模式并执行如下命令结束 paste 模式：

==:set nopaste==







显然，这样非常麻烦。下面我们进行改进。

我们可以利用设置快捷键来简化上面的两个命令。在 ~/.vimrc 中加入如下两行：

```BASH
map <F10> : set paste<CR>
map <F11> : set nopaste<CR>
```

这样，在普通模式下按 F10 （你也可以用自己喜欢的按键）就会进入 paste 模式，按 i 进入插入模式后粘帖，然后按 <ESC> 回到普通模式，再按 F11 结束 paste 模式。

## shell 颜色

```BASH
其中41的位置代表底色， 36的位置是代表字的颜色
原谅色送你echo颜色对应代码
颜色：30—–37
　　echo -e "\033[30m 黑色字 \033[0m"
　　echo -e "\033[31m 红色字 \033[0m"
　　echo -e "\033[32m 绿色字 \033[0m"
　　echo -e "\033[33m 黄色字 \033[0m"
　　echo -e "\033[34m 蓝色字 \033[0m" 
　　echo -e "\033[35m 紫色字 \033[0m" 
　　echo -e "\033[36m 天蓝字 \033[0m" 
　　echo -e "\033[37m 白色字 \033[0m" 
字背景颜色范围：40—–47
　　echo -e "\033[40;37m 黑底白字 \033[0m"
　　echo -e "\033[41;37m 红底白字 \033[0m" 
　　echo -e "\033[42;37m 绿底白字 \033[0m" 
　　echo -e "\033[43;37m 黄底白字 \033[0m" 
　　echo -e "\033[44;37m 蓝底白字 \033[0m" 
　　echo -e "\033[45;37m 紫底白字 \033[0m" 
　　echo -e "\033[46;37m 天蓝底白字 \033[0m" 
　　echo -e "\033[47;30m 白底黑字 \033[0m"
最后面控制选项说明
　　\33[0m 关闭所有属性 
\33[1m 设置高亮度 
\033[3m 斜体
　　\33[4m 下划线 
　　\33[5m 闪烁 
　　\33[7m 反显 
\33[8m 消隐 
\33[9m 删除线
　　\33[30m — \33[37m 设置前景色 
　　\33[40m — \33[47m 设置背景色 
　　\33[nA 光标上移n行 
　　\33[nB 光标下移n行 
　　\33[nC 光标右移n行 
　　\33[nD 光标左移n行 
　　\33[y;xH设置光标位置   \33[2J 清屏 
　　\33[K 清除从光标到行尾的内容 
　　\33[s 保存光标位置 
　　\33[u 恢复光标位置 
　　\33[?25l 隐藏光标 
　　\33[?25h 显示光标

```

检查服务脚本

```sh
#!/bin/bash
services=(etcd nginx docker keepalived kubelet kube-proxy kube-apiserver kube-controller-manager kube-scheduler)
hosts=(shell master)

funtion_services() {
   printf  "\033[44;37m %25s \033[0m  :  " $i && systemctl status $i | grep Active |awk -F"[()]" '{print $2}'
   echo "-----------------------------------------------------"
}

for h in ${hosts[@]};
do

    echo -e "$h Services Is Checking......"
    echo -e "\033[41;31m ////////////////////////////////////////////////////\033[0m"
    for i in ${services[@]};
    do
       funtion_services
    done
    echo -e "\033[43;33m ////////////////////////////////////////////////////\033[0m"                       
done

```





## 有趣的linux工具

### lolcat 看文件彩色显示

```BASH
[root@localhost soft]# unzip lolcat-master.zip
[root@localhost soft]# cd lolcat-master
[root@localhost lolcat-master]# yum -y install gem
[root@localhost lolcat-master]# gem install lolcat
```

### cmatrix 黑客代码雨

```BASH
[root@localhost cmatrix-1.2a]# yum -y install ncurses* gcc gcc-c++
[root@localhost cmatrix-1.2a]# ./configure
[root@localhost cmatrix-1.2a]# make && make install

./configure 
make
make install
./cmatrix
pwd
/root/cmatrix-1.2a/./cmatrix 
ln -s /root/cmatrix-1.2a/./cmatrix /usr/bin
cmatrix
```

### 登陆界面修改

```BASH
[root@lnmp ~]# cat /etc/issue
------------------------------------------------------------------------
                              .       .
                             / `.   .' " 
                     .---.  <    > <    >  .---.
                     |    \  \ - ~ ~ - /  /    |
         _____          ..-~             ~-..-~
        |     |   \~~~\.'                    `./~~~/
       ---------   \__/                        \__/
      .'  O    \     /               /       \  " 
     (_____,    `._.'               |         }  \/~~~/
      `----.          /       }     |        /    \__/
            `-.      |       /      |       /      `. ,~~|
                ~-.__|      /_ - ~ ^|      /- _      `..-'   
                     |     /        |     /     ~-.     `-. _  _  _
                     |_____|        |_____|         ~ - . _ _ _ _ _>

------------------------------------------------------------------------

      	 IP:\4                         Tel:18183510256

------------------------------------------------------------------------

```

### 修改IP地址或UUID

```BASH
修改IP地址以及UUID
    sed -i "s/192.168.1.199/192.168.1.200/g" /etc/sysconfig/network-scripts/ifcfg-eth0 
    sed -i "/UUID/c UUID=$(uuidgen)" /etc/sysconfig/network-scripts/ifcfg-eth0 
    systemctl restart network
修改主机名
vim /etc/hostname
hostnamectl set-hostname <name>
```

## centos7修改网络方法

```BASH
[root@study ~]# nmtui
#systemctl restart network.service

IPADDR=172.16.0.100
PREFIX=24
GETEWAY=172.16.0.1

```

### 修改默认网卡名称

```BASH
01.修改方式：
安装系统时--菜单的界面--install centos 7选择--tab-- net.ifnames=0 biosdevname=0

02. 在系统中进行修改
	   ①. 编辑网卡配置文件中网卡名称信息
               vim /etc/sysconfig/network-scripts/ifcfg-ens33
NAME=eth0    --- net.ifnames=0
DEVICE=eth0	 --- biosdevname=0
               ②. 重命名网卡配置文件名称信息
               mv ifcfg-ens33 ifcfg-eth0
               ③. 修改网卡名称规则内核文件
               /etc/default/grub->net.ifnames=0 biosdevname=0 ”到GRUB_CMDLINE_LINUX变量后
               ④. 使系统重新加载grub配置文件
修改IP地址
LANG=zh_CN.UTF-8 改为中文显示
[root@localhost ~]# yum search ifconfig
net-tools.x86_64 : Basic networking tools
[root@localhost ~]# yum install net-tools
[root@localhost ~]# yum install lrzsz   直接拖拽文件到xshell  
rz 直接添加上传文件

```

### 关闭selinux

/etc/sysconfig/selinux和/etc/selinux/config配置文件的联系及区别

1.一开始/etc/sysconfig/selinux是/etc/selinux/config的软链接关系

2.由于脚本使用sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux

  对/etc/sysconfig/selinux文件进行修改，导致两者软连接关系破裂，变为一个普通文件，并不再被系统作为selinux的配置文件

关闭selinux，直接修改/etc/selinux/config配置文件，并重启，即可生效

### 系统更新

```bash
[root@study ~]# yum update -y
```



### 时间同步方法

```bash
yum install -y ntpdate
ntpdate
[root@oldboyedu oldboy]# ntpdate ntp1.aliyun.com
16 Apr 12:22:33 ntpdate[25718]: step time server 120.25.115.20 offset 268748517.381960 sec
[root@oldboyedu oldboy]# date
Tue Apr 16 12:22:48 CST 2019
```



### 基础常用工具包

```bash
[root@study ~]# yum install -y tree nmap dos2unix lrzsz nc lsof wget tcpdump htop iftop iotop sysstat nethogs psmisc net-tools bash-completion vim-enhanced bind-utils pcre pcre-devel
```



### centos7 基础安装包

```bash
[root@study ~]# yum install -y psmisc net-tools bash-completion vim-enhanced
```

###  bash问题

带时间的终端显示

```bash
[root@study ~]$PS1='[\u@\h \w \t]\$'
[root@study ~ 15:53:34]#
[root@study ~ 15:53:34]#PS1='[\u@\h \w]\$'
[root@study ~]$PS1='[\[\e[36;40m\]\u@\[\e[31;40m\]\h \[\e[33;40m\]\w\[\e[0m\]]\$ '
 vim /etc/bashrc
 if [ "$PS1" ]; then
 \#  PS1="[\u@\h:\l \W]\\$ "
  \#  PS1='[\[\e[36;40m\]\u@\[\e[31;40m\]\h \[\e[33;40m\]\w\[\e[0m\]]\$ '
PS1='[\[\e[31;40m\]\u\[\e[33;40m\]@\[\e[34;40m\]\h \[\e[33;40m\]\w\[\e[0m\]]\$ '
  fi 
  
[root@localhost ~]# source /etc/bashrc
```

### 常见问题

```bash
linux 出现 -bash-4.2# 的解决办法
-bash-4.2# cp /etc/skel/.bashrc .
-bash-4.2# cp /etc/skel/.bash_profile .
-bash-4.2# ll
总用量 12
-rw-r--r--. 1 root root 166 6月  3 13:12 1.tar.gz
-rw-------. 1 root root 1546 6月  1 21:19 anaconda-ks.cfg
drwxrwxr-x. 2 1000 root  34 6月  3 15:27 test
drwxr-xr-x. 2 root root  6 6月  2 15:04 test2
-rw-r--r--. 1 root root 166 6月  3 13:09 test.tar.gz
-bash-4.2# ls -a
.  1.tar.gz     .bash_history .bashrc test2
.. anaconda-ks.cfg .bash_profile test   test.tar.gz
-bash-4.2# reboot 

注意以后不要再/root执行 rm -rf *了。
```

### 系统语言

   永久修改:

   方法一: 更加有先

```bash
[root@oldboyedu ~]# tail -5 /etc/profile
export LANG='en_US.UTF-8'
```

   方法二:

```bash
[root@oldboyedu ~]# cat /etc/locale.conf 
LANG="zh_CN.UTF-8"
```

​     补充：一条命令即临时设置，又永久设置

```bash
localectl set-locale LANG=zh_CN.GB
```

### vim 个性化设置

```BASH
[root@localhost ~]# vi .vimrc
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

### rocky  网络

```BASH
[root@localhost ~]# systemctl restart NetworkManager
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
NAME=eth0
UUID=6e02ac1f-1c64-4652-8dd3-12c900198228
DEVICE=eth0
ONBOOT=yes
IPADDR=172.16.0.19
PREFIX=24
GATEWAY=172.16.0.1
DNS1=114.114.114.114
DNS2=223.5.5.5
```

镜像源 https://mirrors.sjtug.sjtu.edu.cn/docs/rocky

```BASH
[root@localhost ~]# cp /etc/yum.repos.d/Rocky-BaseOS.repo{,.bak}
[root@localhost ~]# sed -e 's|^mirrorlist=|#mirrorlist=|g' \
    -e 's|^#baseurl=http://dl.rockylinux.org/$contentdir|baseurl=https://mirrors.sjtug.sjtu.edu.cn/rocky|g' \
    -i.bak \
    /etc/yum.repos.d/Rocky-*.repo
    
[root@localhost ~]# dnf makecache

```

https://mirror.sjtu.edu.cn/fedora/epel/

epel 源

```BASH
[root@localhost ~]# wget https://mirror.sjtu.edu.cn/fedora/epel/epel-release-latest-8.noarch.rpm
```

常用软件

```BASH
dnf install -y vim wget net-tools tree nmap dos2unix lrzsz nc lsof tcpdump htop iftop iotop sysstat nethogs git iptables conntrack ipvsadm ipset jq sysstat libseccomp bind-utils
```

tab 补全

```BASH
[root@localhost ~]# dnf install -y bash-completion bash-completion-extras
[root@localhost ~]# source /etc/profile.d/bash_completion.sh
```

多网卡修改脚本

```BASH
#!/bin/bash
#----------------------------------------------
# Author        : 349925756
# Email         : 349925756@qq.com
# Last modified : 2021-06-08 21:31
# Filename      : uuid.sh
# Description   : 
# Version       : 1.1 
#----------------------------------------------
#修改网卡UUID IP地址 主机名，关闭防火墙，selinux 安装常用工具关闭swap 导入kubectl tab补全
#uuid  ip
path_eth0="/etc/sysconfig/network-scripts/ifcfg-eth0"
path_eth1="/etc/sysconfig/network-scripts/ifcfg-eth1"

sed -i "/UUID/c UUID=$(uuidgen)" $path_eth0
sed -i "/UUID/c UUID=$(uuidgen)" $path_eth1                                                                    

sed -i "s/$1/$2/g" $path_eth0
sed -i "s/$4/$5/g" $path_eth1

echo "$3" >/etc/hostname

systemctl stop firewalld && systemctl disable firewalld

sed -i "s/SELINUX=.*/SELINUX=disabled/g" /etc/selinux/config

#\cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime 

systemctl enable chronyd

#sed -ri 's/.*swap.*/#&/' /etc/fstab

reboot

```



清华镜像源

```BASH
[root@localhost ~]# cp /etc/yum.repos.d/CentOS-Base.repo{,.bak}
[root@localhost ~]# sudo sed -e 's|^mirrorlist=|#mirrorlist=|g' \
-e 's|^#baseurl=http://mirror.centos.org|baseurl=https://mirrors.tuna.tsinghua.edu.cn|g' \
          -i.bak \
          /etc/yum.repos.d/CentOS-*.repo

```

epel

```bash
yum install epel-release

sed -e 's!^metalink=!#metalink=!g' \
    -e 's!^#baseurl=!baseurl=!g' \
    -e 's!//download\.fedoraproject\.org/pub!//mirrors.tuna.tsinghua.edu.cn!g' \
    -e 's!http://mirrors!https://mirrors!g' \
    -i /etc/yum.repos.d/epel.repo /etc/yum.repos.d/epel-testing.repo
```

```BASH
[root@localhost ~]# yum clean all
[root@localhost ~]# yum makecache
```



## unubtu 常见操作

21.04系统配置IP的方法如下：

1、打开网络配置文件：sudo vim /etc/netplan/******.yaml（不同系统的文件名有差异，认准后缀.yaml即可。*yaml文件中的缩进必须保持一致，否则会出现报错，不用为每行使用特定的缩进，只需要保持缩进一致就行*）

2、配置yaml文件：

使用DHCP

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:  //这里是有线网卡名字，可以通过ifconfig查看获得
      dhcp4: yes //启用dhcp4
```

使用静态IP

```yaml
network:
  version: 2
  ethernets:
    eth0:
      addresses: [192.168.1.100/24]
      dhcp4: no   //禁用dhcp4   
      dhcp6: no
      gateway4: 192.168.1.1
      nameservers: //DNS服务器列表，多个则用逗号分开        
        addresses: [192.168.1.1,114.114.114.114]
```

配置wifi

```yaml
network:
  version: 2
  wifis:
    wlan0:  //这里是无线网卡名字，可以通过ifconfig查看获得    
      dhcp4: no
      dhcp6: no
      address: [192.168.2.155/24]     
      gateway4: 192.168.1.1
      nameservers:
        addresses: [192.168.1.1,8.8.8.8] 
      access-points:
        "****": //无线网络名称（SSID）
          password: "********" //无线网络密码
```

3、配置完成后启用配置sudo netplan apply

参考：https://netplan.io/examples/



修改ubuntu ls 文件夹背景底色

![image-20211107102146088](linux运维01个性化设置.assets/image-20211107102146088.png)

在.bashrc里加一行，

```sh
LS_COLORS=$LS_COLORS:'ow=1;32:'
```

这样即可取消有些文件夹的绿色底色。其中ow的意思是OTHER_WRITABLE，1的意思是粗体，32的意思是绿色前景

```shell
编码   颜色/动作
0    重新设置属性到缺省设置
1    设置粗体
2    设置一半亮度（模拟彩色显示器的颜色）
4    设置下划线（模拟彩色显示器的颜色）
5    设置闪烁
7    设置反向图象
22    设置一般密度
24    关闭下划线
25    关闭闪烁
27    关闭反向图象
30    设置黑色前景
31    设置红色前景
32    设置绿色前景
33    设置棕色前景
34    设置蓝色前景
35    设置紫色前景
36    设置青色前景
37    设置白色前景
38    在缺省的前景颜色上设置下划线
39    在缺省的前景颜色上关闭下划线
40    设置黑色背景
41    设置红色背景
42    设置绿色背景
43    设置棕色背景
44    设置蓝色背景
45    设置紫色背景
46    设置青色背景
47    设置白色背景
49    设置缺省黑色背景
```

