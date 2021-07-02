---
title: CentOS8下安装Node.js和npm
toc: false
date: 2021-03-24 15:37:29
description: CentOS8下安装Node.js和npm
tags: Linux
categories: Linux
---

Node.js 是一个跨平台的 JavaScript 运行环境，简单的说 Node.js 就是运行在服务端的 JavaScript。使用 Node.js，你可以构建扩展的网络应用。

NPM(Node Package Manager)是 Node.js 为了方便开发者使用和重用代码而开发的默认包管理器。它也是世界上用来发布开源 Node.js 包的最大软件仓库。

NVM (Node Version Manager) 是一个 bash 脚本，它允许你管理多个 Node.js 版本。通过 NVM，你可以安装或者卸载任何你想要使用或者测试的 Node.js 版本。

## NVM 的安装

在NVM的GitHub网站上复制安装命令，更新的话重新输一下命令即可

```
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.37.1/install.sh | bash
```

重新打开命令行即可使用nvm了

## Node.js 和 npm 的安装

运行以下命令安装最新的 Node.js 的 LTS 版本

```
nvm install --lts
```

列出所有已经安装的 Node.js 版本

```
nvm ls
```

改变当前可用的版本号

```
nvm use [版本号]
```

改变 Node.js 的默认版本

```
nvm alias default [版本号]
```