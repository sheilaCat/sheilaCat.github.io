title: ECMAScript6学习
date: 2016-09-09 22:17:54
tags: [JavaScript,ECMAScript6]
categories: JavaScript
---

# let和const命令

## let命令

1. let生命的变量仅在块级作用域内有效。

2. 不存在变量提升

3. 暂时性死区（temporal dead zone）

<!-- more -->

只要块级作用域内存在`let`命令，它所声明的变量就binding这个区域，不再受外部的影响。

```javascript
    var tmp = 123;
    if(true){
        tmp = 'abc'; // ReferenceError
        let tmp;
    }
```

ES6明确规定，如果区块中存在`let`和`const`命令，这个区块就对这些命令声明的变量从一开始就形成了封闭作用域。凡是在声明之前就使用这些变量，就会报错。

ES6规定暂时性死区和`let`、`const`语句不出现变量提升，主要是为了减少运行时错误，防止在变量声明前就使用这个变量，从而导致意料之外的行为。

4. 不允许重复声明

也不能在函数内部重新声明参数。

### 块级作用域

ES5只有全局作用域和函数作用域，没有块级作用域，这带来很多不合理的场景。

1. 内层变量可能会覆盖外层变量
2. 用来技术的循环变量泄漏为全局变量

`let`实际上为JavaScript新增了块级作用域。

### 块级作用域与函数声明

ES6规定，函数只能在顶层作用域和函数作用域之中声明，不能在块级作用域中声明。
但是，浏览器没有遵守这个规定。所以只会在严格模式下报错。

ES6引入了块级作用域，明确允许在块级作用域之中声明函数。并且规定，块级作用域之中，函数声明语句的行为类似于`let`，在块级作用域之外不可引用。

但与老代码兼容很有问题，所以ES6规定浏览器实现可以不遵守这样的规定，也就是在支持ES6的浏览器中依然会提前函数声明。

## const命令

`const`声明一个只读的常量。一旦声明，常量的值就不能改变。

`const`的作用域只能在声明的块级作用域内有效。它声明的常量也是不提升，同样存在暂时性死区。不可重复声明。

对于复合类型的变量，变量名不指向数据，而是指向数据所在的地址。const命令只是保证变量名指向的地址不变，并不保证该地址的数据不变，所以将一个对象声明为常量必须非常小心。

```javascript
const foo = {};
foo.prop = 123;

foo.prop
// 123

foo = {}; // TypeError: "foo" is read-only
```

如果真的想将对象冻结，应该使用Object.freeze方法。

```javascript
const foo = Object.freeze({});

// 常规模式时，下面一行不起作用；
// 严格模式时，该行会报错
foo.prop = 123;
```

### 全局对象的属性

ES6规定，`let`命令、`const`命令、`class`命令声明的全局变量，不属于全局对象的属性。也就是说，从ES6开始，全局变量将逐步与全局对象的属性脱钩。

# 变量的解构赋值

ES6允许按照一定模式，从数组和对象中提取值，对变量进行赋值（Destructuring）。

本质上，这种写法属于“模式匹配”，只要等号两边的模式相同，左边的变量就会被赋予对应的值。

任何部署了Iterator接口的对象，都可以进行解构赋值。

## 数组的解构赋值

解构赋值允许指定默认值。

注意，ES6内部使用严格相等运算符（===），判断一个位置是否有值。所以，如果一个数组成员不严格等于undefined，默认值是不会生效的。

```javascript
var [x = 1] = [undefined];
x // 1

var [x = 1] = [null];
x // null
```


## 对象的解构赋值

对象的解构与数组有一个重要的不同。数组的元素是按次序排列的，变量的取值由它的位置决定；而对象的属性没有次序，变量必须与属性同名，才能取到正确的值。

对象的解构赋值的内部机制，是先找到同名属性，然后再赋给对应的变量。真正被赋值的是后者，而不是前者。

```javascript
var { foo: foo, bar: bar } = { foo: 'aaa', bar: 'bbb'};
```

```javascript
var { foo: baz } = { foo: "aaa", bar: "bbb" };
baz // "aaa"
foo // error: foo is not defined
```

上面代码中，真正被赋值的是变量baz，而不是模式foo。

## 圆括号问题

解构赋值虽然很方便，但是解析起来并不容易。对于一个式子到底是模式，还是表达式，必须解析到（或解析不到）等号才能知道。

ES6的规则是，只要有可能导致解构的歧义，就不得使用圆括号。

1. 变量声明语句中，不能带有圆括号；

2. 函数参数中，模式不能带有圆括号。函数参数也属于变量声明；

3. 赋值语句中，不能将整个模式，或嵌套模式中的一层，放在圆括号之中。

可以使用圆括号的情况只有一种：赋值语句的非模式部分。

# 字符串的拓展

## 模板字符串

模板字符串（template string）是增强版的字符串，用反引号（`）标识。它可以当作普通字符串使用，也可以用来定义多行字符串，或者在字符串中嵌入变量。

```javascript
$('#result').append(`
  There are <b>${basket.count}</b> items
   in your basket, <em>${basket.onSale}</em>
  are on sale!
`);
```

如果使用模板字符串表示多行字符串，所有的空格和缩进都会被保留在输出之中。

如果你不想要这个换行，可以使用trim方法消除它。

```javascript
$('#list').html(`
<ul>
  <li>first</li>
  <li>second</li>
</ul>
`.trim());
```


## 标签模板（tagged template）

“标签”指的就是函数，紧跟在后面的模板字符串就是它的参数。

```javascript
var a = 5;
var b = 10;

tag`Hello ${ a + b } world ${ a * b }`;
// 等同于
tag(['Hello ', ' world ', ''], 15, 50);
```

tag函数的第一个参数是一个数组，该数组的成员是模板字符串中**那些没有变量替换的部分**，也就是说，变量替换只发生在数组的第一个成员与第二个成员之间、第二个成员与第三个成员之间，以此类推。

标签模板的一个重要应用，就是过滤HTML字符串，防止用户输入恶意内容。

```javascript
var message =
  SaferHTML`<p>${sender} has sent you a message.</p>`;

function SaferHTML(templateData) {
  var s = templateData[0];
  for (var i = 1; i < arguments.length; i++) {
    var arg = String(arguments[i]);

    // Escape special characters in the substitution.
    s += arg.replace(/&/g, "&amp;")
            .replace(/</g, "&lt;")
            .replace(/>/g, "&gt;");

    // Don't escape special characters in the template.
    s += templateData[i];
  }
  return s;
}
```

标签模板的另一个应用，就是多语言转换（国际化处理）。

## String.raw()

ES6还为原生的String对象，提供了一个`raw`方法。这个方法往往用来充当模板字符串的处理函数，返回一个斜杠都被转义（即斜杠前面再加一个斜杠）的字符串，对应于替换变量后的模板字符串。

如果原字符串的斜杠已经转移，那么不会做任何处理。

# 正则的拓展

## y修饰符

ES6为正则表达式添加了y修饰符，叫做粘结（sticky）修饰符。

y修饰符的作用域g修饰符类似，也是全局匹配，后一次匹配都从上一次匹配成功的下一个位置开始。不同之处在于，y修饰符确保必须从剩余的第一个位置开始。

y修饰符的设计本意，就是让头部匹配的标志`^`在全局匹配中都有效。

## sticky属性

与y修饰符相匹配，ES6的正则对象多了sticky属性，表示是否设置了y修饰符。

# 数值的拓展

## Number.isFinite() Number.isNaN()

这两个方法与传统的全局isFinite()和isNaN()的区别是，传统方法会先转为Number对象，然后在进行判断，所以这两个方法只对数值有效，非数值一律返回`false`。

## Number.parseInt() Number.parseFloat()

ES6将全局方法`parseInt()`和`parseFloat()`，移植到Number对象上，完全一致。减少全局性方法，使得语言逐步模块化。

## Number.isInteger()

在JS内部，整数和浮点数都是同样的储存方法。

## Number.EPSILON

ES6在Number对象上面，新增了一个极小的常量Number.EPSILON。

```javascript
Number.EPSILON
// 2.220446049250313e-16
Number.EPSILON.toFixed(20)
// '0.00000000000000022204'
```

为了浮点数的计算，设置一个误差范围。但是如果这个误差能够小于Number.EPSILON，我们就可以认为得到了正确结果。

```javascript
0.1 + 0.2
// 0.30000000000000004

0.1 + 0.2 - 0.3
// 5.551115123125783e-17

5.551115123125783e-17.toFixed(20)
// '0.00000000000000005551'

5.551115123125783e-17 < Number.EPSILON
// true
```

因此，Number.EPSILON的实质是一个可以接受的误差范围。

```javascript
function withinErrorMargin (left, right) {
  return Math.abs(left - right) < Number.EPSILON;
}
withinErrorMargin(0.1 + 0.2, 0.3)
// true
withinErrorMargin(0.2 + 0.2, 0.3)
// false
```

## 安全整数和Number.isSafeInteger()

JavaScript能够准确表示的整数范围在-2^53到2^53之间（不含两个端点），超过这个范围，无法精确表示这个值。

ES6引入了Number.MAX_SAFE_INTEGER和Number.MIN_SAFE_INTEGER这两个常量，用来表示这个范围的上下限。

Number.isSafeInteger()则是用来判断一个整数是否落在这个范围之内。

## Math对象的拓展

1. Math.trunc() 去除一个数的小数部分，返回整数部分。

2. Math.sign() 用来判断一个数是正数负数还是0。

3. Math.cbrt() 计算一个数的立方根

4. Math.clz32() JavaScript的整数使用32位二进制形式表示，该方法返回一个数的32位无符号整数形式有多少个前导0。

5. Math.imul() 返回两个数以32位带符号整数形式相乘的结果，返回的也是一个32位的带符号整数。

因为JavaScript有精度限制，超过2的53次方的值无法精确表示。对于那些很大的数的乘法，低位数值往往都是不精确的，`Math.imul`方法可以返回正确的低位数值。

6. Math.fround() 返回一个数的单精度浮点数形式。

对于整数来说，Math.fround方法返回结果不会有任何不同，区别主要是那些无法用64个二进制位精确表示的小数。这时，Math.fround方法会返回最接近这个小数的单精度浮点数。

...

# 函数的拓展

## 参数默认值的位置

通常情况下，定义了默认值的参数，应该是函数的尾参数。因为这样比较容易看出到底省略了哪些参数。如果非尾部的参数设置默认值，实际上这个参数是没法省略的。会报错。除非显式地输入`undefined`。

```javascript
// 例一
function f(x = 1, y) {
  return [x, y];
}

f() // [1, undefined]
f(2) // [2, undefined])
f(, 1) // 报错
f(undefined, 1) // [1, 1]

// 例二
function f(x, y = 5, z) {
  return [x, y, z];
}

f() // [undefined, 5, undefined]
f(1) // [1, 5, undefined]
f(1, ,2) // 报错
f(1, undefined, 2) // [1, 5, 2]
```


## 函数的length属性

指定了默认值以后，函数的`length`属性，将返回没有指定默认值的参数个数。

如果设置了默认值的参数不是尾参数，那么`length`属性也不再计入后面的参数了。

```javascript
(function (a = 0, b, c) {}).length // 0
(function (a, b = 1, c) {}).length // 1
```

## rest参数

ES6引入rest参数（...变量名），用于获取函数的多余参数，这样就不需要使用arguments对象了。rest参数搭配的变量是一个数组，该变量将多余的参数放入数组中。

注意，rest参数之后不能再有其他参数，否则会报错。

函数的`length`属性，不包括rest参数。

```javasxript
(function(a) {}).length  // 1
(function(...a) {}).length  // 0
(function(a, ...b) {}).length  // 1
```

## 拓展运算符

拓展运算符（spread）是三个点`...`，好比是rest参数的逆运算，将一个不数组转为用逗号分隔的参数序列。

## 箭头函数的使用注意点

箭头函数有几个使用注意点。

（1）函数体内的this对象，就是定义时所在的对象，而不是使用时所在的对象。

    this对象的指向是可变的，但是在箭头函数中，它是固定的。

```javascript
function foo() {
  setTimeout(() => {
    console.log('id:', this.id);
  }, 100);
}

var id = 21;

foo.call({ id: 42 });
// id: 42
```

`this`指向的固定化，实际上是因为箭头函数根本没有自己的`this`，导致内部的`this`就是外层代码块的`this`。正因为它没有`this`，所以也就不能用作构造函数。

```javascript
// ES6
function foo() {
  setTimeout(() => {
    console.log('id:', this.id);
  }, 100);
}

// ES5
function foo() {
  var _this = this;

  setTimeout(function () {
    console.log('id:', _this.id);
  }, 100);
}
```


（2）不可以当作构造函数，也就是说，不可以使用new命令，否则会抛出一个错误。

（3）不可以使用arguments对象，该对象在函数体内不存在。如果要用，可以用Rest参数代替。

（4）不可以使用yield命令，因此箭头函数不能用作Generator函数。

## 尾调用（Tail Call）优化

指的是在某个函数的最后一步是完全调用另一个函数，并且尾调用不会用到外层函数的变量。由于是函数的最后一步操作，所以不需要保留外层函数的调用帧，因为调用位置、内部变量等信息都不会再用到了，只要直接用内层函数的调用帧来取代外层函数的调用帧就行了。

```javascript
function f() {
  let m = 1;
  let n = 2;
  return g(m + n);
}
f();

// 等同于
function f() {
  return g(3);
}
f();

// 等同于
g(3);
```

## 尾递归

递归非常耗费内存，因为需要同时保存省钱上百个调用帧，很容易发生“栈溢出”错误。但对于尾递归来说，由于只存在一个调用帧，所以永远不会发生“栈溢出”的错误。

ES6第一次明确规定，所有ECMAScript的实现都必须部署尾递归优化。只要在ES6中使用尾递归，就不会发生栈溢出，相对节省内存。

# 对象的拓展

属性可以用简洁的表示法，属性名还可以用表达式。

## Object.is()

ES6提出`Same-value equality`同值相等算法，用来解决`==`自动转换数值类型和`===`NaN不等于自身，以及+0等于-0的缺点。

```javascript
Object.is(+0, -0) // false
Object.is(NaN, NaN) // true
```

## Object.assign()

该方法实行的是浅拷贝。只能拷贝属性的值，所以只能是拷贝简单类型。

对于嵌套的对象，Object.assign会直接替换掉同名属性。

```javascript
var target = { a: { b: 'c', d: 'e' } }
var source = { a: { b: 'hello' } }
Object.assign(target, source)
// { a: { b: 'hello' } }
```

该方法可以用来处理数组，但会把数组视为对象。

```javascript
Object.assign([1, 2, 3], [4, 5])
// [4, 5, 3]
```

该方法将数组视为属性名为0、1、2的对象，因此在对应的位置上直接进行了替换。

该方法拷贝的属性也是有限制的，只拷贝原对象的自身属性（不拷贝继承属性），也不拷贝不可枚举的属性。

## Object.getOwnPropertyDescriptors()

ES7的提案，提出这个方法，返回指定对象所有的自身属性（非继承属性）的描述对象。

这个方法主要是为了解决`Object.assign`无法正确拷贝get和set属性的问题，也就是它本身只能浅拷贝，拷贝到相应的值。

```javascript
const shallowMerge = (target, source) => Object.defineProperties(
  target,
  Object.getOwnPropertyDescriptors(source)
);
```

# Symbol

ES6引入了一种新的原始数据类型Symbol，表示独一无二的值。Symbol值通过`Symbol`函数生成，现在对象的属性名可以是两种类型，一种是原来就有的字符串，另一种就是新增的Symbol类型。凡是属性名属于Symbol类型都是独一无二的，不会与其他属性名发生冲突。

```javascript
// 没有参数的情况
var s1 = Symbol();
var s2 = Symbol();

s1 === s2 // false

// 有参数的情况
var s1 = Symbol("foo");
var s2 = Symbol("foo");

s1 === s2 // false
```

Symbol值不能与其他类型的值进行运算，可以显式地转换为字符串，可以转换为布尔值，但是不能转换为数值。

## 作为属性名的Symbol

Symbol值作为对象属性名时不能用点运算符。因为点运算符后面总是字符串，所以不能读取到Symbol作为标识所指代的值，所以实际指向了那个字符串对应的属性。

同理，在对象内部，使用Symbol值定义属性时，也必须放在`[]`之中。

## 魔术字符串

魔术字符串指的是，在代码之中多次出现、与代码形成强耦合的某一个具体的字符串或者数值。风格良好的代码，应该尽量消除魔术字符串，该由含义清晰的变量代替。

```javascript
var shapeType = {
  triangle: 'Triangle'
};

function getArea(shape, options) {
  var area = 0;
  switch (shape) {
    case shapeType.triangle:
      area = .5 * options.width * options.height;
      break;
  }
  return area;
}

getArea(shapeType.triangle, { width: 100, height: 100 });
```

如果仔细分析，可以发现shapeType.triangle等于哪个值并不重要，只要确保不会跟其他shapeType属性的值冲突即可。因此，这里就很适合改用Symbol值。

```javascript
const shapeType = {
  triangle: Symbol()
};
```

Symbol属性一般不会被遍历循环所取到，但用Object.getOwnPropertySymbols()会返回，`Reflect.ownKeys`方法也可以返回所有类型的键名。

由于以Symbol值作为名称的属性，不会被常规方法遍历得到，可以利用这个特性，为对象定义一些非私有的、但是又只希望用于内部的方法。

Symbol本身是一个公开的值。

Symbol.for方法会在在全局登记一个Symbol类型的key，然后会先查找对应的key，如果没有再创建一个新的，如果有则返回；Symbol.forKey则直接查找，如果没有返回undefined。

注意，Symbol.for为Symbol值登记的名字是全局环境的，可以在不同的iframe或者service worker中取到同一个值。

可以用来实现单例模式。

# Proxy和Reflect

## Proxy

Proxy用于修改某些操作的默认行为，等同于在语言层面做出修改，所以属于一种“元编程”（meta programming），即对编程语言进行编程。

# 二进制数组

`ArrayBuffer`对象代表原始的二进制数据，`TypedArray`视图用来读写简单类型的二进制数据，`DataView`视图用来读写复杂类型的二进制数据。

二进制数组并不是真正的数组，而是类似数组的对象。

这个接口的原始设计目的，与WebGL有关，浏览器与显卡之间的通信接口，为了满足js与显卡之间大量实时的数据交换，他们之间的数据通信必须是二进制的，而不是能传统的文本格式。

很多浏览器的API也用到了二进制数组：

* File API

* XMLHttpRequest

* Fetch API

* Canvas

* WebSockets

ArrayBuffer对象代表储存二进制数据的一段内存，不能直接读写，只能通过视图来读写。

视图的作用就是以指定的格式来解读二进制数据。

视图很像普通的数组，所有数组能用的方法都能在视图上面进行使用。

TypedArray视图无法正确解析大端字节序，DataView可以。

DataView视图提供更多操作选项，而且支持设定字节序。本来，在设计目的上，ArrayBuffer对象的各种TypedArray视图，是用来向网卡、声卡之类的本机设备传送数据，所以使用本机的字节序就可以了；而DataView视图的设计目的，是用来处理网络设备传来的数据，所以大端字节序或小端字节序是可以自行设定的。

如果一次读取两个或两个以上字节，就必须明确数据的存储方式，到底是小端字节序还是大端字节序。默认情况下，DataView的get方法使用大端字节序解读数据，如果需要使用小端字节序解读，必须在get方法的第二个参数指定true。

# Set和Map数据结构

## Set

Set类似于数组，但是区别是Set成员的值都是唯一的，没有重复的值。

这就提供了一种数组去重的方法：

```javascript
function dedupe(array) {
  return Array.from(new Set(array));
}

dedupe([1, 1, 2, 3]) // [1, 2, 3]
```

向Set加入值的时候，不会发生类型转换。Set内部判断两个值是否不同，使用的算法是`Same-value equality`。

在Set内部，两个NaN是相等的，两对象总是不相等的。
```javascript
let set = new Set();

set.add({});
set.size // 1

set.add({});
set.size // 2
```

Set的遍历顺序就是插入顺序。

## WeakSet

WeakSet与Set的区别是，前者的成员只能是对象。其次，他的对象都是弱引用，即垃圾回收机制不考虑WeakSet对该对象的引用，也就是说如果其他对象都不再引用该对象，那么垃圾回收机制会自动回收该对象所占用的内存。这意味着无法引用WeakSet的成员，因此它是不可遍历的。

因为成员是弱引用，随时可能消失，所以遍历机制无法保证成员的存在，所以不可遍历。

WeakSet的一个用处，是储存DOM节点，而不用担心这些节点从文档中移除时，会引发内存泄漏。

## Map

JavaScript的对象Object，本质上是键值对的集合（Hash结构），但是传统上只能用字符串当做键。这给它的使用带来了很大的限制。

Map的出现就是为了解决这个问题，它类似于对象，也是键值对的集合，但是“键”的范围不限于字符串，各种类型的值（包括对象）都可以当做键。也就是说，Object结构提供了字符串到值的对应，Map结构提供了值到值的对应。是一种更完善的Hash结构实现。

```javascript
var m = new Map([
  [true, 'foo'],
  ['true', 'bar']
]);

m.get(true) // 'foo'
m.get('true') // 'bar'
```


注意，只有对同一个对象的引用，Map结构才将其视为同一个键。这一点要非常小心。

```javascript
var map = new Map();

map.set(['a'], 555);
map.get(['a']) // undefined
```

同理，同样的值的两个实例，在Map结构中被视为两个键。

Map的键实际上是跟内存地址绑定的，只要内存地址不一样就视为两个键。这就解决了同名属性碰撞clash的问题，我们扩展别人的库时，如果使用对象作为键名就不用担心自己的属性便于原作者的属性同名。

如果Map的键是一个简单类型的值（数字、字符串、布尔值），则只要两个值严格相等，Map将其视为一个键，包括0和-0。另外，虽然NaN不严格相等于自身，但Map将其视为同一个键。

```javascript
let map = new Map();

map.set(NaN, 123);
map.get(NaN) // 123

map.set(-0, 123);
map.get(+0) // 123
```

注意，Map的遍历顺序就是插入顺序。

WeakMap也是只接受对象作为键名（null除外），而且键名所指向的对象，不计入垃圾回收机制。

# Iterator和for...of循环

## Iterator遍历器

JS中表示集合的数据结构主要是数组和对象，ES6又添加了Map和Set。这样就需要一种统一的接口机制，来处理所有不同的数据结果。

Iterator就是这样一种机制。它是一种接口，为各种不同的数据结构提供统一的访问机制。任何数据结构只要部署Iterator接口，就可以完成遍历操作。

Iterator的作用有三个：一是为各种数据结构，提供一个统一的、简便的访问接口；二是使得数据结构的成员能够按某种次序排列；三是ES6创造了一种新的遍历命令for...of循环，Iterator接口主要供for...of消费。

Iterator的遍历过程是这样的。

（1）创建一个指针对象，指向当前数据结构的起始位置。也就是说，遍历器对象本质上，就是一个指针对象。

（2）第一次调用指针对象的next方法，可以将指针指向数据结构的第一个成员。

（3）第二次调用指针对象的next方法，指针就指向数据结构的第二个成员。

（4）不断调用指针对象的next方法，直到它指向数据结构的结束位置。

每一次调用next方法，都会返回数据结构的当前成员的信息。具体来说，就是返回一个包含value和done两个属性的对象。其中，value属性是当前成员的值，done属性是一个布尔值，表示遍历是否结束。

当使用for...of循环遍历某种数据结构时，该循环会自动去寻找Iterator接口。

ES6规定，默认的Iterator接口部署在数据结构的Symbol.iterator属性，或者说，一个数据结构只要具有Symbol.iterator属性，就可以认为是可遍历的。

在ES6中，有三类数据结构原生具备Iterator接口：数组、某些类似数组的对象、Set和Map结构。

本质上，遍历器是一种线性处理，对于任何非线性的数据结构，部署遍历器接口，就等于部署一种线性转换。

通过遍历器实现指针结构：

```javascript
function Obj(value) {
  this.value = value;
  this.next = null;
}

Obj.prototype[Symbol.iterator] = function() {
  var iterator = {
    next: next
  };

  var current = this;

  function next() {
    if (current) {
      var value = current.value;
      current = current.next;
      return {
        done: false,
        value: value
      };
    } else {
      return {
        done: true
      };
    }
  }
  return iterator;
}

var one = new Obj(1);
var two = new Obj(2);
var three = new Obj(3);

one.next = two;
two.next = three;

for (var i of one){
  console.log(i);
}
// 1
// 2
// 3
```

# Generator

Generator函数是ES6提供的一种异步编程解决方案，语法行为与传统函数完全不同。

1. 它是一个状态机，封装了多个内部状态。

2. 一个遍历器对象生成函数，返回的遍历器对象，可以依次遍历Generator函数内部的每一个状态。

调用Genterator函数后，该函数并不执行，返回的也不是函数运行结果，而是一个指向内部状态的指针对象，也就是一个遍历器对象。

下一步，必须调用遍历器对象的next方法，使得指针移向下一个状态。也就是说，每次调用next方法，内部指针就从函数头部或上一次停下来的地方开始执行，直到遇到下一个yield或者return语句。

也就是说，Generator函数是分段执行的，yield语句是暂停执行的标记，而next是恢复执行。

Genterator函数也可以不用tield语句，这时就变成了一个单纯的暂缓执行函数。

另外yield语句不能用在普通函数中，否则会报错。

## 与Iterator接口的关系

任意一个对象的`Symbol.iterator`方法，等于该对象的遍历器生成函数，调用该函数会返回该对象的一个遍历器对象。

把Generator函数赋值给对象的`Symbol.iterator`属性，就可以让对象具有Iterator接口。

## next方法的参数

yield语句本身没有返回值，或者说总是返回undefined。next方法可以带一个参数，该参数就会被当做上一个yield语句的返回值。

## for...of循环

该循环可以自动遍历Generator函数生成的Iterator对象，且此时不再需要调用next方法。

需要注意的是，一旦next方法的返回对象的done属性为true，for...of循环就会终止，且不包含该返回对象，所以return语句返回的6，不会包括在for...of循环之中。

```javascript
function *foo() {
  yield 1;
  yield 2;
  yield 3;
  yield 4;
  yield 5;
  return 6;
}

for (let v of foo()) {
  console.log(v);
}
// 1 2 3 4 5
```

## Generator.prototype.throw

Generator函数返回的遍历器对象，都有一个throw方法，可以在函数体外抛出错误，然后在函数体内捕获。需要在函数内部部署try...catch。

另外，throw方法被捕获以后，会附带执行下一条yield语句。也就是说，会附带执行一次next方法。

另外，外部的throw命令与生成器函数的throw方法是无关的，两者互不影响。也就是说，throw命令抛出的错误不会影响到遍历器的状态。

```javascript
var gen = function* gen(){
  yield console.log('hello');
  yield console.log('world');
}

var g = gen();
g.next();

try {
  throw new Error();
} catch (e) {
  g.next();
}
// hello
// world
```

只要Generator函数内部部署了try...catch代码块，那么遍历器的throw方法抛出的错误，不影响下一次遍历。

```javascript
var gen = function* gen(){
  try {
    yield console.log('a');
  } catch (e) {
    // ...
  }
  yield console.log('b');
  yield console.log('c');
}

var g = gen();
g.next() // a
g.throw() // b
g.next() // c
```

Generator函数体外抛出的错误，可以在函数体内捕获；反过来，Generator函数体内抛出的错误，也可以被函数体外的catch捕获。

一旦Generator执行过程中抛出错误，且没有被内部捕获，就不会再执行下去了。如果此后还调用next方法，将返回一个value属性等于undefined、done属性等于true的对象，即JavaScript引擎认为这个Generator已经运行结束了。

## yield* 语句

如果在Generator函数内部，调用另一个Generator函数，默认情况下是没有效果的。

yield*语句的作用，就是用来在一个Generator函数里面执行另一个Generator函数。

```javascript
function* inner() {
  yield 'hello!';
}

function* outer1() {
  yield 'open';
  yield inner();
  yield 'close';
}

var gen = outer1()
gen.next().value // "open"
gen.next().value // 返回一个遍历器对象
gen.next().value // "close"

function* outer2() {
  yield 'open'
  yield* inner()
  yield 'close'
}

var gen = outer2()
gen.next().value // "open"
gen.next().value // "hello!"
gen.next().value // "close"
```

实际上，任何数据结构只要有Iterator接口，就可以被yield*遍历。

## Generator与状态机

Generator是实现状态机的最佳结构。

Generator本身就包含了一个状态信息，即目前是否处于暂停态，所以不用外部变量保存状态。

## Generator与协程(coroutine)

协程既可以用单线程实现，也可以用多线程来实现。区别是协程只有一个是运行状态，对于单线程是只有一个栈是运行状态，对于多线程是只有一个线程在执行，其他协程都处于暂停状态。

Generator函数被称为半协程semi-coroutine。意思是只有Generator函数的调用者，才能将函数的执行权还给他。如果是完全执行的，那么任何函数都可以让暂停的协程继续执行。

## Promise对象

Promise对象有以下两个特点。

（1）对象的状态不受外界影响。Promise对象代表一个异步操作，有三种状态：Pending（进行中）、Resolved（已完成，又称Fulfilled）和Rejected（已失败）。只有异步操作的结果，可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。这也是Promise这个名字的由来，它的英语意思就是“承诺”，表示其他手段无法改变。

（2）一旦状态改变，就不会再变，任何时候都可以得到这个结果。Promise对象的状态改变，只有两种可能：从Pending变为Resolved和从Pending变为Rejected。只要这两种情况发生，状态就凝固了，不会再变了，会一直保持这个结果。就算改变已经发生了，你再对Promise对象添加回调函数，也会立即得到这个结果。这与事件（Event）完全不同，事件的特点是，如果你错过了它，再去监听，是得不到结果的。

使用Promise对象实现一个AJAX：

```javacript
var getJSON = function(url) {
  var promise = new Promise(function(resolve, reject){
    var client = new XMLHttpRequest();
    client.open("GET", url);
    client.onreadystatechange = handler;
    client.responseType = "json";
    client.setRequestHeader("Accept", "application/json");
    client.send();

    function handler() {
      if (this.readyState !== 4) {
        return;
      }
      if (this.status === 200) {
        resolve(this.response);
      } else {
        reject(new Error(this.statusText));
      }
    };
  });

  return promise;
};

getJSON("/posts.json").then(function(json) {
  console.log('Contents: ' + json);
}, function(error) {
  console.error('出错了', error);
});
```

```javascript
promise.then(function(value) {
  // success
}, function(error) {
  // failure
});
```

## Promise.prototype.then()

then方法返回的是一个新的Promise实例（注意，不是原来那个Promise实例）。因此可以采用链式写法，即then方法后面再调用另一个then方法。

## Promise.prototype.catch()

Promise.prototype.catch方法是.then(null, rejection)的别名，用于指定发生错误时的回调函数。

```javascript
p.then((val) => console.log("fulfilled:", val))
  .catch((err) => console.log("rejected:", err));

// 等同于
p.then((val) => console.log("fulfilled:", val))
  .then(null, (err) => console.log("rejected:", err));
```

Promise对象的错误具有冒泡的性质，会一直向后传递，直到被捕获为止。

## Promise.all()

Promise.all方法的参数可以不是数组，但必须具有Iterator接口，且返回的每个成员都是Promise实例。

（1）只有p1、p2、p3的状态都变成fulfilled，p的状态才会变成fulfilled，此时p1、p2、p3的返回值组成一个数组，传递给p的回调函数。

（2）只要p1、p2、p3之中有一个被rejected，p的状态就变成rejected，此时第一个被reject的实例的返回值，会传递给p的回调函数。

```javascript
// 生成一个Promise对象的数组
var promises = [2, 3, 5, 7, 11, 13].map(function (id) {
  return getJSON("/post/" + id + ".json");
});

Promise.all(promises).then(function (posts) {
  // ...
}).catch(function(reason){
  // ...
});
```

## Promise.race()

只要p1、p2、p3之中有一个实例率先改变状态，p的状态就跟着改变。那个率先改变的Promise实例的返回值，就传递给p的回调函数。

# 异步操作和Async函数

ES6诞生以前，异步编程主要的方法：

* 回调函数

* 事件监听

* 发布/订阅

* Promise对象

## 回调函数

```javascript
fs.readFile('/etc/passwd', function (err, data) {
  if (err) throw err;
  console.log(data);
});
```

一个有趣的问题是，为什么Node.js约定，回调函数的第一个参数，必须是错误对象err（如果没有错误，该参数就是null）？原因是执行分成两段，在这两段之间抛出的错误，程序无法捕捉，只能当作参数，传入第二段。

## Generator函数的自动执行

Generator函数就是一个异步操作的容器，它的自动执行需要一种机制，当异步操作有了结果，能够自动交回执行权。

两种方法可以实现：

1. 回调函数。将异步操作包装成Thunk函数，在回调函数里面交回执行权。

2. Promise对象。将异步操作包装成Promise对象，用then方法交回执行权。

## async函数

ES7提供了`async`函数，使得异步操作变得更加方便。`async`函数就是Generator函数的语法糖。

1. 内置执行器

只要像普通函数一样调用了async函数，就能够自动执行。

2. 更好的语义

3. 更广的适用性。 co模块约定，yield命令后面只能是Thunk函数或Promise对象，而async函数的await命令后面，可以是Promise对象和原始类型的值（数值、字符串和布尔值，但这时等同于同步操作）。

4. 返回值是Promise。async函数的返回值是Promise对象，这比Generator函数的返回值是Iterator对象方便多了。你可以用then方法指定下一步的操作。

# Class

ES6提供了更接近传统语言的对象写法，引入了Class类的概念，作为对象的模版。

基本上，ES6的class可以看做只是一个语法糖。class写法只是让对象原型的写法更加清晰，更像面向对象编程的语法而已。

定义class的方法的时候，前面不需要加上function关键字，方法之间不需要逗号来进行分隔。

类的数据类型就是function，类本身就指向构造函数。

```javascript

class Point {
  // ...
}

typeof Point // "function"
Point === Point.prototype.constructor // true
```

类的所有方法都定义在类的prototype属性上面。

```javascript
class Point {
  constructor(){
    // ...
  }

  toString(){
    // ...
  }

  toValue(){
    // ...
  }
}

// 等同于

Point.prototype = {
  toString(){},
  toValue(){}
};
```

另外，类的内部所有定义的方法，都是不可枚举的。类的属性名，可以采用表达式。类的构造函数，不使用new是没法调用的，会报错。

与ES5一致，实例的属性除非显式地定义在其本身（即定义在this对象上），否则都是定义在原型上。

不存在变量提升。

类和模块的内部，默认就是严格模式，所以不需要使用`use strict`指定运行模式。只要你的代码写在类或模块之中，就只有严格模式可用。

## Class的继承

Class之间可以通过extends关键字实现继承，这比ES6通过修改原型链实现继承，要清晰和方便很多。

super关键字，用来表示父类的构造函数，用来新建父类的this对象。

子类必须在constructor方法中调用super方法，否则新建实例时会报错。这是因为子类没有自己的this对象，而是继承父类的this对象，然后对其进行加工。如果不调用super方法，子类就得不到this对象。

```javascript
class ColorPoint extends Point {
  constructor(x, y, color) {
    super(x, y); // 调用父类的constructor(x, y)
    this.color = color;
  }

  toString() {
    return this.color + ' ' + super.toString(); // 调用父类的toString()
  }
}
```

ES5的继承，实质是先创造子类的实例对象this，然后再将父类的方法添加到this上面（Parent.apply(this)）。ES6的继承机制完全不同，实质是先创造父类的实例对象this（所以必须先调用super方法），然后再用子类的构造函数修改this。

这意味着ES6可以自定义原生数据结构（比如Array、String等）的子类，这是ES5无法做到的。

如果子类没有定义constructor方法，这个方法会被默认添加，代码如下。也就是说，不管有没有显式定义，任何一个子类都有constructor方法。

```javascript
constructor(...args) {
  super(...args);
}
```

## 静态方法

static静态方法，不会被实例继承，而是直接通过类来调用。

```javascript
class Foo {
  static classMethod() {
    return 'hello';
  }
}

Foo.classMethod() // 'hello'

var foo = new Foo();
foo.classMethod()
// TypeError: foo.classMethod is not a function
```

父类的静态方法，可以被子类继承。

ES6明确规定，Class内部只有静态方法，没有静态属性。

# Module

ES6的Class只是面向对象编程的语法糖，升级了ES5的构造函数的原型链继承的写法，并没有解决模块化的问题。Module就是为了解决这个问题。

ES6的模块自动采用严格模式，不管你有没有在模块头部加上"use strict";。

严格模式主要有以下限制。

变量必须声明后再使用
函数的参数不能有同名属性，否则报错
不能使用with语句
不能对只读属性赋值，否则报错
不能使用前缀0表示八进制数，否则报错
不能删除不可删除的属性，否则报错
不能删除变量delete prop，会报错，只能删除属性delete global[prop]
eval不会在它的外层作用域引入变量
eval和arguments不能被重新赋值
arguments不会自动反映函数参数的变化
不能使用arguments.callee
不能使用arguments.caller
禁止this指向全局对象
不能使用fn.caller和fn.arguments获取函数调用的堆栈
增加了保留字（比如protected、static和interface）


#阅读书籍

[ECMAScript 6入门](http://es6.ruanyifeng.com/)