# CentOS7 Java环境配置

在CentOS7环境下，安装、配置Java环境

## 一、安装JDK



## 二、安装配置Maven

### 2.1 下载Maven

``` shell
wget https://mirrors.tuna.tsinghua.edu.cn/apache/maven/maven-3/3.6.3/binaries/apache-maven-3.8.3-bin.tar.gz
```

### 2.2 解压缩

``` shell
tar -xf apache-maven-3.8.3-bin.tar.gz -C /usr/local/
mv /usr/local/apache-maven-3.8.3/ maven-3.8.3
```

### 2.3 配置环境变量

在`/etc/profile`文件内增加如下配置：

``` shell
MAVEN_HOME=/usr/local/maven-3.8.3
export PATH=${MAVEN_HOME}/bin:${PATH}
```



`export PATH=$PATH:/usr/local/maven3.8.3/bin`

添加完成后，执行如下命令，让环境变量生效：

`source /etc/profile`

### 2.4 验证Maven是否正常运行

`mvn -version`

如果返回如下结果，表示配置正确：

```shell
[root@localhost bin]# mvn -v
Apache Maven 3.8.3 (ff8e977a158738155dc465c6a97ffaf31982d739)
Maven home: /usr/local/maven-3.8.3
Java version: 1.8.0_311, vendor: Oracle Corporation, runtime: /usr/java/jdk1.8.0_311-amd64/jre
Default locale: zh_CN, platform encoding: UTF-8
OS name: "linux", version: "3.10.0-1160.45.1.el7.x86_64", arch: "amd64", family: "unix"
```

### 2.5 修改Maven的配置文件

`vim /usr/local/maven-3.8.3/conf/settings.xml`

#### 2.5.1 替换Maven源，找到`<mirrors></mirrors>`标签对，添加一下代码：

```xml
<mirror>
     <id>alimaven</id>
     <name>aliyun maven</name>
	 <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
     <mirrorOf>central</mirrorOf>
</mirror>
```

#### 2.5.2 指定Maven本地仓库位置

``` xml
<localRepository>/usr/local/maven-3.8.3/repository</localRepository>
```

#### 2.5.3 指定JDK版本

```xml
<profile>    
     <id>jdk-1.8</id>    
     <activation>    
       <activeByDefault>true</activeByDefault>    
       <jdk>1.8</jdk>    
     </activation>    
       <properties>    
         <maven.compiler.source>1.8</maven.compiler.source>    
         <maven.compiler.target>1.8</maven.compiler.target>    
         <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>    
       </properties>    
</profile>
```

## 三、配置运行Sentinel

### 3.1 下载Sentinel

下载sentinel-dashboard-1.8.2.jar，链接：[https://github.com/alibaba/Sentinel/releases/download/1.6.3/sentinel-dashboard-1.8.2.jar](https://github.com/alibaba/Sentinel/releases/download/1.6.3/sentinel-dashboard-1.8.2.jar)，将其自制到指定位置`cp sentinel-dashboard-1.8.2.jar /local/java/`

### 3.2 运行sentinel的Dashboard

``` shell
java -server -Xms64m -Xmx256m  -Dserver.port=8849 -Dcsp.sentinel.dashboard.server=localhost:8849 -Dproject.name=sentinel-dashboard -jar /local/java/sentinel-dashboard-1.8.2.jar
```

### 3.3 开放防火墙

``` shell
# 开放端口：
$ systemctl status firewalld
$ firewall-cmd --zone=public --add-port=8849/tcp --permanent
$ firewall-cmd --reload
$ firewall-cmd --list-ports
```

### 3.4 将Sentinel Doshboard设置成开机自启动

编辑：`/etc/rc.d/rc.local`

添加如下内容：

``` shell
/usr/bin/su - root -c "nohup java -server -Xms64m -Xmx256m -Dserver.port=8849 -Dcsp.sentinel.dashboard.server=192.168.92.130:8849 -Dproject.name=sentinel-dashboard -jar /local/java/sentinel-dashboard-1.8.2.jar 2>&1 &"
```

