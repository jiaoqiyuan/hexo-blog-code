---
title: ipython使用笔记
date: 2018-06-20 21:17:21
tags:
categories: python
thumbnail: http://editerupload.eepw.com.cn/201411/48601416902987.jpg
---

1. ipython中可以使用edit命令编辑文件内容，edit打开文件的方式根据系统已安装编辑器决定，也可以通过执行ipython前使用
```
export EDITOR=vim
```
自己指定编辑器类型。

比如：
```
edit t.py
```
输入：
```
l = range(10)
```

2. 通过run命令可以执行python脚本:

```
run t.py

--------------------

In [8]: l
Out[8]: range(0, 10)
```
可以看到，执行完t.py这个脚本后，l就可以被识别到了。

3. 使用alias设置常用命令别名：

```
alias largest ls -1sSh | grep %s

%largest t
```
要存储别名，需要使用
```
store largest
```

4. 使用bookmark存储书签，相当于linux下的软链接。

```
%bookmark root /home
cd root    //相当于进入了home目录
```

5. hist命令查看历史纪录，并使用save命令存储指定行的内容到文件中

```
save a.py 9-11     //存储9-11行到a.py这个文件中
```

6. 使用debug函数进入调试器调试程序：

```
In [18]: a = 1
In [19]: b = 0
In [20]: a / b
---------------------------------------------------------------------------
ZeroDivisionError                         Traceback (most recent call last)
<ipython-input-20-d8e10abd5ab6> in <module>()
----> 1 a / b
ZeroDivisionError: division by zero
In [21]: debug
> <ipython-input-20-d8e10abd5ab6>(1)<module>()
----> 1 a / b
ipdb> p a
1
ipdb> p b
0
ipdb> exit
```

6. 使用%rehashx命令将linux下的命令都更新进ipython中方便实用：

```
In [23]: echo 1
  File "<ipython-input-23-932d3634cbf0>", line 1
    echo 1
         ^
SyntaxError: invalid syntax


In [24]: %rehashx

In [25]: echo 1
1

```

7. 使用%store -r恢复之前保存的别名

8. 使用load命令只加载程序内容，不执行，跟run有区别，run直接执行。

9. 使用logstart可以将ipython中的内容记录为日志文件保存起来。

```
In [26]: logstart
Activating auto-logging. Current session state plus future input saved.
Filename       : ipython_log.py
Mode           : rotate
Output logging : False
Raw input log  : False
Timestamping   : False
State          : active

In [27]: a = 1

In [28]: b =2

In [29]: c= 1

In [30]: %logoff
Switching logging OFF

```

之后推出ipython后再次启动ipython之前使用：

```
ipython -i ipython_log.py
```
就可以将之前记录的变量以及数据继续使用。

10. 编辑ipython配置文件实现autoreload功能

```
默认是没有ipython_config.py这个配置文件的，需要使用 ipython profile create 这个命令创建一个默认配置文件
编辑.ipython/profile_default/ipython_config.py
将
32：c.InteractiveShellApp.exec_lines = ['autoreload 2']
35：c.InteractiveShellApp.extensions = ['autoreload']
600:c.StoreMagics.autorestore = True

分别改为上述内容
```

再次进入ipython：

```
In [1]: edit test.py
Editing... done. Executing edited code...
-----------------------------------------
文件test.py的内容：
def run():
    return 2
-----------------------------------------

In [2]: from test import run

In [3]: run()
Out[3]: 2

In [4]: edit test.py
Editing... done. Executing edited code...
-----------------------------------------
文件test.py的内容：
def run():
    return 10
-----------------------------------------
In [5]: run()
Out[5]: 10
```

可以看到我们手动修改了test.py中的run函数返回的值后不需要再次import这个run函数就可以自动返回最新的值（10），这就是autoreload的作用。

2018.6.21 北京 晴 天微亮 失眠之夜写下这个笔记