# useEffect

useEffect: 用于在组件渲染后执行一些操作，可以模拟 componentDidMount、componentDidUpdate 和 componentWillUnmount 生命周期

## 简单用法

```tsx
const [number, setNumber] = useState(0);
  // 没有依赖
useEffect(() => {
  console.count("useEffect");
  document.title = `You clicked ${number} times`;
},);
  // 正确
useEffect(() => {
  console.count("useEffect");
  document.title = `You clicked ${number} times`;
}, [number]);
console.count("组件渲染！"); 

```

## `useMemo` VS `useEffect`

```tsx
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
    console.log(`user改变, useEffect`);
  }, [user]);

  //  理解watch成直接监听state属性
  useEffect(() => {
    console.log(`state改变, useEffect`);
  }, [state.name, state.selected]);
```

## 清理计时器

```tsx
  const [number, setNumber] = useState(0);

  //没有一秒更新一次、没有清理定时器、
  useEffect(() => {
    console.log("useEffect")
    setInterval(() => {
      setNumber(number + 1);
    }, [1000]);
  }, [number]);

  // 没有清除定时器，可能导致性能问题和内存泄漏问题
  useEffect(() => {
    console.log("useEffect")
    setInterval(() => {
      setNumber((prev) => prev + 1);
    }, [1000]);
  }, []);

  //正确用法，number一更新就会重新
  useEffect(() => {
    console.log("useEffect 开始！");
    const interval = setInterval(() => {
      setNumber((prev) => prev + 1);
    }, [1000]);
    return () => { // 清理上一次useEffect
      clearInterval(interval);
    };
  }, []);
  console.count("组件渲染！");  
  // 结果 组件渲染！ -》 useEffect 开始！（只执行一次） -》组件渲染！-》组件渲染！
  return <div>{number}</div>;
```

## `useeffect`回调函数

```tsx
  const [toggle, setToggle] = useState(false);

  useEffect(() => {
    console.log("effect执行");
    return () => {
      console.log("执行effect之前我先清除上一次的结果");
    };
  }, [toggle]);
  // 结果 执行effect之前我先清除上一次的结果-》effect执行
  return (
    <div>
      <button onClick={() => setToggle(!toggle)}>Toggle</button>
    </div>
  );

```

## 接口请求和取消

不然会导致进入下一个页面上一个页面还在请求

```tsx
  const [user, setUser] = useState({});
  const id = useLocation().pathname.split("/")[2];
  // 请求和清除，
  useEffect(() => {
    let unsubscribed = false;
    fetch(`https://jsonplaceholder.typicode.com/users/${id}`)
      .then((res) => res.json())
      .then((data) => {
        if (!unsubscribed) {
          setUser(data);
        }
      });

    return () => {
      console.log("cancelled!")
      unsubscribed = true;
    };
  }, [id]);

  //FETCH 和 ABORT
  useEffect(() => {
    const controller = new AbortController();
    const signal = controller.signal;

    fetch(`https://jsonplaceholder.typicode.com/users/${id}`, { signal })
      .then((res) => res.json())
      .then((data) => {
        setUser(data);
      })
      .catch((err) => {
        if (err === "AbortError") {
          console.log("Request canceled!");
        }else{ 
          // TODO:handle error
         }
      });

    return () => {
      controller.abort();
    };
  }, [id]);

  //Axios和取消请求
  useEffect(() => {
    const cancelToken = axios.CancelToken.source();
    axios
      .get(`https://jsonplaceholder.typicode.com/users/${id}`, {
        cancelToken: cancelToken.token,
      })
      .then((res) => {
        setUser(res.data);
      })
      .catch((err) => {
        if (axios.isCancel(err)) {
          console.log("Request canceled!");
        } else {
          //TODO:handle error
        }
      });
    return () => {
      cancelToken.cancel();
    };
  }, [id]);



```

## `useEffect` vs`useLayoutEffect`

`useEffect`会在组件渲染后异步执行，因此它不会阻塞组件渲染。这也意味着，如果在`useEffect`中执行的操作依赖于DOM的状态，则可能会出现闪烁或延迟的情况，因为组件已经渲染出来了，但是`useEffect`尚未执行完毕。

`useLayoutEffect`与`useEffect`非常相似，但是它会在DOM更新前同步执行，这意味着它会在组件渲染之前被执行。如果您需要在更新DOM之前同步地执行某些操作（例如，测量元素的大小或位置），则应该使用`useLayoutEffect`。但是，由于它是同步执行的，因此如果操作耗时很长，它可能会阻塞组件渲染，从而导致用户看到延迟。`useLayoutEffect`有点类似vue `created`

useLayoutEffect可以用于在DOM更新前同步执行某些操作，常见的应用场景包括：

- 测量DOM元素的大小或位置。由于DOM元素的大小和位置可能会影响组件的布局和样式，因此我们需要在DOM更新前同步地获取这些信息。

- 执行对DOM的操作，例如滚动到指定位置或聚焦输入框。这些操作需要在DOM更新前执行，以确保用户看到的是正确的状态。

- 处理一些需要同步执行的副作用，例如更新本地存储中的数据。由于这些操作需要同步执行，因此我们需要使用useLayoutEffect。
