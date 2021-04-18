---
title: dev-null
date: 2021-03-20 16:56:13
tags: linux
categories: 翻译
thumbnail:  https://gitee.com/jiaoqiyuan/pics/raw/master/blog/dev-null/dev-null-title.png
photos: https://gitee.com/jiaoqiyuan/pics/raw/master/blog/dev-null/dev-null-title.png
---

信息来源：[阮一峰每周分享][1]

文章来源：[What Is /Dev/Null – an Introduction to the Bit Bucket][2]

---

# What Is /Dev/Null – an Introduction to the Bit Bucket /Dev/Null 是什么 - Bit Bucket 的简介

If you have used Linux for any amount of time, you most likely have come across the bit bucket, normally written as /dev/null. Understanding this special file, how it works, and it’s importance will become a invaluable tool. In this article, we will discuss what /dev/null is, how to use it, and why it earned the name bit bucket. We will also explore some examples of how it can be used in common practice.

如果你使用过 Linux，那么你和可能碰到过位桶，通常写为 dev/null。理解这个特殊的文件如何工作以及它的重要性，将会大有帮助。这篇文章中，我们将会探讨 `dev/null` 是什么，它是如何工作的，以及为什么它会有 `bit buket` 这个名字。同时我们也会展开几个例子，看一下它如何应用到平常的实践中。

<!--more-->

## Why the Name Bit Bucket? 为什么叫 Bit Bucket

Early computers used punch cards to store data and programming code. When the machine punched the holes out, the unused material would drop into a bucket. Over time the bit bucket became a general term for a place where useless bits were discarded.

早期的计算机使用打孔来存储数据和编程。当机器冲出孔时，那些被冲出的无用材料就被丢弃到桶中。随着时间流逝，bit bucket 成为废弃无用比特的地方的总称。

Other names for /dev/null are black hole, null route, punch bucket, null device, or just null.

/dev/null 的其他名字比如黑洞，null route，打孔桶，空设备或者就是 null。

## The /dev/null File /dev/null 文件

The /dev/null file is a pseudo-device file generated at boot. It has no size (0 bytes), takes up 0 blocks on disk, has read/write permissions for all users. The screenshot below shows that the file and last system reboot have the same date and time.

/dev/null 是一个在启动时产生的伪设备，它没有大小（0 字节），在硬盘上占用 0 个块，所有用户对其都有读写权限。下面的截图显示 /dev/null 文件和上次系统重启具有相同的日期和时间。

![stat-dev-null](https://www.putorius.net/wp-content/uploads/2019/11/dev-null-diag1-min.png.webp)

This is a special file known as a character device file. This allows the file to act like a device (null device) that is unbuffered and can accept streams of data. When you access a device file, you are communicating with a driver. In the case of /dev/null, it acts as a pseudo device with a driver that has a very specific purpose. It discards anything you write to it, and only returns a EOF character when read. Sending an EOF more or less means making the process aware that no additional input will be sent (End of File).

这是一个特殊文件叫字符设备文件，它允许文件像未缓冲的设备（null 设备）一样工作，并且可以接收[流数据](https://www.putorius.net/linux-io-file-descriptors-and-redirection.html)。当你访问一个设备文件时，你就是与驱动程序在通信。对于 /dev/null 来说，它充当的是伪设备，这个伪设备具有特殊目的驱动程序。你传给它的所有东西都会被丢弃，并且在读它的时候只返回一个 EOF 字符，发送 EOF 意味着让进程知道发送其他任何输入数据了。

# Using the /dev/null File 使用 /dev/null 文件

## Suppressing Output with /dev/null 使用 /dev/null 抑制输出

The /dev/null file is most often used to suppress output. In the following example, we use the “>” redirection operator to send the output of the stat command to the bit bucket.

/dev/null 文件通常用来抑制输出，在接下来的例子中，我们会使用 ">" 重定向操作符将 stat 命令的输出发送到 bit bucket。

```
stat /etc/passwd > /dev/null
```

This effectively suppresses the commands output (standard output).

这有效抑制了命令输出（标准输出）。

We can use /dev/null to suppress errors by redirecting standard error (STDERR), like so:

我们可以使用 /dev/null 通过重定向标准错误（STDERR）来抑制错误，如下所示:

```
stat /etc/passwdss 2> /dev/null
```

We changed the filename to passwdss, which doesn’t exist. This would normally show a “No such file or directory” error. Although, the error was discarded since we redirected STDERR (standard error) output to /dev/null.

我们将文件名改为 passwdss，该文件名不存在，这通常会报 “文件不存在” 的错误。由于我们将 STDERR（标准错误）重定向到了 /dev/null，因此错误被丢弃。

![suppress-output](https://www.putorius.net/wp-content/uploads/2019/11/ani-diag1-min.gif.webp)

NOTE: To learn more about redirection and standard data streams (STDIN, STDOUT, STDERR) read "Linux I/O, Standard Streams, and Redirection".

注意：要了解更多关于重定向和标准数据流（STDIN，STDOUT，STDERR）的更多信息，请阅读 "[Linux I/O, Standard Streams, and Redirection](https://www.putorius.net/linux-io-file-descriptors-and-redirection.html)"

NOTE: Data sent to /dev/null is immediately discarded and not recoverable.

注意：发送到 /dev/null 的数据会立即被丢弃，并且无法恢复。

## Using /dev/null to input EOF (End of File) 使用 /dev/null 输入 EOF （文件结尾）

Another common use for /dev/null is supplying a blank or null input that has nothing but the EOF character. It may not be immediately clear why you would need such a function. However, there are a lot of utilities that need a EOF character in order to proceed. For example, the mailx utility will allow you to type an email from the command line. It will continue to wait for input until it receives an EOF character, which is sent by hitting CTRL + D. This tells the utility that you do not intended to send any more input.

/dev/null 的另一个常见用法是提供一个空白或 null 输入时，除了 EOF 外什么都没有，可能还不能理解为什么需要这个功能，但是许多程序需要 EOF 字符才能继续。例如，mailx 程序允许你从命令行输入电子邮件，它在接收到 EOF 字符前会一直等待输入，EOF 字符是通过 ctrl + d 发送的。发送 EOF 字符就意味着你不打算再输入更多了。

![eof-dev-null](https://www.putorius.net/wp-content/uploads/2019/11/ani-diag-mail-fast-min.gif.webp)

NOTE: EOT (End of transmission) and EOF (End of File) have technical differences which comes down to encoding and are outside the scope of this tutorial.'

注意：EOT（End of transmission） 和 EOF （End of File） 具有技术上的差异，这些差异归咎于编码，这不在本文讨论范围内。

What if we wanted to send a blank email, we would have to enter interactive mode and hit CTRL + D with no input. This would be unpractical if being used in a script or other unattended fashion. As a workaround we can redirect /dev/null into the mailx command to do this for us.

如果我们想发送空白邮件怎么办，我们将不得不进入交互模式并在没有任何输入的情况下按 ctrl + d。如果以脚本或其他无人值守的方式使用，这将不切实际。作为一种解决办法，我们可以将 /dev/null 重定向到 mailx 命令，为我们执行此操作。

```
$ mailx -s "Just Another Email" user@putorius.net < /dev/null
 Null message body; hope that's ok
```

# Using /dev/null to Empty a File 使用 /dev/null 清空文件

Yet another common use for the /dev/null file is to empty contents of a file. If you use the “>” redirection operator to redirect /dev/null into a file, it effectively erases the contents of that file.

/dev/null 文件的另一个常用用法是清空文件的内容，如果使用 ">" 重定向操作符将 /dev/null 重定向到文件，它将有效删除该文件的内容。

```
[savona@putor ~]$ cat scp.log
 Changing to backup directory…
 Sending SCP request to download fw-backup-2019_11_10…
 fw-backup-2019_11_10-03_03_27-2.0.32.zip      100%  536MB   2.8MB/s   03:14    
 [savona@putor ~]$ 
 [savona@putor ~]$ cat /dev/null > scp.log
 [savona@putor ~]$ 
 [savona@putor ~]$ ls -l scp.log
 -rw-r--r--. 1 savona savona 0 Nov 11 21:32 scp.log
 [savona@putor ~]$ cat scp.log 
 [savona@putor ~]$ 
```

Another option is to use the cp command to copy /dev/null over the file, like so:

另一种操作是使用 cp 命令在文件上复制 /dev/null，例如：

```
[savona@putor ~]$ cat scp.log 
 Changing to backup directory…
 Sending SCP request to download fw-backup-2019_11_10…
 fw-backup-2019_11_10-03_03_27-2.0.32.zip      100%  536MB   2.8MB/s   03:14
 [savona@putor ~]$ 
 [savona@putor ~]$ cp /dev/null scp.log 
 [savona@putor ~]$ 
 [savona@putor ~]$ cat scp.log 
 [savona@putor ~]$ ls -l scp.log 
 -rw-r--r--. 1 savona savona 0 Nov 11 21:34 scp.log
 [savona@putor ~]$ 
```

# Conclusion 总结

The bit bucket (/dev/null) is an important and useful tool for system administrators, devops engineers and just about anyone using the command line. In this article we discussed some of the most common uses for /dev/null, but once you know what it does, you may find more and more ways to use it.

Bit bucket (/dev/null) 对于管理员、开发工程师以及任何使用命令行的人来说都是一个重要且有用的工具。这篇文章中我们讨论了 /dev/null 的几种常见用法，不过一旦你知道了它是什么，你可能会发现更多使用 /dev/null 的方法。

We would love to hear any of your experiences, questions or comments below!

我们很期待在评论区听到你的使用经验、问题以及评论！

# Resource and Links: 参考链接

[Null Man Page](https://linux.die.net/man/4/null)

[Linux I/O, Standard Streams, and redirection](https://www.putorius.net/linux-io-file-descriptors-and-redirection.html)

[Bit Bucket – WikiPedia](https://en.wikipedia.org/wiki/Bit_bucket)

[Null Device – WikPedia](https://en.wikipedia.org/wiki/Null_device)

[1]: http://www.ruanyifeng.com/blog/2021/03/weekly-issue-149.html
[2]: https://www.putorius.net/introduction-to-dev-null.html