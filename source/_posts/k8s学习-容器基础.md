---
title: k8s学习-容器基础
date: 2021-03-27 10:33:19
tags: k8s
categories: k8s
---

* 简介
* Linux Namespace
* Linux Cgroups
* Linux Chroot
* rootfs
* 其他
  
------

<!-- more -->
## 简介

容器技术的核心功能，就是通过约束和修改进程的动态表现，从而为其创造出一个“边界”。容器本质上是一种特殊的进程。

## Linux Namespace技术
通过Linux Namespace机制实现了对进程空间的隔离，使得进程只能看到重新计算过的进程编号，比如 PID=1。Namespace其实只是Linux创建进程的一个可选参数，在参数中指定`CLONE_NEWPID`参数，比如：
```c
    int pid = clone(main_function, stack_size, CLONE_NEWPID | SIGCHILD, null)
```
Namespace有以下几种：PID、Mount、UTS、IPC、Network、User，用来对各种不同的进程上下文进行“障眼法”操作。

容器只是运行在宿主机上的一种特殊的进程，所以多个容器使用的还是同一个宿主机的操作系统内核。所以如果你要在 Windows 宿主机上运行 Linux 容器，或者在低版本的 Linux 宿主机上运行高版本的 Linux 容器，都是行不通的。Windows 和 Mac 运行 docker 实际上都是通过虚拟机的形式。

## Linux Cgourps
在Linux内核中，有很多资源和对象是不能被Namespace化的，最典型的例子：时间。Linux Cgroups就是Linux内核中用来为进程设置资源限制的一个重要功能。

Linux Cgroups最主要的作用，就是限制一个进程组能够使用的资源上线，包括CPU、内存、磁盘、网络带宽等。

Linux Cgoups的原理是在一个子目录系统上加上一组资源限制文件的组合。比如：通过往`cfs_period_us`中写入100000（100ms，100000us），`cfs_quota`中写入20000（20ms，20000us），把进程PID写入`task`文件，就可以限制该进程只能使用到 20% 的 CPU 带宽（意味着在每 100 ms 的时间里，被该控制组限制的进程只能使用 20 ms 的 CPU 时间）
{% asset_img 容器基础-1.png %}
{% asset_img 容器基础-2.png %}

但是Cgroups对资源的限制还有很多不完善的地方，比如对/proc文件系统问题。如果在容器中运行top命令，显示的信息会是宿主机的CPU、内存等信息。

可以通过lxcfs来解决这个问题，lxcfs通过文件挂载的方式，把cgroups中的信息通过volume的方式挂载到容器内部的proc系统，然后让容器以为读取proc中的信息就是读取宿主机中真是的proc。
{% asset_img 容器基础-3.jpg %}

## Linux Chroot
Mount Namespace修改的是容器进程对文件系统“挂载点”的认知，如果没有mount操作，会看到宿主机的文件。这就是 Mount Namespace 跟其他 Namespace 的使用略有不同的地方：它对容器进程视图的改变，一定是伴随着挂载操作（mount）才能生效。

容器其实需要在进程启动之前重新挂载它的整个根目录“/”，在Linux中可以通过chroot命令来完成，改变进程的根目录到你指定的位置。

实际上，Mount Namespace正是基于chroot不断改良而来的，它也是Linux操作系统中第一个Namespace。

而这个挂载在容器根目录上、用来为容器提供隔离后执行环境的文件系统，就是所谓的“容器镜像”。它还有一个更专业的名字，叫作“rootfs”。

所以，对Docker项目来说，最核心的原理就是为待创建的用户进程：
1. 启用Linux Namespace配置；
2. 设置指定的Cgroups参数；
3. 切换进程的根目录（Change Root）。

## rootfs
rootfs 只是一个操作系统所包含的文件、配置和目录，并不包括操作系统内核。在 Linux 操作系统中，这两部分是分开存放的，操作系统只有在开机启动时才会加载指定版本的内核镜像。

正是由于 rootfs 的存在，容器才有了一个被反复宣传至今的重要特性：一致性。

由于 rootfs 里打包的不只是应用，而是整个操作系统的文件和目录，也就意味着，应用以及它运行所需要的所有依赖，都被封装在了一起。对一个应用来说，操作系统本身才是它运行所需要的最完整的“依赖库”。

如果每次更新都重新制作rootfs，将会变得很麻烦。所以Docker做了一个小小的创新，Docker 在镜像的设计中，引入了层（layer）的概念。也就是说，用户制作镜像的每一步操作，都会生成一个层，也就是一个增量 rootfs。这个想法其实是用到了一种叫作联合文件系统UnionFS（Union File System）的能力。

AuFS 联合文件系统的rootfs由三部分组成：只读层（ro+wh）、Init层（ro+wh）、可读写层(rw)
* 只读层：它们的挂载方式都是只读的（ro+wh，即 readonly+whiteout）
* 可读写层： rootfs 最上面的一层，它的挂载方式为：rw，即 read write。一旦在容器里做了写操作，修改产生的内容就会以增量的方式出现在这个层中。为了实现删除操作，会在可读写层创建一个 whiteout 文件，把只读层里的文件“遮挡”起来。这个功能，就是“ro+wh”的挂载方式，即只读 +whiteout 的含义。
* Init层：它是一个以“-init”结尾的层，夹在只读层和读写层之间。Init 层是 Docker 项目单独生成的一个内部层，专门用来存放 /etc/hosts、/etc/resolv.conf 等信息。


## 其他
1. 容器是一个单进程，但是容器里还可以运行其他命令、进程，这些进程实际上是子进程。容器的单进程意思不是只能运行一个进程，而是只有一个进程是可控的。
2. docker里面跑的进程不是docker的子进程，是entrypoint进程的子进程。docker基本上是旁路控制的作用。
3. cgroups目录留下来的文件清理：先umount卸载，然后删除
4. k8s中的/etc/hosts文件配置，可以通过Pod定义中的`hostAliases`字段向Pod的 /etc/hosts 添加条目。该文件已经被 Kubelet 管理起来，任何对该文件手工修改的内容，都将在 Kubelet 重启容器或者 Pod 重新调度时被覆盖
5. docker如何修改镜像内的文件：采用copy-on-write（CoW）机制，只在需要写时才去复制。CoW技术可以让所有的容器共享image的文件系统，只有当要对文件进行写操作时，才从image里把要写的文件复制到自己的文件系统进行修改。容器所做的写操作都是在复本上进行，并不会修改image的源文件，每个容器修改的都是自己的复本，相互隔离，相互不影响。使用CoW可以有效的提高磁盘的利用率。
6. docker支持的UnionFS实现：aufs, device mapper, btrfs, overlayfs, vfs, zfs。aufs是ubuntu 常用的，device mapper 是 centos，btrfs 是 SUSE，overlayfs ubuntu 和 centos 都会使用，现在最新的 docker 版本中默认两个系统都是使用的 overlayfs，vfs 和 zfs 常用在 solaris 系统。
7. docker的scratch：万能的base镜像，本身就是个空镜像。
8. k8s有GC功能，会清理未使用的镜像和容器：https://kubernetes.io/zh/docs/concepts/cluster-administration/kubelet-garbage-collection/
9. 如果应用依赖某个特定内核版本才有的特性，这个容器是无法跨平台的。有LinuxKit这种类似的方案：LinuxKit就是kernel+busybox实现的一个微缩linux系统，其中直接安装了containerd和runc服务。其他服务全部都使用容器启动。