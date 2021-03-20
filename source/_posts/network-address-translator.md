---
title: network-address-translator
date: 2021-02-02 16:50:17
tags:
categories: linux
thumbnail: https://gitee.com/jiaoqiyuan/pics/raw/master/blog/network-address-traslator/network-address-traslator.png
photos: https://gitee.com/jiaoqiyuan/pics/raw/master/blog/network-address-traslator/network-address-traslator.png
---

由于工作需要，想在家也能登录公司的机器，公司机器是 Win10，装有 WSL2，目的是要在家里 Win10 的 WSL 上登录公司机器的 WSL，初步设想是通过 SSH 远程登录，带宽占用小，使用起来也比较稳定。

<!--more-->

## 需求分析

有了需求那就开干，公司电脑是内网机器，通过公司路由器连接互联网，想要连接公司电脑首先要做的就是内网穿透，使其能够被外部互联网访问。

在网上查询了内网穿透的方法后试用了两个方法：ngrok 和花生壳，先说结论吧，推荐使用花生壳，国内厂商产品，在国内速度比较快，ngrok 速度很卡，难以达到可用目的。

## ngrok

1. 现在 [ngrok 官网](https://ngrok.com/) 注册一个账号，用于分配 token，后期拿这个 token 在本地启动 ngrok 的时候会用到。

    ![token](https://gitee.com/jiaoqiyuan/pics/raw/master/blog/network-address-traslator/token.png)


2. [ngrok 官网](https://ngrok.com/) 下载后，在本地 WSL 中执行：

    ```
    ./ngrok authtoken 1nrLohVXBo1xpuOYZt22pBMnQtR_7Cq7Q5KJdgiVLYgVGu4PL
    ./ngrok tcp 22
    ```

    ```
    ngrok by @inconshreveable                                                                                                                                                                                                                                    (Ctrl+C to quit)                                                                                                                                                                                                                                                                             Session Status                online
    Account                       jijujiu@gmail.com (Plan: Free)
    Version                       2.3.35
    Region                        United States (us)
    Web Interface                 http://127.0.0.1:4040
    Forwarding                    tcp://0.tcp.ngrok.io:xxxx -> localhost:22
    ```

    执行后就完成内网穿透中了，在其他互联网机器上使用 ssh 正常连接即可：

    ```
    ssh user@0.tcp.ngrok.io -p xxxx
    ```

## 花生壳内网穿透

1. 进入 [花生壳官网](https://hsk.oray.com/)，注册一个账号后会赠送一个体验版的功能。

2. 下载 [花生壳 Linux 客户端](https://down.oray.com/hsk/linux/phddns_5_1_amd64.deb) 到本地安装:

    ```
    sudo dpkg -i phddns_5_1_amd64.deb
    phddns status
    phddns restart
    ```

    ```
        ╰─$ phddns status
    +--------------------------------------------------+
    |          Oray PeanutHull Linux 5.1.0             |
    +--------------------------------------------------+
    |              Runstatus: ONLINE                   |
    +--------------------------------------------------+
    |              SN: xxxxxxxxxxxxxxxx                |
    +--------------------------------------------------+
    |    Remote Management Address http://b.oray.com   |
    +--------------------------------------------------+
    ```

    SN 就是序列号，后续登录网站使用。

3. 登录 `http://b.oray.com`，数据上面的 SN，默认密码是 admin，即可管理。

4. 进入 [控制台](https://console.hsk.oray.com/forward) 配置映射关系。

    ![create](https://gitee.com/jiaoqiyuan/pics/raw/master/blog/network-address-traslator/hsk1.png)

    ![console](https://gitee.com/jiaoqiyuan/pics/raw/master/blog/network-address-traslator/hsk.png)

5. 本地执行：

    ```
    ssh -p xxxx user@server_url
    ```

## 总结

花生壳目前已满足我的需要，只要能远程连接，无卡顿即可，不要求传输文件，另外配置向日葵远程桌面可以处理一些内网穿透异常问题（比如 Win10 自动重启了！！！），当然前提是向日葵设置了开机自启动。