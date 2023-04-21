---
layout: post
title: React.StrictMode
categories: front-end
description: React.StrictMode
keywords: react
---

在新公司使用React做开发已经一年多了，期间也解决过解决过一些复杂问题，例如多次setState只有最后一次生效的问题。

在去年，React官网也发生了一次大更新，今天在看的时候发现了一个很有意思的东西

## 案例

```jsx
let guest = 0;

function Cup() {
  // Bad: changing a preexisting variable!
  guest = guest + 1;
  return <h2>Tea cup for guest #{guest}</h2>;
}

export default function TeaSet() {
  return (
    <>
      <Cup />
      <Cup />
      <Cup />
    </>
  );
}
```

下面是render 的结果：

> ## Tea cup for guest #2
>
> ## Tea cup for guest #4
>
> ## Tea cup for guest #6

可以发现，其中的每一句guest+1都执行了两次

问了很多同事和前端同行，表示都不太理解

终于，在github的issue里面找到了相关的解释：

[官方github仓库]: https://github.com/facebook/react/issues/17786	"Concurrent mode renders components twice with different identities"

> This is not a bug. And you'll have the same behavior in [Strict Mode](https://reactjs.org/docs/strict-mode.html) too. We intentionally double-call render-phase lifecycles **in development only** (and function components using Hooks) to help people find issues caused by side effects in render. In our experience, them firing twice is enough for people to notice and fix such bugs.
>
> If component output is always a function of props and state (and not outer scope variables, like in your example), the double rendering in development should have no observable effect.

在开发模式下，render函数会执行两次，已帮助开发者发现副作用的代码

其实，这种写法本身就是React官方不提倡的。React 的渲染应该是纯粹的，组件应该只 **返回** 它们的 JSX，而不 **改变** 在渲染前，就已存在的任何对象或变量。

因此，这种现象有两种解决方法，个人推荐第一种：

1. 避免严格模式下使用在函数调用中产生副作用的代码
2. 不使用严格模式