---
title: UDP协议到底可不可靠(译)
date: 2018-12-07 06:00:57
tags:
categories: 翻译
---

信息来源：[阮一峰每周分享][1]

文章来源：[How unreliable is UDP?][2]

-------

## How unreliable is UDP?

I realized something recently: I know virtually nothing about UDP. Oh, I know it's connectionless, has no handshaking and thus doesn't provide any guarantees about delivery or ordering. But, in practice, what does that actually mean?

我最近留意到：我实质上完全不了解UDP。(⊙o⊙)…，我知道它是无连接式通讯，没有握手因此不为使用者提供任何可靠数据交付。但是在实践中这究竟意味着什么？

I setup 5 VPS to send each other a few UDP packets over a 7 hour period. I didn't send much traffic (though that's certainly worth trying). Each server, every 9-11 second, randomly picked a target and sent 5-10 packets ranging from 16 to 1016 bytes.

我开启了5个VPS在7小时内互相发送了一些UDP数据包。我没有发送太多流量（尽管这确实很值得尝试）。每个服务端，每9到11秒随机选择一个目标发送5到10个大小在16到1016字节的数据包。

2 servers were in the same data center in New Jersey. 1 each in LA, Amsterdam and Tokyo.

两台服务器位于新泽西州的同一个数据中心，另外三个分别位于洛杉矶，阿姆斯特丹和东京。

## [Un]Reliability （不）可靠

The first thing I wanted to know was how unreliable UDP was. Are we talking about a delivery rate of 25%? 50%? 75%?

首先我想知道UDP到底有多不可靠。是我们谈到的25%,50%，75%的交付率吗？

|  | NJ1 | NJ2 |LA|NLD|JPN|
|:--:|:--:|:---:|:--:|:---:|:---:|
| NJ1 | - | 2981/2981 | 2888/2889 | 2964/2964 | 3053/3054 |
| NJ2 | 3016/3016 | - | 3100/3101 | 2374/2375 | 3054/3054 |
| LA  | 2901/2941 | 2932/2975 | - | 2938/2942 | 2712/2712 |
| NLD | 3038/3038 | 2771/2772 | 2724/2724 | - | 2791/2791 |
| JPN | 2551/2552 | 2886/2886 | 2836/2838 | 2887/2887 | - |


These numbers were better than what I had expected. I was specifically thinking NLD <-> JPN would see above normal loss, but there was none. Data being sent out of LA, specifically to the two servers in NJ, seems to have struggled some. Was there a pattern?

这些数字信息比我预想的要好。我还特意认为阿姆斯特丹到东京的数据包的丢失率回避正常情况下的要大，但结果完全不是这样的。从洛杉矶发出的数据包，尤其是发往新泽西州的两台服务器的数据包似乎碰到了些阻碍。有什么模式造成了这种情况的发生吗？

First, I thought maybe the size of the packet would be an issue. Admittedly, I kept them small (16 byte header, 0-1000 byte payload):

首先，或许数据包的大小对结果造成了这种影响，不可否认，我已经保持数据包很小了（16字节的头，0-1000字节的有效载荷）：

package loss per size(bytes):
| 0-115 | 116-215 | 216-315 | 316-515 | 516-715 | 716-915 |
|:--:|:--:|:---:|:--:|:---:|:---:|
| 13 | 11 | 12 | 13 | 23 | 23 |

Nothing obvious there. Did the packet loss happen around the same time? Unfortunately, I didn't keep timestamps (why?!), but I did keep a counter per pair. If you look at the 43 packets that failed to make it from LA to NJ2, 29 were lost during 2 ~1 minute periods. The NJ1 packet loss also largely happened during 2 short periods.

看起来数据包的大小对丢包率没产生什么太大的影响。数据包丢失的情况都发生在同一时间吗？不幸的是，我没有保留时间戳（天啊，为什么？！），但是我在每个时期都保留了计数器。如果你看一下洛杉矶到NJ2失败的43个网络报，其中有29个是在2到1分钟内丢失的。NJ1的数据包丢失也主要发生在连个很短的周期内。

## Ordering 排序

The other thing I was interested int was ordering.

我关心的另一件事是UDP的传输顺序。

The first way I looked at this was to measure the inversion of the array. Essentially, that's the number of pairs that are out of order. If you have an array with the values 10, 8, 3, 7, 4, you end up having to do 8 swaps ((10, 8), (10, 3), (10, 7), (10, 4), (8, 3), (8, 7), (8, 4), (7, 4)).

我第一眼看到这个是先度量数组的逆序度，实际上就是数组不按顺序排列的程度。如果你有一个数值为10,8,3,7,4的数组，你最后需要做8个数据交换才能给数组排序((10, 8), (10, 3), (10, 7), (10, 4), (8, 3), (8, 7), (8, 4), (7, 4)).

|  | NJ1 | NJ2 |LA|NLD|JPN|
|:--:|:--:|:---:|:--:|:---:|:---:|
| NJ1 | - | 0 | 2994 | 2581 | 4658 |
| NJ2 | 0 | - | 3147 | 2459 | 4645 |
| LA  | 3980 | 3861 | - | 3237 | 4010 |
| NLD | 3125 | 1826 | 3133 | - | 4189 |
| JPN | 3920 | 4417 | 4147 | 4425 | - |


Don't know about you, but I'm not sure I find that useful. It sure seems high. Of course, one of the reasons to use UDP is when you're able to discard some packets. If you send 10 000 packets, and they're all ordered, except that the last one is somehow first, you can just discard it rather than doing 9999 swaps.

不了解你，但是我不确定这个方法是有用的。这看起来肯定很高，显然，使用UDP的其中一个原因就是你可以丢失一些数据包时。如果你发送了10000个数据包，而且他们是有序的，除了这种情况：最后一个数据包在最前面的位置，你大可以直接丢弃它而不是进行9999次数据交换来排序。

What if we discard any packet that come after a later packet we've already processed (later meaning the counter is great)? For example, if we get 1, 5, 4, 3, 6, 7, we'd discard 4 and 3 since we've already seen 5. How many "good" packets would that leave?

如果我们处理完一个数据包后丢弃了后面发来的数据包会怎么样（后来的意味着计数器会很大）？如果我们收到了1, 5, 4, 3, 6, 7，我们看到5后4和3的数据包，那么还剩下多少“好”的数据包呢？

|  | NJ1 | NJ2 |LA|NLD|JPN|
|:--:|:--:|:---:|:--:|:---:|:---:|
| NJ1 | - | 2981 | 1514 | 1658 | 1123 |
| NJ2 | 3016 | - | 1627 | 1483 | 1161 |
| LA  | 3980 | 3861 | - | 3237 | 4010 |
| NLD | 3125 | 1826 | 3133 | - | 4189 |
| JPN | 3920 | 4417 | 4147 | 4425 | - |

换算成百分比后：

|  | NJ1 | NJ2 |LA|NLD|JPN|
|:--:|:--:|:---:|:--:|:---:|:---:|
| NJ1 | - | 100 | 52.40 | 55.94	| 36.77 |
| NJ2 | 100 | - | 52.47 | 54.22 | 38.02 |
| LA  | 41.72 | 42.32 | - | 50.48 | 39.34 |
| NLD | 46.32 | 59.34 | 44.79 | - | 39.27 |
| JPN | 38.40 | 37.53 | 40.20 | 37.65 | - |

As a slight tweak, what if we group 5 packets together, sort them, then re-apply the above discarding code:

稍微调整一下，如果我们把5个数据包分成一个组，对他们进行排序，然后重复上面的丢包过程。

|  | NJ1 | NJ2 |LA|NLD|JPN|
|:--:|:--:|:---:|:--:|:---:|:---:|
| NJ1 | - | 2981 | 2061 | 2235 | 1807 |
| NJ2 | 3016 | - | 2214 | 2041 | 1889 |
| LA  | 1868 | 1873 | - | 2066 | 1720 |
| NLD | 2200 | 2273 | 1920 | - | 1712 |
| JPN | 1541 | 1804 | 1735 | 1732 | - |

换算成百分比后：

|  | NJ1 | NJ2 |LA|NLD|JPN|
|:--:|:--:|:---:|:--:|:---:|:---:|
|NJ 1|	-|	100|	71.34|	75.40|	59.17
|NJ 2|	100|	-|	71.40|	74.63|	61.85
|LA	|63.52|	62.96|	-|	70.22|	63.42
|NLD|	72.42|	82.00|	70.48|	-|	61.34
|JPN|	60.38|	62.51|	61.13|	59.99|	-

## Conclusion 结论

It's hard to draw any conclusions without running this for longer and with more data. Still, it seems that UDP reliability is pretty good. Distance usually involves more hops and each hop increases the risk or something going bad, but if things are normally ok, then distance doesn't seem to be an issue.

在没有更长运行时间和发送更多数据之前很难下结论，目前来看，UDP的可靠性是非常不错的，距离通常意味着更多跳跃，更多跳增加了数据传输出错的风险，但是如果通常情况下数据发送是好的，那么距离似乎就不是个问题了。

What is an issue is ordering. Here, distance does appear to play a bigger factor. By grouping the packets we see a substantial and expected improvement. In a lot of cases, ordering might not matter. Unless you're streaming, it's possible that simply keeping a timestamp and re-ordering on the receiving side would work.

问题是排序，这时，距离似乎产生了比较大的影响。通过把数据分组我们看到了可预期的很大改进。在许多情况下，排序或许没那么重要。除非你是流处理，简单地保留个时间戳并且在接收端重新排序似乎是个解决办法。

I'd like to test more things. More data for a longer period of time and more locations. I'd also like to compare the performance to TCP. But, overall, I feel that the better-than-I-expected reliability makes UDP something I should keep in my toolbox.

我很乐意测试更多情况。更多的数据，更长的时间间隔和更多的位置。我也会跟TCP的性能做一下比较。但是，总体来说，UDP传输的可靠性比我预想得要高，我会把它放到我的工具箱中。

2018.12.07 北京 -8°

[1]: http://www.ruanyifeng.com/blog/2018/12/weekly-issue-34.html
[2]: https://www.openmymind.net/How-Unreliable-Is-UDP/