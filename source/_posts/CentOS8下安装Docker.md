---
title: CentOS8下安装Docker
toc: false
date: 2021-03-24 15:39:36
description: CentOS8下安装Docker
tags:
 - Docker
 - Linux
categories: Linux
---

## 0x01 使用存储库安装

### 设置存储库

安装yum-utils软件包并设置**稳定的**存储库。

```
$ sudo yum install -y yum-utils
$ sudo yum-config-manager \
    --add-repo \
    https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

### 安装Docker引擎

安装*最新版本*的Docker Engine和容器：

```
$ sudo yum install docker-ce docker-ce-cli containerd.io
```

如果提示接受GPG密钥，验证指纹是否匹配 060A 61C5 1B55 8A7F 742B 77AA C52F EB6B 621E 9F35，如果是，则接受。

### 启动并设置开机运行Docker

```
$ sudo systemctl start docker
$ sudo systemctl enable docker
```

### 升级Docker引擎

重新安装一下即可

## 0x02 配置镜像加速器

首先执行以下命令，查看是否在 docker.service 文件中配置过镜像地址

```
$ systemctl cat docker | grep '\-\-registry\-mirror'
```

如果以上命令没有任何输出，那么就可以在 /etc/docker/daemon.json 中写入如下内容（如果文件不存在新建该文件）：

```
{
  "registry-mirrors": [
    "https://hub-mirror.c.163.com",
    "https://mirror.baidubce.com"
  ]
}
```

重新启动服务

```
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker
```

使用`docker info` 查看是否配置成功

