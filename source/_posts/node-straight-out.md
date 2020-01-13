title: node直出与同构javascript
date: 2016-09-18 11:06:10
tags: [node直出,同构javascript]
categories: node
---

## node直出简要介绍

node直出，简单地来说，就是把数据直出和页面直出都放在服务端来做，即，在混淆构建阶段服务端完成数据与页面的拼接，最后传给前端一个拼接完成的HTML，那么用户打开页面时，前端只需要进行页面的渲染。为了更好地进行优化，还可以把首屏的CSS也放进内联的CSS里，减少HTTP请求。

<!-- more -->

### node直出目的

1. 为了解决页面显示要等待后台请求（CGI等）返回来才能渲染页面的问题

2. SEO问题

    随着前端的发展，现在很多都是服务端只给出一个空的页面，或者是给出部分HTML，还有很大需要进行数据交互的页面是由前端进行渲染的，那么spider就无法抓取这部分的HTML内容，从而不能进行良好的SEO。

### node直出实现

一个重要的思想就是javascript同构，想要完成直出，必须在前后端都是用javascript，才能在后端完成页面拼接，那么这时这个负责直出的服务端（直出的服务端与CGI处理的服务端不是同一个），就要完成路由等等后端的概念。

这个概念与阿里提出的前后端分离还是有些相似的，因为js同构这一点，那么需要一个中间层服务端来完成直出，就像前后端分离概念中的node中间层来完成诸如路由这样的东西。
 
## 同构javascript

node直出，通过node直出首屏内容和关键性的SEO信息。无法直出所有内容，通常因为太大，所以前端还是需要维护原有的前端代码。

更多情况下，同一个站点中，我们更希望的是在某些场景下使用前端渲染，另一些情况下使用后端直出（例如，希望hybridapp在没有离线包的第一次直出，后面不需要下载静态文件时使用前端拉取渲染的方式，或者在高级的浏览器下使用http2前端渲染，低端浏览器上则使用直出），结果我们不得不维护两套不同的前后台代码，尽管可能都是用js写的。所以同构希望解决的是维护前后端维护两套代码的问题。

同构的目的，就是希望只开发一套项目代码，既可以实现前端的渲染，也可以做后台的直出。而且可以根据我们的需要，来决定是在前端渲染还是在后台直出。

实现的方式都是通过数据加上模版编译的方式生成，前端渲染与后端直出模式生成DOM的区别只在于数据和渲染发生在什么时候。

## 同构直出方案

### reactjs

对于react来说，使用`Rendertostring`这样的方法，就可以把虚拟DOM渲染成string变量传给前端。完成直出十分便捷。
因为virtual dom在前后端都可使用，这样就实现了直出的转换。

### mv*

自己在服务器端实现一个mvvm的核心，通过自己实现dom分析器来解析后端模版中的directives、filter、和事件绑定就可以。

一般的做法是完全根据现有的某个主流mvvm框架的语法来在后端实现一个功能相同的解析插件。mvc的原理也类似。

这种方式的实现参见：[Qjs直出实现过程](http://imweb.io/topic/560bb8eac2317a8c3e08623c)

### 实现的核心

实现的核心，就是保证一个数据渲染机制（react是virtual to html、mvvm是view模版）在前后端都能够正确解析。
使用React或mvvm只是说可以更好地做前端模块化管理。

## node直出与BigPipe

与直出并行的，还有一种方案就是BigPipe。

从开发模式上来说，BigPipe这种写法比较适合组件化、模块化的前端开发。

从网站规模上来说，这种优化效果应该比较明显，像是一些由很多模块组成的信息流很多的网站，比如微博
BigPipe也就是分步吐出内容。对于小型网站而言，直出会比分步速度更快。

## 参考

[降低首屏时间，直出是什么概念](http://www.cnblogs.com/vajoy/p/5079943.html)

[简单的原理介绍:node直出理论与实践总结](http://www.alloyteam.com/2016/07/node-straight-out/)

[基于koa+fis3+swig前后端isomorphic同构实现](http://jixianqianduan.com/frontend-build/2016/04/21/koa-fis3-swig-nodejs-isomorphic.html)

[isomorphic reactjs](http://jixianqianduan.com/frontend-javascript/2015/09/20/isomorphic-reactjs.html)

[rendr](https://github.com/rendrjs/rendr)

rendr是一个库，而不是框架。

backbone.js+handlebar实现的基于backbone应用javascript同构