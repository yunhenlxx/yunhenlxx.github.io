---
title: CentOS8下搭建LNMP
toc: false
date: 2021-03-24 15:36:35
description: CentOS8下搭建LNMP
tags: Linux
categories: Linux
---

## 安装和配置 Nginx

1. 安装 Nginx

以安装 Nginx 1.18.0 为例，可通过 [Ngingx 官方安装包](http://nginx.org/packages/centos/8/x86_64/RPMS/?spm=a2c4g.11186623.2.31.557423bfYPMd6u) 获取适用于 CentOS 8的更多版本。

```
dnf -y install http://nginx.org/packages/centos/8/x86_64/RPMS/nginx-1.18.0-1.el8.ngx.x86_64.rpm
```

2. 查看 Nginx 版本

```
nginx -v
```

3. 查看 Nginx 配置文件路径

```
cat /etc/nginx/nginx.conf
```

include 配置项的 /etc/nginx/conf.d/*.conf 即为 Nginx 配置文件的默认路径

4. 在配置文件默认路径下进行备份

```
cd /etc/nginx/conf.d
cp default.conf *.conf
```

5. 打开 *.conf 文件

```
vim *.conf
```

6. 编辑 default.conf 文件

- 在 location 的 index 项中添加 index.php

- 删除 location ~ .php$ 大括号前的 #，并修改以下配置项：

- - 修改 root 项为网站根目录，即 location 中的 root 项，本文以 /usr/share/nginx/html; 为例
  - 修改 fastcgi_pass 项为 unix:/run/php-fpm/www.sock;
  - 将 fastcgi_param SCRIPT_FILENAME 后的 /scripts$fastcgi_script_name; 替换为 $document_root$fastcgi_script_name;

```
location ~ \.php$ {
    root          /usr/share/nginx/html;
    fastcgi_pass  unix:/run/php-fpm/www.sock;
    fastcgi_index index.php;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include       fastcgi_params;
}
```

7. 启动Nginx并设置为开机自启动并查看状态

```
systemctl start nginx
systemctl enable nginx
systemctl status nginx
```

## 安装和配置MySQL

1. 安装 MySQL

```
dnf -y install @mysql
```

2. 查看 MySQL 版本

```
mysql -V
```

3. 启动 MySQL 并设置为开机自启动并查看状态

```
systemctl start mysqld
systemctl enable mysqld
systemctl status mysqld
```

4. 执行 MySQL 安全性操作并设置密码

```
mysql_secure_installation
```

按照以下步骤，执行对应操作：

- 输入 y 并按 **Enter** 开始相关配置。

- 选择密码验证策略强度，建议选择高强度的密码验证策略。输入 2 并按 **Enter**。

- - 0：表示低。
  - 1：表示中。
  - 2：表示高。

- 设置 MySQL 密码并按 **Enter** ，输入密码默认不显示。

- 再次输入密码并按 **Enter**，输入 y 确认设置该密码。

- 输入 y 并按 **Enter**，移除匿名用户。

- 设置是否禁止远程连接 MySQL：

- - 禁止远程连接：输入 y 并按 **Enter**。
  - 允许远程连接：输入 n 并按 **Enter**。

- 输入 y 并按 **Enter**，删除 test 库及对 test 库的访问权限。

- 输入 y 并按 **Enter**，重新加载授权表。

## 安装和配置PHP

1. 添加并更新 epel 源

```
dnf -y install epel-release
dnf update epel-release
```

2. 删除缓存的无用软件包并更新软件源

```
dnf clean all
dnf makecache
```

3. 安装 remi 源

```
dnf -y install https://rpms.remirepo.net/enterprise/remi-release-8.rpm
```

4. 启动 PHP 7.4 模块

```
dnf module install php:remi-7.4
```

5. 安装所需 PHP 对应模块

```
dnf install php php-curl php-dom php-exif php-fileinfo php-fpm php-gd php-hash php-json php-mbstring php-mysqli php-openssl php-pcre php-xml libsodium
```

6. 查看 PHP 版本

```
php -v
```

7. 打开 www.conf 文件

```
vi /etc/php-fpm.d/www.conf
```

8. 将 user = apache 及 group = apache 修改为 user = nginx 及 group = nginx
9. 启动 PHP-FPM 并设置为开机自启动并查看状态

```
systemctl start php-fpm
systemctl enable php-fpm
systemctl status php-fpm
```

## 验证环境配置

/usr/share/nginx/html 为 Nginx 中已配置的网站根目录

1. 创建测试文件

```
echo "<?php phpinfo(); ?>" >> /usr/share/nginx/html/index.php
```

2. 在本地浏览器中访问服务器公网IP，查看环境配置是否成功

```
http://云服务器实例的公网 IP/index.php
```

