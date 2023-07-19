# 原生JS与React使用SOLID

## SOLID

SOLID 原则是面向对象 class 设计的五大原则。

- **单一职责原则（Single Responsibility Principle，SRP）**：一个类应该只有一个引起它变化的原因。这意味着每个类应该只负责一件事情，而不是承担多个职责。这可以使类更加可维护和易于理解。
- **开放封闭原则（Open-Closed Principle，OCP）**：软件实体（类、模块、函数等）应该对扩展开放，对修改关闭。这意味着当需要添加新的功能时，我们应该通过添加新的代码来扩展它，而不是修改现有的代码。
- **里式替换原则（Liskov Substitution Principle，LSP）**：子类应该能够替换其父类并且不会破坏程序的正确性。这意味着在编写继承层次结构时，子类应该能够完全替代其父类，而不会影响程序的行为。
- **接口隔离原则（Interface Segregation Principle，ISP）**：客户端不应该强制依赖于它们不需要的接口。这意味着在设计接口时，应该将其分解为更小的、更具体的接口，以便客户端只依赖于它们需要的接口。
- **依赖倒置原则（Dependency Inversion Principle，DIP）**：高层模块不应该依赖于低层模块，它们应该依赖于抽象。这意味着在设计系统时，我们应该尽量依赖于抽象而不是具体的实现，以提高系统的灵活性和可维护性。

## SOLID VS 封装、继承、多态

SOLID原则和封装、继承、多态的关联在于它们都是面向对象编程的基本概念，都旨在实现代码的可维护性、可扩展性和可重用性。SOLID 原则和封装、继承、多态的关系如下：

- 单一职责原则（SRP）和封装：封装是实现 SRP 的重要手段之一。通过封装，我们可以将实现细节隐藏在对象内部，让对象只暴露必要的接口，从而实现职责的单一化。

- 开放-封闭原则（OCP）和继承：继承是实现 OCP 的重要手段之一。通过继承，我们可以创建新的子类来扩展父类的功能，而不需要修改父类的代码。这样，我们就可以在不影响原有代码的情况下，灵活地添加新的功能或修改现有功能，从而实现代码的可扩展性和可维护性。

- 里氏替换原则（LSP）和多态：多态是实现 LSP 的重要手段之一。通过多态，我们可以将父类和子类视为同一类型的对象，并且可以在不知道对象具体类型的情况下，调用对象的方法。这样，我们就可以将子类对象替换父类对象，而不影响程序的行为，从而实现代码的可重用性和可扩展性。

- 接口隔离原则（ISP）和接口：接口是实现 ISP 的重要手段之一。通过接口，我们可以将一个大接口拆分为多个小接口，每个小接口只包含一组相关的方法。这样，我们就可以让类只实现自己需要的接口，而不需要实现不需要的方法，从而实现代码的高内聚性和低耦合性。

- 依赖倒置原则（DIP）和抽象：抽象是实现 DIP 的重要手段之一。通过抽象，我们可以将具体的实现细节抽象成接口或抽象类，让高层模块只依赖于抽象而不依赖于具体实现。这样，我们就可以在不修改高层模块的情况下，更换具体实现，从而实现代码的可维护性和可扩展性。

### 单一职责原则（SRP）和封装

TS代码示例：

```ts
class Person {
  private name: string;
  private age: number;

  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }

  getName(): string {
    return this.name;
  }

  getAge(): number {
    return this.age;
  }
}
```

Person 类只暴露了 getName 和 getAge 两个方法，隐藏了对象内部的实现细节，达到了职责的单一化。

```tsx
export function SAP() {
  const { products } = useProducts();

  const { filterRate, handleRating } = useRateFilter();
  const filteredProducts = useFilterProductsByRating(products, filterRate);

  return (
   <div>
      <Filter filterRate={filterRate} handleRating={handleRating} />
      <div>
        {filteredProducts.map((product: IProduct) => (
          <Product product={product} />
        ))}
      </div>
    </div>
  );
}
```

Filter、Product组件只负责呈现数据，隐藏了组件内部的实现细节。useRateFilter负责Filter组件交互逻辑，useProducts负责获取products数据。filterProducts使products、filterRate关联起来。

### 开放-封闭原则（OCP）和继承

```ts
class Person {
  private name: string;
  private age: number;

  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }

  getName(): string {
    return this.name;
  }

  getAge(): number {
    return this.age;
  }
}

class Student extends Person {
  private grade: string;

  constructor(name: string, age: number, grade: string) {
    super(name, age);
    this.grade = grade;
  }

  getGrade(): string {
    return this.grade;
  }
}
```

Student 类继承自 Person 类，扩展了一个新的属性 grade，而不需要修改 Person 类的代码。这样在不影响原有代码的情况下，灵活地添加新的功能或修改现有功能，达到了代码的可扩展性和可维护性。

```tsx
export function OCP() {
  return (
    <div>
      <Button
        text="Go Home"
        // role="forward"
        icon={<HiOutlineArrowNarrowRight />}
      />
      <Button
        text="Go Back"
        // role="back"
        icon={<HiOutlineArrowNarrowLeft />}
      />
    </div>
  );
}

interface IButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  text: string;
  role?: "back" | "forward" | "main" | "not-found";
  icon?: React.ReactNode;
}

export function Button(props: IButtonProps) {
  const { text, role, icon } = props;

  return (
    <button
      {...props}
    >
      {text}
      <div>
        {/* {role === "forward" && <HiOutlineArrowNarrowRight />}// 不符合OCP原则
        {role === "back" && <HiOutlineArrowNarrowLeft />} */}
        {icon} {/* 符合OCP原则*/}
      </div>
    </button>
  );
}
通过角色判断按钮的图标，每次新增角色就得修改组件的代码增加新的role和icon图，这不符合OCP原则。而使用了icon属性来渲染按钮的图标。通过这种方式，我们可以在不修改现有代码的情况下添加新的图标，而无需修改按钮的角色属性。

```

### 里氏替换原则（LSP）和多态

```ts
interface Animal {
  makeSound(): void;
}

class Dog implements Animal {
  makeSound(): void {
    console.log("汪汪汪!");
  }
}

class Cat implements Animal {
  makeSound(): void {
    console.log("喵喵喵!");
  }
}

function makeAnimalSound(animal: Animal): void {
  animal.makeSound();
}

const dog: Dog = new Dog();
const cat: Cat = new Cat();

makeAnimalSound(dog); // 输出: 汪汪汪!
makeAnimalSound(cat); // 输出: 喵喵喵!
```

Animal 接口定义了一个 makeSound 方法，Dog 和 Cat 类都实现了这个方法，但是具体实现不同。makeAnimalSound 函数接收一个 Animal 类型的参数，可以传入 Dog 或 Cat 对象，调用 makeSound 方法，这样就可以将子类对象替换父类对象，而不影响程序的行为，达到了代码的可重用性和可扩展性。

```tsx
export function LSP() {
  const [value, setValue] = useState("");

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    setValue(e.target.value);
  };

  return (
    <SearchInput value={value} onChange={handleChange} isLarge />
  );
}

interface ISearchInputProps
  extends React.InputHTMLAttributes<HTMLInputElement> {
  isLarge?: boolean;
}

export function SearchInput(props: ISearchInputProps) {
  const { value, onChange, isLarge, ...restProps } = props;
  return (
    {/* ... */}
  )
}
```

`SearchInput`组件扩展了`InputHTMLAttributes`接口，并通过`...restProps`语法将未知属性传递给`input`元素，这使得组件更加灵活和可扩展。这个设计允许我们通过添加新的属性来扩展组件的功能，而不会影响到现有的代码。我们通过切换`isLarge`属性来改变组件的样式，而不需要修改现有的代码。因此这种设计方式符合LSP原则，因为它允许我们通过添加新的属性和切换属性的值来扩展组件的功能，而不需要修改现有的代码。

### 接口隔离原则（ISP）和接口

```ts
interface CanFly {
  fly(): void;
}

interface CanSwim {
  swim(): void;
}

class Bird implements CanFly {
  fly(): void {
    console.log("Flying...");
  }
}

class Fish implements CanSwim {
  swim(): void {
    console.log("Swimming...");
  }
}

class Duck implements CanFly, CanSwim {
  fly(): void {
    console.log("Flying...");
  }

  swim(): void {
    console.log("Swimming...");
  }
}
```

CanFly 和 CanSwim 接口将一个大接口拆分为多个小接口，每个小接口只包含一组相关的方法。Bird 和 Fish 类分别实现了 CanFly 和 CanSwim 接口，而 Duck 类同时实现了 CanFly 和 CanSwim 接口，这样可以让类只实现自己需要的接口，达到了代码的高内聚性和低耦合性。

```tsx
export interface IProduct {
  id: string;
  title: string;
  price: number;
  rating: { rate: number };
  image: string;
}

interface IProductProps {
  product: IProduct;
}
export function Product(props: IProductProps) {
  const { product } = props;
  const { id, title, price, rating, image } = product;
  return (
    {/* ... */}
     <Thumbnail imageUrl={product.image} />
    {/* ... */}

  )
}

interface IThumbnailProps {
  // product: IProduct;
  imageUrl: string;
}

export function Thumbnail(props: IThumbnailProps) {
  // const { product } = props;
  const { imageUrl } = props;

  return (
    <img
      src={imageUrl}
      alt="product image"
    />
  );
}

```

Thumbnail组件只需要一个imageUrl属性，用于指定要渲染的图片的URL。这个设计符合ISP原则，因为它将接口拆分为最小的部分，并将实现细节隐藏在组件内部，组件只需要实现它需要的部分接口，并且避免了不必要的耦合。

### 依赖倒置原则（DIP）和抽象

```ts
interface Food {
  getName(): string;
}

class Pizza implements Food {
  getName(): string {
    return "Pizza";
  }
}

class Hamburger implements Food {
  getName(): string {
    return "Hamburger";
  }
}

class Person {
  private food: Food;

  constructor(food: Food) {
    this.food = food;
  }

  eat(): void {
    console.log(`吃 ${this.food.getName()}...`);
  }
}

const pizza: Pizza = new Pizza();
const hamburger: Hamburger = new Hamburger();
const person1: Person = new Person(pizza);
const person2: Person = new Person(hamburger);

person1.eat(); // 输出: 吃 Pizza...
person2.eat(); // 输出: 吃 Hamburger...
```

Person 类依赖于 Food 接口而不是具体的实现类，这样就可以在不修改高层模块的情况下，更换具体实现，达到了代码的可维护性和可扩展性。Pizza 和 Hamburger 类实现了 Food 接口，分别返回了自己的名字，Person 类接收一个 Food 类型的参数，可以传入 Pizza 或 Hamburger 对象，调用 eat 方法，输出相应的食物名字。

```tsx
export function DIP() {
  // return <Form />;
  return <ConnectedForm />;
}
export function ConnectedForm() {
  const handleSubmit = async (email: string, password: string) => {
    await axios.post("https://localhost:3000/login", {
      email,
      password,
    });
  };
  return <Form onSubmit={handleSubmit} />;
}

interface IFormProps {
  onSubmit: (email: string, password: string) => void;
}

export function Form(props: IFormProps) {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");

  const { onSubmit } = props;
  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    onSubmit(email, password);
  };
  return (
    {/* ... */}
    <form onSubmit={handleSubmit}>
    {/* ... */}
  )
}
```

ConnectedForm组件通过Form组件的props属性将handleSubmit函数传递给了Form组件。这个设计将低层模块（Form）的实现细节隐藏在组件内部，同时允许高层模块（ConnectedForm）通过传递函数来决定如何处理表单的提交操作。而Form组件只关注表单的呈现和处理，将表单的提交逻辑委托给了外部函数。这个使得Form组件更加通用和可复用，而不需要关注它被用于哪个上下文中。



总结：使用了SOLID原则与原生js以及react实际开发如何使用
