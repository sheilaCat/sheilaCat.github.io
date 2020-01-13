title: 《编写可维护的JavaScript》
date: 2016-03-23 22:13:07
tags: [JavaScript]
categories: JavaScript
---

## 基本的格式化

### 缩进层级

TAB或者2个空格、4个空格、8个空格。

建议使用4个空格为一个制表符。

<!-- more -->

### 语句结尾

有赖于分析器的自动分号插入（Automatic Semicolon Insertion，ASI）机制，JavaScript代码省略分号也是可以正常工作的。ASI会自动寻找代码中应当使用分号但实际没有分号的位置，并插入分号。大多数场景下ASI都会正确插入分号，不会产生错误。但ASI的分号插入规则非常复杂且很难记住，因此我推荐不要省略分号。

### 行的长度

80

### 换行

运算符后缩进，且增加两个层级的缩进。

逗号是一个运算符，应当作为前一行的行尾。这个换行位置非常重要，因为ASI机制会在某些场景下在行结束的位置插入分号。总是将一个运算符置于行尾，ASI就不会自作主张地插入分号，也就避免了错误的发生。

这个规则有个例外：给变量赋值时

```javascript
var result = something + anotherThing + yetAnotherThing + somethingElse +
                   anotherSomethingElse;
```

### 空行

* 在每个流控制语句之前（比如if和for语句）添加空行。

* 在方法之间。

* 在方法中的局部变量和第一条语句之间。

* 在多行或单行注释之前。

* 在方法内的逻辑片段之间插入空行，提高可读性。


### 变量和函数

命名采用驼峰命名法。变量的前缀是名词，函数前缀是动词。

### 常量

使用大写字母和下划线来命名，下划线用以分隔单词。

### 构造函数

大驼峰命名法。

### null

理解null最好的方式是将它当做对象的占位符（placeholder）。

### undefined

那些没有被初始化的变量都有一个初始值，即undefined，表示这个变量等待被赋值。

通过禁止使用特殊值undefined，可以有效地确保只在一种情况下typeof才会返回“undefined”：当变量未声明时。如果你使用了一个可能赋值为一个对象的变量时，则将其赋值为null。

将变量初始值赋值为null表明了这个变量的意图，它最终很可能赋值为对象。typeof运算符运算null的类型时返回“object”，这样就可以和undefined区分开了。

### 对象直接量

这种方式可以取代先显式地创建Object的实例，然后添加属性的这种做法。

```javascript
// 不好的写法
var book = new Object();
book.title = "Maintainable JavaScript;
book.author = "Nicholas C. Zakas";
// 好的写法
var book = {
     title: "Maintainable JavaScript,
     author: "Nicholas C. Zakas"
};
```

### 数组直接量

```javascript
// 不好的写法
var colors = new Array("red", "green", "blue");
// 好的做法
var colors = ["red", "green", "blue"];
```

## 注释

### 单行注释

建议：

斜杠后一个空格

单行注释前有一个空行

缩进到与下一行代码对齐

如果是代码尾部的注释，与代码之间要有一个缩进的距离

最好不要连续使用单行注释

### 多行注释

JAVA风格的多行注释：

```javascript
/*
* 这是一段注释
* 第二行
*/
```

代码尾部注释不要用多行注释格式

## 语句和表达式

### with语句

with语句用于暂修改作用域链，其语法为

```javascript
     with(object){
         statement 
     }
```
width的本意是用来减少键盘的输入的，比如
```javascript
obj.a = obj.b
obj.c = obj.d
```

可以简写成
```javascript
with(obj){
     a = b;
     c = d;
}
```

但是在实际运行时，解释器会首先判断obj.b和obj.d是否存在，如果不存在的话，再判断全局变量b和d是否存在。这样就导致了低效率，而且可能会导致意外，因此最好不要使用with语句。也就是无法知道变量b和d到底是局部变量还是obj的一个属性。实际上这种困惑对开发者的影响更甚，JavaScript引擎和压缩工具无法对这段代码进行优化，因为它们无法猜出代码的正确含义。

在严格模式中，with语句是被明确禁止的。

### for循环

for循环有两种更改循环执行过程的方法，第一种是break，第二种是continue。

Crockford的编程规范不允许使用continue。他主张代码中与其使用continue不如使用条件语句。更容易理解，且不易出错。可由当前代码可读性决定是否使用。

### for-in 循环

***for-in循环，不仅遍历对象的实例属性（instance property），同样还遍历从原型继承来的属性。***当遍历自定义对象的属性时，往往会因为意外的结果而终止。因此，最好使用`hasOwnProperty()`方法来为for-in循环过滤出实例属性。

通常for循环用来遍历数组成员，for-in循环用来遍历对象的属性。

***for-in循环是用来对实例对象和原型链中的键（key）做遍历的，而不是用来遍历包含数字索引的数组的。***

## 变量、函数和运算符

### 变量声明

推荐使用单var进行变量的声明。

### 函数声明

...

此外，函数声明不应当出现在语句块之内。
```javascript
// 不好的写法
if (condition) {
     function doSomething() {
          alert('Hi!');
     }
} else {
     function doSomething() {
         alert('Yo!'); 
     }
}
```

不论condition计算结果如何，大部分浏览器都会使用第二个函数。Firefox会根据计算结果选用合适的函数声明。
这种场景是ECMAScript的一个灰色地带，应当尽可能地避免。

### 严格模式

最好不要在全局作用域中使用"use strict"。

### eval()

在JavaScript中，`eval()`的参数是一个字符串，`eval()`会将传入的字符串当作代码来执行。开发者可以通过这个函数来载入外部的JavaScript代码，或者随即生成代码并执行它。

```javascript
eval("alert('Hi!')");
```

在JavaScript中，`eval()`并不是唯一可以执行JavaScript字符串的函数，使用Function构造函数、setTimeout()和setInterval也可以。

```javascript
var myfunc = new Function("alert('Hi!')");
setTimeout("document.body.style.background='red'", 50);
setInterval("document.title = 'It is now '" + (new Date()), 1000);
```

一个通用的原则是，严禁使用Function，并且只在别无他法时使用eval()。setTimeout()和setInterval也是可以使用的，但不要用字符串形式而要用函数。

# 编程实践

## 避免使用全局变量

### 全局变量带来的问题

#### 命名冲突

#### 代码的脆弱性

#### 难以测试

### 单全局变量

*YUI定义了唯一一个YUI全局对象。

*jQuery定义了两个全局对象，$和jQuery。只有在$被其他的类库使用了的情况下，为了避免冲突，应当使用jQuery。

单全局变量的意思是所创建的这个唯一全局对象名是独一无二的（不会和内置API产生冲突），并将你所有的功能代码都挂载到这个全局对象上，因此每个可能的全局变量都成为你唯一全局变量的属性，从而不会创建多个全局变量。

### 命名空间

### 模块

### 零全局变量

## 事件处理

### 规则1：隔离应用逻辑

应用逻辑是和应用相关的功能性代码，而不是和用户行为相关的。

将应用逻辑从所有事件处理程序中抽离出来的做法是一种最佳实践。

另一个缺点是和测试有关。测试时需要直接触发功能代码，而不必通过模拟对元素的点击来触发。

### 规则2：不要分发事件对象

最佳的办法是让事件处理程序使用event对象来处理事件，然后拿到所有需要的数据传给应用逻辑。

在处理事件时，最好让事件处理程序成为接触到event对象的唯一的函数。事件处理程序应当在进入应用逻辑之前针对event对象执行任何必要的操作，包括阻止默认事件或阻止事件冒泡，都应当直接包含在事件处理程序中。

例如

```javascript
var MyApplication = {
     handleClick: function(event) {
          event.preventDefault();
          event.stopPropagation();
     
          this.showPopup(event.clientX, event.clientY);
     },

     showPopup: function(x, y) {
          ...
     }
};

addListener(element, "click", function(event) {
     MyApplication.handleClick(event);
});
```

## 避免“空比较”

### 检查原始值

5种原型类型：字符串、数字、布尔值、null和undefined。

最佳选择是用typeof运算符。（实际上也只能准确检测undefined）

```javascript
var num1 = 1;
var num2 = new Number(2);

typeof num1; // "number"
typeof num2; // "object"

typeof null; // "object"
```

如果要检测null，则直接使用===/!==。


### 检测引用值

引用值也称作对象。在JavaScript中除了原始值之外的值都是引用。内置的引用类型：Object、Array、Date和Error。

instanceof，它不仅检测构造这个对象的构造器，还检测原型链。检测自定义类型最好能的做法也是用instanceof运算符。

### 检测函数

大部分情况下，用typeof，不用instanceof。instanceof不能“跨帧”。

### 检测数组

JavaScript中最古老的跨域问题之一就是在帧之间来回传递数组。instanceof Array在此场景中不总是返回正确的结果。每个帧都会有各自的Array构造函数，因此一个帧中的实例在另外一个帧里不会被识别。

鸭式辨型：

```javascript
// 采用鸭式辨型的方法检测数组
function isArray(value) {
     return typeof value.sort === "function";
}
```

这种检测方法依赖一个事实，就是数组是唯一包含sort()方法的对象。

最终，Kangax给出了一种优雅的解决方案：

```javascript
function isArray(value) {
     return Object.prototype.toString.call(value) === "[object Array]";
}
```

ECMAScript5将Array.isArray()正式引入JavaScript。

### 检测属性

因为存在IE8以及更早版本IE的情形（在调用DOM对象的hasOwnProperty方法之前应当先检测其是否存在），在判断实例对象是否存在时，更倾向于使用in运算符，只有在需要判断实例属性时才会用到hasOwnProperty()。

## 将配置数据从代码中分离出来

### 什么是配置数据

*URL

*需要展现给用户的字符串

*重复的值

*设置（比如每页的配置项）

*任何可能发生变更的值

### 抽离配置数据

### 保存配置数据

*java属性文件等进行保存

然后将这个文件转换为JavaScript可用的额文件：

*JSON

*JSONP

*纯JavaScript

## 抛出自定义错误

### try-catch语句

### 错误类型

#### Error

所有错误的基本类型。实际上引擎从来不会抛出该类型的错误。

#### EvalError

通过eval()函数执行代码发生错误时抛出。

#### RangeError

一个数字超过它的边界时抛出。

#### ReferenceError

期望的对象不存在时抛出。引用错误。

#### SyntaxError

给eval()函数传递的代码中有语法错误时。

#### TypeError

变量不是期望的类型时抛出。

#### URIError

给encodeURI()、encodeURIComponent()、decodeURI()或者decodeURIComponent()等函数传递格式非法的URI字符串时抛出。

# 不是你的对象不要动

## 原则

针对原生对象、DOM对象、浏览器对象模型（BOM）对象、类库对象

### 不覆盖方法

### 不新增方法

### 不删除方法

## 更好的途径

### 基于对象的继承

Object.create()

### 基于类型的继承

首先，原型继承；然后，构造器继承。

```javascript
function Person(name) {
     this.name;
}
function Author(name) {
     Person.call(this, name); //继承构造器
}

Author.prototype = new Person();
```

### 门面模式

门面模式，也叫做包装器。它为一个已存在的对象创建一个新的接口。

```javascript

function DOMWrapper(element) { 
     this.element = element;
}

DOMWrapper.prototype.addClass = function(className) {
     element.className += " " + className;
}

DOMWrapper.prototype.remove = function() {
     this.element.parentNode.removeChild(this.element);
}

var wrapper = new DOMWrapper(document.getElementById("my-div"));
wrapper.addClass("selected");
wrapper.remove();
```

### 关于Polyfill的注解

Polyfills也称为shims，是指一种功能的模拟，模拟一些新版本中新特性的原生功能并且要以完全兼容的方式实现。

从维护角度说，最好使用门面模式来代替polyfill。

### 阻止修改

三种锁定修改的级别：

#### 防止扩展

禁止添加，但可以删除和修改。

Object.preventExtension()，防止扩展。Object.isExtensible()，检测是否可拓展。

#### 密封

禁止添加和删除，但可以修改。

Object.seal()
Object.isSealed()

#### 冻结

禁止删除和添加、修改，只读模式。

Object.freeze()
Object.isFrozen()

以上三种模式中，当操作禁止行为时会静默失败，只有在严格模式下才会报错。

一旦一个对象被锁定了，将无法解锁。

## 浏览器嗅探

## 自动化













