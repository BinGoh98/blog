---
title: "开发相关"
date: 2020-10-04T13:01:27+08:00
draft: false
tags: ["preview", "中文", "tag-1"]
categories: ["中文"]
---

### Homebrew安装

[mac下镜像飞速安装Homebrew教程](https://zhuanlan.zhihu.com/p/90508170)

#### 关闭自动更新

```bash
vi ~/.bash_profile

// 添加
export HOMEBREW_NO_AUTO_UPDATE=true

source ~/.bash_profile
```
<!--more-->
### docker

```bash
brew cask install docker
```

#### 加速 

在` /etc/docker/daemon.json` 中写入如下内容

（如果文件不存在请新建该文件，如果permission denial 请直接sudo）

```json
{
  "registry-mirrors": [
    "https://dockerhub.azk8s.cn",
    "https://reg-mirror.qiniu.com",
    "https://registry.docker-cn.com"
  ]
}
```

#### search

```bash
docker search mysql
```

#### pull

```bash
docker pull mysql
```



### qlmarkdown

```bash
brew cask install qlmarkdown
```



### Go

#### Go 源替换

[go proxy](https://github.com/goproxy/goproxy.cn/blob/master/README.zh-CN.md)

```bash
go env -w GO111MODULE=on
go env -w GOPROXY=https://goproxy.cn,direct

export GO111MODULE=on
export GOPROXY=https://goproxy.cn
```

#### GoLang 




### item2

[iTerm2 + Oh My Zsh 打造舒适终端体验](https://zhuanlan.zhihu.com/p/37195261)

#### Connection refused

```bash
正在解析主机 raw.githubusercontent.com (raw.githubusercontent.com)... 0.0.0.0
正在连接 raw.githubusercontent.com (raw.githubusercontent.com)|0.0.0.0|:443... 失败：Connection refused。
```

出现以上问题，需要科学上网。

[iP或域名查询](https://site.ip138.com/)

![Local Picture](https://tva1.sinaimg.cn/large/007S8ZIlgy1gjdbjelo8yj30lm0kkhdr.jpg)
```bash
sudo vi /etc/hosts

// 选择一个域名添加
// eg
151.101.76.133 raw.githubusercontent.com
```



### Github

[GitHub Docs](https://docs.github.com/cn)

#### SSH key

1. 生成SSH密钥

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

2. 添加密钥到 ssh-agent

```bash
// 后台启动ssh代理
eval "$(ssh-agent -s)"

// 打开 ~/.ssh/config 添加
Host *
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_rsa
  
// 将 SSH 私钥添加到 ssh-agent 并将密码存储在密钥链中。
ssh-add -K ~/.ssh/id_rsa
```

3. 新增 SSH 密钥到 GitHub 帐户



#### 提高下载速度

[Gitee](https://gitee.com/)















