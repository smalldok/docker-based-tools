# 基础镜像,基于官方jenkins2.60.3,包括openJDK8
# 见 https://github.com/jenkinsci/docker/blob/master/Dockerfile
# 
# 预安装jenkins各种插件maven/git/nodejs/docker plugin等等,详见: resources/plugins.txt
# 预安装maven、git、nodejs、SSH环境
#
# github: https://github.com/smalldok/jenkins+maven-git-nodejs-docker

FROM jenkins:2.60.3
MAINTAINER smalldok yaojialing5566@gmail.com

# Install Jenkins Plugins
COPY resources/plugins.txt /usr/share/jenkins/plugins.txt
RUN /usr/local/bin/install-plugins.sh $(cat /usr/share/jenkins/plugins.txt | tr '\n' ' ')

# Install maven
USER root
RUN apt-get update && apt-get install -y wget

# get maven 3.5.3
RUN wget --no-verbose -O /tmp/apache-maven-3.5.3.tar.gz http://archive.apache.org/dist/maven/maven-3/3.5.3/binaries/apache-maven-3.5.3-bin.tar.gz

# verify checksum
RUN echo "51025855d5a7456fc1a67666fbef29de /tmp/apache-maven-3.5.3.tar.gz" | md5sum -c

# install maven
RUN tar xzf /tmp/apache-maven-3.5.3.tar.gz -C /opt/ \
	&& ln -s /opt/apache-maven-3.5.3 /opt/maven \
	&& ln -s /opt/maven/bin/mvn /usr/local/bin \
	&& rm -f /tmp/apache-maven-3.5.3.tar.gz

ENV MAVEN_HOME /opt/maven

# Switch back to Jenkins user
USER jenkins

