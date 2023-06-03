# 2023年核心网页指标九大优化

核心Web指标主要对应用户体验的三个方面——**加载性能**、**交互性**和**视觉稳定性**——并包括以下指标（及各指标相应的阈值）：

![image-20230602193244140](https://cdn.jsdelivr.net/gh/xxydrr/my_pic/img/image-20230602193244140.png)

- **[Largest Contentful Paint (LCP)](https://web.dev/lcp/)** ：最大内容绘制，测量*加载*性能。为了提供良好的用户体验，LCP 应在页面首次开始加载后的**2.5 秒**内发生。
- **[First Input Delay (FID)](https://web.dev/fid/)** ：首次输入延迟，测量*交互性*。为了提供良好的用户体验，页面的 FID 应为**100 毫秒**或更短。
- **[Cumulative Layout Shift (CLS)](https://web.dev/cls/)** ：累积布局偏移，测量*视觉稳定性*。为了提供良好的用户体验，页面的 CLS 应保持在 **0.1.** 或更少。

## LCP（最大内容绘制 Largest Contentful Paint）

### 使用标准html方式引入

大约72%的移动端图片作为LCP元素，这意味着想要优化LCP得优化图片加载速度。  图片资源在html中以标准的标准 HTML 属性存在浏览器更好找到并加载它。如下图所示，背景图片、客户端渲染、延迟加载这些都会导致浏览扫描不到从而导致图片资源的 LCP时间延迟

![image-20230602194146279](https://cdn.jsdelivr.net/gh/xxydrr/my_pic/img/image-20230602194146279.png)

更好的解决方式应该是使用传统的图像元素引入方式或者使用预加载链接引入。这些方式都能使用浏览器更快地发现图片资源并且加载它。如下图所示

![image-20230602195530787](https://cdn.jsdelivr.net/gh/xxydrr/my_pic/img/image-20230602195530787.png)

咱们可以根据瀑布流图分析时候导致图片资源被延迟加载了。如下图明显图片资源时候被延迟发现加载了的区别。

<p align='center'>
  <img src='https://cdn.jsdelivr.net/gh/xxydrr/my_pic/img/image-20230602200010707.png' alt='' width='400'/>
  <img src='https://cdn.jsdelivr.net/gh/xxydrr/my_pic/img/image-20230602200419690.png' alt='' width='400'/>
</p>
### fetchpriority

通过设置图片资源新属性fetchpriority设置优先级。如下图引入后瀑布流看到基本图片和其它js静态资源并行加载。不过该API的兼容性只有68.69%。

![image-20230602201044771](https://cdn.jsdelivr.net/gh/xxydrr/my_pic/img/image-20230602201044771.png)

![image-20230602201143829](https://cdn.jsdelivr.net/gh/xxydrr/my_pic/img/image-20230602201143829.png)

### CDN

使用CDN，则将内容分发到全球各地的服务器上，用户请求内容时，请求会发送到最接近用户的服务器，该服务器将响应请求并返回内容。这个过程加速了请求响应时间以及缓存策略，从而减少了首字节时间 (TTFB)。

## 累积布局偏移 （CLS Cumulative Layout Shift）

cls简单来说就是保证页面布局的稳定性，减少回流。从用户来就是保证他们视角的稳定性。

### 明确的内容的大小

```html
<!-- An image without dimensions wil1 cause CLS  -->
<img sre="myur1. jpeg" alt="...">

 <!-- Explicitly setting width and height  -->
<img src="myurl. jpeg" alt=" . .." width="400" height="200">
```

```html
<!-- Example: default videos to 16 / 9 aspect ratio  -->
video {
width: 100%;
height: auto;
aspect-ratio: 16/9;
}
```

使用aspect-ratio可以设置元素的宽高比。从而动态地设置widht、height。

使用min-height设置无法确认的内容大小，例如广告横幅。相比不设置高度，这也是简单减少CLS的方法。

### bfcache

[浏览器使用一种称为后退/前进缓存](https://web.dev/bfcache/)（简称 bfcache）的导航机制，直接从内存快照中立即加载浏览器历史记录中较早或较晚的页面。

浏览器默认开启bfcache，可以在DevToos中查看。如果无法使用，一般常见的 原因是使用了`cache-control:no-store`

![image-20230602204811222](https://cdn.jsdelivr.net/gh/xxydrr/my_pic/img/image-20230602204811222.png)

### transform

尽可能使用用 CSS属性`transform`的过渡和动画，因为它移动发生在合成器中运行。CSS Transform 可以创建一个新的图层，意味着它不会影响其他内容避免CLS。这个新图层中的元素会在合成器中运行。当使用 CSS Transform 对元素进行变换时，浏览器会将这个元素的图层标记为需要重新合成，然后将这个图层发送给合成器进行处理。合成器会将这个图层中的元素进行变换，并将结果与其他图层组合成最终的页面呈现。

```html
<!-- Instead, use transform to move with translate  -->
box {
  position: absolute;
  top: 10px;
  left: 10px;
  animation: move 3s ease
}

@keyframes move {
0% {
  <!-- 而不是 top calc(90vh 160px); -->
  transform: translateY (calc(90vh 160px)); 
  }
}
```

## FID ( First Input Delay  首次输入延迟，交互响应延迟)

### 避免或分解长任务

这个很好理解，javascript是单线程的，长任务将会阻塞主线程响应用户输入响应。将非关键任务放到异步任务中。

```js
function saveSettings() {
  // Do critical work that is user-visible:
  validateForm();
  showSpinner();
  updateUI();

  // Defer work that isn' t user-visible to a separate task:
  setTimeout(()=> {
  saveToDatabase();
  sendAnalytics();
  }, 0);
}
```

另外可以使用任务调度器处理，不过兼容性还不是很好

![image-20230602231001265](https://cdn.jsdelivr.net/gh/xxydrr/my_pic/img/image-20230602231001265.png)

### 避免不必要的 JavaScript

通过减少启动时需要的资源大小，你可以确保你的网站花在解析和编译代码上的时间更少，从而获得更流畅的初始用户体验。
分析javascript文件的使用率。找到"未使用"的地方，因为它在启动时没有被执行，但对于未来的某些功能来说仍然是必要的。这就是你可以通过代码拆分将其移到一个单独的包中的代码。

![image-20230602231524922](https://cdn.jsdelivr.net/gh/xxydrr/my_pic/img/image-20230602231524922.png)

next.js组件允许使用各种策略加载不太重要的第三方代码，

![image-20230602232255996](https://cdn.jsdelivr.net/gh/xxydrr/my_pic/img/image-20230602232255996.png)

### 避免大量的渲染更新

这个很好理解，大量渲染dom是昂贵并且会造成影响FID造成卡顿。所以保持得保持DOM变动粒度更小。就是组件化的思想，组件化无论是html还是css都能做到局部更新。

**合理使用`requestAnimationFrame()`**。`requestAnimationFrame()`事件循环的渲染阶段处理的，如果在这个步骤中做了太多的工作，渲染更新就会被延迟。使用requestAnimationFrame()做的任务都该是和渲染相关的任务，例如使用requestAnimationFrame更小粒度调度渲染相关的数据。

![image-20230602233319569](https://cdn.jsdelivr.net/gh/xxydrr/my_pic/img/image-20230602233319569.png)

## 总结

本文介绍了2023年核心网页指标的九大优化方法，主要涉及到最大内容绘制（LCP）、首次输入延迟（FID）和累积布局偏移（CLS）三个方面。

- LCP的优化建议包括使用标准html方式引入图片、设置图片资源的fetchpriority属性、使用CDN等。

- CLS的优化建议包括明确内容的大小、使用bfcache、使用transform等。

- FID的优化建议包括避免或分解长任务、避免不必要的JavaScript和避免大量的渲染更新等。

这些优化方法能够有效提高网页的加载性能、交互性和视觉稳定性，提供更良好的用户体验。

## 参考资料

[Our top Core Web Vitals recommendations for 2023](https://web.dev/top-cwv-2023/#avoid-large-rendering-updates)

[The 9 most effective Core Web Vitals opportunities of 2023](https://www.youtube.com/watch?v=mdB-J6BRReo&t=28s)
