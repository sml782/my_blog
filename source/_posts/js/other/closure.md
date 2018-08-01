---
title: JavaScript中的闭包
date: 2018-07-01 16:04:02
tags: [ 闭包 ]
categories: [ JS ]
published: true
---
# 作用域

先来说下什么是作用域，简单的说，作用域就是变量与函数的可访问范围，即作用域控制着变量与函数的可见性和生命周期。他减少了名称冲突，并且提供了自动内存管理。
在JavaScript中，变量的作用域有全局作用域和局部作用域两种。

## 全局作用域

```js
var num1 = 1;
function fun1 (){
  num2 = 2;
}
```

以上三个对象 `num1`, `num2` 和 `fun1` 均是全局作用域，这里要注意的是 **末定义直接赋值的变量自动声明为拥有全局作用域**；

## 局部作用域

```js
function wrap(){
  var obj = "我被wrap包裹起来了，wrap外部无法直接访问到我";
  function innerFun(){
      //外部无法访问我
  }
}
```

## 作用域链
当代码在一个环境中执行时，会创建变量对象的一个作用域链。
[{当前环境的变量对象}，{外层变量对象}，{外层的外层的变量对象}, {window全局变量对象}] 每个数组单元就是作用域链的一块，这个块就是我们的变量对象。
作用于链的前端，始终都是当前执行的代码所在环境的变量对象。全局执行环境的变量对象也始终都是链的最后一个对象。
```js
function foo(){
    var a = 12;
    fun(a);
    function fun(a){
        var b = 8;
        console.log(a + b);
    }
}  
foo();
```
再来看上面这个简单的例子，我们可以先思考一下，每个执行环境下的变量对象都是什么？ 这两个函数它们的变量对象分别都是什么？

我们以fun为例，当我们调用它时，会创建一个包含 `arguments`，`a`，`b`的 ** 活动对象 **，对于函数而言，在执行的最开始阶段它的活动对象里只包含一个变量，即`arguments`(当执行流进入，再创建其他的活动对象)。

在活动对象中，它依然表示当前参数集合。对于函数的活动对象，我们可以想象成两部分，一个是固定的`arguments`对象，另一部分是函数中的局部变量。而在此例中，a和b都被算入是局部变量中，即便a已经包含在了`arguments中`，但他还是属于。

有没有发现在环境栈中，所有的执行环境都可以组成相对应的作用域链。我们可以在环境栈中非常直观的拼接成一个相对作用域链。

下面我们大致说下这段代码的执行流程：
  1. 在创建`foo`的时候，作用域链已经预先包含了一个全局对象，并保存在内部属性`[[ Scope ]]`当中。
  2. 执行`foo`函数，创建执行环境与活动对象后，取出函数的内部属性`[[Scope]]`构建当前环境的作用域链(取出后，只有全局变量对象，然后此时追加了一个它自己的活动对象)。
  3. 执行过程中遇到了`fun`，从而继续对`fun`使用上一步的操作。
  4. `fun`执行结束，移出环境栈。`foo`因此也执行完毕，继续移出。
  5. javscript 监听到`foo`没有被任何变量所引用，开始实施垃圾回收机制，清空占用内存。

作用域链其实就是引用了当前执行环境的变量对象的指针列表，它只是引用，但不是包含，因为它的形状像链条，它的执行过程也非常符合，所以我们都称之为 ** 作用域链 **，而当我们弄懂了这其中的奥秘，就可以抛开这种形式上的束缚，从原理上出发。

# 闭包
闭包，官方对闭包的解释是：一个拥有许多变量和绑定了这些变量的环境的表达式（通常是一个函数），因而这些变量也是该表达式的一部分。
## 闭包的特点：
  1. 作为一个函数变量的一个引用，当函数返回时，其处于激活状态。
  2. 一个闭包就是当一个函数返回时，一个没有释放资源的栈区。
其实就是 ** 有权访问另一个函数作用域中的变量的函数 **。简单说就是，假设函数a是定义在函数b中的函数，那么函数a就是一个闭包。正常情况下，在函数的外部访问不到函数内部的变量，但有了闭包就可以间接的实现访问内部变量的需要。也就是说，** 闭包是连接函数内部和外部的桥梁 **。
## 闭包的作用
  1. 访问函数内部的变量。
  2. 让被引用的变量值始终保持在内存中。

```js
function fn1(){
    var a = 1;
    return function(){
        console.log(++a);
    }

}

var fn2 = fn1();

fn2();        //输出2

fn2();        //输出3
```
在这段代码中，fn1中的闭包函数被当作结果返回，在闭包中的引用的变量a因为被引用而没有被清除，一直保存在内存当中，所以执行fn2的时候会输出不断增加的结果：2和3。

当闭包中引用了函数中的变量时，那么，这个变量就会保存在内存中。也就是上面提到的闭包的第二个作用。之所以为这样，是因为JavaScript的回收机制。

基本所有浏览器都是使用“标记清除”的方式回收内存。也就是说，当变量进入执行环境的时候（在函数中声明一个变量），就给变量添加标记，而当函数执行完的，变量不再被引用的时候，再添加删除的标记，垃圾收集器就会自动清楚这个变量占有的内存。但在闭包中引用了函数中的变量，而闭包又被当作结果返回时，闭包中的因为被引用就不会被清除

## 闭包的用途

1. 匿名自执行函数

我们知道所有的变量，如果不加上var关键字，则默认的会添加到全局对象的属性上去，这样的临时变量加入全局对象有很多坏处，
比如：别的函数可能误用这些变量；造成全局对象过于庞大，影响访问速度(因为变量的取值是需要从原型链上遍历的)。
除了每次使用变量都是用`var`关键字外，我们在实际情况下经常遇到这样一种情况，即有的函数只需要执行一次，其内部变量无需维护，
比如UI的初始化，那么我们可以使用闭包：

```js
var data= {
  table : [],
  tree : {}
};

(function(dm){
  for(var i = 0; i < dm.table.rows; i++){
    var row = dm.table.rows[i];
    for(var j = 0; j < row.cells; i++){
      drawCell(i, j);
    }
  }
})(data);
```

我们创建了一个匿名的函数，并立即执行它，由于外部无法引用它内部的变量，因此在函数执行完后会立刻释放资源，关键是不污染全局对象。

2. 结果缓存

我们开发中会碰到很多情况，设想我们有一个处理过程很耗时的函数对象，每次调用都会花费很长时间，

那么我们就需要将计算出来的值存储起来，当调用这个函数的时候，首先在缓存中查找，如果找不到，则进行计算，然后更新缓存并返回值，如果找到了，直接返回查找到的值即可。闭包正是可以做到这一点，因为它不会释放外部的引用，从而函数内部的值可以得以保留。

```js
var CachedSearchBox = (function(){
  var cache = {},
      count = [];
  return {
    attachSearchBox : function(dsid){
      if(dsid in cache){ //如果结果在缓存中
        return cache[dsid]; //直接返回缓存中的对象
      }
      var fsb = new uikit.webctrl.SearchBox(dsid); //新建
      cache[dsid] = fsb; //更新缓存
      if(count.length > 100){ //保正缓存的大小<=100
        delete cache[count.shift()];
      }
      return fsb;
    },

    clearSearchBox : function(dsid){
      if(dsid in cache){
        cache[dsid].clearSelection();
      }
    }
  };
})();

CachedSearchBox.attachSearchBox("input");
```

这样我们在第二次调用的时候，就会从缓存中读取到该对象。

3. 封装

```js
var person = function(){
  //变量作用域为函数内部，外部无法访问
  var name = "default";

  return {
      getName : function(){
          return name;
      },
      setName : function(newName){
          name = newName;
      }
  }
}();

print(person.name);//直接访问，结果为undefined
print(person.getName());
person.setName("abruzzi");
print(person.getName());

// 得到结果如下：  

// undefined  
// default  
// abruzzi
```

4. 实现类和继承

```js
function Person(){
  var name = "default";

  return {
    getName : function(){
      return name;
    },
    setName : function(newName){
      name = newName;
    }
  }
};

var p = new Person();
p.setName("Tom");
alert(p.getName());

var Jack = function(){};
//继承自Person
Jack.prototype = new Person();
//添加私有方法
Jack.prototype.Say = function(){
  alert("Hello,my name is Jack");
};
var j = new Jack();
j.setName("Jack");
j.Say();
alert(j.getName());
```

我们定义了`Person`，它就像一个类，我们`new`一个`Person`对象，访问它的方法。

下面我们定义了`Jack`，继承`Person`，并添加自己的方法。