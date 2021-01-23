# Ubuntu搭建hadoop集群

## 准备3台Linux服务器
- 操作系统：Ubuntu 16.04 LTS
> 一台物理机，两台vm虚拟机，构成一主两从小集群

- 软件：openssh-server
> lsof:22 开启22端口，主要用于远程连接

- 个性化域名
> 定制hostname并配置hosts

```shell
#配置hosts
sudo vim /etc/hosts

#务必在IPv6之前新增，三台服务器新增内容相同：
192.168.1.113	ubuntu-master.com
192.168.1.114	ubuntu-slave1.com
192.168.1.115	ubuntu-slave2.com

#*****************我是分割线********************
#定制hostname
sudo vim /etc/hostname

#三台服务器分别为：
ubuntu-master
ubuntu-slave1
ubuntu-slave2

#以上修改需要重启才能生效
reboot
```

## 参考
[1]ubuntu18.04 搭建hadoop完全分布式集群（Master、slave1、slave2）共三个节点[https://blog.csdn.net/sunxiaoju/article/details/85222290]
