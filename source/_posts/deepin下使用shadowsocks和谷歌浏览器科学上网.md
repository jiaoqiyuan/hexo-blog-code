---
title: deepin下使用shadowsocks和谷歌浏览器科学上网
date: 2018-11-13 09:26:18
tags:
categories: 科技
thumbnail: https://static.fatesinger.com/2015/07/64e84a873a37f832.jpg
---

# deepin安装shadowsockets
    最近装了个deepin，安装shadowsocks时记录一下，免得以后还要到处查看怎么安装。

## 1，安装pip3
```
wget https://bootstrap.pypa.io/get-pip.py
sudo python3 get-pip.py
```
不要使用apt安装pip3，因为后面可能会遇到高版本的pip3会有main函数导入失败的问题，所以就用get-pip安装吧。

PS:在使用ubuntu时，安装pip3可能会遇到这个问题：
```
Traceback (most recent call last):
  File "get-pip.py", line 20890, in <module>
    main()
  File "get-pip.py", line 197, in main
    bootstrap(tmpdir=tmpdir)
  File "get-pip.py", line 82, in bootstrap
    import pip._internal
  File "/tmp/tmpfs3x_4_3/pip.zip/pip/_internal/__init__.py", line 40, in <module>
  File "/tmp/tmpfs3x_4_3/pip.zip/pip/_internal/cli/autocompletion.py", line 8, in <module>
  File "/tmp/tmpfs3x_4_3/pip.zip/pip/_internal/cli/main_parser.py", line 8, in <module>
  File "/tmp/tmpfs3x_4_3/pip.zip/pip/_internal/cli/cmdoptions.py", line 17, in <module>
  File "/tmp/tmpfs3x_4_3/pip.zip/pip/_internal/locations.py", line 10, in <module>
ImportError: cannot import name 'sysconfig'
```
这是因为少安装了一个包：
```
sudo apt install python3-distutils
```

## 2, 安装shadowsocks
```
sudo pip3 install shadowsocks
```
## 3，编写配置json文件shadowsocks.json
```
{
	"server":"xxxxxxx",
	"server_port":00000,
	"local_port":1080,
	"password":"hehe",
	"timeout":600,
	"method":"chacha20-ietf-poly1305"
}
```
把server，port，password改成自己的shadowsocks信息。


## 4, 开启shadowsocks服务

```
/usr/bin/ss-local -c /home/jony/apps/shadowsocks/shadowsocks.json
```

PS: Ubuntu18可能会遇到：Command 'ss-server' not found, 那就用下面命令安装。
```
sudo apt install shadowsocks-libev
```

可以将上面的命令放到一个可执行文件shadowsocks内，并设置为开机自启动：
```
sudo cp shadowsocks /etc/init.d
sudo update-rc.d shadowsocks defaults 100
```
这样就能每次开机自启动shadowsocks服务了。



## 5, 在谷歌浏览器安装SwitchyOmega

[谷歌浏览器下载地址][3]。

[SwitchyOmega下载地址][1]。

下载完直接丢带google chrome的扩展程序中去就行了。

## 6，配置SwitchyOmega插件

[参考网址][2]

只参考配置SwitchyOmega的部分即可。

## 7，愉快科学上网，仅限学习用途，不得做违法的事！切记！

[1]: https://github.com/FelisCatus/SwitchyOmega/releases
[2]: https://www.aliyun.com/jiaocheng/134757.html
[3]: http://www.ubuntuchrome.com/