# CentOS7 安装Docker

## 手动安装脚本

### 1、卸载原有的旧版本

```shell
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

### 2、安装Docker Engine-Community

#### 设置仓库

安装所需的软件包。yum-utils 提供了 yum-config-manager ，并且 device mapper 存储驱动程序需要 device-mapper-persistent-data 和 lvm2。

```shell
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

#### 安装Docker

```shell
$ sudo yum install docker-ce docker-ce-cli containerd.io
```

#### 启动Docker

```shell
$ sudo systemctl start docker
```

#### 测试Docker

```shell
$ sudo docker run hello-world
```

#### 设置Docker开机自启动

```shell
$ systemctl  enable docker.service
```

#### 删除安装包

```shell
$ yum remove docker-ce
```

#### 删除镜像、容器、配置文件等内容

```shell
$ rm -rf /var/lib/docker
```

