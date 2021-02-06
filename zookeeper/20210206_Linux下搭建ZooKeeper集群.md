# Linux下搭建ZooKeeper集群
本文基于Ubuntu 16.04 LTS

## 环境依赖
- 奇数台服务器，且非observer节点数>1；如果是偶数台服务器，可以把多出来的1台设置为observer
- Java 8+
- 防火墙开放2181端口：
```shell
ufw allow 2181
ufw reload
```




## 参考
[1]Apache Zookeeper 集群的搭建[https://blog.csdn.net/qq_33713328/article/details/88854991]