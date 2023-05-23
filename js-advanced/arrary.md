# Array

```js
// Array.of()可以把一组参数转换为数组。这个方法用于替代在 ES6之前常用的 Array.prototype.slice.call(arguments)，一种异常笨拙的将 arguments 对象转换为数组的写法：
console.log(Array.of(1, 2, 3, 4)); // [1, 2, 3, 4]
console.log(Array.of(undefined)); // [undefined]

// Array.fill()
// const zeroes = [0, 0, 0, 0, 0];
// 用 5 填充整个数组
zeroes.fill(5);
console.log(zeroes); // [5, 5, 5, 5, 5]
zeroes.fill(0); // 重置
// 用 6 填充索引大于等于 3 的元素
zeroes.fill(6, 3);
console.log(zeroes); // [0, 0, 0, 6, 6]
zeroes.fill(0); // 重置
// 用 7 填充索引大于等于 1 且小于 3 的元素
zeroes.fill(7, 1, 3);
console.log(zeroes); // [0, 7, 7, 0, 0];
zeroes.fill(0); // 重置
// 用 8 填充索引大于等于 1 且小于 4 的元素
// (-4 + zeroes.length = 1)
// (-1 + zeroes.length = 4)
zeroes.fill(8, -4, -1);
console.log(zeroes); // [0, 8, 8, 8, 0];

// Array.copyWithin()
let ints
let = reset = () => ints = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9];
reset();
// 从 ints 中复制索引 0 开始的内容，插入到索引 5 开始的位置
// 在源索引或目标索引到达数组边界时停止
ints.copyWithin(5);
console.log(ints); // [0, 1, 2, 3, 4, 0, 1, 2, 3, 4]
reset();
// 从 ints 中复制索引 5 开始的内容，插入到索引 0 开始的位置
ints.copyWithin(0, 5);
console.log(ints); // [5, 6, 7, 8, 9, 5, 6, 7, 8, 9]
reset();
// 从 ints 中复制索引 0 开始到索引 3 结束的内容
// 插入到索引 4 开始的位置
ints.copyWithin(4, 0, 3);
alert(ints); // [0, 1, 2, 3, 0, 1, 2, 7, 8, 9]
reset();
// JavaScript 引擎在插值前会完整复制范围内的值
// 因此复制期间不存在重写的风险
ints.copyWithin(2, 0, 6);
alert(ints); // [0, 1, 0, 1, 2, 3, 4, 5, 8, 9]
reset();

// 如果数组中某一项是 null 或 undefined，则在 join()、toLocaleString()、
// toString()和 valueOf()返回的结果中会以空字符串表示。

let colors = new Array(); // 创建一个数组
let count = colors.push("red", "green"); // 推入两项
console.log(count); // 2
```

## 数组打平方法

### Array.prototype.flatten()

下面是如果没有这两个新方法要打平数组的一个示例实现:

```js
function flatten(sourceArray, flattenedArray = []) {
  for (const element of sourceArray) {
    if (Array.isArray(element)) {
      flatten(element, flattenedArray);
    } else {
      flattenedArray.push(element);
    }
  }
  return flattenedArray;
}
const arr = [[0], 1, 2, [3, [4, 5]], 6];
console.log(flatten(arr))
// [0, 1, 2, 3, 4, 5, 6]

```

这个例子在很多方面像一个树形数据结构:数组中每个元素都像一个子节点，非数组元素是叶节点。 因此，这个例子中的输入数组是一个高度为 2 有 7 个叶节点的树。打平这个数组本质上是对叶节点的按 序遍历。
有时候如果能指定打平到第几级嵌套是很有用的。比如下面这个例子，它重写了上面的版本，允许 指定要打平几级:

```js
function flatten(sourceArray, depth, flattenedArray = []) {
  for (const element of sourceArray) {
    if (Array.isArray(element) && depth > 0) {
       flatten(element, depth - 1, flattenedArray);
    } else {
      flattenedArray.push(element);
    }
  }
  return flattenedArray
}
const arr = [[0], 1, 2, [3, [4, 5]], 6];
console.log(flatten(arr, 1)); 
// [0, 1, 2, 3, [4, 5], 6]
```

为了解决上述问题，规范增加了 Array.prototype.flat()方法。该方法接收 depth 参数(默认
值为 1)，返回一个对要打平 Array 实例的浅复制副本。下面看几个例子:

```js
const arr = [[0], 1, 2, [3, [4, 5]], 6];
console.log(arr.flat(2)); // [0, 1, 2, 3, 4, 5, 6]
console.log(arr.flat()); // [0, 1, 2, 3, [4, 5], 6]
```

因为是执行浅复制，所以包含循环引用的数组在被打平时会从源数组复制值：

```js
const arr = [[0], 1, 2,[3,[4,5]], 6];
arr.push(arr); 
console.log(arr.flat());
// [0, 1, 2, 3, 4, 5, 6, [0], 1, 2, [3, [4, 5]], 6]
```

### Array.prototype.flatMap()

Array.prototype.flatMap()方法会在打平数组之前执行一次映射操作。在功能上，arr.flatMap(f) 与 arr.map(f).flat()等价;但 arr.flatMap()更高效，因为浏览器只需要执行一次遍历。
flatMap()的函数签名与 map()相同。下面是一个简单的例子:

```js
const arr = [[1], [3], [5]];
console.log(arr.map(([x]) => [x, x + 1]));
// [[1, 2], [3, 4], [5, 6]]
console.log(arr.flatMap(([x]) => [x, x + 1])); // [1, 2, 3, 4, 5, 6]
```

flatMap()在非数组对象的方法返回数组时特别有用，例如字符串的 split()方法。来看下面的例 子，该例子把一组输入字符串分割为单词，然后把这些单词拼接为一个单词数组:

```js
const arr = ['Lorem ipsum dolor sit amet,', 'consectetur adipiscing elit.'];
console.log(arr.flatMap((x) => x.split(/[\W+]/)));
// ["Lorem", "ipsum", "dolor", "sit", "amet", "", "consectetur", "adipiscing", "elit", ""]
```
