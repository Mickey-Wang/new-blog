---
title: node实现的网站爬虫
author: Mikey
date: 2017-09-22
tags:
 - JavaScript
 - Node
categories:
 - 技术博客
---

> 本篇文章是node实现的网站爬虫这个项目的README.md，因为涉及一些项目开发过程中的问题和知识点的总结，所以在这里单独拿出分享一下。项目链接如下[<i class="icon-github-alt"></i>crawler][3]。


项目以抓取一个[博客网站](http://blog.sina.com.cn/u/1776757314)为例，并将数据保存在MySQL数据库中，可**定时更新**。

## 目录结构

+ **example** 目录：本实例最终的代码
+ **images** 目录：文章内容用到的图片
+ **snippets** 目录：文章内容用到的代码片段
+ **book.md** 文件：文章内容

## 项目启动命令

scripts中有两个命令：
+ crawler命令为爬取网站内容到数据库
+ server命令为带有定时功能并可展示数据（localhost:3000）的服务器程序指令

## 项目用到的相关模块

+ request：使用request模块来获取网页内容
+ cheerio：是jQuery Core的子集，实现了其中绝大部分的DOM操作API，适用于服务端
+ mysql：数据持久化到MySQL的驱动
+ async：一个使用比较广泛的JS异步流程控制模块
+ debug：用于打印调试信息，可通过设置环境变量来控制开发、测试、生产的信息输出
  + 可通过cross-env进行跨平台设置，cross-env DEBUG=blog:*
  + Windows系统在命令行下执行：set DEBUG=blog:*
  + Linux系统在命令行下执行：export DEBUG=blog:*
+ cron：使用[cron模块](https://www.npmjs.com/package/cron)来定时执行任务

## 开发过程中的难点或知识盲点

1. JS中[match](http://www.w3school.com.cn/jsref/jsref_match.asp)与[exec](http://www.w3school.com.cn/jsref/jsref_exec_regexp.asp)方法的区分  
2. npm中的cron模块的定时规则与Linux中的定时任务工具[crontab语法规则](http://crontab.org/)大致相同，只是cron多了一个秒单位。具体如下：f1 f2 f3 f4 f5 f6
   + f1表示秒钟，f2表示分钟，f3表示小时，f4表示一个月份中的第几日，f5表示月份，f6表示一个星期中的第几天。
   + 各部分的取值含义如下（以f1部分为例，其他部分类似）：
     + 当值为*时，表示每秒执行一次；
     + 当值为a-b时，表示从第a秒到第b秒这段时间内执行一次；
     + 当值为*/n时，表示每隔n秒执行一次；
     + 当值为a-b/n时，表示从第a秒到第b秒这段时间内每隔n秒执行一次。
3. 使用node内建模块child_process模块来执行文件
   + child_process模块可以通过spawn()和exec()两种方法来启动一个新进程
   + 使用spawn()启动子进程
    ```javascript
        var child_process = require('child_process');
        
        // spawn()是直接运行文件的，而在Windows系统下dir命令是cmd.exe的内置命令
        // 并不实际存在名为dir.exe的可执行文件，所以这里需要判断一下
        if(process.platform === 'win32'){
            var dir = child_process.spawn('cmd.exe', ['/s', '/c', 'dir', 'c:\\']);
        }else {
            var dir = child_process.spawn('dir', ['/']);
        }
        
        // 当子进程有输出时，自动将其输出到当前进程的标准输出流
        dir.stdout.pipe(process.stdout);
        dir.stderr.pipe(process.stderr);
        
        // 进程结束时触发close事件
        dir.on('close', function(code) {
          console.log('进程结束，代码=%d', code);
        })
    ```
    + 使用exec()启动子进程
    ```javascript
        var child_process = require('child_process');

        // 选项
        var options = {
           // 输出缓冲区的大小，默认是200KB，如果进程的输出超过这个值，会抛出异常，并结束该进程
           maxBuffer: 200*1024
        };
        
        var dir = child_process.exec('dir *',options, function(err, stdout, stderr) {
            if(err) throw err;
            console.log('stdout: ' + stdout);
            console.log('stderr: ' + stderr);
        });
    ```
    + spawn和exec方法的区别
      + spawn执行的命令必须是一个实际存在的可执行文件，而exec执行的命令则与在命令行下执行的命令一样；
      + exec可以在回调函数中一次性返回子进程在stdout和stderr中输出的内容，但调用两者都会返回一个ChildProcess实例，通过监听其stdout和stderr属性的data事件可以获取到进程的输出；
      + exec可以指定maxBUffer参数，默认200KB，如果子进程的输出大于这个值，将会抛出Error: stdout maxBuffer exceeded异常，并结束该子进程；
    + 当执行Node.js程序时，用spawn执行node的可执行文件
    ```javascript
        var child_process = require('child_process');
        function execNodeFile (file){
            var node = spawn(process.execPath, [file]);
            node.stdout.pipe(process.stdout);
            node.stderr.pipe(process.stderr);
            node.on('close', function (code) {
              console.log('进程结束，代码=%d', code);
            });  
        }
        execNodeFile('abc.js')
    ```
4. **处理uncaughtException事件**
   + 大多数情况下，异步I/O操作（如本地磁盘I/O、网络I/O），所发生的错误是无法被try/catch捕获的。如果其所抛出的异常没有被捕捉到，将会导致Node.js进程直接退出。而本项目中有大量的网络I/O操作。
   + 在Node.js中，如果一个抛出的异常没有被try/catch捕获到，其会尝试将这些错误交由uncaughtException事件处理程序来处理，仅当没有注册该事件处理程序时，才会最终导致进程直接退出。因此我们在app.js的末尾做了处理。
5. 使用**pm2**来启动程序
   + 有时候，由于Node.js自身的BUg或者使用到第三方C++模块的缺陷而导致一些底层的错误，比如在Linux系统下偶尔会发生段错误（segment fault）导致程序崩溃，此时上面提到的处理uncaughtException事件的方法就不适用了。
   + pm2是一个功能强大的进程管理器，通过pm2 start来启动Node.js程序，当该进程异常退出时，pm2会自动尝试重启进程，这样可以保证Node.js应用稳定运行。同时pm2还可以很方便地查看其所启动的各个进程的内存占用和日志等信息。
   + 全局安装 npm install -g pm2
   + 加入要启动的程序文件是~/app.js，在命令行下执行pmw start ~/app.js即可启动程序，执行pm2 stop ~/app.js即可停止该程序。 
6. 对于某些使用GBK字符编码的网站，如淘宝网，因为JS内部的字符编码是使用Unicode来表示的，因此在编写爬虫来处理这些GBK编码的网页内容时还需要将其转换成UTF-8编码。
   + 比如运行下面程序来抓取一个GBK编码的网页：
    ```javascript
        var request = require('request');
        var cheerio = require('cheerio');
        
        request('http://www.taobao.com/', function(err, res, body) {
          if(err) throw err;
          var $ = cheerio.load(body);
          // 执行后，输出内容时空白的，这是因为Node.js把GBK编码的网页内容当做Unicode编码来处理了。
          console.log($('head title').text());
        })
    ```
   + 第一种方法可以安装iconv-lite模块
    ```javascript
        var request = require('request');
        var cheerio = require('cheerio');
        var iconv = require('iconv-lite');
        
        request({
           url: 'http://www.taobao.com/',
           // 注意，设置request抓取网页时不要对接收到的数据做任何编码转换
           encoding: null
        }, function(err, res, body) {
          if(err) throw err;
          // 转换gbk编码的网页内容
          body = iconv.decode(body, 'gbk');
          var $ = cheerio.load(body);
          console.log($('head title').text());
        })
    ```
    + 第二种方法可以安装gbk模块
    ```javascript
        var cheerio = require('cheerio');
        var gbk = require('gbk');
        
        gbk.fetch('http://www.taobao.com/', 'utf-8').to('string', function(err, body) {
          if(err) throw err;
          var $ = cheerio.load(body);
          console.log($('head title').text());
        })
    ```


  [1]: https://mickeywang.com
  [2]: https://www.zhihu.com/people/laughingHome
  [3]: https://github.com/Mickey-Wang/crawler

