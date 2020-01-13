title: 重绘与重排
date: 2016-4-1 14:14
tags: [浏览器渲染,重绘与重排]
categories: 浏览器相关
---

先阅读***浏览器渲染机制***，在浏览器渲染中会进行重绘与重排。

<!-- more -->

先阅读***浏览器渲染机制***，在浏览器渲染中会进行重绘与重排。

## 重排与重绘

1. 当render tree中的一部分(或全部)因为元素的规模尺寸，布局，隐藏等改变而需要重新构建。这就称为重排(reflow)。每个页面至少需要一次重排，就是在页面第一次加载的时候。在重排的时候，浏览器会使渲染树中受到影响的部分失效，并重新构造这部分渲染树，完成重排后，浏览器会重新绘制受影响的部分到屏幕中，该过程成为重绘。

2. 当render tree中的一些元素需要更新属性，而这些属性只是影响元素的外观，风格，而不会影响布局的，比如background-color。则就叫称为重绘。

注意：重排必将引起重绘，而重绘不一定会引起重排。

 
## 重排何时发生

当页面布局和几何属性改变时就需要重排。下述情况会发生浏览器重排：

1. 添加或者删除可见的DOM元素；

1. 元素位置改变；

1. 元素尺寸改变——边距、填充、边框、宽度和高度

1. 内容改变——比如文本改变或者图片大小改变而引起的计算值宽度和高度改变；

1. 页面渲染初始化；

1. 浏览器窗口尺寸改变——resize事件发生时；
 
重排的花销跟DOM节点所处的层次有关系，层级越高，开销越大。

以上只是简单列举了出现重排和重绘的情况，附两篇进行详尽列举的文章：

* 在JavaScript中会引起强制重排和重绘的相关属性和方法：[What forces layout / reflow](https://gist.github.com/paulirish/5d52fb081b3570c81e3a)
 
* 在CSS中能触发布局、绘制或渲染层合并的属性：[csstriggers](https://csstriggers.com/)

## flush队列

重排和重绘很容易被引起，而且重排的花销也不小，如果每句JS操作都去重排重绘的话，浏览器可能就会受不了。所以很多浏览器都会优化这些操作，浏览器会维护1个队列，把所有会引起重排、重绘的操作放入这个队列，等队列中的操作到了一定的数量或者到了一定的时间间隔，浏览器就会flush队列，进行一个批处理。这样就会让多次的重排、重绘变成一次重排重绘。

虽然有了浏览器的优化，但有时候我们写的一些代码可能会强制浏览器提前flush队列，这样浏览器的优化可能就起不到作用了。当你请求向浏览器请求一些 style信息的时候，就会让浏览器flush队列，比如：

1. offsetTop, offsetLeft, offsetWidth, offsetHeight

1. scrollTop/Left/Width/Height

1. clientTop/Left/Width/Height

1. width,height

1. 请求了getComputedStyle(), 或者 IE的 currentStyle

当你请求上面的一些属性的时候，浏览器为了给你最精确的值，需要flush队列，因为队列中可能会有影响到这些值的操作。即使你获取元素的布局和样式信息跟最近发生或改变的布局信息无关，浏览器都会强行刷新渲染队列。

## 如何减少重排、重绘

减少重排、重绘其实就是需要减少对render tree的操作（合并多次多DOM和样式的修改），并减少对一些style信息的请求，尽量利用好浏览器的优化策略。具体方法有：

* 不要单独操作对DOM元素的样式修改，直接改变className，如果动态改变样式，则使用cssText（考虑没有优化的浏览器）

* 让要操作的元素进行”离线处理”，处理完后一起更新

    *a)* 使用DocumentFragment进行缓存操作,引发一次重排和重绘；

    *b)* 使用display:none技术，只引发两次重排和重绘；

    *c)* 使用cloneNode(true or false) 和 replaceChild 技术，引发一次重排和重绘；

* 将需要多次重排的元素，position属性设为absolute或fixed，这样此元素就脱离了文档流，它的变化不会影响到其他元素。例如有动画效果的元素就最好设置为绝对定位。

* 将新建的dom节点操作完毕后在插入到body中。

* 避免强制同步布局事件的发生：尽量不要在布局信息改变时做查询（会导致渲染队列强制刷新）。

* 尽可能避免触发布局：

```javascript
.box {
  width: 20px;
  height: 20px;
}

/**
 * Changing width and height
 * triggers layout.
 */
.box--expanded {
  width: 200px;
  height: 350px;
}
```

* 使用flexbox替代老的布局模型。

* 避免快速连续的布局：不要经常访问会引起浏览器flush队列的属性，如果你确实要访问，利用先读后写的原则。

```javascript
function resizeAllParagraphsToMatchBlockWidth() {

  // Puts the browser into a read-write-read-write cycle.
  for (var i = 0; i < paragraphs.length; i++) {
    paragraphs[i].style.width = box.offsetWidth + 'px';
  }
}
```

我们使用`先读后写`的原则，来修复上述代码中的问题：

```javascript
// Read.
var width = box.offsetWidth;

function resizeAllParagraphsToMatchBlockWidth() {
  for (var i = 0; i < paragraphs.length; i++) {
    // Now write.
    paragraphs[i].style.width = width + 'px';
  }
}
```

#### 引用：

[页面重绘和回流以及优化](http://www.css88.com/archives/4996)

[高性能JavaScript 重排与重绘](http://www.cnblogs.com/zichi/p/4720000.html)

[Rendering: repaint, reflow/relayout, restyle](http://www.phpied.com/rendering-repaint-reflowrelayout-restyle/)

[Avoid large, complex layouts and layout thrashing
](https://developers.google.com/web/fundamentals/performance/rendering/avoid-large-complex-layouts-and-layout-thrashing?hl=en)

[中文翻译 避免大规模、复杂的布局
](https://developers.google.com/web/fundamentals/performance/rendering/avoid-large-complex-layouts-and-layout-thrashing?hl=ch)

#### 推荐阅读：

[网页性能管理详解](http://www.ruanyifeng.com/blog/2015/09/web-page-performance-in-depth.html)

>这篇文章讲到了使用`window.requestAnimationFrame()`和`window.requestIdleCallback()`来调节浏览器重新渲染的时间点。

>另外，这篇文章中，所提到的，不要在写操作之后紧跟一个读操作。也就是`flush队列`中提到的，本来是会攒一堆重绘重排操作，但紧跟一个读操作之后，为了给你一个准确值，就会立即进行重绘重排。
