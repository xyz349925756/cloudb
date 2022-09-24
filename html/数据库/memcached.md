

# 概念

mecached是什么？

memcached是分布式内存对象缓存系统，通过减轻数据库负载来加速web应用程序。memcached提供了API供大多数流行语言使用

memcached主要是用于web应用和数据库



memcached组成：

- 客户端软件，它给出了可用的 memcached 服务器的列表。

- 基于客户端的哈希算法，它根据“密钥”选择服务器。

- 服务器软件，将值及其密钥存储到内部哈希表中。

- LRU，用于确定何时抛出旧数据（如果内存不足）或重用内存。

  

## install

官网：https://www.memcached.org/downloads

### yum|apt

```sh
# 最简单的安装
[root@memcached ~]# yum list|grep memcached
libmemcached.i686                           1.0.16-5.el7               base     
libmemcached.x86_64                         1.0.16-5.el7               base     
libmemcached-devel.i686                     1.0.16-5.el7               base     
libmemcached-devel.x86_64                   1.0.16-5.el7               base     
memcached.x86_64                            1.4.15-10.el7_3.1          base     
memcached-devel.i686                        1.4.15-10.el7_3.1          base     
memcached-devel.x86_64                      1.4.15-10.el7_3.1          base     
python-memcached.noarch                     1.48-4.el7                 base     
[root@memcached ~]# yum search memcached
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.aliyun.com
 * extras: mirrors.aliyun.com
 * updates: mirrors.aliyun.com
============================================================ N/S matched: memcached ============================================================
libmemcached.i686 : Client library and command line tools for memcached server
libmemcached.x86_64 : Client library and command line tools for memcached server
libmemcached-devel.i686 : Header files and development libraries for libmemcached
libmemcached-devel.x86_64 : Header files and development libraries for libmemcached
memcached-devel.i686 : Files needed for development using memcached protocol
memcached-devel.x86_64 : Files needed for development using memcached protocol
python-memcached.noarch : A Python memcached client library
memcached.x86_64 : High Performance, Distributed Memory Object Cache
pcp-pmda-memcache.x86_64 : Performance Co-Pilot (PCP) metrics for Memcached
php-pecl-memcache.x86_64 : Extension to work with the Memcached caching daemon

  Name and summary matches only, use "search all" for everything.
  
[root@memcached ~]# yum install -y memcached   # apt install -y memcached
```

memcached 依赖libevent

```sh
# 如果没有
yum install libevent-devel
```

### 源代码安装

```sh
# https://www.memcached.org/downloads
# https://libevent.org/
# 下载最新的包
[root@memcached ~]# ls
anaconda-ks.cfg  memcached-1.6.15.tar.gz

[root@memcached ~]# tar -zxf memcached-1.6.15.tar.gz 
[root@memcached ~]# cd memcached-1.6.15/
[root@memcached ~/memcached-1.6.15]# yum install -y gcc gcc-c++ libevent libevent-devel nc
[root@memcached ~/memcached-1.6.15]# ./configure --prefix=/usr/local/memcached
[root@memcached ~/memcached-1.6.15]# make && make install
[root@memcached ~/memcached-1.6.15]# memcached -m 10 -u root &  #后台启动
```



启动

```sh
[root@memcached ~]# systemctl start memcached
[root@memcached ~]# systemctl enable memcached
[root@memcached ~]# netstat -lntup
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:11211           0.0.0.0:*               LISTEN      1965/memcached      
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      930/sshd            
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      1162/master         
tcp6       0      0 :::11211                :::*                    LISTEN      1965/memcached      
tcp6       0      0 :::22                   :::*                    LISTEN      930/sshd            
tcp6       0      0 ::1:25                  :::*                    LISTEN      1162/master         
udp        0      0 0.0.0.0:11211           0.0.0.0:*                           1965/memcached      
udp6       0      0 :::11211                :::*                                1965/memcached     
```



memcached 主要是硬件就是内存。



## 配置

memcached命令及参数

```sh
[root@memcached ~]# memcached -h
memcached 1.4.15
-p <num>      服务端口 (default: 11211)
-U <num>      UDP port number to listen on (default: 11211, 0 is off)
-s <file>     UNIX socket path to listen on (disables network support)
-a <mask>     access mask for UNIX socket, in octal (default: 0700)
-l <addr>     指定监听的服务器IP地址，可以不设置
-d            守护进程运行
-r            maximize core file limit
-u <username> 指定运行memcached的用户，如果当前用户是root就需要指定用户
-m <num>      指定服务可以缓存数据的最大内存 (default: 64 MB)
-M            内存不够时禁用LRU，如果内存满了会报错
-c <num>      最大并发连接数 (default: 1024)
-k            lock down all paged memory.  Note that there is a
              limit on how much memory you may lock.  Trying to
              allocate more than that would fail, so be sure you
              set the limit correctly for the user you started
              the daemon with (not for -u <username> user;
              under sh this is done with 'ulimit -S -l NUM_KB').
-v            调试，打印较少的信息错误和等待信息
-vv           very verbose (also print client commands/reponses)
-vvv          extremely verbose (also print internal state transitions)
-h            print this help and exit
-i            print memcached and libevent license
-P <file>     设置保存memcached的PID
-f <factor>   chunk size增长因子 (default: 1.25)
-n <bytes>    为key+value+flags分配最小内存空间 (default: 48字节)
-L            启用最大内存页，可以降低内存浪费，改进性能。
-D <char>     Use <char> as the delimiter between key prefixes and IDs.
              This is used for per-prefix stats reporting. The default is
              ":" (colon). If this option is specified, stats collection
              is turned on automatically; if not, then it may be turned on
              by sending the "stats detail on" command to the server.
-t <num>      线程数 (default: 4)
-R            每个事件最大请求 (default: 20)
-C            禁用CAS，禁止版本计数，减少版本开销
-b <num>      Set the backlog queue limit (default: 1024)
-B            Binding protocol - one of ascii, binary, or auto (default)
-I            Override the size of each slab page. Adjusts max item size
              (default: 1mb, min: 1k, max: 128m)
-S            Turn on Sasl authentication
-o            Comma separated list of extended or experimental options
              - (EXPERIMENTAL) maxconns_fast: immediately close new
                connections if over maxconns limit
              - hashpower: An integer multiplier for how large the hash
                table should be. Can be grown at runtime if not big enough.
                Set this based on "STAT hash_power_level" before a 
                restart.

```

配置文件

```sh
[root@memcached ~]# cat /etc/sysconfig/memcached 
PORT="11211"
USER="memcached"
MAXCONN="1024"
CACHESIZE="64"
OPTIONS=""
```

多个实例可以指定端口

```sh
memcached -p 11212
```

默认情况下，最大并发连接数设置为 1024。正确配置这一点非常重要。与 memcached 的额外连接可能会在等待插槽释放时挂起。您可以通过发出命令并查看“listen_disabled_num”来检测您的实例是否已用完连接。该值应为零或接近零。statsMemcached可以非常简单地扩展大量连接。每个连接的内存开销量很低（如果连接处于空闲状态，则甚至更低），因此不要担心将其设置得很高。假设您有 5 个 Web 服务器，每个服务器都运行 apache。每个 apache 进程的 MaxClients 设置为 12。这意味着您可以接收的最大并发连接数为 5 x 12 （60）。如果可以的话，请始终留出一些额外的插槽，用于管理任务，添加更多Web服务器，crons/脚本/等。

### 检查正在运行的配置

```sh
[root@memcached ~]# echo "stats settings" | nc localhost 11211
STAT maxbytes 67108864
STAT maxconns 1024
STAT tcpport 11211
STAT udpport 11211
STAT inter NULL
STAT verbosity 0
STAT oldest 0
STAT evictions on
STAT domain_socket NULL
STAT umask 700
STAT growth_factor 1.25
STAT chunk_size 48
STAT num_threads 4
STAT num_threads_per_udp 4
STAT stat_key_prefix :
STAT detail_enabled no
STAT reqs_per_event 20
STAT cas_enabled yes
STAT tcp_backlog 1024
STAT binding_protocol auto-negotiate
STAT auth_enabled_sasl no
STAT item_size_max 1048576
STAT maxconns_fast no
STAT hashpower_init 0
STAT slab_reassign no
STAT slab_automove 0
END
```

如果多个服务器，每个服务器运行了一个memcached那么memcached就要设置为a，b，c这样才行。

如果您有三个 Web 服务器，并且每个 Web 服务器也运行一个 memcached 实例，则可能会尝试将“本地”实例命名为“localhost”。这将*不会*按预期工作，因为服务器列表现在在 Web 服务器之间是不同的。这意味着 Web 服务器 1 的映射密钥与服务器 2 不同，从而导致用户和业务开发人员的大规模歇斯底里。排序也很重要。某些客户端会对您提供给它们的服务器列表进行排序，但其他客户端不会。如果您有服务器“A，B，C”，请将它们列为“A，B，C”。

mysql与memcached对比

| mysql  | memcached |
| :----: | :-------: |
| insert |    set    |
| select |    get    |
| delete |  delete   |







### poxy

```sh
./configure --enable-proxy  # 启用poxy
make
make test
```

### 分布式集群

分布式数据并不一样，每个节点独立存储

下面是分布式的简单代码：

```sh
"memcached_servers" => array（
  '10.0.0.1:11211',
  '10.0.0.2:11211',
  '10.0.0.3:11211',
）
```

tengine反向代理

```sh
http{
  upstream test{
     consistent_hash $request_uri;
     server 127.0.0.1:11211 id=1001 weight=3;
     server 127.0.0.1:11211 id=1001 weight=10;
     server 127.0.0.1:11211 id=1001 weight=20;
  }
}
```

http://tengine.taobao.org/  淘宝开源的nginx分支

