# HDFS源码阅读_编译Hadoop

## 一、软件包
Hadoop 3.2.2 https://www.apache.org/dyn/closer.cgi/hadoop/common/hadoop-3.2.2/hadoop-3.2.2-src.tar.gz

## 二、环境准备
### Windows系统下的环境要求：
- jdk1.8
- Maven 3.0 or later
- ProtocolBuffer 2.5.0
- CMake 3.1 or newer
- Visual Studio 2010 Professional or Higher
- Windows SDK 8.1 (if building CPU rate control for the container executor)
- zlib headers (if building native code bindings for zlib)
- Internet connection for first build (to fetch all Maven and Hadoop dependencies)
- Unix command-line tools from GnuWin32: sh, mkdir, rm, cp, tar, gzip. These tools must be present on your PATH。（或者 git）
- Python ( for generation of docs using ‘mvn site’)

ProtocolBuffer 2.5.0安装并添加到环境变量

下载连接：https://github.com/protocolbuffers/protobuf/releases/download/v2.5.0/protoc-2.5.0-win32.zip

解压添加到`PATH`环境变量：`C:\Program Files (x86)\protoc-2.5.0-win32`


CMake安装并添加到环境变量

下载连接：https://github.com/Kitware/CMake/releases/download/v3.1.3/cmake-3.1.3-win32-x86.zip

将安装路径下bin目录添加到PATH环境变量：`C:\Program Files (x86)\cmake-3.1.3-win32-x86\cmake-3.1.3-win32-x86\bin`




### 类Unix系统下的环境要求：
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
$ mvn package -Psrc,native-win -DskipTests -Dtar -Dmaven.javadoc.skip=true
```

注意不要加 mvn 不要加clean，否则手动修改过的目录会被清理掉。

Windows系统下的编译选型：
```shell
mvn package [-Pdist][-Pdocs][-Psrc][-Dtar][-Dmaven.javadoc.skip=true]
mvn package -Dmaven.javadoc.skip=true -Dmaven.test.skip=true
mvn package -Pdist,native-win -DskipTests -Dtar -e -X
mvn package -Pdist,native-win -DskipTests -Dtar
mvn package -Pdist -DskipTests -Dtar -Dmaven.javadoc.skip=true
mvn package -Pdist -DskipTests -Dtar -Dmaven.javadoc.skip=true -rf :hadoop-common
```

类Unix系统下的编译选型：
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

## 五、可能会遇到的问题

### （一）hadoop-annotations Could not find artifact jdk.tools:jdk.tools:jar:1.8
1、确保你的机器上正确安装了Java且配置了相关环境变量

2、在hadoop源码根目录下的pom.xml中添加如下依赖：
```xml
<dependencies>
      <dependency>
        <groupId>jdk.tools</groupId>
        <artifactId>jdk.tools</artifactId>
        <version>1.8</version>
        <scope>system</scope>
        <systemPath>${java.home}/../lib/tools.jar</systemPath>
      </dependency>
</dependencies>
```

若是Eclipse，将`<systemPath>${java.home}/../lib/tools.jar</systemPath>`修改为`<systemPath>${JAVA_HOME}/lib/tools.jar</systemPath>`
