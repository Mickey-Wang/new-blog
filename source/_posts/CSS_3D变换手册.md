---
title: CSS 3D变换手册
author: Mikey
date: 2018-11-30
tags:
 - CSS
categories:
 - 技术博客
thumbnail: images/tech/cube.jpg
---

> 编者按：本手册是对 [Intro to CSS 3D transforms](https://3dtransforms.desandro.com/) 的译制。本文为手册的介绍部分，[完整手册请移步此](https://css3d.mikey.wang/)。

## 简介

随着CSS变换（CSS transforms）的推广，元素可以实现移动、旋转、倾斜、压缩和拉伸等效果，赋予了Web设计师追赶上印刷设计者，并且最终超越他们的能力，同时探索一个全新的图形领域。

3D图形渲染在Web领域已经有几年历史了。Flash是第一个出现的技术，之后随着`<canvas>`和WebGL的发展出现了[Three.js](https://threejs.org/)这样的库，如今，Web VR和AR又近在咫尺。虽然这些解决方案在塑造可探索的3D应用环境上特别出色，但是他们对于Web呈现的主要形式--界面来讲，着实有些过分。而对于CSS 3D变换，前端工程师可以通过增加一个新的空间维度，更好的增强改善对于传统网站的设计。

## 应用场景

在我们深入探讨3D之前，我们应该为我们的用户问一个问题，对于3D这个特性，可以给他们带来什么好处？

事实上，CSS是作为样式文件而存在的，如今已经发展成为一个与应用不可分割的部分。然而，CSS对于3D建模并不是最理想的技术，所以，3D变换应该被看作为一个**附加特性**，是与媒体查询、渐变、过渡类似的现代CSS新特性。网站中优秀的3D效果应该是渐进增强式的设计，某个取代某个界面功能。我们有很多场景去设计一些3D变换，来实现各个界面之间的过渡效果。

比如像早期的IOS天气应用。这个应用具有两个视图：一个天气详情视图和一个选项设置视图。这两个视图之间的切换采用的是3D翻页过渡效果，这样可以让用户明确的知道这个界面包含两个视图，同时也只有两个视图，分别在面板的正反两面。

![](/assets/weather-app-transition.jpg)

再来看轮播循环的例子。你如何让轮播的内容循环播放？答案是利用3D变换来排列每幅内容。让每幅内容在3D空间内依次排列为一个圆圈，这样再来实现循环播放便轻而易举了。

3D变换不仅仅只是在视觉上锦上添花，而且我们可以用它来解决实际的界面交互难题，使我们的应用更加易于使用。

## 可支持环境

[CSS 3D变换](https://www.w3.org/TR/css-transforms-1/)这个模块是在[2009年被首次提出的](https://www.w3.org/TR/2009/WD-css3-3d-transforms-20090320/)。它出自苹果公司之手，所以Safari也是第一个支持CSS 3D变换的浏览器。此后，所有的现代浏览器，包括Chrome，Firefox，IE和Edge都对其陆续提供支持。你可以在[caniuse.com](https://caniuse.com/#feat=transforms3d)查看各种浏览器对其支持的最新情况。

截止2018年，不带前缀的`transform`CSS属性已经被98%的浏览器支持。而带前缀的`-webkit-transform`属性可以向下兼容一些老浏览器，更好的得到99%的浏览器支持。

这里有一点需要注意。[IE 11不支持](http://msdn.microsoft.com/en-us/library/ie/hh673529%28v=vs.85%29.aspx#the_ms_transform_style_property)`transform-style: preserve-3d`（我们将会在后续篇幅介绍这个属性）这意味着IE11仍然可以对单独元素使用3D变换，但是其对于嵌套在文档内构建对象的子元素是无法处理的。

接下来我们一起来看编码实现。





