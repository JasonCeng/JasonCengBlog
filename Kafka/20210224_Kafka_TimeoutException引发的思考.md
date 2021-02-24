# Kafka_TimeoutException引发的思考

## 一、报错信息
今天在搭建Canal同步MySQL数据到Kafka时遇到了如下报错：
```log
org.apache.kafka.common.errors.TimeoutException: Failed to update metadata after 60000 ms.
```

Canal的完整报错日志如下：
```log
021-02-24 10:06:35.685 [pool-4-thread-1] ERROR c.a.o.canal.connector.kafka.producer.CanalKafkaProducer - java.util.concurrent.ExecutionException: org.apache.kafka.common.errors.TimeoutException: Failed to update metadata after 60000 ms.
java.lang.RuntimeException: java.util.concurrent.ExecutionException: org.apache.kafka.common.errors.TimeoutException: Failed to update metadata after 60000 ms.
        at com.alibaba.otter.canal.connector.kafka.producer.CanalKafkaProducer.send(CanalKafkaProducer.java:183) ~[na:na]
        at com.alibaba.otter.canal.server.CanalMQStarter.worker(CanalMQStarter.java:185) [canal.server-1.1.5-SNAPSHOT.jar:na]
        at com.alibaba.otter.canal.server.CanalMQStarter.access$500(CanalMQStarter.java:25) [canal.server-1.1.5-SNAPSHOT.jar:na]
        at com.alibaba.otter.canal.server.CanalMQStarter$CanalMQRunnable.run(CanalMQStarter.java:227) [canal.server-1.1.5-SNAPSHOT.jar:na]
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149) [na:1.8.0_281]
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624) [na:1.8.0_281]
        at java.lang.Thread.run(Thread.java:748) [na:1.8.0_281]
Caused by: java.util.concurrent.ExecutionException: org.apache.kafka.common.errors.TimeoutException: Failed to update metadata after 60000 ms.
        at org.apache.kafka.clients.producer.KafkaProducer$FutureFailure.<init>(KafkaProducer.java:1150) ~[na:na]
        at org.apache.kafka.clients.producer.KafkaProducer.doSend(KafkaProducer.java:846) ~[na:na]
        at org.apache.kafka.clients.producer.KafkaProducer.send(KafkaProducer.java:784) ~[na:na]
        at org.apache.kafka.clients.producer.KafkaProducer.send(KafkaProducer.java:671) ~[na:na]
        at com.alibaba.otter.canal.connector.kafka.producer.CanalKafkaProducer.produce(CanalKafkaProducer.java:267) ~[na:na]
        at com.alibaba.otter.canal.connector.kafka.producer.CanalKafkaProducer.send(CanalKafkaProducer.java:260) ~[na:na]
        at com.alibaba.otter.canal.connector.kafka.producer.CanalKafkaProducer.send(CanalKafkaProducer.java:165) ~[na:na]
        ... 6 common frames omitted
```

## 二、分析与解决

是因为没有配置Kafka的Socket服务相关配置：监听器listeners导致的，若没有配置listeners，kafka将通过`java.net.InetAddress.getCanonicalHostName()`进行获取，若获取不到则会出现`Failed to update metadata after 60000 ms.`

修改Kafka的`config/server.properties`相关配置如下：
```shell
$ vim $KAFKA_HOME/config/server.properties

############################# Socket Server Settings #############################

# The address the socket server listens on. It will get the value returned from 
# java.net.InetAddress.getCanonicalHostName() if not configured.
#   FORMAT:
#     listeners = listener_name://host_name:port
#   EXAMPLE:
#     listeners = PLAINTEXT://your.host.name:9092
listeners=PLAINTEXT://192.168.1.114:9092
```

重启Kafka:
```shell
# 停止Kafka
$ cd $KAFKA_HOME/bin #进入到kafka的bin目录 
$ ./kafka-server-stop.sh

# 从后台启动Kafka
$ ./kafka-server-start.sh -daemon ../config/server.properties
```

## 参考
[1] Kafka 客户端TimeoutException问题之坑
各种TimeoutException问题[https://www.jianshu.com/p/2db7abddb9e6]

[2] Kafka java client 连接异常（org.apache.kafka.common.errors.TimeoutException: Failed to update metadata ）[https://my.oschina.net/u/3247419/blog/1189241]