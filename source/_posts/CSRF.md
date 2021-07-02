---
title: CSRF
toc: false
date: 2021-03-24 15:54:30
description: CSRF
tags: Web安全
categories: Web安全
---

## 0x01 CSRF原理

跨站请求伪造，就是访问恶意网站时运行恶意网站上加载的JS，然后对一个已经登录的正常网站发送数据包，达到篡改信息、修改配置等功能。

## 0x02 CSRF必要条件

- 没有Token防御
- 知道目标站点需要传的恶意数据

## 0x03 防御手段

### Token本质

一串无规律的随机值

### Token机制

登录网站后，有Token机制的服务器会生成一串包含着Token的Cookie。

如果网页访问没有Token基本上就可以认定它存在CSRF，校验是必要的，必须要将CSRF打出来才能证明它存在

## 0x04 流程

- 利用Burp快速生成CSRF攻击数据包，将之复制到一个html文件中就可以让用户访问进行攻击

- 将生成的攻击数据包的点击生效修改成延时100ms自动执行submitRequest函数

- - setTimeout(submitRequest(),100)，加在JS最后即可
  - 利用iframe嵌入搭建的网站，使人访问即可访问到CSRF的html文件
    - `<iframe src="***.html" height=0 width=0 />`