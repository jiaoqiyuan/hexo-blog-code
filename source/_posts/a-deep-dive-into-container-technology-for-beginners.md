---
title: 解开容器的神秘面纱-面向新手的容器技术深入解析
date: 2019-07-17 13:37:06
tags:
categories: 翻译
photos: https://cdn-images-1.medium.com/max/800/1*zUYwSHCA2wFF8BW8E7D92w.png
---

信息来源：[阮一峰每周分享](http://www.ruanyifeng.com/blog/2019/07/weekly-issue-64.html)

文章来源：[Demystifying Containers 101: A Deep Dive Into Container Technology for Beginners](https://www.freecodecamp.org/news/demystifying-containers-101-a-deep-dive-into-container-technology-for-beginners-d7b60d8511c1/)

-----

## Introduction 简介

Regardless of whether you are a student in school, a developer at some company, or a software enthusiast, chances are you heard of containers. You may have also heard that containers are lightweight virtual machines, but what does that really mean, how exactly do containers work, and why are they so important?

无论你是在校的学生、公司里的开发人员、软件爱好者，你都应该听说过容器。你可能也听说过容器是轻量级的虚拟机，但这到底指的是什么意思，容器到底是怎样工作的，为什么容器如此重要？

<!--more-->

This story serves as a look into containers, their key great technical ideas, and the applications. I won’t assume any prior knowledge in this field other than a basic understanding of computer science.

下面讲的就是容器相关的内容，包括容器的关键的伟大思想，以及容器的应用。除了需要对计算机有基本的了解外，你不需要其他的专业知识来阅读这篇文章。

## The Kernel and the OS 内核与操作系统

Your laptop, along with every other computer, is built on top of some pieces of hardware like the CPU, persistent storage (disk drive, SSD), memory, network card, etc.

你的笔记本以及其他的计算设备都是建立在其他顶级硬件基础比如 CPU，持久化存储（硬盘，固态硬盘），内存，网络等之上。

To interact with this hardware, a piece of software in the operating system called the kernel serves as the bridge between the hardware and the rest of the system. The kernel is responsible for scheduling processes (programs) to run, managing devices (reading and writing addresses on disk and memory), and more.

为了与硬件进行交互，操作系统中的一些叫做内核服务的软件就在硬件和操作系统其他软件之间充当了桥梁的作用。内核用于调度进程（程序）的运行，管理设备（从硬盘和内存中读写数据）等等。

The rest of the operating system serves to boot and manage the user space, where user processes are run, and will constantly interact with the kernel.

操作系统剩下的部分用于启动和管理运行用户进程的用户空间，并不断与内核进行交互。

![1](https://upload-images.jianshu.io/upload_images/12148317-a95146a3dccea43e?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

The kernel is part of the operating system and interfaces with the hardware. The operating system as a whole lives in the “kernel space” while user programs live in the “user space”. The kernel space is responsible for managing the user space.

内核是操作系统的一部分，也是操作系统与硬件之间的接口。操作系统作为一个整体存在于“内核空间”，而用户的进程存在于“用户空间”。内核空间负责管理用户空间。

## The Virtual Machine 虚拟机

So you have a computer that runs MacOS and an application that is built to run on Ubuntu. Hmmm… One common solution is to boot up a virtual machine on your MacOS computer that runs Ubuntu and then run your program there.

那么你有一个运行 MacOS 的计算机和一个运行于 Ubuntu 上的应用程序。(⊙o⊙)…一个常用的办法是在你的 MacOS 计算机上启动一个运行 Ubuntu 的虚拟机，然后运行这个程序。

A virtual machine is comprised of some level of hardware and kernel virtualization on which runs a guest operating system. A piece of software called a hypervisor creates the virtualized hardware which may include the virtual disk, virtual network interface, virtual CPU, and more. Virtual machines also include a guest kernel that can talk to this virtual hardware.

虚拟机由一定级别的硬件和内核虚拟化组成，在这个虚拟化内核上可以运行用户的操作系统。一块称之为虚拟化管理的程序创建了包括虚拟磁盘、虚拟网络接口、虚拟cpu等等的虚拟硬件，虚拟机通常包含了一个可以与虚拟硬件交互的访客内核。

The hypervisor can be hosted, which means it is some software that runs on the Host OS (MacOS) as in the example. It can also be bare metal, running directly on the machine hardware (replacing your OS). Either way, the hypervisor approach is considered heavy weight as it requires virtualizing multiple parts if not all of the hardware and kernel.

管理程序可以被托管，这意味着它是一些在主机操作系统（MacOS）上运行的软件，如示例中所示。管理程序也可以运行在裸机上，也就是直接运行在硬件设备（取代你的操作系统）。无论哪种方式，管理程序都被认为是重量级的，因为它需要虚拟化多个部分（如果不是全部硬件和内核）。

When there needs to be multiple isolated groups on the same machine, running a VM for each of these groups is way too heavy and wasteful of resources to be a good approach.

当在同一台机器上需要多个隔离的组时，为每个组运行VM是一种非常吃力而且浪费资源的方式。

![2](https://upload-images.jianshu.io/upload_images/12148317-eaf062ea534db904?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

VMs require hardware virtualization for machine level isolation whereas containers operate on isolation within the same operation system. The overhead difference becomes really apparent as the number of isolated spaces increase. A regular laptop can run tens of containers but can struggle to run even one VM well.

VM需要硬件虚拟化才能实现机器级隔离，而容器只需要在同一操作系统内进行隔离操作。随着隔离空间数量的增加，开销差异变得非常明显。 普通的笔记本电脑可以运行数十个容器，但是甚至很难运行一个VM。

## cgroups

In 2006, engineers at Google invented the Linux “control groups”, abbreviated as cgroups. This is a feature of the Linux kernel that isolates and controls the resource usage for user processes.

2006年，谷歌的工程师发明了Linux“控制组”，缩写为cgroups。这是Linux内核的一项功能，可隔离和控制用户进程的资源使用情况。

These processes can be put into namespaces, essentially collections of processes that share the same resource limitations. A computer can have multiple namespaces, each with the resource properties enforced by the kernel.

这些进程可以放入名称空间，实质上是共享相同资源限制的进程集合。 计算机可以有多个名称空间，每个名称空间都有内核强制执行的资源属性。

The resource allocation per namespace can be managed in order to limit the amount of the overall CPU, RAM, etc that a set of processes can use. For example, a background log aggregation application will probably need to have its resources limit in order to not accidentally overwhelm the actual server it’s logging.

每个命名空间的资源分配都可以顺序地进行管理，以便限制一组进程可以使用的总CPU，RAM等的数量。例如，后台日志聚合应用程序可能需要限制其资源，以免意外地影响运行这个日志聚合应用程序的服务器。

While not an original feature, cgroups in Linux were eventually reworked to include a feature called namespace isolation. The idea of namespace isolation itself is not new, and Linux already had many kinds of namespace isolation. One common example is process isolation, which separates each individual process and prevents such things like shared memory.

虽然不是原始功能，但Linux中的cgroup最终被重新设计为包含名为名称空间隔离的功能。 命名空间隔离的想法本身并不新鲜，Linux已经有多种命名空间隔离。 一个常见的例子是进程隔离，它将每个单独的进程分开并防止共享内存之类的事情发生。

Cgroup isolation is a higher level of isolation that makes sure processes within a cgroup namespace are independent of processes in other namespaces. A few important namespace isolation features are outlined below and pave the foundation for the isolation we expect from containers.

Cgroup隔离是一种更高级别的隔离，可确保cgroup命名空间中的进程独立于其他命名空间中的进程。 下面指出了一些重要的命名空间隔离功能，为我们对容器的隔离奠定了基础。

- PID (Process Identifier) Namespaces: this ensures that processes within one namespace are not aware of process in other namespaces.

- PID（流程识别符）：这可以确保一个名称空间内的进程不知道其他名称空间中的进程。

- Network Namespaces: Isolation of the network interface controller, iptables, routing tables, and other lower level networking tools.

- 网络命名空间：网络控制接口、防火墙、路由表以及其他低级网络工具的隔离。

- Mount Namespaces: Filesystems are mounted, so that the file system scope of a namespace is limited to only the directories mounted.

- 载入命名空间：文件系统被挂载后，命名空间所属的文件系统范围就被仅仅被限定在这个已挂载的路径下。

- User Namespaces: Limits users within a namespace to only that namespace and avoids user ID conflicts across namespaces.

- 用户命名空间：使用命名空间限制用户只属于那个命名空间，并且避免用户ID与命名空间冲突。

- To put it simply, each namespace would appear to be its own machine to the processes within it.

- 简而言之，每个命名空间看起来都属于用于运行进程的物理设备。

## Linux Containers  Linux 容器

Linux cgroups paved the way for a technology called linux containers (LXC). LXC was really the first major implementation of what we know today to be a container, taking advantage of cgroups and namespace isolation to create virtual environment with separate process and networking space.

Linux cgroups 为一种叫做 Linux 容器（LXC）的技术铺平了道路。LXC 是我们今天称之为容器的首次真正的实现，它吸取了 cgroups 和 命名空间的隔离性优点，创建了一个带有独立进程和网络空间的虚拟环境。

In a sense, this allows for independent and isolated user spaces. The idea of containers follows directly from LXC. In fact, earlier versions of Docker were built directly on top of LXC.

从某种意义上说，它允许独立和隔离的用户空间。容器的概念直接继承了 LXC。事实上，早期版本的容器就是直接建立在 LXC 之上的。

## Docker

Docker is the most widely used container technology and really what most people mean when they refer to containers. While there are other open source container techs (like rkt by CoreOS) and large companies that build their own container engine (like lmctfy at Google), Docker has become the industry standard for containerization. It is still built on the cgroups and namespacing provided by the Linux kernel and recently Windows as well.

Docker 是使用最广泛的容器而且是人们通常表示容器时就是指的 Docker。虽然有其他的容器技术（比如 CoreOS 的 rkt）以及大公司构建自己的容器引擎（比如 Goole 的 lmctfy），Docker 已经成为容器化的工业标准。容器还是建立在 由 Linux 内核提供的 cgroups 和 命名空间之上， 包括最近的 windows 上也能运行容器。

![3](https://upload-images.jianshu.io/upload_images/12148317-c582819e102dc78c?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

A Docker container is made up of layers of images, binaries packed together into a single package. The base image contains the operating system of the container, which can be different from the OS of the host.

一个 Docker 容器由多层 images 和 binaries 组合在一个包中构成。基础镜像包含容器的操作系统，这个操作系统与主机的操作系统不同。

The OS of the container is in the form an image. This is not the full operating system as on the host, and the difference is that the image is just the file system and binaries for the OS while the full OS includes the file system, binaries, and the kernel.

容器的操作系统存在于镜像中。这更主机中的完整操作系统不同，区别在于这个镜像只包含操作系统的文件系统和二进制，而完整的操作系统包括文件系统、二进制程序和内核。

On top of the base image are multiple images that each build a portion of the container. For example, on top of the base image may be the image that contains the `apt-get` dependencies. On top of that may be the image that contains the application binary, and so on.

基础镜像之上还有多个镜像，每个镜像构建容器的一部分。比如，在基础镜像之上可能包含 `apt-get` 依赖的镜像。在那之上的镜像可能包含二进制应用程序等等的东西。

The cool part is if there are two containers with the image layers `a, b, c` and `a, b, d`, then you only need to store one copy of each image layer `a, b, c, d` both locally and in the repository. This is Docker’s union file system.

比较酷的地方是如果有两个容器分别包含 `a, b, c` 层的镜像和 `a, b, d` 层的镜像，然后你只需要在本地和仓库中保存每层镜像的一个拷贝也就是 `a, b, c, d`。这是 Docker 的独立文件系统。 

![4](https://upload-images.jianshu.io/upload_images/12148317-1674d3b03bc89cc1?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Each image, identified by a hash, is just one of many possible layers of images that make up a container. However a container is identified only by its top level image, which has references to parent images. Two top level images (Image 1 and Image 2) shown here share the first three layers. Image 2 has two additional configuration related layers, but shares the same parent images as Image 1.

每个由 hash 标示的镜像可能仅仅是构成容器的几个层中的那些镜像中的一个。但是一个容器仅仅由其最顶层的镜像标示，这个顶层镜像与他的父镜像存在联系。两个顶层镜像（image1 和 image2）共享前三层。image2 有两个额外的配置相关的层，但是与 image1 共享相同的父镜像。

When a container is booted, the image and its parent images are downloaded from the repo, the cgroup and namespaces are created, and the image is used to create a virtual environment. From within the container, the files and binaries specified in the image appear to be the only files in the entire machine. Then the container’s main process is started and the container is considered alive.

当一个容器启动时，从仓库中下载镜像以及其父镜像，然后创建 cgroups 和命名空间，同时这个镜像也用于创建虚拟环境。在容器内，镜像中的文件和二进制程序似乎是这个机器中仅有的文件。然后容器的主进程开始执行，容器就诞生了。

Docker has some other really really cool features, such as copy on write, volumes (shared file systems between containers), the docker daemon (manages containers on a machine), version controlled repositories (like Github for containers), and more. To learn more about them and see some practical examples of how to use Docker, this Medium [article](https://medium.freecodecamp.org/a-beginner-friendly-introduction-to-containers-vms-and-docker-79a9e3e119b) is extremely useful.

Docker 还有一些其他非常非常酷的特性，比如写时复制，卷机制（与其他容器共享文件系统），docker 守护进程，版本控制仓库（容器版的 Github）等等。想要学习更多关于 Docker 的内容以及如何使用 Docker 实例的话，[这篇文章](https://medium.freecodecamp.org/a-beginner-friendly-introduction-to-containers-vms-and-docker-79a9e3e119b)会非常有用

![5](https://upload-images.jianshu.io/upload_images/12148317-d416830ad8a450bf?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

A command line client (1) tells a process on the machine called the docker daemon (2) what to do. The daemon pulls images from a registry/repository (3). These images are cached (4) on the local machine and can be booted up by the daemon to run containers (5). Image Source: Docker

client（1） 的命令行告诉机器上的一个叫做 docker daemon（2） 的进程该做什么。daemon 程序从远程仓库registry/repository (3) 拉取镜像到本地。这些镜像缓存（4）在本地，daemon 程序通过这些镜像来运行容器（5）。镜像资源： Docker。

## Why Containers 为什么需要容器

Aside from process isolation, containers have many other beneficial properties.

除了进程隔离，容器还有其他不错的特性。

The container serves as a self isolated unit that can run anywhere that supports it. And in each of these instances, the container itself will be exactly identical. It won’t matter if the host OS is CentOS, Ubuntu, MacOS, or even something non UNIX like Windows — from within the container the OS will be whatever OS the container specified. Thus you can be sure the container you built on your laptop will also run on the company’s servers.

容器作为独立的单元向外提供服务，可以运行在任何支持容器地方。在每个使用场景下，容器本身将会是完全相同的。不管主机操作系统是 MacOS、Ubuntu、CentOS，甚至是一些非类 Unix 的操作系统比如 Windows，容器可以部署运行在任意操作系统中。因此你可以确定你在自己笔记本上运行的容器同样可以运行在你公司的服务器上。

The container also acts as a standardized unit of work or compute. A common paradigm is for each container to run a single web server, a single shard of a database, or a single Spark worker, etc. Then to scale an application, you simply need to scale the number of containers.

容器通常也是工作或计算机的标准单元。一个常见的范例是每个容器运行单个Web服务器，数据库的单个分片或单个Spark工作程序等。然后，为了扩展应用程序，您只需要扩展容器的数量。

In this paradigm, each container is given a fixed resource configuration (CPU, RAM, # of threads, etc) and scaling the application requires scaling just the number of containers instead of the individual resource primitives. This provides a much easier abstraction for engineers when applications need to be scaled up or down.

在这个范例中，每个容器被合理配置（CPU，RAM，线程数等等）而且要扩展应用程序只需要扩大容器的数量而不是个别资源。这在程序需要扩展和收缩时为工程师提供了极大的便利。

Containers also serve as a great tool to implement micro service architecture, where each microservice is just a set of co-operating containers. For example the Redis micro service can be implemented with a single master container and multiple slave containers.

容器也是微服务架构中一个非常棒的工具，这些微服务就是一系列相互合作的容器。比如 Redis 微服务就可以使用一个单独的主容器和多个从容器实现。

This (micro)service orientated architecture has some very important properties that make it easy for engineering teams to create and deploy applications (see my earlier [article](https://hackernoon.com/how-microservices-saved-the-internet-30cd4b9c6230) for more details).

这个定向的微服务架构有很多让工程师团队易于创建和部署应用程序的重要属性（可以查看[我的文章](https://hackernoon.com/how-microservices-saved-the-internet-30cd4b9c6230)来获取更多细节）。

## Orchestration 编配

Ever since the time of linux containers, users have tried to deploy large scale applications over many virtual machines where each process runs in its own container. Doing this required being able to efficiently deploy tens to thousands of containers across potentially hundreds of virtual machines and manage their networking, file systems, resources, etc. Docker today makes this a little easier as it exposes abstractions to define container networking, volumes for file systems, resource configurations, etc.

自从 Linux 容器时代开始，用户就试图在虚拟机上部署大型应用程序，这些虚拟机上的每个进程都运行在自己的容器中。要做到这些需要能够高效地在可能数百台虚拟机上部署成百上千个容器，并且要管理好他们的网络、文件系统、资源等内容。今天的 Docker 让这些变得容易了一点，因为它将容器的网络、文件系统卷、资源配置等都抽象并暴露出来了。

However a tool is still needed to:

但是仍需要一个工具来：

- actually take a specification and assign containers to machines (scheduling)

- 采用实际的规范并将容器交由机器（调度）

- actually boot the specified containers on the machines through Docker

- 通过Docker在机器上启动指定的容器

- deal with upgrades/rollbacks/the constantly changing nature of the system

- 处理升级/回滚/系统不断变化的特性

- respond to failures like container crashes

- 报告如容器宕机的错误

- and create cluster resources like service discovery, inter VM networking, cluster ingress/egress, etc.

- 创建集群资源如服务发现、虚拟机间的网络、集群入口/出口等。

This set of problems relates to the orchestration of a distributed system built on top of a set of (possibly transient or constantly changing) containers, and people have built some really miraculous systems to solve this problem.

这一系列问题涉及在一系列（可能是瞬态的或不断变化的）容器之上构建的分布式系统的编排，人们已经构建了一些非常奇妙的系统来解决这个问题。

In my next story I will talk in depth about the implementation of Kubernetes, the major open source orchestrator, along with two equally important but lesser known ones, Mesos and Borg.

在我下一篇文章中我将会深入讨论主要的开源协调器 Kubernetes 的实现，以及两个同样重要但鲜为人知的 Mesos 和 Borg 。

This story is part of a series. I am an undergrad at UC Berkeley. My research is in distributed systems and I am advised by Scott Shenker.

这篇文章是系列中的一部分，我是 UC Berkeley 的一名本科生。我的主要研究方向是由 Scott Shenker 建议的分布式系统