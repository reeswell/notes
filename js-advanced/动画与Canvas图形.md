# 动画与 Canvas 图形

## requestAnimationFrame

```html
<div id="status" style="background-color: blue; height: 20px; width: 0%;"></div>

<script>
  function updateProgress() {
    let div = document.getElementById("status");
    div.style.width = (parseInt(div.style.width, 10) + 5) + "%";
    if (div.style.left != "100%") {
      requestAnimationFrame(updateProgress);
    }
  }
  requestAnimationFrame(updateProgress);
</script>
```

### cancelAnimationFrame

与 setTimeout()类似，requestAnimationFrame()也返回一个请求 ID，可以用于通过另一个 方法 cancelAnimationFrame()来取消重绘任务。下面的例子展示了刚把一个任务加入队列又立即将 其取消:

```js
let requestID = window.requestAnimationFrame(() => {
  console.log('Repaint!');
  window.cancelAnimationFrame(requestID);
})
```

### 通过requestAnimationFrame节流

先来看一个原生实现，其中的滚动事件监听器每次触发都会调用名为 expensiveOperation()(耗 时操作)的函数。当向下滚动网页时，这个事件很快就会被触发并执行成百上千次:

```js
function expensiveOperation() {
      console.log('Invoked at', Date.now());
}
 window.addEventListener('scroll', () => {
      expensiveOperation();
});
```

如果想把事件处理程序的调用限制在每次重绘前发生，那么可以像这样下面把它封装到 request- AnimationFrame()调用中:

```js
function expensiveOperation() {
  console.log('Invoked at', Date.now());
}
window.addEventListener('scroll', () => {
  window.requestAnimationFrame(expensiveOperation);
});
// 60帧的电脑 大概16ms执行一次
// Invoked at 1684252146287
// Invoked at 1684252146303
// Invoked at 1684252146317
```

这样会把所有回调的执行集中在重绘钩子，但不会过滤掉每次重绘的多余调用。此时，定义一个标 志变量，由回调设置其开关状态，就可以将多余的调用屏蔽:

```js
let enqueued = false;
function expensiveOperation() { 
  console.log('Invoked at', Date.now());
  enqueued = false;
}
window.addEventListener('scroll', () => {
  if (!enqueued) {
    enqueued = true;
    window.requestAnimationFrame(expensiveOperation);
  }
});
```

因为重绘是非常频繁的操作，所以这还算不上真正的节流。更好的办法是配合使用一个计时器来限 制操作执行的频率。这样，计时器可以限制实际的操作执行间隔，而 requestAnimationFrame 控制 在浏览器的哪个渲染周期中执行。下面的例子可以将回调限制为不超过 50 毫秒执行一次:

```js

let enabled = true;
function expensiveOperation() {
  console.log('Invoked at', Date.now());
}
window.addEventListener('scroll', () => {
  if (enabled) {
    enabled = false;
    window.requestAnimationFrame(expensiveOperation);
    window.setTimeout(() => enabled = true, 50);
  }
});
```
