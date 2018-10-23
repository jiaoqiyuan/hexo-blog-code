---
title: 自定义Debug工具-输出日志到本地文件
date: 2018-1-13 23:44:18
tags: 日志 可变参数
categories: windows
thumbnail: https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1515867117826&di=cc84e0693b27a37a023c03b0dd2d192d&imgtype=0&src=http%3A%2F%2Fupload.chinaz.com%2F2016%2F0811%2F201608111002446667.jpg
---

最近项目中调试和运行过程中都需要输出日志信息，项目是用QT和C程序两部分组合实现，本来QT部分已经有日志输出接口了，只是C程序中没有日志接口的实现，这里我利用vfprintf函数基于C语言实现了一个本地日志输出接口。另外通过宏定义的方式，可以很方便地打开和关闭调试信息。

# 本地日志输出接口
调试接口具体实现如下：
```
int printLog(char *formate, ...)
{
    va_list args;
    FILE *fp = NULL;

    time_t mtime = time(NULL);
    struct tm now_time;
    char logtime[64] = { 0 };
    localtime_s(&now_time, &mtime);
    sprintf_s(logtime, 64, "%d-%d-%d %d:%d:%d", now_time.tm_year + 1990, now_time.tm_mon + 1, now_time.tm_mday, now_time.tm_hour, now_time.tm_min, now_time.tm_sec);

    fp = fopen(LOGPATH, "a");
    if (fp == NULL) {
        return -1;
    }
    va_start(args, formate);
    fprintf(fp, logtime);
    fprintf(fp, " | ");
    vfprintf(fp, formate, args);
    fprintf(fp, "\n");
    va_end(args);
    fflush(fp);
    fclose(fp);
    return 0;
}
```
这里用到了可变参数的功能，可以像使用printf函数一样使用该接口，这里需要特别说明的是vfprintf函数，这个函数可以处理可变参数，将他们以特定的格式写入文件指针中，之前一直在纠结可变参数的问题，最后发先了这个函数，一切问题迎刃而解。另外加入了记录日志信息的详细时间。写入本地文件后日志的详细信息类似这样：
```
2108-1-12 21:16:34 | This is the test log!

```

# 调试信息的开关
程序调试和运行过程中，一些调试可能在运行时就不需要了，需要屏蔽掉，但是如果没有统一的开关，逐条屏蔽调试信息，这种方法肯定效率特别低下，这里通过宏定义的方式使我们可以随时打开或关闭调试信息，宏定义如下：
```
#ifdef DEBUG_ON
#define DEBUG(fmt, ...) printLog(fmt, ##__VA_ARGS__)
#else
#define DEBUG(...) 
#endif
```
这个宏定义需要注意的地方就是对可变参数的宏定义，可变参数宏定义形式可使用
```
##__VA_ARGS__
```
的方式来处理。

2018.1.13 北京 晴