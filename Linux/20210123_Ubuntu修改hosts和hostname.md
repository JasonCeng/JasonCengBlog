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

