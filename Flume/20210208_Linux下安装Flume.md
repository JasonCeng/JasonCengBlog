# Linux下安装Flume

## 环境依赖
- Java 8+

## 一、下载与解压kafka
### 1、下载
```shell
$ cd /data
$ wget https://mirrors.tuna.tsinghua.edu.cn/apache/flume/1.9.0/apache-flume-1.9.0-bin.tar.gz
```

### 2、解压到指定位置
```shell
$ tar -xzvf apache-flume-1.9.0-bin.tar.gz -C /usr/local

$ mv /usr/local/apache-flume-1.9.0-bin /usr/local/flume_1.9.0

$ cd /usr/local/flume_1.9.0
```

### 3、设置环境变量
```shell
vim /etc/profile

#添加如下内容
export FLUME_HOME=/usr/local/flume_1.9.0
export PATH=$FLUME_HOME/bin:$PATH

#激活环境变量
source /etc/profile
```

### 4、验证
```shell
$ flume-ng version
# 输出如下
Flume 1.9.0
Source code repository: https://git-wip-us.apache.org/repos/asf/flume.git
Revision: d4fcab4f501d41597bc616921329a4339f73585e
Compiled by fszabo on Mon Dec 17 20:45:25 CET 2018
From source with checksum 35db629a3bda49d23e9b3690c80737f9
```

## 二、配置文件
### 1、创建flume.conf
这个配置文件是用来声明flume的抽取源等相关信息，具体使用时可具体配置，比如要抽取mysql数据库的数据，甚至可以新增一个名为`mysql-flume.conf`的配置文件。
```shell
$ cd /usr/local/flume_1.9.0
$ cp conf/flume-conf.properties.template conf/flume.conf
```

### 2、配置flume-env.sh
```shell
$ cd /usr/local/flume_1.9.0
$ cp conf/flume-env.sh.template conf/flume-env.sh
$ vim conf/flume-env.sh

#配置如下内容,这是我的JAVA_HOME路径，请跟进你的情况自行配置
export JAVA_HOME=/usr/jdk-8
```

## 参考
[1]https://cwiki.apache.org/confluence/display/FLUME/Getting+Started