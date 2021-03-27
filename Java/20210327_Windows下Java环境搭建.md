# Windows下Java环境搭建

## 一、下载Java

## 二、安装Java

## 三、配置环境变量

### JAVA_HOME
`C:\Program Files\Java\jdk1.8.0_281`

### CLASSPATH
`.;%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar;`

### Path
```shell
%JAVA_HOME%\bin
%JAVA_HOME%\jre\bin
```

## 四、验证
打开cmd
```shell
$ java -version
Apache Maven 3.6.3 (cecedd343002696d0abb50b32b541b8a6ba2883f)
Maven home: C:\Program Files (x86)\apache-maven-3.6.3-bin\apache-maven-3.6.3\bin\..
Java version: 1.8.0_281, vendor: Oracle Corporation, runtime: C:\Program Files\Java\jre1.8.0_281
Default locale: zh_CN, platform encoding: GBK
OS name: "windows 10", version: "10.0", arch: "amd64", family: "windows"
```


