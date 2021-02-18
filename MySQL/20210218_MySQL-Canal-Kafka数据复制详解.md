# MySQL-Canal-Kafka数据复制详解

## 环境说明


## 一、安装ZooKeeper


## 二、安装KafKa


## 三、安装Canal.server

### （一）下载并解压
1、下载

到Canal官网，下载最新压缩包：`canal.deployer-latest.tar.gz`
```shell
$ cd /data
$ wget https://github.com/alibaba/canal/releases/download/canal-1.1.5-alpha-2/canal.deployer-1.1.5-SNAPSHOT.tar.gz
```

2、解压
```shell
$ mkdir /usr/local/canal

$ tar -zxvf canal.deployer-1.1.5-SNAPSHOT.tar.gz -C /usr/local/canal --strip-components 1
```

