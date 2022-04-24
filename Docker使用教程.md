# Docker使用教程

## 一、Docker的安装

### 1. Windows下安装Docker

略

### 2. Linux下安装Docker

以CentOS7为例。

#### 2.1 制裁原有的旧版本

``` shell
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

#### 2.2 安装社区版Docker Engine-Community

1. 设置仓库

安装所需的软件包。`yum-utils`提供了`yum-config-manager`，并且`device mapper`存储驱动程序需要`device-mapper-persistent-data`和`lvm2`。

``` shell
$ sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```

使用以下的命令来设置稳定的仓库

- 官方源

```shell
$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

- 清华大学源

``` shell
$ sudo yum-config-manager \
    --add-repo \
    https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/docker-ce.repo
```

- 配置镜像加速器

针对Docker客户端大于1.10.0的用户

您可以通过修改daemon配置文件`/etc/docker/daemon.json`来使用加速器

``` shell
$ sudo mkdir -p /etc/docker
$ sudo tee /etc/docker/daemon.json <<-'EOF'
{
	"registry-mirrors": ["https://82m9ar63.mirror.aliyuncs.com"]
}
EOF
$ sudo systemctl daemon-reload
$ sudo systemctl res
```



#### 2.3 安装Docker

```shell
$ sudo yum install docker-ce docker-ce-cli containerd.io
```

#### 2.4 使用官方脚本安装Docker

```shell
# 下载安装脚本
$ curl -fsSL get.docker.com -o get-docker.sh
# 运行安装脚本
$ sh get-docker.sh
# 运行Docker
$ sudo systemctl start docker
```

#### 2.5 启动Docker

``` shell
$ sudo systemctl enable docker
$ sudo systemctl start docker
```

#### 2.6 测试Docker

```shell
# 查看Docker版本
$ docker --version
# Docker的HelloWorld
$ docker run hello-world
```

#### 2.7 设置Docker开机自启动

``` shell
# 设置Docker为服务
$ systemctl enable docker.service
```

#### 2.8 删除安装包

``` shell
$ yum remove docker-ce
```

#### 2.9 删除镜像、容器、配置文件等内容

```shell
$ rm -rf /var/lib/docker
```

#### 2.10 配置Docker的daemon.json

``` shell
$ cat > /etc/docker/daemon.json << EOF
{
	"graph": "/data/docker",
	"storage-driver": "overlay2",
	"insecure-registries": ["registry.access.redhat.com", "quay.io"],
	"registry-mirrors": ["https://q2gr04ke.mirror.aliyuncs.com"],
	"bip": "172.7.5.1/24",
	"exec-opts": ["native.cgroupdriver=systemd"],
	"live-restore": true
}
EOF

$ mkdir -p /data/docker
$ systemctl restart docker
```



### 3. 其它

#### 3.1 Linux防火墙配置

``` shell
# 查看防火墙状态
$ systemctl status firewalld
# 开放端口：
$ firewall-cmd --zone=public --add-port=3306/tcp --permanent
# 重启防火墙
$ firewall-cmd --reload
# 
$ firewall-cmd --list-all
# 列表防火墙端口开放配置
$ firewall-cmd --list-ports
# 关闭防火墙：
$ sudo systemctl stop firewalld
```

#### 3.2 yum相关

``` shell
$ yum install epel-release -y
$ yum list docker --show-duplicates
$ yum install -y yum-utils
$ yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# 安装Docker
$ sudo yum install docker-ce docker-ce-cli containerd.io
```

### 4. Docker的离线安装

#### 4.1 下载Docker离线安装文件

[https://download.docker.com/linux/static/stable/x86_64/](https://download.docker.com/linux/static/stable/x86_64/)

#### 4.2 上传Docker安装文件到服务器

将docker-18.06.3-ce.tgz文件上传到centos7-linux系统上，用ftp工具上传即可。

### 4.3 解压缩

``` shell
[root@localhost java]# tar -zxvf docker-18.06.3-ce.tgz
```

#### 4.4 复制文件

```shell
[root@localhost java]# cp docker/* /usr/bin/
```

#### 4.5 创建docker.service文件

进入`/etc/systemd/system/`目录，并创建`docker.service`文件

```shell
[root@localhost java]# cd /etc/systemd/system/
[root@localhost system]# touch docker.service
# 或者
[root@localhost ~]# vim /etc/systemd/system/docker.service
```

#### 4.6 修改`docker.service`文件内容

```shell
$ vim  docker.service
```

**注意**：--insecure-registry=192.168.200.128 此处改为你自己服务器ip

```shell
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service
Wants=network-online.target

[Service]
Type=notify
# the default is not to use systemd for cgroups because the delegate issues still
# exists and systemd currently does not support the cgroup feature set required
# for containers run by docker
ExecStart=/usr/bin/dockerd --selinux-enabled=false --insecure-registry=192.168.200.128
ExecReload=/bin/kill -s HUP $MAINPID
# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
# Uncomment TasksMax if your systemd version supports it.
# Only systemd 226 and above support this version.
#TasksMax=infinity
TimeoutStartSec=0
# set delegate yes so that systemd does not reset the cgroups of docker containers
Delegate=yes
# kill only the docker process, not all processes in the cgroup
KillMode=process
# restart the docker process if it exits prematurely
Restart=on-failure
StartLimitBurst=3
StartLimitInterval=60s

[Install]
WantedBy=multi-user.target
```

#### 4.7 给`docker.service`文件添加执行权限

``` shell
[root@localhost system]$ chmod 777 /etc/systemd/system/docker.service
```

#### 4.8 重新加载配置文件

每次有修改`docker.service`文件时，都需要重新加载。

```shell
[root@localhost system]$ systemctl daemon-reload
```

#### 4.9 启动Docker

``` shell
[root@localhost system]$ systemctl enable docker.service
```

#### 4.10 设置开机自启动

``` shell
[root@localhost system]$ systemctl enable docker.service
```

#### 4.11 查看Docker状态

```shell
[root@localhost system]$ systemctl status docker
```

#### 4.12 配置镜像加速器

默认是到国外拉取镜像速度慢,可以配置国内的镜像如：阿里、网易等等。下面配置一下网易的镜像加速器。打开docker的配置文件: /etc/docker/daemon.json文件：

``` shell
[root@localhost docker]$ vim /etc/docker/daemon.json

# 内容如下
{"registry-mirrors": ["http://hub-mirror.c.163.com"]}

## 或者使用如下命令：
cat > /etc/docker/daemon.json << EOF
{"registry-mirrors": ["http://hub-mirror.c.163.com"]}
EOF
```

#### 4.13 重启Docker

``` shell
[root@localhost docker]$ service docker restart
```











## 二、创建容器Container

### 1. Docker命令

``` shell
# 查看Docker信息
$ docker info
# 查看容器ID
$ docker ps -aq
# 批量停止容器
$ docker stop $(docker ps -aq)
# 强制删除容器
$ docker rm ${容器ID} -f
# 容器的端口映射
$ docker run -p 80:80 nginx
# 容器的后台运行
$ docker run -d nginx
# 将后台的容器转到前台
$ docker attach ${容器ID}
# 查看容器的运行日志
$ docker logs 容器ID
# 查看容器的运行日志，并进行跟踪
$ docker logs -f ${容器ID}

```

### 2. Docker image命令

``` shell
# 查看镜像信息
$ docker inspect ${镜像名/镜像ID}
# 查看本机Docker镜像
$ docker images
$ docker image ls
# 拉取镜像
$ docker pull ${镜像名:版本号}
$ docker pull mysql:5.7
$ docker pull mysql
# 构建镜像
$ docker image build -t hello:1.0 .
# 指定Dockerfile的名称构建
$ docker image build -f Dockerfile.good -t ipinfo-good .
# 查看构建历史
$ docker image history ${容器ID}
# 清除所有容器
$ docker system prune -f
# 清除所有镜像
$ docker image prune 
# 启动容器
$ docker start ${容器ID/容器名}
# 重启容器
$ docker restart ${容器ID/容器名}
# 停止容器
$ docker stop ${容器ID/容器名}
# 强制停止容器
$ docker kill ${容器ID/容器名}
# 显示容器运行状态的容器
$ docker container ls
# 列表显示所有容器
$ docker ps -a
# 导出镜像为压缩包
$ docker image save -o nacos.tar nacos/nacos-server
# 从压缩包加载镜像
$ docker load --input nacos.tar
# 查看容器日志
$ docker logs ${容器名/容器ID} # 查看容器日志

```

## 三、Docker安装常用软件

### 1. Docker安装MySQL

#### 1.1 拉取官方镜像

``` shell
$ docker pull mysql:5.7  # 拉取MySQL 5.7
$ docker pull mysql      # 拉取最新版MySQL
```

#### 1.2 检查是否拉取成功

``` shell
$ docker images
```

#### 1.3 不建立数据库文件目录映射方式创建容器

``` shell
$ sudo docker run -p 3306:3306 --name mysql -e MYSQL_ROOT_PASSWORD-123456 -d mysql
```

配置参数解释：

- `-name`：容器名，此外命名为：mysql
- `-e`：配置信息，此处配置mysql的root用户的登录密码
- `-p`：端口映射，此处映射主机3306端口到容器的3306端口
- `-d`：后台运行容器，保证在退出终端后容器继续运行

#### 1.4 建立数据库文件目录映射

添加MySQL的配置文件

``` shell
mkdir -p /usr/local/docker/mysql/conf/
touch /usr/local/docker/mysql/conf/my.cnf
vim /usr/local/docker/mysql/conf/my.cnf

# 复制以下内容到my.cnf配置文件中去
[mysqld]
user=mysql
default-storage-engine=INNODB
character-set-client-handshake=FALSE
default_authentication_plugin=mysql_native_password
character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci
init_connect='SET NAMES utf8mb4'
default-time-zone = '+08:00'
[client]
default-character-set=utf8mb4
[mysql]
default-character-set=utf8mb4
```

创建容器

``` shell
sudo docker run -p 3306:3306 --name mysql \
-v /usr/local/docker/mysql/conf:/etc/mysql \
-v /usr/local/docker/mysql/conf/my.cnf:/etc/mysql/my.cnf \
-v /usr/local/docker/mysql/logs:/var/log/mysql \
-v /usr/local/docker/mysql/data:/var/lib/mysql \
-v /usr/local/docker/mysql/mysql-files:/var/lib/mysql-files \
-e MYSQL_ROOT_PASSWORD=123456 \
-e TZ=Asia/Shanghai \
--restart=always \
-d mysql
```

配置参数解析：

- `-v`：主机和容器的目录映射关系，“:”前为主机目录，之后为容器目录

#### 1.5 检查容器是否正确运行

``` shell
docker container ls # 查看现有容器

docker ps -a # 查看容器状态
docker start ${容器ID} # 启动一个已经停止的容器
docker stop ${容器ID} # 停止一个已经启动的容器
```

容器状态解析：

- `created`：已经被创建，但还没有被启动
- `running`：运行中
- `paused`： 容器的进程被暂停
- `restarting`：容器的进程正在重启过程中
- `exited`：容器之前运行过，但现在处于停止状态

#### 1.6 进入Docker容器，本地连接MySQL客户端

``` shell
# sudo docker exec -it ${容器ID/容器名} {使用的shell}
$ sudo docker exec -it mysql /bin/bash
# 进入容器后，连接MySQL，进入客户端
mysql -uroot -p123456
```

#### 1.7 远程访问时应注意的问题

由于已经将容器的`3306`端口映射到了主机的`3306`端口，所以只需要访问主机的`3306`端口即可。

#### 1.8 开放主机防火墙

``` shell
# 开放端口：
$ systemctl status firewalld
$ firewall-cmd --zone=public --add-port=3306/tcp --permanent
$ firewall-cmd --reload
$ firewall-cmd --list-all
$ firewall-cmd --list-ports
# 关闭防火墙：
$ sudo systemctl stop firewalld
```

#### 1.9 进入Docker本地客户端设置远程访问帐号

``` shell
$ sudo docker exec -it mysql bash
$ mysql -uroot -p123456
mysql> grant all privileges on *.* to root@'%' identified by "password";
```

原理：

```shell
# mysql使用mysql数据库中的user表来管理权限，修改user表就可以修改权限（只有root账号可以修改）

mysql> use mysql;
Database changed

mysql> select host,user,password from user;
+--------------+------+-------------------------------------------+
| host                    | user      | password                                                                 |
+--------------+------+-------------------------------------------+
| localhost              | root     | *A731AEBFB621E354CD41BAF207D884A609E81F5E      |
| 192.168.1.1            | root     | *A731AEBFB621E354CD41BAF207D884A609E81F5E      |
+--------------+------+-------------------------------------------+
2 rows in set (0.00 sec)

mysql> grant all privileges  on *.* to root@'%' identified by "password";
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

mysql> select host,user,password from user;
+--------------+------+-------------------------------------------+
| host                    | user      | password                                                                 |
+--------------+------+-------------------------------------------+
| localhost              | root      | *A731AEBFB621E354CD41BAF207D884A609E81F5E     |
| 192.168.1.1            | root      | *A731AEBFB621E354CD41BAF207D884A609E81F5E     |
| %                       | root      | *A731AEBFB621E354CD41BAF207D884A609E81F5E     |
+--------------+------+-------------------------------------------+
3 rows in set (0.00 sec)
```

#### 1.10 修改MySQL的时区设置

MySQL安装后，默认的时区设置为`SYSTEM`，并没有指向中国当地的时区，因此需要进行设置：

```shell
# 登录MySQL
mysql> show variables like "%time_zone%";
+------------------+--------+
| Variable_name  | Value |
+------------------+--------+
| system_time_zone | CST  |
| time_zone    | SYSTEM |
+------------------+--------+
2 rows in set (0.00 sec)

# 修改时区：方法一
mysql> set global time_zone = '+8:00'; ##修改mysql全局时区为北京时间
mysql> set time_zone = '+8:00'; ##修改当前会话时区
mysql> flush privileges; #立即生效
```

#### 1.11 修改MySQL8的密码验证策略

在新版的MySQL8中，`caching_sha2_password`是默认的身份验证插件，而不是`mysql_native_password`，因此如果要使用旧验证插件登录的用户，需要使用新的验证插件，或是将默认的验证插件修改为`mysql_native_password`。

``` mysql
# 本地
# 修改加密规则（非必须）
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY '123456' PASSWORD EXPIRE NEVER; 
# 更新用户的密码
mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '123456'; 
# 刷新权限
mysql> FLUSH PRIVILEGES;
# 重置密码（==非必须==）
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY '123456';

# 远程
# 修改加密规则（非必须）
mysql> ALTER USER 'root'@'%' IDENTIFIED BY '123456' PASSWORD EXPIRE NEVER; 
# 更新用户的密码
mysql> ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
# 刷新权限
mysql> FLUSH PRIVILEGES;
# 重置密码（==非必须==）
mysql> ALTER USER 'root'@'%' IDENTIFIED BY '123456';

# 查看修改结果
mysql> SELECT Host, User, plugin from mysql.user;
```

#### 1.12 Docker下安装MySQL 5.7

```shell
# 拉取mysql5.7的镜像
$ docker pull mysql:5.7

# 创建本地文件夹
$ mkdir -p /local/docker/mysql/conf /local/docker/mysql/logs /local/docker/mysql/data

# 启动mysql镜像
$ sudo docker run -p 3306:3306 --name mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7

# 配置文件存储的启动方式
$ sudo docker run -p 3306:3306 --name mysql \
-v /local/docker/mysql/conf:/etc/mysql \
-v /local/docker/mysql/logs:/var/log/mysql \
-v /local/docker/mysql/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=123456 \
--restart=always \
-d mysql:5.7

# 查看Docker容器
$ docker container ls

# 启动现有的Docker容器
$ docker start ${容器ID/容器名}

# 停止现有的Docker容器
$ docker stop ${容器ID/容器名}

# 重启现有的Docker容器
$ docker restart ${容器ID/容器名}

# 进入容器中的MySQL
$ docker exec -it ${容器ID/容器名} /bin/bash
mysql -uroot -p
(密码：1)


# 开放端口：
$ systemctl status firewalld
$ firewall-cmd --zone=public --add-port=3306/tcp --permanent
$ firewall-cmd --reload
$ firewall-cmd --list-ports
# 关闭防火墙：
$ sudo systemctl stop firewalld
```



### 2. Docker下安装Elastic Search

#### 2.1 下载镜像

``` shell
# 拉取镜像
$ docker pull registry.docker-cn.com/library/elasticsearch

$ docker pull docker.elastic.co/elasticsearch/elasticsearch:8.0.0
docker run --name es01 --net elastic -p 9200:9200 -p 9300:9300 -it docker.elastic.co/elasticsearch/elasticsearch:8.0.0

###########################################
$ docker pull elasticsearch:7.17.0
$ docker run --name elasticsearch -d -e ES_JAVA_OPTS="-Xms512m -Xmx512m" -e "discovery.type=single-node" -p 9200:9200 -p 9300:9300 elasticsearch:7.17.0

# 开放防火墙
# 开放端口：
$ systemctl status firewalld
$ firewall-cmd --zone=public --add-port=9200/tcp --permanent
$ firewall-cmd --zone=public --add-port=9300/tcp --permanent
$ firewall-cmd --reload

$ firewall-cmd --list-ports
```

#### 2.2 制作容器

##### 2.2.1 设置max_map_count

```shell
cat /proc/sys/vm/max_map_count
sysctl -w vm.max_map_count=262144
```

### 3. Docker安装Gitlib及使用

教程地址：

https://www.jianshu.com/p/080a962c35b6

``` shell
# 下载镜像
$ git pull gitlab/gitlab-ce

#  
$ docker run -d  -p 443:443 -p 80:80 -p 222:22 --name gitlab --restart always -v /home/gitlab/config:/etc/gitlab -v /home/gitlab/logs:/var/log/gitlab -v /home/gitlab/data:/var/opt/gitlab gitlab/gitlab-ce
```

### 3. Docker安装Nginx并运行

#### 3.1 安装Nginx并创建容器

``` shell
# 1. 拉取镜像
$ docker pull nginx:latest

# 2. 查看镜像
$ docker images

# 3. 运行容器
$ docker run --name nginx-test -p 80:80 -d nginx
```

#### 3.2 配置Nginx

```shell
# 1. 进入docker容器
$ sudo docker exec -it {container_id} /bin/bash

# 2. 创建Nginx的本地数据卷
$ mkdir -p /local/docker/nginx/conf /local/docker/nginx/html /local/docker/nginx/logs /local/docker/nginx/conf.d

# 3. 创建Nginx配置文件
$ cat > /local/docker/nginx/conf/nginx.conf << EOF
user nginx;
worker_processes 1;

error_log /var/log/nginx/error.log warn;
pid    /var/run/nginx.pid;


events {
  worker_connections 1024;
}


http {
  include    /etc/nginx/mime.types;
  default_type application/octet-stream;

  log_format main '$remote_addr - $remote_user [$time_local] "$request" '
           '$status $body_bytes_sent "$http_referer" '
           '"$http_user_agent" "$http_x_forwarded_for"';

  access_log /var/log/nginx/access.log main;

  sendfile    on;
  #tcp_nopush   on;

  keepalive_timeout 65;

  #gzip on;

  include /etc/nginx/conf.d/*.conf;
}
EOF

# 4. 创建Nginx配置文件
$ cat > /local/docker/nginx/conf.d/default.conf << EOF
server { 
  listen    80; 
  server_name localhost; 
 
  #charset koi8-r; 
  #access_log /var/log/nginx/log/host.access.log main; 
 
  location / { 
    # root  /opt/nginx/html; 
    root   /usr/share/nginx/html;  
    index index.html index.htm; 
    autoindex on; 
    #try_files $uri /index/index/page.html; 
    #try_files $uri /index/map/page.html; 
  } 
 
  #error_page 404       /404.html; 
 
  # redirect server error pages to the static page /50x.html 
  # 
  error_page  500 502 503 504 /50x.html; 
  location = /50x.html { 
    root  /usr/share/nginx/html; 
  } 
 
  # proxy the PHP scripts to Apache listening on 127.0.0.1:80 
  # 
  #location ~ \.php$ { 
  #  proxy_pass  http://127.0.0.1; 
  #} 
 
  # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000 
  # 
  #location ~ \.php$ { 
  #  root      html; 
  #  fastcgi_pass  127.0.0.1:9000; 
  #  fastcgi_index index.php; 
  #  fastcgi_param SCRIPT_FILENAME /scripts$fastcgi_script_name; 
  #  include    fastcgi_params; 
  #} 
 
  # deny access to .htaccess files, if Apache's document root 
  # concurs with nginx's one 
  # 
  #location ~ /\.ht { 
  #  deny all; 
  #} 
}
EOF
```

#### 3.3 创建测试页面

```shell
# 1. 创建Nginx欢迎页面
$ cat > /local/docker/nginx/html/index.html << EOF
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
    <title>系统时间</title>
</head>
<body>
<h1>Nginx系统测试</h1>
<div id="datetime">
    <script>
        setInterval("document.getElementById('datetime').innerHTML=new Date().toLocaleString();", 1000);
    </script>
</div>
</body>
EOF
```

#### 3.4 创建包含数据卷的Nginx容器

```shell
# 创建容器
$ sudo docker run --name nginx80 \
-d -p 80:80 \
-v /local/docker/nginx/html:/usr/share/nginx/html \
-v /local/docker/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
-v /local/docker/nginx/logs:/var/log/nginx \
-v /local/docker/nginx/conf.d:/etc/nginx/conf.d \
-d nginx:latest

# 重启容器
$ docker restart {$container_id}	
```

#### 3.5 开放防火墙端口

```shell
# 开放防火墙端口
$ firewall-cmd --zone=public --add-port=80/tcp --permanent
$ firewall-cmd --reload
$ firewall-cmd --list-ports
```

### 4. Docker安装Nacos

#### 4.1 安装过程

```shell
# 拉取镜像
$ docker pull nacos/nacos-server

# 创建容器
$ docker run --env MODE=standalone --restart=always  --name nacos -d -p 8848:8848 nacos/nacos-server

# 运行容器
$ docker container ls ## 查询容器ID
$ docker start ${容器ID/容器名}

# 开放防火墙商品
$ firewall-cmd --zone=public --add-port=8848/tcp --permanent
$ firewall-cmd --reload
$ firewall-cmd --list-ports
```

#### 4.2 Nacos客户端界面

nacos客户端登录地址：[http://localhost:8848/nacos/index.html](http://localhost:8848/nacos/index.html)  默认账号密码是`nacos/nacos`

#### 4.3 Nacos使用MySQL存储配置信息

##### 4.3.1 在MySQL中为Nacos创建数据库

```mysql
CREATE DATABASE nacos_config;

USE nacos_config;

/*
 * Copyright 1999-2018 Alibaba Group Holding Ltd.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_info   */
/******************************************/
CREATE TABLE `config_info` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` varchar(255) NOT NULL COMMENT 'data_id',
  `group_id` varchar(255) DEFAULT NULL,
  `content` longtext NOT NULL COMMENT 'content',
  `md5` varchar(32) DEFAULT NULL COMMENT 'md5',
  `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
  `src_user` text COMMENT 'source user',
  `src_ip` varchar(50) DEFAULT NULL COMMENT 'source ip',
  `app_name` varchar(128) DEFAULT NULL,
  `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',
  `c_desc` varchar(256) DEFAULT NULL,
  `c_use` varchar(64) DEFAULT NULL,
  `effect` varchar(64) DEFAULT NULL,
  `type` varchar(64) DEFAULT NULL,
  `c_schema` text,
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfo_datagrouptenant` (`data_id`,`group_id`,`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info';

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_info_aggr   */
/******************************************/
CREATE TABLE `config_info_aggr` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` varchar(255) NOT NULL COMMENT 'data_id',
  `group_id` varchar(255) NOT NULL COMMENT 'group_id',
  `datum_id` varchar(255) NOT NULL COMMENT 'datum_id',
  `content` longtext NOT NULL COMMENT '内容',
  `gmt_modified` datetime NOT NULL COMMENT '修改时间',
  `app_name` varchar(128) DEFAULT NULL,
  `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfoaggr_datagrouptenantdatum` (`data_id`,`group_id`,`tenant_id`,`datum_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='增加租户字段';


/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_info_beta   */
/******************************************/
CREATE TABLE `config_info_beta` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` varchar(255) NOT NULL COMMENT 'data_id',
  `group_id` varchar(128) NOT NULL COMMENT 'group_id',
  `app_name` varchar(128) DEFAULT NULL COMMENT 'app_name',
  `content` longtext NOT NULL COMMENT 'content',
  `beta_ips` varchar(1024) DEFAULT NULL COMMENT 'betaIps',
  `md5` varchar(32) DEFAULT NULL COMMENT 'md5',
  `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
  `src_user` text COMMENT 'source user',
  `src_ip` varchar(50) DEFAULT NULL COMMENT 'source ip',
  `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfobeta_datagrouptenant` (`data_id`,`group_id`,`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info_beta';

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_info_tag   */
/******************************************/
CREATE TABLE `config_info_tag` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` varchar(255) NOT NULL COMMENT 'data_id',
  `group_id` varchar(128) NOT NULL COMMENT 'group_id',
  `tenant_id` varchar(128) DEFAULT '' COMMENT 'tenant_id',
  `tag_id` varchar(128) NOT NULL COMMENT 'tag_id',
  `app_name` varchar(128) DEFAULT NULL COMMENT 'app_name',
  `content` longtext NOT NULL COMMENT 'content',
  `md5` varchar(32) DEFAULT NULL COMMENT 'md5',
  `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
  `src_user` text COMMENT 'source user',
  `src_ip` varchar(50) DEFAULT NULL COMMENT 'source ip',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfotag_datagrouptenanttag` (`data_id`,`group_id`,`tenant_id`,`tag_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info_tag';

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_tags_relation   */
/******************************************/
CREATE TABLE `config_tags_relation` (
  `id` bigint(20) NOT NULL COMMENT 'id',
  `tag_name` varchar(128) NOT NULL COMMENT 'tag_name',
  `tag_type` varchar(64) DEFAULT NULL COMMENT 'tag_type',
  `data_id` varchar(255) NOT NULL COMMENT 'data_id',
  `group_id` varchar(128) NOT NULL COMMENT 'group_id',
  `tenant_id` varchar(128) DEFAULT '' COMMENT 'tenant_id',
  `nid` bigint(20) NOT NULL AUTO_INCREMENT,
  PRIMARY KEY (`nid`),
  UNIQUE KEY `uk_configtagrelation_configidtag` (`id`,`tag_name`,`tag_type`),
  KEY `idx_tenant_id` (`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_tag_relation';

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = group_capacity   */
/******************************************/
CREATE TABLE `group_capacity` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `group_id` varchar(128) NOT NULL DEFAULT '' COMMENT 'Group ID，空字符表示整个集群',
  `quota` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '配额，0表示使用默认值',
  `usage` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '使用量',
  `max_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个配置大小上限，单位为字节，0表示使用默认值',
  `max_aggr_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '聚合子配置最大个数，，0表示使用默认值',
  `max_aggr_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个聚合数据的子配置大小上限，单位为字节，0表示使用默认值',
  `max_history_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '最大变更历史数量',
  `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_group_id` (`group_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='集群、各Group容量信息表';

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = his_config_info   */
/******************************************/
CREATE TABLE `his_config_info` (
  `id` bigint(64) unsigned NOT NULL,
  `nid` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
  `data_id` varchar(255) NOT NULL,
  `group_id` varchar(128) NOT NULL,
  `app_name` varchar(128) DEFAULT NULL COMMENT 'app_name',
  `content` longtext NOT NULL,
  `md5` varchar(32) DEFAULT NULL,
  `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `src_user` text,
  `src_ip` varchar(50) DEFAULT NULL,
  `op_type` char(10) DEFAULT NULL,
  `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',
  PRIMARY KEY (`nid`),
  KEY `idx_gmt_create` (`gmt_create`),
  KEY `idx_gmt_modified` (`gmt_modified`),
  KEY `idx_did` (`data_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='多租户改造';


/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = tenant_capacity   */
/******************************************/
CREATE TABLE `tenant_capacity` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `tenant_id` varchar(128) NOT NULL DEFAULT '' COMMENT 'Tenant ID',
  `quota` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '配额，0表示使用默认值',
  `usage` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '使用量',
  `max_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个配置大小上限，单位为字节，0表示使用默认值',
  `max_aggr_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '聚合子配置最大个数',
  `max_aggr_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个聚合数据的子配置大小上限，单位为字节，0表示使用默认值',
  `max_history_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '最大变更历史数量',
  `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_tenant_id` (`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='租户容量信息表';


CREATE TABLE `tenant_info` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `kp` varchar(128) NOT NULL COMMENT 'kp',
  `tenant_id` varchar(128) default '' COMMENT 'tenant_id',
  `tenant_name` varchar(128) default '' COMMENT 'tenant_name',
  `tenant_desc` varchar(256) DEFAULT NULL COMMENT 'tenant_desc',
  `create_source` varchar(32) DEFAULT NULL COMMENT 'create_source',
  `gmt_create` bigint(20) NOT NULL COMMENT '创建时间',
  `gmt_modified` bigint(20) NOT NULL COMMENT '修改时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_tenant_info_kptenantid` (`kp`,`tenant_id`),
  KEY `idx_tenant_id` (`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='tenant_info';

CREATE TABLE `users` (
	`username` varchar(50) NOT NULL PRIMARY KEY,
	`password` varchar(500) NOT NULL,
	`enabled` boolean NOT NULL
);

CREATE TABLE `roles` (
	`username` varchar(50) NOT NULL,
	`role` varchar(50) NOT NULL,
	UNIQUE INDEX `idx_user_role` (`username` ASC, `role` ASC) USING BTREE
);

CREATE TABLE `permissions` (
    `role` varchar(50) NOT NULL,
    `resource` varchar(255) NOT NULL,
    `action` varchar(8) NOT NULL,
    UNIQUE INDEX `uk_role_permission` (`role`,`resource`,`action`) USING BTREE
);

INSERT INTO users (username, password, enabled) VALUES ('nacos', '$2a$10$EuWPZHzz32dJN7jexM34MOeYirDdFAZm2kuWj7VEOJhhZkDrxfvUu', TRUE);

INSERT INTO roles (username, role) VALUES ('nacos', 'ROLE_ADMIN');
```

##### 4.3.2 创建Nacos容器

``` shell
# 创建Nacos容器数据卷本地目录
$ mkdir -p /local/docker/nacos/logs /local/docker/nacos/init.d /local/docker/nacos/data

# 创建Nacos容器
$ docker run --env MODE=standalone --env SPRING_DATASOURCE_PLATFORM=mysql --env MYSQL_MASTER_SERVICE_DB_NAME=nacos_config --env MYSQL_MASTER_SERVICE_HOST=192.168.92.128 --env MYSQL_MASTER_SERVICE_USER=root --env MYSQL_MASTER_SERVICE_PASSWORD=123456 --env MYSQL_SLAVE_SERVICE_HOST=192.168.92.128 --env MYSQL_MASTER_SERVICE_DB_NAME=nacos_config --name nacos -d -p 8848:8848 nacos/nacos-server
```

##### 4.3.3 从配置文件中提取环境变量参数

这个配置文件需要在当前的执行路径下，官方推荐以`.list`为扩展名，也可以采用`.properties`为扩展名，执行命令如下：

``` shell
$ docker run --name nacos \
	-p 8848:8848 \
	--privileged=true \
	--restart=always \
	-e JVM_XMS=256m \
	-e JVM_XMX=256m \
	-e MODE=standalone \
	-e PREFER_HOST_MODE=hostname \
	-v /local/docker/nacos/logs:/home/nacos/logs \
	-v /local/docker/nacos/init.d/custom.properties:/home/nacos/init.d/custom.properties \
	-d nacos/nacos-server
```

对应的env.list内容如下：

``` shell
MODE=standalone
SPRING_DATASOURCE_PLATFORM=mysql
MYSQL_MASTER_SERVICE_DB_NAME=nacos_config
MYSQL_MASTER_SERVICE_HOST=192.168.92.130
MYSQL_MASTER_SERVICE_USER=root
MYSQL_MASTER_SERVICE_PASSWORD=123456
MYSQL_SLAVE_SERVICE_HOST=192.168.92.130
MYSQL_MASTER_SERVICE_DB_NAME=nacos_config
```

对应的custom.properties文件内容如下：

```properties
server.contextPath=/nacos
server.servlet.contextPath=/nacos
server.port=8848
spring.datasource.platform=mysql

db.num=1
# 这里要对应ip，以及对应的数据库
db.url.0=jdbc:mysql://192.168.92.128:3306/nacos_config?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
db.user=root
db.password=123456

nacos.cmdb.dumpTaskInterval=3600
nacos.cmdb.eventTaskInterval=10
nacos.cmdb.labelTaskInterval=300
nacos.cmdb.loadDataAtStart=false

management.metrics.export.elastic.enabled=false
management.metrics.export.influx.enabled=false
server.tomcat.accesslog.enabled=true
server.tomcat.accesslog.pattern=%h %l %u %t "%r" %s %b %D %{User-Agent}i
nacos.security.ignore.urls=/,/**/*.css,/**/*.js,/**/*.html,/**/*.map,/**/*.svg,/**/*.png,/**/*.ico,/console-fe/public/**,/v1/auth/login,/v1/console/health/**,/v1/cs/**,/v1/ns/**,/v1/cmdb/**,/actuator/**,/v1/console/server/**
nacos.naming.distro.taskDispatchThreadCount=1
nacos.naming.distro.taskDispatchPeriod=200
nacos.naming.distro.batchSyncKeyCount=1000
nacos.naming.distro.initDataRatio=0.9
nacos.naming.distro.syncRetryDelay=5000
nacos.naming.data.warmup=true
nacos.naming.expireInstance=true
```

##### 4.3.4 开放防火墙端口

```shell
# 开放端口：
$ systemctl status firewalld
$ firewall-cmd --zone=public --add-port=8848/tcp --permanent
$ firewall-cmd --reload
$ firewall-cmd --list-ports
# 关闭防火墙：
$ sudo systemctl stop firewalld
```

##### 4.3.5 另一种方法

```shell
# 新建logs目录
$ mkdir -p /local/docker/nacos/logs/
$ mkdir -p /local/docker/nacos/init.d/

$ cat > /local/docker/nacos/init.d/env.properties << EOF  #生成配置文件
server.contextPath=/local/docker/nacos
server.servlet.contextPath=/local/docker/nacos
server.port=8848

spring.datasource.platform=mysql

db.num=1
db.url.0=jdbc:mysql://192.168.92.130:3306/nacos_config?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
db.user=root
db.password=123456


nacos.cmdb.dumpTaskInterval=3600
nacos.cmdb.eventTaskInterval=10
nacos.cmdb.labelTaskInterval=300
nacos.cmdb.loadDataAtStart=false

management.metrics.export.elastic.enabled=false

management.metrics.export.influx.enabled=false


server.tomcat.accesslog.enabled=true
server.tomcat.accesslog.pattern=%h %l %u %t "%r" %s %b %D %{User-Agent}i


nacos.security.ignore.urls=/,/**/*.css,/**/*.js,/**/*.html,/**/*.map,/**/*.svg,/**/*.png,/**/*.ico,/console-fe/public/**,/v1/auth/login,/v1/console/health/**,/v1/cs/**,/v1/ns/**,/v1/cmdb/**,/actuator/**,/v1/console/server/**
nacos.naming.distro.taskDispatchThreadCount=1
nacos.naming.distro.taskDispatchPeriod=200
nacos.naming.distro.batchSyncKeyCount=1000
nacos.naming.distro.initDataRatio=0.9
nacos.naming.distro.syncRetryDelay=5000
nacos.naming.data.warmup=true
nacos.naming.expireInstance=true
EOF

# 启动容器
docker  run \
--name nacos -d \
-p 8848:8848 \
--privileged=true \
--restart=always \
-e JVM_XMS=256m \
-e JVM_XMX=256m \
-e MODE=standalone \
-e PREFER_HOST_MODE=hostname \
-v /local/docker/nacos/logs:/home/nacos/logs \
-v /local/docker/nacos/init.d/env.properties:/home/nacos/init.d/custom.properties \
nacos/nacos-server
```

### 5. Docker安装Redis

#### 5.1 安装过程

``` shell
# 拉取镜像
$ docker pull redis:latest

# 创建容器
$ docker run -itd --name redis -p 6379:6379 redis

# 开放防火墙端口
$ systemctl status firewalld
$ firewall-cmd --zone=public --add-port=6379/tcp --permanent
$ firewall-cmd --reload
$ firewall-cmd --list-ports
```

#### 5.2 进入容器，测试Redis

``` shell
$ docker exec -it redis /bin/bash

root@b9fb50ded58e:/data# redis-cli

127.0.0.1:6379> set country China
OK
127.0.0.1:6379> get country
"China"
```



### 6. Docker安装RabbitMQ

#### 6.1 拉取镜像与创建容器

```shell
# 拉取镜像
$ docker pull rabbitmq
# 创建容器
$ docker run -d --name rabbitmq -p 5672:5672 -p 15672:15672 -v `pwd`/data:/var/lib/rabbitmq --hostname myRabbit -e RABBITMQ_DEFAULT_VHOST=my_vhost  -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin rabbitmq
# 另一创建容器的方法
$ docker run -di --name rabbitmq -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin -p 15672:15672 -p 5672:5672 -p 25672:25672 -p 61613:61613 -p 1883:1883 rabbitmq:management
# 查看正在运行的容器
$ docker ps -a
# 开放防火墙端口
$ systemctl status firewalld
$ firewall-cmd --zone=public --add-port=1/tcp --permanent
$ firewall-cmd --reload
$ firewall-cmd --list-ports

```

说明：

-d 后台运行容器；

--name 指定容器名；

-p 指定服务运行的端口（5672：应用访问端口；15672：控制台Web端口号）；

-v 映射目录或文件；

--hostname  主机名（RabbitMQ的一个重要注意事项是它根据所谓的 “节点名称” 存储数据，默认为主机名）；

-e 指定环境变量；（RABBITMQ_DEFAULT_VHOST：默认虚拟机名；RABBITMQ_DEFAULT_USER：默认的用户名；RABBITMQ_DEFAULT_PASS：默认用户名的密码）

#### 6.2 查看管理端

使用浏览器打开web管理端：[http://Server-IP:15672](http://Server-IP:15672)

