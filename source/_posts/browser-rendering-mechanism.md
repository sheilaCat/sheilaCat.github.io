title: 浏览器渲染机制
date: 2016-4-1 11:01
tags: [浏览器渲染,浏览器渲染原理,重绘与重排]
categories: 浏览器相关
---

简要介绍了浏览器渲染的原理，以及CSS和JavaScript的加载。简要介绍了重绘和重排，以及减少重绘重排的一些方法。

<!-- more -->

## 浏览器显示页面的原理

* 获取HTML 文档及样式表文件
* 解析成对应的树形数据结构
    * DOM tree
    * CSSOM tree
* 计算可见节点形成render tree
* 计算DOM 的形状及位置进行布局
* 将每个节点转化为实际像素绘制到视口上（栅格化）

`render tree`（页面上所显示的最终结果）是由`DOM tree`（开发工具中所显示的HTML 所定义的内容结构）与`CSSOM tree`（样式表所定义的规则结构）合并并剔除不可见的节点所形成的，其中不包含如下节点:

* 本身不可见的
    * `<html>`
    * `<head>`
    * `<meta>`
    * `<link>`
    * `<style>`
    * `<script>`
* 设置了`display: none;`样式的

浏览器工作大致流程：

![浏览器工作](http://thumbsnap.com/s/jbVYmtcJ.jpg?0331)

## 浏览器加载和渲染HTML的顺序

1. IE下载的顺序是从上到下，渲染的顺序也是从上到下，下载和渲染是同时进行的。

2. 在渲染到页面的某一部分时，其上面的所有部分都已经下载完成（并不是说所有相关联的元素都已经下载完）

3. 如果遇到语义解释性的标签嵌入文件（JS脚本，CSS样式），那么此时IE的下载过程会启用单独连接进行下载。

4. 并且在下载后进行解析，解析过程中，停止页面所有往下元素的下载。阻塞加载

5. 样式表在下载完成后，将和以前下载的所有样式表一起进行解析，解析完成后，将对此前所有元素（含以前已经渲染的）重新进行渲染。

6. JS、CSS中如有重定义，后定义函数将覆盖前定义函数

## JavaScript的加载

1. 不能并行下载和解析（阻塞下载）
2. 当引用了JS的时候，浏览器发送1个js request就会一直等待该request的返回。因为浏览器需要1个稳定的DOM树结构，而JS中很有可能有代码直接改变了DOM树结构，比如使用 document.write 或 appendChild,甚至是直接使用的location.href进行跳转，浏览器为了防止出现JS修改DOM树，需要重新构建DOM树的情况，所以 就会阻塞其他的下载和呈现.

## 解析CSS

![解析CSS](http://thumbsnap.com/i/iiFti0rb.png?0331)

## Reflow & Repaint

`Reflow`的成本比`Repaint`的成本高得多。DOM Tree里的每个结点都会有`reflow`方法，一个结点的`reflow`很有可能导致子结点、甚至父结点以及同级结点的`reflow`。

所以，下面这些动作有很大可能会是成本比较高的：

* 当你增加、删除、修改DOM结点时，会导致Reflow或Repaint
* 当你移动DOM的位置，或是搞个动画的时候。
* 当你修改CSS样式的时候。
* 当你Resize窗口的时候（移动端没有这个问题），或是滚动的时候。
* 当你修改网页的默认字体时。

基本上来说，reflow有如下的几个原因：

* Initial。网页初始化的时候。
* Incremental。一些Javascript在操作DOM Tree时。
* Resize。其些元件的尺寸变了。
* StyleChange。如果CSS的属性发生变化了。
* Dirty。几个Incremental的reflow发生在同一个frame的子树上。

当然，我们的浏览器是聪明的，它不会你每改一次样式，它就reflow或repaint一次。一般来说，浏览器会把这样的操作积攒一批，然后做一次reflow，这又叫异步reflow或增量异步reflow。但是有些情况浏览器是不会这么做的，比如：resize窗口，改变了页面默认的字体，等。对于这些操作，浏览器会马上进行reflow。

但是有些时候，我们的脚本会阻止浏览器这么干，比如：如果我们请求下面的一些DOM值：

* offsetTop, offsetLeft, offsetWidth, offsetHeight
* scrollTop/Left/Width/Height
* clientTop/Left/Width/Height
* IE中的 getComputedStyle(), 或 currentStyle

因为，如果我们的程序需要这些值，那么浏览器需要返回最新的值，而这样一样会flush出去一些样式的改变，从而造成频繁的reflow/repaint。


## 减少reflow/repaint
下面是一些Best Practices：

1）不要一条一条地修改DOM的样式。与其这样，还不如预先定义好css的class，然后修改DOM的className。

2）把DOM离线后修改。如：
* 使用documentFragment 对象在内存里操作DOM
* 先把DOM给display:none(有一次reflow)，然后你想怎么改就怎么改。比如修改100次，然后再把他显示出来。
clone一个DOM结点到内存里，然后想怎么改就怎么改，改完后，和在线的那个的交换一下。

3）不要把DOM结点的属性值放在一个循环里当成循环里的变量。不然这会导致大量地读写这个结点的属性。

4）尽可能的修改层级比较低的DOM。当然，改变层级比较底的DOM有可能会造成大面积的reflow，但是也可能影响范围很小。

5）为动画的HTML元件使用fixed或absoult的position，那么修改他们的CSS是不会reflow的。

6）千万不要使用table布局。因为可能很小的一个小改动会造成整个table的重新布局。


参考:
[浏览器的渲染原理简介](http://coolshell.cn/articles/9666.html)

拓展阅读：
[PageSpeed Insights规则](https://developers.google.com/speed/docs/insights/rules?csw=1#header)

[Best Practices for Speeding Up Your Web Site](https://developer.yahoo.com/performance/rules.html)

推荐阅读：

[How browsers work](http://taligarsiel.com/Projects/howbrowserswork1.htm)

中文翻译：[浏览器是怎样工作的](http://my.oschina.net/anna153/blog/377259#OSC_h2_18)

[从FE的角度上再看输入url后都发生了什么](http://div.io/topic/609)

[当你在浏览器中输入Google.com并且按下回车之后发生了什么？](http://blog.jobbole.com/84870/)



