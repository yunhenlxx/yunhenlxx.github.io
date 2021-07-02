---
title: XXE
toc: false
date: 2021-03-25 15:24:34
description: 外部实体注入
tags: Web安全
categories: Web安全
---

## 0x01 XXE

外部实体注入

XXE攻击:

* XML声明
* DTD模版
* XML，模版中的数值

XML是用来存储数据的

## 0x02 XXE原理

PHP中存在一个叫做simplexml_load_string的函数，这个函数是将XML转化为对象

