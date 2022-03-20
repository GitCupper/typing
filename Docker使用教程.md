# Docker使用教程

## 一、Docker的安装

### 1. Windows下安装Docker

略

### 2. Linux下安装Docker

```shell
# 下载安装脚本
$ curl -fsSL get.docker.com -o get-docker.sh
# 运行安装脚本
$ sh get-docker.sh
# 运行Docker
$ sudo systemctl start docker
```

## 二、创建容器Container

### 1. Docker命令

``` shell
# 查看容器ID
$ docker ps -aq
# 批量停止容器
$ docker stop $(docker ps -aq)
# 强制删除容器
$ docker rm 容器ID -f
# 容器的端口映射
$ docker run -p 80:80 nginx
# 容器的后台运行
$ docker run -d nginx
# 将后台的容器转到前台
$ docker attach 容器ID
# 查看容器的运行日志
$ docker logs 容器ID
# 查看容器的运行日志，并进行跟踪
$ docker logs -f 容器ID

```

### 2. Docker image命令

``` shell
# 查看镜像信息
$ docker inspect 镜像名/镜像ID
# 构建镜像
$ docker image build -t hello:1.0 .
# 指定Dockerfile的名称构建
$ docker image build -f Dockerfile.good -t ipinfo-good .
# 查看构建历史
$ docker image history 镜像ID
# 清除所有容器
$ docker system prune -f
# 清除所有镜像
$ docker image prune 

```

