---
layout: post
title: "isolate ansible code and source code"
date: 2014-05-22 16:56:16 +0800
comments: true
categories: [ansible,vagrant,DevOps]
keywords: ansible,vagrant,isolate,source code,虚拟环境,隔离,源代码
description: How to isolate ansible code and source code when using vagrant for develop environment.
---

我们经常要开发一些「系统」，这些「系统」具备以下特点：

+ 用到很多系统软件。比如 ngnix, compass, ruby, mongodb 等等；
+ 项目本身有很多组件组成。比如 web 程序，job 系统和对应的消息队列；

那么在开发这类系统时就会遇到一些问题：

+ 开发环境和线上环境不等价。平台可能不同，软件版本可能不同；
+ 开发环境配置复杂，任意环节出错就会影响整个系统的启动；

为了解决这些问题，我们将整个环境安装在虚拟机内，这在安居客被认为是一种最佳实践，成功的应用在很多系统的开发当中。
而开发环境，生产环境的部署则交给 ansible 来完成。那么我们的项目目录看起来是这样:

```
 .                  # root
 |-- .provisioning/ # ansible 脚本目录
 |-- src/           # 源代码目录
 |-- Vagrantfile    # vagrant 启动脚本
```

这样有一个好处，虚拟机启动之后，源代码对应 `/vagrant` 这个共享目录，修改源代码会同时在虚拟机内生效。
但这样 ansible 或是其他一些 CM(Configure Management) 工具的代码和源代码就会混在一个仓库内。

本文就介绍一种简单的方法可以将 DevOps 的代码与源代码隔离，并达到相同的效果。

<!-- more -->

首先我们需要准备两个仓库，一个源代码仓库我们暂且叫做 `src`，另一个仓库 `src-ansible` 放 ansible 脚本。

## src-ansible

```
 .
 |-- host_vars/
 |-- roles/
 |-- playbook.xml
 |-- host.vagrant
 |-- .Vagrantfile
```

其他都是常规的 ansible 脚本，唯独 `.Vagrantfile` 比较特殊。

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "trusty32"
  config.vm.box_url = "http://vagrant.corp.anjuke.com/boxes/trusty32.box"
  config.vm.network :private_network, ip: "192.168.222.22"
  # config.vm.network "public_network"

  config.vm.provision :ansible do |ansible|
    ansible.playbook = '.provisioning/playbook.yml'
    ansible.inventory_path = '.provisioning/hosts.vagrant'
    ansible.host_key_checking = false
    ansible.verbose = 'vvvv'
  end
end
```

你会发现这里面用到的 `playbook` 和 `inventory` 路径都不存在，文件名也以 `.` 开头。其实这个文件就是给源代码仓库用的，而不能在这里直接使用。

这里借鉴了 Ruby On Rails 的一个重要思想「约定大于配置」。比如这里约定了源代码仓库的 ansible 脚本放在 `.provisioning` 这个目录。

## src

```text
 .
 |-- src/
 |-- Makefile
 |-- .gitignore
```

`.gitgnore` 添加以下内容：

```text
.vagrant/
.provisioning/
Vagrantfile
```

`make` 其实就是执行一些 shell 命令，如果翻译成 shell 应该类似下面的脚本。

```
mkdir .provisioning
cd .provisioning
git init
git remote add origin git@gitserver.com:username/src-ansible.git
git pull origin master
cp .Vagrantfile ../Vagrantfile
```

于此对应，再写一个 `make` 的 `target` 用于更新 ansible 脚本。

```
cd .provisioning
git pull origin master
cp .Vagrantfile ../Vagrantfile
```

完成了这些步骤，就可以直接在源代码仓库使用 vagrant 的命令来安装、更新虚拟机。但又可以将这些脚本与源代码隔离开来。
