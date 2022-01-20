---
title: docker 基本使用总结
tags: interview
# article_header:
#   type: cover
#   image:
#     src: /assets/images/helloworld.jpg
--- 

<!-- write excerpt here -->
docker 基本使用总结

<!--more-->


## 基本概念

- images 镜像，可以认为就是iso的文件
- container 可以认为你给一个电脑装了images

## 基本命令

- `docker images` 显示已有的镜像
- `docker ps -a` 显示创建的容器
- `docker rmi <image id>`删除images（镜像），通过image的id来指定删除谁；删除全部`docker rmi $(docker images -q)`
- `docker rm $(docker ps -aq)`删除所有container
- `docker container run -p 8000:3000 koa-demo:0.0.1` 启动容器
- `docker container run -it imagesid /bin/bash`
- `docker start container_id` 开启容器
- `docker stop container_id` 停止容器
- `docker cp`
- `docker exec -it b1a650b2b6f9  /bin/bash` 进入已经启动的container的bash

## Dockerfile

```docker
FROM php:7.0-apache
RUN apt-get update && \
    apt-get install -y zlib1g-dev
RUN docker-php-ext-install mysqli
RUN docker-php-ext-install zip
CMD apache2-foreground
```

- FROM 是基础镜像
- RUN 构造镜像执行的命令
- CMD 是容器启动的时候自动执行的命令
- ADD
- COPY
- build的方式：`docker build -t tag:v1 .`

## Docker Compose

- install
    - pip3 install docker-compose

```docker
mysql:
    image: mysql:5.7
    environment:
     - MYSQL_ROOT_PASSWORD=123456
    volumes:
     - ./init:/docker-entrypoint-initdb.d
web:
    build: ./html
    links:
     - mysql
    ports:
     - "2222:80"
    working_dir: /var/www/html
    volumes:
     - ./html:/var/www/html
```

```docker
mysql:
    image: mysql:5.7
    environment:
     - MYSQL_ROOT_PASSWORD=123456
    volumes:
     - ./init:/docker-entrypoint-initdb.d
web:
    image: php:ctf
    links:
     - mysql
    ports:
     - "2222:80"
    working_dir: /var/www/html
    volumes:
     - ./html:/var/www/html
```

注意在php里面不能写localhost了，要写mysql