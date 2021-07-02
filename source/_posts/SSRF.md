---
title: SSRF
toc: false
date: 2021-03-25 14:37:30
description: SSRF
tags: Web安全
categories: Web安全
---

## 0x01 SSRF

服务器端请求伪造

* 让目标站点发起网络请求
* 能访问公网或内网

## 0x02 挖掘

可以发起网络请求的地方都可能存在SSRF

* 看传参GET和POST是否含有协议判断是否存在SSRF
  * http://
  * url=http://

* 查看本地开放了什么端口
  * dict://127.0.0.1
* 探测文件内容
  * file:///

