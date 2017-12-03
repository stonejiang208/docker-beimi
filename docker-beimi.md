
# 跟我一步一步用Docker搭建Beimi游戏服务端开发环境

Stone 

## 摘要

本文为java 0基础的小伙伴介绍如何用Docker搭建开发环境。如果不关心过程直接可
跳到本文最后取代码。

关键步骤为：
1. 构建Beimi服务端依赖的开发环境
2. 获取 源代码
3. 编译
4. 运行和与前端调试

## 简介

Beimi (贝密) 是一款开源的棋牌软件，它的技术线路是：前端cocos creator，后端 java, spirng boot，数据库采用的mysql。本文针对于对java不太了解的小伙伴而写。

本文涉及到的技术要点要：
  * Docker
  * mysql 数据库导入
  

## 构建Beimi服务端依赖的开发环境

### 容器镜向脚本

```
## file: Dockerfile
## ubuntu-java
FROM ubuntu:16.04
MAINTAINER Stone Jiang <jiangtao@tao-studio.net>
## COPY sources.list /etc/apt/sources.list
RUN apt-get update && apt-get install -y --no-install-recommends \
    net-tools \
    ssh \
    sudo \
    locales \
    git \            
    mysql-client \
    maven


RUN locale-gen zh_CN.UTF-8
ENV LANG zh_CN.UTF-8
ENV LANGUAGE zh_CN:zh
ENV LC_ALL zh_CN.UTF-8

ADD jdk-8u151-linux-x64.tar.gz /opt/java

ENV JAVA_HOME=/opt/java/jdk1.8.0_151
ENV JRE_HOME=${JAVA_HOME}/jre
ENV CLASSPATH=${JAVA_HOME}/lib:${JRE_HOME}/lib:.
ENV PATH=${PATH}:${JAVA_HOME}/bin

VOLUME ["/mnt/workspace"]

```
### 构建容器镜

```
docker build -t ubuntu-java .
```



## 数据库

Beimi的源代码在附带了数据库脚本，在后面通过手功的方式导入。在代码仓库中也保留了一份导入完成的数据库，也可以直接使用。

数据库的Docker 镜像我们直接采用官方版本，启动脚本时，设置数据库的root密码为123456，这以Beimi源代码保持一致，省得再改配置脚本。

``` bash
docker run --name mysql \
-e MYSQL_ROOT_PASSWORD=123456 \
-d \
--restart always \
-h mysql \
-v `pwd`/data/mysql/:/var/lib/mysql \
mysql:latest --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci

```
为了数据库文件存在当前路径的/data/mysql目录中，为的是容器重启后还保留数据。

## 启动开发环境的容器

``` bash
docker run -h java --name "java" --rm -it \
-v `pwd`/home:/root \
-v `pwd`/workspace:/mnt/workspace \
--link "mysql:mysql" \
-p 8080:8080 \
-p 9081:9081 \
ubuntu-java bash

```

在开发环境里，我习惯把home目录绑定到容器的 root 目录上，方便 可以在.bashrc 设置环境变量，以及在.ssh目录下放ssh key等。


## 获取beimi的代码
### 用git 抓到源代码

我们把代码放在/mnt/workspace目录中，也是为了便于修改，不受容器重启的影响 。

代码的结构如下：

其中，数据库的脚本如下，如果是新部署的mysql数据库，需要创建数据库，并导入它。

代码仓库中有一份已导入完成的，想省事可以直接用它。

### 导入数据库

为了验证容器的连通性，所以我们在开发机上安装了mysql-client，利用开发机上的mysql 客户端远程（这里通过的是容器互联的方式）连接另一个容器中的mysql。将不同的服务独立部署在不同的容器中符合Docker 的理念。



## 确认环境，并打包生成war包

打包分以下几操作，具体的含义不是特别清楚，我也是java 不太懂。

进入到项目的主目录，即有pom.xml所在文件的目录，分别执行以下指令。

``` bash
mvn install:install-file -Dfile=src/main/resources/WEB-INF/lib/jave-1.0.2.jar -DgroupId=lt.jave -DartifactId=jave -Dversion=1.0.2 -Dpackaging=jar
```

``` bash
mvn install:install-file -Dfile=src/main/resources/WEB-INF/lib/ip2region-1.2.4.jar -DgroupId=org.lionsoul.ip2region -DartifactId=ip2region -Dversion=1.2.4 -Dpackaging=jar

```

``` bash
mvn package
```

会在target目录中生成 beimi-0.7.0.war。下一步就是部署这个war包，启动服务。


## 部署服务，供前端调试

将生成的war包移到一个单独的目录中，用下面的脚本启动

``` bash
java -Xms1240m -Xmx1240m -Xmn450m -XX:PermSize=512M  -XX:MaxPermSize=512m -XX:+UseParNewGC -XX:+UseConcMarkSweepGC -XX:+UseTLAB -XX:NewSize=128m -XX:MaxNewSize=128m -XX:MaxTenuringThreshold=0 -XX:SurvivorRatio=1024 -XX:+UseCMSInitiatingOccupancyOnly -XX:CMSInitiatingOccupancyFraction=60 -Djava.awt.headless=true  -XX:+PrintGCDetails -Xloggc:gc.log -XX:+PrintGCTimeStamps -jar beimi-0.7.0.war
```
