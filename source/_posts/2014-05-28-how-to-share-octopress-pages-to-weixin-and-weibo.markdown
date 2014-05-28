---
layout: post
title: "如何将 octopress 的文章分享至微信和微博"
date: 2014-05-28 12:54:14 +0800
comments: true
categories: [meta]
keywords: "微信,微博,octopress,分享"
---

昨天想把自己的文章分享到微信，没有仔细思考选了一种很土的办法。就是直接在微信的内容里输入地址，容易出错不说，效率也很低下。
今天再回头思考这个问题，想到可以像 twitter 和 google+ 一样做一个分享按钮。Octopress 自己是不带国内社交网站分享的，要稍微做点功课。

<!-- more -->

稍微想了一下，发现微信较为特殊，因为只有手机应用，想要把网页分享至 app，只能曲线救国，想办法把网址输入到手机，然后使用 app 的功能再分享。

后来搜索了一下，发现 [jiathis.com](http://www.jiathis.com) 已经按照这个思路做过了。自不必说是「模仿」 [addthis.com](http://www.addthis.com)，连域名都如此相似:P。

## 注册

需要先注册一个 jiathis 用户，注册过程很简洁，好像也没有验证邮箱。注册完就直接跳转到加码页，也算是体验不错。

## 修改 Octopress

接下来需要做 4 项源代码级别的修改。修改的流程遵循 Octopress 自己的逻辑，比葫芦画瓢的改法，而不是直接贴代码了事。

### 添加 config

在 `__config.yml` 中添加 jiathis 的相关配置，`$uid` 替换为真正的 `userid`，`userid` 可以从 jiathis 的示例代码中获取。

```yaml __config.yml
## 下面的内容添加至文件尾部

# Jiathis
jiathis_user: $uid
jiathis_share: true
```

### 修改模板文件

下面的代码首先是拆成几部分，上面讲过要符合 octopress 的做法，另外我精简过了。
jiathis 给的代码带一个「+」号还有分享数字，样式很不河蟹被我删掉了。
另外 jiathis 还会默认开启 site tracker 功能，我觉得这个有点多余，这部分代码我也都删掉了。

{% raw %}
```html+jinja source/_include/post/sharing
  <!-- 添加到相应的位置，twitter google+ 前后皆可 -->
  {% if site.jiathis_share %}
  <div class="jiathis_style_24x24" style="text-indent: 0px; margin: 0px; padding: 0px; border-style: none; float: none; line-height: normal; font-size: 1px; vertical-align: baseline; display: inline-block;  background: transparent;">
    <a class="jiathis_button_weixin"></a>
    <a class="jiathis_button_tsina"></a>
  </div>
  {% endif %}
```

```html+jinja source/_includes/after_footer.html
<!-- 只加第 5 行 -->
{% include disqus.html %}
{% include facebook_like.html %}
{% include google_plus_one.html %}
{% include twitter_sharing.html %}
{% include jiathis_share.html %}
{% include custom/after_footer.html %}
```

```bash 添加一个文件
$ touch source/_includes/jiathis_share.html
$ vi source/_includes/jiathis_share.html
```

```html+jinja 该文件的内容
{% if site.jiathis_share %}
<div style="display:none">
<script type="text/javascript" src="http://v3.jiathis.com/code/jia.js?uid={{ site.jiathis_user }}" charset="utf-8"></script>
</div>
{% endif %}
```
{% endraw %}

之后就可以在本地预览并部署了。
