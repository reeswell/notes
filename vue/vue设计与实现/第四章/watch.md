```js
// 这样就实现了回调函数的立即执行功能。 由于回调函数是立即执行的， 所以第一次回调执行
// 时没有所谓的旧值， 因此此时回调函数的oldValue值为undefined， 这也是符合预期的。
// 除了指定回调函数为立即执行之外， 还可以通过其他选项参数来指定回调函数的执行时机，
// 例如在Vue.js 3 中使用flush 选项来指定:
watch(obj, () => {
  console.log(' 变化了')
}, {
  //  回调函数会在watch 创建时立即执行一次
  flush: 'pre' // 还可以指定为' post'｜' sync'
})
// flush本质上是在指定调度函数的执行时机。 前文讲解过如何在微任务队列中执行调度函数
// scheduler， 这与flush的功能相同。 当flush的值为' post' 时， 代表调度函数需要将副作用函数放到一个微任务队列中， 并等待DOM更新结束后再执行， 我们可以用如下代码进行模拟：
function watch(source, cb, options = {}) {
  let getter
  if (typeof source == ' function') {
    getter = source
  } else {
    getter = () => traverse(source)
  }
  let oldValue, newValue
  const job = () => {
    newValue = effectfn()
    cb(newValue, oldValue)
    oldValue = newValue
  }
  const effectFn = effect(
    //执行 getter
    () => getter(), {
      lazy: true,
      scheduler: () => {
        //在调度函数中判断 flush 是否为＇ post＇， 如果是， 将其放到微任务队列中执行
        if (options.flush === 'post') {
          const p = Promise.resolve()
          p.then(job)
        } else {
          job()
        }
      }
    })
  if (options.immediate) {
    job()
  } else {
    oldValue = effectFn()
  }
}

// immediate>sync===default>pre>post
// 如以上代码所示， 我们修改了调度器函数 scheduler 的实现方式， 在调度器函数内检测options.flush的值是否为post，
// 如果是， 则将job函数放到微任务队列中， 从而实现异步延迟执行； 否则直接执行 job函数，
// 这本质上相当于＇ sync＇ 的实现机制， 即同步执行。 对于options.flush的值为＇ pre＇ 的情况，
// 我们暂时还没有办法模拟， 因为这涉及组件的更新时机， 其中＇ pre＇ 和＇ post＇ 原本的语义指的就是组件更新前和更新后， 不过这并不影响我们理解如何控制回调函数的更新时机。
```
