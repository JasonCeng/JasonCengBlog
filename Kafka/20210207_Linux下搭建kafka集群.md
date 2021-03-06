# Linux下搭建Kafka集群

## 摘要
Kafka 是一个分布式的基于push-subscribe的消息系统，它具备快速、可扩展、可持久化的特点。由 LinkedIn 开源，用作 LinkedIn 的活动流（Activity Stream）和运营数据处理管道（Pipeline）的基础。现在是Apache旗下的一个开源系统，作为Hadoop生态系统的一部分，被各种商业公司广泛应用。它的最大的特性就是可以实时的处理大量数据以满足各种需求场景：比如基于Hadoop的批处理系统、低延迟的实时系统、Flink/Storm/Spark流式处理引擎。可以说是现代分布式系统的基石，学习kafka的使用、原理变得十分必要。

本文基于Ubuntu 16.04 LTS，介绍如何搭建1主2备的kafka集群。

## 环境依赖
- 服务器数量 >= 2
- Java 8+
- 搭建好ZooKeeper集群
- 防火墙开放9092端口：
```shell
ufw allow 9092
ufw reload
```

## 一、下载与解压kafka
### 1、下载
```shell
$ wget https://mirrors.tuna.tsinghua.edu.cn/apache/kafka/2.7.0/kafka_2.13-2.7.0.tgz
```

### 2、解压到指定位置
```shell
$ tar -xzvf kafka_2.13-2.7.0.tgz -C /usr/local
$ cd /usr/local/kafka_2.13-2.7.0
```

## 二、配置文件
### 1、创建kafka消息存放目录
```shell
# 创建kafka消息存放目录,配置文件server.properties中的log.dir需要用到
$ mkdir /usr/local/kafka_2.13-2.7.0/kafka_logs
```
### 2、配置server.properties
```shell
$ vim /usr/local/kafka_2.13-2.7.0/config/server.properties

# 修改如下内容，下面以服务器1：我的主机名是：ubuntu-master为例

#每个broker在集群中的唯一标识，不能重复,和zookeeper的myid性质一样
broker.id=0
#端口
port=9092
#broker主机地址
host.name=ubuntu-master.com

#消息存放的目录，这个目录可以配置为“，”逗号分割的表达式，上面的num.io.threads要大于这个目录的个数
#如果配置多个目录，新创建的topic把消息持久化的地方是：当前以逗号分割的目录中，哪个分区数最少就放哪一个
log.dirs=/usr/local/kafka_2.13-2.7.0/kafka_logs

#在log.retention.hours=168 下面新增下面三项:
#消息保存的最大值5M
message.max.byte=5242880
#kafka保存消息的副本数，如果一个副本失效了，另一个还可以继续提供服务
default.replication.factor=2
#取消息的最大直接数
replica.fetch.max.bytes=5242880

#zookeeper集群地址
zookeeper.connect=ubuntu-master.com:2181,ubuntu-slave1.com:2181,ubuntu-slave2.com:2181
```

### 3、[选看]server.properties相关参数解析
```shell
broker.id=0  #当前机器在集群中的唯一标识，和zookeeper的myid性质一样
port=9092 #当前kafka对外提供服务的端口默认是9092
host.name=192.168.1.113 #这个参数默认是关闭的，在0.8.1有个bug，DNS解析问题，失败率的问题。
num.network.threads=3 #这个是borker进行网络处理的线程数
num.io.threads=8 #这个是borker进行I/O处理的线程数
log.dirs=/opt/kafka/kafkalogs/ #消息存放的目录，这个目录可以配置为“，”逗号分割的表达式，上面的num.io.threads要大于这个目录的个数。如果配置多个目录，新创建的topic他把消息持久化的地方是：当前以逗号分割的目录中，那个分区数最少就放那一个
socket.send.buffer.bytes=102400 #发送缓冲区buffer大小，数据不是一下子就发送的，先回存储到缓冲区了到达一定的大小后在发送，能提高性能
socket.receive.buffer.bytes=102400 #kafka接收缓冲区大小，当数据到达一定大小后在序列化到磁盘
socket.request.max.bytes=104857600 #这个参数是向kafka请求消息或者向kafka发送消息的请请求的最大数，这个值不能超过java的堆栈大小
num.partitions=1 #默认的分区数，一个topic默认1个分区数
log.retention.hours=168 #默认消息的最大持久化时间，168小时，7天
message.max.byte=5242880  #消息保存的最大值5M
default.replication.factor=2  #kafka保存消息的副本数，如果一个副本失效了，另一个还可以继续提供服务
replica.fetch.max.bytes=5242880  #取消息的最大直接数
log.segment.bytes=1073741824 #这个参数是：因为kafka的消息是以追加的形式落地到文件，当超过这个值的时候，kafka会新起一个文件
log.retention.check.interval.ms=300000 #每隔300000毫秒去检查上面配置的log失效时间（log.retention.hours=168 ），到目录查看是否有过期的消息如果有，删除
log.cleaner.enable=false #是否启用log压缩，一般不用启用，启用的话可以提高性能
zookeeper.connect=192.168.1.113:2181,192.168.1.114:2181,192.168.1.115:218 #设置zookeeper的连接端口
```

## 三、将Kafka分发到各个节点
```shell
$ cd /usr/local/
$ scp -r kafka_2.13-2.7.0 root@192.168.1.114:/usr/local/
$ scp -r kafka_2.13-2.7.0 root@192.168.1.115:/usr/local/
```

## 四、修改其他节点配置文件

### 1、修改服务器2的server.properties
```shell
$ vim /usr/local/kafka_2.13-2.7.0/config/server.properties

# 需修改内容如下，我的主机名是：ubuntu-slave1

#每个broker在集群中的唯一标识，不能重复,和zookeeper的myid性质一样
broker.id=1
#端口
port=9092
#broker主机地址
host.name=ubuntu-slave1.com
```

### 2、修改服务器3的server.properties
```shell
$ vim /usr/local/kafka_2.13-2.7.0/config/server.properties

# 需修改内容如下，我的主机名是：ubuntu-slave2

#每个broker在集群中的唯一标识，不能重复,和zookeeper的myid性质一样
broker.id=2
#端口
port=9092
#broker主机地址
host.name=ubuntu-slave2.com
```

## 五、配置环境变量
需要在各个服务器上进行配置：
```shell
vim /etc/profile

#添加如下内容
export KAFKA_HOME=/usr/local/kafka_2.13-2.7.0
export PATH=$KAFKA_HOME/bin:$PATH

#激活环境变量
source /etc/profile
```

## 六、启动Kafka集群并测试
### 1、启动服务
```shell
#从后台启动Kafka集群（3台都需要启动）
$ cd /usr/local/kafka_2.13-2.7.0/bin #进入到kafka的bin目录 
$ ./kafka-server-start.sh -daemon ../config/server.properties
```

停止服务：
```shell
$ cd /usr/local/kafka_2.13-2.7.0/bin #进入到kafka的bin目录 
$ ./kafka-server-stop.sh
```

### 2、检查服务是否启动
```shell
#执行命令jps
3749 Jps
1767 QuorumPeerMain
3673 Kafka
```

### 3、创建Topic来验证是否创建成功

更多请看官方文档：http://kafka.apache.org/documentation.html
```shell
$ cd /usr/local/kafka_2.13-2.7.0/bin #进入到kafka的bin目录 

#创建Topic
$ ./kafka-topics.sh --create --zookeeper 192.168.1.113:2181,192.168.1.114:2181,192.168.1.115:2181 --replication-factor 2 --partitions 1 --topic foo
#解释
# --create  表示创建
# --zookeeper 192.168.1.113:2181  后面的参数是zk的集群节点
# --replication-factor 2  表示复本数
# --partitions 1  表示分区数
# --topic foo  表示主题名称为foo

#查看topic 列表：
$ ./kafka-topics.sh --list --zookeeper 192.168.1.113:2181,192.168.1.114:2181,192.168.1.115:2181
    
#查看指定topic：
$ ./kafka-topics.sh --describe --zookeeper 192.168.1.113:2181,192.168.1.114:2181,192.168.1.115:2181 --topic foo

#删除topic
$ ./kafka-topics.sh --delete --zookeeper 192.168.1.113:2181,192.168.1.114:2181,192.168.1.115:2181 --topic foo
```

### 4、创建生产者
```shell
'''在一台服务器上创建一个生产者'''
#创建一个broker，生产者
$ ./kafka-console-producer.sh --broker-list 192.168.1.113:9092,192.168.1.114:9092,192.168.1.115:9092 --topic foo
```

### 5、创建消费者
```shell
'''在另一台服务器上创建一个消费者'''
$ ./kafka-console-consumer.sh --bootstrap-server 192.168.1.113:9092,192.168.1.114:9092,192.168.1.115:9092 --topic foo --from-beginning
```

*注：Kafka 从 2.2 版本开始将` kafka-topic.sh `脚本中的 ` −−zookeeper `参数标注为 “过时”，推荐使用 ` −−bootstrap-server `参数。*

*端口也由之前的zookeeper通信端口`2181`，改为了kafka通信端口`9092`。*

## 参考
[1]Kafka【第一篇】Kafka集群搭建[https://www.cnblogs.com/luotianshuai/p/5206662.html]

[2]kafka集群环境搭建[https://www.cnblogs.com/luzhanshi/p/13369834.html]