---
title: 由setTimeout深入JavaScript执行环境的异步机制
author: Mikey
date: 2017-05-23
tags:
 - JavaScript
categories:
 - 技术博客
---

## 问题背景

在一次开发任务中，需要实现如下一个饼状图动画，基于canvas进行绘图，但由于对于JS运行环境中异步机制的不了解，所以遇到了一个棘手的问题，始终无法解决，之后在与同事交流之后才恍然大悟。问题的根节在于经典的JS定时器异步问题，所以在解决问题之后，又通过了大量的资料阅读扩展和一段时间的实战总结，现在对JS运行环境中异步机制做一个较为深入的分析。

<center>![setTimeout.gif-55.9kB][3]</center>

上图中为最终想要实现的效果，使得各扇形部分可以同时画出并闭合圆形。[点击此处查看代码清单][4]。之前遇到的问题是没有将myLoop作为一个函数抽离出来，而将其中的所有逻辑，包括定时器都写在了for循环中，这样虽然扇形角度、哨兵变量等的计算均正确，但圆形始终无法闭合，很是郁闷。**这里我只是想借此问题来引入JS运行环境中对于异步机制理解的重要性**，大可不必关心canvas画图的实现过程，让大家明白对异步的理解会牵扯到业务逻辑执行的准确性，并非只是用于浮于纸面的面试题之上。至于为什么将定时器的逻辑放在一个函数中就执行正常，而直接写入for循环就无法达到预期，看过下文的详细分析后，这个问题便会迎刃而解。


----------


## 深入异步

关于异步的深入，这里基于现有的知识水平做尽可能详尽准确的分析。大家可以从一篇博客进一步了解牛人之间对于异步理解的争论。一位是技术博客红人阮一峰老师，一位是国内Node技术的开山鼻祖朴灵老师，都是我持续关注的两位偶像。事情发生的比较早了，这里只给出一个[文章链接][5]，其中在阮老师的博文中附带了大量朴灵老师的批注，读过之后定会受益匪浅，也会激发出你对技术外的一些思考。

### 同步与异步

首先来说明同步与异步两个概念。
    
```JavaScript
f1()
f2()
```

对于JavaScript语言的执行方式，执行环境会支持两种模式，一种是同步执行，一种是异步执行。如上面两个方法，同步执行就是调用f1之后，等待返回结果，再执行f2。异步是调用f1后，通过一系列其他的操作才可以得到预期的结果，比如网络IO、磁盘IO等，在线程执行这些其他操作的同时，程序还可以往下执行，继续调用f2，不用等待f1的结果返回再执行f2。

我们知道，大部分的脚本和编程语言都是同步编程，开发者对于同步编程的执行逻辑也比较容易理解。那么为什么对于JS的执行要经常用到异步编程，这应该要追溯到最初JS适用的宿主环境--浏览器。

由于用于浏览器，所以操作DOM的JS只能使用单线程，否则无法保证DOM操作的安全性（比如一个线程将另一个线程正在使用的某个DOM删掉）。又因为使用单线程，同步执行代码的话，如果遇到耗时较长的操作，那么浏览器将会长时间失去响应，用户体验及其不好。但如果将耗时较长的任务，比如ajax请求异步执行，那么客户端的渲染便不会受到耗时任务的阻塞。

对于服务器端，JS异步执行更为重要，因为执行环境是单线程的，如果同步执行所有并发请求，那么对于客户端的响应将会极其迟钝，服务器性能急剧下降，这时必须使用异步模式来处理大量并发请求，不像Java、PHP等语言是通过多线程来解决并发问题。这点在现在高并发司空见惯的网络环境中，反而成为了JS的优势，使得Node在短时间内进入主流视野，成为DIRT应用的最佳解决方案。

### 实现异步的机制

在说实现异步的机制之前，首先需要搞清楚两个概念，分别是JavaScript的**执行引擎**和**执行环境**。我们常说Google的V8虚拟机便是JavaScript的执行引擎，除此之外Safari的JavaScript Core、FireFox的SpiderMonckey都属于Engine。而上述的浏览器和Node等便属于JavaScript的执行环境，是Runtime。前者Engine是去实现ECMAScript标准，后者Runtime是去实现异步的具体机制。所以我们今天讲的JS异步机制都是在说**JS执行环境的异步机制**，与V8这样的执行引擎并无关系，主要是由各大浏览器厂商去做实现。

关于实现异步的方式，有我们接下来要详细介绍的Event Loop，还有轮询、事件等。所谓轮询，就是你在收银台付款之后，不停的问服务员你的饭菜做好了吗。所谓事件，就是你在付款之后，不用不停的问服务员，服务员在做好饭菜之后会主动告诉你。而大部分的执行环境都是通过Event Loop去实现异步机制，所以下面重点来讲解Event Loop。

#### Event Loop

Event Loop的实现逻辑如下图。每当程序启动后，内存会被分为堆（heap）和栈（stack）两部分，其中栈中便是主线程的执行逻辑所需内存，我们根据这块内存的特殊作用，抽象的将其叫做执行栈。在栈中的代码会调用各种WebAPI，比如对DOM的操作，ajax请求，创建定时器等。这些操作会产生一些事件，而事件又会关联相应的handle（也就是注册时的callback），将需要执行的handle按照队列的结构放入callback queue（event queue）中。当执行栈中的代码执行完毕后，主线程会读取callback queue，依次执行其中的回调函数，然后进入下一轮的事件循环，执行清空新产生的事件回调函数。由此可见，**在执行栈中的代码总是在callback queue之前执行**。

<center>![bg2014100802.png-22.4kB][6]</center>
*图片转引自Philip Roberts的演讲[《Help, I'm stuck in an event-loop》][7]*

setTimeout()和setInterval()两个定时器中回调的执行逻辑便是典型的Event Loop机制。相似的，程序在跑完执行栈中的代码后，事件循环会不停的检查系统时间是否到达预设的时间点，每当到达预设的时间点时，就会产生一个timeout事件，并将其放入callback queue，等待下轮Event loop执行。但在实际应用中，有可能执行栈中的代码耗时过长，这样在执行完执行栈中的代码后，再去执行callback queue中由setTimeout()产生的回调时就不能保证在预期的时间点执行，所以JS中的定时器并不总能保证其精准性。而在详细了解其特性原理后，我们可以在编程应用层面做一些优化，尽量使定时器中回调函数的执行时间点与我们预期保持一致。由于setTimeout()与setInterval()在本质上是一致的，所以在下面的实例分析一节中我们将会以setTimeout()来做关于异步机制的分析。

### 异步编程

关于异步编程我的理解是，在JS执行环境所提供的异步机制之上，在应用编码层面上实现整体流程控制的异步风格。具体地，我们可以用类似setTimeout()中的回调函数的形式进行异步编程，或者用类似事件驱动的发布/订阅模式，或者用ES6为我们提供的异步编程的统一接口Promise实现，再或者可以尝试最新最酷的ES7中Async/Await方案，还有一些像Node社区提供的异步流控库Step等。这里只是为大家明确异步编程这个概念范畴，具体用法不再深入。

----------

## 实例分析

这一节中我将会举出多例来分析，请大家结合上述理论细细体会JS中的同步与异步。首先我们从一个经典的JS异步面试题开始，然后逐渐深入。

```JavaScript
for (var i = 0; i < 5; i++) {
    setTimeout(function() {
        console.log(new Date, i);
    }, 1000);
}
 
console.log(new Date, i);
```
上述代码片段的运行结果应该是，先立即输出一个5，然后在1秒以后同时输出五个5。程序开始执行后，首先执行执行栈中的同步代码，几乎同时创建了5个定时器，然后继续执行第7行的同步代码。这样，首先在控制台输出一个5，然后在1s以后，5个定时器同时产生5个timeout事件放入callback queue，Event loop依次执行队列中的回调函数，这里因为闭包的特性，每一个定时器的回调都与其定义上下文，for循环中的i变量做了绑定，而i的值已变为5，所以同时输出五个5。

如果现在提出一个新需求，要求程序运行后，先立即输出一个5，然后在1s以后同时输出0,1,2,3,4，如何改造上述代码？

```JavaScript
//方法一
for (var i = 0; i < 5; i++) {
    (function(j) {  
        setTimeout(function() {
            console.log(new Date, j);
        }, 1000);
    })(i);
}
 
console.log(new Date, i);

//方法二
function output (i) {
    setTimeout(function() {
        console.log(new Date, i);
    }, 1000);
};
 
for (var i = 0; i < 5; i++) {
    output(i);  
}
 
console.log(new Date, i);
```

上面给出的两种方法其实都是一种思路，都是利用JS中，函数作用域作为一个独立的作用域，来保存一个局部的上下文环境，并通过闭包的特性使其与setTimeout中的回调函数做绑定。只不过第一种方法是利用IIFE来实现，第二种方法是通过定义一个函数，再来逐个调用实现。看到这里，应该想到对于篇首问题背景一节中所提到的问题便与此处如出一辙。

接下来我们进一步深入，提出一个新的需求。如何在代码执行时，立即输出 0，之后每隔1s依次输出 1,2,3,4，循环结束后在大概第5秒的时候输出5？

因为前边每隔1s输出的0,1,2,3,4是五个定时器输出的，也就是五个异步操作，那么我们是不是可以把这次的需求抽象为：在一系列异步操作完成（每次循环都产生了 1 个异步操作）之后，再做其他的事情。现在熟悉ES6的同学应该想到了Promise。

```JavaScript
const tasks = []; // 这里存放异步操作的 Promise
const output = (i) => new Promise((resolve) => {
    setTimeout(() => {
        console.log(new Date, i);
        resolve();
    }, 1000 * i);
});
 
// 生成全部的异步操作
for (var i = 0; i < 5; i++) {
    tasks.push(output(i));
}
 
// 异步操作完成之后，输出最后的 i
Promise.all(tasks).then(() => {
    setTimeout(() => {
        console.log(new Date, i);
    }, 1000);
});
```

如果你熟悉ES7中的Async/Await，那么也可以尝试用这种方案解决。

```JavaScript
// 模拟其他语言中的 sleep，实际上可以是任何异步操作
const sleep = (timeountMS) => new Promise((resolve) => {
    setTimeout(resolve, timeountMS);
});
 
(async () => {  // 声明即执行的 async 函数表达式
    for (var i = 0; i < 5; i++) {
        await sleep(1000);
        console.log(new Date, i);
    }
 
    await sleep(1000);
    console.log(new Date, i);
})();
```

这里需要着重注意的是浏览器对Async/Await标准的支持，如果你的浏览器不在以下所支持版本当中，那么可以升级浏览器或使用babel转译处理。

![此处输入图片的描述][8]

能把上边这一系列的实例理解到位，相信对JS中异步的这个概念会一些新的体会。下面这个实例会更加细化的考察一下异步代码中回调的执行时机。

```JavaScript
let a = new Promise(
  function(resolve, reject) {
    console.log(1)
    setTimeout(() => console.log(2), 0)
    console.log(3)
    console.log(4)
    resolve(true)
  }
)
a.then(v => {
  console.log(8)
})
 
let b = new Promise(
  function() {
    console.log(5)
    setTimeout(() => console.log(6), 0)
  }
)
 
console.log(7)
```
这里首先来明确一点，Promise是ES6中为异步编程所提供的一套API标准，其本身是同步的。所以我们在new一个Promise对象的时候，其所执行的构造器中的逻辑是同步的。由此得知，上述代码片段先从上到下依次执行同步代码，输出1,3,4,5,7。然后是先执行then中的异步代码还是先执行setTimeout中的回调代码？这里需要记住前者要比后者先进入执行栈执行，所以后边输出8,2,6。**这是因为立即resolved的Promise是在本轮事件循环的末尾执行，类似于node中的process.nextTick方法，它可以在当前"执行栈"的尾部，下一次Event Loop（主线程读取"任务队列"）之前，触发回调函数。setTimeout(fn, 0)则是在当前"任务队列"的尾部添加事件，也就是说，它指定的任务总是在下一轮次Event Loop时执行，这与node中的setImmediate方法很像。**

最后我们来说一个关于setInterval优化的例子。我们知道setTimeout中的回调触发是不准确的，主要原因是由于在需要执行回调时，可能执行栈中的代码还没有执行完，无法将CPU资源及时的调度给callback queue中的回调执行。而setInterval也会存在一些问题，比如时间间隔可能会跳过，
时间间隔可能小于定时器设定的时间。发生这类情况其实也是由于其他的程序占用长时间的CPU时间片引起，以下面代码片段为例：

```JavaScript
function click() { 
    // code block1... 
    setInterval(function() { 
        // process ... 
    }, 200); 
    // code block2 ...
}
```

如果process中的代码执行时间过长，占用了超过400ms，那么此时JS执行环境就会跳过中间一次时间间隔，因为callback queue中只允许有一份process代码存在，所以也会产生触发时机不精准的情况。

为了避免这种情况的出现，我们可以利用递归的方式进行优化处理，以下提供两种写法，但是建议使用第一种写法。因为第二种写法中，在严格模式下，第5版 ECMAScript (ES5) 禁止使用 arguments.callee()。当一个函数必须调用自身的时候, 避免使用 arguments.callee(), 通过要么给函数表达式一个名字,要么使用一个函数声明[参见MDN解释][9]

```JavaScript
    // 写法一
    setTimeout(function bar (){ 
        // processing
        foo = setTimeout(bar, 1000); 
    }, 1000);
    
    // 写法二
    setTimeout(function(){ 
        // processing 
        foo = setTimeout(arguments.callee, interval); 
    }, interval);
    
    clearTimeout(foo) // 停止循环
```







  [1]: http://mickeywang.com
  [2]: http://weibo.com/MickeyLaughing
  [3]: /images/tech/setTimeout.gif
  [4]: https://www.zybuluo.com/MickeyWang/note/772019
  [5]: http://www.360doc.com/document/14/1011/13/15077656_416048738.shtml
  [6]: /images/tech/async-2.png
  [7]: http://vimeo.com/96425312
  [8]: /images/tech/async-1.JPG
  [9]: https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/arguments/callee
