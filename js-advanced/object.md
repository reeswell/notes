# object

## 属性的类型

在调用Object.defineProperty()时，configurable、enumerable和writable的值如果不 指定，则都默认为 false

### 数据属性

数据属性包含一个保存数据值的位置。值会从这个位置读取，也会写入到这个位置。数据属性有 4 个特性描述它们的行为。

```js
let person = {};
Object.defineProperty(person, "name", {
  configurable: false,
  value: "Nicholas"
});
console.log(person.name); // "Nicholas"
delete person.name; //报错
console.log(person.name); // "Nicholas"
```

### 访问器属性

访问器属性不包含数据值。相反，它们包含一个获取(getter)函数和一个设置(setter)函数，不 过这两个函数不是必需的。在读取访问器属性时，会调用获取函数，这个函数的责任就是返回一个有效 的值。在写入访问器属性时，会调用设置函数并传入新值，这个函数必须决定对数据做出什么修改。访 问器属性有 4 个特性描述它们的行为。

```js

let book = {};
Object.defineProperties(book, {
  year_: {
    value: 2017,
    // get: function () { // 报错，数据属性不能和访问器属性同时使用
    //   return this.edition;
    // },
  },
  edition: {
    value: 1
  },
  year: {
    get: function () {
      return this.year_;
    },
    set: function (newValue) {
      if (newValue > 2017) {
        this.year_ = newValue;
        this.edition += newValue - 2017;
      }
    }
  }
})


console.log(Object.getOwnPropertyDescriptors(book)); 

// {
//   edition: {
//     configurable: false,
//     enumerable: false,
//     value: 1,
//     writable: false
//   },
//   year: {
//     configurable: false,
//     enumerable: false,

//     get: f(),
//     set: f(newValue),
//   },
//   year_: {
//     configurable: false,
//     enumerable: false,
//     value: 2017,
//     writable: false
//   }
// }

const dest = {
  set a(val) {
    console.log(`Invoked dest setter with param ${ val }`);
  }
};
const src = {
  get a() {
    console.log('Invoked src getter');
    return 'foo';
  }
};
Object.assign(dest, src);
console.log(dest);
console.log(src.a); 
```

调用 src 的获取方法
调用 dest 的设置方法并传入参数"foo"
因为这里的设置函数不执行赋值操作
所以实际上并没有把值转移过来 console.log(dest); // { set a(val) {...} }

## 对象解构

解构在内部使用函数 ToObject()(不能在运行时环境中直接访问)把源数据结构转换为对象。这 意味着在对象解构的上下文中，原始值会被当成对象。这也意味着(根据 ToObject()的定义)，null 和 undefined 不能被解构，否则会抛出错误。

```js
let { length } = 'foobar';
console.log(length);        // 6
let { constructor: c } = 4;
console.log(c === Number);  // true
let { _ } = null;           // TypeError
let { _ } = undefined;      // TypeError
// 解构并不要求变量必须在解构表达式中声明。不过，如果是给事先声明的变量赋值，则赋值表达式 必须包含在一对括号中:
let personName, personAge;
let person = {
  name: 'Matt',
  age: 27
};
({ name: personName, age: personAge } = person);
console.log(personName, personAge); // Matt, 27
```

### 嵌套解构

解构对于引用嵌套的属性或赋值目标没有限制。为此，可以通过解构来复制对象属性:

```js
let person = {
  name: 'Matt',
  age: 27,
  job: {
    title: 'Software engineer'
  }
};
let personCopy = {};
({
  name: personCopy.name,
  age: personCopy.age,
  job: personCopy.job
} = person);
// 因为一个对象的引用被赋值给personCopy，所以修改 // person.job 对象的属性也会影响 personCopy person.job.title = 'Hacker'
console.log(person);
// { name: 'Matt', age: 27, job: { title: 'Hacker' } }
console.log(personCopy);
// { name: 'Matt', age: 27, job: { title: 'Hacker' } }
```

解构赋值可以使用嵌套结构，以匹配嵌套的属性:

```js
  name: 'Matt',
  age: 27,
  job: {
    title: 'Software engineer'
  }
};
// 声明 title 变量并将 person.job.title 的值赋给它 let { job: { title } } = person;
console.log(title); // Software engineer

```

在外层属性没有定义的情况下不能使用嵌套解构。无论源对象还是目标对象都一样:

  ```js
let person = {
  job: {
    title: 'Software engineer'
  }
};
let personCopy = {};
// ({
foo 在源对象上是 undefined
foo: {
  bar: personCopy.bar
 }
} = person);
// TypeError: Cannot destructure property 'bar' of 'undefined' or 'null'.
// job 在目标对象上是 undefined 
({
  job: {
    title: personCopy.job.title
  }
} = person); 
// TypeError: Cannot set property 'title' of undefined
  ```

### 参数上下文匹配

在函数参数列表中也可以进行解构赋值。对参数的解构赋值不会影响 arguments 对象，但可以在 函数签名中声明在函数体内使用局部变量:

```js
let person = {
  name: 'Matt',
  age: 27
};
function printPerson(foo, { name, age }, bar) {
  console.log(arguments);
  console.log(name, age);
}
function printPerson2(foo, { name: personName, age: personAge }, bar) {
  console.log(arguments);
  console.log(personName, personAge);
}
printPerson('1st', person, '2nd');
// ['1st', { name: 'Matt', age: 27 }, '2nd']
// 'Matt', 27
printPerson2('1st', person, '2nd');
     // ['1st', { name: 'Matt', age: 27 }, '2nd']
     // 'Matt', 27
```

## 创建对象

### 工厂模式

```js
function createPerson(name, age, job) {
  let o = new Object();
  o.name = name;
  o.age = age;
  o.job = job;
  o.sayName = function() {
    console.log(this.name);
  };
return o; }
let person1 = createPerson("Nicholas", 29, "Software Engineer"); 
let person2 = createPerson("Greg", 27, "Doctor");
```

### 构造函数模式

比如，前面的例子使用构造函数模式可以这样写:

```js
function Person(name, age, job){
  this.name = name;
  this.age = age;
  this.job = job;
  this.sayName = function() {
    console.log(this.name);
}; }
let person1 = new Person("Nicholas", 29, "Software Engineer");
let person2 = new Person("Greg", 27, "Doctor");
person1.sayName(); // Nicholas 10 
person2.sayName(); // Greg
```

在这个例子中，Person()构造函数代替了 createPerson()工厂函数。实际上，Person()内部 的代码跟 createPerson()基本是一样的，只是有如下区别

- 没有显式地创建对象。
- 属性和方法直接赋值给了 this。
- 没有 return。

要创建 Person 的实例，应使用 new 操作符。以这种方式调用构造函数会执行如下操作。
(1) 在内存中创建一个新对象。
(2) 这个新对象内部的[[Prototype]]特性被赋值为构造函数的 prototype 属性。
(3) 构造函数内部的 this 被赋值为这个新对象(即 this 指向新对象)。
(4) 执行构造函数内部的代码(给新对象添加属性)。
(5) 如果构造函数返回非空对象，则返回该对象;否则，返回刚创建的新对象。

**构造函数的问题**
构造函数的主要问题在于，其定义的方法会在每个实例上 都创建一遍。因此对前面的例子而言，person1 和 person2 都有名为 sayName()的方法，但这两个方 法不是同一个 Function 实例

```js
function Person(name, age, job){
  this.name = name;
  this.age = age;
  this.job = job;
  this.sayName = new Function("console.log(this.name)"); // 逻辑等价 }
}
```

要解决这个问题，可以把函数定义转移到构造函数外部:

```js
function Person(name, age, job){
  this.name = name;
  this.age = age;
  this.job = job;
  this.sayName = sayName;
}
function sayName() {
  console.log(this.name);
}
```

### Object.keys() vs Object.getOwnPropertyNames()

```js
function Person() { }
Person.prototype.name = "Nicholas";
Person.prototype.age = 29;
Person.prototype.job = "Software Engineer";
Person.prototype.sayName = function () {
  console.log(this.name);
};
let keys = Object.keys(Person.prototype);
console.log(keys);   // "name,age,job,sayName"
let p1 = new Person();
p1.name = "Rob";
p1.age = 31;
let p1keys = Object.keys(p1);
// console.log(p1keys); // "[name,age]"
let keys = Object.getOwnPropertyNames(p1);
console.log(keys); // "[constructor,name,age,job,sayName]"

```

### 理解原型

```js
/** * * * *
构造函数可以是函数表达式
也可以是函数声明，因此以下两种形式都可以:
function Person() {}
let Person = function() {}
  */
function Person() { }
/**
* 声明之后，构造函数就有了一个 * 与之关联的原型对象:
*/
console.log(typeof Person.prototype);
console.log(Person.prototype);
// {
//   constructor: f Person(),
//   __proto__: Object
// }
/**
* 如前所述，构造函数有一个 prototype 属性 * 引用其原型对象，而这个原型对象也有一个 * constructor 属性，引用这个构造函数
* 换句话说，两者循环引用:
*/
console.log(Person.prototype.constructor === Person); // true
/**
* 正常的原型链都会终止于 Object 的原型对象 * Object 原型的原型是 null
*/
console.log(Person.prototype.__proto__ === Object.prototype); // true
console.log(Person.prototype.__proto__.constructor === Object); // true
console.log(Person.prototype.__proto__.__proto__ === null);// true
console.log(Person.prototype.__proto__);
// {
//   constructor: f Object(),
//   toString: ...
//   hasOwnProperty: ...
//   isPrototypeOf: ...
//   ...
// }
let person1 = new Person(),
  person2 = new Person();
/**
* 构造函数、原型对象和实例 * 是 3 个完全不同的对象: */
console.log(person1 !== Person); // true
console.log(person1 !== Person.prototype); // true
console.log(Person.prototype !== Person);  // true
/**
* 实例通过__proto__链接到原型对象，
* 它实际上指向隐藏特性[[Prototype]] *
* 构造函数通过 prototype 属性链接到原型对象 *
* 实例与构造函数没有直接联系，与原型对象有直接联系 */
console.log(person1.__proto__ === Person.prototype);// true
console.log(person1.__proto__.constructor === Person); // true
/**
* 同一个构造函数创建的两个实例 * 共享同一个原型对象:
*/
console.log(person1.__proto__ === person2.__proto__); // true
/**
* instanceof 检查实例的原型链中
* 是否包含指定构造函数的原型:
 */
console.log(person1 instanceof Person);// true
console.log(person1 instanceof Object);// true
console.log(Person.prototype instanceof Object);  // true

```

![image-20230506233646732](https://cdn.jsdelivr.net/gh/xxydrr/my_pic/img/image-20230506233646732.png)
虽然不是所有实现都对外暴露了[[Prototype]]，但可以使用 isPrototypeOf()方法确定两个对 象之间的这种关系。本质上，isPrototypeOf()会在传入参数的[[Prototype]]指向调用它的对象时 返回 true，如下所示:

```js
console.log(Person.prototype.isPrototypeOf(person1)); // true 
console.log(Person.prototype.isPrototypeOf(person2)); // true
```

这里通过原型对象调用 isPrototypeOf()方法检查了 person1 和 person2。因为这两个例子内 部都有链接指向 Person.prototype，所以结果都返回 true。
ECMAScript 的 Object 类型有一个方法叫 Object.getPrototypeOf()，返回参数的内部特性[[Prototype]]的值。使用 Object.getPrototypeOf()可以 方便地取得一个对象的原型 例如:

```js
console.log(Object.getPrototypeOf(person1) == Person.prototype); // true 
console.log(Object.getPrototypeOf(person1).name); // "Nicholas"
```

Object 类型还有一个 setPrototypeOf()方法，可以向实例的私有特性[[Prototype]]写入一 个新值。这样就可以重写一个对象的原型继承关系:

```js
let biped = {
  numLegs: 2
};
let person = {
  name: 'Matt'
};
Object.setPrototypeOf(person, biped);
console.log(person.name); // Matt
console.log(person.numLegs); // 2
 console.log(Object.getPrototypeOf(person) === biped); // true
```

使用 Object.setPrototypeOf()可能造成的性能下降，可以通过 Object.create()来创 建一个新对象，同时为其指定原型:

```js
let biped = {
  numLegs: 2
};
let person = Object.create(biped);
person.name = 'Matt';
console.log(person.name); // Matt
console.log(person.numLegs); // 2
console.log(Object.getPrototypeOf(person) === biped); // true
```

### 原型的动态性

```js
function Person() { }
Person.prototype = {
  name: "Nicholas",
  age: 29,
  job: "Software Engineer",
  sayName() {
    console.log(this.name);
  }
};
// 恢复 constructor 属性 
Object.defineProperty(Person.prototype, "constructor", {
  enumerable: false,
  value: Person
});
let friend = new Person();
Person.prototype.sayHi = function () {
  console.log("hi");
};
friend.sayHi(); // "hi"，没问题!
```

虽然随时能给原型添加属性和方法，并能够立即反映在所有对象实例上，但这跟重写整个原型是两 回事。实例的[[Prototype]]指针是在调用构造函数时自动赋值的，这个指针即使把原型修改为不同 的对象也不会变。重写整个原型会切断最初原型与构造函数的联系，但实例引用的仍然是最初的原型。 记住，实例只有指向原型的指针，没有指向构造函数的指针。来看下面的例子:

```js
function Person() {}
    let friend = new Person();
    Person.prototype = {
      constructor: Person,
      name: "Nicholas",
      age: 29,
      job: "Software Engineer",
      sayName() {
        console.log(this.name);
      }
};
friend.sayName(); // 错误
```

在这个例子中，Person 的新实例是在重写原型对象之前创建的。在调用 friend.sayName()的时 候，会导致错误。这是因为 firend 指向的原型还是最初的原型，而这个原型上并没有 sayName 属性。图 8-3 展示了这里面的原因。

![image-20230423071348115](https://cdn.jsdelivr.net/gh/xxydrr/my_pic/img/image-20230423071348115.png)

重写构造函数上的原型之后再创建的实例才会引用新的原型。而在此之前创建的实例仍然会引用最

初的原型。

### 原型的问题

原型模式也不是没有问题。首先，它弱化了向构造函数传递初始化参数的能力，会导致所有实例默 认都取得相同的属性值。虽然这会带来不便，但还不是原型的最大问题。原型的最主要问题源自它的共 享特性。

我们知道，原型上的所有属性是在实例间共享的，这对函数来说比较合适。另外包含原始值的属性 也还好，如前面例子中所示，可以通过在实例上添加同名属性来简单地遮蔽原型上的属性。真正的问题 来自包含引用值的属性。来看下面的例子:

```js
    function Person() {}
    Person.prototype = {
      constructor: Person,
      name: "Nicholas",
      age: 29,
      job: "Software Engineer",
      friends: ["Shelby", "Court"],
      sayName() {
        console.log(this.name);
}};
    let person1 = new Person();
    let person2 = new Person();
    person1.friends.push("Van");
    console.log(person1.friends);  // "Shelby,Court,Van"
    console.log(person2.friends);  // "Shelby,Court,Van"
    console.log(person1.friends === person2.friends);  // true
```

这里，Person.prototype 有一个名为 friends 的属性，它包含一个字符串数组。然后这里创建 了两个 Person 的实例。person1.friends 通过 push 方法向数组中添加了一个字符串。由于这个 friends 属性存在于 Person.prototype 而非 person1 上，新加的这个字符串也会在(指向同一个 数组的)person2.friends 上反映出来。如果这是有意在多个实例间共享数组，那没什么问题。但一 般来说，不同的实例应该有属于自己的属性副本。这就是实际开发中通常不单独使用原型模式的原因

## 继承

### 原型链

ECMA-262 把原型链定义为 ECMAScript 的主要继承方式。其基本思想就是通过原型继承多个引用 类型的属性和方法。重温一下构造函数、原型和实例的关系:每个构造函数都有一个原型对象，原型有 一个属性指回构造函数，而实例有一个内部指针指向原型。如果原型是另一个类型的实例呢?那就意味 着这个原型本身有一个内部指针指向另一个原型，相应地另一个原型也有一个指针指向另一个构造函 数。这样就在实例和原型之间构造了一条原型链。这就是原型链的基本构想。

 实现原型链涉及如下代码模式:

```js
function SuperType() {
  this.property = true;
}
SuperType.prototype.getSuperValue = function () {
  return this.property;
};
function SubType() {
  this.subproperty = false;
}
// 继承SuperType
SubType.prototype = new SuperType();
SubType.prototype.getSubValue = function () {
  return this.subproperty;
};
let instance = new SubType();
// 调用 instance.getSuperValue()经过了 3 步搜索:instance、 SubType.prototype 和 SuperType.prototype
console.log(instance.getSuperValue()); // true
console.log(instance.constructor); // [Function: SuperType]

// 原型与实例的关系可以通过两种方式来确定。第一种方式是使用 instanceof 和 isPrototypeOf()方法
console.log(instance instanceof Object);     // true
console.log(instance instanceof SuperType);  // true
console.log(instance instanceof SubType);    // true

console.log(Object.prototype.isPrototypeOf(instance)); // true
console.log(SuperType.prototype.isPrototypeOf(instance)); // true 
console.log(SubType.prototype.isPrototypeOf(instance)); // true
```

![image-20230423075845224](https://cdn.jsdelivr.net/gh/xxydrr/my_pic/img/image-20230423075845224.png)

#### 默认原型

实际上，原型链中还有一环。默认情况下，所有引用类型都继承自 Object，这也是通过原型链实 现的。任何函数的默认原型都是一个 Object 的实例，这意味着这个实例有一个内部指针指向 Object.prototype。这也是为什么自定义类型能够继承包括 toString()、valueOf()在内的所有默 认方法的原因。因此前面的例子还有额外一层继承关系。图 8-5 展示了完整的原型链。

![image-20230423080253762](https://cdn.jsdelivr.net/gh/xxydrr/my_pic/img/image-20230423080253762.png)

### 原型继承

```js

function object(o) {
  function F() { }
  F.prototype = o;
  return new F();
}

let person = {
  name: "Nicholas",
  friends: ["Shelby", "Court", "Van"]
 };
let anotherPerson = object(person);
anotherPerson.name = "Greg";
anotherPerson.friends.push("Rob");
let yetAnotherPerson = object(person);
yetAnotherPerson.name = "Linda";
yetAnotherPerson.friends.push("Barbie");
console.log(person.friends); // "Shelby,Court,Van,Rob,Barbie"
console.log(anotherPerson.name);
```

Object.create()与这里的 object()方法效果相同

```js
let person = {
name: "Nicholas", 11 friends: ["Shelby", "Court", "Van"]
  };
let anotherPerson = Object.create(person);
anotherPerson.name = "Greg";
anotherPerson.friends.push("Rob");
let yetAnotherPerson = Object.create(person);
yetAnotherPerson.name = "Linda";
yetAnotherPerson.friends.push("Barbie");
console.log(person.friends); // "Shelby,Court,Van,Rob,Barbie"
```

Object.create()的第二个参数与 Object.defineProperties()的第二个参数一样:每个新增 属性都通过各自的描述符来描述。以这种方式添加的属性会遮蔽原型对象上的同名属性。比如:

```js
let person = {
  name: "Nicholas",
  friends: ["Shelby", "Court", "Van"]
};
let anotherPerson = Object.create(person, {
  name: {
    value: "Greg"
  }
});
console.log(anotherPerson.name);  // "Greg"
```

原型式继承非常适合不需要单独创建构造函数，但仍然需要在对象间共享信息的场合。但要记住，
属性中包含的引用值始终会在相关对象间共享，跟使用原型模式是一样的。

### 寄生式继承

原型式继承比较接近的一种继承方式是寄生式继承(parasitic inheritance)，也是 Crockford 首倡的 一种模式。寄生式继承背后的思路类似于寄生构造函数和工厂模式:创建一个实现继承的函数，以某种 方式增强对象，然后返回这个对象。基本的寄生继承模式如下:

```js
function createAnother(original){
let clone = object(original); // 通过调用函数创建一个新对象 clone.sayHi = function() { // 以某种方式增强这个对象
  console.log("hi");
};
return clone; // 返回这个对象 }
```

在这段代码中，createAnother()函数接收一个参数，就是新对象的基准对象。这个对象 original 会被传给 object()函数，然后将返回的新对象赋值给 clone。接着给 clone 对象添加一个新方法 sayHi()。最后返回这个对象。可以像下面这样使用 createAnother()函数:

```js

let person = {
  name: "Nicholas",
  friends: ["Shelby", "Court", "Van"]
};
let anotherPerson = createAnother(person);
anotherPerson.sayHi();  // "hi"
```

这个例子基于 person 对象返回了一个新对象。新返回的 anotherPerson 对象具有 person 的所 有属性和方法，还有一个新方法叫 sayHi()。
寄生式继承同样适合主要关注对象，而不在乎类型和构造函数的场景。object()函数不是寄生式 继承所必需的，任何返回新对象的函数都可以在这里使用。
**注意 通过寄生式继承给对象添加函数会导致函数难以重用，与构造函数模式类似。**

### 寄生式组合继承

```js
// 父类构造函数
function Animal(name) {
  this.name = name;
}

Animal.prototype.sayName = function() {
  console.log('My name is ' + this.name);
};

// 子类构造函数
function Dog(name, age) {
  Animal.call(this, name);
  this.age = age;
}

Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;

Dog.prototype.sayAge = function() {
  console.log('I am ' + this.age + ' years old');
};

// 测试
const dog = new Dog('Tom', 3);
dog.sayName(); // 输出 "My name is Tom"
dog.sayAge(); // 输出 "I am 3 years old"
```

在上面的例子中，我们定义了 Animal 构造函数作为父类，它有一个属性 name 和一个方法 sayName。然后我们定义了 Dog 构造函数作为子类，它通过借用 Animal 构造函数来继承它的属性 name，并添加了自己的属性 age 和方法 sayAge。最后，我们使用 Object.create() 方法创建了一个 Animal.prototype 的副本，并将其赋值给 Dog.prototype，实现了对父类方法的继承。
