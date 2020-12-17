---
title: "SSH与Pycharm免密远程登录服务器"
date: 2020-12-17T20:46:34+08:00
draft: false
toc: true
author: "彬Goh"
keywords: ["ssh免密登录","pycharm","远程登录服务器"]
description: "SSH与Pycharm免密远程登录服务器"
tags: ["软件安装"]
categories: ["软件运维"]
---

本文主要解决两个问题
1. 免密 ssh 登录远程服务器
2. pycharm 连接服务器调试
<!--more-->

## SSH免密登录
### 原理
使用了非对称加密：私钥放在自己服务器上，需要好好保存；公钥可以发给别人。流程如下所示：
![](https://tva1.sinaimg.cn/large/0081Kckwly1glr1gnta9qj30l40b8mxy.jpg)
所以想要免密连接，最重要的就是两步：

1. 本地生成一对密钥
2. 将公钥放入远程服务器的`authorized_keys`中

### 步骤
#### 1、生成一对密钥

```bash
cd ~/.ssh # 进入 .ssh 文件夹中。如果没有这个文件夹则创建它。
ssh-keygen -t rsa -C "your_email@example.com" # 之后弹出的直接按回车就行。
```

`ssh-keygen`命令重要选项如下：

| ssh-keygen命令选项 | 描述                                                         |
| ------------------ | ------------------------------------------------------------ |
| -b                 | 指定要创建的密钥中的位数。 最小位长度为768位，默认长度为2048位。 |
| -t                 | 指定要创建的密钥类型。                                       |
| -C                 | 提供新注释。                                                 |
| -f                 | 密钥文件名，默认 id_rsa 和 id_rsa.pub                        |

可以看到多了两个文件：

```bash
id_rsa  # 私钥
id_rsa.pub # 公钥
```

#### 2、将公钥放到服务器上

可以使用`scp`命令来做这件事情。

```bash
scp -P 20002 ~/.ssh/id_rsa.pub root@10x.xxx.xxx:~/.ssh/id_rsa.pub.slave1
```

`-P` 指定对方服务器端口。

`id_rsa.pub.slave1`表示将文件传输过去并且重命名（防止覆盖掉对方服务器自己的公钥）。

此时还是需要你输入密码的。

#### 3、将公钥添加进权限列表

```bash
cat id_rsa.pub.slave1 >> authorized_keys
```

直接追加写到`authorized_keys`文件后面即可。如果没有这么文件，直接自己创建。

#### 4、指定Host

有时候记住对方服务器域名也非常麻烦，所以可以用别名去登录。

在自己的`~/.ssh`下面创建名为`config`的文件，并在其中写入如下东西：

```bash
Host tencent # 别名
    HostName 10x.xxx.xxx.xxx # 对方服务器域名
    Port 22000 # 连接的端口号
    User root # 用户名称
    IdentityFile ~/.ssh/id_rsa # 密钥的位置
```

之后在自己的电脑上使用以下命令就可以直接登录啦！

```bash
ssh tencent
```

#### 5、一些问题

如果发现不能成功登陆的话，请修改`authorized_keys`的权限

```bash
# 查看权限
$ ls -l authorized_keys
# 修改权限
$ chmod 600 authorized_keys
```



## Pycharm 远程连接服务器

### 连接服务器

#### 1、添加一个Server，名字随便取

![](https://tva1.sinaimg.cn/large/0081Kckwly1glr58z3hf0j316y0u0jw0.jpg)

#### 2、连接服务器

![image-20201217190620808](https://tva1.sinaimg.cn/large/0081Kckwly1glr59p398yj31900nm75o.jpg)

点击箭头所指的地方，可以输入对方服务器的域名、端口、密码。（也可以使用密钥方式登录）

`Root path` 指的是远程服务器的那一个目录作为你这里看到的`root`

#### 3、映射

![image-20201217190847973](https://tva1.sinaimg.cn/large/0081Kckwly1glr5a7wk4dj319m0h8mza.jpg)

将服务器的某个文件映射到本地的文件夹下。设置完成后点`OK`即可。

#### 4、文件传输

![image-20201217191103278](https://tva1.sinaimg.cn/large/0081Kckwly1glr5ahtuttj31010u0n6x.jpg)

在该项目的任何一个文件上右键可以看到`Deployment`，选择对应操作即可实现同步。

### python解释器

解释器也就是用哪个`python`来运行代码。包括版本号和安装的一些依赖等。

#### 1、添加解释器

![image-20201217201236772](https://tva1.sinaimg.cn/large/0081Kckwly1glr5aykj5pj31m00qiaer.jpg)

点击右侧的齿轮，然后点击`Add`。使用`SSH Interpreter`。

![image-20201217203003470](https://tva1.sinaimg.cn/large/0081Kckwly1glr5azfij7j319l0u00wi.jpg)

这里重新设置`configuration`

> 直接复用很容易出错。

#### 2、设置解释器映射

![image-20201217203142710](https://tva1.sinaimg.cn/large/0081Kckwly1glr5bbod8bj31ao0te76t.jpg)

`Interpreter` 是服务器`python`的安装路径。`Sync folers`是需要同步的目录。就是本机项目目录和服务器项目目录。

设置完成之后，就可以直接在本机连接服务器跑了。本机的修改也会实时同步到服务器上。



## 参考

[SSH免密登录原理及实现](https://blog.csdn.net/qq_26907251/article/details/78804367)


















