---

layout: post

title: Vue.nextTick

categories:front-end

description: Vue.nextTick

keywords: vue
---



在公司实习的项目中遇到了一个叫做`Vue.nextTick()`的东西，感觉有点意思，想了解一下。

### 1.前言

先来看一个案例:

```html
<div class="app">
  <div ref="msgDiv">{{msg}}</div>
  <div v-if="msg1">Message got outside $nextTick: {{msg1}}</div>
  <div v-if="msg2">Message got inside $nextTick: {{msg2}}</div>
  <div v-if="msg3">Message got outside $nextTick: {{msg3}}</div>
  <button @click="changeMsg">
    Change the Message
  </button>
</div>
```

Vue:

```js
new Vue({
  el: '.app',
  data: {
    msg: 'Hello Vue.',
    msg1: '',
    msg2: '',
    msg3: '',
  },
  methods: {
    changeMsg() {
      this.msg = 'Hello world.';
      this.msg1 = this.$refs.msgDiv.innerHTML;
      this.$nextTick(() => {
        this.msg2 = this.$refs.msgDiv.innerHTML;
      });
      this.msg3 = this.$refs.msgDiv.innerHTML;
    },
  },
});
```

点击前：![image-20200726230758677](2020-07-26-vue-nextTick.assets/image-20200726230758677.png)

点击后：![image-20200726230945283](2020-07-26-vue-nextTick.assets/image-20200726230945283.png)

从图中可以得知：msg1和msg3显示的内容还是变换之前的，而msg2显示的内容是变换之后的。其根本原因是因为Vue中DOM更新是异步的

### 2.应用场景

- 在Vue生命周期的`created()`钩子函数进行的DOM操作一定要放在`Vue.nextTick()`的回调函数中
- 在数据变化后要执行的某个操作，而这个操作需要使用随数据改变而改变的DOM结构的时候，这个操作都应该放进`Vue.nextTick()`的回调函数中。

解释：在`created()`钩子函数执行的时候DOM 其实并未进行任何渲染，而此时进行DOM操作无异于徒劳，所以此处一定要将DOM操作的js代码放进`Vue.nextTick()`的回调函数中。与之对应的就是`mounted()`钩子函数，因为该钩子函数执行时所有的DOM挂载和渲染都已完成，此时在该钩子函数中进行任何DOM操作都不会有问题 。

官方文档中这样说道：

> Vue 异步执行 DOM 更新。只要观察到数据变化，Vue 将开启一个队列，并缓冲在同一事件循环中发生的所有数据改变。如果同一个 watcher 被多次触发，只会被推入到队列中一次。这种在缓冲时去除重复数据对于避免不必要的计算和 DOM 操作上非常重要。然后，在下一个的事件循环“tick”中，Vue 刷新队列并执行实际 (已去重的) 工作。Vue 在内部尝试对异步队列使用原生的 `Promise.then` 和`MessageChannel`，如果执行环境不支持，会采用 `setTimeout(fn, 0)`代替。

> 例如，当你设置`vm.someData = 'new value'`，该组件不会立即重新渲染。当刷新队列时，组件会在事件循环队列清空时的下一个“tick”更新。多数情况我们不需要关心这个过程，但是如果你想在 DOM 状态更新后做点什么，这就可能会有些棘手。虽然 Vue.js 通常鼓励开发人员沿着“数据驱动”的方式思考，避免直接接触 DOM，但是有时我们确实要这么做。为了在数据变化之后等待 Vue 完成更新 DOM ，可以在数据变化之后立即使用`Vue.nextTick(callback)` 。这样回调函数在 DOM 更新完成后就会调用。

### 3.机制

其实官方的解释已经说明了一个问题：**Vue不是数据发生变化后DOM立即变化，而是通过一定的策略进行DOM更新。即异步更新**

（1）所有**同步任务**都在主线程上执行，形成一个执行栈（execution context stack）。
（2）主线程之外，还存在一个**“任务队列”（task queue）**。只要**异步任务有了运行结果**，就在”任务队列”之中**放置**一个事件。
（3）一旦”执行栈”中的所有**同步任务执行完毕**，系统就会读取”任务队列”，看看里面有哪些事件。那些对应的异步任务，于是结束等待状态，进入执行栈，开始执行。
（4）主线程不断**重复**上面的第三步。

![image-20200726231727714](2020-07-26-vue-nextTick.assets/image-20200726231727714.png)

#### 还有一个例子可以说明问题：

![image-20200726231814282](2020-07-26-vue-nextTick.assets/image-20200726231814282.png)

事件循环过程如下：

- 第一个tick（上图的第一个黑方块，即本次更新循环）
  - 首先修改数据，这是同步的任务。同一事件循环的所有同步任务都在主线程上执行，形成一个执行栈，此时仍未涉及DOM
  - 侦听到数据变化，Vue 将开启一个队列，并缓冲在同一事件循环中发生的所有数据变更。如果同一个watcher被多次的触发，只会被推入到队列中一次。
- 第二个tick（上图的第二个黑方块，即’下次更新循环’）
  - 同步任务执行完毕，开始执行异步watcher队列的任务，更新DOM。Vue在内部尝试对异步队列使用原生的`Promise.then()`和`MessageChannel`方法，如果环境不支持，会使用`setTimeout(fn,0)`代替。
- 第三个tick（最后一个方块）
  - 此时便是‘下次DOM更新循环结束后’
  - 此时可以通过`Vue.nextTick`获取到改变之后的DOM。

### 4.源码小解析

```js
/**
 * Defer a task to execute it asynchronously.
 */
export const nextTick = (function () {
  const callbacks = []
  let pending = false
  let timerFunc

  function nextTickHandler () {
    pending = false
    const copies = callbacks.slice(0)
    callbacks.length = 0
    for (let i = 0; i < copies.length; i++) {
      copies[i]()
    }
  }

  // the nextTick behavior leverages the microtask queue, which can be accessed
  // via either native Promise.then or MutationObserver.
  // MutationObserver has wider support, however it is seriously bugged in
  // UIWebView in iOS >= 9.3.3 when triggered in touch event handlers. It
  // completely stops working after triggering a few times... so, if native
  // Promise is available, we will use it:
  /* istanbul ignore if */
  if (typeof Promise !== 'undefined' && isNative(Promise)) {
    var p = Promise.resolve()
    var logError = err => { console.error(err) }
    timerFunc = () => {
      p.then(nextTickHandler).catch(logError)
      // in problematic UIWebViews, Promise.then doesn't completely break, but
      // it can get stuck in a weird state where callbacks are pushed into the
      // microtask queue but the queue isn't being flushed, until the browser
      // needs to do some other work, e.g. handle a timer. Therefore we can
      // "force" the microtask queue to be flushed by adding an empty timer.
      if (isIOS) setTimeout(noop)
    }
  } else if (!isIE && typeof MutationObserver !== 'undefined' && (
    isNative(MutationObserver) ||
    // PhantomJS and iOS 7.x
    MutationObserver.toString() === '[object MutationObserverConstructor]'
  )) {
    // use MutationObserver where native Promise is not available,
    // e.g. PhantomJS, iOS7, Android 4.4
    var counter = 1
    var observer = new MutationObserver(nextTickHandler)
    var textNode = document.createTextNode(String(counter))
    observer.observe(textNode, {
      characterData: true
    })
    timerFunc = () => {
      counter = (counter + 1) % 2
      textNode.data = String(counter)
    }
  } else {
    // fallback to setTimeout
    /* istanbul ignore next */
    timerFunc = () => {
      setTimeout(nextTickHandler, 0)
    }
  }

  return function queueNextTick (cb?: Function, ctx?: Object) {
    let _resolve
    callbacks.push(() => {
      if (cb) {
        try {
          cb.call(ctx)
        } catch (e) {
          handleError(e, ctx, 'nextTick')
        }
      } else if (_resolve) {
        _resolve(ctx)
      }
    })
    if (!pending) {
      pending = true
      timerFunc()
    }
    if (!cb && typeof Promise !== 'undefined') {
      return new Promise((resolve, reject) => {
        _resolve = resolve
      })
    }
  }
})()
```

其中`nextTickHandler()`函数用来执行`callbacks`里存储的所有回调函数。

接下来是将触发方式赋值给`timerFunc`。

- 先判断是否原生支持promise，如果支持，则利用promise来触发执行回调函数；
- 否则，如果支持MutationObserver，则实例化一个观察者对象，观察文本节点发生变化时，触发执行所有回调函数。
- 如果都不支持，则利用setTimeout设置延时为0。

#### 这里有一个新名词：MutationObserver

`MutationObserver`是HTML5中的新API，是个用来监视DOM变动的接口。他能监听一个DOM对象上发生的子节点删除、属性修改、文本内容修改等等。

用的时候要绑个回调函数：

```js
var domTarget = 你想要监听的dom节点
var mo = new MutationObserver(callback)
mo.observe(domTarget, {
      characterData: true //说明监听文本内容的修改。
})
```

![image-20200726232538728](2020-07-26-vue-nextTick.assets/image-20200726232538728.png)

在`nextTick`中 `MutationObserver`的作用就如上图所示。在监听到DOM更新后，调用回调函数。

其实使用 `MutationObserver`的原因就是 `nextTick`想要一个异步API，用来在当前的同步代码执行完毕后，执行想执行的异步回调，包括`Promise`和 `setTimeout`都是基于这个原因。

但是深入的话，就要涉及到同步任务和异步任务（包括弘任务和微任务）了。