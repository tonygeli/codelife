#### Docker Compose

简介：管理容器，定义运行多个容器

docker-compose.yml 的配置案例如下（配置参数参考下文）：

### 创建 docker-compose.yml

在测试目录中创建一个名为 docker-compose.yml 的文件，然后粘贴以下内容：

```
# yaml 配置
version: '3'
services:
  web:
    build: .
    ports:
     - "5000:5000"
  redis:
    image: "redis:alpine"
```

该 Compose 文件定义了两个服务：web 和 redis。

- **web**：该 web 服务使用从 Dockerfile 当前目录中构建的镜像。然后，它将容器和主机绑定到暴露的端口 5000。此示例服务使用 Flask Web 服务器的默认端口 5000 。
- **redis**：该 redis 服务使用 Docker Hub 的公共 Redis 映像。

### 使用 Compose 命令构建和运行您的应用

在测试目录中，执行以下命令来启动应用程序：

`docker-compose up`

如果你想在后台执行该服务可以加上 **-d** 参数：

`docker-compose up -d`

























#### Docker Swarm



#### Docker Stack



#### Docker Secret



### Docker Config



### K8s