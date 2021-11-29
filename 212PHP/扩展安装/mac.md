[TOC]

## Mac OS

```
macOS Mojave
10.14.6
```

## 安装PHP7.4

```
brew install php@7.4

Now homebrew-php has been migrated to homebrew-core and by default, PECL should be installed along with your PHP.
现在，homebrew php已经迁移到homebrew core，默认情况下，PECL应该与php一起安装。
To install PHP extensions, you need to use PECL as a recommended way. 
要安装PHP扩展，建议使用PECL。
For example: pecl install apc or pecl install xdebug.

查找apcu扩展
pecl search apcu
sudo pecl install APCu
```

## Mac环境目录

```
// 通过phpinfo()打印页面

1. php.ini文件目录
Loaded Configuration File
/usr/local/etc/php/7.1/php.ini

2.扩展存放目录
extension_dir
/usr/local/lib/php/pecl/20160303
```

## 安装apcu

```
// 1.下载源码  
git clone https://github.com/krakjoe/apcu
// 2.编译安装  
cd apcu
phpize
./configure
make
sudo make install
// 3. 修改php.ini文件
extension=apcu.so


Installing shared extensions:     /usr/lib/php/extensions/no-debug-non-zts-20160303/
```

## phpize编译提示Cannot find autoconf解决办法

```
$ brew install autoconf
```

## make命令 ./apc.h:67:10: fatal error: 'php.h' file not found

```
 open /Library/Developer/CommandLineTools/Packages/macOS_SDK_headers_for_macOS_10.14.pkg
```

### 错误

```
装不了php的扩展，make install失败

RudonMacBook:igbinary-master rudon$ make install
Installing shared extensions:   /usr/lib/php/extensions/no-debug-non-zts-20131226/
cp: /usr/lib/php/extensions/no-debug-non-zts-20131226/#INST@12567#: Operation not permitted
make: *** [install-modules] Error 1

cp: /usr/lib/php/extensions/no-debug-non-zts-20121212/#INST@17000#: Operation not permitted
```

原来是OSX 10.11 El Capitan（或更高）新添加了一个新的安全机制叫系统完整性保护System Integrity Protection (SIP)，所以对于目录
/System
/sbin
/usr
不包含(/usr/local/)
仅仅供系统使用，其它用户或者程序无法直接使用，而我们的/usr/lib/php/extensions/刚好在受保护范围内

### 解决

所以解决方法就是禁掉SIP保护机制，步骤是：

- 重启系统
- 按住Command + R  （重新亮屏之后就开始按，象征地按几秒再松开，出现苹果标志，ok）
- 菜单“实用工具” ==>> “终端” ==>> 输入**csrutil disable**；执行后会输出：Successfully disabled System Integrity Protection. Please restart the machine for the changes to take effect.
- 再次重启系统

禁止掉SIP后，就可以顺利的安装了，当然装完了以后你可以重新打开SIP，方法同上，只是命令是csrutil enable

### 错误'pcre2.h' file not found

```
/opt/homebrew/Cellar/php@7.4/7.4.16/include/php/ext/pcre/php_pcre.h:25:10: fatal error: 'pcre2.h' file not found
#include "pcre2.h"
         ^~~~~~~~~
1 error generated.
make: *** [php_apc.lo] Error 1
ERROR: `make' failed
```

./configure --with-php-config=/opt/homebrew/opt/php@7.4/bin/php-config