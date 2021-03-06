---
layout: post
title: "使用 SystemRescueCD 安装 gentoo"
date: 2014-08-06 17:50:10 +0800
comments: true
categories: ['linux']
keywords: "linux, gentoo, x11, awesome, SystemRescueCD"
description: 使用 SystemRescueCD 快速的安装 gentoo，然后安装 awesome 桌面环境。
---

在使用 Ubuntu 有 3 年之后，第一次有了换个发行版的想法。
其实我觉得做任何事情都要循序渐进，在合适的时候做合适的事情才能事半功倍。

3 年前选择从 Ubuntu 入手应该是不错的选择，现在想换一个更自由的发行版也是水到渠成。选来选去，选了 gentoo，主要是在浏览的过程中以下几点吸引了我:

1. 自由度高，一切从头开始
2. 升级频繁
3. 很先进的包管理工具

其实大多数时候我不是一个爱折腾的人，所以这次抓住了一闪而过想折腾的机会，第一天下午就开始动手了。第一个动作就是买一个 U 盘，是的，要准备一个 U 盘。

<!-- more -->
在开始之前，先说一下各个工具的版本。

```
SystemRescueCD == 4.3.0
kernel == linux-3.14.14-gentoo
gentoo == Gentoo Base System release 2.2
```

原本我只是打算照着 gentoo 的手册从头看到尾，但第一章我就晕了，至少给了 3 种安装方式。选哪个好呢？经同事介绍：都不好。
有网络的条件下，最好是使用 SystemRescueCD 来安装。一开始我不太理解，装完之后我同意这种看法。

先讲 SystemRescueCD，这是一种特殊的 Live CD，内置了一个基于 Gentoo 内核的小型系统，以及一些很棒的工具。就像名字所指，主要用来恢复系统。
而这些「很棒的工具」就是 SystemRescueCD 来安装 gentoo 的最大优势。比如自带了 dhcp 客户端，很舒服的 bash 环境，elinks 在终端浏览网页等等。

简单来说，使用 SystemRescueCD 来安装 gentoo，可以直接跳到 gentoo 手册的第 4 章。而把 SystemRescueCD 装进 U 盘，只要参考 [wiki][1] 就可以了，分分钟就能搞定。

[1]: http://www.sysresccd.org/Sysresccd-manual-en_How_to_install_SystemRescueCD_on_an_USB-stick

制作完成之后，插上 U 盘，从 U 盘启动，进入 systemrescd。然后我们跳到 [gentoo handbook 第 4 章][2]。第一，不要看中文版，已经不维护了，全部过时；第二，选好 CPU 架构，这篇是针对 amd64 或者叫 x86_64（这名字无力吐槽）。

[2]:https://www.gentoo.org/doc/en/handbook/handbook-amd64.xml?part=1&chap=4

gentoo 手册不只是告诉你步骤，还试图向你解释每一步的原因，当然可以选择略过。所以你也可以读完本文的一些解释，也可以略过:D

第 4 章的几个关键字：MBR， GPT， UEFI， BIOS boot partision。简单的把这些关键字串一下：上一代的计算机启动方式是 BIOS + MBR(Master Boot Record)，但两者有很多限制，也确实太过古老了。比如 MBR 使用 512 bytes 记录主分区信息，所以主分区数量受限制，虽然可以使用扩展分区和逻辑分区来弥补，但不够 native；以及 MBR 没有备份等问题。GPT 就是 MBR 的升级版，而 UEFI 就是针对 BIOS 的改进。而 BIOS boot 分区 主要是为了 GRUB2 准备的，也就是说如果使用 GRUB2 来选择操作系统，最好是有一个。需要特别说明的是，UEFI 需要主板支持，如果没有的话 BIOS + GPT 在大多数情况下可以的，除了要安装 windows（只认 UEFI + GPT 组合）。

给硬盘分好区并 mount 之后，就可以进入第 5 章。这里主要是准备整个安装过程会用到的一些工具，比如 stage3 安装包。名字很有趣，台阶。要知道除了 stage3 之外还有 stage2 和 stage1 可以选择。大概意思是你要从第几层开始工作，基本上越底层自己要做的工作就越多。新手就用 stage3 可以了，想要挑战的可以试试 stage2 和 stage1，这让我想起了 Diablo 3 的地狱和炼狱模式。这里需要注意的是，新的 SystemRescueCD 带的是 elinks，而不是手册上的 links，但用法基本是一样的。

第 6 章是装内核之前的准备，唯一要注意的是在选择 profile 的时候，因为是要装桌面环境，但我又不想用 GNOME 和 KDE，所以选择 desktop 选项就好了。

第 7 章安装内核，配置内核参数太过复杂，第一次安装基本就一路默认吧，或者直接使用 genkernel 让它自己来。这里 initramfs 最好装一下，虽然我还没有深入理解它的作用，但看介绍是需要的。

第 8 章开始装一些必要的软件了，这里唯一的问题是网卡的名字和默认的不一致。原因在[这里][3]有详细解释，大概是说新版的 linux 设备管理器不再使用 `eth0` 这样的名字，而是给每个设备都按照固定规则分配一个固定的名字比如 `enp0s25`，所以网络配置需要拿到真实的名字之后重新配一下。

除了手册上的软件之外，我这里再补充 2 个。`sudo` 居然也没有，这个挺让我惊讶的，另外一个就是 `git`。

[3]: http://www.freedesktop.org/wiki/Software/systemd/PredictableNetworkInterfaceNames/

然后一路到底就可以啦，整个过程最需要的条件是内存和网络。而我选择了跳过最复杂的 USE 配置及内核参数配置，所以还算顺利。

如果重启后无法进入系统，恭喜你获得了再来一次的机会。
