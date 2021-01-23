关于Ubuntu拒绝root用户ssh远程登录
```shell
sudo vim /etc/ssh/sshd_config
```

找到并用#注释掉这行：PermitRootLogin prohibit-password

新建一行 添加：PermitRootLogin yes

重启服务

#sudo service ssh restart

## 附录
PermitRootLogin的可选项
众所周知，sshd_config是sshd的配置文件，其中PermitRootLogin可以限定root用户通过ssh的登录方式，如禁止登陆、禁止密码登录、仅允许密钥登陆和开放登陆，以下是对可选项的概括：

参数类别	是否允许ssh登陆	登录方式	交互shell
yes	允许	没有限制	没有限制
without-password	允许	除密码以外	没有限制
forced-commands-only	允许	仅允许使用密钥	仅允许已授权的命令
no	不允许	N/A	N/A

## 参考
https://www.cnblogs.com/miaodi/p/6718950.html
https://blog.csdn.net/huigher/article/details/52972013