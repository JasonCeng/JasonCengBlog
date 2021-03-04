# Ubuntu搭建Java环境

1、下载jdk，可通过[jdk8官方页面](https://www.oracle.com/java/technologies/javase/javase-jdk8-downloads.html)下载最新的jdk包。并上传到`/data`目录下

2、将下载好的jdk包解压，并重命名目录为`/usr/jdk-8`

```shell
tar -zxvf /data/jdk-8u281-linux-x64.tar.gz -C /usr
mv /usr/jdk1.8.0_281/ /usr/jdk-8
```

3、配置环境变量
```shell
sudo vim /etc/profile

#在文件末尾添加
export JAVA_HOME=/usr/jdk-8
export JRE_HOME=$JAVA_HOME/jre
export CLASSPATH=.:$CLASSPATH:$JAVA_HOME/lib:$JRE_HOME/lib
export PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin

#激活环境变量
source /etc/profile
```

4、使用`java -version`命令进行验证
```shell
$ java -version
java version "1.8.0_281"
Java(TM) SE Runtime Environment (build 1.8.0_281-b09)
Java HotSpot(TM) 64-Bit Server VM (build 25.281-b09, mixed mode)
```
