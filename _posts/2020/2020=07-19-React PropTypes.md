---
layout:     post
title:      "React PropTypes"
subtitle:   ""
date:       2019-07-19 19:23:00
author:     "wangtiegang"
header-img: ""
catalog: false
tags:
    - react
---

平时在 VS CODE 中使用 React 组件时，智能提醒功能非常好用，可以清楚的知道组件有哪些属性，是什么类型，减少了查阅文档的时间，同时也减少了差错。于是在自定义组件时，也想有智能提示，这样同事之间就能很方便的使用各自定义的组件了。于是查了一下如何给组件增加智能提示，发现 VS CODE 的智能提示是根据 ```.d.ts``` 文件生成的，这个文件必须在 ```TS``` 下才起作用，而我的项目是使用 ```javascript``` 的，即使我自己手动定义 ```.d.ts``` 文件也不会起作用，这下就一下找不到解决办法了，只能暂时作罢。

但是突然发现了 ```React PropTypes``` 组件，虽然不能起到智能提示的作用，但是它可以规定组件的属性类型，是否必须等，这样就可以在编译阶段进行校验，避免一部分错误。

* 安装

```shell
npm install --save prop-types
```

* 基本使用
  
```javascript
import PropTypes from 'prop-types';

class Greeting extends React.Component {
  render() {
    return (
      <h1>Hello, {this.props.name}</h1>
    );
  }
}
// 通过 .propTypes 属性定义类型检查
Greeting.propTypes = {
  // 属性必须为 string 类型，否则会在控制台输出警告信息
  name: PropTypes.string
};
```

* PropTypes 验证器

PropTypes 提供一系列验证器，可用于确保组件接收到的数据类型是有效的。当传入的 prop 值类型不正确时，JavaScript 控制台将会显示警告。**出于性能方面的考虑，propTypes 仅在开发模式下进行检查**。

```javascript
import PropTypes from 'prop-types';

MyComponent.propTypes = {
  // 你可以将属性声明为 JS 原生类型，默认情况下
  optionalArray: PropTypes.array,
  optionalBool: PropTypes.bool,
  optionalFunc: PropTypes.func,
  optionalNumber: PropTypes.number,
  optionalObject: PropTypes.object,
  optionalString: PropTypes.string,
  optionalSymbol: PropTypes.symbol,

  // 任何可被渲染的元素（包括数字、字符串、元素或数组）
  // (或 Fragment) 也包含这些类型。
  optionalNode: PropTypes.node,

  // 一个 React 元素。
  optionalElement: PropTypes.element,

  // 一个 React 元素类型（即，MyComponent）。
  optionalElementType: PropTypes.elementType,

  // 你也可以声明 prop 为类的实例，这里使用
  // JS 的 instanceof 操作符。
  optionalMessage: PropTypes.instanceOf(Message),

  // 你可以让你的 prop 只能是特定的值，指定它为
  // 枚举类型。
  optionalEnum: PropTypes.oneOf(['News', 'Photos']),

  // 一个对象可以是几种类型中的任意一个类型
  optionalUnion: PropTypes.oneOfType([
    PropTypes.string,
    PropTypes.number,
    PropTypes.instanceOf(Message)
  ]),

  // 可以指定一个数组由某一类型的元素组成
  optionalArrayOf: PropTypes.arrayOf(PropTypes.number),

  // 可以指定一个对象由某一类型的值组成
  optionalObjectOf: PropTypes.objectOf(PropTypes.number),

  // 可以指定一个对象由特定的类型值组成
  optionalObjectWithShape: PropTypes.shape({
    color: PropTypes.string,
    fontSize: PropTypes.number
  }),
  
  // An object with warnings on extra properties
  optionalObjectWithStrictShape: PropTypes.exact({
    name: PropTypes.string,
    quantity: PropTypes.number
  }),   

  // 你可以在任何 PropTypes 属性后面加上 `isRequired` ，确保
  // 这个 prop 没有被提供时，会打印警告信息。
  requiredFunc: PropTypes.func.isRequired,

  // 任意类型的数据
  requiredAny: PropTypes.any.isRequired,

  // 你可以指定一个自定义验证器。它在验证失败时应返回一个 Error 对象。
  // 请不要使用 `console.warn` 或抛出异常，因为这在 `onOfType` 中不会起作用。
  customProp: function(props, propName, componentName) {
    if (!/matchme/.test(props[propName])) {
      return new Error(
        'Invalid prop `' + propName + '` supplied to' +
        ' `' + componentName + '`. Validation failed.'
      );
    }
  },

  // 你也可以提供一个自定义的 `arrayOf` 或 `objectOf` 验证器。
  // 它应该在验证失败时返回一个 Error 对象。
  // 验证器将验证数组或对象中的每个值。验证器的前两个参数
  // 第一个是数组或对象本身
  // 第二个是他们当前的键。
  customArrayProp: PropTypes.arrayOf(function(propValue, key, componentName, location, propFullName) {
    if (!/matchme/.test(propValue[key])) {
      return new Error(
        'Invalid prop `' + propFullName + '` supplied to' +
        ' `' + componentName + '`. Validation failed.'
      );
    }
  })
};
```

* 默认Prop值

```defaultProps``` 可以指定默认值，通常如果一个属性不是必须的，那么按照规范可以提供一个默认值

```javascript
class Greeting extends React.Component {
  render() {
    return (
      <h1>Hello, {this.props.name}{this.props.code}</h1>
    );
  }
}

// 类型检查
Greeting.propTypes = {
  code: PropTypes.string.isRequired,
  name: PropTypes.string,
};

// 指定 name 的默认值：
Greeting.defaultProps = {
  name: 'Stranger'
};
```

defaultProps 用于确保 this.props.name 在父组件没有指定其值时，**有一个默认值。propTypes 类型检查发生在 defaultProps 赋值后，所以类型检查也适用于 defaultProps**。

如果你正在使用像 transform-class-properties 的 Babel 转换工具，你也可以在 React 组件类中声明 defaultProps 作为静态属性

```javascript
class Greeting extends React.Component {
  // 默认props
  static defaultProps = {
    name: 'stranger'
  }

  render() {
    // ...
  }
}
```