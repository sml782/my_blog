---
title: JS数组专题1️⃣ ➖ 数组扁平化
description: 数组扁平化
date: 2018-07-31 16:30:18
tags: [ 数组, 数组扁平化 ]
categories: [ JS ]
published: false
---
# 一、什么是数组扁平化

1. 扁平化，顾名思义就是减少复杂性装饰，使其事物本身更简洁、简单，突出主题。
2. 数组扁平化，对着上面意思套也知道了，就是将一个复杂的嵌套多层的数组，一层一层的转化为层级较少或者只有一层的数组。

**Ps**: `flatten` 可以使数组扁平化，效果就会如下：

```js
const arr = [1, [2, [3, 4]]];
console.log(flatten(arr)); // [1, 2, 3, 4]
```

> 从中可以看出，使用 `flatten` 处理后的数组只有一层，下面我们来试着实现一下。

# 二、简单实现

## 2.1 普通递归

* 这是最容易想到的方法，简单，清晰！

```js
/* ES6 */
const flatten = (arr) => {
  let result = [];
  arr.forEach((item, i, arr) => {
    if (Array.isArray(item)) {
      result = result.concat(flatten(item));
    } else {
      result.push(arr[i])
    }
  })
  return result;
};

const arr = [1, [2, [3, 4]]];
console.log(flatten(arr));
```

```js
/* ES5 */
function flatten(arr) {
  var result = [];
  for (var i = 0, len = arr.length; i < len; i++) {
    if (Array.isArray(arr[i])) {
      result = result.concat(flatten(arr[i]))
    }
    else {
      result.push(arr[i])
    }
  }
  return result;
}

const arr = [1, [2, [3, 4]]];
console.log(flatten(arr));
```

## 2.2 `toString()`

* 该方法是利用 `toString` 把数组变成以逗号分隔的字符串，然后遍历数组把每一项再变回原来的类型。

先来看下 `toString` 是怎么把数组变成字符串的

```js
[1, [2, 3, [4]]].toString()
// "1,2,3,4"
```

完整的展示

```js
/* ES6 */
const flatten = (arr) => arr.toString().split(',').map((item) => +item);

const arr = [1, [2, [3, 4]]];
console.log(flatten(arr));
```

```js
/* ES5 */
function flatten(arr) {
  return arr.toString().split(',').map(function(item){
    return +item;
  });
}

const arr = [1, [2, [3, 4]]];
console.log(flatten(arr));
```

> 这种方法使用的场景却非常有限，必须数组中元素全部都是 `Number`。
> 也可以全部都为 `String`，具体实现大家自己体会。

## 2.3 `[].concat.apply` + `some`

* 利用 `arr.some` 判断当数组中还有数组的话，循环调用 `flatten` 扁平函数(利用 `[].concat.apply`扁平), 用 `concat` 连接，最终返回 `arr`;

```js
/* ES6 */
const flatten = (arr) => {
  while (arr.some(item => Array.isArray(item))){
    arr = [].concat.apply([], arr);
  }
  return arr;
}

const arr = [1, [2, [3, 4]]];
console.log(flatten(arr));
```

```js
/* ES5 */
/**
* 封装Array.some
* @param {function} callback    - 回调函数
* @param {any}      currentThis - 回调函数中this指向
*/
Array.prototype.some = function (callback, currentThis){
  let context = this;
  let flag = false;
  currentThis = currentThis || this;
  for (var i = 0, len = context.length; i < len; i++) {
    const res = callback.call(currentThis, context[i], i, context);
    if (res) {
      flag = true;
    } else if (!flag) {
      flag = false;
    }
  }
  return flag;
}

function flatten(arr){
  while(arr.some(item => Array.isArray(item))){
    arr = [].concat.apply([], arr);
  }
  return arr;
}

const arr = [1, [2, [3, 4]]];
console.log(flatten(arr));
```

## 2.4 `reduce`

* `reduce` 本身就是一个迭代循环器，通常用于累加，所以根据这一特点有以下：

```js
function flatten(arr){
  return arr.reduce(function(prev, cur){
    return prev.concat(Array.isArray(cur) ? flatten(cur) : cur)
  }, [])
}

const arr = [1, [2, [3, 4]]];
console.log(flatten(arr));
```

## 2.5 ES6 中的 解构运算符 `...`

* `...` 每次只能展开最外层的数组，被 `[].concat` 后，`arr` 就扁平化一次。

```js
function flatten(arr){
  while(arr.some(item => Array.isArray(item))){
    arr = [].concat(...arr);
  }
  return arr;
}

const arr = [1, [2, [3, 4]]];
console.log(flatten(arr));
```

**番外篇将给大家讲解 `lodash` 中 `flatten` 的实现源码，感谢大家阅读！**
