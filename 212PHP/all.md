

[TOC]



# PECL - php扩展安装

## apcu安装

```shell
$ sudo apt-get install php7.x-dev
$ pecl channel-update pecl.php.net
$ pecl install apcu

$ php -i  // 查看加载的php.ini位置
```

# Composer 安装

## 安装指定版本

````sh
$ php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
$ php -r "if (hash_file('sha384', 'composer-setup.php') === '756890a4488ce9024fc62c56153228907f1545c228516cbf63f885e036d37e9a59d27d63f46af1d4d07ee0f76181c7d3') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
$ php composer-setup.php --version=1.7.1
All settings correct for using Composer
Downloading...

Composer (version 1.7.1) successfully installed to: //composer.phar
Use it: php composer.phar  
$ php -r "unlink('composer-setup.php');"
$ mv composer.phar /usr/bin/composer

$ apt-get install php-pear
````



## composer 中版本号使用

~和^的意思很接近，在x.y的情况下是一样的都是代表x.y <= 版本号 < (x+1).0，但在版本号是x.y.z的情况下有区别，举个例子：

~1.2.3 代表  1.2.3 <= 版本号 < 1.3.0

^1.2.3 代表  1.2.3 <= 版本号 < 2.0.0



# 执行ssh-add时出现Could not open a connection to your authentication agent

先执行  eval `ssh-agent`  （是～键上的那个`） 再执行 ssh-add ~/.ssh/rsa成功ssh-add -l 就有新加的rsa了