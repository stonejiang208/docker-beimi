## ubuntu-java
FROM ubuntu:16.04
MAINTAINER Stone Jiang <jiangtao@tao-studio.net>
COPY sources.list /etc/apt/sources.list
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

