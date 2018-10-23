---
title: ubuntu16.04安装iNode客户端
date: 2017-10-19 18:44:18
tags: ubuntu iNode linux
thumbnail: https://www.deepin.org/wp-content/uploads/2015/11/First-Prize-1024x640.jpg
---

由于学习需要，由windows转到啦linux，刚装啦iNode客户端，过程跟这个过程中讲得差不多，链接给出：[ubuntu安装iNode][1]。

为了防止链接失效，再重覆一遍安装过程：iNode for linux[下载地址][2]。

-------------------

下面是在ubuntu16.04下的安装方法：

 1. 下载后得到压缩包：iNodeClient_For_Linux_V3.60-E6210.tar.gz ，解压以后得到文件夹：iNodeClient；解压到任何一个目录都可以，这里我选择的是我的主文件夹；![此处输入图片的描述][3]
 2. 打开终端，cd到iNodeClient中，输入：sudo ./install.sh （据我测试，这一步可做可不做，不知道为啥，直接跳到下一步也可以）.
 3. 提示完成以后运行客户端：./iNodeClient。
 4. 然后会出现./你的安装文件夹/iNodeClient: ```error while loading shared libraries: libjpeg.so.62: cannot open shared object file: No such file or directory```。
 5. 然后输入: ```sudo ln -s /usr/lib/i386-linux-gnu/libjpeg.so.8  /usr/lib/i386-linux-gnu/libjpeg.so.62```。
 6. 接着还会出现一个类似的问题，输入: ```sudo ln -s /usr/lib/i386-linux-gnu/libtiff.so.4 /usr/lib/i386-linux-gnu/libtiff.so.3```(*在ubuntu16.04中，没有libtiff.so.4，只有libtiff.so.5，所以上面这句命令应该改成 ```sudo ln -s /usr/lib/i386-linux-gnu/libtiff.so.5 /usr/lib/i386-linux-gnu/libtiff.so.3```*)。
 7. 再次执行：sudo ./iNodeClient，看到客户端的界面，选择左上角的新建添加链接输入用户名和密码，按照下图点击选项，确定保存。![此处输入图片的描述][4]
 
 8. 新的链接已经建立好了，选中链接，点击上方的“连接”按钮开始连接。连接成功后ubuntu16.04需要先删除有线网络后再手动创建才能正常上网，不知道为啥，我这里是这样，感觉应该是要自动分配IP的原因吧。![此处输入图片的描述][5]


----------


**补充**：每次系统重启后ip地址被重新分配为192.168.1.100，不能正常上网，目前的解决办法就是删除有线网络，再重新建立有线网络，虽然蛋疼，但好歹能上网，回头看有什么方法能解决这个问题。


----------


2016.05.26 成都 阴


----------

**图片查看有问题可以使用电脑端查看，这是从我原来新浪博客转来的，原博客地址着[这里][6]。**

2017.10.19 北京 阴


  [1]: http://wenku.baidu.com/view/c641197402768e9951e73856.html
  [2]: http://pan.baidu.com/s/1skBJcK1
  [3]: http://s13.sinaimg.cn/mw690/002189f5gy71ZcvBfe4ac&690
  [4]: http://s3.sinaimg.cn/mw690/002189f5gy71ZcMthvQ52&690
  [5]: http://s15.sinaimg.cn/mw690/002189f5gy71Zd6KBrg3e&690
  [6]: http://blog.sina.com.cn/s/blog_6e35425b0102wemg.html
