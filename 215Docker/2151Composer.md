Composer镜像

Dockerfile

```
FROM composer:2.0

COPY --from=mlocati/php-extension-installer /usr/bin/install-php-extensions /usr/local/bin/

RUN install-php-extensions apcu-5.1.20 yac-2.3.0 mbstring sockets bcmath bz2 calendar exif gettext http igbinary mcrypt msgpack mysqli pcntl pdo_mysql redis sysvmsg sysvsem sysvshm zip gmp memcache memcached shmop gd
```



创建Dockerfile

```
docker build -t composer .
Usage:  docker build [OPTIONS] PATH | URL | -

生成image名为composer的镜像
```





docker run --rm -it -v "$PWD:/app" -v "$HOME/.ssh:/root/.ssh" composer composer "$@" install

