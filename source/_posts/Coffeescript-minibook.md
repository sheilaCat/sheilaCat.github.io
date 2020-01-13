title: 《Coffeescript小书》笔记
date: 2016-03-23 23:25:48
tags: Coffeescript
categories: Coffeescript
---
## 优点

* 简洁、充分地利用空格
* 有一些优雅的特性，比如列表解析、原型符号别名和类等等，能够有效减少输入
* 有原则地选择了一些JavaScript的特性，巧妙地避开了JavaScript的一些怪癖
* CoffeeScript不是JavaScript的超集

为什么CoffeeScript不是超集？阻止其成为超集最直接的原因是在CoffeeScript程序中空格是有意义的。

<!-- more -->

## 劣势

* 需要编译

CoffeeScript也在尝试尽力通过产生优雅的可读性强的JavaScript，以及在服务器端继承自动编译来弥补这个问题。

## 变量与作用域

CoffeeScript修复了JavaScript中一个最让人头疼的问题——去阿菊变量。在JavaScript中，一不小心，就很容易在定义变量时遗漏var关键字导致产生全局变量。CoffeeScript使用一个匿名函数把所有脚本都包裹起来，将其限定在局部作用域中，并且为所有的变量赋值前自动添加var。

CoffeeScript还更近了一步，让覆盖一个高一级的变量也很困难。这大量地减少了程序员常犯的错误。

然而，有时候全局变量还是有用的。你可以通过直接给全局对象（浏览器中的window）赋值来获得全局变量，也可以如下：

```javascript
     exports = this
     exports.MyVariable = "foo-bar"
```

## 函数

函数的最后一个表达式会作为隐式的返回值。但仍可使用return。

```javascript
func = -> "bar"
```

## 函数参数

```javascript
times = (a, b) -> a*b
```

支持默认参数

```javascript
times =(a = 1, b = 2) -> a*b
```

你还可以使用参数槽（splats）接收多个参数，使用...表示： 

```javascript
sum = (nums...) -> result = 0 nums.forEach (n) -> result += n result 
```

在上面的例子中，nums是一个包含传递给函数全部参数的数组。它不是一个arugments对 象，而是一个真实的数组对象，这样的话在你想操作它的时候就不 不需要先使用 Array.prototype.splice或者jQuery.makeArray()了。


## 函数调用

如果函数被至少一个参数跟着的话，CoffeeScript会自动地调用这个函数。

```javascript
a = "Howdy!"
alert a
alert(a)
```

最好加上括号，以免在多重调用中混淆。

## 函数上下文

在JavaScript上下文会频繁地变化。尤其是在回调函数中，CoffeeScript为此提供了一些辅助。其中之一就是->的变种胖箭头的函数=>

使用胖箭头代替普通箭头是为了确保函数的上下文可以绑定为当前的上下文。

```javascript
this.clickHandler = -> alert "clicked"
element.addEventListener "click", (e) => this.clickHandler(e)
```

## 对象字面量与数组定义

CoffeeScript在定义时可以省略括号，还可以使用缩进和换行来代替起分割作用的符号。

## 流程控制

在if和else的关键字中也可省略括号。

在单行的if语句中，你需要使用then关键字，这样CoffeeScript才能明白执行体从什么地方开始。

CoffeeScript并不支持条件运算符，作为替代应该使用单行的if/else语句。

CoffeeScript还支持一项Ruby的特性，即运行在if语句前使用前缀表达式。

可以用not关键字来代替感叹号。

可使用unless关键字，即if的否定。

is语句，===。

会把==、!=转换为===、!==。

![coffee](http://thumbsnap.com/i/wXqWKCfl.png?0323)

## 字符串插值法

在双引号的字符串中可以包含`#{}`标记，这些标记中可以包含被插入到字符串中的表达式。

```javascript
lala = "hello"
haha = "i can say #{lala}"
```

```javascript
var haha, lala;
lala = "hello";
haha = "i can say " + lala;
```

## 循环和列表解析

如果需要知道当前迭代索引，多加一个参数：

```javascript
for name, i in ["Roger the pickpocket", "Roderick the robber"]
 alert "#{i} - Release #{name}"
```

```javascript
var i, name, _i, _len, _ref;

_ref = ["Roger the pickpocket", "Roderick the robber"];
for (i = _i = 0, _len = _ref.length; _i < _len; i = ++_i) {
  name = _ref[i];
  alert("" + i + " - Release " + name);
}
```

可以过滤：
```javascript
prisoners = ["Roger", "Roderick", "Brian"]
release prisoner for prisoner in prisoners when prisoner[0] is "R"
```

可以使用推导式来迭代对象的全部属性，不过要使用of代替in关键字
```javascript
names = sam: seaborn, donna: moss
alert("#{first} #{last}") for first, last of names
```

## 数组

两个数字之间使用`..`或者`...`来分隔。前者包含最后一个位置，后者不包含。

然而，如果区间被指定到一个变量之后，CoffeeScript则会将其转换为一个`slice()`调用。

在JavaScript中检测数组中是否存在某个值是一件麻烦事，特别是indexOf()并不是所有的 浏览器都支持（IE，我说的就是你！）。CoffeeScript使用in操作符来解决这个问题，例 如：

```javascript 
words = ["rattled", "roudy", "rebbles", "ranks"]
 alert "Stop wagging me" if "ranks" in words
```

## 别名和存在操作符

`@`是`this`的别名。`::`是`prototype`的别名

CoffeeScript存在操作符`?`只会在变量为null或者undefined的时候返回真。

```javascript
praise if brian?
```

还能替换||操作符

如果在访问属性之前进行null检查：
```javascript
blackKnight.getLegs()?.kick()
```

可以用相同的方法检查一个属性是否是函数，是否可以调用。如果属性不存在或者不是一个函数，则就不会被调用。

```javascript
blackKnight.getLegs().kick?()
```
## 类

```javascirpt
class Animal
 constructor: (@name) ->

animal = new Animal("Parrot")
alert "Animal is a #{animal.name}"
```

```javascript
var Animal, animal;

Animal = (function() {
  function Animal(name) {
    this.name = name;
  }

  return Animal;

})();

animal = new Animal("Parrot");

alert("Animal is a " + animal.name);
```

## CoffeeScript惯用法

### Each
尽管forEach()语法非常简洁易读，但有个缺点是在每次数组迭代时都需要调用回调函数，因此它比等价的for循环要慢。

```javascript
myFunction(item) for item in array
```

但在CoffeeScript中，在这背后会被编译为for循环。也就是语法提供了便捷性，但是没有性能的耗损。

## CoffeeScript中没有修正的部分

CoffeeScript竭尽全力地解决了JavaScript设计上的一些缺陷，但不针对JavaScript的关键字提供一层抽象，因此很多缺陷依然带到了CoffeeScript中。

### 使用eval

### 使用typeof

typeof操作符是JavaScript最坑爹的设计，因为它完全就是鸡肋。事实上，它只有一个用途，就是检测一个值是否是undefined。

### 使用instanceof

JavaScript的instanceof关键字几乎就和typeof一样不给力。理想的instanceof将比较 两个对象的构造器，看其中一个是否另外一个的实例而返回真假值。实际上instanceof只 在比较自定义的对象上工作正常。如果用来比较内置类型时，就像typeof一样废。

### 使用delete

delete关键字只能用来移除对象内部的属性，用于其他地方，比如删除一个变量或者函数，就完全不行。
```javascript
aVar = 1
delete aVar
typeof Var # 'integer'
```
如果你想移除一个变量的引用，将其赋值为null即可

### 使用parseInt

使用parseInt函数的时候别忘了传递一个基数，否则会得到意外的值。

参考《Coffeescript小书》

















