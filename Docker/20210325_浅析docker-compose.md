# 浅析docker-compose

Docker Compose是一个工具，用于在使用Compose文件格式定义的Docker上运行多容器应用程序。 Compose文件用于定义构成应用程序的一个或多个容器的配置方式。 拥有Compose文件后，您可以使用一个命令创建并启动您的应用程序：`docker-compose up`。 

## `docker-compose.yml`示例
```shell
version: '2'

networks:
  basic:

services:

  world:
    container_name: world
    image: go-web
    ports:
      - "8099:80"
    volumes:
      - ./app/go/world:/go/src/app:rw
    networks:
      - basic
```

参考：

https://docs.docker.com/compose/