# Docker离线安装MySQL.和Nigix
首先找一个能联网的机器，该机器安装完docker 并且有mysql nginx的镜像：

## Mysql安装
1、下载mysql 5.7镜像：```docker pull mysql:5.7```

2、在能联网的机器上执行：```docker save -o /root/app/mysql57.tar mysql:5.7```

3、把```/root/app/mysql57.tar```文件上传到无法上网的机器上然后执行：```docker load -i /root/Downloads/mysql57.tar```

3、启动mysql：```docker run -d --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql:5.7```

## Nginx安装

1、下载nginx最新版本镜像：```docker pull nginx:latest```

1、在能联网的机器上执行：```docker save -o /root/app/nginx.tar nginx:latest```

2、把```/root/app/nginx.tar```文件上传到无法上网的机器上然后执行：```docker load -i /root/Downloads/nginx.tar```

3、输入```docker images```检查nginx是否已安装

4、启动Nginx: ```docker run --name nginx -p 80:80 -d nginx```
```-name nginx```：容器名称。

```-p 80:80```：端口进行映射，将本地 80 端口映射到容器内部的 80 端口。

```-d nginx```：设置容器在在后台一直运行。