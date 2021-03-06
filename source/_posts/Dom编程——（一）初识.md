---
title: Dom编程——（一）初识
author: Mikey
date: 2016-10-03
tags:
 - JavaScript
categories:
 - 技术博客
---

## 概述 ##
本篇博客是**JavaScript Dom编程系列**的第一篇，既然是第一篇，那么有必要先陈述一下撰写的初衷与起源。我在实际的学习和工作过程中，发现大多数人对于DOM这个概念可以说是既熟悉又陌生。熟悉是因为大家在学习和工作的交流过程中经常会提及DOM，陌生是因为当我们想去认真正的表述对其理解时又不太确切，常常把`JS`、`DOM`甚至`JQuery`这样的JS库混为一谈，对其理解不够深入。认知层面上的理解不充分，自然就会导致应用层面的混乱无序，表现在代码上，就是可读性差，不够标准，不够规范。这些同样也是我所面临的问题，所以自修了*《JavaScript DOM编程艺术(第2版)》*这本书，这本书的其中一位作者**Jeremy Keith**是Web标准的制定者之一，所以书中所阐释的内容更加契合主流标准的设计思路，而标准是具有前瞻性和权威性的，这也是我选择此书作为学习DOM的原因。本系列博客其实就是这样的一次学习笔记，主要以此书的精炼提取来组织内容，在保证知识准确性的基础上，加入了我自己的理解与扩展，希望通过学习与共享的这种方式与各位朋友一起成长。

在陈述完写作目的之后，再对博客的内容侧重做一个简单的概述与解释。首先对于本博客的读者而言，应该了解JavaScript这门语言的基本语法，最好是有过一段时间的编码实践，这并不是说博文的内容有多么深厚难懂，而是为了帮助读者更加便于理解其中之“道”。什么“道”呢？我们知道，一般的初级coder都会纠结于语言的语法结构，局限在“术”的层面上去解决问题，很少去考虑语法之外更加深入的问题，所以就不会理解一些诸如编码原则、编码习惯、编码思路的“道”的层面所涉及到的问题。本博文的内容其实是浅入深出的，只要有些许JS语法基础的读者便能够读懂，文章整体上不会拘泥于语法细节，或许你对其中的一部分早就熟悉了，但真正的目的是让你**理解DOM脚本编程技术背后的思路和原则**。平稳退化、渐进增强、以用户为中心的设计对任何Web开发工作都非常重要。这些思路贯穿在本系列博文中的所有代码示例中。

下面就开始我们的DOM学习。

----------


## JavaScript与DOM ##
本节将为大家引出DOM的准确概念，并说明JavaScript与DOM的具体关系。

### 一、W3C标准
要讲DOM，那么就要从一个Web开发者中无人不知的组织————**W3C（万维网联盟）**说起。我们在开发交流过程中，常常会提及W3C标准，那么W3C标准具体都是些什么呢？

 1. 结构化的标准语言（HTML和XML等）；
 2. 表现标准语言（CSS层叠样式表）；
 3. 行为标准语言（DOM和ECMAScript）；
    
也就是说W3C这个组织是一个标准的制定者，而DOM便是其所制定标准中的一项，我们经常使用的浏览器便扮演了标准实现者这样的角色。目前主流的现代浏览器都不同程度地实现了HTML、CSS、DOM这些标准，从标准的分类上来讲，也体现了W3C所提倡的Web文档的内容、表现与行为相分离的设计思想，而DOM便是负责给文档增加交互能力的。

为什么要设计标准？Web的发展可以说是混沌的，伴随了近十年的浏览器大战，最终在各方的拉锯博弈之后，为了实现利益最大化，存活到最后的浏览器厂商均选择了对标准妥协，这也使得Web开发从一种混乱无序的状态，发展成了一门需要严格训练才能从事的正规职业。而这些大量的标准约束，对于开发人员是最为有利的，统一了标准后，Web开发者便不再因为浏览器的不同而写大量的浏览器嗅探代码和一些低效的分支语句，省却了一个功能多套脚本这样的重复劳动。

### 二、什么是DOM？

> 简单地说，DOM是一套对文档的内容进行抽象和概念化的方法。

`文档对象模型`（Document Object Model，简称`DOM`）,就像现实世界中的“世界对象模型”，大家对于一些抽象的概念有着基本的共识，所以人们才能用非常简单的话表述出复杂的含义并得到对方的理解。比如说，你对别人讲“左边数第三棵树”，那么你们之间对于“左边”和“第三”这样的抽象概念一定是理解一致的，这样才可以进行顺畅的交流。而文档对象模型针对的是一份文档，如果各个浏览器对于操控网页的具体方法均不一致，那么相同的一份代码在不同的浏览器中便会冲突不断。

W3C所定义的DOM会更加广泛：

> 一个与系统平台和编程语言无关的**接口**，程序和脚本可以通过这个接口动态地访问和修改文档的内容、结构和样式。

也就是说标准化的DOM可以让**任何一种程序设计语言**对使用**任何一种标记语言**编写出来的**任何一份文档**进行操控。我们看到，W3C把DOM定义为了一套接口，所以DOM就是一种**API**（应用编程接口）。简单地说，API就是一组已经得到有关各方共同认可的基本约定。在现实世界中，相当于API的例子有很多，比如摩斯电码、国际时区、化学元素周期表，这些都是不同学科领域中的标准，为的就是使得人们能够更方便地交流与和合作。
  
在软件编程领域中，虽然存在着多种不同的语言，但是很多任务是相同或相似的。这也是人们需要API的原因。为了更方便的对接。一旦掌握了某个标准，就可以把它应用在许多不同的环境中。虽然语法会因为使用的程序设计语言而有所变化，但这些约定确实保持不变的。所以说，按照DOM标准，我们可以通过JavaScript去操控HTML文档，也可以通过诸如PHP或Python之类的程序设计语言去解析XML文档等等。

至此，可以为各位读者总结一下**DOM与JavaScript之间的关系**了。在Web技术中，把HTML/XHTML和CSS真正黏合起来的东西是DOM，也就是说使用W3C DOM来处理文档和样式表。如果说“DOM脚本程序设计”，则涵盖了使用任何一种支持DOM API的程序设计语言去处理任何一种标记文档的情况。只不过具体到Web文档，JavaScript的无所不在使它成为了DOM脚本程序设计的最佳选择，这也是为什么我们博文的题目要加入JavaScript关键字的原因。

----------

## 从基础API开始 ##
之前的小结是着重为大家介绍了DOM的抽象概念及其解释，如果是初次接触DOM的朋友，可能现在对于DOM的理解还只是基于文字上的轮廓和印象，不够具体深入，所以本节就用一些基本的DOM方法来为各位强化一下对于DOM的认识，同时也是为下一节的实践案例做一定的技术准备。

### 一、DOM中的“O”
我们知道DOM（Document **Object** Model)是指文档对象模型，这里的这个Object，也就是对象，需要我们再仔细研究一下。首先来解释一下js中对象的概念：

对象
:   就是由一些属性和方法组合在一起而构成的一个数据实体。

> JavaScript中的对象可以分为三种类型： 
> 1、用户定义对象(user-defined object)：由程序员自行创建的对象
> 2、内建对象(native object)：内建在JavaScript语言里的对象，如Array、Math和Date等
> 3、宿主对象(host object)：由浏览器提供的对象，如window、document等

其中第三种宿主对象是我们讨论的重点，因为JavaScript是一种解释型语言，所以必须依赖运行于一个宿主环境中，那么客户端js一般的宿主环境就是指浏览器。window对象对应着浏览器窗口本身，这个对象的属性和方法通常统称为BOM(浏览器对象模型)。我们不需要与BOM打太多的交道，而是把注意力集中在浏览器窗口中的网页内容上。document对象的主要功能就是处理网页内容。所以在后续的内容中，我们几乎只讨论document对象的属性和方法。

下面便是我们要具体理解DOM的一个关键了，**DOM把一份文档表示为一棵树**，这棵树可以理解成一棵家谱树，一般用parent（父）、child（子）、sibling（兄弟）等记号来表明家族成员之间的关系。那么，组成家谱树的基本单元是家族成员，组成DOM树的基本单元则是节点（node），也就是说，以后我们看到一份Web文档时，就把它当做是一棵节点树。那么什么是节点呢？

### 二、节点

DOM中有许多不同类型的节点，我们先来看三种，它们分别是元素节点、文本节点和属性节点。为了方便表述，我们给出一份简单的Web文档`代码清单A`：
```html
<!DOCTYPE html>
<html lang="en">
	<head>
		<meta charset="UTF-8">
		<title>Shopping list</title>
	</head>
	<body>
		<h1>What to buy</h1>
		<p title="a gentle reminder">Don't forget to buy this stuff.</p>
		<ul id="purchases">
			<li>A tin of beans</li>
			<li class="sale">Cheese</li>
			<li class="sale important">Milk</li>
		</ul>
		<script></script>
	</body>
</html>
```

 1. **元素节点**事实上就是每个标签，标签的名字就是元素的名字。比如说整个节点树中的根元素<html>元素。
 2. **文本节点**就是被包含在元素节点内部的文本内容，也是节点的一个类型。比如<p>元素中包含的文本“Don't forget to buy this stuff.”就是一个文本节点。
 3. **属性节点**是用来对元素做出更具体的描述，总是被包含在元素节点之中的。比如<p>元素中包含的title属性，就是一个属性节点。

### 三、获取元素

介绍过了DOM中节点的概念后，我们开始DOM API的学习，首先介绍的是获取元素的三个DOM方法。

#### 1、getElementById

> document.getElementById('id');

这个方法返回一个与那个有着给定id属性值的元素节点对应的对象。也就是说，这个调用将返回一个对象，这个对象对应着document对象里的一个独一无二的元素，因为id这个属性值在全局应该是具有唯一性的。还以上面的代码为例，`document.getElementById('purchases')`，就获取到了id为purchases的`<ul>`这个节点对象;

#### 2、getElementByTagName

> document.getElementsByTagName('tag');

这个方法的参数是标签的名称，返回值是一个对象数组，每个对象分别对应着文档里给定标签的一个元素。也就是说返回的这个数组里，放的都是对应的元素节点对象。在代码清单A中的`<script>`标签里加入以下代码：

```javaScript
var items = document.getElementsByTagName('li');
for(var i = 0; i < items.length; i++){
	console.log(typeof items[i]);
}
```

在控制台中我们会看到三个“object”。

getElementByTagName允许把一个通配符作为它的参数，用于获得某个对象中一共有多少个元素节点。注意通配符也需要用引号括起来，否则js解释器会将*号作为乘运算符处理。比如说，想要知道一份文档中一共有多少个元素，就可以用`document.getElementsByTagName('*').length`;

#### 3、getElementsByClassName

> document.getElementsByClassName('class');

这个方法与上述getElementsByTagName方法较为相似，返回值都是一个对象数组，只是这个函数的参数是HTML中class属性的类名，所以数组中的对象是与参数类名相同的元素节点。参数可以传入多个类名，中间用空格分开，表示**同时**含有多个类名的元素节点将被筛选出来。还是以代码清单A为例，`document.getElementsByClassName('important sale');`，这样就获得了包含第13行所在元素的数组。当有多个类名时，类名之间的顺序变化是不影响结果的，也就是说执行`document.getElementsByClassName('sale important');`，与执行上句结果相同。当一个元素的class属性中包括但不限于指定类名时，该元素也会被选中，如果给代码清单A再添加一个元素，`<li class="sale important xxx">example</li>`，那么此元素也会被筛选出来。

这个方法在实践中非常有用，但只有较新的浏览器才会支持它，为了可靠地使用该方法，我们来自己模拟实现一个与此功能相类似的方法，函数名也是getElementsByClassName：
```javaScript
/**
 * 根据类名获取相应元素节点，此方法只支持传入单个类名
 * @param  {Object} node      表示DOM树上的搜索起点，是一个元素节点对象
 * @param  {String} classname 要搜索的类名（单个类名）
 * @return {Array}           包含指定类名的元素节点对象的结果数组
 */
function getElementsByClassName(node, classname) {
	//判断传入的节点上有没有getElementsByClassName这个方法
	if (node.getElementsByClassName) {
		//浏览器支持该方法，使用现有方法
		return node.getElementsByClassName(classname);
	} else {
		var results = new Array();
		var elems = node.getElementsByTagName("*");
		//遍历以node为起点的所有标签，找到带有相应类名的元素，并添加到结果集中
		for(var i = 0; i < elems.length; i++){
			if (eles[i].className.indexOf(classname) != -1) {
				results[results.length] = elems[i];
			};
		}
		return results;
	}
}
``` 
类似此类支持性的问题，我们在之后的讲解中还会遇到，会对如何解决这类问题进行详细探讨。

> 基于上述讲解，我们在这里为大家做一个小的总结来加深理解。**对于DOM来讲，一份文档就是一颗节点树，而文档中的每个元素节点都是一个对象。**不仅如此，这些对象中的每一个还天生具有一系列非常有用的方法，这要归功于DOM。利用这些预先定义好的方法，我们不仅可以检索出文档里任何一个对象的信息，甚至还可以改变元素的属性。下面我们就来一起学习一些关于获取和设置属性的DOM方法。

### 四、获取和设置属性

获取到特定的元素以后，我们就可以设法获取它的各个属性以及动态更改属性节点的值了。

#### 1、getAttribute

> object.getAttribute('attribute');

与此前我们介绍过的方法不同，getAttribute方法不属于document对象，所以不能通过document对象来调用。它只能通过元素节点对象调用来获得该节点的属性值。如果该元素中没有指定属性，那么返回null。

```javaScript
var paras = document.getElementsByTagName('p');
for (var i = paras.length - 1; i >= 0; i--) {
	var title_text = paras[i].getAttribute('title');
	if(title_text) console.log(title_text);
}
```
因为代码清单A中只有一个p标签含有title属性，所以在控制台中我们会看到打印出了`a gentle reminder`文本。

#### 2、setAttribute

> object.setAttribute('attribute', value);

与getAttribute相同的是，setAttribute也只能用于元素节点。如果该元素节点中存在所设置的属性，那么这个属性值将会被覆盖。

```javaScript
var shopping = document.getElementById('purchases');
shopping.setAttribute("title", "a list of goods");
```

因为id为purchases的元素是没有title这个属性的，所以setAttribute做了两步动作，先创建了title属性，然后设置了它的值为“a list of goods”。

> 这里有一个非常值得关注的细节：通过setAttitude对文档做出修改后，在通过浏览器的view source（查看源代码）选项去查看文档的源代码时，看到的仍将是改变前的属性值，也就是说，setAttribute做出的修改不会反应在文档本身的源代码里。这种“表里不一”的现象源自DOM的工作模式：**先加载文档的静态内容，再动态刷新，动态刷新不影响文档的静态内容。这真是DOM的真正威力：对页面内容进行刷新缺不需要在浏览器里刷新页面。**


----------

## 小结

下面对`JavaScript Dom编程系列`的第一篇内容做一个小结。以上我们依次介绍了对于DOM的基本认识，DOM和javaScript之间的关系，以及一些DOM中的基础API。

 1. getElementById;
 2. getElementsByTagName;
 3. getElementsByClassName;
 4. getAttribute;
 5. setAttribute;

本篇内容偏重于理论，DOM还提供了许多其他的属性和方法，在接下来的博文中，我们将通过一个图片库的案例来进一步展示DOM的强大威力。
