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
- **渲染相关**
  - `requestAnimationFrame`(页面渲染之前执行)
  - `requestIDleCallback`(空闲时间执行)
