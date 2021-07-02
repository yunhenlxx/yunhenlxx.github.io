---
title: 基于Docker制作PHP镜像
toc: false
date: 2021-03-24 15:33:16
description: 基于Docker制作PHP镜像
tags: Docker
categories: Docker
---

拉取ubuntu:20.10的镜像

```
docker pull ubuntu:20.10
```

运行一个ubuntu:20.10实例

```
docker run -it ubuntu:20.10 /bin/bash
```

进入实例后运行

```
apt update
```

然后安装必要的基础软件

```
apt install -y sudo vim wget git make gcc pkg-config autoconf libxml2-dev libbz2-dev libgmp-dev openssl libssl-dev curl libcurl4-gnutls-dev libzip-dev libmemcached-dev libjpeg-dev libpng-dev libfreetype6-dev libwebp-dev sqlite3 libsqlite3-dev libtool
```

php7.4有额外的依赖项目oniguruma

```
cd
git clone https://github.com/kkos/oniguruma.git oniguruma
cd oniguruma
./autogen.sh
./configure
make
make install 
```

到php官网，找到最新版 7.4.6，并下载中国区镜像，解压至根目录

```
cd
wget https://downloads.php.net/~pollita/php-8.0.0beta4.tar.gz
tar -zxvf php-8.0.0beta4.tar.gz
```

查看最新的参数help

```
cd
php-8.0.0beta4
./configure --help
```

本次更新后参数如下

```
./configure --enable-fpm --with-openssl --with-zlib --enable-bcmath --with-bz2 --enable-calendar --with-curl --enable-exif --enable-gd --with-webp --with-jpeg --with-zlib-dir --with-freetype --with-gmp --with-mhash --enable-mbstring --with-mysqli --with-pdo-mysql --enable-shmop --enable-sockets --with-zip --enable-mysqlnd --enable-opcache --with-pear
```

然后make && make install

```
pecl channel-update https://pecl.php.net/channel.xml
```

然后安装业务所需要的扩展yaf,swoole等

依次使用pecl install 名称安装需要的扩展

```
igbinary
msgpack
memcached
mongodb
redis
swoole
yaf
grpc
```

通过php -i看到没有加载任何配置文件

```
cp /root/php-8.0.0beta4/php.ini-production /usr/local/lib/php.ini
```

随后

```
vim /usr/local/lib/php.ini
```

找到如下配置进行修改

```
expose_php = Off
max_execution_time = 60
post_max_size = 20M
upload_max_filesize = 20M
date.timezone = Asia/Shanghai
opcache.enable=1 
```

随后找地方塞入extension

```
zend_extension=opcache.so
extension=igbinary.so
extension=memcached.so
extension=mongodb.so
extension=msgpack.so
extension=redis.so
extension=swoole.so
extension=yaf.so
extension=grpc.so 
```

然后安装nodejs,参考资料 https://github.com/nodesource/distributions/blob/master/README.md

```
curl -sL https://deb.nodesource.com/setup_14.x | sudo -E bash - sudo apt-get install -y nodejs
npm config set registry https://registry.npm.taobao.org 
```

然后收尾

```
apt clean
cd
rm php-7.4.6.tar.gz
rm -rf php-7.4.6
rm -rf /tmp/pear 
```

然后退出容器 docker commit 后，导出镜像即可

```
docker commit -a "xinxu.lu <lzz@wolou-inc.com>" -m "install php with full extension support and node,npm" 4f61a96ceba4 bzinc/php80:0.0
docker save -o bzinc_php80_0_0.tar fc1814317609
```

