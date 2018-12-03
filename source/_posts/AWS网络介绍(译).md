---
title: AWS网络介绍(译)
date: 2018-11-30 10:32:21
tags:
categories: 翻译
thumbnail: https://a0.awsstatic.com/libra-css/images/logos/aws_logo_smile_1200x630.png
---

# Everything You Need To Know About Networking On AWS

Disclaimer: I’m not a network engineer and never have been - a tame network engineer has been consulted to ensure factual and terminological accuracy. The following is an in-exhaustive run down of everything I’ve learnt from building and using network infrastructure on Amazon Web Services. If you find you have no reference point for this information then have a poke around the “VPC” section of the AWS control panel (or get in touch to tell me I’m talking nonsense).

免责声明：我不是一个网络工程师，而且也从来没做过 - 已经向一位专业网络工程师咨询过确保相关术语和所描述的情况的准确性。以下是我在使用亚马逊网络服务过程中学习到的所有知识的详细说明。如果你不能从这篇文章中找到任何可参考的内容，那就在AWS的控制面板的“VPC“那里操作一下（或者直接告诉我我在说些废话）。

# Parts of a Network You Should Know About

# 关于网络你需要你知道的一些内容

If you’re running infrastructure and applications on AWS then you will encounter all of these things. They’re not the only parts of a network setup but they are, in my experience, the most important ones.

如果你在AWS上运行基础架构和应用程序，那么你就会遇到下面说的所有事情。它们不是网络设置中的所有，但是根据我的经验，他们是最重要的那些部分。

## VPC

A virtual private cloud - VPC - is a private network space in which you can run your infrastructure. It has an address space (CIDR range) which you choose e.g. 10.0.0.0/16. This determines how many IP addresses you can assign within the VPC. Each server you create inside the VPC will need an IP address so this address space defines the limit of how many resources you can have within the network. The 10.0.0.0/16 address space can use the addresses from 10.0.0.0 to 10.0.255.255, which is 65,536 IP addresses.

一个虚拟私有云（virtual private cloud）- VPC - 指的是你可以运行你自己的基础架构的私有网络空间。它有一你自己选择的地址空间（CIDR空间），例如：10.0.0.0/16.这决定了你有多少个VPC可以分配的IP地址。你在VPC中启动的每一个服务都需要一个IP地址，所以这个地址空间限制了你在网络中拥有的资源。10.0.0.0/16这个地址空间可以分配10.0.0.0到10.0.255.255的地址，也就是有65536个IP可以分配。

The VPC is the basis of your network on AWS and all new accounts include a default VPC with a subnets in each availability zone.

VPC是AWS网络以及所有新账户的基础，这些新账户包含每个可用区域中都有子网的VPC。

```
+---------------+
|     VPC       |     The Internet
|               |
|               |
|  10.0.0.0/16  |
|               |
|               |
+---------------+

```
## Subnets 子网

A subnet is a section of your VPC, with its own CIDR range and rules on how traffic can flow. Its CIDR range has to be a subset of the VPC’s, for example 10.0.1.0/24 which would allow for IPs from 10.0.1.0 to 10.0.1.255 giving 256 possible IP addresses.

子网就是你VPC中的一部分，它有自己的CIDR范围和流量流量传输规则。子网的CIDR范围必须是VPC的一个子网，比如0.0.1.0/24，它可以分配从10.0.1.0到10.0.1.255之间的IP地址，提供了256个可用IP地址空间。

Subnets are often denominated as ‘public’ or ‘private’ depending on whether traffic can reach them from outside the VPC (the Internet). This visibility is controlled by the traffic routing rules and each subnet can have its own rules.

子网通常可以分为”公有“和”私有“，具体取决于流量是否可以从VPC外部（Internet）到达子网。能不能访问取决于流量控制规则，每个子网都有自己的流量控制规则。

A subnet has to be in a specific availability zone within a region so it’s good practice to have a subnet in each zone. If you plan to have public and private subnets then there should be one of each per availability zone.

子网必须位于区域内的特定可用区域中，因此最好在每个区域中都有一个子网。如果你计划拥有公有和私有子网，那么每个可用区域都应该有一个子网。

```
+---------------------------+
|            VPC            |
|                           |
+------------+ +------------+
||  Subnet 1 | |  Subnet 2 ||
||10.0.1.0/24| |10.0.2.0/24||
||           | |           ||
|------------+ +------------|
+---------------------------+
```
## Availability Zones 可用区域

We’ve said that there should be subnets per availability zone, but what does that actually mean?

我们说过每个区域都应该有子网，但是这到底是什么意思？

Each AWS region is divided into 2 or more different zones which, between them, aim to guarantee a very high level of availability for that region. Essentially, at least one zone should be able to operate, even if others suffer outages (:fire:).

每个AWS区域都被分成2个或则更多不同的区域，主要是为了保证这些区域之间的高可用性。事实上，即使其他区域遭到损坏（比如火灾），至少也要有一个区域能够正常运行。

```
+----------+          +----------+
|us-ea)t-1a|          |us-east-1b|
|_____(____|          |__________|
|     )    |          |          |
|   ( &()  |          |    ✔     |
|  ) () &( |          |    8-)   |
+----------+          +----------+
```

## Routing Tables 路由表

A routing table contains rules about how IP packets in the subnets can travel to different IP addresses. There is always a default route table which will only allow traffic to travel locally, within the VPC. If a subnet has no routing table associated with it then it uses the default one. These would be ‘private’ subnets.

一个路由表包含指导子网内的IP包发送到不同IP地址的规则。通常会有一个只允许路由到本地的默认路由表。如果子网没有与之关联的子网，则它使用默认路由表，这个子网就会变为私有子网。

If you want external traffic to be able to get to a subnet then you need to create a routing table with a rule explicitly allowing this. Subnets associated to that routing table would be ‘public’.

如果你想要外部流量能够访问到子网，那么你需要创建一个包含这种规则的路由表，此时与路由表关联的子网称为公有子网。

All of the subnets in the default VPCs are associated with a route table which makes them public.

默认VPC中的所有子网都会关联一个路由表来使他们成为公有子网。

## Internet Gateways Internet网关

The routing table which makes a subnet public needs to reference an Internet gateway to allow the flow of external IP packets into and out of the VPC. You create your Internet gateway and then create a rule which says that packets to 0.0.0.0/0 - all IP addresses - need to go to there.

使子网变成公有子网的路由表需要引用因特网网关以允许外部IP包流入和流出VPC。创建你的因特网网关，然后创建一个允许所有IP地址都可访问的规则。

```
          Route table
         +-------------------+
         | 10.0.0.0/8: local | Requests within the VPC go over local connections.
      +--+ 0.0.0.0/0: ig-123 | Requests to any other IPs go via the Internet Gateway.
      |  |                   |
      |  +-------------------+
      |
      |
+-----+-------+          +-------------+
|  Subnet 1   |          |  Subnet 2   |
| 10.0.1.0/24 |          | 10.0.2.0/24 |
|             |          |             |
|             | 10.0.2.9 |             |
|             +--------->|             |
|             |          |             |
+-------+-----+          +-------------+
        | 8.8.4.4
        |
        |   +--------+
        +-->| ig-123 |
            |        +-----> The Internet
            +--------+
```

## NAT Gateways NAT网关

If you have an EC2 instance in a private subnet - one which doesn’t allow traffic from the Internet to reach it - then there’s also no way for IP packets to reach the Internet. We need a mechanism for sending those packets out, and then routing the replies correctly. This is called network address translation and is very likely done in your house by your wifi router.

如果私有子网中有EC2实例，该子网是不允许来自互联网的访问的，同样，子网内的IP数据包也是不能流向因特网的。我们需要一个能够将这些网络包发送出去并且正确获取它们返回值的机制。上面这个过程称为网络地址转换，很像你家里的wifi路由器做的事。

A NAT gateway is a device which sits in the public subnets, accepts any IP packets bound for the Internet coming from the private subnets, sends those packets on to their destination and then sends the returning packets back to the source.

NAT网关是位公共子网中的设备，接收来自私有子网的所有Internet请求IP数据包，然后将这些数据包转发到相应目标，最后将返回的数据包正确返回给发送方。

It’s not necessary to have NAT gateways if you don’t intend instances in your private subnets to talk outside if your VPC but if you do need to do that e.g. using an external API, SaaS database etc. then you can simply set up an EC2 instance (might be cheaper, depending on your traffic), configured appropriately, or use an AWS managed NAT gateway resource (will be easier to manage because you won’t be doing it).

如果你不打算私有子网和外部Internet进行通信，那么就没必要拥有NAT网关。你可以简单地设置EC2实例（可能更便宜，具体取决于你的流量），配置得当，或者使用AWS管理的NAT网关资源（跟容易管理，因为你不会亲自做这些事）。

```
  +---------------------+
  |   Public Subnet     |
  |   10.0.1.0/24       |
  |                     |
  |    +------------+   |
  |    |            +----------->   The Internet
  |    |  nat-123   |   |
  |    |            |   |
  |    +-------^----+   |
  |            |        |
  +------------|--------+
               |
               | 8.8.4.4
               |                       Route table
  +------------+---------+    +---------------------+
  |  Private Subnet      +----+  10.0.0.0/16: local |
  |  10.0.20.0/24        |    |  0.0.0.0/0: nat-123 |
  |                      |    +---------------------+
  +----------------------+
```

- The public subnet contains the NAT gateway
- 公有子网包含NAT网关

- A request is made from the private subnet to an IP address somewhere on the Internet
- 请求从私有子网到Internet上某处IP地址

- The route table says that it needs to go to the NAT gateway
- 路由表说他需要转发到NAT网关

- The NAT gateway sends it on
- NAT网关发送它


## Security Groups 安全组

VPC network Security groups denote what traffic can flow to (and from) EC2 instances within your VPC. A security groups can specify ingress (inbound) and egress (outbound) traffic rules, limiting them to certain sources (inbound) and destinations (outbound). They are associated with EC2 instances rather than subnets.

VPC网络安全组表示哪些流量可以流入（或者流出）VPC中的EC2实例。安全组可以指定流量规则的入口和出口，将他们限制为某些源和目的地址可以流过，他们与EC2实例相关联，而不是子网。

By default all traffic is allowed out, but no traffic is allowed in. Inbound rules can specify a source address - either a CIDR block or another security group - and a port range. When the source is another security group then that must be within the same VPC. For example, a VPC is created with a default security group which allows traffic from anything which has that same security group. Assigning the group to everything created in the VPC (not necessarily the most secure practice) means that all those resources can talk to each another.

默认情况下所有流量都可以流出，但是任何流量都无法流入。入口规则可以指定源地址和端口范围。当源地址是另一个安全组时，这必须是在同一个VPC中发生。比如使用默认安全组创建VPC，改安全组允许来自具有相同安全组的任何流量。将组分配给VPC中创建的所有内容意味着所有这些资可以互相通信。

```
                   +---------------+
                   | sg-abcde      |
                   | ALLOW TCP 443 |
                   +----+----------+
                         |
                    +----+------+
                    |  i-67890  |
 10.0.1.123:22      |           | 10.0.1.123:443
------------------>X|           <----------------
                    |           |
                    +-----------+
```

- An instance (i-67890) has a security group (sg-abcde) which allows TCP traffic on port 443
- 实例（i-67890）具有安全组（sg-abcde），其允许端口443上的TCP流量
- A request is made to its IP address (10.0.1.123) on port 22 which doesn’t get through
- 请求端口22上的IP地址（10.0.1.123）无法通过
- A request is made to port 443 on the instance and the traffic is allowed
- 向实例上的端口443发出请求，并允许通信

# Putting it All Together 合到一起

The complete picture of your virtual private network looks something like the picture below, with public and private subnets spread across availability zones, network address translation sitting in the public subnets and route tables to specify how packets are routed. EC2 instances are run in any subnet and have security groups attached to them.

你的虚拟私有云网络的完整图片如下所示，公共和私有子网分布在可用区域中，网络地址转换位于公共子网和路由表中以指定数据包的路由方式。EC2实例在所有子网中运行，并附加到安全组。

```
                                        +-------+                                  
                                        | ig-1  |                                  
                                        |       |                                  
        vpc-123: 10.0.0.0/16  |         |       |        |                         
       +----------------------+---------+-------+--------+---------------------+
       |                      |                          |                     |   
       |  +-----+             |  +-----+                 |  +-----+            |   
       |  | NAT |             |  | NAT |                 |  | NAT |            |   
public |  |     |             |  |     |                 |  |     |            |   
subnets|  +-----+             |  +-----+                 |  +-----+            |   
       |                      |                          |                     |   
       |                      |                          |                     |   
       |                      |                          |                     |   
       |              +-------+                  +-------+             +-------+
       |              | rt-1a |                  | rt-1b |             | rt-1c |
       | 10.0.1.0/24  |       | 10.0.2.0/24      |       | 10.0.3.0/24 |       |   
-------+-----------------------------------------------------------------------+
       | 10.0.4.0/24  | rt-2a | 10.0.5.0/24      | rt-2b | 10.0.6.0/24 | rt-2c |
       |              |       |                  |       |             |       |   
       |              +-------+                  +-------+             +-------+
private|                      |                          |                     |   
subnets|                      |                          |                     |   
       |                      |                          |                     |   
       |                      |                          |                     |   
       |                      |                          |                     |   
       |                      |                          |                     |   
       |                      |                          |                     |   
       |                      |                          |                     |   
       +----------------------+--------------------------+---------------------+
       |         AZ 1         |          AZ 2            |        AZ 3         |
```

翻译得很水，凑合看吧，主要看图！

2018-12-3 北京 雾霾
