---
layout:     post
title:      "TypeScript学习笔记（二）"
subtitle:   "《TypeScript入门教程》阅读笔记"
date:       2019-11-24 15:37:00
author:     "wangtiegang"
header-img: ""
catalog: false
tags:
    - TypeScript
    - JavaScript
---

### 莫名其妙凑篇幅的前言

在上上周的时候决定使用最新的Ant Design Pro做新的项目前端框架，因为v4版本默认使用 TypeScript ，所以就打算学习下。花一天时间看了教程的基础部分，感觉并不难。接着再去看Antd pro的代码，果然整体容易理解了很多，按照现有的页面写些类似的也能上手了，发现虽然类型定义的时候多些代码，但是使用的时候确实很爽，很多代码按照提示就能自动补全了。但是，当我去认真做一个复杂 demo 的时候，我懵逼了。

我需要使用 Ant Design 的 Table 组件实现一个可编辑单元格的表格，由于 Table 组件没有预定义好的编辑功能，需要自己实现这一块，对于一个后端来说还是很有难度的，好在官网有一个例子，于是照着实现。但是问题来了，antd 官网的例子都是 es6 规范的，并没有使用 TS 语法，需要自己改写，我尝试改写却发现很多类型我没法确定，很容易写错，而且很多对象很复杂，如果我全部定义成 any 类型的话，显然就失去了使用 TS 的意义。来回折腾了半天，解决一个问题又有下一个问题，然后我停下来仔细思考了一下

* TS 确实很强大，但是对于一个新手来说，还是有难度的，尤其是应用到实际项目中。
* Antd开源库虽然接入了 TS ，但是文档还是采用 es6 ，很不友好。
* TS 需要花费不少时间在类型和接口定义上，帮助项目更加规范，减少错误，提升后期开发维护效率
* TS 最终还是编译成 es6 运行的，并不会有性能上的提升
* 我目前最需要的应该是如何在新的前端框架下实现效果，而不是一步到位，挑战最高难度

于是，我放弃了。。。对于一个半吊子冒牌前端，现学现用React技术栈，怎么实现效果才是最重要的，如果在 TS 上浪费大量时间，就得不偿失了。所以最终决定使用Antd pro的 es6 版本进行开发。后续等项目顺利实现了，再考虑切换到 TS，起码没有无法实现的风险。

虽然兴奋的学习了一波 TS，想紧跟潮流有逼格一点，但是力不从心啊，不过 TS 的下半部分的学习笔记还是要记录下来的，这样后面切换的时候可以快速回忆起来，以下记录都是简单的知识点，深入的还是需要再查官方文档学习。

### TypeScript进阶

类型别名
---

类型别名用来给一个类型起个新名字，使用 ```type``` 创建类型别名。

```javascript
type Name = string; // string类型别名
type NameResolver = () => string; // 一个返回string的函数类型
type NameOrResolver = Name | NameResolver; // stirng 或 返回string的函数
function getName(n: NameOrResolver): Name {
    if (typeof n === 'string') {
        return n; // 如果string则返回n
    } else {
        return n(); // 如果是函数则执行函数，返回函数的string结果
    }
}
```

字符串字面量类型
---

字符串字面量类型用来约束取值只能是某几个字符串中的一个，注意，类型别名与字符串字面量类型都是使用 type 进行定义。

```javascript
type EventNames = 'click' | 'scroll' | 'mousemove';
const e: EventNames = 'doubleClick'; // 错误，不在约束内
```

元组
---

数组合并了相同类型的对象，而元组（Tuple）合并了不同类型的对象。相当于定义了一个包含不同类型元素的数组，每个元素必须跟对应位置的类型一致，取出元素也直接是对应的类型。

```javascript
// 第一个元素为string，第二个为number
let user: [string, number] = ['wtg', 18];

tom[0].isNaN(); // true
tom[1].isNaN(); // false

// 可以分别赋值
let user2: [string, number];
user2[0] = 'wtg';
// 但是当直接对元组类型的变量进行初始化或者赋值的时候，需要提供所有元组类型中指定的项。
user2 = ['wtg']; // 错误
user2 = ['wtg', 25]; // 正确
// 可以直接添加越界元素，当添加越界的元素时，它的类型会被限制为元组中每个类型的联合类型
user2.push('male'); // 正确
user2.push(true); // 错误
```

枚举
---

TS 的枚举跟 Java 中的非常相似。

```javascript
// 数字枚举，Up的值为 0， Down的值为 1，依次自增
enum Direction {
    Up ,
    Down,
    Left,
    Right
}
// 字符串枚举，字符串枚举没有自增长的行为
enum Direction {
    Up = "UP",
    Down = "DOWN",
    Left = "LEFT",
    Right = "RIGHT",
}

const a = Direction.Up;
```

类
---

TS 中的类就是es6中的类，并且根据新的js规范做了增强，和Java也类似，可以继承，重载等等。还可以使用 static、abstract、private、protected、public 等关键字，但是区别较大的有 readonly、getter、setter 知识点，此处就不详细写了，关键是看一遍不用又忘了。。

接口
---

TS 里面的接口就是用来规范描述一个对象的“形状”的，不但可以描述类，还可以用来描述数据对象和函数。

```javascript
interface LabelledValue {
  label: string;
  color?: string;
}
let labelledObj: LabelledValue;
// 必须含有 label 属性，color可选，可以含有未定义的属性size
labelledObj = {size: 10, label: "Size 10 Object"};

// 函数
interface SearchFunc {
  (source: string, subString: string): boolean;
}
let mySearch: SearchFunc;
mySearch = function(source: string, subString: string) {
  let result = source.search(subString);
  return result > -1;
}

// 类，这个在react class组件中经常定义
interface ClockInterface {
    currentTime: Date;
}
class Clock implements ClockInterface {
    currentTime: Date;
    constructor(h: number, m: number) { }
}
```

泛型
---

TS 的泛型跟 Java 也很类似，在像C#和Java这样的语言中，可以使用泛型来创建可重用的组件，一个组件可以支持多种类型的数据。 这样用户就可以以自己的数据类型来使用组件。

```javascript
interface GenericIdentityFn<T> {
    (arg: T): T;
}

function identity<T>(arg: T): T {
    return arg;
}

let myIdentity: GenericIdentityFn<number> = identity;
```

以上就是 TS 中经常涉及到的知识点，其中枚举、类、接口、泛型都比较复杂，看一遍不使用的话还是会忘，好在很多地方都跟Java类似，只要联想以下就比较容易理解，当然TS中的知识点远不止这些，后续使用的时候还是需要再仔细查询这一块的官方文档。