# canal与MySQL相关的配置详解

instance.properties参数列表：

| 参数名字 |	参数说明 |	默认值 |
| :--------   | :-----  | :----  |
| canal.instance.mysql.slaveId	| mysql集群配置中的serverId概念，需要保证和当前mysql集群中id唯一 (v1.1.x版本之后canal会自动生成，不需要手工指定)	| 无
| canal.instance.master.address	| mysql主库链接地址	| 127.0.0.1:3306
| canal.instance.master.journal.name	| mysql主库链接时起始的binlog文件	| 无
| canal.instance.master.position	| mysql主库链接时起始的binlog偏移量	| 无
| canal.instance.master.timestamp	| mysql主库链接时起始的binlog的时间戳	| 无
| canal.instance.gtidon	| 是否启用mysql gtid的订阅模式	| false
| canal.instance.master.gtid	| mysql主库链接时对应的gtid位点	| 无
| canal.instance.dbUsername	| mysql数据库帐号	| canal
| canal.instance.dbPassword	| mysql数据库密码	| canal
| canal.instance.defaultDatabaseName	| mysql链接时默认schema	 
| canal.instance.connectionCharset	| mysql 数据解析编码	| UTF-8
| canal.instance.filter.regex	| mysql 数据解析关注的表，Perl正则表达式.<br>多个正则之间以逗号(,)分隔，转义符需要双斜杠(\\)<br><br>常见例子：<br>1.  所有表：.*   or  .\*\\\\..\*<br>2.  canal schema下所有表： canal\\\\..\*<br>3.  canal下的以canal打头的表：canal\\\\.canal.\*<br>4.  canal schema下的一张表：canal\\\\.test1<br>5.  多个规则组合使用：canal\\\\..\*,mysql.test1,mysql.test2 (逗号分隔) |  .\*\\\\..\* | 
|canal.instance.filter.black.regex	|mysql 数据解析表的黑名单，表达式规则见白名单的规则| 无 | 
|canal.instance.rds.instanceId	| aliyun rds对应的实例id信息(如果不需要在本地binlog超过18小时被清理后自动下载oss上的binlog，可以忽略该值)| 无
 

几点说明：

1.mysql链接时的起始位置

- canal.instance.master.journal.name +  canal.instance.master.position :  精确指定一个binlog位点，进行启动 
- canal.instance.master.timestamp :  指定一个时间戳，canal会自动遍历mysql binlog，找到对应时间戳的binlog位点后，进行启动
- 不指定任何信息：默认从当前数据库的位点，进行启动。(show master status)

2.mysql解析关注表定义
- 标准的Perl正则，注意转义时需要双斜杠：\\\

3.mysql链接的编码

- 目前canal版本仅支持一个数据库只有一种编码，如果一个库存在多个编码，需要通过filter.regex配置，将其拆分为多个canal instance，为每个instance指定不同的编码
 
## 参考
[1] Canal AdminGuide[https://github.com/alibaba/canal/wiki/AdminGuide]