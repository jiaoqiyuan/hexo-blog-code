---
title: Istall-pygame-on-linux
date: 2017-11-16 21:10:37
tags: pygame linux
categories: python
thumbnail: https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1510848037349&di=5c382e559a80f48ddb615e0a34dcae0d&imgtype=0&src=http%3A%2F%2Fread.pudn.com%2Fdownloads9%2Fsourcecode%2Fwindows%2F29768%2Fpygame-1.6%2Fdocs%2Fpygame_logo.gif
---

最近在看一本书《python编程：从入门到实践》，感觉相当不错，我算是有点python基础的，之前看过一段时间python相关的编程入门书籍和资料，本身也是程序员，感觉python入门还是比较简单的。这本书有基础有实践，第一部分讲的基础，第二部分是项目实践，项目实践分三部分，分别是用python做一个小游戏、python数据处理、python web开发，正好这三部分都是我比较感兴趣的地方，所以我觉得很值得推荐出来，希望这本书对你也有帮助，正版图书图灵社区有卖，另外其他电商平台应该也有卖，有能力的话还是希望能够支持一下正版吧。

# pygame安装 #
在《python编程：从入门到实践》第二部分项目实践中，第一个项目是使用pygame做一个打飞机的小游戏，这里记录一下pygame的安装步骤，以备后期自己回头查看，同时也希望能够帮助到需要它的人。

## 安装步骤 ##

### 安装pip3 ###
安装pip3主要是未了能够方便安装python3和pygame，这里推荐使用python3作为学习和开发工具，别问我为什么，python3是趋势，学完python3再重点了解下python3和python2的区别就能很容易获得python2的开发技能，反之我觉得后期会比较蛋疼。
** pip安装方法： **
```
sudo apt install python3-pip
```
安装pip3后默认安装的pip3是8.x版本的，需要升级到9.0，升级命令是：
```
sudo pip3 install --upgrade pip(如果命令有错误，请留言联系我)
```

### 安装pygame依赖库 ###
```
sudo apt install python3-dev mercuial
```

### 安装pygame的高级功能 ###
pygame中包含一些高级功能，比如声音功能。
** 安装方法： **
```
sudo apt install libsdl-mixer1.2-dev libportmidi-dev
sudo apt-get install libswscale-dev libsmpeg-dev libavformat-dev libavcode-dev(最后这一个好像不用安装就行，因为系统已经安装过了，我的机器是mint-mate18，看情况吧)
sudo apt install python3-numpy
```

### 安装pygame ###
** 安装方法： **
```
sudo pip3 install pygame
```

2017-11-16
北京
晴