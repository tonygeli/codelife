# Install PHP extensions on Docker

I’m developing PHP applications on Docker containers. But there are some points you will struggle to solve even though you use official Docker images. For this time, I will introduce how we can install extensions for PHP on Docker.

### Core extensions

According to [official Docker image](https://hub.docker.com/_/php/), some `docker-php-ext-*` commands are prepared for users to install extensions easily on the containers. For example, you add the following line to Dockerfile to install `curl` extension.

```
RUN docker-php-ext-install curl
```

### Pecl extensions

Pecl extensions will be also easy to be put on the containers. If you want to install `memcached` extension, you can do that like below.

```
RUN pecl install memcached \
    && docker-php-ext-enable memcached
```

Above all, it looks like very easy steps to deal with that. But you will be in trouble when you try to install extensions which need PHP compiling options.

### Pecl extensions with Compiling options

Actually, I consumed a long time to solve this problem when I install `mailparse` extension which requires `--enable-mailparse` option to build PHP.

Finally, I got the solution. In Dockerfile of offical image, I found `$PHP_EXTRA_CONFIGURE_ARGS` .

https://github.com/docker-library/php/blob/9abc1efe542b56aa93835e4987d5d4a88171b232/7.0/alpine/Dockerfile#L121

From Docker version 1.9, `--build-arg` option is provided to define environmental variables when we build Dockerfile. So you can add compiling options like this.

```
docker build . \
    --build-arg PHP_EXTRA_CONFIGURE_ARGS="--enable-mailparse"
```

Then you put pecl installation on Dockerfile.

```
RUN pecl install mailparse \
    && docker-php-ext-enable mailparse
```