# Docker的离线安装与使用



## 一、安装Docker

### 1、下载Docker离线安装文件

[https://download.docker.com/linux/static/stable/x86_64/](https://download.docker.com/linux/static/stable/x86_64/)



### 2、上传Docker安装文件

将docker-18.06.3-ce.tgz文件上传到centos7-linux系统上，用ftp工具上传即可

### 3、解压

```shell
[root@localhost java]# tar -zxvf docker-18.06.3-ce.tgz
```

### 4、复制文件

```shell
[root@localhost java]# cp docker/* /usr/bin/
```

### 5、创建**docker.service**文件

进入**/etc/systemd/system/**目录,并创建**docker.service**文件

```shell
[root@localhost java]# cd /etc/systemd/system/
[root@localhost system]# touch docker.service
```

或者

``` shell
[root@localhost ~]# vim /etc/systemd/system/docker.service
```



### 6、修改**docker.service**文件内容

``` shell
vim docker.service
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

### 7、给**docker.service**文件添加执行权限

```shell
[root@localhost system]# chmod 777 /etc/systemd/system/docker.service
```

### 8、重新加载配置文件

每次有修改docker.service文件时都要重新加载。

``` shell
[root@localhost system]# systemctl daemon-reload
```

### 9、启动Docker

``` shell
[root@localhost system]# systemctl start docker
```

### 10、设置开机启动

```shell
[root@localhost system]# systemctl enable docker.service
```

### 11、查看docker状态

```shell
[root@localhost system]# systemctl status docker
```

### 12、配置镜像加速器

默认是到国外拉取镜像速度慢,可以配置国内的镜像如：阿里、网易等等。下面配置一下网易的镜像加速器。打开docker的配置文件: /etc/docker/daemon.json文件：

```shell
[root@localhost docker]# vim /etc/docker/daemon.json

# 内容如下
{"registry-mirrors": ["http://hub-mirror.c.163.com"]}

## 或者使用如下命令：
cat > /etc/docker/daemon.json << EOF
{"registry-mirrors": ["http://hub-mirror.c.163.com"]}
EOF
```

### 13、重启Docker

```shell
[root@localhost docker]# service docker restart
```

### 14、Docker容器的启动、重启与停止

``` shell
# 01. 启动容器，命令：
docker start 容器ID或容器名

# 02. 重启容器，命令：
docker restart 容器ID或容器名

# 03. 停止容器，命令：
docker stop 容器ID或容器名

# 04. 强制停止容器，命令：
docker kill 容器ID或容器名

# 05. 列表显示所有容器
docker container ls

# 06. 显示所有运行的容器
docker ps -a
```

### 15、Docker安装Nginx并运行

``` shell
# 1. 拉取镜像
docker pull nginx:latest

# 2. 查看镜像
docker images

# 3. 运行容器
docker run --name nginx-test -p 80:80 -d nginx

# 4. 进入docker容器
sudo docker exec -it {container_id} /bin/bash

# 5. 创建Nginx的本地数据卷
mkdir -p /local/docker/nginx/conf /local/docker/nginx/html /local/docker/nginx/logs /local/docker/nginx/conf.d

# 6. 创建Nginx配置文件
cat > /local/docker/nginx/conf/nginx.conf << EOF
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

# 7. 创建Nginx配置文件
cat > /local/docker/nginx/conf.d/default.conf << EOF
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

# 8. 创建Nginx欢迎页面
cat > /local/docker/nginx/html/index.html << EOF
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

sudo docker run --name nginx80 \
-d -p 80:80 \
-v /local/docker/nginx/html:/usr/share/nginx/html \
-v /local/docker/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
-v /local/docker/nginx/logs:/var/log/nginx \
-v /local/docker/nginx/conf.d:/etc/nginx/conf.d \
-d nginx:latest
	
docker restart {$container_id}	

# 开放防火墙端口
$ firewall-cmd --zone=public --add-port=80/tcp --permanent
$ firewall-cmd --reload
$ firewall-cmd --list-ports
```



## 二、在Docker下安装MySQL

### 2.1 拉取MySQL镜像

``` shell
# 拉取mysql5.7的镜像
docker pull mysql:5.7

# 创建本地文件夹
mkdir -p /local/docker/mysql/conf /local/docker/mysql/logs /local/docker/mysql/data

# 启动mysql镜像
sudo docker run -p 3306:3306 --name mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7

# 配置文件存储的启动方式
sudo docker run -p 3306:3306 --name mysql \
-v /local/docker/mysql/conf:/etc/mysql \
-v /local/docker/mysql/logs:/var/log/mysql \
-v /local/docker/mysql/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=123456 \
--restart=always \
-d mysql:5.7

# 查看Docker容器
docker container ls

# 启动现有的Docker容器
docker start ###(容器ID号)

# 停止现有的Docker容器
docker stop ###(容器ID号)

# 重启现有的Docker容器
docker restart ###(容器ID号)

# 导出镜像为压缩包
docker image save -o nacos.tar nacos/nacos-server

# 从压缩包加载镜像
docker load --input nacos.tar

# 进入容器中的MySQL
docker exec -it ${###}(容器号) /bin/bash
mysql -uroot -p
(密码：1)


```

### 2.2 开放防火墙端口

```shell
# 开放端口：
$ systemctl status firewalld
$ firewall-cmd --zone=public --add-port=3306/tcp --permanent
$ firewall-cmd --reload
$ firewall-cmd --list-ports
# 关闭防火墙：
$ sudo systemctl stop firewalld
```



## 三、在Docker下安装Nacos

### 3.1 拉取镜像

``` she
docker pull nacos/nacos-server
```

### 3.2 启动镜像

#### 3.2.1 初次启动

```shell
docker run --env MODE=standalone --name nacos -d -p 8848:8848 nacos/nacos-server
```

#### 3.2.2 再次启动

``` shell
docker ps -a # 查看有哪些容器
docker container ls # 查看正在运行的容器
docker start ${容器号} # 运行指定容器
```



nacos登陆地址：[http://localhost:8848/nacos/index.html](http://localhost:8848/nacos/index.html)  默认账号密码是`nacos/nacos`

### 3.3 nacos使用mysql作为配置化

#### 3.3.1 在mysql中为nacos创建数据库

```sql
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

```shell
# 创建nacos in docker的本地数据卷
mkdir -p /local/docker/nacos/logs /local/docker/nacos/init.d /local/docker/nacos/data
```



#### 3.3.2 直接拼接参数的启动方式

``` shell
docker run --env MODE=standalone --env SPRING_DATASOURCE_PLATFORM=mysql --env MYSQL_MASTER_SERVICE_DB_NAME=nacos_config --env MYSQL_MASTER_SERVICE_HOST=192.168.92.128 --env MYSQL_MASTER_SERVICE_USER=root --env MYSQL_MASTER_SERVICE_PASSWORD=123456 --env MYSQL_SLAVE_SERVICE_HOST=192.168.92.128 --env MYSQL_MASTER_SERVICE_DB_NAME=nacos_config --name nacos -d -p 8848:8848 nacos/nacos-server
```

#### 3.3.3 从文中提取环境变量参数

这个配置文件需要在当前的执行路径下，官方推荐以`.list`为扩展名，也可以采用`.properties`为扩展名，执行命令如下：

``` shell
docker run --name nacos \
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

对应的env.list如下：

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

``` properties
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



#### 3.3.4 开放防火墙端口

```shell
# 开放端口：
$ systemctl status firewalld
$ firewall-cmd --zone=public --add-port=8848/tcp --permanent
$ firewall-cmd --reload
$ firewall-cmd --list-ports
# 关闭防火墙：
$ sudo systemctl stop firewalld
```

#### 3.3.5 另一种方法

```shell
mkdir -p /local/docker/nacos/logs/                      #新建logs目录
mkdir -p /local/docker/nacos/init.d/

cat > /local/docker/nacos/init.d/env.properties << EOF  #生成配置文件
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

## 四、Docker下安装与使用Redis

### 4.1 Redis的安装

下载最新的Redis镜像

``` shell
$ docker pull redis:latest
```

### 4.2 运行Redis容器

``` shell
$ docker run -itd --name redis -p 6379:6379 redis

$ docker ps -a # 查看容器信息
```

### 4.3 进入容器，使用Redis

``` shell
$ docker exec -it redis /bin/bash

root@b9fb50ded58e:/data# redis-cli

127.0.0.1:6379> set country China
OK
127.0.0.1:6379> get country
"China"
```

