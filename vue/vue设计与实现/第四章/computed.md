```js
   // 计算属性
    function computed(getter) {
      let value
      let dirty = true
      // 把getter作为副作用函数，创建一个lazy的effect
      const effectFn = effect(getter, {
        lazy: true,
        scheduler: () => {
          dirty = true
          // 当计算属性依赖发生变化时，手动调用trigger函数触发响应
          console.log(obj);
          trigger(obj, 'value')
        }
      })
      const obj = {
        // 当读取value时才执行effectFn
        get value() {
          if (dirty) {
            value = effectFn();
            dirty = false
          }

          // 当读取value时，手动调用 track 函数进行追踪
          track(obj, 'value')
          return value
        }
      }
      return obj
    }
// 分析问题的原因， 我们发现， 从本质上看这就是一个典型的effect嵌套。 一个计算属性内部拥有自己的effect， 并且它是懒执行的，
// 只有当真正读取计算属性的值时才会执行。 对于计算属性的getter函数来说。 它里面访问的响应式数据只会把computed内部的effect收集为依赖。
//  而当把计算属性用于另外一个 effect 时， 就会发生 effect 嵌套， 外层的 effect 不会被内层effest 中的响应式数据收集。
// 解决办法很简单。 当读取计算属性的值时， 我们可以手动调用track函数进行追踪； 当计算属性依赖的响应式数据发生变化时， 我们可以手动调用trigger函数触发响应：
```
