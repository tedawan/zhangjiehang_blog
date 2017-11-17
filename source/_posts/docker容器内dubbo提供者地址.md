---
title: docker容器dubbo的提供者的地址
date: 2017-11-17 13:59
categories: dubbo
---
# docker容器dubbo的提供者的地址
<!--more-->
---


最新dubbo迎来了三个月进行三次版本的迭代，我惊讶的发现在2.5.7版本上面处理了docker 容器内provider注册到注册中心地址的问题。

点击：[dubbo版本更新日志](https://github.com/alibaba/dubbo/releases)

具体demo:[dubbo-docker-sample](https://github.com/dubbo/dubbo-docker-sample)
1. 有些部署场景需要动态指定服务注册的地址，如docker bridge网络模式下要指定注册宿主机ip以实现外网通信。dubbo提供了两对启动阶段的系统属性，用于设置对外通信的ip、port地址  
- DUBBO_IP_TO_REGISTRY --- 注册到注册中心的ip地址
- DUBBO_PORT_TO_REGISTRY --- 注册到注册中心的port端口
- DUBBO_IP_TO_BIND --- 监听ip地址  
- DUBBO_PORT_TO_BIND --- 监听port端口

2. 上面有个问题，就是说在代码里面宿主机的暴露的协议端口要和docker里面端口保持一致，不然使用代码无法访问

3. 如果是每次打包都要重新生成一个docker容器的话，主要靠里面的exec把DUBBO_IP_TO_REGISTRY和DUBBO_PORT_TO_REGISTRY这两个写入到环境变量中

4. 如果是每次都在容器内重新打包部署项目，需要自己在环境变量里面设置好。同时执行

```
source /etc/profile
```
---
下面描述一下每次都在容器内打包部署步骤：
1. 先准备好jdk的dockerfile
2. docker build构建镜像

```
docker build --no-cache -t dubbo-docker-sample .
```
###### dockerfile:base
```
FROM docker.io/centos:latest

MAINTAINER zjh

# 安装环境
ENV TZ=Asia/Shanghai
RUN yum install -y firewalld firewall-config net-tools.x86_64 vim initscripts gcc g++ gcc-c++ make wget telnet openssh-server zlib pcre pcre-devel \
&& yum clean all \
&& ssh-keygen -q -t rsa -b 2048 -f /etc/ssh/ssh_host_rsa_key -N '' \
&& ssh-keygen -q -t ecdsa -f /etc/ssh/ssh_host_ecdsa_key -N '' \
&& ssh-keygen -t dsa -f /etc/ssh/ssh_host_ed25519_key -N '' \
&& echo "root:root" | chpasswd \

# 配置时间
&& cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
&& echo 'export TZ=Asia/Shanghai' >> /etc/profile && source /etc/profile

ENTRYPOINT ["/usr/sbin/init"]
EXPOSE 22

```
###### dockerfile:jdk
```
FROM songyu/base:0.0.1

MAINTAINER zjh

# 安装jdk
ADD jdk-8u111-linux-x64.tar.gz /usr/local
ADD startup.sh /startup.sh
ADD start_springboot.sh /start_springboot.sh
ENV JAVA_HOME=/usr/local/jdk1.8.0_111
ENV JRE_HOME=/usr/local/jdk1.8.0_111/jre
ENV CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
ENV PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
RUN echo 'JAVA_HOME=/usr/local/jdk1.8.0_111' >> /etc/profile &&\
echo 'JRE_HOME=/usr/local/jdk1.8.0_111/jre' >> /etc/profile &&\
echo 'CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib' >> /etc/profile &&\
echo 'PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin' >> /etc/profile &&\
echo 'export JAVA_HOME' >> /etc/profile &&\
echo 'export PATH' >> /etc/profile &&\
echo 'export CLASS_PATH' >> /etc/profile &&\
echo 'export JRE_HOME' >> /etc/profile &&\
source /etc/profile && chmod 777 /startup.sh && chmod 777 /start_springboot.sh \

# 生成启动需要执行的脚本
&& echo '#!/bin/bash' > /sy_init.sh \
# 输入宿主机ip
&& echo 'DOCKER_IP=' > /sy_init.sh \
&& echo 'cp /etc/hosts /etc/hosts.temp' >> /sy_init.sh \
&& echo 'sed -i "s/.*$(hostname)/$DOCKER_IP $(hostname)/" /etc/hosts.temp' >> /sy_init.sh \
&& echo 'cat /etc/hosts.temp > /etc/hosts' >> /sy_init.sh \
&& chmod 777 /sy_init.sh \

# 加入启动执行
&& chmod +x /etc/rc.d/rc.local \
&& echo '/sy_init.sh' >> /etc/rc.d/rc.local

VOLUME ["/code"]
ENTRYPOINT ["/usr/sbin/init"]

```
3. 从镜像创建容器
```
docker run -d -e DUBBO_IP_TO_REGISTRY=192.168.57.244 -e DUBBO_PORT_TO_REGISTRY=20280  -p 10101:22 -p 192.168.57.244:20280:20280 --name dubbo_docker3 jdk_1.8:0.0.1
```
4. 进入环境变量
```
vi /etc/profile

DUBBO_IP_TO_REGISTRY=192.168.57.244
DUBBO_PORT_TO_REGISTRY=20280

export DUBBO_IP_TO_REGISTRY
export DUBBO_PORT_TO_REGISTRY

source /etc/profile
```
5. 把对应的工程执行起来就可以了。


---

- 由于dubbo上线的官网的方式是采用从镜像创建容器并运行方式，导致我自己刚开始处理一直都处理不了
> 官网例子的Dockerfile
```
FROM openjdk:8-jdk-alpine
ADD target/dubbo-docker-sample-0.0.1-SNAPSHOT.jar app.jar
ENV JAVA_OPTS=""
ENTRYPOINT exec java $JAVA_OPTS -jar /app.jar
```
> 官网执行的容器代码

```
docker run -e DUBBO_IP_TO_REGISTRY=30.5.97.6 -e DUBBO_PORT_TO_REGISTRY=20881 -p 30.5.97.6:20881:20880 --link zkserver:zkserver -it --rm dubbo-docker-sample
```
- 上面可以看到docker run的时候使用可-e的命令，说明在创建时候就把这些直接写到环境变量里面，同时通过执行dockerfile exec里面命令启动，自动添加到环境变量


---

最终，我还是发现除了适应每次生成容器的方式进行打包，否则的话不如直接去改docker里面的hosts文件来的快