---
layout: post
title: "octopress_base"
date: 2012-12-27 11:38
comments: true
categories: Git
description: "在Github用Octopress建Blog" 
keywords: Github Octopress 
---

# 在Github用Octopress建Blog(基础篇) #

1. 安装railsinstaller.  
	点击[这里](http://railsinstaller.org/)到官方网站下载软件.railsinstaller包含了ruby,rails,bundle等一系列工具，具体可以看官方网站的说明。安装过程比较简单，就是不断的Next,除了安装路径，其他使用默认设置即可。

1. 配置rails环境  
	在安装railsinstaller的最后一步，有一个配置项，提示用户是否进入rails环境配置，此配置项默认情况下是选中的。如果进入了rails配置环境，用户需要输入自己的用户名和邮箱，输入完成之后，程序自动生成一个ssh_key,放在用户目录的.ssh子目录下。运行过程的相关信息如下：

		# Rails Environment Configuration.
		
		Your git configuration is incomplete.
		user.name and user.email are required for properly using git and services such
		as GitHub ( http://github.com/ ).
		
		 Please enter your name, for example mine is: Wayne E. Seguin
		name > xxxxxxx
		Setting user.name to xxxxxxx
		
		 Please enter your email address, for example mine is: wayneeseguin@gmail.com
		email > xxxxxxx@gmail.com
		Setting user.email to xxxxxxx@gmail.com
		
		---
		git:
		  user.name:  xxxxxxx
		  user.email: xxxxxxx@gmail.com
		  version:    git version 1.7.9.msysgit.0
		
		ruby:
		  bin:        D:/RailsInstaller/Ruby1.9.2/bin/ruby.exe
		  version:    ruby 1.9.3p125 (2012-02-16) [i386-mingw32]
		
		rails:
		  bin:        D:/RailsInstaller/Ruby1.9.2/bin/rails.bat
		  version:    Rails 3.2.1
		
		ssh:
		  public_key_location: C:\Users\xxxxxxx/.ssh/id_rsa.pub
		  public_key_contents: ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEA0u36KdY6zTS3RpLO1C6Gh
		tVr3ELzYRhGzjU2vcxXdGdAeLrLPfN7G6aUQ65qrW3pTwb8zcZrrEmdhx2/xV3mkAFiE5riB2ZGLw2se
		pRZdnmkiyULV33c51KYsltyO6F37RWtdbAQA2OjPNGuTL+DtxwmdDeObUjKJjkt8RxFtWqCJ0TqH+aMw
		r+MPycNxSamk1Ady5DySM5EH8YTbchD+GFU93VWRAFpydTy8ZAWhlx4PsBoWbONzrlhtqjovNsKRw5VH
		atk58hxsp1HPRR5XvF0LYd401QiwArDuw/m41rhrp8CyMzb0dTOQGxfl3ckNmcBovjSyaqbVYkY32Crm
		Q== xxxxxxx <xxxxxxx@gmail.com>
		
		C:\Sites>

	这里要说明几点:
	- C:\Users\xxxxxxx/.ssh/id_rsa.pub是ssh的公匙，对应的私匙是同样目录下的id_rsa文件.
	- 安装完成之后，如果要执行相关命令,我们可以通过`开始菜单-->railsinstaller-->git bash`来进入bash窗口,输入命令。
	- Windows路径的访问，比如`c:\Sites`,在bash下对应的是`/c/Sites`.

1. 配置Gems源  
	把默认的Gems源改成ruby.taobao.com

		C:\Sites>gem source -l
		*** CURRENT SOURCES ***
		
		http://rubygems.org/
		
		C:\Sites>gem source -r http://rubygems.org/
		http://rubygems.org/ removed from sources
		
		C:\Sites>gem source -a http://ruby.taobao.org
		http://ruby.taobao.org added to sources
		
		C:\Sites>gem source -l
		*** CURRENT SOURCES ***
		
		http://ruby.taobao.org
		
		C:\Sites>	

1. 创建github库
	- 创建github非常简单，需要注意的是库名最好是Username.github.com的形式。这样的话，我们就可以通过http://Username.github.com的URL访问BLOG了。   
		![](/images/git/github_new.png)  
		图片中的叹号表明我已经创建了此库，不能再次创建。  
	- 添加SSH Public_key  
		a. 点击右上角的`account setting`按钮,进入setting页面。接着点击页面中的ssh keys按钮，添加一个SSH key。然后把上面的id_rsa.pub文件中的内容拷贝到Key框中，如下图所示，接着点击`Add Key`按钮,按照提示输入github账户的密码，SSH Key加入成功。  
		![](/images/git/sshkey_add.png)


1. 下载Octopress源码  
	点击[这里](http://octopress.org/)到官方网站,点击[这里](https://github.com/imathis/octopress)下载相关源代码。下载好源代码后,运行git bash,进入源码根目录，后续的操作都在此环境下进行。

1. 下载Octopress的依赖包  
	* 修改根目录下Gemfile文件，将数据源改成source "http://ruby.taobao.org"
	* 运行bundle install 命令,然后等待依赖包更新完成(有些网上教程说用bundle update,最好不要这样。因为update使用最新的包，在ruby世界，最新的不一定是匹配的)。

1. 安装主题  
	运行rake install.此命令就是把主题拷贝到相应的位置，下面是运行的输出结果。

		## Copying classic theme into ./source and ./sass
		mkdir -p source
		cp -r .themes/classic/source/. source
		mkdir -p sass
		cp -r .themes/classic/sass/. sass
		mkdir -p source/_posts
		mkdir -p public
		
1. 用git管理_posts目录  
	在安装完主题后，系统会自动创建一个空目录source/_posts，以后创建的文章都会保存在这个目录内,我们需要对它用Git进行管理.  
	* 初始化版本库.
	>$ cd source/_posts/  
	>$ git init  
	>$ touch README.md  
	>$ git add *  
	>$ git commit -m "first commit"  
	
	* 备份到github
	>$ git remote add origin git@github.com:[Username]/[Repository].git  
	>$ git checkout -b md  
	>$ git push origin md	
		
1. 生成Blog文件  
	现在回到Octopress的根目录，运行下面命令  
	>$ rake generate
	
1. 本地测试  
	* 启动rake服务:
	>$ rake preview
	
    * 打开网页,输入localhost:4000,如果一切正常，你将会看到你的BLOG首页了。当然，首页除了框架，什么内容都没有.

1. Blog同步到github.  
	* 指定github库的URL,运行命令
	>$ rake setup_github_pages  

	 然后会提示你输入github的URL，此URL和上面的“git remote add origin git@github.com:[Username]/[Repository].git"是一样的。
  
		windows下可能会在My Octopress Page is coming soon之后出现hellip;不是内部命令之类的错误, 可以不用管.
		如果一定不想要出现这个错误可以修改myoctopress目录下的Rakefile, 搜My Octopress Page is coming soon,
		在…前加个(这个是Windows cmd的转义符), 如下
		system “echo ‘My Octopress Page is coming soon ^…’ > index.html” rake setup_github_pages
		命令最后出现Now you can deploy to xxxxxxx with rake deploy, 就表示成功.

	* 同步到github,运行命令
	>$ rake deploy

	* 如果一切正常，等上10分钟左右，根据你github库的URL地址输入对应的http地址，就会看到效果了。  
		a.如果你创建库的URL形式是git@github.com:[Username]/[Username].github.com.git,  
			 blog的http地址就是http://Username.github.com  
		b.如果库是非a形式的URL git@github.com:[Username]/[Repository].git  
			 blog的http地址就是http://Username.github.com/Repository
		c.绑定自己的域名，这个就不在讨论范围之内了。

1. 添加第一篇文章  
	* OctoPress的文章默认是用Markdown写的，具体教程点击[这里](http://wowubuntu.com/markdown/)  
	* 我是使用markdownpad工具来书写Marddown的，工具的下载点击[这里](http://markdownpad.com/)
	* 运行`rake new_post["Hello World"]`,会在source/_posts目录下生成一个markdown文件。  
	>Creating new post: source/_posts/2012-12-27-hello-world.markdown

	* 随便在markdown文件上写点东西，如果有中文，会导致错误，解决方法请看下面内容。  
	* 运行`rake generate`。
	* 如果生成成功， 接着运行`rake preview`,在浏览器上输入`localhost:4000`就可以看到你的第一篇文章了。

1. 加入Octopress对中文的支持  
	Octopress默认情况下是不支持中文的，如果你的Markdown文件中包含有中文字符，在`rake generate`阶段会导致生成错误，要解决这个问题，运行下面命令  
	>$ cd  
	>$ echo "export LC_ALL=en_US.UTF-8" > .bash_profile  
	>$ echo "export LANG=en_US.UTF-8" >> .bash_profile  
	
	完成之后，关闭并重进bash即可。	

1. 总结
	1. 执行rake new_post['title']来生成一个博文；
	2. 找对生成的markdown文件，编辑内容；
	3. 执行rake generate来生成文章；
	4. 执行rake preview在本地预览；
	5. 执行rake deploy发布到Github中。
	6. 执行下面命令将修改的源码推送到source分支：
	>git add .  
	>git commit -m “your message”  
	>git push origin source  

1. 参考文章  
	- [Blog = GitHub + Octopress](http://mrzhang.me/blog/blog-equals-github-plus-octopress.html)
	- [搭建一个免费的，无限流量的Blog----github Pages和Jekyll入门](http://www.ruanyifeng.com/blog/2012/08/blogging_with_jekyll.html)  
	- [Github Pages极简教程](http://yanping.me/cn/blog/2012/03/18/github-pages-step-by-step/#ruby)  
	- [Markdown 语法说明](http://wowubuntu.com/markdown/)  
	- [为什么Markdown+R有较大概率成为科技写作主流](http://www.yangzhiping.com/tech/r-markdown-knitr.html)  
	- [在Windows下使用jekyll如何避免出现中文字符集错误](http://yanping.me/cn/blog/2012/10/09/chinese-charset-problems-with-jekyll/)  
	- [How-to-octopress](http://jenwang.org/blog/2013/01/23/how-to-octopress/)
	- [Windows下搭建Octopress博客](http://www.cnblogs.com/oec2003/archive/2013/05/27/3100896.html)