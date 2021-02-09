# MySQL:Binlog选项之binlog_format详解

本文基于MySQL5.7，主要参考MySQL官方用户手册：https://dev.mysql.com/doc/refman/5.7/en/replication-options-binary-log.html#sysvar_binlog_format

## 一、`binlog_format`概览

<table>
    <tr>
        <td>命令行格式</td>
        <td>--binlog-format=format</td>
    </tr>
    <tr>
        <td>系统变量</td>
        <td>binlog_format</td>
    </tr>
    <tr>
        <td>作用范围</td>
        <td>Global, Session</td>
    </tr>
    <tr>
        <td>动态支持</td>
        <td>Yes</td>
    </tr>
    <tr>
        <td>类型</td>
        <td>Enumeration</td>
    </tr>
    <tr>
        <td>默认值</td>
        <td>Row</td>
    </tr>
    <tr>
        <td>可选值</td>
        <td>ROW</br>
            STATEMENT</br>
            MIXED
        </td>
    </tr>
</table>

该变量设置binlog格式，并且可以是`STATEMENT`，`ROW`或`MIXED`中的任何一种。

可以在启动时或在运行时设置binlog_format，除了在某些情况下无法在运行时更改此变量或导致复制失败（如稍后 `六、无法在运行时切换复制格式的例外` 所述）外。

## 二、三种复制格式概述
上一节我们提到，binlog格式有三种，分别是：`STATEMENT`，`ROW`或`MIXED`。

在MySQL 5.7.7之前，默认格式为`STATEMENT`。 在MySQL 5.7.7和更高版本中，默认值为`ROW`。 例外：在NDB群集中，默认值为`MIXED`； NDB群集不支持基于语句`STATEMENT`的复制。

## 三、关于权限、生效时间、有效时间、
设置此系统变量`binlog_format`的会话值是受限制的操作。 会话用户必须具有足以设置受限会话变量的特权。具体请看：<a href="https://dev.mysql.com/doc/refman/5.7/en/system-variable-privileges.html" target='	
_blank'>Section 5.1.8.1, “System Variable Privileges”</a>.

该变量的更改何时生效以及效果持续多久的规则**与其他MySQL服务器系统变量相同**。 想要查询更多的信息，请看<a href="https://dev.mysql.com/doc/refman/5.7/en/set-variable.html" target='_blank'>Section 5.1.8.1, “System Variable Privileges”</a>.

## 四、`MIXED`模式的特殊说明
当指定`MIXED`时，将使用基于语句的复制，除非保证仅基于行的复制会导致正确的结果。 例如，当语句包含用户定义函数（`UDF`）或`UUID（）`函数时，就会发生这种情况。

## 五、如何处理存储程序的说明
有关设置每种二进制日志记录格式时如何处理存储程序（存储过程和函数，触发器和事件）的详细信息，请看<a href="https://dev.mysql.com/doc/refman/5.7/en/stored-programs-logging.html" target='_blank'>Section 22.7, “Stored Program Binary Logging”</a>.

## 六、无法在运行时切换复制格式的例外
这里有一些你无法在运行时切换复制格式的例外：

- 从存储的函数或触发器中。

- 会话当前处于基于`ROW`的复制模式并且有打开着的临时表。

- 来自一个transaction内。

在以上这些情况下尝试切换格式会导致错误。

## 七、源与副本数据库的binlog格式关联关系
更改复制源服务器上的日志记录格式不会导致副本更改其日志记录格式以相互匹配。

如果复制副本启用了binlog，则在复制正在进行的过程中切换复制格式可能会导致问题，当源使用`ROW`或`MIXED`格式日志记录时，更改将导致副本数据库使用`STATEMENT`格式日志记录副本。副本无法将以`ROW`日志记录格式接收的二进制日志条目转换为`STATEMENT`格式，以用于其自己的二进制日志，因此这种情况可能导致复制失败。

想要获取更多信息，请看<a href="https://dev.mysql.com/doc/refman/5.7/en/binary-log-setting.html" target='_blank'>Section 5.4.4.2, “Setting The Binary Log Format”</a>.

## 八、binlog格式对其他服务器选项的影响
binlog格式会影响以下服务器选项的行为：
- [--replicate-do-db](https://dev.mysql.com/doc/refman/5.7/en/replication-options-replica.html#option_mysqld_replicate-do-db)

- [--replicate-ignore-db](https://dev.mysql.com/doc/refman/5.7/en/replication-options-replica.html#option_mysqld_replicate-ignore-db)

- [--binlog-do-db](https://dev.mysql.com/doc/refman/5.7/en/replication-options-binary-log.html#option_mysqld_binlog-do-db)

- [--binlog-ignore-db](https://dev.mysql.com/doc/refman/5.7/en/replication-options-binary-log.html#option_mysqld_binlog-ignore-db)

