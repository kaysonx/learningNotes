## Docker 基本操作

Docker install on Ubuntu:

```shell
sudo apt-get install -y docker-ce
version list: 
	apt-cache madison docker-ce
	sudo apt-get install docker-ce=<VERSION>

service status:
	systemctl status docker
	sudo systemctl start docker


```

**1.镜像操作**

```shell
docker pull xxx - 拉取镜像 默认latest

docker images - 列出所有镜像

docker rmi - 删除镜像

docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]] - 保存已存在的container为新的镜像

docker build [OPTIONS] PATH | URL | - 使用dockerfile构建镜像


```

**2.容器操作**

```dockerfile
docker ps -a - 列出所有container
docker exec -it xxx bash/sh - 执行返回可交互的终端
docker logs containerId - 查看container log

docker run \
  -u root \   -- 使用root用户启动<容器里的root，能操作你挂载的所有资源>
  -d \  -- 后台进程
  -p 8080:8080 \  — 端口映射
  -p 50000:50000 \ 
  -v jenkins-data:/var/jenkins_home \  -- volume挂载
  -v /var/run/docker.sock:/var/run/docker.sock \ 
  jenkinsci/blueocean 

docker cp 6855854febcd:/var/jenkins_home /jenkins-data - 复制container里面的数据

```

**3.docker内使用宿主机docker**

```
docker run -u root -d -p 8080:8080 -p 50000:50000 -v /Users/qspeng/DEV/jenkins-data:/var/jenkins_home **-v /var/run/docker.sock:/var/run/docker.sock -v $(which docker):/bin/docker** jenkins/jenkins 

jenkins container-- 默认jenkins用户，没有权限使用外面的docker。使用 -u root 或者 docker file再包一层权限/权限组设置

权限问题：root不安全 需要在原有的镜像上 包一层，加特定权限 （用户组）

挂载：npm 本地registry maven 

docker run \
-v /var/run/docker.sock:/var/run/docker.sock \
-v $(which docker):/bin/docker 
--name docker-manager -it ubuntu

```

**4.一种运行app的方式**

```
sudo cp -f build/libs/jenkins-demo-*.jar /var/app/

sudo docker rm -f $(sudo docker ps -a | grep jenkins-demo | awk '{print $1}') || echo 'No existing container...'

sudo docker run -u root --name jenkins-demo -d -p 9000:9000 -v /var/app/:/app jenkinsdemo
```

