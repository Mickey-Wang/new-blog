---
title: Mac中使用反编译插件jadclipse
author: Mikey
date: 2016-8-27
tags:
 - 工具与技巧
categories:
 - 技术博客
---

## 问题背景 ##
对于刚刚入手Mac开发Java的同学，可能会发现Mac中打开`.jar`文件时会出现诸多问题，不像在win环境下只要正确安装jdk，就可以轻松打开jar包浏览其中的文件，但对于浏览jar包中的`.class`文件，Mac和windows都会遇到反编译的问题。所以在参考过网上的许多解决方案后，结合我自己的使用习惯，我选择使用Eclipse来打开jar包，并且通过安装jadclipse插件进行反编译浏览`.class`文件。这种方案既不涉及一些繁杂的下载和破解过程，同时在开发过程中使用也十分方便，缺点就是，即使只想打开一个jar包，那么也必须打开Eclipse，关联到Eclipse中任意一个项目的build Path中。不过，对于开发java的同学，相信Eclipse不是已经开启，就是在开启的路上······
## 解决方案 ##
对于下述用到的工具插件，可以通过每一步中的链接从源站获取。

**下面我来说明在Eclipse中如何安装插件jadclipse并使用（以Mac为例，windows环境类似）：**

 1. 下载jad在Eclipse中的插件，通过链接（http://sourceforge.net/projects/jadclipse/）从源站下载相应版本；
 2. 将net.sf.jadclipse_3.3.0.jar拷贝到Eclipse的plugins目录下；
 3. 重启Eclipse；
 4. 下载jad的可执行文件（http://varaneckas.com/jad/），**注意选择不同操作系统下的不同CPU平台**，在我提供的云盘原料中，一共有三个版本：
      - <i class="icon-linux"></i> linux-intel
      - <i class="icon-apple"></i> mac-intel
      - <i class="icon-windows"></i> win-intel
 5. 接下来，在Eclipse中设置jad的可执行文件路径以及生成的临时文件路径；![屏幕快照 2016-08-27 下午6.26.43.png-203.1kB][2]
 6. 最后设置File Association，也就是设置class文件的默认打开方式；![屏幕快照 2016-08-27 下午6.31.35.png-254kB][3]![屏幕快照 2016-08-27 下午6.31.50.png-248.7kB][4]

好了，现在在Eclipse中再打开`.class`文件就可以显示出可爱的源码了<i class="icon-smile"></i>
 


  [2]: images/tech/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-08-27%20%E4%B8%8B%E5%8D%886.26.43.png
  [3]: images/tech/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-08-27%20%E4%B8%8B%E5%8D%886.31.35.png
  [4]: images/tech/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-08-27%20%E4%B8%8B%E5%8D%886.31.50.png
