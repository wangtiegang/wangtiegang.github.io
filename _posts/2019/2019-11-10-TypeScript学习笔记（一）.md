---
layout:     post
title:      "TypeScript学习笔记（一）"
subtitle:   "《TypeScript入门教程》阅读笔记"
date:       2019-11-10 10:34:00
author:     "wangtiegang"
header-img: ""
catalog: false
tags:
    - TypeScript
    - JavaScript
---

> TypeScript 是 JavaScript 的一个超集，主要提供了类型系统和对 ES6 的支持，它由 Microsoft 开发，代码开源于 GitHub 上。

ts代码最终会编译成js代码运行，但是会在编译时做静态检查。

原始数据类型
---

JavaScript类型分两种：原始数据类型（Primitive data types）和对象类型（Object types）。原始数据类型包括：布尔值、数值、字符串、null、undefined 以及 ES6 中的新类型 Symbol。

TypeScript使用 ```:``` 声明变量的类型

```javascript
let isDone: boolean = false;
let decLiteral: number = 6;
let myName: string = 'Tom';
// 模板字符串
let sentence: string = `Hello, my name is ${myName}.`;
```

Js没有空值（void）的概念，在ts中，可以使用 ```void```` 表示没有任何返回值的函数

```javascript
function test():void {
    console.log('test');
}
```

在ts中，```undefined``` 和 ```null``` 是所有类型的子类型，可以赋值给任意类型的变量，不会编译报错。

任意值（any）用来表示允许赋值为任意类型，在任意值上访问任何属性都是允许的。

```javascript
let anyThing: any = 'hello';
console.log(anyThing.myName.firstName);
anyThing.setName('Jerry').sayHello();
```

变量如果在声明的时候，未指定其类型，那么ts会根据规则分析赋值进行类型推论。如果定义的时候没有赋值，不管之后有没有赋值，都会被推断成 ```any``` 类型而完全不被类型检查。

```javascript
// 根据赋值推论出类型为string，所以在赋值7时编译报错
let myFavoriteNumber = 'seven';
myFavoriteNumber = 7;
// index.ts(2,1): error TS2322: Type 'number' is not assignable to type 'string'.

// 推论为 any ，可以赋值任意类型
let myFavoriteNumber;
myFavoriteNumber = 'seven';
myFavoriteNumber = 7;
```

联合类型
---

联合类型（Union Types）表示取值可以为多种类型中的一种，联合类型使用 | 分隔每个类型。

```javascript
let myFavoriteNumber: string | number;
myFavoriteNumber = 'seven';
myFavoriteNumber = 7;
```

联合类型的变量在被赋值的时候，会根据类型推论的规则推断出一个类型

```javascript
let myFavoriteNumber: string | number;
myFavoriteNumber = 'seven'; // 推论为 string
console.log(myFavoriteNumber.length); // 合法访问，返回5
myFavoriteNumber = 7; // 推论为 number
console.log(myFavoriteNumber.length); // 无length属性，编译时报错
```

当 TypeScript 不确定一个联合类型的变量到底是哪个类型的时候，我们只能访问此联合类型的所有类型里共有的属性或方法，

```javascript
function getString(something: string | number): string {
    // return something.length; // 报错，不是共有属性
    return something.toString();
}
```

对象的类型->接口
---

在面向对象语言中，接口（Interfaces）是一个很重要的概念，它是对行为的抽象，而具体如何行动需要由类（classes）去实现（implement）。

TypeScript 中的接口是一个非常灵活的概念，除了可用于对类的一部分行为进行抽象以外，也常用于对「对象的形状（Shape）」进行描述。

简单点来说就是可定义对象有哪些属性，属性是什么类型，是否必须赋值等。

```javascript
interface Person {
    name: string; // 初始化时必须赋值
    age: number; // 初始化时必须赋值 
    sex?: string; // 可选属性
}

let tom: Person = {
    name: 'Tom',
    age: 18
};

let tom: Person = {
    name: 'Tom',
    age: 18,
    sex: 'F'
};
```

上面 ```Person``` 属性不能多也不能少，有时候我们想接收任意多的属性，可以使用如下方式：

```javascript
interface Person {
    name: string;
    age?: number;
    [propName: string]: any; // 表示可以接受任意数量和类型的属性
}
```

**需要注意的是，一旦定义了任意属性，那么确定属性和可选属性的类型都必须是它的类型的子集** ，在上面的例子中，```[propName: string]``` 表示新属性名只能使用 ```string``` 类型，已经定义的name和age不受约束，```any``` 表示值可以为 ```any``` 类型，已经定义的name和age也受约束，由于name和age都是any的子类型，所以编译通过。

```javascript
interface Person {
    name: string;
    age?: number;
    [propName: string]: string;
}

let tom: Person = {
    name: 'Tom',
    age: 25,
    gender: 'male'
};

// index.ts(3,5): error TS2411: Property 'age' of type 'number' is not assignable to string index type 'string'.
// index.ts(7,5): error TS2322: Type '{ [x: string]: string | number; name: string; age: number; gender: string; }' is not assignable to type 'Person'.
//   Index signatures are incompatible.
//     Type 'string | number' is not assignable to type 'string'.
//       Type 'number' is not assignable to type 'string'.
```

上例中，任意属性的值允许是 string，但是可选属性 age 的值却是 number，number 不是 string 的子属性，所以报错了。

有时候我们希望对象中的一些字段只能在创建的时候被赋值，那么可以用 readonly 定义只读属性。**需要注意的是，只读的约束存在于第一次给对象赋值的时候，而不是第一次给只读属性赋值的时候**

```javascript
interface Person {
    readonly id: number;
    name: string;
    age?: number;
    [propName: string]: any;
}

// 会报错，没有给id赋值
let tom: Person = {
    name: 'Tom',
    gender: 'male'
};

// 会报错，tom已经被赋值了，此时id不能再被修改
tom.id = 89757;

```

数组类型
---

ts中数组可以使用以下几种方式定义

```javascript
// 推荐
let fibonacci: number[] = [1, 1, 2, 3, 5];
// 泛型
let fibonacci: Array<number> = [1, 1, 2, 3, 5];
// 用接口表示数组 不推荐
interface NumberArray {
    [index: number]: number;
}
let fibonacci: NumberArray = [1, 1, 2, 3, 5];
```

函数的类型
---

在 JavaScript 中，有两种常见的定义函数的方式——函数声明（Function Declaration）和函数表达式（Function Expression），在ts中进行类型约束，则这样定义：

```javascript
// 函数声明
function sum(x: number, y: number): number {
    return x + y;
}
// 函数表达式 ， mySum会被推断出类型
let mySum = function (x: number, y: number): number {
    return x + y;
};
// 使用接口定义函数
interface SearchFunc {
    (source: string, subString: string): boolean;
}

let mySearch: SearchFunc;
mySearch = function(source: string, subString: string) {
    return true;
}
// ? 表示可选参数，必须放在结尾
let mySum = function (x: number, y?: number): number {
    return x + y;
};
// 参数的默认值，有默认值的参数可以不传
let mySum = function (x: number = 1, y?: number): number {
    return x + y;
};
```

剩余参数，ES6 中，可以使用 ```...rest``` 的方式获取函数中的剩余参数，事实上，items 是一个数组。所以我们可以用数组的类型来定义它

```javascript
function push(array: any[], ...items: any[]) {
    items.forEach(function(item) {
        array.push(item);
    });
}

let a = [];
push(a, 1, 2, 3);
```

重载允许一个函数接受不同数量或类型的参数时，作出不同的处理，我们可以使用重载定义多个 reverse 的函数类型

```javascript
function reverse(x: number): number;
function reverse(x: string): string;
function reverse(x: number | string): number | string {
    if (typeof x === 'number') {
        return Number(x.toString().split('').reverse().join(''));
    } else if (typeof x === 'string') {
        return x.split('').reverse().join('');
    }
}
```

上例中，我们重复定义了多次函数 reverse，前几次都是函数定义，最后一次是函数实现。在编辑器的代码提示中，可以正确的看到前两个提示。

类型断言
---

类型断言（Type Assertion）可以用来手动指定一个值的类型， ```<类型>值``` 或 ```值 as 类型```。

前面说过，当 TypeScript 不确定一个联合类型的变量到底是哪个类型的时候，我们只能访问此联合类型的所有类型里共有的属性或方法，但有时候，我们确实需要在还不确定类型的时候就访问其中一个类型的属性或方法，此时可以使用类型断言，将一个联合类型的变量指定为一个更加具体的类型

```javascript
function getLength(something: string | number): number {
    // 直接 something.length 会编译报错
    // 类型断言不是类型转换，断言成一个联合类型中不存在的类型是不允许的
    if ((something as string).length) {
        return (<string>something).length;
    } else {
        return something.toString().length;
    }
}
```