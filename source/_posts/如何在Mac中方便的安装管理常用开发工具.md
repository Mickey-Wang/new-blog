---
title: 如何在Mac中方便的安装管理常用开发工具
author: Mikey
date: 2016-10-30
tags:
 - 工具与技巧
categories:
 - 技术博客
---

## 问题背景 ##

在用Mac做开发时，不免需要安装一些工具或者其他程式，比如Git、Node、Maven等等。在win下做开发时，这些工具往往需要一个个下载后，再安装运行的，而在Mac下，会有一个神奇的工具--`HomeBrew`，它可以帮我们安装管理一些常用的工具。如果你还是一个没有使用过HomeBrew的Mac新手，那么你就更加需要这篇博文了，因为，在使用Mac开发时，HomeBrew定将会是您居家旅行的必备工具<i class="icon-smile"></i>·····

## 解决方案 ##

先来简单介绍一下HomeBrew。HomeBrew是一个**包管理器（套件管理器）**，用于在Mac上安装一些OS X上没有的Unix工具。下面我们来详细讲解一下HomeBrew的安装与使用：

因为HomeBrew是依赖于Ruby的，所以我们需要先安装Ruby，而在Mac中安装Ruby的一种方法是直接安装Xcode，这是因为Xcode会帮你安装好 Unix 环境需要的开发包，其中就会包含Ruby（Mac的桌面操作系统内核就是Unix）。安装Xcode的方法这里不再赘述，直接从App Store中Get即可。

 1. 首先查看一下是否安装了Xcode，用命令xcode-select -p
 2. 查看一下Ruby的版本是否正常，使用ruby -v
 3. 在浏览器中进入HomeBrew的官网http://brew.sh/，在终端中粘贴网页中的脚本
    ![屏幕快照 2016-10-30 下午9.14.08.png-73.2kB][7]
 4. 等待执行完毕后，可以在终端中输入brew来查看HomeBrew的基本用法
    ![屏幕快照 2016-10-30 下午9.23.38.png-228.9kB][6]

HomeBrew安装完成后，使用brew install这个命令，就可以一次安装很多套件了，比如brew install git node maven mongodb，后边的程式可以写一个或者写多个，按需选取即可。HomeBrew将这些工具统统安装到了 /usr/local/Cellar 目录中，并在 /usr/local/bin 中创建符号链接。使用brew list命令就可以查看用HomeBrew安装了哪些套件了。更为详细的使用说明请参考在GitHub上的[官方文档资料][8]。


  [1]: http://mickeywang.com/
  [2]: http://weibo.com/MickeyLaughing
  [6]: images/tech/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-10-30%20%E4%B8%8B%E5%8D%889.23.38.png
  [7]: images/tech/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-10-30%20%E4%B8%8B%E5%8D%889.14.08.png
  [8]: https://github.com/Homebrew/brew/blob/master/docs/FAQ.md
