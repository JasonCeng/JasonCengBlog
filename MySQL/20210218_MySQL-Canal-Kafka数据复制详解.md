# MySQL-Canal-Kafka数据复制详解

## 摘要
MySQL被广泛用于海量业务的存储数据库，在大数据时代，我们亟需对其中的海量数据进行分析，但在MySQL之上进行大数据分析显然是不现实的，这会影响业务系统的运行稳定。如果我们要实时地分析这些数据，则需要实时地将其复制到适合OLAP的数据系统上。本文介绍一种CDC工具——Canal，由阿里巴巴开源，且广泛用于阿里的生产系统，它模拟MySQL Slave结点，实时获取变化的binlog，我们将把canal获取到的binlog投递到kafka上以供后续系统消费。

本文基于Ubuntu 16.04 LTS

## 环境说明
- Java 8+
- 搭建好ZooKeeper集群
- 搭建好Kafka集群

若未搭建ZooKeeper集群、Kafka集群，可参考：

<a href='https://github.com/JasonCeng/JasonCengBlog/blob/main/zookeeper/20210206_Linux%E4%B8%8B%E6%90%AD%E5%BB%BAZooKeeper%E9%9B%86%E7%BE%A4.md' target='_blank'>Linux下搭建ZooKeeper集群.md</a>

<a href='https://github.com/JasonCeng/JasonCengBlog/blob/main/Kafka/20210207_Linux%E4%B8%8B%E6%90%AD%E5%BB%BAkafka%E9%9B%86%E7%BE%A4.md' target='_blank'>Linux下搭建kafka集群.md</a>

## 一、源MySQL配置
### 1、开启 Binlog 写入功能

对于自建 MySQL , 需要先开启 Binlog 写入功能，配置 binlog-format 为 ROW 模式，my.cnf 中配置如下
```shell
$ vim /etc/my.cnf

[mysqld]
log-bin=mysql-bin # 开启 binlog
binlog-format=ROW # 选择 ROW 模式
server_id=1 # 配置 MySQL replaction 需要定义，不能和 canal 的 slaveId 重复

#重启MySQL数据库
$ service mysql restart
```

### 2、创建并授权canal用户
授权 canal 链接 MySQL 账号具有作为 MySQL slave 的权限, 如果已有账户可直接 grant

```shell
> CREATE USER canal IDENTIFIED BY 'canal';  
> GRANT SELECT, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'canal'@'%';
-->  GRANT ALL PRIVILEGES ON *.* TO 'canal'@'%' ;
> FLUSH PRIVILEGES;
```

## 二、安装ZooKeeper

详细请参考：<a href='https://github.com/JasonCeng/JasonCengBlog/blob/main/zookeeper/20210206_Linux%E4%B8%8B%E6%90%AD%E5%BB%BAZooKeeper%E9%9B%86%E7%BE%A4.md' target='_blank'>JasonCengBlog/zookeeper/20210206_Linux下搭建ZooKeeper集群.md</a>

1、在**所有节点**上启动zkServer
```shell
$ zkServer.sh start&
```

2、查看节点状态
```shell
$ zkServer.sh status
```

## 三、安装KafKa

详细请参考：<a href='https://github.com/JasonCeng/JasonCengBlog/blob/main/Kafka/20210207_Linux%E4%B8%8B%E6%90%AD%E5%BB%BAkafka%E9%9B%86%E7%BE%A4.md' target='_blank'>JasonCengBlog/Kafka/20210207_Linux下搭建kafka集群.md</a>

### 1、在**所有节点**上启动kafka
```shell
#从后台启动Kafka集群（3台都需要启动）
$ cd /usr/local/kafka_2.13-2.7.0/bin #进入到kafka的bin目录 
$ ./kafka-server-start.sh -daemon ../config/server.properties

#查看kafka是否启动
$ jps
```

### 2、创建与查看Topic
```shell
$ cd /usr/local/kafka_2.13-2.7.0/bin #进入到kafka的bin目录 

#创建Topic
$ ./kafka-topics.sh --create --zookeeper 192.168.1.113:2181,192.168.1.114:2181,192.168.1.115:2181 --replication-factor 2 --partitions 1 --topic hello_canal
#解释
# --create  表示创建
# --zookeeper 192.168.1.113:2181  后面的参数是zk的集群节点
# --replication-factor 2  表示复本数
# --partitions 1  表示分区数
# --topic hello_canal  表示主题名称为hello_canal

#查看topic 列表：
$ ./kafka-topics.sh --list --zookeeper 192.168.1.113:2181,192.168.1.114:2181,192.168.1.115:2181

#查看指定topic：
$ ./kafka-topics.sh --describe --zookeeper 192.168.1.113:2181,192.168.1.114:2181,192.168.1.115:2181 --topic hello_canal
Topic: hello_canal	PartitionCount: 1	ReplicationFactor: 2	Configs: 
	Topic: hello_canal	Partition: 0	Leader: 0	Replicas: 0,2	Isr: 0,2
```

### 3、验证Kafka集群是否启动成功
```shell
# 随便在一个zk节点上启动zkCli(zookeeper客户端)
$ sh $ZOOKEEPER_HOME/bin/zkCli.sh

$ [zk: localhost:2181(CONNECTED) 0] ls /brokers/ids
[0, 1, 2]

# 如果能看到三台kafka节点的broker.id，则说明三台kafka节点正常启动
```

## 四、安装Canal.server

### （一）下载并解压
#### 1、下载

到Canal官网，下载最新压缩包：`canal.deployer-latest.tar.gz`
```shell
$ cd /data
$ wget https://github.com/alibaba/canal/releases/download/canal-1.1.5-alpha-2/canal.deployer-1.1.5-SNAPSHOT.tar.gz
```

#### 2、解压
```shell
$ mkdir /usr/local/canal

$ tar -zxvf canal.deployer-1.1.5-SNAPSHOT.tar.gz -C /usr/local/canal
```

### （二）修改配置文件
#### 1、修改instance配置文件
```shell
$ vim /usr/local/canal/conf/hello_canal/instance.properties

## mysql serverId
canal.instance.mysql.slaveId = 1234
#position info，需要改成自己的数据库信息
canal.instance.master.address = 127.0.0.1:3306 
canal.instance.master.journal.name = 
canal.instance.master.position = 
canal.instance.master.timestamp = 
#canal.instance.standby.address = 
#canal.instance.standby.journal.name = 
#canal.instance.standby.position = 
#canal.instance.standby.timestamp = 
#username/password，需要改成自己的数据库信息
canal.instance.dbUsername = canal
canal.instance.dbPassword = canal
canal.instance.defaultDatabaseName = 
canal.instance.connectionCharset = UTF-8
#table regex
canal.instance.filter.regex = .\*\\\\..\*
# mq config
canal.mq.topic=hello_canal
# dynamic topic route by schema or table regex
#canal.mq.dynamicTopic=mytest1.user,mytest2\\..*,.*\\..*
canal.mq.partition=0
```
`canal.instance.connectionCharset`代表数据库的编码方式对应到 java 中的编码类型，比如 UTF-8，GBK，ISO-8859-1

如果系统是1个 cpu，需要将` canal.instance.parser.parallel `设置为`false`

#### 2、修改canal配置文件
```shell
$ vim /usr/local/canal/conf/canal.properties

# ...
# 可选项: tcp(默认), kafka, RocketMQ
canal.serverMode = kafka
# ...
# kafka/rocketmq 集群配置，如果你的mq已经做了集群配置，则需要把所有节点的ip:port都写全在下方
canal.mq.servers = 192.168.1.113:9092,192.168.1.114:9092,192.168.1.115:9092
canal.mq.retries = 0
# flagMessage模式下可以调大该值, 但不要超过MQ消息体大小上限
canal.mq.batchSize = 16384
canal.mq.maxRequestSize = 1048576
# flatMessage模式下请将该值改大, 建议50-200
canal.mq.lingerMs = 1
canal.mq.bufferMemory = 33554432
# Canal的batch size, 默认50K, 由于kafka最大消息体限制请勿超过1M(900K以下)
canal.mq.canalBatchSize = 50
# Canal get数据的超时时间, 单位: 毫秒, 空为不限超时
canal.mq.canalGetTimeout = 100
# 是否为flat json格式对象
canal.mq.flatMessage = false
canal.mq.compressionType = none
canal.mq.acks = all
# kafka消息投递是否使用事务
canal.mq.transaction = false
```

### （三）启动canal
#### 1、启动
```shell
$ cd /usr/local/canal/
$ sh bin/startup.sh
```

#### 2、查看server日志
```shell
$ vim /usr/local/canal/logs/canal/canal.log
```

```log
2021-02-22 15:45:24.422 [main] INFO  com.alibaba.otter.canal.deployer.CanalLauncher - ## set default uncaught exception handler
2021-02-22 15:45:24.559 [main] INFO  com.alibaba.otter.canal.deployer.CanalLauncher - ## load canal configurations
2021-02-22 15:45:24.624 [main] INFO  com.alibaba.otter.canal.deployer.CanalStarter - ## start the canal server.
2021-02-22 15:45:24.834 [main] INFO  com.alibaba.otter.canal.deployer.CanalController - ## start the canal server[172.17.0.1(172.17.0.1):11111]
2021-02-22 15:45:30.351 [main] INFO  com.alibaba.otter.canal.deployer.CanalStarter - ## the canal server is running now ......
```

#### 3、查看instance的日志
```shell
$ vim /usr/local/canal/logs/hello_canal/hello_canal.log
```

```log
2021-02-22 16:54:24.284 [main] INFO  c.a.o.c.i.spring.support.PropertyPlaceholderConfigurer - Loading properties file from class path resource [canal.properties]
2021-02-22 16:54:24.308 [main] INFO  c.a.o.c.i.spring.support.PropertyPlaceholderConfigurer - Loading properties file from class path resource [hello_canal/instance.properties]
2021-02-22 16:54:25.143 [main] INFO  c.a.o.c.i.spring.support.PropertyPlaceholderConfigurer - Loading properties file from class path resource [canal.properties]
2021-02-22 16:54:25.144 [main] INFO  c.a.o.c.i.spring.support.PropertyPlaceholderConfigurer - Loading properties file from class path resource [hello_canal/instance.properties]
2021-02-22 16:54:26.586 [main] INFO  c.a.otter.canal.instance.spring.CanalInstanceWithSpring - start CannalInstance for 1-hello_canal
2021-02-22 16:54:26.642 [main] WARN  c.a.o.canal.parse.inbound.mysql.dbsync.LogEventConvert - --> init table filter : ^.*\..*$
2021-02-22 16:54:26.642 [main] WARN  c.a.o.canal.parse.inbound.mysql.dbsync.LogEventConvert - --> init table black filter : ^mysql\.slave_.*$
2021-02-22 16:54:27.057 [destination = hello_canal , address = ubuntu-master.com/192.168.1.113:3306 , EventParser] WARN  c.a.o.c.p.inbound.mysql.rds.RdsBinlogEventParserProxy - ---> begin to find start position, it will be long time for reset or first position
2021-02-22 16:54:27.176 [destination = hello_canal , address = ubuntu-master.com/192.168.1.113:3306 , EventParser] WARN  c.a.o.c.p.inbound.mysql.rds.RdsBinlogEventParserProxy - prepare to find start position just show master status
2021-02-22 16:54:27.179 [main] INFO  c.a.otter.canal.instance.core.AbstractCanalInstance - start successful....
```

#### 4、关闭
```shell
$ cd /usr/local/canal/
$ sh bin/stop.sh
```

## 五、查看Canal数据同步情况

### (一)通过Kafka消费者查看

#### 1、启动Kafka消费者
在另一台服务器上创建一个消费者：
```shell
$ cd /usr/local/kafka_2.13-2.7.0/bin
$ ./kafka-console-consumer.sh --bootstrap-server 192.168.1.113:9092,192.168.1.114:9092,192.168.1.115:9092 --topic hello_canal --from-beginning
```

*注：Kafka 从 2.2 版本开始将` kafka-topic.sh `脚本中的 ` −−zookeeper `参数标注为 “过时”，推荐使用 ` −−bootstrap-server `参数。*

*端口也由之前的zookeeper通信端口`2181`，改为了kafka通信端口`9092`。*

#### 2、创建测试表
```sql
mysql> create database if not exists test default charset utf8 collate utf8_general_ci;

mysql> use test;

mysql> create table fk (
id int UNSIGNED AUTO_INCREMENT,
name VARCHAR(100) NOT NULL,
age int NOT NULL,
PRIMARY KEY ( id )
);
```

#### 2、在源mysql数据库上修改数据
```sql
mysql> use test;
mysql> insert into fk values(13,'hello_canal',19);
```

#### 3、消费者窗口输出内容
```log
{"data":[{"id":"13","name":"hello_canal","age":"19"}],"database":"test","es":1614252283000,"id":2,"isDdl":false,"mysqlType":{"id":"int(10) unsigned","name":"varchar(100)","age":"int(11)"},"old":null,"pkNames":["id"],"sql":"","sqlType":{"id":4,"name":12,"age":4},"table":"fk","ts":1614252283248,"type":"INSERT"}
```

说明canal已经成功捕获到源MySQL的变化数据binlog并投递到kafka集群的hello_canal主题中。

## 六、遇到的问题
### （一）canal同步mysql binlog到kafka，启动后instance日志报`TimeoutException: Failed to update metadata after 60000 ms.`

#### 1、详细报错信息：
```log
Caused by: java.util.concurrent.ExecutionException: org.apache.kafka.common.errors.TimeoutException: Failed to update metadata after 60000 ms.
```

#### 2、报错可能原因及方案：

**报错原因1**：kafka的配置文件`config/server.properties`中`listeners=PLAINTEXT://your.host.name:9092`以及`advertise.listeners=PLAINTEXT://your.host.name:9092`没有配置，导致canal无法与kafka进行socket通信。

**解决方案**：补充上述两项配置，重启kafka即可。

**报错原因2**：canal的配置文件`conf/canal.properties`中`kafka.bootstrap.servers = x.x.x.x:9092`没有把所有的kafka节点配上。报错是因为只配了一台kafka节点，而我是以集群模式启动了三个kafka节点。

**解决方案**：修改`conf/canal.properties`中为`kafka.bootstrap.servers = x.x.x.1:9092,x.x.x.2:9092,x.x.x.3:9092`，重启canal即可。

### （二）canal无法stop

#### 1、详细报错信息：
```log
bin/stop.sh: 52: kill: No such process

bin/stop.sh: 58: [: unexpected operator
bin/stop.sh: 63: bin/stop.sh let: not found
```

#### 2、报错原因及方案：
报错：`let: not found`

因为在ubuntu默认是指向`bin/dash`解释器的，`dash`是阉割版的`bash`,其功能远没有`bash`强大和丰富。并且`dash`不支持`let`和`i++`等功能.

解决办法：`sudo dpkg-reconfigure dash`，选择`"No"`, 表示用`bash`代替`dash`。

### （三）command:'show master status' has an error!pls check.
#### 1、详细报错
instance日志中有如下报错：
```shell
caused by com.alibaba.otter.canal.parse.exception.CanalParseException:command:'show master status' has an error!pls check.you need (at least one of) the SUPER,REPLICATION CLIENT privilege(s) for thie operation 2021-03-08 14:03:03:13.903 [destination = hello_canal , address = /192.168.0.113:3306 , EventParser] 
```

#### 2、报错原因及方案：
**报错原因：**

从报错信息可以判断是mysql配置出了问题。

在mysql上执行`show master status`，输出结果为空：
```shell
mysql> show master status;
Empty set (0.00 sec)
```

原因是mysql没有开启日志。
查看log_bin选项：
```shell
mysql> show variables like '%log_bin%';
+---------------------------------+-------+
| Variable_name                   | Value |
+---------------------------------+-------+
| log_bin                         | OFF   |
| log_bin_basename                |       |
| log_bin_index                   |       |
| log_bin_trust_function_creators | OFF   |
| log_bin_use_v1_row_events       | OFF   |
| sql_log_bin                     | ON    |
+---------------------------------+-------+
6 rows in set (0.00 sec)
```
可以看到`log_bin`是`OFF`。

**解决方案：**

在mysql 配置文件`·` /etc/my.cnf `中

`[mysqld]`下添加:
```shell
log-bin=mysql-bin
```

保存后重启mysql服务：
```shell
#重启MySQL数据库
$ service mysql restart
```

再次执行`show master status`命令
```shell
mysql> SHOW MASTER STATUS\G
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000002 |      154 | test         | mysql            |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```

### （四）Q：不同instance的slaveId是否需要不同？

Canal官方解答：不和serverId冲突就可以。即不同实例的slaveId相同也没有影响，只需保持不同的机器上的canal服务之间的slaveId不同。

## 参考
[1] Canal QuickStart[https://github.com/alibaba/canal/wiki/QuickStart]

[2] Canal Kafka RocketMQ QuickStart[https://github.com/alibaba/canal/wiki/Canal-Kafka-RocketMQ-QuickStart]

[3] 利用Canal投递MySQL Binlog到Kafka[https://www.jianshu.com/p/93d9018e2fa1]

[4] canal实时同步mysql表数据到Kafka[https://www.cnblogs.com/zpan2019/p/13323035.html]

[5] mysql show master status为空值[https://blog.csdn.net/lanyang123456/article/details/85221071]