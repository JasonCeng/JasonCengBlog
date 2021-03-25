# 浅析Dockerfile

```shell
FROM golang
WORKDIR /go/src/
COPY . .
EXPOSE 80
CMD ["/bin/bash", "/go/src/script/start.sh"]
```

- `FROM` 集成自哪个镜像，这里使用go程序官方提供的golang镜像。

- `WORKDIR` 工作目录。
- `COPY` 一个复制命令，把本地的所有文件复制到工作目录下。
- `EXPOSE` 这是对外开放的端口。
- `CMD` 执行一个带参数的命令。这里是为了让镜像启动时去执行script/start.sh的脚本。

注意：`COPY . . `This command will copy the current working directory of your local machine into the current working directory of the container.

参考：https://www.jianshu.com/p/5939dcf5c96e

