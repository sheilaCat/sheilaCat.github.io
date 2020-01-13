title: JavaScript异步编程
date: 2016-03-23 23:51:22
tags: [JavaScript,异步]
categories: JavaScript
---

如果你想要更正式一点的 JavaScript 语言介绍，请揣摩Marijn Haverbeke 的Eloquent JavaScript一书 。
如果你只是JavaScript的初学者，想按部就班提高，避免掉入常见的陷阱，请花点时间看看 JavaScript Garden。

<!-- more -->

# 深入理解JavaScript事件

## JavaScript异步事件模型

### 事件的调度

如果想让JS中的某段代码将来再运行，可以将它放在回调中。

回调就是一种普通函数，只不过它是传给像`setTimeout`这样的函数，或者绑定为像`document.onready这样的属性。运行回调时，我们称已触发某事件（譬如延时结束或页面加载完毕）。

```javascript
for ( var i = 1; i <= 3; i++) {
setTimeout( function(){ console.log(i); }, 0);
};
4
4
4
```

JavaScript事件处理器在线程空闲之前不会运行。

### 线程的阻塞

### 队列

调用 setTimeout 的时候，会有一个延时事件排入队列。然后setTimeout调用之后的那行代码运行，接着是再下一行代码，直到再也没有任何代码。这时JavaScript虚拟机才会问：“队列里都有谁啊？”

如果队列中至少有一个事件适合于“触发”，则虚拟机会挑选一个事件，并调用此事件的处理器。事件处理器返回后，我们又回到队列处。

输入事件的工作方式完全一样：用户单击一个已附加有单击事件处理器的DOM（Document Object Model ，文档对象模型）元素时，会有一个单击事件排入队列。但是，该单击事件处理器要等到当前所有正在运行的代码均已结束后（可能还要等其他此前已排队的事件也依次结束）才会执行。因此，使用 JavaScript的那些网页一不小心就会变得毫无反应。

## 异步函数的类型

JavaScript环境提供的异步函数通常可以分为两大类：I/O函数和计时函数。如果想在应用中定义复杂的异步行为，就要使用这两类异步函数作为基本的构造块。

console.log是异步的吗？
WebKit的`console.log`由于表现出异步行为而让很多开发者惊诧不已。在Chrome或Safari中，以下这段代码会在控制台记录
{foo:bar}。
```javascript
var obj = {};
console.log(obj);
obj.foo = 'bar ';
```
怎么会这样？ WebKit 的console.log 并没有立即拍摄对象快照，相反，它只存储了一个指向对象的引用，然后在代码返回事件队列时才去拍摄快照。
Node 的console.log 是另一回事，它是严格同步的，因此同样的代码输出的却为 {} 。

`setTimeout`和`setInterval`就是故意设计成慢吞吞的。事实上，HTML规范推行的延时/时隔的最小值就是4毫秒。

那么，如果需要更细粒度的计时，该怎么办呢？有些运行时环境提供了备选方案。
 
>
尽管这些计时函数是异步 JavaScript 混饭吃的家伙什儿，但永远不要忘记， setTimeout 和setInterval 就是些不精确的计时工具。在Node 中，如果只是想产生一个短时延迟，请使用 process.nextTick 。在浏览器端， 请尝试使用垫片技术（ shim ） ③ ： 在支持requestAnimationFrame 的浏览器中， 推荐使用requestAnimationFrame ；在不支持requestAnimationFrame 的浏览器中，则退而使用 setTimeout 。

### 何时称函数为异步的

调用一个函数时，程序只在该函数返回之后才能继续。

JS写手如果称一个函数为“异步”的，其意思是这个函数会导致将来再运行另一个函数，后者取自于事件队列（若后面这个函数是作为参数传递给前者的，则称其为回调函数，简称为回调）。

异步函数还涉及另一个术语，即非阻塞。非阻塞这个词强调了异步函数的高速度：异步MySQL数据库驱动程序做一个查询可能要花上一小时，但负责发送查询请求的那个函数却能以微秒级速度返回。这对于那些需要快速处理海量请求的网站服务器来说，绝对是个福音。

### 回调内抛出的错误

```javascript
setTimeoout(function A() {
     setTimeout(function B(){
          setTimeout(function C(){
               throw new Error('Something terrible has happend!');
          }, 0);
     }, 0);
), 0};
```

```
<<Error: Something terrible has happened!
     at Timer.C (/AsyncJS/nestedErrors.js:4:13)
```

因为运行C的时候，A和B并不在内存堆栈里。这3个函数都是从事件队列直接运行的。
基于同样的理由，***利用try/catch语句块并不能捕获从异步回调中抛出的错误。***下面进行演示：（这就是为什么throw很少用作回调内错误处理的正确工具。)

```javascript
try {
setTimeout( function() {
     throw new Error( 'Catch me if you can!');
}, 0);
} catch (e) {
console.error(e);
}
```

```
Uncaught Error: Catch me if you can!
     (anonymous function) @ VM192473:4
```

看到这里的问题了吗？这里的 try/catch 语句块只捕获 setTimeout函数自身内部发生的那些错误。因为 setTimeout 异步地运行其回调，所以即使延时设置为 0 ，回调抛出的错误也会直接流向应用程序的未捕获异常处理器。

Node.js对于捕获异步回调函数错误的方法是，它的回调几乎总是接受一个错误作为其首个参数，这样就允许回调自己来决定如何处理这个错误。

```javascript
var fs = require(' fs');
fs.readFile(' fhgwgdz.txt', function(err, data) {
if (err) {
return console.error(err);
};
console.log(data.toString(' utf8'));
});
```

可客户端JavaScript库的一致性要稍微差些，不过最常见的模式是，针对成败这两种情况各规定一个单独的回调。jQuery的Ajax方法就遵循了这个模式：

```javascript
$.get(' /data', {
success: successHandler,
failure: failureHandler
});
```

不管 API形态像什么，始终要记住的是，***只能在回调内部处理源于回调的异步错误。***

### 未捕获异常的处理

#### 在浏览器环境中

现代浏览器会在开发人员控制台显示那些未捕获的异常，接着返回事件队列。可以给`window.onerror`附加一个处理器。如果`window.onerror`处理器返回`true`，则能阻止浏览器的默认错误处理行为（直接忽略所有错误）。

#### 在Node.js环境中

在 Node 环境中， window.onerror 的类似物就是 process对象的uncaughtException 事件。正常情况下， Node应用会因未捕获的异常而立即退出。但只要至少还有一个 uncaughtException 事件处理器， Node 应用就会直接返回事件队列。

但是，自 Node 0.8.4 起，uncaughtException 事件就被废弃了。

>
对异常处理而言， uncaughtException 是一种非常粗暴的机制，
它在将来可能会被放弃……
请勿使用 uncaughtException ，而应使用Domain 对象。

Domain 对象又是什么？你可能会这样问。 Domain 对象是事件化对象，它将 throw 转化为'error' 事件。下面是一个例子。
```javascript
var myDomain = require(' domain').create();
myDomain.run( function() {
setTimeout( function() {
     throw new Error(' Listen to me!')
}, 50);
});
myDomain.on(' error', function(err) {
     console.log(' Error ignored!');
});
```
源于延时事件的 throw 只是简单地触发了 Domain 对象的错误处理器。

<<Error ignored!

不管在浏览器端还是服务端，全局的异常处理器都应被视作最后一根救命稻草，请仅在调试时才使用它。

补充：

因为本书当时出版时，nodejs的版本还不高。Domain对象仅作为一种实验性对象。
nodejs中处理回调函数的异常：http://blog.csdn.net/newjueqi/article/details/41278177
nodejs异常捕获的一些实践：http://www.uml.org.cn/AJAX/201407115.asp

### JavaScript的多线程性
JavaScript 中存在一种多线程性：可以孵化出 Worker进程。每个孵化出的进程都可以与其他进程交换数据，其限制等同于任何其他 I/O 进程。Worker 对象使得我们有可能利用多个内核，同时不会破坏 JavaScript 的游戏规则（代码不可能被中断；变量只有处于其作用域内部时才是可访问的）。

# 分布式事件

## Pub/Sub模式（发布/订阅模式）

Node的EventEmitter对象（事件发生器，其他对象可以继承它；Node中几乎所有的I/O源都是EventEmitter对象：文件流、HTTP服务器，甚至是应用进程本身）、Backbone的事件化模型和jQuery的自定义事件。在这些工具的帮助下，我们能解嵌套那些嵌套式回调，减少重复冗余，最终编写出易于理解的事件驱动型代码。

```javascript
PubSub = {handlers: {}};

PubSub.on = function(eventType, handler) {
if (!(eventType in this.handlers)) {
this.handlers[eventType] = [];
}
this.handlers[eventType].push(handler);
return this;
}
PubSub.emit = function(eventType) {
var handlerArgs = Array.prototype.slice.call(arguments, 1);
for ( var i = 0; i < this.handlers[eventType].length; i++) {
     this.handlers[eventType][i].apply(this, handlerArgs);
}
return this;
}
```

## 事件化模型

JavaScript确实没有一种每当对象变化时就触发事件的机制。因此，事件化模型要是想工作的话，必须要使用一些像Backbone.js的set/get这样的方法。

# Promise对象和Deferred对象

jQuery1.5所有Ajax函数（$.ajax、$.get及$.post）现在都会返回Promise对象。Promise对象代表一项有两种可能结果（成功或失败）的任务，它还持有多个回调，出现不同结果时会分别触发相应的回调。

jQuery1.4中的代码必须写成这样：
```javascript
$.get( '/mydata', {
success: onSuccess,
failure: onFailure,
always: onAlways
});
```
而到了jQuery1.5+：
```javascript
var promise = $.get( '/mydata');
promise.done(onSuccess);
promise.fail(onFailure);
promise.always(onAlways);
```

为什么非得在触发Ajax调用之后再附加回调呢？一言以蔽之：封装。如果Ajax调用要实现很多效果，那么仅由负责发出请求的那部分应用代码来处理所有这些效果，显然很拙劣。

不过使用 Promise 对象的最大优势仍然在于，它可以轻松从现有Promise 对象派生出新的 Promise 对象。我们可以要求代表着并行任务的两个 Promise 对象合并成一个 Promise 对象，由后者负责通知前面那些任务都已完成。也可以要求代表着任务系列中首任务的Promise 对象派生出一个能代表任务系列中末任务的 Promise 对象，这样后者就能知道这一系列任务是否均已完成。待会儿我们就会看到， Promise 对象天生就适合用来进行这些操作。

Deferred 是 Promise 的超集，它比 Promise 多了一项关键特性：***可以直接触发***。纯 Promise 实例只允许添加多个调用，而且必须由其他什么东西来触发这些调用。

***使用 resolve （执行）方法和 reject（拒绝）方法均可触发 Deferred对象。***

```javascript
var promptDeferred = new $.Deferred();//生成一个Deferred对象（Promise对象）
promptPromise = promptDeferred.promise();//生成一个不是Deferred的Promise对象
```

promptPromise 只是promptDeferred 对象的一个没有resolve /reject 方法的副本。我们把回调绑定至 Deferred 或其下辖的 Promise 并无不同，因为这两个对象本质上分享着同样的回调。它们也分享着同样的 state （返回的状态值为 "pending"、 "resolved"或 "rejected"）。这意味着，对同一个 Deferred 对象生成多个 Promise对象是毫无意义的。事实上， jQuery 给出的只不过是同一个对象。

```javascript
var promise1 = promptDeferred.promise();
var promise2 = promptDeferred.promise();
console.log(promise1 === promise2); // true
```
而且，对一个纯 Promise 对象再调用promise 方法，产生的只不过是
一个指向相同对象的引用。
```javascript
console.log(promise1 === promise1.promise()); // true
```
## 进度通知

```javascript
var nanowrimoing = $.Deferred();
var wordGoal = 5000;
nanowrimoing.progress( function(wordCount) {
var percentComplete = Math.floor(wordCount / wordGoal * 100);
$( '#indicator').text(percentComplete + '% complete' );
});
nanowrimoing.done (function(){
     $(' #indicator').text('Good job!');
});
$( '#document').on('keypress', function(){
var wordCount = $(this).val().split(/\s+/).length;
if (wordCount >= wordGoal) {
     nanowrimoing.resolve();
};
nanowrimoing.notify(wordCount);
});
```

Deferred 对象的notify （通知）调用会调用我们设定的 progress回调。就像 resolve 和reject 一样， notify 也能接受任意参数。请注意，一旦执行了 nanowrimoing 对象，则再作 nanowrimoing.notify 调用将不会有任何反应，这就像任何额外的 resolve 调用及reject 调用也会被直接无视一样。

Promise 对象接受3 种回调形式： done 、fail 和progress 。执行Promise 对象时，运行的是 done 回调；拒绝 Promise对象时，运行的是 fail 回调；对处于挂起状态的 Deferred 对象调用notify 时，运行的是 progress 回调。

## Promise对象的合并

Promise最终只有两种状态，已执行或已拒绝。Promise如此强大的一个主要原因是，它允许我们把任务当成布尔值来处理。

```javascrit
$.when($.post( '/1', data1), $.post('/2', data2)).then(onPosted, onFailure);
```

不过最好不要通过合并后的promise的回调函数来进行参数的处理，相反应该直接向那些传递至when方法的成员Promise对象附加回调来收集相应的结果。

```javascript
var serverData = {};
var getting1 = $.get(' /1').done( function(result) {serverData['1'] = result;});
var getting2 = $.get( '/2').done( function(result) {serverData['2'] = result;});
$.when(getting1, getting2).done( function() {
     // 获得的信息现在都已位于 serverData ……
});
```

## 管道连接未来

在 JavaScript 中常常无法便捷地执行一系列异步任务，一个主要原因是无法在第一个任务结束之前就向第二个任务附加处理器。

```javascript
var getPromise = $.get( '/query');
getPromise.done( function(data) {
     var postPromise = $.post( '/search', data);
});
// 现在我们想给 postPromise 附加处理器……
```

jQuery1.6为Promise对象新增pipe方法：
```javascript
var getPromise = $.get( '/query');
var postPromise = getPromise.pipe( function(data) {
return $.post( '/search', data);
});
```
假设有个 Promise 对象发出的进度通知表示成 0 与1 之间的某个数，则可以使用 pipe 方法生成一个完全相同的 Promise对象，但它发出的进度通知却转变成可读性更高的字符串。
```javascript
var promise2 = promise1.pipe(null, null, function(progress) {
     return Math.floor(progress * 100) + ' % complete';
});
```
pipe最多能接受3个参数，对应这done、fail和progress。

总的说来， pipe 的回调可以做以下两件事情。
* 如果pipe 回调返回的是 Promise 对象，则pipe 生成的那个 Promise对象会模仿这个 Promise 对象。
* 如果pipe 回调返回的是非 Promise 对象（值或空白），则 pipe 生成的那个 Promise 对象会立即因该赋值而执行、拒绝或得到通知，具体取决于调用 pipe 的那个初始Promise 对象刚刚发生了什么。

### jQuery与Promises/A的对比
jQuery 使用 resolve 作为fail 的反义词，而 Promises/A使用的是 fulfill 。在Promises/A 规范中， Promise 对象不管是已履行还是已失败，都称“已执行”。
在 jQuery 1.8 问世之前， jQuery 的then 方法只是一种可以同时调用done 、fail 和 progress 这 3 种回调的速写法，而 Promises/A 的 then在行为上更像是 jQuery 的pipe 。 jQuery 1.8 订正了这个问题，使得then 成为pipe 的同义词。

## 用Promise对象代替回调函数
```javascript
var timing = new $.Deferred();
setTimeout(timing.resolve, 500);
```
考虑到 API 可能会出错，我们还要写一个根据情况指向 resolve 或
reject 的回调函数。例如，我们会这样写 Node 风格的回调：
```javascript
var fileReading = new $.Deferred();
fs.readFile(filename, 'utf8', function(err) {
if (err) {
     fileReading.reject(err);
} else {
     fileReading.resolve(Array.prototype.slice.call(arguments, 1));
};
});
总这么写很麻烦，所以何不写一个工具函数以根据任何给定 Deferred
对象来生成 Node 风格的回调呢？
```javascript
deferredCallback = function(deferred) {
return function(err) {
if (err) {
     deferred.reject(err);
} else {
     deferred.resolve(Array.prototype.slice.call(arguments, 1));
};
     };
}
```

# Async.js的工作流控制

异步函数序列的运行`async.series`和异步函数的并行运行`async.parallel`，任务列表是静态的，一旦调用了他们，就再也不能增减任务了。而且任务处于黑箱状态，除非我们自行从任务内部派发更新信息。
只有两个选择，要么是完全没有并发性，要么是不受限制的并发性。这对文件 I/O 任务可是个大问题。如果要操作上千个文件，当然不想因按顺序操作而效率低下，但如果试着并行执行所有操作，又很可能会激怒操作系统。
Async.js 提供了一种可以解决上述所有问题的全能方法： async.queue。

async.queue 接受的参数有两个：一个是 worker（办事员）函数，而不是一个函数列表；一个是代表着 concurrency（并发度）的值，代表了办事员最多可同时处理的任务数。


# 异步的脚本加载

* 异步脚本加载技术，要尽量避免使用内联，如果非得用内联脚本，请勿试图对该脚本使用`defer/async`属性。

* 也请勿使用`document.write`

## defer（延迟）

`defer`属性规定是否对脚本执行进行延迟，直到页面加载为止。
如果您的脚本不会改变文档的内容（如documen.write），可将defer属性加入到`<script>`标签中，以便加快处理文档的速度。因为浏览器知道它将能够安全地读取文档的剩余部分而不用执行脚本，它将推迟对脚本的解释，直到文档已经显示给用户为止。

## async（异步）

无序执行脚本。可以在页面需要加入一些小部件脚本时使用。

在那些同时支持这两个属性的浏览器中， async会覆盖掉defer。由于defer有着更广泛的支持，而且具有 async的主要优势（允许在下载脚本的同时进行DOM的渲染），因 此我们建议尽量使用defer代替async。