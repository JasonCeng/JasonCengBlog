# Windows下github代理加速

第一步：打开ipaddress.com,查询如下两个域名，并分别记录下其对应的ip：

1、github.com

2、github.global.ssl.fastly.net

第二步：更新host文件，如下：
```shell
140.82.112.4 github.com

199.232.69.194 github.global.ssl.fastly.net
```

第三步：打开CMD，清理下DNS
```shell
ipconfig /flushdns
```