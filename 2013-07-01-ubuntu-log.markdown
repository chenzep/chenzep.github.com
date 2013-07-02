---
layout: post
title: "Git 源代码 编译"
date: 2013-07-01 09:17
comments: true
categories: Git

description: "Git 代码编译" 
keywords:  git 编译
---

1. 下载源代码

		git clone https://github.com/git/git.git

2.	查看到指定版本

		git tag

3.  切换到指定版本
		
		git checkout -b v1.7.11
4.	编译
		
		make prefix=/usr/local all
		make prefix=/usr/local install
	还有一种编译方法，我一般不用

		autoconf
		./configure
		make 
		sudo make install
5. 验证  
	编译完成之后，重新打开终端，输入**git --version**验证即可。
   
1. CentOS6.4 64Bit  
	出现了expat.h: No such file or direcotry的错误，运行yum install expat-devel即可。
	
1. Unbuntu10.04 64Bit    
	* 缺少**openssl/ssl.h**    
	  1.添加数据源到**/etc/apt/source.list**文件中

			deb http://archive.canonical.com/ lucid parter
		   2.运行

			sudo update
			sudo apt-get libssl-dev
	* 缺少**curl/curl.h**  
	1.更新数据源

			deb http://mirrors.sohu.com/ubuntu/ lucid main restricted universe multiverse
			deb http://mirrors.sohu.com/ubuntu/ lucid-security main restricted universe multiverse
			deb http://mirrors.sohu.com/ubuntu/ lucid-updates main restricted universe multiverse
			deb http://mirrors.sohu.com/ubuntu/ lucid-proposed main restricted universe multiverse
			deb http://mirrors.sohu.com/ubuntu/ lucid-backports main restricted universe multiverse
			deb-src http://mirrors.sohu.com/ubuntu/ lucid main restricted universe multiverse
			deb-src http://mirrors.sohu.com/ubuntu/ lucid-security main restricted universe multiverse
			deb-src http://mirrors.sohu.com/ubuntu/ lucid-updates main restricted universe multiverse
			deb-src http://mirrors.sohu.com/ubuntu/ lucid-proposed main restricted universe multiverse
			deb-src http://mirrors.sohu.com/ubuntu/ lucid-backports main restricted universe multiverse 
		
	2.运行:

			sudo update
			sudo apt-get libcurl4-gnutls-dev
			
	* 	缺少**expat.h**  
		运行

			sudo apt-get install libexpat1-dev
			
	* **/bin/sh: msgfmt: command not found**  
	  运行

			apt-get install gettext
		
	
