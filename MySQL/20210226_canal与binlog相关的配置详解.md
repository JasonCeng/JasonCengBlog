# canal与binlog相关的配置详解

`conf/canal.properties`中与binlog相关配置详解如下：

```shell
# binlog filter config
canal.instance.filter.druid.ddl = true #是否使用druid处理所有的ddl解析来获取库和表名。默认：true
canal.instance.filter.query.dcl = false #是否忽略dcl语句。默认：false
canal.instance.filter.query.dml = false #是否忽略dml语句(mysql5.6之后，在row模式下每条DML语句也会记录SQL到binlog中,可参考MySQL文档)。默认：false
canal.instance.filter.query.ddl = false #是否忽略ddl语句。默认：false
canal.instance.filter.table.error = false #是否忽略binlog表结构获取失败的异常(主要解决回溯binlog时,对应表已被删除或者表结构和binlog不一致的情况)。默认：false
canal.instance.filter.rows = false #是否dml的数据变更事件(主要针对用户只订阅ddl/dcl的操作)。默认：false
canal.instance.filter.transaction.entry = false #是否忽略事务头和尾,比如针对写入kakfa的消息时，不需要写入TransactionBegin/Transactionend事件。默认：false

# binlog format/image check
canal.instance.binlog.format = ROW,STATEMENT,MIXED #支持的binlog format格式列表(otter会有支持format格式限制)。默认：ROW,STATEMENT,MIXED
canal.instance.binlog.image = FULL,MINIMAL,NOBLOB #支持的binlog image格式列表(otter会有支持format格式限制)。默认：FULL,MINIMAL,NOBLOB

# binlog ddl isolation
canal.instance.get.ddl.isolation = false #ddl语句是否单独一个batch返回(比如下游dml/ddl如果做batch内无序并发处理,会导致结构不一致)。默认：false
```

## 参考
[1] Canal AdminGuide[https://github.com/alibaba/canal/wiki/AdminGuide]