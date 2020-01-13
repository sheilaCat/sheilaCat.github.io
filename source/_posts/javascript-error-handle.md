title: "JavaScript错误处理"
date: 2016-08-09 11:06:10
tags: [JavaScript,错误处理]
categories: JavaScript
---
## 基本介绍

### JavaScript Error Hadler

error事件的event handler，Error事件会在不同目标上因各种错误而触发：

* 当JavaScript运行时错误（包括语法错误）发生时，window会触发一个ErrorEvent接口的error事件，并执行`window.onerror()`。

* 当一项资源（如`<img>`或`<script>`）加载失败，加载资源的元素会触发一个Event接口的error事件，并执行该元素上的`onerror()`处理函数。这些error事件不会向上冒泡到window，不过（至少在Firefox中）能被单一的window.addEventListener捕获。

<!-- more -->

加载一个全局的error事件处理函数可用于自动收集错误报告。

一个JavaScript错误由**错误信息**（`error message`）和**追溯栈**（`stack trace`）两个主要部分组成。

追溯栈中的每一帧由以下三个部分组成：一个函数名（发生错误的代码不是在全局作用域中执行），发生错误的脚本在网络中的地址，以及发生错误代码的行数和列数。

### 异步追溯栈

在JavaScript代码中异步代码是非常常见的，比如`setTimeout`的使用，或者`Promise对象`的使用，这些异步调用入口往往会给追溯栈带来问题，因为异步代码会生成一个新的执行上下文，而追溯栈又会重新形成追溯帧。

目前，异步追溯栈只有Chrome DevTools支持，而且只有在DevTools代开的情况下才会捕获，在代码中通过Error对象不会获取到异步追溯栈。

虽然可以模拟异步调用栈，但是这往往会代指应用性能的消耗。

## 出错处理的常用方式

### window.onerror

```javascript
window.onerror = function(msg, url, line, col, err) {
    console.log('Application encountered an error: ' + msg);
    console.log('Stack trace: ' + err.stack);
}
```

在使用这个方法捕获错误的时候会出现如下问题： No Error object provided

这是因为有的浏览器对第五个回调参数并不提供。

浏览器是事件驱动的。JavaScript中的错误也是一个事件。解释器在当前的执行上下文中执行后释放。所以，我们可以利用`onerror`全局异常事件处理函数。

```javascript
if (window.addEventListener) {
    window.addEventListener('error', function(e) {
        var error = e.error;
        console.log(error);
    });
} else if (window.attachEvent) {
    window.attachEvent('onerror', function(e) {
        var error = e.error;
        console.log(error);
    });
} else {
    window.onerror = function(e) {
        var error = e.error;
        console.log(error);
    }
}
```

### try/catch

在代码中使用 try catch 是否对性能有影响：

* 在V8（其他JS引擎也可能出现相同情况）函数中使用了try/catch语句不能够被V8编译器优化。参考 http://www.html5rocks.com/en/tutorials/speed/v8/

* try catch 对性能的影响微乎其微，但是一些用法会让性能受很大的影响， 参考这个实验：http://taobaofed.org/blog/2015/10/28/try-catch-runing-problem/

总结下来就是：在 try 语句块中不要定义太多的变量，最好是只写一个函数调用，避免 try 运行中变量拷贝造成的性能损耗。

try-catch与window.onerror相比的弱势：

1. 它只能在单一的作用域内有效

2. V8引擎不鼓励在函数中使用try-catch 他们的建议是在最外层写这些块

## JavaScript错误收集的关键点

### 行内JS代码或者使用eval情况

在使用eval或者HTML中写JS时，追溯栈通常会使用HTML的URL以及代码执行的行数和列数。

由于一些性能或代码优化的原因，HTML中往往会有行内脚本。为了追溯这些JS的错误，Chrome和Firefox支持`sourceURL`的声明，往往会在追溯帧后面添加一个`inline.js。这里的行数是从HTML文档开始处开始计算的，一般被认为是不正确的。

另外，不同浏览器都有自己处理eval代码错误的追溯栈格式，兼容不同浏览器解析eval代码将变得异常困难。

## Cross domain sanitization

在Chrome中，`window.onerror`能够检测到从别的域引用的script文件中的错误（比如从CDN上面引用的jQuery源文件）并且将这些错误标记为`Script error`。如果你不想处理这些从别的域引入的script文件，那么可以在程序中通过script error标记将其过滤掉。然而，在Firefox、Safari或者IE11中，并不会引入跨域的JS错误，即使在Chrome中，如果使用`try/catch`将这些讨厌的代码包围，那么Chrome也不会再检测到这些跨域错误。

### 解决方法

有两个方案：

1. 在Chrome中，可以设置合适的跨域头信息来捕获跨域JS错误。

在Chrome中，如果你想通过`window.onerror`来获取到完整的跨域错误信息，那么这些跨域资源必须提供合适的跨域头信息。可以参考下面地址 https://mikewest.org/2013/08/debugging-runtime-errors-with-window-onerror

    1. 先给script文件设置一个属性crossorigin

    `<script type="text/javascript" src="xxx.js" crossorigin></script>`

    2. 给请求的文件头返回信息设置合适的跨域头信息`Access-Control-Allow-Origin`

2. 手动包裹一些要检测的代码，没有跨域问题并且可以获得到error对象。这种方式相对麻烦一些，但可以通过全局的hook，处理大部分情况，不必每次都要写`try...catch...`。这里可以参考`tryjs`的做法。

tryjs可以捕获跨域错误，以及包裹异步方法来捕获并定位错误。

#### [tryjs](https://github.com/imweb/tryjs)

>window.onerror在webkit中对于跨域的脚本错误无法捕获其stack，经常让我们无法定位上报的问题，tryjs利用try-catch将函数包裹起来，让错误捕获变得容易。

#### 原理

对于基于AMD和jQuery的网站，几乎所有业务函数都是通过回调异步触发的，所以我们只需要将所有异步函数包裹起来就可以捕获到大部分错误。

例如，对于require函数，一般是这样使用的：

```javascript
require(['./main'],
    // 想办法把这个函数包裹起来 
    function (main) {
        // 实际上这里才是在调用
        main.init();
    });
```

类似的对于setTimeout函数，一般可以这样：

```javascript
setTimeout(
    // 想办法把这个函数包裹起来就行了
    function () {
        dosomthing();
    }, 
    1000
);
```

## JavaScript错误监控上报

### 基本要素

1. onerror的回调 try..catch来进行错误处理

`onerror`适用于任何错误，`try...catch`适用于某个明确知道可能会出错的点，并进行错误处理

2. JavaScript错误上报的**采样率**

没有必要把所有的错误信息全部都送到Log中，量太大。可以限制一个采样率。

```javascript
function needReport (sampling){
  // sampling: 0 - 1
  return Math.random() <= sampling;
}
Reporter.send = function(errInfo, sampling) {
  if(needReport(sampling || 1)){
    Reporter._send(errInfo);
  }
};
```

3. 是否重复上报、是否延迟上报

4. 上报的错误级别 info log debug...

### 为了提升代码的可调试性

1. 为匿名函数取名

2. 将函数赋给一个变量

inner function Firefox stack frame warning 在一个函数定义在另外一个函数内部的情景下（闭包）Firefox会使用不同于其他浏览器厂商的格式来处理函数名
3、displayName属性
除了IE11，函数名的展现也可以通过给函数定义一个`displayName`属性，它会出现在浏览器的devtools debugger中。而Safari，`displayName`还会出现在追溯帧中。

 


#### 推荐阅读

[前端代码异常日志收集与监控](http://www.cnblogs.com/hustskyking/p/fe-monitor.html)

[JSTracker：前端异常数据采集](http://taobaofed.org/blog/2015/10/28/jstracker-how-to-collect-data/)


#### 参考

[【原译】javascript中的错误处理](http://jixianqianduan.com/article-translation/2016/05/12/proper-error-handler-in-javascript.html)

[MDN：GlobalEventHandlers.onerror](https://developer.mozilla.org/zh-CN/docs/Web/API/GlobalEventHandlers/onerror)

[JavaScript Error 指南（翻译）](https://cnodejs.org/topic/56f3a1d5ce1f18c57719a4c2)

[关于javascript错误捕获](http://imweb.io/topic/55e3d6e3771670e207a16bd5)

[前端相关数据监控](http://www.alloyteam.com/2014/03/front-end-data-monitoring/)

