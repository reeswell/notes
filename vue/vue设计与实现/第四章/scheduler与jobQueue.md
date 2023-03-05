```js
//定义一个任务队列
const jobQueue = new Set()
// 使用 Promise.resolve()创建一个 promise 实例，我们用它将一个任务添加到微任务队列 
const p = Promise.resolve() //一个标志代表是否正在刷新队列
let isFlushing = false

function flushJob() { // 如果队列正在刷新， 则什么都不做
  if (isFlushing) return // 设置为 true， 代表正在刷新
  isFlushing = true
  // 在微任务队列中刷新 jobQueue 队列
  p.then(() => {
    jobQueue.forEach(job => job())
  }).finally(() => { //结束后重置 isFlushing 
    isFlushing = false
  })

}
effect(() => {
  console.log(obj.foo)
}, {
  scheduler(fn) {
    //／ 每次调度时， 将副作用函数添加到 jobQueue 队列中 
    jobQueue.add(fn)
    // 调用 flushJob 刷新队列 
    flushJob()
  }
})


//  obj.foo++
//  obj.foo++
// 面的代码， 首先， 我们定义了一个任务队列Joboueve， 它是一个Set数据结构， 目的是利用Set数据结构的自动去重能力。
// 接着我任务以别30u， 它实现， 在每次测度执行将当前副作用函数添加到Joboueue队列中， 有测度器schou新队列。
// 然后我们把自向ushJob函数， 该函数通过tsFlushtng标志判断是否需执行， 只有当其为false时才需要执行，
// 而一旦flushJob函数开始执行， tsFlushing 标志就会设置为true， 意思是无论调用多少次flushJob函数，
// 在一个周期内都只会执行一次。 需要注意的是， 在flushJob内通过p.then将一个函数添加到微任务队列， 在微任务队列内完成对jobQueue的遍历执行。
// 整段代码的效果是， 连续对obj.foo执行两次自增操作， 会同步且连续地执行两次 scheduler调度函数，
// 这意味着同一个副作用函数会被jobQueue.add（ fn） 语句添加两次， 但由于Set数据结构的去重能力，
// 最终JobQueue中只会有一项， 即当前副作用函数。 类似地， flushJob 也会同步且连续地执行两次，
// 但由于isFlushing标志的存在， 实际上flushJob函数在一个事件循环内只会执行一次， 即在微任务队列内执行一次。
// 当微任务队列开始执行时， 就会遍历 jobQueue 并执行里面存储的副作用函数。 由于此时jobQueue队列内只有一个副作用函数，
// 所以只会执行一次， 并且当它执行时， 字段obj.foo的值已经是3了， 这样我们就实现了期望的输出：
//  1
//  3
//  可能你已经注意到了， 这个功能有点类似于在Vue.js中连续多次修改响应式数据但只会触发
//  一次更新， 实际上Vue.js内部实现了一个更加完善的调度器， 思路与上文介绍的相同。
```
