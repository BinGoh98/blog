---
title: "vim配置入门"
date: 2020-10-06T22:28:54+08:00
draft: false
toc: true
author: "彬Goh"
keywords: ["vim","vim插件"]
description: "vim配置入门"
tags: ["vim","软件安装"]
categories: ["软件&运维"]
---

在终端操作vim是常有的事情。vim也以它弹性的配置而受到非常多的人的欢迎。尤其对于某些大神来说，直接就用vim作生产力工具了。如何让我们的vim使用起来非常舒服呢。

## Options

vim的初始化配置都在`$HOME/.vimrc`中。（Vim resource configuration）

打开它，并写入新配置。（初次打开可能为空白）

[链接: Top 50 Vim Configuration Options](https://www.shortcutfoo.com/blog/top-50-vim-configuration-options/)

```
" 打开语法高亮
syntax on

set smarttab " 按tab时，用tabstop个空格代替
set tabstop=4 " tab = 4 空格
set expandtab " 使用空格代替tab

set hlsearch " 搜索时高亮
"set ignorecase " 搜索时忽略大小写
set smartcase " 如果搜索含大写字母，则自动变为大小写敏感

set scrolloff=10 " 光标上下有多少行保留

set mouse=a " 可以使用鼠标
set number " 有行数提示
set cursorline " 光标所在行提示

" 光标形态变化；insert时为一根竖线
let &t_SI = "\<Esc>]50;CursorShape=1\x7"
let &t_SR = "\<Esc>]50;CursorShape=2\x7"
let &t_EI = "\<Esc>]50;CursorShape=0\x7"
```

## Plugin

推荐使用`vim-plug`作插件管理工具。[Github地址：vim-plug](https://github.com/junegunn/vim-plug)

在mac或者linux下用如下命令下载：

```bash
curl -fLo ~/.vim/autoload/plug.vim --create-dirs \
    https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```

然后就可以把要下载的写入`~/.vimrc`中了。

```
" vim-plug
call plug#begin('~/.vim/plugged') " 用begin和end包裹

" On-demand loading
Plug 'preservim/nerdtree'

call plug#end()
```

最后输入`:PlugInstall`来安装。

### 推荐的插件

[Github地址：nerdtree](https://github.com/preservim/nerdtree) 可以展示文件目录

```
" nerdtree
let g:NERDTreeDirArrowExpandable = '▸'
let g:NERDTreeDirArrowCollapsible = '▾'
map <C-n> :NERDTreeToggle<CR> " 按ctrl-n 自动关闭和打开nerdtree

" git 目录指示
let g:NERDTreeIndicatorMapCustom = {
    \ "Modified"  : "✹",
    \ "Staged"    : "✚",
    \ "Untracked" : "✭",
    \ "Renamed"   : "➜",
    \ "Unmerged"  : "═",
    \ "Deleted"   : "✖",
    \ "Dirty"     : "✗",
    \ "Clean"     : "✔︎",
    \ "Unknown"   : "?"
    \ }

```

[Auto Pairs](https://github.com/jiangmiao/auto-pairs) 自动匹配括号







