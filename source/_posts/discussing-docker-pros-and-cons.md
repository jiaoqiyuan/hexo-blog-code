---
title: Docker技术的优缺点
date: 2019-01-26 18:06:16
tags: docker
categories: 翻译
thumbnail: https://cdn-images-1.medium.com/max/1600/0*KBn45TeUMJZSbz9n.png
photos: https://cdn-images-1.medium.com/max/1600/0*KBn45TeUMJZSbz9n.png
---

信息来源：[阮一峰每周分享][1]

文章来源：[Discussing Docker. Pros and Cons.][2]

----

## Discussing Docker. Pros and Cons. 讨论一下Docker的优缺点

Docker allows us to easily create reproducible environments for our application. We automate the setup of the environment and eliminate manual error-prone tasks. This way we reduce the risks and the reliability of the deployment process. But there are also challenges and domains, where the usage of Docker can be difficult. This post discusses several advantages of Docker and points out some drawbacks.

Docker使我们能够轻易地为应用搭建可重复使用的环境，我们让环境搭建变得自动化并且消除了手动执行任务带来的错误，这样我们就降低了任务部署的风险并提高了可靠性。但是还是会有一些领域使用docker起来比较麻烦，这篇文章将会讨论一下Docker的有点并指出Docker的一些缺点。

## Advantages 优势

### Advantages for the Development Team 对于开发团队的好处

- Full control over the execution environment of the application. This includes the necessary infrastructure like the JRE, the application server, VM arguments and other environment variables the application needs to run. The infrastructure can be changed easily and independently by the development team, because they don’t have to wait for the operations team to change the environment.

- 可以对应用运行环境的完全掌控。这包括必要的基础程序如JRE、应用服务、VM参数以及程序运行时需要的其他环境变量。基础程序可以轻易地更改并且独立于开发团队，因为开发人员不需要运营团队来帮他们变更环境。

![docker image][3]

- Reduced Risk. The tests run against the same image that will finally run on the production server. This increases the reliability of the tests. Moreover, releases become less scary, because we just run the same image on the production server as we did for the tests or the other stages of the build pipeline (e.g. pre-production-system, acceptance tests, capacity tests etc.).

- 降低了风险。测试时的镜像环境就是最终产品运行的服务端环境，这就增加了测试的可靠性。此外，版本发布也不再那么让人提心吊胆了，因为我们我们在服务端运行产品的镜像环境跟我们进行测试或者构建产品的其他阶段使用的环境是一样的（例如，预生产系统，验收测试，容量测试等）。


![image registry][4]

Once the Docker image is built, we use the image in all stages of the (continuous) delivery pipeline. This increases the reliability of our delivery.

一旦Docker镜像构建成功，我们就会在交付的各个阶段都是用这个镜像，这就提高了交付的可靠性。

### Advantages for the Operations Team 对运维团队好处

- Less effort for maintaining environments for the application. The creation of an environment is automated. Hence, less manual actions are necessary. This reduces the risks and increases the reliability.

- 减少维护应用运行环境的工作量。环境是自动创建的，因此更少的人为操作是有必要的。这能减小风险并提高可靠性。

- Manually maintaining a consistent environment on multiple servers is error-prone and can be a nightmare. With Docker it’s easy to create several instances of an environment, because we just have to execute the image on the servers. This way it is easy to add further nodes to a cluster and to scale horizontally.

- 在多态服务器上手工搭建相同环境是很容易出错而且让人崩溃的。使用Docker的话就可以很轻松地创建一个环境的多个实例，因为我们最终只需要在服务端运行这个实例镜像。这种方式也可以很轻松将未来的多个节点添加到集群中进行水平扩展。

- Easy updating of an existing application and environment. Using traditional approaches for setting up an environment (like install scripts) run into trouble when there is already an existing environment. Considering the update path in the script can be a very complex task (e.g. checking the existence of files and clean up unused files). With Docker we don’t have to take an existing environment into account (except for the database). We just stop the running container and start the new updated one. This simplifies the setup because we always start with an empty environment.

- 轻松更新现有的应用和环境。当已经存在一个环境时使用老旧的方式设置环境（比如安装脚本）会陷入麻烦。考虑到脚本中的更新路径可能是一项非常复杂的工作（比如检查文件是否存在以及清理无用文件）。使用Docker的话我们不必考虑现有环境（数据库除外），我们只需要停止当前容器，然后开启一个新的即可，这就简化了设置因为我们总是从一个干净的环境开始配置的。

### Infrastructure as Code 代码设施即代码

When using Docker (and other tools like [Chef][5] or [Puppet][6]) to set up our environment, we benefit from the “Infrastructure-as-Code” approach. The infrastructure is not created manually any longer but is a result of an automated process. We describe our infrastructure explicitly. In case of Docker, the Dockerfile is our documentation describing our environment. This leads to the following benefits:

在使用Docker（和其他工具如 [Chef][5] 或 [Puppet][6]）来配置环境时，我们可以从"Infrastructure-as-Code"这个宗旨中受益。基础设施不再是手动创建的了，而是由自动化程序创建，我们明确地描述基础设施，在Docker看来，Dockerfile就是描述环境的文档，这会带来以下好处：

- We can put our Dockerfile under version control and track changes made to the environment. This way, we always know what application release belongs to which version of the environment.

- 我们可以把Dockerfile放到版本管理中，来追踪环境的变化。这种方式下，我们可以知道应用具体版本归输入具体什么环境。

- We get a reliable documentation of our infrastructure. We always know what our applications need to run and what is currently installed in the environment.

- 我们获得了可靠的基础设施文档。通常我们会知道应用程序运行需要些什么，通过文档可以知道现在环境中已经安装了什么。

- Moreover, the documentation is always up to date, because we need to change it in order to change the environment.

- 此外，文档也是最新的，因为我们需要通过更改Dockerfile来改变环境。

- Therefore, setting up a new environment is easy, because we know what we need to install.

- 因此，配置一个环境是相当容易的，因为我们明确知道自己要安装什么。

## Comparing to other Automation Tools 与其他自动化工具的比较

With tools like Chef or Puppet you can also achieve a lot of the advantages of Docker (like reproducible environments, infrastructure documentation and easy setup of a new node). However, when we use virtualization approaches (like Docker) we gain the following advantages:

使用像Chef或者Puppet的工具，你同样也可以得到Docker所带来的多数优势（比如可重复的环境，基础设施文档以及轻松设置一个节点）。但是当我们使用虚拟化方法时（例如Docker）会有如下好处：

- When we want to implement [Continuous Delivery][7], we need to set up multiple instances of our environment for each stage of the delivery pipeline (commit stage, acceptance test, capacity tests, pre-production and production). Using physical machines for this purpose is not practicable.

- 当我们想要持续交付时，我们需要为交付的各个阶段创建大量的环境实例（提交阶段，验收阶段，容量测试，预生产和生产环境），使用物理机器来达到这个目的时不划算的。

- The installation of the application and the necessary environment becomes easy, because we always start with an empty VM (in fact Docker doesn’t start a real VM, see below). The old VM can be removed completely and the new one is set up.

- 安装应用和必要的基础设施变得相当简单，因为我们总是从一个空的VM开始配置的（实际上Docker并非启动了一个实际的VM，往下看）。老的VM可以完全删除掉，新的VM配置以下就能使用。

### Lightweight Virtualization 轻量级的虚拟化

For this reasons, Chef or Puppet are often used in conjunction with virtualization: a virtual machine is started and Chef or Puppet is used to set up the environment within the VM. When using Docker instead of a real VM we have the following benefits:

处于这个原因，Chef和Puppet经常跟虚拟化结合在一起使用：启动一个虚拟机然后Chef和Puppet在这个虚拟机内配置环境。使用Docker时没有使用一个真实的虚拟机带来的好处如下：

- Lightweight virtualization. Starting a Docker container is much faster than starting a VM, because no guest operating system has to be booted. This reduces the overhead. The several containers share the kernel of the host operating system, but have their own file system, users, network and processes. From the perspective of the host operating system we just start another process, when we run a Docker container. This significantly speeds up the startup of a container while still providing a good isolation of the containers.

- 轻量虚拟化。启动一个Docker容器远比启动一个虚拟机快得多，因为不需要启动一个操作系统。这减少了开销。几个容器共享主机操作系统的内核，但是有自己的文件系统、用户、网络和进程。从主机操作系统角度来看，当我们运行Docker容器时，我们就是在运行另一个进程。这显著加速了容器的启动，同时仍然为各个容器提供了良好的隔离。

## Disadvantages and Challenges 缺点和挑战

After praising Docker (too much?), let’s consider the drawbacks and challenges when using Docker:

赞扬了Docker之后（太多了吗），我们看一下Docker的不足和挑战：

- Increased complexity due to an additional layer. This affects not only the deployment but also the development and build.

- 由于附加层增加了复杂性。这不仅影响部署而且影响开发和构建。

- In addition, managing a huge amount of containers is challenging – especially when it comes to clustering containers. Tools like [Google Kubernetes][8] and [Apache Mesos][9] can help here.

- 此外，管理大量的容器非常具有挑战性 - 特别是在集群容器方面。像[Google Kubernetes][8]和[Apache Mesos][9]可以此时帮助我们进行管理。

- The containers share the same kernel and are therefore less isolated than real VMs. A bug in the kernel affects every container.

- 容器之间共享相同的内核，因此跟真正的VM相比独立性差点。内核的要给Bug会影响到所有的容器。

- Docker bases on Linux Containers (LXC), which is a Linux technology. Therefore, we can’t run Docker on other systems and our container is always a Linux system. But [Boot2Docker][10] enables the usage of Docker on Windows and Mac OS X by using VirtualBox. The Docker client runs on the host OS and communicates with the Docker daemon inside the VirtualBox. Unfortunately, this is less comfortable and makes daily use clumsy and more complicated than running Docker natively.

- Dcoker基于Linux Containers（LXC），这是一种linux技术。因此我们无法在其他操作系统上运行docker而且我们的容器都是Linux系统。但是[Boot2Docker][10]可以通过VirtualBox在Windows和Mac OS X上使用Docker。Docker客户端在主机操作系统上运行，并于VirtualBox内的Docker守护进程通信。不幸的是，这使用起来不太舒服，使得日常开发变得笨拙，比本地运行Docker更加复杂。

- Introducing Docker can be a demanding and time-consuming task. We have to evaluate if it is worth the effort and the increased complexity. In a past project of mine, the management rejected Docker with the argument, that 3 nodes (production, pre-production and experimental) are not enough to justify the effort… However, when you have a cluster of nodes (when scaling horizontally) or use Continuous Delivery you need an approach to reproduce environments and Docker is brilliant in this. Moreover, [Continuous Delivery reduces the time-to-market][11] (new features can be brought to production faster because the deployment and environment setup is automated), which can be used to convince the management.

- 引入Docker可能是一项艰巨而好事的任务。我们必须评估是否值得付出努力，增加复杂性。在过去的一个项目中，管理层拒绝了Docker的论点，即3个节点（生产、预生产和实验）不足以证明这一努力的合理性...但是当你拥有一组节点（水平扩展时）或使用持续交付模式时，你需要一种方法来重现环境，而Docker在这方面非常出色。此外，[持续交付可缩短产品上市时间][11]（新功能可以更快地投入生产，因为部署和环境设置是自动化的），可用于说服管理层。

- I also made the experience that there are also reservations about Docker in strictly regulated domains (like the banking sector):

- 我还获得了在严格监管的域名（如银行业）中对Docker也有所保留的经验：

  - To run a container you need root rights. This can be a problem for some companies, in which the colleagues that are supposed to run and update the application on the production server must not have root rights.

  - 要运行容器你需要root权限，对于某些公司而言，这可能是一个问题，在这些公司中，在生产服务器上运行和更新应用程序的同事不能拥有root权限。
  
  - It is unclear if the usage of Docker agrees with certain security standards which have to be fulfilled (like [PCI][12]).

  - 目前还不清楚Docker的使用是否符合必须满足的某些安全标准（如PCI）

  - In some companies, there is the questionable rule, that only software from official/trusted sources can be installed on the machines. For instance, Docker is not included in Red Hat Enterprise Linux 6 and therefore needs to be installed from docker.com, which is an “untrusted source”.

  - 在一些公司中，存在一个值得怀疑的规则，即只有来自官方/可信来源的软件才能安装在机器上。例如，Docker不包含在Red Hat Enterprise Linux 6中，因此需要从docker.com安装，这是一个“不受信任的来源”。

## Related Posts 相关文章

[Building a Docker image with Maven][13]

[使用Maven构建Docker镜像][13]


[1]: http://www.ruanyifeng.com/blog/2019/01/weekly-issue-41.html
[2]: https://blog.philipphauer.de/discussing-docker-pros-and-cons/
[3]: https://blog.philipphauer.de/blog/2015/1025-discussing-docker-pros-and-cons/Docker-Image-everything-our-applications-needs.png
[4]: https://blog.philipphauer.de/blog/2015/1025-discussing-docker-pros-and-cons/Docker-use-same-image-stages-delivery-pipeline.png
[5]: https://www.chef.io/chef/
[6]: https://puppetlabs.com/
[7]: https://blog.philipphauer.de/tutorial-continuous-delivery-with-docker-jenkins/
[8]: http://kubernetes.io/
[9]: http://mesos.apache.org/
[10]: http://boot2docker.io/
[11]: https://blog.philipphauer.de/tutorial-continuous-delivery-with-docker-jenkins/#Continuous_Delivery
[12]: https://www.pcisecuritystandards.org/
[13]: https://blog.philipphauer.de/building-dropwizard-microservice-docker-maven/