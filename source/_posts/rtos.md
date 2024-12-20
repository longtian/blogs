---
title: "[译] 等了 20 年，实时 Linux 终于进入了主线"
date: 2024-11-06 23:55:39
tags:
 - linux
categories:
 - 翻译
---


[原文](https://www.zdnet.com/article/20-years-later-real-time-linux-makes-it-to-the-kernel-really/
)

实时 Linux 相关的工作已经让开源操作系统受益了好多年，但直到这周 Linus Torvalds 才终于接受它的最后一部分进入主线内核。为什么会花这么久呢？

作者： Steven Vaughan-Nichols 资深特约编辑

2024 年 9 月 18 号

维也纳 -- 实时 Linux （`PREEMPT_RT`） 终于在 20 年后进入了主线内核。Linus Torvalds 在欧洲开源峰会上为这份代码祈福。为什么这是个大事件？让我们首先介绍一下实时操作系统（RTOS）和它适用场景。

# 什么是 RTOS ？

RTOS 是一种专门的操作系统，旨在精准和可靠地处理时间关键任务。与 Windows 或者 MacOS 这类通用操作系统不同，RTOS 建立之初是为了在严格的时间限制下响应事件和处理数据，通常是以毫秒或微秒为单位。正如著名的实时 Linux 开发者、谷歌员工 Steven Rostedet 所说：“实时是最快的最坏场景。” 

他的意思是 RTOS 最基本的特征是确定性行为。RTOS 能够保证关键任务在指定期限内完成。很多人认为 RTOS 就是为了快速处理。其实不然，速度不是 RTOS 的关键，可靠性才是。这种可预测性在时间关键应用中尤为重要，例如工业控制系统，医疗设备和航空航天器材。

当今一个正被使用的 RTOS 例子是 VxWorks，它在 NASA 火星探测器上被用来导航，并在波音 787 梦想号客机上用作飞行控制系统，确保飞行控制的实时响应。另一个例子是 QNX Neutrino，它被广泛应用与车载信息娱乐系统和高级辅助驾驶系统中，比如防抱死刹车系统。

# 实时 Linux 的历史

实时 Linux 的代码现在已经通过即将发布的 Linux 6.12 内核被纳入到了所有的 Linux 发行版中。这意味着 Linux 很快会出现在更多的关键任务设备和工业硬件上。 这着实花了不少时间。

实时 Linux 起源于 1990 年代对实时应用的不断增长的需求。早期工作主要致力于在 Linux 内核之外创建一个单独的内核。它包括若干个学术项目，比如堪萨斯大学的 KURT 项目，米兰大学的 RTAI 项目和新墨西哥矿业理工学院的 RTLinux。

到 2004 年，Ingo Molnar，一个资深的 Linux 内核开发者开始收集和重塑这些技术碎片，并奠定了实时抢占补丁集 `PREEMPT_RT` 的基础。

这个方法和早期的实时 Linux 的方案不同，因为它修改了已有的 Linux 内核而不是创建另外一个单独的实时内核。到 2006 年，它已经获得了足够的关注，以至于 Linus  Torvalds 说 “用 Linux 控制一束激光很疯狂，但这个屋子每个人都或多或少有一些疯狂。所以如果你想用 Linux 来控制工业焊接激光器，我对你使用 PREEPT_RT 没意见”

到 2009 年，一个包含了 Thomas Gleixner, Peter Ziljstra 和 Rostedt 的内核开发者小队，已经把之前的若干原型开发整合成了一个单独的树外补丁集。也就是从这个时候开始，很多公司开始使用这个补丁集来创建需要精确到毫秒的硬实时工业系统。

随着项目推进，它的很多组件已经进入了内核。 Rostedt 告诉我，说实时现在才进入 Linux 在某种程度上是错误的。它的很多功能已经进入了主线 Linux 很多年。有些对于你日常使用的 Linux 已经是必不可少。

比如，你可能从来没听说过 “NO_HZ” ，它能够减少系统在空闲状态下的功耗。“NO_HZ” 使得 Linux 能够在有几千个 CPU 的机器上高效运行。“你感受不到实时补丁给 Linux 带来了多大的改进” Rostedt 强调说 “我们的工作是 Linux 能够在数据中心里运行的唯一原因”

因此，如果没有 “NO_HZ”，Linux 就基本上不能在所有数据中运行。这也反过来解释了为什么 Linux 能在云上运行。我不知道没有这个实时补丁的贡献世界将会怎么样，但是它一定不像今天这样。

起初，谁做梦都不会想到实时 Linux 被证明有用的方式。 Rostedt 回忆道 “早在 2005 年的时候，我收到一个关于实时的错误报告，我回复：‘嘿这是修复的代码，你能应用它吗？’那个家伙说‘我不知道我在做什么’，我回复：‘等等你不是一个内核开发者吗？’他答道：‘我是一个吉他手’” 

原来，他之所以会用到早期的实时性补丁是因为他在用一个叫做 JACK 的系统，这是一个低延迟音频链接的声音服务器。他和大多数音乐人一样，穷到没有钱买高级的设备，Rostedt 继续说道 “他搞到一台便宜的笔记本，上面装着 Linux 和 JACK，正因为实时性补丁，它能够录到很好的音频，没有由于硬盘写导致的声音跳跃”

结果就是很多音乐家都是实时 Linux 的早期用户，因为这让他们能够在便宜的设备上录制高质量的唱片。这谁会知道？近几年，其它悄然进入内核的实时特性还包括

- 互斥锁的引入
- Ftrace，可以说是最重要的 Linux 调试工具
- 用户空间应用程序的优先级继承

# 实时 Linux 怎么用了这么长时间？

那么为什么实时 Linux 直到现在才进入内核呢？“实际上我们不会推上去任何东西，除非我们觉得准备好了” Rostedt 解释到 “几乎所有的代码都会被重写三次以上才会进入到主干，因为我们的准入门槛非常高”

此外，通往主线之路并不只是一个技术挑战。政治和观念也起了很重要的作用，“开始的时候我们甚至不能提实时” Rostedt 回忆道，每个人都会说 “哦，我们不在乎实时”

另外一个问题是资金。实时 Linux 开发经费在好几年里飘忽不定。在 2015 年， Linux 基金会成立了实时 Linux （RTL） 的合作项目，来协调围绕主线 `PREEMPT_RT` 的维护工作。

全面集成前最后一个关卡是内核 `print_k` 函数的重构，这是一个可以追溯到 1991 年的重要调试工具。Torvalds 对 `print_k` 有很强的保护欲，他写的原始的代码并且一直用到今天。然而，每当 `print_k` 在被调用的时候也导致了 Linux 程序产生严重的延迟。这种延迟在实时系统里是无法接受的。

Rostedt 解释说：“`print_k` 为了处理上千种不同的场景有上千个 Hack。每当我们想要修改 `print_k` 做些什么的时候，它就会破坏其中某个场景。`print_k` 实际上是非常棒的调试工具，他能让你准确知道一个进程在哪里挂掉了。当我非常努力想要锻造系统的时候，延迟总是在 30 毫秒左右，然后突然的延迟就降到了 5 微秒” 这个延迟就是 `print_k` 消息。

经过大量工作，多次激烈讨论和若干被毙掉的提案后，今年早些时候终于达成了折中方案。这下 Torvalds 高兴了，实时 Linux 开发者高兴了，`print_k` 用户也高兴了，到此实时 Linux 才终成真。

经历了 20 年的不懈努力，Linux 实时补丁终被纳入主线内核。这一里程碑标志着内核开发者将确定性、低延迟性能带进了 Linux 的多年的努力达到了顶峰。

有了它，Linux 内核才可以完全抢占，从而能在微秒之间相应事件。这个能力对于要求精确计时的如工业控制系统、机器人和音频制作的应用来所是非常关键的。

随着实时性补丁的合并， Linux 现在已经成为 RTOS 世界一个重要玩家。这不只是实时开发者的胜利更是所有 Linux 用户的胜利。