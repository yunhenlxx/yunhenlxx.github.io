---
title: Mac下SSH闲时自动中断解决方法
toc: false
date: 2021-03-24 15:25:56
description: Mac下SSH闲时自动中断解决方法
tags: Mac
categories: Mac
---

使用Mac自带的终端SSH连接服务器的时候，只要隔一小段时间不操作的话连接就会自动中断.

解决方法：

1. 连接到服务器
2. 编辑服务端配置文件

```
vim /etc/ssh/sshd_config
```

3. 在最下面添加两行配置代码

```
ClientAliveInterval 30
ClientAliveCountMax 3
```

4. 编辑客户端配置文件

```
vim /etc/ssh/ssh_config
```

5. 在Host * 下面添加两行配置代码

```
ServerAliveInterval 30
ServerAliveCountMax 3
```

6. 同步数据然后重启

```
sync		// 将数据由内存同步到硬盘
reboot		// 重启
```



