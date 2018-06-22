# Docker-in-Docker based on centos original image

`主要用于测试docker-swarm集群；`

在部署集群的时候，如果没有现成机器，可以在本机部署Docker，运行几个容器作为节点服务器使用。你可能想在节点里也部署个Docker，但是直接在容器里安装Docker是有问题，还好有人已经有解决方案-dind，详见：https://github.com/jpetazzo/dind。

直接在本机启动容器即可
```
docker run --privileged -d docker:dind
```
dind没有提供centos版本的Dockerfile，需要自己改一下。
现在Centos 7容器里启动Docker，会报错：Failed to get D-Bus connection: Operation not permitted，这是因为centos 7用fakesystemd替代了systemd，可以在centos7下systemd解决问题。
我对dind Dockerfile做了小修改，可支持运行centos7版本的dind，详见：https://github.com/AixC/dind_centos。


### Step 1:
```sh
$ git clone https://github.com/smalldok/docker-based-tools.git
```

### Step 2: 构建镜像
```sh
$ cd dind_centos
$ ./build.sh
```

### Step 3: 运行
```sh
$ docker run --name dind_centos --privileged -d dind_centos
```

### Step 4: 进入容器后台
```sh
$ docker exec -it -u root dind_centos /bin/bash
$ docker info #查看docker版本等信息
```

参考：https://www.cnblogs.com/cs-zh/p/7878193.html
