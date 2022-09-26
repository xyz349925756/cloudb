elastic :https://www.elastic.co/cn/

全文检索

下载：https://www.elastic.co/cn/downloads/elasticsearch



启动超时把服务启动转后台运行：内存4G起

```BASH
ExecStart=/bin/bash -c /usr/share/elasticsearch/bin/systemd-entrypoint -p ${PID_DIR}/elasticsearch.pid &    
```





# Elasticserach

## 什么是全文检索和Lucene?

全文数据库是全文检索系统的主要构成部分。所谓全文数据库是将一个完整的信息源的全部内容转化为计算机可以识别、处理的信息单元而形成的数据集合。全文数据库不仅存储了信息，而且还有对全文数据进行词、字、段落等更深层次的编辑、加工的功能，而且所有全文数据库无一不是海量信息数据库。

Elasticsearch 是一个分布式、可扩展、实时的搜索与数据分析引擎，通过它我们可以构建出一个强大的全文搜索系统，解决诸如文章检索慢，商品检索慢、MySQL的like查询慢这样的问题。

Elasticsearch是基于hadoop创始人道哥的另一杰作Lucene实现的，速度非常快，核心是使用了倒排索引这样的结构。

lucene,就是一个jar包，里面包含了封装好的各种建立倒排索引，以及进行搜索的代码，包含各种算法，我们就用java开发的时候，引入lucene jar，然后基于lucene的api去进行开发就可以了，我们就可以将已有的数据数据建立索引，lucene会在本地磁盘上面，给我们组织索引的数据结构。另外的话，我们也可以用lucene提供的的功能和api来针对磁盘上的索引数据，进行搜索。



## Elasticsearch 适用场景

### Elasticsearch的功能

（1）分布式搜索和分析引擎

（2）全文检索，结构化检索，数据分析

（3）对海量数据进行近实时的处理

分布式：ES自动可以将海量数据分散到多台服务器上存储和检索

海量数据的处理：分布式以后，就可以采用大量的服务器去存储和检索数据，自然而然就可以实现海量数据的处理了

跟分布式/海量数据相反的，lucene,单机应用，只能在单台服务器上使用，最多只能处理单台服务器可以处理的数据量。

近实时：在秒级别对数据进行搜索和分析

### Elasticsearch的适用场景

（1）维基百科，全文检索，高亮，搜索推荐

（2）新闻网站，用户日志+社交网络数据，分析

（3）Stack Overflow（国外的程序异常讨论论坛），全文检索，搜索相关问题和答案

（4）GitHub(开源代码管理)，搜索上亿行代码

（5）电商网站，检索商品

（6）日志数据分析，logstash采集日志，ES进行复杂的数据分析

（7）商品价格监控网站，用户设定某商品的价格阈值，当低于该阈值的时候，发视消息给用户

（8）BI系统，ES执行数据分析和挖掘，Kibana进行数据可视化

（9）国内，站内搜索(电商，招聘，门户)

（10）BI 系统，商业智能，分析用户消费趋势和用户群体的组成构成。

### Elasticsearch的特点

 （1）可以作为一个大型分布式集群（数百台服务器）技术，处理PB级数据，服务大公司，也可以运行在单机上，服务小公司

（2）Elasticsearch不是什么新技术，主要是将全文检索，数据分析以及分布式技术，合并在一起，才形成了独一无二的ES，lucene(全文检索)

（3）对用户而言，是开箱即用的，非常简单，作为中小型的应用，直接3分钟部署一下ES，就可以作为生产环境的系统来使用了，数据量不大，操作不是太复杂

（4）数据库的功能面对很多领域是不够的，优势：事务，各种联机事务型的操作，特殊的功能，比如全文检索，同义词处理，相关度排名，复杂数据分析，海量数据的近实时处理，Elasticsearch作为传统数据库的一个补充，提供了数据库所不能提供的很多功能。

### Elasticsearch 发展

lucene，最先进，功能最强大的搜索库，直接基于lucene开发，非常复杂，api复杂

Elasticsearch,基于lucene，隐藏复杂性，提供简单易用的restful api接口，java api接口，还有其他语言的接口

（1）分布式的文档存储引擎

（2）分布式的搜索引擎和分析引擎

（3）分布式，支持PB级数据

开箱即用，优秀的默认参数，不需要任何额外配置，完全开源

### Elasticsearch 核心概念

（1）**Near Realtime(NRT)**: 近实时，两个意思，从写入数据到可以被搜索有一个小延迟（大概1秒）; 

（2）**Cluster**：集群，包含多个节点，每个节点属于哪个集群是通过一个配置（集群名称：默认是elasticsearch）来决定的，对于中小型应用来说，刚开始一个集群就一个节点很正常

（3）**Node**：节点，集群中的一个节点，节点也有一个名称（默认是随机分配的），节点名称很重要（在执行运维管理操作的时候），默认节点会去加入一个名称为‘elasticsearch’的集群，如果直接启动一堆节点，那么他们会自动组成一个elasticsearch集群，当然一个节点也可以组成一个elasticsaerch集群

（4）**Document**:文档，ES中最小数据单元，一个document可以是一条客户数据，一条商品分类数据，一条订单数据，通常是JSON数据结构表示，每个index下的type中，都可以去存储多个document，一个document里面有多个field,每个field就是一个数据字段。

（5）**Index**:索引 包含一堆有相似结构的文档数据，比如可以有一个客户索引，商品分类索引，订单索引，索引有一个名称，一个index包含很多document。一个index就代表了一类类似的或者相同的document。比如说建立一个product index,商品索引，里面可能存放了所有的商品数据，所有的商品document.

（6）**Type**: 类型，每个索引里都可以有一个或多个type，type是index中的一个逻辑数据分类，一个type下的document,都有相同的field，比如博客系统，有一个索引，可以定义用户数据type,博客数据type,评论数据type。商品index，里面存放了所有的商品数据，商品document但是商品分很多种类，每个种类的document的field可能不太一样，比如说电器商品，可能还包含一些诸如售后时间范围这样的特殊field:生鲜商品，还包含一些诸如生鲜保质期之类的field

（7）**shard**,单台机器无法存储大量数据，es可以将一个索引中的数据切分为多个shard，分布在多台服务器上存储，有了shard就可以横向扩展，存储更多数据，让搜索和分析等操作分不到多台服务器上去执行，提升吞吐量和性能，每个shard都是一个lucene index

（8）**replica**，任何一个服务器随时可能故障或宕机，此时shard可能就会丢失，因此可以为每个shard创建多个replica副本，replica可以在shard故障时提供备用服务，保证数据不丢失，多个replica还可以提升搜索操作的吞吐量和性能。primary shard（建立索引时一次设置，不能修改，默认5个），replica shard（随时修改数量，默认1个），默认每个索引10个shard，5个primary shard,5个replica shard，最小的高可用配置，是2台服务器



### Elasticsearch数据格式

elasticsearch 使用json作为文档的序列化格式，json序列被大多数编程语言支持，并且成为nosql领域的标准格式，简单，简洁，易读

例子：

```BASH
{
"email": "john@smith.com", "first_name": "John",
"last_name": "Smith",
"info":
     { "bio": "Eco-warrior and defender of the weak", 
     "age": 25,
     "interests": [ "dolphins", "whales" ]
     },
"join_date": "2019/05/01" 
}
```

## 安装

### rpm

官方文档：https://www.elastic.co/guide/en/elasticsearch/reference/current/install-elasticsearch.html

java 安装

```BASH
[root@redis01 /server/soft]# ls
jdk-8u301-linux-x64.rpm  redis-6.2.5.tar.gz
[root@redis01 /server/soft]# rpm -ivh jdk-8u301-linux-x64.rpm
[root@redis01 /server/soft]# java -version
java version "1.8.0_301"
Java(TM) SE Runtime Environment (build 1.8.0_301-b09)
Java HotSpot(TM) 64-Bit Server VM (build 25.301-b09, mixed mode)
```

RPM下载：https://www.elastic.co/guide/en/elasticsearch/reference/current/rpm.html

```BASH
rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
[root@redis01 /server/soft]# rpm -ivh elasticsearch-7.14.0-x86_64.rpm
```

https://www.cnblogs.com/yangjiaoshou/p/15034299.html 参考

```bash
systemctl daemon-reload
systemctl start elasticsearch
systemctl status elasticsearch
systemctl enable elasticsearch

[root@redis01 /server/soft]# ps -ef|grep elasticsearch
[root@redis01 /server/soft]# lsof -i:9200
COMMAND  PID          USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
java    2211 elasticsearch  283u  IPv6  26957  0t0  TCP localhost:wap-wsp (LISTEN)
java    2211 elasticsearch  284u  IPv6  26958  0t0  TCP localhost:wap-wsp (LISTEN)
[root@redis01 /server/soft]# netstat -lntup|grep 9200
tcp6       0      0 127.0.0.1:9200   :::*    LISTEN      2211/java
tcp6       0      0 ::1:9200         :::*    LISTEN      2211/java
```

tar.gz 部署

https://www.elastic.co/guide/en/elasticsearch/reference/current/targz.html

docker

https://www.elastic.co/guide/en/elasticsearch/reference/7.14/docker.html

测试是否安装成功

```BASH
[root@redis01 /server/soft]# curl 'http://localhost:9200/?pretty'
{
  "name" : "redis01",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "cQ1SxvOwS-yowghhBTfHgQ",
  "version" : {
    "number" : "7.14.0",
    "build_flavor" : "default",
    "build_type" : "rpm",
    "build_hash" : "dd5a0a2acaa2045ff9624f3729fc8a6f40835aa1",
    "build_date" : "2021-07-29T20:49:32.864135063Z",
    "build_snapshot" : false,
    "lucene_version" : "8.9.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

### 二进制部署

```bash
[root@redis01 ~]# find / -name "elasticsearch" |xargs rm -rf
[root@redis01 ~]# find / -name "kibana" -exec rm -rf { } \;
[root@redis01 /server/soft]# ls
elasticsearch-7.14.0-linux-x86_64.tar.gz 
[root@redis01 /opt/elasticsearch]# useradd elasticsearch
[root@redis01 /opt/elasticsearch]# passwd elasticsearch
[root@redis01 /opt/elasticsearch]# id elasticsearch
uid=1000(elasticsearch) gid=1000(elasticsearch) groups=1000(elasticsearch)
[root@redis01 ~]# mkdir /opt/elasticsearch
[root@redis01 ~]# chown -R elasticsearch.elasticsearch /opt/elasticsearch
[root@redis01 ~]# mkdir /data/elasticsearch
[root@redis01 ~]# chown -R elasticsearch.elasticsearch /data/elasticsearch
[root@redis01 ~]# mkdir /var/log/elasticsearch
[root@redis01 ~]# chown -R elasticsearch.elasticsearch /var/log/elasticsearch/

[root@redis01 ~]# su - elasticsearch
Last login: Fri Aug 13 11:16:20 CST 2021 on pts/0
[elasticsearch@redis01 /server/soft]$ tar xf elasticsearch-7.14.0-linux-x86_64.tar.gz -C /opt/elasticsearch
[elasticsearch@redis01 /server/soft]$ ln -s /opt/elasticsearch/elasticsearch-7.14.0 /opt/elasticsearch/es
[elasticsearch@redis01 /opt/elasticsearch/es]$ ls
bin  config  jdk  lib  LICENSE.txt  logs  modules  NOTICE.txt  plugins  README.asciidoc

```

java环境

```BASH
[root@redis01 /opt/elasticsearch]# rpm -qa |grep jdk
jdk1.8-1.8.0_301-fcs.x86_64
[root@redis01 /opt/elasticsearch]# rpm -e jdk1.8-1.8.0_301-fcs.x86_64
二合一
rpm -e `rpm -qa |grep jdk`

[root@redis01 /opt/elasticsearch]# ./jdk/bin/java -version
openjdk version "16.0.1" 2021-04-20
OpenJDK Runtime Environment AdoptOpenJDK-16.0.1+9 (build 16.0.1+9)
OpenJDK 64-Bit Server VM AdoptOpenJDK-16.0.1+9 (build 16.0.1+9, mixed mode, sharing)
系统自带了java环境
```

启动

```BASH
[elasticsearch@redis01 /opt/elasticsearch/es]$ ./bin/elasticsearch
[root@redis01 ~]# netstat -lntup 
查看9200 9300端口是否存在
```

这个并不是我们想要的。

修改配置文件

```BASH
[elasticsearch@redis01 /opt/elasticsearch/es]$ cp config/elasticsearch.yml{,.bak} 
[elasticsearch@redis01 /opt/elasticsearch/es]$ vim config/elasticsearch.yml
cluster.name: elasticsearch
node.name: node-1   
path.data: /data/elasticsearch 
path.logs: /var/log/elasticsearch      
bootstrap.memory_lock: true  
network.host: 172.16.0.210  
http.port: 9200   
discovery.seed_hosts: ["172.16.0.210"]
http.cors.enabled: true
http.cors.allow-origin: "*"

[root@redis01 /opt/elasticsearch]# vim config/jvm.options
-Xms512m
-Xmx512m  
  
```

错误：

```BASH
ERROR: [4] bootstrap checks failed. You must address the points described in the following [4] lines before starting Elasticsearch.
bootstrap check failure [1] of [4]: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65535]
bootstrap check failure [2] of [4]: memory locking requested for elasticsearch process but memory is not locked
bootstrap check failure [3] of [4]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
bootstrap check failure [4] of [4]: the default discovery settings are unsuitable for production use; at least one of [discovery.seed_hosts, discovery.seed_providers, cluster.initial_master_nodes] must be configured
```

```sh
[root@redis01 ~]# vim /etc/security/limits.conf 
    elasticsearch  -  nofile  65536    
```

```BASH
ERROR: [3] bootstrap checks failed. You must address the points described in the following [3] lines before starting Elasticsearch.
bootstrap check failure [1] of [3]: memory locking requested for elasticsearch process but memory is not locked
bootstrap check failure [2] of [3]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
bootstrap check failure [3] of [3]: the default discovery settings are unsuitable for production use; at least one of [discovery.seed_hosts, discovery.seed_providers, cluster.initial_master_nodes] must be configured

```

```bash
sysctl -w vm.max_map_count=262144
or
[root@redis01 ~]# vim /etc/sysctl.conf 
vm.max_map_count=262144

[root@redis01 ~]# sysctl -p
```

```BASH
[elasticsearch@redis01 /opt/elasticsearch/es]$ vim config/elasticsearch.yml
discovery.seed_hosts: ["172.16.0.210"]

```

```BASH
ERROR: [1] bootstrap checks failed. You must address the points described in the following [1] lines before starting Elasticsearch.
bootstrap check failure [1] of [1]: memory locking requested for elasticsearch process but memory is not locked
```

```BASH
[root@redis01 ~]# vim /etc/security/limits.conf 
# allow user 'elasticsearch' mlockall
elasticsearch soft memlock unlimited
elasticsearch hard memlock unlimited    
```

https://www.elastic.co/guide/en/elasticsearch/reference/7.14/setup-configuration-memory.html

bootstrap.memory_lock: false 也可以，但是这样不稳定 swap 需要关闭

所以建议开启

守护进程启动

```BASH
[elasticsearch@redis01 ~]$ /opt/elasticsearch/es/bin/elasticsearch -d -p pid
等2分钟左右
[root@redis01 ~]# netstat -lntup
查看端口
#关闭
[root@redis01 ~]# ps -ef |grep elasticsearch  #查看pid
[root@redis01 ~]# kill pid  

```

http://172.16.0.210:9200/

但是elasticsearch-head不能连接

```BASH
[root@redis01 ~]# vim /opt/elasticsearch/es/config/elasticsearch.yml
http.cors.enabled: true
http.cors.allow-origin: "*"
cluster.initial_master_nodes: ["node-1"]

```

```BASH
[elasticsearch@redis01 ~]$ /opt/elasticsearch/es/bin/elasticsearch -d -p pid
```

总结：

修改配置文件

```BASH
[root@redis01 ~]# vim /etc/security/limits.conf 
elasticsearch  -  nofile  65536  
# allow user 'elasticsearch' mlockall
elasticsearch soft memlock unlimited
elasticsearch hard memlock unlimited    

[root@redis01 ~]# vim /etc/sysctl.conf 
vm.max_map_count=262144

sysctl -p
[root@grafana ~]# vim /etc/systemd/system.conf 
DefaultLimitNOFILE=65536
DefaultLimitNPROC=32000
DefaultLimitMEMLOCK=infinity
```

![image-20210813191600433](ELK.assets\image-20210813191600433.png)

#### 二进制集群搭建

redis02

```BASH
创建用户
[root@redis02 ~]# useradd elasticsearch
[root@redis02 ~]# passwd elasticsearch

mkdir /data/elasticsearch
mkdir /var/log/elasticsearch
mkdir /opt/elasticsearch

chown -R elasticsearch.elasticsearch /data/elasticsearch
chown -R elasticsearch.elasticsearch /var/log/elasticsearch/
chown -R elasticsearch.elasticsearch /opt/elasticsearch

[root@redis02 ~]# su - elasticsearch

tar xf /server/soft/elasticsearch-7.14.0-linux-x86_64.tar.gz -C /opt/elasticsearch
ln -s /opt/elasticsearch/elasticsearch-7.14.0/ /opt/elasticsearch/es

```

修改文件

```BASH
[elasticsearch@redis02 ~]$ vim /opt/elasticsearch/es/config/elasticsearch.yml 
cluster.name: elasticsearch
node.name: node-2
path.data: /data/elasticsearch
path.logs: /var/log/elasticsearch
bootstrap.memory_lock: true
network.host: 172.16.0.211
http.port: 9200
discovery.seed_hosts: ["172.16.0.210", "172.16.0.211","172.16.0.212"]
cluster.initial_master_nodes: ["node-1", "node-2","node-3"]
http.cors.enabled: true
http.cors.allow-origin: "*"

[root@redis02 ~]# vim /etc/security/limits.conf 
elasticsearch  -  nofile  65536
# allow user 'elasticsearch' mlockall
elasticsearch soft memlock unlimited
elasticsearch hard memlock unlimited

[root@redis02 ~]# vim /etc/sysctl.conf 
vm.max_map_count=262144

[root@redis02 ~]# sysctl -p
```





redis03

```bash
[root@redis03 ~]# useradd elasticsearch
[root@redis03 ~]# passwd elasticsearch

mkdir /data/elasticsearch
mkdir /var/log/elasticsearch
mkdir /opt/elasticsearch

chown -R elasticsearch.elasticsearch /data/elasticsearch
chown -R elasticsearch.elasticsearch /var/log/elasticsearch/
chown -R elasticsearch.elasticsearch /opt/elasticsearch

[root@redis03 ~]# su - elasticsearch

tar xf /server/soft/elasticsearch-7.14.0-linux-x86_64.tar.gz -C /opt/elasticsearch
ln -s /opt/elasticsearch/elasticsearch-7.14.0/ /opt/elasticsearch/es

```

配置文件修改

```BASH
[elasticsearch@redis03 ~]$ vim /opt/elasticsearch/es/config/elasticsearch.yml 
cluster.name: elasticsearch
node.name: node-3
path.data: /data/elasticsearch
path.logs: /var/log/elasticsearch
bootstrap.memory_lock: true
network.host: 172.16.0.212
http.port: 9200
discovery.seed_hosts: ["172.16.0.210", "172.16.0.211","172.16.0.212"]
cluster.initial_master_nodes: ["node-1", "node-2","node-3"]
http.cors.enabled: true
http.cors.allow-origin: "*"

[root@redis03 ~]# vim /etc/security/limits.conf 
elasticsearch  -  nofile  65536
# allow user 'elasticsearch' mlockall
elasticsearch soft memlock unlimited
elasticsearch hard memlock unlimited

[root@redis03 ~]# vim /etc/sysctl.conf 
vm.max_map_count=262144

[root@redis03 ~]# sysctl -p

```





redis01

```BASH
[root@redis01 ~]# scp /server/soft/elasticsearch-7.14.0-linux-x86_64.tar.gz redis02:/server/soft

[root@redis01 ~]# scp /server/soft/elasticsearch-7.14.0-linux-x86_64.tar.gz redis03:/server/soft


[root@redis03 ~]# mkdir /data/elasticsearch
[root@redis03 ~]# mkdir /var/log/elasticsearch
[root@redis03 ~]# chown -R elasticsearch.elasticsearch /var/log/elasticsearch/
[root@redis03 ~]# chown -R elasticsearch.elasticsearch /data/elasticsearch

```

启动

```BASH
[elasticsearch@redis02 ~]$ /opt/elasticsearch/es/bin/elasticsearch -d -p pid
[root@redis02 ~]# ps -ef |grep [e]lasticsearch|wc -l
3  
# netstat -lntup|grep -E "9200|9300"
tcp6       0      0 172.16.0.212:9200       :::*     LISTEN      1467/java           
tcp6       0      0 172.16.0.212:9300       :::*     LISTEN      1467/java         


[elasticsearch@redis03 /root]$ /opt/elasticsearch/es/bin/elasticsearch -d -p pid
[root@redis03 ~]# ps -ef |grep [e]lasticsearch|wc -l
3
# netstat -lntup|grep -E "9200|9300"
tcp6       0      0 172.16.0.212:9200       :::*     LISTEN      1467/java           
tcp6       0      0 172.16.0.212:9300       :::*     LISTEN      1467/java         
```

![image-20210813201702478](ELK.assets\image-20210813201702478.png)

搭建好了。测试下面的集群功能







### elasticsearch 目录说明

```BASH
[root@redis01 /server/soft]# rpm -qc elasticsearch   #所有配置文件
/etc/elasticsearch/elasticsearch.yml     #配置文件
/etc/elasticsearch/jvm.options           #jvm虚拟机配置文件
/etc/elasticsearch/log4j2.properties     
/etc/elasticsearch/role_mapping.yml
/etc/elasticsearch/roles.yml
/etc/elasticsearch/users
/etc/elasticsearch/users_roles
/etc/init.d/elasticsearch                 #init启动文件
/etc/sysconfig/elasticsearch              #环境配置文件
/usr/lib/sysctl.d/elasticsearch.conf      #sysctl变量文件，修改最大描述
/usr/lib/systemd/system/elasticsearch.service    #systed启动文件
------------------------------------------
/var/lib/elasticsearch		# 数据目录
/var/log/elasticsearch		#日志目录
/var/run/elasticsearch		#pid目录

[root@redis01 /server/soft]# rpm -ql elasticsearch #所有目录

```

Elasticsearch 已经有了很好的默认值，特别是涉及到性能相关的配置或者选项,其它数据库可能需要调优，但总得来说，Elasticsearch不需要。如果你遇到了性能问题，解决方法通常是更好的数据布局或者更多的节点。

### 重要配置

详细介绍：https://www.elastic.co/guide/cn/elasticsearch/guide/current/important-configuration-changes.html

```bash
[root@redis01 /server/soft]# grep -Ev '#|^$' /etc/elasticsearch/elasticsearch.yml
cluster.name: my-application      #集群名称
node.name: node-1                #节点名称
path.data: /data/elasticsearch    #数据目录
path.logs: /var/log/elasticsearch    #日志目录
bootstrap.memory_lock: true          #锁定内存
network.host: 172.16.0.210           #绑定IP地址
http.port: 9200                      #端口号
discovery.zen.ping.unicast.hosts: ["172.16.0.210"]     #集群发现的通信节点
discovery.zen.minimum_master_nodes: 2        #最小节点数

[root@redis01 /server/soft]# mkdir /data/elasticsearch
[root@redis01 /server/soft]# cd
[root@redis01 ~]# chown -R elasticsearch.elasticsearch /data/elasticsearch/
[root@redis01 ~]# systemctl restart elasticsearch

[root@redis01 ~]# tail -f /var/log/elasticsearch/my-application.log
...
[1] bootstrap checks failed. You must address the points described in the following [1] lines before starting Elasticsearch.
bootstrap check failure [1] of [1]: memory locking requested for elasticsearch process but memory is not locked

```

这时候是fail的因为需要修改参数，参考

https://www.elastic.co/guide/en/elasticsearch/reference/6.6/setup-configuration-memory.html

```bash
[root@redis01 ~]# ps -p 1 #查看使用的是init 还是systemd
  PID TTY          TIME CMD
     1 ?        00:00:14 systemd

方法一：
[root@redis01 ~]# systemctl edit elasticsearch
[Service]
LimitMEMLOCK=infinity

方法二：
[root@redis01 ~]# vim /usr/lib/systemd/system/elasticsearch.service
[Service]
LimitMEMLOCK=infinity

[root@redis01 ~]# systemctl daemon-reload
[root@redis01 ~]# systemctl restart elasticsearch
还是不能启动起来
因为我内存只有2G，把jvm虚拟内存调小。
[root@redis01 ~]# vim /etc/elasticsearch/jvm.options
-Xms512m
-Xmx512m  
```

java真应该淘汰，报错信息太模糊。

```bash
[root@redis01 ~]# vim /usr/lib/systemd/system/elasticsearch.service 
...
ExecStart=/usr/share/elasticsearch/bin/systemd-entrypoint -p ${PID_DIR}/elasticsearch.pid  #   --quiet去掉这个参数
...
这个参数会导致elasticsearch 不能记录启动错误信息

[root@redis01 ~]# journalctl -f  实时监控启动日志

```

## Elasticsearch 术语及概念

**索引词**

在elastiasearch中索引词（term）是一个能够被索引的精确值。foo,Foo,FOO几个单词是不同的索引词。索引词（term）是可以通过term查询进行准确的搜索。

**文本**text

文本是一段普通的非结构化文字。通常，文本会被分拆成一个个的索引词，存储在elasticsearch的索引库中。为了让文本能够进行搜索，文本字段需要事先进行分析了；当对文本中的关键词进行查询的时候，搜索引擎应该根据搜索条件搜索出原文本。

**分析**analysis

分析是将文本转换为索引词的过程，分析的结果依赖于分词器。比如：FOO BAR，Foo-Bar和foo bar这几个词有可能会被分析成相同的索引词foo和bar，这些索引词存储在Elasticsearch的索引库中。

**集群**cluster

集群由一个或多个节点组成，对外提供服务，对外提供索引和搜索功能。在所有节点，一个集群有一个唯一的名称默认为“elasticsearch”.此名称是很重要的，因为每个节点只能是集群的一部分，当该节点被设置为相同的集群名称时，就会自动加入集群。当需要有多个集群的时候，要确保每个集群的名称不能重复，否则节点可能会加入到错误的集群。请注意，一个节点只能加入到一个集群。此外，你还可以拥有多个独立的集群，每个集群都有其不同的集群名称。

**节点**node

一个节点是一个逻辑上独立的服务,它是集群的一部分,可以存储数据,并参与集群的索引和搜索功能。就像集群一样,节点也有唯一的名字,在启动的时候分配。如果你不想要默认名称,你可以定义任何你想要的节点名.这个名字在理中很重要,在Elasticsearch集群通过节点名称进行管理和通信.一个节点可以被配置加入到一个特定的集群。默认情况下,每个节点会加人名为Elasticsearch 的集祥中,这意味着如果你在网热动多个节点,如果网络畅通,他们能彼此发现井自动加人名为Elasticsearch 的一个集群中,你可以拥有多个你想要的节点。当网络没有集群运行的时候,只要启动一个节点,这个节点会默认生成一个新的集群,这个集群会有一个节点。

**分片**shard

分片是单个Lucene 实例,这是Elasticsearch管理的比较底层的功能。索引是指向主分片和副本分片的逻辑空间。 对于使用,只需要指定分片的数量,其他不需要做过多的事情。在开发使用的过程中,我们对应的对象都是索引,Elasticsearch 会自动管理集群中所有的分片,当发生故障的时候,Elasticsearch 会把分片移动到不同的节点或者添加新的节点。

一个索引可以存储很大的数据,这些空间可以超过一个节点的物理存储的限制。例如,十亿个文档占用磁盘空间为1TB。仅从单个节点搜索可能会很慢,还有一台物理机器也不一定能存储这么多的数据。为了解决这一问题,Elasticsearch将索引分解成多个分片。当你创建一个索引,你可以简单地定义你想要的分片数量。每个分片本身是一个全功能的、独立的单元,可以托管在集群中的任何节点。

**主分片**

每个文档都存储在一个分片中,当你存储一个文档的时候,系统会首先存储在主分片中,然后会复制到不同的副本中。默认情况下,一个索引有5个主分片。 你可以事先制定分片的数量,当分片一旦建立,则分片的数量不能修改。

**副本分片**

每一个分片有零个或多个副本。副本主要是主分片的复制,其中有两个目的:

\- 增加高可用性:当主分片失败的时候,可以从副本分片中选择一个作为主分片。

\- 提高性能:当查询的时候可以到主分片或者副本分片中进行查询。默认情況下,一个主分片配有一个副本,但副本的数量可以在后面动态地配置增加。副本分片必部署在不同的节点上,不能部署在和主分片相同的节点上。

分片主要有两个很重要的原因是:

\- 允许水平分割扩展数据。

\- 允许分配和井行操作(可能在多个节点上)从而提高性能和吞吐量。

这些很强大的功能对用户来说是透明的,你不需要做什么操作,系统会自动处理。

**复制**

复制是一个非常有用的功能,不然会有单点问题。 当网络中的某个节点出现问题的时候,复制可以对故障进行转移,保证系统的高可用。因此,Elasticsearch 允许你创建一个或多个拷贝,你的索引分片就形成了所谓的副本或副本分片。

复制是重要的,主要的原因有:

\- 它提供了高可用性,当节点失败的时候不受影响。需要注意的是,一个复制的分片不会存储在同一个节点中。

\- 它允许你扩展搜索量,提高并发量,因为搜索可以在所有副本上并行执行。

每个索引可以拆分成多个分片。索引可以复制零个或者多个分片。一旦复制,每个索引就有了主分片和副本分片。分片的数量和副本的数量可以在创建索引时定义。 当创建索引后,你可以随时改变副本的数量,但你不能改变分片的数量。

默认情況下,每个索引分配5个分片和一个副本,这意味着你的集群节点至少要有两个节点,你将拥有5个主要的分片和5个副本分片共计10个分片.

每个Elasticsearch分片是一个Lucene 的索引。有文档存储数量限制,你可以在一个

单一的Lucene索引中存储的最大值为lucene-5843,极限是2147483519(=integer.max_value-128)个文档。你可以使用cat/shards API监控分片的大小。

**索引**

索引是具有相同结构的文档集合。例如,可以有一个客户信息的索引,包括一个产品目录的索引,一个订单数据的索引。 在系统上索引的名字全部小写,通过这个名字可以用来执行索引、搜索、更新和删除操作等。在单个集群中,可以定义多个你想要的索引。

**类型**

在索引中,可以定义一个或多个类型,类型是索引的逻辑分区。在一般情况下,一种类型被定义为具有一组公共字段的文档。例如,让我们假设你运行一个博客平台,并把所有的数据存储在一个索引中。在这个索引中,你可以定义一种类型为用户数据,一种类型为博客数据,另一种类型为评论数据。

**文档**

文档是存储在Elasticsearch中的一个JSON格式的字符串。它就像在关系数据库中表的一行。每个存储在索引中的一个文档都有一个类型和一个ID,每个文档都是一个JSON对象,存储了零个或者多个字段,或者键值对。原始的JSON 文档假存储在一个叫作Sour的字段中。当搜索文档的时候默认返回的就是这个字段。

**映射**

映射像关系数据库中的表结构,每一个索引都有一个映射,它定义了索引中的每一个字段类型,以及一个索引范围内的设置。一个映射可以事先被定义,或者在第一次存储文档的时候自动识别。

**字段**

文档中包含零个或者多个字段,字段可以是一个简单的值(例如字符串、整数、日期),也可以是一个数组或对象的嵌套结构。字段类似于关系数据库中表的列。每个字段都对应一个字段类型,例如整数、字符串、对象等。字段还可以指定如何分析该字段的值。

**主键**

ID是一个文件的唯一标识,如果在存库的时候没有提供ID,系统会自动生成一个ID,文档的 index/type/id必须是唯一的。

| Elasticsearch | 数据库 |
| ------------- | ------ |
| Document      | 行     |
| Type          | 表     |
| Index         | 库     |
| filed         | 字段   |

## 交互

所有其他语言可以使用RESTful API通过端口9200和Elasticsearch进行通信，你可以用你最喜爱的web客户端访问Elasticsearch.事实上，正如你所看到的，你甚至可以使用curl命令来和Elasticsearch交互 。

一个 Elasticsearch 请求和任何 HTTP 请求一样由若干相同的部件组成:

```bash
curl -X<VERB> '<PROTOCOL>://<HOST>:<PORT>/<PATH>?<QUERY_STRING>' -d '<BODY>'
VERB 适当的 HTTP 方法 或 谓词 : GET`、 `POST`、 `PUT`、 `HEAD 或者 `DELETE`。 
PROTOCOL http 或者 https`(如果你在 Elasticsearch 前面有一个 `https 代理)
HOST  Elasticsearch 集群中任意节点的主机名，或者用 localhost 代表本地机器上的节点。 
PORT  运行 Elasticsearch HTTP 服务的端口号，默认是 9200 。
PATH API 的终端路径(例如 _count 将返回集群中文档数量)。
Path 可能包含多个组件，例如: _cluster/stats 和 _nodes/stats/jvm 。
QUERY_STRING 任意可选的查询字符串参数 (例如 ?pretty 将格式化地输出 JSON 返回值，使其更容易 阅读)
BODY 一个 JSON 格式的请求体 (如果请求需要的话)
```

### pretty参数

当你在任何请求中添加了参数?pretty=true时，请求的返回值是经过格式化后的JSON数据，这样阅读起来更加方便。系统还提供了另一种格式的格式化，?format=yaml,YAML格式，这将导致返回的结果具有可读的YAML格式。

### human参数

对于统计数据，系统支持计算机数据，同时也支持比较适合人类阅读的数据。?human=true，默认是false

### 响应过滤 filter_path

所有的返回值通过filter_path减少返回值的内容，多个值可以用逗号分开。也可以使用通配符*



### 三种交互方式

**curl命令：**

 最繁琐,最复杂,最容易出错,不需要安装任何软件，只需要有curl命令



**es-head插件**

 查看数据方便,操作相对容易,需要node环境



**kibana**

 查看数据以及报表格式丰富,操作很简单,需要java环境和安装配置kibana



### curl命令交互

计算文档数量

```BASH
curl -XGET 'http://172.16.0.210:9200/_count?pretty' -H 'Content-Type:
application/json' -d '   
{
  "query": { "match_all": {}
  } 
}
'
---------------------------------------------
#创建索引
curl -XPUT '172.16.0.210:9200/vipinfo?pretty’ 
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : ”vipinfo"
}


---------------------------------------------
#插入数据
curl -XPUT '172.16.0.210:9200/vipinfo/user/1?pretty' -H 'Content-Type: application/json' -d'
{
    "first_name" : "John",
    "last_name": "Smith",
    "age" : 25,
    "about" : "I love to go rock climbing", "interests": [ "sports", "music" ]
}
'

curl -XPUT  'localhost:9200/vipinfo/user/2?pretty' -H 'Content-Type: application/json' -d' {
"first_name": "Jane",
"last_name" : "Smith",
"age" : 32,
"about" : "I like to collect rock albums", "interests": [ "music" ]
}’

curl –XPUT  'localhost:9200/vipinfo/user/3?pretty' -H 'Content-Type: application/json' -d' {
"first_name": "Douglas", "last_name" : "Fir",
"age" : 35,
"about": "I like to build cabinets", "interests": [ "forestry" ]
}’

```

### es-head 插件交互

https://github.com/mobz/elasticsearch-head

#### 使用docker部署elasticsearch-head

```BASH
docker pull alivv/elasticsearch-head
docker run --name es-head -p 9100:9100 -dit elivv/elasticsearch-head
```

#### 使用nodejs编译安装

官网地址：https://nodejs.org/en/download/package-manager/

​                   https://nodejs.org/dist/latest-v10.x/

​                   http://npm.taobao.org

```BASH
yum install nodejs npm openssl screen -y
node -v
npm  -v
npm install -g cnpm --registry=https://registry.npm.taobao.org
cd /opt/
git clone git://github.com/mobz/elasticsearch-head.git
cd elasticsearch-head/
cnpm install
screen -S es-head
cnpm run start
Ctrl+A+D
```

修改ES配置文件支持跨域

```BASH
http.cors.enabled: true 
http.cors.allow-origin: "*"
```

网页访问

使用Elasticsearch head插件

需要翻墙到google app 商城下载

![image-20210811221928275](ELK.assets\image-20210811221928275.png)

![image-20210811221950574](ELK.assets\image-20210811221950574.png)

![image-20210811222017150](ELK.assets\image-20210811222017150.png)

### kibana交互

https://www.elastic.co/cn/kibana/

Kibana 是一个免费且开放的用户界面，能够让您对 Elasticsearch 数据进行可视化，并让您在 Elastic Stack 中进行导航。您可以进行各种操作，从跟踪查询负载，到理解请求如何流经您的整个应用，都能轻松完成。

```BASH
wget https://artifacts.elastic.co/downloads/kibana/kibana-7.14.0-x86_64.rpm
[root@redis01 /server/soft]# rpm -ivh kibana-7.14.0-x86_64.rpm 
[root@redis01 /server/soft]# vim /etc/kibana/kibana.yml 
[root@redis01 /server/soft]# grep -Ev '#|^$' /etc/kibana/kibana.yml 
server.port: 5601
server.host: "172.16.0.210"
elasticsearch.hosts: ["http://172.16.0.210:9200"]
kibana.index: ".kibana"

# systemctl start kibana
# systemctl enable kibana
# systemctl status kibana

[root@redis01 ~]# lsof -i:5601
COMMAND   PID   USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
node    21663 kibana   46u  IPv4  55526      0t0  TCP redis01:esmagent (LISTEN)
```

http://172.16.0.210:5601/  访问

### elasticsearch 文档元数据

一个文档不仅仅包含它的数据 ，也包含元数据 —— 有关文档的信息。

三个必须的元数据元素如下: 

_index 文档在哪存放
       _type 文档表示的对象类别
       _id 文档唯一标识 

**_index** 

一个 索引 应该是因共同的特性被分组到一起的文档集合。 例如，你可能存储所有的产品在索引 products 中，而存储所有销售的交易到索引 sales 中



       **_type**
 数据可能在索引中只是松散的组合在一起，但是通常明确定义一些数据中的子分区是很有用的。 例如，所 有的产品都放在一个索引中，但是你有许多不同的产品类别，比如 "electronics" 、 "kitchen" 和 "lawn- care"。 这些文档共享一种相同的(或非常相似)的模式:他们有一个标题、描述、产品代码和价格。他们只是正 好属于“产品”下的一些子类。 

Elasticsearch 公开了一个称为 types (类型)的特性，它允许您在索引中对数据进行逻辑分区。不同 types 的文档可能有不同的字段，但最好能够非常相似。

 **_id**
 ID 是一个字符串， 当它和 _index 以及 _type 组合就可以唯一确定 Elasticsearch 中的一个文档。 当你创 建一个新的文档，要么提供自己的 _id ，要么让 Elasticsearch 帮你生成 

官网地址：https://www.elastic.co/guide/en/elasticsearch/reference/current/indices.html

插入数据

```BASH
curl -XPUT '172.16.0.210:9200/megacorp/employee/1?pretty' -H 'Content-Type: application/json' -d'
{
    "first_name" : "John",
    "last_name": "Smith",
    "age" : 25,
    "about" : "I love to go rock climbing", "interests": [ "sports", "music" ]
}
'

curl -XPUT '172.16.0.210:9200/megacorp/employee/2?pretty' -H 'Content-Type: application/json' -d' {
"first_name": "Jane",
"last_name" : "Smith",
"age" : 32,
"about" : "I like to collect rock albums", "interests": [ "music" ]
}
'

curl -XPUT '172.16.0.210:9200/megacorp/employee/3?pretty' -H 'Content-Type: application/json' -d' {
"first_name": "Douglas", "last_name" : "Fir",
"age" : 35,
"about": "I like to build cabinets", "interests": [ "forestry" ]
} '

```

查询文档

```BASH
curl -XGET '172.16.0.210:9200/megacorp/employee/1?pretty'
```

删除文档

```BASH
curl -XDELETE '172.16.0.210:9200/megacorp/employee/1?pretty'
```

创建索引

```BASH
curl -XPUT '172.16.0.210:9200/megacorp?pretty'
```

\#查询索引中所有的信息

```bash
curl -XGET '172.16.0.210:9200/megacorp/employee/_search?pretty'
```

\#查询索引符合条件的信息：搜索姓名为：Smith的员工

```bash
curl -XGET '172.16.0.210:9200/megacorp/employee/_search?q=last_name:Smith&pretty'
```

\#使用Query-string查询

```bash
curl -XGET '172.16.0.210:9200/megacorp/employee/_search?pretty' -H 'Content-Type: application/json' -d'      
{
"query" : { "match" : {
"last_name" : "Smith" }
 } 
}
'
```

\#使用过滤器

```bash
curl -XGET '172.16.0.210:9200/megacorp/employee/_search?pretty' -H 'Content-Type: application/json' -d'       
{
"query" : { "bool": {
"must": { "match" : {
"last_name" : "smith" }
}, "filter": {
"range" : {
"age" : { "gt" : 30 }
} }
} }
}
'
```

#删除索引

```BASH
curl -XDELETE '172.16.0.210:9200/megacorp?pretty'     
```

## 集群管理

Elasticsearch 可以横向扩展至数百(甚至数千)的服务器节点，同时可以处理PB级数据 Elasticsearch 天生就是分布式的，并且在设计时屏蔽了分布式的复杂性。

这里列举了一些在后台自动执行的操作: 

分配文档到不同的容器 或 分片中,文档可以储存在一个或多个节点中.按集群节点来均衡分配这些分片，从而对索引和搜索过程进行负载均衡.复制每个分片以支持数据冗余，从而防止硬件故障导致的数据丢失.将集群中任一节点的请求路由到存有相关数据的节点.集群扩容时无缝整合新节点，重新分配分片以便从离群节点恢复 

一个运行中的 Elasticsearch 实例称为一个 节点，而集群是由一个或者多个拥有相同 cluster.name 配置的节点组成,它们共同承担数据和负载的压力。

当有节点加入集群中或者从集群中移除节点时，集群将会重新平均分布所有的数据。

当一个节点被选举成为主节点时,它将负责管理集群范围内的所有变更,例如增加、删除索引,或者增加、删除节点等.而主节点并不需要涉及到文档级别的变更和搜索等操作,所以当集群只拥有一个主节点的情况下,即使流量的增加它也不会成为瓶颈.任何节点都可以成为主节点。我们的示例集群就只有一个节点,所以它同时也成为了主节点。 

作为用户,我们可以将请求发送到 集群中的任何节点,包括主节点.每个节点都知道任意文档所处的位置,并且能够将我们的请求直接转发到存储我们所需文档的节点.无论我们将请求发送到哪个节点,它都能负责从各个包含我们所需文档的节点收集回数据,并将最终结果返回給客户端. Elasticsearch 对这一切的管理都是透明的。 

集群的安装部署和单机没有什么区别，区别在于配置文件里配置上集群的相关参数

### 部署集群

```BASH
配置文件
cluster.name: test    #所有节点必须一致 
node.name: node-1   
path.data: /data/elasticsearch
path.logs: /var/log/elasticsearch
bootstrap.memory_lock: true
network.host: 172.16.0.210 ,127.0.0.1
http.port: 9200
discovery.seed_hosts: ["172.16.0.210", "172.16.0.211"]
cluster.initial_master_nodes: ["node-1","node-2"]  
http.cors.enabled: true 
http.cors.allow-origin: "*"
```

discovery.seed_hosts: ["host1", "host2"]  以前叫做 discovery.zen.ping.unicast.hosts

但是还是支持，向后兼容。https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-discovery-settings.html

cluster.initial_master_nodes: ["node-1"] 以前叫做 discovery.zen.minimum_master_nodes

https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-discovery-bootstrap-cluster.html  首次启动集群需要，后面不需要。

这条设置意思是集群的主节点集合，是一个列表

在集群中的单个主节点上设置 cluster.initial _ master _ nodes 在技术上是足够的，并且只提到设置值中的单个节点，但是在集群完全形成之前，这不提供容错性。因此，最好使用至少三个符合主节点条件的节点进行引导，每个节点使用 cluster.initial _ master _ nodes 设置，其中包含所有三个节点。

必须将 cluster.initial _ master _ nodes 设置为每个节点上设置它的相同节点列表，以确保在引导期间只形成一个集群，从而避免数据丢失的风险。

```bash
http.cors.enabled: true             #是否支持跨域，默认为false
http.cors.allow-origin: "*"         #当设置允许跨域，默认为*,表示支持所有域名，如果我们只是允许某些网站能访问，那么可以使用正则表达式。比如只允许本地地址。 /https?:\/\/localhost(:[0-9]+)?/
```

https://www.elastic.co/guide/en/elasticsearch/reference/7.14/modules-network.html

http.cors.allow-origin

(静态)哪个起源允许。如果您预先设置并在值后面追加一个正斜杠(/) ，那么这将被视为一个正则表达式，从而允许您支持 HTTP 和 HTTPs。例如，使用/https？//localhost (: [0-9] +) ？/将在这两种情况下适当返回请求头。违约不允许有任何来源。通配符(*)是一个有效的值，但被认为是一个安全风险，因为 Elasticsearch 实例可以从任何地方跨原始请求打开。

```BASH
[root@redis01 ~]# systemctl daemon-reload
[root@redis01 ~]# systemctl restart elasticsearch
[root@redis01 ~]# systemctl status elasticsearch
```



部署其他节点

redis02

```BASH
[root@redis02 /server/soft]# rpm -ivh jdk-8u301-linux-x64.rpm 
[root@redis02 /server/soft]# rpm -ivh elasticsearch-7.14.0-x86_64.rpm 
[root@redis02 /server/soft]# mkdir /data/elasticsearch
[root@redis02 /server/soft]# vim /etc/elasticsearch/jvm.options
...
-Xms512m
-Xmx512m    
vim /usr/lib/systemd/system/elasticsearch.service 
...
[service]
LimitMEMLOCK=infinity
--quiet去掉这个参数
chown -R elasticsearch.elasticsearch /data/elasticsearch/
```

redis03

```BASH
[root@redis03 /server/soft]# rpm -ivh jdk-8u301-linux-x64.rpm 
[root@redis03 /server/soft]# rpm -ivh elasticsearch-7.14.0-x86_64.rpm 
[root@redis03 /server/soft]# mkdir /data/elasticsearch
[root@redis03 /server/soft]# vim /etc/elasticsearch/jvm.options
...
-Xms512m
-Xmx512m  
vim /usr/lib/systemd/system/elasticsearch.service 
[service]
LimitMEMLOCK=infinity
--quiet去掉这个参数
chown -R elasticsearch.elasticsearch /data/elasticsearch/
```

redis01

```bash
[rootredis03@redis01 /server/soft]# scp /etc/elasticsearch/elasticsearch.yml redis02:/etc/elasticsearch/elasticsearch.yml 
 
[root@redis01 /server/soft]# scp /etc/elasticsearch/elasticsearch.yml redis03:/etc/elasticsearch/elasticsearch.yml 

```

修改配置

```BASH
node.name: node-2
network.host: 172.16.0.211 ,127.0.0.1
```

```BASH
systemctl daemon-reload && systemctl start elasticsearch
```

![image-20210812220947038](ELK.assets\image-20210812220947038.png)



查看集群状态

https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-health.html

```BASH
[root@redis01 /server/soft]# curl -XGET 'http://172.16.0.210:9200/_cluster/health?pretty'
{
  "cluster_name" : "test",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 3,
  "number_of_data_nodes" : 3,
  "active_primary_shards" : 8,
  "active_shards" : 16,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 100.0
}
```

查看系统信息检索

Cluster Stats API允许从群集范围的角度检索统计信息。

https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-stats.html

```BASH
[root@redis01 ~]# curl -XGET 'http://172.16.0.210:9200/_cluster/stats?pretty'
```

查看集群的设置

https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-get-settings.html

```BASH
curl -XGET 'http://172.16.0.210:9200/_cluster/settings?pretty'
```

查看集群的节点状态

https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-nodes-info.html

```BASH
[root@redis01 ~]# curl -XGET 'http://172.16.0.210:9200/_nodes?pretty'
[root@redis01 ~]# curl -XGET 'http://172.16.0.210:9200/_nodes/process?pretty'
[root@redis01 ~]# curl -XGET 'http://172.16.0.210:9200/_nodes/_all/process?pretty'
[root@redis01 ~]# curl -XGET 'http://172.16.0.210:9200/_nodes/plugins?pretty'
```

重要

```BASH
curl -XGET 'http://172.16.0.210:9200/_nodes/procese?human&pretty'  #查看集群名
curl -XGET 'http://172.16.0.210:9200/_nodes/_all/info/jvm,process?human&pretty'
[root@redis01 ~]# curl -XGET 'http://172.16.0.210:9200/_cat/nodes?human&pretty'
172.16.0.210 60 96 6 0.16 0.14 0.24 cdfhilmrstw * node-1
172.16.0.212 59 95 1 0.03 0.02 0.07 cdfhilmrstw - node-3
172.16.0.211 67 96 5 0.09 0.11 0.20 cdfhilmrstw - node-2

```

### 索引分片

默认ES会创建5分片1副本的配置

```BASH
curl -XPUT '172.16.0.210:9200/blogs?pretty' -H 'Content-Type: application/json' -d'
{
"settings" : { 
   "number_of_shards" : 3,       
   "number_of_replicas" : 1
 } 
}'
-----------------------------
 "number_of_shards" : 3,       #切片数3   
 "number_of_replicas" : 1      #1副本
```

![image-20210812224227979](ELK.assets\image-20210812224227979.png)

系统默认只有1个分片1个副本

```BASH
[root@redis01 ~]# curl -XPUT 'http://172.16.0.210:9200/index1?pretty'
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "index1"
}
```

![image-20210812224940438](ELK.assets\image-20210812224940438.png)

调整福本数

分片数一旦创建就不能再更改了，但是我们可以调整副本数

```BASH
curl -XPUT '172.16.0.210:9200/blogs1?pretty' -H 'Content-Type: application/json' -d'
{
"settings" : {     
   "number_of_replicas" : 2
 } 
}'
```

![image-20210812224956556](ELK.assets\image-20210812224956556.png)

改成5分片1副本

```BASH
curl -XPUT '172.16.0.210:9200/blogs2?pretty' -H 'Content-Type: application/json' -d'
{
"settings" : { 
   "number_of_shards" : 5,       
   "number_of_replicas" : 1
 } 
}'
```

![image-20210812225218910](ELK.assets\image-20210812225218910.png)



分片与复制

此时如果我们关闭一个节点，会发现集群状态发生了改变

在我们关闭 Node 1 的同时也失去了主分片 1 和 2 ，并且在缺失主分片的时候索引也不能正常工作。 如果 此时来检查集群的状况，我们看到的状态将会为 red :不是所有主分片都在正常工作。 幸运的是，在其它节点上存在着这两个主分片的完整副本， 所以新的主节点立即将这些分片在 Node 2 和 Node 3 上对应的副本分片提升为主分片， 此时集群的状态将会为 yellow 。 这个提升主分片的过程是瞬间 发生的，如同按下一个开关一般。 

为什么我们集群状态是 yellow 而不是 green 呢? 虽然我们拥有所有的三个主分片，但是同时设置了每个主 分片需要对应2份副本分片，而此时只存在一份副本分片。 所以集群不能为 green 的状态，不过我们不必 过于担心:如果我们同样关闭了 Node 2 ，我们的程序 依然 可以保持在不丢任何数据的情况下运行，因为 Node 3 为每一个分片都保留着一份副本。

如果我们重新启动 Node 1 ，集群可以将缺失的副本分片再次进行分配 

> 就用上面的环境做测试
>
> 现在我们关node-1

![image-20210812225650017](ELK.assets\image-20210812225650017.png)

## 集群健康值的几种状态如下

绿色:所有条件都满足，数据完整，副本满足
       黄色:数据完整，副本不满足	
       红色:有索引里的数据出现不完整了
       紫色:有分片正在同步中

灰色：未连接到elasticsearch服务

```BASH
[root@redis03 ~]# vim /etc/elasticsearch/elasticsearch.yml 
...
#cluster.initial_master_nodes: ["node-1","node-2","node-3"]
...
```

我们先把配置文件这个注释了

![image-20210812230318862](ELK.assets\image-20210812230318862.png)

刷新看看cluster状态

![image-20210812230758514](ELK.assets\image-20210812230758514.png)

紫色的是正在移动

如果此时宕机2个节点会怎么样

![image-20210812231237501](ELK.assets\image-20210812231237501.png)

当两个节点全宕了，集群也就不能启动起来了。

此时要快速启动起来怎么办？以前可以使用下面配置

```BASH
[root@redis03 ~]# vim /etc/elasticsearch/elasticsearch.yml 
cluster.initial_master_nodes: ["node-3"]
```

以上是rpm包安装的好像里面自带了些库。



下面测试是二进制包部署安装的。

### **已经存在的index 怎么更新副本？**

```BASH
curl -XPUT '172.16.0.210:9200/blogs2/_settings?pretty' -H 'Content-Type: application/json' -d'
{
"index" : {     
   "number_of_replicas" : 2
 } 
}'
```

![image-20210813204200592](ELK.assets\image-20210813204200592.png)

```BASH
curl -XPUT '172.16.0.210:9200/blogs1?pretty' -H 'Content-Type: application/json' -d'
{
"settings" : { 
   "number_of_shards" : 5,       
   "number_of_replicas" : 1
 } 
}'
```

![image-20210813204816734](ELK.assets\image-20210813204816734.png)

上面创建了一个5分片2副本和一个5分片1副本的例子

插入一部分数据

```BASH
curl -XPUT '172.16.0.210:9200/blogs1/user/1?pretty' -H 'Content-Type: application/json' -d'
{
    "first_name" : "John",
    "last_name": "Smith",
    "age" : 25,
    "about" : "I love to go rock climbing", "interests": [ "sports", "music" ]
}
'


curl -XPUT '172.16.0.210:9200/blogs1/user/2?pretty' -H 'Content-Type: application/json' -d'
{
    "first_name" : "Tom",
    "last_name": "cat",
    "age" : 35,
    "about" : "I love to go rock climbing", "interests": [ "sports", "music" ]
}
'

curl -XPUT '172.16.0.210:9200/blogs1/user/3?pretty' -H 'Content-Type: application/json' -d'
{
    "first_name" : "wan",
    "last_name": "cat",
    "age" : 30,
    "about" : "I love to go rock climbing", "interests": [ "sports", "music" ]
}
'

curl -XPUT '172.16.0.210:9200/blogs2/love/1?pretty' -H 'Content-Type: application/json' -d'
{
    "first_name" : "1111",
    "last_name": "2222",
    "age" : 27,
    "about" : "I love to go rock climbing", "interests": [ "sports", "music" ]
}
'

```

![image-20210813205522298](ELK.assets\image-20210813205522298.png)



### 1、宕机一个节点

![image-20210813205650545](ELK.assets\image-20210813205650545.png)

挂了一个节点，上面状态颜色变成黄色了，是因为副本数不满足。

我们把配置文件注释了下面这行

```BASH
#cluster.initial_master_nodes: ["node-1", "node-2","node-3"]
这行配置的意思是第一次集群时候需要后面就不需要了。可以注释了。
```

![image-20210813210725113](ELK.assets\image-20210813210725113.png)

这种情况还可以再挂一台服务器，数据还在。

### 2、同时宕机2台

![image-20210813210933467](ELK.assets\image-20210813210933467.png)

完整的。

宕机之后只有一台，

```BASH
[root@redis01 /opt/elasticsearch/es]# rm -rf /data/elasticsearch/*   #生产中移走
[root@redis01 /opt/elasticsearch/es]# rm -rf data/*
[root@redis01 /opt/elasticsearch/es]# vim config/elasticsearch.yml
discovery.seed_hosts: ["172.16.0.210"]
cluster.initial_master_nodes: ["node-1"]       
```

上面是清空集群节点还原成单节点运行。





还有一个问题，异地机房中间网络故障会导致2个集群，如果出现这种情况那么闲备份好data里面的数据。

修复好节点等待数据还原。概率极低。很棘手的故障。

清空一个数据不全的节点重启就可以加入到已有的集群，配置一样的情况



# ELK

## 介绍

ELK：https://demo.elastic.co/ 官网演示

ELK就是日志分析系统。ELK stack 也就是elastic stack

组件：kibana          elasticsearch图形图标可视化工具

​            elasticsearch    非关系型数据库

​            logstash        服务端数据处理管道，能够从多个来源采集数据，转换数据，然后将数据发送到数据库

==**ELK stack 涉及组件Elasticsearch   Logstash   Kibana Filebeat   Kafka**==

**Logstash** ：数据收集处理引擎。支持动态的从各种数据源搜集数据，并对数据进行过滤、分析、丰富、统一格式等操作，然后存储以供后续使用。

**Kibana** ：可视化化平台。它能够搜索、展示存储在 Elasticsearch 中索引数据。使用它可以很方便的用图表、表格、地图展示和分析数据。

**Elasticsearch** ：分布式搜索引擎。具有高可伸缩、高可靠、易管理等特点。可以用于全文检索、结构化检索和分析，并能将这三者结合起来。Elasticsearch 基于 Lucene 开发，现在使用最广的开源搜索引擎之一，Wikipedia 、StackOverflow、Github 等都基于它来构建自己的搜索引擎。

**Filebeat** ：轻量级数据收集引擎。基于原先 Logstash-fowarder 的源码改造出来。换句话说：Filebeat就是新版的 Logstash-fowarder，也会是 ELK Stack 在 shipper 端的第一选择。

既然要谈 ELK 在沪江系统中的应用，那么 ELK 架构就不得不谈。本次分享主要列举我们曾经用过的 ELK 架构，并讨论各种架构所适合的场景和优劣供大家参考

Filebeat 是基于原先 logstash-forwarder 的源码改造出来的，无需依赖 Java 环境就能运行，安装包10M不到。如果日志的量很大，Logstash 会遇到资源占用高的问题，为解决这个问题，引入了Filebeat。Filebeat 是基于 logstash-forwarder 的源码改造而成，用 Golang 编写，无需依赖 Java 环境，效率高，占用内存和 CPU 比较少，非常适合作为 Agent 跑在服务器上。

### 压测环境

- 虚拟机 8 cores 64G内存 540G SATA盘
- Logstash 版本 2.3.1
- Filebeat 版本 5.5.0

### 压测方案

Logstash / Filebeat 读取 350W 条日志 到 console，单行数据 580B，8个进程写入采集文件

### 压测结果

| 项目     | workers | cpu usr | 总共耗时 | 收集速度    |
| -------- | ------- | ------- | -------- | ----------- |
| Logstash | 8       | 53.7%   | 210s     | 1.6w line/s |
| Filebeat | 8       | 38.0%   | 30s      | 11w line/s  |

Filebeat 所消耗的CPU只有 Logstash 的70%，但收集速度为 Logstash 的7倍。从我们的应用实践来看，Filebeat 确实用较低的成本和稳定的服务质量，解决了 Logstash 的资源消耗问题。

基于 ELK stack 的日志解决方案的优势主要体现于

- 可扩展性：采用高可扩展性的分布式系统架构设计，可以支持每日 TB 级别的新增数据。
- 使用简单：通过用户图形界面实现各种统计分析功能，简单易用，上手快
- 快速响应：从日志产生到查询可见，能达到秒级完成数据的采集、处理和搜索统计。
- 界面炫丽：Kibana 界面上，只需要点击鼠标，就可以完成搜索、聚合功能，生成炫丽的仪表板

## 日志收集分类

代理层：nginx  haproxy

web层：nginx  tomcat

数据库层：MySQL，redis.mongo,elasticsearch

操作系统：source message

## 安装部署ELK

官网：https://www.elastic.co/guide/index.html

elasticsearch install   参考上面

**配置文件**

```BASH
[root@redis01 ~]# rm -rf  /data/elasticsearch/*
[elasticsearch@redis01 /opt/elasticsearch/es]$ vim config/elasticsearch.yml
#cluster.name: elasticsearch
node.name: node-1   
path.data: /data/elasticsearch 
path.logs: /var/log/elasticsearch      
bootstrap.memory_lock: true  
network.host: 172.16.0.210  
http.port: 9200   
discovery.seed_hosts: ["172.16.0.210"]
http.cors.enabled: true
http.cors.allow-origin: "*"

[root@redis01 /opt/elasticsearch]# vim config/jvm.options
-Xms512m
-Xmx512m  

优化项
[root@redis01 ~]# cat /etc/sysctl.conf 
vm.max_map_count=262144
[root@redis01 ~]# cat /etc/security/limits.conf 
elasticsearch  -  nofile  65536

# allow user 'elasticsearch' mlockall
elasticsearch soft memlock unlimited
elasticsearch hard memlock unlimited

[root@redis01 ~]# sysctl -p

#启动
[elasticsearch@redis01 ~]$ /opt/elasticsearch/es/bin/elasticsearch

```

ntpdate

```BASH
yum install ntpdate -y
ntpdate time1.aliyun.com
```



kibana  参考上面

### 二进制kibana 安装部署

```BASH
[elasticsearch@redis01 /server/soft]$ tar xf kibana-7.14.0-linux-x86_64.tar.gz -C /opt/elasticsearch/
[elasticsearch@redis01 /server/soft]$ cd /opt/elasticsearch/
[elasticsearch@redis01 /opt/elasticsearch]$ ln -s kibana-7.14.0-linux-x86_64 kibana
[elasticsearch@redis01 /opt/elasticsearch]$ cd kibana
```

修改配置

```BASH
[elasticsearch@redis01 /opt/elasticsearch/kibana]$ vim config/kibana.yml
  2 server.port: 5601
  7 server.host: "172.16.0.210"
  server.name: "redis01"    
 32 elasticsearch.hosts: ["http://172.16.0.210:9200"]
 36 kibana.index: ".kibana"
```

启动

```BASH
[elasticsearch@redis01 /opt/elasticsearch/kibana]$ ./bin/kibana
```

访问

```BASH
http://172.16.0.210:5601
```

Kibana server is not ready yet  提示这个是因为服务没有启动

```BASH
[root@redis01 /opt/elasticsearch/es]# netstat -lntup|grep 5601
tcp     0     0 172.16.0.210:5601    0.0.0.0:*    LISTEN    10104/node          
```

![image-20210815095152778](ELK.assets\image-20210815095152778.png)

体验数据

### 安装filebeat和logstash

filebeat下载：https://www.elastic.co/cn/downloads/

部署安装说明：

https://www.elastic.co/guide/en/beats/filebeat/7.14/filebeat-installation-configuration.html

logstash 下载:https://www.elastic.co/cn/downloads/logstash

```BASH
[root@redis01 /server/soft]# ls
elasticsearch-7.14.0-linux-x86_64.tar.gz  filebeat-7.14.0-x86_64.rpm  kibana-7.14.0-linux-x86_64.tar.gz  logstash-7.14.0-x86_64.rpm
elasticsearch-7.14.0-x86_64.rpm           jdk-8u301-linux-x64.rpm     kibana-7.14.0-x86_64.rpm           redis-6.2.5.tar.gz

[root@redis01 /server/soft]# rpm -ivh filebeat-7.14.0-x86_64.rpm 
[root@redis01 /server/soft]# rpm -ivh logstash-7.14.0-x86_64.rpm 

```

## 使用filebeat 配置日志收集

```BASH
[root@redis01 /server/soft]# yum install nginx httpd-tools -y
systemctl start nginx &&  systemctl enable nginx
[root@redis01 /server/soft]# netstat -lntup|grep nginx
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      10405/nginx: master 
tcp6       0      0 :::80                   :::*                    LISTEN      10405/nginx: master 

模拟100条正常访问的数据，100条404
[root@redis01 /server/soft]# ab -c 10 -n 100 172.16.0.210/
[root@redis01 /server/soft]# ab -c 10 -n 100 172.16.0.210/test.html

[root@redis01 /server/soft]# wc -l /var/log/nginx/access.log 
200 /var/log/nginx/access.log
[root@redis01 /server/soft]# wc -l /var/log/nginx/error.log 
100 /var/log/nginx/error.log
```

### 配置filebeat

install

https://www.elastic.co/guide/en/beats/filebeat/7.14/filebeat-installation-configuration.html

详细项

https://www.elastic.co/guide/en/beats/filebeat/7.14/configuring-howto-filebeat.html

```BASH
[root@redis01 /server/soft]# vim /etc/filebeat/filebeat.yml 
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/nginx/access.log   
setup.kibana:     
  host: "172.16.0.210:5601"   
output.elasticsearch:
  hosts: ["172.16.0.210:9200"]   
  index: "nginx-%{[beat.version]}-%{+yyyy.MM.dd}"   
setup.template.settings:
  index.number_of_shards: 3     #3分片  
setup.template.name: "nginx"
setup.template.pattern: "nginx-*"
setup.template.enabled: false
setup.template.overwrite: true

```

查看默认开启的模块

```BASH
filebeat modules list
```

手动开启需要的模块

```BASH
filebeat modules enable system nginx mysql
```

到对应的模块下编辑yml

```BASH
[root@redis01 /server/soft]# vim /etc/filebeat/modules.d/nginx.yml 
- module: nginx
  # Access logs
  access:
    enabled: true

    # Set custom paths for the log files. If left empty,
    # Filebeat will choose the paths depending on your OS.
    var.paths: ["/var/log/nginx/access.log*"]
  error:
    enabled: true

    # Set custom paths for the log files. If left empty,
    # Filebeat will choose the paths depending on your OS.
    var.paths: ["/var/log/nginx/error.log*"]

```

输出发送到标准错误，而不是配置的日志输出

```BASH
filebeat setup -e
```

![image-20210815100932023](ELK.assets\image-20210815100932023.png)



![image-20210815102739126](ELK.assets\image-20210815102739126.png)

![image-20210815102754730](ELK.assets\image-20210815102754730.png)

![image-20210815102807961](ELK.assets\image-20210815102807961.png)

![image-20210815103002271](ELK.assets\image-20210815103002271.png)

![image-20210815103041723](ELK.assets\image-20210815103041723.png)

![image-20210815103235166](ELK.assets\image-20210815103235166.png)

完成

![image-20210815103344435](ELK.assets\image-20210815103344435.png)

![image-20210815103931699](ELK.assets\image-20210815103931699.png)

![image-20210815104407941](ELK.assets\image-20210815104407941.png)

![image-20210815105104183](ELK.assets\image-20210815105104183.png)

![image-20210815105138166](ELK.assets\image-20210815105138166.png)

![image-20210815105238255](ELK.assets\image-20210815105238255.png)

![image-20210815105326714](ELK.assets\image-20210815105326714.png)

![image-20210815105335340](ELK.assets\image-20210815105335340.png)

上面是添加kibana添加nginx默认日志。

把nginx 日志改成json的

```BASH
[root@redis01 ~]# vim /etc/nginx/nginx.conf
   log_format json '{ "time_local": "$time_local", '
                       '"remote_addr": "$remote_addr", '
                       '"referer": "$http_referer", '
                       '"request": "$request", '
                       '"status": $status, '
                       '"bytes": $body_bytes_sent, '
                       '"agent": "$http_user_agent", '
                       '"x_forwarded": "$http_x_forwarded_for", '
                       '"up_addr": "$upstream_addr",'
                       '"up_host": "$upstream_http_host",'
                       '"upstream_time": "$upstream_response_time",'
                       '"request_time": "$request_time"'
                       ' }';

    access_log  /var/log/nginx/access.log  json;
    
[root@redis01 ~]# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
[root@redis01 ~]# systemctl restart nginx
[root@redis01 ~]# ab -n 100 -c 100 172.16.0.210/
[root@redis01 ~]# tail /var/log/nginx/access.log 
{ "time_local": "15/Aug/2021:14:37:46 +0800", "remote_addr": "172.16.0.210", "referer": "-", "request": "GET / HTTP/1.0", "status": 200, "bytes": 4833, "agent": "ApacheBench/2.3", "x_forwarded": "-", "up_addr": "-","up_host": "-","upstream_time": "-","request_time": "0.000" }
```

https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-input-log.html 

filebeat log json 配置

```BASH
[root@redis01 ~]# vim /etc/filebeat/filebeat.yml
# ============================== Filebeat inputs ===============================
json.keys_under_root: true
json.overwrite_keys: true

[root@redis01 ~]# systemctl restart filebeat

```

清空es 数据库。或者再kibana上面直接清空

![image-20210815152846060](ELK.assets\image-20210815152846060.png)

然后重新创建一个kibana 。

![image-20210815153314910](ELK.assets\image-20210815153314910.png)

结果就像上图那样，要哪个信息就看那列信息，不像默认的都在message中

![image-20210815153613502](ELK.assets\image-20210815153613502.png)

这样筛选就更方便了。

filebeat OUT选项

https://www.elastic.co/guide/en/beats/filebeat/current/elasticsearch-output.html

```bash
index: "nginx_access-%{[agent.version]}-%{+yyyy.MM.dd}"
```

基本的操作已经完成了。里面涉及到很多知识官网都有解决方案。

测试过程中经常的删除index。不知道后没有什么影响。

![image-20210815173140164](ELK.assets\image-20210815173140164.png)

这个就是调试好之后的界面截图。

![image-20210815223232192](ELK.assets\image-20210815223232192.png)

下面配置成上面架构收集多台机器的日志

### 多台日志收集

redis02

```BASH
[root@redis02 ~]# yum install nginx httpd-tools -y
```

redis01

```BASH
[root@redis01 ~]# scp /etc/nginx/nginx.conf redis02:/etc/nginx/
[root@redis01 ~]# scp /etc/nginx/nginx.conf redis03:/etc/nginx/
[root@redis01 ~]# scp /server/soft/filebeat-7.14.0-x86_64.rpm redis02:/server/soft
[root@redis01 ~]# scp /server/soft/filebeat-7.14.0-x86_64.rpm redis03:/server/soft
```

```BASH
[root@redis02 ~]# rpm -ivh /server/soft/filebeat-7.14.0-x86_64.rpm 
```

redis03

```BASH
[root@redis03 ~]# yum install nginx httpd-tools -y
[root@redis03 ~]# rpm -ivh /server/soft/filebeat-7.14.0-x86_64.rpm 
```

```BASH
[root@redis01 ~]# scp /etc/filebeat/filebeat.yml redis02:/etc/filebeat/
[root@redis01 ~]# scp /etc/filebeat/filebeat.yml redis03:/etc/filebeat/
```

启动

```BASH
[root@redis02 ~]# nginx -t
[root@redis02 ~]# systemctl start nginx && systemctl enable nginx
[root@redis03 ~]# systemctl start nginx && systemctl enable nginx
systemctl start filebeat
systemctl enable filebeat

```

模拟数据

```BASH
[root@redis02 ~]# ab -n 100 -c 100 172.16.0.211/
[root@redis03 ~]# ab -n 100 -c 100 172.16.0.212/
```

![image-20210815230135987](ELK.assets\image-20210815230135987.png)

数据索引如果没有刷新出来，删除重试

kibana 重新添加

![image-20210815230827502](ELK.assets\image-20210815230827502.png)

![image-20210815230917315](ELK.assets\image-20210815230917315.png)

### 日志拆分

```BASH
[root@redis01 ~]# grep -Ev "#|^$" /etc/filebeat/filebeat.yml
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/nginx/access.log
  json.keys_under_root: true
  json.overwrite_keys: true
  tags: ["access"]             #打标签
  
- type: log
  enabled: true
  paths:
    - /var/log/nginx/error.log
  tags: ["error"]      #打标签
  
  
setup.template.settings:
  index.number_of_shards: 3
setup.kibana:
  host: "172.16.0.210:5601"
output.elasticsearch:
  hosts: ["172.16.0.210:9200"]
 
  indices:      #新建索引名称
    - index: "nginx_access-%{[agent.version]}-%{+yyyy.MM}"
      when.contains:
        tags: "access"
    - index: "nginx_error-%{[agent.version]}-%{+yyyy.MM}"
      when.contains:
        tags: "error"
processors:
  - add_host_metadata:
      when.not.contains.tags: forwarded
  - add_cloud_metadata: ~
  - add_docker_metadata: ~
  - add_kubernetes_metadata: ~
  
setup.template.name: "nginx"  #filebeat 模板的名称。
setup.template.pattern: "nginx-*"   # 应用于默认索引设置的模板模式。默认模式是 filebeat-* 。Filebeat 版本始终包含在模式中，因此最终的模式是 Filebeat -% {[ agent.version ]}-* 。通配符-* 用于匹配所有日指数。
setup.template.enabled: false  #false 禁用自动加载预设模板，需要手动加载模板
setup.template.overwrite: true  #s

```

![image-20210815233135881](ELK.assets\image-20210815233135881.png)

```BASH
[root@redis01 ~]# scp /etc/filebeat/filebeat.yml redis02:/etc/filebeat/ 
[root@redis01 ~]# scp /etc/filebeat/filebeat.yml redis03:/etc/filebeat/

systemctl restart filebeat
```

![image-20210815233854779](ELK.assets\image-20210815233854779.png)

kibana 中添加对应的index。

![image-20210815233928319](ELK.assets\image-20210815233928319.png)

上面是ELK 收集nginx 的日志。

### ELK收集tomcat

#### 安装tomcat,yum方式

```BASH
yum install tomcat tomcat-webapps tomcat-admin-webapps tomcat-docs-webapp tomcat-javadoc -y
[root@redis01 ~]# systemctl start tomcat
[root@redis01 ~]# netstat -lntup |grep 8080
tcp6       0      0 :::8080                 :::*                    LISTEN      2762/java           
[root@redis01 ~]# lsof -i:8080
COMMAND  PID   USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
java    2762 tomcat   38u  IPv6  29401      0t0  TCP *:webcache (LISTEN)

```

修改日志为json格式 （二进制的根据解压目录而定）

```BASH
[root@redis01 ~]# vim /etc/tomcat/server.xml  
137         <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
138                prefix="localhost_access_log." suffix=".txt"
139                pattern="{&quot;clientip&quot;:&quot;%h&quot;,&quot;ClientUser&quot;:&quot;%l&quot;,&quot;authenticated&quot;:&quot;%u&    quot;,&quot;AccessTime&quot;:&quot;%t&quot;,&quot;method&quot;:&quot;%r&quot;,&quot;status&quot;:&quot;%s&quot;,&quot;SendBytes&quot;:    &quot;%b&quot;,&quot;Query?string&quot;:&quot;%q&quot;,&quot;partner&quot;:&quot;%{Referer}i&quot;,&quot;AgentVersion&quot;:&quot;%{Us    er-Agent}i&quot;}"/>
```

查看日志格式

```BASH
[root@redis01 ~]# tail -f /var/log/tomcat/localhost_access_log.2021-08-16.txt 
{"clientip":"172.16.0.119","ClientUser":"-","authenticated":"-","AccessTime":"[16/Aug/2021:20:52:04 +0800]","method":"GET /favicon.ico HTTP/1.1","status":"200","SendBytes":"21630","Query?string":"","partner":"http://172.16.0.210:8080/","AgentVersion":"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/92.0.4515.107 Safari/537.36 Edg/92.0.902.62"}

```

![image-20210816205355838](ELK.assets\image-20210816205355838.png)

格式已经变成json格式了。

#### filebeat配置

```BASH
[root@redis01 ~]# grep -Ev '#|^$' /etc/filebeat/filebeat.yml
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/tomcat/localhost_access_log*
  json.keys_under_root: true
  json.overwrite_keys: true
setup.template.settings:
  index.number_of_shards: 3
setup.kibana:
  host: "172.16.0.210:5601"
output.elasticsearch:
  hosts: ["172.16.0.210:9200"]
  index: "tomcat-%{[agent.version]}-%{+yyyy.MM}"       
processors:
  - add_host_metadata:
      when.not.contains.tags: forwarded
  - add_cloud_metadata: ~
  - add_docker_metadata: ~
  - add_kubernetes_metadata: ~
setup.template.name: "tomcat"
setup.template.pattern: "tomcat-*"
setup.template.enabled: false
setup.template.overwrite: true

[root@redis01 ~]# systemctl restart filebeat
```

![image-20210816213156932](ELK.assets\image-20210816213156932.png)

日志拆分

```BASH
[root@redis01 ~]# grep -Ev '#|^$' /etc/filebeat/filebeat.yml
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/nginx/access.log
  json.keys_under_root: true
  json.overwrite_keys: true
  tags: ["access"]
- type: log
  enabled: true
  paths:
    - /var/log/nginx/error.log
  tags: ["error"]
- type: log
  enable: true
  paths: 
    - /var/log/tomcat/localhost_access_log*
  json.keys_under_root: true
  json.add_error_key: true
  tags: ["tomcat"]
setup.template.settings:
  index.number_of_shards: 3
setup.kibana:
  host: "172.16.0.210:5601"
output.elasticsearch:
  hosts: ["172.16.0.210:9200"]
  indices: 
   - index: "nginx_access-%{[agent.version]}-%{+yyyy.MM}"
     when.contains:
       tags: "access"
   - index: "nginx_error-%{[agent.version]}-%{+yyyy.MM}"
     when.contains:
       tags: "error"
   - index: "tomcat-%{[agent.version]}-%{+yyyy.MM}"
     when.contains:
       tags: "tomcat"
processors:
  - add_host_metadata:
      when.not.contains.tags: forwarded
  - add_cloud_metadata: ~
  - add_docker_metadata: ~
  - add_kubernetes_metadata: ~
setup.template.name: "tomcat"
setup.template.pattern: "tomcat-*"
setup.template.enabled: false
setup.template.overwrite: true
```

![image-20210816214832824](ELK.assets\image-20210816214832824.png)

重新创建kibana 数据

### java日志收集

https://www.elastic.co/guide/en/beats/filebeat/7.14/multiline-examples.html

因为java日志的输出信息非常多，需要将多行拼成一个事件，所以需要多行匹配模式

因为elasticsearch本身就是java开发的，所以我们可以直接收集ES的日志

```BASH
[root@redis01 ~]# tail -f /var/log/elasticsearch/elasticsearch.log 
[2021-08-16T21:59:44,801][ERROR][o.e.b.ElasticsearchUncaughtExceptionHandler] [node-1] uncaught exception in thread [main]
org.elasticsearch.bootstrap.StartupException: java.lang.RuntimeException: can not run elasticsearch as root
	at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:163) ~[elasticsearch-7.14.0.jar:7.14.0]
	at org.elasticsearch.bootstrap.Elasticsearch.execute(Elasticsearch.java:150) ~[elasticsearch-7.14.0.jar:7.14.0]
	at org.elasticsearch.cli.EnvironmentAwareCommand.execute(EnvironmentAwareCommand.java:75) ~[elasticsearch-7.14.0.jar:7.14.0]
	at org.elasticsearch.cli.Command.mainWithoutErrorHandling(Command.java:116) ~[elasticsearch-cli-7.14.0.jar:7.14.0]
	at org.elasticsearch.cli.Command.main(Command.java:79) ~[elasticsearch-cli-7.14.0.jar:7.14.0]
	at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:115) ~[elasticsearch-7.14.0.jar:7.14.0]
	at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:81) ~[elasticsearch-7.14.0.jar:7.14.0]
Caused by: java.lang.RuntimeException: can not run elasticsearch as root
	at org.elasticsearch.bootstrap.Bootstrap.initializeNatives(Bootstrap.java:103) ~[elasticsearch-7.14.0.jar:7.14.0]
	at org.elasticsearch.bootstrap.Bootstrap.setup(Bootstrap.java:170) ~[elasticsearch-7.14.0.jar:7.14.0]
	at org.elasticsearch.bootstrap.Bootstrap.init(Bootstrap.java:399) ~[elasticsearch-7.14.0.jar:7.14.0]
	at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:159) ~[elasticsearch-7.14.0.jar:7.14.0]
	... 6 more
uncaught exception in thread [main]
java.lang.RuntimeException: can not run elasticsearch as root
	at org.elasticsearch.bootstrap.Bootstrap.initializeNatives(Bootstrap.java:103)
	at org.elasticsearch.bootstrap.Bootstrap.setup(Bootstrap.java:170)
	at org.elasticsearch.bootstrap.Bootstrap.init(Bootstrap.java:399)
	at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:159)
	at org.elasticsearch.bootstrap.Elasticsearch.execute(Elasticsearch.java:150)
	at org.elasticsearch.cli.EnvironmentAwareCommand.execute(EnvironmentAwareCommand.java:75)
	at org.elasticsearch.cli.Command.mainWithoutErrorHandling(Command.java:116)
	at org.elasticsearch.cli.Command.main(Command.java:79)
	at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:115)
	at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:81)
For complete error details, refer to the log at /var/log/elasticsearch/elasticsearch.log
2021-08-16 13:59:45,176239 UTC [7739] INFO  Main.cc@106 Parent process died - ML controller exiting

```

很多日志是多行的，并没有明显的区别，那么我们提取日志难度就很困难

https://www.elastic.co/guide/en/beats/filebeat/7.14/multiline-examples.html#multiline-examples

参考elastic 多行信息管理

```BASH
multiline.type: pattern
multiline.pattern: '^\['
multiline.negate: true
multiline.match: after
```

时间戳案例

```BASH
[2015-08-24 11:49:14,389][INFO ][env                      ] [Letha] using [1] data paths, mounts [[/
(/dev/disk1)]], net usable_space [34.5gb], net total_space [118.9gb], types [hfs]
```

```BASH
multiline.type: pattern
multiline.pattern: '^\[[0-9]{4}-[0-9]{2}-[0-9]{2}'
multiline.negate: true
multiline.match: after
```

**filebeat配置文件**

```BASH
[root@redis01 ~]# grep -Ev '#|^$' /etc/filebeat/filebeat.yml
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/nginx/access.log
  json.keys_under_root: true
  json.overwrite_keys: true
  tags: ["access"]
- type: log
  enabled: true
  paths:
    - /var/log/nginx/error.log
  tags: ["error"]
- type: log
  enable: true
  paths: 
    - /var/log/tomcat/localhost_access_log*
  json.keys_under_root: true
  json.add_error_key: true
  tags: ["tomcat"]
- type: log
  enable: true
  paths:
    - /var/log/elasticsearch/elasticsearch.log
  multiline.type: pattern
  multiline.pattern: '^\[[0-9]{4}-[0-9]{2}-[0-9]{2}'
  multiline.negate: true
  multiline.match: after
  tags: ["es"]
setup.template.settings:
  index.number_of_shards: 3
setup.kibana:
  host: "172.16.0.210:5601"
output.elasticsearch:
  hosts: ["172.16.0.210:9200"]
  indices: 
   - index: "nginx_access-%{[agent.version]}-%{+yyyy.MM}"
     when.contains:
       tags: "access"
   - index: "nginx_error-%{[agent.version]}-%{+yyyy.MM}"
     when.contains:
       tags: "error"
   - index: "tomcat-%{[agent.version]}-%{+yyyy.MM}"
     when.contains:
       tags: "tomcat"
   - index: "es-%{[agent.version]}-%{+yyyy.MM}"
     when.contains:
       tags: "es"
processors:
  - add_host_metadata:
      when.not.contains.tags: forwarded
  - add_cloud_metadata: ~
  - add_docker_metadata: ~
  - add_kubernetes_metadata: ~
setup.template.name: "tomcat"
setup.template.pattern: "tomcat-*"
setup.template.enabled: false
setup.template.overwrite: true
```

![image-20210816221837353](ELK.assets\image-20210816221837353.png)

![image-20210816222030516](ELK.assets\image-20210816222030516.png)



### docker 日志收集

```BASH
# yum install -y yum-utils

# yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
    
# yum install docker-ce docker-ce-cli containerd.io
# yum list docker-ce --showduplicates | sort -r

安装指定的版本
# yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io

curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
wget -O /etc/yum.repos.d/docker-ce.repo https://mirrors.ustc.edu.cn/docker-ce/linux/centos/docker-ce.repo
sed -i 's#download.docker.com#mirrors.tuna.tsinghua.edu.cn/docker-ce#g' /etc/yum.repos.d/docker-ce.repo
[root@redis01 ~]# docker --version
Docker version 20.10.8, build 3967b7d

[root@redis01 ~]# systemctl start docker
[root@redis01 ~]# docker pull busybox
[root@redis01 ~]# docker images
[root@redis01 ~]# docker run -d --name busybox busybox:latest 
b6ca52367aa76978b6deb2d34e3769106fc9f4540dc6feff89f63207195e1e3c
[root@redis01 ~]# docker pull nginx

[root@redis01 ~]# docker inspect busybox|grep -w "Id"
        "Id": "b6ca52367aa76978b6deb2d34e3769106fc9f4540dc6feff89f63207195e1e3c",
[root@redis01 ~]# docker info|grep -w 'Docker Root'
 Docker Root Dir: /var/lib/docker

[root@redis01 ~]# docker run -itd --name nginx-test -p 80:80 nginx:latest 
b8f6b9e4fc62926a796d26f7c8d4b0fcd41597ad615a73c7e1b1ec3a8392a399
[root@redis01 ~]# cat /var/lib/docker/containers/b8f6b9e4fc62926a796d26f7c8d4b0fcd41597ad615a73c7e1b1ec3a8392a399/b8f6b9e4fc62926a796d26f7c8d4b0fcd41597ad615a73c7e1b1ec3a8392a399-json.log 
里面有访问数据了。
```

https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-input-docker.html

官网参考

filebeat文件配置，单个容器，这种是不适用的，容器是随时销毁的！

```BASH
filebeat.inputs:
- type: docker
  containers.ids: 
    - '8b6fe7dc9e067b58476dc57d6986dd96d7100430c5de3b109a99cd56ac655347'
```

假如我们有多个docker镜像或者重新提交了新镜像，那么直接指定ID的就不是太方便了。

我们从当前的容器提交一个新的镜像并且运行起来

```BASH
[root@redis01 ~]# docker ps -a
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS              PORTS                               NAMES
4deeaed68a57   nginx:latest   "/docker-entrypoint.…"   11 seconds ago   Up 8 seconds        0.0.0.0:81->80/tcp, :::81->80/tcp   nginx-test1
b8f6b9e4fc62   nginx:latest   "/docker-entrypoint.…"   11 hours ago     Up About a minute   0.0.0.0:80->80/tcp, :::80->80/tcp   nginx-test
```

```BASH
[root@redis01 ~]# grep -Ev '#|^$' /etc/filebeat/filebeat.yml
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/nginx/access.log
  json.keys_under_root: true
  json.overwrite_keys: true
  tags: ["access"]
- type: log
  enabled: true
  paths:
    - /var/log/nginx/error.log
  tags: ["error"]
- type: log
  enable: true
  paths: 
    - /var/log/tomcat/localhost_access_log*
  json.keys_under_root: true
  json.add_error_key: true
  tags: ["tomcat"]
- type: log
  enable: true
  paths:
    - /var/log/elasticsearch/elasticsearch.log
  multiline.type: pattern
  multiline.pattern: '^\[[0-9]{4}-[0-9]{2}-[0-9]{2}'
  multiline.negate: true
  multiline.match: after
  tags: ["es"]
- type: docker      #docker
  combine_partial: true 
  containers: 
    path: "/var/lib/docker/containers"
    stream: "stdout"
    ids:
      - "*"     #列出所有容器的日志
  tags: ["docker"]
setup.template.settings:
  index.number_of_shards: 3
setup.kibana:
  host: "172.16.0.210:5601"
output.elasticsearch:
  hosts: ["172.16.0.210:9200"]
  indices: 
   - index: "nginx_access-%{[agent.version]}-%{+yyyy.MM}"
     when.contains:
       tags: "access"
   - index: "nginx_error-%{[agent.version]}-%{+yyyy.MM}"
     when.contains:
       tags: "error"
   - index: "tomcat-%{[agent.version]}-%{+yyyy.MM}"
     when.contains:
       tags: "tomcat"
   - index: "es-%{[agent.version]}-%{+yyyy.MM}"
     when.contains:
       tags: "es"
   - index: "docker-%{[agent.version]}-%{+yyyy.MM}"
     when.contains:
       tags: "docker"
processors:
  - add_host_metadata:
      when.not.contains.tags: forwarded
  - add_cloud_metadata: ~
  - add_docker_metadata: ~
  - add_kubernetes_metadata: ~
setup.template.name: "tomcat"
setup.template.pattern: "tomcat-*"
setup.template.enabled: false
setup.template.overwrite: true
```

还可以通过docker-compose 打标签的形式来提取日志

![image-20210817102024502](ELK.assets\image-20210817102024502.png)

![image-20210817102147630](ELK.assets\image-20210817102147630.png)

其实系统自带的已经很不错了。如果数量很多需要打标签的使用下面的方法

```BASH
[root@redis01 ~]# rpm -qa python3-pip
python3-pip-9.0.3-8.el7.noarch
[root@redis01 ~]# pip3
pip3    pip3.6  

加速
[root@redis01 ~]# pip3 install -i https://pypi.tuna.tsinghua.edu.cn/simple pip -U
# pip3 config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple

[root@redis01 ~]# pip3 install docker-compose

[root@redis01 ~]# docker-compose version
docker-compose version 1.29.2, build unknown
docker-py version: 5.0.0
CPython version: 3.6.8
OpenSSL version: OpenSSL 1.0.2k-fips  26 Jan 2017

```

编写一个docker-compose.yml

```bash
cat docker-compose.yml 
version: '3'
services:
  nginx:
    image: nginx:v2
    # 设置labels
    labels:
      service: nginx
    # logging设置增加labels.service
    logging:
      options:
        labels: "service"
    ports:
      - "8080:80"
  db:
    image: nginx:latest
    # 设置labels
    labels:
      service: db 
    # logging设置增加labels.service
    logging:
      options:
        labels: "service"
    ports:
      - "80:80"
```

配置filebeat

```BASH
cat /etc/filebeat/filebeat.yml    
filebeat.inputs:
- type: log
  enabled: true 
  paths:
    - /var/lib/docker/containers/*/*-json.log
  json.keys_under_root: true
  json.overwrite_keys: true
output.elasticsearch:
  hosts: ["172.16.0.210:9200"]
  indices:
    - index: "docker-nginx-%{[beat.version]}-%{+yyyy.MM.dd}"
      when.contains:
        attrs.service: "nginx"
    - index: "docker-db-%{[beat.version]}-%{+yyyy.MM.dd}"
      when.contains:
        attrs.service: "db"
setup.template.name: "docker"
setup.template.pattern: "docker-*"
setup.template.enabled: false
setup.template.overwrite: true
```

> 如果对docker 熟悉，和kubernetes属性那么这么做真的不怎么样。elasticsearch中可以识别container.name的，我们通过条件就可以查找到对应的日志。通过标签可以查看批量的日志。

针对filebeat通过服务类型创建不同类型的索引已经在上面实例给出

docker日志已经是json格式了。

## Filebeat modules配置

https://www.elastic.co/guide/en/beats/filebeat/current/configuration-filebeat-modules.html

查看当前系统使用了哪些模块

```BASH
[root@redis01 ~]# filebeat modules list
# filebeat modules enable system nginx mysql  #开启模块
# filebeat.yml
filebeat.modules:
- module: nginx
- module: mysql
- module: system
```

https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-modules.html  所有模块配置

nginx 模块设置：https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-module-nginx.html

```BASH
[root@redis01 ~]# vim /etc/filebeat/modules.d/nginx.yml 
[root@redis01 ~]# grep -Ev '#|^$' /etc/filebeat/modules.d/nginx.yml 
- module: nginx
  access:
    enabled: true
    var.paths: ["/var/log/nginx/access.log*"]
  error:
    enabled: true
    var.paths: ["/var/log/nginx/error.log*"]

```

问题：

```BASH
[root@redis01 ~]# filebeat modules list
Error in modules manager: modules management requires 'filebeat.config.modules.path' setting

[root@redis01 ~]# vim /etc/filebeat/filebeat.yml
...
filebeat.config.modules:
  # Glob pattern for configuration loading
  path: ${path.config}/modules.d/*.yml
  reload.enabled: true
  
output.elasticsearch:
# Array of hosts to connect to.
hosts: ["172.16.0.210:9200"]
indices:
 - index: "nginx_access-%{[agent.version]}-%{+yyyy.MM}"
   when.contains:
     fileset.name: "access"
 - index: "nginx_error-%{[agent.version]}-%{+yyyy.MM}"
   when.contains:
     fileset.name: "error"                                                                                                             
为什么使用fileset.name？看下面数据

... 
```

![image-20210819212824346](ELK.assets\image-20210819212824346.png)

![image-20210819212926350](ELK.assets\image-20210819212926350.png)



![image-20210819212949044](ELK.assets\image-20210819212949044.png)

这里的不是json格式

如何让转json呢？

https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-module-nginx.html

```bash
[root@redis01 ~]# vim /etc/nginx/nginx.conf

    access_log  /var/log/nginx/access.log  main;  


[root@redis01 ~]# grep -Ev '#|^$' /etc/filebeat/modules.d/nginx.yml 
- module: nginx
  access:
    enabled: true
    var.paths: ["/var/log/nginx/access.log*"]
  error:
    enabled: true
    var.paths: ["/var/log/nginx/error.log*"]
  ingress_controller:
    enabled: false
```

![image-20210819220929435](ELK.assets\image-20210819220929435.png)

---

kibana 里面重新添加就出现上面的

删除索引   

 ```bash
 [root@redis01 ~]# ab -n 20 -c 20 172.16.0.210/
 模拟日志数据
 ```

## kibana 视图

![image-20210819223613770](ELK.assets\image-20210819223613770.png)

画图，数据要充足才可以

画好之后

![image-20210819223714844](ELK.assets\image-20210819223714844.png)

这就可以出图，大屏显示了。

![image-20210819223804299](ELK.assets\image-20210819223804299.png)

## 使用缓存收集日志

当日志的数量非常多的时候，可能需要引入缓存层作为临时存储数据的地方，防止因为ES处理不过来导致日志丢失的情况。

filebeat支持将日志发送到redis或者kafka作为消息队列缓存。

但是使用了缓存层，就不能使用模版来配置日志收集了。

所以最好日志是json格式

### 使用单台redis作为缓存

这里需要说明，如果使用redis作为缓存

可以将不同的日志类型单独写成一个键，这样好处是清晰，但是缺点是logstash写起来起来复杂

也可以将所有的日志全部写入到一个键中，然后靠后端的logstash去过滤处理。

安装部署redis

这里机器已经安装了

```BASH
[root@redis01 ~]# redis-server /opt/redis_cluster/redis_6379/conf/redis_6379.conf 
[root@redis01 ~]# ps -ef |grep redis
root       9660      1  9 22:45 ?        00:00:01 redis-server 172.16.0.210:6379
root       9679   1612  0 22:46 pts/0    00:00:00 grep --color=auto redis
[root@redis01 ~]# redis-cli -h redis01
redis01:6379> FLUSHALL
OK
redis01:6379> keys *
(empty array)
#清空数据
redis01:6379> set k1 v1
OK
redis01:6379> del k1
(integer) 1
redis01:6379> keys *
(empty array)

```

配置filebeat

```BASH
[root@redis01 ~]# vim /etc/nginx/nginx.conf
     log_format json '{ "time_local": "$time_local", '
                       '"remote_addr": "$remote_addr", '
                       '"referer": "$http_referer", '
                       '"request": "$request", '
                       '"status": $status, '
                       '"bytes": $body_bytes_sent, '
                       '"agent": "$http_user_agent", '
                       '"x_forwarded": "$http_x_forwarded_for", '
                       '"up_addr": "$upstream_addr",'
                       '"up_host": "$upstream_http_host",'
                       '"upstream_time": "$upstream_response_time",'
                       '"request_time": "$request_time"'
                       ' }';

    access_log  /var/log/nginx/access.log  json;      
配置json格式    
    
filebeat分key建索引
[root@redis01 ~]# vim /etc/filebeat/filebeat.yml
[root@redis01 ~]# grep -Ev '#|^$' /etc/filebeat/filebeat.yml
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/nginx/access.log
  json.keys_under_root: true
  json.overwrite_keys: true
  tags: ["access"]
- type: log
  enabled: true
  paths:
    - /var/log/nginx/error.log
  tags: ["error"]
- type: log
  enable: true
  paths: 
    - /var/log/tomcat/localhost_access_log*
  json.keys_under_root: true
  json.add_error_key: true
  tags: ["tomcat"]
- type: log
  enable: true
  paths:
    - /var/log/elasticsearch/elasticsearch.log
  multiline.type: pattern
  multiline.pattern: '^\[[0-9]{4}-[0-9]{2}-[0-9]{2}'
  multiline.negate: true
  multiline.match: after
  tags: ["es"]
- type: docker
  combine_partial: true 
  containers: 
    path: "/var/lib/docker/containers"
    stream: "stdout"
    ids:
      - "*"
  tags: ["docker"]
setup.template.settings:
  index.number_of_shards: 3
setup.kibana:
  host: "172.16.0.210:5601"
output.elasticsearch:
  hosts: ["172.16.0.210:9200"]
  indices: 
   - index: "nginx_access-%{[agent.version]}-%{+yyyy.MM}"
     when.contains:
       tags: "access"
   - index: "nginx_error-%{[agent.version]}-%{+yyyy.MM}"
     when.contains:
       tags: "error"
   - index: "tomcat-%{[agent.version]}-%{+yyyy.MM}"
     when.contains:
       tags: "tomcat"
   - index: "es-%{[agent.version]}-%{+yyyy.MM}"
     when.contains:
       tags: "es"
   - index: "docker-%{[agent.version]}-%{+yyyy.MM}"
     when.contains:
       tags: "docker"
processors:
  - add_host_metadata:
      when.not.contains.tags: forwarded
  - add_cloud_metadata: ~
  - add_docker_metadata: ~
  - add_kubernetes_metadata: ~
setup.template.name: "tomcat"
setup.template.pattern: "tomcat-*"
setup.template.enabled: false
setup.template.overwrite: true
    
```

配置logstash 读取不同的key

```BASH
[root@redis01 ~]# vim /etc/logstash/conf.d/redis.conf

input {
  redis {
    host => "172.16.0.210"
    port => "6379"
    db => "0"
    key => "nginx_access"
    data_type => "list"
  }
  redis {
    host => "172.16.0.210"
    port => "6379"
    db => "0"
    key => "nginx_error"
    data_type => "list"
  }
}

filter {
  mutate {
    convert => ["upstream_time", "float"]
    convert => ["request_time", "float"]
  }
}

output {
    stdout {}
    if "access" in [tags] {
      elasticsearch {
        hosts => "http://172.16.0.210:9200"
        manage_template => false
        index => "nginx_access-%{+yyyy.MM.dd}"
      }
    }
    if "error" in [tags] {
      elasticsearch {
        hosts => "http://172.16.0.210:9200"
        manage_template => false
        index => "nginx_error-%{+yyyy.MM.dd}"
      }
    }
}

```

```BASH
[root@redis01 ~]# systemctl restart logstash
[root@redis01 ~]# grep -Ev '#|^$' /etc/filebeat/filebeat.yml

output.redis:
  hosts: ["172.16.0.210"]
  key: "filebeat"

[root@redis01 ~]# ab -n 10000 -c 20 172.16.0.210/

redis01:6379> KEYS *
1) "filebeat"
redis01:6379> llen filebeat
(integer) 20001
```

```BASH
[root@redis01 ~]# vim /etc/logstash/conf.d/redis.conf 
input {
  redis {
    host => "172.16.0.210"
    port => "6379"
    db => "0"
    key => "filebeat"
    data_type => "list"
  }
}

filter {
  mutate {
    convert => ["upstream_time", "float"]
    convert => ["request_time", "float"]
  }
}

output {
    if "access" in [tags] {
      elasticsearch {
        hosts => "http://172.16.0.210:9200"
        manage_template => false
        index => "nginx_access-%{+yyyy.MM.dd}"
      }
    }
    if "error" in [tags] {
      elasticsearch {
        hosts => "http://172.16.0.210:9200"
        manage_template => false
        index => "nginx_error-%{+yyyy.MM.dd}"
      }                                                                                                                                  
    }
}

[root@redis01 ~]# systemctl restart logstash.service 

```

```bash
[root@redis01 ~]# ab -n 10000 -c 20 172.16.0.210/


redis01:6379> FLUSHALL  #清空数据
OK
redis01:6379> llen filebeat
(integer) 0
redis01:6379> llen filebeat
(integer) 0
redis01:6379> llen filebeat
(integer) 2037
redis01:6379> llen filebeat
(integer) 5664
redis01:6379> llen filebeat
(integer) 9760
redis01:6379> llen filebeat
(integer) 10000
redis01:6379> llen filebeat
(integer) 10000
redis01:6379> llen filebeat
(integer) 10000
redis01:6379> llen filebeat
(integer) 10000
redis01:6379> llen filebeat
(integer) 9875
redis01:6379> llen filebeat
(integer) 8876
redis01:6379> llen filebeat
(integer) 8751
redis01:6379> llen filebeat
(integer) 0

```

![image-20210819233208706](ELK.assets\image-20210819233208706.png)

重新添加

![image-20210819233227469](ELK.assets\image-20210819233227469.png)

![image-20210819234036632](ELK.assets\image-20210819234036632.png)

拓扑结构图就是当外面访问量大的时候，redis加速缓存，之后通过logstash写入到ES

单点redis   开启 aof rdb混用

## 使用nginx+keeplived 代理多台redis

上面我们实验了单台redis作为收集日志的缓存层，但是单台redis存在一个问题，就是单点故障

虽然可以做持久化处理，但是加入服务器坏掉或者修复时间未知的情况下还是有可能会丢数据。

redis集群方案有哨兵和集群，但可惜的是filebeat和logstash都不支持这两种方案。

不过利用我们目前所学的知识完全可以解决这个问题，虽然不是彻底解决，但是已经很好了

方案如下：

1.使用nginx+keepalived反向代理负载均衡到后面的多台redis

2.考虑到redis故障切换中数据一致性的问题，所以最好我们只使用2台redis,并且只工作一台，另外一台作为backup，只有第一台坏掉后，第二台才会工作。

3.filebeat的oputut的redis地址为keepalived的虚拟IP

4.logstash可以启动多个节点来加速读取redis的数据

5.后端可以采用多台es集群来做支撑

![image-20210820205218127](ELK.assets\image-20210820205218127.png)

### nginx

部署nginx 四层负载均衡(两台一样的操作)

```bash
# useradd -s /sbin/nologin -M nginx
[root@redis02 /server/soft]# wget http://nginx.org/download/nginx-1.20.1.tar.gz
[root@redis02 /server/soft]# tar xf nginx-1.20.1.tar.gz -C /opt/
[root@redis02 /server/soft]# cd /opt/nginx-1.20.1/
[root@redis02 /opt/nginx-1.20.1]# yum install -y pcre pcre-devel openssl openssl-devel gcc-c++
[root@redis02 /opt/nginx-1.20.1]# ./configure  \
--user=nginx \
--group=nginx \
--prefix=/usr/share/nginx \
--sbin-path=/usr/local/bin \
--conf-path=/etc/nginx/nginx.conf \
--error-log-path=/var/log/nginx/error.log \
--http-log-path=/var/log/nginx/access.log \
--pid-path=/var/run/nginx.pid \
--with-http_stub_status_module \
--with-http_ssl_module \
--with-stream 
[root@redis02 /opt/nginx-1.20.1]# make && make install
```

```bash
[root@redis02 /opt/nginx-1.20.1]# ln -s /opt/nginx-1.20.1 /opt/nginx

[root@redis02 /opt/nginx-1.20.1]# vim /etc/nginx/nginx.conf
user  nginx;
worker_processes  auto;

stream {

	upstream redis {
		server 172.16.0.211:6379 max_fails=2 fail_timeout=10s;
		server 172.16.0.212:6379 max_fails=2 fail_timeout=10s;  
						   }
				    
	server {
		listen 6388; 
		proxy_connect_timeout 1s;
        proxy_timeout 3s;
		proxy_pass redis;
		   }
}

```

```bash
cat >/etc/systemd/system/nginx.service<<EOF 
[Unit]
Description=Nginx Server
Documentation=http://nginx.org
After=network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
ExecStartPre=/usr/local/bin/nginx -t
ExecStart=/usr/local/bin/nginx
PIDFile=/var/run/nginx.pid 
ExecReload=/usr/local/bin/nginx -s reload
ExecStop=/usr/local/bin/nginx  -s stop
PrivateTmp=true
[Install]
WantedBy=multi-user.target
EOF

[root@redis02 /opt/nginx-1.20.1]# scp /etc/nginx/nginx.conf redis03:/etc/nginx/
[root@redis02 /opt/nginx-1.20.1]# scp /etc/systemd/system/nginx.service redis03:/etc/systemd/system/

systemctl daemon-reload && systemctl start nginx && systemctl enable nginx

```

### keepalive

```bash
yum install -y keepalived
```

```bash
[root@master01 /opt/nginx-1.20.1]# cat > /etc/keepalived/keepalived.conf <<EOF
! Configuration File for keepalived

global_defs {
   router_id NGINX_MASTER
}

#check_nginx
vrrp_script check_nginx {
		    script "/etc/keepalived/check_nginx.sh"
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
        172.16.0.213/24
    }

#track_script
    track_script {
		        check_nginx
				} 

}
EOF
```

```bash
[root@master01 /opt/nginx-1.20.1]# cat /etc/keepalived/check_nginx.sh 
#!/bin/bash
count=$(ps -ef|grep -c [n]ginx)
if [ "$count" -le 2 ];then
    systemctl stop keepalived
fi

chmod +x /etc/keepalived/check_nginx.sh

[root@redis02 ~]# scp -r /etc/keepalived redis03:/etc/

```

```bash
[root@redis03 /opt/nginx-1.20.1]# vim /etc/keepalived/keepalived.conf 
! Configuration File for keepalived
global_defs {
   router_id LVS_BACKUP
}

#check_nginx 

vrrp_script check_nginx {
   script "/etc/keepalivead/check_nginx.sh"
}

vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        172.16.0.213/24
    }
#track_script
track_script {
      check_nginx                                                                                                                        
    }
}

[root@redis03 /opt/nginx-1.20.1]# ll /etc/keepalived/
total 8
-rwxr-xr-x 1 root root 104 Aug 20 10:16 check_nginx.sh
-rw-r--r-- 1 root root 464 Aug 20 10:17 keepalived.conf

systemctl start keepalived && systemctl enable keepalived

[root@redis02 ~]# ip a s eth0|grep 213
    inet 172.16.0.213/24 scope global secondary eth0

```

### redis配置

```bash
[root@redis02 ~]# grep -Ev "#|^$"  /opt/redis_cluster/redis_6379/conf/redis_6379.conf 
bind 172.16.0.211   #VIP地址
protected-mode yes
port 6379     #端口
tcp-backlog 511
timeout 0
tcp-keepalive 300
daemonize yes
pidfile "/opt/redis_cluster/redis_6379/pid/redis_6379.pid"
loglevel notice
logfile "/opt/redis_cluster/redis_6379/logs/redis_6379.log"
databases 16
always-show-logo no
set-proc-title yes
proc-title-template "{title} {listen-addr} {server-mode}"
save 3600 1
save 300 100
save 60 10000
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
dbfilename "redis_6379.rdb"
rdb-del-sync-files no
dir "/data/redis_cluster/redis_6379"
replica-serve-stale-data yes
replica-read-only yes
repl-diskless-sync no
repl-diskless-sync-delay 5
repl-diskless-load disabled
repl-disable-tcp-nodelay no
replica-priority 0
acllog-max-len 128
requirepass "123456"
lazyfree-lazy-eviction no
lazyfree-lazy-expire no
lazyfree-lazy-server-del no
replica-lazy-flush no
lazyfree-lazy-user-del no
lazyfree-lazy-user-flush no
oom-score-adj no
oom-score-adj-values 0 200 800
disable-thp yes
appendonly yes
appendfilename "appendonly.aof"
appendfsync no
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
aof-load-truncated yes
aof-use-rdb-preamble yes
lua-time-limit 5000
slowlog-log-slower-than 10000
slowlog-max-len 128
latency-monitor-threshold 0
notify-keyspace-events ""
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
list-max-ziplist-size -2
list-compress-depth 0
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
hll-sparse-max-bytes 3000
stream-node-max-bytes 4kb
stream-node-max-entries 100
activerehashing yes
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit replica 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
hz 10
dynamic-hz yes
aof-rewrite-incremental-fsync yes
rdb-save-incremental-fsync yes
jemalloc-bg-thread yes
user default on nopass sanitize-payload ~* &* +@all


```

```bash
[root@redis03 ~]# vim /opt/redis_cluster/redis_6379/conf/redis_6379.conf 
bind 172.16.0.212
port 6379   
```

启动redis

```BASH
[root@redis02 ~]# redis-server /opt/redis_cluster/redis_6379/conf/redis_6379.conf 
[root@redis03 ~]# redis-server /opt/redis_cluster/redis_6379/conf/redis_6379.conf 
```

```BASH
[root@redis01 ~]# redis-cli -h 172.16.0.213 -p 6388
172.16.0.213:6388> KEYS *
1) "redis02"
172.16.0.213:6388> 
[root@redis01 ~]# redis-cli -h 172.16.0.213 -p 6388
172.16.0.213:6388> KEYS *
1) "redis03"

```

测试，负载均衡正常。但是这里是有问题的。redis写数据的问题，在nginx.conf 中212这台设置backup



```BASH
[root@redis01 ~]# vim /etc/filebeat/filebeat.yml
output.redis:
  hosts: ["172.16.0.213:6388"]
  key: "filebeat"
```

```bash
[root@redis02 ~]# ps -ef|grep [r]edis
root      46933      1  0 13:05 ?        00:00:00 redis-server 172.16.0.211:6379

[root@redis03 ~]# ps -ef|grep [r]edis
root       1475      1  0 13:05 ?        00:00:00 redis-server 172.16.0.212:6379

------------------------------------------------------
[root@redis02 ~]# redis-cli -h 172.16.0.213 -p 6388
172.16.0.213:6388> SLAVEOF no one
OK
172.16.0.213:6388> FLUSHALL
OK
```

```bash
[root@redis02 ~]# netstat -lntup
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      2068/nginx: master  
tcp        0      0 0.0.0.0:6388            0.0.0.0:*               LISTEN      2068/nginx: master  
tcp        0      0 172.16.0.211:6379       0.0.0.0:*               LISTEN      46933/redis-server  

[root@redis03 ~]# netstat -lntup
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 172.16.0.212:6379       0.0.0.0:*               LISTEN      1475/redis-server 1 
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      998/nginx: master p 
tcp        0      0 0.0.0.0:6388            0.0.0.0:*               LISTEN      998/nginx: master p 

```



### filebeat

```BASH
[root@redis01 ~]# grep -Ev "#|^$" /etc/filebeat/filebeat.yml
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/nginx/access.log
  json.keys_under_root: true
  json.overwrite_keys: true
  tags: ["access"]

- type: log
  enabled: true
  paths:
    - /var/log/nginx/error.log
  tags: ["error"]

setup.kibana:
  host: "172.16.0.210:5601"

output.redis:
  hosts: ["172.16.0.213:6388"]                                                                  
  key: "filebeat" 
```



### logstash

```bash
[root@redis01 ~]# cat /etc/logstash/conf.d/redis.conf 
input {
  redis {
    host => "172.16.0.213"
    port => "6388"
    db => "0"
    key => "filebeat"
    data_type => "list"
  }
}

filter {
  mutate {
    convert => ["upstream_time", "float"]
    convert => ["request_time", "float"]
  }
}

output {
    if "access" in [tags] {
      elasticsearch {
        hosts => "http://172.16.0.210:9200"
        manage_template => false
        index => "nginx_access-%{+yyyy.MM.dd}"
      }
    }
    if "error" in [tags] {
      elasticsearch {
        hosts => "http://172.16.0.210:9200"
        manage_template => false
        index => "nginx_error-%{+yyyy.MM.dd}"
      }
    }
}

[root@redis01 ~]# systemctl start logstash
等价
[root@redis01 ~]# /usr/share/logstash/bin/logstash -f /etc/logstash/conf.d/redis.conf 
```

当两台redis负载均衡的时候，数据只会提交其中一台redis的，另外一台的需要对方停止才会提交数据到ES

这样的数据就不准确。解决方法

```bash
[root@redis02 ~]# vim /etc/nginx/nginx.conf                
 stream {

        upstream redis {
                server 172.16.0.211:6379 max_fails=2 fail_timeout=10s;
                server 172.16.0.212:6379 max_fails=2 fail_timeout=10s backup;
                                                   }                                                                                     

        server {
                listen 6388;
                proxy_connect_timeout 1s;
                proxy_timeout 3s;
                proxy_pass redis;
                   }
}
             
```

![image-20210820162706803](ELK.assets\image-20210820162706803.png)

小结：这个高用用方案是保证redis加速的前提。实际生产中节点最可靠就是双节点，kibana 是单节点就可以。



## kibana的x-pack监控开启

<img src="ELK.assets\image-20210820145546205.png" alt="image-20210820145546205" style="zoom:150%;" />





```BASH
[root@redis01 ~]# vim /etc/filebeat/filebeat.yml
monitoring.enabled: true

```

![image-20210820165900142](ELK.assets\image-20210820165900142.png)

![image-20210820170025701](ELK.assets\image-20210820170025701.png)



# kafka 

Kafka 被称为下一代分布式消息系统，是非营利性组织ASF(Apache Software Foundation，简称为ASF)基金会中的一个开源项目，比如HTTP Server、Hadoop、ActiveMQ、Tomcat等开源软件都属于Apache基金会的开源软件，类似的消息系统还有RbbitMQ、ActiveMQ、ZeroMQ，最主要的优势是其具备分布式功能、并且结合zookeeper可以实现动态扩容。

官网：http://kafka.apache.org/

![image-20210820210317435](ELK.assets\image-20210820210317435.png)

## 安装环境准备

```BASH
[root@redis01 ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
172.16.0.210 redis01
172.16.0.211 redis02
172.16.0.212 redis03

```

软件下载地址

zookeeper:http://zookeeper.apache.org/releases.html

kafka:http://kafka.apache.org/downloads.html

```BASH
[root@redis01 /server/soft]# wget https://mirrors.bfsu.edu.cn/apache/zookeeper/zookeeper-3.7.0/apache-zookeeper-3.7.0-bin.tar.gz

[root@redis01 /server/soft]# wget https://mirrors.tuna.tsinghua.edu.cn/apache/kafka/2.8.0/kafka_2.13-2.8.0.tgz

```

### 安装zookeeper

zookeeper集群特性:整个集群中只要有超过集群数量一半的zookeeper工作是正常的,那么整个集群对外就是可用的,例如有2台服务器做了一个zaookeeper,只要有任何一台故障或宕机,那么这个zookeeper集群就是不可用的了.因为剩下的一台没有超过集群的一半的数量,但是假如有三台zookeeper组成一个集群,那么损坏一台还剩两台,大于3台的一半,所以损坏一台还是可以正常运行的,但是再损坏一台就只剩下一台,集群就不可用了.

 如果是4台组成,损坏一台正常,损坏两台还剩两台,不满足集群总数的一半,所以3台的集群和4台的集群算坏两台的结果都是集群不可用.所以这也是为什么集群一般是奇数的原因.

把所有的软件包都上传到/server/soft目录,所有节点都操作

```bash
[root@redis01 /server/soft]# scp apache-zookeeper-3.7.0-bin.tar.gz kafka_2.13-2.8.0.tgz redis02:/server/soft/

[root@redis01 /server/soft]# scp apache-zookeeper-3.7.0-bin.tar.gz kafka_2.13-2.8.0.tgz redis03:/server/soft/
```

确认java环境

```BASH
rpm -ivh /server/soft/jdk-16.0.2_linux-x64_bin.rpm 
java --version
```

### install zookeeper

#### redis01

```BASH
[root@redis01 /server/soft]# tar xf apache-zookeeper-3.7.0-bin.tar.gz -C /opt
[root@redis01 /server/soft]# ln -s /opt/apache-zookeeper-3.7.0-bin /opt/zookeeper
[root@redis01 /server/soft]# mkdir  -p /data/zookeeper
[root@redis01 /server/soft]# cp /opt/zookeeper/conf/zoo_sample.cfg /opt/zookeeper/conf/zoo.cfg
[root@redis01 /server/soft]# vim /opt/zookeeper/conf/zoo.cfg 
[root@redis01 /server/soft]# grep -Ev '#|^$' /opt/zookeeper/conf/zoo.cfg 
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/data/zookeeper
clientPort=2181
server.1=172.16.0.210:2888:3888
server.2=172.16.0.211:2888:3888
server.3=172.16.0.212:2888:3888
[root@redis01 /server/soft]# echo "1" > /data/zookeeper/myid
```

#### redis02

```BASH
[root@redis02 ~]# cd /server/soft/
[root@redis02 /server/soft]# tar xf apache-zookeeper-3.7.0-bin.tar.gz -C /opt
[root@redis02 /server/soft]# ln -s /opt/apache-zookeeper-3.7.0-bin /opt/zookeeper
[root@redis02 /server/soft]# mkdir  -p /data/zookeeper
[root@redis02 /server/soft]# cp /opt/zookeeper/conf/zoo_sample.cfg /opt/zookeeper/conf/zoo.cfg
[root@redis02 /server/soft]# vim /opt/zookeeper/conf/zoo.cfg 
[root@redis02 /server/soft]# grep -Ev '#|^$' /opt/zookeeper/conf/zoo.cfg 

[root@redis02 /server/soft]# grep '^[a-Z]' /opt/zookeeper/conf/zoo.cfg 
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/data/zookeeper
clientPort=2181
server.1=172.16.0.210:2888:3888
server.2=172.16.0.211:2888:3888
server.3=172.16.0.212:2888:3888

[root@redis02 /server/soft]# echo "2" > /data/zookeeper/myid
```

#### redis03

```bash
[root@redis03 ~]# cd /server/soft/
[root@redis03 /server/soft]# tar xf apache-zookeeper-3.7.0-bin.tar.gz -C /opt
[root@redis03 /server/soft]# ln -s /opt/apache-zookeeper-3.7.0-bin /opt/zookeeper
[root@redis03 /server/soft]# mkdir  -p /data/zookeeper
[root@redis03 /server/soft]# cp /opt/zookeeper/conf/zoo_sample.cfg /opt/zookeeper/conf/zoo.cfg
[root@redis03 /server/soft]# vim /opt/zookeeper/conf/zoo.cfg 
[root@redis03 /server/soft]# grep '^[a-Z]' /opt/zookeeper/conf/zoo.cfg 
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/data/zookeeper
clientPort=2181
server.1=172.16.0.210:2888:3888
server.2=172.16.0.211:2888:3888
server.3=172.16.0.212:2888:3888

[root@redis03 /server/soft]# echo "3" > /data/zookeeper/myid
```

启动

```BASH
[root@redis01 /server/soft]# /opt/zookeeper/bin/zkServer.sh start

[root@redis02 ~]# /opt/zookeeper/bin/zkServer.sh start

[root@redis03 ~]# /opt/zookeeper/bin/zkServer.sh start

```

查看状态

```BASH
# /opt/zookeeper/bin/zkServer.sh status
/usr/bin/java
ZooKeeper JMX enabled by default
Using config: /opt/zookeeper/bin/../conf/zoo.cfg
Client port found: 2181. Client address: localhost. Client SSL: false.
Mode: leader

```

#### zookeeper 简单操作命令

连接到节点1生成数据

```BASH
[root@redis01 ~]# /opt/zookeeper/bin/zkCli.sh -server 172.16.0.210:2181
[zk: 172.16.0.210:2181(CONNECTED) 0] create /test "hello"
Created /test
```

在节点2上验证

```BASH
[root@redis02 ~]# /opt/zookeeper/bin/zkCli.sh -server 172.16.0.211:2181
[zk: 172.16.0.211:2181(CONNECTED) 0] get /test
hello

[root@redis03 ~]# /opt/zookeeper/bin/zkCli.sh -server 172.16.0.212:2181
[zk: 172.16.0.212:2181(CONNECTED) 0] get /test
hello

```

## 安装并测试kafka

redis01

```BASH
[root@redis01 ~]# cd /server/soft/
[root@redis01 /server/soft]# tar xf kafka_2.13-2.8.0.tgz -C /opt       
[root@redis01 /server/soft]# ln -s /opt/kafka_2.13-2.8.0 /opt/kafka
[root@redis01 /server/soft]# mkdir /opt/kafka/logs
[root@redis01 /server/soft]# vim /opt/kafka/config/server.properties 
[root@redis01 /server/soft]# grep '^[a-Z]' /opt/kafka/config/server.properties 
broker.id=1
listeners=PLAINTEXT://172.16.0.210:9092
num.network.threads=3
num.io.threads=8
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
socket.request.max.bytes=104857600
log.dirs=/opt/kafka/logs
num.partitions=1
num.recovery.threads.per.data.dir=1
offsets.topic.replication.factor=1
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1
log.retention.hours=24
log.segment.bytes=1073741824
log.retention.check.interval.ms=300000
zookeeper.connect=172.16.0.210:2181,172.16.0.211:2181,172.16.0.212:2181
zookeeper.connection.timeout.ms=18000
group.initial.rebalance.delay.ms=0
```

redis02

```BASH
[root@redis02 ~]# cd /server/soft/
[root@redis02 /server/soft]# tar xf kafka_2.13-2.8.0.tgz -C /opt       
[root@redis02 /server/soft]# ln -s /opt/kafka_2.13-2.8.0 /opt/kafka
[root@redis02 /server/soft]# mkdir /opt/kafka/logs
[root@redis02 /server/soft]# vim /opt/kafka/config/server.properties 
[root@redis02 /server/soft]# grep '^[a-Z]' /opt/kafka/config/server.properties 
broker.id=2
listeners=PLAINTEXT://172.16.0.211:9092
num.network.threads=3
num.io.threads=8
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
socket.request.max.bytes=104857600
log.dirs=/opt/kafka/logs
num.partitions=1
num.recovery.threads.per.data.dir=1
offsets.topic.replication.factor=1
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1
log.retention.hours=24
log.segment.bytes=1073741824
log.retention.check.interval.ms=300000
zookeeper.connect=172.16.0.210:2181,172.16.0.211:2181,172.16.0.212:2181
zookeeper.connection.timeout.ms=18000
group.initial.rebalance.delay.ms=0
```

redis03

```BASH
[root@redis03 ~]# cd /server/soft/
[root@redis03 /server/soft]# tar xf kafka_2.13-2.8.0.tgz -C /opt       
[root@redis03 /server/soft]# ln -s /opt/kafka_2.13-2.8.0 /opt/kafka
[root@redis03 /server/soft]# mkdir /opt/kafka/logs
[root@redis03 /server/soft]# vim /opt/kafka/config/server.properties 
[root@redis03 /server/soft]# grep '^[a-Z]' /opt/kafka/config/server.properties 
broker.id=3
listeners=PLAINTEXT://172.16.0.212:9092
num.network.threads=3
num.io.threads=8
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
socket.request.max.bytes=104857600
log.dirs=/opt/kafka/logs
num.partitions=1
num.recovery.threads.per.data.dir=1
offsets.topic.replication.factor=1
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1
log.retention.hours=24
log.segment.bytes=1073741824
log.retention.check.interval.ms=300000
zookeeper.connect=172.16.0.210:2181,172.16.0.211:2181,172.16.0.212:2181
zookeeper.connection.timeout.ms=18000
group.initial.rebalance.delay.ms=0
```

### 启动节点kafka

```BASH
[root@redis01 ~]# /opt/kafka/bin/kafka-server-start.sh /opt/kafka/config/server.properties 
前台启动方便查看日志
[2021-08-20 22:17:00,952] INFO [KafkaServer id=1] started (kafka.server.KafkaServer)
看到这条，表示已经成功启动了
切换到后台运行
[root@redis01 ~]# /opt/kafka/bin/kafka-server-start.sh -daemon /opt/kafka/config/server.properties 
[root@redis01 ~]# tail -f /opt/kafka/logs/server.log 
```

```BASH
[root@redis02 ~]# /opt/kafka/bin/kafka-server-start.sh -daemon /opt/kafka/config/server.properties 
[root@redis02 ~]# tail -f /opt/kafka/logs/server.log 
```

```bash
[root@redis03 ~]# /opt/kafka/bin/kafka-server-start.sh -daemon /opt/kafka/config/server.properties 
[root@redis03 ~]# tail -f /opt/kafka/logs/server.log 
```

### 验真进程

```bash
[root@redis01 ~]# find / -name "jps"
/etc/alternatives/jps
/usr/bin/jps
/usr/share/elasticsearch/jdk/bin/jps
/usr/share/logstash/jdk/bin/jps
/usr/java/jdk-16.0.2/bin/jps
/opt/elasticsearch/elasticsearch-7.14.0/jdk/bin/jps

[root@redis01 ~]# jps
18407 QuorumPeerMain
2073 Elasticsearch
21084 Kafka
26460 -- process information unavailable
22191 Jps

[root@redis02 ~]# jps
7587 Jps
1845 QuorumPeerMain
2937 Kafka

[root@redis03 ~]# jps
1827 Kafka
2535 Jps
1401 QuorumPeerMain

```

### 测试创建topic

```BASH
#创建名为kafkatest，partitions(分区)为3，replication(复制)为3的topic(主题),在任意机器操作即可
[root@redis01 ~]# /opt/kafka/bin/kafka-topics.sh --create --zookeeper 172.16.0.210:2181,172.16.0.211:2181,172.16.0.212:2181 --partitions 3 --replication-factor 3 --topic kafkatest
Created topic kafkatest.


[root@redis01 ~]# /opt/kafka/bin/kafka-topics.sh --describe --zookeeper  172.16.0.210:2181,172.16.0.211:2181,172.16.0.212:2181 --topic kafkatest
Topic: kafkatest	TopicId: xU3SpKwEQWeMhgU5V8wt8Q	PartitionCount: 3	ReplicationFactor: 3	Configs: 
	Topic: kafkatest	Partition: 0	Leader: 2	Replicas: 2,3,1	Isr: 2,3,1
	Topic: kafkatest	Partition: 1	Leader: 3	Replicas: 3,1,2	Isr: 3,1,2
	Topic: kafkatest	Partition: 2	Leader: 1	Replicas: 1,2,3	Isr: 1,2,3

[root@redis02 ~]# /opt/kafka/bin/kafka-topics.sh --describe --zookeeper  172.16.0.210:2181,172.16.0.211:2181,172.16.0.212:2181 --topic kafkatest
Topic: kafkatest	TopicId: xU3SpKwEQWeMhgU5V8wt8Q	PartitionCount: 3	ReplicationFactor: 3	Configs: 
	Topic: kafkatest	Partition: 0	Leader: 2	Replicas: 2,3,1	Isr: 2,3,1
	Topic: kafkatest	Partition: 1	Leader: 3	Replicas: 3,1,2	Isr: 3,1,2
	Topic: kafkatest	Partition: 2	Leader: 1	Replicas: 1,2,3	Isr: 1,2,3

[root@redis03 ~]# /opt/kafka/bin/kafka-topics.sh --describe --zookeeper  172.16.0.210:2181,172.16.0.211:2181,172.16.0.212:2181 --topic kafkatest
Topic: kafkatest	TopicId: xU3SpKwEQWeMhgU5V8wt8Q	PartitionCount: 3	ReplicationFactor: 3	Configs: 
	Topic: kafkatest	Partition: 0	Leader: 2	Replicas: 2,3,1	Isr: 2,3,1
	Topic: kafkatest	Partition: 1	Leader: 3	Replicas: 3,1,2	Isr: 3,1,2
	Topic: kafkatest	Partition: 2	Leader: 1	Replicas: 1,2,3	Isr: 1,2,3

```

状态说明：kafkatest有三个分区分别为1、2、3，分区0的leader是2（broker.id），分区0有三个副本，并且状态都为lsr（ln-sync，表示可以参加选举成为leader）。

### 测试删除topic

```bash
[root@redis03 ~]# /opt/kafka/bin/kafka-topics.sh --delete --zookeeper  172.16.0.210:2181,172.16.0.211:2181,172.16.0.212:2181 --topic kafkatest
Topic kafkatest is marked for deletion.
Note: This will have no impact if delete.topic.enable is not set to true.

[root@redis03 ~]# /opt/kafka/bin/kafka-topics.sh --describe --zookeeper  172.16.0.210:2181,172.16.0.211:2181,172.16.0.212:2181 --topic kafkatest
Error while executing topic command : Topic 'kafkatest' does not exist as expected
[2021-08-20 22:51:06,693] ERROR java.lang.IllegalArgumentException: Topic 'kafkatest' does not exist as expected
	at kafka.admin.TopicCommand$.kafka$admin$TopicCommand$$ensureTopicExists(TopicCommand.scala:542)
	at kafka.admin.TopicCommand$ZookeeperTopicService.describeTopic(TopicCommand.scala:447)
	at kafka.admin.TopicCommand$.main(TopicCommand.scala:69)
	at kafka.admin.TopicCommand.main(TopicCommand.scala)
 (kafka.admin.TopicCommand$)
#可以看出已经删除了
```

### 测试获取所有的topic列表

```BASH
创建2个topic
[root@redis01 ~]# /opt/kafka/bin/kafka-topics.sh --create --zookeeper 172.16.0.210:2181,172.16.0.211:2181,172.16.0.212:2181 --partitions 3 --replication-factor 3 --topic kafkatest
Created topic kafkatest.

[root@redis01 ~]# /opt/kafka/bin/kafka-topics.sh --create --zookeeper 172.16.0.210:2181,172.16.0.211:2181,172.16.0.212:2181 --partitions 3 --replication-factor 3 --topic kafkatest2
Created topic kafkatest2.

```

查看所有topic列表

```BASH
[root@redis01 ~]# /opt/kafka/bin/kafka-topics.sh --list --zookeeper 172.16.0.210:2181,172.16.0.211:2181,172.16.0.212:2181
kafkatest
kafkatest2

```

### kafka测试命令发送消息

```bash
[root@redis01 ~]# /opt/kafka/bin/kafka-topics.sh --create --zookeeper 172.16.0.210:2181,172.16.0.211:2181,172.16.0.212:2181 --partitions 3 --replication-factor 3 --topic messagetest
Created topic messagetest.

```

发送消息:端口是 kafka的9092,而不是zookeeper的2181

```bash
[root@redis01 ~]# /opt/kafka/bin/kafka-console-producer.sh --broker-list 172.16.0.210:9092,172.16.0.211:9092,172.16.0.212:9092 --topic messagetest
>hello,world!
>this is a
>
```

其他服务器获取消息

```BASH
[root@redis02 ~]# /opt/kafka/bin/kafka-console-consumer.sh --bootstrap-server 172.16.0.210:9092,172.16.0.211:9092,172.16.0.212:9092 --topic messagetest --from-beginning

[root@redis03 ~]# /opt/kafka/bin/kafka-console-consumer.sh --bootstrap-server 172.16.0.210:9092,172.16.0.211:9092,172.16.0.212:9092 --topic messagetest --from-beginning

```

这个就可以跟第一台实现通信了

![image-20210820232349827](ELK.assets\image-20210820232349827.png)



## 使用kafka作为缓存

### filebeat 配置

```bash
[root@redis01 ~]# grep -Ev '#|^$' /etc/filebeat/filebeat.yml
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/nginx/access.log
  json.keys_under_root: true
  json.overwrite_keys: true
  tags: ["access"]
- type: log
  enabled: true
  paths:
    - /var/log/nginx/error.log
  tags: ["error"]
  
setup.template.settings:
  index.number_of_shards: 3
  
setup.kibana:
  host: "172.16.0.210:5601"
  
output.kafka:
  hosts: ["172.16.0.210:9092","172.16.0.211:9092","172.16.0.212:9092"]
  topic: elklog
```

logstash 配置

```BASH
[root@redis01 ~]# cat /etc/logstash/conf.d/kaka.conf
input{
  kafka{
    bootstrap_servers=>"172.16.0.210:9092"
    topics=>["elklog"]
    group_id=>"logstash"
    codec => "json"
  }
}
filter {
  mutate {
    convert => ["upstream_time", "float"]                                                                                                                        
    convert => ["request_time", "float"]
  }
}
output {
    if "access" in [tags] {
      elasticsearch {
        hosts => "http://172.16.0.210:9200"
        manage_template => false
        index => "nginx_access-%{+yyyy.MM.dd}"
      }
    }
    if "error" in [tags] {
      elasticsearch {
        hosts => "http://172.16.0.210:9200"
        manage_template => false
        index => "nginx_error-%{+yyyy.MM.dd}"
      }
    }
}

```

```BASH
[root@redis01 ~]# systemctl restart logstash.service filebeat.service 

启动kafka配置文件
[root@redis01 ~]# /usr/share/logstash/bin/logstash -f /etc/logstash/conf.d/kaka.conf 

```

```BASH
[root@redis01 ~]# vim /etc/logstash/conf.d/kaka.conf 
input{
  kafka{
    bootstrap_servers=>"172.16.0.210:9092"
    topics=>["elklog"]
    group_id=>"logstash"
    codec => "json"
  }
}
filter {
  mutate {
    convert => ["upstream_time", "float"]                                                                                            
    convert => ["request_time", "float"]
  }
}
output {
    if "access" in [tags] {
      elasticsearch {
        hosts => "http://172.16.0.210:9200"
        manage_template => false
        index => "nginx_kafka_access-%{+yyyy.MM.dd}"
      }
    }
    if "error" in [tags] {
      elasticsearch {
        hosts => "http://172.16.0.210:9200"
        manage_template => false
        index => "nginx_kafka_error-%{+yyyy.MM.dd}"
      }                                                                                                                             
    }
}

```

```bash
[root@redis01 ~]# ab -n 10000 -c 20 172.16.0.210/

[root@redis01 ~]# ab -n 100 -c 20 172.16.0.210/test.html


```

![image-20210820234919236](ELK.assets\image-20210820234919236.png)

![image-20210820235232699](ELK.assets\image-20210820235232699.png)

![image-20210820235242561](ELK.assets\image-20210820235242561.png)

好了完全配置正确

# grafana画图展示

除了kibana外，grafana也支持从es调取数据并展示

https://grafana.com/docs/features/datasources/elasticsearch/#using-elasticsearch-in-grafana

配置logstash 同步mysql数据

需要jdbc插件

https://www.bbsmax.com/A/gVdnppNEJW/

也就是替换了ES为mysql 。

