---
layout:     post
title:      "JavaScript 定时器"
subtitle:   ""
date:       2020-07-12 13:02:00
author:     "wangtiegang"
header-img: ""
catalog: true
tags:
    - Javascript
---

一直以来，使用定时器都是依葫芦画瓢，从来没有仔细看过，今天从头学习一下，记点笔记。

JavaScrpit 提供定时执行代码的功能，叫做定时器，主要由 ```setTimeout()``` 和 ```setInterval()``` 两个函数来完成。它们向任务队列添加定时任务。

* setTimeout

```setTimeout``` 用来指定代码在多少毫秒后执行，并且返回一个 id，可以使用这个 id 来取消定时任务。

```javascript
// 定义定时任务，延迟 delay 毫秒
const timerId = setTimeout(func|code, delay);
// 取消定时任务
clearTimeout(timerId);

// 如果第二个参数不给，默认为 0
setTimeout(func|code);
// 等同于
setTimeout(func|code, 0);

// 如果参数是 func ，则直接给函数名称
const f = () => {};
setTimeout(f, 0);
// 如果参数是 code，则必须以字符串的形式传入
setTimeout('console.log(1);',0);

// setTimeout 可接受更多的参数，将以此传递给定时函数的参数
setTimeout(function (a,b) {
  console.log(a + b);
}, 1000, 1, 1);

// 如果定时函数中使用了 this ，那么调用时，this 将指向全局环境，而不是定义时的对象
// 可以通过 bind 函数或定义包裹函数的方式解决 this 指向问题
var x = 1;
var obj = {
  x: 2,
  y: function () {
    console.log(this.x);
  }
};
setTimeout(obj.y, 1000) // 1

```

* setInterval

```setInterval``` 跟 ```setTimeout``` 用法完全一致，区别在于 ```setInterval``` 是每隔一段时间就执行以此，无限执行，直到关闭当前窗口或取消。

```setInterval``` 不会考虑任务的执行时间，如果定义一个 100 毫秒执行的任务，任务执行需要 5 毫秒，那么实际间隔时间是 95 毫秒，如果执行时间是 105 毫秒，那么执行结束会立马执行下一个。

如果想严格控制任务的执行时间间隔，则可以使用 ```setTimeout```

```javascript
var i = 1;
var timer = setTimeout(function f() {
  // 每隔 2000 毫秒打印
  console.log('log...');
  timer = setTimeout(f, 2000);
}, 2000);
```

* clearTimeout 和 clearInterval

```javascript
var id1 = setTimeout(f, 1000);
var id2 = setInterval(f, 1000);
// 清除定时器
clearTimeout(id1);
clearInterval(id2);
```

```setInterval``` 跟 ```setTimeout``` 返回的整数值是连续的，所以可以利用这个特点取消所有的定时任务

```javascript
function clearAllTimeouts() {
  // 先调用一次获取最大的定时任务 id
  var id = setTimeout(function() {}, 0);
  // 取消所有定时任务
  while (id > 0) {
    clearTimeout(id);
    id--;
  }
};
```

* 运行机制

```setInterval``` 跟 ```setTimeout``` **是将代码移除本次事件循环，等到下一轮事件循环再判断是否到了指定时间，如果到了则执行，否则继续等待。这意味着定时任务必须等到本轮事件循环的所有同步任务都执行完，才会开始执行。由于前面的任务到底需要多少时间执行完，是不确定的，所以没有办法保证指定的任务，一定会按照预定时间执行。**

```javascript
// 1000 毫秒执行一次
setInterval(function () {
  console.log(2);
}, 1000);
// 实际上本次事件循环会休眠 3000 毫秒，所以实际上是 3000 毫秒后才执
// 行第一次定时任务，并且任务不会积累，所以只会输出一个 2
sleep(3000);
```

* setTimeout(f, 0)

setTimeout的作用是将代码推迟到指定时间执行，如果指定时间为0，即setTimeout(f, 0)，任务不一定会立即执行，具体得看本轮事件循环执行的时间，但是 f 会在下轮时间循环一开始时就执行。

```setTimeout(f, 0)``` 有几个非常重要的用途。它的一大应用是，可以调整事件的发生顺序。另一个应用是，用户自定义的回调函数，通常在浏览器的默认动作之前触发，可能达不到想要的效果，可以使用 ```setTimeout(f, 0)``` 将其调整到下一轮事件循环。

* 使用 setTimeout 实现 debounce 函数

```javascript
// 2500 毫秒内只能点击一次
$('textarea').on('keydown', debounce(ajaxAction, 2500));

function debounce(fn, delay){
  var timer = null; // 声明计时器，持有上一次点击的定时任务 id
  return function() {
    var context = this;
    var args = arguments;
    // 每次都取消上一次的定时任务，如果 2500 内则取消成功，2500 外则是下一次有效点击
    clearTimeout(timer);
    timer = setTimeout(function () {
      fn.apply(context, args);
    }, delay);
  };
}
```
