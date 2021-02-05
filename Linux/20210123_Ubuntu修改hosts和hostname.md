# Ubuntu修改hosts和hostname

## 修改hosts并生效
```shell
sudo vim /etc/hosts

#务必在IPv6之前新增
192.168.1.113	ubuntu-master.com
192.168.1.114	ubuntu-slave1.com
192.168.1.115	ubuntu-slave2.com

sudo /etc/init.d/networking restart
#若hosts没有生效，则重启
reboot

#测试
ping ubuntu-master.com
```
*Tips：ip与域名之间需要用tab键隔开*

## 修改hostname
```shell
sudo vim /etc/hostname

#修改第一行为你想要的主机名,如
ubuntu-master

#重启以生效
reboot
```

## 常见问题

### 1、sudo 出现unable to resolve host
假设这台机器名字叫abc(机器的hostname), 每次执行` sudo `就出现这个警告讯息:`sudo: unable to resolve host abc`

虽然` sudo `还是可以正常执行, 但是警告讯息每次都出来,而这只是机器在反解上的问题, 所以就直接从` /etc/hosts `设定, 让 abc(hostname) 可以解回` 127.0.0.1 `的 IP 即可。

在127.0.0.1 localhost 后面加上主机名称(hostname) 即可:
```shell
sudo vim /etc/hosts

127.0.0.1       localhost abc
```

## 参考
[1]Ubuntu sudo 出现unable to resolve host 解决方法[https://www.cnblogs.com/wanzhou0310/p/7407322.html]