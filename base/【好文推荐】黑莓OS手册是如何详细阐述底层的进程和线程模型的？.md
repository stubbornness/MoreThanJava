![](https://imgkr2.cn-bj.ufileos.com/db1944cc-0cc4-4ee0-befd-c3cd2059a58d.png?UCloudPublicKey=TOKEN_8d8b72be-579a-4e83-bfd0-5f6ce1546f13&Signature=oo617Eq22IS84ftmiXEcXP2k4Bs%253D&Expires=1596272169)

- **「MoreThanJava」** 宣扬的是 **「学习，不止 CODE」**，本系列 Java 基础教程是自己在结合各方面的知识之后，对 Java 基础的一个总回顾，旨在 **「帮助新朋友快速高质量的学习」**。
- 当然 **不论新老朋友** 我相信您都可以 **从中获益**。如果觉得 **「不错」** 的朋友，欢迎 **「关注 + 留言 + 分享」**，文末有完整的获取链接，您的支持是我前进的最大的动力！

# 写在前面

该文章的大部分内容都是翻译自是黑莓 `10` 实时操作系统 `QNX Neutrino` 的[开发手册](https://developer.blackberry.com/native/documentation/dev/rtos/arch/about_system_architecture.html)，该手册不仅详细地阐述了 BlackBerry 10 OS 的原理以及 OS 的体系结构，还描述了其 QNX Neutrino 微内核的详细信息 *(包括进程线程、多和处理、网络架构、文件系统等...非常完整..)*。

我阅读了其中「描述进程和线程」的精华部分，觉得写的非常不错，特意翻译 *(主要靠有道和 Google 翻译)* 跟大家分享一下 *(部分内容有改动)*。

手册中详细描述了许多关于 Linux 函数调用涉及底层方面的细节，在本文中我大都没有贴出来，**感兴趣的朋友强烈建议去拜读一下原文！**

![](https://imgkr2.cn-bj.ufileos.com/336441d3-e4fc-450e-9271-1b4ef3a4f6ed.png?UCloudPublicKey=TOKEN_8d8b72be-579a-4e83-bfd0-5f6ce1546f13&Signature=9VR8cSORG5jWR8cqJTrBoc4TXIo%253D&Expires=1596271687)

> - 原文地址：https://developer.blackberry.com/native/documentation/dev/rtos/arch/about_system_architecture.html

# Part 1. 进程和线程基础

在我们开始讨论线程、进程、时间片和所有其他的精彩的概念之前，让我们先来建立一个类比。

我要做的首先是 **说明线程和进程是如何工作的**。我能想到的最好的方法 *(不涉及实时系统的设计)* 是在某种情况下想象我们的线程和进程。

## 进程就像一栋房子

![](https://imgkr2.cn-bj.ufileos.com/e283ed13-3541-4a3d-8ec9-6b845d6434f4.png?UCloudPublicKey=TOKEN_8d8b72be-579a-4e83-bfd0-5f6ce1546f13&Signature=kecaKX3GOyWpSaR9vTCJZBokCR4%253D&Expires=1596244026)

房子实际上是 **具有某些属性的容器** *(例如卧室数量、占地面积、区域划分等)*。

如果您以这个角度来看，房子实际上并不会主动做任何事情————它是一个 **被动的对象**。这实际上就是进程干的事情了 *(充当容器)*。

## 线程就像是居住者

![](https://imgkr2.cn-bj.ufileos.com/3d262b67-f931-41fa-8328-bcce20034a5b.png?UCloudPublicKey=TOKEN_8d8b72be-579a-4e83-bfd0-5f6ce1546f13&Signature=F7QcbvtpKj0DWD1dvrh8Jx7sqhw%253D&Expires=1596244392)

居住在房子里面的人是 **活动的对象**，他们可以使用各种房间，看电视、做饭、洗澡等等等...

### 单线程

如果您独居过，那么您就会知道————**您可以在家里的任何时间做您任何想做的事**，因为家里面没有其他人，您只要遵从内心的规则就好。

### 多线程

如果在您的房子中再另外添加一个人，情况将发生巨变。

假设您结婚了，那么现在您的家里住着您和您的配偶。因此您不能在 **任何时间** 都能够使用厕所，因为您需要首先确保您的配合不在其中！

如果您有两个 **负责任的成年人** 居住在房屋中，那么通常您可以在 **「安全性」** 这一块儿稍微放松一些 *(因为您知道另一个成年人会尊重您的空间，不会试图故意在厨房放火之类的..)*。

但是，如果把几个 **熊孩子** 混在一起，事情就会变得更加有趣了...

## 说回进程和线程

就像是房屋占用土地一样，进程也要占用内存。

也正如房屋拥有者可以随意进入他们想去的任何房间一样，进程中的线程也 **都拥有** 对该内存区域访问的权限。

如果某个线程被分配了某些东西 *(例如哥哥进程出去买了游戏机🎮回家)*，那么其他所有线程都可以立即访问它 *(因为它存在于公共地址空间中————都在房子里)*。

同样，如果给进程分配额外多的一块空间，则新的区域也能够用于所有线程。

**这里的窍门在于识别内存是否应该对进程中的所有线程可用。**

- 如果是，那么您将需要让所有线程同步它们对其的访问。

- 如果不是，那么我们将假定它只能用于某个特定的线程 *(在这种情况下，因为只有特定线程能够访问它，我们可以在假设线程不会自己把自己玩儿坏的前提下，不需要同步操作)*。

**从日常的生活中我们知道，事情并不是那么简单。**

现在我们已经了解了基本特征 *(所有内容都是共享的)*，下面我们来深入探究一下事情变得有趣的地方以及原因。

下图显示了我们表示线程和进程的方式。进程是一个圆圈，代表“容器”概念（地址空间），三个长方形是线程。在本文中，您将看到类似这样的图。

![](https://imgkr2.cn-bj.ufileos.com/2c253102-4bd3-4164-892f-315c5fba4467.png?UCloudPublicKey=TOKEN_8d8b72be-579a-4e83-bfd0-5f6ce1546f13&Signature=WnCM4PjMf048bO3dnw%252FgnGCI8jo%253D&Expires=1596244922)

## 互斥

在生活中，如果您想洗个澡，并且已经有人在洗手间了，您将不得不等待。线程又是如何处理的呢？

这是通过一种叫做 **互斥** *(mutual exclusion)* 的操作完成的。和你想的差不多——当涉及到 **特定资源** 时，许多线程是互斥的。

当你想要独占浴室洗澡时，你通常会走进浴室，从里面锁上门。任何想要上厕所的人都会被锁挡住。当你洗完之后，你会打开门，允许其他人进入。

这就是线程的作用。一个线程使用一个叫做 **互斥锁** *(mutex)* 的对象 *(互斥锁 MUTual EXclusion 的首字母缩写)*。这个对象就像门上的锁 —— 一旦一个线程锁定了互斥锁，其他线程就不能获得该互斥锁，直到拥有它的线程释放它。就像门锁一样，等待获得互斥锁的线程将被阻挡。

互斥锁和门锁的另一个有趣的相似之处是，互斥锁实际上是一种 **“建议” 锁** *(“advisory” lock)*。如果一个线程不遵守使用互斥锁的约定，那么保护就没有用了。在我们的房子比喻中，这就像有人不顾门和锁的惯例，从一堵墙闯进盥洗室。

## 优先级

如果卫生间现在上锁了，有很多人在等着使用怎么办？显然，所有人都坐在外面，等着洗手间里的人出来。

**真正的问题是：当门打开时会发生什么?谁下一个去?**

你会想，让等待时间最长的人下一个走或许是 “公平的”。又或者，让年龄最大的人排在第二位也可能是 “公平的”。有很多方法可以确定什么是“公平”。

我们通过两个因素来解决这个问题：**优先级** 和 **等待的时长**。

假设两个人同时出现在锁着的浴室门口。其中一个有一个有急事 *(例如：他们开会已经迟到了...)*，而另一个没有。让那个有急事的人去做下一个，不是很有意义吗？

当然！问题的关键是你如何决定谁更 “重要”。

这可以通过分配优先级来实现 *(我们可以使用数字，例如 `1` 是最低的可用优先级，`255` 是这个版本的最高优先级)*。房子中那些有急事的人将被给予更高的优先权，而那些没有急事的人将被给予较低的优先权。

线程也是一样。线程从父线程继承自己的调度算法，但是可以调用 `Linux` 系统函数 [`pthread_setschedparam()`](https://man7.org/linux/man-pages/man3/pthread_setschedparam.3.html) 来更改自己的调度策略和优先级 *(如果它有权限这么做的话)*。

如果有很多线程在等待，并且互斥锁被解锁，我们将把互斥锁给优先级最高的等待线程。但是，假设两个人有相同的优先级。现在该做什么呢？

好吧，在这种情况下，让等待时间最长的人下一个去才是 “公平的”。这不仅是“公平的”，也是内核实际所做的。在有一堆线程等待的情况下，我们主要根据优先级，其次是等待的长度。

互斥锁当然不是我们将遇到的唯一 **同步对象**。让我们看看其他一些例子。

## 信号量

让我们从浴室转到厨房，因为那里是一个社会认可的可以同时容纳一个人以上的地方。在厨房里，你可能不想让每个人都同时待在那里。事实上，你可能想要限制厨房里的人数 *(厨师太多等等...)*。

假设你不希望同时有 **超过两个人** 在里面，这可以使用互斥锁来实现吗？这对于我们的类比来说是一个非常有趣的问题。让我们稍微讨论一下。

### 计数为 1 的信号量

在上面类比的卫生间的例子中，可以有两种情况，并且两种状态相互关联:

- 门没锁，房间里没人；
- 门锁了，房间里有人；

这里并没有其他可能的组合 —— 房间里没有人的时候门不能锁上 *(我们怎么能打开它?)*，房间里有人的时候门也不能打开 *(他们怎么能保证自己的隐私呢?)*。这是一个计数为 `1` 的信号量示例：房间中最多只能有一个人，或者有一个线程使用该信号量。

**这里的关键是我们描述锁的方式。**

在你典型的浴室锁里，你只能从里面上锁和解锁 *(没有可以从外部访问的锁)*。实际上，这意味着互斥锁的所有权是一个原子操作 —— 在你获得互斥锁的过程中，其他线程不可能获得它，结果就是一个线程进入 "厨房" 上锁，导致另一个线程将无法进入。在我们用房子来比喻的故事里，这一点就不那么明显了，因为人类比 `1` 和 `0` 聪明得多。

很显然，我们厨房需要的是一种不同类型的锁。

### 计数大于 1 的信号量

假设我们在厨房安装了传统的基于钥匙的锁。这把锁的工作原理是，如果你有一把钥匙，你就可以开门进去。任何使用这把锁的人都同意，当他们进入内部时，他们将立即从内部锁门，这样，任何在外部的人都将始终需要一把钥匙。

好了，现在控制我们想要多少人在厨房就变成一件简单的事情了 —— 把两把钥匙挂在门外！厨房总是锁着。当有人想进厨房时，他们要看是否有钥匙挂在门外。如果是的话，他们就把它带在身边，打开厨房的门，走进去，用钥匙锁门。

因为进入厨房的人必须在他们进入厨房时带着钥匙，所以我们通过限制门外挂钩上可用的钥匙数量来直接控制允许进入厨房的人数。

对于线程，这是通过 **信号量** 来完成的。“普通” 信号量就像互斥锁一样工作 —— 你要么拥有互斥锁，在这种情况下你可以访问资源，要么不拥有，在这种情况下你不能访问资源。我们刚才在厨房中描述的信号量是一个计数信号量 —— 它跟踪计数 *(根据线程可用的 "钥匙" 的数量)*。

## 作为互斥锁的信号量

我们刚才问了这样一个问题:“可以用互斥锁来实现吗?” 对于使用互斥锁实现计数，答案是否定的。反过来怎么样？我们可以使用信号量作为互斥锁吗？

当然可以！事实上，在某些操作系统中，这正是它们所做的 —— 它们没有互斥锁，只有信号量！**那么，为什么要为互斥锁费心呢？**

要回答这个问题，先看看你的洗手间。你房子的建造者是如何实现 “互斥锁” 的？我敢肯定你家的厕所外面并没有挂在墙上的钥匙！

互斥锁是一种 “特殊用途” 的信号量。如果您希望在代码的特定部分中运行一个线程，那么互斥是目前为止最有效的实现。

*(事实上文章的后续介绍了其他同步方案——称为 `condvars`、`barrier` 和 `sleepons`，这些就是 Java 中同步方案的操作系统层面的原型，感兴趣的可以自行去阅读一下.. 差别不大，只不过使用的是操作系统层面的描述...)*

> 为了避免混淆，请认识到 **互斥锁还有其他属性**，比如 **优先级继承**，这是它与 **信号量** 的区别。

# Part 2. 内核的作用

房子的类比很好地解释了同步的概念，但是它在另一个主要领域是解释不通的。在我们家里，有许多 "线程" 是同时运行的。然而，在一个真实的实时系统中，通常只有一个 CPU，所以一次只能运行一个 “东西”。

## 单核 CPU

让我们看看在现实情况中会发生什么，特别是在系统中 **只有一个 CPU** 的情况下。在这种情况下，由于只有一个 CPU，因此在 **任何给定的时间点只能运行一个线程**。内核决定 *(使用一些规则，我们将很快看到)* 到底该运行哪个线程。

## 多核CPU (SMP)

如果您购买的系统具有多个相同的 CPU，它们都共享内存和设备，那么您将拥有一个 SMP box *(SMP 代表对称的多处理器，“对称的” 部分表示系统中的所有 CPU 都是相同的)*。在这种情况下，可以并发 *(同时)* 运行的线程数量受到 CPU 数量的限制。*(实际上，单处理器机箱也是如此!)* 由于每个处理器一次只能执行一个线程，因此如果有多个处理器，多个线程就可以同时执行。

现在让我们先忽略 CPU 的数量 —— 一个有用的抽象是把系统设计成多线程同时运行的样子，即使事实并非如此。*(稍后，在 “使用 SMP 时要注意的事情” 一节中，我们将看到 SMP 的一些非直观影响...)*

## 内核作为仲裁程序

那么，在任何给定的时刻，谁来决定哪个线程将运行？这是内核的工作。

内核确定在特定时刻应该使用 CPU 的线程，并将上下文切换到该线程。让我们研究一下内核对 CPU 做了什么。

CPU 有许多寄存器 *(确切的数目取决于处理器家族，例如，`x86` 对 `MIPS`，以及特定的家族成员，例如，`80486` 对 奔腾)*。当线程运行时，信息存储在这些寄存器中 *(例如，当前程序位置)*。

当内核决定另一个线程应该运行时，它需要:

1. 保存当前运行线程的寄存器和其他上下文信息；
2. 将新线程的寄存器和上下文加载到 CPU 中；

**但是内核如何决定应该运行另一个线程呢？** 它会查看某个特定线程此时是否能够使用该 CPU。例如，当我们谈到互斥锁时，我们引入了一种 **阻塞状态** *(当一个线程拥有互斥锁，另一个线程也想获得它时，就会发生这种情况;第二个线程将被阻塞)*。

因此，从内核的角度来看，我们有一个线程可以消耗 CPU，而另一个线程由于被阻塞而不能等待互斥锁。在这种情况下，内核让可以运行的线程消耗 CPU，并将另一个线程放入一个内部列表 *(以便内核可以跟踪它对互斥锁的请求)*。

显然，这不是一个很有趣的情况。假设有许多线程可以使用该 CPU。还记得我们基于优先级和等待长度来委托对互斥的访问吗？内核使用类似的方案来确定下一个将运行哪个线程。有两个因素：**优先级** 和 **调度算法**，基于此顺序评估。

### 优先级

考虑两个能够使用 CPU 的线程。如果这些线程具有不同的优先级，那么答案真的很简单 —— 内核将 CPU 分配给优先级最高的线程。正如我们在讨论获得互斥锁时提到的，优先级是从 `1` *(可用性最低的)* 开始的。

**注意，优先级为零是为空闲线程保留的 —— 您不能使用它。** *(如果您想知道您的系统的最小值和最大值，可以使用 Linux 的系统函数 `sched_get_priority_min()` 和 `sched_get_priority_max()` —— 它们的原型被定义在 `<sched.h>` 中。我们假设 `1` 是最低可用性，`255` 是最高可用性。)*

如果另一个具有更高优先级的线程突然能够使用 CPU，内核将立即上下文切换到更高优先级的线程。我们称之为 **抢占** —— 高优先级线程抢占了低优先级线程。当高优先级线程完成时，内核上下文切换回之前运行的低优先级线程，我们称之为 **恢复** —— 内核继续运行前一个线程。

现在，假设有两个线程能够使用 CPU，并且具有完全相同的优先级。

### 调度算法

让我们假设其中一个线程当前正在使用该 CPU。在本例中，我们将研究内核用于决定何时进行上下文切换的规则。*(当然，这整个讨论实际上只适用于具有相同优先级的线程)*

内核有两种主要调度算法 *(策略)*：轮询调度 *(或简称“RR”)* 和 FIFO *(先入先出)*。

#### FIFO(First In First Out，先进先出)

在 FIFO 调度算法中，允许线程一直使用 CPU *(只要它想要使用)*。这意味着，如果该线程正在进行非常长的数学计算，并且没有其他具有更高优先级的线程准备好，那么该线程可能会永远运行下去。那么具有相同优先级的线程呢？它们也被锁在外面了。*(显然，此时低优先级的线程也被锁定。)*

如果正在运行的线程放弃或自愿放弃 CPU，那么内核将查找 **具有相同优先级、能够使用该 CPU** 的其他线程。如果没有这样的线程，那么内核将寻找能够使用 CPU 的低优先级线程。

注意，术语 *“自愿放弃CPU”* 可能意味着两种情况。

如果线程进入睡眠状态，或者在信号量上阻塞，那么是的，一个低优先级的线程可以运行 *(如上所述)*。

但是还有一个 **“特殊” 调用，[`sched_yield()`](http://www.qnx.com/developers/docs/6.4.1/neutrino/lib_ref/s/sched_yield.html)** *(基于内核调用 `SchedYield()`)*，它仅将 CPU 让给另一个具有相同优先级的线程 —— 如果高优先级的线程已经准备好运行，那么低优先级的线程将永远不会有机会运行。如果一个线程确实调用了 `sched_yield()`，并且没有其他具有相同优先级的线程准备运行，那么原始线程将继续运行。实际上，`sched_yield()` 用于在 CPU 上对另一个具有相同优先级的线程进行访问。

在下面的图中，我们看到三个线程在两个不同的进程中运行:

![](https://imgkr2.cn-bj.ufileos.com/d1dcc7d2-65c5-411d-b392-e9e5f68b901f.png?UCloudPublicKey=TOKEN_8d8b72be-579a-4e83-bfd0-5f6ce1546f13&Signature=9IsHiil41mGZXGCu0BiXBOkCURM%3D&Expires=1596260602)

如果我们假设线程“A”和“B”已经准备好了，线程“C”被阻塞了(可能在等待互斥)，线程“D”(没有显示)正在执行，那么内核维护的就绪队列的一部分看起来是这样的：

![](https://imgkr2.cn-bj.ufileos.com/990839a3-c5e9-44c1-a4a8-5c57d885b15a.png?UCloudPublicKey=TOKEN_8d8b72be-579a-4e83-bfd0-5f6ce1546f13&Signature=2E%2F2JZeM6FlDNeZBFH0wcsuvv4k%3D&Expires=1596261100)

这显示了内核的 **内部就绪队列**，内核使用它来决定下一步调度谁。注意，线程“C”没有在就绪队列中，因为它被阻塞了；线程“D”也没有在就绪队列中，因为它正在运行。

#### 轮循(Round Robin)

**RR 调度算法** 与 FIFO 相同，只是如果有另一个具有相同优先级的线程，则该线程不会永远运行。它只对系统定义的时间片运行，您可以使用函数 [`sched_rr_get_interval()`](http://www.qnx.com/developers/docs/6.4.1/neutrino/lib_ref/c/clockperiod.html) 来确定时间片的值。时间片通常为 `4` 毫秒，但实际上是 `ticksize` 的 `4` 倍，您可以查询或使用 [`ClockPeriod()`](http://www.qnx.com/developers/docs/6.4.1/neutrino/lib_ref/s/sched_rr_get_interval.html) 设置 `ticksize`。

实际发生的情况是，内核启动一个 RR 线程，并记录时间。如果 RR 线程运行了一段时间，分配给它的时间就会超时 *(时间片已经过期)*。内核查看是否有另一个具有相同优先级的线程已经准备好了。如果有，内核运行它。如果没有，那么内核将继续运行 RR 线程 *(内核授予线程另一个时间片)*。

让我们总结一下 **调度规则** *(对于单个CPU)*，按重要性排序:

- 一次只能运行一个线程。
- 最高优先级的就绪线程将运行。
- 线程将一直运行，直到阻塞或退出。
- RR 线程将运行它的时间片，然后内核将重新安排它 *(如果需要)*。

下面的流程图显示了内核做出的决策：

![](https://imgkr2.cn-bj.ufileos.com/b0053516-c0fb-4655-91d1-a725cba0681d.png?UCloudPublicKey=TOKEN_8d8b72be-579a-4e83-bfd0-5f6ce1546f13&Signature=p6DZfBCt5m%2B4iBy491D3YTrm2%2Fc%3D&Expires=1596263209)


对于多 CPU 系统，规则是相同的，除了多个 CPU 可以并发地运行多个线程。线程运行的顺序 *(在多个 CPU 上运行哪些线程)* 的确定方法与在单个 CPU 上完全相同 —— 最高优先级的就绪线程将在一个 CPU 上运行。对于优先级较低或等待时间较长的线程，内核在安排它们以避免缓存使用效率低下方面具有一定的灵活性。

## 内核状态

我们一直在松散地讨论 “运行”、“就绪” 和 “阻塞” —— 现在让我们更加深入地来讨论这些线程状态。

### 运行态（RUNNING）

运行状态仅仅意味着线程正在积极地消耗 CPU。在一个 SMP 系统上，会有多个线程在运行;在单处理器系统中，只有一个线程在运行。

### 就绪态（READY）

就绪状态意味着这个线程现在就可以运行 —— 但它不会立刻运行，因为另一个线程 *(具有相同或更高优先级)* 正在运行。如果两个线程能够使用 CPU，一个线程优先级为 `10`，一个线程优先级为 `7`，那么优先级为 `10` 的线程将正在运行，优先级为 `7` 的线程将准备就绪。

### 阻塞状态（BLOCKED）

我们把阻塞状态称为什么？问题是，阻塞状态并不只有一个。在内核的作用下，实际上有超过 `12` 种阻塞状态。

为什么那么多？因为内核会跟踪线程被阻塞的原因。

我们已经看到了两种阻塞状态 —— 当一个线程被阻塞等待互斥锁时，这个线程处于互斥锁状态；当线程被阻塞等待信号量时，它处于 `SEM` 状态。这些状态只是表明线程阻塞在哪个队列 *(和哪个资源)* 上。

如果在一个互斥锁上阻塞了许多线程 *(处于互斥锁阻塞状态)*，内核不会注意到它们，直到拥有该互斥锁的线程释放它。此时，阻塞的一个线程已经准备就绪，内核将做出重新调度决策 *(如果需要)*。

为什么说 “如果需要” 呢？刚刚释放互斥锁的线程可能还有其他事情要做，并且比等待的线程有更高的优先级。在本例中，我们使用第二个规则，即 “最高优先级的就绪线程将运行”，这意味着调度顺序没有改变 —— 高优先级线程继续运行。

### 内核状态，完整的列表

下面是内核阻塞状态的完整列表，并简要说明了每个状态。顺便说一下，这个列表可以在 `\<sys/neutrino.h>` —— 你会注意到所有的状态都以 `STATE_` 作为前缀 *(例如，这个表中的 “READY” 在头文件中以 `STATE_READY` 的形式列出)*：

| 状态标识    | 当前线程动作                                                 |
| :---------- | :----------------------------------------------------------- |
| CONDVAR     | 等待一个条件变量被通知。                                     |
| DEAD        | 线程死亡。内核正在等待释放线程的资源。                       |
| INTR        | 等待中断。                                                   |
| JOIN        | 等待另一个线程的完成。                                       |
| MUTEX       | 等待获取互斥锁。                                             |
| NANOSLEEP   | 睡一段时间 *(当前的线程将暂停执行,直到 `rqtp` 参数所指定的时间间隔)*。 |
| NET_REPLY   | 等待通过网络发送的回复。                                     |
| NET_SEND    | 等待一个脉冲或消息通过网络传送。                             |
| READY       | 不在 CPU 上运行，但已准备运行 *(一个或多个更高或同等优先级的线程正在运行)*。 |
| RECEIVE     | 等待客户端发送消息。                                         |
| REPLY       | 等待服务器回复消息。                                         |
| RUNNING     | 正在 CPU 上积极地运行。                                      |
| SEM         | 等待获取信号量。                                             |
| SEND        | 等待服务器接收消息。                                         |
| SIGSUSPEND  | 等待信号。                                                   |
| SIGWAITINFO | 等待信号。                                                   |
| STACK       | 等待分配更多堆栈。                                           |
| STOPPED     | 暂停(SIGSTOP信号)。                                          |
| WAITCTX     | 等待寄存器上下文 *(通常是浮点)* 可用 *(仅在SMP系统上)*。     |
| WAITPAGE    | 等待进程管理者解决页面上的一个错误。                         |
| WAITTHREAD  | 等待创建线程。                                               |

**需要记住的重要一点是，当一个线程被阻塞时，无论它处于何种状态，它都不会消耗CPU。相反，线程消耗 CPU 的唯一状态是运行状态。**


# Part 3. 线程和进程

让我们从真实的实时系统的角度回到对线程和进程的讨论。

我们知道一个进程可以有一个或多个线程。*(一个没有线程的进程将不能做任何事情 —— 也就是说，没有人在家执行任何有用的工作。)* 一个系统可以有一个或多个进程。*(同样的讨论也适用于 —— 一个没有任何进程的系统不会有任何作用。)*

那么这些进程和线程是做什么的呢？最终，它们形成一个系统 —— 执行某个目标的线程和进程的集合。

在最高层次上，系统由许多进程组成。每个进程都负责提供某种性质的服务 —— 无论是文件系统、显示驱动程序、数据采集模块、控制模块，还是其他什么。

在每个进程中，可能有许多线程。线程的数量是不同的。一个只使用一个线程的进程可以完成与另一个使用五个线程的进程相同的功能。有些问题本身是多线程的，实际上解决起来相对简单，而其他进程本身是单线程的，很难实现多线程。

如何用线程进行设计的话题很容易就会被另一本书占用 —— 在这里我们只关注基础知识。

## 为什么需要多个进程？

那么为什么不让一个进程拥有无数线程呢？虽然有些操作系统强迫你这样编码，但将事情分解成多个进程的好处有很多:

- 分离和模块化；
- 可维护性；
- 可靠性；

将问题 “分解” 成几个独立问题的能力是一个强大的概念。它也是系统的核心。一个系统由许多独立的模块组成，每个模块都有一定的职责。这些独立的模块是不同的进程。QSS 的人员使用这个技巧来开发独立的模块，而不需要模块相互依赖。模块之间唯一的 “依赖” 是通过少量定义良好的接口实现的。

由于缺乏相互依赖关系，这自然会增强可维护性。因为每个模块都有自己的特定定义，所以修复一个模块相当容易 —— 尤其是在它不绑定到任何其他模块的情况下。

**然而，可靠性可能是最重要的一点**。进程就像房子一样，有一些定义明确的 “边界”。住在房子里的人很清楚自己什么时候在房子里，什么时候不在房子里。一个线程有一个很好的想法 —— 如果它在进程中访问内存，它可以存活。如果它超出了进程地址空间的范围，它就会被杀死。这意味着运行在不同进程中的两个线程可以有效地相互隔离。

![](https://imgkr2.cn-bj.ufileos.com/bbe2b7f2-7b04-44a7-a5fe-712433286783.png?UCloudPublicKey=TOKEN_8d8b72be-579a-4e83-bfd0-5f6ce1546f13&Signature=LeNN0yU22H6okm89xqzgNB5Afog%3D&Expires=1596263701)


进程地址空间由系统的进程管理器模块维护和执行。当一个进程启动时，进程管理器会分配一些内存给它，并启动一个正在运行的线程。内存被标记为该进程所拥有。

这意味着如果进程中有多个线程，内核需要在它们之间进行上下文切换，这是一个非常高效的操作 —— 我们不需要改变地址空间，只需要改变哪个线程在运行。但是，如果我们必须切换到另一个进程中的另一个线程，那么进程管理器就会介入并导致地址空间的切换。不用担心，虽然在这个额外的步骤中会有一些额外的开销，但是在系统的作用下仍然是非常快的。

## 如何启动一个进程

现在让我们将注意力稍微转向可用于处理线程和进程的函数调用 。**任何线程都可以启动一个进程**；唯一施加的限制是那些来自基本安全性的限制 *(文件访问、特权限制等)*。

想要启动一个进程大概有以下几种方法 *(具体的细节我们在这里不介绍)*：

1. **命令行启动**；
2. **使用 [`system()`](http://www.qnx.com/developers/docs/6.4.1/neutrino/lib_ref/s/system.html) 函数调用启动**：这是最简单的，直接在命令行输入就可以。实际上，`system()` 启动了一个外壳程序来处理您要执行的命令；
3. **使用 [`exec()` 系列函数](http://www.qnx.com/developers/docs/6.4.1/neutrino/lib_ref/lib-e.html) 和 [`spawn()`](http://www.qnx.com/developers/docs/6.4.1/neutrino/lib_ref/s/spawn.html) 函数调用启动**：当一个进程发出 `exec()` 函数，则该进程停止运行当前程序，并开始运行另一个程序；进程 ID 没有改变 ——— 进程变成了另一个程序；而调用 `spwan()` 函数则会创建另一个进程 *(带有新的进程 ID)*，该进程与函数的参数中指定的程序相对应。
4. **使用 [`fork()`](http://www.qnx.com/developers/docs/6.4.1/neutrino/lib_ref/f/fork.html) 调用启动**：完全复制当前进程，所有代码都是相同的，数据也与创建 *(或父)* 进程的数据相同。有意思的是，当您调用 `fork()` 时，您创建了另一个进程，该进程在相同的位置执行相同的代码，两个进程都将从 `fork()` 调用中作为父进程返回。*(感兴趣的同学可以自行搜索一下...)*
5. **使用 [`vfork()`](http://www.qnx.com/developers/docs/6.4.1/neutrino/lib_ref/v/vfork.html) 调用启动**：与普通 `fork()` 函数相比，`vfork()` 函数的资源消耗要少得多，因为它共享父线程的地址空间。`vfork()` 函数创建一个子线程，然后挂起父线程，直到子线程调用 `exec()` 或退出。另外，`vfork()` 可以在物理内存模型系统上工作，而 `fork()` 不能 —— `fork()` 需要创建相同的地址空间，这在物理内存模型中是不可能的。

## 如何启动线程

现在我们已经了解了如何启动另一个进程，让我们看看如何启动另一个线程。

任何线程都可以在同一进程中创建另一个线程。没有任何限制 *（当然，内存空间不足除外！）*。最常见的方法是通过 POSIX [`pthread_create()`](http://www.qnx.com/developers/docs/6.4.1/neutrino/lib_ref/p/pthread_create.html) 调用：

```c
#include <pthread.h>

int
pthread_create (pthread_t *thread,
                const pthread_attr_t *attr,
                void *(*start_routine) (void *),
                void *arg);
```

- 参数细节和更多线程管理的内容这里就不讨论了..


## 线程是个好主意

有两类问题，线程的应用是一个好主意。

- **并行化操作**，如计算许多数学问题 *(图形、数字信号处理等...)*；
- **共享数据执行几个独立的功能**，如服务器同时服务多个客户；

第一类问题很容易理解，把同一个问题分成四份并让四个 CPU 同时计算，这当然感觉会快上不少。

第二类问题稍微麻烦一些，我们借助网络中一台「计算图形结果并发往远端的节点」用来演示，并进一步说明「为什么单核 CPU 系统使用多线程仍然是一个好主意」。

假设我们有一个程序，需要先计算图形的部分 *(假设使用 "C" 表示)*，然后需要传输到远端 *(假设使用 "X" 表示)*，并最终等待远端回应做下一步操作 *(假设使用 "W" 表示)*，以下就是该程序在单核 CPU 下的使用情况：

![序列化的单核 CPU](https://imgkr2.cn-bj.ufileos.com/fee14383-a2e8-484c-bcfa-f7ccf0845420.png?UCloudPublicKey=TOKEN_8d8b72be-579a-4e83-bfd0-5f6ce1546f13&Signature=96sX3T0LAO51JoyM8McWrDSwAfA%253D&Expires=1596268074)

等一下！这浪费了我们许多宝贵的可用于计算的时间！因为等待远端回复的过程，CPU 做的仅仅是 "等待" 而已..

如果我们使用多线程，应该可以更好地利用我们的 CPU，对吗？

![多线程单 CPU](https://imgkr2.cn-bj.ufileos.com/99911d6b-3116-47ea-9f3d-8bca77e5abea.png?UCloudPublicKey=TOKEN_8d8b72be-579a-4e83-bfd0-5f6ce1546f13&Signature=LfyDJ0c6Hs0I9C5jrhYp4C3nETg%253D&Expires=1596268344)

这样好得多，因为现在，即使第二个线程花了一些时间等待，我们也减少了计算所需的总时间。

我们可以来简单计算一下。假设我们计算的时间记为 *T<sub>计算</sub>*，发送和等待的时间分别记为：*T<sub>发送</sub>* 和 *T<sub>等待</sub>*。那么在第一种情况下，我们的总运行时间为：

（*T<sub>计算</sub>* + *T<sub>发送</sub>* + *T<sub>等待</sub>*） x `任务数量`

而使用两个线程：

(*T<sub>计算</sub>* + *T<sub>发送</sub>*) x `任务数量` + *T<sub>等待</sub>*

直接减少了：

*T<sub>等待</sub>* x (`任务数量` - `1`)

> 请注意，我们速度最终还是受到以下因素的决定：
>
> *T<sub>计算</sub>* + *T<sub>发送</sub>* x `任务数量`
>
> 因为我们必须至少进行一次完整的计算，并且必须将数据传输到硬件之外 —— 尽管我们可以使用多线程覆盖计算周期，但只有一个硬件资源可以传输。

现在，我们创建一个四线程的版本，并且在 `4` 核的系统上运行它，那么我们最终将得到如下图示的结果：

![4 核 4 CPU 图示](https://imgkr2.cn-bj.ufileos.com/b669fc01-c058-4df2-9853-73f0b5705c41.png?UCloudPublicKey=TOKEN_8d8b72be-579a-4e83-bfd0-5f6ce1546f13&Signature=QCugWjxGwC63F%252FZ11dL%252BlLs6zyQ%253D&Expires=1596269380)

**请注意，四个 CPU 的每个利用率均未得到充分利用** *(如图示  "利用率" 中的灰色矩形所示)*。上图中有两个有趣的区域。当四个线程启动时，它们各自进行计算。不过，当线程在每次计算完成时，它们都在争夺传输硬件 *(图中的 "X" 部分 —— 一次只能进行一次传输)*。这算是在启动部分给了我们一个小异常。一旦线程经过此阶段，由于传输时间比计算周期的 `1/4` 小得多，因此它们自然会与传输硬件同步。首先，忽略小异常，该系统的特征在于以下公式：

（*T<sub>计算</sub>* + *T<sub>发送</sub>* + *T<sub>等待</sub>*） x `任务数量` / `CPU 数量`

该公式指出，在四个 CPU 上使用四个线程将比我们刚开始使用的单线程模型快大约 `4` 倍。

通过结合从简单拥有多线程单处理器版本中学到的知识，我们理所当然地希望拥有更多的 CPU 运行更多的线程，以便多余的线程可以 “吸收” 发送确认等待 *（和发送信息）* 中的空闲的 CPU 时间。

但是你尝试像我这样画一下 `8` 核 `8` 线程的情况，你就会注意到一些神奇的事情：我们仍然会遭遇利用率不足的情况，并且会发现 CPU 处于 "停滞等待" 状态的可能同时会有很多个。

**这给了我们一个很重要的教训 —— 我们不能简单的增加 CPU 以倍速我们的运算速度。**

我们希望越来越快，但存在许多限制因素。在某些情况下，这些限制因素仅受多 CPU 主板的设计 —— 当许多 CPU 尝试访问相同的内存区域时，会发生多少内存和设备争用。

> 尽管多线程多核给我们带来了许多好处，但 "更加复杂的环境" 也让我们面临更多的困难和挑战。可以参考我们类比的例子，在生活中，你面临的情况如果仔细思考和罗列的话，那将会是多么复杂.... 这里也不展开讲了

# 总结

相信看完的朋友都能够对进程和线程进一步加深印象.. **进程像房子，线程像住户**，非常深入人心！

原文章的后半程还详细地介绍了很多线程同步的更多内容，涉及 Linux 底层的函数调用，再后面一个章节还详细地介绍了进程间的通讯相关的内容，感兴趣的童鞋，**请移步原文！**

> - 本文已收录至我的 Github 程序员成长系列 **【More Than Java】，学习，不止 Code，欢迎 star：[https://github.com/wmyskxz/MoreThanJava](https://github.com/wmyskxz/MoreThanJava)**
> - **个人公众号** ：wmyskxz，**个人独立域名博客**：wmyskxz.com，坚持原创输出，下方扫码关注，2020，与您共同成长！

![](https://imgkr.cn-bj.ufileos.com/ace97ed9-3cfd-425f-85e5-c1a1e5ca7d3f.png)

非常感谢各位人才能 **看到这里**，如果觉得本篇文章写得不错，觉得 **「我没有三颗心脏」有点东西** 的话，**求点赞，求关注，求分享，求留言！**

创作不易，各位的支持和认可，就是我创作的最大动力，我们下篇文章见！