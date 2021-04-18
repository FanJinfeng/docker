# 1. Docker简介

## 1. Docker架构

- 进程
计算机中的程序关于某数据集合上的一次运行活动。
- 宿主(host)
相对于虚拟机而言，正在使用的计算机就是宿主机。

![Docker 架构](https://docs.microsoft.com/en-us/virtualization/windowscontainers/deploy-containers/media/docker-on-linux.png)

(1) 基于Linux内核的cgroups、Namespaces、Union FS等技术，对进程进行封装隔离，属操作系统层面的虚拟化技术。由于隔离的进程独立于宿主和其它的隔离的进程，因此也称其为容器。

(2) runc用于创建和运行容器，containerd用于管理容器生命周期，提供了在一个节点上执行容器和管理镜像的最小功能集。

(3) Docker在容器的基础上，进行了进一步的封装，从文件系统、网络互联到进程隔离等等，极大的简化了容器的创建和维护。

## 2. 与传统虚拟化方式的对比

- Server：运行软件的计算机
- Host OS：宿主机的操作系统
- Hypervisor
虚拟机监视器(virtual machine monitor, VMM), 用来建立与执行虚拟机器的软件、固件或硬件
- 虚拟机
通过软件模拟的具有完整硬件系统功能的、运行在一个完全隔离环境中的完整计算机系统。
- 虚拟化的仓库(repository)

![传统虚拟化](https://vuepress.mirror.docker-practice.com/assets/img/virtualization.bfc621ce.png)

(1) 虚拟机：在Host OS上面是hypervisor, 然后依次建立虚拟机、虚拟化的仓库，然后安装程序。

(2) Docker：在Host OS上面是Docker Engine, 然后直接在Docker Engine安装应用。

## 3. Docker三大基本概念

- 操作系统分为内核空间和用户空间
- 挂载(mounting)
是指由操作系统使一个存储设备(诸如硬盘、CD-ROM或共享资源)上的计算机文件和目录可供用户通过计算机的文件系统访问的一个过程。
- root文件系统
根文件系统首先是内核启动时所mount的第一个文件系统，内核代码映像文件保存在根文件系统中，而系统引导启动程序会在根文件系统挂载之后从中把一些基本的初始化脚本和服务等加载到内存中去运行。
- Linux内核启动后，会挂载root文件系统为其提供用户空间支持
- docker daemon(守护神)
服务器组件，以 Linux 后台服务的方式运行，是 Docker 最核心的后台进程，我们也把它称为守护进程。它负责响应来自 Docker Client 的请求，然后将这些请求翻译成系统调用完成容器管理操作。


![](https://tva1.sinaimg.cn/large/008eGmZEly1gpauta9hchj31hi0qegvt.jpg)

### 3.1 镜像(image)

一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。镜像 不包含 任何动态数据，其内容在构建之后也不会被改变。

官方镜像 ubuntu:18.04 就包含了完整的一套 Ubuntu 18.04 最小系统的 root 文件系统。

因为镜像包含操作系统完整的 root 文件系统，其体积往往是庞大的，因此在 Docker 设计时，就充分利用 Union FS (opens new window)的技术，将其设计为分层存储的架构。

镜像并非由一个文件组成，而是由一组文件系统组成。构建镜像时，会一层层构建，前一层是后一层的基础，并且每一层构建完就不会再发生改变。

### 3.2 容器(container)

镜像（Image）和容器（Container）的关系，就像是面向对象程序设计中的 类 和 实例 一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。

容器的实质是进程，运行于属于自己的独立的命名空间。因此容器可以拥有自己的 root 文件系统、自己的网络配置、自己的进程空间，甚至自己的用户 ID 空间。使用起来好像是在一个独立于宿主的系统下操作一样。

每一个容器运行时，是以镜像为基础层，在其上创建一个当前容器的存储层(为容器运行时读写而准备)。容器消亡时，容器存储层也随之消亡。

容器不应该向其存储层内写入任何数据，所有的文件写入操作，都应该使用数据卷(Volumn)或者绑定宿主目录。在这些位置的读写会跳过容器存储层，直接对宿主机发生读写，其性能和稳定性更高。数据卷的生命周期独立于容器。

### 3.3 仓库(repository)

docker集中存放镜像的地方。

### 3.4 Docker Registry

镜像构建完成后，可以很容易的在当前宿主机上运行，但是，如果需要在其它服务器上使用这个镜像，就需要一个集中的存储、分发镜像的服务，Docker Registry 就是这样的服务。

一个 Docker Registry 中可以包含多个仓库（Repository）；每个仓库可以包含多个标签（Tag）；每个标签对应一个镜像。

例如, ubuntu 是仓库的名字，其内包含 ubuntu:16.04 和 ubuntu:18.04 两个镜像。

- Docker Registry 公开服务

官方的 Docker Hub

国内的一些云服务商提供了针对 Docker Hub 的镜像服务（Registry Mirror），这些镜像服务被称为加速器。常见的有阿里云加速器、DaoCloud 加速器等。

- 私有 Docker Registry

Docker 官方提供了 Docker Registry 镜像，可以直接使用做为私有 Registry 服务。



# 2. Docker 镜像与容器

## 2.1 Docker 镜像
