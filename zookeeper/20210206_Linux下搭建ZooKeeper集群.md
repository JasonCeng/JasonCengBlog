# Linux下搭建ZooKeeper集群
本文基于Ubuntu 16.04 LTS

## 环境依赖
- 奇数台服务器，且非observer节点数>1；如果是偶数台服务器，可以把多出来的1台设置为observer
- Java 8+
- 防火墙开放2181、2888、3888端口：
```shell
ufw allow 2181
ufw allow 2888
ufw allow 3888
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

## 二、配置文件
本小节先在`服务器1`上进行配置，下一小节再通过scp将配置好的ZooKeeper分发到其他服务器上，减少重复工作。

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

### 4、添加server.id
```shell
$ vim /usr/local/zookeeper_3.6.2/conf/zoo.cfg

# 添加如下内容：
server.1=192.168.1.113:2888:3888
server.2=192.168.1.114:2888:3888
server.3=192.168.1.115:2888:3888
# server.id=zookeeper节点主机名 或 主机ip:2888:3888  选举端口和投票端口固定
# 如果有四台zookeeper的话，由于非observer节点数必须为奇数个
# 所以如果有第四台的话，可以使用如下添加方式：
# server.4=xx.xx.xx.xx:2888:3888:observer
# id是自己定义的，0~255间，不必一次递增，自己定即可，id表示该节点所持有的投票编号id，必需保证全局唯一
```

*注意：不要在配置后面加注释#，这会导致cfg文件解析失败！从而导致zookeeper无法正常启动！*

`server.A=B:C:D`中各参数解析如下：
```shell
A：其中 A 是一个数字，表示这个是服务器的编号；

B：是这个服务器的 ip 地址 或 主机名；

C：Leader选举的端口；

D：Zookeeper服务器之间的通信端口。
```

### 5、创建myid文件
在dataDir目录下创建myid文件

机器`192.168.1.113`上：
```shell
$ vim /usr/local/zookeeper_3.6.2/data/myid

#输入server.1=192.168.1.113:2888:3888所指定的id：1
1
```

机器`192.168.1.114`上：
```shell
$ vim /usr/local/zookeeper_3.6.2/data/myid

#输入server.1=192.168.1.114:2888:3888所指定的id：2
2
```

机器`192.168.1.115`上：
```shell
$ vim /usr/local/zookeeper_3.6.2/data/myid

#输入server.1=192.168.1.115:2888:3888所指定的id：3
3
```

## 三、将ZooKeeper分发到各个节点
```shell
$ cd /usr/local/
$ scp -r zookeeper_3.6.2 root@192.168.1.114:/usr/local/
$ scp -r zookeeper_3.6.2 root@192.168.1.115:/usr/local/
```

## 四、配置环境变量
需要再各个服务器上进行配置：
```shell
vim /etc/profile

#添加如下内容
export ZOOKEEPER_HOME=/usr/local/zookeeper_3.6.2
export PATH=$ZOOKEEPER_HOME/bin:$PATH

#激活环境变量
source /etc/profile
```

## 五、启动zkServer
在所有节点上启动zkServer
```shell
$ zkServer.sh start

#输出如下：
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper_3.6.2/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
```

查看集群节点状态：
```shell
$ zkServer.sh status

#一个节点输出如下：
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper_3.6.2/bin/../conf/zoo.cfg
Client port found: 2181. Client address: localhost. Client SSL: false.
Mode: leader

#另外两个节点输出如下：
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper_3.6.2/bin/../conf/zoo.cfg
Client port found: 2181. Client address: localhost. Client SSL: false.
Mode: follower
```
有一台服务器的zk Mode为`leader `，另外两台服务器的zk Mode为`follower`，如果配置了`observer`节点，则会有一台服务器的zk Mode为`observer`。

停止命令：
```shell
zkServer.sh stop
```

重启命令：
```shell
zkServer.sh restart
```

## 六、查看ZooKeeper日志
```shell
$ cd /usr/local/zookeeper_3.6.2/logs
$ cat zookeeper-<somthing>.out
```

至此，ZooKeeper集群环境搭建结束。

## 参考
[1]Apache Zookeeper 集群的搭建[https://blog.csdn.net/qq_33713328/article/details/88854991]

[2]zookeeper 集群搭建[https://www.cnblogs.com/ysocean/p/9860529.html]