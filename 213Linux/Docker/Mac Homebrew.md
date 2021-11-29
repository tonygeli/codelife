



```
FROM composer:2.0

RUN docker-php-source extract \
    && apk add --no-cache --virtual .phpize-deps-configure $PHPIZE_DEPS \
    && pecl install channel://pecl.php.net/apcu-5.1.20 \
    && pecl install channel://pecl.php.net/yac-2.3.0 \
    && docker-php-ext-enable apcu \
    && docker-php-ext-enable yac \
    && apk del .phpize-deps-configure \
    && docker-php-source delete
```



```bash
docker run --rm -it -v "$PWD:/app" -v "$HOME/.ssh:/root/.ssh" xiongyouli/composer composer  install
```





