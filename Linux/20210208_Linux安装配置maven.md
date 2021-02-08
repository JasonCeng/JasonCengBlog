# Linux安装配置maven

## 一、下载与解压
### 1、下载
```shell
$ cd /data
$ wget https://mirrors.tuna.tsinghua.edu.cn/apache/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz
```

### 2、解压
```shell
$ tar -zxvf apache-maven-3.6.3-bin.tar.gz -C /usr/local/
$ mv /usr/local/apache-maven-3.6.3 /usr/local/maven
```

### 3、配置环境变量
```shell
$ vim /etc/profile
$ export MAVEN_HOME=/usr/local/maven
$ export PATH=$PATH:$MAVEN_HOME/bin
$ source /etc/profile
```

## 二、配置
### 1、创建Maven本地仓库：
```shell
$ mkdir -p /usr/data/maven/local_repository
```

### 2、修改配置文件
打开Maven的`settings.xml`配置文件，配置相应的仓库路径以及国内仓库地址：
```shell
$ vim /usr/local/maven/conf/settings.xml

    #修改本地仓库路径：
　　<localRepository>/usr/data/maven/local_repository</localRepository>

    #修改国内仓库地址：配置在<mirrors>中
　　<mirror>
　　　　<id>nexus-aliyun</id>
　　　　<mirrorOf>*</mirrorOf>
　　　　<name>Nexus aliyun</name>
　　　　<url>http://maven.aliyun.com/nexus/content/groups/public</url>
　　</mirror>
```

## 三、测试
在终端输入：`mvn`

## 参考
[1]Maven-Linux安装与配置[https://www.cnblogs.com/luliang888/articles/10864411.html]