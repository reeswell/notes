# Worker

单线程就意味着不能像多线程语言那样把工作委托给独立的线程或进程去做。JavaScript 的单线程可以保证它与不同浏览器API兼容。假如JavaScript可以多线程执行并发更改，那么像DOM这样的API就会出现问题。因此，POSIX线程或Java的Thread类等传统并发结构都不适合JavaScript。

使用工作者线程，浏览器可以在原始页面环境之外再分配一个完全独立的二级子环境。这个子环境不能与依赖单线程交互的API(如 DOM)互操作，但可以与父环境并行执行代码。

## WorkerGlobalScope

在网页上，window 对象可以向运行在其中的脚本暴露各种全局变量。在工作者线程内部，没有 window的概念。这里的全局对象是 WorkerGlobalScope 的实例，通过 self 关键字暴露出来。

### WorkerGlobalScope 属性和方法

self 上可用的属性是 window 对象上属性的严格子集。其中有些属性会返回特定于工作者线程的
版本。

- navigator:返回与工作者线程关联的 WorkerNavigator。
- self:返回 WorkerGlobalScope 对象。
- location:返回与工作者线程关联的 WorkerLocation。
- performance:返回(只包含特定属性和方法的)Performance 对象。
- console:返回与工作者线程关联的 Console 对象;对 API 没有限制。
- caches:返回与工作者线程关联的 CacheStorage 对象;对 API 没有限制。
- indexedDB:返回 IDBFactory 对象。
- isSecureContext:返回布尔值，表示工作者线程上下文是否安全。
- origin:返回 WorkerGlobalScope 的源。
类似地，self 对象上暴露的一些方法也是 window 上方法的子集。这些 self 上的方法也与 window
上对应的方法操作一样。
- atob()
- btoa()
- clearInterval()
- clearTimeout()
- createImageBitmap()
- fetch()
- setInterval()
- setTimeout()
WorkerGlobalScope 还增加了新的全局方法 importScripts()，只在工作者线程内可用。本章 稍后会介绍该方法。

### WorkerGlobalScope 的子类

实际上并不是所有地方都实现了 WorkerGlobalScope。每种类型的工作者线程都使用了自己特定 的全局对象，这继承自 WorkerGlobalScope。

- 专用工作者线程使用 DedicatedWorkerGlobalScope。
- 共享工作者线程使用 SharedWorkerGlobalScope。
- 服务工作者线程使用 ServiceWorkerGlobalScope。 本章稍后会在这些全局对象对应的小节中讨论其差异。

## 工作者线程与线程

作为介绍，通常需要将工作者线程与执行线程进行比较。在许多方面，这是一个恰当的比较，因为 工作者线程和线程确实有很多共同之处。

- 工作者线程是以实际线程实现的。例如，Blink 浏览器引擎实现工作者线程的 WorkerThread 就 对应着底层的线程。
- 工作者线程并行执行。虽然页面和工作者线程都是单线程 JavaScript 环境，每个环境中的指令则 可以并行执行。
- 工作者线程可以共享某些内存。工作者线程能够使用 SharedArrayBuffer 在多个环境间共享 内容。虽然线程会使用锁实现并发控制，但 JavaScript 使用 Atomics 接口实现并发控制。
工作者线程与线程有很多类似之处，但也有重要的区别。
- 工作者线程不共享全部内存。在传统线程模型中，多线程有能力读写共享内存空间。除了 Shared-
ArrayBuffer 外，从工作者线程进出的数据需要复制或转移。
- 工作者线程不一定在同一个进程里。通常，一个进程可以在内部产生多个线程。根据浏览器引
擎的实现，工作者线程可能与页面属于同一进程，也可能不属于。例如，Chrome 的 Blink 引擎对
   共享工作者线程和服务工作者线程使用独立的进程。
- 创建工作者线程的开销更大。工作者线程有自己独立的事件循环、全局对象、事件处理程序和
其他 JavaScript 环境必需的特性。创建这些结构的代价不容忽视 无论形式还是功能，工作者线程都不是用于替代线程的。HTML Web 工作者线程规范是这样说的:
工作者线程相对比较重，不建议大量使用。例如，对一张 400 万像素的图片，为每个像素 都启动一个工作者线程是不合适的。通常，工作者线程应该是长期运行的，启动成本比较高， 每个实例占用的内存也比较大。

## 工作者线程的类型

Web 工作者线程规范中定义了三种主要的工作者线程:专用工作者线程、共享工作者线程和服务工
作者线程。现代浏览器都支持这些工作者线程。

**1. 专用工作者线程**
专用工作者线程，通常简称为工作者线程、Web Worker 或 Worker，是一种实用的工具，可以让脚 本单独创建一个 JavaScript 线程，以执行委托的任务。专用工作者线程，顾名思义，只能被创建它的页 面使用。
**2. 共享工作者线程**
共享工作者线程与专用工作者线程非常相似。主要区别是共享工作者线程可以被多个不同的上下文 使用，包括不同的页面。任何与创建共享工作者线程的脚本同源的脚本，都可以向共享工作者线程发送 消息或从中接收消息。
**3. 服务工作者线程**
服务工作者线程与专用工作者线程和共享工作者线程截然不同。它的主要用途是拦截、重定向和修 改页面发出的请求，充当网络请求的仲裁者的角色。

### 专用工作者线程

专用工作者线程是最简单的 Web 工作者线程，网页中的脚本可以创建专用工作者线程来执行在页面 线程之外的其他任务。这样的线程可以与父页面交换信息、发送网络请求、执行文件输入/输出、进行密 集计算、处理大量数据，以及实现其他不适合在页面执行线程里做的任务(否则会导致页面响应迟钝)。

可以把专用工作者线程称为后台脚本(background script)。JavaScript 线程的各个方面，包括生命周 期管理、代码路径和输入/输出，都由初始化线程时提供的脚本来控制。该脚本也可以再请求其他脚本， 但一个线程总是从一个脚本源开始。

#### 创建专用工作者线程

创建专用工作者线程最常见的方式是加载 JavaScript 文件。把文件路径提供给 Worker 构造函数， 然后构造函数再在后台异步加载脚本并实例化工作者线程。传给构造函数的文件路径可以是多种形式。
下面的代码演示了如何创建空的专用工作者线程:

```js
// emptyWorker.js
// 空的 JS 工作者线程文件 
// main.js
console.log(location.href); // "<https://example.com/>"
const worker = new Worker(location.href + 'emptyWorker.js'); 
console.log(worker); // Worker {}
```

  这个例子非常简单，但涉及几个基本概念。

- emptyWorker.js 文件是从绝对路径加载的。根据应用程序的结构，使用绝对 URL 经常是多余的。
- 这个文件是在后台加载的，工作者线程的初始化完全独立于 main.js。
- 工作者线程本身存在于一个独立的 JavaScript 环境中，因此 main.js 必须以 Worker 对象为代理实
现与工作者线程通信。在上面的例子中，该对象被赋值给了 worker 变量。
- 虽然相应的工作者线程可能还不存在，但该 Worker 对象已在原始环境中可用了。
前面的例子可修改为使用相对路径。不过，这要求 main.js 必须与 emptyWorker.js 在同一个路径下:

```js
const worker = new Worker('./emptyWorker.js'); 
console.log(worker); // Worker {}
```

#### 工作者线程安全限制

工作者线程的脚本文件只能从与父页面相同的源加载。从其他源加载工作者线程的脚本文件会导致 错误，如下所示:

```js
// 尝试基于 <https://example.com/worker.js> 创建工作者线程 
const sameOriginWorker = new Worker('./worker.js');
// 尝试基于 <https://untrusted.com/worker.js> 创建工作者线程
const remoteOriginWorker = new Worker('<https://untrusted.com/worker.js>');
// Error: Uncaught DOMException: Failed to construct 'Worker':
// Script at <https://untrusted.com/main.js> cannot be accessed
// from origin <https://example.com>
```

**注意** 不能使用非同源脚本创建工作者线程，并不影响执行其他源的脚本。在工作者线程 内部，使用 importScripts()可以加载其他源的脚本。

#### 使用 Worker 对象

Worker()构造函数返回的 Worker 对象是与刚创建的专用工作者线程通信的连接点。它可用于在
工作者线程和父上下文间传输信息，以及捕获专用工作者线程发出的事件。
注意 要管理好使用Worker()创建的每个Worker对象。在终止工作者线程之前，它不会被垃圾回收，也不能通过编程方式恢复对之前 Worker 对象的引用。

Worker 对象支持下列事件处理程序属性。

- `onerror`:在工作者线程中发生 ErrorEvent 类型的错误事件时会调用指定给该属性的处理程序。
- 该事件会在工作者线程中抛出错误时发生。
- 该事件也可以通过 `worker.addEventListener('error', handler)`的形式处理。
- `onmessage`:在工作者线程中发生 MessageEvent 类型的消息事件时会调用指定给该属性的处理程序。  
  - 该事件会在工作者线程向父上下文发送消息时发生。
  - 该事件也可以通过使用 `worker.addEventListener('message', handler)`处理。
- `onmessageerror`:在工作者线程中发生 MessageEvent 类型的错误事件时会调用指定给该属性的处理程序。
  - 该事件会在工作者线程收到无法反序列化的消息时发生。
  - 该事件也可以通过使用 `worker.addEventListener('messageerror', handler)`处理。
Worker 对象还支持下列方法。
- `postMessage()`:用于通过异步消息事件向工作者线程发送信息。
- `terminate()`:用于立即终止工作者线程。没有为工作者线程提供清理的机会，脚本会突然停止。

#### DedicatedWorkerGlobalScope

在专用工作者线程内部，全局作用域是 DedicatedWorkerGlobalScope 的实例。因为这继承自
WorkerGlobalScope，所以包含它的所有属性和方法。工作者线程可以通过 self 关键字访问该全局 作用域。

```js
// globalScopeWorker.js
console.log('inside worker:', self);
// main.js
const worker = new Worker('./globalScopeWorker.js');
console.log('created worker:', worker);
// created worker: Worker {}
// inside worker: DedicatedWorkerGlobalScope {}
```

DedicatedWorkerGlobalScope 在 WorkerGlobalScope 基础上增加了以下属性和方法。

- name:可以提供给 Worker 构造函数的一个可选的字符串标识符。
- postMessage():与 worker.postMessage()对应的方法，用于从工作者线程内部向父上下
文发送消息。
- close():与 worker.terminate()对应的方法，用于立即终止工作者线程。没有为工作者线
程提供清理的机会，脚本会突然停止。
- importScripts():用于向工作者线程中导入任意数量的脚本。

### 专用工作者线程的生命周期

调用 Worker()构造函数是一个专用工作者线程生命的起点。调用之后，它会初始化对工作者线程 脚本的请求，并把 Worker 对象返回给父上下文。虽然父上下文中可以立即使用这个 Worker 对象，但 与之关联的工作者线程可能还没有创建，因为存在请求脚本的网格延迟和初始化延迟。
一般来说，专用工作者线程可以非正式区分为处于下列三个状态:初始化(initializing)、活动(active) 和终止(terminated)。这几个状态对其他上下文是不可见的。虽然 Worker 对象可能会存在于父上下文 中，但也无法通过它确定工作者线程当前是处理初始化、活动还是终止状态。换句话说，与活动的专用 工作者线程关联的 Worker 对象和与终止的专用工作者线程关联的 Worker 对象无法分别。
初始化时，虽然工作者线程脚本尚未执行，但可以先把要发送给工作者线程的消息加入队列。这些 消息会等待工作者线程的状态变为活动，再把消息添加到它的消息队列。下面的代码演示了这个过程

```js
// initializingWorker.js.js
self.addEventListener('message', ({data}) => console.log(data));
// main.js
const worker = new Worker('./initializingWorker.js');
// Worker 可能仍处于初始化状态
// 但postMessage()数据可以正常处理 
worker.postMessage('foo'); 
worker.postMessage('bar'); 
worker.postMessage('baz');
// foo
// bar
// baz
```

创建之后，专用工作者线程就会伴随页面的整个生命期而存在，除非自我终止(`self.close()`) 或通过外部终止(`worker.terminate()`)。即使线程脚本已运行完成，线程的环境仍会存在。只要工 作者线程仍存在，与之关联的 Worker 对象就不会被当成垃圾收集掉。
自我终止和外部终止最终都会执行相同的工作者线程终止例程。来看下面的例子，其中工作者线程 在发送两条消息中间执行了自我终止:

```js
closeWorker.js
self.postMessage('foo');
self.close();
self.postMessage('bar');
setTimeout(() => self.postMessage('baz'), 0);
main.js
const worker = new Worker('./closeWorker.js'); 
worker.onmessage = ({data}) => console.log(data);
// foo 
// bar
```

虽然调用了 `close()`，但显然工作者线程的执行并没有立即终止。`close()`在这里会通知工作者线 程取消事件循环中的所有任务，并阻止继续添加新任务。这也是为什么"baz"没有打印出来的原因。工 作者线程不需要执行同步停止，因此在父上下文的事件循环中处理的"bar"仍会打印出来。
下面来看外部终止的例子。

```js
// terminateWorker.js
self.onmessage = ({data}) => console.log(data);
// main.js
const worker = new Worker('./terminateWorker.js');
// 给 1000 毫秒让工作者线程初始化 
setTimeout(() => {
   worker.postMessage('foo');
   worker.terminate();
   worker.postMessage('bar');
   setTimeout(() => worker.postMessage('baz'), 0);
}, 1000);
// foo
```

这里，外部先给工作者线程发送了带"foo"的 postMessage，这条消息可以在外部终止之前处理。
一旦调用了 `terminate()`，工作者线程的消息队列就会被清理并锁住，这也是只是打印"foo"的原因。

**注意** `close()`和 `terminate()`是幂等操作，多次调用没有问题。这两个方法仅仅是将Worker标记为teardown，因此多次调用不会有不好的影响。

在整个生命周期中，一个专用工作者线程只会关联一个网页(Web 工作者线程规范称其为一个文档)。 除非明确终止，否则只要关联文档存在，专用工作者线程就会存在。如果浏览器离开网页(通过导航或 关闭标签页或关闭窗口)，它会将与其关联的工作者线程标记为终止，它们的执行也会立即停止。

### 配置Worker选项

Worker()构造函数允许将可选的配置对象作为第二个参数。该配置对象支持下列属性。

- name:可以在工作者线程中通过 `self.name` 读取到的字符串标识符。
- type:表示加载脚本的运行方式，可以是"classic"或"module"。"classic"将脚本作为常
规脚本来执行，"module"将脚本作为模块来执行。
- credentials:在 type 为"module"时，指定如何获取与传输凭证数据相关的工作者线程模块
脚本。值可以是"omit"、"same-orign"或"include"。这些选项与 fetch()的凭证选项相同。 在 type 为"classic"时，默认为"omit"。

### 在JavaScript行内创建工作者线程

工作者线程需要基于脚本文件来创建，但这并不意味着该脚本必须是远程资源。专用工作者线程也 可以通过 Blob 对象 URL 在行内脚本创建。这样可以更快速地初始化工作者线程，因为没有网络延迟。
下面展示了一个在行内创建工作者线程的例子。

```js
// 创建要执行的 JavaScript 代码字符串 
const workerScript = `
self.onmessage = ({ data }) => console.log(data);
`;
// 基于脚本字符串生成 Blob 对象
const workerScriptBlob = new Blob([workerScript]);
// 基于Blob实例创建对象URL
const workerScriptBlobUrl = URL.createObjectURL(workerScriptBlob);
// 基于对象 URL 创建专用工作者线程
const worker = new Worker(workerScriptBlobUrl);
worker.postMessage('blob worker script');
// blob worker script
```

在这个例子中，通过脚本字符串创建了 Blob，然后又通过 Blob 创建了对象 URL，最后把对象 URL 传给了 Worker()构造函数。该构造函数同样创建了专用工作者线程。

如果把所有代码写在一块，可以浓缩为这样:

```js
const worker = new Worker(URL.createObjectURL(new Blob([`self.onmessage = ({data}) => console.log(data);`])));
worker.postMessage('blob worker script');
// blob worker script
```

工作者线程也可以利用函数序列化来初始化行内脚本。这是因为函数的 toString()方法返回函数 代码的字符串，而函数可以在父上下文中定义但在子上下文中执行。来看下面这个简单的例子:

```js
function fibonacci(n) {
  return n < 1 ? 0
      : n <= 2 ? 1
      : fibonacci(n - 1) + fibonacci(n - 2);
}
const workerScript = `
  self.postMessage(
(${fibonacci.toString()})(9) );
`;
const worker = new Worker(URL.createObjectURL(new Blob([workerScript]))); 
worker.onmessage = ({data}) => console.log(data);
// 34 21
```

这里有意使用了斐波那契数列的实现，将其序列化之后传给了工作者线程。该函数作为 IIFE 调用 并传递参数，结果则被发送回主线程。虽然计算斐波那契数列比较耗时，但所有计算都会委托到工作者 线程，因此并不会影响父上下文的性能。

注意 像这样序列化函数有个前提，就是函数体内不能使用通过闭包获得的引用，也包括全局变量，比如window，因为这些引用在工作者线程中执行时会出错。

### 在工作者线程中动态执行脚本

工作者线程中的脚本并非铁板一块，而是可以使用 importScripts()方法通过编程方式加载和执 行任意脚本。该方法可用于全局 Worker 对象。这个方法会加载脚本并按照加载顺序同步执行。比如， 下面的例子加载并执行了两个脚本:

```js
// main.js
const worker = new Worker('./worker.js');
// importing scripts
// scriptA executes
// scriptB executes
// scripts imported
// scriptA.js
console.log('scriptA executes');
// scriptB.js
console.log('scriptB executes');
// worker.js
console.log('importing scripts');
importScripts('./scriptA.js');
importScripts('./scriptB.js');
console.log('scripts imported');
```

importScripts()方法可以接收任意数量的脚本作为参数。浏览器下载它们的顺序没有限制，但 执行则会严格按照它们在参数列表的顺序进行。因此，下面的代码与前面的效果一样:

```js
console.log('importing scripts');
importScripts('./scriptA.js', './scriptB.js');
console.log('scripts imported');
```

脚本加载受到常规 CORS 的限制，但在工作者线程内部可以请求来自任何源的脚本。这里的脚本导 入策略类似于使用生成的`<script>`标签动态加载脚本。在这种情况下，所有导入的脚本也会共享作用 域。下面的代码演示了这个事实:

```js
main.js
const worker = new Worker('./worker.js', {name: 'foo'});
// importing scripts in foo with bar 
// scriptA executes in foo with bar 
// scriptB executes in foo with bar 
// scripts imported
scriptA.js
console.log(`scriptA executes in ${self.name} with ${globalToken}`);
scriptB.js
console.log(`scriptB executes in ${self.name} with ${globalToken}`);
worker.js
const globalToken = 'bar';
console.log(`importing scripts in ${self.name} with ${globalToken}`); 
importScripts('./scriptA.js', './scriptB.js');
console.log('scripts imported');
```

### 委托任务到子工作者线程

有时候可能需要在工作者线程中再创建子工作者线程。在有多个 CPU 核心的时候，使用多个子工 作者线程可以实现并行计算。使用多个子工作者线程前要考虑周全，确保并行计算的投入确实能够得到 收益，毕竟同时运行多个子线程会有很大计算成本。
除了路径解析不同，创建子工作者线程与创建普通工作者线程是一样的。子工作者线程的脚本路径 根据父工作者线程而不是相对于网页来解析。来看下面的例子(注意额外的 js 目录):

```js
// main.js
const worker = new Worker('./js/worker.js'); // worker
// subworker
// js/worker.js
console.log('worker');
const worker = new Worker('./subworker.js');
// js/subworker.js
console.log('subworker');
```

**注意** 顶级工作者线程的脚本和子工作者线程的脚本都必须从与主页相同的源加载。

### 处理工作者线程错误

如果工作者线程脚本抛出了错误，该工作者线程沙盒可以阻止它打断父线程的执行。如下例所示， 其中的 try/catch 块不会捕获到错误:
main.js

```js
try {
  const worker = new Worker('./worker.js');
  console.log('no error');
  } catch(e) {
  console.log('caught error');
  }
// no error
worker.js
throw Error('foo');
```

不过，相应的错误事件仍然会冒泡到工作者线程的全局上下文，因此可以通过在 Worker 对象上设 置错误事件侦听器访问到。下面看这个例子:

```js
// main.js
const worker = new Worker('./worker.js');
worker.onerror = console.log;
// ErrorEvent {message: "Uncaught Error: foo"}
// worker.js
throw Error('foo');
```

### 与专用工作者线程通信

与工作者线程的通信都是通过异步消息完成的，但这些消息可以有多种形式。

#### 使用postMessage()

最简单也最常用的形式是使用 postMessage()传递序列化的消息。下面来看一个计算阶乘的例子:

```js
// factorialWorker.js
function factorial(n) {
  let result = 1;
  while(n) { result *= n--; }
  return result;
}
self.onmessage = ({data}) => { self.postMessage(`${data}! = ${factorial(data)}`);
};
// main.js
const factorialWorker = new Worker('./factorialWorker.js');
factorialWorker.onmessage = ({data}) => console.log(data);
factorialWorker.postMessage(5);
factorialWorker.postMessage(7); 
factorialWorker.postMessage(10);
// 5! = 120
// 7! = 5040
// 10! = 3628800
```

对于传递简单的消息，使用 postMessage()在主线程和工作者线程之间传递消息，与在两个窗口 间传递消息非常像。主要区别是没有 targetOrigin 的限制，该限制是针对 Window.prototype. postMessage 的，对 WorkerGlobalScope.prototype.postMessage 或 Worker.prototype. postMessage 没有影响。这样约定的原因很简单:工作者线程脚本的源被限制为主页的源，因此没有 必要再去过滤了。

#### 使用 MessageChannel

无论主线程还是工作者线程，通过 postMessage()进行通信涉及调用全局对象上的方法，并定义 一个临时的传输协议。这个过程可以被 Channel Messaging API 取代，基于该 API 可以在两个上下文间明 确建立通信渠道。
MessageChannel 实例有两个端口，分别代表两个通信端点。要让父页面和工作线程通过 MessageChannel 通信，需要把一个端口传到工作者线程中，如下所示:

```js
// worker.js
// 在监听器中存储全局messagePort 
let messagePort = null;
function factorial(n) {
  let result = 1;
  while (n) { result *= n--; }
  return result;
}
// 在全局对象上添加消息处理程序 
self.onmessage = ({ ports }) => {
// 只设置一次端口
if (!messagePort) {
// 初始化消息发送端口， // 给变量赋值并重置监听器
    messagePort = ports[0];
    self.onmessage = null;
  // 在全局对象上设置消息处理程序 
  messagePort.onmessage = ({ data }) => {
    // 收到消息后发送数据
    messagePort.postMessage(`${data}! = ${factorial(data)}`);
  };
  } 
};

// main.js
const channel = new MessageChannel();
const factorialWorker = new Worker('./worker.js');
// 把`MessagePort`对象发送到工作者线程
// 工作者线程负责处理初始化信道 
factorialWorker.postMessage(null, [channel.port1]);
// 通过信道实际发送数据
channel.port2.onmessage = ({ data }) => console.log(data);
// 工作者线程通过信道响应 
channel.port2.postMessage(5);
// 5! = 120

```

#### 使用 BroadcastChannel

同源脚本能够通过 BroadcastChannel 相互之间发送和接收消息。这种通道类型的设置比较简单， 不需要像 MessageChannel 那样转移乱糟糟的端口。这可以通过以下方式实现:

```js
// main.js
const channel = new BroadcastChannel('worker_channel');
const worker = new Worker('./worker.js');
channel.onmessage = ({data}) => {
  console.log(`heard ${data} on page`);
}
setTimeout(() => channel.postMessage('foo'), 1000);
// heard foo in worker
// heard bar on page
// worker.js
const channel = new BroadcastChannel('worker_channel');
channel.onmessage = ({data}) => {
  console.log(`heard ${data} in worker`);
  channel.postMessage('bar');
}
```

这里，页面在通过 BroadcastChannel 发送消息之前会先等 1 秒钟。因为这种信道没有端口所有 权的概念，所以如果没有实体监听这个信道，广播的消息就不会有人处理。在这种情况下，如果没有 setTimeout()，则由于初始化工作者线程的延迟，就会导致消息已经发送了，但工作者线程上的消息 处理程序还没有就位。
