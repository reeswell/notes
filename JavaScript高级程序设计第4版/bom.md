# BOM

## 窗口大小

### 偏移尺寸

第一组属性涉及偏移尺寸(offset dimensions)，包含元素在屏幕上占用的所有视觉空间。元素在页 面上的视觉空间由其高度和宽度决定，包括所有内边距、滚动条和边框(但不包含外边距)。以下 4 个 属性用于取得元素的偏移尺寸。

- offsetHeight，元素在垂直方向上占用的像素尺寸，包括它的高度、水平滚动条高度(如果可 见)和上、下边框的高度。 17
- offsetLeft，元素左边框外侧距离包含元素左边框内侧的像素数。
- offsetTop，元素上边框外侧距离包含元素上边框内侧的像素数。
- offsetWidth，元素在水平方向上占用的像素尺寸，包括它的宽度、垂直滚动条宽度(如果可
见)和左、右边框的宽度。 其中，offsetLeft和offsetTop是相对于包含元素的，包含元素保存在offsetParent属性中。 19
offsetParent 不一定是 parentNode。比如，`<td>`元素的 offsetParent 是作为其祖先的`<table>` 元素，因为`<table>`是节点层级中第一个提供尺寸的元素。图 16-1 展示了这些属性代表的不同尺寸。

![image-20230515012110926](https://cdn.jsdelivr.net/gh/xxydrr/my_pic/img/image-20230515012110926.png)

### 客户端尺寸

元素的客户端尺寸(client dimensions)包含元素内容及其内边距所占用的空间。客户端尺寸只有两 个相关属性:clientWidth 和 clientHeight。其中，clientWidth 是内容区宽度加左、右内边距宽 度，clientHeight 是内容区高度加上、下内边距高度。图 16-2 形象地展示了这两个属性。

![image-20230515012020082](/Users/chenruirong/Library/Application Support/typora-user-images/image-20230515012020082.png)

客户端尺寸实际上就是元素内部的空间，因此不包含滚动条占用的空间。这两个属性最常用于确定 浏览器视口尺寸，即检测document.documentElement的clientWidth和clientHeight。这两个 属性表示视口(`<html>`或`<body>`元素)的尺寸。

**注意** 与偏移尺寸一样，客户端尺寸也是只读的，而且每次访问都会重新计算。

### 滚动尺寸

最后一组尺寸是滚动尺寸(scroll dimensions)，提供了元素内容滚动距离的信息。有些元素，比如 `<html>`无须任何代码就可以自动滚动，而其他元素则需要使用 CSS 的 overflow 属性令其滚动。滚动 尺寸相关的属性有如下 4 个。

- scrollHeight，没有滚动条出现时，元素内容的总高度。
- scrollLeft，内容区左侧隐藏的像素数，设置这个属性可以改变元素的滚动位置。
- scrollTop，内容区顶部隐藏的像素数，设置这个属性可以改变元素的滚动位置。
- scrollWidth，没有滚动条出现时，元素内容的总宽度。
 图 16-3 展示了这些属性的含义。

![image-20230515012613815](https://cdn.jsdelivr.net/gh/xxydrr/my_pic/img/image-20230515012613815.png)

下面这个函数检测元素是不是位于顶部，如果不是则把它滚动回顶部:

```js
function scrollToTop(element) {
  if (element.scrollTop != 0) {
    element.scrollTop = 0;
  }
}
```

这个函数使用 scrollTop 获取并设置值。

### 确定元素尺寸

浏览器在每个元素上都暴露了 getBoundingClientRect()方法，返回一个 DOMRect 对象，包含 6 个属性:left、top、right、bottom、height 和 width。这些属性给出了元素在页面中相对于视 口的位置。图 16-41展示了这些属性的含义。

![image-20230515012957679](https://cdn.jsdelivr.net/gh/xxydrr/my_pic/img/image-20230515012957679.png)

### 客户端坐标

可以通过下面的方式获取鼠标事件的客户端坐标:

```js
let div = document.getElementById("myDiv");
div.addEventListener("click", (event) => {
  console.log(`Client coordinates: ${event.clientX}, ${event.clientY}`);
});
```

![image-20230515081628048](https://cdn.jsdelivr.net/gh/xxydrr/my_pic/img/image-20230515081628048.png)

### 页面坐标

```js
let div = document.getElementById("myDiv");
div.addEventListener("click", (event) => {
  console.log(`Page coordinates: ${event.pageX}, ${event.pageY}`);
});
```

在页面没有滚动时，pageX 和 pageY 与 clientX 和 clientY 的值相同。

IE8及更早版本没有在event对象上暴露页面坐标。不过，可以通过客户端坐标和滚动信息计算出 来。滚动信息可以从 document.body(混杂模式)或 document.documentElement(标准模式)的 scrollLeft 和 scrollTop 属性获取。计算过程如下所示:

```js
let div = document.getElementById("myDiv");
div.addEventListener("click", (event) => {
  let pageX = event.pageX,
    pageY = event.pageY;
  if (pageX === undefined) {
    pageX = event.clientX + (document.body.scrollLeft ||
      document.documentElement.scrollLeft);
  }
  if (pageY === undefined) {
    pageY = event.clientY + (document.body.scrollTop ||
      document.documentElement.scrollTop);
  }
  console.log(`Page coordinates: ${pageX}, ${pageY}`);
});

```

### 屏幕坐标

可以像下面这样获取鼠标事件的屏幕坐标:

```js
let div = document.getElementById("myDiv");
div.addEventListener("click", (event) => {
  console.log(`Screen coordinates: ${event.screenX}, ${event.screenY}`);
});
```

![image-20230515082025158](https://cdn.jsdelivr.net/gh/xxydrr/my_pic/img/image-20230515082025158.png)

在浏览器中，窗口大小有不同的含义，包括以下几个概念：

内容区大小（Viewport Size）：指浏览器窗口内网页的可见区域大小，不包括浏览器的工具栏、菜单栏、边框等。

窗口大小（Window Size）：指整个浏览器窗口的大小，(outerWidth、outerHeight)包括内容区、工具栏、菜单栏、边框等。

文档大小（Document Size）：指网页文档的实际大小，包括了所有内容，无论是否在可视区域内。

工具栏（Toolbar）：通常位于浏览器窗口的顶部，包含一些常用的工具按钮，如前进、后退、刷新、主页等等。工具栏可以通过浏览器的设置进行开启或关闭。

菜单栏（Menu Bar）：通常位于工具栏下面，包含一些菜单，如文件、编辑、查看、收藏夹等等。菜单栏可以通过浏览器的设置进行开启或关闭。

边框（Border）：是浏览器窗口的边框，用来分隔窗口和操作系统的其他部分。边框包括窗口的标题栏、边框线和窗口控制按钮等等。边框的大小和样式可以通过操作系统的设置进行调整。

获取内容区大小：

```js
const viewportWidth = document.documentElement.clientWidth;
const viewportHeight = document.documentElement.clientHeight;
```

获取窗口大小：

```js
const windowWidth = window.outerWidth;
const windowHeight = window.outerHeight;
// innerWidth、innerHeight不包含浏览器边框和工具栏
const pageWidth = window.innerWidth;
const pageHeight = window.innerHeight;
```

获取文档大小：

```js

const documentWidth = document.documentElement.scrollWidth;
const documentHeight = document.documentElement.scrollHeight;
```

需要注意的是，由于浏览器的不同以及浏览器设置的不同，这些值可能会有所不同，使用时需要注意兼容性和浏览器差异。

## 视口位置

```js
// 相对于当前视口向下滚动 100 像素
 window.scrollBy(0, 100);
// 相对于当前视口向右滚动 40 像素 
window.scrollBy(40, 0);
// 滚动到页面左上角 
window.scrollTo(0, 0);

// 滚动到距离屏幕左边及顶边各 100 像素的位置 window.scrollTo(100, 100);
```

这几个方法也都接收一个 ScrollToOptions 字典，除了提供偏移值，还可以通过 behavior 属性 告诉浏览器是否平滑滚动。

```js
 // 正常滚动
window.scrollTo({
  left: 100,
  top: 100,
  behavior: 'auto'
});
// 平滑滚动
window.scrollTo({ 4
  left: 100,
  top: 100,
  behavior: 'smooth'
});
```

## location对象

### 查询字符串

函数解析了查询字符串 location.search

```js
let getQueryStringArgs = function () {
  // 取得没有开头问号的查询字符串
  let qs = (location.search.length > 0 ? location.search.substring(1) : ""),
  // 保存数据的对象 args = {};
  // 把每个参数添加到 args 对象
  for (let item of qs.split("&").map(kv => kv.split("="))) {
    let name = decodeURIComponent(item[0]),
      value = decodeURIComponent(item[1]);
    if (name.length) {
      args[name] = value;
    }
  }
  return args;
}

let args = getQueryStringArgs(); 
// 假设查询字符串为?q=javascript&num=10
alert(args["q"]);    // "javascript"
alert(args["num"]);  // "10"
```

URLSearchParams

URLSearchParams 提供了一组标准 API 方法，通过它们可以检查和修改查询字符串。给 URLSearchParams 构造函数传入一个查询字符串，就可以创建一个实例。这个实例上暴露了 get()、 set()和 delete ()等方法，可以对查询字符串执行相应操作。下面来看一个例子:

```js
let qs = "?q=javascript&num=10";
let searchParams = new URLSearchParams(qs);
alert(searchParams.toString());  // " q=javascript&num=10"
searchParams.has("num"); // true
searchParams.get("num"); // 10
searchParams.set("page", "3");
alert(searchParams.toString());  // " q=javascript&num=10&page=3"

searchParams.delete("q");
alert(searchParams.toString());  // " num=10&page=3"

let searchParams = new URLSearchParams(qs);
for (let param of searchParams) {
  console.log(param);
}
// ["q", "javascript"]  
// ["num", "10"]
```

### 操作地址

可以通过修改 location 对象修改浏览器的地址。首先，最常见的是使用 assign()方法并传入一 个 URL，如下所示:

```js
location.assign("http://www.wrox.com");
```

这行代码会立即启动导航到新 URL 的操作，同时在浏览器历史记录中增加一条记录。如果给 location.href 或 window.location 设置一个 URL，也会以同一个 URL 值调用 assign()方法。比 如，下面两行代码都会执行与显式调用 assign()一样的操作:

```js
window.location = "http://www.wrox.com";
location.href = "http://www.wrox.com";
```

在这 3 种修改浏览器地址的方法中，设置 location.href 是最常见的。
修改 location 对象的属性也会修改当前加载的页面。其中，hash、search、hostname、pathname 和 port 属性被设置为新值之后都会修改当前 URL，如下面的例子所示:

```js
// 假设当前URL为<http://www.wrox.com/WileyCDA/>
// 把URL修改为<http://www.wrox.com/WileyCDA/#section1>
location.hash = "#section1";
// 把URL修改为<http://www.wrox.com/WileyCDA/?q=javascript>
location.search = "?q=javascript";
// 把URL修改为<http://www.somewhere.com/WileyCDA/>
location.hostname = "www.somewhere.com";
// 把URL修改为<http://www.somewhere.com/mydir/>
location.pathname = "mydir";
// 把URL修改为<http://www.somewhere.com:8080/WileyCDA/>
location.port = 8080;
```

除了 hash 之外，只要修改 location 的一个属性，就会导致页面重新加载新 URL。
**注意 修改hash的值会在浏览器历史中增加一条新记录。在早期的IE中，点击“后退” 和“前进”按钮不会更新 hash 属性，只有点击包含散列的 URL 才会更新 hash 的值。**

在以前面提到的方式修改 URL 之后，浏览器历史记录中就会增加相应的记录。当用户单击“后退” 按钮时，就会导航到前一个页面。如果不希望增加历史记录，可以使用 replace()方法。这个方法接 收一个 URL 参数，但重新加载后不会增加历史记录。调用 replace()之后，用户不能回到前一页。比 如下面的例子:

```html
<!DOCTYPE html>
<html>
<head>
  <title>You won't be able to get back here</title>
</head>
<body>
  <p>Enjoy this page for a second, because you won't be coming back here.</p>
  <script>
  setTimeout(() => location.replace("http://www.wrox.com/"), 1000); </script>
</body>
</html>
```

浏览器加载这个页面 1 秒之后会重定向到 `www.wrox.com`。此时，“后退”按钮是禁用状态，即不能 返回这个示例页面，除非手动输入完整的 URL。

最后一个修改地址的方法是 reload()，它能重新加载当前显示的页面。调用 reload()而不传参 数，页面会以最有效的方式重新加载。也就是说，如果页面自上次请求以来没有修改过，浏览器可能会 从缓存中加载页面。如果想强制从服务器重新加载，可以像下面这样给 reload()传个 true:

```js
location.reload(); // 重新加载，可能是从缓存加载 location.reload(true); // 重新加载，从服务器加载
```

 脚本中位于 reload()调用之后的代码可能执行也可能不执行，这取决于网络延迟和系统资源等因 素。为此，最好把 reload()作为最后一行代码。

## history对象

### 导航

```js
// 后退一页
history.go(-1);
// 前进一页
history.go(1);
// 前进两页
history.go(2);

// 导航到最近的 wrox.com 页面 
history.go("wrox.com");
// 导航到最近的 nczonline.net 页面 
history.go("nczonline.net");
// go()有两个简写方法:back()和 forward()。顾名思义，这两个方法模拟了浏览器的后退按钮和 前进按钮:
// 后退一页 history.back();
// 前进一页 history.forward();
```

history 对象还有一个 length 属性，表示历史记录中有多个条目。这个属性反映了历史记录的数 量，包括可以前进和后退的页面。对于窗口或标签页中加载的第一个页面，history.length 等于 1。 通过以下方法测试这个值，可以确定用户浏览器的起点是不是你的页面:

```js
if (history.length == 1){ // 这是用户窗口中的第一个页面
}
```

history 对象通常被用于创建“后退”和“前进”按钮，以及确定页面是不是用户历史记录中的第
一条记录。

### 历史状态管理

hashchange 会在页面 URL 的散列变化时被触发，开发者可以在此时执行某些操作。而状态管理 API 则可以让开发者改变浏览器 URL 而不会加载新页面。为此，可以使用 history.pushState()方 法。这个方法接收 3 个参数:一个 state 对象、一个新状态的标题和一个(可选的)相对 URL。例如:

```js
let stateObject = {foo:"bar"};
history.pushState(stateObject, "My title", "baz.html");
```

pushState()方法执行后，状态信息就会被推到历史记录中，浏览器地址栏也会改变以反映新的相 对 URL。除了这些变化之外，即使 location.href 返回的是地址栏中的内容，浏览器页不会向服务器发送请求。第二个参数并未被当前实现所使用，因此既可以传一个空字符串也可以传一个短标题。第一 个参数应该包含正确初始化页面状态所必需的信息。为防止滥用，这个状态的对象大小是有限制的，通 常在 500KB~1MB 以内。
因为 pushState()会创建新的历史记录，所以也会相应地启用“后退”按钮。此时单击“后退” 按钮，就会触发 window 对象上的 popstate 事件。popstate 事件的事件对象有一个 state 属性，其 中包含通过 pushState()第一个参数传入的 state 对象:

```js
window.addEventListener("popstate", (event) => { let state = event.state;
if (state) { // 第一个页面加载时状态是null
    processState(state);
  }
});
```

基于这个状态，应该把页面重置为状态对象所表示的状态(因为浏览器不会自动为你做这些)。记 住，页面初次加载时没有状态。因此点击“后退”按钮直到返回最初页面时，event.state 会为 null。
可以通过 history.state 获取当前的状态对象，也可以使用 replaceState()并传入与
pushState()同样的前两个参数来更新状态。更新状态不会创建新历史记录，只会覆盖当前状态:

```js
history.replaceState({newFoo: "newBar"}, "New title");
```

传给 pushState()和 replaceState()的 state 对象应该只包含可以被序列化的信息。因此， DOM 元素之类并不适合放到状态对象里保存。

**注意** 使用HTML5状态管理时，要确保通过pushState()创建的每个“假”URL背后 都对应着服务器上一个真实的物理 URL。否则，单击“刷新”按钮会导致 404 错误。所有 单页应用程序(SPA，Single Page Application)框架都必须通过服务器或客户端的某些配 置解决这个问题。
