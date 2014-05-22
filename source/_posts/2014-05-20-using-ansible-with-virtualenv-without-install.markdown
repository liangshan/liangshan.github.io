---
layout: post
title: "使用 vitualenv 绿色安装 ansible"
date: 2014-05-20 14:45:59 +0800
comments: true
categories: [ansible, virtualenv]
keywords: ansible,virtualenv,without install
description: How to use ansible with virtualenv without install.
---

[Ansible](http://www.ansible.com/home) 是一个自动化配置管理工具 (Automate configure management)。用 python 编写，所以安装方式一般有以下几种：

** `pip` 安装 **

    $ pip install ansible

** MacOS 用户可以选择使用 `homebrew` 安装 **


    $ brew install ansible

但这两种方法都不可避免将「污染」系统的 python 环境。所以本文主要介绍如何绿色安装 Ansible。

<!-- more -->

### 安装 Virtualenv

不污染系统的关键在于虚拟化 python 的环境，所以需要准备 virtualenv。

这里偷懒选择使用 pip 安装，如果连 virtualenv 也不想装在系统级别也是可以的，就不在这里介绍了。

    $ pip install virtualenv

### 下载 ansible

    $ git clone git@github.com:ansible/ansible.git
    $ cd ansible

### 安装

    $ virtualenv .virtualenv
    $ .virtualenv/bin/python setup.py develop

### A little trick

这个时候使用 virtualenv 激活当前目录的 python 环境，其实 `bin/ansible` 就已经可以工作了。

但我们还希望将 ansible 命令加在系统的 PATH 里，这就需要一些小技巧。简单来说需要自己准备一个脚本，脚本的内容如下:

```bash
$ touch ~/bin/_ansible_wrapper
$ vim ~/bin/_ansible_wrapper

#!/bin/bash
source "$HOME/apps/ansible/hacking/env-setup" -q
if [ -f "$ANSIBLE_HOME/.virtualenv/bin/activate" ] ; then
    source "$ANSIBLE_HOME/.virtualenv/bin/activate"
fi
PYENV_VERSION=2.7.6 exec "$ANSIBLE_HOME/bin/${0##/*/}" $@
```

`$HOME/apps/ansible/hacking/env-setup` 是 ansible 为 hacker 准备的一个脚本，设置一些环境变量。其中 `$HOME/apps/ansible` 是 ansible 的根目录。


`${0##/*/}` 比较有趣，作用是正则匹配 `/*/` 然后把匹配到的部分从第 0 个参数（即命令本身）中删除。关于 bash 参数，更详细可以看[1]。

`$@` 则比较常见，是 bash 拿到的除命令名之外的所有参数。

其实这个脚本就是一个 wrapper，作用如下：

1. 设置相关环境变量
2. 激活 ansible 下的 python 环境
3. 将收到的所有命令都转发给 `$ANSIBLE_HOME/bin` 下的相同命令执行

假设 `$HOME/bin` 在 PATH 中，设置几个软链就全部搞定了。

    $ ln -s $HOME/bin/_ansible_wrapper $HOME/bin/ansible
    $ ln -s $HOME/bin/_ansible_wrapper $HOME/bin/ansible-playbook
    $ ln -s $HOME/bin/_ansible_wrapper $HOME/bin/ansible-galaxy


### 参考资料

1. [Linux tip: Bash parameters and parameter expansions](http://www.ibm.com/developerworks/opensource/library/l-bash-parameters/index.html)
