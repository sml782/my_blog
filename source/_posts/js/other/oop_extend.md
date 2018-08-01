---
title: JavaScript面向对象程序设计
date: 2018-07-02 23:01:02
tags: [ 面向对象, 继承 ]
categories: [ JS ]
published: true
---
**JavaScript面向对象程序设计**
本文会碰到的知识点：
原型、原型链、函数对象、普通对象、继承

读完本文，可以学到

* 面向对象的基本概念
* JavaScript对象属性
* 理解JavaScript中的函数对象与普通对象
* 理解`prototype`和`proto`
* 理解原型和原型链
* 详解原型链相关的Object方法
* 了解如何用ES5模拟类，以及各种方式的优缺点
* 了解如何用ES6实现面向对象

# 一、面向对象的基本概念

面向对象也即是OOP，Object Oriented Programming，是计算机的一种编程架构，OOP的基本原则是计算机是由子程序作用的单个或者多个对象组合而成，包含属性和方法的对象是类的实例，但是JavaScript中没有类的概念，而是直接使用对象来实现编程。
特性：

* 封装：能够将一个实体的信息、功能、响应都封装到一个单独对象中的特性。

由于JavaScript没有`public`、`private`、`protected`这些关键字，但是可以利用变量的作用域来模拟public和private封装特性

```js
var insObject = (function() {
  var _name = 'hello'; // private
  return {
    getName: function() { // public
      return _name;
    }
  }
})();

insObject._name; // undefined
insObject.getName(); // hello
```

这里只是实现了一个简单的版本，private比较好的实现方式可以参考深入理解`ES6 145页`,protected可以利用ES6的Symbol关键字来实现，这里不展开，有兴趣可以讨论

**继承**：在不改变源程序的基础上进行扩充，原功能得以保存，并且对子程序进行扩展，避免重复代码编写，后面的章节详细描述

# 二、JavaScript对象属性

想弄懂面向对象，是不是先看看对象是啥呢？
我们先看一个题目:

```js
[] + {}; // "[object Object]"
{} + []; // 0
```

解释：
在第一行中，`{}`出现在`+`操作符的表达式中，因此被翻译为一个实际的值（一个空`object`）。而`[]`被强制转换为"",因此`{}`也会被强制转换为一个`string:"[object Object]"`。 
但在第二行中，`{}`被翻译为一个独立的`{}`空代码块儿（它什么也不做）。块儿不需要分号来终结它们，所以这里缺少分号不是一个问题。最终，`+ []`是一个将`[]`明确强制转换 为`number`的表达式，而它的值是`0`。

## 2.1 属性

*对象的属性*

* `Object.prototype` Object 的原型对象，不是每个对象都有`prototype`属性
* `Object.prototype.proto` 不是标准方法，不鼓励使用，每个对象都有`proto`属性，但是由于浏览器实现方式的不同，`proto`属性在chrome、firefox中实现了，在IE中并不支持，替代的方法是`Object.getPrototypeOf()`
* `Object.prototype.constructor`：用于创建一个对象的原型，创建对象的构造函数

可能大家会有一个疑问，为什么上面那些属性要加上`prototype`
在chrome中打印一下`var a = { test: 'test' }`

*属性描述符*
数据属性：

| 特性名称 | 描述 | 默认值 |
| - | - | - |
| value | 属性的值 | undfined |
| writable | 是否可以修改属性的值，true表示可以，false表示不可以 | true |
| enumerable | 属性值是否可枚举，true表示可枚举for-in, false表示不可枚举 | true |
| configurable | 属性的特性是否可配置，表示能否通过delete删除属性后重新定义属性 | true |
例子:
![img](//sml-myoss.oss-cn-beijing.aliyuncs.com/blog/word_img/20180702231814.png)

访问器属性：

| 特性名称 | 描述 | 默认值 |
| - | - | - |
| set | 设置属性时调用的函数 | undefined |
| get | 写入属性时调用的函数 | undefined |
| configurable | 表示能否通过delete删除属性后重新定义属性 | true |
| enumerable | 表示能否通过for-in循环返回属性 | true |

访问器属性不能直接定义，一般是通过`Object.defineProperty()`方法来定义，但是这个方法只支持IE9+， 以前一般用两个非标准方法来实现`__defineGetter__()`和֖`__defineSetter__() `

例子：

```js
var book = { _year: 2004, edition: 1 };

Object.defineProperty(book, "year", {
  get: function(){
    return this._year;
  },
  set: function(newValue){
    if (newValue > 2004){
      this._year = newValue;
      this.edition += newValue - 2004;
    }
  }
});

book.year = 2005; 
alert(book.edition);
```

## 2.2 方法

* `Object.prototype.toString()` 返回对象的字符串表示
* `Object.prototype.hasOwnProperty()` 返回一个布尔值，表示某个对象是否含有指定的属性，而且此属性非原型链继承，也就是说不会检查原型链上的属性
* `Object.prototype.isPrototypeOf()` 返回一个布尔值，表示指定的对象是否在本对象的原型链中
* `Object.prototype.propertyIsEnumerable()` 判断指定属性是否可枚举
* `Object.prototype.watch()` 给对象的某个属性增加监听
* `Object.prototype.unwatch()` 移除对象某个属性的监听
* `Object.prototype.valueOf()` 返回指定对象的原始值
* 获取和设置属性
  * `Object.defineProperty` 定义单个属性
  * `Object.defineProperties` 定义多个属性
  * `Object.getOwnPropertyDescriptor` 获取属性
* `Object.assign()` 拷贝可枚举属性 （ES6新增）
* `Object.create()` 创建对象
* `Object.entries()` 返回一个包含由给定对象所有可枚举属性的属性名和属性值组成的 [属性名，属性值] 键值对的数组，数组中键值对的排列顺序和使用for…in循环遍历该对象时返回的顺序一致
* `Object.freeze()` 冻结一个对象，冻结指的是不能向这个对象添加新的属性，不能修改其已有属性的值，不能删除已有属性，以及不能修改该对象已有属性的可枚举性、可配置性、可写性。也就是说，这个对象永远是不可变的。该方法返回被冻结的对象
* `Object.getOwnPropertyNames()` 返回指定对象的属性名组成的数组
* `Object.getPrototypeOf` 返回该对象的原型
* `Object.is(value1, value2)` 判断两个值是否是同一个值 (ES6 新增)
* `Object.keys()` 返回一个由给定对象的所有可枚举自身属性的属性名组成的数组，数组中属性名的排列顺序和使用for-in循环遍历该对象时返回的顺序一致
* `Object.setPrototypeOf(obj, prototype)` 将一个指定的对象的原型设置为另一个对象或者null
* `Object.values` 返回一个包含指定对象所有的可枚举属性值的数组，数组中的值顺序和使用for…in循环遍历的顺序一样

## 2.3 应用

如何检测某个属性是否在对象中？

* in运算符，判断对象是否包含某个属性，会从对象的实例属性、继承属性里进行检测

```js
function Dogs(name) {
  this.name = name
}

function BigDogs(size) {
  this.size = size;
}

BigDogs.prototype = new Dogs();

var a = new BigDogs('big');

'size' in a;
'name' in a;
'age' in a;
```

* Object.hasOwnProperty()，判断一个对象是否有指定名称的属性，不会检查继承属性

```js
a.hasOwnProperty('size');
a.hasOwnProperty('name');
a.hasOwnProperty('age');
```

* `Object.propertyIsEnumerable()`，判断指定名称的属性是否为实例属性并且是可枚举的

```js
// es6
var a = Object.create({}, {
  name: {
    value: 'hello',
    enumerable: true,
  },
  age: {
    value: 11,
    enumerable: false,
  }
});

// es5
var b = {};
Object.defineProperties(b, {
  name: {
    value: 'hello',
    enumerable: true,
  },
  age: {
    value: 11,
    enumerable: false,
  } 
});

a.propertyIsEnumerable('name');
a.propertyIsEnumerable('age');
```

* 如何枚举对象的属性，并保证不同了浏览器中的行为是一致的？

`for/in` 语句，可以遍历可枚举的实例属性和继承属性

```js
var a = {
  supername: 'super hello',
  superage: 'super name',
}
var b = {};
Object.defineProperties(b, {
  name: {
    value: 'hello',
    enumerable: true,
  },
  age: {
    value: 11,
    enumerable: false,
  }
});

Object.setPrototypeOf(b, a); // 设置b的原型是a 等效的是b.__proto__ = a

for(pro in b) {
  console.log(pro); // name, supername, superage
}
```

* `Object.keys()`， 返回一个数组，内容是对象可枚举的实例属性名称

```js
var propertyArray = Object.keys(b); // name
```

* `Object.getOwnPropertyNames()`，返回一个数组，内容是对象所有实例属性，包括可枚举和不可枚举

```js
var propertyArray = Object.getOwnPropertyNames(b); // name, age
```

* 如何判断两个对象是否相等？

我只想说，这个问题说简单很简单，说复杂也挺复杂的[传送门](https://stackoverflow.com/questions/1068834/object-comparison-in-javascript)
我们看个简单版的

```js
function isEquivalent(a, b) {
  var aProps = Object.getOwnPropertyNames(a);
  var bProps = Object.getOwnPropertyNames(b);
  if (aProps.length != bProps.length){
    return false;
  }

  for (var i = 0; i < aProps.length; i++) {
    var propName = aProps[i];
    if (a[propName] !== b[propName]) {
      return false;
    }
  }
  return true;
}

// Outputs: true
console.log(isEquivalent({a:1},{a:1}));
```

上面这个函数还有啥问题呢

* 没有对传入参数进行校验，例如判断是否是NaN，或者是其他内置属性
* 没有判断传入对象的construct和prototype
* 时间算法复杂度是O(n2)

有同学可能会有疑问，能不能用`Object.is`，答案是否定的，`Object.is`简单来说就是在`===`的基础上特别处理了`NaN`，`+0`，`-0`，保证了`-0`和`+0`不相同，`Object.is(NaN, NaN)`返回`true`。

* 对象的深拷贝和浅拷贝

其实如果大家理解了上面的那些方法，是很容易写出深拷贝和浅拷贝的代码的，我们先看一下这两者的却别。
浅拷贝仅仅是复制引用，拷贝后`a === b`， 注意`Object.assign`方法实现的是浅复制（此处有深刻教训！！！）
深拷贝这是创建了一个新的对象，然后把旧的对象中的属性和方法拷贝到新的对象中，拷贝后 `a !== b`
深拷贝的实现由很多例子，例如`jQuery`的`extend`和`lodash中的cloneDeep`, clone。jQuery可以使用`$.extend(true, {}, ...)`来实现深拷贝, 但是jQuery无法复制JSON对象之外的对象，例如ES6引入的`Map`、`Set`等。而lodash加入的大量的代码来实现ES6新引入的标准对象

# 三、对象分为函数对象和普通对象

** 什么是函数对象和普通对象？**
`Object`、`Function`、`Array`、`Date`等js的内置对象都是函数对象

```js
function a1 () {}
const a2 = function () {}
const a3 = new Function();

const b1 = {};
const b2 = new Object();

const c1 = [];
const c2 = new Array();

const d1 = new a1();
const d2 = new b1(); // ????
const d3 = new c1(); // ????

typeof a1;
typeof a2;
typeof a3;

typeof b1;
typeof b2;

typeof c1;
typeof c2;

typeof d1;
```

上面两行报错的原因，是因为构造函数只能由函数来充当，而b1和c1不是Function的实例，所以不能充当构造器
** 但是只有Function的实例都是函数对象、其他的实例都是普通对象 **
我们延伸一下，在看个例子

```js
const e1 = function *(){};
const e2 = new e1();
// Uncaught TypeError: e1 is not a constructor
console.log(e1.constructor) // 是有值的。。。
// 规范里面就不能new
const e2 = e1();
```

`GeneratorFunction`是一个特殊的函数对象 
`e1.__proto__.__proto__ === Function.prototype`

`e1`的原型实际上是一个生成器函数`GeneratorFunction`，也就是说 
`e1.__proto__ === GeneratorFunction.prototype`

这行代码有问题么，啊哈哈哈，`GeneratorFunction`这个关键字主流的JavaScript还木有暴露出来，所以这个大家理解就好啦

虽然不能直接`new e1`
但是可以`new e1.constructor();`哈哈哈哈

# 四、理解prototype和proto

| 对象类型 | prototype | proto |
| - | - |
| 函数对象 | Yes | Yes |
| 普通对象 | No | Yes |

* 只有函数对象具有`prototype`这个属性
* `prototype`和`__proto__`都是js在定义一个对象时的预定义属性
* `prototype`是被实例的`__proto__`指向
* `__proto__`指向构造函数的`prototype`

```js
const a = function(){}
const b = {}

typeof a // function
typeof b // object

typeof a.prototype // object
typeof a.__proto__ // function

typeof b.prototype // undefined
typeof b.__proto__ // object

a.__proto__ === Function.prototype
b.__proto__ === Object.prototype
```

理解了`prototype`和`__proto__`之后，我们来看看之前一直说的为什么JavaScript里面都是对象

```js
const a = {}
const b = function () {}
const c = []
const d = new Date()

a.__proto__
a.__proto__ === Object.prototype

b.__proto__
b.__proto__ === Function.prototype

c.__proto__
c.__proto__ === Array.prototype

d.__proto__
d.__proto__ === Date.prototype

Object.prototype.__proto__ //null

Function.prototype.__proto__ === Object.prototype

Array.prototype.__proto__ === Object.prototype

Date.prototype.__proto__ === Object.prototype
```

延伸一个问题：如何判断一个变量是否是数组？

* typeof

我们上面已经解释了，这些都是普通对象，普通对象是没有`prototype`的，他们`typeof`的值都是`object`

```js
typeof []
typeof {}
```

从原型来看, 原理就是看Array是否在a的原型链中
a的原型链是 Array->Object

```js
const a = [];
Array.prototype.isPrototypeOf(obj);
```

* instanceof

```js
const a = [];
a instanceof Array
```

从构造函数入手，但是这个方法和上面的方法都有一问题，不同的框架中创建的数组不会相互共享其`prototype`属性
根据对象的class属性，跨原型调用`tostring`方法

```js
const a = [];
Object.prototype.toString.call(a);
// [Object Array]
```

ES5 中所有内置对象的[[Class]]属性的值是由规范定义的，但是 ES6 中已经没有了[[Class]]属性，取代它的是[[NativeBrand]]属性，这个大家有兴趣可以自行去查看规范 
原理：

1. 如果`this`的值为`undefined`,则返回`'[object Undefined]'`. 
2. 如果`this`的值为`null`,则返回`[object Null]`. 
3. 让`O`成为调用`ToObject(this)`的结果. 
4. 让`class`成为`O`的内部属性`[[Class]]`的值. 
5. 返回三个字符串`'[object '`, `'class'`, 以及 `']'`连接后的新字符串.

问题？这个一定是正确的么？不正确为啥？ 
提示ES6的`Symbol`属性
`Array.isArray()` 
部分浏览器中不兼容

# 五、理解原型与原型链

其实上一节中的`prototype`和`proto`就是为了构建原型链而存在的，之前也或多或少的说到了原型链这个概念。

看下面的代码:

```js
const Dogs = function(name) {
    this.name = name;
}

Dogs.prototype.getName = function() {
    return this.name
}

const sijing = new Dogs('sijing');
console.log(sijing);
console.log(sijing.getName());
```

这段代码的执行过程

1. 首先创建了一个构造函数`Dogs`，传入一个参数`name`，`Dogs.prototype`也会自动创建
2. 给对象`dogs`增加了一个方法 
3. 通过构造函数`Dogs`实例化了一个对象`sijing`
4. 输出`sijing`的值,可以看到`sijing`有两个值`name`和`proto`,其中`proto`指向`Dogs.prototype`
5. 执行`getName`方法时，在`sijing`中找不到这个方法，就会继续向着原型链继续往上找，也就是通过`proto`，然后就找到了`getName`方法。

这个过程实际上就是原型继承，实际上JavaScript的原型继承就是利用了`proto`并借助`prototype`来实现的。

```js
sijing.__proto__ === Function.prototype

Dogs.prototype // 指向什么
Dogs.prototype.__proto__ // 指向什么
Dogs.prototype.__proto__.__proto__ // 指向什么
```

上面例子中`getName`最终是查找到了，那么如果在原型链中一直没查找到，会怎么样？ 
例如`console.log(sijing.age)`

```js
sijing // 是一个对象可以继续
sijing.age // 不存在，继续
sijing.__proto__ // 是一个对象可以继续
sijing.__proto__.age // 不存在，继续
sijing.__proto__.__proto__ // 是个对象可以继续
sijing.__proto__.__proto__.age // 不存在，继续
sijing.__proto__.__proto__.__proto__ null，// 不是对象，到头啦
```

**原型链** 的概念其实不重要，重要的是要理解，简单来说，原型链就是利用原型让一个引用类型继承另一个应用类型的属性和方法。

还有三点需要注意的:

* 任何内置函数对象（类）本身的 `_proto_`都指向`Function`的原型对象；
* 除了`Object`的原型对象的`_proto_`指向`null`，其他所有内置函数对象的原型对象的`_proto_`都指向`object`。
* 所有构造函数的的`prototype`方法的`proto`都指向`Object.prototype`（除了….`Object.prototype`自身）

如果理解了上面这些内容，大家可以自行描述一下，构造函数、原型和实例之间的关系.

* 构造函数首字母必须大写，用来区分普通函数，内部使用`this`指针，指向要生成的实例对象，通过`new`来生成实例对象。 
* 实例就是通过`new`一个构造函数产生的对象，它有一个属性`[[prototype]]`指向原型 
* 原型中有一个属性`[[constructor]]`，指向构造函数

# 六、与原型链相关的方法

## 6.1 hasOwnProperty

`Object.hasOwnProperty()`返回一个布尔值，表示某个对象的实例是否含有指定的属性，而且此属性非原型链继承。用来判断属性是来自实例属性还是原型属性。类似还有`in`操作符，`in`操作符只要属性存在，不管实在实例中还是原型中，就会返回`true`。同时使用`in`和`hasOwnProperty`就可以判断属性是在原型中还是在实例中

```js
const Dogs = function (age) {
  this.age = age
}

Dogs.prototype.getAge = function() {
  return this.age;
}

const sijing = new Dogs(14);
sijing.hasOwnProperty('age');
```

## 6.2 isPrototypeOf

`Object.prototype.isPrototypeOf()`返回一个布尔值，表示指定的对象是否在本对象的原型链中

```js
const Dogs = function (age) {
  this.age = age
}

Dogs.prototype.getAge = function() {
  return this.age;
}

const sijing = new Dogs(11);
Object.prototype.isPrototypeOf(Dogs);
Dogs.prototype.isPrototypeOf(sijing);
```

## 6.3 getPrototypeOf

`Object.getPrototypeOf`返回该对象的原型

```js
const Dogs = function (age) {
  this.age = age
}

Dogs.prototype.getAge = function() {
  return this.age;
}

const sijing = new Dogs(11);
sijing.__proto__ === Object.getPrototypeOf(sijing) 
```

# 七、ES5 对象继承

## 7.1 原型继承

原型继承就是利用** 原型链 **来实现继承

```js
function SuperType() {
  this.supername = 'super';
}

SuperType.prototype.getSuperName= function(){
  return this.supername;
}

function SubType () {
  this.subname='subname';
}

SubType.prototype = new SuperType();

SubType.prototype.getSubName = function (){
  return this.subname;
}

var instance1 = new SubType();
console.log(instance1.getSubName());
console.log(instance1.getSuperName());
```

需要注意的地方： 
实现原型继承的时候不要使用对象字面量创建原型方法，因为这样做，会重写原型链。

```js
function SuperType() {
  this.supername = 'super';
}

SuperType.prototype.getSuperName= function(){
  return this.supername;
}

function SubType () {
  this.subname='subname';
}

SubType.prototype = new SuperType();

SubType.prototype =  {
  getSubName: function (){
    return this.subname;
  }
}

var instance1 = new SubType();
console.log(instance1.getSubName());
console.log(instance1.getSuperName()); // error
```

上面使用`SubType.prototype = {...}`之后，`SubType`的原型就是`Object`了，而不是`SuperType`了。

优点：原型定义的属性和方法可以复用
缺点：

1. 引用类型的原型属性会被所有实例共享 
2. 创建子对象时，不能向父对象的构造函数中传递参数

## 7.2 构造函数继承

```js
var a = {
  name: 'a',
};

var name = 'window';

var getName = function(){
  console.log(this.name);
}

getName() // window
getName.call(a) // a
```

执行`getName()`时，函数体的`this`指向`window`，而执行`getName.call(a)`时，函数体的`this`指向的是`a`对象，所以就可以理解啦。接下来我们看如何实现构造函数继承

```js
function SuperType () {
  this.colors = ['red', 'green'];
}

function SubType () {
  // 继承SuperType
  SuperType.call(this);
}

var instance1 = new SubType();
instance1.colors.push('blue'); 
console.log(instance1.colors); // red, green, blue

var instance2 = new SubType();
console.log(instance2.colors); // red, green
```

`SuperType.call(this)`这一行代码，实际上意思是在`SubType`的实例初始化过程中，调用了`SuperType`的构造函数，因此`SubType`的每个实例都有`colors`这个属性

优点：子对象可以传递参数给父对象。

```js
function SuperType(name) {
  this.name = name;
}
function SubType(name, age) {
  name = name || 'hello';
  SuperType.call(this, name);
  this.age = age;
}

var instance1 = new SubType('scofield', 28);
console.log(instance1.name); //
console.log(instance1.age); //
```

需要注意的地方是在调用父对象的构造函数之后，再给子类型中的定义属性，否则会被重写。

缺点：方法都需要在构造函数中定义，难以做到函数的复用，而且在父对象的原型上定义的方法，对于子类型是不可见的。 ??? 为什么不可见

```js
function SuperType(name) {
  this.name = name;
}

SuperType.prototype.getName = function() {
  return this.name;
}

SuperType.prototype.prefix = function() {
  return 'prefix';
}

function SubType(name) {
  SuperType.call(this, name);
}

var instance1 = new SubType('scofield');
console.log(instance1.name);
console.log(instance1.prefix);
console.log(instance1.getName()); // Uncaught TypeError: instance1.getName is not a function
```

## 7.3 组合式继承

**组合式继承** 顾名思义，就是组合两种模式实现JavaScript的继承，借助 **原型链** 和 **构造函数** 来实现。这样子在原型上定义方法实现了函数的复用，而且能够保证每个实例都有自己的属性。

```js
function SuperType (name) {
  this.name = name;
  this.con = [];
}

SuperType.prototype.getName = function() {
  return this.name;
}

function SubType (name, age) {
  SuperType.call(this, name);
  this.age = age;
}

SubType.prototype = new SuperType();
SubType.prototype.constructor = SubType;
SubType.prototype.getAge = function() {
  return this.age;
};

var instance1 = new SubType('li', 18);
instance1.con.push('test1');
console.log(instance1.con); // test1
console.log(instance1.getAge()); // 18
console.log(instance1.getName()); // li

var instance2 = new SubType('hang', 18);
console.log(instance1.con); // test1
console.log(instance1.getAge()); // 18
console.log(instance1.getName()); // hang
```

优点：弥补了 **原型继承** 和 **构造函数** 的缺点 
缺点：父类构造函数调用了两次

## 7.4 原型式继承

原型式继承并没有使用严格意义上的构造函数，借助原型可以基于已有的对象创建新的对象，例如：

```js
function createObject(o) {
  function newOrient () {};
  newOrient.prototype = o;
  return new newOrient();
}
```

简单来说`createObject`函数，对传入的`o`对象进行的一次浅拷贝。在ES5中新增加了一个方法`Object.create()`, 它的作用和`createObject`是一样的，但是只支持IE9+。

```js
var Dogs = {
  name: 'jingmao',
  age: 1
}

var BigDogs = Object.create(Dogs);
BigDogs.name= 'bigjingmao';
BigDogs.size = 'big';
console.log(BigDogs.age);
```

其中`Object.create`还支持传入第二个参数，参数与`Object.defineProperties()`方法的格式相同，并且会覆盖原型上的同名属性。

## 7.5 寄生式继承

**寄生式继承** 其实和 **原型式继承** 很类似，区别在于，**寄生式继承** 创建的一个函数把所有的事情做完了，例如给新的对象增加属性和方法。

```js
function createAnother(o) {
  var clone = Object.create(o);
  clone.size = 'big';
  return clone;
}

var Dogs = {
  name: 'jingmao',
  age: 1
}

var BigDogs = createAnother(Dogs);
console.log(BigDogs.size);
```

## 7.6 寄生组合式继承

到最后一个了，看看我们之前遗留的问题: 
**组合继承** 会调用两次父对象的构造函数，并且父类型的属性存在两组，一组在实例上，一组在SubType的原型上。解决这个问题的方法就是 **寄生组合式继承**。

```js
function inheritPrototype(subType, superType){ 
  // 继承父类的原型
  var prototype = Object.create(superType.prototype);
  // 重写被污染的construct
  prototype.constructor = subType; 
  // 重写子类的原型  
  subType.prototype = prototype; 
}
```

这个函数就是 **寄生组合式继承** 的最简单的实现方式

```js
function SuperType(name){
  this.name = name;
  this.colors = ["red", "blue", "green"];
}

SuperType.prototype.sayName = function(){
  alert(this.name);
};

function SubType(name, age) {
  SuperType.call(this, name);
  this.age = age;
}

inheritPrototype(SubType, SuperType);

SubType.prototype.sayAge = function(){ 
  alert(this.age);
};

var instance1 = new SubType('hello', 18);

instance1.__proto__.constructor == SubType
```

可以看到

1. 子类继承了父类的属性和方法，同时属性没有创建在原型链上，因此多个子类不会共享同一个属性。
2. 子类可以动态传递参数给父类
3. 父类构造函数只执行了一次

但是还有一个问题：
子类如果在原型上添加方法，必须要在继承之后添加，否则会覆盖原来原型上的方法。但是如果这两个类是已存在的类，就不行了

优化一下：

```js
function inheritPrototype(subType, superType){ 
  // 继承父类的原型
  var prototype = Object.create(superType.prototype);
  // 重写被污染的construct
  prototype.constructor = subType; 
  // 重写子类的原型  
  subType.prototype = Object.assign(prototype, subType.prototype); 
}
```

虽然通过`Object.assign`来进行`copy`解决了覆盖原型类型的方法的问题，但是`Object.assign`只能够拷贝可枚举的方法，而且如果子类本身就继承了一个类，这个办法也不行。

# 八、ES6 实现继承

我们知道了ES5中可以通过原型链来实现继承，ES6提供了extends关键字来实现继承，这相对而言更加清晰和方便，首先看看ES6 Class的语法，此处参考[http://es6.ruanyifeng.com/#docs/class](http://es6.ruanyifeng.com/#docs/class)
