# Windows下github代理加速

第一步：打开ipaddress.com,查询如下两个域名，并分别记录下其对应的ip：

1、github.com

2、github.global.ssl.fastly.net

第二步：更新host文件，如下：
```shell
192.30.253.112 github.com

151.101.185.194 github.global.ssl.fastly.net
```

第三步：打开CMD，清理下DNS
```shell
ipconfig /flushdns
```