---
title: JS数组专题1️⃣_番外篇 ➖ lodash中的flatten
description: lodash中的flatten
date: 2018-08-12 16:19:18
tags: [ 数组, 数组扁平化, lodash源码 ]
categories: [ JS ]
published: true
---
> 专题一已经给大家介绍了数组扁平化，本篇将给大家介绍 `lodash` 中的 `flatten` 是如何实现的。

# 一、lodash源码

## 1.基础函数

* `isFlattenable.js`

```js
// isFlattenable.js
import isArguments from '../isArguments.js' // 检查 value 是否是一个类 arguments 对象，在本篇不予讲解。

// ES6中内置属性，可用于判断数组是否可展开. 具体可见 MDN
const spreadableSymbol = Symbol.isConcatSpreadable;
/*
value[Symbol.isConcatSpreadable] === true时可展开，
可手动设置为false value[Symbol.isConcatSpreadable] = false
*/

/**
 * 检查值是否可展开.
 *
 * @private
 * @param {*} value 要检查的值.
 * @returns {boolean} 返回布尔值，可展开 -> true, 不可展开 -> false.
 */
function isFlattenable(value) {
  return Array.isArray(value) || isArguments(value) ||
    !!(spreadableSymbol && value && value[spreadableSymbol]);
}

export default isFlattenable;
```

* `baseFlatten.js`

```js
// baseFlatten.js
import isFlattenable from './isFlattenable.js' // 是否可以扁平化

/**
 * _.flatten 的基本实现与支持限制扁平化。
 *
 * @private
 * @param {Array} array 需要扁平化的数组.
 * @param {number} depth 扁平化的深度.
 * @param {boolean} [predicate=isFlattenable] 该函数每次迭代判断是否可展开操作.
 * @param {boolean} [isStrict] 是否可通过 predicate 检查.
 * @param {Array} [result=[]] 初始结果数组.
 * @returns {Array} 返回新的扁平数组.
 */
function baseFlatten(array, depth, predicate, isStrict, result) {
  predicate || (predicate = isFlattenable);
  result || (result = []);

  if (array == null) {
    return result; // 如果数组为空，则直接返回初始化结果
  }

  for (const value of array) {
    if (depth > 0 && predicate(value)) { // 判断深度和是否可展开
      if (depth > 1) {
        // 如果深度大于1，继续递归扁平化数组.
        baseFlatten(value, depth - 1, predicate, isStrict, result);
      } else {
        // 否则 push 进结果集中
        result.push(...value);
      }
    } else if (!isStrict) {
      // 如果不检查数组，直接不展开放进结果集中
      result[result.length] = value;
    }
  }
  return result;
}

export default baseFlatten;
```

* map.js

```js
/**
 * 通过 iteratee 操作的每个元素，创建一个新数组.
 * iteratee 有三个参数 (value, index, array).
 *
 * @since 5.0.0
 * @category Array
 * @param {Array} array 要操作的数组.
 * @param {Function} iteratee 操作函数.
 * @returns {Array} 返回新的数组.
 * @example
 *
 * function square(n) {
 *   return n * n
 * }
 *
 * map([4, 8], square)
 * // => [16, 64]
 */
function map(array, iteratee) {
  let index = -1;
  const length = array == null ? 0 : array.length;
  const result = new Array(length);

  while (++index < length) {
    result[index] = iteratee(array[index], index, array);
  }
  return result;
}

export default map;
```

## 2.运用函数

实际上所有的 `flatten` 都是依赖上面的基础函数。

* `flatten.js`

```js
// flatten.js
import baseFlatten from './.internal/baseFlatten.js'

/**
 * 扁平化一级数组.
 *
 * @since 0.1.0
 * @category Array
 * @param {Array} array 将扁平化的数组.
 * @returns {Array} 返回扁平化后的数组.
 * @see flatMap, flatMapDeep, flatMapDepth, flattenDeep, flattenDepth
 * @example
 *
 * flatten([1, [2, [3, [4]], 5]])
 * // => [1, 2, [3, [4]], 5]
 */
function flatten(array) {
  const length = array == null ? 0 : array.length;
  return length ? baseFlatten(array, 1) : [];
}

export default flatten;
```

* flattenDeep.js

```js
// flattenDeep.js
import baseFlatten from './.internal/baseFlatten.js'

/** 使用一个无限大的数值. */
const INFINITY = 1 / 0

/**
 * 递归最终扁平为只有一级的数组.
 *
 * @since 3.0.0
 * @category Array
 * @param {Array} array 将扁平化的数组.
 * @returns {Array} 返回扁平化后的数组.
 * @see flatMap, flatMapDeep, flatMapDepth, flatten, flattenDepth
 * @example
 *
 * flattenDeep([1, [2, [3, [4]], 5]])
 * // => [1, 2, 3, 4, 5]
 */
function flattenDeep(array) {
  const length = array == null ? 0 : array.length;
  return length ? baseFlatten(array, INFINITY) : [];
}

export default flattenDeep;
```

* flattenDepth.js

```js
// flattenDepth.js
import baseFlatten from './.internal/baseFlatten.js'

/**
 * 递归将数组扁平化至指定深度.
 *
 * @since 4.4.0
 * @category Array
 * @param {Array} array 将扁平化的数组.
 * @param {number} [depth=1] 最大扁平化深度.
 * @returns {Array} 返回扁平化后的数组.
 * @see flatMap, flatMapDeep, flatMapDepth, flattenDeep
 * @example
 *
 * const array = [1, [2, [3, [4]], 5]]
 *
 * flattenDepth(array, 1)
 * // => [1, 2, [3, [4]], 5]
 *
 * flattenDepth(array, 2)
 * // => [1, 2, 3, [4], 5]
 */
function flattenDepth(array, depth) {
  const length = array == null ? 0 : array.length;
  if (!length) {
    return [];
  }
  depth = depth === undefined ? 1 : +depth; // 没有depth参数时为1，有的时候转化为正数.
  return baseFlatten(array, depth);
}

export default flattenDepth;
```

* flatMap.js

```js
import baseFlatten from './.internal/baseFlatten.js'
import map from './map.js' // 封装map函数

/**
 * 使用 map 重新创建数组，然后使用 baseFlatten 扁平化一级.
 *
 * @since 4.0.0
 * @category Collection
 * @param {Array|Object} collection 将处理的数组.
 * @param {Function} iteratee 数组遍历处理函数
 * @returns {Array} 返回新的扁平化后的数组.
 * @see flatMapDeep, flatMapDepth, flatten, flattenDeep, flattenDepth, map, mapKeys, mapValues
 * @example
 *
 * function duplicate(n) {
 *   return [n, n]
 * }
 *
 * flatMap([1, 2], duplicate)
 * // => [1, 1, 2, 2]
 */
function flatMap(collection, iteratee) {
  return baseFlatten(map(collection, iteratee), 1);
}

export default flatMap;
```

* flatMapDeep.js

和 `flattenDeep.js` 类似，只不过扁平化之前和 `flatMap.js` 一样先 `map` 一遍数组，然后进行扁平化处理。

* flatMapDepth.js

和 `flattenDepth.js` 类似，只不过扁平化之前和 `flatMap.js` 一样先 `map` 一遍数组，然后进行扁平化处理。

# 二、数组原生函数

## 1.`Array.prototype.flat`

* 本函数和 `flatMapDepth` 类似，只不过是挂载在 `Array` 实例下的函数。

```js
// 判断是否可展开
function isFlattenable(value) {
  const spreadableSymbol = Symbol.isConcatSpreadable;
  return Array.isArray(value) || _.isArguments(value) ||
    !!(spreadableSymbol && value && value[spreadableSymbol]);
}

// 判断空
function isEmpty(value) {
  if (value === undefined || value === null) {
    return true;
  }
  return false;
}

function baseFlatten(array, depth, predicate, isStrict, result) {
  predicate || (predicate = isFlattenable);
  result || (result = []);

  if (array == null) {
    return result; // 如果数组为空，则直接返回初始化结果
  }

  for (const value of array) {
    if (depth > 0 && predicate(value)) { // 判断深度和是否可展开
      if (depth > 1) {
        // 如果深度大于1，继续递归扁平化数组.
        baseFlatten(value, depth - 1, predicate, isStrict, result);
      } else {
        // 否则 push 进结果集中
        result.push(...value);
      }
    } else if (!isStrict && !isEmpty(value)) {
      // 如果不检查数组并且不为空，直接不展开放进结果集中
      result[result.length] = value;
    }
  }
  return result;
}

/**
 * 递归将数组扁平化至指定深度.
 *
 * @param {number} [depth=1] 最大扁平化深度.
 * @returns {Array} 返回扁平化后的数组.
 * @example
 *
 * const array = [1, [2, [3, [4]], 5]]
 *
 * array.flat(1)
 * // => [1, 2, [3, [4]], 5]
 *
 * array.flat(2)
 * // => [1, 2, 3, [4], 5]
 */
Array.prototype.flat = function flattenDepth(depth) {
  const array = this;
  const length = array.length;
  if (!length) {
    return [];
  }
  depth = depth === undefined ? 1 : +depth; // 没有depth参数时为1，有的时候转化为正数.
  return baseFlatten(array, depth);
}
```
