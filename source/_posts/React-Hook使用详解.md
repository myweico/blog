---
title: React Hook使用详解
date: 2021-02-14 23:32:55
tags:
- React

categories:
- 前端框架

---
React Hook 是 React16.8 推出的新特性，给我们带来了许多的新特性。这篇主要讲解 React Hook 的使用以及 React Hook 的底层原理。

<!-- more -->

## 使用React Hooks 的动机
- 在组件之间复用状态逻辑很难
    - React 没有提供将可复用性行为“附加”到组件的途径，之前主要是通过 `render props` 和高阶组件
    - 需要重新组织组件的结构，代码难以理解
    - 会导致嵌套地狱
- 复杂组件的代码难以管理，理解，当业务逻辑复杂后，各个完整的逻辑会被分拆到各个生命周期里，容易导致bug，逻辑不一致
- 难以理解的 class
    - 需要管理 this，
    - 事件还要绑定 this，代码冗余
    - React 团队想使用预编译的技术，但是 class 组件会导致工具出现问题，class组件不容易压缩，热重载容易出现不稳定的情况

## Hooks 的用处
- 让函数组件也可以拥有组件状态、生命周期等
- 可以很简单的复用组件的状态
- 可以将相关的业务逻辑聚合在一起，代码清晰容易管理，不再像 class 的组件一样将业务逻辑分散到各个生命周期中

## Hooks 的使用
### useState
useState 用于让函数组件拥有自己的内部状态
```js
const [state, setState] = useState(initialState);
```
- initialState: 状态的初始值
- state: 状态值
- setState: 用于设置状态的函数

#### 更新状态
```js
setState(newState)
```

注意，setState 与 class 组件的 `this.setState` 不一样， hook 的 setState 不会自动帮你合并对象，所以需要合并之前的对象
```js
setState({
    ...prevState,
    ...updatedValues
})
```
#### 函数式更新
如果新的state需要依赖之前的state，则需要将函数传给setState
```js
function Counter({initialCount}) {
  const [count, setCount] = useState(initialCount);
  return (
    <>
      Count: {count}
      <button onClick={() => setCount(initialCount)}>Reset</button>
      <button onClick={() => setCount(prevCount => prevCount - 1)}>-</button>
      <button onClick={() => setCount(prevCount => prevCount + 1)}>+</button>
    </>
  );
}
```

#### 惰性初始值
若初始值需要通过复杂的操作得到，useState可以传入一个函数，函数的返回值将作为初始值
```js
const [state, setState] = useState(() => {
  const initialState = someExpensiveComputation(props);
  return initialState;
});
```

#### 跳过更新
调用更新函数，并传入当前 state 的时候，React 将会跳过子组件的渲染以及 Effect 的执行

### useEffect
```js
useEffect(updateFunc[, dependenciesArray])
```
该 Hook 接收一个包含命令式、且可能有副作用代码的函数。

#### 清除effect
第一个函数返回的函数，将会在组件卸载或者更新前执行，因此可返回一个函数用于清除 effect
```js
useEffect(() => {
  const subscription = props.source.subscribe();
  return () => {
    // 清除订阅
    subscription.unsubscribe();
  };
});
```

#### 执行时机
- effect 接受的函数，将会在每次渲染到屏幕后执行，所以不会阻塞屏幕渲染
- 若想在渲染前执行 effect，请使用 `useLayoutEffect`
- effect 函数默认会在组件挂载以及每次更新的时候都会执行，可添加依赖数组控制 effect 条件执行


#### 条件执行
通过 useEffect 第二个参数的依赖数组，控制 effect 的条件执行
```js
// 只有当 props.source 改变后才会重新创建订阅。
useEffect(
  () => {
    const subscription = props.source.subscribe();
    return () => {
      subscription.unsubscribe();
    };
  },
  [props.source],
);
```
- 依赖数组为 `[]` 时，表示只在组件挂载的时候执行 传入的函数，在卸载的时候执行传入函数返回的清除函数
- 确保传入函数依赖的动态值都在依赖数组里面，否则将会为初始值

### useLayoutEffect
函数签名与 useEffect 一致
```js
useLayoutEffect(updateFunc[, dependenciesArray])
```
- 执行时机：它会在所有的 DOM 变更之后同步调用 effect。在浏览器执行绘制之前，useLayoutEffect 内部的更新计划将被同步刷新。
- 可以使用它来读取 DOM 布局并同步触发重渲染

### useContext
```js
const value = useContext(MyContext);
```
- 接受一个 context 对象（React.createContext的返回值)并返回 context 的当前值。
- 当前的 context 值由上层组件中距离当前组件最近的 <MyContext.Provider> 的 value prop 决定。

使用例子
```js
const themes = {
  light: {
    foreground: "#000000",
    background: "#eeeeee"
  },
  dark: {
    foreground: "#ffffff",
    background: "#222222"
  }
};

const ThemeContext = React.createContext(themes.light);

function App() {
  return (
    <ThemeContext.Provider value={themes.dark}>
      <Toolbar />
    </ThemeContext.Provider>
  );
}

function Toolbar(props) {
  return (
    <div>
      <ThemedButton />
    </div>
  );
}

function ThemedButton() {
  const theme = useContext(ThemeContext);
  return (
    <button style={{ background: theme.background, color: theme.foreground }}>
      I am styled by theme context!
    </button>
  );
}
```

### useReducer
```js
const [state, dispatch] = useReducer(reducer, initialArg, init);
```
实现类似于 redux 的状态管理功能。接受参数如下：
- reducer: 类似于 redux 的 reducer，一个接受state和action的函数，返回新的 state
- initialArg: 初始的状态
- init: 对 initialArg 参数进行处理

使用例子
```js
function init(initialCount) {
  return {count: initialCount};
}

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return {count: state.count + 1};
    case 'decrement':
      return {count: state.count - 1};
    case 'reset':
      return init(action.payload);
    default:
      throw new Error();
  }
}

function Counter({initialCount}) {
  const [state, dispatch] = useReducer(reducer, initialCount, init);
  return (
    <>
      Count: {state.count}
      <button
        onClick={() => dispatch({type: 'reset', payload: initialCount})}>
        Reset
      </button>
      <button onClick={() => dispatch({type: 'decrement'})}>-</button>
      <button onClick={() => dispatch({type: 'increment'})}>+</button>
    </>
  );
}
```

### useMemo
```js
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```
接受一个函数和一个依赖数组，返回一个 memoized 值，即一个缓存的值。

当依赖数组里面的值发生变化时，才会重新执行函数，获取新的值。

使用例子
```js
import React, { useState, useMemo } from "react";
export default function UseMemoPage(props) {
  const [count, setCount] = useState(0);
  const expensive = useMemo(() => {
    console.log("compute");
    let sum = 0;
    for (let i = 0; i < count; i++) {
      sum += i;
    }
    return sum;
    //只有count变化，这⾥才重新执⾏
  }, [count]);
  const [value, setValue] = useState("");
  return (
    <div>
      <h3>UseMemoPage</h3>
      <p>expensive:{expensive}</p>
      <p>{count}</p>
      <button onClick={() => setCount(count + 1)}>add</button>
      <input value={value} onChange={(event) => setValue(event.target.value)} />
    </div>
  );
}
```

### useCallback
```js
const memoizedCallback = useCallback(
  () => {
    doSomething(a, b);
  },
  [a, b],
);
```
- 传入一个函数以及依赖项。当依赖项的值发生变化的时候，才会生成一个新的函数。
- 当你把回调函数传递给经过优化的并使用引用相等性去避免非必要渲染（例如 shouldComponentUpdate）的子组件时，它将非常有用。
- `useCallback(fn, [a, b])` 相当于 `useMemo(() => fn, [a, b])`

使用例子
```
import React, { useState, useCallback, PureComponent } from "react";

export default function UseCallbackPage(props) {
  const [count, setCount] = useState(0);
  const addClick = useCallback(() => {
    let sum = 0;
    for (let i = 0; i < count; i++) {
      sum += i;
    }
    return sum;
  }, [count]);
  const [value, setValue] = useState("");
  return (
    <div>
      <h3>UseCallbackPage</h3> <p>{count}</p>
      <button onClick={() => setCount(count + 1)}>add</button>
      <input value={value} onChange={(event) => setValue(event.target.value)} />
      <Child addClick={addClick} />
    </div>
  );
}

class Child extends PureComponent {
  render() {
    console.log("child render");
    const { addClick } = this.props;
    return (
      <div>
        <h3>Child</h3>
        <button onClick={() => console.log(addClick())}>add</button>
      </div>
    );
  }
}
```

### useRef
```js
const refContainer = useRef(initialValue);
```
`useRef`返回一个可变的 ref 对象。其 current 属性被初始化为 传入的参数。ref对象在组件的整个生命周期不变。

常见的用例就是访问子组件
```js
function TextInputWithFocusButton() {
  const inputEl = useRef(null);
  const onButtonClick = () => {
    // `current` 指向已挂载到 DOM 上的文本输入元素
    inputEl.current.focus();
  };
  return (
    <>
      <input ref={inputEl} type="text" />
      <button onClick={onButtonClick}>Focus the input</button>
    </>
  );
}
```
useRef() 比 ref 属性更有用。它可以很方便地保存任何可变值，其类似于在 class 中使用实例字段的方式。

请记住，当 ref 对象内容发生变化时，useRef 并不会通知你。变更 .current 属性不会引发组件重新渲染。如果想要在 React 绑定或解绑 DOM 节点的 ref 时运行某些代码，则需要使用回调 ref 来实现。
```js
function MeasureExample() {
  const [height, setHeight] = useState(0);

  const measuredRef = useCallback(node => {
    if (node !== null) {
      setHeight(node.getBoundingClientRect().height);
    }
  }, []);

  return (
    <>
      <h1 ref={measuredRef}>Hello, world</h1>
      <h2>The above header is {Math.round(height)}px tall</h2>
    </>
  );
}
```

### useImperativeHandle
```js
useImperativeHandle(ref, createHandle, [deps])
```
- useImperativeHandle 可以让你在使用 ref 时自定义暴露给父组件的实例值。在大多数情况下
- useImperativeHandle 应当与 forwardRef 一起使用：
```js
function FancyInput(props, ref) {
  const inputRef = useRef();
  useImperativeHandle(ref, () => ({
    focus: () => {
      inputRef.current.focus();
    }
  }));
  return <input ref={inputRef} ... />;
}
FancyInput = forwardRef(FancyInput);
```
在本例中，渲染 `<FancyInput ref={inputRef} />` 的父组件可以调用 `inputRef.current.focus()`。

### 自定义 hook
自定义 hook 可以提取出共同的 state 以及 逻辑，封装成可以复用的状态以及逻辑

规则：
- 自定义 hook 需要时一个以use开头的函数

自定义 hook
```js
import { useState, useEffect } from 'react';

function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);

  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }

    ChatAPI.subscribeToFriendStatus(friendID, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(friendID, handleStatusChange);
    };
  });

  return isOnline;
}
```
使用自定义 hook
```js
function FriendStatus(props) {
  const isOnline = useFriendStatus(props.friend.id);

  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}
```

```js
function FriendListItem(props) {
  const isOnline = useFriendStatus(props.friend.id);

  return (
    <li style={{ color: isOnline ? 'green' : 'black' }}>
      {props.friend.name}
    </li>
  );
}
```

### useDebugValue
```js
useDebugValue(value, initFunc)
```
useDebugValue 可用于在 React 开发者工具中显示自定义 hook 的标签。
- value: debug 值
- initFunc: 格式化函数，它接受 debug 值作为参数，并且会返回一个格式化的显示值。

一个栗子：
```js
function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);

  // ...

  // 在开发者工具中的这个 Hook 旁边显示标签
  // e.g. "FriendStatus: Online"
  useDebugValue(isOnline ? 'Online' : 'Offline');

  return isOnline;
}

// 使用延迟格式化的值
useDebugValue(date, date => date.toDateString());
```

## Hook 规则
- 只在最顶层调用 Hook，不要在循环，条件语句中使用 Hook
- 只在 React 函数组件或者自定义 Hook 中使用

可以使用 `eslint-plugin-react-hooks` 的 ESlint 插件执行这两条规则

安装
```
npm install eslint-plugin-react-hooks --save-dev
```

配置 Eslint
```
// 你的 ESLint 配置
{
  "plugins": [
    // ...
    "react-hooks"
  ],
  "rules": {
    // ...
    "react-hooks/rules-of-hooks": "error", // 检查 Hook 的规则
    "react-hooks/exhaustive-deps": "warn" // 检查 effect 的依赖
  }
}
```
