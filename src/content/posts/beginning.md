---
title: Beginning
published: 2026-06-20
description: Beginning of the blog
tags: [日常]
category: 日常
slug: begin-of-blog
draft: false
---

第一篇文章，就当测试了，随便写点什么。  
博客，说白了也就是存放些自己生活记录的地方。从一开始的Word到OneNote，后来换成Notion用到现在。之前就开始觉得Notion的功能愈发臃肿，又是ai又是agent的。感觉到了26年，是个软件都想方设法的和这两玩意沾上些边。而且notion国内直连也变得不太稳定，多端文章常常同步不过来。  
正好几个月前b站给我推荐了些blog搭建和主题相关的内容，当时就有了这个想法。最近也是终于抽出空来把这个想法实现了。  
blog框架在hexo，hugo，WordPress，VuePress等等大大小小的都看了一遍后发现，我对性能与开销反正也没什么特别要求，要实现一些定制化功能也能借助AI实现，所以框架对我来说也没啥区别。那剩下的就是挑选一个适合自己的主题了，毕竟我是一点设计感都没有。用AI搓一堆屎山出来还不如直接用别人的。  
在一顿挑选后，还是觉得之前b站推荐的[Mizuki](https://github.com/LyraVoid/Mizuki)还不错。以及同样基于[Fuwari](https://github.com/saicaca/fuwari)二开的[Firefly](https://github.com/CuteLeaf/Firefly)，在看了一些社区评价后，最后果断选择了Firefly。  
整体的部署还是挺简单的，看着官方文档进行配置基本没有问题，相关的教程也不少。除了配置评论用的twikoo花了些时间，其它基本半天就能搞定了。实际上是注册vercel时候，可能节点ip被风控了，需要发个投诉邮件才能注册。当时想的是鬼知道他们什么时候回，于是换成了Netlify托管，也挺方便的。blog用cf workers，主打一个白嫖。  
本来还想着把notion那边的文章迁移过来，但是一方面它不是markdown，我还要手动排下格式。另一方面，我懒，干脆拉到了。而且2026年就没写过几篇文章，可能再过个几个月回来看还是只有这一篇文章。  
:spoiler[懒狗要睡觉了]