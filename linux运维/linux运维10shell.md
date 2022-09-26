# SHELL编程

## 环境变量 

set 输出所有变量，全局和局部变量。set -o 显示bash shell的所有参数配置信息。

env 只显示全局变量

declare 输出所有的变量、函数、整数和已经导出的变量。

```bash
[root@web01 ~]# set |tail
{ 
    local quoted=${1//\'/\'\\\'\'};
    printf "'%s'" "$quoted"
}
quote_readline () 
{ 
    local quoted;
    _quote_readline_by_ref "$1" ret;
    printf %s "$ret"
}
[root@web01 ~]# declare |tail
{ 
    local quoted=${1//\'/\'\\\'\'};
    printf "'%s'" "$quoted"
}
quote_readline () 
{ 
    local quoted;
    _quote_readline_by_ref "$1" ret;
    printf %s "$ret"
}
[root@web01 ~]# set -o |tail
notify         	 off
nounset        	 off
onecmd         	 off
physical       	 off
pipefail       	 off
posix          	 off
privileged     	 off
verbose        	 off
vi             	 off
xtrace         	 off
[root@web01 ~]# export NAME=oldboy   1方法
[root@web01 ~]# declare -x NAME=oldboy  2方法
[root@web01 ~]# NAME=oldboy;export NAME   3方法
[root@web01 ~]# echo $NAME
```

### 用户环境变量

```bash
[root@web01 ~]# cat /root/.bashrc  优先级最高
# .bashrc
# User specific aliases and functions
alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'
# Source global definitions
if [ -f /etc/bashrc ]; then
	 . /etc/bashrc
fi
[root@web01 ~]# cat /root/.bash_profile 
# .bash_profile

# Get the aliases and functions
if [ -f ~/.bashrc ]; then
	 . ~/.bashrc
fi
# User specific environment and startup programs
PATH=$PATH:$HOME/bin
export PATH
```

### 全局环境变量

```bash
[root@web01 ~]# cat /etc/profile
# /etc/profile

# System wide environment and startup programs, for login setup
# Functions and aliases go in /etc/bashrc

# It's NOT a good idea to change this file unless you know what you
# are doing. It's much better to create a custom.sh shell script in
# /etc/profile.d/ to make custom changes to your environment, as this
# will prevent the need for merging in future updates.

pathmunge () {
    case ":${PATH}:" in
        *:"$1":*)
            ;;
        *)
            if [ "$2" = "after" ] ; then
                PATH=$PATH:$1
            else
                PATH=$1:$PATH
            fi
    esac
}


if [ -x /usr/bin/id ]; then
    if [ -z "$EUID" ]; then
        # ksh workaround
        EUID=`/usr/bin/id -u`
        UID=`/usr/bin/id -ru`
    fi
    USER="`/usr/bin/id -un`"
    LOGNAME=$USER
    MAIL="/var/spool/mail/$USER"
fi

# Path manipulation
if [ "$EUID" = "0" ]; then
    pathmunge /usr/sbin
    pathmunge /usr/local/sbin
else
    pathmunge /usr/local/sbin after
    pathmunge /usr/sbin after
fi

HOSTNAME=`/usr/bin/hostname 2>/dev/null`
HISTSIZE=1000
if [ "$HISTCONTROL" = "ignorespace" ] ; then
    export HISTCONTROL=ignoreboth
else
    export HISTCONTROL=ignoredups
fi

export PATH USER LOGNAME MAIL HOSTNAME HISTSIZE HISTCONTROL

# By default, we want umask to get set. This sets it for login shell
# Current threshold for system reserved uid/gids is 200
# You could check uidgid reservation validity in
# /usr/share/doc/setup-*/uidgid file
if [ $UID -gt 199 ] && [ "`/usr/bin/id -gn`" = "`/usr/bin/id -un`" ]; then
    umask 002
else
    umask 022
fi

for i in /etc/profile.d/*.sh /etc/profile.d/sh.local ; do
    if [ -r "$i" ]; then
        if [ "${-#*i}" != "$-" ]; then 
            . "$i"
        else
            . "$i" >/dev/null
        fi
    fi
done

unset i
unset -f pathmunge
[root@web01 ~]# cat /etc/bashrc  优先级别高
# /etc/bashrc

# System wide functions and aliases
# Environment stuff goes in /etc/profile

# It's NOT a good idea to change this file unless you know what you
# are doing. It's much better to create a custom.sh shell script in
# /etc/profile.d/ to make custom changes to your environment, as this
# will prevent the need for merging in future updates.

# are we an interactive shell?
if [ "$PS1" ]; then
  if [ -z "$PROMPT_COMMAND" ]; then
    case $TERM in
    xterm*|vte*)
      if [ -e /etc/sysconfig/bash-prompt-xterm ]; then
          PROMPT_COMMAND=/etc/sysconfig/bash-prompt-xterm
      elif [ "${VTE_VERSION:-0}" -ge 3405 ]; then
          PROMPT_COMMAND="__vte_prompt_command"
      else
          PROMPT_COMMAND='printf "\033]0;%s@%s:%s\007" "${USER}" "${HOSTNAME%%.*}" "${PWD/#$HOME/~}"'
      fi
      ;;
    screen*)
      if [ -e /etc/sysconfig/bash-prompt-screen ]; then
          PROMPT_COMMAND=/etc/sysconfig/bash-prompt-screen
      else
          PROMPT_COMMAND='printf "\033k%s@%s:%s\033\\" "${USER}" "${HOSTNAME%%.*}" "${PWD/#$HOME/~}"'
      fi
      ;;
    *)
      [ -e /etc/sysconfig/bash-prompt-default ] && PROMPT_COMMAND=/etc/sysconfig/bash-prompt-default
      ;;
    esac
  fi
  # Turn on parallel history
  shopt -s histappend
  history -a
  # Turn on checkwinsize
  shopt -s checkwinsize
  [ "$PS1" = "\\s-\\v\\\$ " ] && PS1="[\u@\h \W]\\$ "
  # You might want to have e.g. tty in prompt (e.g. more virtual machines)
  # and console windows
  # If you want to do so, just add e.g.
   if [ "$PS1" ]; then
  #   PS1="[\u@\h:\l \W]\\$ "
#      PS1='[\[\e[36;40m\]\u@\[\e[31;40m\]\h \[\e[33;40m\]\w\[\e[0m\]]\$ '
      PS1='[\[\e[31;40m\]\u\[\e[33;40m\]@\[\e[34;40m\]\h \[\e[33;40m\]\w\[\e[0m\]]\$ '
 
  fi
  # to your custom modification shell script in /etc/profile.d/ directory
fi

if ! shopt -q login_shell ; then # We're not a login shell
    # Need to redefine pathmunge, it get's undefined at the end of /etc/profile
    pathmunge () {
        case ":${PATH}:" in
            *:"$1":*)
                ;;
            *)
                if [ "$2" = "after" ] ; then
                    PATH=$PATH:$1
                else
                    PATH=$1:$PATH
                fi
        esac
    }

    # By default, we want umask to get set. This sets it for non-login shell.
    # Current threshold for system reserved uid/gids is 200
    # You could check uidgid reservation validity in
    # /usr/share/doc/setup-*/uidgid file
    if [ $UID -gt 199 ] && [ "`/usr/bin/id -gn`" = "`/usr/bin/id -un`" ]; then
       umask 002
    else
       umask 022
    fi

    SHELL=/bin/bash
    # Only display echos from profile.d scripts if we are no login shell
    # and interactive - otherwise just process them to set envvars
    for i in /etc/profile.d/*.sh; do
        if [ -r "$i" ]; then
            if [ "$PS1" ]; then
                . "$i"
            else
                . "$i" >/dev/null
            fi
        fi
    done

    unset i
    unset -f pathmunge
fi
# vim:ts=4:sw=4
[root@web01 ~]# cat /etc/profile.d/  常用命令变量
256term.csh         colorls.sh          sh.local
256term.sh          csh.local           vim.csh
bash_completion.sh  lang.csh            vim.sh
colorgrep.csh       lang.sh             which2.csh
colorgrep.sh        less.csh            which2.sh
colorls.csh         less.sh             
```

## 登录提示

```bash
[root@web01 ~]# cat /etc/motd  方法一
welcome to linux!
[root@web01 ~]# login out


WARNING! The remote SSH server rejected X11 forwarding request.
Last login: Thu Mar 25 14:12:07 2021 from 192.168.1.5
welcome to linux!
[root@web01 ~]# cat /etc/profile.d/oldboy.sh   方法二
echo "welcome to web01!"
[root@web01 ~]# login out

WARNING! The remote SSH server rejected X11 forwarding request.
Last login: Thu Mar 25 15:11:42 2021 from 192.168.1.5
welcome to linux!
welcome to web01!

```

### 打印环境变量echo printf

```bash
[root@web01 ~]# echo $HOME  用户登录进入的家目录
/root
[root@web01 ~]# echo $UID   当前用户的UID
0 
[root@web01 ~]# echo $PWD   当前目录的绝对路径
/root
[root@web01 ~]# echo $SHELL   当前shell
/bin/bash
[root@web01 ~]# echo $USER   当前用户
root
[root@web01 ~]# printf "$HOME \n"
/root
```

### unset 取消本地变量和环境

```bash
unset USER 
```

变量环境小结：

变量名称大写

变量名称可以在shell或者子shell使用

从用exprot来定义环境变量

输出时$变量名 unset变量名

书写crond定时任务要注意，脚本需要的环境变量最好在所执行的shell中重新定义

希望变量永久生效，在用户环境变量文件或全局变量环境文件定义

定义变量基本技巧

1、 没有引号  定义数字类型可以不用加

2、 单引号   所见即所得

3、 双引号   定义变量使用此项

4、反引号   一般用于引用命令相当于$()

```bash
[root@web01 ~]# echo 'today is date'
today is date
[root@web01 ~]# echo 'today is $(date)'
today is $(date)
[root@web01 ~]# echo "today is $(date)"
today is 2021年 03月 27日 星期六 16:27:14 CST
[root@web01 ~]# echo 'today is `date`'
today is `date`
[root@web01 ~]# echo 'today is `$(date)`'
today is `$(date)`
[root@web01 ~]# echo "today is `date`"
today is 2021年 03月 27日 星期六 16:27:47 CST
```

把一个命令的结果定义为变量

变量名=·ls· 反引号一般人不知道其功能

变量名=$(ls) 常用方法

再使用awk调用shell是是单引号最好是 “’’”这种 双引号中间的单引号可以正确输出。还可以用echo 再加 | 管道方式输出。

### shell特殊重要变量

| $0   | 当前shell文件名（包含路径）                             |
| ---- | ------------------------------------------------------- |
| $n   | shell的第n个参数，n=1-9  大于9需要用{} ${10} 空格隔开   |
| $#   | 当前shell脚本后的参数总个数                             |
| $*   | 当前shell所有传参，不加引号和$@相同，加“”视为单个字符串 |
| $@   | 不加引号和$*相同，加””视为不同的独立字符串              |
| $?   | 获取上一个执行状态0 true 1false                         |
| $$   | 获取当前shell进程PID号                                  |
| $!   | 获取上一个再后台工作的进程号PID                         |
| $_   | 获取之前执行命令的最后一个参数                          |

```bash
[root@web01 /server/scripts]# sh args.sh 
USAGE:/bin/sh args.sh arg1 arg2
error:请输入两个参数
[root@web01 /server/scripts]# sh args.sh  oldboy oldgirl
oldboy oldgirl
[root@web01 /server/scripts]# cat args.sh 
#/bin/bash

if [ $# -ne 2 ] 
   then {
       echo "USAGE:/bin/sh $0 arg1 arg2"
       echo "error:请输入两个参数"
       exit 1 
       }

fi
echo $1 $2
```

## bash shell内置变量

echo eval exec export read shift

echo

| echo参数 | 说明           |
| -------- | -------------- |
| -n       | 不换行输出内容 |
| -e       | 解析转义字符   |
| 转义字符 |                |
| \n       | 换行           |
| \r       | 回车           |
| \t       | 制表符tab      |
| \b       | 退格           |
| \v       | 纵向制表符     |

eval shell执行eval时会把参数组合成一个新命令然后执行。

exec 执行完并终止进程

read 从标准输入读取字符

shift 向左位移执行，$3变$2 $2变$1 $1消失

exit 退出，可以指定一个数字为返回值

```bash
[root@web01 ~]# OLD="i am oldboy,yes oldboy"
[root@web01 ~]# echo $OLD
i am oldboy,yes oldboy
[root@web01 ~]# echo ${OLD/oldboy/oldgirl}
i am oldgirl,yes oldboy
[root@web01 ~]# echo ${OLD//oldboy/oldgirl}
i am oldgirl,yes oldgirl
```

| 表达式             | 说明                                                         |
| ------------------ | ------------------------------------------------------------ |
| ${parameter:-word} | 如果parameter变量值为空则返回word替代parameter变量  用途：可以定义缺省变量 |
| ${parameter:=word} | 量为空word为变量值，位置变量和特殊变量不适用                 |
| ${parameter:?word} | 量为空word被作为标准错误输出，有变量则正常输出               |
| ${parameter:+word} | 变量为空word替代变量值                                       |

## 算术运算符

```bash
[root@web01 /server/scripts]# sh yunsaun.sh 100 5
a + b = 105
a - b = 95
a * b = 500
a / b = 20
a % b = 0
a ** b = 10000000000
[root@web01 /server/scripts]# cat yunsaun.sh 
#!/bin/bash
a=$1
b=$2
echo "a + b = $(( $a + $b))"
echo "a - b = $(( $a - $b))"
echo "a * b = $(( $a * $b))"
echo "a / b = $(( $a / $b))"
echo "a % b = $(( $a % $b))"
echo "a ** b = $(( $a ** $b))"
```

### 简易计算器

```bash
[root@web01 /server/scripts]# sh smu.sh 6+7
13
[root@web01 /server/scripts]# sh smu.sh 6\*7
42
[root@web01 /server/scripts]# cat smu.sh 
#！/bin/bash
echo $(($1))
```

### let 

```bash
[root@web01 /server/scripts]# i=2
[root@web01 /server/scripts]# let i+=8    等同于 ((i=i+8))
[root@web01 /server/scripts]# echo $i
10
```

### expr

```bash
[root@web01 /server/scripts]# expr 2+2
2+2
[root@web01 /server/scripts]# expr 2 + 2
4
[root@web01 /server/scripts]# expr 2 \* 2
4
[root@web01 /server/scripts]# expr 2 \* 22
44
[root@web01 /server/scripts]# expr $i + 6 &> /dev/null    不保留任何输出
[root@web01 /server/scripts]# echo $?        输出返回值，0表示整数，非0表示不是整数
0
[root@web01 /server/scripts]# echo $i
10
[root@web01 /server/scripts]# cat macth.sh 
#!/bin/bash
if [[ `expr match "$1" "[0-9][0-9]*$"` == 0 ]]
   then
      echo "$1 is not a num"
   else
      echo "$1 is a num"
fi
[root@web01 /server/scripts]# sh macth.sh 6
6 is a num
[root@web01 /server/scripts]# sh macth.sh old
old is not a num

```

### bc

```bash
[root@web01 /server/scripts]# echo "scale=7;3333/444"|bc
7.5067567
[root@web01 /server/scripts]# echo "scale=2;3333/444"|bc
7.50
```

### seq

```bash
[root@web01 /server/scripts]# seq -s "+" 10
1+2+3+4+5+6+7+8+9+10
[root@web01 /server/scripts]# seq -s "+" 10|bc
55
[root@web01 /server/scripts]# echo {1..10}|tr " " "+"|bc
55
[root@web01 /server/scripts]# echo {1..10}|tr " " "+"
1+2+3+4+5+6+7+8+9+10
```

### awk

```bash
[root@web01 /server/scripts]# echo "7.7 8.9"|awk '{print ($1 - $2)}'
-1.2
```

### declare

```bash
[root@web01 /server/scripts]# declare -i A=30 B=7
[root@web01 /server/scripts]# A=A+B
[root@web01 /server/scripts]# echo $A
37
```

### $[]

```bash
[root@web01 /server/scripts]# i=5 
[root@web01 /server/scripts]# i=$[i+6]
[root@web01 /server/scripts]# echo $i
11
```

### read

```bash
[root@web01 /server/scripts]# read -t 10 -p "please input one num:"   -t 限制时间 -p 提示
please input one num:22
[root@web01 /server/scripts]# read -t 10 -p "please input one num:"
please input one num:[root@web01 /server/scripts]#
[root@web01 /server/scripts]# cat yunsaun.sh 
#!/bin/bash
#a=$1
#b=$2
read -p " please input two nums:" a b
echo "a + b = $(( $a + $b))"
echo "a - b = $(( $a - $b))"
echo "a * b = $(( $a * $b))"
echo "a / b = $(( $a / $b))"
echo "a % b = $(( $a % $b))"
echo "a ** b = $(( $a ** $b))"
[root@web01 /server/scripts]# sh yunsaun.sh 
 please input two nums:6 2
a + b = 8
a - b = 4
a * b = 12
a / b = 3
a % b = 0
a ** b = 36
```

## shell 脚本的条件测试

| 语法              | 说明                                         |
| ----------------- | -------------------------------------------- |
| test <测试表达式> | 利用test命令完成测试                         |
| [<测试表达式> ]   | []  边界至少有一个空格，跟test一样           |
| [[<测试表达式> ]] | 新语法[[  ]]中间至少一个空格。可以使用通配符 |
| ((<测试表达式> )) | 一般用于if语句里，(())内不需要空格           |

[[ ]] 中 &&、||、>、<等操作符可以用于其中，但不能用于[ ] 。[ ]中使用 -a -o -gt（用于整数）-lt（整数）代替上面的逻辑与，逻辑或 大于 小于。

```bash
[root@web01 ~]# test -f 1.txt && echo true || echo false
false
[root@web01 ~]# ls
349925756@qq.com  cmatrix-1.2a.tar.gz       lolcat-master
anaconda-ks.cfg   dead.letter               lolcat-master.zip
a.txt             logrtate                  test
[root@web01 ~]# test -f a.txt && echo true || echo false
true
[root@web01 ~]# test -f a.txt && echo 1
1
[root@web01 ~]# test -f a.txt || echo 0
[root@web01 ~]# test -f 1.txt || echo 0
0
[root@web01 ~]# char=" "
[root@web01 ~]# test -z "$char" && echo true || echo false
false
[root@web01 ~]# char=""
[root@web01 ~]# test -z "$char" && echo 1 || echo 0
1
[root@web01 ~]# [ -f 1.txt ] && echo true||echo false
false
[root@web01 ~]# [[ -f 1.txt ]] && echo true||echo false
false
[root@web01 ~]# [[ -f a.txt ]] && echo 1
1
[root@web01 ~]# [[ -f 1.txt ]] || echo 0
0
```

### 文件测试表达式

| 常用文件操作符            | 说明                               |
| ------------------------- | ---------------------------------- |
| -d   directory            | 文件存在且为目录                   |
| -f   file                 | 普通文件为真                       |
| -e   exist                | 不辩别文件还是目录，文件存在即为真 |
| -r   read                 | 文件可读为真                       |
| -s   size                 | 文件存在大小不为0为真              |
| -w   write                | 文件可以写入为真                   |
| -x   executable           | 文件可以执行为真                   |
| -L   link                 | 文件为链接文件为真                 |
| f1 -nt f2   nt=newer than | 文件1比2新为真，根据修改时间计算   |
| f1 -ot f2   ot=older than | 文件1比文件2旧为真                 |

```bash
[root@web01 /server/scripts]# cat file_test.sh 
#!/bin/bash
[ -f /etc ] || {
          echo "/etc 不是一个文件，而是一个目录"
          echo "$(ls)"
         }
[root@web01 /server/scripts]# sh file_test.sh 
/etc 不是一个文件，而是一个目录
args.sh
cat_conf.sh
dbtest.php
file_test.sh
macth.sh
nginx_create.sh
smu.sh
yunsaun.sh
```

### 字符串测试表达式

| -n                     | 字符串长度不为0为真  |
| ---------------------- | -------------------- |
| -z                     | 字符串长度为0为假    |
| “string1” = “string2”  | 字符串1=字符串2 真   |
| “string1” != ”string2” | 反之则假 =号两边空格 |

```bash
[root@web01 /server/scripts]# [ "abc" == "abc" ] && echo true ||echo false
true
[root@web01 /server/scripts]# [ -n "abc" ] && echo true ||echo false
true
[root@web01 /server/scripts]# [ -z "abc" ] && echo true ||echo false
false
```

### 整数二元比较操作符

| -eq  | == \| = | 等于 equal             |
| ---- | ------- | ---------------------- |
| -ne  | !=      | 不等 not equal         |
| -gt  | >       | 大于 greater than      |
| -ge  | >=      | 大于等于 greater equal |
| -lt  | <       | 小于less than          |
| -le  | <=      | 小于等于less equal     |

```bash
[ ] 方法
[root@web01 /server/scripts]# [ 2 -eq 1 ] && echo 1 || echo 0
0
[root@web01 /server/scripts]# [ 2 -ne 1 ] && echo 1 || echo 0
1
[root@web01 /server/scripts]# [ 2 -gt 1 ] && echo 1 || echo 0
1
[root@web01 /server/scripts]# [ 2 -ge 1 ] && echo 1 || echo 0
1
[root@web01 /server/scripts]# [ 2 -lt 1 ] && echo 1 || echo 0
0
[root@web01 /server/scripts]# [ 2 -le 1 ] && echo 1 || echo 0
0
[root@web01 /server/scripts]# [ 2 == 1 ] && echo 1 || echo 0
0
[root@web01 /server/scripts]# [ 2 != 1 ] && echo 1 || echo 0
1
[root@web01 /server/scripts]# [ 2 > 1 ] && echo 1 || echo 0
1
[root@web01 /server/scripts]# [ 2 >= 1 ] && echo 1 || echo 0
-bash: [: 2: 期待一元表达式
0
[root@web01 /server/scripts]# [ 2 < 1 ] && echo 1 || echo 0
1
[root@web01 /server/scripts]# [ 2 <= 1 ] && echo 1 || echo 0
-bash: [: 2: 期待一元表达式
0
[[ ]]   方法
[root@web01 /server/scripts]# [[ 2 == 1 ]] && echo true||echo false
false
[root@web01 /server/scripts]# [[ 2 != 1 ]] && echo true||echo false
true
[root@web01 /server/scripts]# [[ 2 > 1 ]] && echo true||echo false
true
[root@web01 /server/scripts]# [[ 2 >= 1 ]] && echo true||echo false
-bash: 条件表达式中有语法错误
-bash: `1' 附近有语法错误
（（））方法
[root@web01 /server/scripts]# ((2 == 1)) && echo 1||echo 0
0
[root@web01 /server/scripts]# ((2 >= 1)) && echo 1||echo 0
1
```

[]  [[]]  (())  中最明显是大于等于这样的逻辑关系运算符，中括号都不能识别，需要使用简写

## 逻辑操作符

| -a   | &&   | and逻辑与,真真为真        |
| ---- | ---- | ------------------------- |
| -o   | \|\| | or逻辑或，一真则真        |
| !    | !    | not逻辑非，真亦假，假亦真 |

```bash
[root@web01 /server/scripts]# [ -f "1.txt" -a "a.txt" ] && echo 1 || echo 00
[root@web01 /server/scripts]# [ -f "1.txt" && "a.txt" ] && echo 1 || echo 0 
-bash: [: 缺少 `]'
0
[root@web01 /server/scripts]# [ -f "1.txt" -o "a.txt" ] && echo 1 || echo 0 
1
```

[] -a -o 适用

[[ ]]  && || 适用

（（）） 不需要

```bash
[root@web01 /server/scripts]# ((1>0 -a 2<1)) && echo 1||echo 0
-bash: ((: 1>0 -a 2<1: 表达式中有语法错误 （错误符号是 "2<1"）
0
[root@web01 /server/scripts]# ((1>0 && 2<1)) && echo 1||echo 0
0
```

### 总结test [] [[]] (())的区别

| 表达式符号         | test                      | []                       | [[]]                                        | (())              |
| ------------------ | ------------------------- | ------------------------ | ------------------------------------------- | ----------------- |
| 边界是否需要空格   | 是                        | 是                       | 是                                          | 否                |
| 逻辑操作符         | ! -a -o                   | ! -a -o                  | ! && \|\|                                   | ! && \|\|         |
| 整数操作比较操作符 | -eq -ne -gt   -ge -le -lt | -eq -ne -gt  -ge -le -lt | -eq -ne -gt -lt -ge -le 或 ==  != > >= < <= | == != > >=   < <= |
| 字符串比较操作符   | =   == !=                 | =   == !=                | =   == !=                                   | =   == !=         |
| 是否支持通配符     | 不支持                    | 不支持                   | 支持                                        | 不支持            |

## if 条件语句的知识与实践

> 第一种
>
> if <条件表达式>
>
>   then 
>
>   指令
>
> fi
>
> ------
>
> 第二种
>
> if <条件表达式> ;then 
>
>   指令
>
> fi
>
> ------
>
> 嵌套
>
> if <条件表达式>
>
>  then
>
>    if <条件表达式>
>
> ​    then
>
> ​    指令
>
> fi
>
> fi
>
> ------
>
> 双分支
>
> if <条件表达式>
>
>   then
>
>   指令集
>
> else
>
>   指令集
>
> fi
>
> ------
>
> 多分支
>
> if <条件表达式>
>
>  then 
>
>  指令
>
> elif <条件表达式>
>
>   then
>
>   指令
>
> elif <条件表达式>
>
>   then
>
>   指令
>
> ……
>
> else
>
>   指令
>
> fi

```bash
[root@web01 /server/scripts]# sh duibi.sh 
请输入两个数字：12 34
12 小于 34
[root@web01 /server/scripts]# sh duibi.sh 
请输入两个数字：34 12
34 大于 12
[root@web01 /server/scripts]# sh duibi.sh 
请输入两个数字：12 12
12 等于 12
[root@web01 /server/scripts]# cat duibi.sh 
#!/bin/bash

#no.1
read -p "请输入两个数字：" a b

if [ $a -lt $b ] 
  then 
   echo "$a 小于 $b"
   exit 0
fi

if [ $a -eq $b ]
   then
   echo "$a 等于 $b"
   exit 0
fi
if [ $a -gt $b ]
   then
   echo "$a 大于 $b"
   exit 0
fi


[root@web01 /server/scripts]# cat duibi.sh 
#!/bin/bash

#no.1
read -p "请输入两个数字：" a b
expr $a + 1 &>/dev/null
RETVAL1=$?
expr $b + 1 &>/dev/null
RETVAL2=$?
[ $RETVAL1 -eq 0 -a $RETVAL2 -eq 0 ] || {
     echo "输入错误"
     exit 2
    }

if [ $a -lt $b ] 
  then 
   echo "$a 小于 $b"
   exit 0
fi

if [ $a -eq $b ]
   then
   echo "$a 等于 $b"
   exit 0
fi
if [ $a -gt $b ]
   then
   echo "$a 大于 $b"
   exit 0
fi
[root@web01 /server/scripts]# sh duibi.sh 
请输入两个数字：2d 33
输入错误
[root@web01 /server/scripts]# sh duibi.sh 
请输入两个数字：12 2d
输入错误
[root@web01 /server/scripts]# sh duibi.sh
请输入两个数字：12 12
12 等于 12
[root@web01 /server/scripts]# sh duibi.sh
请输入两个数字：12 
duibi.sh: 第 14 行:[: 12: 期待一元表达式
duibi.sh: 第 20 行:[: 12: 期待一元表达式
duibi.sh: 第 25 行:[: 12: 期待一元表达式
```

### 判断整数5种方法

```bash
利用返回值查看是否是整数
[root@web01 /server/scripts]# unset i
[root@web01 /server/scripts]# i=5
[root@web01 /server/scripts]# expr $i + 1 &>/dev/null
[root@web01 /server/scripts]# echo $?
0
[root@web01 /server/scripts]# echo $i
5
------------------------------------------------------------
[root@web01 /server/scripts]# i="abc"
[root@web01 /server/scripts]# expr $i + 1 &>/dev/null
[root@web01 /server/scripts]# echo $?
2
------------------------------------------------------------
[root@web01 /server/scripts]# i="oldboy"
[root@web01 /server/scripts]# [ -z "`echo $i|sed 's/[0-9]//g'`" ] && echo int ||echo char
char

[root@web01 /server/scripts]# i=123
[root@web01 /server/scripts]# [ -z "`echo $i|sed 's/[0-9]//g'`" ] && echo int ||echo char
int
[root@web01 /server/scripts]# [ -z "`echo "${i//[0-9]/}"`" ] && echo int ||echo char
int

[root@web01 /server/scripts]# i="abc"
[root@web01 /server/scripts]# [ -z "`echo "${i//[0-9]/}"`" ] && echo int ||echo char
char
------------------------------------------------------------
[root@web01 /server/scripts]# i="oldboy123"
[root@web01 /server/scripts]# [ -n "$i" -a "$i" = "${i//[^0-9]/}" ] && echo "it is num"
[root@web01 /server/scripts]# [ -n "$i" -a "$i" = "${i//[^0-9]/}" ] && echo 1||echo 0
0

[root@web01 /server/scripts]# i=521
[root@web01 /server/scripts]# [ -n "$i" -a "$i" = "${i//[^0-9]/}" ] && echo 1||echo 0
1
[root@web01 /server/scripts]# i=51
[root@web01 /server/scripts]# [ -n "$i" -a "$i" = "${i//[^0-9]/}" ] && echo 1||echo 0
1
------------------------------------------------------------
[root@web01 /server/scripts]# [[ oldboy123 =~ ^[0-9]+$  ]] && echo int ||echo char
char
[root@web01 /server/scripts]# [[ 123 =~ ^[0-9]+$  ]] && echo int ||echo char
int
=~（匹配）用来比较是否符合一个正则表达式, ！~
[root@web01 /server/scripts]# echo "$i"|bc
51
[root@web01 /server/scripts]# echo $i
51
[root@web01 /server/scripts]# i="abc123"
[root@web01 /server/scripts]# echo "$i"|bc
0
```

### 判断输入非空

```bash
[root@web01 /server/scripts]# i=""
[root@web01 /server/scripts]# [ ${#i} -le 0 ] && echo 1|| echo 0
1
[root@web01 /server/scripts]# i="old"
[root@web01 /server/scripts]# [ ${#i} -le 0 ] && echo 1|| echo 0
0
```

修改上面if

```bash
read -p "please input two numbers:" a b
#
expr $a + 1 &>/dev/null
RETVAL1=$?
expr $b + 1 &>/dev/null
RETVAL2=$?
#
if [ ${#a} -le 0 -a ${#b} -le 0 ] ;then
          echo "IS empy, please try again"
          exit  1
#elif [ $# -ne 2];then
 #          echo “USAGE:$0 ARG1 ARG2”
#exit 2

判断输入是否是2个参数
#
elif [ $RETVAL1 -ne 0 -o  $RETVAL2 -ne 0 ];then
          echo "please input numbers"
          exit 3
         
#
elif [ $a -lt $b ] ;then
     echo "$a less than $b"
     exit 0
elif [ $a -eq $b ] ;then
     echo "$a equal $b"
     exit 0
else
     echo "$a greater than $b"
     exit 0
fi

[root@web01 /server/scripts]# sh duibi.sh 
please input two numbers:12 12
12 equal 12
[root@web01 /server/scripts]# sh duibi.sh 
please input two numbers:12 23
12 less than 23
[root@web01 /server/scripts]# sh duibi.sh 
please input two numbers:23 12
23 greater than 12
[root@web01 /server/scripts]# sh duibi.sh 
please input two numbers:
IS empy, please try again
[root@web01 /server/scripts]# sh duibi.sh 
please input two numbers:2d 
please input numbers
[root@web01 /server/scripts]# sh duibi.sh 
please input two numbers:12 2d
please input numbers
```

### 判断字符串长度

```bash
[root@web01 /server/scripts]# [ -z "abc" ] && echo 1 || echo 0
0
[root@web01 /server/scripts]# [ -n "abc" ] && echo 1 || echo 0
1
[root@web01 /server/scripts]# char="abc123"
[root@web01 /server/scripts]# [ ${#char} -eq 0 ]&& echo 1 || echo 0
0
[root@web01 /server/scripts]# [ `expr length "oldboy"` -eq 0 ] && echo 1 || echo 0
0
[root@web01 /server/scripts]# [ `echo "oldboy"|wc -L` -eq 0 ] && echo 1 || echo 0
0
[root@web01 /server/scripts]# [ `echo "oldboy"|awk '{print length}'` -eq 0 ] && echo 1 || echo 0 
0
```

## shell函数

常见格式

function 函数名() {

 指令

 return n 

}

 

简化

函数名（）{

指令

return n

}

 ```bash
 [root@web01 /server/scripts]# sh function_test.sh 
 i am oldboy
 I am oldgirl
 [root@web01 /server/scripts]# cat function_test.sh 
 #!/bin/bash
 
 function oldboy(){
            echo "i am oldboy"
           }
 
 oldgirl(){
           echo "I am oldgirl"
          }
 
 oldboy
 oldgirl
 ```

分离函数体和执行函数体的脚本文件

/etc/init.d/functions 是系统函数，添加函数要在return 0 之前才能生效。

```bash
[root@web01 /server/scripts]# cat >>/etc/init.d/functions<<-EOF
oldboy(){
echo "i am oldboy"
}
EOF
[root@web01 /server/scripts]# tail -5 /etc/init.d/functions

oldboy(){
echo "i am oldboy"
}

return 0

[root@web01 /server/scripts]# vim function_test.sh 
[root@web01 /server/scripts]# cat function_test.sh 
#!/bin/bash

#function oldboy(){
#           echo "i am oldboy"
#          }
#
#oldgirl(){
#          echo "I am oldgirl"
#         }
#
#oldboy
#oldgirl

[ -f /etc/init.d/functions ]&& . /etc/init.d/functions || exit 1
oldboy

[root@web01 /server/scripts]# tail -6 /etc/init.d/functions
oldboy(){
  echo "i am oldboy ,you are  $1"
}


return 0
[root@web01 /server/scripts]# tail -2 function_test.sh 
[ -f /etc/init.d/functions ] && . /etc/init.d/functions || exit 1
oldboy $1
[root@web01 /server/scripts]# sh function_test.sh test
i am oldboy ,you are  test
```

传参转换脚本文件执行

```bash
[root@web01 /server/scripts]# cat url_check.sh 
#!/bin/bash
#判断输入是否为1个参数
function usage(){
           echo $"usage $0:url"
           exit 1
          }


function check_url(){
           wget --spider -q -o /dev/null --tries=1 -T 5 $1
           if [ $? -eq 0 ];then
              echo "$1 is ok"
           else
              echo "$1 is down"
           fi
          }
function main(){
          if [ $# -ne 1 ] ;then         
          usage
          fi
          check_url $1
         }
main $*
```

```bash
[root@web01 /server/scripts]# cat url_check.sh 
#!/bin/bash
#判断输入是否为1个参数
. /etc/init.d/functions
function usage(){
           echo $"usage $0:url"
           exit 1
          }


function check_url(){
           wget --spider -q -o /dev/null --tries=1 -T 5 $1
           if [ $? -eq 0 ];then
           #   echo "$1 is ok"
              action "$1 is ok" /bin/true
           else
           #   echo "$1 is down"
              action "$1 is down"  /bin/false
           fi
          }
function main(){
          if [ $# -ne 1 ] ;then         
          usage
          fi
          check_url $1
         }
main $*

[root@web01 /server/scripts]# sh  url_check.sh www.baidu.com
www.baidu.com is ok                                        [  确定  ]
[root@web01 /server/scripts]# sh  url_check.sh www.baiduxxx.com
www.baiduxxx.com is down                                   [失败]
```

函数部分后期自己多实践

## case 条件语句的应用

语法

case “变量” in

  值1 )

​    指令1…

​    ;;

  值2)

​    指令2

​    ;;

*)

​    指令3

esac

```bash
[root@web01 /server/scripts]# cat case_test.sh 
#!/bin/bash
#
function usage(){
       echo "USAGE :$0 input (0-9):"
       exit 1
      }
read -p "please input (0-9) numbers "  a
case $a in 
   1)
      echo "number is $a"
      ;;
   2)
      echo "number is $a"
      ;;
   [3-9])
      echo "number is $a"
      ;;
    *)
      usage
esac 

[root@web01 /server/scripts]# sh case_test.sh 
please input (0-9) numbers 34
USAGE :case_test.sh input (0-9):
[root@web01 /server/scripts]# sh case_test.sh 
please input (0-9) numbers 3
number is 3
[root@web01 /server/scripts]# sh case_test.sh 
please input (0-9) numbers 1
number is 1
[root@web01 /server/scripts]# sh case_test.sh 
please input (0-9) numbers d
USAGE :case_test.sh input (0
```

```bash
[root@web01 /server/scripts]# echo -e "\E[1;31m red \E[0m"
 red 
[root@web01 /server/scripts]# echo -e "\E[2;31m red \E[0m"
 red 
[root@web01 /server/scripts]# echo -e "\E[3;31m red \E[0m"
 red 
[root@web01 /server/scripts]# echo -e "\E[4;31m red \E[0m"
 red 
[root@web01 /server/scripts]# echo -e "\E[5;31m red \E[0m"
 red 
[root@web01 /server/scripts]# echo -e "\E[6;31m red \E[0m"
 red 
[root@web01 /server/scripts]# echo -e "\E[7;31m red \E[0m"
 red 
[root@web01 /server/scripts]# echo -e "\E[8;31m red \E[0m"
 red 
[root@web01 /server/scripts]# echo -e "\E[9;31m red \E[0m"
 red 
[root@web01 /server/scripts]# echo -e "\033[1;31m red \033[0m"
 red
```

E=033

```bash
[root@web01 /server/scripts]# cat menu_test.sh 
#!/bin/bash
function usage(){
          echo "usage :$0 please menu number: "
          exit 1
          }

echo '
   ###################################################
   1.apple
   2.pear
   3.banana
   4.cherry
   ##################################################
   '

read -p "please input menu numbers: " a

case  $a in
  1)
    echo -e "\033[1;31m apple \033[0m"
    ;;
  2)
    echo -e "\033[1;32m pear \033[0m"
    ;;
  3)
    echo -e "\033[1;33m banana \033[0m"
   ;; 
  4)
   echo -e "\033[1;34m cherry \033[0m"
  ;;
  *)
   usage
esac
```

![image-20210818235043091](./linux运维10 shell.assets\image-20210818235043091.png)

## while until循环

while <条件表达式>

do 

 指令

done

------

until <条件表达式>

do

  指令

done

------

until 条件不成立才会循环，使用率极低

## 后台运行的三种方法

1、 sh xxx.sh &

2、 nohup xxx.sh &

3、 screen xxx.sh

| 用法          | 说明                         |
| ------------- | ---------------------------- |
| sh while.sh & | 把脚本放到后台运行           |
| ctl+c         | 停止当前脚本或任务           |
| ctl+z         | 暂停当前脚本或任务           |
| bg            | 把当前任务或脚本放到后台运行 |
| fg            | 前台执行 fg 任务编号         |
| jobs          | 查看当前后台脚本             |
| kill          | 关闭执行 kill % 任务编号     |

```bash
[root@web01 /server/scripts]# jobs
[2]+  运行中               nohup free -h -s 10 &
[root@web01 /server/scripts]# kill % 2
[root@web01 /server/scripts]# jobs
[2]+  已终止               nohup free -h -s 10
[root@web01 /server/scripts]# jobs
```

## 休息命令

```bash
sleep 1  
usleep 1000000   =1秒
```

## 关于进程管理命令

kill killall pkill  杀死进程 

ps  查看进程

pstree  显示进程树

top   显示进程

renice  改变优先权

nohup  用户退出后继续工作

pgrep  查找匹配条件的系统调用情况

strace 跟踪一个进程的系统调用情况

ltrace 跟踪进程调用库函数的情况

## while 循环读取文件

### exec 

```bash
exec <file
sum=0
while read line
do 
cmd
done
```

### cat

```bash
cat file_path|while read line
do
  cmd
done
```

### 循环追加

```bash
while read line
do 
cmd
done <file
```

```bash
[root@web01 /server/scripts]# cat menu_test.sh |while read line; do echo $line; done
#!/bin/bash
function usage(){
echo "usage :$0 please menu number: "
exit 1
}

echo '
###################################################
1.apple
2.pear
3.banana
4.cherry
##################################################
'

read -p "please input menu numbers: " a

case $a in
1)
echo -e "033[1;31m apple 033[0m"
;;
2)
echo -e "033[1;32m pear 033[0m"
;;
3)
echo -e "033[1;33m banana 033[0m"
;;
4)
echo -e "033[1;34m cherry 033[0m"
;;
*)
usage
esac
```

## for select循环

1

```bash
for 变量名 in 变量取值列表
do 
   指令
done
```

2

```bash
for ((exp1;exp2;exp3))
do
   指令
done
```

```bash
[root@web01 /server/scripts]# cat 9_9.sh 
#!/bin/bash
for i in {1..9}
do
 # a=$((i+1)) 
  for j in {1..9}
  do
    if [ $i -ge $j ];then
        if (( $i * $j >9));then
           echo -en " $i * $j = $((i*j)) "
        else
           echo -en " $i * $j = $((i*j))  "
       fi
    fi
  done
  echo  "   "
done
[root@web01 /server/scripts]# sh 9_9.sh 
 1 * 1 = 1     
 2 * 1 = 2   2 * 2 = 4     
 3 * 1 = 3   3 * 2 = 6   3 * 3 = 9     
 4 * 1 = 4   4 * 2 = 8   4 * 3 = 12  4 * 4 = 16    
 5 * 1 = 5   5 * 2 = 10  5 * 3 = 15  5 * 4 = 20  5 * 5 = 25    
 6 * 1 = 6   6 * 2 = 12  6 * 3 = 18  6 * 4 = 24  6 * 5 = 30  6 * 6 = 36    
 7 * 1 = 7   7 * 2 = 14  7 * 3 = 21  7 * 4 = 28  7 * 5 = 35  7 * 6 = 42  7 * 7 = 49    
 8 * 1 = 8   8 * 2 = 16  8 * 3 = 24  8 * 4 = 32  8 * 5 = 40  8 * 6 = 48  8 * 7 = 56  8 * 8 = 64    
 9 * 1 = 9   9 * 2 = 18  9 * 3 = 27  9 * 4 = 36  9 * 5 = 45  9 * 6 = 54  9 * 7 = 63  9 * 8 = 72  9 * 9 = 81    

```

![image-20210818235549782](./linux运维10 shell.assets\image-20210818235549782.png)

```bash
[root@web01 /server/scripts]# cat 1_100.sh 
#!/bin/bash
for ((i=1 ;i<=100;i++))
do
 ((sum+=i))
done
echo $sum
[root@web01 /server/scripts]# sh 1_100.sh 
5050
[root@web01 /server/scripts]# sh 1_100.sh 
5050
[root@web01 /server/scripts]# cat 1_100.sh 
#!/bin/bash
#for ((i=1 ;i<=100;i++))
#do
# ((sum+=i))
#done
#echo $sum

i=0
while ((i<=100))
do
 ((j+=i))
 ((i++))
done
echo $j
```

批量创建用户并设置随机密码

```bash
[root@web01 /server/scripts]# cat useradd.sh 
#!/bin/bash

. /etc/init.d/functions

function usage(){
           echo "USAGE: $0  sh useradd.sh name:"
           exit 1
          }

user="$1"
passfile="/tmp/user_passwd.log"

for  i in {1..5}
do
  mypasswd="`echo "test $RANDOM" |md5sum|cut -c 3-12`"
  useradd $user$i &>/dev/null &&\
  echo "$mypasswd"|passwd --stdin $user$i &>/dev/null &&
  echo -e "user:$user$i\tpassword:$mypasswd">>$passfile
  if [ $? -eq 0 ];then
      action "$user$i create success" /bin/true
  else
      action "$user$i create fail " /bin/false
  fi
done      
[root@web01 /server/scripts]# sh useradd.sh test
test1 create success                                       [  确定  ]
test2 create success                                       [  确定  ]
test3 create success                                       [  确定  ]
test4 create success                                       [  确定  ]
test5 create success                                       [  确定  ]
[root@web01 /server/scripts]# cat /tmp/user_passwd.log 
user:test1	 password:6c846e7419
user:test2	 password:99336cc360
user:test3	 password:939864c92b
user:test4	 password:7717819e60
user:test5	 password:f1e2469c81       
```

文件批量设置用户密码

```bash
chpasswd < file
file 内容connet
username1：password1
username2：password2
```

## linux系统随机数

```bash
[root@web01 /server/scripts]# echo $RANDOM
12413
[root@web01 /server/scripts]# openssl rand -base64 8
7C5gPk5vFgU=
[root@web01 /server/scripts]# openssl rand -base64 80
tkmEXTkHtZAzENWk8fW5NYERRxgITCXcJ2PRWhBYLyaEO71qsN+dLTSj+HfTrTIG
VVMz4yk0hjZf8ZQcggkRIpR96uLIsOyftd4t62cjzfE=
[root@web01 /server/scripts]# date +%s%N
1617187807783418416
[root@web01 /server/scripts]# head /dev/urandom|cksum
560589838 1628
[root@web01 /server/scripts]# head /dev/urandom|cksum
728659032 2020
[root@web01 /server/scripts]# uuidgen 
0d086a2b-a0b3-450c-a6cb-e499b93e100a
[root@web01 /server/scripts]# uuidgen 
8762b0e8-37a0-45c6-9c45-9d4771553a37
[root@web01 /server/scripts]# yum install expect
[root@web01 /server/scripts]# mkpasswd -l 10 -d 2 -c 3 -s 1
2RVxw#m8dv
参数说明
```

| -l   | 指定密码长度             |
| ---- | ------------------------ |
| -d   | 指定密码中数字的数量     |
| -c   | 指定密码中小写字母的数量 |
| -C   | 大写字母的数量           |
| -s   | 特殊字符的数量           |

RANDOM 0-32767 

## select

select 变量名 [in 菜单取值]

do

  指令

done

```bash
[root@web01 /server/scripts]# cat select_test.sh 
#!/bin/bash
select name in appler banana pear 
do 
   echo $name
done

[root@web01 /server/scripts]# sh select_test.sh 
1) appler
2) banana
3) pear
#? 1
appler
#? 2
banana
#? 3
pear
采用数组做列表
#!/bin/bash
array=(appler banana pear)
select name in "${array[@]}"
do 
   echo $name
done

[root@web01 /server/scripts]# sh select_test.sh 
1) appler
2) banana
3) pear
#? 2
banana
#? 3
pear
```

\#？ 提示不友好 可以定义ps3变量

$reply 就是调用菜单项的数字

## 循环控制及状态返回值

| 命令       | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| break n    | 省略n跳出整个循环，n表示跳出几层循环                         |
| continue n | 省略n跳过本次循环，n表示退到n层                              |
| exit n     | 退出当前shell，n为上次执行的状态返回值下个shell $？获取n的值 |
| return n   | 用于在函数里作为函数的返回值判断执行是否正确，在下个shell可以通过$?接收 |

## ip子接口

```bash
[root@web01 /server/scripts]# ifconfig eth0:0 192.168.1.33/24 up
[root@web01 /server/scripts]# ifconfig 
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.197  netmask 255.255.255.0  broadcast 192.168.1.255
        inet6 fe80::203d:5bc8:694a:e234  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:37:ad:33  txqueuelen 1000  (Ethernet)
        RX packets 16119  bytes 11688559 (11.1 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 7244  bytes 884262 (863.5 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

eth0:0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.33  netmask 255.255.255.0  broadcast 192.168.1.255
        ether 00:0c:29:37:ad:33  txqueuelen 1000  (Ethernet)

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

[root@web01 /server/scripts]# ifconfig eth0:0 192.168.1.33/24 down
SIOCSIFNETMASK: 无法指定被请求的地址
```

### ip addr

```bash
[root@web01 /server/scripts]# ip add
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:37:ad:33 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.197/24 brd 192.168.1.255 scope global noprefixroute eth0
       valid_lft forever preferred_lft forever
    inet 192.168.1.33/24 scope global secondary eth0:0
       valid_lft forever preferred_lft forever
    inet6 fe80::203d:5bc8:694a:e234/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
[root@web01 /server/scripts]# ip addr del 192.168.1.33/24 dev eth0 label eth0:0
```

## 数组

array=(1 2 3 4 ….)

array=([1]=a [2]=b [3]=c)

array[0]=a;array[1]=b;array[2]=c

array=($(cmd)) | array=(`cmd`)

unset array 删除整个数组

unset array [1] 删除第二个数组元素

替换数组内容

${array [@或者*]/查找字符/替换字符}，不会修改数组，

${array[@]#o*}   从左边匹配删除最短的数组元素

${array[@]##o*}  匹配最长元素删除

${array[@]%f*}  从右匹配最短元素删除

${array[@]%%f*}  从右边匹配最长元素删除

```bash
[root@web01 /server/scripts]# array=($(ls))
[root@web01 /server/scripts]# echo $array 
1_100.sh
[root@web01 /server/scripts]# echo $array[2] 
1_100.sh[2]
[root@web01 /server/scripts]# echo ${array[2]}
args.sh
[root@web01 /server/scripts]# echo ${array[@]}
1_100.sh 9_9.sh args.sh args.sh.bak case_test.sh cat_conf.sh dbtest.php duibi.sh file_test.sh function_test.sh macth.sh menu_test.sh nginx_create.sh nohup.out passfile select_test.sh smu.sh url_check.sh useradd.sh yunsaun.sh
```

### shell数组的重要命令

| array=(1 2 3 4)               | 静态数组         |
| ----------------------------- | ---------------- |
| array=($(ls))                 | 动态数组         |
| array[4]=5                    | 数组元素赋值     |
| ${array[@]} \| ${array[*]}    | 打印所有元素     |
| ${#array[@]}  \| ${#array[*]} | 打印字符串长度   |
| ${array[i]}                   | 打印单个数组元素 |

## 60秒倒计时

```bash
[root@web01 /server/scripts]# cat 60.sh 
#!/bin/bash
for ((i=1;i<60;i++))
do
   echo -en "\r"
   echo -en "\033[1;31m $i \033[0m "
   sleep 1
done
#!/bin/bash
for ((i=1;i<60;i++))
do
   echo -en "\r"
   echo -en "\033[1;31m $i \033[0m "
   sleep 1
done
echo "$(animalsay h)"
```

## shell必须掌握

1、 系统及各类服务的监控脚本，文件，磁盘，内存，端口，URL监控报警，短信，微信，邮件

2、 监控网站目录下的文件是否被篡改以及网站站点被破坏如何快速恢复脚本

3、 各类服务rsync,nginx,mysql等启动及停止脚本

4、 mysql主从复制监控报警，以及自动处理不复制故障脚本

5、 一键配置mysql多实例，一键配置mysql主从部署脚本

6、 监控httpmysql,rsync,nfs,memcached等服务是否异常的生产脚本

7、 一键软件安装及优化

8、 mysql多实例启动脚本，分库，分表自动备份

9、 根据网络连接数及web日志PV数封IP

10、 监控PV及流量，并且对信息统计

11、 web服务器多个URL地址是否异常

12、 系统配置优化

13、 TCP链接状态及IP统计报警脚本

14、 批量创建用户并设置随机8位密码

## shell脚本规范

第一行注释

\#！/bin/bash  #!/bin/sh

版权注释

```bash
"设置行号"
set nu
"自动语法高亮"
syntax on
"自动缩进"
set autoindent
"关闭兼容模式"
set nocompatible
"激活鼠标"
set mouse=a
"开启语法"
syntax enable
"tab缩进4个空格"
set tabstop=4
"设定<< >>移动宽度4"
set shiftwidth=4
"自动缩进"
set ai
"智能缩进"
set si
"显示标尺"
set ruler
"显示匹配的[]{}"
set showmatch
"编码设置"
set encoding=utf-8
set fileencodings=utf-t
set termencoding=utf-8
"开启新行时使用智能自动缩进"
set smartindent
set cin
set showmatch
"背景色"
"set background=dark"
"设置光标下划线"
set cursorline

autocmd BufNewFile *.sh exec ":call AddTitle()"
"map <F7> ms:call AddAuthor()<cr>'s"

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
```

后期编辑sh结尾的文件都会自动加上以上注释

高级命名

1、 使用统一后缀.sh

2、 模块的启动或停止统一脚本start_模块.sh stop_模块.sh

3、 监控脚本*_mon.sh

4、 控制脚本一般为 *_ctl.sh为后缀

 

shell最好能使用框架信息，方便后期使用。机器名，端口，用户名，密码这些信息定义为变量统一放到一个.conf文件中脚本开始调用  source conf/xx.conf

将程序的功能分段、分模块采用函数来实现。/etc/profile 定义全局变量

公共函数放/etc/init.d/functions source方法调用

 

bash 参数

sh -nvx 

n 不执行，只检查语法错误

v 先显示再检查

x 执行的脚本内容及输出显示到屏幕上

 

set -nvx

n读不执行

v显示读取的所有行

x显示所有命令及参数

 

set -v 之后tab会出现很对文件 set +v 取消设置

set -x 之后会显示很多变量 set +x 取消

bashdb shellcheck 两个小工具也可以但是很少使用。

## shell开发环境的配置和优化

```bash
把vi 别名改成vim
[root@web01 /server/scripts]# tail -1 /etc/profile
alias vi=vim
[root@web01 /server/scripts]# echo 'alias vi=vim' >>/etc/profile
 vimrc 文件的修改和优化
VIM下遇到复制的代码格式很乱可以esc返回命令模式再进入可视模式选择要调整的行最后按=按键即可。
```

## linux信号及trap命令

```bash
[root@web01 /server/scripts]# kill -l
 1) SIGHUP	  2) SIGINT	  3) SIGQUIT	  4) SIGILL	  5) SIGTRAP
 6) SIGABRT	  7) SIGBUS	  8) SIGFPE	  9) SIGKILL	 10) SIGUSR1
11) SIGSEGV	 12) SIGUSR2	 13) SIGPIPE	 14) SIGALRM	 15) SIGTERM
16) SIGSTKFLT	 17) SIGCHLD	 18) SIGCONT	 19) SIGSTOP	 20) SIGTSTP
21) SIGTTIN	 22) SIGTTOU	 23) SIGURG	 24) SIGXCPU	 25) SIGXFSZ
26) SIGVTALRM	 27) SIGPROF	 28) SIGWINCH	 29) SIGIO	 30) SIGPWR
31) SIGSYS	 34) SIGRTMIN	 35) SIGRTMIN+1	 36) SIGRTMIN+2	 37) SIGRTMIN+3
38) SIGRTMIN+4	 39) SIGRTMIN+5	 40) SIGRTMIN+6	 41) SIGRTMIN+7	 42) SIGRTMIN+8
43) SIGRTMIN+9	 44) SIGRTMIN+10	 45) SIGRTMIN+11	 46) SIGRTMIN+12	 47) SIGRTMIN+13
48) SIGRTMIN+14	 49) SIGRTMIN+15	 50) SIGRTMAX-14	 51) SIGRTMAX-13	 52) SIGRTMAX-12
53) SIGRTMAX-11	 54) SIGRTMAX-10	 55) SIGRTMAX-9	 56) SIGRTMAX-8	 57) SIGRTMAX-7
58) SIGRTMAX-6	 59) SIGRTMAX-5	 60) SIGRTMAX-4	 61) SIGRTMAX-3	 62) SIGRTMAX-2
63) SIGRTMAX-1	 64) SIGRTMAX	 
[root@web01 /server/scripts]# trap -l
 1) SIGHUP	  2) SIGINT	  3) SIGQUIT	  4) SIGILL	  5) SIGTRAP
 6) SIGABRT	  7) SIGBUS	  8) SIGFPE	  9) SIGKILL	 10) SIGUSR1
11) SIGSEGV	 12) SIGUSR2	 13) SIGPIPE	 14) SIGALRM	 15) SIGTERM
16) SIGSTKFLT	 17) SIGCHLD	 18) SIGCONT	 19) SIGSTOP	 20) SIGTSTP
21) SIGTTIN	 22) SIGTTOU	 23) SIGURG	 24) SIGXCPU	 25) SIGXFSZ
26) SIGVTALRM	 27) SIGPROF	 28) SIGWINCH	 29) SIGIO	 30) SIGPWR
31) SIGSYS	 34) SIGRTMIN	 35) SIGRTMIN+1	 36) SIGRTMIN+2	 37) SIGRTMIN+3
38) SIGRTMIN+4	 39) SIGRTMIN+5	 40) SIGRTMIN+6	 41) SIGRTMIN+7	 42) SIGRTMIN+8
43) SIGRTMIN+9	 44) SIGRTMIN+10	 45) SIGRTMIN+11	 46) SIGRTMIN+12	 47) SIGRTMIN+13
48) SIGRTMIN+14	 49) SIGRTMIN+15	 50) SIGRTMAX-14	 51) SIGRTMAX-13	 52) SIGRTMAX-12
53) SIGRTMAX-11	 54) SIGRTMAX-10	 55) SIGRTMAX-9	 56) SIGRTMAX-8	 57) SIGRTMAX-7
58) SIGRTMAX-6	 59) SIGRTMAX-5	 60) SIGRTMAX-4	 61) SIGRTMAX-3	 62) SIGRTMAX-2
63) SIGRTMAX-1	 64) SIGRTMAX	 
```

| 信号     | 说明                               |
| -------- | ---------------------------------- |
| HUP(1)   | 挂起，通常终端掉线或者用户退出导致 |
| INT(2)   | 中断ctrl+c                         |
| QUIT(3)  | 退出ctrl+\                         |
| ABRT(6)  | 中止严重执行错误导致               |
| ALRM(14) | 报警，处理超时                     |
| TERM(15) | 终止，系统关机                     |
| TSTP(20) | 停止进程的运行CTRL+z               |

SIG省略

trap command signal

signal 接收到的信号

```bash
[root@web01 /server/scripts]# trap 'echo oldboy' 2
[root@web01 /server/scripts]# ^Coldboy
[root@web01 /server/scripts]# ^Coldboy
[root@web01 /server/scripts]# trap 'echo oldboy' INT
[root@web01 /server/scripts]# ^Coldboy
[root@web01 /server/scripts]# stty -a
speed 38400 baud; rows 33; columns 114; line = 0;
intr = ^C; quit = ^\; erase = ^?; kill = ^U; eof = ^D; eol = <undef>; eol2 = <undef>; swtch = <undef>; start = ^Q;
stop = ^S; susp = ^Z; rprnt = ^R; werase = ^W; lnext = ^V; flush = ^O; min = 1; time = 0;
-parenb -parodd -cmspar cs8 -hupcl -cstopb cread -clocal -crtscts
-ignbrk -brkint -ignpar -parmrk -inpck -istrip -inlcr -igncr icrnl ixon -ixoff -iuclc -ixany -imaxbel -iutf8
opost -olcuc -ocrnl onlcr -onocr -onlret -ofill -ofdel nl0 cr0 tab0 bs0 vt0 ff0
isig icanon iexten echo echoe echok -echonl -noflsh -xcase -tostop -echoprt echoctl echoke
[root@web01 /server/scripts]# ^Coldboy
[root@web01 /server/scripts]# trap "" INT
[root@web01 /server/scripts]# 
[root@web01 /server/scripts]# trap ":" INT
[root@web01 /server/scripts]# ^C
```

案例：应用trap 实现普通用户登录就打印菜单，选择有限操作。

## expect 自动化交互式程序

```bash
[root@web01 ~]# yum install -y expect
[root@web01 ~]# rpm -qa expect
expect-5.45-14.el7_1.x86_64
```

```bash
[root@web02 ~]# yum install -y expect
[root@web01 ~]# ssh -p22 root@192.168.1.196 uptime
root@192.168.1.196's password: 
 20:24:03 up 17 min,  1 user,  load average: 0.26, 0.20, 0.15
[root@web01 ~]# ssh -p22 root@192.168.1.196 df -h
root@192.168.1.196's password: 
文件系统        容量  已用  可用 已用% 挂载点
devtmpfs        476M     0  476M    0% /dev
tmpfs           487M     0  487M    0% /dev/shm
tmpfs           487M  7.7M  479M    2% /run
tmpfs           487M     0  487M    0% /sys/fs/cgroup
/dev/sda3        18G  2.4G   16G   14% /
/dev/sda1       509M  153M  356M   31% /boot
tmpfs            98M     0   98M    0% /run/user/0
注意：自动交互
[root@web01 /server/scripts]# expect test.exp 
spawn ssh root@192.168.1.196 uptime
root@192.168.1.196's password: 
 20:46:08 up 39 min,  1 user,  load average: 0.00, 0.03, 0.07
[root@web01 /server/scripts]# cat test.exp  注意后缀名
#!/usr/bin/expect   注意执行脚本的程序
#----------------------------------------------
# Author        : 349925756
# Email         : 349925756@qq.com
# Last modified : 2021-04-02 20:41
# Filename      : test.sh
# Description   : 
# Version       : 1.1 
#----------------------------------------------
 
#Notes:  
spawn ssh root@192.168.1.196 uptime    链接命令
 
expect "*password"   选项密码
 
send "*******\n"    发送密码

expect eof
[root@web01 /server/scripts]# which expect
/usr/bin/expect
```

### expect 重点

spawn expect必须的命令，没有任何命令都无法执行

spawn 【选项】 【需要自动交互的命令或程序】

-open 启动文件进程

-ignore 忽略某些信号

expect 表达式 [动作]

```sh
expect "*password" {send "*******\r"} 
```

```bash
[root@web01 /server/scripts]# cat test.exp 
#!/usr/bin/expect

spawn ssh root@192.168.1.196 uptime

expect {
	  "yes/no" {exp_send "yes\r";exp_continue}
	  "*password" {exp_send "********\r"}  类似多个expect
}
#send "hongfei007\n"

expect eof
```

```bash
[root@web01 /server/scripts]# cat read_test.*
#!/usr/bin/expect
spawn /bin/sh read_test.sh
expect {
     "username" {exp_send "test\r";exp_continue}
	  "*pass*" {send "123456\r";exp_continue}
	  "*mail*" {exp_send "349925756@qq.com\r"}
     }

expect eof
#!/bin/bash
#----------------------------------------------
# Author        : 349925756
# Email         : 349925756@qq.com
# Last modified : 2021-04-02 21:05
# Filename      : read_test.sh
# Description   : 
# Version       : 1.1 
#----------------------------------------------
 
#Notes:  
read -p "please input your username:"  name
read -p "please input your password:"  password
read -p "please input your e-mail:"  mail
echo -n "your username is $name "
echo -n "your password is $password"
echo "your email is $mail"
```

以实现自动交互

```bash
[root@web01 /server/scripts]# expect read_test.exp 
spawn /bin/sh read_test.sh
please input your username:test
please input your password:123456
please input your e-mail:349925756@qq.com
your username is test your password is 123456your email is 349925756@qq.com
```

send 参数

-i 指定spawn_id 用来向不同spawn_id进程发送信号，多进程控制参数

-s slowly 控制发送速度，使用的时候与expect中的send_slow关联

exp_continue 动作命令匹配多个字符串之间，结尾不需要

send_user  打印expect信息。类似echo -e 功能

exit 退出

```bash
[root@web01 /server/scripts]# expect send_user_exit.exp 
i am  oldboy
i am a linuxer	 my blog is xxx.org
googd bye!
[root@web01 /server/scripts]# cat send_user_exit.exp 
#!/usr/bin/expect
send_user "i am  oldboy\n"
send_user "i am a linuxer\t"
send_user "my blog is xxx.org\n"
exit -onexit {
	 send_user "googd bye!\n"
}
expect eof
```

#### expect 程序变量

set 变量名 变量值

```bash
[root@web01 /server/scripts]# cat set_.exp 
#!/usr/bin/expect
set password "123456"
puts $password
send_user "$password\n"
[root@web01 /server/scripts]# expect set_.exp 
123456
123456
```

expect $argv 表示参数数组 可以使用[index $argv n] 接收expect传参，n从0开始

```bash
[root@web01 /server/scripts]# expect argv_.exp nginx.log 192.168.1.197 /tmp
nginx.log	 192.168.1.197	 /tmp
nginx.log	 192.168.1.197	 /tmp

[root@web01 /server/scripts]# cat argv_.exp
#!/usr/bin/expect
#defline var
set file [lindex $argv 0]
set host [lindex $argv 1]
set dir  [lindex $argv 2]
send_user "$file\t$host\t$dir\n"
puts "$file\t$host\t$dir\n"
```

$argc 传参个数

$argv0 脚本名字

#### expect if语句

1

if {条件表达式}{

 cmd

}

2

if {条件表达式}{

cmd

}

else{

cmd

}

set timeout xx 设置超时时间。单位默认秒

0 立即超时 -1 永不超时

shell中expect xxx.exp 传参1 shell变量“$cmd” 双引号接受传参的命令。

ssh慢/etc/ssh/sshd_confif

GSSAPIAuthentication no

UseDNS no 

# 企业常用运维案例

把当前目录下的文件移动到当前文件夹下的src目录

```bash
[root@web01 /server/scripts]# ls
1_100.sh  args.sh.bak   dbtest.php        macth.sh         passfile       select_test.sh      test.exp
60.sh     argv_.exp     duibi.sh          menu_test.sh     read_test.exp  send_user_exit.exp  url_check.sh
9_9.sh    case_test.sh  file_test.sh      nginx_create.sh  read_test.sh   set_.exp            useradd.sh
args.sh   cat_conf.sh   function_test.sh  nohup.out        scr            smu.sh              yunsaun.sh
[root@web01 /server/scripts]# for i in `ls /server/scripts`; do [ -f $i ] && mv $i /server/scripts/scr/; done
[root@web01 /server/scripts]# ls
scr
[root@web01 /server/scripts]# ls scr/
1_100.sh  args.sh.bak   dbtest.php        macth.sh         passfile        send_user_exit.exp  url_check.sh
60.sh     argv_.exp     duibi.sh          menu_test.sh     read_test.exp   set_.exp            useradd.sh
9_9.sh    case_test.sh  file_test.sh      nginx_create.sh  read_test.sh    smu.sh              yunsaun.sh
args.sh   cat_conf.sh   function_test.sh  nohup.out        select_test.sh  test.exp

```

#### 批量生成随机字符文件名

使用for循环在/oldboy目录下批量创建10个html文件，其中每个文件需要包含10个随机小写字母加固定字符串oldboy名称：示例：

abcdsasdf_oldboy.html

分析：首先创建文件并不难for touch 即可

   随机小写字母，可以实现随机数的方法很多，一起10个，date +%s%N 就不适用了。

openssl rand -base64 40生成40个，但是生成的包含特殊符号和数字利用sed ‘s/[^a-z]//g’匹配a-z，提取出来就是10个随机小写。

   echo $RANDOM 也可以就是转换之后需要把数字替换，tr -s “需要替换的数字” “替换完成的字母”

```bash
[root@web01 /server/scripts]# ls /test/
alloakhoat_oldboy.html  fxntmlzzqr_oldboy.html  tojlydtbii_oldboy.html  ybjovxlbnd_oldboy.html
depxygtjsc_oldboy.html  qstizwckmx_oldboy.html  uwieszoyzx_oldboy.html  yyekoutoeu_oldboy.html
egjlgplxeh_oldboy.html  suedbbwnwi_oldboy.html  wcfagnydju_oldboy.html
[root@web01 /server/scripts]# cat 1_rangdom_number.sh 
#!/bin/bash
path=/test
[  -d $path ] || mkdir -p $path     #判断目录是否存在，如果不存在就创建
for ((i=0;i<=10;i++))   
do
   first_name=$(openssl rand -base64 40|sed 's/[^a-z]//g'|cut -c 1-10)   #生成随机数 1
   #name= echo "$RANDOM"|md5sum|tr -s "[0-9]" "[a-j]"|cut -c 1-10     2
   
   touch $path/${first_name}_oldboy.html
done
```

上面for 还可以写成其他for n in {1..10} 或者 for n in `seq 10`

上面openssl rand -base64 40 改成其他的方法要复杂一些，其他生成随机数方法有缺陷。

#### 批量改名

把上面示例中的oldboy改成oldgirl并且将扩展名html改成大写

分析：

```BASH
*oldboy.html   --*olgirl.HTML *号内容一样
```

```bash
[root@web01 /server/scripts]# sh 2_rename.sh 
[root@web01 /server/scripts]# ls /test/
achgchciec_oldgirlHTML  dajfdfdabc_oldgirlHTML  efdcijdbdh_oldgirlHTML  geibcbcdfg_oldgirlHTML
bghfcbgbfj_oldgirlHTML  eafcjadgba_oldgirlHTML  eieadfgcae_oldgirlHTML
cbdfafdedc_oldgirlHTML  edajchgbdi_oldgirlHTML  eijgdahghb_oldgirlHTML
[root@web01 /server/scripts]# cat 2_rename.sh 
#!/bin/bash
path="/test/"
cd $path	 
for n in *oldboy.html        #共同点
do
mv "$n" "${n/oldboy.html/oldgirlHTML}"
#rename oldboy.html oldgirl.HTML *.html   
#man rename  rename .htm .html *.htm
done
```



#### 批量创建特殊要求用户

批量创建10个系统账号，密码为随机数（完美版）

```bash
[root@web01 /server/scripts]# cat scr/useradd.sh 
#!/bin/bash

. /etc/init.d/functions

function usage(){
           echo "USAGE: $0  sh useradd.sh name:"
           exit 1
          }
user="$1"
passfile="/tmp/user_passwd.log"

for  i in {1..5}
do
  mypasswd="`echo "test $RANDOM" |md5sum|cut -c 3-12`"
  useradd $user$i &>/dev/null &&\
  echo "$mypasswd"|passwd --stdin $user$i &>/dev/null &&
  echo -e "user:$user$i\tpassword:$mypasswd">>$passfile
  if [ $? -eq 0 ];then
      action "$user$i create success" /bin/true
  else
      action "$user$i create fail " /bin/false
  fi
done             
```

#### 扫描网络内存活主机

```bash
[root@web01 /server/scripts]# cat 4_scanip.sh 
#!/bin/bash
. /etc/init.d/functions
Cmd="ping -W 2 -c 2"
host="192.168.1."
for i in `seq 254`
do 
   {
   $Cmd $host$i &>/dev/null
   if [ $? -eq 0 ];then
       action "$host$i " /bin/true
   fi
   }&
done
```

```bash
[root@web01 /server/scripts]# sh 4_scanip_2.sh 
192.168.1.1
192.168.1.2
192.168.1.5
192.168.1.8
(192.168.1.197)
[root@web01 /server/scripts]# cat 4_scanip_2.sh 
#!/bin/bash
CMD="nmap -sP"
Ip="192.168.1.0/24"
CMD2="nmap -sS"
$CMD $Ip|awk '/Nmap scan report for/{print $NF}'
```

#### 解决DOS攻击

根据web日志或者系统网络连接数，监控某个Ip的并发连接数，若短时PV达到100，即调用防火墙命令封掉对应IP地址。iptables -I INPUT -s IP 1.1.1.1 -j DROP

```bash
[root@web01 /server/scripts]# cat 3_dos.sh 
#!/bin/bash

file=$1
while true
do
    awk '{print $1}' $1|grep -v "^$"|sort|uniq -c >/tmp/tmp.log
	 exec </tmp/tmp.log
	 while read line
	 do 
	     ip=`echo $line|awk '{print $2}'`
	 	 count=`echo $line|awk '{print $1}'`
	 	 if [ $count -gt 500 ] && [ `iptables -L -n|grep "$ip"|wc -l` -lt 1 ]
	 	     then
	 	 	     iptables -I INPUT -s $ip -j DROP
	 	 	 	 echo "$line is dropped" >>/tmp/droplist_$(date +%F).log
	 	 fi
	 done
	 sleep 3600
done
[root@web01 /server/scripts]# sh 3_dos.sh /var/log/nginx/access.log-20210325
```

![image-20210819001653599](./linux运维10 shell.assets\image-20210819001653599.png)

#### 智能关机

```bash
[root@web01 ~]# cat /server/scripts/p_.sh 
#!/bin/bash
#----------------------------------------------
# Author        : 349925756
# Email         : 349925756@qq.com
# Last modified : 2021-04-04 20:44
# Filename      : ping_.sh
# Description   : 
# Version       : 1.1 
#----------------------------------------------
 
#Notes: 当发现目标IP不能ping通300秒之后执行关机

#   `ps -ef|grep 'ping'|grep -v grep|awk '{print $2}'|xargs kill`


while true
do
    ping -c 3 192.168.1.196 &>/dev/null
    if [ $? -ne 0 ];then
	     ping -c 10 192.168.1.196 &>/dev/null
	 	 if [ $? -eq 0 ];then
	 	     continue
	 	 else
	 	    poweroff
	 	    break
	     fi
   fi
done
```

注意后台一直在运行需要kill 

#### 分模块方式

```bash
[root@web01 /server/scripts]# cat 3_dos_2.sh 
#!/bin/bash
file=$1
JudgeExt(){
    if expr "$1" : "*\.log" &>/dev/null
        then
	 	 	 :
	 else
	 	 echo $"usage:$0 xxx.log"
	 	 exit1
    fi     
          }

IpUount(){
	 grep "ESTABLISHED" $1|awk -F "[ :]+" '{ ++S[$(NF-3)]}END {for(key in S)print S[key],key}'|sort -rn -k1|head -5 >/tmp/tmp.log
         }
ipt(){
    local ip=$1
	 if [ `iptables -L -n|grep "$ip"|wc -l` -lt 1 ]
	 	 then
	 	 	 iptables -I INPUT -s $ip -j DROP
	 	 	 echo "$line is dropped" >>/tmp/droplist_$(date +%F).log
    fi
    }
main(){
    JudgeExt $file
	 while true
	 do
	 	 IpCount $file
	 	 	 while read line
	 	 	 do
	 	 	 	 ip=`echo $line|awk '{print $2}'`
	 	 	 	 count=`echo $line|awk '{print $1}'`
	 	 	 	 if [ $count -gt 3 ]
	 	 	 	 	 then
	 	 	 	 	 	 ipt $ip
	 	 	 	 fi
	 	 	 done</tmp/tmp.log
	 	 	 sleep 180
	 	 done
      }
main

```

#### mysql数据库备分库备份

```bash
[root@web01 /server/scripts]# cat 5_mysql_backup.sh 
#!/bin/bash
#定义mysql命令所在路径
PATH=“"/application/mysql/bin:$PATH"
DBPATH=/server/backup
MYUSER=root
MYPASS=123456
SOCKET=/data/3306/mysql.sock
MYCMD="mysql -u$MYUSER -p$MYPASS -S $SCOKET"
MYDUMP="mysqldump -u$MYUSER -p$MYPASS -S $SCOKET"
[ ! -d $DBPATH ] && mkdir -p $DBPATH
for dbname in `$MYCMD -e "show databases;"|sed '1,2d'|egrep -v "*mysqlschema"`
do
    $MYDUMP $dbname|gzip >$DBPATH/${dbname}_$(date +%F).sql.gz
done
```

#### mysql分库分表备份

```bash
[root@web01 /server/scripts]# cat 6_mysql_.sh 
#!/bin/bash
#定义mysql命令所在路径
PATH=“"/application/mysql/bin:$PATH"
DBPATH=/server/backup
MYUSER=root
MYPASS=123456
SOCKET=/data/3306/mysql.sock
MYCMD="mysql -u$MYUSER -p$MYPASS -S $SCOKET"
MYDUMP="mysqldump -u$MYUSER -p$MYPASS -S $SCOKET"
[ ! -d $DBPATH ] && mkdir -p $DBPATH
for dbname in `$MYCMD -e "show databases;"|sed '1,2d'|egrep -v "*mysqlschema"`
do
    mkdir $DBPATH/${dbname}_$(date +%F) -p
	 for table in `$MYCMD -E "show table from $dbname;"|sed '1d'`
	 do
	     $MYDUMP $dbname $table|gzip >$DBPATH/${dbname}_$(date +%F)/${dbname}_${table}.sql.gz
    done  
done
```

#### 筛选符合长度的字符

打印数组中小于6个字符的单词

```bash
[root@web01 /server/scripts]# cat 7_choes.sh 
#!/bin/bash
array=(I am oldboy teacher welcome to here)
for ((i=0;i<${#array[*] };i++ ))
do 
    if [ ${#array[$i] } -lt 6 ];then
	     echo "${array[$i] }"
	 fi
done
echo "----------------------------------"

for n in ${array[*]}
do
    if [ `expr length $n` -lt 6 ];then
	     echo "$n"
	 fi
done
```

```bash
[root@web01 /server/scripts]# chart="I am oldboy teacher welcome to here"
[root@web01 /server/scripts]# echo $chart|awk '{for(i=1;i<=NF;i++) if(length($i)<=6) print $i}'
I
am
oldboy
to
here
```

#### 猜数字游戏

```bash
[root@web01 /server/scripts]# cat guest_num.sh 
#!/bin/bash

num=$((RANDOM%100+1))
while true
do
read -p "please input numbers:" a
expr $a+1 &>/dev/null
#[ $? -ne 0 ] || echo "usaoe:$0 numbers:"
   if (($a>$num));then 
       echo "big"
   elif (($a<$num));then
       echo "small"
   else 
       echo "you are right"
	    echo "$num"
	    exit 0
   fi
done
[root@web01 /server/scripts]# sh guest_num.sh 
please input numbers:45
big
please input numbers:40
big
please input numbers:22
big
please input numbers:11
small
please input numbers:17
big
please input numbers:15
big
please input numbers:13
small
please input numbers:14
you are right
14
```

#### mysql主从复制异常监控

```bash
[root@web01 /server/scripts]# cat 8_mysqljinakong.sh 
#!/bin/bash

path=/server/scripts
MIAL_GROUP="111@qq.com"
PAGER_GROUP="11111111111 138888888888"
LOG_FILE="/tmp/web_check.log"
USER=root
PASSWORD=123456
PORT=3306
MYSQLCMD="mysql -u$USER -p$PASSWORD -S /data/$PORT/mysql.scok"

error=(1008 1007 1062)
RETVAL=0
[ ! -d "$path" ] && mkdir -p $path
function JudgeError(){
      for((i=0;i<${#error[*]};i++))
	   do 
	 	   if [ $1 == ${error[$i]} ];then
	 	 	   echo "MYSQL slave errorno is $1,auto repairing it"
	 	 	   $MYSQLCMD -e "stop slave;set global sql_slave_skip_counter=1;start slave;"
	 	   fi
      done
	   return $1
         }

function CheckDb(){
    status=($(awk -F ':' '/_Running|Last_Error|_Behind/{print $NF}' slave.log))
	 expr ${status[3]} + 1 &>/dev/null
	 if [ $? -ne 0 ];then
	 	 status[3]=300
    fi

	 if [ "${status[0]}" == "Yes" -a "${status[1]}" == "Yes" -a ${status[3]} -lt 120 ]
	 	 then
	 	 	 return 0
	 else
	 	 JudgeError ${status[2]}
    fi
         }


function MAIL(){
    local SUBJECT_CONTENT=$1
	 for MAIL_USER in `echo $MAIL_GROUP`
	 do
	 	 mail -s "$SUBJECT_CONTENT" $MAIL_USER <$LOG_FILE
	 done
          }
#手机发送短信，参数来自供应商
function PAGER(){
    for PAGER_USER in `echo $PAGER_GROUP`
	 do
	 	 TITLE=$1
	 	 CONTACT=$PAGER_USER
	 	 HTTPGW=http://oldboy.sms.cn/smsproxy/sendsms.action
	 	 curl -d cdkey=xxxx-xxx -d password=OLDBOY -d phone=$CONTACT -d message="$TITLE[$2]" $HTTPGW
	 done
         }

function SendMsg(){
    if [ $1 -ne 0 ];then
	 	 RETVAL=1
	 	 NOW_TIME=`date +%Y-%m-%d %H:%M:%S`
	 	 SUBJECT_CONTENT="mysql slave is error ,errorno is $2,${NOW_TIME}."
	 	 echo -e "$SUBJECT_CONTENT"|tee $LOG_FILE
	 	 MAIL $SUBJECT_CONTENT
	 	 PAGER $SUBJECT_CONTENT $NOW_TIME
	 else
	 	 echo "Mysql slave status is ok"
	 	 return RETVAL
         }

function main(){
    while true
	 do
	 	 CheckDb
	 	 SendMsg $?
        sleep 30
	 done
         }
main
```

#### 比较整数大小

方法很多

```bash
[root@web01 /server/scripts]# cat 9_int_.sh 
#!/bin/bash

read -p "please input two int numbers:"  a  b
[ -z "$a" ] || [ -z "$b" ]&&{ 
    echo "please input two numbers"
    exit 1
   }
#[ $# -ne 2 ] && {
#    echo "two numbers"
#	 exit 2
#    }
expr $a + 1 &>/dev/null
RETVAL1=$?
expr $b + 1 &>/dev/null
RETVAL2=$?

test $RETVAL1 -eq 0 -a $RETVAL2 -eq 0 || {
    echo "please input int"
	 exit 2
}

[ $a -gt $b ] && {
    echo "$a > $b"
	 exit 0
 }

[ $a -lt $b ]&&{
    echo "$a < $b"
    exit 0
}

[ $a -eq $b ]&&{
    echo "$a = $b"
    exit 0
}
```

上面方法不好。可以使用if while case 等很多办法实现。

#### 菜单自动化部署软件

```bash
[root@web01 /server/scripts]# cat 10_menu_install.sh 
#!/bin/bash
path=/server/scripts
[ -d $path ] && mkdir -p $path
menu(){
cat <<END
    1.[INSTALL LNMP]
	 2.[INSTALL LAMP]
	 3.[EXIT]
    please input then num you want:
END
}
menu

read num
expr $num + 1 &>/dev/null
[ $? -ne 0 ] && {
    echo "num in [1|2|3]"
    exit 1
}

[ $num -eq 1 ]&&{
    echo "installing LNMP"
    sleep 5;
	 [ -x "$path/lnmp.sh" ]&&{
	     echo "$path/lnmp.sh does not exit or can not be exec." 
	     exit 2
	 }
    source $path/lnmp.sh
	 exit $?
}

[ $num -eq 2 ]&&{
    echo "installing LAMP"
    sleep 5;
	 [ -x "$path/lnmp.sh" ]&&{
	     echo "$path/lnmp.sh does not exit or can not be exec." 
	     exit 2
	 }
    source $path/lamp.sh
	 exit $?
}

[ $num -eq 3 ]&&{
    echo bye
    exit 3
}

[[ ! $num =~ [1-3] ]] && {
    echo "error"
    exit 4
}
```

#### web及mysql服务异常监控

方法很多

首先端口方法

```bash
[root@web01 /server/scripts]# netstat -lntup|grep 3306
tcp        0      0 0.0.0.0:3306            0.0.0.0:*               LISTEN      1044/mysqld         
[root@web01 /server/scripts]# netstat -lntup|grep 3306|wc -l
1
[root@web01 /server/scripts]# netstat -lntup|grep mysql
tcp        0      0 0.0.0.0:3306            0.0.0.0:*               LISTEN      1044/mysqld         
[root@web01 /server/scripts]# netstat -lntup|grep mysql|wc -l
1
[root@web01 /server/scripts]# ss -lntup|grep 3306
tcp    LISTEN     0      50        *:3306                  *:*                   users:(("mysqld",pid=1044,fd=14))
[root@web01 /server/scripts]# ss -lntup|grep 3306|wc -l
1
[root@web01 /server/scripts]# ss -lntup|grep mysql
tcp    LISTEN     0      50        *:3306                  *:*                   users:(("mysqld",pid=1044,fd=14))
[root@web01 /server/scripts]# ss -lntup|grep mysql|wc -l
1
[root@web01 /server/scripts]# lsof -i tcp:3306
COMMAND  PID  USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
mysqld  1044 mysql   14u  IPv4  20572      0t0  TCP *:mysql (LISTEN)
[root@web01 /server/scripts]# lsof -i tcp:3306|wc -l
2

[root@web01 /server/scripts]# nmap 127.0.0.1

Starting Nmap 6.40 ( http://nmap.org ) at 2021-04-08 14:45 CST
Nmap scan report for localhost (127.0.0.1)
Host is up (0.000011s latency).
Not shown: 994 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
25/tcp   open  smtp
80/tcp   open  http
808/tcp  open  ccproxy-http
3306/tcp open  mysql
9000/tcp open  cslistener

Nmap done: 1 IP address (1 host up) scanned in 1.96 seconds
[root@web01 /server/scripts]# nmap 127.0.0.1 -p 3306

Starting Nmap 6.40 ( http://nmap.org ) at 2021-04-08 14:46 CST
Nmap scan report for localhost (127.0.0.1)
Host is up (0.000044s latency).
PORT     STATE SERVICE
3306/tcp open  mysql

Nmap done: 1 IP address (1 host up) scanned in 0.24 seconds
[root@web01 /server/scripts]# nmap 127.0.0.1 -p 3306|grep open
3306/tcp open  mysql
[root@web01 /server/scripts]# nmap 127.0.0.1 -p 3306|grep open|wc -l
1

[root@web01 /server/scripts]# yum install telnet nc -y
[root@web01 /server/scripts]# telnet 127.0.0.1 3306
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
R
5.5.68-MariaDBzaXt^[/ }9Azfo{=\:H?mysql_native_passwordConnection closed by foreign host.
[root@web01 /server/scripts]# echo -e "\n"|telnet 127.0.0.1 3306 2>/dev/null|grep Connected|wc -l
1
[root@web01 /server/scripts]# nc -w 2 127.0.0.1 3306 &>/dev/null

以上是服务器检测
[root@web01 /server/scripts]# wget --spider --timeout=10 --tries=2 www.baidu.com &>/dev/null|echo $?
0
[root@web01 /server/scripts]# wget -T 10 -q --spider http://www.baidu.com &>/dev/null|echo $?
0
[root@web01 /server/scripts]# curl -s -o /dev/null www.baidu.com|echo $?
0
数据库连接测试
[root@web01 /server/scripts]# cat /html/www/index.php 
<?php
 $servername = "localhost";
 $username = "root";
 $password = "123456";
 //$link_id=mysql_connect('主机名','用户','密码');
 //mysql -u用户 -p密码 -h 主机
 $conn = mysqli_connect($servername, $username, $password);
 if ($conn) {
       echo "mysql successful by root ! \n";
       echo " ";
       echo "数据库连接成功 !\n";
    }else{
       die("Connection failed: " . mysqli_connect_error());
    }
?>

利用wc -l 把查询到的转换为数组做比较。当结果大于等于1 ，$? 返回值0则表示执行正常，非0则说明不成功
[root@web01 /server/scripts]# sh 11_ser_run.sh 
mysql nginx  running
[root@web01 /server/scripts]# systemctl stop mariadb
[root@web01 /server/scripts]# sh 11_ser_run.sh 
mysql nginx  running
[root@web01 /server/scripts]# sh 11_ser_run.sh 
mysql nginx  running
[root@web01 /server/scripts]# systemctl stop nginx
[root@web01 /server/scripts]# sh 11_ser_run.sh 
mysql not running
[root@web01 /server/scripts]# cat 11_ser_run.sh 
#!/bin/bash


if [ `netstat -lntup|egrep '3306|80'|wc -l` -ge 2 ];then
    echo "mysql nginx  running"
else
    echo "mysql not running"
	 systemctl start mariadb
	 systemctl start nginx
fi
后期再改进

```

#### 监控memcached缓存服务

```bash
[root@web01 ~]# yum install memcached
[root@web01 ~]# systemctl start memcached
[root@web01 ~]# netstat -lntup|grep memcached
tcp        0      0 0.0.0.0:11211           0.0.0.0:*               LISTEN      2788/memcached      
tcp6       0      0 :::11211                :::*                    LISTEN      2788/memcached      
udp        0      0 0.0.0.0:11211           0.0.0.0:*                           2788/memcached      
udp6       0      0 :::11211                :::*                                2788/memcached    
[root@web01 /server/scripts]# cat 12_memcached.sh 
#!/bin/bash

if [ `netstat -lntup|grep memcached|wc -l` -lt 1 ];then
    echo "Memcached service is error"
	 exit 1
fi

printf "del key\r \n"|nc 127.0.0.1 11211 &>/dev/null
printf "set key 0 0 10 \r \noldboy1234\r \n"|nc 127.0.0.1 11211 &>/dev/null

McValues=`printf "get key \r \n"|nc 127.0.0.1 11211|grep oldboy1234|wc -l`

if [ $McValues -eq 1 ];then
    echo "Mencached status is ok"
else
    echo "Memcached status is down"
fi
[root@web01 /server/scripts]# sh 12_memcached.sh 
Memcached status is down
```

#### 入侵检测与报警

监控www目录下所有文件是否被篡改。

