# class

```js
const nameSymbol = Symbol('name');
const ageSymbol = Symbol('age');

class Person {
  constructor(name, age) {
    this[nameSymbol] = name;
    this[ageSymbol] = age;
  }

  getAge() {
    return this[ageSymbol];
  }
  getName() {
    return this[nameSymbol];
  }
}
var person = new Person("John", 30);
console.log(person.getAge()); // 30
console.log(person[nameSymbol]); // undefined
console.log(person[ageSymbol]); // undefined
```

在 JavaScript 中，私有属性和私有方法是类的实例的非公开成员，它们只能在类的内部访问和修改。从 ECMAScript 2020(ES11) 开始，JavaScript 提供了一种新的语法来实现私有属性和私有方法，即使用井号（#）作为前缀。

私有属性：
在类内部使用 # 作为私有属性的前缀。
只能在类的方法中访问和修改私有属性。
私有属性不能在类的外部访问。
私有方法：
在类内部使用 # 作为私有方法的前缀。
私有方法只能在类的其他方法中调用。
私有方法不能在类的外部调用。
以下是一个使用私有属性和私有方法的例子：

```js
class MyClass {
  #privateProperty = "I'm a private property";
  #privateMethod() {
    console.log("I'm a private method");
  }

  publicMethod() {
    console.log("Public method:");
    console.log("Accessing private property:", this.#privateProperty);
    console.log("Calling private method:");
    this.#privateMethod();
  }
}

const myInstance = new MyClass();
myInstance.publicMethod(); // 正常运行
console.log(MyClass.#privateProperty) // 报错：私有属性无法在类外访问
console.log(MyClass.#privateMethod()) // 报错：私有方法无法在类外访问
// console.log(myInstance.#privateProperty); // 报错：私有属性无法在类外访问
// myInstance.#privateMethod(); // 报错：私有方法无法在类外调用
```

在这个例子中，MyClass 类有一个私有属性 #privateProperty 和一个私有方法 #privateMethod。这些私有成员只能在类的内部访问，无法在类的实例上进行访问。publicMethod 是一个公共方法，可以访问和调用私有属性和私有方法。在类的外部，尝试访问或调用私有成员会导致错误。
