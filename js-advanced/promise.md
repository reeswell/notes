# Promise

## Promise基础

1、 通过执行函数控制期约状态

```js
new Promise(() => setTimeout(console.log, 0, 'executor'));
setTimeout(console.log, 0, 'promise initialized');
// executor
// promise initialized

let p = new Promise((resolve, reject) => setTimeout(resolve, 1000));
setTimeout(console.log, 0, p); 
// 在console.log打印期约实例的时候，还不会执行超时回调(即resolve()) // Promise <pending>

```

2、为避免期约卡在待定状态，可以添加一个定时退出功能。比如，可以通过 setTimeout 设置一个 7 10 秒钟后无论如何都会拒绝期约的回调:

```js
let p = new Promise((resolve, reject) => { setTimeout(reject,10000); //10秒后调用reject() // 执行函数的逻辑
});
setTimeout(console.log, 0, p);      // Promise <pending>
setTimeout(console.log,11000,p); //11秒后再检查状态 // (After 10 seconds) Uncaught error
// (After 11 seconds) Promise <rejected>
```

3、 Promise.resolve()
期约并非一开始就必须处于待定状态，然后通过执行器函数才能转换为落定状态。通过调用
Promise.resolve()静态方法，可以实例化一个解决的期约。下面两个期约实例实际上是一样的:

```js
let p1 = new Promise((resolve, reject) => resolve()); 
let p2 = Promise.resolve();
```

对这个静态方法而言，如果传入的参数本身是一个期约，那它的行为就类似于一个空包装。因此， Promise.resolve()可以说是一个幂等方法，如下所示:

```js
let p = Promise.resolve(7);
setTimeout(console.log, 0, p === Promise.resolve(p));
// true
setTimeout(console.log, 0, p === Promise.resolve(Promise.resolve(p))); // true
```

这个幂等性会保留传入期约的状态:

```js

let p = new Promise(() => {});
setTimeout(console.log, 0, p); // Promise <pending> setTimeout(console.log, 0, Promise.resolve(p)); // Promise <pending>
setTimeout(console.log, 0, p === Promise.resolve(p)); // true
```

4、 Promise.reject()
与 Promise.resolve()类似，Promise.reject()会实例化一个拒绝的期约并抛出一个异步错误 (这个错误不能通过 try/catch 捕获，而只能通过拒绝处理程序捕获)。下面的两个期约实例实际上是
一样的:

```js
let p1 = new Promise((resolve, reject) => reject());
let p2 = Promise.reject();
```

这个拒绝的期约的理由就是传给 Promise.reject()的第一个参数。这个参数也会传给后续的拒 绝处理程序:

```js
let p = Promise.reject(3);
setTimeout(console.log, 0, p); // Promise <rejected>: 3
p.then(null, (e) => setTimeout(console.log, 0, e)); // 3
```

关键在于，Promise.reject()并没有照搬 Promise.resolve()的幂等逻辑。如果给它传一个期 约对象，则这个期约会成为它返回的拒绝期约的理由:

```js
setTimeout(console.log, 0, Promise.reject(Promise.resolve()));
// Promise <rejected>: Promise <resolved>
```

5、 同步/异步执行的二元性
Promise 的设计很大程度上会导致一种完全不同于 JavaScript 的计算模式。下面的例子完美地展示
了这一点，其中包含了两种模式下抛出错误的情形:

```js
try {
  throw new Error('foo');
} catch(e) {
console.log(e); // Error: foo
}
try {
  Promise.reject(new Error('bar'));
} catch(e) {
  console.log(e);
}
// Uncaught (in promise) Error: bar
```

第一个 try/catch 抛出并捕获了错误，第二个 try/catch 抛出错误却没有捕获到。乍一看这可能 有点违反直觉，因为代码中确实是同步创建了一个拒绝的期约实例，而这个实例也抛出了包含拒绝理由 的错误。这里的同步代码之所以没有捕获期约抛出的错误，是因为它没有通过异步模式捕获错误。从这 里就可以看出期约真正的异步特性:它们是同步对象(在同步执行模式中使用)，但也是异步执行模式 的媒介。
在前面的例子中，拒绝期约的错误并没有抛到执行同步代码的线程里，而是通过浏览器异步消息队 列来处理的。因此，try/catch 块并不能捕获该错误。代码一旦开始以异步模式执行，则唯一与之交互 的方式就是使用异步结构——更具体地说，就是期约的方法。

## Promise实例方法

### Promise.prototype.then()

如前所述，两个处理程序参数都是可选的。而且，传给 then()的任何非函数类型的参数都会被静 默忽略。如果想只提供 onRejected 参数，那就要在 onResolved 参数的位置上传入 undefined。这 样有助于避免在内存中创建多余的对象，对期待函数参数的类型系统也是一个交代。

```js
function onResolved(id) {
  setTimeout(console.log, 0, id, 'resolved');
}
function onRejected(id) {
  setTimeout(console.log, 0, id, 'rejected');
}
let p1 = new Promise((resolve, reject) => setTimeout(resolve, 3000)); 
let p2 = new Promise((resolve, reject) => setTimeout(reject, 3000));
// 非函数处理程序会被静默忽略，不推荐 
p1.then('gobbeltygook');
// 不传 onResolved 处理程序的规范写法 
p2.then(null, () => onRejected('p2'));
// p2 rejected(3 秒后) 
// Promise.prototype.then()方法返回一个新的期约实例:
let p1 = new Promise(() => {});
let p2 = p1.then();
setTimeout(console.log, 0, p1);         // Promise <pending>
setTimeout(console.log, 0, p2);         // Promise <pending>
setTimeout(console.log, 0, p1 === p2);  // false
```

这个新期约实例基于 onResovled 处理程序的返回值构建。换句话说，该处理程序的返回值会通过 Promise.resolve()包装来生成新期约。如果没有提供这个处理程序，则 Promise.resolve()就会 包装上一个期约解决之后的值。如果没有显式的返回语句，则 Promise.resolve()会包装默认的返回 值 undefined。

```js
let p1 = Promise.resolve('foo');
// 若调用then()时不传处理程序，则原样向后传 let p2 = p1.then();
setTimeout(console.log, 0, p2); // Promise <resolved>: foo
// 这些都一样
let p3 = p1.then(() => undefined);
let p4 = p1.then(() => {});
let p5 = p1.then(() => Promise.resolve());
setTimeout(console.log, 0, p3);  // Promise <resolved>:
setTimeout(console.log, 0, p4);  // Promise <resolved>:
setTimeout(console.log, 0, p5);  // Promise <resolved>:
```

```js
如果有显式的返回值，则 Promise.resolve()会包装这个值: ...
// 这些都一样
let p6 = p1.then(() => 'bar');
let p7 = p1.then(() => Promise.resolve('bar'));
let p1 = Promise.reject('foo'); 13
setTimeout(console.log, 0, p6);  // Promise <resolved>: bar
setTimeout(console.log, 0, p7);  // Promise <resolved>: bar
undefined
undefined
undefined

     // Promise.resolve()保留返回的期约
let p8 = p1.then(() => new Promise(() => {})); let p9 = p1.then(() => Promise.reject());
// Uncaught (in promise): undefined
setTimeout(console.log, 0, p8);  // Promise <pending>
setTimeout(console.log, 0, p9);  // Promise <rejected>:
```

抛出异常会返回拒绝的期约:

```js
...
let p10 = p1.then(() => { throw 'baz'; });
// Uncaught (in promise) baz
setTimeout(console.log, 0, p10);  // Promise <rejected> baz
```

注意，返回错误值不会触发上面的拒绝行为，而会把错误对象包装在一个解决的期约中:

```js
...
let p11 = p1.then(() => Error('qux'));
setTimeout(console.log, 0, p11); // Promise <resolved>: Error: qux
```

Promise.reject()替代之前例子中的 Promise.resolve()是一样的道理

### Promise.prototype.catch()

Promise.prototype.catch()方法用于给期约添加拒绝处理程序。这个方法只接收一个参数:
onRejected 处理程序。事实上，这个方法就是一个语法糖，调用它就相当于调用 Promise.prototype. then(null, onRejected)。

 下面的代码展示了这两种同样的情况:

 ```js
let p = Promise.reject();
let onRejected = function(e) {
  setTimeout(console.log, 0, 'rejected');
};
// 这两种添加拒绝处理程序的方式是一样的:
p.then(null, onRejected); // rejected 
p.catch(onRejected); // rejected
 ```

Promise.prototype.catch()返回一个新的期约实例:

```js
let p1 = new Promise(() => {});
let p2 = p1.catch();
setTimeout(console.log, 0, p1);
setTimeout(console.log, 0, p2);
setTimeout(console.log, 0, p1 === p2);  // false
```

### 拒绝期约与拒绝错误处理

```js
let p1 = new Promise((resolve, reject) => reject(Error('foo')));
let p2 = new Promise((resolve, reject) => { throw Error('foo'); });
let p3 = Promise.resolve().then(() => { throw Error('foo'); });
let p4 = Promise.reject(Error('foo'));
setTimeout(console.log, 0, p1);  // Promise <rejected>: Error: foo
setTimeout(console.log, 0, p2);  // Promise <rejected>: Error: foo
setTimeout(console.log, 0, p3);  // Promise <rejected>: Error: foo
setTimeout(console.log, 0, p4);  // Promise <rejected>: Error: foo
```

```shell
Uncaught (in promise) Error: foo
at Promise (test.html:5)
at new Promise (<anonymous>)
at test.html:5

Uncaught (in promise) Error: foo
at Promise (test.html:6) 6 at new Promise (<anonymous>)
at test.html:6
Uncaught (in promise) Error: foo
at test.html:8
Uncaught (in promise) Error: foo
at Promise.resolve.then (test.html:7) 
```

这个例子同样揭示了异步错误有意思的副作用。正常情况下，在通过 throw()关键字抛出错误时， JavaScript 运行时的错误处理机制会停止执行抛出错误之后的任何指令:
所有错误都是异步抛出且未处理的，通过错误对象捕获的栈追踪信息展示了错误发生的路径。注意 错误的顺序:Promise.resolve().then()的错误最后才出现，这是因为它需要在运行时消息队列中 添加处理程序;也就是说，在最终抛出未捕获错误之前它还会创建另一个期约。

```js
 throw Error('foo'); console.log('bar'); // 这一行不会执行
// Uncaught Error: foo
```

但是，在期约中抛出错误时，因为错误实际上是从消息队列中异步抛出的，所以并不会阻止运行时 继续执行同步指令:

```js

 Promise.reject(Error('foo')); console.log('bar');
// bar
// Uncaught (in promise) Error: foo
```

如本章前面的 Promise.reject()示例所示，异步错误只能通过异步的 onRejected 处理程序 捕获:

```js
// 正确
Promise.reject(Error('foo')).catch((e) => {});
// 不正确
try {
Promise.reject(Error('foo'));
} catch(e) {}
```

### 串行期约合成

```js
用的合成函数可以这样实现:
function addTwo(x) {return x + 2;}
function addThree(x) {return x + 3;}
function addFive(x) {return x + 5;}
function compose(...fns) {
return (x) => fns.reduce((promise, fn) => promise.then(fn), Promise.resolve(x))
}
let addTen = compose(addTwo, addThree, addFive); addTen(8).then(console.log); // 18
```

### 期约取消

```html
<body>
  <button id="start">Start</button>
  <button id="cancel">Cancel</button>
  <script>
    class CancelToken {
      constructor(cancelFn) {
        this.promise = new Promise((resolve, reject) => {
          cancelFn(() => {
            setTimeout(console.log, 0, "delay cancelled");
            resolve();
          });
        })
      }

    }
    const startButton = document.querySelector('#start');
    const cancelButton = document.querySelector('#cancel');
    function cancellableDelayedResolve(delay) {
      setTimeout(console.log, 0, "set delay");
      return new Promise((resolve, reject) => {
        const id = setTimeout((() => {
          setTimeout(console.log, 0, "delayed resolve");
          resolve();
        }), delay);

        const cancelToken = new CancelToken((cancelCallback) =>
          cancelButton.addEventListener("click", cancelCallback));
        cancelToken.promise.then(() => clearTimeout(id));
      });
    }
    startButton.addEventListener("click", () => cancellableDelayedResolve(1000)); 
  </script>
</body>
```

### 期约进度通知

```js
class TrackablePromise extends Promise {
  constructor(executor) {
    const notifyHandlers = [];
    super((resolve, reject) => {
      return executor(resolve, reject, (status) => {
        notifyHandlers.map((handler) => handler(status));
      });
    });
    this.notifyHandlers = notifyHandlers;
  }
  notify(notifyHandler) {
    this.notifyHandlers.push(notifyHandler);
    return this;
  }
}
let p = new TrackablePromise((resolve, reject, notify) => {
  function countdown(x) {
    if (x > 0) {
      notify(`${20 * x}% remaining`);
      setTimeout(() => countdown(x - 1), 1000);
    } else {
      resolve();
    }
  }
  countdown(5);
});
p.notify((x) => setTimeout(console.log, 0, 'progress:', x));
p.then(() => setTimeout(console.log, 0, 'completed'));
// (约 1 秒后)80% remaining 
// (约 2 秒后)60% remaining
// (约 3 秒后)40% remaining
// (约 4 秒后)20% remaining
// (约 5 秒后)completed

// notify()函数会返回期约，所以可以连缀调用，连续添加处理程序。多个处理程序会针对收到的
// 每条消息分别执行一遍，如下所示:
p.notify((x) => setTimeout(console.log, 0, 'a:', x))
  .notify((x) => setTimeout(console.log, 0, 'b:', x));
p.then(() => setTimeout(console.log, 0, 'completed'));
// (约 1 秒后) a: 80% remaining 
// (约 1 秒后) b: 80% remaining 
// (约 2 秒后) a: 60% remaining 
// (约 2 秒后) b: 60% remaining 
// (约 3 秒后) a: 40% remaining 
// (约 3 秒后) b: 40% remaining 
// (约 4 秒后) a: 20% remaining 
// (约 4 秒后) b: 20% remaining 
// (约 5 秒后) completed
// 总体来看，这还是一个比较粗糙的实现，但应该可以演示出如何使用通知报告进度了
```

```js
  async function foo() {
      console.log(2);
      console.log(await Promise.resolve(8));
      console.log(9);
    }
    async function bar() {
      console.log(4);
      console.log(await 6);
      console.log(7);
    }
    console.log(1);
    foo();
    console.log(3);
    bar();
    console.log(5);

  // 1
  // 2
  // 3
  // 4
  // 5
  // 8
  // 9
  // 6
  // 7
```

## 异步函数策略

### 实现 sleep()

```js
async function sleep(delay) {
  return new Promise((resolve) => setTimeout(resolve, delay));
}
async function foo() {
  const t0 = Date.now();
  await sleep(1500); // 暂停约 1500 毫秒 
  console.log(Date.now() - t0);
}
foo();
// 1502
```

### 利用平行执行

```js
async function randomDelay(id) {
  // 延迟 0~1000 毫秒
  const delay = Math.random() * 1000;
  return new Promise((resolve) => setTimeout(() => {
    console.log(`${id} finished`);
    resolve();
  }, delay));
}
async function foo() {
  const t0 = Date.now();
  await randomDelay(0);
  await randomDelay(1);
  await randomDelay(2);
  await randomDelay(3);
  await randomDelay(4);
  console.log(`${Date.now() - t0}ms elapsed`);
} 
foo();
// 0 finished
// 1 finished
// 2 finished
// 3 finished
// 4 finished
// 877ms elapsed
```

用一个 for 循环重写，就是:

```js
async function randomDelay(id) {
// 延迟 0~1000 毫秒
const delay = Math.random() * 1000;
return new Promise((resolve) => setTimeout(() => {
    console.log(`${id} finished`);
    resolve();
  }, delay));
}
async function foo() {
const t0 = Date.now();
for (let i = 0; i < 5; ++i) {
    await randomDelay(i);
}
console.log(`${Date.now() - t0}ms elapsed`);
}
foo();
// 0 finished
// 1 finished
// 2 finished
// 3 finished
// 4 finished
// 877ms elapsed
```

就算这些期约之间没有依赖，异步函数也会依次暂停，等待每个超时完成。这样可以保证执行顺序， 7 但总执行时间会变长。
如果顺序不是必需保证的，那么可以先一次性初始化所有期约，然后再分别等待它们的结果。比如:

```js
  async function foo() {
    const t0 = Date.now();
    const p0 = randomDelay(0);
    const p1 = randomDelay(1);
    const p2 = randomDelay(2); 
    const p3 = randomDelay(3);
    const p4 = randomDelay(4);

    await p0;
    await p1;
    await p2;
    await p3;
    await p4;
    console.log(`${Date.now() - t0}ms elapsed`);
  }
   foo();
  // 2 finished
  // 0 finished
  // 1 finished
  // 3 finished
  // 4 finished
  // 1002ms elapsed

```

用数组和 for 循环再包装一下就是:

```js

async function foo() {
  const t0 = Date.now();
  const promises = Array(5).fill(null).map((_, i) => randomDelay(i));
  for (const p of promises) {
   await p;
  }
  console.log(`${Date.now() - t0}ms elapsed`);
}
foo();
// 4 finished
// 2 finished
// 1 finished
// 0 finished
// 3 finished
// 877ms elapsed
```

注意，虽然期约没有按照顺序执行，但 await 按顺序收到了每个期约的值:

```js
async function randomDelay(id) {
  // 延迟 0~1000 毫秒
  const delay = Math.random() * 1000;
  return new Promise((resolve) => setTimeout(() => {
    console.log(`${id} finished`);
    resolve(id);
  }, delay));
}
async function foo() {
  const t0 = Date.now();
  const promises = Array(5).fill(null).map((_, i) => randomDelay(i));
  for (const p of promises) { // 与Promise.all(promises)类似
    console.log(`awaited ${await p}`);
  } 
  console.log(`${Date.now() - t0}ms elapsed`);
}
foo();
// 1 finished
// 2 finished
// 4 finished
// 3 finished
// 0 finished
// awaited 0
// awaited 1
// awaited 2
// awaited 3
// awaited 4
// 645ms elapsed
```

### 平行控制并发数

```js


async function asyncPool(poolLimit, array, iteratorFn) {
  const results = [];
  const pending = [];

  for (const item of array) {
    const p = iteratorFn(item).then((result) => {
      const index = pending.indexOf(p);
      if (index !== -1) {
        pending.splice(index, 1);
      }
      return result;
    });

    results.push(p);

    if (poolLimit <= array.length) {
      const e = p.then(() => pending.length);
      console.log(e)
      pending.push(e);
      if (pending.length >= poolLimit) {
        await Promise.race(pending);
      }
    }
  }

  return Promise.all(results);
}

async function randomDelay(id) {
  const delay = Math.random() * 1000;
  return new Promise((resolve) => setTimeout(() => {
    console.log(`${id} finished`);
    resolve();
  }, delay));
}

async function foo() {
  const t0 = Date.now();
  const tasks = Array(5).fill(null).map((_, i) => i);
  await asyncPool(3, tasks, randomDelay);
  console.log(`${Date.now() - t0}ms elapsed`);
}
// asyncPool ES9版本 最新版
async function* asyncPool(concurrency, iterable, iteratorFn) {
  const executing = new Set();
  async function consume() {
    const [promise, value] = await Promise.race(executing);
    executing.delete(promise);
    return value;
  }
  for (const item of iterable) {
    // Wrap iteratorFn() in an async fn to ensure we get a promise.
    // Then expose such promise, so it's possible to later reference and
    // remove it from the executing pool.
    const promise = (async () => await iteratorFn(item, iterable))().then(
      value => [promise, value]
    );
    executing.add(promise);
    if (executing.size >= concurrency) {
      yield await consume();
    }
  }
  while (executing.size) {
    yield await consume();
  }
}
// asyncPool ES7版本 1.0版本
async function asyncPool(poolLimit, iterable, iteratorFn) {
  const ret = [];
  const executing = new Set();
  for (const item of iterable) {
    const p = Promise.resolve().then(() => iteratorFn(item, iterable));
    ret.push(p);
    executing.add(p);
    const clean = () => executing.delete(p);
    p.then(clean).catch(clean);
    if (executing.size >= poolLimit) {
      await Promise.race(executing);
    }
  }
  return Promise.all(ret);
}
foo();
```

### 串行执行期约

这里，await 直接传递了每个函数的返回值，结果通过迭代产生。当然，这个例子并没有使用期约， 如果要使用期约，则可以把所有函数都改成异步函数。这样它们就都返回期约了:

```js
async function addTwo(x) {return x + 2;}
async function addThree(x) {return x + 3;}
async function addFive(x) {return x + 5;}
async function addTen(x) {
  for (const fn of [addTwo, addThree, addFive]) {
    x = await fn(x);
  }
return x; 
}
addTen(9).then(console.log); // 19
```

### 栈追踪与内存管理

期约与异步函数的功能有相当程度的重叠，但它们在内存中的表示则差别很大。看看下面的例子，
它展示了拒绝期约的栈追踪信息:

```js
function fooPromiseExecutor(resolve, reject) {
  setTimeout(reject, 1000, 'bar');
}
function foo() {
  new Promise(fooPromiseExecutor);
}
foo();
// Uncaught (in promise) bar // setTimeout
// setTimeout (async)
// fooPromiseExecutor
// foo
```

根据对期约的不同理解程度，以上栈追踪信息可能会让某些读者不解。栈追踪信息应该相当直接地 表现 JavaScript 引擎当前栈内存中函数调用之间的嵌套关系。在超时处理程序执行时和拒绝期约时，我 们看到的错误信息包含嵌套函数的标识符，那是被调用以创建最初期约实例的函数。可是，我们知道这 些函数已经返回了，因此栈追踪信息中不应该看到它们。
答案很简单，这是因为 JavaScript 引擎会在创建期约时尽可能保留完整的调用栈。在抛出错误时， 调用栈可以由运行时的错误处理逻辑获取，因而就会出现在栈追踪信息中。当然，这意味着栈追踪信息 会占用内存，从而带来一些计算和存储成本。
如果在前面的例子中使用的是异步函数，那又会怎样呢?比如:

```js
function fooPromiseExecutor(resolve, reject) {
  setTimeout(reject, 1000, 'bar');
}
async function foo() {
  await new Promise(fooPromiseExecutor);
} 
foo();
// Uncaught (in promise) bar
// foo
// async function (async)
// foo

```

这样一改，栈追踪信息就准确地反映了当前的调用栈。fooPromiseExecutor()已经返回，所以 它不在错误信息中。但 foo()此时被挂起了，并没有退出。JavaScript 运行时可以简单地在嵌套函数中 存储指向包含函数的指针，就跟对待同步函数调用栈一样。这个指针实际上存储在内存中，可用于在出 错时生成栈追踪信息。这样就不会像之前的例子那样带来额外的消耗，因此在重视性能的应用中是可以 优先考虑的。

### 使用 ReadableStream 主体

ReadableStream 暴露了 getReader()方法，用于产生 ReadableStream- DefaultReader，这个读取器可以用于在数据到达时异步获取数据块。数据流的格式是 Uint8Array。

```js
fetch('https://fetch.spec.whatwg.org/')
  .then((response) => response.body)
  .then((body) => {
    let reader = body.getReader();
    function processNextChunk({ value, done }) {
      if (done) {
        return;
      }
      console.log(value);
      return reader.read()
        .then(processNextChunk);
    }
    return reader.read()
      .then(processNextChunk);
  });
// { value: Uint8Array{}, done: false }
// { value: Uint8Array{}, done: false }
// { value: Uint8Array{}, done: false }
// ...
```

异步函数非常适合这样的 fetch()操作。可以通过使用 async/await 将上面的递归调用打平:

```js
  fetch('https://fetch.spec.whatwg.org/')
    .then((response) => response.body)
    .then(async function (body) {
      let reader = body.getReader();
      while (true) {
        if (done) {
          break;
        }
        console.log(value);
      }
    });
// { value: Uint8Array{}, done: false }
// { value: Uint8Array{}, done: false }
// { value: Uint8Array{}, done: false }
// ...
```

另外，read()方法也可以真接封装到 Iterable 接口中。因此就可以在 for-await-of 循环中方 便地实现这种转换:

```js
fetch('https://fetch.spec.whatwg.org/')
  .then((response) => response.body)
  .then(async function (body) {
      let reader = body.getReader();
      let asyncIterable = {
        [Symbol.asyncIterator]() {
          return {
            next() {
              return reader.read();
            }
          };
        }
      };
      for await (chunk of asyncIterable) {
        console.log(chunk);
      }
});
// { value: Uint8Array{}, done: false }
// { value: Uint8Array{}, done: false }
// { value: Uint8Array{}, done: false }
// ...
```

通过将异步逻辑包装到一个生成器函数中，还可以进一步简化代码。而且，这个实现通过支持只读 取部分流也变得更稳健。如果流因为耗尽或错误而终止，读取器会释放锁，以允许不同的流读取器继续 操作:

```js
async function* streamGenerator(stream) {
  const reader = stream.getReader();
  try {
    while (true) {
      const { value, done } = await reader.read();
      if (done) {
        break;
      }
      yield value;
    }
  } finally {
    reader.releaseLock();
  }
}
fetch('https://fetch.spec.whatwg.org/')
  .then((response) => response.body)
  .then(async function (body) {
    for await (chunk of streamGenerator(body)) {
      console.log(chunk);
    }
  });
```

## 异步迭代

### 创建并使用异步迭代器

要理解异步迭代器，最简单的办法是用它跟同步迭代器进行比较。下面代码中创建了一个简单的 Emitter 类，该类包含一个同步生成器函数，该函数会产生一个同步迭代器，同步迭代器输出 0~4:

```js
class Emitter {
  constructor(max) {
    this.max = max;
    this.syncIdx = 0;
  }
  *[Symbol.iterator]() {
    while (this.syncIdx <= this.max) {
      yield this.syncIdx++;
    }
  }
}
const emitter = new Emitter(5);
function syncCount() {
  const syncCounter = emitter[Symbol.iterator]();
  for (const x of syncCounter) {
    console.log(x);
  }
}
syncCount();
// 0
// 1
// 2
// 3 
// 4
```

这个例子之所以可以运行起来，主要是因为迭代器可以立即产生下一个值。假如你不想在确定下一 个产生的值时阻塞主线程执行，也可以定义异步迭代器函数，让它产生期约包装的值。
为此，要使用迭代器和生成器的异步版本。ECMAScript 2018 为此定义了 Symbol.asyncIterator， 以便定义和调用输出期约的生成器函数。同时，这一版规范还为异步迭代器增加了 for-await-of 循环， 用于使用异步迭代器。
  相应地，前面的例子可以扩展为同时支持同步和异步迭代:

```js
class Emitter {
  constructor(max) {
    this.max = max; this.syncIdx = 0; this.asyncIdx = 0;
  }
  *[Symbol.iterator]() {
    while (this.syncIdx < this.max) {
      yield this.syncIdx++;
    }
  }
  async *[Symbol.asyncIterator]() {
    // *[Symbol.asyncIterator]() {
    while (this.asyncIdx < this.max) {
      // yield new Promise((resolve) => resolve(this.asyncIdx++));
      yield this.asyncIdx++
    }
  }
}
const emitter = new Emitter(5);
function syncCount() {
  const syncCounter = emitter[Symbol.iterator]();
  for (const x of syncCounter) {
    console.log(x);
  }
}
async function asyncCount() {
  const asyncCounter = emitter[Symbol.asyncIterator]();
  for await (const x of asyncCounter) {
    console.log(x);
  }
}
syncCount();
// 0
// 1
// 2
// 3 
// 4
asyncCount(); // 这里是串行异步迭代
// 0
// 1
// 2
// 3 
// 4
```

为了加深理解，可以把前面例子中的同步生成器传给 for-await-of 循环:

```js
const emitter = new Emitter(5);
async function asyncIteratorSyncCount() {
  const syncCounter = emitter[Symbol.iterator]();
  for await (const x of syncCounter) {
    console.log(x);
  }
}
asyncIteratorSyncCount(); // 这里是同步迭代
// 0
// 1
// 2
// 3 
// 4
```

虽然这里迭代的是同步生成器产生的原始值，但 for-await-of 循环仍像它们被包装在期约中一 样处理它们。这说明 for-await-of 循环可以流畅地处理同步和异步可迭代对象。但是常规 for 循环 就不能处理异步迭代器了:

```js
function syncIteratorAsyncCount() {
  const asyncCounter = emitter[Symbol.asyncIterator]();
  for (const x of asyncCounter) {
    console.log(x);
  }
}
syncIteratorAsyncCount(); 
// TypeError: asyncCounter is not iterable
```

关于异步迭代器，要理解的非常重要的一个概念是Symbol.asyncIterator符号不会改变生成器 函数的行为或者消费生成器的方式。注意在前面的例子中，生成器函数加上了 async 修饰符成为异步 函数，又加上了星号成为生成器函数。Symbol.asyncIterator 在这里只起一个提示的作用，告诉将 来消费这个迭代器的外部结构如 for-await-of 循环，这个迭代器会返回期约对象的序列。

### 处理异步迭代器的reject()

因为异步迭代器使用期约来包装返回值，所以必须考虑某个期约被拒绝的情况。由于异步迭代会按 顺序完成，而在循环中跳过被拒绝的期间是不合理的。因此，被拒绝的期约会强制退出迭代器:

```js
async *[Symbol.asyncIterator]() {
  while (this.asyncIdx < this.max) {
    if(this.asyncIdx < 3) {
      yield this.asyncIdx;
    } else {
      throw 'Exited loop'
    }
  
  }
}
asyncCount();
// 0
// 1
// 2  
// Uncaught (in promise) Exited loop

```

### 使用next()手动异步迭代

for-await-of 循环提供了两个有用的特性:一是利用异步迭代器队列保证按顺序执行，二是隐藏 异步迭代器的期约。不过，使用这个循环会隐藏很多底层行为。
因为异步迭代器仍遵守迭代器协议，所以可以使用 next()逐个遍历异步可迭代对象。如前所述， next()返回的值会包含一个期约，该期约可解决为{ value, done }这样的迭代结果。这意味着必须 使用期约 API 获取方法，同时也意味着可以不使用异步迭代器队列。

```js
const emitter = new Emitter(5);
const asyncCounter = emitter[Symbol.asyncIterator]();
console.log(asyncCounter.next());
// Promise<{value, done}>
```
