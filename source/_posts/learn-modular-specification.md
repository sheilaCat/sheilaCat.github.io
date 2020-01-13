title: "模块化规范学习"
date: 2015-04-28 22:13:07
tags: [模块化,seajs,commonjs,requirejs]
categories: 前端规范化
---
目前，实现AMD的库有RequireJS、curl、Dojo、Nodules等。

<!-- more -->

#CommonJS规范

*CommonJS是服务器端模块的规范，Node.js采用了这个规范。Node.js首先采用了JS模块化的概念。
根据CommonJS规范，一个单独的文件就是一个模块。每一个模块都是一个单独的作用域，也就是说，在该模块内部定义的变量，无法被其他模块读取，除非定义为global对象的属性。

输出模块变量的最好方法是使用module.exports对象。


#SeaJS与RequireJS最大的区别:

SeaJS对模块的态度是懒执行, 而RequireJS对模块的态度是预执行

#为什么要用requireJS

试想一下，如果一个网页有很多的js文件，那么浏览器在下载该页面的时候会先加载js文件，从而停止了网页的渲染，如果文件越多，浏览器可能失去响应。其次，要保证js文件的依赖性，依赖性最大的模块（文件）要放在最后加载，当依赖关系很复杂的时候，代码的编写和维护都会变得困难。

RequireJS就是为了解决这两个问题而诞生的：

（1）实现js文件的异步加载，避免网页失去响应；
（2）管理模块之间的依赖性，便于代码的编写和维护。

#AMD和CMD

CMD（Common Module Definition） 通用模块定义。该规范明确了模块的基本书写格式和基本交互规则。该规范是在国内发展出来的。AMD是依赖关系前置，CMD是按需加载。

AMD 是 RequireJS 在推广过程中对模块定义的规范化产出。
CMD 是 SeaJS 在推广过程中对模块定义的规范化产出。

CMD 推崇依赖就近，AMD 推崇依赖前置。看如下代码：

// CMD
define(function(require, exports, module) {
var a = require('./a')
a.doSomething()
// 此处略去 100 行
var b = require('./b') // 依赖可以就近书写
b.doSomething()
// ... 
})

// AMD 默认推荐的是
define(['./a', './b'], function(a, b) { // 依赖必须一开始就写好
a.doSomething()
// 此处略去 100 行
b.doSomething()
...
}) 

另外一个区别是：

AMD:API根据使用范围有区别，但使用同一个api接口
CMD:每个API的职责单一

参考文章：
http://segmentfault.com/a/1190000000733959

#RequireJS与SeaJS的执行差异

equireJS你坑的我一滚啊, 这也就是为什么我不喜欢RequireJS的原因, 坑隐藏得太深了.
终于明白玉伯说的那句: "RequireJS 是没有明显的 bug，SeaJS 是明显没有 bug"是什么意思了


因此我们得出结论(分别使用SeaJS 2.0.0和RequireJS 2.1.6进行测试)
-------------------------
SeaJS只会在真正需要使用(依赖)模块时才执行该模块
SeaJS是异步加载模块的没错, 但执行模块的顺序也是严格按照模块在代码中出现(require)的顺序, 这样才更符合逻辑吧! 你说呢, RequireJS?

而RequireJS会先尽早地执行(依赖)模块, 相当于所有的require都被提前了, 而且模块执行的顺序也不一定100%就是先mod1再mod2
因此你看到执行顺序和你预想的完全不一样! 颤抖吧~ RequireJS!

http://www.douban.com/note/283566440/