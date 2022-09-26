# 十、MongoDB

MongoDB 是一个基于分布式文件存储的数据库。由 C++ 语言编写。旨在为 WEB 应用提供可扩展的高性能数据存储解决方案。

MongoDB 是一个介于关系数据库和非关系数据库之间的产品，是非关系数据库当中功能最丰富，最像关系数据库的

 

MongoDB 逻辑结构

Mongodb 逻辑结构             MySQL逻辑结构

库database                库

集合（collection）             表

文档（document）             数据行

 

## 安装部署

**环境准备**

[root@DB01 ~]# for i in {3307..3310};do systemctl stop mysqld$i && systemctl disable mysqld$i;done

Removed symlink /etc/systemd/system/multi-user.target.wants/mysqld3307.service.

Removed symlink /etc/systemd/system/multi-user.target.wants/mysqld3308.service.

Removed symlink /etc/systemd/system/multi-user.target.wants/mysqld3309.service.

Removed symlink /etc/systemd/system/multi-user.target.wants/mysqld3310.service.

[root@DB01 ~]# for i in {3307..3310};do systemctl status mysqld$i|grep A ;done

  Active: inactive (dead)

  Active: inactive (dead)

  Active: inactive (dead)

  Active: inactive (dead)

（1）redhat或centos6.2以上系统

（2）系统开发包完整

（3）ip地址和hosts文件解析正常

（4）iptables防火墙&SElinux关闭

（5）关闭大页内存机制

**方法一：**

root用户下

在vi /etc/rc.local最后添加如下代码

if test -f /sys/kernel/mm/transparent_hugepage/enabled; then

 echo never > /sys/kernel/mm/transparent_hugepage/enabled

fi

if test -f /sys/kernel/mm/transparent_hugepage/defrag; then

  echo never > /sys/kernel/mm/transparent_hugepage/defrag

fi

   

临时关闭

echo never > /sys/kernel/mm/transparent_hugepage/enabled      

echo never > /sys/kernel/mm/transparent_hugepage/defrag 

查看  

cat /sys/kernel/mm/transparent_hugepage/enabled    

cat /sys/kernel/mm/transparent_hugepage/defrag 

其他系统关闭参照官方文档：  

https://docs.mongodb.com/manual/tutorial/transparent-huge-pages/

**方法二：**

创建systemd unit file

/etc/systemd/system/disable-transparent-huge-pages.service

[Unit]

Description=Disable Transparent Huge Pages (THP)

DefaultDependencies=no

After=sysinit.target local-fs.target

Before=mongod.service

 

[Service]

Type=oneshot

ExecStart=/bin/sh -c 'echo never | tee /sys/kernel/mm/transparent_hugepage/enabled > /dev/null'

 

[Install]

WantedBy=basic.target

启动

sudo systemctl daemon-reload

sudo systemctl start disable-transparent-huge-pages

cat /sys/kernel/mm/transparent_hugepage/enabled  #检查是否设置成功

sudo systemctl enable disable-transparent-huge-pages

关闭大页内存机制是因为性能需要-THP

 

 

MongoDB安装

[root@DB01 ~]# useradd mongod && passwd mongod

[root@DB01 ~]# mkdir -p /mongodb/{conf,log,data}

 

官网：https://www.mongodb.com/

下载页面：https://www.mongodb.com/try/download/community

进去选择 server centos7 

​                               

[root@DB01 /server/soft]# ls

mha4mysql-node-0.56-0.el6.noarch.rpm  

Mycat-server-1.6.7.6-release-20210303094759-linux.tar.gz

**mongodb-linux-x86_64-rhel70-4.4.6.tgz** 

mysql-5.7.33-linux-glibc2.12-x86_64.tar.gz

[root@DB01 /server/soft]# cp -r mongodb-linux-x86_64-rhel70-4.4.6/bin/ /mongodb/

[root@DB01 /server/soft]# ls /mongodb/

bin/ conf/ data/ log/ 

[root@DB01 /server/soft]# chown -R mongod:mongod /mongodb

切换用户

[root@DB01 /server/soft]# su - mongod

[mongod@DB01 ~]$ echo 'export PATH=/mongodb/bin:$PATH' >>.bash_profile 

[mongod@DB01 ~]$ source .bash_profile 

启动

[mongod@DB01 ~]$ mongod --dbpath=/mongodb/data --logpath=/mongodb/log/mongodb.log --port=27017 --logappend --fork

about to fork child process, waiting until server is ready for connections.

forked process: 1544

child process started successfully, parent exiting

登录

[mongod@DB01 ~]$ mongo

关闭

[mongod@DB01 ~]$ mongod --dbpath=/mongodb/data --shutdown

 

注：连接之后会有warning，需要修改(使用root用户)

vim /etc/security/limits.conf 

\#*    -    nofile    65535 

 

*reboot重启生效*

 

 

 

使用配置文件(弃用)

*vim /mongodb/conf/mongodb.conf*

*logpath=/mongodb/log/mongodb.log*

*dbpath=/mongodb/data* 

*port=27017*

*logappend=true*

*fork=true*

 

### 启动参数结合yaml 文件讲解

（YAML模式：）

\--

NOTE：

YAML does not support tab characters for indentation: use spaces instead.

 

--系统日志有关 

systemLog:

  destination: file    

  path: "/mongodb/log/mongodb.log"  --日志位置

  logAppend: true                   --日志以追加模式记录

 

--数据存储有关  

storage:

  journal:

   enabled: true

  dbPath: "/mongodb/data"      --数据路径的位置

 

 

-- 进程控制 

processManagement:

  fork: true             --后台守护进程

  pidFilePath: <string>           --pid文件的位置，一般不用配置，可以去掉这行，自动生成到data中

  

--网络配置有关  

net:        

  bindIp: <ip>            -- 监听地址，如果不配置这行是监听在0.0.0.0

  port: <port>                    -- 端口号,默认不配置端口号，是27017

  

-- 安全验证有关配置   

security:

 authorization: enabled       --是否打开用户名密码验证

 

 

------------------以下是复制集与分片集群有关---------------------- 

 

replication:

 oplogSizeMB: <NUM>

 replSetName: "<REPSETNAME>"

 secondaryIndexPrefetch: "all"

 

sharding:

  clusterRole: <string>

  archiveMovedChunks: <boolean>

   

---for mongos only

replication:

  localPingThresholdMs: <int>

 

sharding:

  configDB: <string>

\---

.........

++++++++++++++++++++++

YAML例子

cat > /mongodb/conf/mongo.conf <<EOF

systemLog:

  destination: file

  path: "/mongodb/log/mongodb.log"

  logAppend: true

storage:

  journal:

   enabled: true

  dbPath: "/mongodb/data/"

processManagement:

  fork: true

net:

  port: 27017

  bindIp: 192.168.1.200,127.0.0.1

EOF

  

  

mongod -f /mongodb/conf/mongo.conf --shutdown

mongod -f /mongodb/conf/mongo.conf  

 

++++++++++++++++++++++

Yaml 方式启动。跟kubernetes很像

[mongod@DB01 /mongodb/conf]$ mongod -f mongo.cnf --shutdown

killing process with pid: 1499

[mongod@DB01 /mongodb/conf]$ mongod -f mongo.cnf

about to fork child process, waiting until server is ready for connections.

forked process: 1572

child process started successfully, parent exiting

 

### systemd 管理(root)

 

[root@db01 ~]# tee > /etc/systemd/system/mongod.service <<EOF

[Unit]

Description=mongodb 

After=network.target remote-fs.target nss-lookup.target

[Service]

User=mongod

Type=forking

ExecStart=/mongodb/bin/mongod --config /mongodb/conf/mongo.conf

ExecReload=/bin/kill -s HUP $MAINPID

ExecStop=/mongodb/bin/mongod --config /mongodb/conf/mongo.conf --shutdown

PrivateTmp=true 

[Install]

WantedBy=multi-user.target

EOF

 

[root@db01 ~]# systemctl restart mongod

[root@db01 ~]# systemctl stop mongod

[root@db01 ~]# systemctl start mongod

[root@DB01 ~]# systemctl restart mongod

[root@DB01 ~]# systemctl status mongod|grep running

  Active: active (**running**) since Sat 2021-06-12 11:14:18 CST; 35s ago

 

## mongodb常用基本操作

mongodb 默认存在的库

\> show databases;

admin  0.000GB

config 0.000GB

local  0.000GB

 

命令种类

数据库对象(库(database),表(collection),行(document))

 

db.命令:

DB级别命令

db    当前在的库

\> db

test

db.[TAB] 类似于linux中的tab功能

 

db.help() db级别的命令使用帮助

 

 

collection级别操作:

db.Collection_name.xxx

\> db.adminCommand(Math)

{

​    "ok" : 0,

​    "errmsg" : "no such command: 'sigFig'",

​    "code" : 59,

​    "codeName" : "CommandNotFound"

}

 

document级别操作:

db.t1.insert()

\> db.t1.insert({"id":"1","name":"zhangsan","age":"25"})

WriteResult({ "nInserted" : 1 })

\> db.t1.find({"id":"1"})

{ "_id" : ObjectId("60c44541a388b1c6f25b61ec"), "id" : "1", "name" : "zhangsan", "age" : "25" }

 

复制集有关(replication set):

rs.

\> rs.

rs.add(              rs.hasOwnProperty(         rs.reconfig(

rs.addArb(             rs.hello(             rs.remove(

rs.apply(             rs.help(             rs.secondaryOk(

rs.bind(              rs.initiate(            rs.slaveOk(

rs.call(              rs.isMaster(            rs.status(

rs.compareOpTimes(         rs.isValidOpTime(         rs.stepDown(

rs.conf(              rs.printReplicationInfo(      rs.syncFrom(

rs.config(             rs.printSecondaryReplicationInfo( rs.toLocaleString(

rs.constructor           rs.printSlaveReplicationInfo(   rs.toString(

rs.debug              rs.propertyIsEnumerable(      rs.valueOf(

rs.freeze(             rs.prototype

 

分片集群(sharding cluster)

sh.

\> sh.

sh.addShard(         sh.getBalancerState(     sh.setBalancerState(

sh.addShardTag(        sh.getBalancerWindow(     sh.shardCollection(

sh.addShardToZone(      sh.getRecentFailedRounds(   sh.splitAt(

sh.addTagRange(        sh.getRecentMigrations(    sh.splitFind(

sh.apply(           sh.getShouldAutoSplit(    sh.startBalancer(

sh.balancerCollectionStatus( sh.hasOwnProperty(      sh.status(

sh.bind(           sh.help(           sh.stopBalancer(

sh.call(           sh.isBalancerRunning(     sh.toLocaleString(

sh.constructor        sh.moveChunk(         sh.toString(

sh.disableAutoSplit(     sh.propertyIsEnumerable(   sh.updateZoneKeyRange(

sh.disableBalancing(     sh.prototype          sh.valueOf(

sh.enableAutoSplit(      sh.removeRangeFromZone(    sh.waitForBalancer(

sh.enableBalancing(      sh.removeShardFromZone(    sh.waitForPingChange(

sh.enableSharding(      sh.removeShardTag(

sh.getActiveMigrations(    sh.removeTagRange(

 

 

### 帮助

help

KEYWORDS.help()  #> db.t1.help()

KEYWORDS.[TAB]  #> db.t1.[tab]

 

show

use 

db.help()

db.a.help()

rs.help()

sh.help()

 

 

### 常用操作

#### 查

--查看当前db版本

test> db.version()

 

--显示当前数据库

test> db

test

或

\> db.getName()

test

 

--查询所有数据库

test> show dbs

\> show database

– 切换数据库

\> use local

switched to db local

 

\- 查看所有的collection

show tables;

 

– 显示当前数据库状态

test> use local

switched to db local

 

local> db.stats()

 

– 查看当前数据库的连接机器地址

\> db.getMongo()

connection to 127.0.0.1

指定数据库进行连接

默认连接本机test数据库

 

**mongodb对象操作：**

mongo     mysql

库  -----> 库

集合 -----> 表

文档 -----> 数据行

 

#### 库-插

– 创建数据库：

当use的时候，系统就会自动创建一个数据库。

如果use之后没有创建任何集合。

系统就会删除这个数据库。

 

– 删除数据库

如果没有选择任何数据库，会删除默认的test数据库

//删除test数据库

 

test> show dbs

local 0.000GB

test 0.000GB

 

test> use test

switched to db test

 

test> db.dropDatabase()  

{ "dropped" : "test", "ok" : 1 }

 

#### 表|集合的操作：

 创建集合

方法1

admin> use app

switched to db app

app> db.createCollection('a')

{ "ok" : 1 }

app> db.createCollection('b')

{ "ok" : 1 }

\> show collections //查看当前数据下的所有集合

a b 或

\> db.getCollectionNames()

[ "a", "b" ]

 

方法2：当插入一个文档的时候，一个集合就会自动创建。

{id : "101" ,name : "zhangsan" ,age : "18" ,gender : "male"}

 

use oldboy

db.oldguo.insert({id : "1021" ,name : "zhssn" ,age : "22" ,gender : "female",address : "sz"})

 

\> db.oldguo.find({id:"101"})

{ "_id" : ObjectId("5d36b8b6e62adeeaf0de00dc"), "id" : "101", "name" : "zhangsan", "age" : "18", "gender" : "male" }

 

 

查询数据:

\> db.oldguo.find({id:"101"}).pretty()

{

​    "_id" : ObjectId("5d36b8b6e62adeeaf0de00dc"),

​    "id" : "101",

​    "name" : "zhangsan",

​    "age" : "18",

​    "gender" : "male"

}

\> db.oldguo.find().pretty()

{

​    "_id" : ObjectId("5d36b8b6e62adeeaf0de00dc"),

​    "id" : "101",

​    "name" : "zhangsan",

​    "age" : "18",

​    "gender" : "male"

}

{

​    "_id" : ObjectId("5d36b8eae62adeeaf0de00dd"),

​    "id" : "1021",

​    "name" : "zhssn",

​    "age" : "22",

​    "gender" : "female",

​    "address" : "sz"

}

{

​    "_id" : ObjectId("5d36b913e62adeeaf0de00de"),

​    "name" : "ls",

​    "address" : "bj",

​    "telnum" : "110"

}

 

#### 删除集合

app> use app

switched to db app

\> db.ap.drop()

true         //删除集合

 

– 重命名集合

//把a改名为c

\> db.a.renameCollection("c")

{ "ok" : 1 }

app> show collections

a b c

\> show tables

b

c

批量插入数据

 

for(i=0;i<10000;i++){db.log.insert({"uid":i,"name":"mongodb","age":6,"date":new

Date()})}

 

 

Mongodb数据查询语句:

 

– 查询集合中的记录数

app> db.a.find() //查询所有记录

 

注：默认每页显示20条记录，当显示不下的的情况下，可以用it迭代命令查询下一页数据。

设置每页显示数据的大小：

 

\> DBQuery.shellBatchSize=50; //每页显示50条记录

 

app> db.a.findOne() //查看第1条记录

app> db.a.count() //查询总的记录数

 

– 删除集合中的记录数

app> db.a.remove({}) //删除集合中所有记录

\> db.a.distinct("name") //查询去掉当前集合中某列的重复数据

 

– 查看集合存储信息

app> db.a.stats()

app> db.a.dataSize() //集合中数据的原始大小

app> db.a.totalIndexSize() //集合中索引数据的原始大小

app> db.a.totalSize() //集合中索引+数据压缩存储之后的大小  *****

app> db.a.storageSize() //集合中数据压缩存储的大小

 

### 用户管理

用户管理 *****

 

注意：

验证库，建立用户时use到的库，在使用用户时，要加上验证库才能登陆。

对于管理员用户,必须在admin下创建.

\1. 建用户时,use到的库,就是此用户的验证库

\2. 登录时,必须明确指定验证库才能登录

\3. 通常,管理员用的验证库是admin,普通用户的验证库一般是所管理的库设置为验证库

\4. 如果直接登录到数据库,不进行use,默认的验证库是test,不是我们生产建议的.

 

use admin 

mongo 192.168.1.200/admin

 

db.createUser

{

  user: "<name>",

  pwd: "<cleartext password>",

  roles: [

​    { role: "<role>",

   db: "<database>" } | "<role>",

  ...

  ]

}

 

 

基本语法说明：

user:用户名

pwd:密码

roles:

  role:角色名

  db:作用对象 

role：root, readWrite,read  

 

验证数据库：

mongo -u wang -p wang123 192.168.1.200/admin

 

\-------------

用户管理例子：

 

-- 1. 创建超级管理员：管理所有数据库（必须use admin再去创建） *****

$ mongo

use admin

db.createUser(

{

  user: "root",

  pwd: "root123",

  roles: [ { role: "root", db: "admin" } ]

}

)

 

验证用户

**db.auth('root','root123')**

 

\> db.createUser({

... user:"wang",

... pwd:"wang123",

... roles:[{role:"root",db:"admin"}]

... })

Successfully added user: {

​    "user" : "wang",

​    "roles" : [

​       {

​           "role" : "root",

​           "db" : "admin"

​       }

​    ]

}

\> **db.auth("wang","wang123")**

1

 

 

 

my_repl:PRIMARY> db.createUser({

... user:"root",

... pwd:"root123",

... roles:[{role:"root",db:"admin"}]

... })

Successfully added user: {

​    "user" : "root",

​    "roles" : [

​       {

​           "role" : "root",

​           "db" : "admin"

​       }

​    ]

}

my_repl:PRIMARY> db.auth("root","root123")

1

 

配置文件中，加入以下配置

security:

 authorization: enabled

 

重启mongodb

mongod -f /mongodb/conf/mongo.conf --shutdown 

mongod -f /mongodb/conf/mongo.conf 

 

登录验证

mongo -uroot -proot123 admin

mongo -uroot -proot123 192.168.1.200/admin

 

或者

mongo

use admin

db.auth('root','root123')

 

查看用户:

use admin

db.system.users.find().pretty()

 

==================

-- 2、创建库管理用户

mongo -uroot -proot123 admin

 

use app

 

db.createUser(

{

user: "admin",

pwd: "admin",

roles: [ { role: "dbAdmin", db: "app" } ]

}

)

 

db.auth('admin','admin')

 

登录测试

mongo -uadmin -padmin 10.0.0.51/app

 

#### 系统role

roles 详解如下：

**数据库用户角色（Database User Roles)**

read : 授权User只读数据的权限，允许用户读取指定的数据库

readWrite 授权User读/写数据的权限，允许用户读/写指定的数据库

**数据库管理角色（Database Admininstration Roles)**

dbAdmin：在当前的数据库中执行管理操作，如索引的创建、删除、统计、查看等

dbOwner：在当前的数据库中执行任意操作，增、删、改、查等

userAdmin ：在当前的数据库中管理User，创建、删除和管理用户。

**备份和还原角色（Backup and Restoration Roles)******

backup

restore

**跨库角色（All-Database Roles)**

readAnyDatabase：授权在所有的数据库上读取数据的权限，只在admin 中可用

readWriteAnyDatabase：授权在所有的数据库上读写数据的权限，只在admin 中可用

userAdminAnyDatabase：授权在所有的数据库上管理User的权限，只在admin中可用

dbAdminAnyDatabase： 授权管理所有数据库的权限，只在admin 中可用

**集群管理角色（Cluster Administration Roles)**

clusterAdmin：授权管理集群的最高权限，只在admin中可用

clusterManager：授权管理和监控集群的权限

clusterMonoitor：授权监控集群的权限，对监控工具具有readonly的权限

hostManager：管理server

**超级角色（super master  Roles)**

root ：超级账户和权限，只在admin中可用le

 

 

 

-- 3、创建对app数据库，读、写权限的用户app01 *****

 

（1）超级管理员用户登陆

mongo -uroot -proot123 admin

 

（2）选择一个验证库

use app

 

(3)创建用户

db.createUser(

​    {

​       user: "app01",

​       pwd: "app01",

​       roles: [ { role: "**readWrite**" , db: "app" } ]

​    }

)

db.auth('app01','app01')

mongo -uapp01 -papp01 10.0.0.51/app

 

-- 4、创建app数据库读写权限的用户并对test数据库具有读权限：

mongo -uroot -proot123 10.0.0.51/admin

use app

db.createUser(

{

user: "app03",

pwd: "app03",

roles: [ { role: "readWrite", db: "app" },

{ role: "read", db: "test" }]})

 

-- 5、查询mongodb中的用户信息

mongo -uroot -proot123 10.0.0.51/admin

db.system.users.find().pretty()

 

 

{

​    "_id" : "admin.root",

​    "user" : "root",

​    "db" : "admin",

​    "credentials" : {

​       "SCRAM-SHA-1" : {

​           "iterationCount" : 10000,

​           "salt" : "HsSHIKBQyMnFEzA/PSURYA==",

​           "storedKey" : "dbOoQserGa/fB+JQyLqr1yXQZBM=",

​           "serverKey" : "h+b/vARfWp6cmDquUN6bJo4whdc="

​       }

​    },

​    "roles" : [

​       {

​           "role" : "root",

​           "db" : "admin"

​       }

​    ]

}

 

 

 

-- 6、删除用户（root身份登录，use到验证库）

 

删除用户

\# mongo -uroot -proot123 192.168.1.200/admin

use app

db.dropUser("app01")

 

### MongoDB复制集RS（ReplicationSet）******

 

基本原理

基本构成是1主2从的结构，自带互相监控投票机制（Raft（MongoDB） Paxos（mysql MGR 用的是变种））

如果发生主库宕机，复制集内部会进行投票选举，选择一个新的主库替代原有主库对外提供服务。同时复制集会自动通知

客户端程序，主库已经发生切换了。应用就会连接到新的主库。

#### Replication Set配置过程详解

规划

三个以上的mongodb节点（或多实例）

环境准备

多个端口：

**28017、28018、28019、28020**

多套目录：

su - mongod 

[mongod@DB01 /mongodb/conf]$ for i in {28017..28020};do mkdir -p /mongodb/$i/{conf,log,data};done

[mongod@DB01 /mongodb/conf]$ tree /mongodb/ -L 2

/mongodb/

├── 28017

│  ├── conf

│  ├── data

│  └── log

├── 28018

│  ├── conf

│  ├── data

│  └── log

├── 28019

│  ├── conf

│  ├── data

│  └── log

├── 28020

│  ├── conf

│  ├── data

│  └── log

 

配置文件内容:

cat > /mongodb/28017/conf/mongod.conf <<EOF

systemLog:

 destination: file

 path: /mongodb/28017/log/mongodb.log

 logAppend: true

storage:

 journal:

  enabled: true

 dbPath: /mongodb/28017/data

 directoryPerDB: true

 \#engine: wiredTiger

 wiredTiger:

  engineConfig:

   cacheSizeGB: 1

   directoryForIndexes: true

  collectionConfig:

   blockCompressor: zlib

  indexConfig:

   prefixCompression: true

processManagement:

 fork: true

net:

 bindIp: 192.168.1.200,127.0.0.1

 port: **28017**

replication:

 oplogSizeMB: 2048

 replSetName: my_repl

EOF

复制并修改多个文件    

[mongod@DB01 /mongodb]$ for i in {28018..28020};do cp /mongodb/28017/conf/mongod.conf /mongodb/$i/conf/ ;sed -i "s/28017/"$i"/g" /mongodb/$i/conf/mongod.conf;done

 

启动多个实例备用:（实际使用systemd启动并设置开机自启）

[mongod@DB01 /mongodb]$ for i in {28017..28020};do mongod -f /mongodb/$i/conf/mongod.conf ;done

 

[mongod@DB01 /mongodb]$ netstat -lntup|grep 280

 

 

 

 

#### 配置普通复制集：

1主2从，从库普通从库

 

mongo --port 28017 admin

config = {_id: 'my_repl', members: [

​             {_id: 0, host: '192.168.1.200:28017'},

​             {_id: 1, host: '192.168.1.200:28018'},

​             {_id: 2, host: '192.168.1.200:28019'}]

​     }  

​         

rs.initiate(config) 

 

 

查询复制集状态

rs.status();

 

 

 

主1从1个arbiter

 

mongo -port 28017 admin

config = {_id: 'my_repl', members: [

​             {_id: 0, host: '192.168.1.200:28017'},

​             {_id: 1, host: '192.168.1.200:28018'},

​             {_id: 2, host: '192.168.1.200:28019',"arbiterOnly":true}]

​     }        

rs.initiate(config) 

 

 

#### 复制集管理操作

 

**查看复制集状态**

rs.status();  //查看整体复制集状态

rs.isMaster(); // 查看当前是否是主节点

rs.conf()；  //查看复制集配置信息

 

**添加删除节点**

rs.remove("ip:port"); // 删除一个节点

rs.add("ip:port"); // 新增从节点

rs.addArb("ip:port"); // 新增仲裁节点

例子：

my_repl:PRIMARY> rs.add("192.168.1.200:28020")

my_repl:PRIMARY> rs.remove("192.168.1.200:28020")

my_repl:PRIMARY> rs.addArb("192.168.1.200:28020")

添加 arbiter节点

my_repl:PRIMARY> rs.isMaster()

 

 

 

##### 特殊从节点

arbiter节点：主要负责选主过程中的投票，但是不存储任何数据，也不提供任何服务

hidden节点：隐藏节点，不参与选主，也不对外提供服务。

delay节点：延时节点，数据落后于主库一段时间，因为数据是延时的，也不应该提供服务或参与选主，所以通常会配合hidden（隐藏）

一般情况下会将delay+hidden一起配置使用

 

**配置延时节点（一般延时节点也配置成hidden）**

cfg=rs.conf() 

cfg.members[2].priority=0

cfg.members[2].hidden=true

cfg.members[2].slaveDelay=120

rs.reconfig(cfg)  

上面的2是第三个。从0开始计算

   

取消以上配置

cfg=rs.conf() 

cfg.members[2].priority=1

cfg.members[2].hidden=false

cfg.members[2].slaveDelay=0

rs.reconfig(cfg)  

 

 

配置成功后，通过以下命令查询配置后的属性

rs.conf(); 

 

##### 副本集其他操作命令

查看副本集的配置信息

admin> rs.conf()

查看副本集各成员的状态

admin> rs.status()

++++++++++++++++++++++++++++++++++++++++++++++++

 

--副本集角色切换（不要人为随便操作）

admin> rs.stepDown()

注：

admin> rs.freeze(300) //锁定从，使其不会转变成主库

freeze()和stepDown单位都是秒。

+++++++++++++++++++++++++++++++++++++++++++++

设置副本节点可读：在副本节点执行

admin> rs.slaveOk()

从库默认不能读写需要读写执行上面命令

eg：

admin> use app

switched to db app

app> db.createCollection('a')

{ "ok" : 0, "errmsg" : "not master", "code" : 10107 }

 

查看副本节点（监控主从延时）

admin> rs.printSlaveReplicationInfo()

source: 192.168.1.22:27017

  syncedTo: Thu May 26 2016 10:28:56 GMT+0800 (CST)

  0 secs (0 hrs) behind the primary

 

## MongoDB Sharding Cluster 分片集群

 

 

规划：

10个实例：38017-38026

（1）configserver:

3台构成的复制集（1主两从，不支持arbiter）38018-38020（复制集名字configsvr）

 

（2）shard节点：

sh1：38021-23  （1主两从，其中一个节点为arbiter，复制集名字sh1）

sh2：38024-26  （1主两从，其中一个节点为arbiter，复制集名字sh2）

 

配置过程

​           

### shard复制集配置：

目录创建：

for i in {38021..38026};do mkdir -p /mongodb/$i/{conf,log,data};done

 

修改配置文件：

 

sh1:

cat > /mongodb/38021/conf/mongodb.conf<<EOF 

systemLog:

 destination: file

 path: /mongodb/38021/log/mongodb.log  

 logAppend: true

storage:

 journal:

  enabled: true

 dbPath: /mongodb/38021/data

 directoryPerDB: true

 \#engine: wiredTiger

 wiredTiger:

  engineConfig:

   cacheSizeGB: 1

   directoryForIndexes: true

  collectionConfig:

   blockCompressor: zlib

  indexConfig:

   prefixCompression: true

net:

 bindIp: 192.168.1.200,127.0.0.1

 port: 38021

replication:

 oplogSizeMB: 2048

 replSetName: sh1

sharding:

 clusterRole: shardsvr

processManagement: 

 fork: true

EOF

 

for i in {38022..38023};do cp /mongodb/38021/conf/mongodb.conf /mongodb/$i/conf;

sed -i "s/38021/"$i"/g" /mongodb/$i/conf/mongodb.conf;done

 

sh2:

cat > /mongodb/38024/conf/mongodb.conf<<EOF 

systemLog:

 destination: file

 path: /mongodb/38024/log/mongodb.log  

 logAppend: true

storage:

 journal:

  enabled: true

 dbPath: /mongodb/38024/data

 directoryPerDB: true

 wiredTiger:

  engineConfig:

   cacheSizeGB: 1

   directoryForIndexes: true

  collectionConfig:

   blockCompressor: zlib

  indexConfig:

   prefixCompression: true

net:

 bindIp: 192.168.1.200,127.0.0.1

 port: 38024

replication:

 oplogSizeMB: 2048

 replSetName: sh2

sharding:

 clusterRole: shardsvr

processManagement: 

 fork: true

EOF

 

for i in {38025..38026};do cp /mongodb/38024/conf/mongodb.conf /mongodb/$i/conf;

sed -i "s/38024/"$i"/g" /mongodb/$i/conf/mongodb.conf;done

 

启动所有节点，并搭建复制集：

 

for i in {38021..38026};do mongod -f /mongodb/$i/conf/mongodb.conf;done

 

 

mongo --port 38021 admin

 

config = {_id: 'sh1', members: [

​             {_id: 0, host: '192.168.1.200:38021'},

​             {_id: 1, host: '192.168.1.200:38022'},

​             {_id: 2, host: '192.168.1.200:38023',"arbiterOnly":true}]

​      }

 

rs.initiate(config)

 

 

mongo --port 38024 admin

config = {_id: 'sh2', members: [

​             {_id: 0, host: '192.168.1.200:38024'},

​             {_id: 1, host: '192.168.1.200:38025'},

​             {_id: 2, host: '192.168.1.200:38026',"arbiterOnly":true}]

​      }

 

rs.initiate(config)

 

=-=----=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-

config节点配置：

目录创建

 

for i in {38018..38020};do mkdir -p /mongodb/$i/{conf,log,data};done

 

修改配置文件：

cat > /mongodb/38018/conf/mongodb.conf <<EOF

systemLog:

 destination: file

 path: /mongodb/38018/log/mongodb.log

 logAppend: true

storage:

 journal:

  enabled: true

 dbPath: /mongodb/38018/data

 directoryPerDB: true

 \#engine: wiredTiger

 wiredTiger:

  engineConfig:

   cacheSizeGB: 1

   directoryForIndexes: true

  collectionConfig:

   blockCompressor: zlib

  indexConfig:

   prefixCompression: true

net:

 bindIp: 192.168.200,127.0.0.1

 port: 38018

replication:

 oplogSizeMB: 2048

 replSetName: configReplSet

sharding:

 clusterRole: configsvr

processManagement: 

 fork: true

EOF

 

for i in {38019..38020};do cp /mongodb/38018/conf/mongodb.conf /mongodb/$i/conf/;

sed -i "s/38018/"$i"/g" /mongodb/$i/conf/mongodb.conf;done

 

遇到的错误解决

 

 

 

查看日志。权限不足

 

 

 

 

 

### 启动节点，并配置复制集

 

for i in {38018..38020};do mongod -f /mongodb/$i/conf/mongodb.conf;done

 

 

mongo --port 38018 admin

 

config = {_id: 'configReplSet', members: [

​             {_id: 0, host: '192.168.1.200:38018'},

​             {_id: 1, host: '192.168.1.200:38019'},

​             {_id: 2, host: '192.168.1.200:38020'}]

​      }

 

rs.initiate(config) 

 

  

注：configserver 可以是一个节点，官方建议复制集。configserver不能有arbiter。

新版本中，要求必须是复制集。

注：mongodb 3.4之后，虽然要求config server为replica set，但是不支持arbiter

 

### mongos节点配置：

创建目录：

mkdir -p /mongodb/38017/{conf,log}

 

配置文件：

tee>/mongodb/38017/conf/mongos.conf<<EOF

systemLog:

 destination: file

 path: /mongodb/38017/log/mongos.log

 logAppend: true

net:

 bindIp: 192.168.1.200,127.0.0.1

 port: 38017

sharding:

 configDB: configReplSet/192.168.1.200:38018,192.168.1.200:38019,192.168.1.200:38020

processManagement: 

 fork: true

EOF

4.3启动mongos

mongos -f /mongodb/38017/conf/mongos.conf 

 

 

### 分片集群操作：

 

连接到其中一个mongos（192.168.1.200），做以下配置

（1）连接到mongs的admin数据库

\# su - mongod

$ mongo 192.168.1.200:38017/admin

（2）添加分片

db.runCommand( { addshard : "sh1/192.168.1.200:38021,192.168.1.200:38022,192.168.1.200:38023",name:"shard1"} )

db.runCommand( { addshard : "sh2/192.168.1.200:38024,192.168.1.200:38025,192.168.1.200:38026",name:"shard2"} )

 

（3）列出分片

mongos> db.runCommand( { listshards : 1 } )

 

（4）整体状态查看

mongos> sh.status();

 

#### 使用分片集群

 

\## **RANGE****分片**配置及测试

 

test库下的vast大表进行手工分片

 

1、激活数据库分片功能

mongo --port 38017 admin

admin> ( { enablesharding : "数据库名称" } )

 

eg：

admin> db.runCommand( { enablesharding : "xyz" } )

 

2、指定分片建对集合分片

eg：范围片键

--创建索引

use xyz

\> db.vast.ensureIndex( { id: 1 } )

 

--开启分片

use admin

\> db.runCommand( { shardcollection : "xyz.vast",key : {id: 1} } )

 

3、集合分片验证

admin> use xyz

xyz> for(i=1;i<1000000;i++){ db.vast.insert({"id":i,"name":"shenzheng","age":66,"date":new Date()}); }

test> db.vast.stats()

 

**分片结果测试**

 

shard1:

mongo --port 38021

sh1:PRIMARY> use xyz

switched to db xyz

sh1:PRIMARY> db.vast.count()

500001

 

shard2:

mongo --port 38024

sh2:PRIMARY> use xyz

switched to db xyz

sh2:PRIMARY> db.vast.count()

999999

这里不可以重复操作，操作中不能断电！

重新搞了。。。

mongos> db.vast.drop()

true

mongos> show tables;

vtest

mongos> db.vtest.count()

0

\----------------------------------------------------

#### Hash分片例子：

对xyz库下的vast大表进行hash

创建哈希索引

（1）对于oldboy开启分片功能

mongo --port 38017 admin

use admin

mongos> db.runCommand({enablesharding:"xyz"})

（2）对于xyz库下的vast表建立hash索引

mongos> use xyz

switched to db xyz

mongos> db.vast.ensureIndex({id:"hashed"})

 

（3）开启分片 

use admin

admin > sh.shardCollection( "xyz.vast", { id: "hashed" } )

 

（4）录入10w行数据测试

use xyz

for(i=1;i<100000;i++){ db.vast.insert({"id":i,"name":"shenzheng","age":70,"date":new Date()}); }

 

（5）hash分片结果测试

mongo --port 38021

use xyz

db.vast.count();

 

mongo --port 38024

use xyz

db.vast.count();

\---------------------------

 

5、判断是否Shard集群

admin> db.runCommand({ isdbgrid : 1})

 

6、列出所有分片信息

admin> db.runCommand({ listshards : 1})

 

7、列出开启分片的数据库

admin> use config

 

config> db.databases.find( { "partitioned": true } )

或者：

config> db.databases.find() //列出所有数据库分片情况

 

8、查看分片的片键

mongos> db.collections.find().pretty()

{

​    "_id" : "config.system.sessions",

​    "lastmodEpoch" : ObjectId("60c7729e09c64eadca8ac030"),

​    "lastmod" : ISODate("1970-02-19T17:02:47.296Z"),

​    "dropped" : false,

​    "key" : {

​       "_id" : 1

​    },

​    "unique" : false,

​    "uuid" : UUID("c3e679de-6ef6-42de-9350-0867a12bc539"),

​    "distributionMode" : "sharded"

}

{

​    "_id" : "xyz.vast",

​    "lastmodEpoch" : ObjectId("60c8484810046df92c69576b"),

​    "lastmod" : ISODate("1970-02-19T17:02:47.296Z"),

​    "dropped" : false,

​    "key" : {

​       "id" : 1

​    },

​    "unique" : false,

​    "uuid" : UUID("be7b8ee6-5f99-41a7-ab82-c7aef4d116f6"),

​    "distributionMode" : "sharded"

}

{

​    "_id" : "xyz.vtest",

​    "lastmodEpoch" : ObjectId("000000000000000000000000"),

​    "lastmod" : ISODate("2021-06-15T06:24:44.774Z"),

​    "dropped" : true

}

9、查看分片的详细信息

admin> db.printShardingStatus()

或

admin> sh.status()  *****

 

10、删除分片节点（谨慎）

（1）确认blance是否在工作

sh.getBalancerState()

（2）删除shard2节点(谨慎)

mongos> db.runCommand( { removeShard: "shard2" } )

注意：删除操作一定会立即触发blancer。

 

 

#### balancer操作 *****

 

介绍：

mongos的一个重要功能，自动巡查所有shard节点上的chunk的情况，自动做chunk迁移。

 

什么时候工作？

1、自动运行，会检测系统不繁忙的时候做迁移

2、在做节点删除的时候，立即开始迁移工作

3、balancer只能在预设定的时间窗口内运行 *****

 

有需要时可以关闭和开启blancer（备份的时候）

mongos> sh.stopBalancer()

mongos> sh.startBalancer()

 

#### 自定义 自动平衡进行的时间段

https://docs.mongodb.com/manual/tutorial/manage-sharded-cluster-balancer/#schedule-the-balancing-window

// connect to mongos

 

mongo --port 38017 admin

use config

sh.setBalancerState( true )

db.settings.update({ _id : "balancer" }, { $set : { activeWindow : { start : "3:00", stop : "5:00" } } }, true )

 

sh.getBalancerWindow()

sh.status()

 

关于集合的balancer（了解下）

关闭某个集合的balance

sh.disableBalancing("students.grades")

打开某个集合的balancer

sh.enableBalancing("students.grades")

确定某个集合的balance是开启或者关闭

db.getSiblingDB("config").collections.findOne({_id : "students.grades"}).noBalance;

\----------------

#### 备份恢复 *****

1、备份恢复工具介绍：

（1）**  mongoexport/mongoimport

（2）***** mongodump/mongorestore

 

2、备份工具区别在哪里？

2.1. 

**mongoexport/mongoimport** 导入/导出的是JSON格式或者CSV格式

**mongodump/mongorestore**  导入/导出的是BSON格式。

​    

[root@DB01 /server/tools]# tar xf mongodb-database-tools-rhel70-x86_64-100.3.1.tgz 

[root@DB01 /server/tools]# mv mongodb-database-tools-rhel70-x86_64-100.3.1/bin/* /mongodb/bin/ 

[root@DB01 /server/tools]# ls /mongodb/bin/

bsondump     mongo  mongodump  mongofiles  mongorestore mongostat

install_compass mongod mongoexport mongoimport mongos    mongotop

 

​    

2.2. JSON可读性强但体积较大，BSON则是二进制文件，体积小但对人类几乎没有可读性。

 

2.3. 在一些mongodb版本之间，BSON格式可能会随版本不同而有所不同，**所以不同版本之间用mongodump/mongorestore可能不会成功，具体要看版本之间的兼容性**。当无法使用BSON进行跨版本的数据迁移的时候，

使用JSON格式即mongoexport/mongoimport是一个可选项。

跨版本的mongodump/mongorestore个人并不推荐，实在要做请先检查文档看两个版本是否兼容（大部分时候是的）    

2.4. JSON虽然具有较好的跨版本通用性，但其只保留了数据部分，不保留索引，账户等其他基础信息。使用时应该注意。

 

 

应用场景总结:

mongoexport/mongoimport:json csv 

1、异构平台迁移 mysql <---> mongodb

2、同平台，跨大版本：mongodb 2 ----> mongodb 3

 

mongodump/mongorestore

***日常备份恢复时使用.\***

 

================================================

##### 一、导出工具mongoexport

 

Mongodb中的mongoexport工具可以把一个collection导出成JSON格式或CSV格式的文件。

可以通过参数指定导出的数据项，也可以根据指定的条件导出数据。

（1）版本差异较大

（2）异构平台数据迁移

 

mongoexport具体用法如下所示：

$ mongoexport --help 

参数说明：

-h:指明数据库宿主机的IP

-u:指明数据库的用户名

-p:指明数据库的密码

-d:指明数据库的名字

-c:指明collection的名字

-f:指明要导出那些列

-o:指明到要导出的文件名

-q:指明导出数据的过滤条件

--authenticationDatabase admin

 

准备

my_repl:PRIMARY> db.createUser({

... user:"root",

... pwd:"root123",

... roles:[{role:"root",db:"admin"}]

... })

Successfully added user: {

​    "user" : "root",

​    "roles" : [

​       {

​           "role" : "root",

​           "db" : "admin"

​       }

​    ]

}

my_repl:PRIMARY> db.auth("root","root123")

1

 

 

[mongod@DB01 ~]$ mongo -uroot -proot123 --port 28017 admin

模拟点数据

my_repl:PRIMARY> use oldboy

switched to db oldboy

my_repl:PRIMARY> for(i=1;i<10000;i++){db.student.insert({"id":i,"name":"zhang_$i","age":25,"date":new Date()});}

WriteResult({ "nInserted" : 1 })

 

 

1.单表备份至json格式

[root@DB01 ~]# mongoexport -uroot -proot123 --port 28017 --authenticationDatabase admin -d oldboy -c student -o /mongodb/student.json

2021-06-15T21:08:23.273+0800 connected to: mongodb://localhost:28017/

2021-06-15T21:08:23.706+0800 exported 9999 records

 

注：备份文件的名字可以自定义，默认导出了JSON格式的数据。

 

\2. 单表备份至csv格式

如果我们需要导出CSV格式的数据，则需要使用----type=csv参数：

 

 [root@DB01 ~]# mongoexport -uroot -proot123 --port 28017 --authenticationDatabase admin -d oldboy -c student --type=csv -f id,name,age,date -o /mongodb/student.csv

2021-06-15T21:11:45.579+0800 connected to: mongodb://localhost:28017/

2021-06-15T21:11:46.387+0800 exported 9999 records 

 

##### 二、导入工具mongoimport

Mongodb中的mongoimport工具可以把一个特定格式文件中的内容导入到指定的collection中。该工具可以导入JSON格式数据，也可以导入CSV格式数据。具体使用如下所示：

$ mongoimport --help

参数说明：

-h:指明数据库宿主机的IP

-u:指明数据库的用户名

-p:指明数据库的密码

-d:指明数据库的名字

-c:指明collection的名字

-f:指明要导入那些列

-j, --numInsertionWorkers=<number> number of insert operations to run concurrently                         (defaults to 1)

//并行

 

删除数据

my_repl:PRIMARY> db.dropDatabase()

 

数据恢复:

1.恢复json格式表数据到student1

[root@DB01 ~]# mongoimport -uroot -proot123 --port 28017 --authenticationDatabase admin -d oldboy -c student1 /mongodb/student.json

2021-06-15T21:34:21.124+0800 connected to: mongodb://localhost:28017/

2021-06-15T21:34:22.270+0800 9999 document(s) imported successfully. 0 document(s) failed to import.

 

2.恢复csv格式的文件到log2

上面演示的是导入JSON格式的文件中的内容，如果要导入CSV格式文件中的内容，则需要通过--type参数指定导入格式，具体如下所示：

错误的恢复

 

my_repl:PRIMARY> db.dropDatabase()

{

​    "dropped" : "oldboy",

​    "ok" : 1,

​    "$clusterTime" : {

​       "clusterTime" : Timestamp(1623764748, 2),

​       "signature" : {

​           "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),

​           "keyId" : NumberLong(0)

​       }

​    },

​    "operationTime" : Timestamp(1623764748, 2)

}

my_repl:PRIMARY> db

oldboy

my_repl:PRIMARY> use admin

switched to db admin

my_repl:PRIMARY> show databases

admin  0.000GB

config 0.000GB

local  0.001GB

 

注意：

（1）csv格式的文件头行，有列名字

[root@DB01 ~]# mongoimport -uroot -proot123 --port 28017 --authenticationDatabase admin -d oldboy -c student2 --type=csv --headerline --file /mongodb/student.csv

2021-06-15T21:53:46.349+0800 connected to: mongodb://localhost:28017/

2021-06-15T21:53:47.299+0800 9999 document(s) imported successfully. 0 document(s) failed to import.

（2）csv格式的文件头行，没有列名字

mongoimport  -uroot -proot123 --port 28017 --authenticationDatabase admin  -d oldboy -c student3 --type=csv -f id,name,age,date --file /mongodb/student.csv

 

*--headerline:指明第一行是列名，不需要导入。*

##### -----异构平台迁移案例

mysql  -----> mongodb 

world数据库下city表进行导出，导入到mongodb

 

（1）mysql开启安全路径

 

vim /etc/my.cnf  --->添加以下配置

secure-file-priv=/tmp

 

--重启数据库生效

/etc/init.d/mysqld restart

 

（2）导出mysql的city表数据

source /root/world.sql

 

select * from world.city into outfile '/tmp/city1.csv' fields terminated by ',';

 

（3）处理备份文件

desc world.city

 ID       | int(11)    | NO  | PRI | NULL  | auto_increment |

| Name     | char(35)     | NO  |   |     |        |

| CountryCode    | char(3)   | NO  | MUL |     |        |

| District       | char(20)    | NO  |   |     |        |

| Population

 

vim /tmp/city.csv  ----> 添加第一行列名信息

ID,Name,CountryCode,District,Population

 

(4)在mongodb中导入备份

mongoimport -uroot -proot123 --port 28017 --authenticationDatabase admin -d world -c city --type=csv -f ID,Name,CountryCode,District,Population --file /tmp/city1.csv

 

use world

db.city.find({CountryCode:"CHN"});

 

\-------------

world共100张表，全部迁移到mongodb

 

select * from world.city into outfile '/tmp/world_city.csv' fields terminated by ',';

 

select concat("select * from ",table_schema,".",table_name ," into outfile '/tmp/",table_schema,"_",table_name,".csv' fields terminated by ',';")

from information_schema.tables where table_schema ='world';

 

导入：

   **提示，使用infomation_schema.columns + information_schema.tables**

 

\------------

 

mysql导出csv：

select * from test_info  

into outfile '/tmp/test.csv'  

fields terminated by ','　　　 ------字段间以,号分隔

optionally enclosed by '"'　　 ------字段用"号括起

escaped by '"'  　　　　　　 ------字段中使用的转义符为"

lines terminated by '\r\n';　　------行以\r\n结束

 

 

mysql导入csv：

load data infile '/tmp/test.csv'  

into table test_info  

fields terminated by ',' 

optionally enclosed by '"' 

escaped by '"'  

lines terminated by '\r\n'; 

 

\----------------------------------

##### 三、mongodump和mongorestore介绍

 

mongodump能够在Mongodb运行时进行备份，它的工作原理是对运行的Mongodb做查询，然后将所有查到的文档写入磁盘。但是存在的问题时使用mongodump产生的备份不一定是数据库的实时快照，如果我们在备份时对数据库进行了写入操作，则备份出来的文件可能不完全和Mongodb实时数据相等。另外在备份时可能会对其它客户端性能产生不利的影响。

 

**mongodump用法如下：**

$ mongodump --help

参数说明：

-h:指明数据库宿主机的IP

-u:指明数据库的用户名

-p:指明数据库的密码

-d:指明数据库的名字

-c:指明collection的名字

-o:指明到要导出的文件名

-q:指明导出数据的过滤条件

-j, --numParallelCollections= number of collections to dump in parallel (4 by default)

--oplog 备份的同时备份oplog

 

 

**mongodump和mongorestore基本使用**

**全库备份**

 

mkdir /mongodb/backup

mongodump -uroot -proot123 --port 28017 --authenticationDatabase admin -o /mongodb/backup

 

**--备份world库**

 mongodump  -uroot -proot123 --port 28017 --authenticationDatabase admin -d world -o /mongodb/backup/

 

--备份oldboy库下的student集合

mongodump  -uroot -proot123 --port 28017 --authenticationDatabase admin -d oldboy -c student -o /mongodb/backup/

 

 --压缩备份

$ mongodump  -uroot -proot123 --port 28017 --authenticationDatabase admin -d oldguo -o /mongodb/backup/ --gzip

 mongodump  -uroot -proot123 --port 28017 --authenticationDatabase admin -o /mongodb/backup/ --gzip

$ mongodump  -uroot -proot123 --port 28017 --authenticationDatabase admin -d app -c vast -o /mongodb/backup/ --gzip

 

--恢复world库

$ mongorestore  -uroot -proot123 --port 28017 --authenticationDatabase admin -d world1  /mongodb/backup/world

 

--恢复oldguo库下的t1集合

[mongod@db03 oldboy]$ mongorestore  -uroot -proot123 --port 28017 --authenticationDatabase admin -d world -c t1 --gzip /mongodb/backup.bak/oldboy/log1.bson.gz 

 

--drop表示恢复的时候把之前的集合drop掉(危险)

$ mongorestore -uroot -proot123 --port 28017 --authenticationDatabase admin -d oldboy --drop /mongodb/backup/oldboy

 

##### *****mongodump和mongorestore高级企业应用（--oplog）

 

***注意：这是replica set或者master/slave模式专用\***

 

--oplog

 use oplog for taking a point-in-time snapshot

 

oplog介绍

在replica set中oplog是一个定容集合（capped collection），它的默认大小是磁盘空间的5%（可以通过--oplogSizeMB参数修改）.

 

位于local库的db.oplog.rs，有兴趣可以看看里面到底有些什么内容。

其中记录的是整个mongod实例一段时间内数据库的所有变更（插入/更新/删除）操作。

当空间用完时新记录自动覆盖最老的记录。

其覆盖范围被称作oplog时间窗口。需要注意的是，因为oplog是一个定容集合，

所以时间窗口能覆盖的范围会因为你单位时间内的更新次数不同而变化。

想要查看当前的oplog时间窗口预计值，可以使用以下命令：

my_repl:PRIMARY> db.oplog.rs.find()

 

 mongod -f /mongodb/28017/conf/mongod.conf 

 mongod -f /mongodb/28018/conf/mongod.conf 

 mongod -f /mongodb/28019/conf/mongod.conf 

 mongod -f /mongodb/28020/conf/mongod.conf 

 

 

 use local 

 db.oplog.rs.find().pretty()

 

   "ts" : Timestamp(1553597844, 1),

​    "op" : "n"

​    "o" :

 

  "i": insert

  "u": update

  "d": delete

  "c": db cmd

​    

\------------

test:PRIMARY> rs.printReplicationInfo()

configured oplog size:  1561.5615234375MB <--集合大小

log length start to end: 423849secs (117.74hrs) <--预计窗口覆盖时间

oplog first event time: Wed Sep 09 2015 17:39:50 GMT+0800 (CST)

oplog last event time:  Mon Sep 14 2015 15:23:59 GMT+0800 (CST)

now:           Mon Sep 14 2015 16:37:30 GMT+0800 (CST)

 

 

\------------

##### oplog企业级应用

（1）实现热备，在备份时使用--oplog选项

注：为了演示效果我们在备份过程，模拟数据插入

（2）准备测试数据

use oldboy

for(var i = 1 ;i < 100; i++) {

  db.foo.insert({a:i});

}

my_repl:PRIMARY> db.oplog.rs.find({"op":"i"}).pretty()

\-----------------------------------------------------

oplog 配合mongodump实现热备

mongodump --port 28017 --oplog -o /mongodb/backup

作用介绍：--oplog 会记录备份过程中的数据变化。会以oplog.bson保存下来

 

恢复

mongorestore --port 28017 --oplogReplay /mongodb/backup

 

!!!!!!!!!!**oplog****高级应用** ==========binlog应用

 

背景：每天0点全备，oplog恢复窗口为48小时

某天，上午10点world.city 业务表被误删除。

 

恢复思路：

​    0、停应用

​    2、找测试库

​    3、恢复昨天晚上全备

​    4、截取全备之后到world.city误删除时间点的oplog，并恢复到测试库

​    5、将误删除表导出，恢复到生产库

 

​    

\--------------

恢复步骤：

模拟故障环境：

1、全备数据库

 

模拟原始数据

 

mongo --port 28017

use wo

for(var i = 1 ;i < 20; i++) {

  db.ci.insert({a: i});

}

全备:

rm -rf /mongodb/backup/*

mongodump --port 28017 --oplog -o /mongodb/backup

--oplog功能:在备份同时,将备份过程中产生的日志进行备份

 

文件必须存放在/mongodb/backup下,自动命令为oplog.bson

再次模拟数据

db.ci1.insert({id:1})

db.ci2.insert({id:2})

 

 

2、上午10点：删除wo库下的ci表

10:00时刻,误删除

db.ci.drop()

show tables;

 

3、备份现有的oplog.rs表

mongodump --port 28017 -d local -c oplog.rs -o /mongodb/backup

 

4、截取oplog并恢复到drop之前的位置

更合理的方法：登陆到原数据库

[mongod@db03 local]$ mongo --port 28017

my_repl:PRIMARY> use local

db.oplog.rs.find({op:"c"}).pretty();

 

{

​    "ts" : Timestamp(1553659908, 1),

​    "t" : NumberLong(2),

​    "h" : NumberLong("-7439981700218302504"),

​    "v" : 2,

​    "op" : "c",

​    "ns" : "wo.$cmd",

​    "ui" : UUID("db70fa45-edde-4945-ade3-747224745725"),

​    "wall" : ISODate("2019-03-27T04:11:48.890Z"),

​    "o" : {

​       "drop" : "ci"

​    }

}

 

"ts" : Timestamp(1563958129, 1),

 

获取到oplog误删除时间点位置:

"ts" : Timestamp(1553659908, 1)

​    

 5、恢复备份+应用oplog

[mongod@db03 backup]$ cd /mongodb/backup/local/

[mongod@db03 local]$ ls

oplog.rs.bson oplog.rs.metadata.json

[mongod@db03 local]$ cp oplog.rs.bson ../oplog.bson 

rm -rf /mongodb/backup/local/

mongorestore --port 28018 --oplogReplay --oplogLimit "1563958129:1" --drop  /mongodb/backup/

*恢复数据再把日志恢复到删除之前，有点像mysql 二进制日志的那种思路*

\-----------------------------------------

**分片集群的备份思路（了解）

 

1、你要备份什么？

config server

shard 节点

 

单独进行备份

2、备份有什么困难和问题

（1）chunk迁移的问题

​    人为控制在备份的时候，避开迁移的时间窗口

（2）shard节点之间的数据不在同一时间点。

​    选业务量较少的时候      

​       

Ops Manager 