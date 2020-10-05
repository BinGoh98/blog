---
title: "Hugo搭建静态博客与服务器部署"
date: 2020-10-05T13:04:16+08:00
draft: false
toc: true
author: "彬Goh"
keywords: ["hugo","博客"]
description: "Hugo搭建静态博客与服务器部署"
tags: ["hugo","软件安装"]
categories: ["软件安装"]
---
   
   疫情期间，比较清闲，所以抽时间自学了一些计算机的知识。后来运气非常好地进入了某大厂实习，也运气非常好地实习转正了。考虑到个人发展与家庭的因素，最后决定放弃保研，提早成为一名社畜。不过之前准备实习面试，是直接背面经的，没啥实操经验，知道自己底子特别薄弱。所以趁着实习结束，还未正式入职的这段时间，好好补一补计算机基础。并将我学习到的一些东西记录在这里，希望可以养成及时记录的好习惯，以后无事也可翻看翻看。
   
   ## Hugo简介
   
   Hugo是由Go语言实现的静态网站生成器，据称是「The world's fastest framework for building websites」。静态网页，可以理解为是由一系列`html`文件和`css`,`js`等静态文件构成的。`Hugo`负责把我们写的`markdown`转换成`html`的形式，我们直接拿着它生成的文件就可以直接发布了。
   
   ## Hugo安装
   
   ### 1. 二进制安装
   
   这里强推`homebrew`！Mac直接可以用以下命令安装：
   
   ```bash
   brew install hugo
   ```
   
   每次使用brew的时候都会自动更新，需要等待很长时间，可以配置环境变量关闭自动更新。
   
   #### 临时关闭
   
   ```bash
   export HOMEBREW_NO_AUTO_UPDATE=true # shell 关闭之后就不生效了
   ```
   
   #### 永久关闭
   
   ```bash
   # bash
   vi ~/.bash_profile
   # 添加
   export HOMEBREW_NO_AUTO_UPDATE=true
   source ~/.bash_profile
   
   
   # zsh
   vi ~/.zshrc
   # 添加
   export HOMEBREW_NO_AUTO_UPDATE=true
   source ~/.zshrc
   ```
   
   ### 2. 源码安装
   
   安装之前需要已经装好`Git` 和 `Go`
   
   - [Git官网](https://git-scm.com)
   - [Go官网](https://golang.org/dl/)「最好装最新版本」
   
   ```bash
   mkdir $HOME/src
   cd $HOME/src
   git clone https://github.com/gohugoio/hugo.git
   cd hugo
   go install
   ```
   
   ## Hugo快速开始
   
   输入 `hugo version`，如果显示出版本号，就说明你的`hugo`已经安装好了！Just enjoy it!
   
   ### 生成站点
   
   期望在当前目录下快速生成站点
   
   ```bash
   hugo new site personal_blog
   
   # output
   Congratulations! Your new Hugo site is created in /Users/bingoh/BlogContent/personal_blog.
   
   Just a few more steps and you're ready to go:
   
   1. Download a theme into the same-named folder.
      Choose a theme from https://themes.gohugo.io/ or
      create your own with the "hugo new theme <THEMENAME>" command.
   2. Perhaps you want to add some content. You can add single files
      with "hugo new <SECTIONNAME>/<FILENAME>.<FORMAT>".
   3. Start the built-in live server via "hugo server".
   
   Visit https://gohugo.io/ for quickstart guide and full documentation.
   ```
   
   然后就可以看到当前目录下，多了一个名为`personal_blog`的文件。
   
   ```bash
   # 查看文件结构
   personal_blog
   ├── archetypes
   │   └── default.md
   ├── config.toml # 更改配置
   ├── content # 博客正文的内容
   ├── data
   ├── layouts # html等展示文件
   ├── static # js/css/font等静态文件
   └── themes # 下载的主题
   ```
   
   ### 下载主题
   
   [Hugo Themes](https://themes.gohugo.io/)
   
   在这里下载你喜欢的主题吧！
   
   主题有自己的下载说明，以我下载的`even`主题为例。
   
   ```bash
   cd personal_blog
   git clone https://github.com/olOwOlo/hugo-theme-even themes/even
   ```
   
   下载完成之后，发现`themes`文件夹下多了个`even`文件夹，里面有很多文件。一般作者都会在`README.md`中作详细说明，这里就不多赘述。
   
   > 比如 even 要求将 themes/even/exampleSite/config.toml 文件复制到站点目录下！
   
   ### 新建PAGE
   
   在`personal_blog`文件夹下
   
   ```bash
   hugo new post/your_file_name.md
   
   # example
   hugo new post/first.md
   ```
   
   发现`content`下面多了一些文件
   
   ```bash
   content
   └── post
       └── first.md
       
       
   # 打开first.md
   # output
   ---
   title: "First"
   date: 2020-10-05T12:32:29+08:00
   draft: true
   ---
   在这里输入你的正文内容
   ```
   
   `title`是文章标题
   
   `date`是文章创建的日期
   
   `draft`表示是否是草稿
   
   更多的参数，可以到`themes/even/archetypes/default.md`中查看并设置！
   
   ## 运行hugo
   
   ```bash
   # personal_blog 目录下运行
   hugo server --theme=even --buildDrafts
   
   # output
   ... 省略
   Web Server is available at http://localhost:1313/ (bind address 127.0.0.1)
   ...
   ```
   
   浏览器打开 `http://localhost:1313/`，就可以看到新的站点啦！
   
   
   
   ## 部署hugo
   
   我之前自己买了腾讯云的服务器和域名，并且关联好了域名和服务器，所以就直接部署在自己的服务器上了。如果没有服务器的话，直接部署在`github`上也是完全ok的！
   
   我使用`宝塔面板`来运维！真的是超级方便，一键部署，适合小白。[宝塔面板](https://www.bt.cn)
   
   按照指示在自己的服务器上部署好了之后，就可以直接用了！
   
   `网站` -->  `添加站点`
   
   <img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gjed4rt5clj31900u0gs8.jpg" style="zoom:50%;" />
   
   ```bash
   hugo --theme=even -b https://www.binfromfd.com/ # 发现生成 public 文件夹
   ```
   
   将生成的`public`文件夹，复制到网站的根目录下，访问你的网站！就可以看到啦！
   
   ### 自动发布
   
   将自己的代码放到`github`仓库中，然后在服务器上开一个定时任务去拉取代码。
   
   宝塔：`计划任务` --> `添加计划任务`
   
   添加一个`shell`脚本
   
   ```bash
   cd ~/blog # 已经和git仓库关联
   git pull
   cp -rf ~/blog/public/. /www/wwwroot/www.binfromfd.cn/
   ```
   
   之后，在自己电脑上写好代码之后
   
   ```bash
   hugo --theme=even --baseUrl="http://www.binfromfd.cn/"
   ```
   
   如果md开头有`draft=true`，说明是草稿文件，这里是不会生成文章的。可以令`draft=false`。
   
   最后`push`到`git`上就可以了。可以等定时任务的时间到了再执行，也可以直接在宝塔面板中执行脚本！