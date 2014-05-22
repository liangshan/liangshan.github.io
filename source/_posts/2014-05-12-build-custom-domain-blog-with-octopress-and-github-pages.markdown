---
layout: post
title: "使用 Octopress 在 GitHub Pages 上快速搭建自定义域名Blog"
date: 2014-05-12 14:17:19 +0800
comments: true
categories: [meta]
keywords: octopress,github  
description: How to build custom domain blog with octopress and GitHub Pages
---

一直没有搭博客，因为一直感觉自己不是一个标准的技术人，主要是不够狂热。但受身边人熏陶，偶尔还是想写点东西。
尤其是看中 `liangshan.me` 这个域名，买下之后觉得闲着也是浪费，所以有了利用 GitHub Pages 来搭一下的想法。

## 预备
+ 你需要具备一些 `Git` 的常识
+ 你需要一张 Visa 卡或是支付宝账号
+ 如果有 Ruby 经验，将会帮你更好的理解这个系统

## Quick Start
整体来说，整个部署过程分3大步

1. 购买域名
2. 配置解析
3. 部署至 GitHub Pages
4. 评论插件

<!-- more -->

### 购买域名
这里选用 [Godaddy](http://www.godaddy.com)， 理由很简单，就是他支持支付宝。购买和支付流程很顺畅，应该不会有什么障碍。

### 配置解析
为了防止 Godaddy 自己的 DNS 解析被墙[1]，所以选用了国内的 [DNSPod](https://www.dnspod.cn) 来解析域名，免费服务够用，还有手机二次验证。

1. 在 Godaddy 上设置 NameServer 到 DNSPod 提供的地址
2. 如果是顶级域名，需要在 DNSPod 上设置一个 A 记录到 Github Pages 的 IP 地址，这个地址可能会变化，可以查看[这里](https://help.github.com/articles/my-custom-domain-isn-t-working)获取最新的地址
3. 如果是子域名，可以设置 CNAME

关于 DNS 的设置，更详细的可以看 [GitHub Pages 的说明文档](https://help.github.com/articles/setting-up-a-custom-domain-with-github-pages#step-1-add-a-cname-file-to-your-repository)

### 部署至 GitHub Pages
关于 GitHub Pages 的技术细节，看[2]可能会有收获，一句话来概括就是**使用动态语言来编辑纯文本文件，生成静态 HTML 代码，然后部署在 GitHub 上。**

这种方式很巧妙，我之前也一直在用[类似的做法](https://github.com/liangshan/markdown2deckjs)管理我的 Slides。

这里并没有直接使用 Jekyll，而是使用基于他的 [Octopress](http://octopress.org/docs/setup/)。遵循页面的 `Next Steps` 应该可以很快搭好环境。

并且 Octopress 做好了和 GitHub Pages 对接的[脚手架](http://octopress.org/docs/deploying/github/)。

这里主要记录一些踩到的坑：

1. 使用 Homebrew 安装 rbenv, 再用 rbenv 安装 ruby, 提醒最新的 MacOS 安装的 Xcode 并没有 GCC，需要手动安装 GCC
2. `~/.rbenv/shims` 目录需要添加至 PATH，Homebrew 安装过程中好像并没有做这一步，并且 Octopress 关于这部分的文档可能过时了，文档中使用的 `~/.rbenv/bin`

### 评论插件
由于是纯静态页面，所以要增加评论的动态功能 需要借助一些第三方前端工具。比如 [Disqus](http://disqus.com)，配置非常简单 更多参考[3]。

## 参考资料
1. [使用Github Pages建独立博客](http://beiyuu.com/github-pages/)
2. [搭建一个免费的，无限流量的Blog----github Pages和Jekyll入门](http://www.ruanyifeng.com/blog/2012/08/blogging_with_jekyll.html)
3. [给octopress添加Disqus评论功能](http://seagg.github.io/blog/2012/09/03/config-comment-on-octopress/)
