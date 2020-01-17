title: JavaScript内存泄漏与内存管理
date: 2016-04-06 16:22
tags: [内存泄漏,内存管理,引用计数,标记清除,循环引用]
categories: 浏览器相关
---

本文说明了JavaScript中内存的生命周期，以及内存管理与内存泄漏，内存泄漏的常见情况和解决办法。

![内存管理与内存泄漏](http://thumbsnap.com/i/UhcQVx2r.png?0406)

<!-- more -->

## 内存管理

JS运行的时候，会有栈内存（stack）和堆内存（heap），当我们用new实例化一个类的时候，这个new出来的对象就保存在heap里面，而这个对象的引用则存储在stack里。程序通过stack里的引用找到这个对象。例如var a = [1,2,3];，a是存储在stack里的引用，heap里存储着内容为[1,2,3]的Array对象。当栈中的变量被重新符值，原来在堆中存储的对象就会被释放，这个过程就叫垃圾回收。


## 内存生命周期

不管什么程序语言，内存生命周期基本一致：   

1. 分配你所需要的内存
1. 使用它（读、写）
1. 当它不被使用时释放   ps：和“把大象装冰箱“一个意思

第一二部分过程在所有语言中都很清晰。最后一步在低级语言中很清晰，但是在像JavaScript等高级语言中，最后一步不清晰。

## 垃圾回收

JavaScript具有**自动垃圾收集机制**，也就是说，执行环境会负责管理代码执行过程中使用的内存。这种垃圾收集机制的原因很简单：**找出那些不再继续使用的变量，然后释放其占用的内存。为此，垃圾收集器会按照固定的时间间隔（或代码执行中预定的收集时间），周期性地执行这一操作。**

### 引用计数算法 Reference Counting

这是最简单的垃圾收集算法。此算法把“对象是否不再需要”简化定义为“对象有没有其他对象引用到它”。如果没有引用指向该对象（零引用），对象将被垃圾回收机制回收。

**限制：循环引用**

这个简单的算法有一个限制，就是如果一个对象引用另一个（形成了循环引用），他们可能“不再需要”了，但是他们不会被回收。

```javascript
function f(){
  var o = {};
  var o2 = {};
  o.a = o2; // o 引用 o2
  o2.a = o; // o2 引用 o

  return "azerty";
}

f();
// 两个对象被创建，并互相引用，形成了一个循环
// 他们被调用之后不会离开函数作用域
// 所以他们已经没有用了，可以被回收了
// 然而，引用计数算法考虑到他们互相都有至少一次引用，所以他们不会被回收
```

实际当中的例子：

IE 6, 7 对DOM对象进行引用计数回收。对他们来说，一个常见问题就是内存泄露：

```javascript
var div = document.createElement("div");
div.onclick = function(){
  doSomething();
}; 
// div有了一个引用指向事件处理属性onclick
// 事件处理也有一个对div的引用可以在函数作用域中被访问到
// 这个循环引用会导致两个对象都不会被垃圾回收
```

我们知道，IE中有一部分对象并不是原生JavaScript对象。例如，其BOM和DOM中的对象就是使用C++以COM（Component Object Model，组件对象模型）对象的形式实现的，而COM对象的垃圾收集机制采用的就是引用计数策略。因此，即使IE的JavaScript引擎是使用标记清除策略来实现的，但JavaScript访问的COM对象依然是基于引用计数策略的。换句话说，只要在IE中涉及COM对象，就会存在循环引用的问题：

```javascript
var element = document.getElementById("some_element");
var myObject = new Object();
myObject.element = element;
element.someObject = myObject;
```

DOM元素与原生JavaScript对象之间创建了循环引用。由于存在这个循环引用，即使将例子中的DOM从页面中移除，它也永远不会被回收。

在不使用它们的时候手工断开连接：

```javascript
myObject.element = null;
element.someObject = null;
```

为了解决上述问题，IE9把BOM和DOM对象都转换成了真正的JavaScript对象。

JavaScript引擎目前都不再使用这种算法；但在IE中访问非原生JavaScript对象（如DOM元素）时，这种算法仍然可能会导致问题。

### 标记-清除算法 Mark and sweep

这个算法把“对象是否不再需要”简化定义为“对象是否可以获得”。

这个算法假定设置一个叫做根的对象（在Javascript里，根是全局对象）。定期的，垃圾回收器将从根开始，找所有从根开始引用的对象，然后找这些对象引用的对象……从根开始，垃圾回收器将找到所有可以获得的对象和所有不能获得的对象。

>垃圾回收器在运行的时候会给存储在内存中的所有变量都加上标记（当然，可以使用任何标记方式）。然后它会去掉环境中的变量以及被环境中的变量引用的变量标记。而在此之后再被加上标记的变量将被视为准备删除的变量，原因是环境中的变量已经无法访问到这些变量了。最后，垃圾收集器完成内存清除工作，销毁那些带标记的值并回收它们所占用的内存空间。

这个算法比前一个要好，因为“有零引用的对象”总是不可获得的，但是相反却不一定，参考“循环引用”。

从2012年起，所有现代浏览器都使用了标记-清除垃圾回收算法。所有对JavaScript垃圾回收算法的改进都是基于标记-清除算法的改进，并没有改进标记-清除算法本身和它对“对象是否不再需要”的简化定义。

**循环引用不再是问题了**

在上面的示例中，函数调用返回之后，两个对象从全局对象出发无法获取。因此，他们将会被垃圾回收器回收。
第二个示例同样，一旦 div 和其事件处理无法从根获取到，他们将会被垃圾回收器回收
。

**限制: 那些无法从根对象查询到的对象都将被清除**

尽管这是一个限制，但实践中我们很少会碰到类似的情况，所以开发者不太会去关心垃圾回收机制。

## 内存泄漏

能导致内存泄漏的一定是引用类型的变量，比如函数和其他自定义对象。而值类型的变量是不存在内存泄漏的，比如字符串、数字、布尔值等。

因为值类型是靠复制来传递的，而引用类型是靠类似c语言中的指针来传递的。
可以认为一个引用类型的变量就是一个指向某个具体的内存地址的指针。

当我们用JavaScript代码创建一个引用类型的时候（以下简称对象），js引擎会在内存中开辟一块空间来存放数据，并把指针引用交给那个变量。内存是有限的，js引擎必须保证当开辟的对象没用的时候，把所分配的内存空间释放出来，这个过程叫做垃圾回收，负责回收的叫做垃圾回收器（GC）。

内存泄漏是指我们已经无法再通过js代码来引用到某个对象，但垃圾回收器却认为这个对象还在被引用，因此在回收的时候不会释放它。导致了分配的这块内存永远也无法被释放出来。如果这样的情况越来越多，会导致内存不够用而系统崩溃。

许多功能无需考虑内存管理即可实现，但却忽略了它可能在程序中带来重大的问题。不当清理的对象可能会存在比预期要长得多的时间。这些对象继续响应事件和消耗资源。它们可强制浏览器从一个虚拟磁盘驱动器分配内存页，这显著影响了计算机的速度（在极端的情形中，会导致浏览器崩溃）。

过去导致内存泄漏的许多经典模式在现代浏览器中以不再导致泄漏内存。但是，如今有一种不同的趋势影响着内存泄漏。许多人正设计用于在没有硬页面刷新的单页中运行的 Web 应用程序。在那样的单页中，从应用程序的一个状态到另一个状态时，很容易保留不再需要或不相关的内存。

### 造成内存泄漏的主要原因

#### 循环引用

要想破坏循环引用，引用DOM元素的对象或DOM对象的引用需要被赋值为null。

#### JavaScript闭包

闭包可以导致内存泄露是因为内部方法保持一个对外部方法变量的引用，所以尽管方法返回了内部方法还可以继续访问在外部方法中定义的私有变量。对Javascript程序员来说最好的做法是在页面重载前断开所有的事件处理器。

在一个闭包中引用的任何局部变量都会被该闭包保留，只要该闭包存在就永远保留。

##### 销毁对象和对象所有权

一种不错的做法是，创建一个标准方法来负责让一个对象有资格被垃圾回收。destroy 功能的主要用途是，集中清理该对象完成的具有以下后果的操作的职责：

* 阻止它的引用计数下降到0（例如，删除存在问题的事件侦听器和回调，并从任何服务取消注册）。

* 使用不必要的 CPU 周期，比如间隔或动画。

destroy 方法常常是清理一个对象的必要步骤，但在大多数情况下它还不够。在理论上，在销毁相关实例后，保留对已销毁对象的引用的其他对象可调用自身之上的方法。因为这种情形可能会产生不可预测的结果，所以仅在对象即将无用时调用 destroy 方法，这至关重要。

一般而言，destroy 方法最佳使用是在一个对象有一个明确的所有者来负责它的生命周期时。此情形常常存在于分层系统中，比如 MVC框架中的视图或控制器，或者一个画布呈现系统的场景图。

#### DOM插入顺序

DOM 对象应该按照从当前页面存在的最上面的 DOM 元素开始往下直到剩下的 DOM 元素的顺序添加，这样它们就总是有同样的范围，不会产生临时对象。

#### 控制台日志

控制台日志记录对总体内存配置文件的影响可能是许多开发人员都未想到的极其重大的问题。记录错误的对象可以将大量数据保留在内存中。注意，这也适用于：

* 在用户键入 JavaScript 时，在控制台中的一个交互式会话期间记录的对象。
* 由 console.log 和 console.dir 方法记录的对象。

#### XMLHttpRequest对象泄漏

在IE低版本中，除了上面说的BOM和DOM对象的循环引用会造成内存泄露外，还有一个js对象也会产生泄漏，就是XMLHttpRequest对象，XMLHttpRequest在IE<9会造成内存泄露，看下面的代码：

```javascript
window.onload = function() {
  setInterval(function(){
     var xhr = new XMLHttpRequest();
     xhr.largeStr = new Array(100000).join('xxoo');
     xhr.open('GET', '/users', true);
     xhr.onreadystatechange = function() {
        if(this.readyState == 4 && this.status == 200) {
           document.getElementById('test').innerHTML++
        }
     };
     xhr.send();
    //xhr = null; 解决办法 切断xhr引用
  }, 50)
}
```

关于这个在低版本中XMLHttpRequest对象内存泄露问题，网上资源很少，英文资料也特别少，搜到一篇分析不错的文章： http://nullprogram.com/blog/2013/02/08/

结合作者的理解：

按照正常情况，在结束定时器闭包时，xhr及绑定事件的闭包应该被标计清除法清除掉，但是想像一下，如果真清除掉了，这个XHR实例也就没有意思了，因为后面即使请求成功了，也没有XHR来响应了。猜测浏览器内部应该是把这个xhr及闭包函数放在了一个不能被外部直接访问空间里来响应回调。并且通过一套机制来等响应成功后清除掉这块内存空间，但是早期的IE在这个设计上存在不足，没法回收。总结一句就是：XMLHttpRequest特殊就在于它是异部执行的，所以不能立马从内存中释放。

### 易出现泄露的场景

1. XMLHttpRequest 泄漏发生在IE7-8

1. DOM&BOM等COM对象循环绑定 泄漏发生在IE6-8

1. 定时器(严格上说不能算是泄露，是被闭包持有了，是正常的表现)

### 总结

为了预防内存泄露及持有不必要的内存，做到如下几点：

1. 在使用XMLHttpRequest，定时器等异步对象时，在函数结尾对不需要引用的对象进行释放

1. 在DOM循环引用中，记得在函数结束时对循环引用切断

解除变量的引用不仅有助于消除循环引用现象，而且对垃圾收集也有好处。为了确保有效地回收内存，应该及时解除不再使用的全局对象、全局对象属性以及循环引用变量的引用。

引用：

《JavaScript高级程序设计》4.3 垃圾回收

[内存泄漏](https://segmentfault.com/q/1010000000414875)

[MDN：内存管理](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Memory_Management)

[关于Javascript内存泄露的那些小事儿](http://www.ituring.com.cn/article/109716).

[了解 JavaScript 应用程序中的内存泄漏](https://www.ibm.com/developerworks/cn/web/wa-jsmemory/)

推荐：

JavaScript内存分析实战：[JavaScript内存分析](https://github.com/CN-Chrome-DevTools/CN-Chrome-DevTools/blob/master/md/Performance-Profiling/javascript-memory-profiling.md)

[V8 之旅： 垃圾回收器](http://newhtml.net/v8-garbage-collection/)