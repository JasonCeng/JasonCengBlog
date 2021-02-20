# Ubuntu搭建hadoop集群

## 一、准备3台Linux服务器
- 操作系统：Ubuntu 16.04 LTS
> 一台物理机，两台vm虚拟机，构成一主两从小集群

- 软件：openssh-server
> lsof:22 开启22端口，主要用于远程连接

- 个性化域名
> 定制hostname并配置hosts

```shell
#配置hosts
sudo vim /etc/hosts

#务必在IPv6之前新增，三台服务器新增内容相同：
192.168.1.113	ubuntu-master.com
192.168.1.114	ubuntu-slave1.com
192.168.1.115	ubuntu-slave2.com

#*****************我是分割线********************
#定制hostname
sudo vim /etc/hostname

#三台服务器分别为：
ubuntu-master
ubuntu-slave1
ubuntu-slave2

#以上修改需要重启才能生效
reboot
```

## 二、安装Java
参考：<a href='https://github.com/JasonCeng/JasonCengBlog/blob/main/Java/20210201_Ubuntu%E6%90%AD%E5%BB%BAJava%E7%8E%AF%E5%A2%83.md' target='_blank'>20210201_Ubuntu搭建Java环境.md</a>

## 三、创建hadoop用户
1、在master节点上使用如下命令来创建hadoop用户
```shell
$ sudo adduser hadoop
```

2、使用如命令把hadoop用户加入到hadoop用户组，前面一个hadoop是组名，后面一个hadoop是用户名
```shell
$ sudo usermod -a -G hadoop hadoop
```

3、使用如下命令来查看结果
```shell
$ cat /etc/group |grep hadoop
```

4、把hadoop用户赋予root权限，让他可以使用sudo命令，使用如下命令编辑
```shell
sudo vim /etc/sudoers

#修改文件如下：
#root   ALL=(ALL) ALL 下一行新增：
hadoop  ALL=(root) NOPASSWD:ALL
```

5、用同样方法在slave1和slave2上创建hadoop用户。

***注意：后续操作均切换到hadoop用户执行：***
```shell
$ su hadoop
```

## 四、建立ssh无密码登录本机


## 五、安装hadoop
1、下载hadoop

官方下载地址：https://hadoop.apache.org/releases.html下载，这里hadoop 3.2.2版本

```shell
$ cd /data
$ wget https://mirrors.tuna.tsinghua.edu.cn/apache/hadoop/common/hadoop-3.2.2/hadoop-3.2.2.tar.gz
```

2、将安装包发送到slave1、slave2节点
```shell
$ scp hadoop-3.2.2.tar.gz root@192.168.1.114:/data
$ scp hadoop-3.2.2.tar.gz root@192.168.1.115:/data
```

3、解压安装包
```shell
$ tar -zxvf hadoop-3.2.2.tar.gz -C /usr/local
```

4、添加hadoop环境变量
```shell
$ vim /etc/profile

#添加如下配置
export HADOOP_HOME=/usr/local/hadoop-3.2.2
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export PATH=$PATH:$HADOOP_HOME/
bin:$HADOOP_HOME/sbin

#使修改生效
$ source /etc/profile
```

## 六、创建hdfs相关目录
在$HADOOP_HOME目录中一个hdfs目录和三个子目录:
```shell
$ mkdir $HADOOP_HOME/hdfs
$ mkdir $HADOOP_HOME/hdfs/tmp
$ mkdir $HADOOP_HOME/hdfs/name
$ mkdir $HADOOP_HOME/hdfs/data
```

## 七、修改配置文件
需要修改的配置文件在`$HADOOP_HOME/etc/hadoop`目录下，如下文件：
```shell
core-site.xml
hadoop-env.sh
hdfs-site.xml
mapred-site.xml.template
yarn-env.sh
yarn-site.xml
mapred-env.sh
slaves
```

1、首先配置core-site.xml文件
```shell
$ sudo vim $HADOOP_HOME/etc/hadoop/core-site.xml

#在<configuration></configuration>中配置如下属性
##<configuration>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>file:/usr/local/hadoop-3.2.2/hdfs/tmp</value>
        <description>A base for other temporary directories.</description>
    </property>
    <property>
        <name>io.file.buffer.size</name>
        <value>131072</value>
        </property>
        <property>
        <name>fs.defaultFS</name>
        <value>hdfs://ubuntu-master.com:9000</value>
    </property>
##</configuration>
```

`hadoop.tmp.dir`是用来指定hadoop临时目录。前面用file:表示是本地目录,也可以不加`file:`。hadoop在运行过程中肯定会有临时文件或缓冲之类的，必然需要一个临时目录来存放，这里就是指定这个的。这个目录前面已经创建好了。

`io.file.buffer.size`是读写sequence file的buffer size，可减少 I/O 次数，该属性值单位为KB。在大型的Hadoop cluster，建议可设定为`65536`到`131072`，默认值`4096`。按照<a href='https://hadoop.apache.org/docs/r3.2.2/hadoop-project-dist/hadoop-common/ClusterSetup.html' target='_blank'>Hadoop集群搭建官方教程</a>配置了`131072`。

`fs.defaultFS`用来指定namenode的hdfs协议的文件系统通信地址。值为一个主机+端口，也可以指定为一个namenode服务（这个服务内部可以有多台namenode实现ha的namenode服务）。

2、配置hadoop-env.sh文件

主要是配置jdk目录
```shell
$ sudo vim $HADOOP_HOME/etc/hadoop/hadoop-env.sh

#添加如下
export JAVA_HOME=/usr/jdk-8
```

3、配置mapred-env.sh文件

主要是配置jdk目录
```shell
$ sudo vim $HADOOP_HOME/etc/hadoop/mapred-env.sh

#添加如下
export JAVA_HOME=/usr/jdk-8
```

4、配置yarn-env.sh文件

主要是配置jdk目录
```shell
$ sudo vim $HADOOP_HOME/etc/hadoop/yarn-env.sh

#添加如下
export JAVA_HOME=/usr/jdk-8
```

5、配置hdfs-site.xml

```shell
$ sudo vim $HADOOP_HOME/etc/hadoop/hdfs-site.xml

#在<configuration></configuration>中加入以下代码：
#<configuration>
    <property>
        <name>dfs.replication</name>
        <value>2</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:/usr/local/hadoop-3.2.2/hdfs/name</value>
        <final>true</final>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:/usr/local/hadoop-3.2.2/hdfs/data</value>
        <final>true</final>
    </property>
    <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>ubuntu-master.com:9001</value>
    </property>
    <property>
        <name>dfs.webhdfs.enabled</name>
        <value>true</value>
    </property>
    <property>
        <name>dfs.permissions</name>
        <value>false</value>
    </property>
#</configuration>
```

6、配置mapred-site.xml
```shell
$ sudo vim $HADOOP_HOME/etc/hadoop/mapred-site.xml

# 在标签<configuration>中添加以下代码：
#<configuration>
    <property>
        <name>yarn.resourcemanager.address</name>
        <value>ubuntu-master.com:18040</value>
    </property>
    <property>
        <name>yarn.resourcemanager.scheduler.address</name>
        <value>ubuntu-master.com:18030</value>
    </property>
    <property>
        <name>yarn.resourcemanager.webapp.address</name>
        <value>ubuntu-master.com:18088</value>
    </property>
    <property>
        <name>yarn.resourcemanager.resource-tracker.address</name>
        <value>ubuntu-master.com:18025</value>
    </property>
    <property>
        <name>yarn.resourcemanager.admin.address</name>
        <value>ubuntu-master.com:18141</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.auxservices.mapreduce.shuffle.class</name>
        <value>org.apache.hadoop.mapred.ShuffleHandler</value>
    </property>
#</configuration>
```

7、编辑slaves文件
```shell
$ sudo vim $HADOOP_HOME/etc/hadoop/slaves

#修改为：
ubuntu-slave1.com
ubuntu-slave2.com
```

8、对$HADOOP_HOME添加写入权限
```shell
$ sudo chmod -R 777 /usr/local/hadoop-3.2.2
```

9、格式化master节点的namenode

在master主机上执行：
```shell
$ hdfs namenode -format
```

## 八、启动Hadoop
在master上开启hadoop

## 参考
[1]ubuntu18.04 搭建hadoop完全分布式集群（Master、slave1、slave2）共三个节点[https://blog.csdn.net/sunxiaoju/article/details/85222290]
