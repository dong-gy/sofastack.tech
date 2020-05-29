---
title: "（含直播报名）Kata Containers 创始人：安全容器导论"
author: "王旭"
authorlink: "https://github.com/gnawux"
description: "隔离，让云原生基础设施更完美。"
categories: "Kata Container"
tags: ["Kata Container"]
date: 2020-05-08T15:00:00+08:00
cover: "https://cdn.nlark.com/yuque/0/2020/png/226702/1588995462747-f057c015-34f2-4b42-ba59-6b129bc27f88.png"
---

从2015年5月初开始创业开发 HyperContainer (runV) 到现在，也快五年了，在这个时候还来写一篇什么是安全容器，显得略有尴尬。不过，也正是经过这五年，越来越多到人开始感到，我需要它却说不清它，这个时候来给大家重新解释 “> **安全容器**” 也正是时候。

### 缘起：安全容器的命名

Phil Karlton 有一句名言—— 

_计算机科学界只有两个真正的难题——缓存失效和命名。_ 

就容器圈而言，我相信命名绝对配得上这句话，这毫无疑问是一件让老开发者沉默，让新人落泪的事情。

仅就系统软件而言，我们当今比较通行地称为 Linux 容器（LinuxContainer）的这个概念，曾经用过的名字大概还有——jail, zone, virtualserver, sandbox... 而同样，在早期的虚拟化技术栈里，也曾经把一个虚拟机环境叫做容器。毕竟这个词本身就指代着那些用来包容、封装和隔离的器物，实在是太过常见。以至于，以严谨著称的 Wikipedia 里，这类技术的词条叫做“系统级虚拟化”，从而回避了“什么是容器”这个问题。

当2013年 Docker 问世之后，容器这个概念伴随着“不可变基础设施”、“云原生”这一系列概念，在随后的几年间，以摧枯拉朽之势颠覆了基于“软件包+配置”的细粒度组合的应用部署困境，用简单地声明式策略+不可变容器就清爽地描述了软件栈。应用怎么部署似乎离题了，我们这里想要强调的是——

**云原生语境下的容器，实质是“应用容器”——以标准格式封装的，运行于标准操作系统环境（常常是 Linux ABI）上的应用打包——或运行这一应用打包的程序/技术。**

这个定义是我在这里写下的，但是它并不是我的个人意志，这是基于 OCI (Open ContainerInitiative) 规范这一共识的，这个规范规定了容器之中的应用将被放置在什么环境中，如何运行，比如启动容器根文件系统上的哪个可执行文件，用什么用户，需要什么样的 CPU、内存资源，有什么外置存储，有什么样的共享需求等等。

有了这一共识做基础，我们来说安全容器，这又是一段命名血泪史。当年，我的联合创始人赵鹏是用“虚拟化容器”命名的我们的技术的，为了搏人眼球，用了“Secure as VM, Fast asContainer”的大字标语，自此，被容器安全性问题戳中心坎的人们立刻用“Secure Container”来称呼这类东西，一发而不可收。而我们的内心中，这项技术提供了一层额外的隔离，隔离可能意味着安全性中的一环，也意味着某些运维效率、某些优化可能或者其他的功能。实际上，给安全容器这样的定义更合理——

**一种运行时技术，为容器应用提供一个完整的操作系统执行环境（常常是 LinuxABI），但将应用的执行与宿主机操作系统隔离开，避免应用直接访问主机资源，从而可以在容器主机之间或容器之间提供额外的保护。**

### 间接层：安全容器的精髓

_安全问题的唯一正解在于允许那些（导致安全问题的）Bug 发生，但通过额外的隔离层来阻挡住它们。_
—— LinuxCon NA 2015, Linus Torvalds 

为了安全，为什么要引入间接层？因为以 Linux 之类的目前主流宿主机操作系统的规模，是无法从理论上验证程序是没有 Bug 的，而一旦合适的 Bug 被合适地利用，安全性风险就变成了安全性问题了，框架和修补并不能确保安全，所以，进行额外的隔离、缩减攻击面，就成了缓解安全问题的法宝——我们不能确保没有漏洞，但我们通过组合来减少漏洞被彻底攻破的风险。

### Kata: 云原生化的虚拟化

2017年12月，我们在 KubeCon上对外发布了 Kata Containers 安全容器项目，这个项目的两个前身——我们发起的的 runV 和Intel 发起的 Clear Container 都发布于2015年5月（是的，早于上面 Linus 的引语）。这组项目的思路很简单 —— 

1.  操作系统本身的容器机制没办法解决安全性问题，需要一个隔离层；
2.  虚拟机是一个现成的隔离层，AWS 这样的云服务已经让全世界相信，对户来说，"secure of VM" 是可以满足需求的；
3.  虚机里面只要有个内核，就可以支持 OCI 规范的语义，在内核上跑个 Linux 应用这并不太难实现；
4.  虚机可能不够快，阻碍了它在容器环境的应用，那么可不可以拥有 "speed of container" 呢？ 

现在，如果最后一个问题可以解决，那么它就是我们要的“安全的容器”了——这就是  Kata Containers。 目前 Kata Containers 通常是在 Kubernetes 环境中使用的，Kubelet 通过CRI 接口让 containerd 或 CRI-O 执行运行时操作，通常镜像操作由这些 CRI Daemon 来进行，而根据请求，把 Runtime 操作写成一个 OCI Spec 交给 OCI Runtime 执行。这里，对于 1.2 以上的 containerd 和 1.5 版本上后的 Kata Containers（对应 ），通常是利用 containerd 来完成的：

- 每个 Pod 会有一个 shim-v2 进程来为 containerd/CRI-O 执行各种运行时操作，这个进程和整个 Pod 的生命周期一致，通过一个 ttRPC 接口为 containerd/CRI-O 提供服务；
- Shim-v2 会为 Pod 启动一个虚拟机作为 PodSandbox提供隔离性，其中运行着一个 Linux 内核，通常这个 Linux 内核是一个裁剪过的内核，不会支持没有必要的设备；
- 这里用的虚拟机可以是 Qemu 或是 Firecracker，如果是 Firecracker，那么根本就没有模拟的设备，而如果是 Qemu，通过配置和补丁，也会让它尽量小一些，支持的其他虚拟机还包括 ACRN 和 cloud-hypervisor，未来后者可能会被越来越多的用到；
- 这个虚拟机在启动的时候可能根本就是一个 initrd 而没有完整操作系统，或者有个极小的操作系统，总之，它完全不是按照一台模拟机的操作系统来配置的，它只是支撑容器应用运行的基础设施，以及相关的应用环境 Metrics 采集或应用跟踪调试需要的资源；
- 容器本身的 rootfs 会在 Sandbox 启动之后，在收到 contaienrd/CRI-O 的创建容器的 OCI 请求之后，以热插拔的方式动态插入到虚机中，容器的 rootfs 准备和虚机本身的启动是可以并行的；
- 依照 CRI 语义和 OCI 规范，Pod 里可以启动多个相关联的容器，它们被放在同一个虚机里，并且可以互相共享 namespace；
- 外来的存储卷可以以块设备或文件系统共享的方式插入到 PodSandbox 中，但对于 Pod 里的容器来说，它们都是加载好的文件系统，目前开始逐渐普及的文件系统方式是专为 Kata 这样的场景设计的 virtio-fs，它不仅比传统的 9pfs 更快、有更完整的 POSIX 文件系统的支持，而且借由所谓 vhost-user 和 DAX 技术，它还可以在不同的 Pod 之间共享相同存储内容的缓存页 (page-cache)，让它们可以和普通的 runC 容器一样，节约宝贵的内存；
- 对于网络，使用 tcfilter，可以直接支持各种 CNI 的插件来提供容器网络，这样的好处是无需做什么调整就自然的工作，但是效率上因为做一次桥接会有损失，在生产环境中，也可以考虑使用 enlightened 网络模式，用一个特制的 CNI 插件来高效接入容器网络。

可以看到，首先 Kata Containers 是个全功能的容器运行时引擎，它用起来完全不像是传统虚机，分明就是个容器引擎，并且，通过“少用不必要的内存”和“共享能共享的内存”来降低内存的开销，更小的内存不仅开销更小，启动也更轻快，对于大多数场景来说，这实现了"secure of VM, speed of container"。在安全性之外，它比起传统的虚机，更具有容器的弹性，更少了机器的那种物理操作手感，我们把这种技术称为“云原生化虚拟化”或者“面向云原生的虚拟化”技术。

### gVisor: 进程级虚拟化

在 Kata Containers 之后半年的哥本哈根KubeCon 上，Google 开源了他们内部开发了五年的 gVisor 安全容器作为回应。

如果说 Kata Containers 是对通过对有隔离技术进行组合和改造来构建容器间的隔离层的话，gVisor 的设计显然是更加简洁的——gVisor 是一个用 Go 语言重写的运行在用户态的操作系统内核，称为 Sentry，它并不依赖于虚拟机，相反，它借助“平台(Platform)”的能力，让宿主机把应用的所有访问都重新转交给 Sentry，在 Sentry 中处理后再将一些必要的操作请宿主机帮忙来完成。

可以说 gVisor 在做的是一个纯粹的面向应用的隔离层，从 Linux ABI 到 Linux ABI 的“过滤器”。全新写作的优势在于不需要迁就太多已有技术栈的桎梏，可以写得更轻，启动肯定也会更快，事实上，资源的伸缩也更方便，或者说更容器一些。好多操作系统圈的朋友都毫不掩饰地说，他们更喜欢 gVisor 的架构，如果能解决一些目前不容易解决掉的问题的话。

gVisor 作为隔离层，它的安全性依据在于：

- 首先是攻击面变小，宿主操作系统将只为沙箱里的应用执行大约20%的 Linux 系统调用。通过研究，gVisor 的作者们发现，大多数攻击都是通过一些不常用系统调用中的缺陷进行的，不常用的系统调用的实现路径一般也比较少被审阅，相对于那些热路径来说，安全性要更差一些，gVisor 的设计让应用对那些不常用系统调用的访问根本不会落到宿主机操作系统内核上，从而避免了大部分的攻击；
- 其次是他们发现了最最常被攻击到的系统调用是 open()，于是他们直接将真有必要的 open() 调用交给了一个专门的称为 Gopher 的进程来执行，从而可以更容易被限制、审计和管控；
- 最后，他们是用高级语言 Go 写的内核，优势当然是更加内存安全，当然，他们也坦诚，这个语言其实不太“系统级”，他们为此不得不做了很多手脚，也为 Go Runtime 贡献了很多修改。

当然，gVisor 的架构是很漂亮，但重新实现一个内核这件事情，除了 Google 这样的巨头恐怕也没有几家可以做（类似的基本做到的也就是微软的初代 WSL了），而且这个超前的设计还是有些现实问题：

- 首先，就是它不是 Linux，所以，在兼容性方面和 Kata 这样的解决方案尚有差距；
- 其次，对于当前的 Linux 系统调用方式和 CPU 指令系统，每个系统调用的拦截都会有相当的性能开销，尽管全栈优化可以有一定缓解，但对系统调用比较多的场景来说，性能损失无法忽略。

所以，短时间内 gVisor 方案并不能成为一个终极解决方案，当然，它不仅可以适应一些特定的场景，而且带来的启示性可能对未来的操作系统乃至 CPU 指令集的演进发生作用，从而推动我们可以拥有一个更完美的安全容器解决方案。

### 安全容器：不止于安全

安全容器的隔离层让应用的问题——不论是恶意攻击，还是意外错误——都不至于影响宿主机，也不会在不同的 Pod 之间相互影响。而且实际上，额外隔离层带来的影响并不仅是安全，对于调度、服务质量和应用信息的保护都有好处。

传统的操作系统容器技术是内核进程管理的一个延伸，容器进程本身是一组关联的进程，对于宿主机的调度器来说是完全可见的，一个 Pod 里的所有容器或进程，同时也都被宿主机调度和管理。这就意味着，在有大量容器的环境下，宿主机内核的负担很重，在很多实际环境中已经可以观察到这个负担带来的开销了。而采纳安全容器之后，从宿主机上是看不到这些完整的信息的，隔离层同时也降低了宿主机的调度开销，减少了维护负担，避免了容器之间、容器和宿主机之间的服务质量干扰。从另一个方向看，安全容器作为一道屏障，可以让宿主机的运维管理操作不能直接访问到应用的数据，这样，把用户的应用数据直接保护在沙箱里就可以降低对用户的授权要求，保障用户的数据私密性。

当我们的目光向投向未来，可以看到，安全容器不仅仅是在做安全隔离，因为安全容器隔离层的内核，相对于宿主机的内核是独立的，专门对应用服务，从这个角度说，主机/应用的功能之间做合理的功能分配和优化，展现出让人期待的潜力，将来的安全容器，可能不仅是隔离性开销的降低，甚至是提升应用的效能—— 

**隔离，让云原生基础设施更完美。**

Kata Containers 开源地址：[https://katacontainers.io/](https://katacontainers.io/)

### 相关直播推荐：《不得不说的云原生隔离性》

![SOFAChannel#16.jpg](https://cdn.nlark.com/yuque/0/2020/jpeg/226702/1588991853415-63027aa3-7c13-40ba-943d-78c2e27c263a.jpeg)

线上直播 SOFAChannel#16，主题：不得不说的云原生隔离型，将邀请 Kata Containers 维护者彭涛（花名：巴德）带我们走近文中提到的云原生基础设施 -- Kata Containers，详细分享它是如何在云原生框架下解决容器隔离性问题的。

将从以下几个方面，与大家交流分享：

- 从 Kubernetes Pod 说起，经典容器场景下的 Pod 分享；
- 共享内核存在的问题以及解决办法；
- 上帝说，要有光；我们说，要有 Kata；
- The speed of containers, the security of VMs；
- Kata Containers 特性大放送；
- What? 你刚说过增加一个 VM 间接层的问题？

**直播主题**：SOFAChannel#16：不得不说的云原生隔离型

**直播时间**：2020/5/21（周四）19:00-20:00

**报名方式**：点击“[这里](https://tech.antfin.com/community/live/1197)”，即可报名