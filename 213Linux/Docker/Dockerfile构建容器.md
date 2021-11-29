### 使用Dockerfile构建容器

$touch Dockerfile

```bash
FROM ubuntu:14.04
MAINTAINER Tony 
ENV REFRESHED_AT 2017-05-08
RUN apt-get update
RUN apt-get -y -q install nginx
RUN mkdir -p /var/www/html
ADD nginx/global.conf /etc/nginx/conf.d/

ADD nginx/nginx.conf /etc/nginx/nginx.conf
EXPOSE 80
```

这个Dockerfile内容包括以下几项。

- 安装Nginx
- 在容器中创建一个目录/var/www/html
- 将来自我们下载的本地文件的Nginx配置文件添加到镜像中。
- 公开镜像80端口

将文件nginx/global.conf用ADD指令复制到/etc/nginx/conf.d目录中。

global.conf

```bash
server{
    listen 0.0.0.0:80;
    server_name _;
    
    root /var/www/html/website;
    index index.html index.htm;

    access_log /var/log/nginx/default_access.log;
    error_log /var/log/nginx/default_error.log;
}
```

这个文件将Nginx设置为监听80端口，并将网络服务的根路径设置为/var/www/html/website，这个目录是我们用RUN指令创建的。

然后我们还需要将Nginx配置为非守护进程的模式，这样可以让Nginx在Docker容器里工作。将文件nginx/nginx.conf复制到/etc/nginx目录就可以达到这个目的。

nginx.conf配置文件

```bash
user www-data;
worker_processes 4;
pid /run/nginx.pid;
daemon off;

events {  }
http{
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;
    gzip on;
    gzip_disable "msie6";
    include /etc/nginx/conf.d/*.conf;
}
```