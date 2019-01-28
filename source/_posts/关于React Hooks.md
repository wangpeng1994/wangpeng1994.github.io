---
title: 关于React Hooks
date: 2019-01-28 16:18:28
categories:
  - react
tags:
  - react
  - react hooks
---


研习 react hooks 文档后，一些个人认为值得理解和关注的内容。


## useState

- react hooks 令 react 函数式组件拥有了类似 class 组件的 state 状态变量，以往观念中，class 组件被认为是所谓的“有状态的”组件，而函数式组件是无状态的组件，现在有了 react hooks 后，函数式组件就是函数式组件，并且也能在多次 re-render 中读写某个 state；
- 关于 react hooks 为什么调用 `useState` 并没有传入所谓的 `this` 来指定声明的 state 变量是属于当前的组件，却依然令函数组件拥有了内部状态？ 
因为 react 内部会自动维护一份 “记忆单元” 对象，用来维护记录组件的 state 变量；
- `useState` hooks 中的状态更新，是新值完全覆盖旧值，而不像 class 组件中那样，内部对比 state 不同属性的差异后部分合并；


## useEffect

- 和 `useState` 一样必须在 react 函数式组件中调用；
-  `useEffect` 是异步的，DOM变化后就会执行，不会阻塞浏览器渲染；
- `useEffect` 类似于 `componentDidMount`、`componentDidUpdate` 和`componentWillUnmount`的结合；
- 包括首次 render 在内的每次 re-render 后都会被调用；
- 每次将要 re-render 时，react 函数组件会重新执行，`useEffect` 调用时如果传入了 state，则都是最新的 state，因此每次的 ·useEffect· 调用，传入的 effect 都 “专属于” 当前的 render；
- ``` js
  useEffect(() => {
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    // Specify how to clean up after this effect:
    return function cleanup() {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });
  ```
  `useEffect` 传入的函数中，可以 return 一个函数，effect 在每次 render 后被执行，而 effect 中 return 的函数，会在每次组件卸载后被执行，这个可选机制可用来做 cleanup，不需要的话，就不用指定函数的返回；


## Hooks 使用规范

- 只在函数组件内顶层、或者自定义 hooks 函数内调用，不要再循环、条件判断（但可以在 effect 函数内部做判断）或者普通的嵌套函数里调用。不仅是为了 react 内部依序维持 hooks，也令状态逻辑更清晰；
- 必须保证首次 render 和之后的 re-render 时调用 hooks 顺序是一致的，首次 render 时依次调用，`useState` 是初始化状态变量，`useEffect` 是添加 effect 函数；之后 re-render 时也是依次调用，`useState` 是读取状态变量（传入的参数被忽略），`useEffect` 是替换新的 effect 函数（因为可能为 effect 传入了最新的 state）。可见 react 调用 hooks 时需要保证每次调用顺序都一样；


## 自定义 Hooks

- 所谓自定义 hooks 就是遵守规范以 `use` 开头的、内部可能调用其他 hooks 的普通 js 函数，一般作为公共状态逻辑函数，可以传入任何需要的参数，并返回任何需要的结果，除此之外，外观上和 使用了 hooks 的 react 函数式组件一样：
  ```
  function useFriendStatus(friendID) {
    const [isOnline, setIsOnline] = useState(null);
    // ...
    return isOnline;
  }
  ```
- 自定义 hooks 使用时直接在 react 函数组件内调用即可：
  ```
    function FriendStatus(props) {
    const isOnline = useFriendStatus(props.friend.id);

    if (isOnline === null) {
      return 'Loading...';
    }
    return isOnline ? 'Online' : 'Offline';
  }
  ```
- 在不同函数组件里调用同一个自定义 hooks 时，并不会因此而共享内部状态，不同 react 函数组件之间是独立的。调用自定义 hooks 本质就是在函数组件作用域内给 hooks 函数传入参数执行 hooks 内部的语句：`useState()`、`useEffect()` 等，一个函数组件内是可以多次调用 hooks 的，只要保证每次执行顺序一致就ok；


## 结束

`useState`、`useEffect` 是最基础两个内置 hooks，更多使用详情，请查阅 [react hooks 文档](https://reactjs.org/docs/hooks-intro.html)。