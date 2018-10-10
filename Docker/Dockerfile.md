### Docker file

一般镜像通常包括：

1.一个轻量级的操作系统发行版

2.相关依赖(例如JDK Tomcat、Nginx)

3.单个应用或者服务

也即是dockerfile中应包含以上



以springboot应用为例(https://github.com/qspeng/serivices-for-docker)

1.linux

2.JDK

3.jar

4.java -jar xxx.jar



**DOCKER FILE命令**

FROM 

> 功能：指定基础镜像，新的镜像将基于此镜像来构建。 
>
> 格式：FROM <image name>/<image_name>:<tag>
>
> 一个有效的Dockerfile必须以FROM作为第一条非注释指令 
>
> 除了ARG指令 用于指定变量参数
>
> tag为空 默认为latest -- 最好不要使用latest version

MAINTAINER

>功能：设置该镜像的作者 。
>
>格式：MAINTAINER <author name> 
>
>此命令已经废弃  推荐使用LABEL的方式
>
>eg: LABEL maintainer="qspeng@thoughtworks.com"

RUN

>功能：在shell或exec的环境下执行的命令 。指令会在新创建的的镜像上添加新的层面，接下来会提交结果用在下一条指令中。
>
>也即是从之前的基础镜像启动一个容器 然后 run 然后commit为新的image供后续指令使用
>
>格式：RUN <command> 

WORKDIR

>功能：指定RUN、CMD与ENTRYPOINT命令的工作目录 — 后续指令路径需要以此为相对路径 或者 使用绝对路劲
>
>格式：WORKDIR PATH 

ADD/COPY

>功能：复制文件指令。destination是容器内的路径，source可以是url或者是启动配置上下文中的一个文件/目录。 
>
>格式：ADD/COPY <src> <destination> 
>
>​            COPY <源路径>... <目标路径>
>
>远程URL推荐使用curl 或者 wget之类的命令来获取 而不是以上命令
>
>以上指令的区别在于ADD包含类似tar的解压功能 安装一些使用tar压缩的软件包时会有帮助 纯复制推荐COPY

EXPOSE

>功能：暴露容器运行时监听的端口
>
>格式:   EXPOSE <port> 

CMD

>功能：CMD指令的主要功能是在build完成后，为了给docker run启动到容器时提供默认命令或参数，在容器每次启动时都会被运行（类似于开机启动程序）
>
>Dockerfile只允许使用一次CMD命令。多个CMD命令只有最后一条生效。
>
>格式：CMD [“executable”,”param1”,”param2”]
>
>​            CMD [“param1”, “param2”]
>
>​            CMD command param1 param2

ENTRYPOINT

>功能：与CMD功能相似，不同的地方在于 ENTRYPOINT 不会被忽略，一定会被执行，即使运行 docker run 时指定了其他命令。而CMD的额外参数可以在容器启动时动态替换掉。
>
>格式：ENTRYPOINT <command> (shell格式)
>
>​           ENTRYPOINT [“executable”, “param1”, “param2”] (exec格式)

ENV

>功能：设置环境变量 。使用键值对。
>
>格式：ENV <key> <value>

VOLUME

>功能：授权访问从容器内到主机的目录
>
>格式：VOLUME [“/data”]
>
>挂载volume DOCKERFILE中部允许直接共享主机目录到container 因为兼容性，HOST上不一定有这个目录存在
>
>直接指定一个目录会在容器运行时自动挂载到/var/docker/volume/*的一个随机唯一目录下
>
>或者 指定一个volume名称，然后手动创建一个volume

最佳实践

>**目标**
>
>更快的构建速度
>
>更小的*Docker*镜像大小
>
>更少的*Docker*镜像层
>
>充分利用镜像缓存
>
>增加*Dockerfile*可读性
>
>让*Docker*容器使用起来更简单
>
>
>
>**最佳实践**
>
>编写*.dockerignore*文件
>
>容器只运行单个应用
>
>将多个*RUN*指令合并为一个
>
>基础镜像的标签不要用*latest*
>
>每个*RUN*指令后删除多余文件
>
>选择合适的基础镜像*(alpine*版本最好*)*
>
>设置*WORKDIR*和*CMD*
>
>使用*ENTRYPOINT (*可选*)*
>
>在*entrypoint*脚本中使用*exec*
>
>*COPY*与*ADD*优先使用前者
>
>合理调整*COPY*与*RUN*的顺序
>
>设置默认的环境变量，映射端口和数据卷
>
>使用*LABEL*设置镜像元数据
>
>添加*HEALTHCHECK*

**镜像命令**

构建镜像

> 命令：docker build -t <imageName>:<tag>  .
>
> ​	    docker build -t <imageName>:<tag>  -f

查看镜像

> 命令：docker image list

拉取镜像<向注册过的仓库拉取镜像>

> 命令：docker pull <image>:<tag>

提交镜像

> 命令： docker push <image>:<tag>

删除镜像

> 命令：docker rmi <imageId>/<imageName>

镜像历史

> 命令：docker history <image>:<tag>



Ex:

>FROM alpine:edge
>
>LABEL maintainer="qspeng"
>
>RUN apk add --no-cache openjdk8
>
>VOLUME /tmp
>
>COPY  user-service-*.jar user-service-*.jar
>
>WORKDIR /app
>
>EXPOSE 8081
>
>ENTRYPOINT ["java", "-jar", "/user-service-*.jar"]

