---
layout:     post
title:      "React Context的使用"
subtitle:   ""
date:       2019-12-29 20:20:00
author:     "wangtiegang"
header-img: ""
catalog: false
tags:
    - react
    - javascript
---

前面学习 react 的时候，了解到父组件跟子组件通信只能通过 props 传递属性和回调函数，父组件通过属性控制子组件，子组件又通过属性控制子子组件，层层嵌套到最底层。但这种做法对于某些类型的属性而言是极其繁琐的（例如：地区偏好，UI 主题），这些属性是应用程序中许多组件都需要的。在这种情况下，Context 被设计出来了，专门存储共享这些数据。

#### 如何使用Context

在使用 Context 前，我们可以先写一个列子，使用 props 层层传递属性。比如我们前端系统常常会设定一种主题，然后所有的组件都需要知道该主题的值是什么，然后根据主题显示对应的风格。

```javascript
// 假设 App 的主题是 dark 模式
class App extends React.Component {
  render() {
    return <PageContext> theme="dark" />;
  }
}

function PageContext(props) {
  // PageContext 组件接受一个“theme”属性，然后传递给 Toolbar 和 Table 组件。
  // 然后 Toolbar 再将主题传递给最终使用该属性的 Button 按钮，如果还有其他的就继续传递
  // 这个过程中，PageContext，Toolbar，Table 这类容器组件实际上都不是主题的使用者，它们只是起中间传递作用。
  return (
    <div>
      <Toolbar theme={props.theme} />
      <div>
        <Table theme={props.theme} >
      </div>
    </div>
  );
}

class Toolbar extends React.Component {
  render() {
    return <Button theme={this.props.theme} />;
  }
}
```

显然，如果是一个真实的页面的话，这种方式肯定非常繁琐，大量重复的传递代码，即麻烦又很不简洁。如果使用 Context 来改造的话，就是下面的样子了

```javascript
// Context 可以让我们无须明确地传遍每一个组件，就能将值深入传递进组件树。
// 通过 React.createContext 为当前的 theme 创建一个 context（“light”为默认值）。
const ThemeContext = React.createContext('light');

// 假设 App 的主题是 dark 模式
class App extends React.Component {
  render() {
    // 使用一个 Provider 来将当前的 theme 传递给以下的组件树。
    // 无论多深，任何 Provider 的子组件都能读取这个值，而不需要每一级都传递了。
    // value 给值的话，会替换掉默认值 light
    return (<ThemeContext.Provider value="dark">
                <PageContext> theme="dark" />;
            </ThemeContext.Provider>)
  }
}

function PageContext(props) {
  // Toolbar 和 Table 都不需要通过 props 传递 theme 了
  return (
    <div>
      <Toolbar/>
      <div>
        <Table>
      </div>
    </div>
  );
}

class Toolbar extends React.Component {
  render() {
    // 通过创建的 context 的 Consumer 取数据
    // 注意数据会通过一个箭头函数传递出来
    return (<ThemeContext.Consumer>
                {value => <Button theme={value} />}
            </ThemeContext.Consumer>);
  }
}
```

上面的例子中展示了，只要创建一个 context ，就能通过它的 Provider 和 Consumer 传递和读取数据，省略了中间所有层次的组件，如果在实际开发中在最上层的布局组件中使用 context，则可以很方便的传递用户的id，权限，主题等通用数据，任何子组件随时可以取用，可以省略很多冗余的代码。

Context 给我的感觉有点像 java 把数据放到 ThreadLocal 中一样，不需要层层传递，只要是同一个线程就能拿到数据。

#### 注意事项

* context 可以同时嵌套多个，子组件根据名字取对应的值

```javascript
// Theme context，默认的 theme 是 “light” 值
const ThemeContext = React.createContext('light');

// 用户登录 context
const UserContext = React.createContext({
  name: 'Guest',
});

class App extends React.Component {
  render() {
    const {signedInUser, theme} = this.props;

    // 提供初始 context 值的 App 组件
    return (
      <ThemeContext.Provider value={theme}>
        <UserContext.Provider value={signedInUser}>
          <Layout />
        </UserContext.Provider>
      </ThemeContext.Provider>
    );
  }
}

function Layout() {
  return (
    <div>
      <Sidebar />
      <Content />
    </div>
  );
}

// 一个组件可能会消费多个 context
function Content() {
  return (
    <ThemeContext.Consumer>
      {theme => (
        <UserContext.Consumer>
          {user => (
            <ProfilePage user={user} theme={theme} />
          )}
        </UserContext.Consumer>
      )}
    </ThemeContext.Consumer>
  );
}
```

* context可以在传递的对象中包含回调函数，然后子组件通过回调函数修改值

* 某些写法下，Provider 重新渲染时可能会导致子组件全部渲染

```javascript
class App extends React.Component {
  render() {
    return (
      // value直接给一个对象，Provider每次渲染时，value都是一个新对象，会导致子组件全部渲染
      <Provider value={{something: 'something'}}>
        <Toolbar />
      </Provider>
    );
  }
}

class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      value: {something: 'something'},
    };
  }

  render() {
    return (
      // 将 value 状态提升到父节点的 state 里，可以避免这个问题
      <Provider value={this.state.value}>
        <Toolbar />
      </Provider>
    );
  }
}
```