# JavaScript API

## postMessage()

postMessage()方法接收 3 个参数:消息、表示目标接收源的字符串(现在为任何结构的数据)和可选的可传输对象的数组(只 与工作线程相关)。第二个参数对于安全非常重要，其可以限制浏览器交付数据的目标。下面来看一个 例子:

```js
let iframeWindow = document.getElementById("myframe").contentWindow; 
iframeWindow.postMessage("A secret", "<http://www.wrox.com>");
```

传给 onmessage 事件处理程序的 event对象包含以下 3 方面重要信息。

- data:作为第一个参数传递给 postMessage()的字符串数据。
- origin:发送消息的文档源，例如"<http://www.wrox.com"。>
- source:发送消息的文档中 window 对象的代理。这个代理对象主要用于在发送上一条消息的
窗口中执行 postMessage()方法。如果发送窗口有相同的源，那么这个对象应该就是 window
对象。

```js
window.addEventListener("message", (event) => {  
  // 确保来自预期发送者
  if (event.origin == "http://www.wrox.com") {
  // 对数据进行一些处理
  processMessage(event.data);
  // 可选:向来源窗口发送一条消息 
  event.source.postMessage("Received!", "http://p2p.wrox.com");
  }
 });
```

postMessage()的第一个参数的最初实现始终是一个字符串。后来， 第一个参数改为允许任何结构的数据传入，不过并非所有浏览器都实现了这个改变。为此，最好就是只 通过 postMessage()发送字符串。如果需要传递结构化数据，那么最好先对该数据调用 JSON.stringify()，通过 postMessage()传过去之后，再在 onmessage 事件处理程序中调用 JSON.parse()。

## File API 与 Blob API

### File类型

File API 仍然以表单中的文件输入字段为基础，但是增加了直接访问文件信息的能力。HTML5 在 DOM 上为文件输入元素添加了 files 集合。当用户在文件字段中选择一个或多个文件时，这个 files 集合中会包含一组 File 对象，表示被选中的文件。每个 File 对象都有一些只读属性。

- name:本地系统中的文件名。
- size:以字节计的文件大小。
- type:包含文件 MIME 类型的字符串。
- lastModifiedDate:表示文件最后修改时间的字符串。这个属性只有 Chome 实现了。 例如，通过监听 change 事件然后遍历 files 集合可以取得每个选中文件的信息:

```html
<form>
  <label for="file-input">Select one or more files:</label>
  <input type="file" id="file-input" name="files[]" multiple>
</form>

<ul id="files-list"></ul>

<script>
  let fileInput = document.getElementById("file-input");

  fileInput.addEventListener("change", (event) => {
    let files = event.target.files,
      i = 0,
      len = files.length;
    while (i < len) {
      const f = files[i];
      const li = document.createElement("li");
      li.textContent = `${f.name} (${f.type}, ${f.size} bytes)`;
      filesList.appendChild(li);
      i++;
    }
  });
</script>

```

这个例子简单地在控制台输出了每个文件的信息。仅就这个能力而言，已经可以说是 Web 应用向 前迈进的一大步了。不过，File API 还提供了 FileReader 类型，让我们可以实际从文件中读取数据。

### FileReader类型

FileReader 类型表示一种异步文件读取机制。可以把 FileReader 想象成类似于 XMLHttpRequest， 只不过是用于从文件系统读取文件，而不是从服务器读取数据。FileReader 类型提供了几个读取文件 数据的方法。

- readAsText(file, encoding):从文件中读取纯文本内容并保存在 result 属性中。第二个 参数表示编码，是可选的。
- readAsDataURL(file):读取文件并将内容的数据 URI 保存在 result 属性中。
- readAsBinaryString(file):读取文件并将每个字符的二进制数据保存在 result 属性中。
- readAsArrayBuffer(file):读取文件并将文件内容以 ArrayBuffer 形式保存在 result 属性。
这些读取数据的方法为处理文件数据提供了极大的灵活性。例如，为了向用户显示图片，可以将图
片读取为数据 URI，而为了解析文件内容，可以将文件读取为文本。
因为这些读取方法是异步的，所以每个 FileReader 会发布几个事件，其中 3 个最有用的事件是
progress、error 和 load，分别表示还有更多数据、发生了错误和读取完成。
progress 事件每 50 毫秒就会触发一次，其与 XHR 的 progress 事件具有相同的信息:
lengthComputable、loaded和total。此外，在progress事件中可以读取FileReader的result
属性，即使其中尚未包含全部数据。
error 事件会在由于某种原因无法读取文件时触发。触发 error 事件时，FileReader 的 error
属性会包含错误信息。这个属性是一个对象，只包含一个属性:code。这个错误码的值可能是 1(未找 到文件)、2(安全错误)、3(读取被中断)、4(文件不可读)或 5(编码错误)。
load 事件会在文件成功加载后触发。如果 error 事件被触发，则不会再触发 load 事件。下面的 例子演示了所有这 3 个事件:

```html
<form>
  <label for="file-input">Select one or more files:</label>
  <input type="file" id="file-input" name="files[]" multiple>
</form>
output: <div id="output"></div>
progress: <div id="progress"></div>

<script>
  let fileInput = document.getElementById("file-input");

  fileInput.addEventListener("change", (event) => {
    let info = "",
      output = document.getElementById("output"),
      progress = document.getElementById("progress"),
      files = event.target.files,
      type = "default",
      reader = new FileReader();
    if (/image/.test(files[0].type)) {
      reader.readAsDataURL(files[0]);
      type = "image";
    } else {
      reader.readAsText(files[0]);
      type = "text";
    }
    reader.onerror = function () {
      output.innerHTML = "Could not read file, error code is " +
        reader.error.code;
    };
    reader.onprogress = function (event) {
      if (event.lengthComputable) {
        progress.innerHTML = `${event.loaded}/${event.total}`;
      }
    };
    reader.onload = function () {
      let html = "";
      switch (type) {
        case "image":
          html = `<img src="${reader.result}">`;
          break;
        case "text":
          html = reader.result;
          break;
      }
      output.innerHTML = html;
    };
  });

</script>
```

以上代码从表单字段中读取一个文件，并将其内容显示在了网页上。如果文件的 MIME 类型表示它 是一个图片，那么就将其读取后保存为数据 URI，在 load 事件触发时将数据 URI 作为图片插入页面中。 如果文件不是图片，则读取后将其保存为文本并原样输出到网页上。progress 事件用于跟踪和显示读 取文件的进度，而 error 事件用于监控错误。
如果想提前结束文件读取，则可以在过程中调用 abort()方法，从而触发 abort 事件。在 load、 error 和 abort 事件触发后，还会触发 loadend 事件。loadend 事件表示在上述 3 种情况下，所有读 取操作都已经结束。readAsText()和 readAsDataURL()方法已经得到了所有主流浏览器支持。

### FileReaderSync类型

FileReaderSync 类型就是 FileReader 的同步版本。这个类型拥有与 FileReader 相同的方法，只有在整个文件都加载到内存之后才会继续执行。FileReaderSync 只在工作线程中可用， 因为如果读取整个文件耗时太长则会影响全局。
假设通过 postMessage()向工作线程发送了一个 File 对象。以下代码会让工作线程同步将文件 读取到内存中，然后将文件的数据 URL 发回来:

```js
// worker.js
self.onmessage = (messageEvent) => {
const syncReader = new FileReaderSync(); 
console.log(syncReader); // FileReaderSync {}
// 读取文件时阻塞工作线程
const result = syncReader.readAsDataURL(messageEvent.data);
// PDF 文件的示例响应
console.log(result); // data:application/pdf;base64,JVBERi0xLjQK...
// 把URL发回去
  self.postMessage(result);
};
```

### Blob与部分读取

某些情况下，可能需要读取部分文件而不是整个文件。为此，File 对象提供了一个名为 slice() 的方法。slice()方法接收两个参数:起始字节和要读取的字节数。这个方法返回一个 Blob 的实例， 而 Blob 实际上是 File 的超类。
blob 表示二进制大对象(binary larget object)，是 JavaScript 对不可修改二进制数据的封装类型。包 含字符串的数组、ArrayBuffers、ArrayBufferViews，甚至其他 Blob 都可以用来创建 blob。Blob 构造函数可以接收一个 options 参数，并在其中指定 MIME 类型:

```js
console.log(new Blob(['foo']));
// Blob {size: 3, type: ""}
console.log(new Blob(['{"a": "b"}'], { type: 'application/json' }));
// {size: 10, type: "application/json"}
console.log(new Blob(['<p>Foo</p>', '<p>Bar</p>'], { type: 'text/html' })); 
// {size: 20, type: "text/html"}
```

Blob 对象有一个 size 属性和一个 type 属性，还有一个 slice()方法用于进一步切分数据。另 外也可以使用 FileReader 从 Blob 中读取数据。下面的例子只会读取文件的前 32 字节:

```js
let fileInput = document.getElementById("file-input");

fileInput.addEventListener("change", (event) => {
  let info = "",
    output = document.getElementById("output"),
    progress = document.getElementById("progress"),
    files = event.target.files,
    reader = new FileReader()

  const blob = new Blob(files, { type: "text/plain" });
  const slice = blob.slice(0, 32);
  if (slice) {
    reader.readAsText(slice);
    reader.onerror = function () {
      output.innerHTML = "Could not read file, error code is " +
        reader.error.code;
    };
    reader.onload = function () {
      output.innerHTML = reader.result;
    };
  } else {
    console.log("Your browser doesn't support slice()."); 22
  }
});
```

### 对象URL与Blob

对象URL有时候也称作Blob URL，是指引用存储在File或Blob中数据的 URL。对象URL的优点是不用把文件内容读取到JavaScript也可以使用文件。只要在适当位置提供对象URL即可。要创建对
象URL，可以使用 window.URL.createObjectURL()方法并传入File或Blob对象。这个函数返回的值是一个指向内存中地址的字符串。因为这个字符串是URL，所以可以在DOM中直接使用。例如， 以下代码使用对象URL在页面中显示了一张图片:

```js
fileInput.addEventListener("change", (event) => {
  let info = "",
    output = document.getElementById("output"),
    progress = document.getElementById("progress"),
    files = event.target.files,
    reader = new FileReader(),
    url = window.URL.createObjectURL(files[0]);
  if (url) {
    if (/image/.test(files[0].type)) {
      output.innerHTML = `<img src="${url}">`;
    } else {
      output.innerHTML = "Not an image.";
    }
  } else {
    output.innerHTML = "Your browser doesn't support object URLs.";
  }
});
```

如果把对象 URL 直接放到`<img>`标签，就不需要把数据先读到 JavaScript 中了。`<img>`标签可以直 接从相应的内存位置把数据读取到页面上。
使用完数据之后，最好能释放与之关联的内存。只要对象 URL 在使用中，就不能释放内存。如果 想表明不再使用某个对象 URL，则可以把它传给 window.URL.revokeObjectURL()。页面卸载时， 所有对象 URL 占用的内存都会被释放。不过，最好在不使用时就立即释放内存，以便尽可能保持页面 占用最少资源。

### 读取拖放文件

组合使用 HTML5 拖放 API 与 File API 可以创建读取文件信息的有趣功能。在页面上创建放置目标 后，可以从桌面上把文件拖动并放到放置目标。这样会像拖放图片或链接一样触发 drop 事件。被放置 的文件可以通过事件的 event.dataTransfer.files 属性读到，这个属性保存着一组 File 对象，就 像文本输入字段一样。
  下面的例子会把拖放到页面放置目标上的文件信息打印出来:

```js
let droptarget = document.getElementById("file-input");
function handleEvent(event) {
  let info = "",
    output = document.getElementById("output"),
    files, i, len;
  event.preventDefault();
  if (event.type == "drop") {
    files = event.dataTransfer.files;
    i = 0;
    len = files.length;
    while (i < len) {
      info += `${files[i].name} (${files[i].type}, ${files[i].size} bytes)<br>`; i++;
    }
    output.innerHTML = info;
  }
}
droptarget.addEventListener("dragenter", handleEvent); 
droptarget.addEventListener("dragover", handleEvent);
droptarget.addEventListener("drop", handleEvent);
```

与后面要介绍的拖放的例子一样，必须取消 dragenter、dragover 和 drop 的默认行为。在 drop 事件处理程序中，可以通过 event.dataTransfer.files 读到文件，此时可以获取文件的相关 信息。

### 自定义媒体播放器

```html
<div class="mediaplayer">
  <div class="video">
    <video id="player" src="./media/movice.mp4" poster="./media/movice.png" width="300" height="200">
      Video player not available.
    </video>
  </div>
  <div class="controls">
    <input type="button" value="Play" id="video-btn">
    <span id="curtime">0</span>/<span id="duration">0</span>
  </div>
</div>

<script>
  // 取得元素的引用
  let player = document.getElementById("player"),
    btn = document.getElementById("video-btn"),
    curtime = document.getElementById("curtime"),
    duration = document.getElementById("duration");
  // 更新时长
  duration.innerHTML = player.duration;
  console.log(player);
  // 为按钮添加事件处理程序 
  btn.addEventListener("click", (event) => {
    if (player.paused) {
      player.play();
      btn.value = "Pause";
    } else {
      player.pause();
      btn.value = "Play";
    }
  });
  // 周期性更新当前时间 
  setInterval(() => {
    curtime.innerHTML = player.currentTime;
  }, 250);

</script>
```

## HTML 模板

### DocumentFragment

```html
<template id="foo">
  #document-fragment
  <p>I'm inside a template!</p>
</template>
<script>
const fragment = document.querySelector('#foo').content;
console.log(document.querySelector('p')); // null
console.log(fragment.querySelector('p')); // <p>...<p>
```

DocumentFragment 也是批量向 HTML 中添加元素的高效工具。比如，我们想以最快的方式给某 个 HTML 元素添加多个子元素。如果连续调用 document.appendChild()，则不仅费事，还会导致多 次布局重排。而使用 DocumentFragment 可以一次性添加所有子节点，最多只会有一次布局重排:

```js
//开始状态：
// <div id="foo"></div>
//期待的最终状态：
// <div id="foo">
//   <p></p>
//   <p></p>
//   <p></p>
// </div>
//也可以使用document.createDocumentFragment()
const fragment = new DocumentFragment();

const foo = document.querySelector('#foo');

//为DocumentFragment添加子元素不会导致布局重排
fragment.appendChild(document.createElement('p'));
fragment.appendChild(document.createElement('p'));
fragment.appendChild(document.createElement('p'));

console.log(fragment.children.length); // 3

foo.appendChild(fragment);

console.log(fragment.children.length); // 0

console.log(document.body.innerHTML);
// <div id="foo">
//   <p></p>
//   <p></p>
//   <p></p>
// </div>

```
