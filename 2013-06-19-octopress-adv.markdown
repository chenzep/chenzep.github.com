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
 
<!-- more -->
1. 常用设置 
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

1. 修改代码区块的颜色
	Octopress默认的代码区块字体的颜色太暗，改明亮一下，修改`sass\partials\_syntax.scss`文件
		
		p, li {
		  code {
		    @extend .mono;
		    display: inline-block;
		    white-space: no-wrap;
		    background: #fff;
		    font-size: .8em;
		    line-height: 1.5em;
		    color: #888888; /* 原来是#555 */
		    border: 1px solid #ddd;
		    @include border-radius(.4em);
		    padding: 0 .3em;
		    margin: -1px 0;
		  }
		  pre code { font-size: 1em !important; background: none; border: none; }
		}
1. 本地图片预览  
	a. MarkdownPad2是支持图片预览功能的(好像要Pro版本?)。如果使用的本地图片，比如`![](/pics/1.jpg)`, 需要在Markdown文件的目录下存在`pics/1.jpg`图片，MarkdownPad2才能正确显示图片。因此，本地图片一rak般放到`octopress\source\_posts\pics`文件夹下。  
	b. 为了避免使用`git add .`命令的时候把`pics`文件夹的图片也添加进入，我们需要添加一个`.gitignore`文件让git来忽略pics文件夹，文件的内容很简单,输入忽略的目录名称即可：`pics`  
	c. 加入了`.gitignore`文件之后，会导致`rake deploy`阶段出错，我们修改`octopress/Rakefile`文件，忽略对`.gitignore`文件的处理，修改如下:  

			desc "copy dot files for deployment"
			task :copydot, :source, :dest do |t, args|
			  FileList["#{args.source}/**/.*"].exclude("**/.","**/..", "**/.DS_Store", "**/._*","**/.gitignore").each do |file|
			    cp_r file, file.gsub(/#{args.source}/, "#{args.dest}") unless File.directory?(file)
			  end
			end
	d. 用MarkdownPad2编译好文件之后，运行`rake generate`,`rake preview`更新和启动预览，然后再浏览器输入`localhost:4000`，发现浏览页面的图片显示也不正确。这是因为`octopress\public`目录下没有对应的图片导致的。修改`octopress/Rakefile`文件，把`octopress\source\_posts\pics`目录的内容拷贝到`octopress\source`目录,然后jelly程序会把`octopress\source`目录的内容拷贝到`octopress\public`目录。修改如下：

		desc "Generate jekyll site"
		task :generate do
		  raise "### You haven't set anything up yet. First run `rake install` to set up an Octopress theme." unless File.directory?(source_dir)
		  puts "## Copy Pictures"
		  cp_r "#{source_dir}/_posts/pics", "#{source_dir}"
		  puts "## Generating Site with Jekyll"
		  system "compass compile --css-dir #{source_dir}/stylesheets"
		  system "jekyll"
		end

	e. 经过上面的修改之后，浏览器本地预览的时候图片显示就正常了。在`rake deploy`阶段，脚本会把`public`目录下的图片拷贝到`_deploy`目录，并更新到github服务器。这个过程是脚本完成的，我们不需要参与。  
	f. 至此，本地图片预览处理的整个流程完成。

1. MinGW中文支持  
	a.进入RailsInstaller下的Git\etc目录。  
	b.编译`profile`文件，添加如下内容
	
		export LANG=en
		alias l='/bin/ls --show-control-chars --color=auto'
		alias la='/bin/ls -aF --show-control-chars --color=auto'
		alias ll='/bin/ls -alF --show-control-chars --color=auto'
		alias ls='/bin/ls --show-control-chars --color=auto'

	c. 修改inputrc文件，修改如下:

		# disable/enable 8bit input
		set meta-flag on
		set input-meta on
		set output-meta on
		set convert-meta off

1. 设置git commit 编辑器  
	运行命令`git config --global core.editor notepad.exe`,即可设置记事本为编辑器.

1. Bug  
	1. `<!-- more -->`后面接代码区块存在一些问题，具体原因不清楚。
