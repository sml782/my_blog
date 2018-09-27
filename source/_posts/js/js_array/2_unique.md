---
title: JS数组专题2️⃣ ➖ 数组去重
description: 数组去重
date: 2018-09-27 10:15:18
tags: [ 数组, 数组去重 ]
categories: [ JS ]
published: true
---
> 距离上次发文，已经有一段时间了，最近工作比较忙，这不眼看快双十一了，就相当于给大家一些福利吧！

<div style="text-align: center">
![装逼图](https://img03.sogoucdn.com/app/a/100520021/C2FA8779E802FEF6C5454D5949F3AF44)
</div>

# 一、什么是数组去重

简单说就是把数组中重复的项删除掉，你 GET 到了吗 ？下面我将简单介绍下几种基本的方法及其优缺点。

# 二、方法汇总

- 两层循环

** 无相同值直接 `push` 进新数组，有相同的值则直接跳过本次内部循环 **

```js
/*
 * @param {Array} arr    -要去重的数组
 * @param {Array} result -初始化结果数组
 */
const unique = (arr, result = []) => {
  const len = arr.length;
  for (let i = 0; i < len; i++) {
    for (let j = i + 1; j < len; j++) {
      if (arr[i] === arr[j]) {
        // 相等则直接跳过
        j = ++i;
      }
    }
    result.push(arr[i]);
  }
  return result;
}
```

** 相同的做标记，与新数组作比较，没有则插入 **

```js
/*
 * @param {Array} arr    -要去重的数组
 * @param {Array} result -初始化结果数组
 */
const unique = (arr, result = []) => {
  result.push(arr[0]);
  const len = arr.length;
  let rLen = result.length;

  for (let i = 1; i < len; i++) {
    let flag = false;
    for (var j = 0; j < rLen; j++) {
      if (arr[i] === result[j]) {
        flag = true;
        break;
      }
    }
    if (!flag) {
      rLen++;
      result.push(arr[i]);
    }
  }
  return result;
}

```

** 原地算法(在数组本身操作) **

```js
const unique = arr => {
  const len = arr.length;
  for (let i = 0; i < len; i++) {
    for (let j = i + 1; j < len; j++) {
      if (arr[i] == arr[j]) {
        arr.splice(j,1);
        len--;
        j--;
      }
    }
  }
  return arr;
};
```

> 看似代码代码简单，实则内存占用高，不实用

- 单层循环

** 对象键不能重复 **

```js
const unique = (arr, result = []) => {
  const obj = {};
  const len = arr.length;
  for (let i = 0; i< len; i++) {
    if (!obj[arr[i]]) {
      // 键没有，则添加
      obj[arr[i]] = 1;
      result.push(arr[i]);
    }
  }
  return result;
};
```

> 这种方法无法判断 `'1'` 和 `1` 等类型，解决方案:
> 1. 添加判断数据类型，比如 `typeof` ，`obj[typeof arr[i] + arr[i]]` 不过这还是判断不了 `['1']` 和 `[1]`，因为这被相加后，结果都一样
> 2. 添加 `JSON.stringify()` 对结果进行去格式化，这时就可以判断了

** 排序后比较前后两位，不相等则添加进新数组 **

```js
const unique = (arr, result = []) => {
  arr.sort();
  result.push(arr[0]);
  const len = arr.length;
  let rLen = result.length;
  for (let i = 1; i < len; i++) {
    if (arr[i] !== result[rLen - 1]) {
      result.push(arr[i]);
      rLen++;
    }
  }
  return result;
}
```

> 方法比较直接

** 原地算法(排序后比较前后两位，相等则删除) **

```js
const unique = (arr) => {
  arr.sort();
  let len = arr.length;
  for (let i = 1; i < len; i++) {
    if (arr[i] === arr[i - 1]) {
      arr.splice(i, 1)
      len--;
    }
  }
  return arr;
}
```

> 不消耗额外的空间

- 偷懒的节奏

** `indexOf` 判断数组元素第一次出现的位置是否相同 **

```js
const unique = (arr, result) => {
  arr.forEach((item, index, array) => {
    if(array.indexOf(item) === index) {
      result.push(item);
    }
  });
  return result;
}

// 使用ES6 filter
const unique = (arr) =>
  arr.filter((item, index) =>  array.indexOf(item) === index);
```

> 使用ES6 方法更简洁性能更好

** `indexOf` 的ES6 方法通过 `includes` 判断新数组中是否有该元素 **

```js
const unique = (arr, result) => {
  arr.forEach((item, index, array) => {
    if(!result.includes(item)) {
      // 或者 result.indexOf(item) === -1
      result.push(item);
    }
  });
  return result;
}
```

> 建议使用 `includes`

** `Map` 数据结构，不懂 `Map` 的自行解决，[传送门](http://es6.ruanyifeng.com/#docs/set-map#Map) **

```js
const unique = arr => {
  const map = new Map();
  return arr.filter((item) => !map.has(item) && map.set(item, 1));
}
```

> 对象关系映射可以设置不同类型的键，使之很快能收集 `arr` 中不一样的数据

** `Set` 数据结构，不允许出现重复数据，而且 `Set` 支持解构 [传送门](http://es6.ruanyifeng.com/#docs/set-map#Set) **

```js
const unique = arr => Array.from(new Set(arr));

// 或者通过 ES6 的 ...解构
const unique = arr => [...new Set(arr)];
```

> 简单粗暴

** `reduce`，给定初始值，根据数组循环给出最终值 **

```js
const unique = (arr, result = []) => arr.reduce((prev,curr) => prev.includes(curr) ? prev : [...prev, curr], result);
```

# 三、总结

方法已经说了差不多了，就看你怎么用了，其中有一些差不多的方法，只是给了说明，没给具体的例子，希望大家自己去试一下，告辞！
