---
title: 容器基础docker
date: 2020-07-11 22:09:10
tags:
- linux
- 学习
- docker
categories : "Linux"
---

> - 容器基础
> - docker的原理

<!-- more-->

# Namespace机制
linux中有一个系统调用为clone，用来创建一个子进程。
'''cpp
int pid = clone(main_function, stack_size, CLONE_NEWPID | SIGCHLD, NULL);
'''
- CLONE_NEWPID可以为子进程创建一个全新的进程空间，在这个进程空间，它的pid为1。可以看作是一个障眼法，让每个子进程都以为自己是独一无二的。

# Cgroup机制（Linux Control Group）
每个容器相当于一个子进程，如果不对每个进程加以限制的话，会让资源被无限制占用。因此Cgroup机制就是用以限制一个进程组的资源上线，如CPU，内存，硬盘，网络带宽。

- 暴露接口：操作系统的文件系统，位于/sys/fs/cgroup
- 一个正在运行的 Docker 容器，其实就是一个启用了多个Linux Namespace 的应用进程，而这个进程能够使用的资源量，则受 Cgroups 配置的限制。

# rootfs
## Mount Namespace
虽然有个隔离（Namespace），限制（Cgroup）机制，但是宿主机的文件系统却没办法隐藏。依然可以使用clone函数：
```cpp
int container_pid = clone(container_main, container_stack+STACK_SIZE, CLONE_NEWNS | SIGCHLD
```
- CLONE_NEWNS标志，启用Mount Namespace。使用了这个参数后，在容器启动前，进行mount文件路径，即可达到文件系统隔离

## rootfs原理
rootfs可以看成（根文件系统），包含/bin,/etc,/proc等基础目录。因此对于一个docker项目来说，可以近似看成以下三个步骤：
1. 启用Namespace
2. 设置Cgroup
3. 切换进程根目录（mount namespace）

## 联合文件系统（union file system）
docker引入了layer层这个创新，用户制作镜像时每一个操作都会生成一个新的layer，而联合系统就是将多个目录挂载到同一个目录上面，并且将相同的文件进行合并。如果在新目录当中进行修改，也会影响到被挂载方的目录。

被挂载后，rootfs主要由三个层组成：可读写层(rw)，Init层(ro+wh)，只读层(ro+wh)
- ro readonly
- wh whiteout
### whiteout
如果想对只读层修改，修改的记录会被记录到.wh.文件名中，并未实际删除，只是被遮挡住

# 未完待续