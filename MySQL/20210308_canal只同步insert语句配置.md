# canal只同步insert语句配置

instance配置
```shell
# table regex
canal.instance.filter.regex=hmalldb\\.hamll_order_base
```

canal.properties配置
```shell
# binlog filter config
canal.instance.filter.druid.ddl = true
canal.instance.filter.query.dcl = true #忽略dcl
canal.instance.filter.query.dml = false
canal.instance.filter.query.ddl = true #忽略ddl
canal.instance.filter.table.error = false
canal.instance.filter.rows = false
canal.instance.filter.transaction.entry = false 
```