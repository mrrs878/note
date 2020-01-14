---
title: JavaScript-执行机制
date: 2019-06-30 11:15:28
tags: JavaScript 执行机制 EventLoop
categories: JavaScript
---

# **JavaScript 执行与运行**

执行和运行不太相同，在不同环境下，比如 Node、浏览器下，JavaScript 的执行结果是不一样的；而运行大多指的是基于 JavaScript 引擎，如 V8，结果是一致的

# **关于 JavaScript**

众所周知，JavaScript 是一门**单线程**、**异步执行**的语言，虽然在 HTML5 中提出了 **web workers**(可以理解为**浏览器**为 JavaScript 开的“外挂”，下一篇会谈)，但 JavaScript 是单线程运行的这一核心仍未改变，所有**多线程**都是通过单线程模拟出来的，都是“纸老虎”

# JavaScript 中的异步--event loop

JavaScript 中的异步主要通过 event loop 进行模拟，那么什么是 event loop 呢？当我们执行 JS 代码的时候其实就是往执行栈中放入函数，那么遇到异步代码的时候该怎么办？其实当遇到异步的代码时，会被挂起并在需要执行的时候加入到 Task（有多种 Task） 队列中。一旦执行栈为空，Event Loop 就会从 Task 队列中拿出需要执行的代码并放入执行栈中执行，所以本质上来说 JS 中的异步还是同步行为。

## 浏览器中的 event loop

![浏览器中的event loop](/../imgs/browser_event_loop.png)
![浏览器中的event loop](/../imgs/browser_event_loop2.png)
不同的任务源会被分配到不同的 Task 队列中，任务源可以分为 微任务（microtask） 和 宏任务（macrotask）。在 ES6 规范中，microtask 称为 jobs，macrotask 称为 task。
微任务包括 process.nextTick ，promise ，MutationObserver。
宏任务包括 script ，setTimeout ，setInterval ，setImmediate ，I/O ，UI rendering。

**宏任务中包括了 script ，浏览器会先执行一个宏任务，接下来有异步代码的话才会先执行微任务。**

Event Loop 执行顺序如下所示：

- 首先执行同步代码，这属于宏任务
- 当执行完所有同步代码后，执行栈为空，查询是否有异步代码需要执行
- 执行所有微任务
- 当执行完所有微任务后，如有必要会渲染页面
- 然后开始下一轮 Event Loop，执行宏任务中的异步代码，也就是 setTimeout(宏任务) 中的回调函数

```javascript
console.log(1);
setTimeout(() => {
  console.log(2);
  Promise.resolve().then(() => {
    console.log(3);
  });
});
new Promise((resolve, reject) => {
  console.log(4);
  resolve(5);
}).then(data => {
  console.log(data);
});
setTimeout(() => {
  console.log(6);
});
console.log(7);

// 执行结果：1 4 7 5 2 3 6
```

## Node.js 中的 event loop

Node 的 Event Loop 分为 6 个阶段，它们会按照顺序反复运行。每当进入某一个阶段的时候，都会从对应的回调队列中取出函数去执行。当**队列为空**或者**执行的回调函数数量到达系统设定的阈值**，就会进入下一阶段。
![Node.js中的event loop](/../imgs/node_event_loop.png)

- timer
  timers 阶段会执行 setTimeout 和 setInterval 回调，并且是由 **poll** 阶段控制的。
  同样，在 Node 中定时器指定的时间也不是准确时间，只能是尽快执行。
- I/O
  I/O 阶段会处理一些上一轮循环中的少数未执行的 I/O 回调
- idle, prepare
  idle, prepare 阶段内部实现，这里就忽略不讲了。
- poll
  poll 是一个至关重要的阶段，这一阶段中，系统会做两件事情
  - 回到 timer 阶段执行回调
  - 执行 I/O 回调
    并且在进入该阶段时如果没有设定了 timer 的话，会发生以下两件事情
  - 如果 poll 队列不为空，会遍历回调队列并同步执行，直到队列为空或者达到系统限制
  - 如果 poll 队列为空时，会有两件事发生
    - 如果有 setImmediate 回调需要执行，poll 阶段会停止并且进入到 check 阶段执行回调
    - 如果没有 setImmediate 回调需要执行，会等待回调被加入到队列中并立即执行回调，这里同样会有个超时时间设置防止一直等待下去
      当然设定了 timer 的话且 poll 队列为空，则会判断是否有 timer 超时，如果有的话会回到 timer 阶段执行回调。
- check
  check 阶段执行 setImmediate
- close callbacks
  close callbacks 阶段执行 close 事件
  对于 microtask 来说，它会在以上每个阶段完成前清空 microtask 队列，下图中的 Tick 就代表了 microtask
  ![Node中的event loop](/../imgs/node_event_loop2.png)
