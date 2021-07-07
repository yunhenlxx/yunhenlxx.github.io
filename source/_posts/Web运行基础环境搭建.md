---
title: Web运行基础环境搭建
toc: false
date: 2021-07-07 20:08:30
description: Web运行基础环境搭建
tags: Linux
categories: Linux
---

## 一、搭建LNMP环境

### 0x01 环境

本安装过程使用以下版本：

* Ubuntu： 20.04 64位
* Nginx：1.18.0
* MySQL： 8.0.25
* PHP：7.4.3

### 0x02 安装

```
// 更新Linux系统套件
$ apt update
$ apt upgrade

// 安装Nginx
$ apt install nginx
$ nginx -v
$ systemctl status nginx

// 安装MySQL
$ apt install mysql-server
$ mysql -V
$ systemctl status mysql

// 安装PHP和php-fpm，Nginx本身不能解析PHP，需要php-fpm
$ apt install php php-fpm
$ php -v
$ systemctl status php-fpm
```

### 0x03 配置

```
// 配置Nginx，备份Nginx服务器配置文件
$ cp /etc/nginx/sites-available/default /etc/nginx/sites-available/default.bak
// 修改配置文件，如下去掉注释
$ vim /etc/nginx/sites-available/default
	location ~ \.php$ {
  		include snippets/fastcgi-php.conf;
   		# With php-fpm (or other unix sockets):
    	fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
    }
// 重启Nginx服务
$ systemctl restart nginx

// 配置MySQL，增加一个root用户
$ mysql
$ ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'root';
```

### 0x04 Test

由`/etc/nginx/sites-available/default`文件可知，/var/www/html是网站根目录

1. 新建`index.php`，输入

```
<?php
	echo phpinfo();
?>
```

访问 虚拟机IP/index.php，如果看到phpinfo的信息就说明LNMP环境搭建完成。

2. 安装PHP的扩展`php-mysqli`，在根目录下新建一个sql.php，输入

```
<?php
$servername = "localhost";
$username = "root";
$password = "root";

// 创建连接
$conn = new mysqli_connect($servername, $username, $password);

// 检测连接并新建数据库myDB
if ($conn) {
		echo "连接成功";
    $sql = "CREATE DATABASE myDB";
    if (mysqli_query($conn, $sql)) {
        echo "数据库创建成功";
    } else {
        echo "Error creating database: " . mysqli_error($conn);
    }
} else {
		echo "连接失败";
}

// 关闭连接
$conn->close();
?>
```

访问 虚拟机IP/sql.php，如果打印出连接成功和数据库创建成功字样并且查看数据库时多出一个myDB的数据库，则说明能够使用PHP连接数据库并执行MySQL语句

## 二、搭建LAMP环境

### 0x01 环境

本安装过程使用以下版本：

* Ubuntu： 20.04 64位
* Apache：2
* MySQL： 8.0.25
* PHP：7.4.3

### 0x02 安装

```
// 安装Apache2
$ apt install apache2 libapache2-mod-php

// 安装MySQL
$ apt install mysql-server
$ mysql -V
$ systemctl status mysql

// 安装PHP
$ apt install php
$ php -v
```

### 0x03 配置

```
// 配置MySQL，增加一个root用户
$ mysql
$ ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'root';
```

### 0x04 Test

和Nginx一样，`/var/www/html`是网站根目录,新建`index.php`，输入

```
<?php
	echo phpinfo();
?>
```

访问 虚拟机IP/index.php，如果看到phpinfo的信息就说明LNMP环境搭建完成。

## 三、服务器加固

### 用户账号安全加固

1. 设定密码策略

```
#!/bin/bash
# Function: 实现对用户密码策略的设定，如密码最长有效期等
read -p  "设置密码最多可多少天不修改：" A
read -p  "设置密码修改之间最小的天数：" B
read -p  "设置密码最短的长度：" C
read -p  "设置密码失效前多少天通知用户：" D
sed -i '/^PASS_MAX_DAYS/c\PASS_MAX_DAYS   '$A'' /etc/login.defs
sed -i '/^PASS_MIN_DAYS/c\PASS_MIN_DAYS   '$B'' /etc/login.defs
sed -i '/^PASS_MIN_LEN/c\PASS_MIN_LEN     '$C'' /etc/login.defs
sed -i '/^PASS_WARN_AGE/c\PASS_WARN_AGE   '$D'' /etc/login.defs
echo "已设置好密码策略......"
```

2. 对用户密码的强度做设定
3. 对用户的登陆次数进行限制
4. 禁止root用户远程登陆
5. 设置账户超时时间
6. 设置只有指定用户组才能切换到root用户

### 网络服务安全加固

1. 只对外开放所需要的服务，关闭所有不需要的服务
2. 将不同的服务分布在不同的主机上面

### 系统设置安全加固

1. 系统关闭ping命令
2. 限制控制台的使用
3. 禁止IP源路径路由

### 文件系统安全加固

1. 去掉不必要的文件权限
2. 定期备份
