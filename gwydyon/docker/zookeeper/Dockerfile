# DOCKER-VERSION 1.7
# VERSION        0.01
# CREDITS        Based on Justin Plock's <justin@plock.net> Zookeeper Dockerfile, https://github.com/jplock/docker-zookeeper

FROM phusion/baseimage:0.9.17

MAINTAINER Software AG, Karsten Gresch <karsten.gresch@softwareag.com>

# Use baseimage-docker's init system.
CMD ["/sbin/my_init"]

# ------- JAVA 8 START -------
# from https://github.com/dockerfile/java/blob/master/oracle-java8/Dockerfile

RUN \
  echo oracle-java8-installer shared/accepted-oracle-license-v1-1 select true | debconf-set-selections && \
  add-apt-repository -y ppa:webupd8team/java && \
  apt-get update && \
  apt-get install -y oracle-java8-installer && \
  rm -rf /var/lib/apt/lists/* && \
  rm -rf /var/cache/oracle-jdk8-installer

# Define working directory.
WORKDIR /data

# Define commonly used JAVA_HOME variable
ENV JAVA_HOME /usr/lib/jvm/java-8-oracle

# Define default command.
CMD ["bash"]
# ------- JAVA 8 END -------

# ------- ZOOKEEPER START -------

# jplock RUN apt-get update && apt-get install -y openjdk-8-jre-headless wget
RUN wget -q -O - http://apache.mirrors.pair.com/zookeeper/zookeeper-3.4.6/zookeeper-3.4.6.tar.gz | tar -xzf - -C /opt \
    && mv /opt/zookeeper-3.4.6 /opt/zookeeper \
    && cp /opt/zookeeper/conf/zoo_sample.cfg /opt/zookeeper/conf/zoo.cfg \
    && mkdir -p /tmp/zookeeper

# jplock ENV JAVA_HOME /usr/lib/jvm/java-8-openjdk-amd64

EXPOSE 2181 2888 3888

WORKDIR /opt/zookeeper

VOLUME ["/opt/zookeeper/conf", "/tmp/zookeeper"]

ENTRYPOINT ["/opt/zookeeper/bin/zkServer.sh"]
CMD ["start-foreground"]

# ------- ZOOKEEPER END -------

# Clean up APT when done.
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*