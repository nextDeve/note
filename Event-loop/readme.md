### 定义

- **Event Loop：**即事件循环，是指浏览器或`Node`的一种解决`javaScript`单线程运行时不会阻塞的一种机制，也就是我们经常使用**异步**的原理。

- **进程：**进程是计算机中的程序关于某数据集合上的一次运行活动，是系统进行资源分配和调度的基本单位，是操作系统结构的基础，进程是线程的容器（来自百科）。进程是资源分配的最小单位。我们启动一个服务、运行一个实例，就是开一个服务进程，例如 `Java` 里的 `JVM` 本身就是一个进程，`Node.js` 里通过 `node app.js` 开启一个服务进程，多进程就是进程的复制（`fork`），`fork` 出来的每个进程都拥有自己的独立空间地址、数据栈，一个进程无法访问另外一个进程里定义的变量、数据结构，只有建立了 `IPC `通信，进程之间才可数据共享。

- **线程：**是操作系统能够进行运算调度的最小单位，首先我们要清楚线程是隶属于进程的，被包含于进程之中。**一个线程只能隶属于一个进程，但是一个进程是可以拥有多个线程的**。

### 浏览器模型

- 浏览器是一个多进程模型，**每个页卡都是一个独立的进程**。
- 常见的线程：
  - GUI渲染（页面渲染：绘制、绘图、3d动画）
  - JS渲染引擎（执行js代码）
  - 事件触发线程 EventLoop
  - webApi也会创建线程  事件、定时器、ajax请求
  - webWorker

- Chrome中的线程模型：
  - Chrome中发起HTTP发起请求最多可以使用6个并发的线程。
  - Chrome中负责向页面中执行绘制任务（执行HTML/CSS/JS代码）的只有一个线程——UI主线程。

### 宏任务(MacroTask)与微任务(微任务)

- **宏任务**
  -   执行脚本 `script`
  -   `setTimeout`
  -   `setInterval`
  -   `setImmediate`（浏览器暂时不支持，只有IE10支持，具体可见[`MDN`](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FAPI%2FWindow%2FsetImmediate)）
  -   `I/O` (文件读写)
  -   `UI Rendering`
  -   `Ajax`
  -   `MessageChannel`
- **微任务**
  - `promise`
  - `Process.nextTick`（`Node`独有）
  - `Object.observe`(废弃)
  - `MutationObserver`
- **渲染相关**(不算事件环里的事件)
  - `requestAnimationFrame`(页面渲染之前执行)
  - `requestIDleCallback`(空闲时间执行)

### Event Loop

- `Javascript` 有一个 `main thread` 主线程和 `call-stack` 调用栈(执行栈)，所有的任务都会被放到调用栈等待主线程执行。
- JS调用栈采用的是后进先出的规则，当函数执行的时候，会被添加到栈的顶部，当执行栈执行完成后，就会从栈顶移出，直到栈内被清空。
- `Javascript`单线程任务被分为**同步任务**和**异步任务**，同步任务会在调用栈中按照顺序等待主线程依次执行，异步任务会在异步任务有了结果后，将注册的回调函数放入任务队列中等待主线程空闲的时候（调用栈被清空），被读取到栈内等待主线程的执行。
- 执行栈在执行完**同步任务**后，查看**执行栈**是否为空，如果执行栈为空，就会去检查**微任务**(`microTask`)队列是否为空，如果为空的话，就执行`Task`（宏任务），否则就一次性执行完所有微任务。 每次单个**宏任务**执行完毕后，检查**微任务**(`microTask`)队列是否为空，如果不为空的话，会按照**先入先**出的规则全部执行完**微任务**(`microTask`)后，设置**微任务**(`microTask`)队列为`null`，然后再执行**宏任务**，如此循环。
- **Event Loop**流程
  - 先执行script脚本，将宏任务和微任务进行分类，如果调用的是浏览器的api，浏览器会开一个线程，等时间到了，会自动的放到宏任务队列中，微任务是直接放到微任务队列中
  - js执行完毕后，会清空所有的微任务，如果微任务再产生微任务，会放到当前微任务队列的尾部。
  - 如果页面需要渲染，则会执行渲染流程
  - Event Loop会不断扫描宏任务队列，如果宏任务队列中有回调，获取出来执行一个，继续执行上诉过程
  - 宏任务每次调用一个，微任务是清空所有
  - 浏览器渲染时机：在执行script和当前script中产生的微任务之后，浏览器渲染。而当前script产生的宏任务会在渲染之后执行。

![eventloop](./img/eventloop.png)

例题：

```javascript
console.log(1);
async function Async() {
    console.log(2);
    await console.log(3);
    console.log(4);
}
setTimeout(() => {
    console.log(5);
}, 0)
const promise = new Promise((resolve, rejrct) => {
    console.log(6);
    resolve(7);
})
promise.then((res) => {
    console.log(res);
})
Async();
console.log(8);
```

答案：

```
1
6
2
3
8
7
4
5
```

在异步函数中，可将

```
async function f() {
  await p
  console.log('ok')
}
```

简化理解为：

```
function f() {
  return RESOLVE(p).then(() => {
    console.log('ok')
  })
}
```

在执行脚本时：先从上往下一次执行同步任务，也就是

```
console.log(1);
console.log(6);
console.log(2);
console.log(3);
console.log(8);
```

此时微任务队列依次中包括：

```
promise.then((res) => {
    console.log(res);
})
console.log(4);
```

宏任务包括：

```
setTimeout(() => {
    console.log(5);
}, 0)
```

当script执行完毕时，查询微任务，微任务将按入队先后顺序依次执行。微任务清空以后，查询宏任务队列，取出第一个宏任务，执行其回调。

### Node.js Event Loop

NodeJS中的EventLoop使NodeJS能够进行非阻塞的I/O操作，尽管JavaScript是单线程的，可以通过将I/O操作交给系统内核去执行。

当某个I/O操作结束的时候，内核会将对应的callback函数元信息交给NodeJS中的poll queue(轮询队列)。

#### 概述

当NodeJS开始工作的时候就会初始化EventLoop机制，处理我们的JavaScript脚本。当我们的脚本中调用了异步函数，计时器或者process.nextTick()的时候，EventLoop机制就会开始介入。

下图简明地展示了EventLoop的操作顺序：

```
   ┌───────────────────────────┐
┌─>│           timers          │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │     pending callbacks     │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │       idle, prepare       │
│  └─────────────┬─────────────┘      ┌───────────────┐
│  ┌─────────────┴─────────────┐      │   incoming:   │
│  │           poll            │<─────┤  connections, │
│  └─────────────┬─────────────┘      │   data, etc.  │
│  ┌─────────────┴─────────────┐      └───────────────┘
│  │           check           │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
└──┤      close callbacks      │
   └───────────────────────────┘
```

> 上面的每个方格展示了EventLoop的不同阶段。对应上文中的task queue，代表不同的taskQueue在不同阶段的处理机制。

每个阶段都维持着一个先进先出的回调函数队列。每个阶段都有特殊的职能，当EventLoop执行到该阶段的时候，会将该时刻在该阶段的队列中存在的所有回调函数执行完或者执行到回调函数数量的上限(NodeJS自己规定的参数)，然后跳转到下一个阶段。

由于脚本中的某些操作会调度更多的操作，在poll(轮询)阶段新事件是由内核排队。因此在处理poll阶段事件的时候可以将其他的事件加入到poll阶段中。因此，长时间运行回调函数会让poll阶段的运行时间比计时器规定的阈值长地多，我们设定的计时阈值不一定准确在该时刻执行。

## 阶段描述

**timers（计时阶段）**：这个阶段执行由setTimeout() 和 setInterval() 调度的回调。

**pending callbacks（回调待处理阶段）**：这个阶段执行延迟到下一次循环迭代的I/O回调。

**idle, prepare（闲置阶段）**：仅仅被NodeJS内部使用。（这里是根据官方文档描述的，不能甚解）

**poll（轮询阶段）**：这个阶段检索I/O事件，执行I/O事件对应的回调函数（除了关闭回调、计时器调度的回调和 setImmediate() 之外的几乎所有异步操作），都将会在这个阶段被处理。

**check（检测阶段）**：setImmediate()函数的回调函数将会在这里被处理。

**close callbacks（关闭回调阶段）**：例如：`socket.on('close', ...)`

> 每个阶段都有对应的异步操作，当异步操作需要触发回调函数的时候是先将回调函数放在对应阶段的queue中，等待EventLoop机制进入到该阶段再去执行queue中排好队的回调函数。放入队列中的是回调函数的元信息。

### timers（计时阶段）

计时器指定可以执行提供的回调的阈值，但是这个阈值不一定是人们希望执行的确切时间。当执行到setTimeout() 和 setInterval()，经过指定的时间阈值之后，Timers queue中对应的回调函数将会尽快被执行。可是，操作系统的调度机制和其他回调函数都有可能延迟执行Timers queue中的回调函数。

> 事实上，poll阶段决定着什么时候执行timers queue中的回调。

如下代码所示：

```
const fs = require('fs');

function someAsyncOperation(callback) {
  // Assume this takes 95ms to complete
  fs.readFile('/path/to/file', callback);
}

const timeoutScheduled = Date.now();

setTimeout(() => {
  const delay = Date.now() - timeoutScheduled;

  console.log(`${delay}ms have passed since I was scheduled`);
}, 100);

// do someAsyncOperation which takes 95 ms to complete
someAsyncOperation(() => {
  const startCallback = Date.now();

  // do something that will take 10ms...
  while (Date.now() - startCallback < 10) {
    // do nothing
  }
});
```

`fs.readFile('/path/to/file', callback);`会使EventLoop进入poll阶段，由于`fs.readFile()`没有执行结束，此时poll queue中的回调函数为空，因此在没有意外的情况下等待一段时间等到timers阶段规定的时间阈值100ms去执行timers阶段对应的回调函数。但是在95ms的时候。`fs.readFile()`执行完毕，文件读取结束，接下来会将`fs.readFile()`对应的回调函数添加进poll queue并执行10ms。当poll queue中`fs.readFile()`对应的回到函数执行结束以后，EventLoop会查看在poll queue中是否有其他的回调函数，目前查看暂无，于是EventLoop将进入timers queue中按照队列的弹出顺序执行对应的回调函数。在本例中，setTimeout()对应的回调函数将会在105ms后执行。

### pending callbacks（回调待处理阶段）

这个阶段执行操作系统的一些操作，例如TCP errors。例如，如果 TCP 套接字在尝试连接时收到 ECONNREFUSED，则某些 *nix 系统会将该错误挂起在pending callback queue中排队执行。

### poll（轮询阶段）

轮询阶段有两个主要的职能：

- 计算I/O操作应该阻塞和轮询的时间
- 在poll queue中处理事件。

当EventLoop进入poll 阶段并且当前timers queue为空，将会：

- 如果poll queue不为空，EventLoop将会一直执行poll queue中的回调函数，直到poll queue为空或者到了NodeJS的硬限制。
- 如果poll queue为空，将会：
  - 如果调用了setImmediate()函数，EventLoop会结束poll阶段的执行，进入check阶段执行setImmediate()对应的回调函数。
  - 如果没有调用setImmediate()函数，EventLoop会停留在poll阶段，当poll queue中添加了回调函数的时候会被立即执行。

一旦poll queue为空，EventLoop会立即检查timers queue中是否有回调函数，如果有将会立刻从poll阶段返回timers阶段去执行timers queue中的回到函数。

### check（检测阶段）

这个阶段允许用户立即执行回调函数（在poll queue为空的情况）。setImmediate() 实际上是一个特殊的计时器，它在事件循环的一个单独阶段运行。 它使用 libuv API 来安排在轮询阶段完成后执行的回调。

通常，随着代码的执行，EventLoop最终会进入poll阶段，在那里它将等待传入的连接、请求等。但是，如果使用 setImmediate() 安排了回调并且poll阶段变为空闲，则EventLoop将结束并进入check阶段，而不是等待轮询事件。

### close callbacks（关闭回调阶段）

如果套接字或句柄突然关闭（例如 socket.destroy()），则将在此阶段发出 'close' 事件。 否则它将通过 process.nextTick() 发出。

