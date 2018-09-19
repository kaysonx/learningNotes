##容器化第一讲：Docker 的实现原理和基本法
Docker 的出现一定是因为目前的后端在开发和运维阶段确实需要一种虚拟化技术解决开发环境和生产环境环境一致的问题，通过 Docker 我们可以将程序运行的环境也纳入到**版本控制**中，排除因为环境造成不同运行结果的可能。

1. docker（指的是docker daemon，用于控制每个container）就是一个进程，里面跑的container也就是一个进程，container里面跑的业务也就是container的子进程。因而业务进程必须是一个前台进程（阻塞进程），否则容器就会判定业务运行完成，自动终结。
2. docker 包含： client,server, image, image registry
3. docker是一个C/S架构。类比于MySQL, 所以可以用client去连接远程主机上的docker进行操作，如docker-machine
4. 容器共用操作系统内核，但在内核上却是互相隔离的。因为操作系统的内核不一样，所以windows和linux上打出来的镜像不兼容。
5. 虚拟机是虚拟的一整套操作系统，例如文件系统、网络、端口、内核这些等等。所以虚拟机启动起来很慢，因为要启动一个操作系统先，再能启动业务进程。而容器共用操作系统内核，启动也即是一个进程。所以启动、重启、销毁速度很快。虚拟机使用Snapshoot的方式来创建一致性的环境，而容器使用image的方式来保存。
6. container停止时，会向container里面的业务进程发送信号。以便于业务进程做一些数据的操作。


## 容器化第二讲：掌握Dockerfile的编写以及最佳实践
1. 镜像：用于创建容器的只读模板
2. 使用Dockerfile和docker build构建镜像。类比于Jenkins file
3. 好处：as code，像代码一样设计运行环境、实现版本控制
4. copy on write，每一层都是只读的，每一个RUN都是新起一个临时容器，共用下层的，但不可更改下层的资源，也即是copy 下层的 and wirte 在当前层。
5. dockerfile 每条命令都会在原有的层上创建一个制度的层。最后的run在image层上创建一个临时的可读写层。


## 容器化第三讲：Docker 
### 网络
1. 容器与容器之间都是相互隔离的，怎么实现通信？
2. 容器默认使用私有IP地址。通过各端口暴露服务、映射到主机端口（避免冲突）
3. 在不同主机上的容器之间不能相互通信的
<传统的link>
4. docker0 <subnet: 172.17.0.0/16>： 桥接网卡,虚拟交换机，每个容器分配私有IP然后连接到docker0. docker0在iptables里面做一个从主机到容器的数据映射.
5. 使用IP通信，不方便。因为IP是动态变化的。
6. Docker links<docker run --link>  
		限制：同一宿主机内可以使用，重新创建容器将会移除之前的链接，被链接的容器必须是一个已经启动的容器  
		原理：设置接收容器的环境变量，更新接收容器的etc/hosts，通过containername -> ip映射  

<新的link>  
1. 必须先创建一个新的用户自定义网络
2. docker network create
3. docker run --net=

### 容器的安全一般会上升到宿主机的安全问题

### 底层实现：
Namespace + CGroups + Union Filesystem  
Namespace: 文件系统、网络并与宿主机器之间的进程相互隔离  
CGroups: 隔离宿主机器上的物理资源，例如 CPU、内存、磁盘 I/O 和网络带宽  
UnionFS: 镜像问题解决

### Docker数据管理与网络
数据管理:  
共享本地主机数据到容器  
共享容器数据到另一个容器  
持久化  

原理:  
write to / -> 可读写的临时容器层  
write to /data -> 映射Volume  



	
### 集群化部署
####Docker compose
1. services、networks、volumes
2. 一个services代表一个container
3. volumes: 挂载一个目录或者已存在的数据卷容器
4. networks对应 docker network的操作

#### 复杂化的容器编排 -- k8s










