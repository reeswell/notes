# JSON

JSON 语法支持表示 3 种类型的值。

- 简单值:字符串、数值、布尔值和 null 可以在 JSON 中出现，就像在 JavaScript 中一样。特殊值 undefined 不可以。
- 对象:第一种复杂数据类型，对象表示有序键/值对。每个值可以是简单值，也可以是复杂类型。 25
- 数组:第二种复杂数据类型，数组表示可以通过数值索引访问的值的有序列表。数组的值可以是任意类型，包括简单值、对象，甚至其他数组。
JSON 没有变量、函数或对象实例的概念。JSON 的所有记号都只为表示结构化数据，虽然它借用了JavaScript 的语法，但是千万不要把它跟 JavaScript 语言混淆。

## JSON.stringify()

JSON.stringify()会输出不包含空格或缩进的 JSON 字符串,值为 undefined 的任何属性也会被跳过

### 过滤结果

实际上，JSON.stringify()方法除了要序列化的对象，还可以接收两个参数。这两个参数可以用 于指定其他序列化 JavaScript 对象的方式。第一个参数是过滤器，可以是数组或函数;第二个参数是用 于缩进结果 JSON 字符串的选项。单独或组合使用这些参数可以更好地控制 JSON 序列化。

```js
let book = {
    title: "Professional JavaScript",
    authors: [
      "Nicholas C. Zakas",
      "Matt Frisbie"
    ],
    name: undefined,
    edition: 4,
    year: 2017
};
let jsonText = JSON.stringify(book,["title", "edition"]);
```

在这个例子中，JSON.stringify()方法的第二个参数是一个包含两个字符串的数组:"title" 和"edition"。它们对应着要序列化的对象中的属性，因此结果 JSON 字符串中只会包含这两个属性:

```json
{"title":"Professional JavaScript","edition":4}
```

如果第二个参数是一个函数，则行为又有不同。提供的函数接收两个参数:属性名(key)和属性 值(value)。可以根据这个 key 决定要对相应属性执行什么操作。这个 key 始终是字符串，只是在值 不属于某个键/值对时会是空字符串。

```js
let jsonText = JSON.stringify(book, (key, value) => {
  switch (key) {
    case "authors":
      return value.join(",")
    case "year":
      return 5000;
    case "edition":
      return undefined;
    default:
      return value;
  }
});
// {"title":"Professional JavaScript","authors":"Nicholas C. Zakas,Matt Frisbie","year":5000}
```

### 字符串缩进

```js
let jsonText = JSON.stringify(book, null, 4);
```

这样得到的 jsonText 格式如下:

```json
{
    "title": "Professional JavaScript",
    "authors": [
        "Nicholas C. Zakas",
        "Matt Frisbie"
    ],
    "edition": 4,
    "year": 2017
}
```

注意，除了缩进，JSON.stringify()方法还为方便阅读插入了换行符。这个行为对于所有有效的缩进参数都会发生。(只缩进不换行也没什么用。)最大缩进值为 10，大于 10 的值会自动设置为 10。
如果缩进参数是一个字符串而非数值，那么 JSON 字符串中就会使用这个字符串而不是空格来缩进。
使用字符串，也可以将缩进字符设置为 Tab 或任意字符，如两个连字符:

```js
let jsonText = JSON.stringify(book, null, "--" );
```

这样，jsonText 的值会变成如下格式:

```json
{
--"title": "Professional JavaScript",
--"authors": [
----"Nicholas C. Zakas",
----"Matt Frisbie"
--],
--"edition": 4,
--"year": 2017
}
```

使用字符串时同样有 10 个字符的长度限制。如果字符串长度超过 10，则会在第 10 个字符处截断。

## JSON.parse()

JSON.parse()方法也可以接收一个额外的参数，这个函数会针对每个键/值对都调用一次。为区别 于传给 JSON.stringify()的起过滤作用的替代函数(replacer)，这个函数被称为还原函数(reviver)。 实际上它们的格式完全一样，即还原函数也接收两个参数，属性名(key)和属性值(value)，另外也 需要返回值。
如果还原函数返回 undefined，则结果中就会删除相应的键。如果返回了其他任何值，则该值就 会成为相应键的值插入到结果中。还原函数经常被用于把日期字符串转换为 Date 对象。例如:

```js
let book = {
  title: "Professional JavaScript",
  authors: [
    "Nicholas C. Zakas",
    "Matt Frisbie"
  ],
  edition: 4,
  year: 2017,
  releaseDate: new Date(2017, 11, 1)
};
let jsonText = JSON.stringify(book); 
let bookCopy = JSON.parse(jsonText,
  (key, value) => key == "releaseDate" ? new Date(value) : value);
console.log(bookCopy.releaseDate.getFullYear());
```

以上代码在 book 对象中增加了 releaseDate 属性，是一个 Date 对象。这个对象在被序列化为 JSON 字符串后，又被重新解析为一个对象 bookCopy。这里的还原函数会查找"releaseDate"键，如 果找到就会根据它的日期字符串创建新的 Date 对象。得到的 bookCopy.releaseDate 属性又变回了 Date 对象，因此可以调用其 getFullYear()方法。
