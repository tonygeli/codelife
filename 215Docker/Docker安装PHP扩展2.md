# Docker 中的 PHP 安装扩展插件

### 1. PHP 源码

为了保证 Docker 镜像尽量小，PHP 的源文件是以压缩包的形式存在镜像中，官方提供了 `docker-php-source` 快捷脚本，用于对源文件压缩包的解压（extract）及解压后的文件进行删除（delete）的操作。

示例：

```javascript
FROM php:7.1-apache
RUN docker-php-source extract \
    # 此处开始执行你需要的操作 \
    && docker-php-source delete
```

**注意：一定要记得删除，否则解压出来的文件会大大增加镜像的文件大小。**

### 2. 安装扩展

2.1. 核心扩展

这里主要用到的是官方提供的 `docker-php-ext-configure` 和 `docker-php-ext-install` 快捷脚本，如下

```javascript
FROM php:7.1-fpm
RUN apt-get update \
	# 相关依赖必须手动安装
	&& apt-get install -y \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libmcrypt-dev \
        libpng-dev \
    # 安装扩展
    && docker-php-ext-install -j$(nproc) iconv mcrypt \
    # 如果安装的扩展需要自定义配置时
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd
```

**注意**：这里的 `docker-php-ext-configure` 和 `docker-php-ext-install` 已经包含了 `docker-php-source` 的操作，所有不需要再手动去执行。

2.2. PECL 扩展

因为一些扩展并不包含在 PHP 源码文件中，所有需要使用 [PECL](https://secure.php.net/manual/zh/install.pecl.intro.php)（PHP 的扩展库仓库，通过 [PEAR](http://pear.php.net/) 打包）。用 `pecl install` 安装扩展，然后再用官方提供的 `docker-php-ext-enable` 快捷脚本来启用扩展，如下示例

```javascript
FROM php:7.1-fpm
RUN apt-get update \
	# 手动安装依赖
	&& apt-get install -y libmemcached-dev zlib1g-dev \
	# 安装需要的扩展
   && pecl install memcached-2.2.0 \
   # 启用扩展
   && docker-php-ext-enable memcached
```

2.3. 其它扩展

一些既不在 PHP 源码包，也不再 PECL 扩展仓库中的扩展，可以通过下载扩展程序源码，编译安装的方式安装，如下示例：

```javascript
FROM php:5.6-apache
RUN curl -fsSL 'https://xcache.lighttpd.net/pub/Releases/3.2.0/xcache-3.2.0.tar.gz' -o xcache.tar.gz \
    && mkdir -p xcache \
    && tar -xf xcache.tar.gz -C xcache --strip-components=1 \
    && rm xcache.tar.gz \
    && ( \
        cd xcache \
        && phpize \
        && ./configure --enable-xcache \
        && make -j$(nproc) \
        && make install \
    ) \
    && rm -r xcache \
    && docker-php-ext-enable xcache
```

**注意**：官方提供的 `docker-php-ext-*` 脚本接受任意的绝对路径（不支持相对路径，以便与系统内置的扩展程序进行区分），所以，上面的例子也可以这样写：

```javascript
FROM php:5.6-apache
RUN curl -fsSL 'https://xcache.lighttpd.net/pub/Releases/3.2.0/xcache-3.2.0.tar.gz' -o xcache.tar.gz \
    && mkdir -p /tmp/xcache \
    && tar -xf xcache.tar.gz -C /tmp/xcache --strip-components=1 \
    && rm xcache.tar.gz \
    && docker-php-ext-configure /tmp/xcache --enable-xcache \
    && docker-php-ext-install /tmp/xcache \
    && rm -r /tmp/xcache
```

将以下代码保存为一份xxx.sh 并执行，即可扩展mysql、gd、phalcon

```javascript
#! /usr/bin

PHP_VERSION=7.0.10

docker run --name php \
-v /home/wwwroot:/home/wwwroot \
-v ~/php_config/php.ini:/usr/local/etc/php/php.ini \
-p 9000:9000 \
-d php:${PHP_VERSION}-fpm

docker exec -it php sed -i "s/33/2016/g" /etc/passwd
docker exec -it php sed -i "s/33/2016/g" /etc/group

docker exec -it php bash -c "set -ex \
&& cd ~ \
&& mv /etc/apt/sources.list /etc/apt/sources.list.bak \
&& { \
echo deb http://mirrors.aliyun.com/debian/ jessie main non-free contrib; \
echo deb http://mirrors.aliyun.com/debian/ jessie-proposed-updates main non-free contrib; \
echo deb-src http://mirrors.aliyun.com/debian/ jessie main non-free contrib; \
echo deb-src http://mirrors.aliyun.com/debian/ jessie-proposed-updates main non-free contrib; \
} | tee /etc/apt/sources.list \
&& apt-get update \
&& apt-get install -y git \
libpcre3-dev \
libfreetype6-dev \
libjpeg62-turbo-dev \
libmcrypt-dev \
libpng12-dev \
&& docker-php-ext-install -j$(nproc) iconv mcrypt \
&& docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
&& docker-php-ext-install -j$(nproc) gd \
&& docker-php-ext-install mysql \
&& docker-php-ext-install pdo_mysql \
&& curl -o /usr/local/etc/php/php.ini https://raw.githubusercontent.com/php/php-src/PHP-${PHP_VERSION}/php.ini-production \
&& git clone -b 2.1.x --depth=1 git://github.com/phalcon/cphalcon.git ~/cphalcon \
&& cd ~/cphalcon/ext \
&& export CFLAGS=\"-O2 -finline-functions -fvisibility=hidden\" \
&& phpize \
&& ./configure --enable-phalcon \
&& make \
&& make install \
&& docker-php-ext-enable phalcon \
&& rm -rf ~/cphalcon"

docker commit -a "technofiend <2281551151@qq.com>" -m "install gd、 phalcon、pdo_mysql、mysql extsions" php phalcon:${PHP_VERSION}-fpm

docker rm -f php
docker run --name php \
-v /home/wwwroot:/home/wwwroot \
-v ~/php_config/php.ini:/usr/local/etc/php/php.ini \
-p 9000:9000 \
-d phalcon:${PHP_VERSION}-fpm

docker exec -it php sed -i "s/33/2016/g" /etc/passwd
docker exec -it php sed -i "s/33/2016/g" /etc/group
```

# [docker 中安装PHP扩展](https://www.cnblogs.com/lglblogadd/p/9173627.html)

```javascript
可以通过两种方式实现
1.pecl pdo_msql 

方式二：
docker-php-ext-install pdo pdo_mysql
如果报  /usr/local/bin/docker-php-ext-enable: cannot create /usr/local/etc/php/conf.d/docker-php-ext-pdo_mysql.ini: Directory nonexistent    
解决方案：
   直接在/usr/local/etc/php目录下面新建 conf.d目录和对应的docker-php-ext-pdo_msql.ini文件
其中docker-php-ext-pdo_msql.ini的内容为：
extension=pdo_mysql.so
```