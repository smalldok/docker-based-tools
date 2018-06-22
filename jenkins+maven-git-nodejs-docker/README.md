
# 获取镜像
```sh
# 获取镜像
docker pull registry.cn-hangzhou.aliyuncs.com/smalldok/jenkins:1.0

# 删除镜像
docker rmi registry.cn-hangzhou.aliyuncs.com/smalldok/jenkins:1.0
```

# 运行容器
```sh
# 启动容器
docker run --name myjenkins -d --env JAVA_OPTS="-Xmx8192m" -p 8080:8080 -p 50000:50000 -v /Users/smalldok/work/idea-work/docker/volume/jenkins_home:/var/jenkins_home -e ROOT_PASS="jenkins" smalldok/jenkins:1.0

# 停止容器运行
docker stop myjenkins
# 启动容器
docker start myjenkins

# 删除停止的容器
docker rm myjenkins
```
`docker-compose` 方式运行
```sh
# 启动容器
docker-compose -p demo up -d
docker-compose ps

# 后台方式进入容器
docker exec -it -u root demo_myjenkins_1 /bin/bash

# 停止容器运行
docker-compose stop
# 启动容器
docker-compose start 

# 删除停止的容器
docker-compose rm 
```


# 访问jenkins  
启动后访问`http://127.0.0.1:8080/` 出现Unlock Jenkins页面，按提示在jenkins容器中获取密码；  
命令行进入容器:
```sh
docker ps -l 
docker exec -it <容器id/容器名称> /bin/bash
cat /var/jenkins_home/secrets/initialAdminPassword
```
以root用户进入容器
```sh
docker exec -it -u root myjenkins /bin/bash
```

# 参考地址
此次构建Dockerfile参考: https://github.com/eggsy84/jenkins-maven-git-docker/blob/master/Dockerfile

jenkins Dockerfile: https://github.com/jenkinsci/docker/blob/master/Dockerfile  
jenkins镜像的使用/日志输出/参数设置等，参考文档： https://hub.docker.com/_/jenkins/

docker官方jenkins镜像地址：https://hub.docker.com/_/jenkins/  
jenkins官方jenkins镜像地址：https://hub.docker.com/r/jenkinsci/jenkins/  

docker国内镜像地址：https://hub.daocloud.io(需要注册)  
jenkins官方plugins: https://plugins.jenkins.io/