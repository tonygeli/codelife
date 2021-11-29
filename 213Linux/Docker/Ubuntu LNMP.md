由于当前多数服务器都是Linux的原因，本文只讲ubuntu下的安装！

## 第一步：安装docker

安装教程：http://www.runoob.com/docker/ubuntu-docker-install.html

## 第二步：拉取ubuntu:16.04镜像

> docker pull ubuntu:16.04

拉取成功后，查看所有镜像

> docker images

## 第三步：将该镜像在一个容器中运行，并进入容器

```
docker run -dit --name my-lnmp ubuntu:16.04

docker exec -it my-lnmp /bin/bash
```

## 第四步：更新容器 apt 源，安装curl，vim

```
apt-get update

apt-get install curl

apt-get install vim
```

## 第五步：安装nginx

```
apt-get install nginx
# 配置文件位置#>

 /etc/nginx/nginx.conf

 /etc/nginx/conf.d/*.conf
# 默认主目录#> 

/usr/share/nginx/html/
# 管理nginx服务

service nginx start       // 启动

service nginxstop        // 停止

service nginx restart   // 重启

测试 curl localhost
```

## 第六步：安装php7

```
apt-get install php

apt-get install php7.0-mysql php7.0-curl php7.0-xml php7.0-mcrypt php7.0-json php7.0-fpm

php7.0-gd php7.0-mbstring php-mongodb php-memcached php-redis
```

测试 (如果有结果，则表示安装成功)

```
php-v
```

配置php.ini

```
vim /etc/php/7.0/fpm/php.ini

# 将cgi.fix_pathinfo=1这一行去掉注释，将1改为0

#>  / 是vi查找的命令
```

配置php-fpm

```
vim /etc/php/7.0/fpm/pool.d/www.conf

# 修改 listen = /var/run/php/php7.0-fpm.sock
```

配置nginx

```
vim /etc/nginx/sites-enabled/default

将index index.html index.htm;改成index index.php index.html index.htm;
```

在service里面，location /{}下面增加以下配置

```
location ~ \.php$ {

fastcgi_split_path_info ^(.+\.php)(/.+)$;

# NOTE: You should have "cgi.fix_pathinfo = 0;" in php.ini

# With php5-cgi alone:

# fastcgi_pass 127.0.0.1:9000;

# With php5-fpm:

fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;

fastcgi_index index.php;

fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;

include fastcgi_params;

}
```

## 容器中运行

1. **启动 php-fpm**

> service php7.0-fpm start

1. **重启 Nginx ，检测配置是否成功**

> service nginx restart

1. **新建 index.php 测试文件**

```
<?php

echo "Hello World";
```

1. **执行**

> curl localhost #如果看到`hello world!`表示运行成功

## 第七步：安装mysql

```
apt-get install mysql-server

//测试
service mysql start
mysql -uroot -p
```

## 第八步：设置容器开机启动项

```
在.bashrc写入开机启动项

vim~/.bashrc
写入以下内容，保存

# 开机启动项

service php7.0-fpm start

service mysql start

service nginx start

# tail -f /var/log/nginx/error.log
将配置好的Docker容器，打包上传阿里云

退出 docker

exit

查看容器对应的 CONTAINER ID

docker ps -as

将容器打包成新镜像

docker commit  [CONTAINER ID]  new-lnmp

停止正在运行的容器

docker stop my-lnmp
```

## 第九步：设置ssh登录docker

```
//暴露docker端口22到主机端口9000
dokcer run -p 9000:22

apt-get update
apt-get install openssh-server
vim /etc/ssh/sshd_config
//修改 
PermitRootLogin yes
# StrictMode yes

#修改用户密码
passwd
#启动ssh服务器
service ssh start


docker restart container

ssh 192.168.99.100 -p 9000
```

## 使用刚打包的镜像，创建容器，-p 端口映射# -v 本地目录映射到容器内

```
docker run -dit -p 80:80 -p 3306:3306 -p 9000:22 -v /var/www/:/var/www/  --name nginx-mysql-php7 

new-lnmp /bin/bash
```

-p的意思是 将docker的80端口和container的80端口绑定

#### 在浏览器通过访问docker的ip（192.168.99.100）响应成功，则大功告成。

#### 如果是Windows操作系统，docker的根目录是/c/user/User，可通过pwd查看。此时通过-v挂载磁盘的时候，建议直接在此目录下生成一个www文件夹。那么就可以通过以下命令生成容器了。

```
docker run -dit -p 80:80 -p 3306:3306 -p 9000:22 -v ~/www:/var/www/html --name lnmp1 new-lnmp /bin/bash

或e:/path/to
docker ip：192.168.99.100  
container ip：172.17.0.3
```

------

下面的是打包和拉取容器到阿里云的方法：

登录阿里云docker registry:

$ sudo docker login --username=laopo890220 [registry.cn-hangzhou.aliyuncs.com](http://registry.cn-hangzhou.aliyuncs.com/)

登录registry的用户名是您的阿里云账号全名，密码是您开通namespace时设置的密码。

你可以在镜像管理首页点击右上角按钮修改docker login密码。

从registry中拉取镜像：

$ sudo docker pull [registry.cn-hangzhou.aliyuncs.com/gaven/nginx-mysql-php7:[镜像版本号\]](http://registry.cn-hangzhou.aliyuncs.com/gaven/nginx-mysql-php7:[镜像版本号])

将镜像推送到registry：

$ sudo docker login --username=laopo890220 [registry.cn-hangzhou.aliyuncs.com](http://registry.cn-hangzhou.aliyuncs.com/)

$ sudo docker tag [ImageId] [registry.cn-hangzhou.aliyuncs.com/gaven/nginx-mysql-php7:[镜像版本号\]](http://registry.cn-hangzhou.aliyuncs.com/gaven/nginx-mysql-php7:[镜像版本号])

$ sudo docker push [registry.cn-hangzhou.aliyuncs.com/gaven/nginx-mysql-php7:[镜像版本号\]](http://registry.cn-hangzhou.aliyuncs.com/gaven/nginx-mysql-php7:[镜像版本号])

其中[ImageId],[镜像版本号]请你根据自己的镜像信息进行填写。