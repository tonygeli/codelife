| Docker Index(Host) |
| ------------------ |
| Docker Daemon      |
| Container 1        |
| Container 2        |
| Container 3        |

Docker ClientDocker Indexdocker pulldocker rundocker ...Docker ClientDocker Index

> The Docker daemon
> 如上图所示，daemon在主机上面执行。用户只能通过client同daemon通讯。
> The Docker client
> Docker client是用户与Docker之间的重要接口。它从用户那里接受命令，并且将daemon的返回数据展现出来。 Inside Docker
> 为了深入理解docker的内部机制，需要了解以下三个组件：
> Docker images.(镜像)
> Docker registries.(仓库)
> Docker containers.(容器)

### So how does Docker work?

目前为止，我们已经可以完成以下几个工作：

1、创建一个包含你需要执行应用的镜像。
2、根据这个镜像，你可以创建一个容器。
3、你可以将这个容器上传到仓库中提供给其他人使用。

下面，我们看一下如何执行Docker。

### How does a Docker Image work?

我们知道Docker containers启动时所以来的Docker images其实是一个只读性质的模板。每个模板都包含若干层。Docker采取了 union file systems 的机制将这些曾聚合为一个image。Union file systems 允许物理隔离的文件或者目录，相互重叠覆盖，形成线性的文件系统。

Docker也正是基于上述层的实现方式而做到了轻量级。当你修改一个image时，docker并没有修改原有的image数据，而是新建了一个数据层。当你在docker中修改整个image或者重建实体时，原有数据都没有变化，只是若干层发生了变化。所以当image发生了变化时，不需要重新同步整个image，而只要将发生变化的层同步一次就可以。这样就使docker image做的快速并且简单。

每个image都是从base image演变过来的。你可以创建你的base image。如果你有apache的image，就可以把这个镜像作为你应用程序的base image。

Note：Docker 一般都是从Dock Hub上面获取base images。

Docker通过一些很简单的步骤就可以依据base images创建新的image。每执行一个步骤，新的image就会创建一个新层(layer)。基本的步骤如下：

- Run a command.
- Add a file or directory.
- Create an environment variable.
- What process to run when launching a container from this image.

这些命令可以再Dockerfile中定义。当你需要新建一个image是，docker可以自动读取Dockerfile中的命令，并且执行这些命令。最终生成一个新的image。

### How does a Docker registry work?

Docker registry是用来保存images的。当你新建好image后，就可以将image上传到Dock Hub或者你私有的store中。

借助于Docker client，你可以在Dock Hub检索你所需的image，同时将这些image下载到本地。

同时Dock Hub也提供公开和私有两种模式，处于公开模式下的image，所有人都可以下载和使用这些image。而处于私有模式下的image，只有本人或者经过授权后的人才能下载并且使用这些image。

### How does a container work?

一个标准容器包括：操作系统，用户自定义的文件和原数据。正如我们所知的那样，每个容器都是由image所创建的。image告诉docker，这个容器运行时，应该有哪些进程和应该有哪些配置参数。因为image是只读的，所以容器在运行时会在image原有层的基础上面创建一些可读可写的新层。而你的应用运行所需的数据将会被记录到这些数据层中。

### What happens when you run a container?

不论是使用docker程序或者API，docker client都会通知docker daemon如何操作容器。

当我们执行如下命令时：

```
$ docker run -i -t ubuntu /bin/bash
```

docker client会启动，然后使用后面的run参数来通知docker daemon启动一个新容器。这个简短的命令将会通知docker daemon以下信息：

1.容器所需的image在哪里，这里image名称是ubuntu，是一个base image。
2.当容器启动时，你想让容器初始化的动作，这里我们需要容器启动时自动切换到/bin/bash下面。

所以当我们敲下回车后，docker将会如何处理呢？

- **Pulls the ubuntu image:**
  Docker检测image是否存在，如果本地不存在，则默认从Dock Hub下载。如果本地存在，则使用本地的image启动容器。
- **Creates a new container:**
  Docker加载image，然后创建容器。 Allocates a filesystem and mounts a read-write layer : 容器开始创建文件系统，并且在image上面添加可读可写的数据层。
- **Allocates a network / bridge interface:** Docker开始创建网络接口，并且允许容器同主机进行关联。
- **Sets up an IP address:**
  Docker从IP资源池中挑选一个分配给容器。
- **Executes a process that you specify:** Docker开始执行指定的应用或者命令
- **Captures and provides application output:** Docker将执行过程当中的输出或者错误信息返回给Client。让用户可以知道当前应用执行的情况.

以上是容器的执行过程，下面我们将开始描述如何管理容器，包括：结束，停止和移除。

### The underlying technology

Docker 底层使用的是Linux内核中的虚拟化技术，来呈现我们刚才所看到的一切功能。

### Namespaces

Docker采用了称之为"Namespaces"的技术解决方案来隔离不同的workspace(也就是上面所定义的容器)。当你执行一个容器时，docker会为这个容器创建一系列的namespace。

以下是docker所创建的namespace：

- **The pid namespace**: 用来隔离进程。(PID就是process id)
- **The net namespace**: 用来管理网络接口
- **The ipc namespace**: 用来控制IPC资源的访问。
- **The mnt namespace**: 用来管理挂载点(mnt是 mount point)
- **The uts namespace**: 用来隔离内核和版本信息(UTS，分时复用系统 Unix Timesharing System)

### Control groups

Docker同时也采用了一种称之为"cgroups"的技术来控制group。不同应用之间隔离的关键在于，每个应用只能访问属于自己的资源。这样才能确保主机上面同时存在多个用户。Cgroups可以确保docker将可用的硬件资源共享给所有容器，并且可以在必要时间，对容器限制硬件资源。例如可以限制每个容器可以访问的内存容量。

### Union file systems

Union file systems 或者称为"UnionFS"是docker在创建层时采用的文件系统。这种文件系统使docker变得很轻量级并且执行速度很快。Docker使用UnionFS为容器提供相对应的数据块(data blocks)。Docker可以使用多种类型的UnionFS，比如：AUFS, btrfs, vfs, and DeviceMapper.

### Container format

Docker将上面我们所描述的各种组件封装成container数据类型(我们就称其为容器)。默认的容器类型是libcontainer。Docker同样也支持传统Linux使用LXC实现的容器类型。再未来，Docker也将支持其他类型的容器，比如：BSD Jails 或者Solaris Zones 版本的容器类型。

#### 安装Docker

```
sudo apt install docker.io
sudo usermod -a -G docker <your username>
restart your linux

usermod:
-modify a user account
-a add the user to the supplementary group. Use only with -G option
```

Q:Cannot connect to the Docker daemon. Is the docker daemon running on this host?

> A:The docker daemon binds to a Unix socket instead of a TCP port. By default that Unix socket is owned by the user root and other users can access it with sudo. For this reason, docker daemon always runs as the **root** user.

> To avoid having to use sudo when you use the docker command, create a Unix group called **docker** and **add users to it**. When the docker daemon starts, it makes the ownership of the Unix socket read/writable by the docker group.

#### 测试Docker

```
service docker status
sudo docker run hello-world
sudo docker run docker/whalesay cowsay xxx123
```

#### 显示所有images

```
sudo docker images
```

#### 显示所有containers

```
sudo docker ps -a
```

#### 删除container和image

```
sudo docker rm xxx_container
sudo docker rmi yyy_image
```

#### 交互启动container

```
sudo docker start -a -i xxx_container
```

#### 交互式运行image

```
sudo docker run -it xxx_container bash
```

#### 挂载某container

```
sudo docker attach xxx_container
```

#### 显示container或者image相关信息

```
sudo docker inspect img_name | container_name
```

#### 显示container IP

```
sudo docker inspect -f '{{.NetworkSettings.IPAddress}}' xxx_container
```

#### 复制文件

```
sudo docker cp local_file_path container:container_file_path
sudo docker cp container:container_file_path local_file_path
```

#### 创建镜像image

```
sudo docker build -f dockerfile -t img_name .
```

#### 保存镜像为文件

```
docker save -o filename.tgz img_name
```

#### 导入镜像

```
docker load -i img_name 
```

#### SSH 到镜像，即登录进行进行debug

```
docker exec -it img_name /bin/bash
Docker Registry基本操作

docker registry: https://github.com/docker/distribution.git
Start docker registry server:

docker run -d -it \ --name con_docker_registry \ -h hostname \ -v /data/docker-registry:/var/lib/registry \ -p 5000:5000 \ docker-registry
View the image list

curl -v -X GET localhost:5000/v2/_catalog
```