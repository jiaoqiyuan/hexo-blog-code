---
title: Vim插件配置
date: 2019-08-21 03:15:46
tags: 
categories: vim
photos: https://github.com/jiaoqiyuan/pics/raw/master/blog/vim-plugins/vim-pligin-title.png
---

[vim 慕课网课程](https://www.imooc.com/video/19463)，内容挺全的，这里大部分内容都是基于该视频整理的。

<!--more-->

------

## 基本配置（无插件）

编辑 `~/.vimrc` 文件：

```
" 设置使用F2进入粘贴模式
set pastetoggle=<F2>
" 设置语法高亮
syntax on
" 设置缩进
set tabstop=4
```

## vim-plug 插件安装工具

[vim-plug](https://github.com/junegunn/vim-plug)是一款 vim 插件安装工具。

![vim-plug](https://raw.githubusercontent.com/junegunn/i/master/vim-plug/installer.gif)

安装方法很简单：

```
curl -fLo ~/.vim/autoload/plug.vim --create-dirs \
    https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```

使用方法也很简单，在 `vimrc` 中添加相关配置即可，以 `vim-startify` 插件为例，在 `vimrc` 中添加如下配置内容：

```
" Specify a directory for plugins
" - For Neovim: ~/.local/share/nvim/plugged
" - Avoid using standard Vim directory names like 'plugin'
call plug#begin('~/.vim/plugged')

Plug 'mhinz/vim-startify'

call plug#end()
```

然后 `:PlugInstall` 即可，启动 vim 时可以看到欢迎界面：

![vim-startify](https://github.com/jiaoqiyuan/pics/raw/master/blog/vim-plugins/vim-starify.png)

## vim-airline 状态栏美化插件

在 `~/.vimrc` 中添加如下配置内容：

```
call plug#begin('~/.vim/plugged')

Plug 'vim-airline/vim-airline'
Plug 'vim-airline/vim-airline-themes'

call plug#end()
```

安装完效果如下：

![vim-airline](https://github.com/vim-airline/vim-airline/wiki/screenshots/demo.gif)

有很多彩色提示，比较友好。

## indentline 增加缩进线

安装方法很简单，跟上面的插件一样，在 `~/.vimrc` 中添加如下内容：

```
call plug#begin('~/.vim/plugged')

Plug 'Yggdroot/indentLine'

call plug#end()
```

效果大概是这样：

![vim-indentline](https://camo.githubusercontent.com/8e0f6822f859a9a8a7069219f6816174a4737f8e/687474703a2f2f692e696d6775722e636f6d2f325a41376f615a2e706e67)

## Nerdtree 目录搜索和文件跳转插件

大概长这样：

![vim-Nerdtree](https://github.com/scrooloose/nerdtree/raw/master/screenshot.png)

安装方式同样，并且设置一下快捷键映射：

```
let mapleader = ','
call plug#begin('~/.vim/plugged')

Plug 'scrooloose/nerdtree'

call plug#end()

nnoremap <leader>v :NERDTreeFind<cr>
nnoremap <leader>g :NERDTreeToggle<cr>
let NERDTreeShowHidden=1
let NERDTreeIgnore=[       
         \ '\.git$', '\.hg$', '\.svn$', '\.stversions$', '\.swp$', '\.DS_Store$',
         \ ]     
```

然后 `source ~/.vimrc`，在 vim 中执行 `PlugInstall`。

## ctrlp 模糊搜索工具

可以快速搜索并打开一个文件。

安装方法一样，设置快捷键：

```
call plug#begin('~/.vim/plugged')
Plug 'kien/ctrlp.vim'
call plug#end()
let g:ctrlp_map = '<c-p>'
```

大概这个样子：

![ctrlp](https://camo.githubusercontent.com/0a0b4c0d24a44d381cbad420ecb285abc2aaa4cb/687474703a2f2f692e696d6775722e636f6d2f7949796e722e706e67)

这个 `ctrlp` 工具真心好用。
