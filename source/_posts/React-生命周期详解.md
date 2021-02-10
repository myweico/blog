---
title: React 生命周期详解
date: 2021-02-10 16:10:08
tags:
- React

categories:
- 框架
---

组件有不同的生命周期，也就是有不同的阶段。通过生命周期方法，我们可以指定在不同阶段的行为。

比如：
- 在组件挂载后，请求网络数据、订阅数据，设置定时器等。
- 在组件更新前，通过判断 props 或者 state 控制组件是否更新，从而实现性能优化。
- 在组件卸载销毁前，清理定时器，取消数据的订阅。

<!-- more -->

## 参考资料
- [官网文档](https://zh-hans.reactjs.org/docs/react-component.html)
- [生命周期图谱](https://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/)

## React 的生命周期变化
### React 16 前
生命周期有：
- constructor()
- componentWillMount()
- render()
- componentDidMount()
- componentWillReceiveProps(nextProps)
- shouldComponentUpdate(nextProps, nextState)
- componentWillUpdate(nextProps，nextState)
- componentDidUpdate(prevProps, prevState)
- componentWillUnmount()

之前的生命周期图谱：
![](https://blog-1257268092.cos.ap-guangzhou.myqcloud.com/notes/20210209180032.png)
> 图片来自：https://hackernoon.com/reactjs-component-lifecycle-methods-a-deep-dive-38275d9d13c0?gi=630d5f23e5a

### React 16 后
因为 React 使用了 Fiber 的结构，之前在 render 之前的生命周期函数有可能会被执行多次，所以删除了以下生命周期函数，过渡方案就是添加`UNSAFE_`前缀，过渡方案可使用至 React 17
- ~~componentWillReceiveProps~~
    - 暂时可用 UNSAFE_componentWillReceiveProps
- ~~componentWillUpdate~~
    - 暂时可用 UNSAFE_componentWillUpdate
- ~~componentWillMount~~
    - 暂时可用 UNSAFE_componentWillMount

可以使用 [rename-unsafe-lifecycles](https://github.com/reactjs/react-codemod#rename-unsafe-lifecycles) 这个工具，将工程中旧的生命周期函数名更改到过渡的生命周期名


新增了：
- static getDerivedStateFromProps()
- getSnapshotBeforeUpdate()
- static getDerivedStateFromError()
- componentDidCatch()

也可使用 `static getDerivedStateFromProps()`、`getSnapshotBeforeUpdate()` 这两个新的生命周期函数替代被删除掉的`componentWillReceiveProps`、`componentWillUpdate`和`componentWillMount`

React 16 的生命周期图谱
16.3：
![](https://blog-1257268092.cos.ap-guangzhou.myqcloud.com/notes/20210210155156.png)

版本>= 16.4 的：
![](https://blog-1257268092.cos.ap-guangzhou.myqcloud.com/notes/20210210155036.png)

> 图片来自：https://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/

## React的生命周期详解
### 初始化阶段
#### constructor()
也就是执行 constructor 这个阶段，这时候一般用于初始化 state 状态

### 挂载阶段
#### componentWillMount()
```
componentWillMount()
```
- 在挂载阶段，render前面被调用。
- 它在 render() 之前调用，因此在此方法中同步调用 setState()不会触发额外渲染。通常，我们建议使用 constructor() 来初始化 state。
- 避免在此方法中引入任何副作用或订阅。如遇此种情况，请改用 componentDidMount()。
- 在 16.3及以后，更名为 `UNSAFE_componentWillMount()`，可以被使用至 react17

#### static getDerivedStateFromProps()
```js
static getDerivedStateFromProps(props, state)
```
- 调用 render 方法之前调用，并且在初始挂载及后续更新时都会被调用。
- 它应返回一个对象来更新 state，如果返回 null 则不更新任何内容。
- 16.3 的时候，forUpdate不会触发 `getDerivedStateFromProps`，后面16.3 后 forceUpdate() 同样会触发 `getDerivedStateFromProps`
- 在每次渲染前都会执行，而对比之下，废除的 `componentWillReceiveProps` 则只在父组件重新渲染的时候触发

派生状态会导致代码冗余，并使组件难以维护。 确保你已熟悉这些简单的替代方案：

- 如果你需要执行副作用（例如，数据提取或动画）以响应 props 中的更改，请改用 componentDidUpdate。
- 如果只想在 prop 更改时重新计算某些数据，请使用 memoization helper 代替。
- 如果你想在 prop 更改时“重置”某些 state，请考虑使组件完全受控或使用 key 使组件完全不受控 代替。

#### render()
```
render()
```
- 根据 props 以及 state 获取虚拟 dom
- render 是 class 组件必需要实现的方法
- render 函数可以返回以下类型
    - React 元素
    - 数组或 fragment
    - Portals。可以渲染子节点到不同的 DOM 子树中
    - 字符串或者数值类型。
    - null 或者 布尔类型，不会渲染任何东西

#### componentDidMount()
```js
componentDidMount()
```
- 将组件挂载（插入到 dom树后）立即执行。
- 可在此处订阅和请求数据，获取到 dom 的一些信息。
- 在此处 setState 会导致两次渲染，但此次渲染会发生在浏览器更新屏幕前。因此在两次调用下，用户也不会看到中间状态。

### 更新阶段
更新阶段主要有三种情形：
- 父组件渲染，导致 props 改变，触发组件更新
- 组件使用 setState() 改变组件内部 state，触发更新
- 使用 forceUpdate() 方法

#### componentWillReceiveProps()
```js
componentWillReceiveProps(nextProps)
```
- 在已挂载的组件接受新的 props 触发。（不会在接受初始 props 的时候更新，而是在更新 props 后更新）
- 如果父组件导致组件重新渲染，即使 props 没有更改，也会调用此方法。
- 在16.3及以后更名为 `UNSAFE_componentWillReceiveProps`，可继续使用至 React 17，后续将废置

#### shouldComponentUpdate()
```
shouldComponentUpdate(nextProps, nextState)
```
- 在已挂载组件接受新 props 或者 更新了 state 的时候触发
- 返回 true 表示允许组件更新， 返回 false 表示跳过此次更新
- 通常通过 `this.state` 和 `nextState`，`this.props` 和 `nextProps` 的比较，控制是否更新，从而实现性能优化

#### componentWillUpdate()
```
componentWillUpdate(nextProps, nextState)
```
- 触发时机：在更新阶段 shouldComponentUpdate 为 true 之后，render 阶段之前触发
- 不应该执行 setState 或者 redux 等触发组件更新的操作，否则会死循环
- 在 React 16.3 及以后，更名为 `UNSAFE_componentWillUpdate`，后续将会删除这个生命周期，可使用至 React 17

#### render()
```
render()
```
同挂载阶段的 render函数

#### componentDidUpdate()
```js
componentDidUpdate(prevProps, prevState, snapshot)
```
- 触发时机：在组件更新阶段，dom 完成更新挂载后立即执行。
- 组件更新后，可以再次获取到更新后的 dom 信息。
- 可以在这里对比更新的 props 或者 state，判断是否要请求数据更新
- 在这里可以直接使用 setState，但必需要有条件才更新组件，否则会引发更新组件的死循环
- （React 16.3及以后）如果组件实现了 getSnapshotBeforeUpdate() 生命周期（不常用），则它的返回值将作为 componentDidUpdate() 的第三个参数 “snapshot” 参数传递。

#### getDerivedStateFromProps()
同上面挂载阶段的描述

#### getSnapshotBeforeUpdate()
```js
getSnapshotBeforeUpdate(prevProps, prevState)
```
- 触发时机：在最近一次渲染输出（提交到 DOM 节点）之前调用
- 可以获取到组件更新前 dom 的一些信息（比如滚动信息）
- 返回值会作为 componentDidUpdate 的第三个参数 snapshot

官方文档上，获取更新前的滚动位置信息的一个例子：
```js
class ScrollingList extends React.Component {
  constructor(props) {
    super(props);
    this.listRef = React.createRef();
  }

  getSnapshotBeforeUpdate(prevProps, prevState) {
    // 我们是否在 list 中添加新的 items ？
    // 捕获滚动​​位置以便我们稍后调整滚动位置。
    if (prevProps.list.length < this.props.list.length) {
      const list = this.listRef.current;
      return list.scrollHeight - list.scrollTop;
    }
    return null;
  }

  componentDidUpdate(prevProps, prevState, snapshot) {
    // 如果我们 snapshot 有值，说明我们刚刚添加了新的 items，
    // 调整滚动位置使得这些新 items 不会将旧的 items 推出视图。
    //（这里的 snapshot 是 getSnapshotBeforeUpdate 的返回值）
    if (snapshot !== null) {
      const list = this.listRef.current;
      list.scrollTop = list.scrollHeight - snapshot;
    }
  }

  render() {
    return (
      <div ref={this.listRef}>{/* ...contents... */}</div>
    );
  }
}
```

### 卸载阶段
卸载阶段只有一个生命周期函数：
- componentWillUnmount

#### componentWillUnmount()
```js
componentWillUnmount()
```
- 触发时机：在组件卸载销毁前，执行该方法。
- 通常用于一些清理操作：清除定时器，取消数据订阅等

### 错误边界
在 React 16 后，若组件发生 js 错误，将不会显示错误的组件，而是显示一片空白。

通过错误边界，可以显示降级的UI 而不是显示错误的组件树或一片空白。错误边界是 React 的组件，可在子树中捕获到 javascript 错误，并记录这些错误。

错误边界在渲染期间、生命周期方法和整个组件树的构造函数中捕获错误。

错误边界不可以捕获以下错误：
- 事件处理
- 异步代码
- 服务端渲染
- 自身组件抛出的错误

#### static getDerivedStateFromError()
```js
static getDerivedStateFromError(error)
```
- 触发时机：在后代组件中抛出错误的时候触发
- 接受抛出的错误作为参数，返回的值将会更新state，可根据该 state 控制显示的降级UI
- 因为 `getDerivedStateFromProps` 会在渲染阶段调用，所以不允许使用副作用，若要实现副作用，请使用 `componentDidCatch`

根据错误，显示降级 UI 的例子：
```js
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    // 更新 state 使下一次渲染可以显降级 UI
    return { hasError: true };
  }

  render() {
    if (this.state.hasError) {
      // 你可以渲染任何自定义的降级  UI
      return <h1>Something went wrong.</h1>;
    }

    return this.props.children;
  }
}
```

#### componentDidCatch()
```js
componentDidCatch(error, info)
```
- 触发时机：在后代组件中抛出错误的时候触发，发生在 commit 阶段，因此可以使用副作用
- 返回的参数为
    - error: 错误信息
    - info: 带有 componentStack Key 的信息

官方文档的例子：
```js
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    // 更新 state 使下一次渲染可以显示降级 UI
    return { hasError: true };
  }

  componentDidCatch(error, info) {
    // "组件堆栈" 例子:
    //   in ComponentThatThrows (created by App)
    //   in ErrorBoundary (created by App)
    //   in div (created by App)
    //   in App
    logComponentStackToMyService(info.componentStack);
  }

  render() {
    if (this.state.hasError) {
      // 你可以渲染任何自定义的降级 UI
      return <h1>Something went wrong.</h1>;
    }

    return this.props.children;
  }
}
```

错误边界的使用方法：
```html
<ErrorBoundary>
    <Child></Child>
</ErrorBoundary>
```