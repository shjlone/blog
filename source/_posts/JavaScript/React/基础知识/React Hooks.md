---
title: React Hooks
tags: React
---



## React Hooks产生的原因

Functional（Stateless）Component，功能组件也叫无状态组件，一般只负责渲染。

```JavaScript

function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}
```

Class（Stateful）Component，类组件也是有状态组件，一般有交互逻辑和业务逻辑。

```JavaScript
class Welcome extends React.Component {
    state = {
        name: ‘tori’,
    }
  componentDidMount() {
        fetch(…);
        …
    }
  render() {
    return (
        <>
            <h1>Hello, {this.state.name}</h1>
            <button onClick={() => this.setState({name: ‘007’})}>改名</button>
        </>
      );
  }
}

```


Presentational Component则是和功能组件类似

```JavaScript
const Hello = (props) => {
  return (
    <div>
      <h1>Hello! {props.name}</h1>
    </div>
  )
}

```

![](./hooks_1.png)

- useState：状态管理，某个状态发生变化，组件重新渲染，使用该状态的组件及其子组件都会重新渲染

```JavaScript
function App() {
  return (
    <div>
      <ComponentA />
      <ComponentB />
    </div>
  );
}

function ComponentA() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Click me</button> // ComponentA、ComponentC会重新渲染
      <ComponentC />
    </div>
  );
}

function ComponentB() {
  // ...
}

function ComponentC() {
  // ...
}
```

- useEffect： 在**函数组件中**执行副作用操作，方便，且避免不必要的bug

```JavaScript
import React, { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const timerId = setInterval(() => {
      setCount(prevCount => prevCount + 1);
    }, 1000);

    // 返回一个清理函数，执行时机：1. 组件卸载时 2. 依赖数组发生变化时
    return () => {
      clearInterval(timerId);
    };
  }, []); // 依赖数组为空，副作用函数只在首次渲染后执行

  return <div>{count}</div>;
}
```

- useRef：用于在不进行渲染的情况下存储和访问最新的值

1. 访问 DOM 元素：你可以将 ref 对象赋值给 JSX 元素的 ref 属性，然后在其他地方通过 .current 属性访问这个元素。例如：

```JavaScript
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

2. 存储可变值：ref 对象的 .current 属性是可变的，它可以用来存储任何可变值，而不仅仅是 DOM 元素。与在组件中使用 state 不同，修改 ref 对象的 .current 属性不会触发组件重新渲染。例如：

```JavaScript
function Timer() {
  const intervalId = useRef();

  useEffect(() => {
    intervalId.current = setInterval(() => {
      console.log('Timer tick');
    }, 1000);

    return () => {
      clearInterval(intervalId.current);
    };
  }, []);

  // ...
}
```

- useReducer：接受一个reducer函数和一个初始值，返回新的状态和一个dispatch函数，通过调用dispatch函数来更新状态

```JavaScript
import React, { useReducer } from 'react';

const initialState = {count: 0};

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return {count: state.count + 1};
    case 'decrement':
      return {count: state.count - 1};
    default:
      throw new Error();
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);
  return (
    <>
      Count: {state.count}
      <button onClick={() => dispatch({type: 'decrement'})}>-</button>
      <button onClick={() => dispatch({type: 'increment'})}>+</button>
    </>
  );
}
```

- useCallback：


- useImperativeHandle

React Hooks组件从v16.8开始

- [官方文档Introducing Hooks](https://legacy.reactjs.org/docs/hooks-intro.htm)
- [React Hooks 入门教程](https://www.ruanyifeng.com/blog/2019/09/react-hooks.html)
- [使用 Effect Hook](https://zh-hans.legacy.reactjs.org/docs/hooks-effect.html)
- [https://segmentfault.com/a/1190000021261588](https://segmentfault.com/a/1190000021261588)
- [https://juejin.cn/post/7118937685653192735](https://juejin.cn/post/7118937685653192735)