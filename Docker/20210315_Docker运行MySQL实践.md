# Docker运行MySQL实践

1、拉取镜像
```shell
$ docker pull mysql5.7
```
然后查看拉取得镜像：
```shell
$ docker images
REPOSITORY      TAG       IMAGE ID         CREATED          SIZE
mysql           5.7       cd3ed0dfff7e      4 weeks ago        437MB
```
2、创建mysql目录
```shell
$ cd ~/docker
$ mkdir -p mysql5.7/{data,conf,logs}
$ cd conf 
$ touch my.cnf
```
3、启动mysql镜像
```shell
$ docker run -d \
--name mysql57 mysql:5.7 \
-p 33306:3306 \
--privileged=true \
-v /home/docker/mysql5.7/conf/my.cnf:/etc/mysql/my.cnf \
-v /home/docker/mysql5.7/data/:/var/lib/mysql \
-v /home/docker/mysql5.7/logs/:/var/log/mysql \
-e MYSQL_ROOT_PASSWORD=123456 \
--restart=on-failure:3 \
--character-set-server=utf8mb4 \
--collation-server=utf8mb4_general_ci
```
命令说明：
```shell
 -d　 表示后台运行
 --name mysql 设值容器名称为mysql57
 -d　 表示后台运行
 -p 3306:3306：将容器的3306端口映射到主机的3306端口
 --privileged=true　设值MySQL 的root用户权限, 否则外部不能使用root用户登陆
 -v /home/docker/mysql5.7/conf/my.cnf:/etc/mysql/my.cnf
将主机/mysql/conf/my.cnf挂载到容器的 /etc/mysql/my.cnf 
 -v /home/docker/mysql5.7/data/:/var/lib/mysql 
将主机/home/docker/mysql57/data/目录挂载到容器的/var/lib/mysql
 -v /home/docker/mysql5.7/logs/:/var/log/mysql/ \
将主机/home/docker/mysql5.7/logs/目录挂载到容器的/var/log/mysql/
 -e MYSQL_ROOT_PASSWORD=1qaz@WSX：初始化root用户的密码
 --restar=always：自动重启，比如服务器突然断电，重启服务器之后不需要你重新手动启动
 --character-set-server=utf8mb4 设置数据库默认编码--collation-server=utf8mb4_general_ci 设置数据库默认编码
```
4、查看mysql容器
```shell
$ docker ps -a
```

5、新建MySQL用户
先进入容器
```shell
docker exec -it mysql57 bash
```
执行MySQL命令, 输入root密码, 连接MySQL
```shell
mysql -uroot -p
```
输入密码后, 执行下面命令创建新用户 (用户名: test , 密码: test123)

GRANT ALL PRIVILEGES ON *.* TO 'test'@'%' IDENTIFIED BY 'test123' WITH GRANT OPTION;

## 报错分析
### 1、mysqld failed while attempting to check config
```shell
2021-03-15 08:10:23+00:00 [ERROR] [Entrypoint]: mysqld failed while attempting to check config
	command was: mysqld -p 33306:3306 --privileged=true -v /home/docker/mysql5.7/conf/my.cnf:/etc/mysql/my.cnf -v /home/docker/mysql5.7/data/:/var/lib/mysql -v /home/docker/mysql5.7/logs/:/var/log/mysql -e MYSQL_ROOT_PASSWORD=123456 --restart=on-failure:3 --character-set-server=utf8mb4 --collation-server=utf8mb4_general_ci --verbose --help --log-bin-index=/tmp/tmp.feMNumn5e6
	2021-03-15T08:10:23.635141Z 0 [ERROR] mysqld: unknown option '-p'
2021-03-15T08:10:23.639916Z 0 [ERROR] Aborting
```

stackoverflow看到一个与这个问题看似无关的答案：
```
The docker command line is order sensitive. The order of args goes:

docker ${args_to_docker} run ${args_to_run} image_ref ${cmd_in_container}
Everything after ubuntu in your command goes to the command trying to be run. In your case -it. What you want instead is to pass -it to "run" so that you get interactive input with a tty terminal associated.

docker run -it ubuntu /bin/bash
```
意思差不多是说，docker命令是对（指令的）顺序敏感的。

我们回头检查下我们刚才的docker命令；
```shell
$ docker run -d \
--name mysql57 mysql:5.7 \
-p 33306:3306 \
--privileged=true \
-v /home/docker/mysql5.7/conf/my.cnf:/etc/mysql/my.cnf \
-v /home/docker/mysql5.7/data/:/var/lib/mysql \
-v /home/docker/mysql5.7/logs/:/var/log/mysql \
-e MYSQL_ROOT_PASSWORD=123456 \
--restart=on-failure:3 \
--character-set-server=utf8mb4 \
--collation-server=utf8mb4_general_ci
```
好的，是我错了;

到底是怎么回事呢，这里说明一下：

这里对于docker命令，有个小细节就是：
```shell
docker ${args_to_docker} run ${args_to_run} image_ref ${cmd_in_container}
```
比如`-e`是向容器设置环境变量参数、`-p`是指端口映射
这两项应该是作用于我们run命令的参数，是必须放在run之后，IMAGE_ID之前设置；

如果像上面的那样顺序，就会被mysql容器内部当成要给当前mysql服务设置的参数，mysql当然是不认识它们的，所以才会报错！

所以正确的一种启动命令格式是：
```shell
$ docker run \
--name mysql57 \
-p 33306:3306 \
-v /home/docker/mysql5.7/conf/my.cnf:/etc/mysql/my.cnf \
-v /home/docker/mysql5.7/data/:/var/lib/mysql \
-v /home/docker/mysql5.7/logs/:/var/log/mysql \
-e MYSQL_ROOT_PASSWORD=123456 \
--privileged=true \
--restart=on-failure:3 \
-d mysql:5.7 \
--character-set-server=utf8mb4 \
--collation-server=utf8mb4_general_ci
```

### 2、docker: Error response from daemon: OCI runtime create failed
报错日志：
```shell
docker: Error response from daemon: OCI runtime create failed: container_linux.go:367: starting container process caused: process_linux.go:495: container init caused: rootfs_linux.go:60: mounting "/home/docker/mysql5.7/conf/my.cnf" to rootfs at "/var/lib/docker/overlay2/05612ad8c259e25a84dabc8b62ecd75009181745b9845a604bac89c99dc6543d/merged/etc/mysql/mysql.cnf" caused: not a directory: unknown: Are you trying to mount a directory onto a file (or vice-versa)? Check if the specified host path exists and is the expected type.
```
原因分析：

` -v /home/docker/mysql5.7/conf/my.cnf:/etc/mysql/my.cnf `映射出错，找不到容器中的/etc/mysql/my.cnf

解决方案：

（1）去掉`-v /home/docker/mysql5.7/data/:/var/lib/mysql`以正常启动：
```shell
$ docker run \
--name mysql57 \
-p 33306:3306 \
-v /home/docker/mysql5.7/data/:/var/lib/mysql \
-v /home/docker/mysql5.7/logs/:/var/log/mysql \
-e MYSQL_ROOT_PASSWORD=123456 \
--privileged=true \
--restart=on-failure:3 \
-d mysql:5.7 \
--character-set-server=utf8mb4 \
--collation-server=utf8mb4_general_ci


-v /home/docker/mysql5.7/conf/my.cnf:/etc/mysql/my.cnf \
```

（2）进入容器，获取文件
```shell
$ docker exec -it mysql57 /bin/bash

```

