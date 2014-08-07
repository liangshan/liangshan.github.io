---
layout: post
title: "在 gentoo 上安装 awesome"
date: 2014-08-07 14:06:30 +0800
comments: true
categories: ['linux']
keywords: "linux, gentoo, x11, awesome, SystemRescueCD"
description: 使用 SystemRescueCD 快速的安装 gentoo，然后安装 awesome 桌面环境。
---

都装 gentoo 了，趁热装桌面环境吧。linux 的视窗系统称为 X Window System。
在开源世界里，通常是协议先行，各自实现。同样 X 只是协议，在各种实现中以 X.Org 最受欢迎，当时使用的协议为 X11，所以后来 X 也被人们称为 X11。

<!-- more -->

X 分为 server 和 client, 但这里的 server 和 client 和我直觉上的理解有点相反。比如接收用户行为的反而是 server。另外一大特色就是 client 之间是不知道对方存在的，比如桌面上运行了 terminal 和 chrome，它们之间的位置关系互相是不清楚的。所以需要一个软件来管理 client 的位置、大小、外观，同时提供类似于桌面的环境，这个软件就叫 X window manager，我们这里选择的是 awesome。X 具体的协议要去翻协议或者看 wiki 可以了解大概。

安装也分为 server 和 window manager 来进行。gentoo 安装每个软件基本上都会有一个 wiki 页面，照着手册来就可以了，所以建议照着 gentoo 的 wiki 来安装软件，而不是软件本身的手册。下面只讲手册上没说的内容，也就是我踩到的坑。

先讲 X server，第一次使用的驱动是开源版本 nouveau，也是手册上推荐的版本。但 awesome 装好之后无法启动。重新改了 make.conf 把显卡模块改为 nvidia 重装 server 就成功了。不知道是某款型号的个体问题还是普遍问题，仅供参考吧。

再说 awesome，首先要装较新版本的 awesome，这时候安装使用下面的命令：

```bash
$ ACCEPT_KEYWORDS="~*" emerge x11-wm/awesome
```

装完之后运行 `startx` 应该有一个朴素的桌面启动了，这时候我们要花很多时间来美化它。

我是说在没有社区的情况下:D

github 上搜索 awesome themes，结果一大把了，早有人帮大家归纳了很多漂亮的皮肤，即使没有一个喜欢的，参考配置文件总是不错的。

我使用的叫做 [awesome-copycats][1]，除此之外还有中文字体，输入法，终端配置等等要折腾。我把用到的配置文件都放进 [github][2] 上，省的再次折腾。这里只是需要几个备注：

1. 这个皮肤的网络控件需要先安装 `iproute`
2. 编译终端需要 `USE=xft` 才能使用 X Font Server 的字体

最后安装一下最新版的 firefox（仍然使用 `ACCEPT_KEYWORDS="~*"` 确保新版），需要提到的是 perl 的 `XML-Parser` 模块过期了，可能要先升级一下，否则编译过程会报错。

要注意整个过程都没有安装 DM(display manager)，还是手动的 `startx` 来启动，以后遇到问题再装吧。

上个图吧（为什么有种女人写美容攻略的感觉？），其实我最早就是想弄个 awesome，没想到整了这么多。

<img src="{{ root_url }}/images/custom/awesome.png" />
