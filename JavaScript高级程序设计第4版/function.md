# function

函数是对象，函数名是指针
因为函数名就是指向函数的指针，所以它们跟其他包含对象指针的变量具有相同的行为。这意味着 一个函数可以有多个名称，如下所示:

```js
function sum(num1, num2) {
  return num1 + num2;
}
console.log(sum(10, 10));
// 20
let anotherSum = sum;
console.log(anotherSum(10, 10));  // 20
sum = null;
console.log(anotherSum(10, 10));  // 20
```

以上代码定义了一个名为 sum()的函数，用于求两个数之和。然后又声明了一个变量 anotherSum， 5 并将它的值设置为等于 sum。注意，使用不带括号的函数名会访问函数指针，而不会执行函数。此时， anotherSum 和 sum 都指向同一个函数。调用 anotherSum()也可以返回结果。把 sum 设置为 null 之后，就切断了它与函数之间的关联。而 anotherSum()还是可以照常调用，没有问题。

## 函数作为值

```js
function callSomeFunction(someFunction, someArgument) {
  return someFunction(someArgument);
}
function add10(num) {
  return num + 10;
}
let result1 = callSomeFunction(add10, 10);
console.log(result1);  // 20
function getGreeting(name) {
  return "Hello, " + name;
}
let result2 = callSomeFunction(getGreeting, "Nicholas");
console.log(result2);  // "Hello, Nicholas"
```

从一个函数中返回另一个函数也是可以的，而且非常有用。例如，假设有一个包含对象的数组，而 我们想按照任意对象属性对数组进行排序。为此，可以定义一个 sort()方法需要的比较函数，它接收 两个参数，即要比较的值。但这个比较函数还需要想办法确定根据哪个属性来排序。这个问题可以通过 定义一个根据属性名来创建比较函数的函数来解决。比如:

```js
function createComparisonFunction(propertyName) {
return function(object1, object2) {
  let value1 = object1[propertyName];
  let value2 = object2[propertyName];
  if (value1 < value2) {
    return -1;
  } else if (value1 > value2) {
    return 1;
  } else {
    return 0;
} };
}
```

这个函数的语法乍一看比较复杂，但实际上就是在一个函数中返回另一个函数，注意那个 return 操作符。内部函数可以访问 propertyName 参数，并通过中括号语法取得要比较的对象的相应属性值。 取得属性值以后，再按照 sort()方法的需要返回比较值就行了。这个函数可以像下面这样使用:

```js
let data = [
  {name: "Zachary", age: 28},
  {name: "Nicholas", age: 29}
];
data.sort(createComparisonFunction("name"));
console.log(data[0].name);  // Nicholas
data.sort(createComparisonFunction("age"));
console.log(data[0].name);  // Zachary
```

## 尾调用优化的代码

```js
function fib(n) {
  if (n < 2) {
    return n;
  }
  return fib(n - 1) + fib(n - 2);
}
console.log(fib(0)); // 0
console.log(fib(1)); // 1
console.log(fib(2)); // 1 6 console.log(fib(3)); // 2
console.log(fib(4)); // 3
console.log(fib(5)); // 5
console.log(fib(6)); // 8
```

显然这个函数不符合尾调用优化的条件，因为返回语句中有一个相加的操作。结果，fib(n)的栈 帧数的内存复杂度是 O(2n)。因此，即使这么一个简单的调用也可以给浏览器带来麻烦:
fib(1000);
当然，解决这个问题也有不同的策略，比如把递归改写成迭代循环形式。不过，也可以保持递归实 现，但将其重构为满足优化条件的式。为此可以使用两个嵌套的函数，外部函数作为基础框架，内部 函数执行递归:

```js
// 基础框架 function fib(n) {
function fib(n) {
  return fibImpl(0, 1, n)
}
function fibImpl(a, b, n) {
  if (n == 0) {
    return a;
  }
  return fibImpl(b, a + b, n - 1);
}
fib(4)
```

## 其它

```js
function selectFrom(lowerValue, upperValue) {
  let choices = upperValue - lowerValue + 1;
  return Math.floor(Math.random() * choices + lowerValue);
}
let num = selectFrom(2, 10);
console.log(num); //2~10范围内的值，其中包含2和10
let colors = ["red", "green", "blue", "yellow", "black", "purple", "brown"];
let color = colors[selectFrom(0, colors.length - 1)]; //随机颜色
console.log(color);

// 要知道数组中的最大值和最小值，可以像下面这样使用扩展操作符:
let values = [1, 2, 3, 4, 5, 6, 7, 8];
let max = Math.max(...val);
```
