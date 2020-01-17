title: 微信小程序概念及实践
date: 2016-12-19 20:28
tags: [微信小程序]
categories: 前端技术
---

微信小程序年前即将发布，自发布以来就广受关注。在这里，介绍下微信小程序的概念，以及一些实践（笔记）。

![微信小程序](http://thumbsnap.com/i/GLhHVkkv.jpg?1219)

<!-- more -->

# 微信小程序 概念

小程序可以借助微信联合登录，和开发者已有的APP后台的用户数据进行打通，但不会支持小程序和APP直接的跳转。

微信小程序提供了底层API和组件，引入了新的文件格式。

微信定义了新的文件格式，`wxml/wxss`，然后对这些文件做编译解析，所以微信小程序实际上是基于微信的原生应用。

微信小程序只能在微信中运行，并没有减少开发成本，而是多了一个开发流水线。

首先，抛出结论，微信小程序项目：

1. 不是移植H5

2. 需要一定的学习成本

3. 只能在微信中运行，没有减少开发成本，反而是多了一个开发流水线

4. 微信小程序本身还不完善

微信小程序与其说是拿H5来移植，不如说是需要学习一套新的前端框架，然后进行开发。这里科普几个重要的概念，以便把他们区分开来：

## H5+Hybrid：
Hybrid依然运行在IOS/Android的Webview容器中，JS执行环境与传统浏览器类似，Hybrid只是提供了可以用于Hybrid的Native接口，这些接口只是对于H5框架的补充。

## RN：
React Native相当于实现了一个完整的JS执行环境，完全不同于以往的浏览器环境，因而所涉及的性能、UI等核心问题必须通过Native的方式来解决，否则就无法获得类似Native的用户体验。RN的诞生就是为了使用JS来开发媲美原生应用的用户体验的产品。

## 微信小程序：
微信关于小程序里面的QA提到，页面的脚本逻辑是在JsCore中执行的，这也就是微信自己实现的一个JS执行环境，不同于浏览器环境。
由于window对象和document对象都是由浏览器提供的，这些在微信小程序里面都不能使用。

## H5：

HTML5的概念是W3C制定的一种标准，里面提供了一些API和新的语法特征。

---

由上可知，Hybrid（混合模式）是在H5编写的页面的基础上，进行了一层原生的包装，使H5页面能够用原生应用的形式表达出来，或者是嵌入到原生应用中的webview里进行执行，webview就相当于提供给JS执行的浏览器环境。

与其用微信小程序与H5做对比，不如说它更像是RN的一种实现。为了达到原生的效果，它有自己的组件语法等等，而且它本身处于一种封闭式环境。也就是说，微信提供底层技术支持，但玩法都得按照他的规则来。

微信小程序有自己的wxss和wxml，在wxss方面有自己的数据绑定、事件系统、组件、路由，wxss方面相当于一个CSS的子集，并且做了一些自己在移动端的适应拓展。

因此，评估这部分的技术工作量，不应与“移植H5”来做横向对比，而应当是开发RN+JS来进行评估。有一定的学习成本，而且微信小程序本身还不完善，是个封闭的环境，组件的使用等等方面都受到该环境的限制。

题外话，可以知道微信小程序实际上是自己造了一个类似RN+JS的轮子，它引爆互联网主要是因为提出的去中心化的思想，用户可以随用即走。个人感觉这只是PWA+RN的一个融合，概念是Google的PWA，技术是RN。

实际上PWA在2015年中旬就已经提出，并且Chrome、Opera、Firefox都提供了支持。只是苹果的浏览器还不支持，也就是说PWA在国外的安卓市场上实现比较多，而现在国内还缺乏这样一个大环境。貌似腾讯的浏览器也做过类似的实践，但论国内的影响力，现在这个在微信里面提供这样的支持，才能引爆大环境。

即，微信利用自己的平台流量优势，提供封闭的技术支持，来部署它自己的生态。由此，个人感觉这并不是一项技术的创新，更多的是一种概念的实施（不过这项概念在前端工程师眼中算是期待已久，这样JS能做的事情越来越多，从web页面到hybrid，现在脱离了浏览器环境，可以媲美原生应用），甚至是一种商业部署。

>
补充一下，感觉pwa这部分论述有误，可以说pwa是在微信小程序的对立面。pwa是让浏览器提供支持，主要是对发送通知，建立桌面入口这样的支持。而微信提供的支持是，按照它的规则来，然后微信进行编译，在它自己的js环境里运行，是媲美原生应用的成果。所以，微信小程序还是比pwa高端一点的，pwa没有跳出浏览器的限制，提出的是渐进式的网页应用，而微信小程序的目标就是轻量级的原生应用。现在感觉可以理解为什么有人会说微信小程序实际上是一个os，几乎可以说，java对应安卓，swift对应ios，那么js就对应微信os了…也就可以理解为什么说是前端工程师的春天…

以上基于本人对相关概念的理解，如有不正确的地方，还望指正。以下是相关笔记，学习微信小程序建议直接看官方API啦。

# 目录结构

根目录下的app,js、app.json、app.wxss
微信小程序会读取这些文件，生成 小程序实例

## app.js

可以在这个文件中监听并处理小程序的生命周期函数、声明全局变量。

## app.json

是对整个小程序的全局配置。可以在这个文件中配置小程序是由哪些文件组成的，小程序的窗口背景色、导航条样式、默认标题。
注意，该文件不可添加任何注释。

相关配置项：
https://mp.weixin.qq.com/debug/wxadoc/dev/framework/config.html?t=1475052047016

在pages中，文件名不需要写后缀，因为框架会自动去寻找路径.json、.js、,wxml、.wxss的四个文件进行整合。

## Debug

调试信息以info的形式给出，其信息有Page的注册、页面路由、数据更新、事件触发。

## page.json

每一个小程序页面都可以用.json的文件来对本页面的窗口表现进行配置，页面中配置项会覆盖app.json中的全局配置。

## app.wxss

是全局样式表。

## pages/

微信小程序中的每一个页面都需要写在pages/目录下，并且配置在.json文件中。

# 框架

## 逻辑层

小程序开发框架的逻辑层是由`JavaScript`编写。

在JS的基础上，由以下附加功能：

* 增加App和Page方法，进行程序和页面的注册
* 提供微信特有的扫一扫、支付等功能的API
* 每个页面都有独立的作用域，模块化
* 开发者写的所有代码最终都会打包成一份JS，并在小程序启动的时候运行直到小程序销毁。类似于ServiceWorker，所以逻辑层也叫App Service。


App方法：https://mp.weixin.qq.com/debug/wxadoc/dev/framework/app-service/app.html?t=1475052056481

Page方法：https://mp.weixin.qq.com/debug/wxadoc/dev/framework/app-service/page.html?t=1475052056377

## Page.prototype.setData()

`setData`函数用于将数据从逻辑层发送到视图层，同时改变对应的`this.data`的值。

注意

1. 直接修改this.data无效，无法改变页面的状态，还会造成数据的不一致。
2. 单次设置的数据不能超过1024kB。

## wx:if VS hidden

当wx:if的条件值切换时，框架有一个局部渲染的过程，因为它会确保条件块在切换时销毁或重新渲染。

如果初始时是false，那么框架什么也不做，是惰性的。

而hidden，组件始终都会被渲染，只是简单地控制显示与隐藏。

所以，需要频繁切换时用hidden，否则用wx:if。

## 事件

### dataset

在组件中可以定义数据，这些数据将会通过事件传递给 SERVICE。 书写方式： 以data-开头，多个单词由连字符-链接，不能有大写(大写会自动转成小写)如data-element-type，最终在 event.target.dataset 中会将连字符转成驼峰elementType。

## 引用

### import

import可以在该文件中使用目标文件定义的template。

import的作用域，只会import目标文件中定义的template，而不会import目标文件import的template。

### include

include可以将目标文件除了`<template />`的整个代码引入，相当于拷贝。

## WXSS

WXSS相当于CSS的子集，并在这个基础上，进行了扩展和修改。

扩展：

* 尺寸单位

* 样式导入

### 尺寸单位

rpx （responsive pixel）: 规定屏幕宽度为750rpx。

rem （root em）:规定屏幕宽度为20rem。

### 样式导入

@import

# 开发感受

1. 官方已经给出一些固有的组件，并且在设计方面也有一定的参考

2. 小程序本身是要小而美的，目前对应用的大小有限制。

3. 目前还在内部测试阶段，更新版本频繁甚至会出现相互冲突的情况，致使开发者不得不在每个版本上线后，都要进行对应的修改。

4. 更新混乱，安卓版本的微信和苹果版本更新的时间不一致，且实现的结果也与API不一致，留坑。

#### 参考

[微信小程序API](https://mp.weixin.qq.com/debug/wxadoc/dev/)

[微信小程序设计参考](https://mp.weixin.qq.com/debug/wxadoc/design/)