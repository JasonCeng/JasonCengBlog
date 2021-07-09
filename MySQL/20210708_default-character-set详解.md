## 配置文件加载顺序
mysql配置文件my.cnf读取顺序：
1、/etc/my.cnf
2、/etc/mysql/my.cnf
3、~/.my.cnf

mysql配置文件mysql.cnf读取顺序：
1、/etc/mysql/mysql.cnf
2、/etc/mysql/conf.d/mysql.cnf

mysql配置文件mysqld.cnf读取顺序：
1、/etc/mysql/mysql.conf.d/mysqld.cnf

## default-character-set
```shell
# 设置所有客户端的字符集
[client]
default-character-set=charset_name

# 设置mysql命令连接时的客户端字符集
[mysql]
default-character-set=charset_name
```
这通常是不必要的。 但是，当 character_set_system 不同于 character_set_server 或 character_set_client 并且您手动输入字符（作为数据库对象标识符、列值或两者）时，这些字符可能会在客户端的输出中显示不正确，或者输出本身的格式可能不正确。 在这种情况下，使用 --default-character-set=system_character_set 启动 mysql 客户端 - 即，将客户端字符集设置为与系统字符集匹配 - 应该可以解决问题。

--default-character-set 用于设置客户端的输入字符集。

客户端程序连接字符集配置
当客户端连接到服务器时，它指示它要使用哪个字符集与服务器通信。 （实际上，客户端指示该字符集的默认排序规则，服务器可以从中确定字符集。）服务器使用此信息将 character_set_client、character_set_results、character_set_connection 系统变量设置为字符集，并将 collat​​ion_connection 设置为字符设置默认排序规则。实际上，服务器执行等效于 SET NAMES 操作。

如果服务器不支持请求的字符集或排序规则，它会回退到使用服务器字符集和排序规则来配置连接。有关此回退行为的其他详细信息，请参阅连接字符集错误处理。

mysql、mysqladmin、mysqlcheck、mysqlimport 和 mysqlshow 客户端程序确定要使用的默认字符集如下：

在没有其他信息的情况下，每个客户端使用编译的默认字符集，通常是 latin1。

每个客户端都可以根据操作系统设置自动检测要使用的字符集，例如 Unix 系统上的 LANG 或 LC_ALL 语言环境环境变量的值或 Windows 系统上的代码页设置。对于 OS 提供语言环境的系统，客户端使用它来设置默认字符集，而不是使用编译的默认值。例如，将 LANG 设置为 ru_RU.KOI8-R 会导致使用 koi8r 字符集。因此，用户可以在他们的环境中配置语言环境以供 MySQL 客户端使用。

如果没有精确匹配，操作系统字符集将映射到最接近的 MySQL 字符集。如果客户端不支持匹配的字符集，则使用编译的默认值。例如，不支持 ucs2 作为连接字符集，因此它映射到编译的默认值。

C 应用程序可以通过在连接到服务器之前调用 mysql_options() 来使用基于操作系统设置的字符集自动检测：

mysql_options(mysql,
              MYSQL_SET_CHARSET_NAME，
              MYSQL_AUTODETECT_CHARSET_NAME);
每个客户端都支持一个 --default-character-set 选项，该选项使用户能够明确指定字符集以覆盖客户端以其他方式确定的任何默认值。

笔记
某些字符集不能用作客户端字符集。尝试将它们与 --default-character-set 一起使用会产生错误。请参阅不允许的客户端字符集。

对于 mysql 客户端，要使用不同于默认的字符集，您可以在每次连接到服务器时显式执行 SET NAMES 语句（请参阅客户端程序连接字符集配置）。要更轻松地完成相同的结果，请在选项文件中指定字符集。例如，以下选项文件设置在每次调用 mysql 时更改设置为 koi8r 的三个与连接相关的字符集系统变量：

[mysql]
默认字符集=koi8r
如果您使用启用了自动重新连接的 mysql 客户端（不推荐），最好使用 charset 命令而不是 SET NAMES。例如：

mysql> 字符集 koi8r
字符集已更改
charset 命令发出 SET NAMES 语句，并且还会更改 mysql 在连接断开后重新连接时使用的默认字符集。

在配置客户端程序时，您还必须考虑它们执行的环境。请参见第 10.5 节，“配置应用程序字符集和排序规则”。

## 原文（参考[2]）：
Client Program Connection Character Set Configuration
When a client connects to the server, it indicates which character set it wants to use for communication with the server. (Actually, the client indicates the default collation for that character set, from which the server can determine the character set.) The server uses this information to set the character_set_client, character_set_results, character_set_connection system variables to the character set, and collation_connection to the character set default collation. In effect, the server performs the equivalent of a SET NAMES operation.

If the server does not support the requested character set or collation, it falls back to using the server character set and collation to configure the connection. For additional detail about this fallback behavior, see Connection Character Set Error Handling.

The mysql, mysqladmin, mysqlcheck, mysqlimport, and mysqlshow client programs determine the default character set to use as follows:

In the absence of other information, each client uses the compiled-in default character set, usually latin1.

Each client can autodetect which character set to use based on the operating system setting, such as the value of the LANG or LC_ALL locale environment variable on Unix systems or the code page setting on Windows systems. For systems on which the locale is available from the OS, the client uses it to set the default character set rather than using the compiled-in default. For example, setting LANG to ru_RU.KOI8-R causes the koi8r character set to be used. Thus, users can configure the locale in their environment for use by MySQL clients.

The OS character set is mapped to the closest MySQL character set if there is no exact match. If the client does not support the matching character set, it uses the compiled-in default. For example, ucs2 is not supported as a connection character set, so it maps to the compiled-in default.

C applications can use character set autodetection based on the OS setting by invoking mysql_options() as follows before connecting to the server:

mysql_options(mysql,
              MYSQL_SET_CHARSET_NAME,
              MYSQL_AUTODETECT_CHARSET_NAME);
Each client supports a --default-character-set option, which enables users to specify the character set explicitly to override whatever default the client otherwise determines.

Note
Some character sets cannot be used as the client character set. Attempting to use them with --default-character-set produces an error. See Impermissible Client Character Sets.

With the mysql client, to use a character set different from the default, you could explicitly execute a SET NAMES statement every time you connect to the server (see Client Program Connection Character Set Configuration). To accomplish the same result more easily, specify the character set in your option file. For example, the following option file setting changes the three connection-related character set system variables set to koi8r each time you invoke mysql:

[mysql]
default-character-set=koi8r
If you are using the mysql client with auto-reconnect enabled (which is not recommended), it is preferable to use the charset command rather than SET NAMES. For example:

mysql> charset koi8r
Charset changed
The charset command issues a SET NAMES statement, and also changes the default character set that mysql uses when it reconnects after the connection has dropped.

When configuration client programs, you must also consider the environment within which they execute. See Section 10.5, “Configuring Application Character Set and Collation”.

## 参考
[1] https://blog.csdn.net/menghuanzhiming/article/details/78065072

[2] https://dev.mysql.com/doc/refman/5.7/en/charset-connection.html

[3] https://www.cnblogs.com/mangu-uu/p/4162984.html