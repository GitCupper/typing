# Docker常用操作

## 一、Docker相关



### 1.0 Docker安装

``` shell
# 卸载旧版本
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
                  
                  
```



### 1.1 yum 相关

```shell
$ yum install epel-release -y
$ yum list docker --show-duplicates
$ yum install -y yum-utils
$ yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# 安装D
$ sudo yum install docker-ce docker-ce-cli containerd.io
```

### 1.2 配置Docker开机自启动

```shell
$ systemctl enable docker
$ systemctl start docker
```

配置docker的daemon.json

```shell
cat > /etc/docker/daemon.json << EOF
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

mkdir -p /data/docker

systemctl restart docker
```



### 1.3 Docker常用命令

```shell
$ docker info

$ docker inspect ${容器名/容器ID} # 查看容器信息

$ docker logs ${容器名/容器ID} # 查看容器日志
```

## 2 Docker运行常用软件

### 2.1 Docker安装MySQL与使用

#### 2.1.1 拉取官方镜像

``` shell
docker pull mysql:5.7    # 拉取mysql 5.7
docker pull mysql        # 拉取最新版mysql
```

#### 2.1.2 检查是否拉取成功

``` shell
sudo docker images
```

#### 2.1.3 不建立数据库文件目录映射

```shell
sudo docker run -p 3306:3306 --name mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql
```

- `-name`：容器名，此外命名为：mysql
- `-e`：配置信息，此处配置mysql的root用户的登录密码
- `-p`：端口映射，此处映射主机3306端口到容器的3306端口
- `-d`：后台运行容器，保证在退出终端后容器继续运行

#### 2.1.4 建立数据库文件目录映射

添加mysql的配置文件

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

```shell
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

- `-v`：主机和容器的目录映射关系，“:”前为主机目录，之后为容器目录

#### 2.1.5 检查容器是否正确运行

```shell
docker container ls # 查看现有容器

docker ps -a # 查看容器状态
docker start ${容器ID} # 启动一个已经停止的容器
docker stop ${容器ID} # 停止一个已经启动的容器
```

- 可以看到容器ID，容器的源镜像，启动命令，创建时间，状态，端口映射信息，容器名字

#### 2.1.6 进入docker本地连接MySQL客户端

```shell
sudo docker exec -it mysql /bin/bash
mysql -uroot -p123456
```

#### 2.1.7 远程访问时应注意的问题

由于已经将容器的`3306`端口映射到了主机的`3306`端口，所以只需要访问主机的`3306`端口即可。

#### 2.1.8 开放主机防火墙

```shell
# 开放端口：
$ systemctl status firewalld
$ firewall-cmd --zone=public --add-port=3306/tcp --permanent
$ firewall-cmd --reload
$ firewall-cmd --list-all
$ firewall-cmd --list-ports
# 关闭防火墙：
$ sudo systemctl stop firewalld
```

#### 2.1.9 进入docker本地客户端设置远程访问账号

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

#### 2.1.10 修改MySQL的时区设置

MySQL安装后，默认的时区设置为`SYSTEM`，并没有指向中国当地的时区，因此需要进行设置：

``` shell
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

#### 2.1.11 修改MySQL8的密码验证策略

在新版的MySQL8中，`caching_sha2_password`是默认的身份验证插件，而不是`mysql_native_password`，因此如果要使用旧验证插件登录的用户，需要使用新的验证插件，或是将默认的验证插件修改为`mysql_native_password`。

``` shel
# 本地
# 修改加密规则（非必须）
ALTER USER 'root'@'localhost' IDENTIFIED BY '123456' PASSWORD EXPIRE NEVER; 
# 更新用户的密码
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '123456'; 
# 刷新权限
FLUSH PRIVILEGES;
# 重置密码（==非必须==）
ALTER USER 'root'@'localhost' IDENTIFIED BY '123456';

# 远程
# 修改加密规则（非必须）
ALTER USER 'root'@'%' IDENTIFIED BY '123456' PASSWORD EXPIRE NEVER; 
# 更新用户的密码
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
# 刷新权限
FLUSH PRIVILEGES;
# 重置密码（==非必须==）
ALTER USER 'root'@'%' IDENTIFIED BY '123456';

# 查看修改结果
SELECT Host, User, plugin from mysql.user;
```



## 三、Docker下安装Elastic Search

### 3.1 下载镜像

``` shell
$ docker pull registry.docker-cn.com/library/elasticsearch

$ docker pull docker.elastic.co/elasticsearch/elasticsearch:8.0.0
docker run --name es01 --net elastic -p 9200:9200 -p 9300:9300 -it docker.elastic.co/elasticsearch/elasticsearch:8.0.0

###########################################
$ docker pull elasticsearch:7.17.0
$ docker run -e ES_JAVA_OPTS="-Xms512m -Xmx512m" -d -p 9200:9200 -p 9300:9300 --name es01 elasticsearch:7.17.0

# 开放防火墙
# 开放端口：
$ systemctl status firewalld
$ firewall-cmd --zone=public --add-port=9200/tcp --permanent
$ firewall-cmd --zone=public --add-port=9300/tcp --permanent
$ firewall-cmd --reload

$ firewall-cmd --list-ports


```

### 3.2 Elastic Search在Docker下的安装

#### 3.2.1 设置max_map_count



``` shell
cat /proc/sys/vm/max_map_count
sysctl -w vm.max_map_count=262144
```

#### 3.2.2 下载镜像并运行

``` she
#拉取镜像
docker pull elasticsearch:7.17.0

#启动镜像
docker run --name elasticsearch -d -e ES_JAVA_OPTS="-Xms512m -Xmx512m" -e "discovery.type=single-node" -p 9200:9200 -p 9300:9300 elasticsearch:7.17.0

```

## 四、Gitlib在Docker下的安装与使用

教程地址：

https://www.jianshu.com/p/080a962c35b6

``` shell
# 下载镜像
$ git pull gitlab/gitlab-ce

#  
$ docker run -d  -p 443:443 -p 80:80 -p 222:22 --name gitlab --restart always -v /home/gitlab/config:/etc/gitlab -v /home/gitlab/logs:/var/log/gitlab -v /home/gitlab/data:/var/opt/gitlab gitlab/gitlab-ce
```

