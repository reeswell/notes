# useMemo

useMemo: 用于缓存计算结果，避免重复计算，类似于 shouldComponentUpdate 生命周期。和vue的计算属性类似

## 计算量大的方法

```tsx
 const [text, setText] = useState("");
  const [number, setNumber] = useState(0);

  const expensiveFunction = (n) => {
    console.log("方法开始！");
    let total = 0;
    for (let i = 1; i <= n; i++) {
      total += i;
    }
    return total;
  };
  // 依赖number
  const sum = useMemo(() => expensiveFunction(number), [number]);

  console.log("组件渲染！");
  // 结果 方法开始！-》 组件渲染！

  return (
    <div>
      <input
        onChange={(e) => setText(e.target.value)}
        placeholder="enter a text"
      />
      <input
        type="number"
        value={number}
        onChange={(e) => setNumber(e.target.value)}
      />
      <span>Total: {sum}</span>
    </div>
  );

```

## useMemo VS useEffect

```tsx
  // demo1
  const [state, setState] = useState({
    name: "",
    selected: false,
  });
  // useMemo相当于vue的计算属性
  const user = useMemo(
    () => ({
      name: state.name,
      selected: state.selected,
    }),
    [state.name, state.selected]
  );

  // 监听计算属性
  useEffect(() => {
    console.log(`user发生变化了`);
  }, [user]);

  //  理解watch成直接监听state属性
  useEffect(() => {
    console.log(`state.name、 state.selected发生变化了`);
  }, [state.name, state.selected]);


  // demo2
  const [age, setAge] = useState(null);
  const [country, setCountry] = useState("");

  const userType = {
    underAge: age < 18 ? true : false,
    citizen: country === "USA" ? true : false,
  };

  const userType = useMemo(
    () => ({
      underAge: age < 18 ? true : false,
      citizen: country === "USA" ? true : false,
    }),
    [age, country]
  );

  useEffect(() => {
    console.log("userType发生变化了");
  }, [userType]);

  console.log("component rendered!");
```

## 计算量大的组件与同级组件完全解耦

```tsx
const Expensive = () => {

  console.log("expensive component rendered!")

  let total = 0;
  for (let i = 0; i < 1000000000; i++) {
    total += i;
  }

  return <div>Expensive</div>;
};

// 错误做法，这样name改变会导致Expensive组件也重新渲染
// 结果： component rendered!  expensive component rendered!
function App() {
    const [name, setName] = useState("");
    console.log("App rendered!");
    return (
      <div>
        <input onChange={(e) => setName(e.target.value)} placeholder="name" />
        <Expensive />
      </div>
    );
}

// 正确做法，这个每次改变Form的name只会重新渲染Form组件
// 打印：一开始 App rendered! -》 Form component rendered! -》 expensive component rendered!
// 每次改变Form的name只会打印 Form component rendered! 不会打印App rendered! 和 expensive component rendered!
const Form = () => {
  console.log("Form component rendered!");
  const [name, setName] = useState("");
  return <input onChange={(e) => setName(e.target.value)} placeholder="name" />;
};
function App() {
    console.log("App rendered!");
    return (
    <div>
      <Form /> 
      <Expensive />
    </div>
  );
}

```

## 计算量大的组件通过组件参数传递传递

```tsx
  // 错误做法，和上一个demo同理
  const [backgroundColor, setBackgroundColor] = useState("white");

  return (
    <div style={{ backgroundColor }}>
      <input onChange={(e) => setBackgroundColor(e.target.value)} />
      <Expensive />
    </div>
  );

// 正确做法 ?:vue的插槽
// 打印：一开始 app component rendered! -》BgProvider component rendered! -》 expensive component rendered!
// 修改backgroundColor，只打印 BgProvider component rendered!

const BgProvider = ({ children }) => {
  let [backgroundColor, setBackgroundColor] = useState("white");
  console.log("BgProvider component rendered!");

  return (
    <div style={{ backgroundColor }}>
      <input onChange={(e) => setBackgroundColor(e.target.value)} />
      {children}
    </div>
  );
};

function App() {
  console.log("App component rendered!");
  return (
    <BgProvider>
      <Expensive />
    </BgProvider>
  );
}
```

## 缓存组件 useMemo vs React.memo

```jsx
import React, { useContext, useMemo, useState } from 'react';

// 定义主题上下文
const ThemeContext = React.createContext();

function Consumer() {
  console.log('Consumer render');
  const { color, background } = useContext(ThemeContext);
  return <div style={{ color, background }}>消费者</div>;
}

function Son() {
  console.log('Son render');
  return <Consumer />;
}

function ThemeProvider({ children }) {
  const [theme, setTheme] = useState({ color: '#ccc', background: '#pink' });

  const handleThemeChange = () => {
    setTheme({ color: '#fff', background: '#blue' });
  };

  const contextValue = useMemo(() => theme, [theme]);

  return (
    <ThemeContext.Provider value={contextValue}>
      {children}
      <button onClick={handleThemeChange}>切换主题</button>
    </ThemeContext.Provider>
  );
}
// const Son = React.memo(()=> <Consumer />)  缓存Son组件方案一，默认对子组件 props 进行浅比较处理。
export default function App() {
  const memoizedSon = useMemo(() => <Son />, []); // 缓存Son组件方案二，利用React 本身对 React element 对象的缓存。
  return (
    <ThemeProvider>
      {memoizedSon}
    </ThemeProvider>
  );
}
// 一开始
// Son render
// Consumer render
// 点击切换主题
// 只是打印 Consumer render
```

**`React.memo` vs `useMemo`**

`React.memo` 用于对组件进行浅层比较，适用于优化组件的渲染性能，特别是当组件的渲染代价较高时，避免不必要的重新渲染。`React.memo` 适用于优化组渲染性能，特别是当组件的渲染代价较高时。`React.memo` 接受两个参数，第一个参数 Component 原始组件本身，第二个参数 compare 是一数,第二个参数不存在时，会浅层比较处理 props。

- `useMemo` 则是一个 `Hook`，用于缓存计算结果，避免重复计算。`useMemo` 接受两个参数，第一个参数是一个函数，用于进行计算；第二个参数是一个依赖数组，只有依赖数组中的值发生变化时，才会重新计算并返回计算结果。因此，`useMemo` 适用于优化计算性能，特别是当计算代价较高时。

- React.memo 和 `useMemo` 的作用虽然不同，但它们都是基于浅层比较的，因此只能比较 `props` 或依赖数组中的简单类型值。如果 `props` 或依赖数组中包含复杂类型值（如对象或数组），则需要自行实现比较逻辑，否则可能会导致不正确的渲染结果或缓存结果。

**`useMemo` vs `useCallback`**

- `useCallback` 用于缓存函数，记忆的是函数内存地址。而 `useMemo` 用于缓存第一个函数运算结果。
