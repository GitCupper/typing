# Docker常用操作命令

常用的Docker操作命令

``` shell
docker images: 查看镜像，后可跟 "| grep 内容"，可根据内容进行筛选。
如：docker images | grep nginx

docker images [OPTIONS] [REPOSITORY[:TAG]]
OPTIONS说明:
-a: 列出本地所有的镜像
--digests: 显示镜像的摘要信息
-f: 显示满足条件的镜像
--format: 指定返回值的模板文件
--no-trunc: 显示完整的镜像信息
-q: 只显示镜像ID

docker run: 创建一个新的容器并运行一个命令
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
OPTIONS说明:
-d: 后台运行容器，并返回容器ID
-i: 以交互模式运行容器，通常与 -t 同时使用
-p: 指定端口映射，格式为：主机(宿主)端口:容器端口
-t: 为容器重新分配一个伪输入终端，通常与 -i 同时使用
--name "nginx": 为容器指定一个名称
-h "localhost": 指定容器的hostname
-e spring.profiles.active="dev": 设置环境变量
--env-file=[]: 从指定文件读环境变量
-m :设置容器使用内存最大值
--volume /home/data:/etc/data :  绑定一个卷
and so on

如：docker run -d -t -p 80:80 -v /home/data:/usr/data --name nginx nginx:latest

docker create: 创建一个新的容器但不启动它

docker stop: 停止一个运行的容器
docker stop containerName

docker restart: 重启一个容器
docker restart containerName

docker start: 启动一个被停止的容器
docker start containerName

docker ps [OPTIONS]: 列出容器
OPTIONS说明:
-a: 显示所有的容器，包括未运行的
-f: 根据条件过滤显示的内容
--format: 指定返回值的模板文件
-l: 显示最近创建的容器
-n: 列出最近创建的n个容器
--no-trunc: 不截断输出
-q: 静默模式，只显示容器编号
-s: 显示总的文件大小

docker ps -a: 查看所有容器

docker ps: 查看正在运行的容器

docker exec: 进入一个运行中的容器执行命令
如：docker exec -it 容器id sh or bash or /bin/bash
表示在容器中开启一个交互模式的终端

docker rm: 删除一个容器，可加-f 表示强制 -v：并删除挂载卷
删除所有停止的容器：docker rm $(docker ps -a -q)

docker rmi: 删除一个镜像，可加-f 表示强制

docker inspect : 获取容器/镜像的元数据
如：docker inspect [OPTIONS] NAME|ID [NAME|ID...]
OPTIONS说明:
-f: 指定返回值的模板文件
-s: 显示总文件大小
-type: 为指定类型返回json数据

获取正在运行的容器 nginx 的 IP:
docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' nginx

docker kill: 杀死一个运行中的容器
如： docker kill -s killyou nginx

docker logs: 获取容器的日志
如：docker logs -f -t 容器id or docker logs -f -t --tail=100 容器id

docker build: 命令用于使用 Dockerfile 创建镜像
docker build [OPTIONS] PATH | URL | -
OPTIONS说明:
-f: 指定要使用的Dockerfile路径
-m: 设置内存最大值
--memory-swap: 设置Swap的最大值为内存+swap，"-1"表示不限swap
--no-cache: 创建镜像的过程不使用缓存
--pull: 尝试去更新镜像的新版本
-q: 安静模式，成功后只输出镜像 ID
--rm: 设置镜像成功后删除中间容器
--shm-size: 设置/dev/shm的大小，默认值是64M
--tag: 镜像的名字及标签，通常 name:tag 或者 name 格式；可以在一次构建中为一个镜像设置多个标签
--network: 默认 default。在构建期间设置RUN指令的网络模式

docker build -t 镜像标签名 .: docker build -t nginx:latest .
docker build -f /path/to/a/Dockerfile .

docker tag: 标记本地镜像，将其归入某一仓库

docker tag nginx nginx:old

docker save: 将指定镜像保存成 tar 归档文件
docker save -o nginx.tar nginx:latest

docker load: 导入使用 docker save 命令导出的镜像
docker load -i tar文件名

docker info: 查看docker环境信息

docker version: 查看docker版本信息

docker login: 登录一个Docker镜像仓库
docker login -u 用户名 -p 密码

docker logout: 退出登录

docker pull: 拉取或者更新指定镜像 -a 拉取所有的tag的镜像

docker push: 将本地的镜像上传到镜像仓库
```

