title: angular与MVVM框架
date: 2015-09-16 17:22:23
tags: [angularjs,mvvm]
---
本文从新人角度讲一讲对angular中MVVM模式的理解，以及angular特性的源码实现。
![img](https://angularjs.org/img/AngularJS-large.png)

<!--more-->

# MVVM核心原理

MVVM模式是`Model-View-ViewMode`（模型-视图-视图模型）模式的简称，其最早出现在微软的WPF和Silverlight框架中。MVVM模式利用框架内置的双向绑定技术对MVP（Model-View-Presenter）模式的变型，引入了专门的ViewModel（视图模型）来实现View和Model的粘合，让View和Model的进一步分离和解耦。

主要思想其实也很简单：在ViewModel中构建一组状态数据（state data），作为View状态的抽象。然后通过双向数据绑定（data binding）使ViewModel中的状态数据（state data）与View中的显示状态（screen state）保持一致。这样，ViewModel中的展示逻辑只需要修改对应的状态数据，就可以控制View的状态，从而避免在View上开发大量的接口。

![img](http://thumbsnap.com/i/aUxBOggL.png?0804)

MVVM模式的优势有如下四点：
* 低耦合：View可以独立于Model变化和修改，同一个ViewModel可以被多个View复用；并且可以做到View和Model的变化互不影响；
* 可重用性：可以把一些视图的逻辑放在ViewModel，让多个View复用；
* 独立开发：开发人员可以专注与业务逻辑和数据的开发（ViewModel），界面设计人员可以专注于UI(View)的设计；
* 可测试性：清晰的View分层，使得针对表现层业务逻辑的测试更容易，更简单。

# angular中的MVVM模式

Igor Minar[发布在Google+的文章中](https://plus.google.com/+IgorMinar/posts/DRUAkZmXjNV)提到：
>I’d rather see developers build kick-ass apps that are well-designed and follow separation of concerns, than see them waste time arguing about MV* nonsense. And for this reason, I hereby declare AngularJS to be MVW framework – Model-View-Whatever. Where Whatever stands for “whatever works for you”.

在文中特别指出angular在多次的API重构和改善，它越来越接近于MVVM模式，$scope可以被认为是ViewModel，而Controller则是装饰、加工处理这个ViewModel的JavaScript函数。作者更希望大家关注于实现一个成功的，具有好的设计以及遵循“分离关注点”原则的应用程序，而不是去争论MV*，所以他将angular称为MVW框架，是什么并不重要，只要适合你的应用就行。

下图是angular中关于MVVM模式的运用：
![img](http://thumbsnap.com/i/VeSX9sNM.png?0804)

在angular中MVVM模式主要分为四部分：

* View：它专注于界面的显示和渲染，在angular中则是包含一堆**声明式Directive**的视图模板。
* ViewModel：它是View和Model的粘合体，负责View和Model的交互和协作，它负责给View提供显示的数据，以及提供了View中Command事件操作Model的途径；**在angular中$scope对象充当了这个ViewModel的角色**；
* Model：它是与应用程序的业务逻辑相关的数据的封装载体，它是业务领域的对象，Model并不关心会被如何显示或操作，所以模型也不会包含任何界面显示相关的逻辑。在web页面中，大部分Model都是来自Ajax的服务端返回数据或者是全局的配置对象；而angular中的service则是封装和处理这些与Model相关的业务逻辑的场所，这类的业务服务是可以被多个Controller或者其他service复用的领域服务。
* Controller：这并不是MVVM模式的核心元素，但它负责ViewModel对象的初始化，它将组合一个或者多个service来获取业务领域Model放在ViewModel对象上，使得应用界面在启动加载的时候达到一种可用的状态。

# 源码分析

AngularJS通过使用自己的事件处理循环，改变了传统的Javascript工作流。这使得Javascript的执行被分成原始部分和拥有AngularJS执行上下文的部分。只有在AngularJS执行上下文中运行的操作，才能享受到AngularJS提供的数据绑定，异常处理，资源管理等功能和服务。

angular中关于源码的理解可按下图来进行学习，这里只总结几个比较重要的特性实现。

![img](http://thumbsnap.com/i/rfbxp8AT.png?0804)

## $compile

在angular中，指令的**编译链接、双向数据绑定、各种监听**等都是通过`$compile`来完成的。

`$compile`是通过编译HTML字符串或者DOM到模版里，产生一个`template function`，之后可以被用于`scope`和`template`的链接。

这个方法会遍历DOM并找到匹配的指令。一旦找到一个，它就会被加入一个指令列表中，这个列表是用来记录所有和当前DOM相关的指令的。 一旦所有的指令都被确定了，会按照优先级被排序，并且他们的compile方法会被调用。 指令的$compile()函数能修改DOM结构，并且要负责生成一个link函数。$compile方法最后返回一个合并起来的链接函数，这是链接函数是每一个指令的compile函数返回的链接函数的集合。

通过调用上一步所说的链接函数来将模板与作用域链接起来。这会轮流调用每一个指令的链接函数，让每一个指令都能对DOM注册监听事件，和建立对作用域的的监听。这样最后就形成了作用域的DOM的动态绑定。任何一个作用域的改变都会在DOM上体现出来。

```javascript
var $compile = ...; // injected into your code
var scope = ...;

var html = '<div ng-bind='exp'></div>';

// Step 1: parse HTML into DOM element
var template = angular.element(html);

// Step 2: compile the template
var linkFn = $compile(template);

// Step 3: link the compiled template with the scope.
linkFn(scope);
```

启动的方法在这里,只摘取关键代码.

```javascript
injector.invoke(['$rootScope', '$rootElement', '$compile', '$injector', '$animate',
       function(scope, element, compile, injector, animate) {
        scope.$apply(function() {
          element.data('$injector', injector);
          compile(element)(scope);
        });
      }]
    );
```

上面的代码主要作用就是，初始化相关的依赖,然后执行全局编译,最后更新所有的$watch.

核心的代码就这一句

```javascript
compile(element)(scope);
```
其实这里有两步
* `compile(element)` 收集完整个页面内的指令，然后返回publicLinkFn函数
* 执行`publicLinkFn(scope)` 此处的scope即为$rootScope

**使用`compile`函数可以改变原始的dom(template element),在ng创建原始dom实例以及创建scope实例之前。**
可以应用于当需要生成多个element实例,只有一个template element的情况,ng-repeat就是一个最好的例子,它就在是compile函数阶段改变原始的dom生成多个原始dom节点,然后每个又生成element实例.因为compile只会运行一次,所以当你需要生成多个element实例的时候是可以提高性能的.

更多可以参考[[译]ng指令中的compile与link函数解析](http://www.ifeenan.com/angularjs/2014-09-04-[%E8%AF%91]NG%E6%8C%87%E4%BB%A4%E4%B8%AD%E7%9A%84compile%E4%B8%8Elink%E5%87%BD%E6%95%B0%E8%A7%A3%E6%9E%90/)

## $digest
`$watch`存储了监听函数，当作用域里的变量发生变化时，调用`$digest`方法便会执行该作用域以及它的所有子作用域上的相关的监听函数，从而做一些操作（如：改变view）。

不过一般情况下，我们不需要手动调用`$digest`或者`$apply`（如果一定需要手动调用的话，我们通常使用`$apply`，因为它里面除了调用`$digest`还做了异常处理），因为内置的`directive`和`controller`内部（即Angular Context之内）都已经做了`$apply`操作，只有在Angular Context之外的情况需要手动触发`$digest`，如: 使用setTimout修改scope（这种情况我们除了手动调用`$digest`，更推荐使用`$timeout`服务，因为它内部会帮我们调用`$apply`）。

`digest`方法是`dirty check`的核心，也是双向绑定的主要实现，主要思路是先执行`$$asyncQueue`队列中的表达式，然后开启一个`loop`来的执行所有的`watch`里的监听函数，前提是前后两次的值是否不相等，假如`ttl`超过系统默认值，则`dirty check`结束，最后执行`$$postDigestQueue`队列里的表达式。

```javascript

      $digest: function() {
        var watch, value, last,
            watchers,
            length,
            dirty, ttl = TTL,
            next, current, target = this,
            watchLog = [],
            logIdx, logMsg, asyncTask;

        beginPhase('$digest');
        // Check for changes to browser url that happened in sync before the call to $digest
        $browser.$$checkUrlChange();

        if (this === $rootScope && applyAsyncId !== null) {
          // If this is the root scope, and $applyAsync has scheduled a deferred $apply(), then
          // cancel the scheduled $apply and flush the queue of expressions to be evaluated.
          $browser.defer.cancel(applyAsyncId);
          flushApplyAsync();
        }

        lastDirtyWatch = null;

        // 外层循环至少执行一次
        // 如果scope中被监听的变量一直有改变（dirty为true），那么外层循环会一直下去（TTL减1），这是为了防止监听函数有可能改变scope的情况，
        // 另外考虑到性能问题，如果TTL从默认值10减为0时，则会抛出异常
        do { // "while dirty" loop
          dirty = false;
          current = target;
         
          while (asyncQueue.length) {
            try {
              asyncTask = asyncQueue.shift();
              asyncTask.scope.$eval(asyncTask.expression, asyncTask.locals);
            } catch (e) {
              $exceptionHandler(e);
            }
            lastDirtyWatch = null;
          }
         
          traverseScopesLoop:
          do { // "traverse the scopes" loop
            if ((watchers = current.$$watchers)) {
              // process our watches
              length = watchers.length;
              while (length--) {
                try {
                  watch = watchers[length];
                  // Most common watches are on primitives, in which case we can short
                  // circuit it with === operator, only when === fails do we use .equals
                  if (watch) {
                    if ((value = watch.get(current)) !== (last = watch.last) &&
                        !(watch.eq
                            ? equals(value, last)
                            : (typeof value === 'number' && typeof last === 'number'
                               && isNaN(value) && isNaN(last)))) {
                      dirty = true;
                      lastDirtyWatch = watch;
                      watch.last = watch.eq ? copy(value, null) : value;
                      watch.fn(value, ((last === initWatchVal) ? value : last), current);
                      if (ttl < 5) {
                        logIdx = 4 - ttl;
                        if (!watchLog[logIdx]) watchLog[logIdx] = [];
                        watchLog[logIdx].push({
                          msg: isFunction(watch.exp) ? 'fn: ' + (watch.exp.name || watch.exp.toString()) : watch.exp,
                          newVal: value,
                          oldVal: last
                        });
                      }
                    } else if (watch === lastDirtyWatch) {
                      // If the most recently dirty watcher is now clean, short circuit since the remaining watchers
                      // have already been tested.
                      dirty = false;
                      break traverseScopesLoop;
                    }
                  }
                } catch (e) {
                  $exceptionHandler(e);
                }
              }
			}
```

通过上面的代码，可以看出,核心就是两个loop，外loop保证所有的model都能检测到,内loop则是真实的检测每个watch,watch.get就是计算监控表达式的值,这个用来跟旧值进行对比,假如不相等，则执行监听函数

注意这里的watch.eq这是是否深度检查的标识,equals方法是angular.js里的公共方法，用来深度对比两个对象,这里的不相等有一个例外，那就是NaN ===NaN,因为这个永远都是false,所以这里加了检查。

另外：`$RootScopeProvider`中提供了`digestTtl`方法，用于修改TTL的值（默认是10）,可以这样修改：
```javascript
angular.module('ng').config(['$rootScopeProvider', function ($RootScopeProvider) {
  $RootScopeProvider.digestTtl(20);
}]);
```

## isolate scope

Isolate标识来创建独立作用域，这个在创建指令并且scope属性定义的情况下，会触发这种情况，还有几种别的特殊情况，如果是独立作用域的话，会多一个$root属性，这个默认是指向rootscope的

如果不是独立的作用域，则会生成一个内部的构造函数，把此构造函数的prototype指向当前scope实例

## $injector

#### 依赖注入

每一个AngularJS应用都有一个注入器(injector)用来处理依赖的创建。注入器是一个负责查找和创建依赖的服务定位器。

```javascript
var FN_ARGS = /^function\s*[^\(]*\(\s*([^\)]*)\)/m;
var FN_ARG_SPLIT = /,/;
    // 获取服务名
var FN_ARG = /^\s*(_?)(\S+?)\1\s*$/;
var STRIP_COMMENTS = /((\/\/.*$)|(\/\*[\s\S]*?\*\/))/mg;
var $injectorMinErr = minErr('$injector');

function anonFn(fn) {
  // For anonymous functions, showing at the very least the function signature can help in
  // debugging.
  var fnText = fn.toString().replace(STRIP_COMMENTS, ''),
      args = fnText.match(FN_ARGS);
  if (args) {
    return 'function(' + (args[1] || '').replace(/[\s\r\n]+/, ' ') + ')';
  }
  return 'fn';
}

function annotate(fn, strictDi, name) {
  var $inject,
      fnText,
      argDecl,
      last;

  if (typeof fn === 'function') {
    if (!($inject = fn.$inject)) {
      $inject = [];
      if (fn.length) {
        if (strictDi) {
          if (!isString(name) || !name) {
            name = fn.name || anonFn(fn);
          }
          throw $injectorMinErr('strictdi',
            '{0} is not using explicit annotation and cannot be invoked in strict mode', name);
        }
        fnText = fn.toString().replace(STRIP_COMMENTS, '');
        argDecl = fnText.match(FN_ARGS);
        forEach(argDecl[1].split(FN_ARG_SPLIT), function(arg) {
          arg.replace(FN_ARG, function(all, underscore, name) {
            $inject.push(name);
          });
        });
      }
      fn.$inject = $inject;
    }
  } else if (isArray(fn)) {
    last = fn.length - 1;
    assertArgFn(fn[last], 'fn');
    $inject = fn.slice(0, last);
  } else {
    assertArgFn(fn, 'fn', true);
  }
  return $inject;
}
```
annotate函数通过对入参进行针对性分析，若传递的是一个函数，则依赖模块作为入参传递，此时可通过序列化函数进行正则匹配，获取依赖模块的名称并存入$inject数组中返回，另外，通过函数入参传递依赖的方式在严格模式下执行会抛出异常；第二种依赖传递则是通过数组的方式，数组的最后一个元素是需要使用依赖的函数。annotate函数最终返回解析的依赖名称。

# Angular优缺点及应用场景

angular功能全，利用它开发效率可以得到提高，有庞大的社区支持，没有内存泄露隐患，但是在性能上`dirty check`算是拖了后腿。

angular适合构建CRUD应用，因为它具有构建一个CRUD应用时可能用到的所有技术：数据绑定、基本模板指令、表单验证、路由、深度链接、组件重用、依赖注入。对于像游戏和有图形界面的编辑器之类的应用，会进行频繁且复杂的DOM操作，和CRUD应用不同。因此，可能不适合用Angular来构建。在这种场景下，使用更低抽象层次的类库可能会更好。

参考：

[浅析 MVC, MVP 与 MVVM之间的异同](http://wzhscript.com/2015/02/03/mvc-mvp-and-mvvm/)

[angular中的MVVM模式](http://www.cnblogs.com/whitewolf/p/4581254.htm)

[angularjs原理分析，及正确$apply的方法](http://hellohtml5.com/2014/10/16/how-angularjs-apply-works/)

[angularjs1.3.0源码解析之scope](http://www.cnblogs.com/lovesueee/p/4062247.html)

中文API：

http://docs.ngnice.com/#!/guide