---
title: 各种printf总结
date: 2018-06-13 03:45:33
tags:
categories: linux
thumbnail: https://teddysun.com/wp-content/uploads/2014/Linux.jpg
---

    编程中printf是每个人都很熟悉的，但是与printf相关的那些函数你了解过吗？这里总结一下。

## printf

功能：很常用的printf函数，用于将数据以特定的格式输出到标准输出设备stdout（通常就是我们的终端界面）。

函数声明：

```
int printf(const char *format, ...)
```

返回值：int类型数据。看到没，其实printf是有返回值的，而且是int类型，这个以前还真没有注意过。

参数：

- format：格式说明。
- ...：变长参数列表

## dprintf

功能：将数据写入指定的文件描述符。注意与fprintf的区别，二者数据写入对象不同，dprintf指定只能写入文件描述符所指定的文件中。

函数声明：

```
int dprintf(int fd, const char *format, ...);
```
返回值：int类型。

参数：
- fd：文件描述符
- format：格式说明
- ...：变长参数列表

dprintf用的不多，反正我是没怎么用过，可能接触到linux内核的话使用的比较多吧。

## fprintf

功能：将数据以标准格式输出到流steam中。[参考链接][1]. 

函数声明：

```
int fprintf(FILE *stream, const char *format, ...)
```

返回值：int类型数据。

参数：

- stream：指向 FILE 对象的指针，该 FILE 对象标识了流。
- format：格式说明。
- ...：变长参数

fprintf挺常用的，通常配合fopen、fclose使用。这里注意fprintf是将数据以字符的形式写入流中的，即使我们写的是int类型的数据，也是转化成了字符进行写入的。

比如我们使用fprintf将54写入stream，那么54就会被分别以'5'和'4'写入到stream，共占据两个字节，同样，如果写入100，那么就会占据三个字节。

这里要跟fwrite做一个区别，fwrite也是想流stream中写入数据，但是fwrite是以二进制的格式将数据写入的，即我们写入整形数据54的话，是以int类型进行存储的，占用4字节，存储100时也是一样，同样是int类型数据，所以占据的空间也是4字节。

fwrite是将数据不经转换直接以二进制的形式写入文件，而fprintf是将数据转换为字符后再写入文件。[参考链接][2].

## vfprintf

函数功能：将参数列表以标准格式输出到流stream中。

函数声明：

```
int vfprintf(FILE *stream, const char *format, va_list arg)
```

参数：
- stream：指向 FILE 对象的指针，该 FILE 对象标识了流。
- format：跟上面的一样，是格式说明。
- arg：参数列表。

返回值：int数据

vfprintf与fprintf的区别，在于第三个参数的形式。比如要写一个自定义log接口，自己体会一下：

```
void log(FILE *file, const char* format, ... )
{
    va_list args;
    va_start (args, format);
    fprintf(file, "%s: ", getTimestamp());
    vfprintf (file, format, args);      //在这个地方用vfprintf函数就很合适，因为第三个参数可以直接得到
    va_end (args);
}
```

## vprintf

函数功能：使用参数列表将数据输出到stdout标准输出设备。

函数声明：

```
int vprintf(const char *format, va_list arg)
```

返回值：int类型数据。

参数：不写了，跟上面的一样。

这里与vfprintf功能相似，就是输出对象改成了标准输出，所以就隐藏不写了，那就没什么好说的了，参考vfprintf。

## sprintf

函数功能：发送格式化输出到字符串

函数声明：

```
int sprintf(char *str, const char *format, ...)
```

返回值：写入成功返回写入的字符总数，失败返回一个负数。

参数：

- str：字符串指针
- format：格式说明
- ...：变长参数

这里需要注意str指向的内存空间的地址大小要保证能够存放的下format指定的数据内容，否则会出现缓冲区溢出问题，所以这个函数使用起来不安全，有缓冲区溢出的风险。

更安全的使用请参考snprintf。

## asprintf

函数功能：sprintf增强版，能够根据格式化字符串长度申请内存空间，保证不会有缓冲区溢出风险。原生级别防止缓冲区溢出。

函数声明：

```
int asprintf(char **strp, const char *fmt, ...);
```

返回值：成功返回写入的字符总数，失败返回一个负数。

参数：

- strp：指向存放自动申请内存空间的地址的指针。
- fmt：格式化说明。
- ...：变长参数

这里执行完asprintf后，strp会指向一个已经申请好的内存指针，需要注意的是这块自动申请的内存需要手动释放，比如：

```
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

int main()
{
    char *buf;
    if (asprintf(&buf, "%s-%d", "tesdjkjsdlgjdt", 1334) < 0) {
        printf("asprintf error!\n");
        return;
    }
    printf("buf:%s\n", buf);
    printf("strlen(buf):%d\n", strlen(buf));
    free(buf);
}

}
```

## vsprintf

函数功能：使用参数列表发送格式化输出到字符串。

函数声明：

```
int vsprintf(char *str, const char *format, va_list arg)
```

返回值：如果成功，则返回写入的字符总数，否则返回一个负数。

这个函数与sprintf类似，除了第三个参数，这里比较一下第三个参数，这里用的是va_list，sprintf的第三个参数是...，二者的区别是什么？

...表示的是任意个数的参数，va_list是一种参数列表，通过va_star和va_end可以从...中获取所有的参数，然后把它们组织到va_list中，供程序使用，可以简单理解为可变参数从...中存放到了va_list后才能被程序使用。（可能这样理解太肤浅了，具体原理网上有详细介绍，这里只是为方便理解这样说的。）

与sprintf一样，这里还是有缓冲区溢出的风险。

## snprintf

函数功能：将格式化的数据写入字符串。

函数声明：

```
 int snprintf(char *str, int n, char * format, ...);
```

返回值：成功则返回参数str 字符串长度，失败则返回-1，错误原因存于errno 中。

参数：

- str:要写入的字符串指针
- n：写入的长度（注意这里对写入的长度有限制，与sprintf的区别）
- format：格式说明
- ...：可变参数

上面数到sprintf有缓冲区溢出风险，而asprintf从接口上就杜绝了这种风险，因为它会给目标指针分配足够大的空间来写入数据。这里的snprintf则是从使用者角度提供了规避缓冲区溢出风险的方法，使用者可以指定写入长度从而确保不会拷贝过多数据到目标内存空间，如果使用者太傻，还是会有风险，相比较而言，我觉得asprintf挺安全的，就是使用完毕后需要手动释放申请的内存空间，这个不能忘记。

## vsnprintf

函数功能：将可变参数格式化输出到一个字符数组。

函数声明：

```
int vsnprintf(char *str, size_t size, const char *format, va_list arg)
```

参数：不说了

返回值：执行成功，返回写入到字符数组str中的字符个数（不包含终止符），最大不超过size；执行失败，返回负值，并置errno.

跟vsprintf类似，只不过加上了防止缓冲区溢出的措施，通过指定输出字符个数来规避这种风险。


## 总结

这些是我从公司项目源代码中所使用到的与printf相关的函数，寻找办法是先使用source insight建立源码工程，然后搜索printf关键字的所有结果，保存输出结果为文件。然后使用python正则表达式匹配出所有与printf相似的字段名称，找到了这些函数。

里面呢有我经常用到的printf、sprintf、snprintf，fprintf。其中vfprintf之前做自定义日志接口时用到了，觉得还是挺有用的。其他平时没怎么用到但是有些还是给我的感觉还是挺有用的，比如asprinf竟然可以自动帮助申请内存，还挺有意思的。其他的一些我觉得功能还是挺强大的，以后需要的时候可以用一下。

贴上python代码吧，说实话python做这些小辅助工具还挺好用的，不是说python就适合做小工具，而是说明python的用途是多种多样的，大到人工智能，小到小工具，简直了。

``` "python"
import re

BUFFSIZE = 65536

result = []

def find_printf(line):
    matchs = re.findall(r'[a-zA-Z]*printf', line)
    for match in matchs:
        if match in result:
            continue
        elif match not in result:
            result.append(match)

with open('result') as f:
    while 1:
        try:
            lines = f.readlines(BUFFSIZE)
        except UnicodeDecodeError:
            print('UnicodeDecodeError')
            pass
        if not lines:
            break
        for line in lines:
            find_printf(line)
            #print(line)

print(result)
print(len(result))
```

2018-6-13 北京 凉爽的一周

[1]: http://www.runoob.com/cprogramming/c-function-fprintf.html
[2]: https://blog.csdn.net/godenlove007/article/details/7721647
