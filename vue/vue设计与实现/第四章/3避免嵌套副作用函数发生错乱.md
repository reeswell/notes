```js
// 用一个全局变量存储被注册的副作用函数
let activeEffect;
// effect 栈（避免嵌套副作用函数发生错乱）
const effectStack = [];

// 存储副作用函数的桶
const bucket = new WeakMap();
const data = { ok: true, text: "hello world" };

const obj = new Proxy(data, {
  //拦截读取操作
  get(target, key) {
    //将副作用函数 activeEffect 添加到存储副作用函数的桶中
    track(target, key);
    // 返回属性值
    return target[key];
  },
  //拦截设置操作
  set(target, key, newVal) {
    //设置属性值
    target[key] = newVal;
    // 把副作用函数从桶里取出并执行
    trigger(target, key);
  },
});

effect(function effectFn() {
  document.body.innerText = obj.ok ? obj.text : "";
});

//在get 拦截函数内调用 track 函数追踪变化
function track(target, key) {
  // 没有 activeEffect，直接 return
  if (!activeEffect) return;
  let depsMap = bucket.get(target);
  if (!depsMap) {
    bucket.set(target, (depsMap = new Map()));
  }
  let deps = depsMap.get(key);
  if (!deps) {
    depsMap.set(key, (deps = new Set()));
  }
  // 把当前激活的副作用函数添加到依赖集合deps中
  deps.add(activeEffect);
  // 将其添加到activeEffect.deps数组中
  activeEffect.deps.push(deps);
}
//在set 拦截函数内调用 trigger 函数触发变化
function trigger(target, key) {
  const depsMap = bucket.get(target);
  if (!depsMap) return;
  const effects = depsMap.get(key);
  const effectsToRun = new Set(effects);
  effectsToRun && effectsToRun.forEach((effectFn) => effectFn());
}

function effect(fn) {
  const effectFn = () => {
    // debugger
    //  调用cleanup 函数清除依赖
    cleanup(effectFn);
    // 当 effectFn 执行时，将其设置为当前激活的副作用函数
    activeEffect = effectFn;
     // 在调用副作用函数之前，将当前副作用函数压入 effectStack 栈中
    effectStack.push(effectFn);
    fn();
    // 当 effectFn 执行完毕，将其从 effectStack 中移除
    effectStack.pop();
    activeEffect = effectStack[effectStack.length - 1];
  };
  // activeEffect.deps 用来存储所有与该副作用函数相关联的依赖集合 effectFn.deps［］
  effectFn.deps = [];
  //执行副作用函数
  effectFn();
}

function cleanup(effectFn) {
  //遍历 effectFn.deps 数组
  for (let i = 0; i < effectFn.deps.length; i++) {
    // deps 是依赖集合
    const deps = effectFn.deps[i];
    //将 effectFn 从依赖集合中移除
    deps.delete(effectFn);
  }
  // 最后需要重置 effectFn.deps 数组
  effectFn.deps.length = 0;
}
```
