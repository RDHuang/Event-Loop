## JS中的宏任务微任务

首先js一大特点就是单线程，同一时间只能干一件事情。所以为了协调事件，用户交互，脚本，ui渲染和网络处理等行为，防止线程不堵塞，Event Loop的方案应用而生。
Event Loop包含两大类： 一类是Browsing Context（浏览器上下文），一类是基于Worker。二者的运行都是独立的，也就是说，每一个javascript运行的‘线程环境‘都有一个独立的Event Loop，每一个web worker也是一个独立的Event Loop 

### Browsing Content

- 任务队列

    根据规范，事件循环是根据任务队列的机制来进行协调的。一个Event Loop中，可以有一个或多个任务队列(task queue)， 一个任务队列便是一系列有序任务(task)的集合；每一个任务都有一个任务源(task source), 源自同一任务源的 task 必须放在同一任务队列中，从不同源来的则被添加到不同队列。setTimeout/Promise等API便是任务源，而进入任务队列的是他们指定的具体执行任务。

- 宏任务

  (macro)task,可以理解是每次执行栈执行的代码就是一个宏任务（包括每次从事件队列中获取的一个事件回调，并放到执行栈中执行）

  浏览器为了能够使得JS内部(macro)task与DOM任务能够有序的执行，会在一个 宏任务执行结束后，在下一个宏任务执行开始钱，对页面进行重新渲染。所以考虑到性能方面，尽量少用宏任务。每次宏任务执行完之后都会自动触发浏览器的页面刷新

    > 宏任务包含：
    ```
    setTimeout
    setInterval
    script
    I/O
    UI交互事件
    postMessage
    MessageChannel
    setImmediate(Nodejs环境)
    ```

- 微任务

  microtask，可以理解成在当前task执行结束后立即执行的任务，也就是说，在当前task任务后，下一个task执行之前，在渲染之前。

  所以微任务的响应速度会更快，因为无需等待渲染。

    > 微任务包含

    ```
    Promise.then
    Object.observe
    MutaionObserve
    process.nextTick(Node.js环境)
    ```

- 运行机制

  在事件循环中，每进行一次循环操作成为tick，每一次tick的任务处理模型都是比较复杂的，但关键步骤如下：

    - 执行一个宏任务（栈中没有就从事件队列中获取）
    - 执行过程中如果遇到微任务，就将它添加到微任务的任务队列中
    - 宏任务执行完毕后，立即执行当前微任务队列中所有的微任务，按顺序执行
    - 当前宏任务执行完毕，开始检查渲染，然后GUI线程接管渲染
    - 渲染完毕后，JS线程继续接管，开始下一个宏任务（从事件队列中获取)

    ```
    宏任务 --->  执行结束 ---> 有微任务？
    Y --->  执行所有微任务 --->  浏览器渲染
    N --->  浏览器渲染
    ```

> 实战面试题

下面代码执行结果
```js
console.log('script start');

setTimeout(function () {
  console.log('setTimeout');
}, 0);

Promise.resolve()
  .then(function () {
    console.log('promise1');
  })
  .then(function () {
    console.log('promise2');
  });

console.log('script end');
```

结果为：
```log
script start
script end
promise1
promise2
setTimeout
```

> 难度升级

```html
<div class="outer">
  <div class="inner"></div>
</div>
```

```js
// Let's get hold of those elements
var outer = document.querySelector('.outer');
var inner = document.querySelector('.inner');

// Let's listen for attribute changes on the
// outer element
new MutationObserver(function () {
  console.log('mutate');
}).observe(outer, {
  attributes: true,
});

// Here's a click listener…
function onClick() {
  console.log('click');

  setTimeout(function () {
    console.log('timeout');
  }, 0);

  Promise.resolve().then(function () {
    console.log('promise');
  });

  outer.setAttribute('data-random', Math.random());
}

// …which we'll attach to both elements
inner.addEventListener('click', onClick);
outer.addEventListener('click', onClick);
```

当点击内部的class为inner的div时，会打印什么？
```js
click
pormise
mutate
click
promise
mutate
timeout
timeout
```

当点击外部的class为outer的div时，会打印什么？
```js
click
promise
mutate
timeout
```

最后附上一个[网址链接](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/)，国外一个关于宏任务微任务的详解，会有动画可以很好的帮助理解执行时机


