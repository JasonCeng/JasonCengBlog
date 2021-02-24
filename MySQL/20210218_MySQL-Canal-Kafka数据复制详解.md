# MySQL-Canal-Kafka数据复制详解

## 环境说明

## 一、源MySQL配置
### 1、开启 Binlog 写入功能

对于自建 MySQL , 需要先开启 Binlog 写入功能，配置 binlog-format 为 ROW 模式，my.cnf 中配置如下
```shell
$ vim /etc/my.cnf

[mysqld]
log-bin=mysql-bin # 开启 binlog
binlog-format=ROW # 选择 ROW 模式
server_id=1 # 配置 MySQL replaction 需要定义，不要和 canal 的 slaveId 重复

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

参考：<a href='https://github.com/JasonCeng/JasonCengBlog/blob/main/zookeeper/20210206_Linux%E4%B8%8B%E6%90%AD%E5%BB%BAZooKeeper%E9%9B%86%E7%BE%A4.md' target='_blank'>JasonCengBlog/zookeeper/20210206_Linux下搭建ZooKeeper集群.md</a>

1、在**所有节点**上启动zkServer
```shell
$ zkServer.sh start&
```

2、查看节点状态
```shell
$ zkServer.sh status
```



## 三、安装KafKa

参考：<a href='https://github.com/JasonCeng/JasonCengBlog/blob/main/Kafka/20210207_Linux%E4%B8%8B%E6%90%AD%E5%BB%BAkafka%E9%9B%86%E7%BE%A4.md' target='_blank'>JasonCengBlog/Kafka/20210207_Linux下搭建kafka集群.md</a>

1、在**所有节点**上启动kafka
```shell
#从后台启动Kafka集群（3台都需要启动）
$ cd /usr/local/kafka_2.13-2.7.0/bin #进入到kafka的bin目录 
$ ./kafka-server-start.sh -daemon ../config/server.properties

#查看kafka是否启动
$ jps
```

2、创建Topic
```shell
$ cd /usr/local/kafka_2.13-2.7.0/bin #进入到kafka的bin目录 

#创建Topic
$ ./kafka-topics.sh --create --zookeeper 192.168.1.113:2181,192.168.1.114:2181,192.168.1.115:2181 --replication-factor 2 --partitions 1 --topic example
#解释
# --create  表示创建
# --zookeeper 192.168.1.113:2181  后面的参数是zk的集群节点
# --replication-factor 2  表示复本数
# --partitions 1  表示分区数
# --topic foo  表示主题名称为example

#查看topic 列表：
$ ./kafka-topics.sh --list --zookeeper 192.168.1.113:2181,192.168.1.114:2181,192.168.1.115:2181

#查看指定topic：
$ ./kafka-topics.sh --describe --zookeeper 192.168.1.113:2181,192.168.1.114:2181,192.168.1.115:2181 --topic example
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
$ vim /usr/local/canal/conf/example/instance.properties

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
canal.mq.topic=example
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
# kafka/rocketmq 集群配置: 192.168.1.113:9092,192.168.1.114:9092,192.168.1.115:9092 
canal.mq.servers = 127.0.0.1:6667
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
$ vim /usr/local/canal/logs/example/example.log
```

```log
2021-02-22 16:54:24.284 [main] INFO  c.a.o.c.i.spring.support.PropertyPlaceholderConfigurer - Loading properties file from class path resource [canal.properties]
2021-02-22 16:54:24.308 [main] INFO  c.a.o.c.i.spring.support.PropertyPlaceholderConfigurer - Loading properties file from class path resource [example/instance.properties]
2021-02-22 16:54:25.143 [main] INFO  c.a.o.c.i.spring.support.PropertyPlaceholderConfigurer - Loading properties file from class path resource [canal.properties]
2021-02-22 16:54:25.144 [main] INFO  c.a.o.c.i.spring.support.PropertyPlaceholderConfigurer - Loading properties file from class path resource [example/instance.properties]
2021-02-22 16:54:26.586 [main] INFO  c.a.otter.canal.instance.spring.CanalInstanceWithSpring - start CannalInstance for 1-example
2021-02-22 16:54:26.642 [main] WARN  c.a.o.canal.parse.inbound.mysql.dbsync.LogEventConvert - --> init table filter : ^.*\..*$
2021-02-22 16:54:26.642 [main] WARN  c.a.o.canal.parse.inbound.mysql.dbsync.LogEventConvert - --> init table black filter : ^mysql\.slave_.*$
2021-02-22 16:54:27.057 [destination = example , address = ubuntu-master.com/192.168.1.113:3306 , EventParser] WARN  c.a.o.c.p.inbound.mysql.rds.RdsBinlogEventParserProxy - ---> begin to find start position, it will be long time for reset or first position
2021-02-22 16:54:27.176 [destination = example , address = ubuntu-master.com/192.168.1.113:3306 , EventParser] WARN  c.a.o.c.p.inbound.mysql.rds.RdsBinlogEventParserProxy - prepare to find start position just show master status
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
$ ./kafka-console-consumer.sh --bootstrap-server 192.168.1.114:9092 --topic example --from-beginning
```

*注：Kafka 从 2.2 版本开始将` kafka-topic.sh `脚本中的 ` −−zookeeper `参数标注为 “过时”，推荐使用 ` −−bootstrap-server `参数。*

*端口也由之前的zookeeper通信端口`2181`，改为了kafka通信端口`9092`。*

#### 2、在源mysql数据库上修改数据
```sql
mysql> use test;
mysql> insert into fk values(6,'test',18);
```