title: 面向对象的程序设计——创建对象
date: 2016-07-14 19:23
tags: [面向对象,JavaScript]
categories: JavaScript
---

参考JavaScript高级程序设计（第3版） 第六章

先放总结图：

![总结](http://thumbsnap.com/i/r9JtabZ7.png?1008)

<!--more-->

## 属性类型

### 数据属性

数据属性包含一个数据值的位置。在这个位置可以读取和写入只。
数据属性有4个描述其行为的特性。

[[Configurable]]：**表示能否通过delete删除属性从而重新定义属性，能否修改属性的特性，或者能否把属性修改为访问器属性。默认为true。**
一旦把属性定义为不可配置，就不能在变回可配置。
[[Enumerable]]： 表示能否通过for-in循环返回属性。默认为true。
[[Writable]]：表示能否修改属性的值。默认true。
[[Value]]：包含这个属性的数据值。默认为undefined。

要修改属性默认的特性，必须使用ECMAScript5的`Object.defineProperty()`方法。

### 访问器属性

访问器属性包含以下4个特性：
[[Configurable]]：**表示能否通过delete删除属性从而重新定义属性，能否修改属性的特性，或者能否把属性修改为数据属性。默认为true。**
[[Enumerable]]：同上
[[Get]]：在读取属性时调用的函数，默认undefined
[[Set]]：在写入属性时调用的函数。默认undefined

要修改属性默认的特性，必须使用ECMAScript5的`Object.defineProperty()`方法。
在这个方法之前，要创建访问器属性，一般都是用两个非变准的方法：__defineGetter__()和__defineSetter__()。

Object.defineProperty有三个参数，操作的对象，对象的属性，和修改的内容。

```javascript
var book = {
_year: 2004,
edition: 1
};

book.__defineGetter__("year", function(){
return this._year;
});
book.__defineSetter__("year", function(newValue){
if (newValue > 2004) {
this._year = newValue;
this.edition += newValue - 2004;
}
});
book.year = 2005;
alert(book.edition); //2
```

## 定义多个属性

Object.defineProperties()

## 读取属性的特性

Object.getOwnPropertyDescriptor()

## 创建对象

### 最简单的方式 Object构造函数或对象字面量

最简单的创建对象的方式，就是创建一个对象实例，再给它赋值

var person = new Object();
person.name = "Nicholas";
person.sayName = function() {
     alert(this.name)
};

对象字面量就是这种创建方式：

var person = {
     name: 'Nicholas',
     sayName: function() {
     alert(this.name);
     };
};

虽然Object构造函数或对象字面量都可以用来创建单个对象，但这些方式有个明显的缺点：使用同一个接口创建很多对象，会产生大量的重复代码。

### 工厂模式

```javascript
function createPerson(name, age, job){
var o = new Object();
o.name = name;
o.age = age;
o.job = job;
o.sayName = function(){
alert(this.name);
};
return o;
}
var person1 = createPerson("Nicholas", 29, "Software Engineer");
var person2 = createPerson("Greg", 27, "Doctor");
```

工厂模式虽然解决了创建多个相似对象的问题，但却没有解决对象识别的问题，不能知道一个对象的类型。

### 构造函数模式

创建自定义的构造函数，从而定义自动以对象类型的属性和方法。

```javascript
function Person(name, age, job){
this.name = name;
this.age = age;
this.job = job;
this.sayName = function(){
alert(this.name);
};
}
var person1 = new Person("Nicholas", 29, "Software Engineer");
var person2 = new Person("Greg", 27, "Doctor");
```

与工厂模式的不同点在于：

1. 没有显式地创建对象

1. 直接将属性和方式赋给了this对象

1. 没有return语句

要创建Person的新实例，必须使用new操作符。会经历以下4个步骤：

1. 创建一个新对象

2. 将构造函数的作用域赋给新对象

3. 执行构造函数中的代码

4. 返回新对象

这种方法可以用constructor和instanceof来检测实例的对象类型。

另外，我们应该明白

1. 把构造函数当作函数

构造函数与其他函数唯一的区别就在于他们的调用方式。任何函数，只要通过new操作符来调用，那么它就可以作为构造函数。

2. 构造函数的问题

每个方法都要在每个实例上重新创建一遍。例如，person1和person2都有一个名为sayName()的方法，但那两个方法不是同一个Function的实例。
即，不同实例上的同名函数是不相等的。
在ECMAScript中，函数就是对象，那么每定义一个函数，也就是实例化了一个对象。

在这里，我们可以通过把函数定义转移到构造函数外部来解决这个问题。

```javascript
function Person(name, age, job){
this.name = name;
this.age = age;
this.job = job;
this.sayName = sayName;
}
function sayName(){
alert(this.name);
}
var person1 = new Person("Nicholas", 29, "Software Engineer");
var person2 = new Person("Greg", 27, "Doctor");
```

这样做的确解决了上面所说的问题，但新的问题是，这样在全局作用域定义的函数实际上只能被某个对象所定义，让全局作用域有点名不副实。如果对象需要定义很多方法，就要定义很多全局函数。那么自定义的引用类型就丝毫没有封装性可言了。所以引出了原型模式。

### 原型模式

#### 理解原型对象

![理解原型对象](https://app.yinxiang.com/shard/s31/res/0506d0fd-d5b6-4c7e-b362-c2b7dab01bb8)

无论什么时候，只要创建了一个新函数，就会根据一组特定的规则为该函数创建一个prototype属性，这个属性指向函数的原型对象。
就是函数自身会有一个prototype属性，指向该函数的原型对象。

在默认情况下，所有原型对象都会自动获得一个constructor（构造函数）属性，这个属性包含一个指向prototype属性所在函数的指针。

注意，这样的连接是存在于实例与构造函数的原型对象之间的。换句话说，实例与构造函数没有直接的关系。

如图是Person构造函数、Person的原型属性以及Person现有的两个实例之间的关系。

在这里，可以用Object.getPrototypeof()来检测实例与对象呀unxingzhijian的关系。

```javascript
Person.prototype.isPrototypeOf(person1); //true
```

ECMAScript5 增加了一个新方法，叫Object.getPrototypeOf()，这个返回[[Prototype]]的值。

当为对象实例添加一个属性时，这个属性就会屏蔽原型对象中保存的同名属性；换句话说，添加这个属性只会阻止我们访问原型中的那个属性，但不会修改那个属性。
使用detele操作符则可以完全删除实例属性，从而让我们能够重新访问原型中的属性。

原型模式的缺点是：

1. 省略了为构造函数传递初始化参数这一环节，导致所有实例都取得相同的值

1. 所有属性都被很多实例共享。

对于包含基本只的属性倒也说得过去，通过前面说的屏蔽原型中对应的值，可以取得独立的值。但是对于包含引用类型的属性来说，这个就是完全共享的了。

## 组合使用构造函数模式和原型模式

创建自定义类型的最常见方式，就是组合使用构造函数与原型模式。

**构造函数模式用于定义实例属性，而原型模式用于定义方法和共享的属性。**

最大限度地节省了内存，而且这种模式还支持向构造函数传递参数。

```javascript
function Person(name, age, job){
this.name = name;
this.age = age;
this.job = job;
this.friends = ["Shelby", "Court"];
}
Person.prototype = {
constructor : Person, //这里用字面量的方式重新定义原型对象 会导致constructor不再指向Person 会指向Object构造函数 所以这里要指回来
sayName : function(){
alert(this.name);
}
}
var person1 = new Person("Nicholas", 29, "Software Engineer");
var person2 = new Person("Greg", 27, "Doctor");
person1.friends.push("Van");
alert(person1.friends); //"Shelby,Count,Van"
alert(person2.friends); //"Shelby,Count"
alert(person1.friends === person2.friends); //false
alert(person1.sayName === person2.sayName); //true

```

这种构造函数与原型混成的模式。一般来说是用来定义引用类型的一种默认模式。

## 动态原型模式

```javascript
function Person(name, age, job){
//属性
this.name = name;
this.age = age;
this.job = job;
//方法
if (typeof this.sayName != "function"){ //这个判断语句 只需要判断 其中的任何一个属性或者方法即可 因为只在第一次进行声明
Person.prototype.sayName = function(){
alert(this.name);
};
}
}
var friend = new Person("Nicholas", 29, "Software Engineer");
friend.sayName();
```
这种方法，是为了把构造函数和原型相关的信息都封装在构造函数中。便于其他OO语言经验的开发人员进行理解。

`
使用动态原型模式时，不能使用对象字面量重写原型。
如果在已经创建了实例的情况下重写原型，那么就会切断现有实例与新原型之间的联系。

## 寄生构造函数模式

```javascript
function Person(name, age, job){
var o = new Object();
o.name = name;
o.age = age;
o.job = job;
o.sayName = function(){
alert(this.name);
};
return o;
}
var friend = new Person("Nicholas", 29, "Software Engineer");
friend.sayName(); //"Nicholas"
```

除了使用new操作符并把使用的包装函数叫做构造函数之外，这个模式跟工厂模式是一模一样的。
这个模式可以在特殊情况下来为对象创建构造函数。比如创建一个新的对象来继承Array的属性，并且写额外的方法。

不能依赖instanceof操作符来确定对象类型。

## 稳妥构造函数模式

