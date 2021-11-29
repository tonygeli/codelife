## Docker via Homebrew

Running Docker on Mac requires VirtualBox so if you don’t have it already:

```bash
brew cask install virtualbox
```

Then install Docker and the helper tool boot2docker:

```bash
brew install docker
brew install boot2docker
```

## 使用命令行

```bash
1、 创建一个新的 Boot2Docker 虚拟机
>$ boot2docker init
这会创建一个新的虚拟主机，你只需要运行一次这个命令就可以了，以后就不需要了。

2、 启动 boot2docker 虚拟机。
>$ boot2docker start
To connect the Docker client to the Docker daemon, please set:
    export DOCKER_CERT_PATH=/Users/user/.boot2docker/certs/boot2docker-vm
    export DOCKER_TLS_VERIFY=1
    export DOCKER_HOST=tcp://192.168.59.103:2376

Or run: `eval "$(boot2docker shellinit)"`


3、 通过 docker 客户端来查看环境变量
>$ boot2docker shellinit


4、 使用 shell 命令来设置环境变量。
>$ eval "$(boot2docker shellinit)"
Writing /Users/user/.boot2docker/certs/boot2docker-vm/ca.pem
Writing /Users/user/.boot2docker/certs/boot2docker-vm/cert.pem
Writing /Users/user/.boot2docker/certs/boot2docker-vm/key.pem

5、 运行 hello-word 容器来验证安装。
>$ docker run hello-world
```

## Boot2Docker 基本练习

```bash
>$ boot2docker status
>$ docker version 

容器端口访问
1、 在 Docker 主机上启动一个 Nginx 容器。
>$ docker run -d -P --name web nginx
```

一般来说，docker run 命令会启动一个容器，运行这个容器，然后退出。-d 标识可以让容器在 docker run 命令完成之后继续在后台运行。 -P 标识会将容器的端口暴露给主机，这样你就可以从你的 MAC 上访问它。

```bash
2、 使用 docker ps 命令来查看你运行的容器
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                           NAMES
d478595f206d        nginx               "nginx -g 'daemon off"   4 minutes ago       Up 4 minutes        0.0.0.0:32769->80/tcp, 0.0.0.0:32768->443/tcp   web

3、 查看容器端口
> docker port web
443/tcp -> 0.0.0.0:32768
80/tcp -> 0.0.0.0:32769
上边的显示告诉我们，web 容器将 80 端口映射到 Docker 主机的 32769 端口上。

4、 在浏览器输入地址 http://localhost:32769/ (localhost 是 0.0.0.0):

没有正常工作。没有正常工作的原因是 DOCKER_HOST 主机的地址不是 localhost (0.0.0.0),但是你可以使用 boot2docker 虚拟机的IP地址来访问。

5、 获取 boot2docker 主机地址
>$ boot2docker ip
192.168.59.103
6、在浏览器中输入 http://192.168.59.103:49157
7、 通过如下方法，停止并删除 nginx 容器。
>$ docker stop web
>$ docker rm web
```

## 给容器挂载一个卷

当你启动 boot2docker 的时候，它会自动共享 /Users 目录给虚拟机。你可以利用这一点，将本地目录挂载到容器中。这个练习中我们将告诉你如何进行操作。

```bash
1、 回到你的 $HOME 目录
 $ cd $HOME
2、 创建一个新目录，并命名为 site
$ mkdir site
3、 进入 site 目录。
$ cd site
4、 创建一个 index.html 文件。
$ echo "my new site" > index.html
5、 启动一个新的 nginx 容器,并将本地的 site 目录替换容器中的 html 文件夹。
>$ docker run -d -P -v $HOME/site:/usr/share/nginx/html --name mysite nginx
6、 获取 mysite 容器端口
>$ docker port mysite
80/tcp -> 0.0.0.0:49166
443/tcp -> 0.0.0.0:49165
7、 在浏览器中打开站点。

8、 现在尝试在 $HOME/site 中创建一个页面
>$ echo "This is cool" > cool.html
9、 在浏览器中打开新创建的页面。

10、 停止并删除 mysite 容器。
>$ docker stop mysite
>$ docker rm mysite  
```