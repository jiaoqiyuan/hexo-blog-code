---
title: 基于hexo和github搭建个人博客
date: 2018-02-03 17:16:43
tags: Heox Github linux
categories: windows
thumbnail: http://yzhtml01.book118.com/2016/11/27/19/45710191/1.files/file0001.jpeg
---

## 安装步骤
在本地新建一个目录名叫hexo（名字自己可以任意取，主要是存放搭建博客过程中需要用到的文件）。
## 安装 nodejs
我的机器是mint-mate18,之前用apt安装过nodejs，感觉不太好用，所以我都是直接下载官方编译好的代码来安装node的，官网可用node [64位下载地址][1]，[32位下载地址][2]。直接下载后解压node-v8.7.0-linux-x64.tar.xz,因为是xz文件，所以解压命令跟一般tar解压命令不太一样，这里的解压命令是：tar xJvf node-v8.7.0-linux-x64.tar.xz。
解压后进入node/node-v8.7.0-linux-x64/bin目录，创建hexo和npm的软链接，这个目录下还有一个npx不知道是做什么用的，目前用不到就先不用管，创建软链接的目的是方便在Terminal中使用hexo和npm这两个工具。创建软链接的命令如下：
```
ln -s /home/jiao/hexo/node/node-v8.7.0-linux-x64/bin/node /usr/bin/node
ln -s /home/jiao/hexo/node/node-v8.7.0-linux-x64/bin/npm /usr/bin/npm
```
创建完链接就可以直接在终端中使用node命令和npm命令了。这里面的/home/jiao/hexo/node是我存放node-v8.7.0-linux-x64的目录，要根据实际情况修改成本机存放node-v8.7.0-linux-x64目录的路径。
关于nodejs的作用以及node命令的用处，我觉得[官网][3]说的是最权威的了，npm命令是nodejs的包管理器，后面的hexo的安装需要借助npm安装。
 ![node官网][4]

## 安装 npm（cnpm）
通过上面的创建软链接，我们已经安装了npm，这里推荐使用cnpm，cnpm是淘宝源的npm的替代命令，在国内更稳定点。
cnpm安装方法：
```
npm install -g cnpm --registry=https://registry.npm.taobao.org
```
如果遇到权限问题就在前面加上sudo：
```
sudo npm install -g cnpm --registry=https://registry.npm.taobao.org
```
将cnpm加入系统环境变量目录/usr/bin目录中
```
sudo ln -s /home/jiao/hexo/node/node-v8.7.0-linux-x64/bin/node /usr/bin/cnpm
```

如果还是不成功可以[参考这里][5]。
## 安装 hexo
安装完node和cnpm，接下来就需要安装hexo了，可以直接使用cnpm安装hexo，安装命令如下：
```
sudo cnpm install hexo-cli -g
sudo cnpm install hexo-server --save
sudo cnpm install hexo-deployer-git --save
```
安装完成后在终端输入hexo，如果有hexo用法提示，则代表安装成功。
![此处输入图片的描述][6]

## 在github创建个人博客项目
这部分比较简单，只有一点需要**特别注意**，就是github项目名一定要写成XXX.github.io，读者可以参考[『这里』][7]创建自己的gihub博客项目。
## 配置hexo
到这里，需要安装的基本上都安装完了，下面就是对hexo的配置和一些必要的工具的安装。
手动创建一个文件夹blog，然后进入blog目录执行：hexo init，这里hexo会创建一个静态页面项目到本地。
可以使用hexo s命令部署到本地，然后在浏览器输入http://localhost:4000/就可以看到一个静态的页面了。
退出后在当前目录下配置_config.yml文件，在文件的最下面修改配置为：
```
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git #这里是部署的方式选用git
  #repo: https://github.com/jiaoqiyuan/Blog.git
  repo: git@github.com:jiaoqiyuan/jiaoqiyuan.github.io #这里填写之前创建的github项目的地址。
  branch: master
```
这样修改完成后就可以上传本地项目到github上了，使用
```
hexo g -d
```
命令生成新的静态页面并上传到服务端。

## 部署
github pages是个免费的静态站点，我们可以将自己的博客免费托管到上面，当然你也可以自己购买云服务器进行部署。

1. 创建github pages

根据你的github账号创建一个仓库，但是这个用于静态托管博客的仓库是有格式限制的，它的名字必须是yourname.github.io，其他跟正常仓库创建一样就行。
![github pages][8]

2. 本地电脑(hexo)配置github pages

hexo的安装上面已经介绍，接下来就是使用hexo创建本地博客环境的过程，在终端创建使用如下命令：
```
hexo init yourblogname
cd yourblogname
cnpm install
```
然后打开yourblogname目录下的_config.yml文件，将type自该改为git，repo改为你刚才创建的github pages仓库的地址，branck字段改为master。

然后终端输入：
```
hexo s
```
就可以开启本地服务，在浏览器输入：
```
http://localhost:4000
```
就可以预览本地创建的静态页面了。
ok！到此就基本成功了。

## 发布
到目前为止，项目在本地已经搭建起来，但是还没有跟github pages联系起来，其实刚才我们已经配置了github pages的地址，只不过还没有上传到服务端，在终端使用一下命令发布博客到服务端：
```
hexo clean    //清除本地缓存
hexo g        //生成本地发布的文件夹
hexo d        //发布到github pages上
```
或者：
```
hexo clean
hexo g -d
```
二者等效。

## 后记
至此，基于github pages已经搭建起我们的博客网站了，只要访问：https://github.com/yourname/yourname.github.io，就可以访问了。

还有一些优化的功能：
1. 主题设置
2. 申请自己的域名绑定博客地址

…………
其他还有很多好玩的等待你去探索！

2018-04-09 
北京 
晴

  [1]: https://npm.taobao.org/mirrors/node/v8.7.0/node-v8.7.0-linux-x64.tar.xz
  [2]: https://npm.taobao.org/mirrors/node/v8.7.0/node-v8.7.0-linux-x86.tar.xz
  [3]: http://nodejs.cn/
  [4]: https://raw.githubusercontent.com/jiaoqiyuan/hexo-blog-code/master/source/img/nodejs.png
  [5]: https://npm.taobao.org/
  [6]: https://raw.githubusercontent.com/jiaoqiyuan/hexo-blog-code/master/source/img/hexo-install.png
  [7]: https://jingyan.baidu.com/article/f7ff0bfc7181492e27bb1360.html
  [8]:https://upload-images.jianshu.io/upload_images/660127-c4c554b26669999f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700
