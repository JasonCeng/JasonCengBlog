# HDFS源码阅读_编译Hadoop

## 一、软件包
Hadoop 3.2.2 https://www.apache.org/dyn/closer.cgi/hadoop/common/hadoop-3.2.2/hadoop-3.2.2-src.tar.gz

## 二、环境准备

- Unix System
- JDK 1.8
- Maven 3.3 or later
- ProtocolBuffer 2.5.0  --必须是这个版本
- CMake 3.1 or newer (if compiling native code)   
- Zlib devel (if compiling native code)
- Cyrus SASL devel (if compiling native code)
- One of the compilers that support thread_local storage: GCC 4.8.1 or later, Visual Studio,
- Clang (community version), Clang (version for iOS 9 and later) (if compiling native code)
- openssl devel (if compiling native hadoop-pipes and to get the best HDFS encryption performance)
- Linux FUSE (Filesystem in Userspace) version 2.6 or above (if compiling fuse_dfs)
- Jansson C XML parsing library ( if compiling libwebhdfs )
- Doxygen ( if compiling libhdfspp and generating the documents )
- Internet connection for first build (to fetch all Maven and Hadoop dependencies)
- python (for releasedocs)
- bats (for shell code testing)
- Node.js / bower / Ember-cli (for YARN UI v2 building)

## 三、导入IDEA，设置maven仓库源
在源码根目录下的pom.xml
```xml
<repositories>
    <repository>
      <id>aliyunmaven</id>
      <name>aliyunmaven</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
    </repository>
</repositories>
```

## 四、编译
在源码根目录下执行：
```
$ mvn mvn package -Psrc -DskipTests
```

其他编译选型：
```shell
Building distributions:
 
Create binary distribution without native code and without documentation:
 
$ mvn package -Pdist -DskipTests -Dtar -Dmaven.javadoc.skip=true
 
Create binary distribution with native code and with documentation:
 
$ mvn package -Pdist,native,docs -DskipTests -Dtar
 
Create source distribution:
 
$ mvn package -Psrc -DskipTests
 
Create source and binary distributions with native code and documentation:
 
$ mvn package -Pdist,native,docs,src -DskipTests -Dtar
 
Create a local staging version of the website (in /tmp/hadoop-site)
 
$ mvn clean site -Preleasedocs; mvn site:stage -DstagingDirectory=/tmp/hadoop-site

Note that the site needs to be built in a second pass after other artifacts.
```