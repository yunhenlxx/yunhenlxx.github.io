---
title: XSS
toc: false
date: 2021-03-24 15:53:47
description: XSS
tags: Web安全
categories: Web安全
---

## 0x01 XSS原理

跨站脚本攻击

前端代码注入，用户输入的数据被当作前端代码执行

窃取Cookie，利用Cookie登录

## 0x02 XSS分类

- 反射型XSS

- - 一次性的，必须是用户自己传参了恶意代码才能触发

- 存储型XSS

- - 东西会被存储，页面输出时会被当作前端代码执行

- DOM型XSS

## 0x03 JS触发方式

- `<script>alert(1)</script>` 

- 伪协议触发

- - javascript:alert(1)

- 事件触发

- - onload=alert(1)
  - onerror=alert(1)

## 0x04 反射型XSS

见框就插

## 0x05 存储型XSS

### 存储型XSS会出现的地方

- 任何可能插入数据库的地方
- 用户注册的时候
- 留言板
- 上传文件的时候
- (管理员可见的)报错信息
- ......

用户的输入，一般控制的很严格

系统的获取，一般控制的不严格

防御：

- 正则表达式过滤
- html实体编码 => `htmlspecialchars函数` 

没有办法绕过

- 找一个不编码的地方
- 编码又解码

## 0x06 DOM型XSS

DOM就是通过JS操作浏览器。通过JS处理后才产生的XSS

页面：动态页面、静态页面 （伪静态）

伪静态：动态页面伪装成静态，让黑客不来攻击。静态页面无法达成厉害的攻击

document.write()

document.cookie