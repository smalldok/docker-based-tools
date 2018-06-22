
# 拉取官方镜像
* 启动docker服务

* 拉取jenkins镜像(docker hub 官方)   
官方镜像搜索：https://hub.docker.com  
`docker pull jenkins:2.60.3` 或者`docker pull jenkins`
* 查看镜像   
`docker images`
* 删除镜像   
`docker rmi 7b210b6c238a` 或者 `docker rmi -f 7b210b6c238a`
* 运行镜像  
```
docker run --name myjenkins -p 8080:8080 -p 50000:50000 -v /Users/smalldok/work/idea-work/docker/volume/jenkins_home:/var/jenkins_home jenkins:2.60.3
```
或者以交互方式运行(这种适合更新修改镜像内部的东西)
```
docker run --name myjenkins -i -t -p 8080:8080 -p 50000:50000 -v /Users/smalldok/work/idea-work/docker/volume/jenkins_home:/var/jenkins_home jenkins:2.60.3 /bin/bash

然后exit退出、commit来重新创建镜像副本 
```
 
或者后台方式运行(推荐)
```
docker run --name myjenkins -d --env JAVA_OPTS="-Xmx8192m" -p 8080:8080 -p 50000:50000 -v /Users/smalldok/work/idea-work/docker/volume/jenkins_home:/var/jenkins_home jenkins:2.60.3
```
```
docker run --name myjenkins -d --env JAVA_OPTS="-Xmx8192m" -p 8080:8080 -p 50000:50000 -v /Users/smalldok/work/idea-work/docker/volume/jenkins_home:/var/jenkins_home -e ROOT_PASS="jenkins" jenkins:2.60.3
```

* jenkins解锁  
启动后访问http://127.0.0.1:8080/ 出现Unlock Jenkins页面，按提示在jenkins容器中获取密码；  
命令行进入容器:
```
docker ps -l 
docker exec -it 91e4b117c7f0 /bin/bash
cat /var/jenkins_home/secrets/initialAdminPassword
```
以root用户进入容器
```
docker exec -it -u root myjenkins /bin/bash
```
# 制作Dockerfile
### 验证jenkins plugin安装
##### jenkins plugin list
* Organization and Administration      
dashboard-view  
cloudbees-folder  
antisamy-markup-formatter   
backup
* Build Features  
token-macro  
build-timeout  
credentials-binding  
ws-cleanup  
timestamper 
* Build Tools   
nodejs  
build-name-setter  
scm-api 源码管理工具
* Build Analysis and Reporting  
无
* Pipelines and Continuous Delivery  
build-pipeline-plugin  
parameterized-trigger  
copyartifact
* Source Code Management  
git-client  
git  
gitlab-plugin    
github-pullrequest  
* User Management and Security  
matrix-auth   
pam-auth   
ldap   
role-strategy  
* Notifications and Publishing  
Publish Over SSH   
ssh-credentials  
SSH
Mailer  
email-ext
* project and view  
nested-view  
sectioned-view  
* docker  
docker-commons  
docker-build-publish  
docker-plugin  
kubernetes

##### jenkins plugin.txt
查找或下载插件 https://plugins.jenkins.io/
```
dashboard-view:latest
cloudbees-folder:latest
antisamy-markup-formatter:latest
backup:latest

token-macro:latest
build-timeout:latest
credentials-binding:latest
ws-cleanup:latest
timestamper:latest

nodejs:latest
build-name-setter:latest
scm-api:latest

build-pipeline-plugin:latest
parameterized-trigger:latest
copyartifact:latest

git-client:latest
git:latest
gitlab-plugin:latest
github-pullrequest:latest

matrix-auth:latest
pam-auth:latest
ldap:latest
role-strategy:latest

publish-over-ssh:latest
ssh-credentials:latest
ssh:latest
mailer:latest
email-ext:latest

nested-view:latest
sectioned-view:latest

docker-commons:latest
docker-build-publish:latest
docker-plugin:latest
kubernetes:latest
```
验证插件安装  
* 进入容器命令行
```
docker exec -it -u root myjenkins /bin/bash
```
* 准备plugins.txt
```
cd /usr/share/jenkins/
vi plugins.txt
```
* 执行插件安装脚本
```
/usr/local/bin/install-plugins.sh $(cat /usr/share/jenkins/plugins.txt | tr '\n' ' ')
```
单个失败，则重新安装,如：  
`/usr/local/bin/install-plugins.sh jira-trigger:latest`  
安装完成后，查看`/usr/share/jenkins/ref/plugins`目录；
* 登录jenkins 控制台页面查看  
http://127.0.0.1:8080/pluginManager/installed
### 验证JDK、GIT、SSH安装
JDK，默认在jenkins基础镜像中已经安装，见官方Dockerfile: `https://github.com/jenkinsci/docker/blob/master/Dockerfile`  
GIT，默认在上面安装jenkins插件git时已经安装;  
SSH，默认在上面安装jenkins插件ssh时已经安装;
```
java -version
git --version
ssh
```

### 验证maven安装
```
apt-get update && apt-get install -y wget
# get maven 3.5.3
wget --no-verbose -O /tmp/apache-maven-3.5.3.tar.gz http://archive.apache.org/dist/maven/maven-3/3.5.3/binaries/apache-maven-3.5.3-bin.tar.gz

# verify checksum
echo "51025855d5a7456fc1a67666fbef29de /tmp/apache-maven-3.5.3.tar.gz" | md5sum -c

# install maven
tar xzf /tmp/apache-maven-3.5.3.tar.gz -C /opt/
ln -s /opt/apache-maven-3.5.3 /opt/maven
ln -s /opt/maven/bin/mvn /usr/local/bin
rm -f /tmp/apache-maven-3.5.3.tar.gz

# env
vi ~/.bashrc

MAVEN_HOME="/opt/maven"
PATH="$PATH:$MAVEN_HOME"
export MAVEN_HOME
export PATH

保存wq! 然后生效source ~/.bashrc

#check MAVEN
mvn -v
echo $MAVEN_HOME
```
### 验证nodejs安装
```
# get nodejs 9.9.0
wget --no-verbose -O /tmp/node-v9.9.0-linux-x64.tar.gz https://nodejs.org/dist/v9.9.0/node-v9.9.0-linux-x64.tar.gz

# install nodejs
tar xzf /tmp/node-v9.9.0-linux-x64.tar.gz -C /opt/
ln -s /opt/node-v9.9.0-linux-x64 /opt/nodejs
ln -s /opt/nodejs/bin/node /usr/local/bin/node
ln -s /opt/nodejs/bin/npm /usr/local/bin/npm
rm -f /tmp/node-v9.9.0-linux-x64.tar.gz

#check nodejs
node -v
npm -v
```

### Dockerfile
```
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

```
### Build image
```
# build
docker build -t smalldok/jenkins:1.0 .

# check image
docker images

# run
docker run --name myjenkins -d --env JAVA_OPTS="-Xmx8192m" -p 8080:8080 -p 50000:50000 -v /Users/smalldok/work/idea-work/docker/volume/jenkins_home:/var/jenkins_home -e ROOT_PASS="jenkins" smalldok/jenkins:1.0

# open console
http://127.0.0.1:8080/

# stop container
docker stop myjenkins

# delete container
docker rm myjenkins

```
### Push image
* 注册Hub账号： https://hub.docker.com/
* 在账户中创建仓库：Create Repository
* login hub
```
docker login
```
* upload 
```
docker push smalldok/jenkins:1.0
```
* 其他命令  
```
# 修改镜像tag
docker tag <imageID> <namespace>/<image name>:<version tag eg latest>；
```

# 常见问题
### 问题1
删除容器，会删除`-v /your/home:/var/jenkins_home`指定挂载目录下的数据？  
不会，例如：执行`docker rm myjenkins`,并不会删除宿主机/your/home下的数据

### 问题2
构建Image Size过大问题？  
一条指令构建一层layer，应该减少层即可减少体积;  
```
RUN aaaaa
RUN bbbbb
RUN ccccc
```
应尽量改为：
```
RUN aaaaa \
    && bbbbb \
    && ccccc
```
### 问题3
docker: Error response from daemon: error creating overlay mount to /Users/smalldok/install/docker/docker-root-dir/overlay2/517ba0e3473b5196356d2c970322371964dff4c8803acc88421e71a34aabe783-init/merged: invalid argument.
不要修改docker root dir，还原为默认的/var/lib/docker下

### 问题4：
ocker: Error response from daemon: Conflict. The container name "/myjenkins" is already in use by container "e4c7f088c941063dfbea65c65ec683978dab2d309fa252a9f735594f6bb481d9". You have to remove (or rename) that container to be able to reuse that name.  
主机上运行docker rm xxxx 删除容器

### 问题5：
在容器中执行vi xxx.txt 报错 vi: command not found；  
在使用docker容器时，有时候里边没有安装vim，敲vim命令时提示说：vim: command not found，这个时候就需要安装vim，可是当你敲apt-get install vim命令时，提示：
```
Reading package lists... Done
Building dependency tree       
Reading state information... Done
E: Unable to locate package vim
```
这时候需要敲：apt-get update，这个命令的作用是：同步 /etc/apt/sources.list 和 /etc/apt/sources.list.d 中列出的源的索引，这样才能获取到最新的软件包。  
等更新完毕以后再敲命令：apt-get install vim命令即可。