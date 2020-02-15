---
layout:     post
title:      "React函数式组件和Hook学习笔记"
subtitle:   ""
date:       2020-02-15 17:53:00
author:     "wangtiegang"
header-img: ""
catalog: true
tags:
    - react
    - Javascript
---

最近下载最新的 ant desig pro 代码下来后，发现短短的一个多月没看，陌生了很多，仔细一看发现原来的项目主要使用 class 组件和 state，现在的项目主要使用函数式组件和 Hook ，风格变了之后，一下变得难理解了。不过稍微了解下 React 的 Hook 之后，又变得熟悉了。

### 函数式组件

引用官方的说法，定义组件最简单的方式就是编写 JavaScript 函数：

```javascript
// 定义一个组件 Welcome
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

// react会自动将 name = "Sara" 转成 {name: 'Sara'} 作为 props 传入 Welcome
const element = <Welcome name="Sara" />;
```

该函数是一个有效的 React 组件，因为它接收唯一带有数据的 “props”（代表属性）对象与并返回一个 React 元素。这类组件被称为“函数组件”，因为它本质上就是 JavaScript 函数。


### Hook

我们在 class 组件中使用 state 作为属性来管理组建的内部状态，state 改变会触发 class 的 render 函数来渲染新的状态。但是这个在函数式组件中却无法使用，因为没法在函数内定义一个 state 来保存状态。所以 react 在 16.8 的新版本中推出了 Hook ，它可以让你在不编写 class 的情况下使用 state 以及其他的 React 特性。

Hook 是完全可选的，100%向下兼容，我不打算详细讲解 Hook 的设计思想，有哪些好处，而是如何快速上手，看懂别人使用 Hook 定义组件。

#### State Hook

首先看一个官方例子

```javascript
import React, { useState } from 'react';

function Example() {
  // 声明一个叫 “count” 的 state 变量。
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

上面的例子中 useState 就是一个 Hook ，通过在函数组件里调用它来给组件添加一些内部 state。React 会在重复渲染时保留这个 state。useState 会返回一对值：当前状态和一个让你更新它的函数。useState 的参数 0 就是 count 的初始值。这个我感觉就像状态提升，react 将函数组件的内部状态提升到了更高级组件，就像 dva 一样。你可以直接使用 count 变量读取状态值，也可以直接使用 setCount 函数更新 count 的值，一旦 count 的值发生改变，就会触发组件更新，需要注意的是，setXXXX 函数是每次替换原来的对象，不会像 setState 那样将新旧 state 进行合并。当然，useState 在组件中是可以定义多个的，不像在 class 中只能定义一个 state。

#### Effect Hook

在 React 组件中执行过数据获取、订阅或者手动修改过 DOM。我们统一把这些操作称为“副作用”，或者简称为“作用”。

useEffect 就是一个 Effect Hook，给函数组件增加了操作副作用的能力。它跟 class 组件中的 componentDidMount、componentDidUpdate 和 componentWillUnmount 具有相同的用途，只不过被合并成了一个 API。

* componentDidMount 组件挂载完成，render后执行一次
* componentDidUpdate 组件每次更新都会执行，在第一次初始化时不会执行
* componentWillUnmount 组件卸载前执行一次

**由于 componentDidMount 和 componentDidUpdate 的执行顺序，有些副作用，比如查询数据，我们需要在这两个函数中都定义一遍，造成重复代码**，而 useEffect 会在每次渲染后调用副作用函数 —— 包括第一次挂载渲染的时候，如果 useEffect 返回一个函数，则 react 会在卸载组件时调用它，相当于 componentWillUnmount。

```javascript
import React, { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  // 相当于 componentDidMount 和 componentDidUpdate:
  useEffect(() => {
    // 组件挂载和每次更新都会使用浏览器的 API 更新页面标题
    document.title = `You clicked ${count} times`;
    // 非必须，如果返回一个函数，则卸载组件时会被调用
    return function cleanup() {
      // do something
    };
  });

  useEffect(() => {
    // 跟 useState 一样，你可以在组件中多次使用 useEffect

  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

有时候我们并不想每次组件更新都执行 useEffect，以此来提高性能，则可以这样做

```javascript
import React, { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  // 传入第二个参数count，仅在 count 更改时更新
  useEffect(() => {
    // 跟 useState 一样，你可以在组件中多次使用 useEffect

  }, [count]);

  // 传入第二个参数 []，一个空数组，则仅在组件挂载时执行一次
  useEffect(() => {
    // 跟 useState 一样，你可以在组件中多次使用 useEffect

  }, []);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```


#### 自定义Hook

有时候我们会想要在组件之间重用一些状态逻辑。目前为止，有两种主流方案来解决这个问题：高阶组件和 render props。自定义 Hook 可以让你在不增加组件的情况下达到同样的目的。

```javascript
import React, { useState, useEffect } from 'react';

/**
* 自定义Hook，通过好友id来订阅好友的在线状态
* @param friendID
* @return 返回好友状态
*/
function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);

  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }

  useEffect(() => {
    ChatAPI.subscribeToFriendStatus(friendID, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(friendID, handleStatusChange);
    };
  });

  return isOnline;
}
```

在两个组件中使用它

```javascript
// 组件一
function FriendStatus(props) {
  const isOnline = useFriendStatus(props.friend.id);

  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}
// 组件二
function FriendListItem(props) {
  const isOnline = useFriendStatus(props.friend.id);

  return (
    <li style={{ color: isOnline ? 'green' : 'black' }}>
      {props.friend.name}
    </li>
  );
}
```

这两个组件的 state 是完全独立的。Hook 是一种复用状态逻辑的方式，它不复用 state 本身。**事实上 Hook 的每次调用都有一个完全独立的 state** —— 因此你可以在单个组件中多次调用同一个自定义 Hook。

自定义 Hook 更像是一种约定而不是功能。**如果函数的名字以 “use” 开头并调用其他 Hook，我们就说这是一个自定义 Hook**。 useSomething 的命名约定可以让我们的 linter 插件在使用 Hook 的代码中找到 bug。


### Hook使用规则

* 只能在函数最外层调用 Hook。不要在循环、条件判断或者子函数中调用。
* 只能在 React 的函数组件中调用 Hook。不要在其他 JavaScript 函数中调用（自定义Hook中也可以）。

### 其他Hook

除此之外，还有一些使用频率较低的但是很有用的 Hook。比如，```useContext```, ```useReducer```。
