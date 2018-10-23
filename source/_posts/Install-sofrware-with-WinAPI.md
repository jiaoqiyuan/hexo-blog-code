---
title: Install-sofrware-with-WinAPI
date: 2017-11-21 20:36:50
tags: windows API
categories: windows
thumbnail: https://ss2.bdstatic.com/70cFvnSh_Q1YnxGkpoWK1HF6hhy/it/u=62450791,588619291&fm=27&gp=0.jpg
---

# 使用Windows自带API安装软件

这是我在[知乎上提的一个问题][1]，后来自己尝试着把问题解决了。

最近用QT做了个软件管理的客户端，现在可以从服务端下载软件啦，但是进行软件安装的时候我使用的是QProcess的execute函数进行安装的，参数传递的是被安装软件的绝对路径。但是函数返回了-2，也就是说这个process不能被执行。

我是在windows下的QT中想直接安装已下载的软件包，最开始是直接想用QT的QProcess中的execute（const QString & program, const QStringList & arguments ）函数安装，但是传入安装包的路径和参数列表后安装没反应。查了一下资料好像QT下没有什么其他方法了，那就使用windows的api来安装吧，之前在linux下也是使用shell来辅助安装的，所以在windows下肯定也会有其他方法来完成安装。

之后又尝试了system命令，这次可以正常弹出安装界面，但是有个背景黑框，很难看，也很不友好，所以还是要继续尝试其他安装方法。

windows下调用第三方程序的方法有很多，除了system，还有WinExec , CreateProcess, ShellExecute, ShellExecuteEx。这里我有尝试winexec，但是调用该方法时没有弹出安装界面，不知道是不是我的调用方法有问题，贴到这里给大家看一下：
```
strPath.replace("/", "\\");
qDebug() << "strPath: " << strPath <<endl;
char *ch;
QByteArray ba = strPath.toLatin1();
ch = ba.data();
nRet = WinExec(ch, SW_SHOW);
```
这里的strpath是我的安装包的绝对路径，因为QT下获取的是QString类型的路径，先转化成了char* 数据，然后传递给WinExec，第二个参数设置为SW_HIDE可隐藏终端黑框，这里设置为SW_SHOW是为了测试。

因为我只是想安装这个软件包，并获取安装结果（成功/失败），这里我又尝试了用法相对简单的ShellExecuteEx，这次可以正常安装软件了，而且隐藏了终端黑框，同时可以获取到安装结果信息。代码附上：
```
int installsoft(char *path)
{
    SHELLEXECUTEINFO si;
    ZeroMemory(&si, sizeof(si));
    si.cbSize = sizeof(si);
    si.fMask = SEE_MASK_NOCLOSEPROCESS;
    si.lpVerb = _T(char_wchar("open"));
    si.lpFile = _T(char_wchar(path));
    si.nShow = SW_HIDE;

    ShellExecuteEx(&si);
    DWORD dwExitCode;
    GetExitCodeProcess(si.hProcess, &dwExitCode);
    while(dwExitCode == STILL_ACTIVE)
    {
        Sleep((DWORD)3);
        GetExitCodeProcess(si.hProcess, &dwExitCode);
    }
    CloseHandle(si.hProcess);
    return (int)dwExitCode;
}
```
同样是把安装包的路径转化为char* 类型后传递给ShellExecuteEx。


  [1]: https://www.zhihu.com/question/67931448/answer/260607115