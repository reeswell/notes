# 面向对象

## SOLID

SOLID 原则是面向对象 class 设计的五大原则。

- **单一职责原则（Single Responsibility Principle，SRP）**：一个类应该只有一个引起它变化的原因。这意味着每个类应该只负责一件事情，而不是承担多个职责。这可以使类更加可维护和易于理解。
- **开放封闭原则（Open-Closed Principle，OCP）**：软件实体（类、模块、函数等）应该对扩展开放，对修改关闭。这意味着当需要添加新的功能时，我们应该通过添加新的代码来扩展它，而不是修改现有的代码。
- **里式替换原则（Liskov Substitution Principle，LSP）**：子类应该能够替换其父类并且不会破坏程序的正确性。这意味着在编写继承层次结构时，子类应该能够完全替代其父类，而不会影响程序的行为。
- **接口隔离原则（Interface Segregation Principle，ISP）**：客户端不应该强制依赖于它们不需要的接口。这意味着在设计接口时，应该将其分解为更小的、更具体的接口，以便客户端只依赖于它们需要的接口。
- **依赖倒置原则（Dependency Inversion Principle，DIP）**：高层模块不应该依赖于低层模块，它们应该依赖于抽象。这意味着在设计系统时，我们应该尽量依赖于抽象而不是具体的实现，以提高系统的灵活性和可维护性。

## SOLID VS 封装、继承、多态

SOLID原则和封装、继承、多态的关联在于它们都是面向对象编程的基本概念，都旨在实现代码的可维护性、可扩展性和可重用性。SOLID 原则和封装、继承、多态的关系如下：

- **单一职责原则（SRP）和封装**：封装是实现 SRP 的重要手段之一。通过封装，我们可以将实现细节隐藏在对象内部，让对象只暴露必要的接口，从而实现职责的单一化。

- **开放-封闭原则（OCP）和继承**：继承是实现 OCP 的重要手段之一。通过继承，我们可以创建新的子类来扩展父类的功能，而不需要修改父类的代码。这样，我们就可以在不影响原有代码的情况下，灵活地添加新的功能或修改现有功能，从而实现代码的可扩展性和可维护性。

- **里氏替换原则（LSP）和多态**：多态是实现 LSP 的重要手段之一。通过多态，我们可以将父类和子类视为同一类型的对象，并且可以在不知道对象具体类型的情况下，调用对象的方法。这样，我们就可以将子类对象替换父类对象，而不影响程序的行为，从而实现代码的可重用性和可扩展性。

- **接口隔离原则（ISP）和接口**：接口是实现 ISP 的重要手段之一。通过接口，我们可以将一个大接口拆分为多个小接口，每个小接口只包含一组相关的方法。这样，我们就可以让类只实现自己需要的接口，而不需要实现不需要的方法，从而实现代码的高内聚性和低耦合性。

- **依赖倒置原则（DIP）和抽象**：抽象是实现 DIP 的重要手段之一。通过抽象，我们可以将具体的实现细节抽象成接口或抽象类，让高层模块只依赖于抽象而不依赖于具体实现。这样，我们就可以在不修改高层模块的情况下，更换具体实现，从而实现代码的可维护性和可扩展性。

## 单一职责原则（SRP）和封装

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

这段代码`Person`类只暴露了`getName`和`getAge`两个方法，隐藏了对象内部的实现细节，达到了职责的单一化。

```tsx
interface PersonProps {
  name: string;
  age: number;
}

const usePerson = (props: PersonProps) => {
  const [name, setName] = useState(props.name);
  const [age, setAge] = useState(props.age);

  const updateName = (newName: string) => {
    setName(newName);
  };

  const updateAge = (newAge: number) => {
    setAge(newAge);
  };

  return { name, age, updateName, updateAge };
};

const Person = (props: PersonProps) => {
  const { name, age, updateName, updateAge } = usePerson(props);
  
  return (
    <div>
      <p>Name: {name}</p>
      <p>Age: {age}</p>
      <button onClick={() => updateName('New Name')}>Update Name</button>
      <button onClick={() => updateAge(18)}>Update Age</button>
    </div>
  );
};
```

这段代码展示了如何使用React Hooks实现单一职责原则和封装。我们创建了一个名为`usePerson`的自定义Hook，它接收一个类型为`PersonProps`的参数，其中包括`name`和`age`属性。`usePerson`返回一个包含`name`、`age`、`updateName`和`updateAge`函数的对象，这些函数可以用于更新`name`和`age`状态。然后我们使用`usePerson`Hook 创建`Person`组件，它渲染`name`和`age`属性以及两个按钮，分别用于更新`name`和`age`状态。

## 开放-封闭原则（OCP）和继承

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

这段代码`Student` 类继承自 `Person` 类，扩展了一个新的属性 `grade`，而不需要修改 `Person` 类的代码。这样在不影响原有代码的情况下，灵活地添加新的功能或修改现有功能，达到了代码的可扩展性和可维护性。

```tsx
interface StudentProps extends PersonProps {
  grade: number;
}

const useStudent = (props: StudentProps) => {
  const { name, age ,updateAge,updateName} = usePerson(props);
  const [grade, setGrade] = useState(props.grade);

  const updateGrade = (newGrade: number) => {
    setGrade(newGrade);
  };

  return { person: { name, age }, grade, updateGrade,updateAge,updateName };
};

const Student = (props: StudentProps) => {
  const { person, grade, updateGrade,updateAge,updateName } = useStudent(props);

  return (
    <div>
      <p>Name: {person.name}</p>
      <p>Age: {person.age}</p>
      <p>Grade: {grade}</p>
      <button onClick={() => updateGrade(90)}>Update Grade</button>
      <button onClick={() => updateAge(19)}>Update Age</button>
      <button onClick={() => updateName('React')}>Update Name</button>
    </div>
  );
};
```

这段代码使用React Hooks实现开放-封闭原则和继承。我们创建了一个名为`useStudent`的自定义Hook，它接收一个类型为`StudentProps`的参数，其中包括`name`、`age`和`grade`属性。`useStudent`使用`usePerson` Hook 创建一个`person`对象，然后添加一个`grade`状态和一个`updateGrade`函数。`useStudent`返回一个包含`person`、`grade`和`updateGrade`函数的对象，这些函数可以用于更新`name`、`age`和`grade`状态。然后我们使用`useStudent` Hook 创建`Student`组件，它渲染`name`、`age`和`grade`属性以及三个按钮，分别用于更新`name`、`age`和`grade`状态。

## 里氏替换原则（LSP）和多态

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

`Animal` 接口定义了一个 `makeSound`方法，`Dog` 和 `Cat` 类都实现了这个方法，但是具体实现不同。`makeAnimalSound` 函数接收一个 `Animal` 类型的参数，可以传入 `Dog` 或`Cat` 对象，调用 `makeSound`方法，这样就可以将子类对象替换父类对象，而不影响程序的行为，达到了代码的可重用性和可扩展性。

```tsx
interface Animal {
  makeSound: () => void;
}

interface CanRun {
  run: () => void;
}

interface CanEat {
  eat: () => void;
}

const useDog = (): Animal & CanEat => {
  const makeSound = () => {
    console.log('汪汪汪!');
  };

  const eat = () => {
    console.log('狗正在吃！');
  };

  return { makeSound, eat };
};

const useCat = (): Animal & CanRun => {
  const makeSound = () => {
    console.log('喵喵喵!');
  };

  const run = () => {
    console.log('猫正在跑！');
  };

  return { makeSound, run };
};

const AnimalSound = ({ animal }: { animal: Animal }) => {
  animal.makeSound();

  return null;
};

const Animal = () => {
  const dog = useDog();
  const cat = useCat();

  return (
    <div>
      <AnimalSound animal={dog} />
      <AnimalSound animal={cat} />
    </div>
  );
};
```

这段代码使用React Hooks实现里氏替换原则和多态。我们定义了一个接口`Animal`，它包含一个`makeSound`函数。然后我们创建了两个接口`CanFly`和`CanSwim`，分别包含一个fly和一个swim函数。接着，我们创建了两个自定义Hooks `useDog`和`useCat`，它们分别返回一个实现了`Animal`和`CanFly`或`CanSwim`接口的对象。最后，我们使用`useDog`和`useCat` Hook 创建了两个实例`dog`和`cat`，并将它们传递给`AnimalSound`组件的`animal`属性。`AnimalSound`组件调用a`nimal.makeSound`函数，这样无论传递的是`dog`还是`cat`，都会输出相应的声音。

## 接口隔离原则（ISP）和接口

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

`CanFly`和 `CanSwim` 接口将一个大接口拆分为多个小接口，每个小接口只包含一组相关的方法。`Bird` 和 `Fish` 类分别实现了`CanFly` 和 `CanSwim` 接口，而 `Duck`类同时实现了 `CanFly` 和 `CanSwim` 接口，这样可以让类只实现自己需要的接口，达到了代码的高内聚性和低耦合性。

```tsx

interface CanFly {
  fly: () => void;
}

interface CanSwim {
  swim: () => void;
}

interface CanFlyAndSwim extends CanFly, CanSwim {}

const useBird = (): CanFly => {
  const fly = () => {
    console.log('Bird is flying...');
  };

  return { fly };
};

const useFish = (): CanSwim => {
  const swim = () => {
    console.log('Fish is swimming...');
  };

  return { swim };
};

const useDuck = (): CanFlyAndSwim => {
  const [isFlying, setIsFlying] = useState(false);
  const [isSwimming, setIsSwimming] = useState(false);

  const fly = () => {
    setIsFlying(true);
    console.log('Duck is flying...');
  };

  const swim = () => {
    setIsSwimming(true);
    console.log('Duck is swimming...');
  };

  return { fly, swim };
};

const Bird = () => {
  const { fly } = useBird();

  return (
    <div>
      <button onClick={fly}>Fly</button>
    </div>
  );
};

const Fish = () => {
  const { swim } = useFish();

  return (
    <div>
      <button onClick={swim}>Swim</button>
    </div>
  );
};

const Duck = () => {
  const { fly, swim } = useDuck();

  return (
    <div>
      <button onClick={fly}>Fly</button>
      <button onClick={swim}>Swim</button>
    </div>
  );
};

```

这段代码使用React Hooks实现接口隔离原则和接口。我们定义了两个接口`CanFly`和`CanSwim`，分别包含一个`fly`和一个`swim`函数。然后，我们创建了两个自定义Hooks `useBird`和`useFish`，它们分别返回一个实现了`CanFly`或`CanSwim`接口的对象。最后，我们创建了两个组件`Bird`和`Fish`，它们分别使用`useBird`和`useFish` Hook 创建了实例，并渲染了一个按钮，分别用于调用`fly`或`swim`函数。让`Duck`实现自己自己需要的接口。

## 依赖倒置原则（DIP）和抽象

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

`Person` 类依赖于 `Food` 接口而不是具体的实现类，这样就可以在不修改高层模块的情况下，更换具体实现，达到了代码的可维护性和可扩展性。`Pizza` 和 `Hamburger` 类实现了 `Food` 接口，分别返回了自己的名字，`Person` 类接收一个 `Food` 类型的参数，可以传入 `Pizza` 或 `Hamburger` 对象，调用`eat`方法，输出相应的食物名字。

```tsx

interface Food {
  getName(): string;
}

class Pizza implements Food {
  getName(): string {
    return 'Pizza';
  }
}

class Hamburger implements Food {
  getName(): string {
    return 'Hamburger';
  }
}

interface PersonProps {
  food: Food;
}

const Person: React.FC<PersonProps> = ({ food }) => {
  const [message, setMessage] = useState('');

  useEffect(() => {
    setMessage(`吃 ${food.getName()}...`);
  }, [food]);

  return <div>{message}</div>;
};

const App: React.FC = () => {
  const [food, setFood] = useState<Food>(new Pizza());

  const toggleFood = () => {
    setFood(food instanceof Pizza ? new Hamburger() : new Pizza());
  };

  return (
    <div>
      <Person food={food} />
      <button onClick={toggleFood}>Toggle Food</button>
    </div>
  );
};

```

这段代码通过抽象接口 `Food`，将具体实现细节抽象成了接口或抽象类，让高层模块 `Person` 组件和 `App` 组件只依赖于抽象（`Food` 接口）而不依赖于具体实现（`Pizza` 类和 `Hamburger` 类），因此 `Person` 组件不依赖于具体实现，符合了依赖倒置原则（DIP）和抽象的要求。
