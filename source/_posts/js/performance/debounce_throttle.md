---
title: JS简单实现防抖和节流
description: 主要针对于时间频繁调用事件而做的性能优化
date: 2018-07-27 17:35:12
tags: [ 防抖, 节流 ]
categories: [ JS ]
published: true
---
# 一、什么是防抖和节流

**Ps**: 比如搜索框，用户在输入的时候使用`change`事件去调用搜索，如果用户每一次输入都去搜索的话，那得消耗多大的服务器资源，即使你的服务器资源很强大，也不带这么玩的。

## 1. 防抖 - debounce

其中一种解决方案就是每次用户停止输入后，延迟超过`500ms`时，才去搜索此时的`String`，这就是防抖。

* 原理：将若干个函数调用合成为一次，并在给定时间过去之后仅被调用一次。
* 代码实现：

```js
function debounce(fn, delay) {
  // 维护一个 timer，用来记录当前执行函数状态
  let timer = null;

  return function() {
    // 通过 ‘this’ 和 ‘arguments’ 获取函数的作用域和变量
    let context = this;
    let args = arguments;
    // 清理掉正在执行的函数，并重新执行
    clearTimeout(timer);
    timer = setTimeout(function() {
      fn.apply(context, args);
    }, delay);
  }
}
let flag = 0; // 记录当前函数调用次数
// 当用户滚动时被调用的函数
function foo() {
  flag++;
  console.log('Number of calls: %d', flag);
}

// 在 debounce 中包装我们的函数，过 2 秒触发一次
document.body.addEventListener('scroll', debounce(foo, 2000));
```

> 1. `debounce`函数封装后，返回内部函数
> 2. 每一次事件被触发，都会清除当前的`timer`然后重新设置超时并调用。这会导致每一次高频事件都会取消前一次的超时调用，导致事件处理程序不能被触发
> 3. 只有当高频事件停止，最后一次事件触发的超时调用才能在`delay`时间后执行

## 2. 节流 - throttle

另一种解决方案比 *防抖* 要宽松些，这时我们不想用户一味的输入，而是给用户一些搜索提示，所以在当中限制每过`500ms`就查询一次此时的`String`，这就是节流。

* 原理：节流函数不管事件触发有多频繁，都会保证在规定时间内一定会执行一次真正的事件处理函数。
* 代码实现有两种，一种是时间戳，另一种是定时器
  1）时间戳实现：

```js
function throttle(func, delay){
  let prev = Date.now();
  return function(){
    const context = this;
    const args    = arguments;
    const now     = Date.now();
    if(now - prev >= delay){
      func.apply(context, args);
      prev = Date.now();
    }
  }
}
```

> 当高频事件触发时，第一次应该会立即执行（给事件绑定函数与真正触发事件的间隔如果大于`delay`的话），而后再怎么频繁触发事件，也都是会每`delay`秒才执行一次。而当最后一次事件触发完毕后，事件也不会再被执行了。

  2）定时器实现：
  当触发事件的时候，我们设置一个定时器，再触发事件的时候，如果定时器存在，就不执行；直到`delay`秒后，定时器执行执行函数，清空定时器，这样就可以设置下个定时器。

```js
fucntion throttle(func, delay){
  let timer = null;

  return funtion(){
    let context = this;
    let args    = arguments;
    if(!timer){
      timer = setTimeout(function(){
        func.apply(context, args);
        timer = null;
      }, delay);
    }
  }
}
```

> 当第一次触发事件时，肯定不会立即执行函数，而是在`delay`秒后才执行。
> 之后连续不断触发事件，也会每`delay`秒执行一次。
> 当最后一次停止触发后，由于定时器的`delay`延迟，可能还会执行一次函数。

  3）综合使用时间戳与定时器，完成一个事件触发时立即执行，触发完毕还能执行一次的节流函数

```js
function throttle(func, delay){
  let timer = null;
  let startTime = Date.now();

  return function(){
    let curTime = Date.now();
    let remaining = delay - (curTime - startTime);
    const context = this;
    const args = arguments;

    clearTimeout(timer);
    if(remaining <= 0){
      func.apply(context,args);
      startTime = Date.now();
    }else{
      timer = setTimeout(func, remaining);
    }
  }
}
```

> 需要在每个`delay`时间中一定会执行一次函数，因此在节流函数内部使用开始时间、当前时间与`delay`来计算`remaining`，当`remaining <= 0`时表示该执行函数了，如果还没到时间的话就设定在`remaining`时间后再触发。当然在`remaining`这段时间中如果又一次发生事件，那么会取消当前的计时器，并重新计算一个`remaining`来判断当前状态。