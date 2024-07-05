---
title: Redux
toc: true
tags: JavaScript Redux
---

了解历史

- Flux：[https://www.ruanyifeng.com/blog/2016/01/flux.html](https://www.ruanyifeng.com/blog/2016/01/flux.html)


## 基本概念和API



Redux 引入的概念其实并不多，主要就是三个：State、Action 和 Reducer。

- 其中 State 即 Store，一般就是一个纯 JavaScript Object。
- Action 也是一个 Object，用于描述发生的动作。
- 而 Reducer 则是一个函数，接收 Action 和 State 并作为参数，通过计算得到新的 Store。

### Store

Store是保存数据的地方，你可以把它看成一个容器。整个应用只能有一个Store。

Redux提供createStore这个函数，用来生成Store。

```js
import { createStore } from 'redux';
const store = createStore(fn);



store.getState()
store.dispatch()
store.subscribe()

```

### State

Store对象包含所有数据。如果想得到某个时点的数据，就要对Store生成快照。这种时点的数据集合，就叫做State。

当前时刻的State，可以通过store.getState()拿到。

```js
import { createStore } from 'redux';
const store = createStore(fn);

const state = store.getState();
```

Redux规定，一个State对应一个View。只要State相同，View就相同。你知道State，就知道View是什么样，反之亦然。

### Action

State的变化，会导致View的变化。但是，用户接触不到State，只能接触到View。所以，State的变化必须是View导致的。Action就是View发出的通知，表示State应该要发生变化了。

Action是一个对象。其中的type属性是必须的，表示Action的名称。其他属性可以自由设置，社区有一个规范可以参考。

```js
{
  type: 'ADD_TODO',
  payload: 'Learn Redux'
}
```

### Action Creator

View要发送多少种消息，就会有多少种Action。如果都手写，会很麻烦。可以定义一个函数来生成Action，这个函数就叫Action Creator。

```js
const ADD_TODO = '添加 TODO';

function addTodo (text) {
  return {
    type: ADD_TODO,
    text
  }
}
```

### store.dispatch()

store.dispatch()是View发出Action的唯一方法。

```js
import { createStore } from 'redux';
const store = createStore(fn);

store.dispatch({
  type: 'ADD_TODO',
  payload: 'Learn Redux'
});
```

### Reducer

Store收到Action以后，必须给出一个新的State，这样View才会发生变化。这种State的计算过程就叫做Reducer。

Reducer是一个函数，它接受Action和当前State作为参数，返回一个新的State。

Reducer 必需符合以下规则：

- 仅使用 state 和 action 参数计算新的状态值
- 禁止直接修改 state。必须通过复制现有的 state 并对复制的值进行更改的方式来做 不可变更新（immutable updates）。
- 禁止任何异步逻辑、依赖随机值或导致其他“副作用”的代码

```js
const defaultState = 0;
const reducer = (state = defaultState, action) => {
  switch (action.type) {
    case 'ADD':
      return state + action.payload;
    default: 
      return state;
  }
};

const state = reducer(1, {
  type: 'ADD',
  payload: 2
});
```

### store.subscribe()

Store允许使用store.subscribe方法设置监听函数，一旦State发生变化，就自动执行这个函数。

```js
import { createStore } from 'redux';
const store = createStore(fn);

store.subscribe(listener);
```



![](./redux_1.jpg)

![](./redux_2.gif)




```shell

# 安装Redux Toolkit
npm install @reduxjs/toolkit
yarn add @reduxjs/toolkit

#
npx create-react-app my-app --template redux
npx create-react-app my-app --template redux-typescript

```


## React Redux


## React Toolkit


## 参考

- [https://redux-toolkit-cn.netlify.app/introduction/quick-start](https://redux-toolkit-cn.netlify.app/introduction/quick-start)
- [https://react-redux.js.org/introduction/getting-started](https://react-redux.js.org/introduction/getting-started)