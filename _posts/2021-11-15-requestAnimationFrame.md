---
layout: post
title: requestAnimationFrame 执行机制探索
categories: javascript
tags: [javascript]
excerpt: requestAnimationFrame 执行机制探索
---

`转载于凹凸实验室`

### 什么是 requestAnimationFrame

`window.requestAnimationFrame()` 告诉浏览器——你希望执行一个动画，并且要求浏览器在下次重绘之前调用指定的回调函数更新动画。该方法需要传入一个回调函数作为参数，该回调函数会在浏览器下一次重绘之前执行。根据以上 MDN 的定义，`requestAnimationFrame` 是浏览器提供的一个按帧对网页进行重绘的 API 。先看下面这个例子，了解一下它是如何使用并运行的：

![requestAnimationFrame]({{ site.url }}{{site.baseurl}}/assets/images/post/20211115/01.png)

上面的代码 1s 大约执行 60 次，因为一般的屏幕硬件设备的刷新频率都是 `60Hz`，然后每执行一次大约是 `16.6ms`。使用 `requestAnimationFrame` 的时候，只需要反复调用它就可以实现动画效果。

同时 `requestAnimationFrame` 会返回一个请求 `ID`，是回调函数列表中的一个唯一值，可以使用 `cancelAnimationFrame` 通过传入该请求 `ID` 取消回调函数。

![requestAnimationFrame]({{ site.url }}{{site.baseurl}}/assets/images/post/20211115/02.png)

![requestAnimationFrame]({{ site.url }}{{site.baseurl}}/assets/images/post/20211115/03.gif)

### requestAnimationFrame 执行困惑

使用 JavaScript 实现动画的方式还可以使用 `setTimeout` ，下面是实现的代码：  
![requestAnimationFrame]({{ site.url }}{{site.baseurl}}/assets/images/post/20211115/04.png)

在这里将 `setTimeout` 的执行间隔设置为 0，来模仿 `requestAnimationFrame`。

下图是 setTimeout 执行结果：

![requestAnimationFrame]({{ site.url }}{{site.baseurl}}/assets/images/post/20211115/05.gif)

很明显能看出，`setTimeout` 比 `requestAnimationFrame` 实现的动画“快”了很多。这是什么原因呢？

可能你也猜到了，`Event Loop` 和 `requestAnimationFrame` 在执行的时候有些特殊的机制，下面就来探究一下 `Event Loop` 和 `requestAnimationFrame` 的关系。

### Event Loop 与 requestAnimationFrame

`Event Loop`（事件循环）是用来协调事件、用户交互、脚本、渲染、网络的一种浏览器内部机制。

`Event Loop` 在浏览器内也分几种：

- `window event loop`
- `worker event loop`
- `worklet event loop`

我们这里主要讨论的是 `window event loop`。也就是浏览器一个渲染进程内主线程所控制的 `Event Loop`。

#### task queue

一个 `Event Loop` 有一个或多个 `task queues`。一个 `task queue` 是一系列 `tasks` 的集合。

> 注：一个 task queue 在数据结构上是一个集合，而不是队列，因为事件循环处理模型会从选定的 task queue 中获取第一个可运行任务（runnable task），而不是使第一个 task 出队。上述内容来自 HTML 规范。这里让人迷惑的是，明明是集合，为啥还叫 "queue" 啊

#### task

一个 task 可以有多种 `task sources` (任务源)，有哪些任务源呢？来看下规范里的 `Gerneric task sources` ：

- DOM 操作任务源，比如一个元素以非阻塞的方式插入文档
- 用户交互任务源，用户操作（比如 click）事件
- 网络任务源，网络 I/O 响应回调
- history traversal 任务源，比如 history.back()

除此之外还有像 `Timers (setTimeout、setInterval等)`、`IndexDB` 操作也是 `task source`。

#### microtask

一个 `event loop` 有一个 `microtask queue`，不过这个 "queue" 它确实就是那个 "`FIFO`" 的队列。

规范里没有指明哪些是 microtask 的任务源，通常认为以下几个是 microtask：

- promises
- MutationObserver
- Object.observe
- process.nextTick (这个东西是 Node.js 的 API，暂且不讨论)

#### Event Loop 处理过程

1. 在所选 task queue (taskQueue)中约定必须包含一个可运行任务。如果没有此类 task queue，则跳转至下面 microtasks 步骤。
2. 让 taskQueue 中最老的 task (oldestTask) 变成第一个可执行任务，然后从 taskQueue 中删掉它。
3. 将上面 oldestTask 设置为 event loop 中正在运行的 task。
4. 执行 oldestTask。
5. 将 event loop 中正在运行的 task 设置为 null。
6. 执行 microtasks 检查点（也就是执行 microtasks 队列中的任务）。
7. 设置 hasARenderingOpportunity 为 false。
8. 更新渲染。
9. 如果当前是 window event loop 且 task queues 里没有 task 且 microtask queue 是空的，同时渲染时机变量 hasARenderingOpportunity 为 false ，去执行 idle period（requestIdleCallback）。
10. 返回到第一步。

以上是来自规范关于 event loop 处理过程的精简版整理。

大体上来说，`event loop` 就是不停地找 task queues 里是否有可执行的 task ，如果存在即将其推入到 `call stack` （执行栈）里执行，并且在合适的时机更新渲染。

下图是 `event loop` 在浏览器主线程上运行的一个清晰的流程：

![requestAnimationFrame]({{ site.url }}{{site.baseurl}}/assets/images/post/20211115/06.jpg)

在上面规范的说明中，渲染的流程是在执行 microtasks 队列之后，更进一步，再来看看渲染的处理过程。

#### 更新渲染

1. 遍历当前浏览上下文中所有的 document ，必须按在列表中找到的顺序处理每个 document 。
2. 渲染时机（Rendering opportunities）：如果当前浏览上下文中没有到渲染时机则将所有 docs 删除，取消渲染（此处是否存在渲染时机由浏览器自行判断，根据硬件刷新率限制、页面性能或页面是否在后台等因素）。
3. 如果当前文档不为空，设置 hasARenderingOpportunity 为 true 。
4. 不必要的渲染（Unnecessary rendering）：如果浏览器认为更新文档的浏览上下文的呈现不会产生可见效果且文档的 animation frame callbacks 是空的，则取消渲染。（终于看见 requestAnimationFrame 的身影了
5. 从 docs 中删除浏览器认为出于其他原因最好跳过更新渲染的文档。
6. 如果文档的浏览上下文是顶级浏览上下文，则刷新该文档的自动对焦候选对象。
7. 处理 resize 事件，传入一个 performance.now() 时间戳。
8. 处理 scroll 事件，传入一个 performance.now() 时间戳。
9. 处理媒体查询，传入一个 performance.now() 时间戳。
10. 运行 CSS 动画，传入一个 performance.now() 时间戳。
11. 处理全屏事件，传入一个 performance.now() 时间戳。
12. 执行 requestAnimationFrame 回调，传入一个 performance.now() 时间戳。
13. 执行 intersectionObserver 回调，传入一个 performance.now() 时间戳。
14. 对每个 document 进行绘制。
15. 更新 ui 并呈现。

下图是该过程一个比较清晰的流程：  
![requestAnimationFrame]({{ site.url }}{{site.baseurl}}/assets/images/post/20211115/07.jpg)

至此，`requestAnimationFrame` 的回调时机就清楚了，它会在 `style/layout/paint` 之前调用。

再回到文章开始提到的 `setTimeout` 动画比 `requestAnimationFrame` 动画更快的问题，这就很好解释了。

首先，**浏览器渲染有个渲染时机（Rendering opportunity）的问题，也就是浏览器会根据当前的浏览上下文判断是否进行渲染，它会尽量高效，只有必要的时候才进行渲染，如果没有界面的改变，就不会渲染。**按照规范里说的一样，因为考虑到硬件的刷新频率限制、页面性能以及页面是否存在后台等等因素，有可能执行完 `setTimeout` 这个 task 之后，发现还没到渲染时机，所以 `setTimeout` 回调了几次之后才进行渲染，此时设置的 marginLeft 和上一次渲染前 marginLeft 的差值要大于 1px 的。

下图是 `setTimeout` 执行情况，红色圆圈处是两次渲染，中间四次是处理 `setTimout task`，因为屏幕的刷新频率是 60 Hz，所以大致在 16.6ms 之内执行了多次 `setTimeout task` 之后才到了渲染时机并执行渲染。

![requestAnimationFrame]({{ site.url }}{{site.baseurl}}/assets/images/post/20211115/08.jpg)

`requestAnimationFrame` 帧动画不同之处在于，每次渲染之前都会调用，此时设置的 marginLeft 和上一次渲染前 marginLeft 的差值为 1px 。

下图是 `requestAnimationFrame` 执行情况，每次调用完都会执行渲染：

![requestAnimationFrame]({{ site.url }}{{site.baseurl}}/assets/images/post/20211115/09.jpg)

所以看上去 `setTimeout` “快”了很多。
