# 浅谈ddl、dcl、dml

## DML(Data Manipulation Language)数据操纵语言

DML(Data Manipulation Language 数据操控语言)用于操作数据库对象中包含的数据，也就是说操作的单位是记录。

我们常用到的` SELECT、UPDATE、INSERT、DELETE`，均属于DML。

## DDL(Data Definition Language)数据库定义语言

DDL(Data Definition Language 数据定义语言)用于操作对象和对象的属性，这种对象包括数据库本身，以及数据库对象，例如表、视图等等。

如`CREATE、ALTER、DROP`等，均属于DML。

DDL主要是用在定义或改变表的结构，数据类型，表之间的链接和约束等初始化工作上。

## DCL(Data Control Language)数据库控制语言

DCL(Data Control Language 数据控制语句)的操作是数据库对象(用户)的权限，这些操作的确定使数据更加的安全。

是用来设置或更改数据库用户或角色权限的语句，包括`grant、deny、revoke`等语句。

## 参考
[1] 浅谈 DML、DDL、DCL的区别[https://blog.csdn.net/fbz123456/article/details/89709598]
[2] DDL/DML/DCL区别概述[https://www.cnblogs.com/kawashibara/p/8961646.html]