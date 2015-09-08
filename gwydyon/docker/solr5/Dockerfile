# DOCKER-VERSION 1.7
# VERSION        0.01
# CREDITS        Based on


# *never* use 'latest'!
 FROM phusion/baseimage:0.9.17

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

 # ------- SOLR START -------
 # from https://github.com/makuk66/docker-solr/blob/master/5.3/Dockerfile

 ENV SOLR_VERSION 5.3.0
 ENV SOLR solr-$SOLR_VERSION
 ENV SOLR_USER solr

 RUN export DEBIAN_FRONTEND=noninteractive && \
   apt-get update && \
   apt-get -y install lsof && \
   groupadd -r $SOLR_USER && \
   useradd -r -g $SOLR_USER $SOLR_USER && \
   mkdir -p /opt && \
   wget -nv --output-document=/opt/$SOLR.tgz http://www.us.apache.org/dist/lucene/solr/$SOLR_VERSION/$SOLR.tgz && \
   tar -C /opt --extract --file /opt/$SOLR.tgz && \
   rm /opt/$SOLR.tgz && \
   ln -s /opt/$SOLR /opt/solr && \
   mkdir -p /opt/solr/server/solr/lib && \
   chown -R $SOLR_USER:$SOLR_USER /opt/solr /opt/$SOLR

 EXPOSE 8983
 WORKDIR /opt/solr
 USER $SOLR_USER
 CMD ["/bin/bash", "-c", "/opt/solr/bin/solr -f"]

 # ------- SOLR END -------

 # Clean up APT when done.
 USER root
 RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*