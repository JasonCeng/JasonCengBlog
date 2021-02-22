# Shell语法问题之let_not_found

## 报错：let: not found

在ubuntu默认是指向bin/dash解释器的,dash是阉割版的bash,其功能远没有bash强大和丰富.并且dash不支持let和i++等功能.

## 解决办法:

sudo dpkg-reconfigure dash   选择 "否", 表示用bash代替dash

## 参考
[1] 编写shell时，遇到let: not found错误及解决办法[https://www.cnblogs.com/fanjp666888/p/10022038.html]

