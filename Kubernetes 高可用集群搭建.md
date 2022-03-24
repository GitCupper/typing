# Kubernetes 高可用集群搭建

使用Kubernetes搭建服务集群，其中的master节点是必须的，因其是整个集群的核心，所以一旦这个节点宕机，则会造成整个集群瘫痪，因此单master节点的集群不能应用于生产环境。

为了解决集群的高可用问题，就需要部署一个以上的master节点，官方提供了2种高可用的部署方案，一种是外部ETCD的方式，好部署一个单独的ETCD集群，另一种就是混合部署，ETCD和apiserver一起部署。在此，我们采用第二种方案进行部署，这个方案部署简单，节约服务器。

![img](https://upload-images.jianshu.io/upload_images/14339262-c1675a03d58f8eb7.png?imageMogr2/auto-orient/strip|imageView2/2/w/800/format/webp)

第一种方案

![img](https://upload-images.jianshu.io/upload_images/14339262-a5f110e82eb66858.png?imageMogr2/auto-orient/strip|imageView2/2/w/800/format/webp)

第二种方案

## 一、服务器规划

首先准备几台服务器，计划部署3台master，7台node。服务器名称、IP地址规划如下：

| 服务器名称    | 节点类型 | IP地址         | 节点名称    |
| ------------- | -------- | -------------- | ----------- |
| VIP(虚IP)     |          | 192.168.126.20 |             |
| CentOS7x64_01 | master   | 192.168.126.21 | K8sMaster01 |
| CentOS7x64_02 | master   | 192.168.126.22 | K8sMaster02 |
| CentOS7x64_03 | master   | 192.168.126.23 | K8sMaster03 |
| CentOS7x64_04 | node     | 192.168.126.24 | K8sNode01   |
| CentOS7x64_05 | node     | 192.168.126.25 | K8sNode02   |
| CentOS7x64_06 | node     | 192.168.126.26 | K8sNode03   |
| CentOS7x64_07 | node     | 192.168.126.27 | K8sNode04   |
| CentOS7x64_08 | node     | 192.168.126.28 | K8sNode05   |
| CentOS7x64_09 | node     | 192.168.126.29 | K8sNode06   |
| CentOS7x64_10 | node     | 192.168.126.30 | K8sNode07   |

## 二、服务器设置

### 2.1 设置服务器IP地址

``` shell
# 编辑如下网络配备文件
$ vim /etc/sysconfig/network-scripts/ifcfg-ens33

# 添加如下配置

```

### 2.2 设置hostname

```shell
$ hostnamectl set-hostname <hostname>
```

### 2.3 修改hosts文件

``` shell
vim /etc/hosts
## 添加如下内容
192.168.126.20 k8s-vip
192.168.126.21 k8s-master01
192.168.126.22 k8s-master02
192.168.126.23 k8s-master03
192.168.126.24 k8s-node01
192.168.126.25 k8s-node02
192.168.126.26 k8s-node03
192.168.126.27 k8s-node04
192.168.126.28 k8s-node05
192.168.126.29 k8s-node06
192.168.126.30 k8s-node07
```

### 2.4 关闭防火墙、SELinux

``` shell
## 配置内核参数，将桥接的IPv4流量传递到iptables的转发链
cat << EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.birdge-nf-call-iptables = 1
EOF

## 手动加载配置文件
sysctl --system

## 关闭防火墙
systemctl stop firewalld
systemctl disable firewalld

## 将SELinux设置为permissive模式（相当于将其禁用）
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

## 关闭交换空间
swapoff -a
sed -i 's/.*swap.*/#&/' /etc/fstab

## ip转发
echo '1' > /proc/sys/net/ipv4/ip_forrward
```

## 三、安装keepalived和haproxy

### 3.1 安装keepalived和haproxy

``` shell
yum install -y keepalived haproxy
```

### 3.2 配置keepalived

``` shell
cat << EOF > /etc/keepalived/keepalived.conf
! /etc/keepalived/keepalived.conf
! Configuration File for keepalived
global_defs {
	router_id LVS_DEVEL
}
vrrp_script check_apiserver {
	script "/etc/keepalived/check_apiserver.sh"
	interval 3
	weight -2
	fall 10
	rise 2
}

vrrp_instance VI_1 {
	state ${STATE}
	interface ${INTERFACE}
	virtual_router_id ${ROUTER_ID}
	priority ${PRIORITY}
	authentication {
		auth_type PASS
		auth_pass ${AUTH_PASS}
	}
	virtual_ipaddress {
		${APISERVER_VIP}
	}
	track_script {
		check_apiserver
	}
}
EOF
```

在上面的文件中替换如下变更为自己的内容：

`${STATE}`如果是主节点，则为`MASTER`其他则为`BACKUP`。这里选择`k8s-master01`为`MASTER`；`k8s-master02`、`k8s-master03`为`BACKUP`；

`${INTERFACE}`是网络接口，即服务器网卡名，这里的服务器均为`eth33`；

`${ROUTER_ID}`这个值只要在`keepalived`集群中保持一致即可，这里使用默认值`51`;

`${PRIORITY}`优先级，在`master`上比在备份服务器上高就可以，这里的`master`设置为`100`，备份节点为`50`；

`${AUTH_PASS}`这个值只要在`keepalived`集群中保持一致即可；

`${APISERVER_VIP}`就是`VIP`的地址，这里设置为`192.168.126.20`。

### 3.3 配置keepalived健康检查

在上面的配置史们也配置健康检查的参数，比如检查间隔时间，权重等。

创建如下脚本：

```shell
vim /etc/keepalived/check_apiserver.sh
### 添加内容
#! /bin/bash

errorExit() {
	echo "*** $*" 1>&2
	exit 1
}

curl --silent --max-time 2 --insecure https://localhost:${APISERVER_DEST_PORT}/ -o /dev/null || errorExit "Error GET https://localhost:${APISERVER_DEST_PORT}/"
if ip addr | grep -q ${APISERVER_VIP}; then
    curl --silent --max-time 2 --insecure https://${APISERVER_VIP}:${APISERVER_DEST_PORT}/ -o /dev/null || errorExit "Error GET https://${APISERVER_VIP}:${APISERVER_DEST_PORT}/"
fi
```

`${APISERVER_VIP}`就是`VIP`的地址，`192.168.126.20`；

`${APISERVER_DEST_PORT}`这个是和`apiserver`交互的端口号，其实就是`HAProxy`绑定的端口号，因为`HAProxy`和k8s一起部署，这里做一个区分，我们使用了`16443`这个端口。

### 3.4 配置haproxy

``` shell
# /etc/haproxy/haproxy.cfg
# -------------------------------------------------
# Global settings
# -------------------------------------------------
global
	log /dev/log local0
	log /dev/log local1 notice
	daemon
	
# -------------------------------------------------
# common defaults that all the 'listen' and 'backend'
# sections will use if not designated in their block
# -------------------------------------------------
defaults
	mode				http
	log					global
	option				httplog
	option				dontlognull
	option http-server-close
	option forwardfor	 except 127.0.0.0/8
	option				redispatch
	retries				1
	timeout http-request	10s
	timeout queue			20s
	timeout connect			5s
	timeout client			20s
	timeout server			20s
	timeout http-keep-alive		10s
	timeout check			10s
	
# -------------------------------------------------
# apiserver frontend which proxys to the masters
# -------------------------------------------------
frontend apiserver
	bind *:${APISERVER_DEST_PORT}
	mode tcp
	option tcplog
	default_backend apiserver
	
# -------------------------------------------------
# round robin balancing for apiserver
# -------------------------------------------------
backend apiserver
	option httpchk GET /healthz
	http-check expect status 200
	mode tcp
	option ssl-hello-chk
	balance roundrobin
		server ${HOST1_ID} ${HOST1_ADDRESS}:${APISERVER_SRC_PORT} check
```

上面的配置需要修改为自己的配置

`${APISERVER_DEST_PORT}`这个值同上面的健康检查脚本里面的值一样，这里使用`16443`；

`${HOST1_ID} ${HOST1_ADDRESS}:${APISERVER_SRC_PORT}`其实就是你的k8s主节点的配置，比如我的配置是：

```shell
## server ${HOST1_ID} ${HOST1_ADDRESS}:${APISERVER_SRC_PORT} check

server k8s-master01 192.168.126.21:6443 check
server k8s-master02 192.168.126.22:6443 check
server k8s-master03 192.168.126.23:6443 check
```

上面的配置完成后启动`keepalived`和`haproxy`，并设置为自动启动。

```shell
systemctl enable haproxy --now
systemctl enable keepalived --now
```

## 四、安装Kubernetes

### 4.1 安装Docker

安装所需的包

```shell
yum install -y yum-utils device-mapper-persistent-data lvm2
```

添加docker国内仓库

``` shell
yum-config-manager --add-repo  http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

安装Docker

``` shell
# 最新版本
$ yum install -y docker-ce docker-ce-cli containerd.io
# 指定版本
$ yum install -y containerd.io-1.2.13 docker-ce-19.03.11 docker-ce-cli-19.03.11
```

### 4.2 配置docker

创建`docker`目录

``` shell
$ mkdir /etc/docker
```

添加配置

```shell
cat > /etc/docker/daemon.json << EOF
{
	"exec-opts": ["native.cgroupdriver=systemd"],
	"log-driver": "json-file",
	"log-opts": {
		"max-size": "100m"
	},
	"registry-mirrors": ["https://0gbs116j.mirror.aliyuncs.com","https://registry.docker-cn.com","https://mirror.ccs.tencentyun.com","https://docker.mirrors.ustc.edu.cn"],
	"storage-driver": "overlay2",
	"storage-opts": [
		"overlay2.override_kernel_check=true"
	]
}
```

设置docker自启动

```shell
mkdir -p /etc/systemd/system/docker.service.d

systemctl enable docker
systemctl daemon-reload
systemctl restart docker
```

### 4.3 安装kubelet、kubeadm、kubectl

配置阿里云仓库

``` shell
cat << EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

安装kubelet、kubeadm、kubectl

```shell
## 最新版本
yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
## 指定版本
yum install -y kubelet-1.18.6 kubeadm-1.18.6 kubectl-1.18.6 --disableexcludes=kubernetes
```

设置kubelet自启动

```shell
systemctl enable --now kubelet
```

以上步骤需要在3个`master`节点都执行。



## 五、master节点初始化

在master节点初始化，这里选择在节点k8s-master01上执行：

```shell
kubeadm init --control-plane-endpoint "k8s-vip:16443" \
--image-repository registry.aliyuncs.com/google_containers \
--service-cidr=10.1.0.0/16 --pod-network-cidr=10.244.0.0/16 \
--upload-certs | tee kubeadm-init.log
```

待初始化完成后，会有日志输出，这里的两部分内容需要记住：

![img](https://upload-images.jianshu.io/upload_images/14339262-8e1ec9f1c5e630ec.png?imageMogr2/auto-orient/strip|imageView2/2/w/1120/format/webp)

根据日志内容我们可以看到有两个`join`的输出，其中最上面的是`control-plane node`，即主节点的`join`命令，下面的则是`worker node`的`join`命令。

在k8s-master01节点配置环境变量：

```shell
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

在k8s-master02和k8s-master03节点上执行`join`命令，如下：

``` shell
kubeadm join k8s-vip:16443 --token j3vjyp.9namdndnyigysdp2 \
    --discovery-token-ca-cert-hash sha256:ad6b80c857adc41cb585f5fab771a064b13c2d069090365fa1f6d4cbefb5c257 \
    --control-plane --certificate-key a2828d4279238f976ce580977e5e113dddacd4ae0dd00903d4ebef8627ba33d1
```

这时候在k8s-master01节点执行

```shell
kubectl get node
```

会发现都是`NotReady`，接下来就配置网络组件，这里使用calico，在k8s-master01节点上执行：

``` shell
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

然后再次查看，可以发现节点全部处于正常状态了，如下图：

![img](https://upload-images.jianshu.io/upload_images/14339262-41c3a4202c086316.png?imageMogr2/auto-orient/strip|imageView2/2/w/699/format/webp)

再次查看系统下的其他组件，在k8s-master01节点执行：

```shell
kubectl get pod -n kube-system -o wide
```

结果如下图：

![img](https://upload-images.jianshu.io/upload_images/14339262-d1144e75bf190886.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

## 六、最后

接下来我们需要把k8s-master01的`/etc/kubernetes/admin.conf`配置文件拷贝到其他主节点上去，当然也可以复制到普通的worker节点，这都没什么问题，也可以加入普通的worker节点。需要注意的是初始化生成的证书只有2个小时的有效时间，如果证书过期了的话，需要在master节点上重新生成证书。

其实相对来讲高可用的k8s集群和之前的master节点和worker节点加入的命令稍微有一点区别。整体来看其实不复杂，但是因为自己对keepalived、haproxy都不熟悉，所以中间还是走了一些弯路，好在文档还是比较详细的，所以最终还是顺利完成了高可用集群的部署。

Kubernetes高可用集群部署，可参见[官方文档](https://links.jianshu.com/go?to=https%3A%2F%2Fkubernetes.io%2Fdocs%2Fsetup%2Fproduction-environment%2Ftools%2Fkubeadm%2Fhigh-availability%2F%23)。

