# Form

## 表单字段的公共事件

```html
<form>
    <label for="myInput">Enter only numbers:</label>
    <input type="text" id="myInput" name="myInput">
</form>

  <script>
let textbox = document.forms[0].elements[0];
textbox.addEventListener("focus", (event) => {
  let target = event.target;
  if (target.style.backgroundColor != "red") {
    target.style.backgroundColor = "yellow";
  }
});
textbox.addEventListener("blur", (event) => {
  let target = event.target;
  target.style.backgroundColor = /[^\d]/.test(target.value) ? "red" : "";
});
textbox.addEventListener("change", (event) => {
  let target = event.target;
  target.style.backgroundColor = /[^\d]/.test(target.value) ? "red" : "";
});
</script>

```

这里的 onfocus 事件处理程序会把文本框的背景改为黄色，更清楚地表明它是当前活动字段。 onblur 和 onchange 事件处理程序会在发现非数值字符时把背景改为红色。为测试非数值字符，这里 使用了一个简单的正则表达式来检测文本框的 value。这个功能必须同时在 onblur 和 onchange 事件 处理程序上实现，以确保无论文本框是否改变都能执行验证。

## 处理剪贴板

剪贴板相关的 6 个事件。

- beforecopy:复制操作发生前触发。
- copy:复制操作发生时触发。
- beforecut:剪切操作发生前触发。
- cut:剪切操作发生时触发。
- beforepaste:粘贴操作发生前触发。
- paste:粘贴操作发生时触发

```js
function getClipboardText(event) {
  let clipboardData = (event.clipboardData || window.clipboardData);
  return clipboardData.getData("text");
}
function setClipboardText(event, value) {
  if (event.clipboardData) {
    return event.clipboardData.setData("text/plain", value);
  } else if (window.clipboardData) { //兼容IE
    return window.clipboardData.setData("text", value);
  }
}
// onpaste 事件处理程序确保只有数字才能粘贴到文本框中
textbox.addEventListener("paste", (event) => {
  let text = getClipboardText(event);
  if (!/^\d*$/.test(text)) {
    event.preventDefault();
  }
});

```

## 富文本编辑 使用contenteditable

```html
<div class="editable" id="richedit" contenteditable></div>
<script>
  // 通过设置 contentEditable 属性，也可以随时切换元素的可编辑状态:
  let div = document.getElementById("richedit");
  richedit.contentEditable = "true";
</script>
</body>
```

contentEditable 属性有 3 个可能的值:"true"表示开启，"false"表示关闭，"inherit"表示 继承父元素的设置(因为在 contenteditable 元素内部会创建和删除元素)。IE、Firefox、Chrome、 Safari 和 Opera 及所有主流移动浏览器都支持 contentEditable 属性。
