# MySQL基于Binlog单向主从复制配置详解

## 一、依赖环境
- 主从服务器操作系统版本和位数一致
- Master和Slave数据库的版本要一致
- Master和Slave数据库中的数据要一致
- Master开启二进制日志，Master和Slave的server_id在局域网内必须唯一

## 二、实验环境
- 操作系统：Ubuntu 16.04 LTS
- MySQL：5.7
- Master IP/hostname：192.168.1.113/Ubuntu-master
- Slave IP/hostname：192.168.1.114/Ubuntu-slave1

## 三、搭建步骤
### （一）Master搭建步骤
#### 1、修改配置文件
```shell
$ vim /etc/my.cnf

# 在 [mysqld] 中增加以下配置项
## 开启二进制日志功能，可以随便取，最好有含义
log-bin=mysql-bin

## 设置server_id，局域网内唯一
server-id=113

## 复制过滤：需要备份的数据库，输出binlog
binlog-do-db=test

## 复制过滤：不需要备份的数据库，不输出（mysql库一般不同步）
binlog-ignore-db=mysql

## 为每个session 分配的内存，在事务过程中用来存储二进制日志的缓存
binlog_cache_size=1M

## 主从复制的格式（mixed,statement,row，默认格式是statement）
binlog_format=mixed

## 二进制日志自动删除/过期的天数。默认值为0，表示不自动删除。
expire_logs_days=7

## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断。
## 如：1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致
slave_skip_errors=1062

## 在使用带有事务的InnoDB的复制设置中获得最大的持久性和一致性
innodb_flush_log_at_trx_commit=1
sync_binlog=1
```

重启MySQL数据库
```shell
$ service mysql restart
```

#### 2、创建用于复制的用户
```shell
$ mysql -uroot -p

mysql> CREATE USER 'repl'@'xx.xx.xx.xx' IDENTIFIED BY '123456';
mysql> GRANT REPLICATION SLAVE ON *.* TO 'repl'@'xx.xx.xx.xx';
MYSQL> flush privileges;
```

*注：`xx.xx.xx.xx`是备库所在服务器ip*

#### 3、查看position号
(1) 通过使用命令行客户端连接到源上来启动会话，并通过执行`FLUSH TABLES WITH READ LOCK`语句刷新所有表并阻止写语句：
```shell
mysql> FLUSH TABLES WITH READ LOCK;
```

(2) 记下position号（Slave机上需要用到这个position号和现在的日志文件)
```shell
mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000002 |      154 | test         | mysql            |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```

#### 4、创建测试表并插入模拟数据
为了演示创建数据快照以同步历史数据到备库，这里我们创建测试表并插入模拟数据，以供后续步骤使用。
```sql
create database if not exists test default charset utf8 collate utf8_general_ci;

use test;

create table fk (
id int UNSIGNED AUTO_INCREMENT,
name VARCHAR(100) NOT NULL,
age int NOT NULL,
PRIMARY KEY ( id )
);

insert into fk values(1,'xiaohong',18),(2,'xiaobai',20),(3,'heiheiheiheihei',22);
```

#### 5、备份数据库
使用`mysqldump`工具进行备份，如果使用的是`InnoDB`引擎，这是比较推荐的方法。

（1）临时锁表
```sql
mysql> flush tables with read lock;
```

（2）进行备份

这里只备份test库（`--databases`指定），`--add-drop-table`参数用于在每一个建表语句前添加drop table语句。
```sql
$ mysqldump -p3306 -uroot -p --add-drop-table test > /tmp/master-testdb.sql;
```

（3）解锁表
```sql
mysql> unlock tables;
```

*注意：实际生产环境中大数据量（超2G数据）的备份，建议不要使用`mysqldump`进行备份，因为会非常慢。此时推荐使用`XtraBackup`进行备份。*

#### 6、传送备份数据到Slave
将Master上备份的数据远程传送到Slave上，以用于Slave配置时恢复数据
```shell
$ scp /tmp/master-testdb.sql root@192.168.1.114:/tmp/
```

### （二）Slave搭建步骤
#### 1、修改配置文件
```shell
$ vim /etc/my.cnf

## 在 [mysqld] 中增加以下配置项

## 设置server_id，一般设置为IP
server_id=114

## 复制过滤：需要备份的数据库，输出binlog
#binlog-do-db=test

##复制过滤：不需要备份的数据库，不输出（mysql库一般不同步）
binlog-ignore-db=mysql

## 开启二进制日志，以备Slave作为其它Slave的Master时使用
log-bin=mysql-slave1-bin

## 为每个session 分配的内存，在事务过程中用来存储二进制日志的缓存
binlog_cache_size = 1M

## 主从复制的格式（mixed,statement,row，默认格式是statement）
binlog_format=mixed

## 二进制日志自动删除/过期的天数。默认值为0，表示不自动删除。
expire_logs_days=7

## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断。

## 如：1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致
slave_skip_errors=1062

## relay_log配置中继日志
relay_log=mysql-relay-bin

## log_slave_updates表示slave将复制事件写进自己的二进制日志
log_slave_updates=1

## 防止改变数据(除了特殊的线程)
read_only=1
```

#### 2、重启Slave服务器MySQL数据库
```shell
$ service mysql restart
```

#### 3、还原备份数据
(1) Slave上创建相同的数据库
```sql
create database if not exists test default charset utf8 collate utf8_general_ci;

use test;
```

(2) 导入数据
```shell
$ mysql -uroot -p test < /tmp/master-testdb.sql
```

#### 4、登录Slave数据库，添加相关参数
```shell
$ mysql -uroot -p

mysql> change master to master_host='192.168.1.113', master_user='repl', master_password='123456', master_port=3306, master_log_file='mysql-bin.000002', master_log_pos=154, master_connect_retry=30;
```

上面执行的命令的解释如下：
```shell
master_host='192.168.1.113' ## Master的IP地址

master_user='repl'  ## 用于同步数据的用户（在Master中授权的用户）

master_password='123456'  ## 同步数据用户的密码

master_port=3306  ## Master数据库服务的端口

master_log_file='mysql-bin.000002'  ##指定Slave从哪个日志文件开始读复制数据（可在Master上使用show master status查看到日志文件名）

master_log_pos=154  ## 从哪个POSITION号开始读

master_connect_retry=30  ##当重新建立主从连接时，如果连接建立失败，间隔多久后重试。单位为秒，默认设置为60秒，同步延迟调优参数。
```

#### 5、查看主从同步状态
```shell
mysql> show slave status\G;

#可看到Slave_IO_State为空， Slave_IO_Running和Slave_SQL_Running是No，表明Slave还没有开始复制过程。
```

#### 6、开启主从同步
```shell
mysql> start slave;
```

#### 7、再查看主从同步状态
```shell
mysql> show slave status\G;

*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.1.113
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 30
              Master_Log_File: mysql-bin.000002
          Read_Master_Log_Pos: 154
               Relay_Log_File: mysql-relay-bin.000002
                Relay_Log_Pos: 320
        Relay_Master_Log_File: mysql-bin.000002
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 154
              Relay_Log_Space: 527
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 113
                  Master_UUID: 4f18f977-6787-11eb-8774-1cb72c9140be
             Master_Info_File: /usr/local/mysql_5.7/data/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 
                Auto_Position: 0
         Replicate_Rewrite_DB: 
                 Channel_Name: 
           Master_TLS_Version: 
1 row in set (0.01 sec)

ERROR: 
No query specified
```

主要看以下两个参数，这两个参数如果是Yes就表示主从同步正常
```shell
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
```

#### 8、查看master和slave上线程的状态
在master上，可以看到slave的I/O线程创建的连接：
```shell
Master : mysql> show processlist\G;

*************************** 1. row ***************************
     Id: 5
   User: repl
   Host: ubuntu-slave1.com:42710
     db: NULL
Command: Binlog Dump
   Time: 178
  State: Master has sent all binlog to slave; waiting for more updates
   Info: NULL
*************************** 2. row ***************************
     Id: 6
   User: root
   Host: localhost
     db: NULL
Command: Query
   Time: 0
  State: starting
   Info: show processlist
2 rows in set (0.00 sec)

ERROR: 
No query specified
```

1.row 为处理slave的I/O线程的连接。

2.row 为处理本地命令行的线程。


在Slave上，可以看到slave的I/O线程创建的连接：
```shell
Slave : mysql> show processlist\G;

*************************** 1. row ***************************
     Id: 5
   User: root
   Host: localhost
     db: NULL
Command: Query
   Time: 0
  State: starting
   Info: show processlist
*************************** 2. row ***************************
     Id: 6
   User: system user
   Host: 
     db: NULL
Command: Connect
   Time: 296
  State: Waiting for master to send event
   Info: NULL
*************************** 3. row ***************************
     Id: 7
   User: system user
   Host: 
     db: NULL
Command: Connect
   Time: 296
  State: Slave has read all relay log; waiting for more updates
   Info: NULL
3 rows in set (0.00 sec)

ERROR: 
No query specified
```

1.row 为处理本地命令行的线程。

2.row 为I/O线程状态。

3.row 为SQL线程状态。

#### 9、查看Slave上数据情况
```shell
mysql> SELECT * FROM test.fk;
+----+-----------------+-----+
| id | name            | age |
+----+-----------------+-----+
|  1 | xiaohong        |  18 |
|  2 | xiaobai         |  20 |
|  3 | heiheiheiheihei |  22 |
+----+-----------------+-----+
3 rows in set (0.00 sec)
```
可以看到，Slave上的MySQL数据库成功备份了Master上的数据。

### （三）主从复制同步测试
#### 1、在Master中的roncoo库上变更数据
```shell
Master:mysql> INSERT INTO test.fk VALUES (4,'同步测试1','40'),(5,'同步测试2','50');
```
Master中添加完之后，登录Slave中查看数据是否已同步。

在Slave中查看数据库：
```shell
Slave:mysql> SELECT * FROM test.fk;
+----+-----------------+-----+
| id | name            | age |
+----+-----------------+-----+
|  1 | xiaohong        |  18 |
|  2 | xiaobai         |  20 |
|  3 | heiheiheiheihei |  22 |
|  4 | 同步测试1       |  40 |
|  5 | 同步测试2       |  50 |
+----+-----------------+-----+
5 rows in set (0.00 sec)
```
最终的测试结果是，在Master中的操作，都成功同步到了Slave。

#### 2、在Master上新建一个test1库
```shell
Master:mysql> create database if not exists test1 default charset utf8 collate utf8_general_ci;
```

在Slave中查看数据库
```shell
Slave:mysql> show databases;

```

因为我是在Master服务器的MySQL配置文件`my.cnf`中指定了需备份的数据库，所以这里新建test1数据库没有被同步到Slave服务器上。
```shell
## 复制过滤：需要备份的数据库，输出binlog
binlog-do-db=test
```

#### 3、<可选>同步出错在Slave上重置主从复制设置：
测试过程中，如果遇到同步出错，可在Slave上重置主从复制设置：
```shell
mysql> reset slave;

mysql> change master to master_host='192.168.1.113',
master_user='repl',
master_password='123456',
master_port=3306,
master_log_file='mysql-bin.00000x',  
master_log_pos=xx,
master_connect_retry=30;
```

此时，`master_log_file`和`master_log_pos`要在Master中用`show master status`命令查看。

*注意：如果在Slave没做只读控制的情况下，千万不要在Slave中手动插入数据，那样数据就会不一致，主从就会断开，就需要重新配置了。*

## 四、小结
上面所搭建的是单向复制的主从，也是用的比较多的。而双向主从其实就是Master和Slave都开启日志功能，然后在Master执行授权用户（这里授权的是自己作为从服务器，也就是这里的IP地址是Master的IP地址），然后再在Master上进行`chang master`操作。

## 参考
[1] MySQL主从复制的配置(https://www.cnblogs.com/qingfengbuluo/p/5363796.html)

[2] Setting Up Binary Log File Position Based Replication(https://dev.mysql.com/doc/refman/5.7/en/replication-howto.html)