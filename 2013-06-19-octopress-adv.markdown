---
layout: post
title: "用Octopress建Blog(高级篇)"
date: 2013-06-19 11:53
comments: true
categories: Octopress
description: "用Octopress建Blog" 
keywords: Github Octopress 
---

1. 准备
	- 编码。octopress的配置文件如果包含有中文，则此文件必须使用无BOM的UTF-8编码，否则`rake generate`的时候会出错。 使用UltraEdit的`文件-->另存为`可以把文件保存为无BOM的UTF-8编码格式。
	
		![](/pics/octopress_adv_encode.jpg) 

1. 参考  
   本来是想自己写的，但是发现网上有一篇整理的很好的文章，就不重复劳动了。文章链接如下：  
	[http://www.cnblogs.com/oec2003/archive/2013/05/31/3109577.html](http://www.cnblogs.com/oec2003/archive/2013/05/31/3109577.html)

1. Disqus设置
	- 设置语言
		Setting-->General-->Language-->chinese
		![](/pics/octopress_adv_disqus_language.jpg)
	- 去掉广告
	![](/pics/octopress_adv_disqus_justcomment.jpg)

1. 关于More
	正确的书写应该如下，注意空格，很多网站就是空格错误，导致无效。

		<!-- more -->

