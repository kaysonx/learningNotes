### Docker Compose

> 多容器部署方案
>
> compose file -> $docker-compose up -> containers
>
> 是一个工具 需要单独安装
>
> 通过yml文件定义多容器应用 默认名称为docker-compose.yml

**核心组件**

1.Services

>一个service 表示一个 container
>
>service的启动类似于docker run 可以给其指定network和volume
>
>



2.Network

3.Volumes



外层的network和volumes可供所有service里面的container使用

另外

environment：用于指定环境变量

env_file：用于从一个文件中加入环境变量 -- 就近原则 environment中指定的优先级更高

external_links：连接到在这个*docker-compose.yml*文件或者*Compose*外部启动的容器，特别是对于提供共享和公共服务的容器。在指定容器名称和别名时，*external_links*遵循着和*links*相同的语义用法

>*external_links:*  
>
>  *- redis_1*   
>
>  *- project_db_1:mysql*   
>
>  *- project_db_1:postgresql*

**基本命令**

构建

>命令： docker-compose up -d 
>
>​             -d 后台运行

查看容器状态

>命令： docker-compose ps

查看容器日志

>命令： docker-compose logs

停止并清除容器

>命令： docker-compose down (需在docker-compose.yml文件同目录下,或 -f 指定docker-compose.yml路径)



Ex

```dockerfile
version: '3'
services:
  orderservice:
    container_name: orderservice
    depends_on:
      - userservice
      - logisticservice
      - productservice
    image: orderservice:v1
    ports:
      - 8080:8080
    networks: 
      - onlineNetwork
    restart: always
    environment:
      - TEST=true
  userservice:
    container_name: userservice
    image: userservice:v1
    ports:
      - 8081:8081
    networks: 
      - onlineNetwork
    restart: always
  productservice:
    container_name: productservice
    image: productservice:v1
    ports:
      - 8083:8083
    networks: 
      - onlineNetwork
    restart: always
  logisticservice:
    build: ./logistics-service/
    ports:
      - 8082:8082
    networks: 
      - onlineNetwork
    restart: always
networks: 
  onlineNetwork:
    driver: bridge
volumes:
  logvolume: {}
```

>http://localhost:8082/logistics-service/logistics/xxxx
>
>http://localhost:8080/order-service/orders/1234567890
>
>http://localhost:8083/product-service/products/1234567890
>
>http://localhost:8081/user-service/users/10000

> 以下命令需要再docker-compose.yml目录下使用
>
> docker-compose ps
>
> docker-compose build
>
> docker-compose down
>
> docker-compose start/stop
>
> docker-compose logs

