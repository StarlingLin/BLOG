---
title: 关于Shoka主题Valine设置邮件提醒
date: 2024-04-07 11:07:36
category:
  - [AboutShoka]
tags:
  - Shoka
---

:::warning
本篇写于2024年4月7日，请注意时效性。
:::

本人在配置Shoka主题的Valine评论提醒时，发现霜月琉璃对其配置方法描述不够详细且已经过时，故有此文。

原贴链接：

{% links %}
- site: Shoka
  owner: 霜月琉璃
  url: https://shoka.lostyu.me/computer-science/note/theme-shoka-doc/config/#%E6%96%87%E7%AB%A0%E8%AF%84%E8%AE%BA
  desc: 文章评论
  image: https://cdn.jsdelivr.net/gh/amehime/shoka@latest/images/avatar.jpg
  color: "#e9546b"
  {% endlinks %}

Valine的评论系统是基于LeanCloud运转的，其评论内容以结构化数据存储在LeanCloud。

而邮件提醒，也就离不开LeanCloud的云引擎功能。原贴给出的Valine-Admin和本文给出的Valine-Admin，都是基于云引擎运作的。

这就不得不提到LeanCloud的问题了，LeanCloud分国内版和国际版双端。对于国内版，云引擎域名必须绑定已经在国内备案的域名，因此很多人都是转向国际版。

但是我转向LeanCloud国际版后发现Shoka主题预设的Valine似乎已经不能在LeanCloud国际版正常运作，所有评论都会遇到403/405，无法传递到Valine的服务器，只能使用国内版。

那么再使用原贴的需要绑定域名和添加定时器的Valine-Admin就不那么好了。

我最终选择了[这个Valine-Admin](https://github.com/billchen2k/Valine-Admin/)。可以正常运作在LeanCloud上，收到的邮件提醒如下：

![效果图](1.png)