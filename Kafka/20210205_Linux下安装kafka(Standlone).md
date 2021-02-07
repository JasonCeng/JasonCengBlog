# Linux下安装kafka(Standlone)

## 环境依赖
- Java 8+

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

### 3、设置环境变量
```shell
vim /etc/profile

#添加如下内容
export KAFKA_HOME=/usr/local/kafka_2.13-2.7.0
export PATH=$KAFKA_HOME/bin:$PATH

#激活环境变量
source /etc/profile
```

## 二、启动与停止kafka
这里使用kafka自带的脚本启动一个简单的单一节点Zookeeper实例：
```shell
# Start the ZooKeeper service
# Note: Soon, ZooKeeper will no longer be required by Apache Kafka.
$ bin/zookeeper-server-start.sh config/zookeeper.properties
```

Open another terminal session and run:
```shell
# Start the Kafka broker service
$ bin/kafka-server-start.sh config/server.properties
```
Once all services have successfully launched, you will have a basic Kafka environment running and ready to use.

停止 kafka 服务则运行下面命令：
```shell
$ bin/kafka-server-stop.sh config/server.properties
```

## 三、创建一个Topic来存储你的Events

These events are organized and stored in topics. Very simplified, a topic is similar to a folder in a filesystem, and the events are the files in that folder.

So before you can write your first events, you must create a topic. Open another terminal session and run:
```shell
$ bin/kafka-topics.sh --create --topic quickstart-events --bootstrap-server localhost:9092
```

For example, it can also show you details such as the partition count of the new topic:
```shell
$ bin/kafka-topics.sh --describe --topic quickstart-events --bootstrap-server localhost:9092
Topic:quickstart-events  PartitionCount:1    ReplicationFactor:1 Configs:
    Topic: quickstart-events Partition: 0    Leader: 0   Replicas: 0 Isr: 0
```

## 四、写一些Events到Topic中
Run the console producer client to write a few events into your topic. By default, each line you enter will result in a separate event being written to the topic.
```shell
$ bin/kafka-console-producer.sh --topic quickstart-events --bootstrap-server localhost:9092
This is my first event
This is my second event
```

You can stop the consumer client with Ctrl-C at any time.

## 五、读取Events
Open another terminal session and run the console consumer client to read the events you just created:
```shell
$ bin/kafka-console-consumer.sh --topic quickstart-events --from-beginning --bootstrap-server localhost:9092
This is my first event
This is my second event
```

You can stop the consumer client with Ctrl-C at any time.

## 六、使用Kafka Connect组件将数据作为事件流导入/导出

You probably have lots of data in existing systems like relational databases or traditional messaging systems, along with many applications that already use these systems. Kafka Connect allows you to continuously ingest data from external systems into Kafka, and vice versa. It is thus very easy to integrate existing systems with Kafka. To make this process even easier, there are hundreds of such connectors readily available.

Take a look at the Kafka Connect section learn more about how to continuously import/export your data into and out of Kafka.

## 七、使用Kafka Streams处理你的Events
To give you a first taste, here's how one would implement the popular WordCount algorithm:

```shell
KStream<String, String> textLines = builder.stream("quickstart-events");

KTable<String, Long> wordCounts = textLines
            .flatMapValues(line -> Arrays.asList(line.toLowerCase().split(" ")))
            .groupBy((keyIgnored, word) -> word)
            .count();

wordCounts.toStream().to("output-topic"), Produced.with(Serdes.String(), Serdes.Long()));
```

## 八、终止kafka环境
Now that you reached the end of the quickstart, feel free to tear down the Kafka environment—or continue playing around.

Stop the producer and consumer clients with Ctrl-C, if you haven't done so already.
Stop the Kafka broker with Ctrl-C.
Lastly, stop the ZooKeeper server with Ctrl-C.
If you also want to delete any data of your local Kafka environment including any events you have created along the way, run the command:

```shell
$ rm -rf /tmp/kafka-logs /tmp/zookeeper
```

