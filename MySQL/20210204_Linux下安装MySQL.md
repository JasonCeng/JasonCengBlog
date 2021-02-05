# Linux下安装MySQL
本文以Ubuntu16.04为例

## 一、下载MySQL
```shell
cd /data

wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.23-linux-glibc2.12-x86_64.tar.gz
```

## 二、解压
```shell
mkdir /usr/local/mysql_5.7

tar -zxvf mysql-5.7.23-linux-glibc2.12-x86_64.tar.gz -C /usr/local/mysql_5.7 --strip-components 1
```

*ps：`--strip-components 1 `表示跳过第一层目录进行解压*


## 三、配置文件
### 1、配置mysql启动文件
```shell
#若系统中无/etc/my.cnf文件，则需要创建
touch /etc/my.cnf
vim /etc/my.cnf
 
#添加以下内容
[mysql]
default-character-set=utf8
 
[mysqld]
default-storage-engine=INNODB
character_set_server=utf8
```

### 2、复制mysql.server到/etc/init.d/目录下(目的：实现开机自启动)
```shell
cp /usr/local/mysql_5.7/support-files/mysql.server /etc/init.d/mysql
```

### 3、修改/etc/init.d/mysql参数
```shell
vim /etc/init.d/mysql

#修改以下内容：
basedir=/usr/local/mysql_5.7
datadir=/usr/local/mysql_5.7/data
```

### 4、创建一个操作数据库的专门用户（出于安全）
```shell
#建立一个mysql的组
groupadd mysql
 
#建立mysql用户，并且把用户放到mysql组
useradd -r -g mysql mysql
 
#为mysql用户设置密码
passwd mysql
Enter new UNIX password:123456
Retype new UNIX password:123456

#给目录/usr/local/mysql 更改拥有者
chown -R mysql:mysql /usr/local/mysql_5.7/
```

注：`Linux chown（英文全拼：change owner）`命令用于设置文件所有者和文件关联组的命令。` -R `表示处理指定目录以及其子目录下的所有文件。

## 四、安装初始化mysql
### 1、初始化数据库
```shell
mkdir /usr/local/mysql_5.7/data

cd /usr/local/mysql_5.7/bin/

#下载libaio1，mysqld依赖到该包
apt-get install -y libaio1

./mysqld --initialize --user=mysql --basedir=/usr/local/mysql_5.7/ --datadir=/usr/local/mysql_5.7/data

# 2021-02-05T03:29:00.491416Z 1 [Note] A temporary password is generated for root@localhost: yAtdgq?%s6rq
```

- 注意：若报`libaio.so.1`错:`ubuntu error while loading shared libraries: libaio.so.1`，则`apt-get install -y libaio1`，`apt-get`是`libaio1`，`yum`下载是`libaio`
- 注意：初始化后会生成一个临时密码 root@localhost:：*(最好先记录这个临时密码)

### 2、给数据库加密
```shell
./mysql_ssl_rsa_setup --datadir=/usr/local/mysql_5.7/data/
```

问题：linux 生成证书错误
```shell
Can't load /root/.rnd into RNG
140337498755520:error:2406F079:random number generator:RAND_load_file:Cannot open file:../crypto/rand/randfile.c:88:Filename=/root/.rnd
```

### 3、启动mysql
```shell
#启动mysql，如果mysqld进程异常终止，mysqld_safe将自动重启mysqld
./mysqld_safe --user=mysql &
#检查mysql是否启动
ps -ef|grep mysql
```

### 4、修改密码
```shell
#进入客户端
./mysql -uroot -p
Enter password:这里输入之前的临时密码
 
#修改密码
mysql> set password=password('新密码');
```

解决办法：
```shell
cd /root
openssl rand -writerand .rnd
```

## 参考
[1]Linux系统安装mysql数据库[https://blog.csdn.net/qq_15092079/article/details/81629238]
[2]ubuntu安装mysql遇到的坑----解决Mysql报错缺少libaio.so.1[https://www.cnblogs.com/hufulinblog/p/10124001.html]
[3]linux 生成证书错误[https://www.cnblogs.com/whm-blog/p/12836865.html]