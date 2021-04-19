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

### (1) 获取镜像

```s
$ docker pull --help
```

```s
$ docker pull [OPTIONS] [ADDRESS/]NAME[:TAG]
```

- ADDRESS: Docker镜像仓库地址，格式一般是 <域名/IP>[:端口号]。默认地址是 Docker Hub 的地址docker.io。

- NAME：仓库名，格式一般是<用户名>/<软件名>。对于 Docker Hub，如果不给出用户名，则默认为 library，也就是官方镜像。

```s
$ docker pull docker.io/library/ubuntu:18.04
$ docker pull ubuntu:18.04
```

从下载过程中可以看到我们之前提及的分层存储的概念，镜像是由多层存储所构成。下载也是一层层的去下载，并非单一文件。下载过程中给出了每一层的 ID 的前 12 位。并且下载结束后，给出该镜像完整的 `sha256` 的摘要，以确保下载一致性。

### (2) 列出镜像

```s
$ docker image ls
```

- repository 仓库名
- tag 标签
- image id 镜像id
- ctrated 创建时间
- size 所占空间

镜像的唯一标识是器id和摘要，一个镜像可以有多个标签

### (3) 删除本地镜像

```s
$ docker image rm xx1 xx2 
```

可是使用镜像短id, 镜像长id, 镜像名, 镜像摘要来删除镜像

- 短id, 一般取长id的前3个字符以上，只要可以区别于别的镜像即可
- 镜像名, `<仓库名>:<标签>`
- 镜像摘要

```s
$ docker image ls --digests
$ docker image rm node@sha256:xxxxxx
```

删除行为分为两类, 一类是untagged, 另一类是deleted

- untagged: 使用上述命令删除镜像, 实际上是在要求删除某个标签的镜像, 首先需要将镜像的标签取消
- deleted: 如果只有一个标签指向镜像, 且没有其他镜像和容器依赖, 则会执行删除镜像的行为

配合 docker image ls命令批量删除镜像

- 删除所有仓库名为redis的镜像

```s
$ docker image rm $(docker image ls -q redis)
```

- 删除所有在mongo:3.2之前的镜像

```s
$ docker image rm $(docker image ls -1 -f before=mongo:3.2)
```

### (4) Dockfile制作镜像

Step1: Dockerfile是一个文本文件，包含一条条的指令，每一条指令构建一层

```s
FROM nginx
RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
```

- FROM 指定基础镜像

在一个Dockerfile中，FROM是必备的指令，并且必须是第一条指令。Docker Hub上有可以直接拿来使用的服务类的镜像，也有更为基础的操作系统镜像。FROM scratch 表示基于一个空白的镜像。

- RUN 执行命令

shell格式：RUN <命令>

exec格式：RUN ["可执行文件", "参数1", "参数2"]

Step2: 在Dockerfile文件所在目录创建镜像

```s
$ docker build -t nginx:v3 .
```

- .指定上下文路径


### (5) 跨平台构建镜像

### (6) 镜像存储位置



## 2.2 docker 容器

### (1) 新建并启动容器

输出 hello world, 之后终止容器

```s
$ docker run ubuntu:18.04 /bin/eche 'Hello world'
```

启动bash终端, 允许用户进行交互

```s
$ docker run -t -i ubuntu:18.04 /bin/bash
```

- -t 让 docker 分配一个伪终端并绑定到容器的标准输入上
- -i 让容器的标准输入保持打开

### (2) 启动已终止容器

```s
$ docker container start xxx
```

### (3) 停止容器

```s
$ docker stop xxx
```

### (4) 重启容器

```s
$ docker restart xxx
```
### (5) 后台运行程序

希望在docker后台运行程序而不是直接把执行命令的结果输出在当前宿主机下。

```s
$ docker run -d ununtu:18.04 /bin/sh -c "while true; do echo hello world; sleep 1; done"
```

查看容器信息

```s
$ docker container ls
```

获取容器输出信息

```s
$ docker container logs
```

### (6) 进入容器

只使用 -i 参数，由于没有分配伪终端，姐买你没有Linux命令提示符

```s
$ docker exec -i 69d1 bash
```

当 -i -t 参数一起使用时，可以看到Linux命令提示符

```s
$ docker exec -it 691d bash
```

### (7) 暂停容器

希望让容器暂停工作一段时间

```s
$ docker pause xxx
```

### (8) 删除容器
 
 删除处于终止状态的容器
 
 ```s
 $ docker container rm xxx
 ```
 
 删除一个运行中的容器
 
 ```s
 $ docker -f container rm xxx
 ```
 
删除所有处于终止状态的容器

```s
$ docker container prune
```

删除所有已经退出的容器

```s
$ docker rm -v $(docker ps -aq -f status=exited)
```

### (9) 导出容器

导出容器快照到本地文件

```s
$ docker export xxx > ubuntu.tar
```

### (10) 导入容器快照

从容器快照文件中再导入为镜像

```s
cat ubuntu.tar | docker import  - test/ubuntu:v1.0
```



# 3. 数据管理

## 3.1 数据卷

一个可供一个或多个容器使用的特殊目录

- 数据卷可以在容器之间共享和重用
- 对数据卷的修改会立马生效
- 对数据卷的更新不会影响镜像
- 数据卷默认会一直存在，即使容器被删除

### (1) 创建一个数据卷

```s
$ docker volumn create datawhale
```

查看所有的数据卷

```s
$ docker volumn ls
```

查看指定数据卷的信息

```s
$ docker volumn inspect datawhale
```

### (2) 启动一个挂载数据卷的容器

在 docker run 的时候，使用 --mount 来将数据卷挂载到容器里，可以挂载多个数据卷

创建名为web的容器，并加载一个数据卷到容器
 
```s
$ docker run -d -P \
    --name web \
    --mount source=datawhale,target=/usr/share/nginx/html \
    nginx:alpine
```

- source: 数据卷
- traget: 容器内文件系统挂载点

可以不需要提前创建数据卷，直接在运行容器的时候mount，这是如果不存在指定的数据卷，docker会自动创建。

当数据卷为空时，镜像中被指定为挂载点的目录中的文件会复制到数据卷中。

### (3) 查看数据见的具体信息

```s
$ docker inspect web
```

### (4) 删除数据卷

```s
$ docker volumn rm datawhale
```

删除容器的同时移除数据卷

```s
$ docker rm -v xxx
```

清理无主的数据卷

```s
$ docker volumn prune
```

## 3.2 挂载主机目录

### (1) 挂载一个主机目录作为数据卷

加载主机的 /src/webapp 目录到容器的 /usr/share/ngnix/html 目录

```s
$ docker run -d -P \
    --name web \
    --mount type=bind,source=/src/webapp,target/usr/share/nginx/html \
    ngnix:alpine
```

- docker挂载主机目录的默认权限是读写
- 如果挂载的主机目录不存在，创建容器时，docker不会自动创建，此时会报错

加载主机目录，权限为只读

```s
$ docker run -d -P \
    --name web \
    --mount type=bind,source=/src/webapp,target/usr/share/nginx/html,readonly \
    ngnix:alpine
```

### (2) 挂载一个本地主机文件作为数据卷

```s
$ docker run --rm -it \
    --mount type=bind,source=$HOME/.bash_history,target=/root/.bash_history \
    ubuntu:18.04 \
    bash
```


