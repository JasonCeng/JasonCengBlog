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

### 3、创建一个操作数据库的专门用户（出于安全）
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
解决办法：
```shell
cd /root
openssl rand -writerand .rnd
```

### 3、启动mysql
```shell
#启动mysql，如果mysqld进程异常终止，mysqld_safe将自动重启mysqld
./mysqld_safe --user=mysql &

#检查mysql是否启动
ps -ef|grep mysql
```

用`mysqld_safe`脚本来启动MySQL服务器的做法在BSD风格的unix系统上很常见，非BSD风格的UNIX系统中的` mysql.server`脚本其实也是调用`mysqld_safe`脚本去启动MySQL服务器的，如以下这种方式：
```shell
/usr/local/mysql_5.7/support-files/mysql.server start
```

### 4、修改密码
```shell
#进入客户端
./mysql -uroot -p
Enter password:这里输入之前的临时密码
 
#修改密码
mysql> set password=password('新密码');
```

## 五、设置远程访问
### 1、打开mysql的默认端口3306
```shell
#设置3306为开放端口
ufw allow 3306
#重新加载防火墙
ufw reload
```

*注意：如果你使用的是云服务器，记得在云服务器控制台->安全组中添加开放3306端口*

### 2、设置mysql的远程访问
```shell
#设置远程访问账号，若最后加上with grant option，则同时可以赋予权限的权限
mysql> grant all privileges on *.* to root@'%' identified by '密码';

#刷新
mysql> flush privileges;
```

```shell
设置远程访问参数说明
grant [previleges] on [dbName].[tableName] to [userName]@[hostName] identified by "password";

previlege：授予的权限，有select,insert,update,delete,create,drop,index,alter,grant,references,reload,shutdown,process,file等14个权限，若all则表示赋予所有权限；

dbName：指定被访问的数据库名称，如果指定所有数据库可使用*星号；

tableName：指定被访问的数据表，如果指定某个数据库下的所有数据表可使用*星号；

userName：远程主机的登录用户名称；

hostName：远程主机名或者IP地址，%为所有主机均可登陆；

password：远程主机用户访问MySQL使用的密码。
```

## 六、设置开机自启动mysql
### 1、复制mysql.server到/etc/init.d/目录下(目的：实现开机自启动)
```shell
cp /usr/local/mysql_5.7/support-files/mysql.server /etc/init.d/mysql
```

### 2、修改/etc/init.d/mysql参数
```shell
vim /etc/init.d/mysql

#修改以下内容：
basedir=/usr/local/mysql_5.7
datadir=/usr/local/mysql_5.7/data
```

以上第1、2步已经在`三、配置文件`中实现过一次，若您已配置上述两个步骤，即可直接使用update-rc.d添加mysql为开机自启动。

### 3、使用update-rc.d添加mysql为开机自启动
```shell
sudo update-rc.d -f mysql defaults
```

## 七、配置环境变量
```shell
vim /etc/profile

#在最后一行加入以下内容
export PATH=/usr/local/mysql_5.7/bin:$PATH

#使修改生效
source /etc/profile
```

## 八、查看mysql版本
### 1、终端中：mysql -V(大写)

```shell
$ mysql -V
mysql  Ver 14.14 Distrib 5.7.23, for linux-glibc2.12 (x86_64) using  EditLine wrapper
```

### 2、在mysql中：mysql> status;(引号要加)

```shell
# 进入mysql
$ mysql -u root -p

mysql> status;
--------------
mysql  Ver 14.14 Distrib 5.7.23, for linux-glibc2.12 (x86_64) using  EditLine wrapper

Connection id:		8
Current database:	
Current user:		root@localhost
SSL:			Not in use
Current pager:		stdout
Using outfile:		''
Using delimiter:	;
Server version:		5.7.23 MySQL Community Server (GPL)
Protocol version:	10
Connection:		Localhost via UNIX socket
Server characterset:	utf8
Db     characterset:	utf8
Client characterset:	utf8
Conn.  characterset:	utf8
UNIX socket:		/tmp/mysql.sock
Uptime:			2 hours 51 min 2 sec

Threads: 1  Questions: 19  Slow queries: 0  Opens: 113  Flush tables: 1  Open tables: 106  Queries per second avg: 0.001
--------------
```

### 3、在help里面查找
```shell
$ mysql --help | grep Distrib
mysql  Ver 14.14 Distrib 5.7.23, for linux-glibc2.12 (x86_64) using EditLine wrapper
```

### 4、使用mysql的函数
```shell
# 进入mysql
$ mysql -u root -p

mysql> select version();
+-----------+
| version() |
+-----------+
| 5.7.23    |
+-----------+
1 row in set (0.00 sec)
```

## 参考
[1]Linux系统安装mysql数据库[https://blog.csdn.net/qq_15092079/article/details/81629238]

[2]ubuntu安装mysql遇到的坑----解决Mysql报错缺少libaio.so.1[https://www.cnblogs.com/hufulinblog/p/10124001.html]

[3]linux 生成证书错误[https://www.cnblogs.com/whm-blog/p/12836865.html]

[4]Ubuntu中设置mysql开机自动启动[https://blog.csdn.net/qq_43655686/article/details/112368739]

[5]ubuntu查看mysql版本的几种方法[https://www.cnblogs.com/zlsgh/p/8645589.html]

[6]centos安装mysql5.7.19报 error while loading shared libraries: libaio.so.1[https://blog.csdn.net/m0_38113129/article/details/79056047]

[7]Ubuntu添加和设置开机自动启动程序的方法[https://www.cnblogs.com/the-wang/p/11230087.html]

[8]Ubuntu启动项设置——之update-rc.d 命令使用[https://blog.csdn.net/typ2004/article/details/38712887/]

[9]Linux下ps -ef和ps aux的区别及格式详解[https://www.cnblogs.com/mydriverc/p/8303242.html]

[10]mysql safe 和mysqld_mysqld与mysqld_safe的区别[https://blog.csdn.net/weixin_29358053/article/details/113434173]