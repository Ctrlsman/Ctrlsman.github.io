---
title: DevOps利器之Docker入门篇
---

## 简介

### What?

Docker是基于Go语言实现的开源容器项目,诞生于2013年年初,由dotCloud公司发起,此公司后改名为Docker Inc.Docker项目已加入Linux基金会,并遵循Apache2.0协议,代码托管在Github:[Docker源码地址](https://github.com/docker)

各大操作系统现都支持Docker,并且最新的Linux发行版RedHat、CentOS、Ubuntu中均已默认带有Docker软件包.

Docker的构想是要实现“Build,Ship and Run Any App, Anywhere”,即通过对应用的封 装( Packaging)、分发( Distribution)、部署( Deployment)、运行( Runtime)生命周期进行管 理，达到应用组件级别的“ 一次封装 ，到处运行” 。 这里的应用组件， 既可以 是一个 Web 应 用、一个编译环境，也可以是一套数据库平台服务，甚至是一个操作系统或集群。

基于 Linux 平 台上的多项开源技术， Docker 提供了高效、敏捷和轻量级的容器方案，并 支持部署到本地环境和多种主流云平台 。 可以说 ， Docker首次为应用 的开发 、运行和部署提 供了“一站式”的实用解决方案。

早期Docker代码实现是基于LXC(Linux Containers,即Linux容器),自0.9版本开始,Docker开发了libcontainer项目作为更广泛的容器驱动实现.

简单地讲，读者可以将 Docker 容器理解为一种轻量级的沙盒( sandbox)。 每个容器内 运行着一个应用，不同的容器相互隔离，容器之间也可以通过网络互相通信。 容器的创建和 停止十分快速，几乎跟创建和终止原生应用 -致;另外，容器自身对系统资源的额外需求也十分有限，远远低于传统虚拟机 。 很多时候，甚至直接把容器当作应用本身也没有任何 问题。

### Why?

+ 新型的创建分布式应用程序的方式，快速分发和部署
+ 通过容器来打包应用、解藕应用和运行平台 
+ 更快速的交付和部署,启动快速,启动和停止可以在妙计实现,节约开发、测试、部署的大量时间
+ 更高效的资源利用,一台主机上可以运行上千个Docker容器,运行Docker不需要额外的虚拟化管理程序(virtual machine manager,以及Hypervisor)的支持,Docker是内核级的虚拟化,可以实现更高级的性能
+ 更轻松的迁移和扩展,Docker容器几乎可以在任意的平台上运行，包括物理机、虚
拟机、公有云、私有云 、 个人电脑 、 服务器等
+ 更简单的更新管理,使用Dockerfile,只需要小小的配置修改就可以实现以前繁琐的更新,提高工作效率，并标准 化流程
+ Docker通过类似 Git设计理念的操作来方便用户获取、分发和更新应用镜像，存储复 用，增量更新
+ Docker与传统虚拟机的比较如下图

![Docker与传统虚拟机](https://tva1.sinaimg.cn/large/006tNbRwly1gap7rpo04jj312w0d4gv5.jpg)


### How?


**Docker 运行基本命令和模式:**
![Docker 运行基本命令和模式](https://tva1.sinaimg.cn/large/006tNbRwly1gap7oxl945j30go0df0ty.jpg)

**Docker Engine:**
![Docker Engine](https://tva1.sinaimg.cn/large/006tNbRwly1gap7lqqakyj30tq0kagpb.jpg)

**Docker architecture**:
![Docker architecture](https://tva1.sinaimg.cn/large/006tNbRwly1gap7mjlktrj318s0n80wi.jpg)


### Docker与虚拟化

虚拟化 (virtualization)技术是一个通用的概念，在不同领域有不同的理解。 在计算领 域，一般指的是计算虚拟化 (computingvirtualization)，或通常说的服务器虚拟化。 维基百科 上的定义如下:

> “在计 算机技 术中，虚拟化 是 一种资 源管理技术，是将 计 算机 的各种实 体资 源，如服务器 、 网络、 内存及存储等，予以抽 象、转换后呈现出来，打破实体 结 构间的不可切割的障碍，使用户可以用比原本的纽态更好的方式来应用这些资源 。”

可见，虚拟化的核心是对资源的抽象，目标往往是为了在同一个主机上同时运行多个系
统或应用，从而提高系统资源的利用率，并且带来降低成本、方便管理和容错容灾等好处 。

**Docker 和常见的虚拟机方式的不同之处:**
![](https://tva1.sinaimg.cn/large/006tNbRwly1gap9h9zkv0j30yc0dstnr.jpg)


## 安装

用户可以访问 Docker 官网的 Get Docker [Docker 官网](https://www.docker.com/get-docker)页面，查看获取 Docker 的方式，以及 Docker 支持的平台类型目前 Docker 支持 Docker 引 擎、 Docker Hub、 Docker Cloud 等多种服务 。


+ Docker 引擎:包括支持在桌面系统或云平台安装 Docker，以及为企业提供简单安全
弹性的容器集群编排和管理;
+ DockerHub:官方提供的云托管服务，可以提供公有或私有的镜像仓库;
+ DockerCloud :官方提供的容器云服务，可以完成容器的部署与管理，可以完整地支
持容器化项目，还有CI、 CD功能。

**Linux安装**

```shell
$sudo apt-get update
$sudo apt-get install docker-ce docker-ce-cli containerd.io
```

**Mac安装**

```shell
$brew cask install docker
```

或者:

手动下载安装 :[Docker Mac传送门](https://docs.docker.com/docker-for-mac/install/)

**运行:**

安装完成后启动Docker服务,然后尝试运行如下命名:

```shell
$docker run -d -p 80:80 --name webserver_test nginx

# 然后在浏览器访问:http://127.0.0.1 看看效果,一个nginx服务器已经搭建完成了,是不是很香
```

## Docker有三大核心概念:

+ 镜像 (Image)
+ 容器( Container)
+ 仓库( Repository)

### 镜像 (Image)

镜像是Docker三大核心概念中最重要的,是创建 Docker容器的基础。本质上是一个文件,通过版本管理和增量的文件系统， Docker 提供了一套十分简单的机制来创建和更新现有的镜像，用户甚至可以从网上下载一个已经做好的应用镜像，并直接使用.

Docker 运行容器前需要本地存在对应的镜像， 如果镜像不存在， Docker 会尝试先从默 认镜像仓库下载(默认使用 Docker Hub 公共注册服务器中的仓库)， 用户也可以通过配置， 使用自定义的镜像仓库。

#### 获取镜像

```shell
$docker pull NAME [:TAG] 
$docker pull ubuntu:18.04
命令相当于
docker pull registry.hub.docker.com/ubuntu:18.04
如果从非官方的仓库下载，则需要在仓库名称前指定完整的仓库地址

NAME:镜像仓库名称
TAG:镜像标签,如果不指定默认为latest,即会下载仓库中最新版本的镜像
```

**注意:**
一般来说,镜像的latest 标签意味着该镜像的内容会跟踪最新版本的变更而变化，内容是不稳定的。因此，从稳定性上考虑，不要在生产环境中忽略镜像的标签信息或使 用默认的latest 标记的镜像。

#### 查看镜像信息

1. 查看本地主机上的镜像列表:

```shell
$docker images
或者
$docker image ls

# 参数
> --digests=true|false: 列出镜像的数字摘要值，默认为否;
> -f, --filter=[] : 过滤列出的镜像， 如dangling=true 只显示没有被使用的
镜像;也可指定带有特定标注的镜像等;
> --format="TEMPLATE" : 控制输出格式，如: .ID代表ID信息，.Repository
代表仓库信息等;
> -q, --quiet=true|false: 仅输出ID信息， 默认为否

更多子命令选项还可以通过man docker-images来查看
```

显示如图:

![](https://tva1.sinaimg.cn/large/006tNbRwly1gaqhosav08j31e003ajs6.jpg)

+ REPOSITORY:来自于哪个仓库
+ TAG: 镜像的标签信息
+ IMAGE ID:镜像的ID,这里同样的ID可以是不同TAG,完整的ID是由64个十六进制字符,这里默认展示前12位            
+ CREATED: 镜像的创建时间             
+ SIZE: 镜像大小,优秀的镜像往往都比较小,镜像大小信息只是表示了该镜像的逻辑体积大小， 实际上由于相同的镜像层本地只会存储一份， 物理上占用的存储空间会小于各镜像逻辑体积之和

2. 添加镜像标签

为了方便在后续工作中使用特定镜像，还可以使用`docker tag`命令来为本地镜像任 意添加新的标签。 例如，添加一个新的myubuntu: latest镜像标签:

```shell
$docker tag ubuntu:latest myubuntu:latest
```

3. 查看镜像详细信息

使用 `docker inspect` 命令可以获取该镜像的详细信息，包括制作者、适应架构、各层的数字摘要等:

```shell
$docker inspect ubuntu:18.04

# 上面代码返回的是一个JSON格式的消息，如果我们只要其中一项内容时，可以使用 -f 来指定，例如，获取镜像的ContainerConfig:

$docker inspect -f {{".ContainerConfig"}} ubuntu:18.04
```

4. 使用`history`命令查看镜像历史
既然镜像文件由多个层组成，那么怎么知道各个层的内容具体是什么呢?这时候可以使用 `history`子命令，该命令将列出各层的创建信息。

![](https://tva1.sinaimg.cn/large/006tNbRwly1gaqibffwtdj31gq07mq5w.jpg)


#### 搜索镜像
```shell
$ docker search nginx

参数:
-f, --filter filter: 过滤输出内容
--format string: 格式化输出内容
--limit int: 显示个数,默认25
--no-trunc: 不截断输出结果

$ docker search --filter=is-official=true nginx
```

#### 删除和清理镜像

使用 `docker rmi` 或 `docker image rm` 命令可以删除镜像， 命令格式为 `docker
rmi IMAGE [IMAGE ... ]`, 其中 IMAGE 可以为标签或 ID。

```shell
$docker rmi myubuntu:latest

参数:
-f, -force: 强制删除镜像， 即使有容器依赖它;
-no-prune: 不要清理未带标签的父镜像。

注意:
当同 一 个镜像拥有多个标签的时候，docker rmi 命令只是删除了该镜像多个标签中的指定 标签而巳， 并不影响镜像文件

# 删除所有镜像
$docker rmi $(docker ps -a)  # 慎用啊!!!!
```

使用Docker一段时间后， 系统中可能会遗留一些临时的镜像文件， 以及一些没有被使 用的镜像， 可以通过`docker image prune`命令来进行清理。


#### 创建镜像
创建镜像的方法主要有三种: 
+ 基于已有镜像的容器创建
+ 基千本地模板导入
+ 基于Dockerfile创建(比较常用)

##### 利用Dockerfile创建镜像

Dockerfile是一个文本格式的配置文件，用户可以使用Dockerfile来快速创建自定义的镜像。由一行行命令语句组成，并且支持以#开头的注释行。一般而言，Dockerfile主体内容分为四部分:
+ 基础镜像信息
+ 维护者信息
+ 镜像操作指令
+ 容器启动时执行指令

`未完待续......`