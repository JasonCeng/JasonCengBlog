# Linux下安装ZooKeeper
本文基于Ubuntu 16.04 LTS

## 环境依赖
- Java 8+
- 防火墙开放2181端口：
```shell
ufw allow 2181
ufw reload
```

## 一、下载与解压ZooKeeper
### 1、下载
```shell
$ wget https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/zookeeper-3.6.2/apache-zookeeper-3.6.2-bin.tar.gz
```

### 2、解压到指定位置
```shell
$ tar -zxvf apache-zookeeper-3.6.2-bin.tar.gz -C /usr/local

$ mv /usr/local/apache-zookeeper-3.6.2-bin /usr/local/zookeeper_3.6.2

$ cd /usr/local/zookeeper_3.6.2
```

### 3、设置环境变量
```shell
vim /etc/profile

#添加如下内容
export ZOOKEEPER_HOME=/usr/local/zookeeper_3.6.2
export PATH=$ZOOKEEPER_HOME/bin:$PATH

#激活环境变量
source /etc/profile
```

## 二、配置文件
### 1、创建配置文件
利用模板文件zoo_sample.cfg，通过拷贝创建配置文件zoo.cfg
```shell
$ cd /usr/local/zookeeper_3.6.2/conf
$ cp zoo_sample.cfg zoo.cfg
```

### 2、创建数据目录
```shell
$ mkdir /usr/local/zookeeper_3.6.2/data
```

### 3、修改配置文件dataDir
```shell
$ vim /usr/local/zookeeper_3.6.2/conf/zoo.cfg

#修改dataDir为如下内容
dataDir=/usr/local/zookeeper_3.6.2/data
```

## 三、启动ZooKeeper服务器
### 1、启动zk
默认是以standlone(单机)模式运行
```shell
$ cd /usr/local/zookeeper_3.6.2

# 前台运行
$ bin/zkServer.sh start

# 后台运行
$ nohup bin/zkServer.sh start

# 前台运行输出如下信息：
# ZooKeeper JMX enabled by default
# Using config: /usr/local/zookeeper_3.6.2/bin/../conf/zoo.cfg
# Starting zookeeper ... STARTED
```

另，关闭zookeeper命令：
```shell
$ bin/zkServer.sh stop
```

### 2、查看zk状态
```shell
$ bin/zkServer.sh status

# 输出如下信息：
# ZooKeeper JMX enabled by default
# Using config: /usr/local/zookeeper_3.6.2/bin/../conf/zoo.cfg
# Client port found: 2181. Client address: localhost. Client SSL: false.
# Mode: standalone
```

### 3、启动zk客户端
```shell
$ bin/zkCli.sh -server 127.0.0.1:2181

# 输出如下信息,说明zk客户端启动成功：
Connecting to 127.0.0.1:2181
2021-02-06 21:20:48,557 [myid:] - INFO  [main:Environment@98] - Client environment:zookeeper.version=3.6.2--803c7f1a12f85978cb049af5e4ef23bd8b688715, built on 09/04/2020 12:44 GMT
2021-02-06 21:20:48,560 [myid:] - INFO  [main:Environment@98] - Client environment:host.name=ubuntu-master
2021-02-06 21:20:48,560 [myid:] - INFO  [main:Environment@98] - Client environment:java.version=1.8.0_281
2021-02-06 21:20:48,562 [myid:] - INFO  [main:Environment@98] - Client environment:java.vendor=Oracle Corporation
2021-02-06 21:20:48,562 [myid:] - INFO  [main:Environment@98] - Client environment:java.home=/usr/jdk-8/jre
2021-02-06 21:20:48,562 [myid:] - INFO  [main:Environment@98] - Client environment:java.class.path=/usr/local/zookeeper_3.6.2/bin/../zookeeper-server/target/classes:/usr/local/zookeeper_3.6.2/bin/../build/classes:/usr/local/zookeeper_3.6.2/bin/../zookeeper-server/target/lib/*.jar:/usr/local/zookeeper_3.6.2/bin/../build/lib/*.jar:/usr/local/zookeeper_3.6.2/bin/../lib/zookeeper-prometheus-metrics-3.6.2.jar:/usr/local/zookeeper_3.6.2/bin/../lib/zookeeper-jute-3.6.2.jar:/usr/local/zookeeper_3.6.2/bin/../lib/zookeeper-3.6.2.jar:/usr/local/zookeeper_3.6.2/bin/../lib/snappy-java-1.1.7.jar:/usr/local/zookeeper_3.6.2/bin/../lib/slf4j-log4j12-1.7.25.jar:/usr/local/zookeeper_3.6.2/bin/../lib/slf4j-api-1.7.25.jar:/usr/local/zookeeper_3.6.2/bin/../lib/simpleclient_servlet-0.6.0.jar:/usr/local/zookeeper_3.6.2/bin/../lib/simpleclient_hotspot-0.6.0.jar:/usr/local/zookeeper_3.6.2/bin/../lib/simpleclient_common-0.6.0.jar:/usr/local/zookeeper_3.6.2/bin/../lib/simpleclient-0.6.0.jar:/usr/local/zookeeper_3.6.2/bin/../lib/netty-transport-native-unix-common-4.1.50.Final.jar:/usr/local/zookeeper_3.6.2/bin/../lib/netty-transport-native-epoll-4.1.50.Final.jar:/usr/local/zookeeper_3.6.2/bin/../lib/netty-transport-4.1.50.Final.jar:/usr/local/zookeeper_3.6.2/bin/../lib/netty-resolver-4.1.50.Final.jar:/usr/local/zookeeper_3.6.2/bin/../lib/netty-handler-4.1.50.Final.jar:/usr/local/zookeeper_3.6.2/bin/../lib/netty-common-4.1.50.Final.jar:/usr/local/zookeeper_3.6.2/bin/../lib/netty-codec-4.1.50.Final.jar:/usr/local/zookeeper_3.6.2/bin/../lib/netty-buffer-4.1.50.Final.jar:/usr/local/zookeeper_3.6.2/bin/../lib/metrics-core-3.2.5.jar:/usr/local/zookeeper_3.6.2/bin/../lib/log4j-1.2.17.jar:/usr/local/zookeeper_3.6.2/bin/../lib/json-simple-1.1.1.jar:/usr/local/zookeeper_3.6.2/bin/../lib/jline-2.14.6.jar:/usr/local/zookeeper_3.6.2/bin/../lib/jetty-util-9.4.24.v20191120.jar:/usr/local/zookeeper_3.6.2/bin/../lib/jetty-servlet-9.4.24.v20191120.jar:/usr/local/zookeeper_3.6.2/bin/../lib/jetty-server-9.4.24.v20191120.jar:/usr/local/zookeeper_3.6.2/bin/../lib/jetty-security-9.4.24.v20191120.jar:/usr/local/zookeeper_3.6.2/bin/../lib/jetty-io-9.4.24.v20191120.jar:/usr/local/zookeeper_3.6.2/bin/../lib/jetty-http-9.4.24.v20191120.jar:/usr/local/zookeeper_3.6.2/bin/../lib/javax.servlet-api-3.1.0.jar:/usr/local/zookeeper_3.6.2/bin/../lib/jackson-databind-2.10.3.jar:/usr/local/zookeeper_3.6.2/bin/../lib/jackson-core-2.10.3.jar:/usr/local/zookeeper_3.6.2/bin/../lib/jackson-annotations-2.10.3.jar:/usr/local/zookeeper_3.6.2/bin/../lib/commons-lang-2.6.jar:/usr/local/zookeeper_3.6.2/bin/../lib/commons-cli-1.2.jar:/usr/local/zookeeper_3.6.2/bin/../lib/audience-annotations-0.5.0.jar:/usr/local/zookeeper_3.6.2/bin/../zookeeper-*.jar:/usr/local/zookeeper_3.6.2/bin/../zookeeper-server/src/main/resources/lib/*.jar:/usr/local/zookeeper_3.6.2/bin/../conf:.:.::/usr/jdk-8/lib:/usr/jdk-8/jre/lib:/usr/jdk-8/lib:/usr/jdk-8/jre/lib
2021-02-06 21:20:48,563 [myid:] - INFO  [main:Environment@98] - Client environment:java.library.path=/usr/java/packages/lib/amd64:/usr/lib64:/lib64:/lib:/usr/lib
2021-02-06 21:20:48,563 [myid:] - INFO  [main:Environment@98] - Client environment:java.io.tmpdir=/tmp
2021-02-06 21:20:48,563 [myid:] - INFO  [main:Environment@98] - Client environment:java.compiler=<NA>
2021-02-06 21:20:48,563 [myid:] - INFO  [main:Environment@98] - Client environment:os.name=Linux
2021-02-06 21:20:48,563 [myid:] - INFO  [main:Environment@98] - Client environment:os.arch=amd64
2021-02-06 21:20:48,563 [myid:] - INFO  [main:Environment@98] - Client environment:os.version=4.4.0-201-generic
2021-02-06 21:20:48,563 [myid:] - INFO  [main:Environment@98] - Client environment:user.name=root
2021-02-06 21:20:48,563 [myid:] - INFO  [main:Environment@98] - Client environment:user.home=/root
2021-02-06 21:20:48,563 [myid:] - INFO  [main:Environment@98] - Client environment:user.dir=/usr/local/zookeeper_3.6.2
2021-02-06 21:20:48,564 [myid:] - INFO  [main:Environment@98] - Client environment:os.memory.free=173MB
2021-02-06 21:20:48,565 [myid:] - INFO  [main:Environment@98] - Client environment:os.memory.max=228MB
2021-02-06 21:20:48,565 [myid:] - INFO  [main:Environment@98] - Client environment:os.memory.total=180MB
2021-02-06 21:20:48,571 [myid:] - INFO  [main:ZooKeeper@1006] - Initiating client connection, connectString=127.0.0.1:2181 sessionTimeout=30000 watcher=org.apache.zookeeper.ZooKeeperMain$MyWatcher@379619aa
2021-02-06 21:20:48,578 [myid:] - INFO  [main:X509Util@77] - Setting -D jdk.tls.rejectClientInitiatedRenegotiation=true to disable client-initiated TLS renegotiation
2021-02-06 21:20:48,585 [myid:] - INFO  [main:ClientCnxnSocket@239] - jute.maxbuffer value is 1048575 Bytes
2021-02-06 21:20:48,591 [myid:] - INFO  [main:ClientCnxn@1716] - zookeeper.request.timeout value is 0. feature enabled=false
Welcome to ZooKeeper!
2021-02-06 21:20:48,600 [myid:127.0.0.1:2181] - INFO  [main-SendThread(127.0.0.1:2181):ClientCnxn$SendThread@1167] - Opening socket connection to server localhost/127.0.0.1:2181.
2021-02-06 21:20:48,600 [myid:127.0.0.1:2181] - INFO  [main-SendThread(127.0.0.1:2181):ClientCnxn$SendThread@1169] - SASL config status: Will not attempt to authenticate using SASL (unknown error)
2021-02-06 21:20:48,606 [myid:127.0.0.1:2181] - INFO  [main-SendThread(127.0.0.1:2181):ClientCnxn$SendThread@999] - Socket connection established, initiating session, client: /127.0.0.1:44658, server: localhost/127.0.0.1:2181
JLine support is enabled
2021-02-06 21:20:48,617 [myid:127.0.0.1:2181] - INFO  [main-SendThread(127.0.0.1:2181):ClientCnxn$SendThread@1433] - Session establishment complete on server localhost/127.0.0.1:2181, session id = 0x100003851180002, negotiated timeout = 30000

WATCHER::

WatchedEvent state:SyncConnected type:None path:null
[zk: 127.0.0.1:2181(CONNECTED) 0] 
```

在`[zk: 127.0.0.1:2181(CONNECTED) 0]`终端输入`help`可以查看可在客户端执行的一系列命令：
```shell
[zk: 127.0.0.1:2181(CONNECTED) 0] help
ZooKeeper -server host:port -client-configuration properties-file cmd args
	addWatch [-m mode] path # optional mode is one of [PERSISTENT, PERSISTENT_RECURSIVE] - default is PERSISTENT_RECURSIVE
	addauth scheme auth
	close 
	config [-c] [-w] [-s]
	connect host:port
	create [-s] [-e] [-c] [-t ttl] path [data] [acl]
	delete [-v version] path
	deleteall path [-b batch size]
	delquota [-n|-b] path
	get [-s] [-w] path
	getAcl [-s] path
	getAllChildrenNumber path
	getEphemerals path
	history 
	listquota path
	ls [-s] [-w] [-R] path
	printwatches on|off
	quit 
	reconfig [-s] [-v version] [[-file path] | [-members serverID=host:port1:port2;port3[,...]*]] | [-add serverId=host:port1:port2;port3[,...]]* [-remove serverId[,...]*]
	redo cmdno
	removewatches path [-c|-d|-a] [-l]
	set [-s] [-v version] path data
	setAcl [-s] [-v version] [-R] path acl
	setquota -n|-b val path
	stat [-w] path
	sync path
	version 
Command not found: Command not found help

```

尝试`ls`命令：
```shell
[zk: 127.0.0.1:2181(CONNECTED) 1] ls /
[zookeeper]
```
创建一个新的znode，并关联字符串“my_data”到这个节点上：
```shell
[zk: 127.0.0.1:2181(CONNECTED) 2] create /zk_test my_data
Created /zk_test
```

再次`ls`查看当前目录结构：
```shell
[zk: 127.0.0.1:2181(CONNECTED) 3] ls /
[zk_test, zookeeper]
```
可以看到`zk_test`目录已经被创建。

接下来，通过`get`命令验证data是否已和znode关联：
```shell
[zk: 127.0.0.1:2181(CONNECTED) 4] get /zk_test
my_data
```

我们可以通过`set`命令修改znode的相关数据：
```shell
[zk: 127.0.0.1:2181(CONNECTED) 5] set /zk_test junk
[zk: 127.0.0.1:2181(CONNECTED) 6] get /zk_test 
junk
```

最后，我们使用`delete`命令删除node：
```shell
[zk: 127.0.0.1:2181(CONNECTED) 7] delete /zk_test 
[zk: 127.0.0.1:2181(CONNECTED) 8] ls /
[zookeeper]
```

### 4、原理：客户端建立连接的步骤

- 1.客户端启动程序建立一个会话

- 2.客户端尝试连接zookeeper 主机

- 3.客户端连接成功，服务器尝试初始化这个新的会话

- 4.会话初始化完成

- 5.服务器端向客户端发送一个SyncConnect连接

## 参考
[1]https://zookeeper.apache.org/doc/r3.6.2/zookeeperStarted.html

[2]zookeeper系列 （第一章 ：ubuntu 下安装zookeeper）[https://www.cnblogs.com/blogxiao/p/10738818.html]