---
title: linux下的静态库、动态链接库、动态加载库
date: 2018-06-05 07:34:08
tags:
categories: linux
thumbnail: https://teddysun.com/wp-content/uploads/2014/Linux.jpg
photos: https://teddysun.com/wp-content/uploads/2014/Linux.jpg
---

    还是最近项目中遇到的问题，是关于动态库的，这里做个总结。
    
## linux的库文件

我们在某个程序文件中写了一段通用性比较浅的代码，比如某个算法，或者某个操作，通常会在其他的项目或者文件中使用到这部分功能，我们当然可以在当前项目中再次实现一次，但是这种方式未免太麻烦了，而且代码会变得比较冗长，不利于管理和维护。另外如果我们写的程序需要知识产权保护，那么也不能直接将源码公布出来给其他人使用。所以库文件出现了！

通常库文件有两种：静态库和动态库，~~而动态库又可以分为动态链接库和动态加载库~~，动态库就是动态库，没有分动态链接库和动态加载库，应该说是动态库的加载方式有两种：动态链接和动态加载。

库文件是为了程序的复用，以及隐藏部分程序实现细节，从而让程序开发变得简单，我们不必知道库文件中接口的具体实现方法，只需要知道接口的使用方法即可，这也有利于开发人员之间的协作。

下面具体说明一下各种库的编译手段和使用方法。

以下的讲解都是以这个简单的例子为基础：
```
*******main.c*********************************
#include <stdio.h>
extern void print_hello();    //这里声明为exter变量是说明print_hello这个函数是在别处实现的，
                              //我们也可以去掉extern关键字，用头文件的形式将print_hello包含其中，
                              //然后将头文件包含进main文件即可
int main()
{
        print_hello();
        return 0;
}

*******hello.c*********************************
#include <stdio.h>

int print_hello()
{
        printf("Hello world!\n");
        return 0;
}

```

## 静态库

静态库是一个或多个编译生成的.o文件打包，组合成一个.a文件，供其他程序在链接时使用。

静态库的创建方法：

```
gcc -c hello.c
ar -rcs libhello.a hello.o
```

这样就创建了一个静态链接库。

静态库的使用方法：

```
gcc -o main main.c libhello.a

```

直接在编译最后加上静态库的名字就可以编译出包含该静态库的二进制可执行文件，这里有一个需要注意的地方就是静态库的路径，如果静态库在当前目录下，可以直接按上面的方法编译程序，如果静态库在其他路径下，需要在编译的时候加上静态库的路径（绝对路径和相对路径都可以）：

```
gcc -o main main.c libhello.a
```
或者
```
gcc -o main main.c -L./ -lhello
```

第一中方式适用于静态库在当前目录下，第二种方式可以任意指定静态库的存放位置。

执行程序：

```
./main

Hello world!
```

## 动态链接库

动态库链接库是程序运行时加载的库，在程序启动的时候立刻加载，库中函数和变量的地址是相对地址，不是绝对地址，其真实地址在调用动态库的程序加载时形成。

动态链接库创建方法：

```
gcc -fPIC -shared -o libhelloso.so hello.c
```
使用-shared指定创建的库为动态链接库。如果不指定-shared，则创建的是动态加载库，下面再介绍动态加载库，这里先看动态链接库怎么使用。

动态链接库使用犯法如下：

```
gcc -o mainso main.c -L./ -lhelloso -Wl,-rpath=.
```

动态链接库使用时必须加上编译路径和执行时的路径，这两个是不一样的，如果不加路径可能导致找不到该动态链接库。

这里我把动态链接库放到了与main.c相同的目录，所以指用-L指定编译路径为当前目录，用-Wl,-rpath=指定运行路径也是当前目录，这样main.c在编译和运行时都会在当前目录寻找libhelloso.so。

运行一下mainso：

```
➜  library git:(master) ✗ ./mainso
Hello world!
```

使用ldd查看可执行程序加载的动态库文件：

```
➜  library git:(master) ✗ ldd mainso
        linux-vdso.so.1 =>  (0x00007ffcbadfd000)
        libhelloso.so => ./libhelloso.so (0x00007f173700c000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f1736c42000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f173720e000)
```
可以看到，我们编译的libhelloso.so库文件是在程序运行时就加载进来的，注意这个细节，跟我们下面使用的动态加载库的方法是不一样的。

动态加载库是在程序用到的时候由程序员手动加载的，所有用ldd命令是看不到的。


## 动态加载库

讲到动态加载库就不得不与动态链接库对比一下了，动态加载库顾名思义就是程序在用到的时候才会加载，而动态链接库是在程序启动时就加载。

动态加载库需要使用到dlopen、dlclose、dlsys等动态库函数来控制。

动态加载库的创建方法和动态链接库一样，其实这里不是说动态库分动态加载库和动态链接库，准确地说应该是加载库的方法分为动态链接和动态加载，两种方法的主要区别就是加载动态库的时机。

这里main.c需要修改一下了，重新创建一个文件叫maindl.c，内容如下：

```
#include <stdio.h>
#include <dlfcn.h>

extern void print_hello();

int main()
{
        void (*printhello)();
        void *fb = NULL;
        char *err = NULL;
        fb = dlopen("./libhelloso.so", RTLD_LAZY);
        if(!fb){
                printf("Failed load library!\n");
        }
        err = dlerror();
        if(err != NULL) {
                printf("%s\n", err);
                return (-1);
        }

        printhello = dlsym(fb, "print_hello");
        err = dlerror();
        if(err != NULL) {
                printf("%s\n", err);
                return (-1);
        }

        (*printhello)();

        dlclose(fb);

        return 0;
}
```

动态加载库的方法是首先使用dlopen打开动态库，从这里就可以看出来使用动态加载库的方法是比较灵活的，我们可以让开发人员来控制动态库的加载时机，而不是程序一开始运行就把动态库都加载进来。

dlopen打开动态库后使用dlsym将动态库内的函数地址加载到我们定义的函数指针printhello，这样我们使用printhello的时候就等于是在调用动态库内的print_hello函数了。


动态加载库的使用方法：

```
gcc -o mainsoso maindl.c -ldl
```

这里注意最后的-ldn，这是动态加载库必须要加的选项，加上-ldl编译器才知道这个库是需要动态加载的而不是动态链接的。

执行mainsoso：

```
➜  library git:(master) ✗ ./mainsoso
Hello world!
```

使用ldd命令查看mainsoso的动态库加载情况：
```
➜  library git:(master) ✗ ldd mainsoso
        linux-vdso.so.1 =>  (0x00007fff92580000)
        libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007fab8aee3000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007fab8ab19000)
        /lib64/ld-linux-x86-64.so.2 (0x00007fab8b0e7000)
```

可以看到没有libhelloso.so这个动态库文件，也就是说这个动态库文件没有在程序运行的时候就加载进来，而是在程序执行过程中需要它的时候才被手动加载进来。

## 总结

使用静态库编译出来的程序即使删除静态库程序也能正常执行，因为静态库内的函数和变量信息已经加载到可执行文件中，所以有没有静态库都不影响程序的执行，只会影响程序的编译过程。

使用动态库编译程序时没有把动态库的内容加入可执行文件中，而是在程序执行的时候才开始加载动态库内的函数和变量，所以如果把动态库删除后程序就不能正确加载动态库信息，导致程序出错。

动态库的加载方式有两种，动态链接和动态加载。动态链接是在程序启动的时候就加载进程序内存空间的，而动态链接是在程序运行过程中被加载进程序内存空间的。